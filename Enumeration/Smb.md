

> Basic
```bash 
smbclient -L //10.10.11.35/

smbclient  //10.10.11.35//<share_name>

smbclient //10.10.11.174/support-tools -U guest

smbclient //10.10.11.174/support-tools -U oscp/guest
```

>CVE

```sh
#check for os running if:
| smb-os-discovery: 
|   OS: Windows Server (R) 2008 Standard 6001 Service Pack 1  
#its vulnerable likely to have vulnerbale smb
nmap -script=smb-vuln\* -p445 192.168.103.40 

```

>Download all files
``` shell
smbclient  //IP/SHARE

recurse ON 

prompt OFF

mget *

#example
smbclient //192.168.195.175/Password\ Audit -U 'V.Ventz' -c 'prompt OFF;recurse ON;mget *'


smbget  --recursive  smb://192.168.109.175/"Password Audit" --user V.Ventz -W resourced.local --password HotelCalifornia194!

```


>Basic
```bash
#enum usernames in the samba and needs a valid user ffor the sid
nxc smb <IP> -u 'guest' -p '' --rid-brute

nxc smb <IP> -u 'guest' -p '' --shares
nxc smb <IP> -u user.txt -p password.txt #we can add --continue-on-success

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
