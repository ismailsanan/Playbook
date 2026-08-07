- recon  ?
    - full port scan then service scan on whats open
    - grab the domain name + dc hostname from the scan
    - add domain + dc to hosts file
    - point dns at the DC if kerberos stuff acts weird

- ldap ? (389 / 636 ssl / 3268 gc / 3269 gc ssl)
    - check for usernames ldapsearch
    - try anonymous bind first, dump naming contexts
    - dump all users and groups
    - ALWAYS check the description field, passwords get left there
    - check for password-not-required accounts

- web ? (80 / 443 / 8080 / 8443)
    - check for some rce
    - check for list of usernames that we can bruteforce 
    - check default creds on any login panel
    - look for lfi / upload / cms version
    - dir brute and note anything that leaks names or versions

- smb ? (139 / 445)
    - check for anonymous usrnames access 'guest, anonymous'
    - list shares and check read/write perms
    - spider shares for creds / scripts / configs
    - check for some leaked version in nmap maybe there could be some vuln
    - eternalblue / ms17-010 if the version is old
    - `--rid-brute` check also with a valid user like guest or ''
    - great when ldap is locked down


- enum4linux
    - run it , pulls users / groups / shares / password policy in one shot

- rdp ? (3389)
    - check for null username access
    - if there is a list of username try to bruteforce password using hydra
    - careful with lockout policy

- winrm ? (5985 / 5986 ssl)
    - check for anonymous usrnames access 'guest, anonymous'

- kerberos ? (88)
    - confirms its a DC, use it for username enum and roasting

- dns ? (53)
    - useful for zone transfer and finding hosts

- Collect usernames from website or any pace we can collect like ldap --rid-brute in smb
    - brute force usernames on laps using kerbrute (fast, no lockout)
    - using `username-anarchy` if we have names we can create a list of possible usernames 


- valid usersname ? (no creds yet)
    - try arproasting (asrep, accounts that dont require preauth)

- we have a valid credentials ?
    - try kerboraosting (spn accounts) then crack offline
    - try using `nxc` all protocols, hunting for a pwned box
    - check the password policy before spraying anything

- drop sharphound

    - check for kerberoastable users
    - all domain admins
    - arproasting (asreproastable users)
    - paths from owned users to other pc, mark owned and run shortest path to domain admins
    - check the group of the owned user maybe something intersting 

- mimikatz (need local admin / system first)
    - lsadump::secrets
    - sekurlsa::logonpasswords
    - sekurlsa::tickets
    - lsadump::sam
    - lsadump::lsa
    - if we find any crednetials save them and pass them through the whole network
    - check for default passwords and spray it agains users including net user /domain

- pass stuff around (lateral movement)
    - pass the hash to psexec / wmiexec / smbexec / winrm
    - pass the ticket if you exported one
    - reuse cracked passwords everywhere, people recycle


- drop ligolo for pivoting
    - start the proxy on kali and bring up the tun interface
    - run the agent on the pivot host, connect back
    - start the session and add a route to the internal subnet
    - now scan / hit the internal net straight through



**others**

- mssql ? (1433) - check for creds reuse, xp_cmdshell, impersonation
- ftp ? (21) - anonymous login, look for configs / creds
- ssh ? (22) - creds reuse
- smtp ? (25) - user enum with vrfy / rcpt
- wsus / other web (8530) - if unencrypted could poison updates
- netbios name (137 / 138



## OLD


- ldap ?
    - check for usernames ldapsearch
- web ?
    - check for some rce
    - check for list of usernames that we can bruteforce
- smb ?
    - check for anonymous usrnames access 'guest, anonymous'
    - check for some leaked version in nmap maybe there could be some vuln
    - `--rid-brute` check also with a valid user like guest or ''
- enum4linux
- rdp ?
    - check for null username access
    - if there is a list of username try to bruteforce password using hydra
- Collect usernames from website or any pace we can collect like ldap --rid-brute in smb
    - brute force usernames on laps using kerbrute
    - add @domain to usernames
    - using `username-anarchy` if we have names we can create a list of possible usernames like `firt.last` or `first.last@domain`
- valid usersname ?
    - try arproasting
- we have a valid credentials ?
    - try kerboraosting
    - try using `nxc` all protocols
- drop sharphound
    - check for kerberoastable users
    - all domain admins
    - arproasting
    - paths from owned users to other pc
- mimikatz
    - lsadump::secrets
    - sekurlsa::logonpasswords
    - sekurlsa::logonpasswords
    - sekurlsa::tickets
    - if we find any crednetials save them and pass them through the whole network
    - check for default passwords and spray it agains users including net user /domain
- drop ligolo for pivoting

Orchestrated comprehensive AD enumeration methodology across multiple attack vectors

This is a solid AD/pentest methodology skeleton — you've got