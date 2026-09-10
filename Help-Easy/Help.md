# Help - Easy

Target IP: **10.129.230.159**

## User Flag

Initial Nmap scans: ```sudo nmap --open 10.129.230.159 -vvv```

The output shows an unusual service hosted on port 3000. PPP?

```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-10 12:16 EDT
Initiating Ping Scan at 12:16
Scanning 10.129.230.159 [4 ports]
Completed Ping Scan at 12:16, 0.04s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 12:16
Completed Parallel DNS resolution of 1 host. at 12:16, 0.00s elapsed
DNS resolution of 1 IPs took 0.00s. Mode: Async [#: 2, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating SYN Stealth Scan at 12:16
Scanning 10.129.230.159 [1000 ports]
Discovered open port 80/tcp on 10.129.230.159
Discovered open port 22/tcp on 10.129.230.159
Discovered open port 3000/tcp on 10.129.230.159
Completed SYN Stealth Scan at 12:16, 0.19s elapsed (1000 total ports)
Nmap scan report for 10.129.230.159
Host is up, received reset ttl 63 (0.0100s latency).
Scanned at 2026-09-10 12:16:01 EDT for 0s
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 63
80/tcp   open  http    syn-ack ttl 63
3000/tcp open  ppp     syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.38 seconds
           Raw packets sent: 1004 (44.152KB) | Rcvd: 1001 (40.052KB)
```

A search online reveals that TCP port 3000 is used for web development servers like Node.js, React, or Express, but Nmap commonly misclassifies it as a diagnostic or server control port for Point-to-Point Protocol (PPP) daemon. Either can be true. 

Let's try to get a better idea of what we are dealing here with a more thorough scan: ```sudo nmap -sC -sV 10.129.230.159 -v```

The output for this scan confirms the online search, as port 3000 was identified as hosting an instance of Node.js Express framework. Additionally, we can see the url ```http://help.htb/``` on the output of the ```http-title``` NSE script for port 80 running Apache version 2.4.18 on Ubuntu.

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e5:bb:4d:9c:de:af:6b:bf:ba:8c:22:7a:d8:d7:43:28 (RSA)
|   256 d5:b0:10:50:74:86:a3:9f:c5:53:6f:3b:4a:24:61:19 (ECDSA)
|_  256 e2:1b:88:d3:76:21:d4:1e:38:15:4a:81:11:b7:99:07 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://help.htb/
|_http-server-header: Apache/2.4.18 (Ubuntu)
3000/tcp open  http    Node.js Express framework
|_http-title: Site doesn't have a title (application/json; charset=utf-8).
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Let's try navigating to the web server after adding the ```help.htb``` to our ```/etc/hosts``` file. We are greeted with the default welcome page of an Apache2 server installation. This is definitely a potential vector that we should dig deeper.

1

I was curious about port 3000, so I tried navigating to it. The page shows a static message naming a potential ```shiv``` user, and finding "credentials with given query." This is definitely interesting, but I didn't see a clear pathway forward from here.

2

Returning to the default Apache2 page, I performed directory enumeration with ```ffuf``` and got back some interesting results. We have the ```support``` and ```javascript``` directories redirecting us with a HTTP 301 status code, and a ```server-status``` giving us a HTTP 403 'Forbidden' status code, which means that we lack the permissions to access it.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-freuyexdea]─[/usr/share/wordlists/dirbuster]
└──╼ [★]$ ffuf -u http://help.htb/FUZZ -w directory-list-2.3-medium.txt:FUZZ -ic

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://help.htb/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

support                 [Status: 301, Size: 306, Words: 20, Lines: 10, Duration: 7ms]
javascript              [Status: 301, Size: 309, Words: 20, Lines: 10, Duration: 6ms]
                        [Status: 200, Size: 11321, Words: 3503, Lines: 376, Duration: 2295ms]
                        [Status: 200, Size: 11321, Words: 3503, Lines: 376, Duration: 7ms]
server-status           [Status: 403, Size: 296, Words: 22, Lines: 12, Duration: 7ms]
:: Progress: [220547/220547] :: Job [1/1] :: 1538 req/sec :: Duration: [0:01:24] :: Errors: 0 ::
```

Let's navigate to the newly found directories. First up: ```support```. We are greeted with an instance of a "Help Desk Software" by HelpDeskZ. We are able to log in and submit a ticket. I smell some potential web vulnerabilities here.

3

Next up: ```javascript```. Contrary to the scan results, it gives us a forbidden error, telling us that we can't access ```/javascript/``` on this server.

4







## Root Flag

## Contact

Email: <johnyang4406@gmail.com>, <john_s_yang@brown.edu>

LinkedIn: <https://www.linkedin.com/in/john-yang-747726292/>

HackTheBox: <https://profile.hackthebox.com/profile/019c423f-9b9b-708f-8b31-55983b89dddd?utm_medium=copy_url/>
