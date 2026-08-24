# Registry Lookup API & Site Declaration

The two mechanisms that connect content in the wild to registry records — a batch fingerprint lookup with signed receipts, and a domain-level declaration of registered works.

This document specifies the two discovery mechanisms that connect content in the wild to registry records: the Lookup API (pull — an AI system asks whether content is registered) and the Site Declaration (push — a publisher declares registrations at the domain level). A third element, the public record URL, is the citable anchor both resolve to, and a fourth, the page embed (§5), surfaces registration data on the published page itself. A plain-language overview for rights holders is at [how discovery works](/discovery).

## 1. Design principles

- Verification flows toward the registry. Content need not carry any mark; the fingerprint is intrinsic to the file. Marks and declarations are accelerants, not requirements.

- The registry holds the record, never the work. No endpoint accepts or returns content. Lookups exchange hashes and metadata only.

- Every lookup is evidence. Query logs are retained and exportable so a licensee can demonstrate diligence (EU AI Act Article 53 documentation; analogous regimes).

- Honest by design. v0.1 matches exact fingerprints only. Perceptual matching for derived and transformed media is roadmap (§6) and will be tiered by confidence, consistent with the registry's four verification tiers.

## 2. Record identifiers and record URLs

Live today. Every registration receives:

- Record ID: a 64-character SHA-256 hex digest of the record's own canonical payload — e.g. `9f86d081…4e1b2c3d`. Content-addressed, so a record is immutable: any change produces a different record, and the old one is superseded rather than edited.

- Record URL: `https://alltio.com/r/{recordId}` — permanent public page, with the same record as JSON under `Accept: application/json`. Shows: rights holder, work title, content type, SHA-256 fingerprint, registration timestamp, licensing preference, version label, supersession links, and the countersignature status.

- Verify URL: `https://alltio.com/api/verify/{recordId}` — the JSON API. Unchanged and permanent; records filed before record pages shipped carry this URL.

Record URLs are the citation target for takedowns, C2PA manifest references, site badges, and contracts.

### 2.1 Version chains and supersession

Records are immutable: a revision of a work is a new registration with a new fingerprint and a new record. The revising registration MAY declare `supersedes` (the prior record's ID) and MUST carry a version label when it does. The registry validates that the prior record exists and, on verified registrant continuity, maintains the forward pointer:

- Back-link — part of the new record's countersigned content: `supersedes: { recordId, verified }`.

- Forward pointer — registry-maintained metadata on the prior record (`supersededBy`, `supersededAtUTC`), outside the signed payload, since signed content is never modified after issuance.

Superseded records remain permanently resolvable and are marked `superseded: true` in verification responses. Consumers SHOULD treat the newest unsuperseded record in a chain as current, and MAY traverse the chain for dated custody history. Silent revision of a published work against an unsuperseded record breaks fingerprint continuity and SHOULD be treated as a mismatch, not as a match.

## 3. Lookup API

### 3.1 Endpoint

`POST https://api.alltio.com/v1/lookup Authorization: Bearer {api-key} Content-Type: application/json` API keys are free for lookup at standard rate limits; volume tiers for ingestion-scale batch use. Free lookup is deliberate: the query itself is the adoption wedge.

### 3.2 Request

Batch of up to 10,000 fingerprints per call:

`{ "fingerprints": [ { "alg": "sha256", "hash": "9f86d081884c7d65...", "ref": "client-opaque-id-001" }, { "alg": "sha256", "hash": "60303ae22b998861...", "ref": "client-opaque-id-002" } ], "context": "training | retrieval | evaluation", "attest": true }` `ref` — the caller's opaque correlation ID, echoed back; never interpreted. `context` — optional; refines which policy fields are returned (training and inference terms may differ). `attest: true` — requests a signed lookup receipt (§3.4). 3.3 Response `{ "results": [ { "ref": "client-opaque-id-001", "status": "registered", "record_id": "9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08", "record_url": "https://alltio.com/r/9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08", "policy": { "ai_use": "licensable", "training": { "allowed": true, "pricing": "usage_based" }, "retrieval": { "allowed": true, "pricing": "per_retrieval" }, "contract": "https://alltio.com/contracts/std-v1" }, "registered_at": "2026-03-14T09:22:31Z" }, { "ref": "client-opaque-id-002", "status": "registered", "record_id": "60303ae22b998861bce3b28f33eec1be758a213c86c93c076dbe9f558c11c752", "record_url": "https://alltio.com/r/60303ae22b998861bce3b28f33eec1be758a213c86c93c076dbe9f558c11c752", "policy": { "ai_use": "excluded" }, "registered_at": "2026-05-02T17:03:10Z" } ], "unmatched": [], "receipt": { "id": "rcpt_8842...", "signature": "..." } }` `status` values: `registered`, `not_found`, `pending_verification` (a record exists; ownership verification for licensing is not yet complete — a self-attested claim only, per the registry's honesty posture).

