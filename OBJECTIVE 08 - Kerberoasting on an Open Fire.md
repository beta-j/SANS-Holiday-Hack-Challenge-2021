# OBJECTIVE 8 - Kerberoasting on an Open Fire #

## OBJECTIVE : ##
>Obtain the secret sleigh research document from a host on the Elf University domain. What is the first secret ingredient Santa urges each elf and reindeer to consider for a wonderful holiday season? Start by registering as a student on the [ElfU Portal](https://register.elfu.org/). Find Eve Snowshoes in Santa's office for hints..

#  

## HINTS: ##
<details>
  <summary>Hints provided for Objective 8</summary>
  
>-	Check out [Chris Davis' talk](https://www.youtube.com/watch?v=iMh8FTzepU4) and [scripts](https://github.com/chrisjd20/hhc21_powershell_snippets) on Kerberoasting and Active Directory permissions abuse.
>-	Learn about [Kerberoasting](https://gist.github.com/TarlogicSecurity/2f221924fef8c14a1d8e29f3cb5c5c4a) to leverage domain credentials to get usernames and crackable hashes for service accounts.
>-	There will be some `10.X.X.X` networks in your routing tables that may be interesting. Also, consider adding `-PS22,445` to your `nmap` scans to "fix" default probing for unprivileged scans.
>-	[OneRuleToRuleThemAll.rule](https://github.com/NotSoSecure/password_cracking_rules) is great for mangling when a password dictionary isn't enough.
>-	[CeWL](https://github.com/digininja/CeWL) can generate some great wordlists from website, but it will ignore digits in terms by default.
>-	Administrators often store credentials in scripts. These can be coopted by an attacker for other purposes!
>-	Investigating Active Directory errors is harder without [Bloodhound](https://github.com/BloodHoundAD/BloodHound), but there are [native](https://social.technet.microsoft.com/Forums/en-US/df3bfd33-c070-4a9c-be98-c4da6e591a0a/forum-faq-using-powershell-to-assign-permissions-on-active-directory-objects?forum=winserverpowershell) [methods](https://www.specterops.io/assets/resources/an_ace_up_the_sleeve.pdf).

</details>

#  

## PROCEDURE : ##

We start off this one by registering at the [elfu registration portal](https://register.elfu.org/) and we are given a username, password and server address and instructed to `ssh` to it on port `2222`.  We are then met by a menu screen which only accepts `1` or `e` as keyboard inputs.  `1` brings up a list of grades and `e` exits the program any other keypress doesn’t appear to do anything.

After hours and hours and hours of trying different combinations, I discovered that `Ctrl+D` exits to a Python prompt – well I wish I’d tried that earlier!

From the Python prompt I was able to run the following commands to exit to a bash prompt:
```
>>> import subprocess
>>> out, err = subprocess.Popen(['bash', '-l'], env={}).communicate()
```

Now that I’ve broken out into a bash prompt,  I looked around (following the suggestions in the hint) and noticed that the routing table includes routes to another three networks; `10.128.1.0/24`, `10.128.2.0/24` and `10.128.3.0/24`.  The `172.17.0.0/16` network is the one we’re on.
I also have a look at the ARP table and find four hosts on it which are on the same network as I am.  I ran a `nmap` scan towards each of these four IPs and `172.17.0.4` attracted my attention due to the number of open ports it had including LDAP, SMB and RPC.
![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/2a1040a6-442e-4343-ae6b-fc3c5422c845)

Using `rpcclient`, I was able to connect to `172.17.0.4` with the same credentials provided by the Elf-U registration page
![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/535ce633-9add-4d14-b6b0-736254065460)

At this point I Googled some ways of enumerating a server using `rpcclient` and I was able to get some interesting information.

By running ``rpcclient $> enumdomusers`` we get a list of users on the domain which include a number of particularly interesting ones:
-  ``user: [test] rid:[0x60f]``
- ``user:[Administrator] rid: [0x1f4]``
-  ``user:[admin] rid:[0x3e8]``
- ``user:[elfu_admin] rid[0x450]``
-  ``user:[elfu_svc] rid:[0x451]``
-  ``user:[remote_elf] rid:[0x452]``

I could also run ``rpcclient $> enumdomgroups`` to get a list of domain groups
![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/3874ac46-5c04-4431-a253-5001eb455c5f)

and ``rpcclient $> get dompwinfo`` to find out what kind of password requirements are enforced on the domain.  From this I learned that the domain requires a **minimum password length of 7 characters**.  Something tells me this might be useful later on in this challenge...

![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/72cfd07c-b4f6-4a24-b1da-0e1f2be9a068)

Running `nmap` towards the `10.x.x.x` networks with the `-PS22,445` switch as suggested in the hint gives us two hosts that are particularly interesting: `10.128.3.30` and `10.128.1.53`.  The latter’s hostname is `hhc21-windows-dc.c.holidayhack2021.internal` and by connecting to it using `rpcclient` and running ``rpcclient $> querydominfo``, I confirmed that the Server’s role is `ROLE_DOMAIN_PDC` – presumably that stands for **P**rimary **D**omain **C**ontroller.  Similarly, I was able to determine that `10.128.3.30` is the BDC – **B**ackup **D**omain **C**ontroller.

![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/13e71f00-3f20-4403-b767-931b5fc6f2e8)

Now that I found the domain controller, I should be able to use [Kerbroasting](https://attack.mitre.org/techniques/T1558/003/) to get a password hash.  I start by uploading the script `GetUserSPNs.py`[^1]  using `scp`:
```
> Scp -P 2222 GetUserSPNs.py username@grades.elfu.org:
```

I can now run the script on the `grades.elfu.org` machine which is directly connected to the ELFU domain and I can use my own credentials for this:
```
$ Python3 GetUserSPNs.py -outputfile spns.txt -dc-ip 10.128.1.53 elfu.local/username:'Password!' -request
```

Once the script ran, I got a text file; `spns.txt` which contains a hash for user `elfu_svc` and I could copy this back to my  local machine using `scp` once again:
```
> Scp -P 2222 username@grades.elfu.org:/home/username/spns.txt spns.txt
```

For the next step I need to crack the hash using [Hashcat](https://hashcat.net/hashcat/).  For this I’ll need a suitable wordlist and a mangling ruleset.  The hints suggest using `Cewl` to generate the wordlist and `OneRuleToRuleThemAll.rule`[^2]  as a mangling rule.  From the enumeration I did earlier on the domain connected machines, I know that the domain password rules expect a password with a minimum length of 7 characters.  So, I can use `cewl` to scrape `https://register.elfu.org/register` for words of suitable length.  I also include the `–with-numbers` switch as recommended by the hints (by looking at the page source we see the names of karaoke groups which look a bit like potential passwords and include numbers in them).
![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/4a478247-50e0-46d3-84ad-045b65308d12)
```
$ cewl -m 7 -w custom_wordlist.txt –with-numbers https://register.elfu.org/register
```

Now I run `hashcat` with the generated wordlist:
```
$ hashcat -m 13100 -a 0 spns.txt –potfile-disable -r OneRuleToRuleThemAll.rule –force -O -w 4 –opencl-device-types 1,2 custom_wordlist.txt
```

Once hashcat finishes cracking the hash I find out that the user `elfu_svc` has password `Snow2021!` 😊

I can use these new credentials now to connect to the BDC server and see what shares are available on it:
```
$ rpcclient -U elfu_svc 10.128.3.30 -c netshareenum
```
![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/91857eb5-5daa-4782-8e4e-2bf93b9ca39d)

I see that there are 4 shares available; `netlogon`, `sysvol`, `elfu_svc_shr` and `research_dep`.
I can access all of these apart from `research_dep`.   On the other hand, `elfu_svc_shr` is the only share I have access to that has any files on it.  
I can access this share and download all the files in a tar archive:
```
$ smbclient -U elfu_svc //10.128.3.30/elfu_svc_shr
smb: \> tar c all.tar
```

Now I can untar the file and search through the contents[^3]:
```
$ tar -xvf all.tar
$ grep -l remote_elf   
```

This pointed me to a PowerShell script file called `GetProcessInfo.ps1` (can’t believe it’s actually the exact same filename as in the tutorial video) which includes some kind of hash of `remote_elf`‘s password.  
![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/a529a93a-250b-4fb0-9f1b-709b5dc708dc)

Looking through the script I can see that is actually using `remote_elf`’s credentials to connect to the DC and get the running processes.  I can therefore edit the same script using the hints provided [here](https://github.com/chrisjd20/hhc21_powershell_snippets#added-bonus-here-is-how-you-can-enter-pssession-into-a-remote-computer) to allow me to establish a PowerShell Session.  
![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/e028cee7-b23c-4fb8-9a58-f6f2cd0ed613)

Now I hope (or expect, given the hints in this challenge so far) that `remote_elf` has been given `WriteDacl` rights to a group that is of interest to us.  After some trial-and-error I find that `remote_elf` does indeed have `WriteDacl` enabled for the group `Research Department`.  This is verified by running the following at the powershell prompt:
```
$ADSI = [ADSI]"LDAP://CN=Research Department,CN=Users,DC=elfu,DC=local"
$ADSI.psbase.ObjectSecurity.GetAccessRules($true,$true,[Security.Principal.NTAccount])
```

And here we see that WriteDacl is enabled for `remote_elf`... excellent!
![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/1a6df0c0-4ff4-48e2-9701-f30363b6f70f)

So now I can use `remote_elf`’s privileges to give `Generic All` access to our username provided by the `register.elfu.org` system.  For this I can use [this script](https://github.com/chrisjd20/hhc21_powershell_snippets#in-the-below-example-the-genericall-permission-for-the-chrisd-user-to-the-domain-admins-group-if-the-user-your-running-it-under-has-the-writedacl-permission-on-the-domain-admins-group).  The script runs with a number of errors, but when I [check permissions](https://github.com/chrisjd20/hhc21_powershell_snippets#you-can-read-the-dacl-of-an-ad-group-object-using) again I see that my user now has `GenericAll` rights.
![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/3bb62529-b447-4609-b9a6-77cdde7b2606)

This means that I should now be able to add myself to the `ResearchDepartment` group.  To do this I run [this script](https://github.com/chrisjd20/hhc21_powershell_snippets#in-the-below-example-the-genericall-permission-for-the-chrisd-user-to-the-domain-admins-group-if-the-user-your-running-it-under-has-the-writedacl-permission-on-the-domain-admins-group ) and I can now verify that my user; `fmcygawtjd` has indeed been added to the `ResearchDepartment` group.
![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/d94a8ea3-df2a-40b3-af4d-873fad5f01fa)

Now I am able to access `//10.128.3.30/research_dep` with my own username and password and there I (**FINALLY**) find a nice file waiting for me: [`SantaSecretToAWonderfulHolidaySeason.pdf`](Assets/SantasSecretToAWonderfulHolidaySeason.pdf).  It would have been nice if it had been a simple txt file I guess – but anyway – I can use `scp` to transfer the file to my PC. 

But `scp` gives me a *“TERM Environment Variable Not Set”* Error and helpfully suggests I use a bash shell on the remote pc.  So I switch to bash by running ``$ chsh -s /bin/bash`` then exiting back to the python prompt and calling up the bash prompt once again.  Now I was able to `scp` the document to my PC and open the pdf to find the Santa’s ingredient at the top of the list.

This was one super challenging Objective!  So many of the elements were completely new to me; rpcclient enumeration, working with AD, working with PowerShell, etc.. not to mention Kerberoasting!  But this is what keeps bringing me back to the HolidayHack Challenge year after year – the challenges are all challenging but achievable and it’s a great feeling when I get to accomplish each one 😊  
![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/413e9a0a-61b1-4d36-9ef1-dc0e414c4adb)



[^1]:[https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py )
[^2]:[https://github.com/NotSoSecure/password_cracking_rules/blob/master/OneRuleToRuleThemAll.rule](https://github.com/NotSoSecure/password_cracking_rules/blob/master/OneRuleToRuleThemAll.rule)
[^3]:I finally looked for `remote_elf` after trying a whole bunch of other keywords and then decided to start looking for some of the usernames I had enumerated earlier.
