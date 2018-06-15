---
layout: post
title: "NTLM authentication fails on Linux when accessing QlikView Access Point"
excerpt: I was unable to connect to QlikView Access Point from a Linux machine. My browser presented me with a username/password prompt and after entering the correct credentials, it seemed like no authentication was made.
modified: 2014-10-09 21:01:12 +0300
categories: articles
tags: [qlikview, qv, access point, ntlm, linux, ubuntu, windows]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
aging: true
---

I was unable to connect to QlikView Access Point from a Linux machine. My browser presented me with a username/password prompt and after entering the correct credentials, it seemed like no authentication was made. At first I thought the web server has been misconfigured. After testing I found that using a Windows computer the authentication worked flawlessly. So something seemed to be wrong on my side.

The web server was configured to use NTLM authentication.

> In a Windows network, NT LAN Manager [NTLM](https://en.wikipedia.org/wiki/NT_LAN_Manager "NTLM Wikipedia page") is a suite of Microsoft security protocols that provides authentication, integrity, and confidentiality to users

It turns out that Linux builds of Firefox and Chromium do not support NTLMv2 which is the version the server was configured with. Only NTLMv1 is supported.

I'm not totally aware of the differences between the two versions but let's change the Windows Server to use NTLMv1.

* To achieve that, first you need to open Local Security Policy (secpol.msc).

* Go to Security Settings -> Local Policies -> Security Options

![local security policy](/images/local_sec_pol.png)

* Change 'Network security: LAN Manager authentication level' to 'Send LM & NTLM - use NTLMv2 session security if negotiated'.

![local security policy](/images/network_security.png)

* Uncheck both options in Network security: Minimum session security for NTLM SSP based (including secure RPC) clients.

As I mentioned previously, I'm not entirely sure about the differences between NTLMv1 and NTLMv2. I'm guessing one is more secure. You can leave a comment if you have any input on that. All in all, if you need to have access to QlikView Access Point from a Linux machine, the only solution I could come up with was to downgrade to NTLMv1.
