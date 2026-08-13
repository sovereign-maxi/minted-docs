# Published Libraries

The reference implementation is built from a set of standalone primitives, each in its own repository. Every library is MIT-licensed and independently forkable.

- **[Cashew](https://github.com/sovereign-maxi/cashew)** — Server-side BDHKE primitives on secp256k1, implemented as a Rust NIF. The cryptographic core — every token a mint issues is signed by this code.
- **[Core](https://github.com/sovereign-maxi/core)** — Precision arithmetic and shared conversion constants — sats, msats, and BTC handled consistently everywhere the mint touches numbers.
- **[Firebird](https://github.com/sovereign-maxi/firebird)** — Lightning primitives — client, invoice manager, payment executor, liquidity monitor. Everything the mint uses to talk to a Lightning node.
- **[Locker](https://github.com/sovereign-maxi/locker)** — Storage primitives — write-ahead log, atomic counters, pluggable backends. Keeps the spent-set durable across restarts.
- **[Nutty](https://github.com/sovereign-maxi/nutty)** — Client-side BDHKE compiled to WebAssembly. Runs in the browser wallet so blinding factors never leave the client.
- **[Oracle](https://github.com/sovereign-maxi/oracle)** — Shared price-feed primitives for the display-only BTC/USD figure in the wallet UI.
- **[Seer](https://github.com/sovereign-maxi/seer)** — Anti-sybil primitives — PoW challenges, rate limiting, circuit extraction. Sits between Tor and the mint.
- **[Vault](https://github.com/sovereign-maxi/vault)** — Reserve proof generation, attestation, and deficit detection. Powers the `/v1/reserves` endpoint.

The [MINTED reference implementation](https://github.com/sovereign-maxi/minted) composes these into a working mint. Fork any primitive independently or the whole set together.
