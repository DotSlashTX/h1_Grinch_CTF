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
