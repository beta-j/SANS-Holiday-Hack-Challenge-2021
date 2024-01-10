# OBJECTIVE 13 - FPGA Programming #

## OBJECTIVE : ##
>Write your first FPGA program to make a doll sing. You might get some suggestions from Grody Goiterson, near Jack's elevator.
#  

## HINTS: ##
<details>
  <summary>Hints provided for Objective 13</summary>
  
>-  Prof. Qwerty Petabyte is giving [a lesson](https://www.youtube.com/watch?v=GFdG1PJ4QjA) about Field Programmable Gate Arrays (FPGAs). 
>-	There are [FPGA enthusiast sites](https://www.fpga4fun.com/MusicBox.html).

</details>

#  

## PROCEDURE : ##

To implement the code for this objective I ended up ignoring professor petabyte’s advice entirely and not using any rounding-up function.  I simply replicated and modified the code he explained in his presentation which he used to program a blinking LED.  
The main part of the code is the limit variable which determines how many clock cycles to count before flipping the output bit on the speaker.  This takes half the clock frequency  (i.e., in this case 0.5 * 125MHz = 62.5MHz) and divides it by the frequency requested by the user input; freq. 
freq is divided by 100, since the input is given with two decimal points included as part of the integer (e.g. 532.12Hz is given as 53212).  
The rest of the code counts down from limit with every high clock edge and flips the output bit for the speaker every time the counter hits zero…. rinse and repeat.
Full Code used may be seen here.
