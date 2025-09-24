
> allows you to read and write meta information in files
```sh
 exiftool ocean.jpg
```

>file allows you to see the file type
```sh
file ocean.jpg
```


>xxd allows you to dump a file in a hexadecimal (hex) format.
```sh
 xxd computer.jpg
```

>The strings command will print out strings from binary 
```sh
strings computer.jpg
```


>Binwalk is a tool that allows you to search binary images for embedded files and executable code.
```sh
binwalk dog.jpg

binwalk -e dog.jpg         ->  Automatically extract known file types

```


>check the dynamic libraries 
```
ldd psp64
```
