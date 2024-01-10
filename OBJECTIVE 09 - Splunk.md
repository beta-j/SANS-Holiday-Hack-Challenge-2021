# OBJECTIVE 9 - Splunk!#

## OBJECTIVE : ##
>Help Angel Candysalt solve the Splunk challenge in Santa's great hall. Fitzy Shortstack is in Santa's lobby, and he knows a few things about Splunk. What does Santa call you when when you complete the analysis?
#  

## HINTS: ##
<details>
  <summary>Hints provided for Objective 9</summary>
  
>-	Sysmon network events don't reveal the process parent ID for example. Fortunately, we can pivot with a query to investigate process creation events once you get a process ID.
>-	Did you know there are multiple versions of the Netcat command that can be used maliciously? `nc.openbsd`, for example.
>-	Between GitHub audit log and webhook event recording, you can monitor all activity in a repository, including common git commands such as `git add`, `git status`, and `git commit`.
</details>

#  

## PROCEDURE : ##

### Task 1: ###
```
index=main sourcetype=journald source=Journald:Microsoft-Windows-Sysmon/Operational EventCode=1| top limit=50 CommandLine
```

**Answer:** `git status`

![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/caced254-5088-41cd-94a8-561b1a0cd544)


### Task 2: ###
```
index=main sourcetype=journald source=Journald:Microsoft-Windows-Sysmon/Operational EventCode=1 user=eddie CommandLine=git*origin*
```

**Answer:** ``git@github.com:elfnp3/partnerapi.git``

### Task 3:  ###
```
index=main sourcetype=journald source=Journald:Microsoft-Windows-Sysmon/Operational CommandLine=docker*
```

**Answer:** ``docker compose up``


### Task 4: ###
```
index=main sourcetype=github_json "alert.html_url"=*
```

**Answer:** `https://github.com/snoopysecurity/dvws-node`


### Task 5: ###
```
index=main *javascript*
```

**Answer:** `holiday-utils-js`


### Task 6: ###
**Answer:** `/usr/bin/nc.openbsd`


### Task 7: ###
```
index=main EventDescription="Process creation" ParentProcessId=6788
```

**Answer:** 6
 
### Task 8: ###
```
index=main EventDescription="Process creation" ProcessId=6788
```

**Answer:** `preinstall.sh`

![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/98f952be-7b30-434c-81c6-9eb1141fb7b4)
