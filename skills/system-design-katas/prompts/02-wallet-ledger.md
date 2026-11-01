# Prompt 02: Wallet ledger

## Prompt

"We're building an e-wallet. Users top up with PromptPay, pay merchants, and transfer to friends. Design the ledger and the top-up flow."

## Hidden requirements

- Money must never be created or lost. Every balance must be explainable from history. Regulators and auditors will ask.
- Top-up: we show a PromptPay QR; the bank notifies us by webhook when paid. Webhooks can be late, duplicated, or missing (then we reconcile against the bank's daily statement file).
- Payments to merchants and transfers must be instant from the user's point of view.
- Balances can't go negative. Refunds and reversals happen.
- Users can view their history, going back years.

## Numbers

- 5M users, 1M daily active.
- ~2M transactions a day, peaks of 3,000 per second (lunchtime, salary day).
- History reads dominate the app; balance reads on every screen.
- Latency target for payments: p99 under 300 ms.

## A strong design covers

- **Double-entry ledger:** every transaction is a set of entries that sum to zero across accounts (user wallets, a bank clearing account, merchant accounts, fee accounts). Entries are append-only; corrections are new entries.
- **Balance** as a derived value, kept in a balance table updated in the same database transaction as the entries, with a constraint or check that prevents negatives (`balance >= amount` conditional update).
- **Idempotency keys** on every money-moving API and every webhook (the bank's transaction reference), stored with a unique constraint.
- **Top-up state machine:** created → QR shown → paid (webhook) → credited; expired; late payment after expiry handled (credit anyway or auto-refund, a policy decision).
- **Reconciliation:** daily match of the bank statement against our clearing account; unmatched items go to an exceptions queue with a human workflow.
- **Hot accounts:** merchant accounts receiving thousands of payments a second need sharding of the account (sub-accounts) or batched settlement.
- **Audit and history:** an immutable log; history served from a read replica or a separate store.

## Probes

- "The webhook arrives twice. Then a third time, after a deploy. What happens?"
- "How do you prove to an auditor that the sum of all wallet balances matches the money in our bank account?"
- "A merchant receives 2,000 payments a second. Where's the contention, and what do you do?"
- "A user transfers to a friend and our service crashes between debiting and crediting."

## Curveballs

- "The bank's webhook system is down for 3 hours. Users are angry their top-ups haven't arrived."
- "We need to support multiple currencies for travellers."
- "Finance wants real-time fraud checks before payments, adding 150 ms."

## Level notes

- **Senior:** double-entry, idempotency, atomic balance updates, a clear top-up state machine.
- **Staff:** reconciliation as a first-class system, hot-account contention, failure modes across the bank boundary, and operability (exceptions queue, runbooks, data correction without mutating history).
- **Principal:** regulatory and audit framing, build-versus-buy for the ledger core, how the ledger becomes a platform for future products, and the organisational ownership between engineering and finance.

## Pitfalls

- A single `balance` column updated with no entries behind it.
- Treating webhooks as exactly-once.
- Distributed transactions across services for every transfer.
- No story for the day the bank file disagrees with you.
