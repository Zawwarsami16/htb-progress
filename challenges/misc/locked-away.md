# Locked Away

**Event:** HTB MCP TryOut · **Category:** misc · **Points:** 1000 · **Difficulty:** very easy

## TL;DR

Python `exec()` on user input with a substring blacklist that blocks `import`, `eval`, `open`, both quote characters, both bracket characters, and a long list of keywords. The bypass: there is already a function `open_chest()` defined in the module that reads the flag, and you can reach it by constructing its name from `chr()` calls and calling it through `globals().get(...)`.

## What I saw first

```python
banned = ["import","os","sys","breakpoint","flag","txt","read","eval","exec",
          "dir","print","subprocess","[","]","echo","cat",">","<","\"","'", "open"]

def open_chest():
    open('flag.txt').read()  # then prints
```

No quotes, no brackets, no `open`. Any of those substrings in your input is rejected.

## What I tried that did not work

`getattr(__builtins__, ...)` is dead because `__` is fine but no string literal can survive without quotes. Tried building strings with `chr(...)` then concatenation, which works, then tried to use them as subscripts into `globals()` and `__builtins__`. Subscript needs brackets. Brackets are banned. Tried `.__getitem__(...)` which works but the name `__getitem__` would need to be a string literal or built from `chr()`, and the result is fine until you remember the dot still requires the attribute to exist, which `__getitem__` does on dicts.

Got close, but the cleaner path was already sitting there.

## What worked

Dicts have a `.get(key)` method. No brackets needed. `globals()` returns the module's namespace as a dict. Build the function name as a string of `chr()` calls, do `.get(name)`, get the function back, call it.

```python
globals().get(chr(111)+chr(112)+chr(101)+chr(110)+chr(95)+chr(99)+chr(104)+chr(101)+chr(115)+chr(116))()
# constructs "open_chest", retrieves the function, invokes it
```

## Flag

`HTB{bYp4sSeD_tH3_fIlT3r5?_aLw4Ys_b3_c4RefUL!_7548307883c08de04e93bd9531bdcead}`

## What this taught me

Substring blacklists on `exec` fall over when both quotes and brackets are denied if `chr()` and method-call syntax are still available. The substitution is mechanical. String literal becomes `chr()+chr()+...`. Subscript becomes `.get()`. The grammar of the language has more than one way to do most things, and the blacklist almost never covers all of them.
