
```sh
#Port Scanning in windows
Test-Netconnection -Port 445 $IP
```

```sh
#Local tcp sockets
netstat -lpnt

#local udp socket 
netstat -lpnu
```


# Nmap

```sh
sudo nmap -T4 -p- -sCV --open -v $ip  -oN  results 

#UDP
sudo nmap -sU $ip

#Scripts
nmap --script "ftp*" -p 21 $ip

nmap -v -p 139,445 --scipt smb-os-discovery $ip

#Get open ports
nmap -p - -Pn -n 10.10.10.10
```


