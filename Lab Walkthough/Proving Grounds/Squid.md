192.168.113.189

```sh
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3128/tcp  open  http-proxy    Squid http proxy 4.14
|_http-server-header: squid/4.14
|_http-title: ERROR: The requested URL could not be retrieved
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2026-01-07T11:01:06
|_  start_date: N/A

```

- smb -> nothing
- we can check what is squid 4.14 and if there is some cve 
- subdir enum -> nothing  


- found a page on `hacktricks`

using spose that was suggested by hacktrics we discovered 

![[Pasted image 20260108104621.png]]


![[Pasted image 20260108110410.png]]

confirming this we go to burp and configure it to make our life easier 

we basically are configuring to the proxy server to palce a rule that applies to the host it uses so the host should be pointing at IP:3128 
![[Pasted image 20260108111731.png]]

![[Pasted image 20260108111821.png]]

- now to enumerate  subdir or files we always point it to the burp to ensure the rules apply 

![[Pasted image 20260108112246.png]]

- phpmyadmin was found it could be intersting since 3306 mysql was detected 

![[Pasted image 20260108112327.png]]

- we tried the default credentials and it worked `root: `

- now lets try to find an rce or something to get in the system 
- so we searched online for shells using phpmyadmin

found  a media page : `https://medium.com/@toon.commander/uploading-a-shell-in-phpmyadmin-61b066b481a7`

![[Pasted image 20260108115314.png]]

![[Pasted image 20260108115324.png]]

![[Pasted image 20260108115232.png]]

now we make a sofiticated revshell
![[Pasted image 20260108123252.png]]

- upload it using certurtil
- then call it

![[Pasted image 20260108123227.png]]

flag was in `c:\ `
![[Pasted image 20260108123412.png]]
### Reference 

- https://book.hacktricks.wiki/en/network-services-pentesting/3128-pentesting-squid.html