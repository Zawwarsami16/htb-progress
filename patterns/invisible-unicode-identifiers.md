# Invisible Unicode identifiers

**Trigger:** the source code looks too clean to be exploitable.

Most languages with first-class Unicode identifier support let you name a variable with a codepoint that renders as zero pixels. `U+3164` (Hangul Filler) is the canonical example. `U+200B` (zero-width space) and `U+200D` (zero-width joiner) also bind as identifiers in JavaScript and a few other languages, though some parsers strip them.

If a CTF challenge or audit gives you source that "obviously has no bugs", and the source is short enough that you have read every line, run it through a character-class filter before giving up.

```python
for c in open("server.js").read():
    if ord(c) > 127:
        print(repr(c), hex(ord(c)))
```

What you are looking for:

- Identifiers that are not in your editor's visible glyph set.
- Right-to-left or left-to-right overrides (`U+202E`, `U+202D`) inside string literals.
- Zero-width joiner inside what looks like an ASCII keyword.
- Trojan-source tricks where a comment looks like one thing and parses as another.

The fix is a habit, not a tool. Pipe suspicious files through the loop before you read them. The cost is one screen of output. The benefit is that you cannot get ambushed by a codepoint that your editor declined to draw.

**Seen in:** [Hidden Path](../challenges/web/hidden-path.md).
