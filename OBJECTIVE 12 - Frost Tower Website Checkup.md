# OBJECTIVE 12 - Frost Tower Website Checkup #

## OBJECTIVE : ##
>Investigate [Frost Tower's website for security issues](https://staging.jackfrosttower.com/). [This source code will be useful in your analysis](Assets/frosttower-web.zip). In Jack Frost's TODO list, what job position does Jack plan to offer Santa? Ribb Bonbowford, in Santa's dining room, may have some pointers for you. 
#  

## HINTS: ##
<details>
  <summary>Hints provided for Objective 12</summary>
  
>-  When you have the source code, API documentation becomes [tremendously](https://www.npmjs.com/package/express-session) [valuable](https://github.com/mysqljs/mysql).
</details>

#  

## PROCEDURE : ##

### Getting Past the Splash Page ###

We start this challenge with access only to a “Coming Soon” splash-page with a text box that allows us to submit an email address.  From the hints I can guess that this objective will most likely involve SQLi so any fields that accept user input will be of particular interest, unfortunately the input to the text box is being validated in this case.  
I used Atom to load up the source code for the whole project and this allowed me to search through all the files and compare more easily.  I noticed that the “Coming Soon” splash-page had a hidden variable called csrf with a seemingly randomly generated token value.  Other pages on the website check for this token to ensure that they are not accessed directly and that you can only visit them from other places on the website. 
By searching for the keyword csrf in the source code I noticed that there is one page that does not include this variable and that’s /testsite. In fact, I was able to enter the URL https://staging.jackfrosttower.com/testsite directly to my browser and access the staged website.

Auth Bypass
Once I was able to access the Jack Frost Tower website, I had access to the ‘About Us’, ‘Services’ and ‘Contact Us’ pages but only the latter had any user input fields.  There is also a link to a ‘Dashboard login’ but this requires us to have login credentials which I don’t (yet).
It took me several hours of staring at the source code and finally ended up solving this bit completely by accident!  If you submit a contact form and then try to re-submit with the same email address you get a message saying that the email already exists.  If you then navigate to http://staging.jackfrosttower.com/dashboard you are allowed in.  From what I can understand the session.unique.ID is set as session.uniqueID = email in the /postcontact part of the code.  The /dashboard part of the code then checks to see if this variable is set before allowing access to the dashboard page.

SQL Injection
Now that I had access to the Dashboard, I was able to view details and edit some fields for the users that registered on the ‘Contact Us’ pages.   I decided to start looking for a possible SQLi entry point.  I did this by searching the code for every instance where there is a SELECT * FROM statement in the code as this could be a potential SQLi entry point.  All the website parts I have access to appear to pass on sanitised user input to the SELCT * FROM operation by using the escape()  function.  There is one exception however; the /detail page accepts a user id from the URI (eg. https://staging.jackfrosttower.com/detail/1) and passes the value on directly to detail.ejs .   I can also pass on multiple comma-separated user ids (…/detail/1,2,3) and the page lists all of them.  So, this now looks like a very likely SQLi vulnerability.
(NOTE: from here on the https://staging.jackfrosttower.com/ part of the URI will be implied for ease of legibility).
If I pass the following string, I can see a list of all entries in the uniqeucontact database.
.../detail/1,2 OR 1=1
I can also order the results by a particular column value – so this page is most definitely vulnerable to SQLi:
…/detail/1,2 OR 1=1 ORDER by email
