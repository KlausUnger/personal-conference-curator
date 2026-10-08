<!-- harness: 2026-09-30 -->
# System Design Document: personal-conference-curator

<!-- AGENT: Current state only. The path to each decision lives in adr/. -->
<!-- AGENT: Source code is authoritative for types, interfaces, parameters, and constant values. Name each contract once, say what it guarantees and which requirement it implements, and point at the source file. Do not transcribe fields, parameters, or constant literals — in a table OR in prose. They rot when the code changes and add no design information. -->
<!-- AGENT: Cross-reference prd.md for requirements, adr/ for decisions. -->

## Overview

The product is two deployable units that share one data contract. A static browser application runs every attendee-facing behavior on the attendee's device. A scheduled conference fetcher publishes public conference data as static files. No runtime backend exists. Per [ADR: Static browser app and scheduled conference fetcher](adr/2026-10-08-static-browser-app-and-scheduled-fetcher.md), an S3 bucket behind CloudFront serves both the page and the published files.

The conference bundle is the contract between the two units. The fetcher writes one bundle per conference and a conference index. A bundle carries stable session identities, a bundle version, and explicit cancellation and replacement markers. The project owns the bundle schema at both ends, so the page maps it directly. The fetcher's per-source parsers are the anti-corruption boundary, one mapper per conference source.

The inference gateway is the only path from the page to a model. It calls one provider from an allowlist of OpenAI and Anthropic, with the attendee's own key. It sends only the current slot's sessions and a profile summary, and requests structured output against a schema. Bundle text inside a prompt is untrusted input.

The model produces structured judgments about one session against one profile: scores, explanations, and profile distillation. Code owns every rule with an invariant: must-attend constraints, slot membership, conflict detection, schedule merging across bundle versions, persistence, and export, import, and erase.

Speaker fit relates a speaker's published bio to the attendee's stated speaker preferences `[REQ-SPKR-001]`. The bundle carries each speaker bio as the conference publishes it, and the fetcher adds no other speaker information. No module requests, produces, or stores a judgment of a speaker's competence, trustworthiness, or credibility, per [ADR: Non-goal: judging speakers](adr/2026-10-08-non-goal-speaker-judgment.md). A speaker fit explanation cites the bio and the attendee's preference.

The module map, dependency policy, and security context follow.

## Package Structure

Both units are planned; no TypeScript source exists yet. Each module is a bounded context with a public API.

```text
.
├── web/                       (planned) static browser application, served from S3 behind CloudFront
│   ├── conference-catalog/    loads the conference index and bundles; sessions, slots, speakers as published
│   ├── attendee-profile/      the distilled attendee profile and its corrections
│   ├── schedule/              attendance marks, must-attend constraints, conflicts, merging across bundle versions
│   ├── curation/              per-slot ranking and explanations assembled from model judgments
│   ├── conversation/          the preference conversation and natural-language preference feedback
│   ├── inference-gateway/     the allowlisted provider call, consent, prompt assembly, structured-output validation
│   └── local-store/           browser persistence of personal data; export, import, and erase
├── fetcher/                   (planned) scheduled Lambda function; per-source parsers write bundles and the index
└── bundle-schema/             (planned) the conference bundle and index types shared by both units
```

## Constants

<!-- Name each constant and cite the source file that owns its value; do not copy the value (source is authoritative). -->

| Name | Source | Description |
|------|--------|-------------|

## Contracts

<!-- One row per public type, interface, or function. Purpose in one line; the source file owns the signature; Implements names the requirement(s). No field or parameter lists — those live in source. Add a short prose note above the table only for an invariant a row cannot carry. -->

| Contract | Purpose | Source | Implements |
|----------|---------|--------|------------|

## Scale and Load

<!-- PROJECT: The workload the code is sized for. The design owner writes each row at triage; the implementer selects against it and the code-quality reviewer judges against the same row. A figure nobody knows is written as "unrecorded, treated as bounded" and corrected here when the real figure arrives. "Bounded" is a complete row. -->

| Data set or path | Realistic size | Growth | Access pattern | Form |
|------------------|----------------|--------|----------------|------|
| Conference bundle | Unrecorded, treated as bounded: the sessions of one conference | Per conference; republished on each scheduled run | Shared read-mostly; loaded whole into memory on each page load | One published file per conference, validated against the bundle schema on load |
| Conference index | Unrecorded, treated as bounded: one entry per published conference | One entry per conference added | Shared read-mostly | One small published file |
| Personal conference data | Unrecorded, treated as bounded: one profile, marks, and rankings per conference the attendee plans | Grows with the attendee's conferences | Request-private; held in browser storage on the attendee's device | Browser storage, read and written by the local store |
| Inference request payload | Unrecorded, treated as bounded: one slot's sessions plus a profile summary | Fixed per request by the slot's size | Request-private | One structured-output request per ranking or distillation call |
| Fetcher run | Unrecorded, treated as bounded: one source per conference | One source per conference added | Scheduled batch | One run per schedule; a failed source keeps its last published bundle |
<!-- One row per collection or path that scales with use: rows, requests per second, payload size. Access pattern: request-private, shared read-mostly, shared read-write. Form: the structure or algorithm chosen. On a hot path it carries its time and space bound with its kind (worst-case, average, amortized). A hand-written one, or a measured claim, links its ADR. -->

