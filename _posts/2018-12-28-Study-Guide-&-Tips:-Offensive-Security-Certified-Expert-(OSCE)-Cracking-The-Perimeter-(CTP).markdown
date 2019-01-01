---
layout:     post
title:      "Study Guide & Tips: Offensive Security Certified Expert (OSCE) / Cracking The Perimeter (CTP)"
#subtitle:   "Exploit Development"
date:       2018-12-28
author:     "Tech"
header-img: "img/post-OSCE.jpg"
tags:
    - Reverse Engineering
    - Exploit Development
    - Shellcoding
    - x86
---

## Study Guide & Tips: Offensive Security Certified Expert (OSCE) / Cracking The Perimeter (CTP)

![](/img/in-post/post-js-version/offsec-student-certified-emblem-rgb-osce.png)
![](/img/in-post/post-js-version/osce-email.JPG)

--------

Glad you made it here, I was in your spot one time looking for additional resources to prime myself for the OSCE. There are plenty of reviews about the course/lab itself, I'll stick to something I wished more people gave, tips!

Some background, I was ecstatic to finish my exam + writeup in just under the 24 hour mark (With 4/4 on the exam!)

---------

### Tip #0

If you don't know C, I'd just go through the whole thing. I'd start here, so you can make some sense of the assembly you'll learn in the following course). Second link is a great primer on x86 ASM.

<https://users.cs.cf.ac.uk/Dave.Marshall/C/>

<http://opensecuritytraining.info/IntroX86.html>



---------

### Tip #1

```bash
egghunter.rb  -f raw  -e b00b | xxd  |awk -F":" '{print $2}' |sed 's/ //g' |awk '{print substr($0,0,32)}' | sed 's/\(..\)/\\x\1/g' |tac

```
> One liner to convert .bin



---------

### Tip #2

Hexdump files in 4bytes chunks; uselful if you want to (alphanum) encode a payload with SUB or ADD 

```bash
hexdump -e '4/1 "%02X" "\n"' FILE
```

Reverse every byte of a string (every 2 chars) in python: 
(also useful for encoding and pushing data into stack -little endian-):

```python
a = 'ABCDEFGH'
"".join(reversed([a[i:i+2] for i in range(0, len(a), 2)]))
output: 'GHEFCDAB'
```
---------
### Tip #3

This software is intended mainly as a tool for learning how to find and exploit buffer overflow bugs, and each of the bugs it contains is subtly different from the others, requiring a slightly different approach to be taken when writing the exploit.

Try and solve all the parameters, focus on:

* **KSTET**
* HTER
* GMON

<http://www.thegreycorner.com/2010/12/introducing-vulnserver.html>

---------
### Tip #4

**x86 simulator**:
Super useful for quickly checking and adjusting calculations for the alphanumeric shellcode techniques described in module 8 of the PDFs.

> You'll want to click on 'windows' in the upper right and then check off registers

<http://carlosrafaelgn.com.br/asm86/index.html?language=en>

---------
### Tip #5

Good resource about creating shellcode for linux and windows, checkout part "Advanced Shellcoding".

<http://www.vividmachines.com/shellcode/shellcode.html>

---------
### Tip #6

Take the time completed: x86 Assembly Language and Shellcoding on Linux from pentesteracademy.com

It's a great course that focuses on teaching the basics of 32-bit assembly language for the Intel Architecture (IA-32) family of processors on the Linux platform and applying it to Infosec. 

<https://www.pentesteracademy.com/course?id=3>

---------
### Tip #7

Be sure to google various AV bypass technique that was used back in the day, some blogs have covered it extensively.

Here's a great read:

<https://www.rapid7.com/globalassets/_pdfs/whitepaperguide/rapid7-whitepaper-metasploit-framework-encapsulating-av-techniques.pdf>

---------
### Tip #8

Don't focus on ROP. The OSCE does not get into it, be vary of some blogs that mention it. If your going for your OSEE, than ROP away!

But.. If your going for OSEE, here are some awesome links to follow and practice:

<https://jipanyang.wordpress.com/2014/06/09/glibc-malloc-internal-arena-bin-chunk-and-sub-heap-1/>

<https://github.com/shellphish/how2heap>

* <http://phrack.org/issues/57/8.html> - Vudo Malloc Tricks
* <http://phrack.org/issues/57/9.html> - Once Upon a Free()
* <http://phrack.org/issues/61/6.html> - Advanced Doug Lea's Malloc exploits
* <http://seclists.org/vuln-dev/2004/Feb/25> - Exploiting the Wilderness
* <http://seclists.org/bugtraq/2005/Oct/118> - Malloc Maleficarum
* <https://awarenetwork.org/etc/aware.ezine.1.alpha.txt> .aware eZine
* <http://phrack.org/issues/66/10.html> - Malloc Des-Maleficarum 
* <http://phrack.org/issues/67/8.html> - The House of Lore

<https://www.corelan.be/index.php/2011/12/31/exploit-writing-tutorial-part-11-heap-spraying-demystified/>
> More windows oriented, so if you're not familiar with windows exploitation, start with tutorial part 1, and/or opensecuritytraining Exploits 2

><https://arjunsreedharan.org/post/148675821737/memory-allocators-101-write-a-simple-memory>


---------
### Tip #9

Learn IDA, seriously. IDA can help you understand what an executable is doing behind the scenes.  It's hard to capture that with immunity/ollyDbg.

---------
### Tip #10

I have to mention this, as **EVERYONE** falls for it.. watch your **\xCC**, if your exploit breaks somewhere... remember this tip.

>\xCC is INT3 and will break your debugger once it reads it

---------
### Tip #11

```bash
echo $shellcode | sed s/"\x"/""/g | fold -w2|tac|tr -d "\n"|  sed 's/(..)/\1\x/g' | sed 's/.{2}$//' | awk '$0="\x"$0'
```
> To swap an entire string of shellcode to little endian in oneliner using bash.


---------
### Tip #12

```bash
msf-egghunter -a x86 -f raw -e w00t -b '\x00' | msfvenom -p - -a x86 --platform windows -b "\x00" -e x86/alpha_mixed -f python
```
> A quick oneliner to generate encoded egghunter.


---------
### Tip #13

```bash
cat shellcode.bin | msfvenom -p - windows/exec CMD=calc.exe -e ENCODERYOUWANT -f python
```
> Convert the output from a script to a .bin file and than cat it through msfvenom. Saves time.

---------
### Tip #14

Restarting a program with the debugger is a pretty annoying process. Here's a shortcut I found:

>Ctrl+F2 > Ctrl+G > Enter > F2 > F9 (restart / go to / last address / set breakpoint / run) only takes a second or two


---------
### Tip #15

Transfering files from box to box can be a pain with how limited the machine can be.

The code below will make a webserver on your host machine, which another machine can access and upload files.

<https://github.com/Techryptic/Random-Scripts/blob/master/No-Access-Upload.php>



![](/img/in-post/post-js-version/osce-certs.png)

