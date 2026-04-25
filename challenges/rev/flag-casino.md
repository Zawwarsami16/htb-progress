# FlagCasino

**Event:** HTB MCP TryOut · **Category:** rev · **Points:** 800 · **Difficulty:** very easy

## TL;DR

Twenty-nine rounds of `scanf("%c") ; srand(c) ; rand()` compared against a hardcoded twenty-nine-entry table in `.data`. Each input byte deterministically maps to one `rand()` output. Brute-force the byte values per slot through libc's `srand`/`rand` and recover the flag.

## What I saw first

`main` loops `i = 0 .. 0x1c`. Reads a byte, seeds the RNG with it, calls `rand()` once, compares against `check[i]`. If all twenty-nine pass, prints the flag.

## What I tried that did not work

Wrote my own `srand`/`rand` reimplementation first. Mine drifted from libc's after the third byte because I had a subtle integer-widening bug. Switched to calling libc directly through `ctypes`, which removed the bug source entirely.

## What worked

```python
import ctypes
libc = ctypes.CDLL("libc.so.6")

# 1. Read the check[] table out of the .data segment.
# .data is at vaddr 0x4060, file offset 0x3060. check[] starts at vaddr 0x4080,
# file offset 0x3080. 29 little-endian 32-bit ints.
import struct
with open("casino","rb") as f:
    f.seek(0x3080)
    expected = list(struct.unpack("<29i", f.read(29*4)))

# 2. For each expected[i], brute byte values 1..255 to find which one produces it.
flag = b""
for want in expected:
    for c in range(1, 256):
        libc.srand(c)
        if libc.rand() == want:
            flag += bytes([c]); break
print(flag)
```

## Flag

`HTB{r4nd_1s_v3ry_pr3d1ct4bl3}`

## What this taught me

Never reimplement libc functions when you can just call them. `ctypes` plus `CDLL("libc.so.6")` gives you the exact bytes the target was compiled against. Reimplementations always drift on the third or fourth edge case.
