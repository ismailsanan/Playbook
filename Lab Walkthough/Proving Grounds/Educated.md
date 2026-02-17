```
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 98:4e:5d:e1:e6:97:29:6f:d9:e0:d4:82:a8:f6:4f:3f (RSA)
|   256 57:23:57:1f:fd:77:06:be:25:66:61:14:6d:ae:5e:98 (ECDSA)
|_  256 c7:9b:aa:d5:a6:33:35:91:34:1e:ef:cf:61:a8:30:1c (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Wisdom Elementary School
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

found a login portal
![[Pasted image 20260104180236.png]]

 - start dir enum for management also 


![[Pasted image 20260104180807.png]]

- that suits a bit our profile lets read about it a bit 

![[Pasted image 20260104180843.png]]

- so we need to some how to upload a php file that we can run though the subdir `exam_question`


```
/login                (Status: 200) [Size: 6374]
/uploads              (Status: 301) [Size: 329] [--> http://192.168.223.13/management/uploads/]
/admin                (Status: 200) [Size: 0]
/assets               (Status: 301) [Size: 328] [--> http://192.168.223.13/management/assets/]
/system               (Status: 403) [Size: 279]
/Login                (Status: 200) [Size: 6374]
/README               (Status: 200) [Size: 66]
/js                   (Status: 301) [Size: 324] [--> http://192.168.223.13/management/js/]
/dist                 (Status: 301) [Size: 326] [--> http://192.168.223.13/management/dist/]
/application          (Status: 403) [Size: 279]
/payment              (Status: 200) [Size: 0]
/installation         (Status: 301) [Size: 334] [--> http://192.168.223.13/management/installation/]
/$FILE                (Status: 400) [Size: 1134]
/$file                (Status: 400) [Size: 1134]
/Admin                (Status: 200) [Size: 0]
/$File                (Status: 400) [Size: 1134]
/video games          (Status: 403) [Size: 279]
/msgReader$1          (Status: 400) [Size: 1134]
/spyware doctor       (Status: 403) [Size: 279]
/nero 7               (Status: 403) [Size: 279]
/cell phones          (Status: 403) [Size: 279]
/long distance        (Status: 403) [Size: 279]
/cable tv             (Status: 403) [Size: 279]
```


 - we still need to access this portal in some way

we found the db.sql but no credentials were usefull 

![[Pasted image 20260104185849.png]]

- we noticed in the rce we had the whole request ready lets try to copy paste it  and see if we can do an unauthurized request of file upload

![[Pasted image 20260104191355.png]]

it worked !
i edited the payload again using pentestmonkey revshell 

![[Pasted image 20260104192236.png]]

 --PRIV ESCALATIOn -- 

- local.txt is found in msander but we cant read it  no permession


linpeas showed that 

![[Pasted image 20260104194254.png]]

- checked for some credentials 
- check for /etc/mysql

found intersting file `database connectivity settings`

lets try and connect 

![[Pasted image 20260104194217.png]]


connected to db using `mysql` locally and started enum 

![[Pasted image 20260104200706.png]]


found sander 

![[Pasted image 20260104200826.png]]

cracked password using crackstation

`greatteacher123`

`su msander`

![[Pasted image 20260104200946.png]]


--- PRIV SEC ----
- in emiller we found an apk 
- tried to send the file back to the us and extract some passwrds using `strings`
- there was too much data 
- i found some intersting tools to inverstigate apparently everyone uses https://github.com/MobSF/Mobile-Security-Framework-MobSF  for apk researching so lets check it out  

