
AD - Medium 


```sh
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-04 18:17:17Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-08-04T18:18:22+00:00; +3h59m59s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Issuer: commonName=htb-AUTHORITY-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-08-09T23:03:21
| Not valid after:  2024-08-09T23:13:21
| MD5:     d494 7710 6f6b 8100 e4e1 9cf2 aa40 dae1
| SHA-1:   dded b994 b80c 83a9 db0b e7d3 5853 ff8e 54c6 2d0b
|_SHA-256: e1d2 e894 2960 a961 bbf7 b4e4 c110 c6d7 e5a1 7a29 8987 85dc 3553 fb90 458a 5cb7
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Issuer: commonName=htb-AUTHORITY-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-08-09T23:03:21
| Not valid after:  2024-08-09T23:13:21
| MD5:     d494 7710 6f6b 8100 e4e1 9cf2 aa40 dae1
| SHA-1:   dded b994 b80c 83a9 db0b e7d3 5853 ff8e 54c6 2d0b
|_SHA-256: e1d2 e894 2960 a961 bbf7 b4e4 c110 c6d7 e5a1 7a29 8987 85dc 3553 fb90 458a 5cb7
|_ssl-date: 2026-08-04T18:18:22+00:00; +3h59m59s from scanner time.
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-08-04T18:18:22+00:00; +3h59m59s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Issuer: commonName=htb-AUTHORITY-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-08-09T23:03:21
| Not valid after:  2024-08-09T23:13:21
| MD5:     d494 7710 6f6b 8100 e4e1 9cf2 aa40 dae1
| SHA-1:   dded b994 b80c 83a9 db0b e7d3 5853 ff8e 54c6 2d0b
|_SHA-256: e1d2 e894 2960 a961 bbf7 b4e4 c110 c6d7 e5a1 7a29 8987 85dc 3553 fb90 458a 5cb7
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-08-04T18:18:22+00:00; +3h59m59s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Issuer: commonName=htb-AUTHORITY-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-08-09T23:03:21
| Not valid after:  2024-08-09T23:13:21
| MD5:     d494 7710 6f6b 8100 e4e1 9cf2 aa40 dae1
| SHA-1:   dded b994 b80c 83a9 db0b e7d3 5853 ff8e 54c6 2d0b
|_SHA-256: e1d2 e894 2960 a961 bbf7 b4e4 c110 c6d7 e5a1 7a29 8987 85dc 3553 fb90 458a 5cb7
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8443/tcp  open  ssl/http      Apache Tomcat (language: en)
|_http-title: Site doesn't have a title (text/html;charset=ISO-8859-1).
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  h2
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| ssl-cert: Subject: commonName=172.16.2.118
| Issuer: commonName=172.16.2.118
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-08-02T17:27:09
| Not valid after:  2028-08-04T05:05:33
| MD5:     4a4b 149b cb3f 9412 1146 d072 ff50 7bc6
| SHA-1:   4e87 7c33 319a c1d5 2f0f b0fd 24a7 1af3 ec20 8c32
|_SHA-256: 0c9d e2ee 7454 3561 9460 2183 abe0 05a2 a02b d15f 5ddc 7937 844f 42a3 cc1a c3c8
|_http-favicon: Unknown favicon MD5: F588322AAF157D82BB030AF1EFFD8CF9
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  msrpc         Microsoft Windows RPC
49690/tcp open  msrpc         Microsoft Windows RPC
49691/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49693/tcp open  msrpc         Microsoft Windows RPC
49694/tcp open  msrpc         Microsoft Windows RPC
49700/tcp open  msrpc         Microsoft Windows RPC
49712/tcp open  msrpc         Microsoft Windows RPC
49731/tcp open  msrpc         Microsoft Windows RPC
49778/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: AUTHORITY; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
|_clock-skew: mean: 3h59m59s, deviation: 0s, median: 3h59m58s
| smb2-time: 
|   date: 2026-08-04T18:18:17
|_  start_date: N/A


```

