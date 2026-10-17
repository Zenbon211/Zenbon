<!--
输入：tests/fixtures/02-howto-cache-en.md
技能版本：v3（最终）
结论：通过 —— 7 张，落在 5–7；英文进英文出，结构标签与来源行已翻译；末尾代码块未被当作知识点
-->

# Knowledge Cards: Cache Static Assets Without Breaking Deploys

Source: 02-howto-cache-en.md | 7 cards

## 1. Fingerprint the filename, then cache forever

**Core knowledge**: Fingerprint the filename with a content hash, then cache that file for a year as immutable (`Cache-Control: public, max-age=31536000, immutable`).

**Explanation**: The hash in the name is a fingerprint of the content, so any content change produces a different filename and a new deploy publishes new URLs instead of overwriting old ones. The one-year TTL therefore only ever applies to bytes that have not changed, and no client can hold a stale reference.

**Example**: The article's build output `app.3f9c1a.js` is sent with `Cache-Control: public, max-age=31536000, immutable`.

## 2. Never long-cache the HTML document

**Core knowledge**: HTML must stay short-lived: send `Cache-Control: no-cache`, which stores the response but revalidates it before every reuse.

**Explanation**: The HTML document is the thing that points at the fingerprinted files, so it is the response that has to stay current. `no-cache` does not mean "do not cache" — the response is still stored, it is just checked before each reuse.

**Example**: Cache `index.html` for an hour and users keep loading yesterday's document, which references assets you may already have deleted.

## 3. Add ETags so revalidation is cheap

**Core knowledge**: With a strong ETag the browser sends `If-None-Match` and the server answers `304 Not Modified` with no body.

**Explanation**: The round trip still costs latency, but the payload is not sent again — which matters most for large HTML or JSON responses.

**Self-test**: Revalidation still costs a round trip, so what does the ETag actually save?

## 4. Let stale content cover the tail

**Core knowledge**: `stale-while-revalidate=60` lets a CDN serve the stale copy for up to 60 seconds while it refreshes in the background.

**Explanation**: Users see no wait, and the next request gets fresh content.

**Example**: Prices and inventory — a copy that is 60 seconds old can cost money, so the stale window is the wrong choice there.

## 5. Keep personalized responses out of shared caches

**Core knowledge**: A response that depends on the logged-in user must carry `Cache-Control: private` or `no-store`, and when in doubt it should be marked private.

**Explanation**: A shared cache serves every user, so a personalized response stored in it can be handed out to someone else.

**Example**: One user's dashboard served to another user out of a shared CDN cache.

## 6. Reserve `no-store` for responses that must never persist

**Core knowledge**: `no-store` forbids writing the response to disk at all, so it is only for responses that must never persist, such as a page containing a session token.

**Explanation**: Nothing is stored, so there is nothing to revalidate. Applying `no-store` to HTML throws away the revalidation win and usually makes the site slower.

**Self-test**: Why is `no-store` a poor choice for the HTML document?

## 7. A CDN purge is not a cache strategy

**Core knowledge**: If you find yourself purging the CDN after every deploy, the TTLs are wrong.

**Explanation**: Purging is not what keeps content fresh; the cache headers are. A purge after every deploy is a symptom that the TTLs above are misconfigured rather than a strategy.

**Self-test**: You purge the CDN after every deploy. What does the author conclude from that?
