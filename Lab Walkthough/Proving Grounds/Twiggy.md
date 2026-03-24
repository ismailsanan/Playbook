
```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 44:7d:1a:56:9b:68:ae:f5:3b:f6:38:17:73:16:5d:75 (RSA)
|   256 1c:78:9d:83:81:52:f4:b0:1d:8e:32:03:cb:a6:18:93 (ECDSA)
|_  256 08:c9:12:d9:7b:98:98:c8:b3:99:7a:19:82:2e:a3:ea (ED25519)
53/tcp   open  domain  NLnet Labs NSD
80/tcp   open  http    nginx 1.16.1
|_http-server-header: nginx/1.16.1
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
|_http-favicon: Unknown favicon MD5: 11FB4799192313DD5474A343D9CC0A17
|_http-title: Home | Mezzanine
4505/tcp open  zmtp    ZeroMQ ZMTP 2.0
4506/tcp open  zmtp    ZeroMQ ZMTP 2.0
8000/tcp open  http    nginx 1.16.1
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: nginx/1.16.1
|_http-title: Site doesn't have a title (application/json).
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS

```


![[Pasted image 20251223172024.png]]

i noticed on port 8000 that there is an x upstream for salt stack

![[Pasted image 20251223172139.png]]

i tried with different ports like 1337 and so they were all getting blocked so i started using ports that the firewall wont block like port 4505 , 4506 since there is a service inside the victim open  
![[Pasted image 20251223183344.png]]


![[Pasted image 20251223183515.png]]