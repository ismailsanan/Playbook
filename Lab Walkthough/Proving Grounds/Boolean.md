192.168.137.231

```sh
Not shown: 65531 filtered ports
PORT      STATE  SERVICE VERSION
22/tcp    open   ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 37:80:01:4a:43:86:30:c9:79:e7:fb:7f:3b:a4:1e:dd (RSA)
|   256 b6:18:a1:e1:98:fb:6c:c6:87:55:45:10:c6:d4:45:b9 (ECDSA)
|_  256 ab:8f:2d:e8:a2:04:e7:b7:65:d3:fe:5e:93:1e:03:67 (ED25519)
80/tcp    open   http
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns: 
|     HTTP/1.1 400 Bad Request
|   FourOhFourRequest, GetRequest, HTTPOptions: 
|     HTTP/1.0 403 Forbidden
|     Content-Type: text/html; charset=UTF-8
|_    Content-Length: 0
|_http-favicon: Unknown favicon MD5: D41D8CD98F00B204E9800998ECF8427E
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-title: Boolean
|_Requested resource was http://192.168.137.231/login
3000/tcp  closed ppp
33017/tcp open   http    Apache httpd 2.4.38 ((Debian))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Development
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.80%I=7%D=1/9%Time=69611E1F%P=x86_64-pc-linux-gnu%r(GetRe
SF:quest,55,"HTTP/1\.0\x20403\x20Forbidden\r\nContent-Type:\x20text/html;\
SF:x20charset=UTF-8\r\nContent-Length:\x200\r\n\r\n")%r(HTTPOptions,55,"HT
SF:TP/1\.0\x20403\x20Forbidden\r\nContent-Type:\x20text/html;\x20charset=U
SF:TF-8\r\nContent-Length:\x200\r\n\r\n")%r(RTSPRequest,1C,"HTTP/1\.1\x204
SF:00\x20Bad\x20Request\r\n\r\n")%r(X11Probe,1C,"HTTP/1\.1\x20400\x20Bad\x
SF:20Request\r\n\r\n")%r(FourOhFourRequest,55,"HTTP/1\.0\x20403\x20Forbidd
SF:en\r\nContent-Type:\x20text/html;\x20charset=UTF-8\r\nContent-Length:\x
SF:200\r\n\r\n")%r(GenericLines,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n
SF:\r\n")%r(RPCCheck,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(DN
SF:SVersionBindReqTCP,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(D
SF:NSStatusRequestTCP,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(H
SF:elp,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(SSLSessionReq,1C
SF:,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(TerminalServerCookie,1
SF:C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(TLSSessionReq,1C,"HTT
SF:P/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(Kerberos,1C,"HTTP/1\.1\x204
SF:00\x20Bad\x20Request\r\n\r\n")%r(SMBProgNeg,1C,"HTTP/1\.1\x20400\x20Bad
SF:\x20Request\r\n\r\n")%r(LPDString,1C,"HTTP/1\.1\x20400\x20Bad\x20Reques
SF:t\r\n\r\n")%r(LDAPSearchReq,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\
SF:r\n")%r(LDAPBindReq,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(
SF:SIPOptions,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(LANDesk-R
SF:C,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(TerminalServer,1C,
SF:"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(NCP,1C,"HTTP/1\.1\x2040
SF:0\x20Bad\x20Request\r\n\r\n")%r(NotesRPC,1C,"HTTP/1\.1\x20400\x20Bad\x2
SF:0Request\r\n\r\n")%r(JavaRMI,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n
SF:\r\n")%r(WMSRequest,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(
SF:oracle-tns,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(ms-sql-s,
SF:1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(afp,1C,"HTTP/1\.1\x2
SF:0400\x20Bad\x20Request\r\n\r\n")%r(giop,1C,"HTTP/1\.1\x20400\x20Bad\x20
SF:Request\r\n\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel


```



interesting port 33017

![[Pasted image 20260109164943.png]]

we found out its a `dev` environment 

```
/cgi-bin (Status: 301)
/admin (Status: 301)
/info (Status: 301)
/server-status (Status: 403)

```

nothing we can i do so far 

going back to `8080`

```
/login (Status: 200)
/register (Status: 200)
/404 (Status: 200)
/500 (Status: 200)
/filemanager (Status: 302)
```


we registered a user `root:root` admin was taken that could be potential username enum 



![[Pasted image 20260109165445.png]]


lets try changing the email maybe we can trigger an ssrf or something 
![[Pasted image 20260109173233.png]]
i tried tempering with some values to change the username to admin, maybe change the id = 1 but we can only change the  param `confirmation=true` which bypasses the confirmation  stage 

![[Pasted image 20260109173541.png]]

uploaded a revshell 

![[Pasted image 20260109174748.png]]


when clicking on the file its just downloading 


i noticed in on url a `cwd ` would it be a current working dir ? 

IT IS !
![[Pasted image 20260109174946.png]]


i went back to the original url to potentially download `etc/passwd` and it worked
![[Pasted image 20260109175039.png]]
now since we dont have anythingf to crack lets check `shadows` `id_rsa`

we enumerated `/home/` to see the user remi
we used that auth_key
![[Pasted image 20260109175215.png]]

to access the machine 

