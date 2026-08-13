# Threat Model

## What the protocol protects against

- **Transaction graph analysis** — Blind signatures sever the link between deposit and withdrawal at the moment of issuance. There is no transaction graph because the data connecting the two events is never created. Chain surveillance produces nothing.
- **Timing correlation** — Tokens are bearer instruments. Deposit and withdrawal timing is user-controlled, not protocol-observable. Redeem later, in different amounts, and an observer watching both Lightning edges has nothing to align.
- **Amount correlation** — Fixed power-of-2 denominations. A 1,000-sat balance produces the same denomination structure as every other 1,000-sat balance. Amounts are not fingerprints.
- **IP identification** — Tor hidden service throughout. Your IP is never received by the mint's infrastructure. Not logged and deleted — never transmitted.
- **Account freezing** — No accounts. Tokens are bearer instruments in the client's browser. The mint cannot identify which tokens belong to which session.
- **Records requests** — No user database, no accounts, no balances, no transaction history. Operational logs contain no user-identifiable data — secrets and payment details are redacted before writing.
- **Double spending** — Every proof is checked against the spent set. A proof already spent is rejected atomically. No replay window; commit and spend are one operation.
- **Single-server cold seizure** — Data volume is LUKS-encrypted (AES-256), no swap partition, keyset private keys encrypted at rest under a boot-time environment key. A cold seizure yields encrypted noise.

## What the protocol does not protect against

- **Compromised endpoint** — If your device is compromised (keylogger, malware, screen capture), an attacker can read tokens straight out of your browser's localStorage. Universal limitation across every wallet that stores value on the client. Use a clean device, ideally a dedicated Tor Browser session.
- **Lost tokens** — No recovery by design. Enabling recovery would mean tying the account to something else the architecture doesn't collect. Lose the tokens without a backup and the value goes with them, the way physical cash does.
- **Dishonest mint** — Blind signatures protect privacy, not custody. The mint could sign tokens without receiving deposits (unbacked liabilities) or refuse to pay withdrawals. This is exactly why [proof of reserves](./proof-of-reserves.md) is publishable and verifiable. Verify. Withdraw if the numbers don't add up.
- **Lightning liquidity** — Withdrawals require the mint's Lightning node to have sufficient outbound channel capacity. If liquidity is temporarily constrained, a specific withdrawal may fail; the tokens remain valid and can be spent later or at a smaller amount.
- **Mint downtime** — If the mint is offline you cannot deposit, withdraw, or swap until it returns. Tokens remain valid; the mint can be rebuilt from seed and backup on new infrastructure with the same keyset and `.onion` address.
- **Tor network attacks** — A global passive adversary with visibility into your Tor ingress and the mint's Tor ingress could theoretically correlate traffic. Known limitation of Tor hidden services, not specific to this protocol. Sufficient for most threat models.