- **Hot paths:** The per-slot ranking round trip from the page to the attendee's provider is the attendee-visible latency path. The "What next?" request during the event runs on the same path while the attendee waits between sessions. No latency figure is recorded yet.
- **Limits:** The deployment imposes three limits. The provider caps request size, which bounds the inference request payload. The browser caps storage per origin, which bounds personal conference data. The Lambda timeout bounds one fetcher run. No figure is recorded for any of them yet.

## Dependency Policy

Minimize external dependencies. Every dependency is an attack surface and a maintenance burden.

### Approved Sources

| Source | Examples | Rationale |
|--------|----------|-----------|
| Vendor-owned packages on the npm registry (language and platform owners) | TypeScript compiler, AWS SDK for JavaScript v3 (fetcher only) | Owned by the vendor of the language or runtime |
| Browser platform APIs | `fetch`, Web Crypto, IndexedDB, Web Storage | No dependency; part of the runtime |

Every other package is an exception recorded in an ADR.

### Adding a New Dependency

Before adding a dependency, verify:

1. **Necessity** — Can the standard library solve the problem? A dependency for one function is justified only when the function is more than a few lines of tested code and the library is established. The decision weighs writing and maintaining the code against the dependency and its transitive tree.
2. **Source** — Is it from an approved source above? If not, create an ADR.
3. **Audit** — Review transitive dependencies. Flag unknown modules.
4. **Verification** — Verify checksums and commit the lockfile.

### Prohibited

| Class | Reason |
|-------|--------|
| Scripts loaded at runtime from a third-party origin | The page holds the attendee's API key; per [ADR: Static browser app and scheduled conference fetcher](adr/2026-10-08-static-browser-app-and-scheduled-fetcher.md), the content security policy admits same-origin scripts only |
| Analytics, tracking, or advertising packages | Excluded by REQ-PRIV-003 |
| Provider SDKs or clients that reach hosts outside the provider allowlist | The allowlist is fixed per [ADR: Static browser app and scheduled conference fetcher](adr/2026-10-08-static-browser-app-and-scheduled-fetcher.md) |

## Security Context

Per [ADR: Static browser app and scheduled conference fetcher](adr/2026-10-08-static-browser-app-and-scheduled-fetcher.md), the page and the fetcher share no runtime and no credentials.

- **Inputs it processes:** The page reads the conference index and bundles from its own origin, attendee input (conversation, preference feedback, attendance marks, imported files), and structured model output. The fetcher reads conference schedules from conference sources.
- **Outputs it produces:** The page renders the personal schedule and rankings, writes personal data to browser storage, and produces export files. The fetcher writes bundles and the conference index to the S3 bucket.
- **External services it connects to:** The page connects to OpenAI or Anthropic, as the attendee chooses, and to its own CloudFront origin. The fetcher connects to the conference sources and the S3 bucket.
- **Credential handling:** The attendee supplies a provider API key; the page holds it in browser storage and sends it only to the chosen provider. The fetcher holds only the credentials source fetching requires, plus an execution role scoped to writing the bucket. No credential reaches the published files.
- **Runtime:** The page is static files in S3 behind CloudFront, which terminates TLS and sets a strict content security policy through a response-headers policy. The fetcher is a scheduled AWS Lambda function.

## Threat Model

Per [ADR: Static browser app and scheduled conference fetcher](adr/2026-10-08-static-browser-app-and-scheduled-fetcher.md), bundle text is untrusted at every boundary it crosses.

| Threat | Attack Vector | Mitigation |
|--------|--------------|------------|
| API key exfiltration | A cross-site scripting flaw runs script in the page and reads the key from browser storage | Strict content security policy from CloudFront: same-origin scripts only, no inline script, connect targets limited to the origin and the two providers; bundle and model text rendered as text, never as markup |
| Prompt injection | A session abstract or speaker bio carries instructions aimed at the attendee's model | Bundle text enters prompts as delimited data; the model has no tools with side effects; output is validated against a schema and invariant rules stay in code |
| Malicious or compromised conference source | A source serves oversized, malformed, or hostile schedule data to the fetcher | One parser per source validates shape and size and rejects what the bundle schema does not allow; a failed source leaves its last published bundle in place |
| Tampered bundle | An attacker alters a bundle or the index in the bucket or in transit | TLS from CloudFront; the bucket is writable only by the fetcher's execution role and readable only through CloudFront; the page validates every bundle against the schema before use |
| Key or personal data sent to an unlisted host | A crafted bundle or setting points inference at an attacker endpoint | The provider allowlist is fixed in code and enforced by the content security policy's connect targets |

## Implementation Order

| ID | Name | Depends On |
|----|------|------------|

## State Machine

<!-- Define state transitions as parseable tables when the system carries state. -->
