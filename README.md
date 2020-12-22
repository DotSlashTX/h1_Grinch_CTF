**Flag 1 : flag{48104912-28b0-494a-9995-a203d1e261e7}**  
**Endpoint -> /robots.txt**  
**Tools used : nmap**  
  
Just a simple nmap scan with ``nmap -sC -sV hackyholidays.h1ctf.com``, result showed ```http-robots.txt: 1 disallowed entry```, looked into hackyholidays.h1ctf.com/robots.txt and the flag can be found there. 

--------------------------------------------------------------------------------

**Flag 2 : flag{b7ebcb75-9100-4f91-8454-cfb9574459f7}**  
**Endpoint -> /s3cr3t-ar3a**  
**Tools used: Developer Tools (Firefox)**  
  
The page said COME BACK TOMORROW when visited on Day 1, definitely a place to look, webpage has a heading that says Page Moved and "I've moved this page to keep people out! If you're allowed access you'll know where to look for the proper page!". A possible rabbit hole to keep the competitors busy but as I have been fuzzing on Day 1 for hours and ended up into something which was right there and I missed it, my intuition says this flag must not reach sky heights of difficulty. Thinking like a beginner asks me to go through the source code. Source code had nothing so special, fired up Developer Tools in Firefox (F12 key), inspected the elements, found ``"info="flag{b7ebcb75-9100-4f91-8454-cfb9574459f7}" next-page="/apps">"`` which also has the hint for possible Day 3 flag that says "next-page="/apps"", which at the moment gives a 404.   

--------------------------------------------------------------------------------
  
**Flag 3 : flag{b705fb11-fb55-442f-847f-0931be82ed9a}**  
**Endpoint -> /apps**  
**Tools used: Developer Tools (Firefox)**  
  
Request for the first person named "Tea Avery" had a ``"id"`` parameter that was set equals to ``"eyJpZCI6Mn0="``, this parameter looks like to be sufficed with a base64 string, on decoding, we found that it translates to ``{"id":2}``, the last person on the web application named "Charlton Dillon" had similar parameter, base64 encoded again that translates to {"id":17}, so I modified the request of ``{id:17}`` to ``{"id":18}`` that was encoded to be ``"eyJpZCI6MTh9"``, but this request dropped a 404 with response as "Entry not found".
Let's try encoding ``{"id":1}`` as base64 and using it as the parameter, on making a POST request with id set to ``"eyJpZCI6MX0="``, we get a 200 response code with the flag in the response panel. 

-------------------------------------------------------------
  
**Flag 4 : flag{972e7072-b1b6-4bf7-b825-a912d3fd38d6}**  
**Endpoint: /swag-shop/api/user?=**  
**Tools used: Developer Tools (Firefox), ffuf, dirb**  
  
This challenge was found in the ```/apps``` endpoint, on clicking the button called Swag Shop, we are redirected to ```/swag-shop```, we have a couple of products with different prices and a Purchase button. On purchasing any item, we are requested to login, seems like we need to bruteforce the logins but before advancing any further, let's do some directory bruteforcing for which I used dirb agianst the common.txt wordlist, and we find an endpoint called ```/api```, on bruteforcing further on /api endpoint, we find 3 endpoints which are ```/sessions```, ```/stock``` and ```/user```, on inspecting the ```/sessions``` endpoint, we get a good amount of JSON data that has an object called "sessions:" followed by another array of objects that range from 0 to 7, and all the strings inside those objects look like base64 encoded strings, on decoding 0th string, we get ``{"user":null,"cookie":"<some_huge_string>"}``, on decoding 1st string we again get {"user":null,"cookie":"<some_huge_string>"} but on decoding 2nd string, we get some different data that says ``{"user":"C7DCCE-0E0DAB-B20226-FC92EA-1B9043","cookie":"some_huge_string"}``, so we have a user field but now the cookie parameter has string that has another "=" at the end of it unlike previous strings, maybe a double encoded string using base64, on decoding we get some data ``4548292f7d6624b1a42f74d11a48313860a5ada174b8daa735526c489046cbab67a1acd7b0fa987d9ed91d99ad5a6222ffc36c047899fb8f6c9e48ba2206ed16`` that looks like MD5 and also much like a cookie, we can use this data for further exploitation. At first I made a POST request to the ``/swag-shop`` endpoint with additional cookie headers ``(Set-Cookie: user="C7DCCE-0E0DAB-B20226-FC92EA-1B9043",cookie="4548292f7d6624b1a42f74d11a48313860a5ada174b8daa735526c489046cbab67a1acd7b0fa987d9ed91d99ad5a6222ffc36c047899fb8f6c9e48ba2206ed16")``
A successful request with code 200, tried purchasing but failed, went on to bruteforcing, failed also stopped as someone on hacker101 server #hacky-holidays channel recommended not to waste time there. Last thing that I was left with was fuzzing, used ffuf against the ``/user?FFUF=C7DCCE-0E0DAB-B20226-FC92EA-1B9043&cookie=<long_cookie_text>`` with a parameter wordlist from Seclists and that gave us the parameter ``"uuid"``. On making a request with the URL ``https://hackyholidays.h1ctf.com/swag-shop/api/user?uuid=C7DCCE-0E0DAB-B20226-FC92EA-1B9043&cookie=<long_cookie_text>`` we get the flag with more information in the JSON data like 
``"username":"grinch","address":{"line_1":"The Grinch","line_2":"The Cave","line_3":"Mount Crumpit","line_4":"Whoville"}``

