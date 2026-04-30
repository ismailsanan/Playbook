
**Fileless Remote execution**

![[Pasted image 20260416172857.png]]

use mshta for downloading ffiles  since `mshta.exe` is the legitimate Windows utility for running HTML Applications (HTA) located in `C:\Windows\System32\`. Because it is signed and published by Microsoft, Unlike traditional malware that requires dropping a file to disk (where it can be scanned by antivirus), `mshta.exe` can execute scripts directly from a remote URL or an inline command without saving anything locally. This "fileless" approach keeps the payload in memory, which significantly reduces the forensic footprint and bypasses disk-based detection engines