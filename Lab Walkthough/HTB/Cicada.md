AD
10.129.231.149



nmap 

```sh
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-07-17 22:23:43Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Issuer: commonName=CICADA-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-08-22T20:24:16
| Not valid after:  2025-08-22T20:24:16
| MD5:     9ec5 1a23 40ef b5b8 3d2c 39d8 447d db65
| SHA-1:   2c93 6d7b cfd8 11b9 9f71 1a5a 155d 88d3 4a52 157a
|_SHA-256: c8b9 54cb f36f 460f 859f 24c6 f4b1 7245 3eec 001b ce26 2f62 7229 4374 b24d 0772
|_ssl-date: 2026-07-17T22:25:12+00:00; +6h59m59s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-07-17T22:25:12+00:00; +6h59m59s from scanner time.
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Issuer: commonName=CICADA-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-08-22T20:24:16
| Not valid after:  2025-08-22T20:24:16
| MD5:     9ec5 1a23 40ef b5b8 3d2c 39d8 447d db65
| SHA-1:   2c93 6d7b cfd8 11b9 9f71 1a5a 155d 88d3 4a52 157a
|_SHA-256: c8b9 54cb f36f 460f 859f 24c6 f4b1 7245 3eec 001b ce26 2f62 7229 4374 b24d 0772
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-07-17T22:25:12+00:00; +6h59m59s from scanner time.
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Issuer: commonName=CICADA-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-08-22T20:24:16
| Not valid after:  2025-08-22T20:24:16
| MD5:     9ec5 1a23 40ef b5b8 3d2c 39d8 447d db65
| SHA-1:   2c93 6d7b cfd8 11b9 9f71 1a5a 155d 88d3 4a52 157a
|_SHA-256: c8b9 54cb f36f 460f 859f 24c6 f4b1 7245 3eec 001b ce26 2f62 7229 4374 b24d 0772
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-07-17T22:25:12+00:00; +6h59m59s from scanner time.
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Issuer: commonName=CICADA-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-08-22T20:24:16
| Not valid after:  2025-08-22T20:24:16
| MD5:     9ec5 1a23 40ef b5b8 3d2c 39d8 447d db65
| SHA-1:   2c93 6d7b cfd8 11b9 9f71 1a5a 155d 88d3 4a52 157a
|_SHA-256: c8b9 54cb f36f 460f 859f 24c6 f4b1 7245 3eec 001b ce26 2f62 7229 4374 b24d 0772
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
63242/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: CICADA-DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
|_clock-skew: mean: 6h59m58s, deviation: 0s, median: 6h59m58s
| smb2-time: 
|   date: 2026-07-17T22:24:35
|_  start_date: N/A

```




![[Pasted image 20260717180704.png]]

![[Pasted image 20260717180745.png]]



there is defualt password 

`Cicada$M6Corpb*@Lp#nZp!8`

lets try to login using this pass aybe brute forice username 


![[Pasted image 20260720175405.png]]


now i found these i used bash to extrac the userames only 

`r00t@r00t-linux:~/Downloads$ cat test | cut -d "\\" -f 2  | awk '{print $1}'`


![[Pasted image 20260720175618.png]]


notice that  michael.wrightson was the only one without logout failure niether with guest flag  meaning the rest gave access as guest permission while michael as his permisison

![[Pasted image 20260721174714.png]]


i notices in the description of one 

![[Pasted image 20260721174738.png]]

lets try to enum with this 

`david.orelious`

`aRt$Lp#7t*VQ!3`


![[Pasted image 20260721175025.png]]


lets check DEV

![[Pasted image 20260721175114.png]]


so there is `emily.oscars` with password `Q!3@Lp#M6b*7t*Vt`

lets try to  enum with it 

![[Pasted image 20260721175311.png]]


we will use evil-winrm to connect  

![[Pasted image 20260722102602.png]]



executing whomi /all

![[Pasted image 20260722102910.png]]

emily has `sebackuprivilege` 


![[Pasted image 20260722104605.png]]


![[Pasted image 20260722104903.png]]

lets try and crack them 

```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b87e7c93a3e8a0ea4a581937016f341:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::

```


i couldnt crack it so i used Pass the hash on evil-winrm 

![[Pasted image 20260722111409.png]]

DONE