--------------------------------------------------------------------------------
  
**Flag 5: flag{2e6f9bf8-fdbd-483b-8c18-bdf371b2b004}**  
**Endpoint : /my_secure_files_not_for_you.zip**  
**Tools used : A handmade zip bruteforcer, ffuf, BurpSuite, Seclists, rockyou.txt Developer Tools (Firefox)**  
  
On testing the web application, we noticed that it gives us a weird warning when checking the credentials, usually in applications like Facebook, Twitter etc we get to see, "Incorrect username or password" but here the warning says, "Invalid username", according to which this application first checks if username is correct or not, then it goes on to match the passwords. So on intercepting the request with username and password as "test_user" and "test_pass" respectively, we get "username=test_user&password=test_pass" in request body. In order to make this data usable to iterate through a wordlist in ffuf, we change the username to FUZZ and password remains unchanged in the Request Body, we then have to save this data as a .raw file and then feed to ffuf. Iterating our raw file through xato-net-10-million wordslist using the command  
``./ffuf -request /home/user1/Desktop/ow.raw -w /home/user1/Downloads/SecLists/Usernames/xato-net-10-million-usernames.txt -fr "Invalid Username"``  
, we hit ``"access"`` as a username. On using "access" as a username, we get the warning called "Invalid Password"! Boom. Now we just need to do the same with the ``password``, using rockyou.txt,  but that doesn't work as all the requests are dropping HTTP Response Code 200, we never get to know what the actuall password is, I realised this after letting the wordlist iterate for hours, went back to basic, used Burpsuite Intruder and launched a sniper attack on the password parameter in the Response Body using same rockyou.txt and after a couple of 100 requests, we hit a 302 response code for the string ``"password"``.  
On using "access" and "computer" as username and password repectively, we get access with yet another hurdle that warns with 
``"No Files To Download"``, definitely some hint here. On checking the headers, we will find ``Cookie
	securelogin=eyJjb29raWUiOiIxYjVlNWYyYzlkNThhMzBhZjRlMTZhNzFhNDVkMDE3MiIsImFkbWluIjpmYWxzZX0%3D``, boom, hello base64 my old friend, this translated to ``{"cookie":"1b5e5f2c9d58a30af4e16a71a45d0172","admin":false}``, on setting admin key to ``true`` and encoding it as base64, feeding it to the the Cookie header keeping in mind of adding that ``%3D`` at the end and making a GET request, we get a file called "my_secure_files_not_for_you.zip". Downloading and extracting which, we get to know it is password protected, I had an old and obsolete zip bruteforcing tool I wrote for my brother, used the same with rockyou.txt and found the password, ``hahahaha``. The zip file has a picture called xxx.png and a flag.txt file which has the flag.  
  
-------------------------------------------------------------------------------
  
**Flag 6: flag{18b130a7-3a79-4c70-b73b-7f23fa95d395}**  
**Endpoint: /?template=secretsecretadminadmin.php.phpadminadmin.php.php**  
**Tools used: Developer Tools (Firefox),ffuf, Wappalyzer, php (interactive mode)**  

