---
title: Wonderland
date: 2023-01-08T13:35:16-08:00
draft: true
tags: ['tryhackme', 'CTF', 'Writeup']
---

# Wonderland

{{< figure src="/wonderland/wonderland.jpg" width=240px height=240px attr="Enter Wonderland and capture the flags." >}}

___

[Wonderland](http://tryhackme.com/room/wonderland) is an Alice in Wonderland themed CTF room!

We'll start with adding the machine's IP to our hosts file.

``` bash
echo target-ip wonderland.thm >> /etc/hosts
```



___

### Recon

``` bash
nmap -sC -sV -T4 wonderland.thm
```

``` bash
Starting Nmap 7.93 ( https://nmap.org ) at 2023-01-08 14:02 EST
Nmap scan report for wonderland.thm (10.10.146.5)
Host is up (0.20s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8eeefb96cead70dd05a93b0db071b863 (RSA)
|   256 7a927944164f204350a9a847e2c2be84 (ECDSA)
|_  256 000b8044e63d4b6947922c55147e2ac9 (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Follow the white rabbit.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.23 seconds
```

{{< figure src="/wonderland/wonderland.png" link="/wonderland/wonderland.png" >}}


``` bash
dirbuster -H -l /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://wonderland.thm

```

``` bash
Starting OWASP DirBuster 1.0-RC1
Starting dir/file list based brute forcing
Dir found: / - 200
Dir found: /img/ - 200
Dir found: /r/ - 200
Dir found: /r/a/ - 200
Dir found: /r/a/b/ - 200
Dir found: /r/a/b/b/ - 200
Dir found: /r/a/b/b/i/ - 200
Dir found: /r/a/b/b/i/t/ - 200
Dir found: /poem/ - 200
DirBuster Stopped
```
{{< figure src="/wonderland/rabbit.png" link="/wonderland/rabbit.png" attr="" >}}


``` html
<!DOCTYPE html>

<head>
    <title>Enter wonderland</title>
    <link rel="stylesheet" type="text/css" href="/main.css">
</head>

<body>
    <h1>Open the door and enter wonderland</h1>
    <p>"Oh, you’re sure to do that," said the Cat, "if you only walk long enough."</p>
    <p>Alice felt that this could not be denied, so she tried another question. "What sort of people live about here?"
    </p>
    <p>"In that direction,"" the Cat said, waving its right paw round, "lives a Hatter: and in that direction," waving
        the other paw, "lives a March Hare. Visit either you like: they’re both mad."</p>
    <p style="display: none;">alice:**********************************************</p>
    <img src="/img/alice_door.png" style="height: 50rem;">
</body>
```



___

### User Flag



``` bash
ssh alice@wonderland.thm
```


