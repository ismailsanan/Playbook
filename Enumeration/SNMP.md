
### Snmpwalk

```bash
for community in public private manager

snmpwalk -c $community -v1 $ip
# `-v` snmp version 1-3
```
