# Cashu NUT Compliance

The protocol implements the **Cashu NUT** (Notation, Usage, and Terminology) specification — an open specification for Chaumian eCash mints. The full specification is published at [github.com/cashubtc/nuts](https://github.com/cashubtc/nuts). This reference implementation supports the following NUTs:

- **[NUT-00](https://github.com/cashubtc/nuts/blob/main/00.md)** — Notation and models. BDHKE blind signature scheme, token format, proofs, promises, blind messages.
- **[NUT-01](https://github.com/cashubtc/nuts/blob/main/01.md)** — Mint public keys. Clients fetch the active public keys and keyset IDs per denomination.
- **[NUT-02](https://github.com/cashubtc/nuts/blob/main/02.md)** — Keysets. Multiple keyset support, keyset ID derivation (SHA256 of sorted concatenated pubkeys, first 8 bytes), rotation, lifecycle.
- **[NUT-03](https://github.com/cashubtc/nuts/blob/main/03.md)** — Swap. Atomic spend-and-mint in one request: old proofs in, new blind signatures out.
- **[NUT-04](https://github.com/cashubtc/nuts/blob/main/04.md)** — Mint (deposit). Quote returns a Lightning invoice; claim submits blind messages and receives signatures.
- **[NUT-05](https://github.com/cashubtc/nuts/blob/main/05.md)** — Melt (withdrawal). Quote returns fee estimate; melt spends tokens and pays the invoice. Atomic — both succeed or neither does.
- **[NUT-06](https://github.com/cashubtc/nuts/blob/main/06.md)** — Mint information. Discovery endpoint at `/v1/info`: name, version, supported NUTs, contact, public keys.
- **[NUT-07](https://github.com/cashubtc/nuts/blob/main/07.md)** — Token state check. Verify whether proofs are spent, pending, or unspent by their Y-point (hash-to-curve of the secret), without revealing the secret.
- **[NUT-08](https://github.com/cashubtc/nuts/blob/main/08.md)** — Lightning fee return. Melt overpayment (routing fee lower than quoted) is returned as change tokens signed against blank outputs supplied at melt time.
- **[NUT-12](https://github.com/cashubtc/nuts/blob/main/12.md)** — DLEQ proofs. Every blind signature carries a Discrete Log Equality proof so clients can verify the mint used the correct private key without seeing it.

Payload formats and per-endpoint schemas are defined by the individual NUT specifications above. Protocol-specific extensions (proof of reserves) are documented in [Proof of Reserves](./11-proof-of-reserves.md).
