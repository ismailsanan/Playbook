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