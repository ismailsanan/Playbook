
```sh
#Find specific file
find / -iname local.txt -type f 2>/dev/null

# Converts output find into arguments for another command
find . -name "*.txt" | xargs rm  

#decompress file
tar -xvzf community_images.tar.gz

#Python  virtual envirn
source ./env/bin/activate

#check alias
alias ls

#execute shell with privs if we have SUID 
/bin/dash -p

#Check OS info
uname -a 
lsb_release -a 
cat /etc/os-release 
hostnamectl

#Mounted devices
cat /proc/mount
#usually sda but if its not compare the result befoer and after pluging in

lsblk -o NAME,SIZE,TRAN
dmesg

#list partitions on our drive 
fdisk -l 

#Show information about the file system
df

#folder information 
du -h

#Processes that are used in the current termina
ps
ps -aux
ps -axjf -> hierarchy of the process


#Host Discovery
arp -a (neighbor IPs)
route print (reachable subnets)
netstat -an (active connections)
nbtstat -A <ip> (NetBIOS names).

#enum network ports
netstat -ltpn
ss -ltpn

#start a reverse proxy on tun0 
sudo responder -I tun0 -wv

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

#Sublime
#replace multiple new line with one
ctrl+H

#switch to regulare expression
\n{2,}

#Replace All 
\n

# curl a page with credentials
curl --user offsec:elite 192.168.68.46:242/pwn.php


# What will the shell actually run when I type this command
type -a 

---------------- BASH ---------------------

#delemeter \  take the first field and print the first arguement output them as user.txt 

cat user | cut -d "\\" -f 1 | awk '{print $1}' | tee user.txt

#print first argument 
cat ports | awk -F/ '{print $1}' > ports.nunber

#All arguments passed to this function
$@
----------- WINDOWS ------------------

#spawns a shell with specific user  
runas /netonly /users:USER1 cmd.exe

ps> Invoke-Runas


#Network
Test-NetConnection -ComputerName $ip -Port $port 

#recursive search in powershell
Get-ChildItem -Recurse -Filter '*.txt'

#recursive search in cmd
dir /s /b *.txt


#all prebuild exe are in windows/system32
C:\Windows\System32\whoami.exe
C:\Windows\System32\systeminfo.exe #to check if the system is x64 arch
#path to powershell 
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe

#execute scripts

powershell -ep bypass
. .\Powerview.ps1

#check Service 
sc qc "<Service>"
sc qc | findstr /i "apache"

Get-Service -Name "KiteService"
Get-Service | Where-Object {$_.DisplayName -like "*Apache*"} | Format-List Name,DisplayName

Get-CimInstance -ClassName Win32_Service -Filter "Name='ApacheHTTPServer'" | Select-Object Name, StartName

Get-WmiObject Win32_Service -Filter "Name='KiteService'"
#start and stop services
net stop <service_name>
net start <servcice_name>

sc stop <servcice_name>
sc start <servcice_name>

Start-Service -Name "ServiceName"
Stop-Service -Name "KiteService"

wmic service get name,state,startmode
wmic service where "name='KiteService'" call stopservice



#start Service 
net start | findstr "KiteService"

#check schedules tasks
tasklist /svc | findstr "svchost"
taskkill /f /pid [PID]


#GPP refers to a feature in Windows Group Policy that allowed administrators to deploy settings (like mapped drives, scheduled tasks, and local user passwords) across a domain

gpp-decrypt STRING


```


**SSH**

```sh
#ssh with priv key
chmod 700 private_key.txt
ssh -i /path/to/your/private_key user@server_ip
```


**XFreeRDP**

```sh
xfreerdp /v:<IP-ADD> /u:Administrator /p:'P@$$W0rd'

#pass the hash
xfreerdp3 /u:"heist.offsec\svc_apache$" /pth:9943473CE1243E91129513BB932E9C90 /v:192.168.235.165 +clipboard /cert:ignore 
```


**wordlist**

```sh
#Create a wordlist
#min 6 max 6 given chars  output in file 6chars
crunch 6 6 123456789abcdefg -o 6chars.txt


# create combination of names for potential usernames 
username-anarchy -i users -f first,flast,first.last,firstl > users.txt
```