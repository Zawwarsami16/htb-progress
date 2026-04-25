# Satellite Hijack

**Event:** HTB MCP TryOut · **Category:** rev · **Points:** 900 · **Difficulty:** very easy

## TL;DR

A binary plus a stripped shared library. The binary hijacks the `read` GOT entry, points it at a JIT page decoded from a section of `library.so` via XOR-42, and the JIT verifies a twenty-eight-byte input against a key the binary loads onto the stack. Pull the key out statically, XOR it with `0..27`, and the flag falls out. The binary never has to run.

## What I saw first

`send_satellite_message` is called from main. It decodes a stack string from a `-1` rolling cipher to get `SAT_PROD_ENVIRONMENT`, calls `getenv` on it, and if set, runs a trampoline at `0x23e3`. The trampoline walks the program headers, finds the GOT entry for `read`, `mmap`s an RWX page, copies bytes `0x11a9..0x21a9` from `library.so` into it, and runs `memfrob` (XOR with 42) on the page. Then it overwrites the GOT entry with the page address. All future `read` calls go through the decoded JIT.

## What I tried that did not work

Tried to set the environment variable and run the binary locally. That triggered the verifier path, but the verifier is at `JIT_PAGE + 0x8c`, which only exists after the trampoline runs, which only happens when you trigger the special path. Lots of moving parts to set up for what turned out to be a recoverable static problem.

## What worked

```python
import struct

# Extract bytes 0x11a9..0x21a9 from library.so, XOR each byte with 42.
data = open("library.so","rb").read()[0x11a9:0x21a9]
jit  = bytes(b ^ 42 for b in data)

# The verifier at offset 0x8c loads a 28-byte key via four overlapping movabs writes
# into stack slots at -0x23, -0x1b, -0x13, and so on. After resolving the overlaps:
key = bytes([
    0x6c, 0x35, 0x7b, 0x30, 0x76, 0x30, 0x59, 0x37,
    0x66, 0x56, 0x66, 0x3f, 0x75, 0x3e, 0x7c, 0x3a,
    0x4f, 0x21, 0x7c, 0x4c, 0x78, 0x21, 0x6f, 0x24,
    0x6a, 0x2c, 0x3b, 0x66,
])

# Verifier loop: for i in 0..27, assert (input[i] ^ key[i]) == i. Recover input.
flag_inner = "".join(chr(k ^ i) for i, k in enumerate(key))
print("HTB{" + flag_inner)
# HTB{l4y3r5_0n_l4y3r5_0n_l4y3r5!}
```

## Flag

`HTB{l4y3r5_0n_l4y3r5_0n_l4y3r5!}`

## What this taught me

A GOT hijack that mmaps an encoded page from a known offset, decodes it with a known transform, and runs a fixed verifier is statically recoverable end to end. Decode the page yourself, find the verifier inside it, identify the key buffer, invert the math. The binary never runs. Symbolic execution and emulation are for adversarial state spaces, not for chains of fixed transforms.

Also: be careful with overlapping `movabs` writes to adjacent stack slots. The later write only partially overwrites the earlier one. You have to resolve the layout byte-by-byte, not assume each `movabs` is independent.
