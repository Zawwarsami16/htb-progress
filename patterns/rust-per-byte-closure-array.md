# Rust per-byte closure array

**Trigger:** a reversing binary, Rust-compiled, with a stack-resident array of function pointers, one slot per character of the expected input.

Rust closure-based verifiers look intimidating until you notice the closures are not actually doing different things. Each closure in the array is the same template:

```
cmp $0xNN, %dil
jb panic
jne panic
ret
```

The only thing that varies between closures is the immediate `NN` in the compare. That immediate is the byte the closure expects at that position.

The static recovery is mechanical. For each closure address, pull the immediate. Map slot offset to position via `(offset - first_slot) / 8`. Concatenate the bytes in order. That is the flag.

No symbolic execution. No emulation. No `angr`. The verifier is not branching on input; it is comparing input against a fixed table that the compiler spread out across an array of trivially-similar closures.

If the assembly looks more complex than the template above, the closures probably do something genuinely different per position, and dynamic analysis becomes worth it. If they all match the template, static is faster.

**Seen in:** [Don't Panic](../challenges/rev/dont-panic.md).
