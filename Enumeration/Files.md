
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

#Automatically extract known file types
binwalk -e dog.jpg 
```


>check the dynamic libraries 
```
ldd psp64
```


>information gathering metadata
```sh
#Metagoofil is an open-source information-gathering tool designed to extract metadata from public documents available on websites
metagoofil -d [domain] -t [filetypes] -l [limit] -n [number] -o [output_directory] -f [output_file]

```