`policy.ai_use` values: `licensable`, `excluded` (“not available for AI use” is a first-class answer), `contact` (bespoke terms; the rights holder opted for inquiry-based licensing).

### 3.4 Lookup receipts (attestation)

When `attest: true`, the response includes a signed receipt binding: caller identity, timestamp, fingerprint-set digest, and result digest. Receipts are retained by Alltio and exportable as an audit bundle — the artifact a licensee shows a regulator or counterparty to prove pre-use diligence. Signature: Ed25519 over a canonical JSON serialization; Alltio's public keys are published at `https://alltio.com/.well-known/alltio-keys.json`.

### 3.5 Rate, privacy, and abuse posture

- Hashes submitted for lookup are not published and not added to the registry; `not_found` submissions are retained only in the caller's own receipt log.

- Lookup traffic is never used to infer or market to unregistered rights holders.

- Enumeration protection: record pages are public, but asset IDs are non-sequential; bulk export of the registry is a licensed product for verified counterparties, not an open dump.

## 4. Site Declaration

### 4.1 Location and relationship to RSL and robots.txt

A publisher declares registered works at:

`https://{domain}/.well-known/alltio.json` and optionally signals its existence in `robots.txt` / RSL:

`# robots.txt Alltio-declaration: https://example.com/.well-known/alltio.json` Positioning: RSL registers sites; the Alltio declaration registers works. The declaration complements site-level licensing signals by enumerating asset-level records — it does not replace crawl directives and must not be interpreted as one.

### 4.2 Format

`{ "alltio_declaration": "0.1", "publisher": { "name": "Example Media LLC", "registry_profile": "https://alltio.com/p/example-media" }, "default_policy": "https://alltio.com/policy/default", "assets": [ { "record_id": "9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08", "record_url": "https://alltio.com/r/9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08", "canonical": "https://example.com/reports/2026-market-study.pdf", "sha256": "9f86d081884c7d65..." } ], "assets_index": "https://alltio.com/decl/example-media/assets.json", "generated_at": "2026-07-19T12:00:00Z", "signature": "..." }` Large catalogs use `assets_index` (paginated, registry-hosted) instead of inlining thousands of entries. `default_policy` lets a publisher assert blanket terms for works on the domain not yet individually registered — a claim, flagged as such, weaker than per-asset records. The file is generated and signed by Alltio from the publisher's registrations; publishers download and place it. Signing prevents third parties from fabricating declarations that claim someone else's terms. 4.3 Crawler contract A compliant crawler, on encountering the declaration: (a) resolves policies before fetching listed canonicals, or (b) fetches and verifies by fingerprint afterward. Either path yields the same record; the declaration simply moves discovery earlier. Crawlers should treat a failed signature as no-declaration, not as permission.

## 5. Page embed and human-visible mark

### 5.1 The registered-with-Alltio page embed

