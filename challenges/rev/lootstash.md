# LootStash

**Event:** HTB MCP TryOut · **Category:** rev · **Difficulty:** very easy

## TL;DR

`strings | grep HTB{` returns the flag. The binary is not packed, the flag is not obfuscated, and the file name should be a tell.

## What I saw first

A dynamically-linked PIE x86-64 binary called `stash`. Not stripped.

## What I tried that did not work

Opened Ghidra out of habit before checking `strings`. Lost ninety seconds.

## What worked

```bash
strings stash | grep -E "HTB\{"
# HTB{n33dl3_1n_a_l00t_stack}
```

## Flag

`HTB{n33dl3_1n_a_l00t_stack}`

## What this taught me

Very-easy rev challenges with names like "Stash" or "Find the Loot" or "Hidden Treasure" are almost always literal string searches. Run `strings | grep HTB` before Ghidra. Every single time.
