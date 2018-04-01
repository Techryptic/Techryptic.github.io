---
layout:     post
title:      "National Security Agency (NSA) Code Breaker 2016 Challenge Writeup"
#subtitle:   "Pentesting"
date:       2017-01-01
author:     "Tech"
header-img: "img/post-nsa-challenge.jpg"
tags:
    - Reverse Engineering
    - Exploit Development
    - Encryption
---


**This was my first year I participated in the NSA Codebreaker Challenge, and I'm glad I took the ride. This year focused on reverse engineering and some modern vulnerability exploitations. They included a live leader-boards which is great to see what schools are in the top lead. My writeup is based of Linux point of view.**

Below is the story line for the challenges ahead:

*Terrorists have recently developed a new type of remotely controlled Improvised Explosive Device (IED), making it harder for the U.S. Armed Forces to detect and ultimately prevent roadside bomb attacks against troops deployed overseas. The National Security Agency (NSA), in accordance with its support to military operations mission, has been asked to develop capabilities for use against this new threat. This will consist of six tasks of increasing difficulty, with the ultimate goals of being able to disarm the IEDs remotely and permanently render them inoperable without the risk of civilian casualties.*

---

# Challenge One: Information Gathering and Triage, Part 1:
*A military organization captured a laptop of a known explosives expert within a terrorist organization.  Further analysis revealed that the laptop contained a debug version of the remote client interface that the individual used to communicate with the IEDs. To help detect other client programs in use, we are cataloging binary signatures and basic network signatures for every version of the IED software we find. To support these efforts, your task is to compute the SHA256 hash of the client binary and identify the source and destination TCP ports that it uses when connecting to an IED.*

### Solving:
This challenge was reasonably painless, although I originally took the long path in IDA (analysis option). I ran their given program in an sandbox environment and checked my firewall logs to see what the incoming and outgoing ports were. 

Some important commands to know would be:

* netstat --listen
* lsof -i

---

# Challenge Two: Information Gathering and Triage, Part 2
*Great work! Based on the signatures you provided, we were able to collect network communications that we believe contains traffic to an IED that is about to be detonated. Unfortunately, there appears to be a lot of unrelated network traffic in the collected data since other programs use the same port. Using the provided packet capture file (PCAP), we need your help to create more specific signatures for identifying network communications with the IED. This would be a huge first step in detecting when an IED has been armed, for example, which would allow us to alert troops in the region around where the signal was collected. For this task, your goals are to identify the version string sent by the client software when initiating a connection to the IED and to determine the IP address of the undetonated IED from the packet capture.*

*UPDATE: Intelligence suggests that the version strings are 11 characters long and look something like x.x-xxxxxxx*

### Solving:
We are given a traffic.pcap file, and need to find the version string. This can be solved multiple ways. The fastest and most efficient to me was to use ngrep.

```python
ngrep -q -I traffic.pcap '[a-zA-Z0-9]\.[a-zA-Z0-9]\-[a-zA-Z0-9]{7}'
``` 
The above code will search the pcap file for any combination of "x.x-xxxxxxx".

---

# Challenge Three: Disarm Capability, Part 1
*Thanks to your hard work we were able to eventually geolocate the device and work with military partners to retrieve the system for further analysis. It turned out to be a test system that one of the IED developers had been using in lieu of a live device. We provided the system to a team of software reverse engineers and their preliminary assessment is that we have a fully functional copy of the IED software, a key file, and a dummy driver that emulates the various IED states. Analysts believe that this key file contains the information needed to authenticate to a real IED (somewhere in the field) and send commands to it. Presumably this test system was used to validate the software and key file before it was deployed to an actual IED. Since the key file appears to be encrypted, we are going to need your help to figure out a way to decrypt it. This should enable us to disarm the fielded IED that uses this key, though we will still need to figure out exactly how it is being used for authentication (next task). The goal of this task is for you to obtain the decrypted contents of the key file.*

