<!-- harness: 2026-09-30 -->
# Product Requirements Document: personal-conference-curator

<!-- AGENT: This PRD is narrative prose. State WHAT the system does and WHY it matters. Never HOW — mechanism (flags, exit codes, file layouts, algorithms) lives in system-design.md. Never the decision trail — alternatives and trade-offs live in adr/. -->
<!-- AGENT: Annotate each requirement inline with its [REQ-XX-NNN] tag where the prose expresses it, and give it one "Done when" acceptance bullet carrying the same tag. The prose is the intent; the tagged bullet is the bounded, testable contract. Drop an <a id="req-xx-nnn"></a> anchor at first mention so other docs deep-link to it. -->
<!-- AGENT: A requirement is active by being in the narrative — there is no per-requirement Status field. Retire one by moving it to the Superseded list; never renumber an ID. -->

## Context

Conference attendees choose between concurrent sessions with limited information. Conference schedules expose titles, abstracts, speakers, tracks, and difficulty labels. They do not account for an attendee's expertise, goals, prior sessions, or changing interests.

The Personal Conference Curator builds a personal conference plan from those factors. It learns preferences through conversation, ranks sessions within each time slot, explains the ranking, and adapts as the conference progresses.

The product optimizes for expected learning and value across the full conference. Topic similarity alone does not determine a recommendation.

An attendee opens a conference and completes a short preference conversation. The product distills the conversation into an attendee profile. The attendee can inspect and correct that profile. For each time slot, the product ranks every available session. Each ranking includes a concise explanation. During the conference, new preference feedback changes later rankings. The attendee can ask "What next?" to receive recommendations for the upcoming time slot.

## Goals

| ID | Goal | Success Metric |
|----|------|----------------|
| G-1 | Establish an attendee profile through conversation | After the preference conversation, the attendee sees a ranked time slot with an explanation for each session, without completing a fixed questionnaire |
| G-2 | Rank all sessions within a time slot | Every session in a time slot receives a personal ranking |
| G-3 | Understand why each session fits or does not fit | Each session has a short explanation |
| G-4 | Balance relevance, novelty, depth, speaker fit, and variety | The ranking changes when relevant preferences change |
| G-5 | Define fixed attendance commitments | Must-attend sessions act as fixed schedule constraints |
| G-6 | Adapt the schedule after new preferences | Natural-language preference feedback affects later rankings |
| G-7 | Ask what to attend next during the event | A "What next?" request ranks the next available sessions |
| G-8 | Retain control over all personal conference data | No server stores personal conference data |

## Non-Goals

<!-- Declined scope with the reason it was declined; never silently dropped. A non-goal ADR records the path to each decision. The first REQ id on a row is the declined one; a later id names a successor, which stays open. -->

| ID | Non-Goal | Rationale |
|----|----------|-----------|
| NG-1 | Ticketing, messaging, social networking, attendee directories, or advertising | Not recorded; see Open Questions |
| NG-2 | Organizer-facing attendee profiles | Not recorded; see Open Questions |
| NG-3 | Replacing the conference schedule as the source of session times and rooms | The conference schedule remains the source of session times and rooms |
| NG-4 | Rating attended sessions (REQ-FEED-001) | A rating measures the gap between a session's description and its reality, which no later session shares |
| NG-5 | Assessing a speaker's competence, trustworthiness, or credibility, hosted or on the device | Speaker fit describes the match between a speaker's published background and the attendee's stated preferences, not the person |

## Requirements

### Preference discovery

<a id="req-pref-001"></a>
The product conducts an adaptive preference conversation `[REQ-PREF-001]`. The conversation captures interests, expertise, desired depth, preferred session characteristics, and explicit dislikes.

<a id="req-pref-002"></a>
The attendee can update preferences through natural-language feedback `[REQ-PREF-002]`.

**Done when:**
- `[REQ-PREF-001]` after the preference conversation, the attendee sees a ranked time slot with an explanation for each session, without completing a fixed questionnaire;
- `[REQ-PREF-001]` the attendee can review and correct the distilled attendee profile;
- `[REQ-PREF-002]` statements such as "too basic" or "more architecture" affect later rankings.

### Session ranking

