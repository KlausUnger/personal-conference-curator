<!-- harness: 2026-09-30 -->
# Security Principles

This brief specializes the harness's non-negotiable security laws for this project. The four laws — security as an emergent property, defense in depth, least privilege, fail secure — are harness-owned: a project decides *how* it meets them, never *whether*. This document carries that "how": the project's trust-boundary map and the stack's state-of-the-art security defaults. The feature-implementer designs against it; the security-reviewer enforces it; the exhaustive item-by-item checklist lives in the `security-checks` skill.

## Trust Boundaries

A trust boundary is any point where data or control crosses from less-trusted to more-trusted. Every boundary the change introduces or crosses gets the same treatment.

| At the boundary | The rule |
|---|---|
| External input arrives (request, file, env, message) | Validate type, range, and shape before use; reject what the contract does not allow |
| A secret is read (token, key, password) | It never reaches a log, an error message, a URL, or a process argument |
| A fetcher or deployment secret is stored (source credentials, execution role) | It lives in the environment or a secret store, never in committed source |
| An attendee-supplied secret is stored (the provider API key) | It lives in browser storage on the attendee's device, under the content security policy, and is sent only to the allowlisted provider; it never reaches the published files, a log, or a URL |
| Personal conference data leaves the device | It goes only to the allowlisted provider the attendee chose, only after consent that names the provider and the purpose, and carries only the current slot's sessions and a profile summary; it never reaches the published files or any other host |
| Model output arrives | Structured output from the provider is external input: validate it against its schema before use and reject it whole on failure |
| A privilege is exercised (file, network, process, query) | Grant the minimum scope; deny by default |
| An error crosses back out | The message carries debugging context inward, never internal detail outward |

Internal code, past the boundary, trusts its contracts — defensive checks belong at the boundary, not scattered through the core.

## Language Realization

These classes derive from the Threat Model and Dependency Policy in `docs/system-design.md`. Each control is the bar a change starts from, not a floor to argue down. The `security-checks` skill holds the exhaustive checklist.

| Class | Principle at risk | Control |
|-------|-------------------|---------|
| Cross-site scripting and API key exfiltration | Defense in depth | The content security policy admits same-origin scripts only and no inline script; bundle and model text render as text, never as markup |
| Prompt injection through bundle text | Fail secure | Bundle text enters prompts as delimited data; model output is schema-validated; invariant rules stay in code; the model gets no tools with side effects |
| Personal data or the key sent to an unlisted host | Least privilege | The provider allowlist is fixed in code and mirrored in the policy's connect targets |
| Malformed or oversized conference source data | Fail secure | The fetcher's per-source parser validates shape and size and rejects what the bundle schema does not allow; the last good bundle stays published |
| Tampered bundle | Defense in depth | TLS from CloudFront; only the fetcher's role writes the bucket; the page validates every bundle against the schema before use |
| Supply chain | Least privilege | The lockfile is committed and checksums are verified; no third-party runtime scripts; no analytics, tracking, or advertising packages |
| Fetcher privilege | Least privilege | The fetcher's execution role is scoped to writing the bucket |

## Standing Gaps

A reviewer states a listed gap in one clause and raises nothing for it. A gap not listed here is raised once, as a `clarify` finding to the owner, and then listed.

- **No dependency vulnerability scanner runs.** The gate verbs in `scripts/stack.sh` are unbound until the TypeScript toolchain is chosen.
- **No linter runs.** The lint verb is unbound for the same reason.
- **No modularity test exists.** The dependency-boundary check waits on the same toolchain decision.
