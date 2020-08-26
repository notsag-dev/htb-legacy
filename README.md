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
kali@kali:~$ nmap -Pn -sV -sC -oN legacy.nmap $LEGACY
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
