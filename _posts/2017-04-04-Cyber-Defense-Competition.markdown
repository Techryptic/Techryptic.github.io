---
layout:     post
title:      "Cyber Defense Competition: Writeup as Blue Team Leader"
#subtitle:   "Pentesting"
date:       2017-04-04
author:     "Tech"
header-img: "img/post-defensecompetition.jpg"
tags:
    - Pentesting
    - Exploit Development
    - Shellcoding
---


Hello all, here is my story of an amazing event that took place this past weekend.

# Preparation
My team and I started as 6 members, in the end two members "left" and it was only four of us. We practice every weekend up until competition day. All said and done, I have written over 3,000+ lines of shell code to secure each and every machine we were given at that time. 

These include:

* Centos OS, for our Mail Server (Using iRedMail+dovcot on centos)
* Ubuntu OS, for our FTP Server
* Debian OS, for our Web Server
* Drupal Web Server
* Windows 2008 R2 for our Active Directory

Since we couldn't access the HMI remotely, we had only a few hours to get this up and secure, more on this later..

* Bonus: The HMI that ran off a raspberry pi!

For all my coded scripts: 
[https://github.com/techryptic/Cyber-Defense-Competition-Scripts](https://github.com/techryptic/Cyber-Defense-Competition-Scripts)

![](/img/in-post/post-js-version/prep-1.jpg)
![](/img/in-post/post-js-version/prep-2.jpg)

# HMI
The day before competition day, we got to see the HMI hands on, it is made  with a Raspberry Pi that acts as an Power-Light Grid and Water Grid. Since raspbian ran off the pi, anything we change couldn't be reverted back like the other OSs that we used the Vmware Snapshot feature on.

The pi served a flask python app at 10.0.100.50:80, by default when you visit that page, you will hit the Admin section to turn things on and off. The kicker is that the Pi couldn't talk out to other machines. This correlates with the drupal web server that linked to the HMI page, and contained an Iframe that pulled the pi information.

### How, why, and what
It's now about 12 am, competition starts in 7 hours and HMI needs to get secure. Without breaking the system and getting into programming heavy material. Ontop of all of that, we didn't have access to it anymore, so raspiban = debian = it sorta works.

### First Try (1am):
First I came up with just using regular authentication, but since the flask application server html pages, it was leaning towards JavaScript Authentication methods that can be compromised depending on red team skill level. Moving on...


### Second Try (3am):
How can the Pi and Webserver both talk, without essentially talk per se? and if red team figures out the Pi address they'll have full control of our system, time is ticking.

I know from working on the OSCP, that it takes a few mintues to do a full scan of a given subnet, with that in mind.. what if we can change to another address!

Theoretically, when red team scans our subnet it will show the original Pi address: 10.0.100.50, if we can change this address to a random permutation of several possible variations, they'll be two steps behind.. theoretically. The issue arise when the webserver HMI page iframes out using the PI address, so that would need to be changed also, at the same time, to the same number.

Removing random from the equation, a preset list of numbers that will change address on the Pi itself and another script to change the web servers iframe address. Another Issue arise where we were connecting to the pi via SSH, changing the address follow by a network restart/reboot would disconnect me thus stopping the script on the PI, but not on the web server, the outcome would be both will have different rolling addresses. Moving on...


### Third Try (5am):
Competition begins in 2 hours, bring on jeti mind.
I realized that both machines do have something in common, and that is time. Since both machines had NTP server installed on them, time were both sync up.

By combining both Try#2 and time, I came up with one of the most outrageous algorithm, that worked!

### Algorithm Details:
I quickly grabbed the erasable marker and starting writing on the picture frame in the hotel room, below is the picture but here is the break down:

On the left side is the current setup from Try#2, on the right side is with using time sync (explaining the craziness to the team)

The breakdown:
Time changes on both machines with a few seconds delay (No Issues).
What if I take the current time: `5:24`
and strip just the minutes: `24`
and add a one in-front of it: `124`
and use this as the ip address.. `10.0.100.124`

![](/img/in-post/post-js-version/prep-4.jpg)

On the Pi shell script:

```bash
#!/bin/sh
TIME=$(date +%M)
sed -i "4s/.*/"static\ ip_address=10.0.100.1"$TIME/" /etc/dhcpcd.conf
```

On the drupal web server, I made an additional page: whoami.php and included:

```php
<?php
$SUBNET = 1;
$TIME = date ('i');
$IP = $SUBNET + $ TIME;
?>
<iframe src='10.0.100.1".$IP."' width='100%' scrolling='vertical'></iframe>;
```

and on both machines to loop the script every 5 minutes. (You can set it to background, which would be better)

```bash
/bin/bash -c "sleep 15 && while true; do <path_to_your_script.sh> ; sleep 300; done"
```
Tested it on a makeshift debian OS and everything worked as play!
Slept for a few minutes and headed out to the competition.

### Competition:

![](/img/in-post/post-js-version/prep-5.jpg)

For all my coded scripts: 
[https://github.com/techryptic/Cyber-Defense-Competition-Scripts](https://github.com/techryptic/Cyber-Defense-Competition-Scripts)
