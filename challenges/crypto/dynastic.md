# Dynastic

**Event:** HTB MCP TryOut · **Category:** crypto · **Points:** 725 · **Difficulty:** very easy

## TL;DR

Positional Caesar, also known as the Trithemius cipher. Each letter's offset is its own index in the string. Decrypt by subtracting `i` instead of adding it.

## What I saw first

`source.py` shows `encrypt` adding the character index `i` to each alphabetic byte's `ord - 0x41`, modulo twenty-six, and leaving non-alphabetic bytes alone. The output is in `output.txt`.

## What I tried that did not work

Submitted the decrypted text in lowercase first, the way I usually submit flags. Rejected. Resubmitted with the original case preserved. Accepted. The platform takes the literal text the challenge intends, not a normalized form.

## What worked

```python
ct = open("output.txt").read().strip()
pt = ""
for i, c in enumerate(ct):
    if c.isalpha():
        base = ord('A') if c.isupper() else ord('a')
        pt += chr((ord(c) - base - i) % 26 + base)
    else:
        pt += c
print(pt)
# HTB{DID_YOU_KNOW_ABOUT_THE_TRITHEMIUS_CIPHER?!_IT_IS_SIMILAR_TO_CAESAR_CIPHER}
```

## Flag

`HTB{DID_YOU_KNOW_ABOUT_THE_TRITHEMIUS_CIPHER?!_IT_IS_SIMILAR_TO_CAESAR_CIPHER}`

## What this taught me

Submit the flag in the case the source produces. Do not auto-lowercase. HTB's MCP TryOut platform accepts the literal text, not a canonical form.
