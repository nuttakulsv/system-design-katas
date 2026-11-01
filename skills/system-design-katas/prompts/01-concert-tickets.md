# Prompt 01: Concert tickets

## Prompt

"A huge concert goes on sale at 10:00 tomorrow. Design the ticketing system."

## Hidden requirements

- 50,000 seats, assigned seating, across 12 price zones. Max 4 tickets per account.
- About 1 million people will be waiting at 10:00. Sales usually finish in under 10 minutes.
- Fairness is the top complaint from past sales: bots and scalpers bought most seats, and the site crashed.
- A seat is held for the buyer for 10 minutes while they pay. Payment is via card or PromptPay QR; QR payments confirm through a bank callback, sometimes minutes later.
- Never sell the same seat twice. Overselling means a public scandal and refunds.
- Mobile app and web. One region (Thailand) is fine.

## Numbers

- Peak: ~1M concurrent users at 10:00:00; 200k requests per second for the first minute if nothing throttles it.
- 50k seats, ~15k orders.
- Seat map reads are heavy; seat holds are the contended writes.
- Target: the system stays up, and every user gets a clear answer (in the queue, holding seats, sold out).

## A strong design covers

- **A virtual waiting room** in front of everything: admit users in batches at a rate the inventory system can handle. Randomised position for everyone who arrived before 10:00 (fairness), FIFO afterwards. Signed admission tokens.
- **Inventory as the single source of truth** for seat state: available, held (with expiry), sold. Atomic hold with a conditional write (for example `UPDATE seats SET state='held', holder=?, expires_at=? WHERE id IN (...) AND state='available'`), or per-zone partitioning in Redis with Lua or a single-writer actor per section.
- **Hold expiry** that reliably returns seats (TTL plus a sweeper), and **idempotent payment confirmation** that handles late callbacks: a callback for an expired hold must not double-sell.
- **Bot defence:** verified accounts or phone numbers, per-account limits, rate limits, challenges at admission, not on every request.
- **Static and cacheable everything else:** seat maps served from CDN with availability overlaid in coarse buckets.
- **Load testing and a dress rehearsal**, plus a kill switch to pause admission.

## Probes

- "Two people click the same seat at the same millisecond. Walk me through it."
- "The payment callback arrives 12 minutes after the hold expired and the seat was resold. What happens?"
- "How do you keep the waiting room itself from falling over at 10:00:00?"
- "How does a user know their queue position is real?"

## Curveballs

- "Halfway through, the payment provider slows down: confirmations take 3 minutes."
- "Marketing wants to add a presale for credit card holders an hour earlier, with a separate allocation."
- "We found 30% of the queue is bots. What changes?"

## Level notes

- **Senior:** correct atomic holds, expiry, idempotent confirmation, a queue in front.
- **Staff:** drives straight to contention and fairness as the hard parts; capacity math for admission rate; failure handling of payment callbacks; how to rehearse and operate the event (dashboards, pause switch, comms).
- **Principal:** questions the format (lottery or ballot instead of first-come for fairness), discusses business trade-offs (scalpers versus revenue, presale policy), and the organisational side: support, refunds, PR if it goes wrong.

## Pitfalls

- Starting with microservices diagrams before handling contention.
- Distributed locks per seat without thinking about lock expiry.
- Ignoring late payment callbacks.
- Autoscaling as the answer to a 10:00:00 spike.
