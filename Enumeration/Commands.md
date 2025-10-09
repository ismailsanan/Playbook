
```sh
#Find specific file
find / -iname local.txt -type f 2>/dev/null

#Takes input from standard input  Converts that input into arguments** for another command
find . -name "*.txt" | xargs rm  
```

```sh
#recursive search in cmd
dir /s /b *.txt

#recursive search in powershell
Get-ChildItem -Recurse -Filter '*.txt'
```


```sh
#decompress file
tar -xvzf community_images.tar.gz
```

- RunAs
```sh
#spawns a shell with specific user 
runas /netonly /users:USER1 cmd
```


>Python  virtual envirn
```
source ./env/bin/activate
```



>check alias
```
alias ls
```

> Check OS info
```sh
uname -a 
lsb_release -a 
cat /etc/os-release 
hostnamectl
```

>Mounted devices
```sh
#usually sda but if its not compare the result befoer and after pluging in

lsblk -o NAME,SIZE,TRAN
dmesg
```

**Disk management** 

> list partitions on our drive 
```
fdisk -l 
```


**Disk Usage** 

>information about the disk
```
df -h
```

**dir management** 

> each folder information meaing size used and so
```
du -h
```


**Processes** 
>Processes that are used in the current terminal 
```
ps
ps -axjf -> hierarchy of the process
```


**Network interfaces**

```
ip a
```


```
Test-NetConnection -ComputerName $ip -Port $port 
```



>enum network ports
```
netstat -ltpn
ss -ltpn
```



> start a reverse proxy on tun0 
```
sudo responder -I tun0 -wv
```


```sh
#create new user
sudo useradd -m <username>

#change password
sudo  passwd <username>

#check the groups
groups

#specific group
groups root

#create a group
groupadd <group>

#add user to group
usermod -a -G <group> <username>

#os version
lsb_release -a

#change owner of the file
sudo chown root test.sh

sudo chown <user>:<group> file
```


