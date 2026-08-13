# Swap Flow

Exchanging existing tokens for new tokens of the same total value ([NUT-03](https://github.com/cashubtc/nuts/blob/main/03.md)). Used for denomination splitting or combining, and for privacy refresh after a token has been in circulation.

1. **Prepare inputs and outputs**
   Client selects proofs to spend (inputs) and generates fresh blinded messages for the target denominations (outputs). Total input value must equal total output value.

2. **Submit swap**
   Client sends `POST /v1/swap` with the old proofs and new blinded messages.

3. **Atomic spend-and-sign**
   Mint verifies every input, marks them spent, and signs all outputs in one atomic operation. If commit fails after signing (a durability failure), a conforming mint MUST halt rather than return signatures — refusing to open a double-spend window is the honest failure mode.

4. **Unblind new tokens**
   Client unblinds the returned signatures and stores the fresh proofs. The old proofs are discarded. The new tokens have no cryptographic relationship to the old ones — a fresh anonymity set.
