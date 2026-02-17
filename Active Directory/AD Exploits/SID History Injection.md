

feature The focus of the **SID History Injection Attack** is aiding **user migration between domains** while ensuring continued access to resources from the former domain.
Adversaries may use SID-History Injection to escalate privileges and bypass access controls. The Windows security identifier (SID) is a unique value that identifies a user or group account. SIDs are used by Windows security in both security descriptors and access tokens. An account can hold additional SIDs in the SID-History Active Directory attribute  allowing inter-operable account migration between domains (e.g., all values in SID-History are included in access tokens).

With Domain Administrator (or equivalent) rights, harvested or well-known SID values [[3]] may be inserted into SID-History to enable impersonation of arbitrary users/groups such as Enterprise Administrators. This manipulation may result in elevated access to local resources and/or access to otherwise inaccessible domains via lateral movement techniques such as [Remote Services](https://attack.mitre.org/techniques/T1021), [SMB/Windows Admin Shares](https://attack.mitre.org/techniques/T1021/002), or [Windows Remote Management](https://attack.mitre.org/techniques/T1021/006).

the attacker must have the ability to write the `SIDHistory` attribute for a user. This is typically achieved by compromising an account that has:

- The **`SeEnableDelegationPrivilege`** (often held by Domain Controllers via their `Account is trusted for delegation` setting).
    
- Or, more directly, the **`mS-DS-MachineAccountQuota`** attribute allows any authenticated user to create up to 10 computer accounts by default. An attacker can create a computer account and then use it in a Kerberos-based attack (like **DCSync**) to obtain the necessary privileges.
- 
user A domain ad  move or recreate  to other domain  so idealy if we grab sid we can access resources to previous one grab from B access in domain A


```powershell
#find the SID of a group of the other domain

Get-DomainGroup -Identity "Domain Admins" -Domain parent.io -Properties ObjectSid

```

we can then either perform DCSync , Golden Ticket and so 

### Refernces

- https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/sid-history-injection.html
- https://attack.mitre.org/techniques/T1134/005/