# cdkey

After opening the binary in Ghidra, the main function was identified as `FUN_00101728`. Inside this function, the program initializes a buffer with the string:
```c
builtin_strncpy(local_218, "sharingiscaring", 0x10);
```

This string is used as a key. The program also stores an encrypted password inside `local_488`:
```c
local_488[0] = 'E';
local_488[1] = -0x4b;
local_488[2] = -0x2d;
local_488[3] = -0x6c;
...
local_488[0x1d] = '}';
```

Then the binary calls two functions:
```c
FUN_00101397(&local_118, local_218, sVar2);
FUN_00101603(&local_118, local_488, local_10);
```

The first function initializes the crypto state using the key. The second function decrypts `local_488` in place. The structure of the algorithm matches RC4: it creates a 256-byte state array, shuffles it using the key, then XORs the encrypted bytes with the generated keystream. After decrypting `local_488`, the binary reads user input:
```c
__isoc99_scanf(&DAT_0010200c, local_288);
```

Then it compares our input with the decrypted password:
```c
iVar1 = strncmp(local_288, local_488, 0x1e);
if (iVar1 == 0) {
    FUN_00101377();
}
```

The function `FUN_00101377` is the win function, because it executes:
```c
execve("/bin/sh", 0, 0);
```

The encrypted bytes from `local_488` were copied into a Python script. Ghidra shows some bytes as negative values because they are signed chars, so they must be converted to unsigned bytes.

For example:
```text
-0x4b -> 0xb5
-0x2d -> 0xd3
-0x6c -> 0x94
```

The decryption script:

```python
key = b"sharingiscaring"

enc = bytes([
    0x45, 0xb5, 0xd3, 0x94, 0xf9, 0xb8, 0x55, 0x50,
    0xdd, 0x3c, 0xa9, 0x86, 0x7b, 0x93, 0x2a, 0x21,
    0x1c, 0x89, 0xb1, 0xf2, 0x87, 0x57, 0xe2, 0xf6,
    0xa5, 0xa3, 0x64, 0x16, 0xcd, 0x7d
])

S = list(range(256))
j = 0

for i in range(256):
    j = (j + S[i] + key[i % len(key)]) & 0xff
    S[i], S[j] = S[j], S[i]

i = 0
j = 0
out = []

for byte in enc:
    i = (i + 1) & 0xff
    j = (j + S[i]) & 0xff
    S[i], S[j] = S[j], S[i]

    k = S[(S[i] + S[j]) & 0xff]
    out.append(byte ^ k)

print(bytes(out).decode())
```

Running it gives:

```text
SSS_CTF{IS_THIS_THE_REAL_FLAG}
```

I connected to the remote service: Then I entered the decrypted password: `SSS_CTF{IS_THIS_THE_REAL_FLAG}`

The password matched, so the program called the win function and spawned a shell. From there, I listed the files and read the flag:
```bash
ls
ls /home/ctf
cat /home/ctf/flag
SSS_CTF{Found_iT_d0n_T_n33d_hlp_anymor3}
```