The page embed surfaces a work's registration on the page where it is published — the granularity at which crawlers ingest content. It complements §3 and §4: the registry remains the source of truth; the embed moves discovery to the point of publication. It is generated verbatim by the registry at registration time (and shown on the registration result screen); publishers paste it and replace it wholesale when a new version is registered. It has two parts:

(a) Visible notice — one line, adjacent to the work:

`Registered with Alltio · record 9f86d081…4b8ed012 · v2 · registered 2026-07-19 · supersedes record 60303ae2…2b998861` linking to the record URL. Version and supersession fragments appear only when the record carries them.

(b) Machine-readable block — JSON-LD, schema.org `CreativeWork` with the Dublin Core `dcterms` namespace for supersession:

`<script type="application/ld+json"> { "@context": ["https://schema.org", {"dcterms": "http://purl.org/dc/terms/"}], "@type": "CreativeWork", "name": "2026 Market Study", "author": "Example Media LLC", "version": "v2", "identifier": [ {"@type": "PropertyValue", "propertyID": "alltio:recordId", "value": "9f86d081884c7d65...4b8ed012"}, {"@type": "PropertyValue", "propertyID": "sha256", "value": "60303ae22b998861...bc48c2af"} ], "subjectOf": {"@type": "WebPage", "@id": "https://alltio.com/api/verify/9f86d081884c7d65...", "name": "Alltio registry record"}, "license": "https://alltio.com/api/verify/9f86d081884c7d65...", "dcterms:replaces": "https://alltio.com/api/verify/60303ae22b998861..." } </script>` `identifier` — the record ID under `propertyID: "alltio:recordId"` and the content fingerprint under `propertyID: "sha256"`. Both REQUIRED. `version` and `dcterms:replaces` — present iff the record declares them (§2.1). `dcterms:replaces` points at the superseded record's URL, never at a page. `subjectOf` / `license` — the record URL, where the machine-readable policy lives. New registrations emit `https://alltio.com/r/{recordId}`. Embeds generated before record pages shipped carry `/api/verify/{recordId}`; both resolve permanently, so those embeds do not need regeneration. A consumer encountering an embed SHOULD verify it against the registry (§3) rather than trusting page-asserted values; a fingerprint mismatch between the page's file and the embedded record is a mismatch (§2.1). Reference implementation: [maximin.ai](https://maximin.ai), publications of Alltio's founding entity, is the designated reference implementation and will carry the embed on every registered page at launch — the worked example this section is normative for.

### 5.2 Badge (informative)

A “Registered with Alltio” badge (SVG, hotlinked or self-hosted) links to the publisher's registry profile or a specific record URL. The badge is deterrence and normalization — it carries no machine semantics; machines use §3, §4 and §5.1. Filename conventions and embedded watermarks are explicitly out of scope: they strip on copy and would signal informality.

For media with C2PA support, the record URL may be embedded as an assertion in the content-credentials manifest, binding provenance metadata to the registry record.

## 6. Roadmap (non-normative)

- Perceptual fingerprints (image, video, audio) as additional `alg` values, returned with an explicit confidence field and settled at the corresponding confidence tier.

- Chunk-level hashes for long documents, enabling excerpt matching in RAG pipelines.

- Agent credentials: the lookup-receipt format extends to agent-presented verifiable credentials, per the trust-layer architecture.

## 7. Summary of the flow

`Rights holder registers ──► fingerprint + terms on record │ ├──► places /.well-known/alltio.json (site: terms known pre-crawl) └──► cites record URL / badge (humans: claim is visible) AI system ingests file ──► sha256 ──► POST /v1/lookup │ ├── registered + licensable ──► standard contract ──► usage report ──► settlement ├── registered + excluded ──► exclude; receipt proves compliance └── not_found ──► proceed under own policy; receipt proves diligence` Both sides integrate once. The record does the rest.

## Put your knowledge on the record.

Free. Read and fingerprinted in your browser, not stored.
