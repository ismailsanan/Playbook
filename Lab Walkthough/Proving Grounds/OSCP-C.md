
192.168.181.153 - ms1
10.10.141.154 - ms2 
10.10.141.152 - DC01 


![[Pasted image 20260114212823.png]]



![[Pasted image 20260114213223.png]]


![[Pasted image 20260114214604.png]]

![[Pasted image 20260114214619.png]]


![[Pasted image 20260114222151.png]]

with local auth  since apparently ldap is not working in that domain

![[Pasted image 20260114223153.png]]


![[Pasted image 20260114223345.png]]



![[Pasted image 20260114224252.png]]

found  what seams to be a backup folder in `c:\windows.old`

we downloaded `system` and `sam` from `system32`

![[Pasted image 20260114224845.png]]



![[Pasted image 20260114225314.png]]

since its the DC i ran the pass spray 2 times first with localaiuth second with domain and it  worked with the domain 
![[Pasted image 20260114225929.png]]


![[Pasted image 20260114230840.png]]