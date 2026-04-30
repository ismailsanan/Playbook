

use winrs when we want to connect to another server with the same credentials

```
winrs -r:dcorp-adminsrv cmd
 
Enter-PSSession -ComputerName dcorp-adminsrv.dollarcorp.moneycorp.local
```

use runas to connect to different users in the domain 

PS>
`$env:computername`
`$env:username`


CMD> 
`set username`
`set computername`