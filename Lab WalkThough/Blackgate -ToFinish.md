
192.168.137.176


```sh
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.3p1 Ubuntu 1ubuntu0.1 (Ubuntu Linux; protocol 2.0)
6379/tcp open  redis   Redis key-value store 4.0.14
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```


- search online for known vulns in redis 
- check for pentesting redis


- https://github.com/n0b0dyCN/redis-rogue-server
- another path  https://d4luc1.medium.com/unauthenticated-redis-server-leads-to-rce-6c175c75b293


using the first exploit which was suggested by hacktricks  i got a shell

![[Pasted image 20260109102042.png]]


![[Pasted image 20260109102157.png]]


--------- priv esc -------------------


- dropped linpeas

```sh

Sudo version 1.9.1

5.8.0-63-generic

check cronjobs 
root         860  0.0  0.2   6800  2776 ?        Ss   09:17   0:00 /usr/sbin/cron -f


User prudence may run the following commands on blackgate:
    (root) NOPASSWD: /usr/local/bin/redis-status

```


- i couldn't do it i checked the solution from the walkthough it happens to be a reverse engineering for `redis-status`
- knowing that i started reversing redis using `ghidra`

simply going to its main function we notice that after calling the authorization key its doing a compare to `ClimbingParrotKickingDonkey321` so we can confirm that it was the auth key
![[Pasted image 20260109113839.png]]


![[Pasted image 20260109113958.png]]


- it worked but we can't do anything 
- looking back at the bin we notice its using strcmp with unconstrained input function `gets` we can buffer overflow it maybe to overwrite the system call to a bash ? 

 - ToDo Later 