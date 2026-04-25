# Character

**Event:** HTB MCP TryOut · **Category:** misc · **Points:** 825 · **Difficulty:** very easy

## TL;DR

Raw TCP oracle. Server prompts for an index and returns the character of the flag at that position. Loop until you hit `}`. The flag is 104 characters long, which is longer than most index-loops default to.

## What I saw first

```
Which character (index) of the flag do you want?
Enter an index: 0
Character at Index 0: H
```

A single index per query. Keep the connection open between queries.

## What I tried that did not work

First loop ran zero to seventy-nine. Standard flag length. The server ran out of flag at eighty and started repeating, and my loop terminated without ever seeing the closing `}`. Had to extend.

## What worked

```python
import socket, re
s = socket.socket(); s.connect((host, PORT))
flag = ""
for i in range(300):
    s.send(f"{i}\n".encode())
    chunk = b""
    while b"Enter an index:" not in chunk:
        chunk += s.recv(4096)
    m = re.search(rb"Character at Index \d+: (.)", chunk)
    if not m: break
    c = m.group(1).decode()
    flag += c
    if c == "}": break
print(flag)
```

## Flag

`HTB{tH15_1s_4_r3aLly_l0nG_fL4g_i_h0p3_f0r_y0Ur_s4k3_tH4t_y0U_sCr1pTEd_tH1s_oR_els3_iT_t0oK_qU1t3_l0ng!!}`

## What this taught me

Default the index ceiling to three hundred, not eighty. Long-flag challenges exist specifically to punish people who hardcoded their bounds.
