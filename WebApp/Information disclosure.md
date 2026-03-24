


# Lab: Information disclosure in error messages


in the view product ttry to fuzz the sql with '  you will have an error and the answer is Apache Struts 2 2.3.31



# Lab: Information disclosure on debug page

in the coment section there was a link cgi-sometyhing/php.info

there you can find the secret key






# Lab: Source code disclosure via backup files


first try /robots.txt this leaked a /backup subdirectory  in this file is a sql code that leaked the password of the db 



# Lab: Authentication bypass via information disclosure

 change the GET /admin to TRACE you will find a X-Custom-IP-Authorization: 212.31.229.126

in proxy burp match insert a X-Custom-IP-Authorization: 127.0.0.1

done


# Lab: Information disclosure in version control history


wget -r http://website/.git/

check the logs you will find that there were 2 commits one of them is with a message of remove admin password from the confi

git reset "commit id"
check both

then you will find the password on the last one 
done

