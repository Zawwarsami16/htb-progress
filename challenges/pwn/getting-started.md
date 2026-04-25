# Getting Started

**Event:** HTB MCP TryOut · **Category:** pwn · **Difficulty:** very easy

## TL;DR

Tutorial buffer overflow. A forty-eight-byte stack buffer sits next to an eight-byte target qword initialized to `0xdeadbeef`. If the target is not `0xdeadbeef` when `main` returns, the program calls `win()` which reads the flag. Forty bytes of padding plus any single byte flips the target.

## What I saw first

`main` calls `scanf("%s", buf)` into a forty-eight-byte buffer at `-0x30(%rbp)`. The target qword is at `-0x8(%rbp)`. The win condition is `if (target != 0xdeadbeef)`. The win function reads `flag.txt` and prints it.

## What I tried that did not work

Started with a forty-eight-byte payload, expecting to overwrite the saved RBP first and then the target. Off by a few. The buffer is at `-0x30` so the gap to `-0x8` is forty bytes, not forty-eight.

## What worked

```python
payload = b"A"*40 + b"X" + b"\n"
```

`scanf` null-terminates one past the input, so even a single non-null byte at offset forty is enough to make the target not equal to `0xdeadbeef`.

## Flag

`HTB{b0f_tut0r14l5_4r3_g00d}`

## What this taught me

Read the win condition first on tutorial-shaped binaries. The constraint is often inverted from what you expect. Here the program wants the canary to be *wrong*, not right.
