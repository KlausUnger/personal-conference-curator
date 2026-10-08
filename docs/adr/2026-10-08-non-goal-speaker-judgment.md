# Non-goal: judging speakers

**Status:** Accepted

## Context

NG-5 declined hosted or precomputed speaker credibility assessments. It still allowed a credibility summary that the attendee's device requested and stored. The owner asked for a more inclusive wording than "Speaker credibility".

During intake the owner chose speaker fit: "How well a speaker's published background matches the attendee's stated preferences. Describes the match, not the person."

## Options Considered

1. **Keep NG-5 as written.** The product declines only hosted or precomputed credibility assessments. A credibility summary on the device stays in scope.
2. **Decline judging speakers at all.** The product declines any assessment of a speaker, hosted or on the device. Speaker fit replaces the credibility summary.

## Decision

The product never judges a speaker. The owner's decision reads: "The product never produces an assessment of a speaker's competence, trustworthiness, or credibility, hosted or on the device. Broader than NG-5 and replaces it."

REQ-SPKR-001 keeps its ID and becomes speaker fit. The product relates a speaker's published background to the attendee's stated preferences. An explanation cites the published bio and the attendee's preference, never a judgment of the person.

## Consequences

- The NG-5 row now covers assessments on the device as well as hosted ones.
- Speaker Background, Speaker Preference, and Speaker Fit replace Credibility Summary and Speaker Evidence in the domain vocabulary.
- Speaker fit draws only on the bio the conference publishes.
- The architecture ADR's reference to on-device credibility summaries no longer holds and needs new wording.

## Implementation

**Non-goal:** NG-5

## References

- [PRD Non-Goals](../prd.md#non-goals)
- [REQ-SPKR-001](../prd.md#req-spkr-001)
- [Static browser app and scheduled conference fetcher](2026-10-08-static-browser-app-and-scheduled-fetcher.md)
