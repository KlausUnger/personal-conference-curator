<!-- harness: 2026-09-30 -->
# Ubiquitous Language

<!-- The canonical domain vocabulary for this project — the "ubiquitous language" in the Domain-Driven Design sense (Eric Evans, Chapter 2). The same terms are used by stakeholders, the PRD, the system design, and the source code.

  This file is durable memory. Agents and developers across sessions read it to keep the same vocabulary; drift here ripples into variable, function, and file names across the codebase.

  ENTRY FORMAT

    **TermName**: One-sentence definition. Define what it IS, not what it does. Relationships: (optional) one line naming the related concepts and the cardinality where obvious. Avoid: (optional) other words used in the wild for the same concept that this project rejects. Listing them heads off drift.

  Example:

    **Order**: A customer-placed request for one or more line items, accepted for fulfillment but not yet shipped. Relationships: An Order contains one or more LineItems and references one Customer. Avoid: Purchase, Transaction (those are billing-context terms).

  WHEN TO ADD A TERM

  - The moment a term resolves during a requirements interview or a design discussion. Do not batch. Do not wait for a second use.
  - When recurring domain terms in existing docs or source code aren't yet captured (adoption case — the system-design-expert may write here during a foundational triage to seed the initial vocabulary).

  WHAT NOT TO PUT HERE

  - Harness methodology vocabulary (slice, loop, triage verdict, etc.) — that is the team's method, not this project's domain. This file holds project-domain terms only.
  - Implementation details. Definitions describe domain concepts, not code shape.

  CONSUMERS

  - The product-requirements-expert resolves and writes PRD terms.
  - The system-design-expert resolves design terms; it writes here only during the foundational triage path on adoption.
  - The doc-reviewer lints cross-document term consistency.
  - The feature-implementer names new domain-facing code from the entries; the code-quality-reviewer checks those names against them.

  CADENCE

  Slow. The ubiquitous language changes less often than the PRD or system design. Treat updates here as load-bearing — they ripple into variable, function, and file names across the codebase. -->

## Domain Terms

**Attendee**: A person attending a conference who uses the product to build a personal conference plan. Relationships: An Attendee has one Attendee Profile and owns all of their Personal Conference Data.

**Conference**: A single event whose published schedule of sessions, time slots, rooms, and speakers the product curates. Relationships: A Conference is published as one Conference Bundle and listed in the Conference Index.

**Session**: One scheduled presentation in a conference, with the title, abstract, speakers, track, difficulty label, time, and room the conference publishes. Relationships: A Session belongs to one Time Slot, has one or more Speakers, and carries one Session Identity.

**Time Slot**: A period of the conference schedule during which concurrent sessions run. Relationships: A Time Slot contains one or more Sessions and has one Ranking per Attendee.

**Speaker**: A person who presents a session, described by the short bio the conference publishes. Relationships: A Speaker presents one or more Sessions, has one Speaker Background, and has one Speaker Fit per Attendee. Avoid: speaker credibility, speaker assessment (the product judges no speaker).

**Attendee Profile**: The distilled, attendee-correctable summary of an attendee's interests, expertise, desired depth, preferred session characteristics, and explicit dislikes. Relationships: An Attendee Profile is distilled from the Preference Conversation, updated by Preference Feedback, and holds any Speaker Preferences. Avoid: personal profile, personal conference profile.

**Preference Conversation**: The adaptive conversation through which the product learns an attendee's preferences before ranking. Relationships: One Preference Conversation produces the initial Attendee Profile. Avoid: questionnaire (the product rejects a fixed questionnaire).

**Preference Feedback**: A natural-language statement about the attendee's preferences, such as "too basic" or "more architecture", that applies to every later ranking. Relationships: Preference Feedback updates the Attendee Profile. Avoid: rating, session rating (session ratings are a non-goal).

**Attendance Mark**: The attendee's recorded intent for one session, with exactly one of four values: Attend, Must attend, Watch recording, or Skip. Relationships: An Attendee places at most one Attendance Mark per Session; a Must attend mark creates one Must-attend Constraint.

**Must-attend Constraint**: A fixed schedule commitment created by marking a session Must attend. Relationships: A Must-attend Constraint fixes one Session in its Time Slot; conflicting Sessions remain available as alternatives.

**Ranking**: The personal ordering of every available session within one time slot, weighing relevance, expected novelty, depth, speaker fit, prior attendance, repetition, and variety. Relationships: A Ranking covers one Time Slot and gives each Session one Explanation.

**Explanation**: The short statement of why a session fits or does not fit the attendee, which expands to relate the session to the attendee profile and conference history. Relationships: Each ranked Session has one Explanation.