Big Ass nmap 

**CheckList**
ldap -> KO



![[Pasted image 20260804172003.png]]

therea re basically 2 shares non defauly dep shres and development

downloading both and lets see if there is anything intersting

potential user
![[Pasted image 20260804174301.png]]


![[Pasted image 20260804174331.png]]



![[Pasted image 20260804174416.png]]

![[Pasted image 20260805095317.png]]

ill make a list of all potential password and potential users and lets brute force them  concerning the vault i need to check if its crackable 

![[Pasted image 20260805102719.png]]

hashit using ansible2john and cracked them i got all the same pass 

!@#$%^&*  


![[Pasted image 20260805103603.png]]

using that pass i decrrypted them and added them all into a wordlists of users and pass so lets try to brute force them 


![[Pasted image 20260805103819.png]]


all false +ve 

check some other places

![[Pasted image 20260805105102.png]]


in the config manager i see that there is svc_pwm loggied in so thats a hint of valid user 

inside tehere is config password i tried to brute force it 

![[Pasted image 20260805105248.png]]

![[Pasted image 20260805105353.png]]

![[Pasted image 20260805105220.png]]

no we have different page 

![[Pasted image 20260805105435.png]]



i downloaded the localDB 

its .bak
```
cp PWM-LocalDB.bak PWM-LocalDB.gz
gunzip PWM-LocalDB.gz
```

![[Pasted image 20260805110307.png]]


in the config file i found these:


`$2a$10$gC/eoR5DVUShlZV4huYlg.L2NtHHmwHIxF3Nfid7FfQLoh17Nbnua`

![[Pasted image 20260805105726.png]]

`
ENC-PW:ySDQYMGCvRQH5mxR3sjBkd1/yazM/KeMBwEe8dFXQIINpLsGMmzf9tNsynD1MbK1TYfsZfkLaNHbjGfbQldz5EW7BqPxGqzMz+bEfyPIvA8=`

![[Pasted image 20260805105751.png]]



`ENC-PW:Md6KROzPIeHwY9PIe8LDAjD5DvwQ3a0VQX5ntRY+vGcTGXZz0dH7LphAKUyPi9SdKue/Um6dkZm1RrcECBHk358zc045rDyFL2fDku2kusl79NE+Tww8gC8QQ0CX+VS2yyD46+ZS6Jriyu1Y7BOXnJifXXXsHzTmBTkodvnY33V6Puc0Zze0PGYHN+CGFtx/g5WaBTQbQwZwNLA+8Qe11GqCz+rBjGzQp0w6yLHJn+ZYBlLWgvZwN2KUHOiUIq5eKKDgjv+mga4zcB1STcpMJRaIiSnLdY3VCfsEj6p4BGz9jj+N7gQHBFAvI05JexXq8HyL7ZUEzLXU5FMQXvhhWSbhxoz7LH/iamvoOg13WnI3MRUzrXv91Uh7gdNZuXa1NmSBOe/g1GgmFV+0sxLIJ/99VT+GHIwrfjPNNV6jtKHhURPwp0a38c6aBGjpvB3AgAoZ0/KVLvQK1pAevO4NK2XFF2nPD8gQCQJMCsb62I+XMitkO2zKytrYEwZhl9VUGF0bAXQhC5I9xX1tEQAGBcENt1NGfM8iE+PlrZWwlr1yDjw+GZEm2KHyjnUFpBubqD7l7mvEJbEV26SQkR0v4R5LSEPbElOKGbGXMKkDEi53SQ5P0ZZQbega9XtBOHs+/s1EZ4p/qGVCvpD9dgc0SyS0auXU0PUddjxyXthHdqRbEWHhAduXYQgXF0eM2yWlbd7fTgSUMERlpjdFX/QZG3D6Ghp+iOCwfelEfKMQDO1myQcpq5YTE94YDz+aSWvi7ZGRIq+hRkwuR8E0EbEUE7CApDwF3LjGi+UEd9Y3Q9SPSMVxg4Ra2FB4sYCT19N7KV3TpGvJYD4SE8Mrn0cH9ihvlvDJFOxoLC9xM8FA9EAvSZN1w6lV4pUsVpUSM0LRKLqCmBCRJvaRNbhRymM96NFSSi4PwCCJQ7WVJjiS+oLQ+7qwHhqLQFy0+gtkGSQnBoq1FMYSCyGz/fUG84Xe0CSTPt4SwTq+L2M2jqsiB+HXq1z2LdkAFo6xm1Mqs6H/x5ZP1esjvRxDzHod31jRizu+rJw4LNRb172A36dQWmiq/OJQBJrnPu87s+KmoNyCJGrT2+1QttMgM62qy2/Eb6xByQ8RiLl6v87vf24TuWhxJhXfNWMRuHXJp2IWt5BWAYdiQNUjCuvRhfiyxsIqelpEpsOnm8WDVEsN0hqaEt9Db2e/d3Wpx1as4luVtA/MZtKy+gsH0qZUmouj7LCfN5TJpm00MiBTxYSkapKvAGchkE4UVc3AHGIxeyy+t2LwqT9fDSlS/VofOELNcQD3OfPi+asOrgaqcRbZVXdQumoJsubLMiPpHTZtOH2Nt13cEh9ZG/XebrAkchsMjsyLo5KX0nL6RKbMNUA3BmM2cd+bjj+Jar2aeAeqBdW+LU5ALshAsF986N1BGSsQ8aZkJwLi3PUYG8vGR88ZqEMMziQ=
`
nothing much to do with these creds


