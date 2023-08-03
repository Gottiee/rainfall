# Reverse

## Context

:warning: NX enable

The program stock the pass word inside buffer[16].

Main() quit if there isn't two arg, or if you can't open the password (for exemple you can't reverse it with gdb):

```c
if ((pass_fd == (FILE *)0x0) || (argc != 2)) {
	i = -1;
```

Then it atoi our argv[1]. And write '\0' inside buffer[16] at i position.

```c
int i = atoi(argv[1]);

buffer[i] = '\0';
```

If the buffer equal the argv[1], it spawn a shell.

```c
i = strcpm(buffer[16], argv[1]).
if (i == 0)
	/bin/sh
```

## Exploit

If you `./bonus3 ""` its win.

```c
atoi("") = 0;
buffer[0] = '\0';
strcmp(buffer, argv[1]);
//strcmp("", "");
//strcmp return 0, and it spawn a shell
```