```
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-01-02 15:51:14Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: resourced.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: resourced.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-01-02T15:52:43+00:00; -2s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: resourced
|   NetBIOS_Domain_Name: resourced
|   NetBIOS_Computer_Name: RESOURCEDC
|   DNS_Domain_Name: resourced.local
|   DNS_Computer_Name: ResourceDC.resourced.local
|   DNS_Tree_Name: resourced.local
|   Product_Version: 10.0.17763
|_  System_Time: 2026-01-02T15:52:03+00:00
| ssl-cert: Subject: commonName=ResourceDC.resourced.local
| Issuer: commonName=ResourceDC.resourced.local
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-01-01T15:45:03
| Not valid after:  2026-07-03T15:45:03
| MD5:     9a97 5c0b 8ceb bf56 18ae 74a6 67eb 0a3d
| SHA-1:   422d 4f2a 1d5b e586 481c ba56 b9d4 7db5 cff5 0efc
|_SHA-256: e566 55b5 6d4e 21f5 0c88 0b8b 2a85 d7e4 3cb1 5599 2b78 1edc 46c4 8e29 3d22 0042
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49675/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49676/tcp open  msrpc         Microsoft Windows RPC
49694/tcp open  msrpc         Microsoft Windows RPC
49712/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: RESOURCEDC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-01-02T15:52:06
|_  start_date: N/A
|_clock-skew: mean: -2s, deviation: 0s, median: -2s

```


- checked for users in ldap  nothing
- anon access on smb  nothing
- enum4linux though showed us this
![[Pasted image 20260102170852.png]]



we have a username and password  in v.ventz

- i tried using the list of username the password mentioned i found out that  `v.vent` has access on smb


- found some intersting files

![[Pasted image 20260102172648.png]]


i got a bunch of hashes by dumping ntds and system 


![[Pasted image 20260102183040.png]]

i extracted the hashes which is the last argument after :  since the sam password is dividied like this LM hash : NTLM hash 

and i remote accessed it 

![[Pasted image 20260102184402.png]]

- after executing the command we got auth error meaning that we try some other passwords 

- i tried bruteforcing with nxc 

![[Pasted image 20260102185024.png]]

- lets try l.living

worked 
![[Pasted image 20260102185121.png]]

lets try priv escalation 

 - download winpease 
 - download sharphound


in sharphound we found out that that account has a genericall over the DC

![[Pasted image 20260104141658.png]]


so we followed this walkthough of a take over  using shadow credentials https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/shadow-credentials




![[Pasted image 20260104142106.png]]



![[Pasted image 20260104143053.png]]


so this domain doesnt support **PKINIT**  which is **Public Key Cryptography for Initial Authentication** which It's the **Kerberos extension that allows certificate-based authentication** instead of password-based.


which we can conferm using 

![[Pasted image 20260104143425.png]]


another attack we can abuse is **Resource-Based Constrained Delegation attack**

Adding computer 
![[Pasted image 20260104152519.png]]

RBCD
![[Pasted image 20260104163821.png]]

Verifying the AD computer that we added 

![[Pasted image 20260104163808.png]]


impersonating ticket 

![[Pasted image 20260104163928.png]]


exporting the ticket to use it 


```
export KRB5CCNAME=Administrator@cifs_resourcedc.resourced.local@RESOURCED.LOCAL.ccache

```

`/etc/hosts`

write the domain controller ip

![[Pasted image 20260104165258.png]]

DONE
![[Pasted image 20260104165200.png]]

### Refernces

- https://www.hackingarticles.in/active-directory-penetration-testing-using-impacket/

- https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/shadow-credentials