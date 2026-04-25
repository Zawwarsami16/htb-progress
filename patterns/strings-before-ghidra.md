# Strings before Ghidra

**Trigger:** a reversing challenge with a name that frames the task as finding something.

LootStash. Stashed Treasure. Find the Loot. Hidden Gem. Buried Secret. Whatever the variant, the title is telling you the binary contains the flag as a literal string, and the difficulty rating ("very easy", "easy") is confirming it.

Run `strings | grep HTB{` before opening Ghidra. Every single time.

```bash
strings binary | grep -E "HTB\{"
```

When this works, it returns the flag in under a second. When it does not work, you have spent that second and learned that the flag is encoded, packed, or computed at runtime, and you should now open Ghidra. The cost of the failed attempt is negligible. The cost of opening Ghidra and disassembling a binary that has the flag in plaintext is fifteen minutes you will never get back.

The same rule applies to forensics challenges with custom protocols. A "custom protocol" that the challenge is teaching you about is allowed to be a fancy framing around plaintext. `strings | grep HTB` on the pcap is the same preflight.

**Seen in:** [LootStash](../challenges/rev/lootstash.md), [Shush Protocol](../challenges/hardware/shush-protocol.md).
