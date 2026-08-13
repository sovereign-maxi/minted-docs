# Threat Model

## What the protocol protects against

- **Transaction graph analysis** — Blind signatures sever the link between deposit and withdrawal at the moment of issuance. There is no transaction graph because the data connecting the two events is never created. Chain surveillance produces nothing.
- **Timing correlation** — Tokens are bearer instruments. Deposit and withdrawal timing is user-controlled, not protocol-observable. Redeem later, in different amounts, and an observer watching both Lightning edges has nothing to align.
- **Amount correlation** — Fixed power-of-2 denominations. A 1,000-sat balance produces the same denomination structure as every other 1,000-sat balance. Amounts are not fingerprints.
- **Account freezing** — No accounts. Tokens are bearer instruments in the client's browser. A mint cannot identify which tokens belong to which session.
- **Records requests** — No user database, no accounts, no balances, no transaction history at the protocol layer. There is nothing to compel a mint to hand over about a specific user because no such association exists.
- **Double spending** — Every proof is checked against the spent set. A proof already spent is rejected atomically. No replay window; commit and spend are one operation.
- **Dishonest mint (partial)** — Blind signatures protect unlinkability at the mint. Reserve proofs published via the HTTP + Nostr paths provide an ongoing signal that a mint is fully backed. Verification is the reader's; the protocol just makes the data available.

## What the protocol does not protect against

- **Compromised endpoint** — If your device is compromised (keylogger, malware, screen capture), an attacker can read tokens straight out of your browser's localStorage. Universal limitation across every wallet that stores value on the client. Use a clean device, ideally a dedicated Tor Browser session.
- **Lost tokens** — No recovery by design. Enabling recovery would mean tying the account to something else the architecture doesn't collect. Lose the tokens without a backup and the value goes with them, the way physical cash does.
- **Dishonest mint (fully)** — Blind signatures protect privacy, not custody. A mint could sign tokens without receiving deposits (unbacked liabilities) or refuse to pay withdrawals. This is exactly why [proof of reserves](./proof-of-reserves.md) is publishable and verifiable. Verify. Withdraw if the numbers don't add up.
- **Lightning liquidity** — Withdrawals require the paying mint's Lightning node to have sufficient outbound channel capacity. If liquidity is temporarily constrained, a specific withdrawal may fail; the tokens remain valid and can be spent later or at a smaller amount.
- **Mint downtime** — If a mint is offline you cannot deposit, withdraw, or swap against it until it returns. Tokens remain valid; a mint can be rebuilt from seed and backup on new infrastructure with the same keyset.

## Reference implementation deployment posture

The reference implementation is designed to run as a Tor hidden service on a single Linux host with an encrypted data volume. Under that deployment posture, additional threats are addressed:

- **IP identification** — Tor hidden service throughout. User IPs are never received by the mint's infrastructure. Not logged and deleted — never transmitted.
- **Single-server cold seizure** — Data volume is LUKS-encrypted (AES-256), no swap partition, keyset private keys additionally encrypted at rest under a boot-time environment key. A cold seizure yields encrypted noise.
- **Operational log leakage** — Operational logs redact secrets and payment details before writing. Log rotation is aggressive.

These are choices of the reference implementation's Linux/Tor deployment shape. A mint implementing the protocol on different infrastructure would need to solve the equivalent problems in its own way.

- **Tor network attacks** — A global passive adversary with visibility into a Tor user's ingress and a Tor-hosted mint's ingress could theoretically correlate traffic. Known limitation of Tor hidden services, not specific to any mint. Sufficient for most threat models.
