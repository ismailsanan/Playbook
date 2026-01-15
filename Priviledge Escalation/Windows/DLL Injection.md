
basically when executing a binary a linker loads dynamic libraries into the process memory  in linux they are called shared objects `.so`  in windows they are `.dll`

since this its dynamic this introduces the possibility for an attacker to inject malicious code in the form of  DLL 

- find a dll used by the victim binary that we can overwrite with mal one
- introduce a new mal dll and trick the default search order used by windows in order to load the mal one instead


First we need to create and compile the library code 

Simple Example
```
#include<windows.h>
__declspec(dllexport) int add_numbers(int a , int b){
	return a+b;
}
```


we compile it 
```sh
#shared for compiling dll
x86_64-w64-wingw32-gcc -shared -o lib.dll dib.c

#cross program compile
x86_64-w64-wingw32-gcc -mwinodws -minuicode -O2 -s -O simple.exe simple.c
```
and let say a program actually uses this lib.dll the scenario would be that `program.exe` without the lib.dll it wouldn't work since the functions  its referencing is not found and with it working the function its referencing is lets say `add_numbners`

what we can do is this we overwrite this lib.dll and call the main lib.dll so that when the program starts it gets executed immediately 
```
#include<windows.h>
	BOOL WINAPI Dllmain(HIINSTANCE hinstdll, DWORD fdwReason, LPVOID lpReserved){
	system("echo hacked")
	return TRUE
	}
```



we can hijack the dll using the search order 
- absolute path
```
loadlibrary(text("C:\\users\\user\\downloads\\lib.dll"));
```
then try to overwrite it


- relative path 

```
loadlibrary(text("lib.dll"));
```

find it and try to hackjack it before the original one 
- Working directory.
- The system directory
- The 16-bit system directory.
- The Windows directory.
- The current directory.
- The directories that are listed in the `PATH` environment variable.

so if the dll is found within the windows directory `C:\WINDOWS`
then we can inject the dll in the system directory `C:\WINDOWS\SYSTEM32` which has more priority 


DLL Hijacking involves a few more steps. First, you need to pinpoint a DLL the target is attempting to locate. Specific tools can simplify this task:

1. Process Explorer: Part of Microsoft's Sysinternals suite, this tool offers detailed information on running processes, including their loaded DLLs. By selecting a process and inspecting its properties, you can view its DLLs.
    
2. PE Explorer: This Portable Executable (PE) Explorer can open and examine a PE file (such as a .exe or .dll). Among other features, it reveals the DLLs from which the file imports functionality.


procmon only captures information while it is actively running


trigger  DLL if system32 is writable 

https://github.com/sailay1996/WerTrigger