Moving forward to basicallly the application i see there is a configuration edit with a test ldap using pwn_ldap and password store ill use responder to maybe capture this password

![[Pasted image 20260805125436.png]]

![[Pasted image 20260805125403.png]]


 svc_ldap / lDaP_1n_th3_cle4r!

![[Pasted image 20260805125641.png]]

PWNED



![[Pasted image 20260805130547.png]]

lets check for some template abuse then 


![[Pasted image 20260805131134.png]]

![[Pasted image 20260805131143.png]]

we have an esc1 

since we have the enrolee right for domain computer lets do that create a computer domain , make that computer domain request the ticket impersonating admin using UPN  user prinsipal name 

![[Pasted image 20260805133101.png]]


![[Pasted image 20260805133533.png]]

no i got an error  the DC returned `KDC_ERR_PADATA_TYPE_NOSUPP` — the DC **isn't configured to support PKINIT**.

pkinit is basiclally **Public Key Cryptography for Initial Authentication**  an extension that  lets us  get kerberos tgt using a cerificate 


so a deep search i figured out we can use auth -ldap-shell to get a shell 


![[Pasted image 20260805135200.png]]

we can do this 

1. Write `fak$` into the DC's `msDS-AllowedToActOnBehalfOfOtherIdentity` (the `set_rbcd DC$ fak$` command).
2. `fak$` is now trusted to impersonate anyone to the DC.
3. Use Kerberos (S4U2Self + S4U2Proxy, done automatically by `getST.py`) to request a service ticket **as administrator**.
4. Present that ticket to the DC → you're administrator → shell or hash dump.

since the flow is that we are in DC so just write into msds fak to impersonate anyone ot the dc then bascially we can request TGS as admin  and login using winrm pth

![[Pasted image 20260805135822.png]]

get the hash 
![[Pasted image 20260805141118.png]]


i am having timing issue  WTF is wrong with HTB why is the machine 2 hours before the current time  FUCK IT im diching this 


OK it pissed me  off we can fix this by changing our date to the date of the DC  sudo ntpdate -u authority.authority.htb


![[Pasted image 20260806180353.png]]


![[Pasted image 20260806181254.png]]

DONE 