
AD  - Medium 


```sh
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-05 22:11:05Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Issuer: commonName=sequel-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-01-18T23:03:57
| Not valid after:  2074-01-05T23:03:57
| MD5:     ee4c c647 ebb2 c23e f472 1d70 2880 9d82
| SHA-1:   d88d 12ae 8a50 fcf1 2242 909e 3dd7 5cff 92d1 a480
|_SHA-256: 9b16 318b 7bc0 f508 b5cd 98a5 3a80 d1d7 54e1 e158 45b1 4956 003b 4bb5 05f8 7f98
|_ssl-date: 2026-08-05T22:12:39+00:00; +7h59m40s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-08-05T22:12:40+00:00; +7h59m41s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Issuer: commonName=sequel-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-01-18T23:03:57
| Not valid after:  2074-01-05T23:03:57
| MD5:     ee4c c647 ebb2 c23e f472 1d70 2880 9d82
| SHA-1:   d88d 12ae 8a50 fcf1 2242 909e 3dd7 5cff 92d1 a480
|_SHA-256: 9b16 318b 7bc0 f508 b5cd 98a5 3a80 d1d7 54e1 e158 45b1 4956 003b 4bb5 05f8 7f98
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info: 
|   10.129.228.253:1433: 
|     Target_Name: sequel
|     NetBIOS_Domain_Name: sequel
|     NetBIOS_Computer_Name: DC
|     DNS_Domain_Name: sequel.htb
|     DNS_Computer_Name: dc.sequel.htb
|     DNS_Tree_Name: sequel.htb
|_    Product_Version: 10.0.17763
| ms-sql-info: 
|   10.129.228.253:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_ssl-date: 2026-08-05T22:12:39+00:00; +7h59m41s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-08-05T22:01:37
| Not valid after:  2056-08-05T22:01:37
| MD5:     bb16 7c63 bfea e551 d483 0664 8a0e 9c42
| SHA-1:   b70d 3aeb 8313 a526 0ba3 be1d de4e 7b83 8da0 82e6
|_SHA-256: 9146 dcf9 0693 d658 3fd4 2562 0756 4eaf 880c 992e 6b8f 6ffb 9a69 0c6c 8262 5098
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-08-05T22:12:39+00:00; +7h59m40s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Issuer: commonName=sequel-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-01-18T23:03:57
| Not valid after:  2074-01-05T23:03:57
| MD5:     ee4c c647 ebb2 c23e f472 1d70 2880 9d82
| SHA-1:   d88d 12ae 8a50 fcf1 2242 909e 3dd7 5cff 92d1 a480
|_SHA-256: 9b16 318b 7bc0 f508 b5cd 98a5 3a80 d1d7 54e1 e158 45b1 4956 003b 4bb5 05f8 7f98
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Issuer: commonName=sequel-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-01-18T23:03:57
| Not valid after:  2074-01-05T23:03:57
| MD5:     ee4c c647 ebb2 c23e f472 1d70 2880 9d82
| SHA-1:   d88d 12ae 8a50 fcf1 2242 909e 3dd7 5cff 92d1 a480
|_SHA-256: 9b16 318b 7bc0 f508 b5cd 98a5 3a80 d1d7 54e1 e158 45b1 4956 003b 4bb5 05f8 7f98
|_ssl-date: 2026-08-05T22:12:40+00:00; +7h59m41s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49681/tcp open  msrpc         Microsoft Windows RPC
49700/tcp open  msrpc         Microsoft Windows RPC
52797/tcp open  msrpc         Microsoft Windows RPC
52821/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-08-05T22:12:01
|_  start_date: N/A
|_clock-skew: mean: 7h59m40s, deviation: 0s, median: 7h59m39s

```




![[Pasted image 20260805162002.png]]

so public is accessible 


i found a pdf

![[Pasted image 20260805162522.png]]

potential access

![[Pasted image 20260805162538.png]]


there is several potential users here
![[Pasted image 20260805163016.png]]


tom
ryan
brandon.browm@sequel.htb


maybe we have a winner 

![[Pasted image 20260805163520.png]]


we have access

![[Pasted image 20260805163620.png]]


first thing since its AD i checked for cmd shell and for dirtree



![[Pasted image 20260805172228.png]]



![[Pasted image 20260805172359.png]]

sql_svc / REGGIE1234ronnie


![[Pasted image 20260805172607.png]]




![[Pasted image 20260806110501.png]]


![[Pasted image 20260806110614.png]]

win peas and bloodhound didnt help i am in a deadend with the certification abuse ill check some logs and so for  something 

![[Pasted image 20260806115544.png]]

there is a logs backup in sql adter checking the logs there is a failed backup for ryan

![[Pasted image 20260806115800.png]]


USER.txt PWNED


i goet an esc1 
![[Pasted image 20260806120115.png]]


so lets take it


![[Pasted image 20260806132021.png]]


![[Pasted image 20260806213151.png]]

DONEEE