**Speaker Background**: The bio the conference publishes for a speaker. Relationships: A Speaker Background describes one Speaker and is the only speaker information Speaker Fit draws on. Avoid: speaker evidence, credentials, dossier.

**Speaker Preference**: A preference the attendee states about the speakers they want to hear, such as a preference for practitioner speakers. Relationships: A Speaker Preference belongs to one Attendee Profile.

**Speaker Fit**: How well a speaker's published background matches the attendee's stated preferences; it describes the match, not the person. Relationships: A Speaker Fit relates one Speaker Background to one Attendee's Speaker Preferences and informs the Ranking and the Explanation. Avoid: credibility, credibility summary, speaker evidence, speaker rating, speaker assessment (the product assesses no speaker's competence, trustworthiness, or credibility).

**Personal Schedule**: The attendee's plan for the conference, combining their attendance marks and must-attend constraints with the session times, rooms, cancellations, and replacements of the current conference bundle. Relationships: A Personal Schedule belongs to one Attendee and one Conference.

**Personal Conference Data**: All persistent data the product holds about one attendee: the attendee profile, preference feedback, attendance marks, attendance history, and recommendation history. Relationships: Personal Conference Data lives only in browser storage; the Attendee can erase, export, and import it. Avoid: local conference data, personal conference state, personal profile.

**Conference Bundle**: The published static file carrying one conference's public schedule data, with stable session identities, a bundle version, and explicit cancellation and replacement markers. Relationships: One Conference Bundle per Conference, written by the Conference Fetcher. Avoid: schedule bundle.

**Conference Index**: The small published list of the conferences for which a conference bundle exists. Relationships: The Conference Index lists every Conference Bundle the Conference Fetcher publishes.

**Bundle Version**: The marker that identifies one publication of a conference bundle and lets the page detect schedule changes. Relationships: Each Conference Bundle carries one Bundle Version.

**Session Identity**: The stable identifier of a session that persists across republished conference bundles. Relationships: Attendance Marks reference a Session by its Session Identity, so marks stay valid across Bundle Versions.

**Conference Source**: An external origin of a conference's published schedule. Relationships: The Conference Fetcher reads each Conference Source through its own parser.

**Conference Fetcher**: The scheduled function that pulls conference data from conference sources and publishes conference bundles and the conference index. Relationships: The Conference Fetcher reads one or more Conference Sources and writes one Conference Bundle per Conference.

**Provider**: The cloud inference service, OpenAI or Anthropic, that the attendee chooses and accesses with the attendee's own key. Relationships: Every inference request goes to exactly one Provider, through the Inference Gateway. Avoid: model (when meaning the service), inference endpoint.

**Model**: The language model a provider runs, which produces structured judgments about one session against one attendee profile: scores, explanations, and profile distillation. Relationships: A Model is reached only through a Provider.

**Consent**: The attendee's explicit agreement to a cloud inference request, naming the processing purpose and the provider that processes it. Relationships: Every request through the Inference Gateway requires Consent.

**Inference Gateway**: The single path from the page to a provider, holding consent, request assembly, and validation of the model's structured output. Relationships: The Inference Gateway calls one Provider per request and sends only the current Time Slot's Sessions and an Attendee Profile summary.


## Example Dialogue

**Developer:** When the attendee opens a conference, do we load the conference bundle or the conference index first?

**Domain expert:** The conference index, so the attendee can pick a conference. Then the conference bundle for that conference. After that the preference conversation runs and produces the attendee profile.

**Developer:** The fetcher republished the bundle with a new bundle version, and one session moved rooms. Does the attendee lose their attendance mark?

**Domain expert:** No. The mark points at the session identity, which stays stable. The personal schedule shows the new room.

**Developer:** If the attendee marks a session Must attend, do we hide the other sessions in that time slot?

**Domain expert:** No. The must-attend constraint fixes that session, but the ranking still shows every session in the time slot. Conflicting sessions stay available as alternatives, each with its explanation.

**Developer:** The attendee typed "too basic" after a session. Is that a rating of the session?

**Domain expert:** No. It is preference feedback. It describes the attendee, updates the attendee profile, and affects every later ranking. We store no session ratings.

**Developer:** The explanation says a session "matches your preference for practitioner speakers". Is that a credibility rating of the speaker?

**Domain expert:** No. It is speaker fit. It relates the speaker background, the bio the conference publishes, to the attendee's speaker preference. It describes the match, not the person. The product never judges a speaker's competence, trustworthiness, or credibility.
