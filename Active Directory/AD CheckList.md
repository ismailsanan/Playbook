- ldap ? 
	- check for usernames ldapsearch
- web ? 
	- check for some rce
	- check for list of usernames that we can bruteforce 
- smb ? 
	- check for anonymous usrnames access 'guest, anonymous'
	-  check for some leaked version in nmap  maybe there could be some vuln
	- `--rid-brute`

- enum4linux
- rdp ?
	- check for null username access 
	- if there is a list of username try to bruteforce password  using hydra 

 - last resort  is brute force usernames on laps using **kerbrute**
	 - add @domain to usernames
	 - using `username-anarchy ` if we have names  we can create a list of possible usernames  like `firt.last `or `first.last@domain` 


we have a valid credentials ?
	- try kerboraosting 
	- try arproasting
	- try using `nxc`  all protocols 


- drop sharphound
	- check for kerberoastable users
	- all domain admins
	- arproasting
	- paths from owned users to other pc
- mimikatz 
	- lsadump::secrets
	- sekurlsa::logonpasswords
	- sekurlsa::logonpasswords
	- sekurlsa::tickets
	- if we find any crednetials save them and pass them through the whole network
	- check for default passwords and spray it agains users including   net user /domain


- drop ligolo for pivoting