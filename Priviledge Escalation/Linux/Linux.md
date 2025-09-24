- [ ] install and run Linpeas



> Read Arbitrary FIle in these dirs
```
/etc/ssh/sshd_config ->  ssh permissions
/etc/shadow
/proc/self/cmdline
```


>Linux privilege escalation auditing tool
```sh
./Linux_Exploit_Suggester.pl -k 3.0.0


./linpeas
```



> check sudo permessions 
```sh
sudo -l

#notes

(root) /usr/bin/pip install * ->  basically anything we will install we can install  it as root

ALL NOPASSWD  /usr/bin/try_harder ->  means we can execuyte it with root without a password 
```

![[Pasted image 20250608181258.png]]


### CronJob

In Linux Based systems `cronjob` is a command that is executed periodically with specific time rule.

```sh
#List out the cronjob  configuration for user we can execute
crontab -l

#edit configuration 
crontab -e 

#format 
MIN HOUR DOM MON DOW #minute , hour , day of the month , month , day of the week

example: 
* * * * echo "hello" > /tmp/test  #this will create a file and print hello every minute 
```

# Commands



```sh
#Find specific file
find / -iname local.txt -type f 2>/dev/null

#Takes input from standard input  Converts that input into arguments** for another command
find . -name "*.tmp" | xargs rm  
```



```sh
#decompress file
tar -xvzf community_images.tar.gz
```


```sh
#shows running processes 
ps aux
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

#suid/guid find command
find / -perm -u=s 2>/dev/null

find / -perm -g=s -type f 2>/dev/null


#change owner of the file
sudo chown root test.sh

sudo chown <user>:<group> file
```