On using Wappalyzer extension, we get to know that programming language used here is PHP, next on directory bruteforcing using ffuf, we get to know about a page called ``index.php`` which shows nothing but the diary entries. on using ``index.php`` as a query to the parameter ``/?template=``, we get a HTTP response code 200 and in the response payload, we get a PHP code that is:  
```php
<?php
if( isset($_GET["template"])  ){
    $page = $_GET["template"];
    //remove non allowed characters
    $page = preg_replace('/([^a-zA-Z0-9.])/','',$page);
    //protect admin.php from being read
    $page = str_replace("admin.php","",$page);
    //I've changed the admin file to secretadmin.php for more security!
    $page = str_replace("secretadmin.php","",$page);
    //check file exists
    if( file_exists($page) ){
       echo file_get_contents($page);
    }else{
        //redirect to home
        header("Location: /my-diary/?template=entries.html");
        exit();
    }
}else{
    //redirect to home
    header("Location: /my-diary/?template=entries.html");
    exit();
}
```
  
So in a nutshell, the code checks if ``"template"`` parameter is being fulfilled with some query or not, it then grabs whatever our query is, removes certain symbols and numerals in the query. Later in case if we feed the parameter with ``admin.php``, it gets replaced with a blank. Also as already told in the comments that `` //I've changed the admin file to secretadmin.php for more security!``, we know that ``admin.php`` is replaced as ``secretadmin.php``.  
On running the code locally using the command ``php -i``, for now we will only work on the code that hold the logic to sanitize user input, also I have tweaked the code a little bit to give my payload as input and actually see what happens to my inputs.  
```php
<?php
	$page = "payload_here" // this will be used to store and test payloads
	$page = preg_replace('/([^a-zA-Z0-9.])/','',$page);
	$page = str_replace("admin.php","",$page);
	echo $page = str_replace("secretadmin.php","",$page);
```
Here is a small list of testcases that I used as payload and next to it is what happens to the payload in order to demonstrate how strings get sanitized:  
```
secretadmin.php -> secret
admin.php       -> 
secretadminadmin.php -> secretadmin
secretadminadmin.php.php -> secret
secretsecretadminadmin.php.php ->  secretadminsecret.php 
```
The most apt payload that I had crafted which translated to ``secretadmin.php`` was ``secretsecretadminadmin.php.phpadminadmin.php.php``. On using this payload as an input to the ``/?template=`` parameter, we hit a 200 HTTP Response Code along with the flag.  

-------------------------------------------------------------------------------  
  
