# Warrant Canary

The project publishes a monthly PGP-clearsigned warrant canary at `/canary.txt`. The statement attests the mint has received no legal process, gag orders, backdoors, key disclosures, or infrastructure seizures. Absence of a fresh signed canary past its valid-until date is the signal the canary has been broken.

- **Signing key fingerprint** — `147D AF57 BFE7 4DE1 CDE0  14E8 E338 C4DC 8A7B 5C20`. Stable for the mint's lifetime. Verify every canary against this exact number. If it changes without a PGP-signed rotation announcement first, treat as compromise.
- **Cadence** — Reissued monthly, on or before the 14th. A canary past its valid-until date is dead by design. A late reissue is not a legitimate reissue.
- **Where to find it** — The signed text at `/canary.txt`, the public key at `/canary.asc`. Nostr posts announce new canaries. The signature on the file is what you verify. Not the Nostr post.

## Verify

```shell
curl -O https://minted.is/canary.asc
gpg --import canary.asc
curl -O https://minted.is/canary.txt
gpg --verify canary.txt
```

The `Good signature from` line MUST show the fingerprint above. Expect a `WARNING: The key's User ID is not certified` line. Trust anchors to the fingerprint, not your local keyring.
