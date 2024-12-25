ANTI-DEBUG PE REVERSE-ENGINEERING
---

* First off let me just say what a nigger humza is for making this PE wicked in a debugger, and it was my first attempt at anti-debug analysis. The learning process was amazing though, cant knock that.

---

# Upon opening the Fucked PE in BINJA and inspecting the main function in the disassembly:

![Screenshot 2024-12-24 152241](https://github.com/user-attachments/assets/246b148e-a888-4914-bfab-c45527c0cf51)
---
 - Bam there it is staring back at us the reason our debugger goes to shit when executing this executable. So basically, whenever there is a debugger attached to the process when executing it, the program will know it is being debugged by a debugger and use an API call like IsDebuggerPresent(); in kernel32.dll.

 - We see NtSetInformationThread and NtQueryInformationThread when scrolling through this main function code block and we know whenever these api's are called there most likely causing the debugger to stop debugging the process and cause it to crash the debugger. We know NtSetInformationThread has a parameter called THREADINFOCLASS, which contains "ThreadHideFromDebugger = 0x11".
 
 - Why the fuck would windows let us use these api's??? Well, see here is why it exists: Whenever you attach a debugger to a remote process a new thread is created and if it was a normal thread the debugger would endlessly loop as it attempts to stop its own execution. Under the hood when a debugging thread is created Windows calls NtSetInformationThread with the flag set to (1) allowing the process to be debugged and continue as aspected. 
---
# Example of some c++ code for this method being used:

```
/*Timb3r's Code From GH*/

#include <stdio.h>
#include <windows.h>

enum THREADINFOCLASS { ThreadHideFromDebugger = 0x11 };

typedef NTSTATUS (WINAPI *NtQueryInformationThread_t)(HANDLE, THREADINFOCLASS, PVOID, ULONG, PULONG);
typedef NTSTATUS (WINAPI *NtSetInformationThread_t)(HANDLE, THREADINFOCLASS, PVOID, ULONG);

NtQueryInformationThread_t fnNtQueryInformationThread = NULL;
NtSetInformationThread_t fnNtSetInformationThread = NULL;

DWORD WINAPI ThreadMain(LPVOID p) {
        while(1) {

                if(IsDebuggerPresent())
                        asm("int3");
                Sleep(500);
        }
        return 0;
}


int main(void)
{
        DWORD dwThreadId = 0;
        HANDLE hThread = CreateThread(NULL, 0, ThreadMain, NULL, 0, &dwThreadId);

        HMODULE hDLL = LoadLibrary("ntdll.dll");
        if(!hDLL) return -1;

        fnNtQueryInformationThread = (NtQueryInformationThread_t)GetProcAddress(hDLL, "NtQueryInformationThread");
        fnNtSetInformationThread = (NtSetInformationThread_t)GetProcAddress(hDLL, "NtSetInformationThread");

        if(!fnNtQueryInformationThread || !fnNtSetInformationThread)
                return -1;

        ULONG lHideThread = 1, lRet = 0;

        fnNtSetInformationThread(hThread, ThreadHideFromDebugger, &lHideThread, sizeof(lHideThread));
        fnNtQueryInformationThread(hThread, ThreadHideFromDebugger, &lHideThread, sizeof(lHideThread), &lRet);

        printf("Thread is hidden: %s\n", val ? "Yes" : "No");
 
        WaitForSingleObject(hThread, INFINITE);
        return 0;
}
```
 - 
