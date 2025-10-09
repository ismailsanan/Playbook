
Manual  Enumeration 

```sh
# Current user info.
C> whoami
C> whoami /groups
C> whoami /priv

# Current user info (PS).
PSC> powershell
PSC> Get-LocalUser
PSC> Get-LocalGroup
PSC> Get-LocalGroupMember Administrators

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
PSC> Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue 
PSC> Get-ChildItem -Path C:\xampp -Include *.txt,*.ini -File -Recurse -ErrorAction SilentlyContinue
# AD.
PSC> Get-ChildItem -Path C:\Users\ -Include *.txt, *.pdf, *.xls, *.xlsx, *.doc, *.docx, *.ppt, *.pptx, *.odt, *.ods, *.odp, *.ps, *.log, *.ini, *.cfg, *.conf, *.json, *.xml, *.yaml, *.yml, *.md, *.accdb, *.pub, *.one, *.onepkg, *.psd, *.ai, *.indd, *.jpg, *.jpeg, *.png, *.gif, *.bmp, *.tiff, *.ps1, *.bat, *.cmd, *.sh, *.py, *.java, *.cpp, *.c, *.cs, *.js, *.ts, *.html, *.css, *.rb, *.php, *.csv, *.sql, *.zip, *.rar, *.7z, *.tar, *.gz, *.bz2, *.mp3, *.wav, *.mp4, *.avi, *.mov, *.wmv, *.epub, *.mobi -File -Recurse -ErrorAction SilentlyContinue | Out-File C:\temp\out.txt | Select-String -Pattern "NTLM" | Select-Object -Unique Path

# User's powershell history.
PSC:\Users\dave> Get-History
PSC:\Users\dave> (Get-PSReadlineOption).HistorySavePath
PSC:\Users\dave> type C:\Users\dave\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```


```sh
#Enable RDP
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f


#Turn off firewall
netsh advfirewall set allprofiles state off
```
### pspy64

Tool that Enumerate processes without root permession

Steps: 
- Enumerate arch to know which binary we need `uname -a`
- download the correct binary
- transfer it from attacker machine `python server`
- execute it on victim machine



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

net user
net user <User>

#auth Checks
icalcs <file>

# Grant Permissions
icalcs  test.txt /grant hostname\username:R /t /c
```



### Serivce DLL ENUM

```sh
Listdlls64.exe  /acceteula <serviceName>
```

- https://learn.microsoft.com/en-us/sysinternals/downloads/listdlls
- https://learn.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-search-order
- https://malicious.link/posts/2020/compiling-a-dll-using-mingw/