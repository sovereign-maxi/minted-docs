# Deposit Flow

Converting Bitcoin into eCash tokens ([NUT-04](https://github.com/cashubtc/nuts/blob/main/04.md)). Two-step: quote, then claim.

1. **Request quote**
   Client sends `POST /v1/mint/quote` with the desired amount and unit (`sat`). Mint calculates the 0.75% fee (plus any splice-fee uplift if the amount exceeds inbound liquidity), returns a bolt11 Lightning invoice for the total and a `quote` ID.

2. **Pay invoice**
   Client pays the bolt11 from any Lightning wallet. The mint's Lightning node receives it and the quote is marked paid.

3. **Generate blinds (client)**
   Client decomposes the paid amount into power-of-2 denominations, generates a secret and blinding factor for each, computes `B' = hash_to_curve(secret) + rG` per denomination.

4. **Claim tokens**
   Client sends `POST /v1/mint/quote/:id` with the quote ID and the list of blinded messages. Mint verifies the quote is paid, writes a durable mint-intent record before signing, claims the quote atomically to prevent double-mint, then signs each blinded message: `C' = kB'`. Returns the blind signatures with DLEQ proofs.

5. **Signature persistence (server)**
   Before pushing signatures to the browser, the server writes them to a durable pending-signatures store. A crash between sign and client-receipt keeps the signatures durable; the browser re-requests them on reconnect.

6. **Unblind (client)**
   Client removes each blinding factor: `C = C' - rK`. The result is a valid signature on the secret that the mint has never seen. Client stores the resulting proof tuples in localStorage, then acknowledges storage back to the server, which drops the pending-signatures entry.

An orphaned pending-signatures entry (browser closed before the storage ACK) is reconciled by a background process after the configured threshold (default 1 hour). A compensating burn record keeps the outstanding-liability counter accurate; the tokens are never redeemable because the client secrets don't exist anywhere.
