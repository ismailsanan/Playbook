AD - Medium 


```sh
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-04 14:33:30Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: administrator.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: administrator.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
51237/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
51242/tcp open  msrpc         Microsoft Windows RPC
51245/tcp open  msrpc         Microsoft Windows RPC
51262/tcp open  msrpc         Microsoft Windows RPC
58527/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
|_clock-skew: 7h00m00s
| smb2-time: 
|   date: 2026-08-04T14:34:22
|_  start_date: N/A

NSE: Script Post-scanning.

```


there is basically 2 http services  with 404 not found home page lets see what can we find

i ran the DIR and files  scan 
![[Pasted image 20260804094543.png]]



Domain name: administrator.htb


ldap -> KO
smb -> KO 
http:
	5985 -> KO 
	47001 -> KO

ftp -> KO



my stupid ass didnt see the credentials given by the machine 

olivia / ichliebedich



drop bloodhound 

![[Pasted image 20260804101542.png]]


![[Pasted image 20260804103502.png]]



![[Pasted image 20260804103514.png]]


intersting groups

![[Pasted image 20260804103729.png]]


so lets start with first genericAll

![[Pasted image 20260804134451.png]]

for some reason that didnt work probalbly cz of the timing lets try windows

![[Pasted image 20260804141258.png]]

![[Pasted image 20260804141211.png]]


![[Pasted image 20260804141200.png]]


michael /  Password123!


![[Pasted image 20260804141619.png]]


we did the same for benjamin

![[Pasted image 20260804141856.png]]



benjamin /  Password123!

it doesnt have anything outbound in the AD so lets check some protocols to access , winpeas and so 

![[Pasted image 20260804142233.png]]




![[Pasted image 20260804143139.png]]\


![[Pasted image 20260804143826.png]]

![[Pasted image 20260804143941.png]]

backu / tekieromucho

![[Pasted image 20260804144610.png]]

![[Pasted image 20260804144714.png]]

emily is member or RM so lets go with that first 


emily / UXLCI5iETUsIBoFVTj8yQFKoHjXmb

alexander / UrkIbagoxMyUGw0aPlj9B0AXSea4Sw

emma / WwANQWnmJnGV07WQN8bMS7FMAbjNur


![[Pasted image 20260804145527.png]]


![[Pasted image 20260804145515.png]]

Bingo 

lets take down ethan first 

we have genericWrite lets go with teh easy way target kerberoastable set ethan to spn then request ticket anc crack it for the password 


![[Pasted image 20260804150044.png]]

get-domainuser

![[Pasted image 20260804150031.png]]


![[Pasted image 20260804151211.png]]


copied all of it from hash till bottom for sure i removed new lines and space


![[Pasted image 20260804151925.png]]

ethan / limpbizkit

![[Pasted image 20260804152058.png]]


lets dump stuff 

![[Pasted image 20260804152125.png]]



![[Pasted image 20260804152227.png]]


![[Pasted image 20260804152306.png]]

DONE 

