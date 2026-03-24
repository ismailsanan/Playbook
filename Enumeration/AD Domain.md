
for domain enumeration we can use the following tools

 - The ActiveDirectory PowerShell module (MS signed and works even in PowerShell CLM)
	https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps
	
	https://github.com/samratashok/ADModule
```
Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll
Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1 
```

- BloodHound (C# and PowerShell Collectors)
	 BloodHound Legacy - https://github.com/BloodHoundAD/BloodHound
	 BloodHound CE (Community Edition) - https://github.com/SpecterOps/BloodHound

- PowerView (PowerShell)
https://github.com/ZeroDayLab/PowerSploit/blob/master/Recon/PowerView.ps1
` C:\AD\Tools\PowerView.ps1`


-  SharpView (C#) - Doesn't support filtering 
 https://github.com/tevora-threat/SharpView/




Domain Enum ACL:


Enum Gpo:

```powershell

#List of Group Policy
Get-DomainGPO

#List of GPO applied on an OU 
AD> Get-DomainGPO -Identity '<gpLinkcn>'

 
```


Enum OU:

```powershell
Get-DomainOU
```



Enum Domain Trust:

```powershell


#get a list of all domain trust in the current domain
Get-DomainTrust 

Get-DomainTrust -Domain  <Domain>


AD> Get-ADTrust 


#details about the current forest
Get-Forest
Get-Forest -Forest <root domain>

#all domain in the current forest

Get-ForestDomain


#all global catalogs for the current forest
Get-ForestGlobalCatalog
Get-ForestGlobalCatalog -Forest <root domain>


#forest trust 
Get-ForestTrust

```


User Hunting: 
```powershell 
#Find  if a domain user has local access
Find-LocalAdminAccess


#find a computer where a domain admin  has a session on :

Find-DomainUserLocation -Verbose


```