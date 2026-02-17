
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

