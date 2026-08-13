# Published Libraries

The primitives the reference implementation runs on are published on a public Gitea instance. Anyone can clone or review the code without an account. Third-party dependencies (Phoenix, Erlang/OTP, CubDB) are not listed here because they are not part of this project.

- **Cashew** — Server-side BDHKE primitives on secp256k1, implemented as a Rust NIF. The cryptographic core — every token the mint issues is signed by this code. [Source (Tor)](http://pwq45mhk2fzyldzycvmi6alcxj7ku3ht2oo4qkngya52dwcpvm57d7ad.onion/Rylatech/cashew)
- **Core** — Precision arithmetic and shared conversion constants — sats, msats, and BTC handled consistently everywhere the mint touches numbers. [Source (Tor)](http://pwq45mhk2fzyldzycvmi6alcxj7ku3ht2oo4qkngya52dwcpvm57d7ad.onion/Rylatech/core)
- **Firebird** — Lightning primitives — client, invoice manager, payment executor, liquidity monitor. Everything the mint uses to talk to a Lightning node. [Source (Tor)](http://pwq45mhk2fzyldzycvmi6alcxj7ku3ht2oo4qkngya52dwcpvm57d7ad.onion/Rylatech/firebird)
- **Locker** — Storage primitives — write-ahead log, atomic counters, pluggable backends. Keeps the spent-set durable across restarts. [Source (Tor)](http://pwq45mhk2fzyldzycvmi6alcxj7ku3ht2oo4qkngya52dwcpvm57d7ad.onion/Rylatech/locker)
- **Nutty** — Client-side BDHKE compiled to WebAssembly. Runs in the browser wallet so blinding factors never leave the client. [Source (Tor)](http://pwq45mhk2fzyldzycvmi6alcxj7ku3ht2oo4qkngya52dwcpvm57d7ad.onion/Rylatech/nutty)
- **Oracle** — Shared price-feed primitives for the display-only BTC/USD figure in the wallet UI. [Source (Tor)](http://pwq45mhk2fzyldzycvmi6alcxj7ku3ht2oo4qkngya52dwcpvm57d7ad.onion/Rylatech/oracle)
- **Seer** — Anti-sybil primitives — PoW challenges, rate limiting, circuit extraction. Sits between Tor and the mint. [Source (Tor)](http://pwq45mhk2fzyldzycvmi6alcxj7ku3ht2oo4qkngya52dwcpvm57d7ad.onion/Rylatech/seer)
- **Vault** — Reserve proof generation, attestation, and deficit detection. Powers the `/v1/reserves` endpoint. [Source (Tor)](http://pwq45mhk2fzyldzycvmi6alcxj7ku3ht2oo4qkngya52dwcpvm57d7ad.onion/Rylatech/vault)

## What isn't published

The mint application itself is not open source. Publishing the operational codebase would enlarge the attack surface without adding verifiable value; the security properties that matter to users (BDHKE unlinkability, reserve attestations, denomination bearer format) live in the primitives above rather than in the mint's business logic. Those custody and attestation schemes are standard cryptographic constructions with the primitives published, and the math is independently verifiable.

> Trust is a choice. Verification is a right. These are published so you can exercise the second one.
