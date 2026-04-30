
Kerberos Delegation allows a service to impersonate a user to access another resource. Authentication is delegated, and the final resource responds to the service as if it had the first user's rights.


### Unconstrained delegation

This a feature that a Domain Administrator can set to any **Computer** inside the domain. Then, anytime a **user logins** onto the Computer, a **copy of the TGT** of that user is going to be **sent inside the TGS** provided by the DC **and saved in memory in LSASS**. So, if you have Administrator privileges on the machine, you will be able to **dump the tickets and impersonate the users** on any machine.

So if a domain admin logins inside a Computer with "Unconstrained Delegation" feature activated, and you have local admin privileges inside that machine, you will be able to dump the ticket and impersonate the Domain Admin anywhere (domain privesc)



![[unconstrained_delegation_setting.webp]]
Only an administrator or a privileged user to whom these privileges have been explicitly given can set this option to other accounts. More specifically, it is necessary to have the SeEnableDelegationPrivilege privilege to perform this action. A service account cannot modify itself to add this option.

We can achieve the same output with the AD module by running the `**Get-ADObject**` cmdlet and filtering for the `**msds-allowtodelegate**` property.

```sh

#powerview
## A DCs always appear and might be useful to attack a DC from another compromised DC from a different domain (
 Get-DomainComputer -Unconstrained -Properties name
 

# Export tickets with Mimikatz
## Access LSASS memory
privilege::debug
sekurlsa::tickets /export #Recommended way
kerberos::list /export #Another way

# Monitor logins and export new tickets
## Doens't access LSASS memory directly, but uses Windows APIs
Rubeus.exe dump
Rubeus.exe monitor /interval:10 [/filteruser:<username>] #Check every 10s for new TGTs
```
### Constrained delegation

Using this a Domain admin can **allow** a computer to **impersonate a user or computer** against any **service** of a machine.

- **Service for User to self (_S4U2self_):** If a **service account** has a _userAccountControl_ value containing [TrustedToAuthForDelegation](https://msdn.microsoft.com/en-us/library/aa772300\(v=vs.85\).aspx) (T2A4D), then it can obtain a TGS for itself (the service) on behalf of any other user.

- **Service for User to Proxy(_S4U2proxy_):** A **service account** could obtain a TGS on behalf any user to the service set in **msDS-AllowedToDelegateTo.** To do so, it first need a TGS from that user to itself, but it can use S4U2self to obtain that TGS before requesting the other one.


```bash
# Powerview
Get-DomainUser -TrustedToAuth | select userprincipalname, name, msds-allowedtodelegateto
Get-DomainComputer -TrustedToAuth | select userprincipalname, name, msds-allowedtodelegateto
```


### Resource-based Constrained Delegation


his is similar to the basic [Constrained Delegation](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/constrained-delegation.html) but **instead** of giving permissions to an **object** to **impersonate any user against a machine**. Resource-based Constrain Delegation **sets** in **the object who is able to impersonate any user against it**.

he constrained object will have an attribute called _**msDS-AllowedToActOnBehalfOfOtherIdentity**_ with the name of the user that can impersonate any other user against it.

Another important difference from this Constrained Delegation to the other delegations is that any user with **write permissions over a machine account** (_GenericAll/GenericWrite/WriteDacl/WriteProperty/etc_) can set the **_msDS-AllowedToActOnBehalfOfOtherIdentity_** (In the other forms of Delegation you needed domain admin privs).
### References 

https://medium.com/r3d-buck3t/attacking-kerberos-constrained-delegations-4a0eddc5bb13
https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/unconstrained-delegation.html