UAC (User Account Control) is a Windows feature that prevents applications from running with full administrative rights by default.it only elevate to administrative privileges after user consent through a UAC prompt.

MIC (Mandatory Integrity Control) is the operating system mechanism that enforces separationdn between processes a objects based on integrity levels. Each process or object is assigned an integrity level such as Low, Medium, High, or System. A lower-level process cannot write to or modify a higher-level one, regardless of whether both belong to the same user.

The relationship between UAC and MIC is that UAC provides the prompt and user interaction needed to grant elevation, while MIC enforces the boundaries between processes of different integrity levels. Together they ensure that applications cannot silently escalate privileges or tamper with higher-trust processes without explicit user approval.


Check current user with `whoami /user`

Check admin group membership: `net localgroup administrators`

Check privileges: `whoami /priv`

  If you get a 1, then UAC is enabled. Otherwise is disabled.
```sh
   Get-ItemProperty -Path 'HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\System' | Select-Object EnableLUA
   reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA
```


Check UAC level:

```sh
Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System | Select-Object ConsentPromptBehaviorAdmin
```

If its is 0x5 then `Always notify is enabled`