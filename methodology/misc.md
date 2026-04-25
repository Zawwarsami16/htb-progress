# Misc

Misc is the category for anything that does not fit the other categories cleanly. In practice it is dominated by three subgenres: TCP scenario games, Python sandbox escapes, and chains that cross multiple technologies in a single challenge.

## TCP scenario games

A server prompts the player with a fixed set of states and accepts a fixed set of replies. The player has to read the prompt, decide on the right action, and respond before the server times out. Some run a hundred-plus rounds before emitting the flag.

The right move is always non-blocking I/O with synchronization on the prompt itself rather than newlines. Servers are allowed to batch a death message and the next prompt into a single TCP segment, and a line-buffered reader will stall waiting for a newline the server already considers sent.

## Python sandbox escapes

A server runs `exec()` or `eval()` on user input with a substring blacklist. The blacklist invariably blocks `import`, `eval`, `exec`, `open`, the file names, both quote characters, both bracket characters, and a long list of method names.

The bypass tree branches at the first thing they forgot:

- If `chr()` is allowed, you can build any string from char codes.
- If `globals()` is allowed, you can reach module-scope variables.
- If method-call syntax is allowed but subscript is not, `dict.get(...)` substitutes for `dict[...]`.
- If `__getattr__` is reachable through method resolution, you can chain to `__import__` indirectly.
- If `breakpoint()` is allowed (rare), you have an immediate `pdb` shell.

The pattern is: build the target name as `chr()+chr()+...`, reach it via `globals().get(name)` (no brackets), and call it. The blacklist almost never covers all of these at once.

## Chains across technologies

The interesting misc challenges chain four or five technologies. nginx routes by Host header. The app exposes a server-side fetch. The fetch hits a verdaccio registry. The registry accepts your malicious package. A cronjob installs it. The flag reads through the original fetch.

For these the discipline is to map the topology first. What talks to what. Where the trust boundaries are. Which component is the user-facing surface and which is internal but reachable through the right Host header. Draw the graph before writing exploit code.

## What I keep handy

- A non-blocking-socket template that synchronizes on a specific prompt string.
- A short list of Python sandbox bypasses for the common blacklist shapes.
- A topology-sketch template for multi-component chains. One box per service, arrows for the protocols, ports labeled.
