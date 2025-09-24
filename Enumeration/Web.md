
### FFUF

```bash 
ffuf  -x http://127.0.0.1:8080 -u http://alert.htb/FUZZ -w /usr/share/worlists/seclists/Discovery/Web-Content/raft-large-files.txt
```

> Vhost  fuzz
```bash 
ffuf -w /wordlists/subdomains-top1million-110000.txt -u http://titanic.htb/ -H  "Host:FUZZ.titanic.htb" -fc 301
```

> Param fuzz
```sh
ffuf  -x http://127.0.0.1:8080 -u http://{URL}/search?FUZZ=FUZZ -w /usr/share/worlists/seclists/Discovery/Web-Content/raft-large-files.txt
```


### Gobuster

```sh
gobuster dir -u http://{URL} -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories.txt

# VHost
gobuster vhost --append-domain --wordlist /usr/share --u http://{URL} 

#--append-domain basically append domain.locahost
```


### Wordpress
Wordpress Vuln Scanner

```
wpscan -u https://{URL} --disable-tls-checks
```

### Nikto
Web App Vuln scanner

```
nikto -h {IP}:{PORT}

extensions
-ssl -Format txt -output file
```


### dirsearch
searches for sub directories simple and efficient 

>Command 
```shell
dirsearch -u dev.linkvortex.htb -t 50 -i 200
```


### Git

```sh
wget -r -np -R "index.html*" -e robots=off http://dev.linkvortex.htb/.git

#Restore .git
git restore .
```
