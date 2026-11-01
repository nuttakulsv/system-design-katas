# Prompt 03: Short links that get abused

## Prompt

"Design a URL shortener."

## Hidden requirements

- This one's public and free, which means it's used heavily for phishing and malware. Last month the domain got blocklisted by a major email provider for a week. Abuse is the real problem.
- Click analytics per link (counts, country, referrer), available to the link owner within a minute.
- Custom aliases for paying customers. Links can be edited or disabled.
- Some links go viral: a single link can take 50k clicks per second.

## Numbers

- 100M new links a month; 10B redirects a month.
- Read-to-write ratio about 100:1.
- Redirect latency target: p99 under 50 ms globally.
- Links live forever unless disabled.

## A strong design covers

- **ID generation:** base62 from a counter range per writer, or random IDs with a collision check. Seven characters is plenty. Avoid sequential IDs that let people enumerate links.
- **Redirect path:** CDN or edge cache plus a key-value store; 301 versus 302 (302 if you need analytics and the ability to disable).
- **Analytics** off the redirect path: log the event asynchronously to a stream, aggregate into per-minute counters; hot keys handled by pre-aggregation at the edge.
- **Abuse pipeline:** check new links against Safe Browsing-style lists and internal reputation, rate limits per account and IP, a delayed or interstitial "preview" page for risky or new accounts, re-scanning of existing links (targets change after creation), fast takedown that also purges caches, and an abuse report endpoint.
- **Disabling** that propagates to the edge within seconds.

## Probes

- "A link was clean when created and now points to phishing. How do you find out, and how fast can you kill it?"
- "A celebrity tweets a link. Walk me through the first 60 seconds."
- "How do you stop people enumerating everyone's links?"

## Curveballs

- "An email provider has blocklisted our domain again. What do you do today, and what do you change?"
- "Enterprise customers want links on their own domains."

## Level notes

- **Senior:** solid ID scheme, caching, async analytics, the 301/302 trade-off.
- **Staff:** finds abuse as the real problem through questions, designs the trust and safety pipeline and cache invalidation for takedowns, handles hot keys.
- **Principal:** questions the free tier's economics and reputation risk, proposes policy (verified accounts, paid features), and owns the relationship with blocklist providers.

## Pitfalls

- Spending 20 minutes on base62 math.
- Analytics written synchronously on the redirect path.
- No answer for takedowns that are stuck in CDN caches.
