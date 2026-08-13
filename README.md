# MINTED Protocol

Reference specification for the MINTED Chaumian eCash mint.

MINTED specifies a Chaumian blind-signature eCash mint on Bitcoin's Lightning Network, denominated in sats. Tokens are bearer instruments; a conforming mint signs blinded messages it cannot see and later verifies unblinded signatures it cannot trace back to the signing request. The reference implementation is Tor-native by default, uses a single-node architecture with WAL-backed durability, and encrypts keysets at rest. Reserves are Lightning balances tracked against outstanding token liability, with signed proofs published on a cadence.

Source: [github.com/sovereign-maxi/minted](https://github.com/sovereign-maxi/minted)

## Contents

- [Overview](overview.md) — architecture at a glance, fee schedule, transport, runtime
- [Identity & Tokens](identity-and-tokens.md) — no accounts, bearer-instrument model, client-side storage
- [Cashu NUT Compliance](cashu-nut-compliance.md) — which NUTs the reference implementation supports
- [BDHKE Protocol](bdhke.md) — the blind-signature scheme, step by step
- [Fees](fees.md) — deposit, withdrawal, swap, splice

### Flows

- [Deposit](deposit-flow.md) — Bitcoin → tokens (NUT-04)
- [Withdrawal](withdrawal-flow.md) — tokens → Bitcoin (NUT-05)
- [Swap](swap-flow.md) — tokens → tokens (NUT-03)

### Data Model

- [Token Format](token-format.md) — wire encoding, denominations
- [Keysets & Rotation](keysets.md) — active / retired / expired lifecycle
- [Spent Set](spent-set.md) — double-spend defence

### Verifiability

- [Proof of Reserves](proof-of-reserves.md) — HTTP + Nostr, verification code
- [Nostr Event Schemas](nostr-schemas.md) — kind 30078 reserves attestations

### Operational

- [Rate Limiting & PoW](rate-limiting.md) — the request-gate
- [Published Libraries](published-libraries.md) — the crypto + storage + Lightning primitives

### Reference

- [API](api.md) — public HTTP endpoints
- [Threat Model](threat-model.md) — what the protocol protects against and what it doesn't
