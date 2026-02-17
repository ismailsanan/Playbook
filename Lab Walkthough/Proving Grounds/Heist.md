


![[Pasted image 20251230131543.png]]


- port 8080 has basically a web page that takes in an URL so we start with trying some ssrf attacks
- i opened an http server  and tried inserting my ip to test for vuln
![[Pasted image 20251230131702.png]]

- it is vulnerable 
- lets try inserting a revshell and then try to call it though localhost  - no luck 
- since we are in an active dir environment we can try to authenticate to us since  Windows sees it as a network authentication request Windows may automatically attempt **Integrated Windows Authentication (IWA)** using **NTLM**. hence it starts an NTML challenge response


![[Pasted image 20251230141859.png]]


![[Pasted image 20251230142105.png]]


`enox:california`


![[Pasted image 20251230142255.png]]

![[Pasted image 20251230142359.png]]


![[Pasted image 20251231184848.png]]

![[Pasted image 20251231184832.png]]


-  we have a 2 hashes 1 old and the current one 
- we are using the current one we can either crack it or pass the hash  so lets go with pass the hash since its simpler 


![[Pasted image 20251231190147.png]]

![[Pasted image 20251231191654.png]]

![[Pasted image 20251231191903.png]]

- after having the desktop  we press win+U to obtain the shell 

![[Pasted image 20251231192720.png]]