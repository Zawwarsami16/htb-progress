# Reverse engineering

The first thing I do on a reversing binary is `strings | grep HTB`. The second thing I do is `objdump -d` and read main. The third thing I do is decide whether the verifier is a table lookup, a transform, or a state machine. Each one calls for a different style of recovery.

## The first pass

`strings | grep HTB` catches the very-easy challenges where the flag is literal. It costs one second. Always run it first.

If the flag is not in strings, the binary either holds it encoded, computes it from input, or verifies user input against a fixed table. The shape of `main` and `check_flag` tells me which one.

## Three shapes I see most often

**Table lookup.** The binary holds a per-position array of expected bytes (sometimes as `cmp $imm, %dil` immediates per closure, sometimes as a flat array in `.data`, sometimes as an XOR-mask). Recovery is static: read the immediates, invert the transform, concatenate. No dynamic analysis needed. Rust closure-array verifiers, Caesar-like ciphers, and XOR with a fixed key all sit here.

**Transform.** The binary takes user input, transforms it (XOR by position, RC4 with a fixed key, a custom PRNG seeded by input bytes), and compares the transformed result against a constant. Recovery is the inverse transform applied to the constant. Sometimes the inverse is trivial (XOR is its own inverse), sometimes you have to brute force one byte at a time (a PRNG that does `srand(byte); rand()` is 256-way searchable per slot).

**State machine.** The binary has branching internal state that depends on previous input. You cannot solve byte-by-byte; the path constraints couple. This is where `angr` or `z3` earn their setup cost.

## What I check before anything else

- Is the binary stripped? Unstripped binaries name functions, which save twenty minutes.
- Is it PIE? PIE binaries have to be RE'd at a known load base; pick a base in Ghidra and stick with it.
- Is it Rust, Go, or C? Rust and Go binaries have huge standard-library inlining that drowns the actual logic. Look for the user's `main` first; ignore the runtime.
- Is there a GOT hijack, a `dlopen`, or a `mprotect` of an anonymous page? Those are signs the binary builds a JIT page at runtime, and the JIT is where the real logic lives.

## What I avoid

Running the binary first. Dynamic analysis is useful when static is too slow. Static is rarely too slow on very-easy and easy challenges. Save dynamic for when the constants are computed at runtime.

Symbolic execution as a first move. `angr` is the right tool for branching state spaces. It is the wrong tool for a thirty-iteration loop with one comparison per iteration.

Treating Ghidra's decompiler output as ground truth. The decompiler is helpful and wrong. The assembly is the source of truth. When the decompiler says one thing and the assembly says another, the assembly wins.

## What I keep handy

- Python scripts that read a known binary offset and apply a known transform. XOR-by-constant. XOR-by-position. ROT-N. Trithemius.
- `objdump -d -M intel` is my default disassembler view. AT&T syntax is fine; Intel is faster for my eyes.
- `ctypes.CDLL("libc.so.6")` for calling `srand` and `rand` against the exact bytes the target compiled against. Never reimplement libc functions when you can call them.
