
```
rpcclient -U "" -N $ip
```

>ToEnum
```
enumdomusers #get users
enumprinters
querydispinfo #users and user info
```

>To add users to a userlist file:
```
rpcclient -U "" <ip> -N -c "enumdomusers" | grep -oP '\[.*?\]' | grep "0x" -v | tr -d '[]' > userlist.txt
```