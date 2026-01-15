
```
snmp-check <IP>
```


```bash
for community in public private manager

snmpwalk -c $community -v 1 $ip
# `-v` snmp version 1-3

snmpwalk -v 2  -c public  <IP> NET-SNMP-EXTEND-MIB::nsExtendObjects
```
