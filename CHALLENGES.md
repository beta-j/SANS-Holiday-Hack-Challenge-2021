# CHALLENGE 1 - ExifMetadata #

## OBJECTIVE : ##
>HELP!  That wily Jack Frost modified one of our naughty/nice records, and right before Christmas!  Can you help us figure out which one?  We’ve installed `exiftool` for your convenience!

#  

## PROCEDURE : ##

This is a super quick one just ran the following command to filter out any exif entries containing the word “Jack” and to display the preceding 40 lines to get the document’s file name.
```
:~$ exiftool . | grep -B40 Jack
```
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

#  
#  
#  
