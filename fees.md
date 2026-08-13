# Fees

The protocol supports arbitrary per-deployment fee schedules. Fee amounts are surfaced to clients at quote time so the total is always visible before payment. Withdrawals may be free at the mint layer, with users paying only the Lightning routing fee the network charges to relay their payment.

- **Deposit** — Collected upfront on the invoice. Amount is disclosed at quote time.
- **Withdrawal** — May be free at the mint layer. If charged, the routing-fee reserve is estimated at melt-quote time and any overpayment is refunded as change ([NUT-08](https://github.com/cashubtc/nuts/blob/main/08.md)).
- **Swap** — Typically free. Denomination changes and privacy refreshes cost nothing to a mint.
- **P2P transfer** — Free. Handing a token string to someone else is not a mint operation.
- **Splice fee (deposit)** — If a deposit exceeds the mint's inbound Lightning liquidity, opening additional capacity may incur a splice-out fee. The quote surfaces the exact total before payment.

## Reference implementation defaults

- **Deposit** — **0.75%** (7,500 ppm), collected upfront.
- **Withdrawal** — **0%** mint fee. User pays the Lightning routing fee only.
- **Swap** — **Free**.

All three are configurable per deployment.
