# OBJECTIVE 4 - Slot Machine Investigation #

## OBJECTIVE : ##
>Test the security of Jack Frost's [slot machines](https://slots.jackfrosttower.com/). What does the Jack Frost Tower casino security team threaten to do when your coin total exceeds 1000? Submit the string in the server **`data.response`** element. Talk to Noel Boetie outside Santa's Castle for help.
#  

## HINTS: ##
<details>
  <summary>Hints provided for Objective 4</summary>
  
>-	It seems they're susceptible to [parameter tampering](https://owasp.org/www-community/attacks/Web_Parameter_Tampering).
>-	Web application testers can use tools like [Burp Suite](https://portswigger.net/burp/communitydownload) or even right in the browser with Firefox's [Edit and Resend](https://itectec.com/superuser/how-to-edit-parameters-sent-through-a-form-on-the-firebug-console/) feature.

</details>

#  

## PROCEDURE : ##

The hints are very helpful here.  By opening the slot machine in Firefox and looking in the “network” tab of the developers tools we see that a `POST` request to a file called `spin` is made every time that the **spin** button is pressed on the slot machine.  The `POST` request passes on the bet amount, the number of lines and the bet size each time.

By using the **Edit & Resend** option in Firefox, I was able to edit the parameter for the bet amount to a negative value, so for every unsuccessful spin, my balance would increase instead of decrease.  

Looking at the server response it looks like someone’s not too happy about this!

![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/77886b4a-d4d8-4b71-9cb1-8bb0974214f7)

 

