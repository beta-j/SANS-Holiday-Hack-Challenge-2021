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

### 3. Getting Started ###
If that last objective is anything to go by, things will start getting a lot tougher now…
For this objective we are 
