# Alltio Registry Lookup API

Alltio is the registry of works for the AI knowledge economy.  Registration creates a permanent, timestamped record: this work, this owner, these terms.  This repository holds the public specification for querying that registry.

## Contents

- `spec/lookup-api.md`: the Lookup API and Site Declaration.  Batch fingerprint lookup with signed receipts, and the domain-level declaration of registered works.
- `spec/embed.md`: the page embed and human-visible mark.
- `examples/site-declaration.json`: a minimal `/.well-known/alltio.json` declaration.

## Status

v0.1 matches exact SHA-256 fingerprints only.  Canonical and perceptual matching are on the roadmap and will carry stated confidence values when they ship, never a bare yes.

The canonical text of this specification lives at [alltio.com/lookup](https://alltio.com/lookup).  This repository mirrors it for implementers; where the two differ, the site governs.

## Implementing

Implementation is encouraged and needs no permission.  Signed lookup responses verify against the published keys at `https://alltio.com/.well-known/alltio-keys.json` using Ed25519 over a canonical JSON serialization.  The registry holds the record, never the work: no endpoint accepts or returns content, and lookups exchange hashes and metadata only.

## License

The specification texts in this repository are licensed under [CC BY 4.0](LICENSE).

Contact: contact@alltio.com
