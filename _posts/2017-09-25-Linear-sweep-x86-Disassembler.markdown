---
layout:     post
title:      "Linear Sweep x86 Disassembler"
#subtitle:   "Reverse Engineering"
date:       2017-09-25
author:     "Tech"
header-img: "img/post-Linear-Sweep.png"
tags:
    - Reverse Engineering
    - x86
    - Shellcoding
---


My Disassembler attempts to parse arbitrary binary input into x86 assembly code.


   Uses Linear Sweep algorithm to disassemble an arbitrary binary file:

*    Only the following mnemonics are implemented: add nop and not call or cmp pop dec push idiv repne cmpsd imul retf inc retn jmp sal jz/jnz sar lea sbb mov shr movsd test mul xor neg
*    All register references will be 32-bit references.

## The Necessaries:

If you go to my github page and get one of the examples, we'll use example1.o binary file in the breakdown below.

> https://github.com/anthonys-io/Linear-x86-Disassembler


To start off, example.o nasm file would be:

```nasm
[BITS 32]
xor eax, eax
add eax, ecx
add eax, edx
push ebp
mov ebp, esp
push edx
push ecx
mov eax, 041424344h
mov edx, dword [ ebp + 08h]
mov ecx, dword [ ebp + 0ch]
add ecx, edx
mov eax, ecx
pop edx
pop ecx
pop ebp
retn 08h
```

With the example1 binary file, we'll open it in notepad++ (You can hexdump from here if needed)
![bin1](/img/in-post/post-js-version/bin1.png)
You'll notice that notepad++ tries and read the file, which is unreadable at this point, you can download the plugin hex-editor to open the hex view next.
![bin2](/img/in-post/post-js-version/bin2.png)

```nasm
31 c0 01 c8 01 d0 55 89 e5 52 51 b8 44 43 42 41
8b 55 08 8b 4d 0c 01 d1 89 c8 5a 59 5d c2 08  
```

With the binary in hand, it's time to use the linear sweep algorithm to break down the binary into assembly code.

* **Linear Sweep**: the most simple algorithm, since it takes a region and disassemble it sequentially, converting each byte it founds to an instruction, regardless of whether it really is an instruction or simply some data.

## The opcodes:
The Intel 64 and IA-32 Architectures Software Developer's Manual, Volume 3: Instruction Set Reference is part of a set that describes the architecture and programming environment of all Intel 64 and IA-32 architecture processors.

> https://www.intel.com/content/www/us/en/architecture-and-technology/64-ia-32-architectures-software-developer-manual-325462.html

The example files in my github doesn't contain any headers, which would make understanding this process a little more easier.

```nasm
eax = '000' or '50'
ecx = '001' or '51'
edx = '010' or '52'
ebx = '011' or '53'
esp = '100' or '54'
ebp = '101' or '55'
esi = '110' or '56'
edi = '111' or '57'
```
The above chart is used when we break down the operands to see which registers they correlate with. The 50-57 are used for the PUSH/POP operands. You'll notice that most of the 16-bit and 32-bit instruction have the same opcode. The processor will operate in either 16-bit mode or 32-bit mode, and depending on the mode it is operating in will dictate how the opcode is interpreted.

At the end of each instruction reference, there is a "Instruction Operand Encoding" table that breaks down each Op/En that is supported by the instruction. Those that have "ModRM:reg" and/or "ModRM:r/m" will require a MODRM byte.

For example, encodings such as A, B, C, M, RM, MR, MI, RMI, RVM, M1, MC all require the MODR/M byte. However encodings such as, O, OI, FD, TD, ZO, and I do not.

Taking the first few bytes in the binary file:

```nasm
31 c0 01 c8 01 d0 55 89 e5 52 51 b8 44 43 42 41
8b 55 08 8b 4d 0c 01 d1 89 c8 5a 59 5d c2 08  
```

We see that it starts with 0x31, in the intel manual you'll notice that XOR has an Opcode that starts with 31:

![bin3](/img/in-post/post-js-version/bin3.png)

The line were looking at:

> 31 /r XOR r/m32, r32 MR Valid Valid r/m32 XOR r32.

In quick terms, this tells us which registers go where if you follow the Op/En (in this case it's MR). Taking the next byte:

```nasm
c0 
```

We'll need to break down the hex 0xc0 to binary, which is: **11000000**

With this binary, it breaks down even further:
The last operand is the last 3 bytes of the binary, the first operand is the 3 bytes after the first two bytes.

Mod: **11**
Operand #1: **000** (eax)
Operand #2: **000** (eax)
We know that 000 is the EAX register.
Putting it all together, we'll get:

> **Xor eax, eax**

Continue on, the next byte is:

```nasm
01 
```

After you check on the intel manual, you see the ADD register uses 01:

![bin5](/img/in-post/post-js-version/bin5.png)
The line were looking at (remember were working with 32bit):

> 01 /r ADD r/m32, r32 MR Valid Valid Add r32 to r/m32.

Following the next hex byte, we get **0xc8** which translate to **11001000** in binary.

Breaking down this binary to it's mod and operands:

Mod: **11**
Operand #2: **001** (ecx)
Operand #1: **000** (eax)

Putting it all together following the intel instruction:

**add eax, ecx**


I hope you get the initial idea!
It was fun writing ***6,776*** lines of code.

>  **https://github.com/anthonys-io/Linear-x86-Disassembler/blob/master/disasm.py**

