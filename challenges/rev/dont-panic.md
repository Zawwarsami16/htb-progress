# Don't Panic

**Event:** HTB MCP TryOut · **Category:** rev · **Points:** 875 · **Difficulty:** very easy

## TL;DR

Rust binary that reads thirty-one bytes and runs each byte through a per-position closure stored in an array of function pointers. Every closure is the same shape, `cmp $0xNN, %dil ; jb panic ; jne panic ; ret`, where `NN` is the only thing that differs. Pull the immediate out of each closure, concatenate, recover the flag. No dynamic analysis needed.

## What I saw first

`_ZN3src10check_flag…` is not stripped. The function reads thirty-one bytes and dispatches each through a closure at a fixed slot in an on-stack array. The closures are at `0x8a40, 0x8a80, 0x8ac0, …` stride `0x40`.

## What I tried that did not work

First instinct was `angr` or symbolic execution. Overkill. The closures are not branchy. They are a comparison with a fixed immediate and a return. The immediate is the answer.

## What worked

```bash
objdump -d ./dontpanic | grep -A1 "cmp.*%dil" | grep "cmp.*\\$0x" | head -40
# Pull the immediates in order. Each immediate is one byte of the flag.
```

The slot-to-position mapping is `(offset - 0x10) / 8`, so the closure at stack offset `0x10` checks position 0, `0x18` checks position 1, and so on.

```
0 H · 1 T · 2 B · 3 { · 4 d · 5 0 · 6 n · 7 t · 8 _
9 p · 10 4 · 11 n · 12 1 · 13 c · 14 _ · 15 c · 16 4
17 t · 18 c · 19 h · 20 _ · 21 t · 22 h · 23 e · 24 _
25 3 · 26 r · 27 r · 28 o · 29 r · 30 }
```

## Flag

`HTB{d0nt_p4n1c_c4tch_the_3rror}`

## What this taught me

Rust per-byte closure-array challenges decompose without symbolic execution. Each closure is a template. Extract `(slot_offset, immediate_byte)` pairs and concatenate. Symbolic execution is for branching state spaces. This is a lookup table dressed up as code.
