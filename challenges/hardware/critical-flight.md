# Critical Flight

**Event:** HTB MCP TryOut · **Category:** hardware · **Difficulty:** very easy

## TL;DR

Gerber files for a four-layer PCB. The silkscreen layers are decoys. The flag is split across two internal copper layers, rendered as visible text in the copper traces.

## What I saw first

A standard Gerber set: top silk, bottom silk, top copper (`F_Cu`), bottom copper (`B_Cu`), two internal copper layers (`In1_Cu`, `In2_Cu`), drill files, board outline.

## What I tried that did not work

Rendered the silkscreens first. Nothing useful. Rendered the top and bottom copper. The top copper has functional traces. The bottom copper has half the flag near the pin header, but I missed it on the first pass at low DPI because copper text reads like fill polygons until you zoom in.

## What worked

```bash
gerbv --export=png --dpi=600 --output=bcu.png HadesMicro-B_Cu.gbr
gerbv --export=png --dpi=600 --output=in1.png HadesMicro-In1_Cu.gbr
gerbv --export=png --dpi=600 --output=in2.png HadesMicro-In2_Cu.gbr
# Open all three at 1:1. The flag halves are visible in B_Cu (left of the header)
# and In1_Cu (right of center). In2_Cu has no text.
```

`HadesMicro-B_Cu.gbr` has `HTB{533_7h3_1nn32_w02k1n95`. `HadesMicro-In1_Cu.gbr` has `_0f_313c720n1c5#$@}`. Concatenated, that is the flag.

## Flag

`HTB{533_7h3_1nn32_w02k1n95_0f_313c720n1c5#$@}`

## What this taught me

Hardware challenges with Gerber sets need every copper layer rendered, not just silkscreen. Designers regularly bury flag text inside copper fills, and copper fills look like polygons until you render at six hundred DPI. The render-everything-at-high-DPI step is now my first move on any Gerber set.
