# Nostr Event Schemas

The protocol defines one kind of event for public consumption on Nostr and additionally emits NIP-17 gift-wrapped DMs for internal alerting. All events are canonical NIP-01 events signed with BIP-340 Schnorr signatures on secp256k1 x-only pubkeys.

## Kind 30078 — Reserves Attestation

NIP-33 addressable event carrying the mint's proof-of-reserves attestation. Published each proof cycle under the mint's Nostr signing key. The event `content` field is a JSON-encoded string on the wire; shown here as a nested object for readability.

```json
{
  "kind": 30078,
  "content": {
    "epoch_id": 42,
    "reserve_ratio": 1.028806,
    "total_held": 12345678,
    "outstanding": 12000000,
    "attestation_count": 1,
    "asset_ids": ["..."],
    "captured_at": "2026-07-16T14:32:00Z"
  },
  "tags": [
    ["d", "<16-char hex, per-cycle>"],
    ["t", "proof-of-reserves"],
    ["epoch", "42"],
    ["ratio", "1.028806"]
  ],
  "created_at": 1721140320,
  "pubkey": "<mint signing pubkey>",
  "id": "<sha256 of canonical serialisation>",
  "sig": "<BIP-340 Schnorr signature>"
}
```

The `d` tag is a 16-character hex hash derived from `SHA256(domain || pubkey_hex || ":" || captured_at_unix)`. Distinct timestamps yield distinct d-tags, so each cycle is its own addressable event and Nostr carries the full historical sequence — deliberately avoiding the standard NIP-33 replace semantic.

## Signature Verification

Standard NIP-01 event id + signature scheme. The event id is `SHA256(canonical_serialisation)` where the canonical form is:

```
[0, pubkey_hex, created_at, kind, tags, content]
```

JSON-encoded per NIP-01. The signature is a BIP-340 Schnorr signature over the event id bytes. Any client library that speaks NIP-01 can verify these events without mint-specific knowledge.
