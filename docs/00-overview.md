# Overview

MINTED specifies a Chaumian blind-signature eCash mint on Bitcoin's Lightning Network, denominated in sats. Tokens are bearer instruments; a conforming mint signs blinded messages it cannot see and later verifies unblinded signatures it cannot trace back to the signing request. Reserves are Lightning balances tracked against outstanding token liability, with signed proofs published on a cadence.

## Protocol properties

- **Withdrawal fee** — 0% at the mint layer; users pay only the Lightning routing fee, passed through.
- **Swap fee** — 0. Denomination splitting and privacy refresh are free.
- **Custody** — Lightning-only. Balance held in Lightning channels. Cashu BDHKE signing uses a secp256k1 keyset; single-signer and threshold-signer deployments are both allowed by the protocol.
- **Identity** — None. Tokens are the account. Clients hold them locally; a conforming mint has no user record to link them to.
- **Reserves** — Held-vs-outstanding ratio published via HTTP + Nostr on a per-deployment cadence.

## Reference implementation

The specific choices this reference implementation makes; a conforming mint is free to make different ones.

- **Deposit fee (default)** — 0.75% (7,500 ppm), collected upfront.
- **Deposit bounds (default)** — Practical minimum 1,000 sats (Lightning routing floor); maximum 33,000,000 sats per quote. Both configurable per deployment.
- **Architecture** — Single-node with WAL-backed durability. Keysets encrypted at rest.
- **Transport** — Tor hidden service (v3 onion), including outbound Nostr publishing and price feeds via a Tor HTTP CONNECT tunnel.
- **Runtime** — Elixir/OTP on the BEAM VM with Rust NIFs for the BDHKE hot path. Fault-tolerant supervision trees; WAL-backed state.
