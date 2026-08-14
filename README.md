# MINTED - Protocol

Reference specification for the MINTED Chaumian eCash mint.

MINTED specifies a Chaumian blind-signature eCash mint on Bitcoin's Lightning Network, denominated in sats. Tokens are bearer instruments; a conforming mint signs blinded messages it cannot see and later verifies unblinded signatures it cannot trace back to the signing request. The reference implementation is Tor-native by default, uses a single-node architecture with WAL-backed durability, and encrypts keysets at rest. Reserves are Lightning balances tracked against outstanding token liability, with signed proofs published on a cadence.

Source: [github.com/sovereign-maxi/minted](https://github.com/sovereign-maxi/minted)

## Contents

- [Overview](docs/00-overview.md) — architecture at a glance, fee schedule, transport, runtime
- [Identity & Tokens](docs/01-identity-and-tokens.md) — no accounts, bearer-instrument model, client-side storage
- [Cashu NUT Compliance](docs/02-cashu-nut-compliance.md) — which NUTs the reference implementation supports
- [BDHKE Protocol](docs/03-bdhke.md) — the blind-signature scheme, step by step
- [Fees](docs/04-fees.md) — deposit, withdrawal, swap, splice

### Flows

- [Deposit](docs/05-deposit-flow.md) — Bitcoin → tokens (NUT-04)
- [Withdrawal](docs/06-withdrawal-flow.md) — tokens → Bitcoin (NUT-05)
- [Swap](docs/07-swap-flow.md) — tokens → tokens (NUT-03)

### Data Model

- [Token Format](docs/08-token-format.md) — wire encoding, denominations
- [Keysets & Rotation](docs/09-keysets.md) — active / retired / expired lifecycle
- [Spent Set](docs/10-spent-set.md) — double-spend defence

### Verifiability

- [Proof of Reserves](docs/11-proof-of-reserves.md) — HTTP + Nostr, verification code
- [Nostr Event Schemas](docs/12-nostr-schemas.md) — kind 30078 reserves attestations

### Operational

- [Rate Limiting & PoW](docs/13-rate-limiting.md) — the request-gate
- [Published Libraries](docs/14-published-libraries.md) — the crypto + storage + Lightning primitives

### Reference

- [API](docs/15-api.md) — public HTTP endpoints
- [Threat Model](docs/16-threat-model.md) — what the protocol protects against and what it doesn't
