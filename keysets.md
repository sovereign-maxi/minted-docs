# Keysets & Rotation

A **keyset** is the set of 21 signing keys (one per denomination) the mint uses to blind-sign tokens. Every proof carries the `keyset_id` it was signed under, so retired keysets can still redeem their own tokens indefinitely while new mints go against the active keyset only.

- **Active** — The current signing keyset. New mints and swap outputs are signed under this keyset.
- **Retired** — Superseded by rotation but still holds its private keys. Tokens signed under a retired keyset remain redeemable via melt or swap indefinitely.
- **Expired** — Kill switch for a compromised keyset. Private keys are scrubbed and replaced with a sentinel. Neither mint nor melt can proceed for tokens signed under an expired keyset.
- **Encryption at rest** — Private keys are encrypted with AES-256-GCM under a key derived from `MINTED_ENCRYPTION_KEY` (mandatory in production, refuses to boot without it).

## Rotation

Rotation is manual, not scheduled. A `keyset_rotation_due` warning alert fires when the active keyset is older than 30 days; that is the cue to rotate. The old keyset transitions to *retired* so its outstanding tokens continue to redeem indefinitely.

Compaction of the spent set is keyset-scoped: entries for a keyset can be pruned only after that keyset is expired (i.e. can no longer redeem), so pruning cannot delete a live-defence entry.
