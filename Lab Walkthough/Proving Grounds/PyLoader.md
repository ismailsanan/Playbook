nmap scan

![[Pasted image 20251209181759.png]]


i seen that there is a tcp port  on 9666 i accessed it though burp and found this

![[Pasted image 20251209181934.png]]


checking online  i ound out tehre is a pre auth rce wich is literally perfect since we dont have any credentials

![[Pasted image 20251209182037.png]]

in the payload its indicator of a success is accessing  `/flash/addcrypted2`

![[Pasted image 20251209182121.png]]

if this is page its reached or available it means its possible 

running the exploit indicated its success but its blind exploit nothing returns to us so the only way reverse shell or anything frtom the victim to contact us


![[Pasted image 20251209182238.png]]


tried couple only busybox worked  with root account 

>PWNED !
![[Pasted image 20251209182440.png]]