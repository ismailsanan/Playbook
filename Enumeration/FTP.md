
```sh
ftp <target-ip> <target-port>

#Use try guest ,  anonymous  , admin , administrator
#to download files 
wget ftp://user:password@ftp.mydomain.com/path/file.ext  
```

#### Bruteforcing with Hydra
```
hydra [-L users.txt or -l user_name] [-P pass.txt or -p password] -f [-S port] ftp://X.X.X.X
```


#### Bruteforcing with Nmap

It is also possible to perform brute force on FTP with `Nmap` scripts:

```
nmap -p 21 --script ftp-brute X.X.X.X
```