
When a windows service path contains **space** with no proper **Quotation** used, windows OS will search the service binary using each sub directory for the binary not the end binary of the dir where it suppose to 


```
C:\Users\user\Downloads\Example Direcory\simplService.exe 
```

first executable that it would search to execute is Example.exe  why since we have a whitespace in the path  so we can basically check if we have write permissions  in the downloads and deploy a revshell  



#### Enum

Winpeas quiet serviceinfo

- check for file permissions 
- notes for no quotes and space detected 