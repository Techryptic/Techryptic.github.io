---
layout:     post
title:      "Leading a Red Team through and through.."
date:       2021-03-18
author:     "Tech"
header-img: "img/post-red-team.jpg"
tags:
    - Pentesting
    - RedTeam
---

#  Leading a Red Team through and through

Over the years of leading red teams for both the public and private sectors, I started developing a knack. I feel comfortable enough to start writing about some stories, advice, and things I learned along the way.

### #1 When Joining an established red team, ask for their definition and differences between Red and Penetration Testing.

I had the pleasure of meeting multiple red teams throughout the years, and I quickly picked up that each group had its own 'definition' of red teaming. As your building up your career, you will want to make sure their version of red teaming aligns with your own goals. 

### #2 In-house Red Team vs Contracting Red Team

Having sat on both sides, there are MAJOR differences between both. 

The Company hires In-house red teamers to conduct internal assessments. They are an employee of that Company and follow said companies policies. A Contracting Firm hires contracting red teamers to send you to a company that has paid for a type of assessment to be conducted. They are an employee of the Contracting Firm and hired to do a job. 

##### In-house Red Teamers

In short, they wear multiple hats. Red, blue, and purple, to be exact. They are hired by the company to essential improve the posture of the business. They learn the structure, network flow, the complex over-arching enterprise technologies and use their expertise to find any security gaps.

##### Contracting Red Teamers

Companies can contract 'things' out to third-party companies. One of these things can be a security assessment. The company will hire an external team to come in and conduct a type of assessment. These assessments can happen on a long-term contract or job by job basis. Typically they do not wear multiple hats unless their contract tells them to.

### #3 Purple Teaming is more beneficial

Purple is essentially both sides working with each other to remediated the finding/vulnerability. For example, an internal red team reported an application with default credentials on the network that can be leveraged for remote code execution. A simple fix would be to change the credentials and move on, but most of the time, it is more than that.

Some effects of changing the password can:

- Disrupt a bunch of internal mechanisms that would be unknown to the tester.
- It is so legacy you cannot change the password.
- Vendor issued, who does not exist anymore. 

The purple aspect here is red working with the various blue teams on a remediation plan. In this particular scenario, red would suggest a few things:

- Potential firewall to whitelist/blacklist unwanted attention.
- Lockdown for specific users who are not part of an active directory group.
- Make accessible via Jump Server.
- Work with threat intel team on proper detections in place

After the suggestion, it is best for any retesting to be done. The point is, red will be working hand in hand with different teams on mitigating this issue.

### #4 Utilize the SIEM

**Note:** I was talking in the context of red teams that do multiple assessments and not an audit checkbox. Most Orgs that I have previously joined used Splunk Enterprise as their main SIEM and Tenable.SC, but the following tips can be used and adjusted for pretty much any of them.
An excellent read teamer is someone with blue teams experience, hands down.

**SIEM:**

One of the blue teams' tools is Splunk. It is a tool that parses large files, particularly log files, searches for specific kinds of data, correlates data from different files, and puts it together in a graphical format, all in real-time.
The result is pretty sophisticated error detection and tracking system. An extensive enterprise application can have dozens of servers and produce gigabytes of log files every hour. Splunk helps you manage all that and find the errors and other anomalies that matter.

**Red Team Mindset:**

It is not all about domain admins. That is one mindset of the red team that needs to change. Every assessment SHOULDN'T have the goal of DA/EA. That is a terrible practice and not suitable for business impact.

Reflecting on what I mentioned earlier, the goal of the red team is to find any security gaps.

**Splunk (with Tenable.SC data):**

These tips can be used across other SIEMs, and you would need to adjust your query appropriately. With all the ingested data, Tenable has some sweet Plugins which should be utilized. 

**Title Grabber:**
	We all love and used Eyewitness from Christopher Truncer, but the application has many gaps. I addressed these gaps in my version of Eyewitness called FiltrationWitness (Located on my Github). Even so, you will eventually run into issues with Certs, proxy, timeouts, legacy software, etc.
	
This plugin shows the HTTP request and, most notably, the HTTP response from Tenable's request. The response can be parsed in multiple ways to look for the low-hanging fruits. Some examples include parsing out the Title of the page, the redirect URL to find which appliance is running, Basic authentication cookies, etc.
    

