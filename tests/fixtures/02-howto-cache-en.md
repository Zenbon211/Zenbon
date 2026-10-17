# How to Cache Static Assets Without Breaking Deploys

Caching is the cheapest performance win you have, and the easiest way to ship a broken site. The rules below are what I apply on every project.

**Rule 1: fingerprint the filename, then cache forever.** For build output such as `app.3f9c1a.js`, send `Cache-Control: public, max-age=31536000, immutable`. Because the hash changes whenever the content changes, a one-year TTL is safe: a new deploy produces a new URL, so no client can hold a stale reference.

**Rule 2: never long-cache HTML.** The HTML document is the thing that points at the fingerprinted files. If you cache it for an hour, users keep loading yesterday's `index.html`, which references assets you may have already deleted. Send `Cache-Control: no-cache` for HTML. Note that `no-cache` does not mean "do not cache": it means "store it, but revalidate before every reuse".

**Rule 3: do not confuse `no-cache` with `no-store`.** `no-store` forbids writing the response to disk at all. Use it only for responses that must never persist, such as a page containing a session token. Applying `no-store` to HTML throws away the revalidation win and usually makes the site slower.

**Rule 4: add ETags so revalidation is cheap.** With a strong ETag, the browser sends `If-None-Match`, and the server answers `304 Not Modified` with no body. The round trip still costs latency, but you save the payload, which matters most for large HTML or JSON responses.

**Rule 5: let stale content cover the tail.** `stale-while-revalidate=60` allows a CDN to serve the stale copy for up to 60 seconds while it refreshes in the background. Users see no wait, and the next request gets fresh content. This is the wrong choice for prices and inventory, where a 60-second-old number can cost money.

**Rule 6: keep personalized responses out of shared caches.** A response that depends on the logged-in user must carry `Cache-Control: private` or `no-store`. If a shared CDN caches it, one user's dashboard can be served to another. When in doubt, mark it private.

Finally, remember that a CDN purge is not a cache strategy. If you find yourself purging after every deploy, the TTLs above are wrong.

```http
Cache-Control: public, max-age=31536000, immutable
ETag: "3f9c1a-9b2"
```
