# Ghidra

## Simply reverse 

```c
  if (iVar1 == 0x1a7) { // 0x1a7 = 423
    local_20 = strdup("/bin/sh");
    local_1c = 0;
    local_14 = getegid();
    local_18 = geteuid();
    setresgid(local_14,local_14,local_14);
    setresuid(local_18,local_18,local_18);
    execv("/bin/sh",&local_20);
```

If we exec ./level0 with 423 as argv[1] its win !