**Flag 7: flag{5bee8cf2-acf2-4a08-a35f-b48d5e979fdd}** 
**Endpoint: /preview**
**Tools used: Developer Tools (Firefox), URL Encoder/Decode (https://meyerweb.com/eric/tools/dencoder/), ffuf**

When we check the default campaign called `Guess What`, we get to see that it has 3 fields that are Name, Subject and Markup. the markup data says:  
```
{{template:cbdj3_grinch_header.html}} Hi {{name}}..... Guess what..... <strong>YOU SUCK!</strong>{{template:cbdj3_grinch_footer.html}}
```  
When we click the little `Preview` button, the results are something like `Hi Bob..... Guess what..... YOU SUCK!`. This reminds me of Jinja Templates in Django where we used to fetch data from somewhere and print it out within the curly braces (`{{some_data}}`). Here the name that was previewed as Bob is also being fetched from somewhere and the `{{template:cbdj3_grinch_header.html}` that is the Header that says `Message From The Grinch!` is also getting fetched from somewhere. To begin with discovery, I used a tool called ffuf with dirb wordlist. The results give out some directories and one of them is `/templates`. On visiting the link https://hackyholidays.h1ctf.com/hate-mail-generator/templates/ , we get to see three html pages in which two of them (`cbdj3_grinch_header.html` and `cbdj3_grinch_footer.html`) were used in the previous campaign but one seems strange and useful which is `38dhs_admins_only_header.html`.  
To make use of the same, we visit the home page of the challenge. Hit `Create New` button and in the markup feild which is already fulfilled with `Hi {{name}} .... `, we add {{38dhs_admins_only_header.html}}. On previewing which, we get to see a warning that says `Hello Alice ....38dhs_admins_only_header.html `. Okay that means we cannot see the file but something here is wrong with the name. The previous campaign was showing `Bob` as a name but in this new campaign, we get to see `Alice`. To check if we can make sense out of the request, we toggle on the Developer Tools, refresh the page and check the Headers. Here response body says `preview_markup=Hello+%7B%7Bname%7D%7D+....38dhs_admins_only_header.html++&preview_data=%7B%22name%22%3A%22Alice%22%2C%22email%22%3A%22alice%40test.com%22%7D`. On decoding this using meyerweb.com, we get to see `preview_markup=Hello {{name}} ....38dhs_admins_only_header.html  &preview_data={"name":"Alice","email":"alice@test.com"}`. Now this makes sense, there are two dictionaries, one is name that has `Alice` as a key, other is email that has `alice@test.com` as a key. Let's check if the email given in the dictionary is being printed out in the preview or not. So in the markup area, we give `Hello {{name}} ....38dhs_admins_only_header.html {{email}}` and after previewing it, we get the result that says `Hello Alice ....38dhs_admins_only_header.html alice@test.com`. Alright so that means the name and email are being fetched out of this dictionary. We also know the name of the inaccessible file and it's template which is `template`, just like for the header (`cbdj3_grinch_header.html`)  and footer (`cbdj3_grinch_footer.html`). Now this time, we copy our request body, decode it and change it to `preview_markup=Hello {{name}} ....38dhs_admins_only_header.html  &preview_data={"name":"{{template:38dhs_admins_only_header.html}}","email":"alice@test.com"}`, encode and send as request body, we get `Hi <flag> alice@test.com` in the response.  
  
-------------------------------------------------------------------------------  
  
**Flag 8: flag{677db3a0-f9e9-4e7e-9ad7-a9f23e47db8b}**
**Endpoint: /forum/3**
**Tools used: ffuf, Seclists, https://crackstation.net/ , others(OSINT, common sense)**

Before beginning with the see through of the web application, I decided to begin with directory and endpoint discovery using ffuf with Seclists wordlist. On visiting the challenge, we get to see two sections, `General` and `Admin` respectively. `General` has two posts, `Christmas!!!`, which has 1 post and `Nice Things To Do` which has no posts. On visiting `Christmas!!!`, we get a comment that says ` 	
Why I hate Christmas` on visiting which, we get to see two users, one is `grinch` and other is `max`. Getting back to the ffuz scans, we have 4 endpoints that are `1`,`2`,`login` and `phpmyadmin`. On visiting `phpmyadmin` endpoint, we get to see a login bruteforcing which for a couple of minutes, say 30-45 with `grinch`,`admin` and `max` as the usernames, I gave up as passwords generally don't take that long to crack keeping in mind previous challenges. Same had happened while bruteforcing the `login` endpoints, I gave up as it took way too long. Meanwhile in the hacker101 Discord server, one of the solver said something about the source code. This bugged me and I began looking for more directories, meanwhile I had an intuition about that this has something to do with GitHub. Went straight to nahamsec's GitHub account and found nothing so juicy in his repositories or commits. But Adam's GitHub account had a commit on Dec 7 this year which was to https://github.com/Grinch-Networks/forum . Awesome! After cloning the repository and looking for all possible vulnerable code, ended up finding nothing. But in the commits I found a commit with the comment, `Initial Commit`. On looking at `models/Db.php` Line 134, we get to see:
```php
self::$read = new DbConnect( false, 'forum', 'forum','6HgeAZ0qC9T6CQIqJpD' );
```
Here `6HgeAZ0qC9T6CQIqJpD` seems like some credential which does not looks like a encrypted string with some traditional algorithms like Base64 of MD5, on trying this as a password everywhere on the `login` and `phpmyadmin` endpoint, we get nothing. On reading the code again, we get to see a keyword `forum` which can be a possible username. And we hit success on using `forum` as a username and `6HgeAZ0qC9T6CQIqJpD` as the password. After logging in, we see a database named `forum` with 4 tables, `comment`, `post`, `session` and `user` respectively. Here first three throw and error which says `Error reading database encoding...` but the `user` table has two entries, with passwords. 1 is `grinch` with password `35D652126CA1706B59DB02C93E0C9FBF` and other is `max` with password `388E015BC43980947FCE0E5DB16481D1`. These password strings seem to be MD5, on decoding them on Crackstation, we get `BahHumbug` for `grinch` but nothing for `max`
