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

