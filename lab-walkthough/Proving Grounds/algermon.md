

![[Pasted image 20251218105006.png]]



![[Pasted image 20251218105038.png]]
checking the ftp port we found a list of logs

![[Pasted image 20251218110009.png]]

and it seams that there is smtp and pop in the system so we will do a udp scan 


![[Pasted image 20251218110542.png]]

nothing intersting 

lets check the web smartermail for cve

![[Pasted image 20251218111633.png]]

for some reason the code had like invalid formats with <0x00b> i removed those and it worked 

![[Pasted image 20251218112123.png]]