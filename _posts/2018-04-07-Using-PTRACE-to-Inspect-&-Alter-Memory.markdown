---
layout:     post
title:      "Using PTRACE to Inspect & Alter Memory"
#subtitle:   "Linux"
date:       2018-04-07
author:     "Tech"
header-img: "img/post-ptrace.jpg"
tags:
    - Linux
---


# Using PTRACE to Inspect & Alter Memory.

This is a fun project I worked on to use the **ptrace** functionality to inspect and alter memory (PEEKDATA/POKEDATA).

Some general statements:
  - I'll be using an Ubunut 16.04 system.
  - I'll be making a '**traceme**' and '**tracer**' program.
  - Magic

First I attached the running process. By using ptrace_attach, when this is called with the pid that is to be traced, it is roughly equivalent to the process calling ptrace(PTRACE_TRACEME, ..) and becoming a child of the tracing process. 

The traced process is sent a SIGSTOP, so we can examine and modify the process as usual. After we are done with modifications or tracing, we can let the traced process continue on its own by calling ptrace(PTRACE_DETACH, ..). 

From here I utilize the ptrace_getregs to get the registers for me. SingleStep tells the kernel to stop the child at each instruction and let the parent take control. To get the arguments of the system call you have to read the registers one by one. For that you need to know which registers will be storing which parameters of the system call.

 - regs.rdi - Stores the first argument
 - regs.rsi - Stores the second argument
 - regs.rdx - Stores the third argument
 - regs.r10 - Stores the fourth argument
 - regs.r8 - Stores the fifth argument
 - regs.r9 - Stores the sixth argument
 
 ---

# ./Traceme
The code below is for the traceme program, I will be tracing this program using the ptrace() API to single step and trace ALL system calls.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int function( char * a )
{
    char buffer[1024];
    int ic;
    int i;
    int m;
    
    printf("Entered %s\n", __FUNCTION__ );
    m = open( a, O_RDONLY );
    i = 0;

    while( read( m, buffer, sizeof(buffer) ) > 0 ) 
    {
          printf("%s", buffer );
    }
    printf("LEAVING %s\n", __FUNCTION__ );
    return 0; 
}
int main ( int argc, char ** argv ) 
{
    printf("My pid..%d\n", getpid() );
    printf("Press any key to continue...\n");
    getchar();
    function( argv[1] );
    return 0;
}      
```

You'll need to compile the program: **gcc -o Traceme Traceme.c**

```javascript
root@techryptic:~/Desktop/tmp# ./Traceme
My pid..2555
Press any key to continue...
```
When you run the program , it will display the pid. Before we press a key to continue, we'll need to run that pid in ./Tracer below.

---

# ./Tracer

```c
#include <stdlib.h>
#include <stdio.h>
#include <stdint.h>
#include <unistd.h>
#include <signal.h>
#include <wait.h>
#include <sys/ptrace.h>
#include <sys/user.h>

int main (int argc, char* argv[])
{
	if (argc < 2) {
		printf("Usage: %s pid\n", argv[0]);
		exit(1);
	}
    struct user_regs_struct regs;

    int waitStat = 0;
	pid_t pid = atoi(argv[1]);

	if ((ptrace(PTRACE_ATTACH, pid, NULL, NULL)) < 0) {
		perror("ptrace(ATTACH)");
		exit(1);
	}
	int waitRes = waitpid(pid, &waitStat, WUNTRACED);
	if (waitRes != pid || !WIFSTOPPED(waitStat))
	{
		perror("....:");
		printf("Something went wrong...\n");
		exit(1);
	}
	while (1)
	{
		int signum = 0;
		printf ("Getting Registers \n");

		if ((ptrace (PTRACE_GETREGS, pid, NULL, &regs)) < 0)
		{
			perror ("ptrace(GETREGS):");
			exit (1);
		}
		if ((ptrace(PTRACE_SINGLESTEP, pid, NULL, NULL)) < 0)
		{
			perror ("ptrace(CONT):");
			exit(1);
		}
		waitRes = wait(&waitStat);
		signum = WSTOPSIG(waitStat);
		if (signum == SIGTRAP)
		{
			signum = 0;

			printf("RAX:\t%llx\n", regs.rax);
			printf("RCX:\t%llx\n", regs.rcx);
			printf("RDX:\t%llx\n", regs.rdx);
			printf("RSP:\t%llx\n", regs.rsp);
			printf("RBP:\t%llx\n", regs.rbp);
			printf("R8:\t%llx\n", regs.r8);
			printf("R9:\t%llx\n", regs.r9);
			printf("RIP:\t%llx\n", regs.rip);
		}
		else
		{
			printf("Unexpected signal %d\n", signum);
			ptrace(PTRACE_CONT, pid, 0, signum);
			break;
		}
	}
    return 0;
}
```
You'll need to compile the program: **gcc -o Tracer Tracer.c**

When you run the Tracer program with the PID from trace me, it will be waiting on Getting Registers as seen below.

```javascript
root@techryptic:~/Desktop/tmp# ./Tracer 2555
Getting Registers
```
From here you'll need to go back to **./Traceme** terminal and '**Press any key to continue...**' (Followed by the Enter Key)

Look back at the **./Tracer** terminal window, you'll now see a flow of data coming in as seen above.

```javascript
root@techryptic:~/Desktop/tmp# ./Tracer 2555
Getting Registers
RAX:	fffffffffffffe00
RCX:	7f6839aa88d1
RDX:	400
RSP:	7ffc89d01588
RBP:	d68
R8:	7f6839d748a0
R9:	7f6839f654c0
RIP:	7f6839aa88d1
Getting Registers
RAX:	3
RCX:	7f6839aa88d1
RDX:	400
RSP:	7ffc89d01588
RBP:	d68
R8:	7f6839d748a0
R9:	7f6839f654c0
RIP:	7f6839aa88d1
```
