
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1:99:4b:95:22:25:ed:0f:85:20:d3:63:b4:48:bb:cf (RSA)
|   256 0f:44:8b:ad:ad:95:b8:22:6a:f0:36:ac:19:d0:0e:f3 (ECDSA)
|_  256 32:e1:2a:6c:cc:7c:e6:3e:23:f4:80:8d:33:ce:9b:3a (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: 09BDDB30D6AE11E854BFF82ED638542B
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-robots.txt: 7 disallowed entries 
| /backup/ /cron/? /front/ /install/ /panel/ /tmp/ 
|_/updates/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://exfiltrated.offsec/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```


- gobuster couldn't provide results


interesting subdirs in http 

```
/backup/ /cron/? /front/ /install/ /panel/ /tmp/ /updates/
```


- we have a panel login `/panel`

` /updates/ /install ` ->  forbidden 

`admin:admin` for the login in `/panel`

![[Pasted image 20260105114841.png]]

- info leak `Subrion CMS  v 4.2.1`
- lets check some known cve 

Found a CVE  for arbitrary file upload 
![[Pasted image 20260105114854.png]]

- i was lazy i tried to execute that cve immediatly but it seams return a credentials error even when supplying a correct credentials 
- so lets read the concept of the cve and apply it manually

we can confirm that we can upload files and execute them by double clicking on the uploaded file 
![[Pasted image 20260105115352.png]]

- upload a simple php rev shell  with the extension of `.phar`
- call it `exfiltrated.offsec/uploads/notvil.phar?cmd=id`

worked
![[Pasted image 20260105120245.png]]

-  now lets get a revshell with more sofesticated  payload ill be using the `pentestMoney` revshell

![[Pasted image 20260105120522.png]]


----- PRIV SEC -----

- only 1 user `coaran` no  permission to access it 


- lunched linpeas 

>check 
```
Services with writable paths 
```

- nothing really intersting in linpeas 
- started pspy to monitor processes and i saw this
![[Pasted image 20260105133110.png]]

 - after investigating i saw that `/op/image...` is root   we dont have permessions to write on them 


![[Pasted image 20260105133309.png]]
- lets check for some cve on exiftool 
![[Pasted image 20260105134454.png]]

```py
# Description: Improper neutralization of user data in the DjVu file format in ExifTool versions 7.44 and up allows arbitrary code execution when parsing the malicious image
```


downloaded this it didnt work for some reason so i searched for another explout and then figured out we needed to install `djvulibre-bin`  for poper execution


- exeuted the exploit it generated a `image.jpg` file 
- uploaded it into the proper path

DONE !!!
![[Pasted image 20260105140411.png]]
