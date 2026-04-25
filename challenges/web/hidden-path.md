# Hidden Path

**Event:** HTB MCP TryOut · **Category:** web (filed as misc on the platform) · **Points:** 1000 · **Difficulty:** very easy

## TL;DR

An Express app exposes six numbered commands and runs whichever one you pick. The seventh element of the command array is a variable whose name is a Unicode character that renders as zero pixels, so the source looks like it has six commands when it really has seven. Set the invisible body field to your command, set the choice to six, and the validator does not catch you.

## What I saw first

`POST /server_status` reads a numeric `choice` from the body, looks it up in an array, and calls `exec()` on the result. Standard pattern. The array has six clearly named commands like `uptime` and `ps aux`. The flag is not one of them, and the validator clamps `choice` to the range zero through five. The source looks airtight on a casual read.

```js
const { choice } = req.body;
const commands = ["uptime", "free -h", "df -h", "uname -a", "whoami", "ps aux"];
exec(commands[choice]);
```

It is not airtight. There is an extra character in two places, and they are not visible in any editor that renders sane fonts at normal zoom.

## What I tried that did not work

I spent the first ten minutes inspecting the six commands themselves, looking for argument injection or for one of them to do something more interesting than it appeared. None of them did. I tried path traversal through the filenames the commands operated on. I tried sending non-numeric choice values to see if the validator would coerce them to something useful. The validator was honest, and `choice` outside zero-to-five just returned a polite error. The bug was not in the validator or in the commands. It was in the source itself, in a place my eye did not look.

## What worked

The destructure line in the handler actually reads `const { choice, ㅤ } = req.body`. That second identifier is `U+3164`, the Hangul Filler character. It is a valid JavaScript identifier. It is also a real key in the request body if you supply one. The commands array has the same character as its seventh element. The validator checks `choice >= 0 && choice <= 5`, not `choice < commands.length`. Set `choice` to six and the invisible field to your command of choice, and the server runs it.

```bash
curl -X POST "$HOST/server_status" \
  --data-urlencode "choice=6" \
  --data-urlencode $'\xe3\x85\xa4=cat /flag.txt'
```

The `%E3%85%A4` is the UTF-8 encoding of `U+3164`. Most HTTP clients let you set it as a key without complaining.

## Flag

`HTB{1nvi5IBl3_cH4r4cT3rS_n0t_sO_v1SIbL3_7af78ef07c0815ed395ecdc02508ac83}`

## What this taught me

Source review is a visual operation. Anything your editor does not render, your eye does not catch. The fix is a habit, not a tool. Before reading a file you suspect, pipe it through three lines of Python that print every non-ASCII codepoint as a hex code. You will not get ambushed by a character class that decided to be invisible.

```python
for c in open("server.js").read():
    if ord(c) > 127:
        print(repr(c), hex(ord(c)))
```

The boring lesson under the colorful trick is older. Validators that clamp inputs to a known range and arrays that hold the same inputs are two different sources of truth. If you change one and forget to change the other, the gap is exactly the size of one off-by-one bug, and the bug will sit there for as long as the code does.
