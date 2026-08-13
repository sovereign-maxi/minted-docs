# Proof of Reserves

A conforming mint publishes proof of reserves through **two independent verification paths**:

- **HTTP endpoint** — `GET /v1/reserves`: current proof, queryable anytime via Tor. Read-only, no auth. Paginated history at `/v1/reserves/history?limit=N`.
- **Nostr NIP-33 events** — Kind 30078 replaceable events published to configured relays each cycle. Same numbers, tamper-evident historical record cross-signable against the HTTP surface.

The `ratio` field (held ÷ outstanding) is the number to watch: `≥ 1.0` means fully backed. A drop below 1.0 should never happen; if it does, treat as under-collateralised and stop depositing.

## HTTP Payload Shape

A successful `GET /v1/reserves` returns a JSON object of the form:

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

Fields:

- **`ratio`** — Held ÷ outstanding as a float. 1.0 when outstanding is zero. Above 1.0 means over-reserved.
- **`held`** — Total sats held in the mint's Lightning channels at snapshot time.
- **`outstanding`** — Total outstanding eCash liability, in sats. Computed as `minted - burned` from the liability counter.
- **`proof`** — Hex-encoded proof-cycle identifier. Uniquely identifies this attestation across the history.
- **`attestation_count`** — Number of signer attestations that combined to produce the threshold signature. 1 in a single-node deployment (the mint is its own signer).
- **`threshold_signature`** — Hex-encoded Ed25519 signature over `Vault.Proof.canonical_bytes/1`. The signer pubkey validates against this.
- **`nostr_event_id`** — Hex-encoded ID of the matching Nostr kind:30078 event. Cross-reference: fetch the event from a relay, verify its BIP-340 signature under the publisher pubkey, confirm the content matches.
- **`verifier`** — Names the pubkeys, curves, and signature input for both signatures the reader is expected to validate. Two curves, two purposes: never validate the Ed25519 threshold signature with the secp256k1 Nostr pubkey, or vice versa.

## Sanity Check with curl

Quick liveness check that the endpoint is up, the ratio is at or above 1.0, and the JSON parses cleanly. Requires `jq` and a Tor SOCKS proxy on port 9050 (Tor Browser's default; standalone Tor daemon's default).

```shell
curl --socks5-hostname 127.0.0.1:9050 \
     http://<mint>.onion/v1/reserves | jq

# Or with torsocks installed:
torsocks curl http://<mint>.onion/v1/reserves | jq

# Ratio-only check
curl --socks5-hostname 127.0.0.1:9050 \
     http://<mint>.onion/v1/reserves | jq .ratio
```

## Programmatic Verification

Fetches the endpoint, confirms the JSON shape, checks that outstanding is fully collateralised, and validates that the timestamp is recent. Real verification of the Ed25519 threshold signature and the Nostr BIP-340 signature requires curve libraries beyond the standard runtime: use `cryptography` (Python) or `:crypto.verify/5` (Elixir) for Ed25519, and `coincurve` or an equivalent BIP-340 Schnorr binding for the Nostr signature.

### Python

```python
import json
from datetime import datetime, timezone

import requests

# Route through Tor SOCKS. Requires PySocks:
#   pip install requests[socks]
SESSION = requests.Session()
SESSION.proxies = {
    "http":  "socks5h://127.0.0.1:9050",
    "https": "socks5h://127.0.0.1:9050",
}

MINT = "http://<mint>.onion"

def fetch_reserves():
    r = SESSION.get(f"{MINT}/v1/reserves", timeout=60)
    r.raise_for_status()
    return r.json()

def check(proof):
    held = proof["held"]
    outstanding = proof["outstanding"]
    ratio = proof["ratio"]

    assert held >= outstanding, (
        f"under-collateralised: held={held} outstanding={outstanding}"
    )
    assert ratio >= 1.0, f"ratio below 1.0: {ratio}"

    ts = datetime.fromisoformat(proof["timestamp"].replace("Z", "+00:00"))
    age = (datetime.now(timezone.utc) - ts).total_seconds()
    assert age < 24 * 3600, f"proof stale by {age:.0f}s"

    return {
        "ratio": ratio,
        "held_sats": held,
        "outstanding_sats": outstanding,
        "delta_sats": held - outstanding,
        "attestation_count": proof["attestation_count"],
        "nostr_event_id": proof["nostr_event_id"],
        "guardian_pubkey": proof["verifier"]["guardian_signer_pubkey"],
        "nostr_pubkey": proof["verifier"]["nostr_publisher_pubkey"],
    }

if __name__ == "__main__":
    print(json.dumps(check(fetch_reserves()), indent=2))
```

### Elixir

```elixir
defmodule ReservesVerifier do
  @moduledoc "Read-only fetch + validation of /v1/reserves."

  @mint "http://<mint>.onion"

  def check do
    with {:ok, proof} <- fetch(),
         :ok <- assert_collateralised(proof),
         :ok <- assert_fresh(proof, _max_age_s = 86_400) do
      {:ok,
       %{
         ratio: proof["ratio"],
         held_sats: proof["held"],
         outstanding_sats: proof["outstanding"],
         delta_sats: proof["held"] - proof["outstanding"],
         attestation_count: proof["attestation_count"],
         nostr_event_id: proof["nostr_event_id"]
       }}
    end
  end

  defp fetch do
    url = @mint <> "/v1/reserves"
    req = Req.new(url: url, connect_options: [proxy: {:socks5, ~c"127.0.0.1", 9050, []}])

    case Req.get(req) do
      {:ok, %{status: 200, body: body}} -> {:ok, body}
      {:ok, %{status: s}} -> {:error, {:http, s}}
      {:error, reason} -> {:error, reason}
    end
  end

  defp assert_collateralised(%{"held" => h, "outstanding" => o}) when h >= o, do: :ok
  defp assert_collateralised(_), do: {:error, :under_collateralised}

  defp assert_fresh(%{"timestamp" => iso}, max_age_s) do
    {:ok, ts, _} = DateTime.from_iso8601(iso)

    case DateTime.diff(DateTime.utc_now(), ts, :second) do
      age when age < max_age_s -> :ok
      age -> {:error, {:stale, age}}
    end
  end
end
```

## Cross-Verification with Nostr

The HTTP proof and the Nostr kind:30078 event carry the same numbers. Fetch the Nostr event by its `nostr_event_id` from any relay, verify its BIP-340 signature under the `nostr_publisher_pubkey`, and confirm the content JSON matches the HTTP payload's `held` / `outstanding` / `ratio`. The mint cannot tell the API one number and Nostr a different one without breaking a signature.

## History

Prior proofs are available at `GET /v1/reserves/history` with an optional `?limit=<n>` parameter (default 20, clamped 1..100). The Nostr stream is a parallel historical record.