```python
Plugin Name: HyperText Transfer Protocol (HTTP) Information
Plugin ID: 24260
Plugin URL: https://www.tenable.com/plugins/nessus/24260
Description: Some information about the remote HTTP configuration can be extracted.

Query:

index=tenable daysago="30" pluginID=24260
| rex field =pluginText "&lt;title&gt;(?<Title>.*)&lt;/title&gt"
| eval lastSeen=strftime(lastSeen, "%m%d%y %H:%M:%S")
| rex field=pluginText "SSL\s:\s(?<ssl>.*)"
| where Title!=""
| eval socket= ip.":".port
| dedup socket
| eval protocol = if(match(ssl, "(?yes)"), "https://", "http://")
| eval address = protocol."".socket
| table Title, address, ssl, lastSeen
| stats count by Title
| sort -count
```



**Vulns Hunt:**
This is a quick way to find all the exploitable machines on the network that should be fixed ASAP. Two filters we are using include exploitAvailable and checkType. We are setting the exploitAvilable value to Yes for low hanging fruits and the checkType value to remote as that vulnerability will be exploitable over the network. These vulns include Ethernal-Blue, BlueKeep, Remote Code Execution vulns, Deserialization, etc.

```python
URL: https://www.tenable.com/blog/understanding-exploitability
    
Query:
index=tenable daysago="30" expploitAvailable=Yes checkType=remote
| table pluginName, pluginID, riskFactor
| stats count by pluginName, riskFactor, pluginID
| eval socket = ip.":".port
| table pluginName, pluginID, riskFactor, count
| sort -count
```



**Port Scanner:**
Super helpful to know which machines had what port open at any given time.  


```python
Plugin Name: Nessus SNMP Scanner
Plugin ID: 14274
Plugin URL: https://www.tenable.com/plugins/nessus/14274
Description: This plugin runs an SNMP scan against the remote machine to find open ports.

Plugin Name: Nessus SYN scanner
Plugin ID: 11219
Plugin URL: https://www.tenable.com/plugins/nessus/11219
Description: This plugin is an SYN 'half-open' port scanner. It shall be reasonably quick even against a firewalled target. 
Note that SYN scans are less intrusive than TCP (full connect) scans against broken services, but they might cause problems for less robust firewalls and leave unclosed connections on the remote target if the network is loaded.

Why is this useful:
	Super helpful to know which machines had what port open at any given time.  
    
Query:

index=tenable daysago="30" daysago="30" (pluginID=14274 OR pluginID=11219)
| eval lastSeen=strftime(lastSeen, "%m%d%y %H:%M:%S")
| eval socket= ip.":".port
| dedup socket
| table port
```

Tons and tons of usefulness come from adapting to a SIEM!

### #5 Establish Threat Modelling / Rules of Engagement

It is common sense here, but this is a very fine line that can be cross and should be discussed more. I had been in situations where I had credentials to executives and C-Level folks, but you will need to proceed with caution on that. Also, think of any business impact that might happen. It's not all about running commands, and it's the value that you can bring. For instance, you have a C-level cred but no active directory paths that lead to your goal, what do you do? 

### #6 Jack of All Trades, Master of None

"*Jack of all trades*, *master of none*" is a figure of speech used in reference to a person who has dabbled in many skills rather than gaining expertise by focusing on one. This one might end up being a personal choice, but my advice is for you to never limit yourself by what you see from your peers. Do what interests you, even though your current gig might not be that. For example, exploit development is exciting, but my current gig isn't always about pop pop ret'ing. Find an excellent medium to keep up with your skill sets that your current gig might not excel in.

### #7 Hire Passion over Certifications

**Passion:** a strong feeling of enthusiasm or excitement for something or about doing something

**Certifications:** the action or process of providing someone or something with an official document attesting to a status or level of achievement.

I have interviewed maybe ~50-75 red teamers, cyber experts, penetration testers, hackers, whichever name you want to call it. I found that the ones with a passion for cybersecurity tend to be better hired in growth and within the team.

### Ending Notes

If you made it this far, I hope you picked up on a thing or two to use in your journey. Please reach out anytime via twitter if you have any questions.



![](/img/in-post/post-js-version/mr-robot.gif)
