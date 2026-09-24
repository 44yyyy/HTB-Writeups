# Monitored - Medium

Target IP: **10.129.230.96**

## User Flag

Initial scan: ```sudo nmap -sC -sV 10.129.230.96 -v```

Output shows standard port 22 for ssh, port 80 and 443 running Nagios XI hosted on Apache 2.4.56, and port 389 running OpenLDAP 2.2.X - 2.3.X.

```
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
| ssh-hostkey: 
|   3072 61:e2:e7:b4:1b:5d:46:dc:3b:2f:91:38:e6:6d:c5:ff (RSA)
|   256 29:73:c5:a5:8d:aa:3f:60:a9:4a:a3:e5:9f:67:5c:93 (ECDSA)
|_  256 6d:7a:f9:eb:8e:45:c2:02:6a:d5:8d:4d:b3:a3:37:6f (ED25519)
80/tcp  open  http     Apache httpd 2.4.56
|_http-server-header: Apache/2.4.56 (Debian)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to https://nagios.monitored.htb/
389/tcp open  ldap     OpenLDAP 2.2.X - 2.3.X
443/tcp open  ssl/http Apache httpd 2.4.56 ((Debian))
|_ssl-date: TLS randomness does not represent time
|_http-title: Nagios XI
|_http-server-header: Apache/2.4.56 (Debian)
| tls-alpn: 
|_  http/1.1
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| ssl-cert: Subject: commonName=nagios.monitored.htb/organizationName=Monitored/stateOrProvinceName=Dorset/countryName=UK
| Issuer: commonName=nagios.monitored.htb/organizationName=Monitored/stateOrProvinceName=Dorset/countryName=UK
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-11-11T21:46:55
| Not valid after:  2297-08-25T21:46:55
| MD5:   b36a:5560:7a5f:047d:9838:6450:4d67:cfe0
|_SHA-1: 6109:3844:8c36:b08b:0ae8:a132:971c:8e89:cfac:2b5b
Service Info: Host: nagios.monitored.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

There are two UDP ports open as well.

```
PORT      STATE         SERVICE      VERSION
53/udp    closed        domain
67/udp    closed        dhcps
68/udp    open|filtered dhcpc
69/udp    closed        tftp
123/udp   open          ntp          NTP v4 (unsynchronized)
135/udp   closed        msrpc
137/udp   closed        netbios-ns
138/udp   closed        netbios-dgm
139/udp   open|filtered netbios-ssn
161/udp   open          snmp         SNMPv1 server; net-snmp SNMPv3 server (public)
162/udp   open          snmp         net-snmp; net-snmp SNMPv3 server
445/udp   open|filtered microsoft-ds
500/udp   closed        isakmp
514/udp   closed        syslog
520/udp   closed        route
631/udp   open|filtered ipp
1434/udp  open|filtered ms-sql-m
1900/udp  closed        upnp
4500/udp  open|filtered nat-t-ike
49152/udp open|filtered unknown
Service Info: Host: monitored
```

Port 80 redirects us to ```nagios.monitored.htb```, so let's add it to our ```/etc/hosts``` file and navigate to it.

We are greeted with the home page of Nagios XI.

![1](Screenshots/M_1.jpg)

Let's try logging in with the default credentials of Nagios XI, ```nagiosadmin:nagiosadmin```.

It doesn't work.

Additional directory enumeration doesn't yield any useful information as well. I feel like we should pivot to SNMP.

Let's enumerate: ```snmpwalk -v 1 -c public 10.129.230.96```

Inside the output, there is a set of credentials.

```
iso.3.6.1.2.1.25.4.2.1.5.1390 = STRING: "-u svc /bin/bash -c /opt/scripts/check_host.sh svc XjH7VCehowpR1xZB"
```

Let's see if this works on the Nagios log in page.

It doesn't work but we get a different error, suggesting that the account exists but has been disabled.

![2](Screenshots/M_2.jpg)

I want to see if we can use the API to log in with these credentials. A Google search reveals that we can request a temporary token via API authentication by sending a POST request to the ```http://<YOUR_NAGIOS_XI_IP>/nagiosxi/api/v1/authenticate?pretty=1``` endpoint.

## Root Flag

## Contact

Email: <johnyang4406@gmail.com>, <john_s_yang@brown.edu>

LinkedIn: <https://www.linkedin.com/in/john-yang-747726292/>

HackTheBox: <https://profile.hackthebox.com/profile/019c423f-9b9b-708f-8b31-55983b89dddd?utm_medium=copy_url/>
