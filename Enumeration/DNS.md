> 
```shell

#enum for DNS
dnsrecon -d 10.10.10.100 -r 10.0.0.0/8

#zone transfer  
dig DOMAIN NS
dig @NS DOMAIN axfr 

dig axfr {DOMAIN}@{IP}

# check if there control over mail server 
dig DOMAIN MX


nslookup
#> server 10.0.1.1  # Switch to a custom DNS server
#> example.com      # Query the domain


#create a fake dns-server  for cases where we are in AD that does not have a visible DNS server  and we need the fdnq resolved  and we point out using some tool to use the interface localhost "127.0.0.1"
 python .\dnschef.py  --fakeip 10.1.1.1.1 --fakedomain ad.it -q
 
 
```


