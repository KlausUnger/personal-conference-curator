# Static browser app and scheduled conference fetcher

**Status:** Accepted

## Context

The privacy requirements keep all persistent personal data in browser storage and require consent before any cloud inference (REQ-PRIV-001 to REQ-PRIV-004). One possible shape is a server-hosted service that calls the model. Such a server receives the profile and the conversation on every request. The question is where the product runs and how conference data reaches the attendee.

Conference data is public and changes during the event. Personal data is private and must never leave the device except in a consented inference request. The two flows have different owners and different trust levels.

## Options Considered

1. **Server-hosted service** - The server holds the model call and serves the page. Every inference request passes through infrastructure the product operates.
2. **Static page with a runtime proxy** - The page calls a thin backend that forwards requests to the provider. The proxy exists to work around cross-origin limits.
3. **Static page with user-typed endpoints** - The attendee enters any inference endpoint and key.
4. **Static page, direct provider calls from an allowlist, scheduled fetcher** - The page talks only to OpenAI or Anthropic. A scheduled function publishes conference data as static files.

## Decision

We choose option 4.

The product is a static browser application written in TypeScript with little or no framework. An S3 bucket serves it behind CloudFront. CloudFront terminates TLS and sets the response headers, including a strict content security policy. There is no runtime backend.

The page calls the attendee's chosen provider directly with a key the attendee supplies and the browser stores. The provider allowlist holds exactly OpenAI and Anthropic. A fixed allowlist keeps the policy's network targets to two known hosts. It also lets the consent dialog name the provider that processes each request.

A scheduled Lambda function, also in TypeScript, pulls conference information from the conference sources. It writes one bundle per conference and a small conference index into the same bucket. Each bundle carries stable session identities, a bundle version, and explicit cancellation and replacement markers. Stable identities keep attendee marks valid across republishes. The version lets the page detect schedule changes.

The project owns the bundle schema at both ends, so the page maps it directly. The Lambda's per-source parsers form the anti-corruption boundary, with one mapper per conference source.

The model produces structured judgments about one session against one profile: scores, explanations, and profile distillation. Code owns every rule with an invariant. These rules are must-attend constraints, slot membership, conflict detection, schedule merging across bundle versions, persistence, and export, import, and erase.

The inference boundary is a security boundary. Session abstracts and speaker bios are untrusted text inside prompts. Requests ask for structured output against a schema and grant the model no tools with side effects. A request carries only the current slot's sessions and a profile summary.

## Consequences

- No server ever holds personal data, so REQ-PRIV-001 holds by construction rather than by policy.
- The product operates no inference infrastructure and pays no model costs. Each attendee pays their own provider.
- The API key lives in the browser. A cross-site scripting flaw can exfiltrate it, so the content security policy and the absence of third-party scripts carry real weight.
- Adding a third provider is a recorded decision, because it widens the content security policy and the consent text.
- Conference data is only as fresh as the last scheduled run.
- Speaker fit is computed on the device from the published speaker bio and the attendee's stated preferences. No component judges a speaker, consistent with NG-5.

## Implementation

**Requirements:** REQ-PRIV-001, REQ-PRIV-002, REQ-PRIV-004, REQ-SCHED-001, REQ-SCHED-002, REQ-RANK-001, REQ-RANK-002, REQ-SPKR-001

## References

- [System design: Overview](../system-design.md#overview)
- [System design: Security Context](../system-design.md#security-context)
- [PRD: REQ-PRIV-002](../prd.md#req-priv-002)
- [PRD: REQ-SCHED-002](../prd.md#req-sched-002)
- [Non-goal: judging speakers](2026-10-08-non-goal-speaker-judgment.md)
