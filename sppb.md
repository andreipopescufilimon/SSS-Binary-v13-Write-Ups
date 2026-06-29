# sppb

After decompiling the binary, I found the password check:
```c
printf("Your password is: %d. Evaluating it...\n", readValue);
sleep(2);

if (((99 < readValue) && (readValue == (readValue / 0xd) * 0xd)) && (readValue < 0x143)) {
    password_accepted();
}
```

The condition means:
```c
readValue > 99
readValue % 13 == 0
readValue < 0x143
```

`0xd` is `13`, and `0x143` is `323`.

So the password has to be a number bigger than `99`, smaller than `323`, and divisible by `13`. One valid value is:  104. I used `104`, which passes the check:
```bash
andreifilimon@DESKTOP-UBHTODG:~$ nc 141.85.224.104 42069
Please provide password:
104
Your password is: 104. Evaluating it...
ls
bin
boot
dev
etc
home
lib
lib64
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
```

After getting shell access, I listed the CTF directory:
```bash
ls home/ctf
```

Output:
```text
flag
sppb
```

Then I read the flag:
```bash
cat home/ctf/flag
SSS_CTF{decompiling_spoils_the_fun}
```