<a id="req-rank-001"></a>
The product ranks all available sessions within each time slot `[REQ-RANK-001]`. Ranking considers relevance, expected novelty, depth, speaker fit, prior attendance, repetition, and variety.

<a id="req-rank-002"></a>
The product explains each ranking `[REQ-RANK-002]`.

<a id="req-rank-003"></a>
Official difficulty labels inform ranking but do not act as filters `[REQ-RANK-003]`.

**Done when:**
- `[REQ-RANK-001]` every session remains visible;
- `[REQ-RANK-001]` each session receives a personal ranking;
- `[REQ-RANK-001]` the ranking changes when relevant preferences change;
- `[REQ-RANK-002]` each session has a short explanation;
- `[REQ-RANK-002]` expanded explanations relate the session to the attendee profile and conference history;
- `[REQ-RANK-003]` a beginner-labelled session can rank highly when its content remains relevant to an experienced attendee.

### Schedule management

<a id="req-sched-001"></a>
The attendee can mark sessions as Attend, Must attend, Watch recording, or Skip `[REQ-SCHED-001]`.

<a id="req-sched-002"></a>
The product adapts to conference schedule changes `[REQ-SCHED-002]`.

**Done when:**
- `[REQ-SCHED-001]` must-attend sessions act as fixed schedule constraints;
- `[REQ-SCHED-001]` conflicting sessions remain available as alternatives;
- `[REQ-SCHED-002]` updated times, rooms, cancellations, or replacements appear in the personal schedule.

### Live conference use

<a id="req-live-001"></a>
The product supports a "What next?" interaction `[REQ-LIVE-001]`.

**Done when:**
- `[REQ-LIVE-001]` the result ranks the next available sessions;
- `[REQ-LIVE-001]` the result accounts for current preferences, prior attendance, and fixed commitments;
- `[REQ-LIVE-001]` the result presents room and time information when available.

### Speaker context

<a id="req-spkr-001"></a>
The product relates each speaker's published background to the attendee's stated speaker preferences `[REQ-SPKR-001]`. Speaker fit describes that match, not the person. A ranking explanation states the match as a preference, such as "matches your preference for practitioner speakers". The product never judges a speaker's competence, trustworthiness, or credibility (NG-5).

**Done when:**
- `[REQ-SPKR-001]` each speaker fit explanation cites the speaker's published bio and the attendee's stated speaker preference;
- `[REQ-SPKR-001]` speaker fit draws on no speaker information beyond the bio the conference publishes;
- `[REQ-SPKR-001]` no explanation or stored data states a judgment of a speaker's competence, trustworthiness, or credibility.

### Privacy

<a id="req-priv-001"></a>
Persistent personal data remains in browser storage `[REQ-PRIV-001]`. This includes preferences, schedules, feedback, attendance history, and recommendation history.

<a id="req-priv-002"></a>
Cloud inference requires explicit consent `[REQ-PRIV-002]`.

<a id="req-priv-003"></a>
The product does not use personal conference data for advertising, resale, organizer profiling, or model training by the application's operator `[REQ-PRIV-003]`.

<a id="req-priv-004"></a>
The attendee can erase, export, and import personal conference data `[REQ-PRIV-004]`.

**Done when:**
- `[REQ-PRIV-001]` no server stores personal conference data;
- `[REQ-PRIV-002]` consent identifies the cloud processing purpose;
- `[REQ-PRIV-002]` the product states which provider processes the request;
- `[REQ-PRIV-002]` the product sends only context required for that request;
- `[REQ-PRIV-003]` the application includes no analytics, tracking, or advertising dependency, and the page connects to no host other than its own origin and the chosen provider;
- `[REQ-PRIV-004]` erase removes all personal conference data from the device;
- `[REQ-PRIV-004]` export and import preserve all personal conference data.

## Superseded

<!-- Retired requirements: each ID maps to its successor (or to the reason it was withdrawn) so existing links still resolve. Keep this a list, so every retired ID stays in a list item. -->

- `REQ-FEED-001` withdrawn: rating attended sessions is a non-goal (NG-4). Natural-language preference feedback stays under `REQ-PREF-002`.

## Open Questions

<!-- Unresolved product questions. Each resolves into a requirement, a non-goal, or an ADR. -->

- NG-1 and NG-2 carry no recorded rationale. The owner supplies one for each, and a non-goal ADR records it.
