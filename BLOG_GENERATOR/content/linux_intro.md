Title: The Ultimate Linux Guide for Hackers ;)
Date: 2014-11-10 4:20
Category: life
Tags: Linux, Python, Unicode, tr, ifconfig, ssh, netcat, Stripe


 Being a Linux user is, above all, a *lifestyle*. Interestingly, more and more people have been joining this community, keeping it dynamic and organic.

 Linux has been in my life since my high school years, and I'm still always inspired by the fact that it has not lost any of its appeals. There is always something new or deeper to be learned. Linux is *the* tool for those who choose to spend their excess of curiosity flying around bytes.

With the novice in mind, this blog post is my introduction to the Linux machinery, containing some of my favorite tricks. I hope you enjoy this exciting adventure of taking control of *The Machine*.


**Disclaimer I**: Keep in mind that the true way for mastering Linux is making the **man** and **help** pages in the command line your best friends:

```
$ man <COMMAND>
```


**Disclaimer II**: The Linux distributions I'm currently using are Fedora and Kali (Debian-based). You should be comfortable exploring several distributions until you find your favorite. You should aim to go beyond Ubuntu.

**Disclaimer III**: This guide is written for [bash](http://www.gnu.org/software/bash/). I encourage you to go further and search for your favorite shell.


------

# The Linux Environment



## The Linux Filesystem

* Let's start getting an idea of our system. The *Linux filesystem* is composed of several system directories locate at **/**:

```
$ ls /
```

![cyber](http://i.imgur.com/qhYyiK8.png)

* You can verify their sizes and where they are mounted with:

```
$ df -h .
Filesystem Type Size Used Avail Use% Mounted on
/dev/mapper/fedora-home ext4 127G 62G 59G 51% /home
```


* The filesystem architecture is generally divided into the following folders:

### /bin, /sbin and /user/sbin
* **/bin** is a directory containing executable binaries, essential commands used in single-user mode, and essential commands required by all system users.

* **/sbin** contains commands that are not essential for the system in single-user mode.

### /dev
* **/dev** contains device nodes, which are a type of pseudo-file used by most hardware and software devices (except for network devices).

* The directory also contains entries that are created by the **udev** system, which creates and manages device nodes on Linux (creating them dynamically when devices are found).

### /var
* **/var** stands for *variable* and contains files that are expected to be changing in size and content as the system is running.

* For example, the system log files are located at **/var/log**, the packages and database files are located at **/var/lib**, the print queues are located at **/var/spool**, temporary files stay inside **/var/tmp**, and networks services can be found in subdirectories such as **/var/ftp** and **/var/www**.

### /etc

* **/etc** is short for *et cetera* and contains the system configuration files. It contains no binary programs, but it might have some executable scripts.

* For instance, the file **/etc/resolv.conf** tells the system where to go on the network to obtain the hostname of some IP address (*i.e.* DNS).

* The **/etc/passwd** file is the authoritative list of users on any Unix system. It does not contain the passwords: the encrypted password information was migrated into **/etc/shadow**.

* The **/etc/hosts** contains a list of hostname to IP address mappings, which can be used to assign names to servers, independently of the public DNS service.

### /lib
* **/lib** contains libraries (common code shared by applications and needed for them to run) for essential programs in **/bin** and **/sbin**.

* This library filenames either start with ```ld``` or ```lib``` and are called *dynamically loaded libraries* (or shared libraries).


### /boot

* **/boot** contains the few essential files needed to boot the system.

* For every alternative kernel installed on the system, there are four files:

 * ```vmlinuz```: the compressed Linux kernel, required for booting.

 * ```initramfs``` or ```initrd```: the initial RAM filesystem, required for booting.

 * ```config```: the kernel configuration file, used for debugging.

 * ```system.map```: the kernel symbol table.

 * [GRUB](http://null-byte.wonderhowto.com/how-to/hack-like-pro-linux-basics-for-aspiring-hacker-part-21-grub-bootloader-0154965/) files can also be found here (in /boot/grub/grub.cfg)



### /opt
* Optional directory for application software packages, usually installed manually by the user.

### /tmp
* **/tmp** contains temporary files that are erased in a reboot.

### /usr

* **/usr** contains multi-user applications, utilities and data. The common subdirectories are:
 * **/usr/include**: header files used to compile applications.

 * **usr/lib**: libraries for programs in **usr/(s)bin**.

 * **usr/sbin**: non-essential system binaries, such as system daemons. In modern Linux systems, this is linked together to **/sbin**.

 * **usr/bin**: primary directory of executable commands of the system.

 * **usr/share**: shaped data used by applications, generally architecture-independent.

 * **usr/src**: source code, usually for the Linux kernel.

 * **usr/local**: data and programs specific to the local machine, things that apply to a computer rather than things provided by the operating system go.

### /proc

* **/proc** contains dynamically-created virtual filesystem that contains data about each running process and the system as a whole.

---
## /dev Specials

* There exist files provided by the operating system that does not represent any physical device, but provide a way to access special features:

 * **/dev/null** ignores everything written to it. It's convenient for discarding the unwanted output.

 * **/dev/zero** contains an *infinite* number of zero bytes, which can be useful for creating files of a specified length.

 * **/dev/urandom** and **/dev/random** contain an infinite stream of operating-system-generated random numbers, available to any application that wants to read them. The difference between them is that the second guarantees strong randomness (it will wait until enough is available) and so it should be used for encryption, while the former can be used for games.

* For example, to output random bytes, you can type:

```
$ cat /dev/urandom | strings
```



## The Kernel


* The **Linux Kernel** is the program that manages *input/output requests* from software, and translates them into *data processing instructions* for the *central processing unit* (CPU).

* To find the Kernel information you can type:

```
$ cat /proc/version
Linux version 3.14.9-200.fc20.x86_64 (mockbuild@bkernel02.phx2.fedoraproject.org) (gcc version 4.8.3 20140624 (Red Hat 4.8.3-1) (GCC) ) #1 SMP Thu Jun 26 21:40:51 UTC 2014
```

* You can also print similar system information with the specific command to print system information, ```uname```. The flag **-a** stands for all:

```
 $ uname -a
 Linux XXXXX 3.14.9-200.fc20.x86_64 #1 SMP Thu Jun 26 21:40:51 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
```

* For instance, we might be interested in **checking whether you are using the latest Kernel**. You can do this by checking whether the outputs of the following commands match:

```
$ rpm -qa kernel | sort -V | tail -n 1
$ uname -r
```

* Additionally, for Fedora (and RPM systems) you can check what kernels are installed with:

```
$ rpm -q kernel
```


---
## Processes

* A running program is called **process**. Each process has an **owner** (in the same sense as when we talk about file permissions below).

* You can find out which programs are running with the **ps** command. This also gives the **process ID** or **PID**, which is a unique long-term identity for the process (different copies of a given program will have separate PIDs).

* To put a job (process) in the background we either run it with **&** or we press CTRL-Z and then type **bg**. To bring back to the foreground, we type **fg**.

* To get the list of running jobs in the shell, we type **jobs**. Each job has a **job ID** which can be used with the percent sign **%** to **bg**, **fg** or **kill** (described below).


### ps

* To see the processes that were not started from your current session you can run:

```
$ ps x
```

* To see your processes and those belonging to other users:

```
$ ps aux
```

* To list all zombie processes you can either do:

```
$ ps aux | grep -w Z
```

or

```
$ ps -e
```


### top

* Another useful command is **top** (table of processes). It tells you which programs are using the most of memory or CPU:

```
$ top
```

* I particularly like [htop](http://hisham.hm/htop/) over the top, which needs to be installed if you want to use it.

### kill

* To stop running a command you can use **kill**. This will send a message called **signal** to the program. There are [64 different signals](http://www.linux.org/threads/kill-commands-and-signals.4423/), some having distinct meanings from *stop running*:

```
$ kill <SIGNAL or OPTION> <ID or PID>
```

* A user can kill all her process but not another user's process or processes the System is using (unless it's root).

* The default signal sent by kill is **SIGTERM** (15), telling the program that you want it to quit. This is just a request, and the program can choose to ignore it. **SIGHUP** (1) is a less secure way of killing the process.

* The signal **SIGKILL** is mandatory and cause the immediate end of the process. The only exception is if the program is in the middle of making a request to the operating system, *i.e.,* a system call). This is because the request needs to finish first. This is the most **unsafe** way to kill the process (check [this post from Stripe's Game Day](https://stripe.com/blog/game-day-exercises-at-stripe)). **SIGKILL** is the 9th signal in the list, and it is usually sent with:

```
$ kill -9 <ID or PID>
```

* To know all the process with their PID, run:

```
$ ps -A
```

* To find the PID of a specific process:

```
$ pidof subl
```
or

```
$ ps aux | grep subl
```

or even

```
$ pgrep subl
```

* Pressing CTRL-C is a simpler way to tell the program to quit, and it sends a message called **SIGINT**. You can also specify the PID as an argument to kill.

* Another way of killing process:

```
$ pkill <PROCESS NAME>
```


### uptime

* Another great command is **uptime**, which shows how long the system has been running, with a measure of its load average as well:

```
$ uptime
```

### nice and renice

* Finally, you can change processes priority using ```nice``` (runs a program with modified scheduling priority) and ```renice```(alter the priority of running processes).


---
## Environment Variables

* *Environment variables* are several dynamic named values in the operating system that can be used in running processes.


### set and env
* You can see the *environment variables and configuration* in your system with:

```
$ set
```
or
```
$ env
```

### export and echo
* The value of an environment variable can be changed with:

```
$ export VAR=<value>
```

* The value can be checked with:

```
$ echo $VAR
```

* The **PATH** (search path) is the list of directories that the shell looks in to try to find a particular command. For example, when you type ```ls``` it will look at ```/bin/ls```. The path is stored in the variable **PATH**, which is a list of directory names separated by colons, and it's coded inside **./bashrc**. To export a new path you can do:

```
$ export PATH=$PATH:/<DIRECTORY>
```

### Variable in Scripts

* Inside a running shell script, there are pseudo-environment variables that are called with **$1**, **$2**, etc., for individual arguments that were passed to the script when it was run. In addition, **$0** is the name of the script and **$@** is for the list of all the command-line arguments.





---
## The "~/." Files (dot-files)

* The leading dot in a file is used as an indicator not to list these files typically, but only when they are specifically requested. The reason is that, generally, dot-files are used to store configuration and sensitive information for applications.

### ~/.bashrc

* **~/.bashrc** contains scripts and variables that are executed when Bash is invoked.

* It's a good experience to customize your **~/.bashrc**. Just google for samples, or take a look at this [site dedicated for sharing dot-files](http://dotfiles.org), or at [mine](https://github.com/go-outside-labs/Dotfiles-and-Bash-Examples/blob/master/configs/bashrc). Don't forget to ```source``` your **./bashrc** file every time you make a change (opening a new terminal has the same effect):

```
$ source ~/.bashrc
```

### Sensitive dot-files

* If you use cryptographic programs such as [ssh](http://en.wikipedia.org/wiki/Secure_Shell) and [gpg](https://www.gnupg.org/), you'll find that they keep a lot of information in the directories **~/.ssh** and **~/.gnupg**.

* If you are a *Firefox* user, the **~/.mozilla** directory contains your web browsing history, bookmarks, cookies, and any saved passwords.

* If you use [Pidgin](http://pidgin.im/), the **~/.purple** directory (after the name of [the IM library](https://developer.pidgin.im/wiki/WhatIsLibpurple)) contains private information. This includes sensitive cryptographic keys for users of cryptographic extensions to Pidgin such as [Off-the-Record](https://otr.cypherpunks.ca/).


---
## File Descriptors

* A **file descriptor** (FD) is a number indicator for accessing an I/O resource. The values are the following:
 * fd 0: stdin (standard input).
 * fd 1: stdout (standard output).
 * fd 2: stderr (standard error).

* This naming is used for manipulation of these resources in the command line. For example, to send an **input** to a program, you use **<**:

```
$ <PROGRAM> < <INPUT>
```

* To send a program's **output** somewhere else than the terminal (such as a file), you use **>**. For example, to just discard the output:

```
$ <PROGRAM> > /dev/null
```

* To send the program's error messages to a file you use the file descriptor 2:

```
$ <PROGRAM> 2> <FILE>
```

* To send the program's error messages to the same place where **stdout** is going, *i.e.* merging it into a single stream (this works great for pipelines):

```
$ <PROGRAM> 2>&1
```

----
## File Permissions

* Every file/directory in Linux is said to belong to a particular **owner** and a particular **group**. Files also have permissions stating what operations are allowed.


### chmod
* A resource can have three permissions: read, write, and execute:

 * For a file resource, these permission are: read the file, to modify the file, and to run the file as a program.

 * For a directory, these permissions are the ability to list the directory's contents, to create and delete files inside the directory, and to access files within the directory.

* To change the permissions you use the command ```chmod```.


### chown and chgrp

* Unix permissions model does not support *access control lists* allowing a file to be shared with an enumerated list of users for a particular purpose. Instead, the admin needs to put all the users in a group and make the file to belong to that group. File owners cannot share files with an arbitrary list of users.


* There are three agents relate to the resource: user, group, and all. Each of them can have separated permissions to read, write, and execute.

* To change the owner of a resource you use ```chown```. There are two ways of setting permissions with chmod:

 * A numeric form using octal modes: read = 4, write = 2, execute = 1, where you multiply by user = x100, group = x10, all = x1, and sum the values corresponding to the granted permissions. For example 755 = 700 + 50 + 5 = rwxr-xr-x: ``` $ chmod 774 <FILENAME>```

 * An abbreviated letter-based form using symbolic modes: u, g, or a, followed by a plus or minus, followed by a letter r, w, or x. This means that u+x "grants user execute permission", g-w "denies group write permission", and a+r "grants all read permission":```$ chmod g-w <FILENAME>```.


* To change the group you use ```chgrp```, using the same logic as for chmod.



* To see the file permissions in the current folder, type:

```
$ ls -l
```

* For example, ```-rw-r--r--``` means that it is a file (-) where the owner has read (r) and write (w) permissions, but not execute permission (-).


---

# Shell Commands and Tricks

## Reading Files

### cat

* Prints the content of a file in the terminal:

```
$ cat <FILENAME>
```

### tac

* Prints the inverse of the content of a file in the terminal (starting from the bottom):

```
$ tac <FILENAME>
```

### less and more

* Both print the content of a file, but adding page control:

```
$ less <FILENAME>
$ more <FILENAME>
```

### head and tail
* To read 20 lines from the begin:

```
$ head -20 <FILENAME>
```

* To read 20 lines from the bottom:

```
$ tail -10 <FILENAME>
```

### nl

* To print (cat) a file with line numbers:

```
$ nl <FILENAME>
```

### tee

* To save the output of a program and see it as well:

```
$ <PROGRAM> | tee -a <FILENAME>
```

### wc

* To print the length and number of lines of a file:

```
$ wc <FILENAME>
```



---
## Searching inside Files

### diff and diff3

* **diff** can be used to compare files and directories. Useful flags include: **-c** to list differences, **-r** to recursively compare subdirectories, **-i** to ignore case, and **-w** to ignore spaces and tabs.

* You can compare three files at once using **diff3**, which uses one file as the reference basis for the other two.


### file

* The command **file** shows the real nature of a file:

```
$ file requirements.txt
requirements.txt: ASCII text
```

### grep
* **grep** finds matches for a particular search pattern. The flag **-l** lists the files that contain matches, the flag **-i** makes the search case insensitive, and the flag **-r** searches all the files in a directory and subdirectory:

```
$ grep -lir <PATTERN> <FILENAME>
```

* For example, to remove lines that are not equal to a word:

```
$ grep -xv <WORD> <FILENAME>
```

---
## Listing or Searching for Files


### ls

* **ls** lists directory and files. Useful flags are **-l** to list the permissions of each file in the directory and **-a** to include the dot-files:

```
$ ls -la
```

* To list files sorted by size:

```
$ ls -lrS
```

* To list the names of the ten most recently modified files ending with .txt:

```
$ ls -rt *.txt | tail -10
```


### tree

* The **tree** command lists contents of directories in a tree-like format.


### find

* To find files in a directory:

```
$ find <DIRECTORY> -name <FILENAME>
```

### which

* To find binaries in PATH variables:

```
$ which ls
```

### whereis

* To find any file in any directory:

```
$ whereis <FILENAME>
```

### locate

* To find files by name (using a database):

```
$ locate <FILENAME>
```

* To test if a file exists:

```
$ test -f <FILENAME>
```

---
## Modifying Files

### true

* To make a file empty:
```
$ true > <FILENAME>
```

### tr

* **tr** takes a pair of strings as arguments and replaces, in its input, every letter that occurs in the first string by the corresponding characters in the second string. For example, to make everything lowercase:

```
$ tr A-Z a-z
```

* To put every word in a line by replacing spaces with newlines:

```
$ tr -s ' ' '\n'
```

* To combine multiple lines into a single line:


```
$ tr -d '\n'
```

* **tr** doesn't accept the names of files to act upon, so we can pipe it with cat to take input file arguments (same effect as ```$ <PROGRAM> < <FILENAME>```):

```
$ cat "$@" | tr
```



### sort

* Sort the contents of text files. The flag **-r** sort backward, and the flag **-n** selects numeric sort order (for example, without it, 2 comes after 1000):

```
$ sort -rn <FILENAME>
```

* To output a frequency count (histogram):

```
$ sort <FILENAME> | uniq -c | sort -rn
```

* To chose random lines from a file:

```
$ sort -R <FILENAME> | head -10
```

* To combine multiple files into one sorted file:

```
$ sort -m <FILENAME>
```

### uniq


* **uniq** remove *adjacent* duplicate lines. The flag **-c** can include a count:

```
$ uniq -c <FILENAME>
```

* To output only duplicate lines:

```
$ uniq -d <FILENAME>
```

### cut

* **cut** selects particular fields (columns) from structured text files (or particular characters from each line of any text file). The flag **-d** specifies what delimiter should be used to divide columns (default is tab), the flag **-f** specifies which field or fields to print and in what order:

```
$ cut -d ' ' -f 2 <FILENAME>
```

* The flag **-c** specifies a range of characters to output, so **-c1-2** means to output only the first two characters of each line:

```
$ cut -c1-2 <FILENAME>
```

### join
* **join** combines multiple file by common delimited fields:

```
$ join <FILENAME1> <FILENAME2>
```




----
## Creating Files and Directories

### mkdir

* **mkdir** creates a directory. A useful flag is **-p** which creates the entire path of directories (in case they don't exist):

```
$ mkdir -p <DIRNAME>
```


### cp

* Copying directory trees is done with **cp**. The flag **-a** is used to preserve all metadata:

```
$ cp -a <ORIGIN> <DEST>
```

* Interestingly, commands enclosed in **$()** can be run and then the output of the commands is substituted for the clause and can be used as a part of another command line:

```
$ cp $(ls -rt *.txt | tail -10) <DEST>
```


### pushd and popd

* The **pushd** command saves the current working directory in memory so it can be returned to at any time, optionally changing to a new directory:

```
 $ pushd ~/Desktop/
```

* The **popd** command returns to the path at the top of the directory stack.

### ln

* Files can be linked with different names with the **ln**. To create a symbolic (soft) link you can use the flag **-s**:

```
$ ln -s <TARGET> <LINKNAME>
```


### dd

* **dd** is used for disk-to-disk copies, being useful for making copies of raw disk space. For example, to back up your [Master Boot Record](http://en.wikipedia.org/wiki/Master_boot_record) (MBR):

```
$ dd if=/dev/sda of=sda.mbr bs=512 count=1
```

* To use **dd** to make a copy of one disk onto another:

```
$ dd if=/dev/sda of=/dev/sdb
```



----
## Network and Admin

### du

* **du** shows how much disk space is used for each file:

```
$ du -sha
```

* To see this information sorted and only the ten largest files:

```
$ du -a | sort -rn | head -10
```

* To determine which subdirectories are taking a lot of disk space:

```
$ du --max-depth=1 | sort -k1 -rn
```

### df

* **df** shows how much disk space is used on each mounted filesystem. It displays five columns for each filesystem: the name, the size, how much is used, how much is available, percentage of use, and where it is mounted. Note the values won't add up because Unix filesystems have **reserved** storage blogs which only the root user can write to.

```
$ df -h
```



### ifconfig

* You can check and configure your network interface with:

```
$ ifconfig
```

* In general, you will see the following devices when you issue **ifconfig**:

 * ***eth0***: shows the Ethernet card with information such as: hardware (MAC) address, IP address, and the network mask.

 * ***lo***: loopback address or localhost.


* **ifconfig** is supposed to be deprecated. See [my short guide on ip-netns](https://coderwall.com/p/uf_44a).


* A good trick is to change the IP address, [netmask](http://en.wikipedia.org/wiki/Subnetwork) and broadcast address for your network interface:

```
$ ifconfig eth0 192.168.1.115 netmask 255.255.255.0 broadcast 192.168.1.255
```

### dhclient

* Linux has a DHCP server that runs a daemon called ```dhcpd```, assigning IP address to all the systems on the subnet (it also keeps logs files):

```
$ dhclient
```

### dig


* **dig** is a DNS lookup utility (similar to ```dnslookup``` in Windows).

### netstat


* **netstat** prints the network connections, routing tables, interface statistics, among others. Useful flags are **-t** for TCP, **-u** for UDP, **-l** for listening, **-p** for program, **-n** for numeric. For example:

```
$ netstat -s
```

* To display all open network ports:

```
$ netstat -tulpn
```

* Display all TCP sockets:

```
$ netstat -nat
```

* Display all UDP sockets:

```
$ netstat -nau
```

* View established connections only:

```
$ netstat -natu | grep 'ESTABLISHED'
```


### ss


* **ss** is used to dumps socket (network connection) statistics.

* It can display stats for [PACKET sockets, TCP sockets, UDP sockets, DCCP sockets, RAW sockets, Unix domain sockets, and more](http://www.cyberciti.biz/tips/linux-investigate-sockets-network-connections.html).


```
$ ss -s
```

* To display all open network ports:

```
$ ss -l
```

* To display all TCP sockets:

```
$ ss -t -a
```


* To display all UCP sockets:

```
$ ss -u -a
```


### tcptrack

* **tcptrack** displays information about TCP connections it sees on a network and displays bandwidth usage on some interface by a host.

```
$ tcptrack -i eth0
```

### netcat, telnet and ssh

* To connect to a host server, you can use **netcat** (nc) and **telnet**. To connect under an encrypted session, **ssh** is used. For example, to send a string to a host at port 3000:

```
$ echo 4wcYUJFw0k0XLShlDzztnTBHiqxU3b3e | nc localhost 3000
```

* To telnet to localhost at port 3000:

```
$ telnet localhost 3000
```



### lsof

* **lsof** lists open files (remember that everything is considered a file in Linux):

```
$ lsof <STRING>
```

* To see open TCP ports:

```
$ lsof | grep TCP
```

* To see IPv4 port(s):

```
$ lsof -Pnl +M -i4
```

* To see IPv6 listing port(s):

```
$ lsof -Pnl +M -i6
```



---

## Useful Stuff

### echo


* **echo** prints its arguments as output. It can be useful for pipelining, and in this case, you use the flag **-n** not to output the trailing new line:
```
$ echo -n <FILENAME>
```

* **echo** can be useful to generate commands inside scripts (remember the discussion about file descriptor):

```
$ echo 'Done!' >&2
```

* Or to find shell environment variables (remember the discussion about them):

```
$ echo $PATH
```

* For example, we can send the current date information to a file:

```
$ echo Completed at $(date) >> log.log
```

### MD5 and SHA Hashing

* You can calculate hashes straight from the command line:

```
$ echo -n awesome | md5sum
$ echo -n awesome | sha1sum
03d67c263c27a453ef65b29e30334727333ccbcd -
$ echo -n awesome | sha256sum
705db0603fd5431451dab1171b964b4bd575e2230f40f4c300d70df6e65f5f1c -
```

### bc

* A calculator program is given by the command **bc** The flag **-l** stands for the standard math library:

```
$ bc -l
```

* For example, we can make a quick calculation with:
```
$ echo '2*15454' | bc
30908
```



### w, who, finger, users


* To find information about logged users you can use the commands **w, who, finger**, and **users**.





---
## Regular Expression 101

* **Regular expressions** (regex) are sequences of characters that forms a search pattern for use in pattern matching with strings.

* Letters and numbers match themselves. Therefore, 'awesome' is a regular expression that matches 'awesome'.

* The main rules that can be used with **grep** are:
 * ```.``` matches any character.
 * ```*``` any number of times (including zero).
 * ```.*``` matches any string (including empty).
 * ```[abc]``` matches any character a or b or c.
 * ```[^abc]``` matches any character other than a or b or c.
 * ```^``` matches the beginning of a line.
 * ```$``` matches the end of a line.

* For example, to find lines in a file that begin with a particular string you can use the regex symbol **^**:

```
$ grep ^awesome
```

* Additionally, to find lines that end with a particular string you can use **$**:

```
$ grep awesome$
```

* As an extension, **egrep** uses a version called *extended regular expresses* (EREs) which include things such:
 * ```()``` for grouping
 * ```|``` for or
 * ```+``` for one or more times
 * ```\n``` for back-references (to refer to an additional copy of whatever was matched before by parenthesis group number n in this expression).

* For instance, you can use ``` egrep '.{12}'```to find words of at least 12 letters. You can use ```egrep -x '.{12}'``` to find words of exactly twelve letters.




---

## Awk and Sed

* **awk** is a pattern scanning tool while **sed** is a stream editor for filtering and transform text. While these tools are extremely powerful, if you have knowledge of any very high-level languages such as Python or Ruby, you don't necessarily need to learn them.

### sed

* Let's say we want to replace every occurrence of *mysql* and with MySQL (Linux is case sensitive), and then save the new file to <FILENAME>. We can write a one-line command that says "search for the word mysql and replace it with the word MySQL":

```
$ sed s/mysql/MySQL/g <FILEORIGIN> > <FILEDEST>
```

* To replace any instances of a period followed by any number of spaces with a period followed by a single space in every file in this directory:

```
$ sed -i 's/\. */. /g' *
```

* To pass an input through a stream editor and then quit after printing the number of lines designated by the script's first parameter:

```
$ sed ${1}q
```




----

# Some More Advanced Stuff


## Scheduling Recurrent Processes


### at
* A very cute bash command is **at**, which allows you to run processes later (ended with CTRL+D):

```
$ at 3pm
```


### cron
* If you have to run processes periodically, you should use **cron**, which is already running as a [system daemon](http://en.wikipedia.org/wiki/Daemon_%28computing%29). You can add a list of tasks in a file named **crontab** and install those lists using a program also called **crontab**. **cron** checks all the installed crontab files and run cron jobs.


* To view the contents of your crontab, run:

```
$ crontab -l
```

* To edit your crontab, run:

```
$ crontab -e
```

* The format of the cron job is *min, hour, day, month, dow* (day of the week, where Sunday is 0). They are separated by tabs or spaces. The symbol * means any. It's possible to specify many values with commas.

* For example, to run a backup every day at 5am, edit your crontab to:

```
0 5 * * * /home/files/backup.sh
```

* Or if you want to remember some birthday, you can edit your crontab to:

```
* * 16 1 * echo "Remember Mom's bday!"
```


---
## rsync

* **rsync** performs file synchronization and file transfer. It can compress the data transferred using *zlib* and can use SSH or [stunnel](https://www.stunnel.org/index.html) to encrypt the transfer.

* **rsync** is very efficient when recursively copying one directory tree to another because only the differences are transmitted over the network.

* Useful flags are: **-e** to specify the SSH as remote shell, **-a** for archive mode, **-r** for recurse into directories, and **-z** to compress file data.

* A very common set is **-av** which makes **rsync** to work recursively, preserving metadata about the files it copies, and displaying the name of each file as it is copied. For example, the command below is used to transfer some directory to the **/planning** subdirectory on a remote host:

```
$ rsync -av <DIRECTORY> <HOST>:/planning
```



----
## File Compression

* Historically, **tar** stood for tape archive and was used to archive files to a magnetic tape. Today **tar** is used to allow you to create or extract files from an archive file, often called a **tarball**.

* Additionally you can add *file compression*, which works by finding redundancies in a file (like repeated strings) and creating a more concise representation of the file's content. The most common compression programs are **gzip** and **bzip2**.

* When issuing **tar**, the flag **f** must be the last option. No hyphen is needed. You can add **v** as verbose.

* A simple tarball is created with the flag **c**:

```
$ tar cf <FILE.tar> <CONTENTS>
```

* To extract a tarball you use the flag **x**:

```
$ tar xf <FILE.tar>
```

### gzip


* **gzip** is the most frequently used Linux compression utility. To create the archive and compress with gzip you use the flag **z**:

```
$ tar zcf <FILE.tar.gz>
```

* You can directly work with gzip-compressed files with ```zcat, zmore, zless, zgrep, zegrep```.

### bzip2

* **bzip2** produces files significantly smaller than those produced by gzip. To create the archive and compress with bz2 you use the flag **j**:

```
$ tar jcf <FILE.tar.bz2>
```

### xz

* **xz** is the most space efficient compression utility used in Linux. To create the archive and compress with xz:

```
$ tar Jcf <FILE.tar.xz>
```


----
## Logs

* Standard logging facility can be found at ```/var/log```. For instance:
 * ```/var/log/boot.log``` contains information that is logged when the system boots.
 * ```/var/log/auth.log``` contains system authorization information.
 * ```/var/log/dmesg``` contains kernel ring buffer information.


* The file ```/etc/rsyslog.conf``` controls what goes inside the log files.

* The folder ```/etc/services``` is a plain ASCII file providing a mapping between friendly textual names for internet services, and their underlying assigned port numbers and protocol types. To check it:

```
$ cat /etc/services
$ grep 110 /etc/services
```

* To see what your system is logging:

```
$ lastlog
```


-----
## /proc and inodes

* If the last link to a file is deleted but this file is open in some editor, we can still retrieve its content. This can be done, for example, by:
 1. attaching a debugger like **gdb** to the program that has the file open,

 2. commanding the program to read the content out of the file descriptor (the **/proc** filesystem), copying the file content directly out of the open file descriptor pseudo-file inside **/proc**.

* For example, if one runs ```$ dd if=/dev/zero of=trash & sleep 10; rm trash```, the available disk space on the system will continue to go downward (since more contents get written into the file by which **dd** is sending its output).

* However, the file can't be seen everywhere in the system! Only killing the **dd** process will cause this space to be reclaimed.

* An **index node** (inode) is a data structure used to represent a filesystem object such as files or directories. The true name of a file, even when it has no other name, is in fact its *inode number* within the filesystem it was created, which can be obtained by
```
$ stat
```
or
```
$ ls -i
```

* Creating a hard link with **ln** results in a new file with the same *inode number* as the original. Running *rm* won't affect the other file:

```
$ echo awesome > awesome
$ cp awesome more-awesome
$ ln awesome same-awesome
$ ls -i *some
7602299 awesome
7602302 more-awesome
7602299 same-awesome
```

----
## Text, Hexdump, and Encodings

* A Linux text file contains lines consisting of zero or more text characters, followed by the **newline character** (ASCII 10, also referred to as hexadecimal 0x0A or '\n').

* A text with a single line containing the word 'Hello' in ASCII would be 6 bytes (one for each letter, and one for the trailing newline). For example, the text below:

```
$ cat text.txt
Hello everyone!
Linux is really cool.
Let's learn more!
```

is represented as:

```
$ hexdump -c < text.txt
0000000 H e l l o e v e r y o n e ! \n
0000010 L i n u x i s r e a l l y
0000020 c o o l . \n L e t ' s l e a r
0000030 n m o r e ! \n
0000038
```

* The numbers displayed at left are the hexadecimal byte offsets of each output line in the file.

* Unlike text files on other operating systems, Linux files do not end with a special end-of-file character.


* Linux text files were traditionally always interpreted as **ASCII**. In ASCII, each character is a single byte, the ASCII standard as such defines exactly **128 characters** from **ASCII 0 to ASCII 127**. Some of them are non-printable (such as a newline). The printable starts at **32**. In that case, **ISO 8859** standards were extensions to ASCII where the character positions **128 to 255** are given a foreign-language interpretation.

* Nowadays, Linux files are most often interpreted as **UTF-8**, which is an encoding of **Unicode**, a character set standard able to represent a very large number of languages.

* For East Asian languages, **UTF-8 **chars are interpreted with **3 bytes** and **UTF-16** chars are interpreted with **2 bytes**. For western languages (such as German, for example), **UTF-16** characters are interpreted with **2 bytes**, and all the regular characters have **00** in front of it.

* In **UTF-16**, sentences start with two bytes **fe ff** (decimal 254 255) which don't encode as any part of the text. These are the **Unicode byte order mark** (BOM), which guards against certain kinds of encoding errors [1].


* Linux has a command to translate between character sets:

```
$ recode iso8859-1..utf-8
```

* This is useful if you see a **mojibake**, which is a character set encoding mismatch bug.


* There are only two mandatory rules about characters that can't appear in the filename: null bytes (bytes that have numeric value zero) and forward slashes **/**.




----

# Extra Juice: (pseudo)-Random Tricks

## Creating Pseudo-Random Passwords

* Add this to your **~/.bashrc**:

```
genpass() {
 local p=$1
 [ "$p" == "" ] && p=16
 tr -dc A-Za-z0-9_ < /dev/urandom | head -c ${p} | xargs
}
```

* Then, to generate passwords, just type:

```
$ genpass
```

* For example:

```sh
$ genpass
dIBObynGX9epYogz
$ genpass 8
c_yhmaXt
$ genpass 12
FZI2wz2LzyVQ
$ genpass 14
ZEfgQvpY4ixePt
```

---

## Most Common words plus Frequency


* We can use the commands we learned to build a script that prints the most common words plus their frequency:

```
tr -cs A-Za-z '\n' |
tr A-Z a-z |
sort |
uniq -c |
sort -rn |
sed ${1}q
```

* This is the same as splitting and tokenizing the file using Python:

```
import sys, collections
def common_words(n):
 return collections.Counter(sys.stdin.read().lower().split()).most_common(n)
```


---
## Password Asterisks

* By default, when you type your password in the terminal you should see no feedback. If you would like to see asterisks instead, edit:

```
$ sudo visudo
```

to have the value:

```
Defaults pwfeedback
```


----
## imagemagick

* You can create a gif file from terminal with ***imagemagick***:

```
$ mogrify -resize 640x480 *.jpg
$ convert -delay 20 -loop 0 *.jpg myimage.gif
```

---
## Easy access to the History


* Type ```!!``` to run the last command in the history, ```!-2``` for the command before that, and so on.




-----------------

# Further References:

- Unix: Culture and Command-Line, Seth Schoen

- [Understand STDERROR, STDOUT, STDIN](http://null-byte.wonderhowto.com/how-to/hack-like-pro-linux-basics-for-aspiring-hacker-part-16-stdin-stdout-stderror-0150693/).

- [Introduction to filesystems](http://www.howtogeek.com/196051/htg-explains-what-is-a-file-system-and-why-are-there-so-many-of-them).

- [Guide to Crontab](http://null-byte.wonderhowto.com/how-to/hack-like-pro-linux-basics-for-aspiring-hacker-part-18-scheduling-jobs-0154969/)

- [10 Useful scp commands](http://www.tecmint.com/scp-commands-examples/)

- [Bash shortcuts for productivity](http://www.skorks.com/2009/09/bash-shortcuts-for-maximum-productivity/)

- [Linux Bash Shell Cheat Sheet](http://cli.learncodethehardway.org/bash_cheat_sheet.pdf)

- [Rise of Linux, a Hacker History](http://www.linuxuser.co.uk/features/rise-of-linux-a-hackers-history)

- [9 commands to check hard disk partitions and disk space](http://www.binarytides.com/linux-command-check-disk-partitions/)

- [Checking Memory Usage in Linux](http://www.tecmint.com/check-memory-usage-in-linux/)

- [Usermode commands example](http://www.tecmint.com/usermod-command-examples/)

 [1] Encoding is a problem between Python 2 and Python 3:

 - In Python 2, a UTF-8 environment, len(" 美 國 ") is 6 and "美國 "[0] is a string containing the byte 0xe7 (which is the first byte of the three that encode 美 in UTF-8).

 - In Python 3, len("美國") is 2 (it's two Unicode characters), and " 美國"[0] is the string "美" (the first character in the string).

 - Each version of Python provides a data type that produces the other version's default behavior (the Unicode type in Python 2 and the bytes type in Python 3).

