# Public Endpoints

Cashu-standard JSON endpoints with no authentication and no user-identifying data exposed. All endpoints return `application/json; charset=utf-8`. The protocol is transport-agnostic — a mint may serve these endpoints over any HTTP transport. The reference implementation deploys as a Tor hidden service; the `curl` examples below assume that shape (route through Tor SOCKS at `127.0.0.1:9050` — Tor Browser's default — or `torsocks`). A mint on clearnet is the same API without the SOCKS hop.

## GET /v1/info

Mint metadata (NUT-06).

Name, version, active pubkey, supported NUTs, and the Nostr publisher pubkey for reserves cross-verification.

**Auth:** None.

**Path parameters:** None.

**Query parameters:** none.

**Request:**

```shell
curl --socks5-hostname 127.0.0.1:9050 \
     http://<mint>.onion/v1/info
```

**Response · `200 OK`:**

```json
{
  "name": "Minted",
  "pubkey": "<active keyset pubkey>",
  "version": "Minted/0.1.0",
  "nuts": {"0": {}, "1": {}, "2": {}, "3": {}, "4": {}, "5": {}, "6": {}, "7": {}, "8": {}, "12": {}},
  "nostr": {
    "publisher_pubkey": "<secp256k1 x-only hex>",
    "publisher_curve": "secp256k1-bip340",
    "guardian_pubkey": "<ed25519 hex>",
    "guardian_curve": "ed25519",
    "kind": 30078,
    "threshold_signature_input": "Vault.Proof.canonical_bytes/1"
  },
  "contact": []
}
```

## GET /v1/keysets

List all keysets (NUT-01 / NUT-02).

Every keyset the mint knows about, with active status. Clients enumerate this to discover which keysets to fetch keys for.

**Auth:** None.

**Path parameters:** None.

**Query parameters:** None.

**Request:**

```shell
curl --socks5-hostname 127.0.0.1:9050 \
     http://<mint>.onion/v1/keysets
```

**Response · `200 OK`:**

```json
{
  "keysets": [
    {"id": "<8-byte hex>", "unit": "sat", "active": true},
    {"id": "<8-byte hex>", "unit": "sat", "active": false}
  ]
}
```

## GET /v1/keysets/:id

Public keys for a specific keyset (NUT-01 / NUT-02).

All 21 denomination public keys for the given keyset ID.

**Auth:** None.

**Path parameters:**

- **`id`** — 8-byte hex keyset identifier from the [keysets list](#get-v1keysets).

**Query parameters:** None.

**Request:**

```shell
curl --socks5-hostname 127.0.0.1:9050 \
     http://<mint>.onion/v1/keysets/<id>
```

**Response · `200 OK`:**

```json
{
  "keysets": [
    {
      "id": "<8-byte hex>",
      "unit": "sat",
      "keys": {
        "1": "<secp256k1 pubkey hex>",
        "2": "<secp256k1 pubkey hex>",
        "4": "<secp256k1 pubkey hex>",
        "...": "...",
        "1048576": "<secp256k1 pubkey hex>"
      }
    }
  ]
}
```

## POST /v1/mint/quote

Request a deposit quote (NUT-04).

Returns a bolt11 Lightning invoice for the requested amount plus the mint's configured fee (plus any splice-fee uplift if the amount exceeds inbound liquidity).

**Auth:** None.

**Path parameters:** None.

**Query parameters:** None.

**Request body:**

```json
{"amount": 10000, "unit": "sat"}
```

**Response · `200 OK`:**

```json
{
  "quote": "<quote id>",
  "request": "lnbc...",
  "state": "unpaid",
  "expiry": 1721143920
}
```

## POST /v1/mint/quote/:id

Claim tokens for a paid mint quote (NUT-04).

Submits blinded messages after the Lightning invoice is paid; receives blind signatures with DLEQ proofs (NUT-12).

**Auth:** None.

**Path parameters:**

- **`id`** — Mint quote id returned by [POST /v1/mint/quote](#post-v1mintquote). The Lightning invoice referenced by this quote must be paid before claim will succeed.

**Query parameters:** None.

**Request body:**

```json
{
  "quote": "<quote id>",
  "outputs": [
    {"amount": 8, "id": "<keyset id>", "B_": "<blinded message point hex>"},
    {"amount": 2, "id": "<keyset id>", "B_": "<blinded message point hex>"}
  ]
}
```

**Response · `200 OK`:**

```json
{
  "signatures": [
    {
      "amount": 8,
      "id": "<keyset id>",
      "C_": "<blind signature point hex>",
      "dleq": {"e": "<hex>", "s": "<hex>"}
    }
  ]
}
```

## POST /v1/melt/quote

Request a withdrawal quote (NUT-05).

Parses the destination bolt11, estimates the Lightning routing fee, returns the total token amount required.

**Auth:** None.

**Path parameters:** None.

**Query parameters:** None.

**Request body:**

```json
{"request": "lnbc...", "unit": "sat"}
```

**Response · `200 OK`:**

```json
{
  "quote": "<quote id>",
  "amount": 5000,
  "fee_reserve": 25,
  "state": "unpaid",
  "expiry": 1721143920
}
```

## POST /v1/melt/quote/:id

Execute a withdrawal (NUT-05 + NUT-08).

Submits proofs covering the amount plus fee reserve, optionally with blank outputs for change. On success the mint pays the invoice and returns overpayment as change tokens signed with the active keyset.

**Auth:** None.

**Path parameters:**

- **`id`** — Melt quote id returned by [POST /v1/melt/quote](#post-v1meltquote). The referenced bolt11 invoice will be paid on successful submission.

**Query parameters:** None.

**Request body:**

```json
{
  "quote": "<quote id>",
  "inputs": [
    {"amount": 4096, "id": "<keyset id>", "secret": "<hex>", "C": "<hex>"},
    {"amount": 1024, "id": "<keyset id>", "secret": "<hex>", "C": "<hex>"}
  ],
  "outputs": [
    {"amount": 1, "id": "<keyset id>", "B_": "<blank blinded output hex>"}
  ]
}
```

**Response · `200 OK`:**

```json
{
  "quote": "<quote id>",
  "state": "paid",
  "payment_preimage": "<hex>",
  "change": [
    {"amount": 1, "id": "<active keyset id>", "C_": "<hex>", "dleq": {"e": "<hex>", "s": "<hex>"}}
  ]
}
```

## POST /v1/swap

Atomic swap (NUT-03).

Spends old proofs and issues new blind signatures for the same total amount. Free. Used for denomination changes and privacy refresh.

**Auth:** None.

**Path parameters:** None.

**Query parameters:** None.

**Request body:**

```json
{
  "inputs": [
    {"amount": 4096, "id": "<keyset id>", "secret": "<hex>", "C": "<hex>"}
  ],
  "outputs": [
    {"amount": 2048, "id": "<keyset id>", "B_": "<hex>"},
    {"amount": 2048, "id": "<keyset id>", "B_": "<hex>"}
  ]
}
```

**Response · `200 OK`:**

```json
{
  "signatures": [
    {"amount": 2048, "id": "<keyset id>", "C_": "<hex>", "dleq": {"e": "<hex>", "s": "<hex>"}},
    {"amount": 2048, "id": "<keyset id>", "C_": "<hex>", "dleq": {"e": "<hex>", "s": "<hex>"}}
  ]
}
```

## POST /v1/check

Check proof state (NUT-07).

Reports whether each proof is spent, pending, or unspent, keyed by the Y-point (`hash_to_curve(secret)`) so the secret itself is never disclosed.

**Auth:** None.

**Path parameters:** None.

**Query parameters:** None.

**Request body:**

```json
{"Ys": ["<Y-point hex>", "<Y-point hex>"]}
```

**Response · `200 OK`:**

```json
{
  "states": [
    {"Y": "<hex>", "state": "SPENT"},
    {"Y": "<hex>", "state": "UNSPENT"}
  ]
}
```

## GET /v1/reserves

Latest signed proof of reserves.

Returns the most recent reserves attestation as a signed JSON object. See [Proof of Reserves](./11-proof-of-reserves.md) for the field-by-field reference.

**Auth:** None.

**Path parameters:** None.

**Query parameters:** none.

**Request:**

```shell
curl --socks5-hostname 127.0.0.1:9050 \
     http://<mint>.onion/v1/reserves
```

**Response · `200 OK`:**

```json
{
  "ratio": 1.028806,
  "held": 12345678,
  "outstanding": 12000000,
  "proof": "a1b2c3d4...",
  "attestation_count": 1,
  "threshold_signature": "9f8e7d...",
  "nostr_event_id": "1122334455...",
  "timestamp": "2026-07-16T14:32:00Z",
  "verifier": {
    "nostr_publisher_pubkey": "5566778899...",
    "nostr_pubkey_curve": "secp256k1-bip340",
    "guardian_signer_pubkey": "aabbccddee...",
    "guardian_pubkey_curve": "ed25519",
    "threshold_signature_input": "Vault.Proof.canonical_bytes/1"
  }
}
```

## GET /v1/reserves/history

Paginated history of proofs, most recent first.

Returns up to `limit` historical proofs in the same signed-attestation format as `/v1/reserves`.

**Auth:** None.

**Path parameters:** None.

**Query parameters:**

- **`limit`** — Integer, optional. Number of proofs to return, clamped to `1..100`. Default `20`.

**Request:**

```shell
curl --socks5-hostname 127.0.0.1:9050 \
     "http://<mint>.onion/v1/reserves/history?limit=5"
```

**Response · `200 OK`:**

```json
{
  "proofs": [
    {
      "ratio": 1.028806,
      "held": 12345678,
      "outstanding": 12000000,
      "proof": "a1b2c3d4...",
      "attestation_count": 1,
      "threshold_signature": "9f8e7d...",
      "nostr_event_id": "1122334455...",
      "timestamp": "2026-07-16T14:32:00Z"
    }
  ],
  "limit": 5,
  "cursor": null,
  "verifier": {
    "nostr_publisher_pubkey": "5566778899...",
    "nostr_pubkey_curve": "secp256k1-bip340",
    "guardian_signer_pubkey": "aabbccddee...",
    "guardian_pubkey_curve": "ed25519",
    "threshold_signature_input": "Vault.Proof.canonical_bytes/1"
  }
}
```

The `cursor` field is reserved for future pagination and is currently always `null`.

## GET /health/live

Liveness probe.

Returns `200 OK` once the server process is responsive. Does not check whether the application is ready to serve traffic.

**Auth:** None.

**Path parameters:** None.

**Query parameters:** None.

**Request:**

```shell
curl --socks5-hostname 127.0.0.1:9050 \
     http://<mint>.onion/health/live
```

**Response · `200 OK`:**

```json
{"status": "ok"}
```

## GET /health/ready

Readiness probe.

Returns `200 OK` once the application supervision tree is up and the mint is ready to serve traffic. Returns `503 Service Unavailable` while booting or when the mint is halted.

**Auth:** None.

**Path parameters:** None.

**Query parameters:** None.

**Request:**

```shell
curl --socks5-hostname 127.0.0.1:9050 \
     http://<mint>.onion/health/ready
```

**Response · `200 OK`:**

```json
{"status": "ok"}
```

**Response · `503 Service Unavailable`:**

```json
{"status": "unavailable"}
```

## Rate Limiting

Every endpoint runs behind the request gate documented in [Rate Limiting & PoW](./13-rate-limiting.md). Sensible clients cache reserves responses for at least the proof-cycle interval; per-second polling is unnecessary and wasteful.
