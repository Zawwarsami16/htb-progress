# Pwn

The first thing I do on a pwn binary is `checksec`. The second thing I do is read main. The third thing I do is read the function that takes user input. By that point the path is usually visible.

## The first pass

```bash
checksec ./binary
file ./binary
strings ./binary | grep -i flag
ROPgadget --binary ./binary | head -20
```

`checksec` tells me which mitigations are off. If NX is off, shellcode is a tool. If PIE is off, addresses are constants. If canary is off, classic overflow works. If RELRO is partial, GOT overwrites are on the table.

Reading main is fast. The expected-flag function is usually one of three: it is called from main on a specific condition (overflow flips the condition), it is reachable through a function pointer that overflow corrupts, or it reads `flag.txt` after a check that the input must satisfy. Each path implies a different exploit class.

## What I check before anything else

- The wrapper around `read` or `scanf` or whatever takes user input. The overflow length is rarely just "buffer size minus saved RIP". The wrapper often sets up registers, and those registers persist past the wrapper return. A `lea rsi, [rsp]` that survives the return is a gadget shaped like nothing.
- Whether RWX segments exist. `readelf -l` will tell you. If yes, shellcode lives in the buffer.
- Whether there is a `win()` function or a `read_flag()` function. Tutorial binaries put it in plaintext, often with a clear name.

## What I avoid

Reaching for `ret2libc` on a static binary. Static binaries have no PLT and no libc symbols. ROP into syscalls is the right answer.

Spending more than five minutes building a ROP chain before checking whether shellcode in the buffer works. RWX segments make ROP unnecessary.

Symbolic execution on a binary that has a single linear path. `angr` is a fantastic tool when the state space branches. It is also slow to set up. If main has one path and one constraint, the right move is to read it.

## What I keep handy

- `pwntools` template with `context.binary` already set so I can `asm()` strings inline.
- A short table of one-byte shellcode primitives for the common syscalls: `open`, `read`, `write`, `execve`. The `flag.txt` constant `0x7478742e67616c66` is in my muscle memory.
- A `ropper` invocation that filters to `pop`-only gadgets first, then `mov`-to-reg gadgets.
- `seccomp-tools dump ./binary` for binaries that drop into a sandbox before reading input.
