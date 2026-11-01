# Prompt 04: Notifications

## Prompt

"Design a notification service that other teams in the company can use to reach users."

## Hidden requirements

- Channels: push (iOS and Android), SMS, email, LINE, and in-app inbox.
- Mixed priorities: one-time passwords must arrive within seconds; order updates within a minute; marketing can wait hours and must respect quiet hours (no marketing pushes 21:00–08:00 local time).
- Users have per-channel and per-category preferences, and legal opt-outs for marketing.
- No duplicates: last year a bug sent a "your order shipped" push 14 times to 200k users.
- SMS costs real money; the finance team wants cost per team.

## Numbers

- 30M users. 200M notifications a day; a marketing campaign can enqueue 20M at once.
- OTP peak: 2,000 per second.
- Providers have their own rate limits (for example, SMS provider 500 per second).

## A strong design covers

- **A single API** with idempotency keys from callers, priority, category and template.
- **Separate queues per priority** so a 20M campaign never delays OTPs; per-provider workers with rate limiting and backoff.
- **Preference and policy check** (opt-outs, quiet hours by user time zone, frequency caps) before sending, not in each calling team.
- **Deduplication** by idempotency key within a window, stored durably.
- **Delivery tracking:** provider callbacks, retries with limits, fallback channel for OTP (push failed, then SMS).
- **Templates and localisation** (Thai and English), versioned.
- **Cost attribution** per calling team; budgets and alerts.

## Probes

- "A campaign of 20M is queued at 20:55. What happens at 21:00?"
- "The push provider is down. What happens to OTPs?"
- "How exactly do you prevent the 14-duplicates incident?"

## Curveballs

- "Marketing wants to send to 'everyone who hasn't opened the app in 30 days', computed at send time."
- "A provider starts returning success but not delivering."

## Level notes

- **Senior:** queues, workers, retries, preferences, idempotency.
- **Staff:** priority isolation, rate limits per provider, dedupe guarantees, a policy layer, an operational view per team; makes the service safe for other teams to use.
- **Principal:** platform governance (who may send what), cost model and chargeback, user trust (notification fatigue as a product risk), and the migration path off each team's homegrown senders.

## Pitfalls

- One queue for everything.
- Leaving preferences to every calling team.
- Retrying forever.
