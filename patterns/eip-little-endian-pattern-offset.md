# EIP little-endian display vs memory bytes for pattern offset

When a stack-based buffer overflow puts a `msf-pattern_create` / `ERC --pattern c` payload into EIP, the value the debugger prints in the EIP register is **not** what `msf-pattern_offset` / `ERC --pattern o` searches for. The pattern lives on the stack as four bytes, low address to high; EIP loads those four bytes as a little-endian 32-bit integer, then the debugger prints the integer in the natural high-to-low order. So the four ASCII characters shown beside EIP are the four pattern bytes **reversed**.

Translation rule: if EIP displays the ASCII string `c1 c2 c3 c4`, the bytes that actually overwrote the saved return address (memory order, low → high) are `c4 c3 c2 c1`. Search the pattern for `c4 c3 c2 c1`, not `c1 c2 c3 c4`. Run `ERC --pattern o c4c3c2c1` or `msf-pattern_offset -q c4c3c2c1`.

I burned an unfair amount of time on this in the cdextract.exe / Free CD to MP3 lab. x32dbg printed `EIP = 42 35 65 42  "B5eB"`. Feeding `B5eB` into `--pattern o` returned -1. Reversing to `Be5B` returned offset 915, and the rest of the exploit fell into place. The same trick caught a second case where EIP read `1hF0` — the real search string was `0Fh1` (offset 4112).

Quick Python replacement if `msf-pattern_offset` is not on the box:

```python
def msf_pattern(length):
    sets = ['ABCDEFGHIJKLMNOPQRSTUVWXYZ', 'abcdefghijklmnopqrstuvwxyz', '0123456789']
    out, offs = [], [0] * 3
    while sum(len(c) for c in out) < length:
        out.append(''.join(s[o] for s, o in zip(sets, offs)))
        i = 2
        while i >= 0:
            offs[i] += 1
            if offs[i] < len(sets[i]):
                break
            offs[i] = 0
            i -= 1
        if i < 0:
            break
    return ''.join(out)[:length]

def offset_from_eip_display(eip_ascii, length=5000):
    """eip_ascii is the 4-char ASCII from the EIP register column.
    Reverse for little-endian, then search the pattern."""
    return msf_pattern(length).find(eip_ascii[::-1])
```

The same rule applies to ESP and any other register that loads dword-sized values off the stack. Whenever a debugger shows you a register as ASCII, the displayed string is the byte-reversed form of what is actually in memory.
