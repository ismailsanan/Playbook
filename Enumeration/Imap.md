
The **Internet Message Access Protocol (IMAP)** is designed for the purpose of enabling users to **access their email messages from any location**, primarily through an Internet connection.

By default, the IMAP protocol works on two ports:

- **Port 143** - this is the default IMAP non-encrypted port
- **Port 993** - this is the port you need to use if you want to connect using IMAP securely


```sh
nc 192.168.244.140 143  
  a1 login <user> <password?  
  a1 LIST "" "*"  
  ta1 SELECT INBOX
  
# if port 993 
openssl s_client -connect <IP>:993 -quiet
```

>Commands
```sh
Login
    A1 LOGIN username password
    A1 LOGIN "username" "password"

List Folders/Mailboxes
    A1 LIST "" *
    A1 LIST INBOX *
    A1 LIST "Archive" *
#LIST (\NoInferiors) "/" INBOX -> here’s a mailbox called `INBOX`, it cannot have subfolders, and `/` is the folder separator
Create new Folder/Mailbox
    A1 CREATE INBOX.Archive.2012
    A1 CREATE "To Read"

Delete Folder/Mailbox
    A1 DELETE INBOX.Archive.2012
    A1 DELETE "To Read"

Rename Folder/Mailbox
    A1 RENAME "INBOX.One" "INBOX.Two"

List Subscribed Mailboxes
    A1 LSUB "" *

Status of Mailbox (There are more flags than the ones listed)
    A1 STATUS INBOX (MESSAGES UNSEEN RECENT)

Select a mailbox
    A1 SELECT INBOX

List messages
    A1 FETCH 1:* (FLAGS)
    A1 UID FETCH 1:* (FLAGS)

Retrieve Message Content
    A1 FETCH 2 body[text]
    A1 FETCH 2 all
    A1 UID FETCH 102 (UID RFC822.SIZE BODY.PEEK[])

Close Mailbox
    A1 CLOSE

Logout
    A1 LOGOUT
```


## evolution GUI

use it much easier to navigate with

https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-imap.html?highlight=143#143993---pentesting-imap