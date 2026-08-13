# Fees

All mint fees are collected upfront on deposit. Withdrawals are free at the mint layer; users pay only the Lightning routing fee the network charges to relay their payment.

- **Deposit** — **0.75%** (7,500 ppm). Shown before payment, collected on the invoice.
- **Withdrawal** — **0%** mint fee. User pays the Lightning routing fee the network charges the mint to relay the payment. Estimated at melt-quote time, refunded if the actual route is cheaper (see NUT-08).
- **Swap** — **Free**. Denomination changes and privacy refreshes cost nothing.
- **P2P transfer** — **Free**. Handing a token string to someone else is not a mint operation.
- **Splice fee (deposit)** — When a deposit exceeds the mint's inbound Lightning liquidity, opening additional capacity incurs a splice-out fee. The quote surfaces the exact total before payment; no surprise on top of the 0.75%.

Fee schedule is configurable per deployment. Defaults are the published rates above.
