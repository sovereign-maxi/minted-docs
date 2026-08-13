# Overview

MINTED specifies a Chaumian blind-signature eCash mint on Bitcoin's Lightning Network, denominated in sats. Tokens are bearer instruments; a conforming mint signs blinded messages it cannot see and later verifies unblinded signatures it cannot trace back to the signing request. The reference implementation is Tor-native by default, uses a single-node architecture with WAL-backed durability, and encrypts keysets at rest. Reserves are Lightning balances tracked against outstanding token liability, with signed proofs published on a cadence.

- **Deposit fee** — 0.75% (7,500 ppm), collected upfront
- **Withdrawal fee** — 0% mint fee; user pays only the Lightning routing fee, passed through
- **Swap fee** — 0 — denomination splitting and privacy refresh are free
- **Deposit bounds** — Practical minimum 1,000 sats (Lightning routing floor); maximum 33,000,000 sats per quote (configurable)
- **Custody** — Lightning-only. Balance held in Lightning channels; single signer for BDHKE (secp256k1 keyset)
- **Transport** — Tor hidden service (v3 onion), including outbound Nostr publishing and price feeds via a Tor HTTP CONNECT tunnel
- **Identity** — None. Tokens are the account. Browser holds them in localStorage; the mint has no user record to link them to
- **Runtime** — Elixir/OTP on the BEAM VM with Rust NIFs for the BDHKE hot path. Fault-tolerant supervision trees; WAL-backed state
