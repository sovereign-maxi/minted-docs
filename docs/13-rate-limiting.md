# Rate Limiting & PoW

## Protocol requirements

A conforming mint MUST rate-limit requests per anonymous per-request identifier so that no single caller can exhaust its resources or brute-force operations. Abuse signals — notably repeated double-spend attempts — MUST escalate cost for the offending caller.

- **Anonymous request identifiers** — Rate-limit keys MUST NOT be derived from IP address or user agent. There is no path from the identifier to a real-world identity.
- **Per-class budgets** — Operations are classed by cost (cheap / medium / expensive) and each class carries its own budget.
- **Cost escalation on abuse** — Repeated abuse from the same identifier raises the per-request cost for a cooldown window. The mechanism (proof-of-work, longer delays, hard denial) is an implementation choice.
- **Fail-closed** — A request that fails rate limiting or the abuse gate MUST NOT reach the business-logic controllers.

## Reference implementation

The reference implementation delegates all rate-limit storage and abuse-cost issuance to the [Seer](./14-published-libraries.md) library.

- **Circuit hash** — Anonymous per-request identifier Seer extracts from the request envelope. Never the IP address. Never the user agent.
- **Operation class** — Requests are classified cheap, medium, or expensive. The class drives the rate budget and the PoW difficulty.
- **Rate budget** — Per-class, per-circuit-hash request count over a sliding window. Over-budget callers are denied at the plug before any business logic runs.
- **Proof of work** — Expensive operations require a client-solved PoW challenge (leading-zero-bits hashcash-style). Difficulty is baseline plus an escalation multiplier.
- **Abuse escalation** — Abuse signals — notably repeated double-spend attempts — raise the offending circuit hash's difficulty multiplier for a cooldown window.
- **Fail-closed in production** — The gate is enabled in production by default. Tests opt out explicitly.
