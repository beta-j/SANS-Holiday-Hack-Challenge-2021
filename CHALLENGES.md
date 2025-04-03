# CHALLENGES #

## CONTENTS: ##

**[CHALLENGE 1 - ExifMetadata](#challenge-1---exifmetadata)**

**[CHALLENGE 2 - Grepping for Gold](#challenge-2---grepping-for-gold)**

**[CHALLENGE 3 - Frostvator](#challenge-3---frostvator)**

**[CHALLENGE 4 - IPv6 Sandbox](#challenge-4---ipv6-sandbox)**

**[CHALLENGE 5 - HoHo-No](#challenge-5---hoho-no)**

**[CHALLENGE 6 - Yara Analysis](#challenge-6---yara-analysis)**

**[CHALLENGE 7 - Strace Ltrace Retrace Challenge!](#challenge-7---strace-ltrace-retrace-challenge)**

**[CHALLENGE 8 - Elf Code](#challenge-8---elf-code)**

**[CHALLENGE 9 - Holiday Hero](#challenge-9---holiday-hero)**
#  
#  
#  

# CHALLENGE 1 - ExifMetadata #

## OBJECTIVE : ##
>HELP!  That wily Jack Frost modified one of our naughty/nice records, and right before Christmas!  Can you help us figure out which one?  We’ve installed `exiftool` for your convenience!

#  

## PROCEDURE : ##

This is a super quick one just ran the following command to filter out any exif entries containing the word “Jack” and to display the preceding 40 lines to get the document’s file name.
```
:~$ exiftool . | grep -B40 Jack
```

[back to top](#contents)
#  
#  
#  

# CHALLENGE 2 - Grepping for Gold #

## OBJECTIVE : ##
>Howdy howdy!  Mind helping me with this homew- er, challenge?
>Someone ran `nmap –oG` on a big network and produced this `bigscan.gnmap` file.  The `quizme` program has the questions and hints, and incidentally, has NOTHING to do with an Elf University assignment. Thanks!

#  

## HINTS: ##
<details>
  <summary>Hints provided for Objective 13</summary>
  
>-	Check [this](https://ryanstutorials.net/linuxtutorial/cheatsheetgrep.php) out if you need a grep refresher.

</details>

  
## PROCEDURE : ##

>-	**Q:** What port does 34.76.1.22 have open?
```
elf@e49df7806f10:~$ cat bigscan.gnmap | grep 34.76.1.22
Host: 34.76.1.22 ()     Status: Up
Host: 34.76.1.22 ()     Ports: 62078/open/tcp//iphone-sync///      Ignored State: closed (999)
```
**A: 62078**

>-	**Q:** What port does 34.77.207.226 have open?
```
elf@e49df7806f10:~$ cat bigscan.gnmap | grep 34.77.207.226
Host: 34.77.207.226 ()     Status: Up
Host: 34.77.207.226 ()     Ports: 8080/open/tcp//http-proxy///      Ignored State: filtered (999)
```
**A: 8080**

>-	**Q:** How many hosts appear “Up” in the scan?
```
elf@e49df7806f10:~$ cat bigscan.gnmap | grep Up | wc -l
26054
```
**A: 26054**

>-	**Q:** How many hosts have a web port open? (Let’s just use TCP ports 80, 443 and 8080)
```
elf@e49df7806f10:~$ cat bigscan.gnmap | grep -E "(80|443|8080)/open" | wc -l
14372
```
**A: 14372**

>-	**Q:** How many hosts with status Up have no (detected) open TCP ports?
```
elf@e49df7806f10:~$ echo $((`grep Up bigscan.gnmap | wc -l` - `grep open bigscan.gnmap | wc -l`))
402
```
**A: 402**

>-	**Q:** What’s the greatest number of TCP ports any one host has open?
```
elf@e49df7806f10:~$ cat bigscan.gnmap | grep -E "(/open/tcp.*){11}" | wc -l
58
elf@e49df7806f10:~$ cat bigscan.gnmap | grep -E "(/open/tcp.*){12}" | wc -l
5
elf@e49df7806f10:~$ cat bigscan.gnmap | grep -E "(/open/tcp.*){13}" | wc -l
0
```
**A: 12**

[back to top](#contents)
#  
#  
#  
# CHALLENGE 3 - Frostvator #

## PROCEDURE : ##

Just a matter of re-arranging the available logic gates to turn the outputs on.
The following table shows the logic output for each gate available:

Input A|	Input B| AND|	OR|	NOR|	NAND|	XOR|	XNOR|
:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:
**0**|	**0**|	0|	0|	1|	1|	0|	1|
**0**|	**1**| 0|	1|	0|	1|	1|	0|
**1**|	**0**|	0|	1|	0|	1|	1|	0|
**1**|	**1**|	1|	1|	0|	0|	0|	1|

![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/88cd8def-3f5c-4a99-a0e6-e9f4b60582cb)

[back to top](#contents)
#  
#  
#  
# CHALLENGE 4 - IPv6 Sandbox #

## HINTS: ##
<details>
  <summary>Hints provided for Objective 13</summary>
  
>-	Check out [this Github Gist](https://gist.github.com/chriselgee/c1c69756e527f649d0a95b6f20337c2f) with common tools used in an IPv6 context.

</details>

  
## PROCEDURE : ##

I started by following the advice in the hint and having a look at [the Github Gist](https://gist.github.com/chriselgee/c1c69756e527f649d0a95b6f20337c2f).  The Gist suggests using ``ping6 ff02::1 -c2`` to 
>“find link local addresses for systems in your network segment”

``ff02::1`` is a special multicast address that addresses all nodes.

Surely enough, running this command returns IPv6 addresses for 3 different hosts and from `ifconfig` I can determine which one is my own host address:

![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/12649cc1-d12e-4552-bc13-5eaae6b6e2a9)
![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/8f30e088-d365-4c1a-b8c7-37f02f3950ff)

So that leaves two possible hosts:
    ``fe80::42:1cff:feb5:4f73``
    ``fe80::42:c0ff:fea8:a002``

Running `nmap` on the two hosts shows that one of them has a http service running which looks interesting:
![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/60f9b8b9-6f1b-461b-86b9-98813b72ac14)

I used `curl` with a special IPv6 notation to make a request to the web server and got a marquee message with a convenient hint:
```
~$ curl http://[fe80::42:c0ff:fea8:a002]:80/ --interface eth0
```
![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/41fcd4fc-c00f-45d9-925e-e03f8d8da7a3)

OK – so let’s follow its advice and try to connect to the service on port 9000 with `netcat`:
![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/fe04d280-2e57-4f36-936e-9a36bc840752)

That’s it – that must be the passphrase! 

[back to top](#contents)
#  
#  
#  

# CHALLENGE 5 - HoHo-No #

## OBJECTIVE: ##
>Jack is trying to break into Santa's workshop!
>Santa's elves are working 24/7 to manually look through logs, identify the malicious IP addresses, and block them. We need your help to automate this so the elves can get back to making presents!
>Can you configure Fail2Ban to detect and block the bad IPs?
> * You must monitor for new log entries in /var/log/hohono.log
> * If an IP generates 10 or more failure messages within an hour then it must be added to the naughty list by running naughtylist add <ip>    /root/naughtylist add 12.34.56.78
> * You can also remove an IP with naughtylist del <ip>        /root/naughtylist del 12.34.56.78
> * You can check which IPs are currently on the naughty list by running        /root/naughtylist list
>
>You'll be rewarded if you correctly identify all the malicious IPs with a Fail2Ban filter in /etc/fail2ban/filter.d, an action to ban and unban in /etc/fail2ban/action.d, and a custom jail in /etc/fail2ban/jail.d. Don't add any nice IPs to the naughty list!
>
>*** IMPORTANT NOTE! ***
>Fail2Ban won't rescan any logs it has already seen. That means it won't automatically process the log file each time you make changes to the Fail2Ban config. When needed, run /root/naughtylist refresh to re-sample the log file and tell Fail2Ban to reprocess it.
>
  
## PROCEDURE : ##

Since this was the first time I’d ever even heard of `Fail2ban`, I decided to start by following the [relevant KringleCon talk by Andy Smith](https\www.youtube.com\watch?v=Fwv2-uV6e5I).  The talk includes a pretty easy to follow walkthrough for creating custom filters, actions and jails in `Fail2ban` and implementing them.  Based on this, I created the following .conf files:

```
[elf_jail]
enabled = true
logpath = /var/log/hohono.log
maxretry = 10
findtime = 1h
filter = elf_filter
action = elf_action
```
```
[Definition]
actionban = naughtylist add <ip>
actionunban = naughtylist del <ip>
```
```
[Definition]
failregex = Failed login from <HOST> for .*$
Invalid heartbeat .* from <HOST>$
Login from <HOST> rejected due to unknown user name$
<HOST> sent a malformed request$
```

I restarted `Fail2ban` and refreshed the naughtylist and that’s it – mission accomplished 😊

[back to top](#contents)
#  
#  
#  

# CHALLENGE 6 - Yara Analysis #

## OBJECTIVE: ##
>HELP!!!
>This critical application is supposed to tell us the sweetness levels of our candy manufacturing output (among other important things), but I can't get it to run.
>It keeps saying something something yara. Can you take a look and see if you can help get this application to bypass Sparkle Redberry's Yara scanner?
>If we can identify the rule that is triggering, we might be able change the program to bypass the scanner.
>We have some tools on the system that might help us get this application going: `vim`, `emacs`, `nano`, `yara`, and `xxd`
>The children will be very disappointed if their candy won't even cause a single cavity.

  
## PROCEDURE : ##

If I try to run the app I get an error message saying the app is failing on `yara_rule_135`.  I can open yara rules to see that rule 135 is looking for a match to the string `candycane`.

![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/03bb7066-ace4-4318-8f76-d8b71175e2da)

Opening the app with `nano` and changing the string `candycane` to `candyc@ne` allows me to bypass yara rule 135 but I am now being blocked by `yara_rule_1056`.

![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/f067dde2-fadd-4f2c-8b9f-b582c9181db9)

To bypass this one I opened the binary as a hex file in vim using:
```
$ xxd the_critical_elf_app | vi –
```

I then found one of the hex strings defined in yara rule 1056 and modified it slightly (I only needed to change one of the strings), saved the output to a new file and made it executable (``chmod +x``).

Now I was getting stuck on `yara_rule_1732` which looks for a number of matches.  For the program to be halted it needs to match ***all*** of the following:
-	The program must contain at least 10 of the 20 defined strings
-	The program must be less than 50kB large
-	The second byte of the program (i.e. the file start offset by 1) should be `45 4c46 02` (note that the order of the octets is reversed due to endianness[^1] ).
  
![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/e48df643-0179-4f12-af3a-0fd8c768332d)

I decided that it would be easiest to change the last rule only and changed the hex from `45 4c46 02` to `45 4c46 22`.  *NOTE: changing one of the characters in the preceding `45 4c46` part would result in an error as this forms part of the standard Linux binary header that the system is expecting*.

To make the above change I used `vim` once again, saved the output to the new file, made it executable and then I could finally run it successfully without tripping over any Yara rules.

[back to top](#contents)
#  
#  
#  

 # CHALLENGE 7 - Strace Ltrace Retrace Challenge! #

## OBJECTIVE: ##
>Please, we need your help! The cotton candy machine is broken!
>We replaced the SD card in the Cranberry Pi that controls it and reinstalled the software. Now it's complaining that it can't find a registration file!
>Perhaps you could figure out what the cotton candy software is looking for...

## PROCEDURE : ##

First things first – I tried to run  ``~$ ./make_the_candy`` and got an error saying Unable to open configuration file. OK, so the program must be looking for some sort of configuration file (obviously enough).  

By running ``ltrace ./make_the_candy`` I see that the program is trying to open `registration.json`.

I therefore created an file called `registration.json` that contains a random letter (eg. `x`) and ran `ltrace` again.
```
~$ echo x > registration.json
~$ ltrace ./make_the_candy
fopen("registration.json", "r")                           = 0x555f5aa2b260
getline(0x7ffe839191e0, 0x7ffe839191e8, 0x555f5aa2b260, 0x7ffe839191e8) = 2
strstr("x\n", "Registration")                             = nil
getline(0x7ffe839191e0, 0x7ffe839191e8, 0x555f5aa2b260, 0x7ffe839191e8) = -1
puts("Unregistered - Exiting."Unregistered - Exiting.
)                           = 24
+++ exited (status 1) +++
```

I can now see from the output of the `ltrace` operation that instead of the letter `x` I placed in the file, the program is looking for the string `Registration` – so I can simply update my `registration.json` file and run `ltrace` again.
```
~$ echo Registration > registration.json 
~$ ltrace ./make_the_candy 
fopen("registration.json", "r")                           = 0x56072f26c260
getline(0x7ffc465d2a00, 0x7ffc465d2a08, 0x56072f26c260, 0x7ffc465d2a08) = 13
strstr("Registration\n", "Registration")                  = "Registration\n"
strchr("Registration\n", ':')                             = nil
getline(0x7ffc465d2a00, 0x7ffc465d2a08, 0x56072f26c260, 0x7ffc465d2a08) = -1
puts("Unregistered - Exiting."Unregistered - Exiting.
)                           = 24
+++ exited (status 1) +++
```

This time I see that the program is looking for a `:` following `Registration` and if I repeat the process one more time I see that the program needs to find the string `Registration:True` in a file called `registration.json`.

So I simply update the json file with the expected string for the program to run successfully 😊  

[back to top](#contents)
#  
#  
#  

# CHALLENGE 8 - Elf Code #

## PROCEDURE : ##
### LEVEL 3 - Don't Get Yeeted: ###
```
import elf, munchkins, levers, lollipops, yeeters, pits
lever0 = levers.get(0)
lollipop0 = lollipops.get(0)
soln = lever0.data() + 2		# get data integer the lever and increment it by 2
elf.moveTo(lever0.position)
lever0.pull(soln)
elf.moveTo(lollipop0.position)
elf.moveUp(10)
```

### LEVEL 4 - Data Types: ###
```
import elf, munchkins, levers, lollipops, yeeters, pits
lever0, lever1, lever2, lever3, lever4 = levers.get()
elf.moveLeft(2)
lever4.pull("A String")				# this lever expects a string as input
elf.moveUp(2)
lever3.pull(True)					# this lever expects a boolean object as input
elf.moveUp(2)
lever2.pull(10)					# this lever expects an integer input
elf.moveUp(2)
lever1.pull(["happy", "holidays", "everyone"])		# this lever expects a list input
elf.moveUp(2)
lever0.pull({'santa':'claus', 'jack':'frost'})		# this lever expects a dict input
elf.moveUp(2)
```

### LEVEL 5 – Conversions and Comparisons: ###
```
import elf, munchkins, levers, lollipops, yeeters, pits
lever0, lever1, lever2, lever3, lever4 = levers.get()
elf.moveTo(lever4.position)
ans4 = lever4.data() + " concatenate"		# take the lever data and add a string to the end
lever4.pull(ans4)
elf.moveTo(lever3.position)
ans3 = not(lever3.data())			# negate the lever’s Boolean data
lever3.pull(ans3)
elf.moveTo(lever2.position)
ans2 = 1 + lever2.data()			# add 1 to the lever’s integer data
lever2.pull(ans2)
elf.moveTo(lever1.position)
ans1 = lever1.data()			
ans1.append(1)				# append 1 to the end of the lever’s list
lever1.pull(ans1)
elf.moveTo(lever0.position)
ans0 = lever0.data()
ans0["strkey"] = "strvalue"			# add a key and value pair to the lever’s dict data
lever0.pull(ans0)
elf.moveUp(2)
```

### LEVEL 6 – Types and Conditionals: ###
```
import elf, munchkins, levers, lollipops, yeeters, pits
lever = levers.get(0)
data = lever.data()
if type(data) == bool:		# check if the data type is boolean
    data = not data		# if it is negate it (i.e. change true to false and false to true)
elif type(data) == int:		# check if data type is integer
    data = data * 2 		# if so, double it
elif type(data) == list:		# check if data type is a list
    for x in range(len(data)):	# if so, go through the list elements one by one… 
        data[x] += 1		# ... and increment them by 1
elif type(data) == str:		# check if data type is a string
    data = data + data		# if so append it to itself – i.e “string” becomes “stringstring”
elif type(data) == dict:		# check if data type is a dictionary
    data['a']=data['a']+1		# find the entry corresponding to index ‘a’ and increase it by 1
elf.moveTo(lever.position)		# go to the lever
lever.pull(data)			# submit the answer
elf.moveUp(2)
```

### LEVEL 7 – Up Down Loopiness: ###
```
import elf, munchkins, levers, lollipops, yeeters, pits
for num in range(3):	# Repeat three times:	 
    elf.moveLeft(3)	# this is set to 3 so that on the last run of the loop we get to the lollipop
    elf.moveUp(15)		# move up as far as the elf can go – 15 is greater than the board’s size
    elf.moveLeft(3)
    elf.moveDown(15)
```

### LEVEL 8 – Two Paths, Your Choice: ###
```
import elf, munchkins, levers, lollipops, yeeters, pits
all_lollipops = lollipops.get()
lever0 = levers.get(0)
for lollipop in all_lollipops:
    elf.moveTo(lollipop.position)  	# move from one lollipop to the next
elf.moveTo(lever0.position)  	# go to the lever
ans = lever0.data()   		# get the array from the lever
ans.insert(0,"munchkins rule")	# insert a string at index 0 of the array
lever0.pull(ans)  			# return the array as the answer
elf.moveDown(5)  			# move to the exit
elf.moveLeft(6)
elf.moveUp(5)
				#That’s 12 lines of code exactly!
```

[back to top](#contents)
#  
#  
#  

 # CHALLENGE 9 - Holiday Hero #

## PROCEDURE: ##
For this challenge we’re presented with a two-player game, which in itself is pretty cool as it encourages interaction and cooperation with other KringleCon attendees – it would be really awesome to see more of this at future KringleCons!  But KringleCon being what it is, this also presents an opportunity to hack the game and make it work with a single player (feeding in to the stereotype of hackers being loners I guess 😊 ).

The first step to this challenge was pretty obvious; by using the in-browser developer tools in Chrome I found a Cookie called `HOHOHO` and changed its value from `{“single_player”:false}` to `{“single_player”:true}`.

Next, I reloaded the iframe with the game and created a room in the game.  I looked around in the code until I spotted a number of interesting variables including one which **begged me not to change it**!  Nevertheless, in the end all I had to set was a single global variable called `single_player_mode` which when set to true in the console started the game with a virtual ‘computer’ Player 2.  Cool stuff!

![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/bfbe2a77-114b-4131-9615-265d7a69b5eb)

[back to top](#contents)

[^1]:[https://www.varonis.com/blog/yara-rules/#:~:text=uint16(0)%20%3D%3D%200x5A4D,due%20to%20endianness](https://www.varonis.com/blog/yara-rules/#:~:text=uint16(0)%20%3D%3D%200x5A4D,due%20to%20endianness)
