ANTI-DEBUG PE REVERSE-ENGINEERING
---

* First off let me just say what a nigger humza is for making this PE wicked in a debugger, and it was my first attempt at anti-debug analysis. The learning process was amazing though, cant knock that.

---

# Upon opening the Fucked PE in BINJA and inspecting the main function in the disassembly:

![Screenshot 2024-12-24 152241](https://github.com/user-attachments/assets/246b148e-a888-4914-bfab-c45527c0cf51)
---
 - Bam there it is staring back at us the reason our debugger goes to shit when executing this executable. So basically, whenever there is a debugger attached to the process when executing it the program will know it being debugged by a debugger and use another an API call like IsDebuggerPresent(); in kernel32.dll.

 - We see NtSetInformationThread and NtQueryInformationThread when scrolling through this code block and we know whenever these api's are called there most likely using 
 - 
