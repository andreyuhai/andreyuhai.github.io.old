---
layout: post
title: "5 steps to improve SSH security"
excerpt: SSH is something we (developers, sysadmins) use daily to connect to remote servers. Here are a few tips to keep your SSH connection secure.
modified: 2014-08-12 07:53:49 +0300
categories: articles
tags: [ssh, linux, security, shell, secure shell, pki]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
---

SSH is something we (developers, sysadmins) use daily to connect to remote servers. Here are a few tips to keep your SSH connection secure.

## 1. Disable root login

Allowing `root` to log in is not the safest idea. If you have a public facing server then there's a good change bots are attempting to log in. They do so by using known usernames that probably exists. One of them is `root`. Using uncommon usernames introduces an additional parameter the bots have to guess.

Every user should have their own account. Later, if needed, they can use `sudo` to gain more privileges. This adds one extra layer of security. If a standard user's account is compromised, the attacker still needs a sudo password to do more damage. Additionally, mandating each user to use their own account facilitates auditability as well.

In `/etc/ssh/sshd_config`

{% highlight xml %}
PermitRootLogin no
AllowUsers <username>
{% endhighlight %}

## 2. Disable password authentication

The reason here is simple. People come up with bad passwords. Besides being too short and never truly random, same passwords are likely being used in multiple places. If you still must use password authentication, use something like [Fail2ban](http://www.fail2ban.org/ Fail2ban webpage) to monitor for malicious signs.

Disable password login in `/etc/ssh/sshd_config`
{% highlight php %}
PasswordAuthentication no
{% endhighlight %}

## 3. Use SSH keys

I already told you a few reasons why passwords are not perfect. SSH keys on the other hand are long, random and if used correctly, a key pair is not shared between multiple services. Moreover, your private key never leaves your computer. The server creates a [nonce](https://en.wikipedia.org/wiki/Cryptographic_nonce "link to Wikipedia page about nonce") and encrypts it with your public key. This is sent to you over a secure channel and only the private key can decrypt it. After nonce has been decrypted it is sent back to the server and compared against the original. If they match, authentication is successful. This is the classic **challenge-response model**. Now looking back at passphrase based authentication, passphrases are sent to the server. If the secure channel is compromised, passwords can leak.

Generate a key pair with `ssh-keygen`
{% highlight bash %}
$ ssh-keygen -t rsa -C <email>
{% endhighlight %}

Use `ssh-copy-id` to move your public key to the server
{% highlight bash %}
$ ssh-copy-id -i <pub key> <user>@<host>
{% endhighlight %}

## 4. Run SSH on a non-standard port

Although this is security through obscurity, it still helps against most bots and script kiddies because they're expecting that your SSH daemon is running on port 22.

In `/etc/ssh/sshd_config`
{% highlight php %}
Port <port number>
{% endhighlight %}

Keep in mind, using tools like `nmap` it is possible to find which ports are open and your SSH port can still be found.

## 5. Run on protocol 2

SSH can use either protocol 1 or protocol 2. The older protocol 1 is less secure and should be disabled. Make sure protocol 2 is used.

In `/etc/ssh/sshd_config`
{% highlight php %}
Protocol 2
{% endhighlight %}
