# Shush Protocol

**Event:** HTB MCP TryOut · **Category:** ics · **Points:** 800 · **Difficulty:** very easy

## TL;DR

PCAPNG with a "custom protocol" that does no actual obfuscation. The flag sits as a plain ASCII string in the payload.

## What I saw first

A capture file. The name suggested an ICS or SCADA-flavored protocol dissection.

## What I tried that did not work

Opened the capture in Wireshark and started looking for the protocol's framing. Wasted four minutes before remembering the cheat path.

## What worked

```bash
strings -n 8 traffic.pcapng | grep HTB
# HTB{50m371m35_cu570m_p2070c01_423_n07_3n0u9h7}
```

## Flag

`HTB{50m371m35_cu570m_p2070c01_423_n07_3n0u9h7}`

## What this taught me

Always `strings | grep HTB` on a forensics challenge before diving into protocol dissection. The challenge name is the hint here. A "custom protocol that is not enough" is a custom protocol that carries plaintext.