### Solving:
We are given 3 files in both Windows and Linux versions.

* Server Binary
* Test Driver
* Key File

We need to figure out a way to decrypt the file.. Lets first pull any info we can before we go more in-depth:

```python
$file 123123123.key.enc
123123123.key.enc: data
``` 

Don't bring back anything, which we knew but it's always good to double check for anything hidden.

```language-python
$strings 123123123.key.enc
D@fl
pM#b
}(^Bb
ma#4
```

Since it's encrypted the data shown here would be non relevant. Lets try an objdump and see if we get anything.

```python
$objdump -bbinary -D -m i386 123123123.key.enc test.bin
123123123.key.enc:     file format binary
```

Again, were just testing the waters here to make sure the file says what it is. I don't know the scope of how difficult they will make these challenges so it's always good to run by everything.

After those attempts, I took the server binary file and ran a hexdump on it. Scrolled through the dump and found a private key sitting 5/8s of the way down. SWEET, take that key and save it as a .PEM file to decrypt the original encrypted file that we were testing earlier.

To decrypt a file when you have the private key is relatively primitive:

```python
openssl rsautl -decrypt -inkey privatekey.pem -in 123123123.key.enc -out key.bin
``` 
and the output into key.bin:

```python
otpauth://totp/457468756?secret=YVVMOG5BZYODJWA2QGGU6SRN3AAEIMRV4RWLZXLF2AXEJDGTZ7PQ
```

---

# Challenge Four: Disarm Capability, Part 2
*Perfect! Now that we have the key file we can work on a disarm capability. Several intelligence reports suggest that terrorists use a secure token (i.e., small hardware device) for generating unique one-time codes for authenticating to the IED when sending commands. We believe these codes change over time and are only valid for a certain time window and for specific device serial numbers. Based on previous signatures you provided, we have located the armed IED that is using the same version of software and key serial number from Task 3 and we need to disarm it ASAP. We do not have the secure token that corresponds to the device, but we still need to be able to authenticate to it with the correct code in order to disarm it. Your objective for this task is to figure out how to generate valid one-time codes and provide one that we can use to disarm the IED. The decrypted key file you provided earlier should help with this part.*

### Solving:
From the last challenge we decrypted the given file, than put its contents into a key.bin file, now we need to turn that key.bin final into an OTP code to disarm the IED.

I actually solved this in 10 seconds right after the previous challenge. From the last challenge I took the secret key:

```python
otpauth://totp/457468756?secret=YVVMOG5BZYODJWA2QGGU6SRN3AAEIMRV4RWLZXLF2AXEJDGTZ7PQ
```
Secret Key = YVVMOG5BZYODJWA2QGGU6SRN3AAEIMRV4RWLZXLF2AXEJDGTZ7PQ

and to pull an OTP code out of that we will be using the oathtool command.

```python
oathtool --totp --base32 YVVMOG5BZYODJWA2QGGU6SRN3DDEIMRV4RWLZXLF2AXEJDGTZ7PQ
099600
```
A few things to note, we know that we will use TOTP as they mention server tile, base32 as thats the variable were dealing with. Pretty much plug in play if you have messed with oathtool before. It outputs 099600, which is the OTP to past that challenge. Do note that when you do generate an OTP code it's only for around 30 seconds before it will show up as invalid.

---

### BreakDown Info

Time-based OTP (TOTP) algorithm generates a password based on current time-stamp ,shared secret key ( or It may be something unique to each account). Passwords generated close together in time from the same secret key will be equal.

The second characteristic is very important in term of security. An OTP depends on 2 parameters:

    A secret key
    A counter

The correct term for that string is “secret key”; if it is leaked, the CSPRNG is basically broken (since recovering the current counter is just a matter of generating the whole sequence up until the match).

Obviously this trick requires some coordination between the token generating the passwords for the user and the server that is checking them. This coordination basically consists of two things:

    The token key
    The current counter value

