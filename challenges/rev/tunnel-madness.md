# Tunnel Madness

**Event:** HTB MCP TryOut · **Category:** rev · **Points:** 1000 · **Difficulty:** very easy

## TL;DR

Twenty-by-twenty-by-twenty 3D maze in `.rodata`. Eight thousand cells, sixteen bytes each, cell type at offset twelve. BFS finds the goal in about sixty moves. The catch is the movement keys. The prompt says L/R/F/B/U/D mean what they normally mean. They do not. The real axes are decoded from a twenty-entry jump table.

## What I saw first

A `.rodata` block one hundred and twenty-eight kilobytes long starting at the `maze` symbol. Eight thousand sixteen-byte cells. Type byte at offset twelve: zero is floor, one is alternate floor, two is wall, three is goal. The movement function indexes a twenty-entry jump table by `toupper(input) - 'B'`.

## What I tried that did not work

Trusted the prompt. Built a BFS that treated L as `x--`, R as `x++`, U as `z++`, and so on. The path it produced got me to position seven and then stopped working. Spent fifteen minutes thinking the maze data was wrong before reading the jump table.

## What worked

Decoded the jump table. The mapping is:

- L → x--, R → x++
- B → y--, F → y++
- D → z--, U → z++

Note that `B` decrements `y`, not increments. And `D` decrements `z`. The prompt swapped two pairs deliberately.

```python
maze = open("binary","rb").read()[0x20e0 : 0x20e0 + 128000]
def cell(x,y,z): return maze[((z*400) + (y*20) + x) * 16 + 12]

# BFS from (0,0,0). Wall = 2, goal = 3.
from collections import deque
q = deque([((0,0,0), "")])
seen = {(0,0,0)}
moves = {
    "R": ( 1, 0, 0), "L": (-1, 0, 0),
    "F": ( 0, 1, 0), "B": ( 0,-1, 0),
    "U": ( 0, 0, 1), "D": ( 0, 0,-1),
}
while q:
    (x,y,z), path = q.popleft()
    if cell(x,y,z) == 3:
        print(path); break
    for k,(dx,dy,dz) in moves.items():
        nx,ny,nz = x+dx, y+dy, z+dz
        if 0 <= nx < 20 and 0 <= ny < 20 and 0 <= nz < 20 \
           and (nx,ny,nz) not in seen and cell(nx,ny,nz) != 2:
            seen.add((nx,ny,nz)); q.append(((nx,ny,nz), path + k))
```

Sixty moves to (19, 19, 19). Send each letter on its own line. The binary prints "You break into the vault" and reads `flag.txt`.

## Flag

`HTB{tunn3l1ng_ab0ut_in_3d_e102cc4f47583ea05ec618e980a3de05}`

## What this taught me

Always reconstruct movement axes from the jump table. The prompt is text the author can lie in. The jump table is the truth. This rule has now caught two challenges for me where the obvious mapping was deliberately wrong.
