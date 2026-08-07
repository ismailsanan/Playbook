AD

Nmap 
```sh
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-07-28 22:30:31Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: timelapse.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: timelapse.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5986/tcp  open  ssl/wsmans?
|_ssl-date: 2026-07-28T22:32:00+00:00; +7h59m59s from scanner time.
| tls-alpn: 
|   h2
|_  http/1.1
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
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49695/tcp open  msrpc         Microsoft Windows RPC
58485/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-07-28T22:31:23
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
|_clock-skew: mean: 7h59m58s, deviation: 0s, median: 7h59m58s

```


since there is smb open on port 445 

check the default users
![[Pasted image 20260730143624.png]]


since there is a  non defualt share check it there is list of tools there i downloaded them  by `rget` command 


so there is an exe 
![[Pasted image 20260730150631.png]]


there is a public token in the config

![[Pasted image 20260730150806.png]]


checking  that userinfo is actually .net i used ilspy to decompile it 

![[Pasted image 20260730160351.png]]

thats intersting  

![[Pasted image 20260730160404.png]]


0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E




![[Pasted image 20260730162700.png]]


another intersting code that leaks username  try  support / ldap  for usernames


it didnt work now i investigated more the code and that enc_password is acutally encrpyted this way  since the authetnication part of ldap its basicallly calling password which called getpassword()

![[Pasted image 20260730165403.png]]


so basiclly array2 is byte of array[i] that is xored with armando [index % length] xored with 0xDF 


so i did python script with exactly what was written in .net
![[Pasted image 20260730172214.png]]

i got this 

![[Pasted image 20260730172239.png]]


nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz


we are in 

![[Pasted image 20260730172256.png]]


we have list of users  
![[Pasted image 20260730173144.png]]



ldap dump 
![[Pasted image 20260730173513.png]]


i found this

![[Pasted image 20260730174318.png]]

support / Ironside47pleasure40Watchful


lets check it out 


![[Pasted image 20260730174419.png]]


![[Pasted image 20260730174528.png]]


for priv esc now lets throw some bloodhound there 

![[Pasted image 20260730174800.png]]


lets check bloodhound for something intersting 


![[Pasted image 20260731100437.png]]

okay  we have genericAll on DC  

which has a session of admin 

![[Pasted image 20260731101318.png]]


so maybe we can abuse delegation here ?

lets check 

can create a machine 
![[Pasted image 20260731102709.png]]


![[Pasted image 20260731104910.png]]

lets check 

![[Pasted image 20260731105128.png]]

![[Pasted image 20260731105903.png]]

```
[*]     notevil$     (S-1-5-21-1677581083-3380853377-188903654-6101)

```

![[Pasted image 20260731110321.png]]


```
 export KRB5CCNAME=administrator@cifs_dc.support.htb@SUPPORT.HTB.ccache

if it didnt work write the fullpath/adminsitrator...
```


![[Pasted image 20260731111922.png]]