# BDHKE Protocol

Blind Diffie-Hellman Key Exchange on secp256k1: the modern elliptic-curve adaptation of Chaum's 1982 blind signature scheme. The mint signs blinded messages it cannot inspect and later verifies unblinded signatures it cannot correlate with the signing request. The unlinkability is a mathematical property of the scheme, not a policy.

1. **Blinding (client)**
   The client generates a secret `x` and a random blinding factor `r`. Computes `Y = hash_to_curve(x)` and the blinded message `B' = Y + rG`, where `G` is the secp256k1 generator. Only `B'` ever leaves the client; `x`, `Y`, and `r` stay local.

2. **Signing (mint)**
   The mint signs the blinded message with the private key `k` for the requested denomination: `C' = kB'`. It has signed something whose plaintext it has never seen. Returns `C'` along with a DLEQ proof (NUT-12) that lets the client verify the signature came from the correct denomination key.

3. **Unblinding (client)**
   The client removes the blinding factor: `C = C' - rK`, where `K = kG` is the mint's denomination public key. The result `C` is a valid signature on `Y` that has no cryptographic link to the blinded message `B'` the mint signed.

4. **Redemption (client → mint)**
   The client presents `(x, C)`. The mint computes `Y = hash_to_curve(x)` and verifies `C == kY`. A valid signature that the mint *cannot* correlate with the original blinded signing request.

The blinding factor `r` is destroyed after unblinding. Even a mint that logged every blinded message it ever signed cannot determine which unblinded token corresponds to which request — the link never existed.

Server-side BDHKE is implemented as a Rust NIF in [Cashew](./published-libraries.md). Client-side blinding and unblinding run in the browser via [Nutty](./published-libraries.md) compiled to WebAssembly — the blinding factor never leaves the client.
