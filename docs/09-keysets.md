# Keysets & Rotation

A **keyset** is the set of 21 signing keys (one per denomination) a mint uses to blind-sign tokens. Every proof carries the `keyset_id` it was signed under, so retired keysets can still redeem their own tokens indefinitely while new mints go against the active keyset only.

- **Active** — The current signing keyset. New mints and swap outputs are signed under this keyset.
- **Retired** — Superseded by rotation but still holds its private keys. Tokens signed under a retired keyset remain redeemable via melt or swap indefinitely.
- **Expired** — Kill switch for a compromised keyset. Private keys are scrubbed and replaced with a sentinel. Neither mint nor melt can proceed for tokens signed under an expired keyset.
- **Encryption at rest** — Protocol requires that active and retired keyset private keys be encrypted when persisted to disk. The specific cipher and key-management strategy are implementation choices.

## Rotation

Rotation is manual, not scheduled — a mint operator chooses when to rotate based on their own policy. The old keyset transitions to *retired* so its outstanding tokens continue to redeem indefinitely.

Compaction of the spent set is keyset-scoped: entries for a keyset can be pruned only after that keyset is expired (i.e. can no longer redeem), so pruning cannot delete a live-defence entry.

## Reference implementation

- **At-rest encryption** — Private keys are encrypted with AES-256-GCM under a key derived from `MINTED_ENCRYPTION_KEY` (mandatory in production; the process refuses to boot without it).
- **Rotation cadence signal** — A `keyset_rotation_due` warning alert fires when the active keyset is older than 30 days as the operator's cue to rotate. The alert threshold is configurable.
