# Stop Drop and Roll

**Event:** HTB MCP TryOut · **Category:** misc · **Points:** 825 · **Difficulty:** very easy

## TL;DR

Raw TCP scenario game. Server prints three uppercase event tokens, asks "What do you do?", and you reply with a dashed sequence of action keywords. The mapping is fixed: `GORGE→STOP`, `PHREAK→DROP`, `FIRE→ROLL`. Run a hundred-plus rounds, get the flag.

## What I saw first

`start_container` returns a host and a port. The service rejects HTTP. Raw TCP only. The first byte is a yes/no prompt, then the rounds start.

## What I tried that did not work

A line-buffered `recv` loop missed the prompts twice. The server sometimes batches the death message and the next prompt into a single chunk, so reading until newline can stall.

## What worked

Non-blocking socket. Read everything available. Synchronize on the literal string "What do you do?" and not on the newline. Extract the uppercase tokens from the segment before it, translate them, send back the dashed answer.

```python
import socket, re
s = socket.socket()
s.connect(("host", PORT))
s.setblocking(False)
buf = b""
mapping = {"GORGE":"STOP", "PHREAK":"DROP", "FIRE":"ROLL"}

def step():
    global buf
    while True:
        try: buf += s.recv(4096)
        except BlockingIOError: break
    if b"(y/n)" in buf:
        s.send(b"y\n"); buf = b""
        return
    if b"What do you do?" in buf:
        scene = buf.split(b"What do you do?")[0]
        toks = re.findall(rb"\b[A-Z]{4,}\b", scene)
        reply = "-".join(mapping.get(t.decode(), "?") for t in toks[-3:])
        s.send(reply.encode() + b"\n"); buf = b""
```

## Flag

`HTB{1_wiLl_sT0p_dR0p_4nD_r0Ll_mY_w4Y_oUt!}`

## What this taught me

For TCP automation against a server that prompts with text, do not synchronize on newlines. Synchronize on the prompt itself. The server is allowed to batch a death message and the next prompt into one TCP segment, and a line-buffered reader will stall waiting for a newline that the server already considers sent.
