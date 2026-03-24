
```
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.0.6001 (17714650) (Windows Server 2008 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.0.6001 (17714650)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Windows Server (R) 2008 Standard 6001 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ms-wbt-server Microsoft Terminal Service
|_ssl-date: 2026-01-05T17:13:43+00:00; -7s from scanner time.
| ssl-cert: Subject: commonName=internal
| Issuer: commonName=internal
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2025-07-24T21:18:58
| Not valid after:  2026-01-23T21:18:58
| MD5:     d2d9 1772 60cd 2a6b 1cd0 ed66 27ab b8cd
| SHA-1:   d5f0 03cb 2df3 c4c2 b6a7 2f5e 39b2 6c6f 491c 0587
|_SHA-256: 32e0 d762 4c5a adcd 0a86 0a8a 20e0 5677 9e6b e05c 47d3 facd e53c 2415 44b8 ace3
| rdp-ntlm-info: 
|   Target_Name: INTERNAL
|   NetBIOS_Domain_Name: INTERNAL
|   NetBIOS_Computer_Name: INTERNAL
|   DNS_Domain_Name: internal
|   DNS_Computer_Name: internal
|   Product_Version: 6.0.6001
|_  System_Time: 2026-01-05T17:13:35+00:00
5357/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Service Unavailable
|_http-server-header: Microsoft-HTTPAPI/2.0
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49156/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  msrpc         Microsoft Windows RPC
49158/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: INTERNAL; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008::sp1, cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_server_2008:r2

Host script results:
| nbstat: NetBIOS name: INTERNAL, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:9e:4c:ae (VMware)
| Names:
|   INTERNAL<00>         Flags: <unique><active>
|   WORKGROUP<00>        Flags: <group><active>
|_  INTERNAL<20>         Flags: <unique><active>
| smb-os-discovery: 
|   OS: Windows Server (R) 2008 Standard 6001 Service Pack 1 (Windows Server (R) 2008 Standard 6.0)
|   OS CPE: cpe:/o:microsoft:windows_server_2008::sp1
|   Computer name: internal
|   NetBIOS computer name: INTERNAL\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-01-05T09:13:35-08:00
| smb2-security-mode: 
|   2.0.2: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2026-01-05T17:13:35
|_  start_date: 2025-07-25T21:18:51
|_clock-skew: mean: 1h35m53s, deviation: 3h34m40s, median: -7s

```


- interesting service info the fact that there is a OS leak which is  from 2008 known for vulns  
 ```
| smb-os-discovery: 
|   OS: Windows Server (R) 2008 Standard 6001 Service Pack 1 (Windows Server (R) 2008 Standard 6.0)

|   OS CPE: cpe:/o:microsoft:windows_server_2008::sp1
```

`sudo nmap -p139,445 --script smb-vuln* 10.10.10.40`

![[Pasted image 20260105185729.png]]

found a poc 	  

![[Pasted image 20260105185802.png]]



edited the shell to connect to me with its specific mentioned format

installed module `pysmb ` for python2 using pip2 
not working for some reason

![[Pasted image 20260105193327.png]]

- is it possible that this user doesnt exist ?
	- lets try to use null ,guest , anonymous user
	- if it didnt work we will go for metasploit 

fuck it its not working i reverted this machine for like 1000 times


i think the issue is here 

![[Pasted image 20260105193008.png]]

that its using stager sysenter hook from metaspooit  so we need to use metasploit


so lets run metsaploit 

![[Pasted image 20260105194839.png]]

DONE !
