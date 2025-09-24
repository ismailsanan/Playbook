> Reconnaissance for DNS
```shell
dnsrecon -d 10.10.10.100 -r 10.0.0.0/8
```

```
dig axfr test.com@{IP}
```


### NSLookup

```shell
nslookup
> server 10.0.1.1  # Switch to a custom DNS server
> example.com      # Query the domain
```
