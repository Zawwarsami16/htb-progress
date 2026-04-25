# Flag Command

**Event:** HTB MCP TryOut · **Category:** web · **Difficulty:** very easy

## TL;DR

A terminal-styled web game has a server-side allow-list that includes a hidden `secret` entry the client never displays. Submit the secret string and the server hands over the flag at step one.

## What I saw first

A JS terminal game in the browser. Choices fetched from `/api/options`, choices POSTed back to `/api/monitor`. The game funnels you through a fixed flow.

## What I tried that did not work

Read the client code first looking for the actual flow logic. The client just renders whatever the API says, so manipulating the client buys nothing. Tried jumping past steps on the POST endpoint. Server rejected because it actually validates against the options the API returns.

## What worked

`GET /api/options` returns the full set of `allPossibleCommands`, including a key called `secret` that holds one strange string. The server-side check is `options[currentStep].includes(cmd) || options['secret'].includes(cmd)`. So anything in the `secret` list is accepted at any step, including the first.

```bash
curl "$HOST/api/options" | jq '.secret'
# ["Blip-blop, in a pickle with a hiccup! Shmiggity-shmack"]

curl -X POST "$HOST/api/monitor" -H 'Content-Type: application/json' \
  -d '{"command":"Blip-blop, in a pickle with a hiccup! Shmiggity-shmack"}'
```

## Flag

`HTB{D3v3l0p3r_t00l5_4r3_b35t_wh4t_y0u_Th1nk??!_ac6fc38bc6512eabe1145ee916767fb3}`

## What this taught me

The server validated against the same options object it shipped to the client. Always pull every static JS file and every API endpoint they reference before touching the UI. The bug is rarely in what the UI shows. It is usually in what the UI does not show that the server already trusted.
