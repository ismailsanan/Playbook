
### FFUF

```bash 
ffuf  -x http://127.0.0.1:8080 -u http://alert.htb/FUZZ -w /usr/share/worlists/seclists/Discovery/Web-Content/raft-large-files.txt

# we can brute force uisername list with this
ffuf -u http://192.168.181.10/login.php -X POST \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -H "Cookie: PHPSESSID=aom2u5vofc35bjgqfcl4486872" \
     -d "username=FUZZ&password=test" \
     -w /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt \
     -fs <size>  # Filter by response size (find what "wrong user" response size is first)
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
wpscan --url https://{URL} --disable-tls-checks
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

#a gui tool for working with git repositories.
GitKraken


#download all files 
wget -r -np -R "index.html*" -e robots=off http://dev.linkvortex.htb/.git

#Restore .git
git restore .
```



### Intersting FIles

```sh
#if we have an LFI access
/var/log/apache2/access.log -> HTTP access logs on port 80
nc -nv  $IP 80
#write any code and check if its logging in the bottom of the page
#if its getting truncate it means its saving it use php system cmd
/var/log/apache2/access.log&cmd=id

```


### nuclei
vulnerability scanner

```sh
nuclei -u http://example.com
```
