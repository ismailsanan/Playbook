

```sh
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2021-03-17 17:46  grav-admin/
|_
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Index of /
Service Info: Host: 127.0.0.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```


subdir enum
```
/cache (Status: 301)
/images (Status: 301)
/tmp (Status: 301)
/bin (Status: 301)
/user (Status: 301)
/admin (Status: 200)
/login (Status: 200)
/logs (Status: 301)
/backup (Status: 301)
/assets (Status: 301)
/home (Status: 200)
/system (Status: 301)
/vendor (Status: 301)
/forgot_password 
```


intersting page

![[Pasted image 20260107101631.png]]


- searching for some CVE i found a potential rce
![[Pasted image 20260107102618.png]]


so i tried this poc

- https://github.com/CsEnox/CVE-2021-21425?tab=readme-ov-file

what it does it schedule a task in the system its a vuln based on the cms config schedule fil
so to test it i simple inserted a command wget to my http server

WORKED !
![[Pasted image 20260107103606.png]]

- lets call a rev shell  i used simply bash -i from revshell 


-------- priv esc ----------

what is that in visitor.json 
```
{"6557ad1f05a2d4cb840b13eb6452fc923989b7be":1767778077}
```


in `user/account` admin password 
```sh
email: admin@gravity.com
access:
  admin:
    login: true
    super: true
  site:
    login: true
fullname: admin
title: null
hashed_password: $2y$10$dlTNg17RfN4pkRctRm1m2u8cfTHHz7Im.m61AYB9UtLGL2PhlJwe.
pw_resets:
  - 1767777403
reset: '40d9d2f85eed6aabe256d83b61c23c8e::1768382203'


```


- couldn't crack the hash

intersting suid

![[Pasted image 20260107113429.png]]


WORKED !

![[Pasted image 20260107113554.png]]