# Varnish cache key with no URL

**Trigger:** reverse-proxy VCL with `hash_data(req.http.X)` and no fallback to `req.url`.

```vcl
sub vcl_hash {
    hash_data(req.http.CacheKey);
    return (lookup);
}
```

This is a universal cache-confusion bug. The cache key depends entirely on the value of one request header, with no contribution from the URL or the Host. Any cacheable backend response gets served for any future request whose `CacheKey` value matches, regardless of what URL or origin it claims to be.

Chained with a backend that:

1. Reflects user input into a response header without sanitization, and
2. Has a route that does not auto-set `Content-Type`,

you have full response splitting that survives across requests. Inject `Content-Type: text/html` and a body into the response. Set the cache TTL. Any user who hits the same `CacheKey` value now gets your HTML.

Two practical gotchas worth keeping:

The default `vcl_recv` in Varnish passes through any request with a Cookie or Authorization header without consulting the cache. To make a session-bound response cacheable, your exfil fetch needs `credentials: 'omit'`.

If the backend's route uses a serialization helper (`res.json()` in Zig httpz, `JsonResponse` in Flask) that sets its own `Content-Type`, your injected `text/html` will lose. Pivot to a route that writes raw bytes without setting `Content-Type` itself.

**Seen in:** [OmniWatch](../challenges/web/omniwatch.md).
