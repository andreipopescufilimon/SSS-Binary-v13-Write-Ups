# Domino

I opened the binary in Ghidra and checked `main`. The program reads an unsigned integer with `scanf("%u", &input)`, then passes it to function `a()`:
```c
scanf("%u", &input);

if (a(input) == 4) {
    system("/bin/bash");
}
```

So the goal is to make `a(input)` return `4`.

The code splits the input into four bytes:
```c
A = input >> 24;
B = (input >> 16) & 0xff;
C = (input >> 8) & 0xff;
D = input & 0xff;
```

Then it checks:
```c
A == 2
B == 0
C >= 9 && C <= 11
D == 1
```

All four conditions must be true.

So a valid value is: `0x02000901`

But the program reads the input as decimal, so we convert it: `0x02000901 = 33556737`

Then I connected to the remote service. Entered the value: `33556737`

This gave a shell. Then I listed the files and read the flag:
```bash
ls home/ctf
cat home/ctf/flag
SSS_CTF{now_you_know_your_abcs}
```
