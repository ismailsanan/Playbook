```bash

#allows you to read and write meta information in files

exiftool ocean.jpg


#file allows you to see the file type

file ocean.jpg



#dump a file in a hexadecimal (hex) format.

 xxd computer.jpg


# print out strings from binary 

strings computer.jpg



#search binary images for embedded files and executable code.

binwalk dog.jpg

#Automatically extract known file types
binwalk -e dog.jpg 



#check the dynamic libraries 

ldd psp64




#extract metadata from public documents available on websites

metagoofil -d [domain] -t [filetypes] -l [limit] -n [number] -o [output_directory] -f [output_file]

```

