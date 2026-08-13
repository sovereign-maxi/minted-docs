# Token Format

Cashu tokens are encoded as the string `cashuA` followed by base64url-encoded JSON. The format is defined in [NUT-00](https://github.com/cashubtc/nuts/blob/main/00.md).

```
cashuA<base64url-encoded JSON>
```

The decoded JSON contains:

- **`token`** — Array of token entries. Each entry pairs a mint URL with a list of proofs.
- **`mint`** — The mint URL that issued these tokens. Required for redemption.
- **`proofs`** — Array of individual proofs. Each proof: `amount` (power-of-2 sats), `id` (keyset ID), `secret` (hex), and `C` (unblinded signature point, hex).

## Denominations

Tokens use 21 fixed power-of-2 denominations: `2^0` (1 sat) through `2^20` (1,048,576 sats). Any amount is representable as the binary sum of these denominations. Uniform denomination structure is a privacy property — a 1,000-sat balance looks the same across every wallet that holds one, so amounts are not fingerprints.
