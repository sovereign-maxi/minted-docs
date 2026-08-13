# Rate Limiting & PoW

Every request runs through a single gate that extracts an anonymous per-request identifier — the *circuit hash* — classifies the operation, enforces a per-class rate budget, and issues a proof-of-work challenge for expensive or recently-exhausted callers. All rate-limit storage and PoW issuance is delegated to the [Seer](./published-libraries.md) library.

- **Circuit hash** — Anonymous per-request identifier extracted by Seer from the request envelope. Never the IP address. Never the user agent. There is no path from a circuit hash to an identity.
- **Operation class** — Requests are classified cheap, medium, or expensive. The class drives the rate budget and the PoW difficulty.
- **Rate budget** — Per-class, per-circuit-hash request count over a sliding window. Over-budget callers are denied at the plug before any business logic runs.
- **Proof of work** — Expensive operations require a client-solved PoW challenge (leading-zero-bits hashcash-style). Difficulty is baseline plus an escalation multiplier.
- **Abuse escalation** — Abuse signals — notably repeated double-spend attempts — raise the offending circuit hash's difficulty multiplier for a cooldown window.
- **Fail-closed in production** — The gate is enabled in production by default. Tests opt out explicitly. A request that fails rate limiting or PoW never reaches a controller.
