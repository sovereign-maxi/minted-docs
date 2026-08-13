# Withdrawal Flow

Converting eCash tokens back into Bitcoin ([NUT-05](https://github.com/cashubtc/nuts/blob/main/05.md)). Atomic: either the Lightning payment succeeds and the tokens are marked spent, or the payment fails definitively and the tokens are released for retry.

1. **Request quote**
   Client sends `POST /v1/melt/quote` with a bolt11 Lightning invoice (the destination) and unit. Mint parses the invoice, estimates the routing fee via its Lightning node, returns a quote with the total token amount required.

2. **Submit proofs**
   Client sends `POST /v1/melt/quote/:id` with the quote ID, proofs (inputs) covering the amount plus routing-fee reserve, and optionally blank outputs for change ([NUT-08](https://github.com/cashubtc/nuts/blob/main/08.md)).

3. **Verify and reserve**
   Mint verifies each proof: recomputes `Y = hash_to_curve(secret)` and checks `C == kY` against the keyset that signed it. Valid proofs are moved into the pending reservation table — not committed to the durable spent set yet, but blocked from re-spending.

4. **Pay invoice**
   Mint pays the client's bolt11 via its Lightning node. On success, the reservation is committed to the spent set. On definitive failure, the reservation is released and the client can retry with the same proofs.

5. **Return change (NUT-08)**
   If the actual routing fee is lower than the quoted reserve, the mint signs the client's blank outputs for the overpayment. Change is signed with the *active* keyset, not the input keyset, so change tokens carry forward the current signing policy.

## Settlement Resolution

Lightning payments occasionally have ambiguous outcomes (payment stuck in flight past the timeout, the Lightning node briefly unresponsive at the wrong moment). The mint refuses to guess: the quote enters an unknown-settlement state and the proofs stay reserved (fail-closed) rather than being released early. A background resolver then polls the Lightning node every 60 seconds for quotes older than 10 minutes and either commits the reservation (payment did settle) or releases it (payment did not). Most stuck quotes resolve automatically within minutes.
