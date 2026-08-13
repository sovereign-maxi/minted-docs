# Spent Set & Double-Spend

The **spent set** is the sole defence against double-spending. Every redeemed token's secret is `SHA256`-hashed and stored in a durable spent-set that survives crashes. Presenting a proof whose hash is already in the set fails redemption immediately, and a conforming mint MUST raise the offending caller's cost via whatever rate-limit / abuse-cost mechanism it uses (see [Rate Limiting & PoW](./13-rate-limiting.md)).

- **Hashes only** — The spent set stores `SHA256(secret)` — not the secret itself, no amount, no timestamp, nothing that links a token to a person or transaction.
- **Y-index for NUT-07** — A secondary index keyed by `Y = hash_to_curve(secret)` answers `/v1/check` queries. Y-index writes are ordered *after* the primary hash is durably persisted, so the index is never ahead of the truth.
- **Reserve / commit / release** — Melts and swaps use a pending reservation table. `verify_and_reserve` blocks re-spend for the duration of the operation; `commit_reservation` promotes to the durable spent set on success; `release_reservation` drops the pending entry on definitive failure.
- **Fail-closed on ambiguity** — On payment-outcome ambiguity, the reservation is *held*, never released speculatively. Fail-closed means the tokens stay locked until the payment resolves — a background resolver eventually breaks the tie by reading the Lightning node's authoritative state.
