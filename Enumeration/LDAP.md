
Lightweight Directory Access Protocol (LDAP) directories It allows you to search and retrieve information from an Active Directory (AD) or LDAP server such as users, groups, computers, and other directory objects.

```sh

#full search 
ldapsearch -x -H ldap://$TARGET -D "$USER@domain" -w "$PASS" -b "DC=support,DC=htb" "(objectClass=user)"

#`objectClass` is an attribute that defines what kind of object it is (user, group, computer, organizationalUnit, etc.).

#usernames
ldapsearch -v -x -b "DC=hutch,DC=offsec" -H "ldap://192.168.175.122" "(objectclass=*)" sAMAccountName description info

#read laps pass
nxc ldap 192.168.104.122 -d "hutch.offsec" -u "fmcsorley" -p "CrabSharkJellyfish192" --module laps                                              

#enum users
nxc ldap $TARGET -u "$USER" -p "$PASS" --users 

#enum groups
nxc ldap $TARGET -u "$USER" -p "$PASS" --groups

```


for gui tool `Apache Directory Studio `