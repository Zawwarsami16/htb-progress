# Web

The first thing I do on a web box is read the source if I can get it. The second thing I do is enumerate the routes if I cannot. There is no third generic step, because web is the broadest category and the right move past the first two depends on what the source says.

## The first pass

If the challenge ships source code, that is the only enumeration step I need. Source review wins over fuzzing every time the source is available. The source tells me which endpoints exist, what they accept, what they call, and which dependencies they trust. Fuzzing tells me which endpoints respond. The first is a superset of the second.

If the challenge does not ship source, the first pass is content discovery against the routes the home page references. Pull every static JS bundle. Read it. The bundle usually names every API endpoint the UI ever calls, plus the API endpoints it stopped calling but did not delete from the code. Those are the interesting ones.

## What I check before anything else

- Static assets for inline API endpoints, hardcoded keys, or commented-out code that names dev routes.
- The framework's signature default routes (`/api/health`, `/debug`, `/__webpack_hmr`, `/admin`, `/swagger`, `/graphql`).
- The headers on a baseline request, especially anything Cache, CDN, or proxy-related. A reverse-proxy in the path opens a different attack surface than a single-server stack.
- The Set-Cookie attributes. Insecure `JWT` cookies with `httpOnly` missing are an XSS-to-token-theft chain.
- The Content-Security-Policy. Missing or permissive CSP changes which XSS payloads survive.

## What I avoid

Brute-force fuzzing on a tiny challenge. The 30-minute timer evaporates if you give half of it to `ffuf`. Source review or focused endpoint exploration first; fuzzing only when the surface is genuinely opaque.

Assuming the validator does what its name says. Validators are written by humans. The validator that clamps `choice` to `0..5` may not clamp `choice` to `< commands.length`. Read the validator code, not the variable name.

Following the URL the UI uses when the JS bundle names a different one. The UI uses the public route. The undocumented route is the one with the bug.

## When the obvious channels are closed

If there is no error message, no reflection, no status-code split, the box is asking me to find a covert channel. Timing is the most reliable one. A request that takes 200 ms when the guessed character is wrong and 2.2 s when right is a confirmed oracle even if the response body is byte-identical. Discipline matters here: stop if the baseline request is already over 3 s — the noise floor makes every signal ambiguous. Use ASCII codepoints for the comparison column instead of literal characters, so a quote or a backslash in the value does not break the payload.

Caches are the second covert channel. A page that responds in 8 ms when its key matches a previous request and 400 ms when it does not just leaked the key. Vary headers, cookies, query keys — anything the cache hashes — are tells. The `Age` header is a sometimes-leak; an `X-Cache: HIT` is the gift wrapping.

The third is the path through a proxy. A reverse proxy that drops headers, normalizes path segments, or terminates TLS rewrites the request twice — once for itself, once for the origin. Find a header the proxy strips that the origin still trusts. Find a path the proxy normalizes that the origin parses differently. The bug lives where the two pieces of software disagree about what they just saw.

## What I keep handy

- `curl` with `--data-urlencode` for URL-encoded body parameters that may contain `&` or `=`.
- `jq` for slicing API responses without writing parsers.
- A scratch script that calls `Class.forName` chains for Velocity / SpEL / Freemarker SSTI without re-deriving the syntax each time.
- A short list of XXE payloads with and without an XML declaration, because `lxml` rejects declarations when the input is a string.
- `gron` for converting deep JSON into greppable lines.
