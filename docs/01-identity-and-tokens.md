# Identity & Tokens

The protocol has no accounts. There is no email, password, keyphrase, or session identifier that ties a person to a balance. Identity *is* the token — a token string, once issued, is a bearer instrument. Whoever holds the string owns the value.

- **Bearer instrument** — A token is an `(amount, secret, unblinded_signature, keyset_id)` tuple. Possession is ownership. Redemption verifies the signature and checks the secret against the spent set; nothing else.
- **Client-side storage** — Tokens live in the browser's localStorage. The mint holds signing keys and a spent set of hashed secrets; it does not hold user balances or track who owns what.
- **No recovery** — There is no "forgot password" flow because there is no password. If you lose the tokens without a backup, the value is gone the way physical cash is gone.
- **Peer-to-peer transferable** — The `cashuA...` token string can be copied and handed to anyone through any channel. The recipient redeems at the mint without either party revealing an identity.

See [Token Format](./08-token-format.md) for the wire encoding and [Deposit Flow](./05-deposit-flow.md) for how a token is issued.
