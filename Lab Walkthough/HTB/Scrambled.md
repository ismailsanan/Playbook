AD-Medium


```sh
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: Scramble Corp Intranet
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-06 12:21:28Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: scrm.local, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC1.scrm.local
| Issuer: commonName=scrm-DC1-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2024-09-04T11:14:45
| Not valid after:  2121-06-08T22:39:53
| MD5:     2ca2 5511 c96e d5c5 3601 17f2 c316 7ea3
| SHA-1:   9532 78bb e082 70b2 5f2e 7467 6f7d a61d 1918 685e
|_SHA-256: 92c7 0b9a 427f b479 ef48 1832 0275 b752 f64c 850b 0c08 cee0 71e4 afc8 55dd 2f87
|_ssl-date: 2026-08-06T12:23:01+00:00; -3h55m06s from scanner time.
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: scrm.local, Site: Default-First-Site-Name)
|_ssl-date: 2026-08-06T12:23:01+00:00; -3h55m06s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC1.scrm.local
| Issuer: commonName=scrm-DC1-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2024-09-04T11:14:45
| Not valid after:  2121-06-08T22:39:53
| MD5:     2ca2 5511 c96e d5c5 3601 17f2 c316 7ea3
| SHA-1:   9532 78bb e082 70b2 5f2e 7467 6f7d a61d 1918 685e
|_SHA-256: 92c7 0b9a 427f b479 ef48 1832 0275 b752 f64c 850b 0c08 cee0 71e4 afc8 55dd 2f87
1433/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-info: 
|   10.129.41.191:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-08-06T12:20:13
| Not valid after:  2056-08-06T12:20:13
| MD5:     c845 df7c 5b4d d1b2 c4f2 f7ca bb6f 1f8b
| SHA-1:   4969 4cae de27 4363 0468 08d5 3e59 3836 3f3b 3645
|_SHA-256: f692 46c5 6e86 b2e9 a0c5 35d6 86e6 01af 807e dd60 932f d02d 3f38 dcb6 ccd9 f2fe
|_ssl-date: 2026-08-06T12:23:01+00:00; -3h55m06s from scanner time.
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: scrm.local, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC1.scrm.local
| Issuer: commonName=scrm-DC1-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2024-09-04T11:14:45
| Not valid after:  2121-06-08T22:39:53
| MD5:     2ca2 5511 c96e d5c5 3601 17f2 c316 7ea3
| SHA-1:   9532 78bb e082 70b2 5f2e 7467 6f7d a61d 1918 685e
|_SHA-256: 92c7 0b9a 427f b479 ef48 1832 0275 b752 f64c 850b 0c08 cee0 71e4 afc8 55dd 2f87
|_ssl-date: 2026-08-06T12:23:01+00:00; -3h55m06s from scanner time.
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: scrm.local, Site: Default-First-Site-Name)
|_ssl-date: 2026-08-06T12:23:01+00:00; -3h55m06s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC1.scrm.local
| Issuer: commonName=scrm-DC1-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2024-09-04T11:14:45
| Not valid after:  2121-06-08T22:39:53
| MD5:     2ca2 5511 c96e d5c5 3601 17f2 c316 7ea3
| SHA-1:   9532 78bb e082 70b2 5f2e 7467 6f7d a61d 1918 685e
|_SHA-256: 92c7 0b9a 427f b479 ef48 1832 0275 b752 f64c 850b 0c08 cee0 71e4 afc8 55dd 2f87
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: Host: DC1; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-08-06T12:22:17
|_  start_date: N/A
|_clock-skew: mean: -3h55m06s, deviation: 1s, median: -3h55m06s

```


![[Pasted image 20260806190003.png]]

something stinks in this 

![[Pasted image 20260806190043.png]]

ntlm authentication doesnt work so we neeed to check purely for tgt ticket for kerberos



![[Pasted image 20260806200924.png]]

! it says we will reset your password to be the same as the username

we have email structure leak
`support@scramblecorp.com`

domain: `scrm.local`

potential user `ksimpson`

![[Pasted image 20260806190257.png]]

dir enum -> KO


A log file named `ScrambleDebugLog`

![[Pasted image 20260806170244.png]]

since ntlm is not working we only have klerberos option meaning getTGT or nxc -k  according to the msg in reset password i spreayed the same worlist in the username and password

i used the list in kerbrute to see which uernames are valid and updated my worlist with those only 

i got a scw errot the dame time again
![[Pasted image 20260806170111.png]]
fixed it it ntupdate 

![[Pasted image 20260806170102.png]]

![[Pasted image 20260806170657.png]]


`export KRB5CCNAME=ksimpson.ccache`

![[Pasted image 20260806170826.png]]

now we enum the domain in nxc using the ticket

`nxc smb 10.129.41.191 -k --use-kcache`



![[Pasted image 20260806171949.png]]


downloading 

![[Pasted image 20260806172129.png]]


![[Pasted image 20260806173057.png]]

hint for our next move maybe ?



list of users

![[Pasted image 20260806174014.png]]


ill just spray that again 


i dropped bloodhound 


![[Pasted image 20260806175951.png]]


we have a kerberostable user

![[Pasted image 20260807103307.png]]

that potentially would lead us to mssql since it basically mentioened it before lets take it and crack the pass


![[Pasted image 20260807103633.png]]


cracked it using hashcat


sqlsvc / Pegasus60


i asked again the tgt 
![[Pasted image 20260807103905.png]]

no luck with mssql 

but the note said only admins are allowed so we have service account lets do a silver ticket


bloodhound:
`S-1-5-21-2743207045-1827831105-2542523200`


NThash used cyberchef "NT hash" of the password SA

`b999a16500b87d17ec7f2e2a68778f05`



![[Pasted image 20260807112804.png]]


we are in 

![[Pasted image 20260807112936.png]]


![[Pasted image 20260807113547.png]]




lets check first some db first 

![[Pasted image 20260807114311.png]]

since the pdf mentioned HR i selected the HR db 

![[Pasted image 20260807114254.png]]



![[Pasted image 20260807114245.png]]
member of remote management 


MiscSvc /  ScrambledEggs9900

lets get a tgt ticket and check if we can access it 

now to use evilwirm need a configurated kerberos in my machine

Kerberos config issue ->  evil-winrm doesn't know where the KDC is for `SCRM.LOCAL`. Fix `/etc/krb5.conf`.

```sh
sudo tee /etc/krb5.conf > /dev/null <<'EOF'
[libdefaults]
    default_realm = SCRM.LOCAL
    dns_lookup_kdc = false
    dns_lookup_realm = false

[realms]
    SCRM.LOCAL = {
        kdc = dc1.scrm.local
        admin_server = dc1.scrm.local
    }

[domain_realm]
    .scrm.local = SCRM.LOCAL
    scrm.local = SCRM.LOCAL
EOF
```

![[Pasted image 20260807120233.png]]

DONE 


lets check this account around

![[Pasted image 20260807121312.png]]

i need an rdp for that maybe also rev engineer it maybe with wine ?


![[Pasted image 20260807130411.png]]

its .net lets decompile it first 

![[Pasted image 20260807131546.png]]

nothing intersting in the .exe lets check the .dll

that is intersting 

![[Pasted image 20260807131611.png]]

if we have username scrmdev we bascially are good to go 

i dont feel like switching to windows now ill do this later 