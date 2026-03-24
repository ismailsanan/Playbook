

192.168.181.141 - ms1
`Eric.Wallows:EricLikesRunning800`
10.10.141.142 - ms2
10.10.141.140 - dc1


ms1 

user has impersonate privs

![[Pasted image 20260112181634.png]]


![[Pasted image 20260112181857.png]]

lets create a backdoor


![[Pasted image 20260112185517.png]]


![[Pasted image 20260112191532.png]]


after analyzing sharphound  i found 


![[Pasted image 20260112192634.png]]

intesting that 

![[Pasted image 20260112192837.png]]

if we can crack `sql_svc` we could potentially have a service account on ms02


lets try that 
- we couldn't crack it 

`web_svc:Diamond1`

`Mary.Williams:9a3121977ee93af56ebd0ef4f527a35e`

`7k8XHk3dMtmpnC`


` celia.almeda:e728ecbadfb02f51ce8eed753f3ff3fd`

mimikatz we didnt find anything intersting just its wierd that we have a user  called mary williams that couldnt be found on sharphound 

![[Pasted image 20260112193445.png]]


![[Pasted image 20260112204723.png]]


![[Pasted image 20260112205503.png]]





found an intersting dir

![[Pasted image 20260112213346.png]]


![[Pasted image 20260112213328.png]]



![[Pasted image 20260112215243.png]]


![[Pasted image 20260112220626.png]]

