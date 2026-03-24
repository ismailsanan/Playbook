
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

**HPING3**

```sh
#sends arbitrary TCP/IP packets 

#send a Sync packet on port 443 with traceroute mode 
hping3 -T -S -p 443  <HOST>

#send a sync ack  data size 766 we need 806 but header is 40 port 22  on the domain 
sudo hping3 -S -A  -d 766 -p 22 DOMAIN

```

 **Nmap**

```sh
#TCP
sudo nmap -sCV -v -p- -Pn <IP> -oN ./enum/nmap.logs

#UDP
sudo nmap -sU -F <IP> -oN ./enum/nmap.UDP.logs


#Scripts
nmap --script "ftp*" -p 21 $ip

#if smb was running on old windows 2008  or lower 
nmap -p139,445 --script smb-vuln* 192.168.227.40


#Get open ports
nmap -p- -Pn -n 10.10.10.10
```


**Unicornscan**

equivilant to nmap and uses all payloads


**SSH**

```

```


NOTE


```txt

if the result of the hping3 packet has id set adn incrementing it means its windows if its always 0 it means its linux 

```