
When a windows service path contains **space** with no proper **Quotation** used, windows OS will search the service binary using each sub directory for the binary not the end binary of the dir where it suppose to 


```
C:\Users\user\Downloads\Example Direcory\simplService.exe 
```

**How the Unquoted Service Path Exploit Works**

When a service path is unquoted, Windows tries to execute it in a specific order. For example, if the service path is:


`C:\Program Files\My App\My Service\service.exe`
Windows will try to execute, **in order**:

1. `C:\Program.exe`
    
2. `C:\Program Files\My.exe`
    
3. `C:\Program Files\My App\My.exe`
    
4. `C:\Program Files\My App\My Service\service.exe`


#### Enum

Winpeas quiet serviceinfo

- check for file permissions 
- notes for no quotes and space detected 