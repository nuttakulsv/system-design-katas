# Prompt 06: LLM gateway

## Prompt

"Twenty teams in our company call LLM APIs directly. Design an internal gateway they should all go through instead."

## Hidden requirements

- Several providers and models; teams want to switch models without code changes, and fall back automatically when a provider has an outage.
- Spending is out of control: last month one team's runaway agent loop cost $40k in a weekend. Finance wants budgets per team and per project, enforced.
- Responses stream (server-sent events) and that has to keep working.
- Some prompts contain customer personal data; security wants it redacted or blocked for certain models, and every request logged for audit with retention rules.
- Latency overhead from the gateway must be small: under 20 ms added before the first token.

## Numbers

- 3,000 requests per second at peak; prompts from 1k to 200k tokens; long-running streams up to several minutes.
- Provider rate limits are per API key and per model, in tokens per minute.

## A strong design covers

- **Thin streaming proxy** with an OpenAI- or Anthropic-compatible API, so teams change a base URL, not their code.
- **Routing:** logical model names mapped to providers by config; health checks and fallback on errors and timeouts; awareness that fallbacks can change output quality.
- **Token-based rate limiting and budgets:** estimate input tokens before sending, count actual usage from the response, reconcile; per-team and per-key limits; hard stops and alerts; kill switch for runaway loops.
- **Provider key management:** pooled keys, rate limit headroom per key, rotation.
- **Caching:** use providers' prompt caching; exact-match response cache only where safe (deterministic, non-personal).
- **Data controls:** PII detection and redaction or blocking per policy and model; audit logs with retention; no logging of secrets.
- **Observability:** per-team cost, latency, error rates and token usage dashboards.

## Probes

- "How do you enforce a budget when you only know the cost after the stream ends?"
- "Provider A goes down mid-stream. What does the client see?"
- "Where does the 20 ms go, and how do you keep the gateway from becoming the outage?"

## Curveballs

- "A team's agent makes 500 requests a minute, each with 150k tokens of context. What does the gateway do?"
- "Security says prompts must never leave the country for one product line."

## Level notes

- **Senior:** a working proxy with routing, auth, rate limits and logging.
- **Staff:** token-based budgets with streaming reconciliation, fallback semantics, keeping the gateway off the critical path for failure, and a migration plan for twenty teams.
- **Principal:** governance and policy (model approval, data classes), cost strategy across providers, build-versus-buy, and how this becomes the company's AI platform.

## Pitfalls

- Request-count rate limits instead of tokens.
- Buffering whole responses and breaking streaming.
- A single point of failure with no bypass.
