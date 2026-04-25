# Decode the jump table, not the prompt

**Trigger:** a movement or input system whose keys "obviously" map to a known scheme.

When a reversing challenge has movement keys, or any other input that selects between actions, the prompt string is allowed to lie. The author wrote the prompt. They also wrote the jump table that maps your input bytes to handler functions. The two are separate sources of truth, and the cheap trick is to make them disagree.

A 3D maze that prompts "L/R/F/B/U/D" almost certainly means *something* by each letter, but not necessarily what compass directions or video games would suggest. Decode the table. Read the immediate the handler increments or decrements. Trust that, not the prompt.

The same principle applies to:

- Menu-driven C programs where option numbers print one description but call a different function.
- "Configuration languages" that look like INI but parse some keys as integers and some as strings of digits.
- Cryptography challenges that name a function `decrypt` and have it call the encrypt path with a different key.

The table is the truth. The prompt is decoration. Read the table.

**Seen in:** [Tunnel Madness](../challenges/rev/tunnel-madness.md).
