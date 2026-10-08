# personal-conference-curator

Builds a personal conference plan on the attendee's own device.

Conference schedules list titles, abstracts, speakers, tracks, and difficulty. They know nothing about the attendee. This product learns an attendee's interests, expertise, and goals through a short conversation, ranks every session in each time slot, explains each ranking, and adapts as preferences change during the event. The attendee can ask "What next?" between sessions.

Personal data never leaves the device. No server stores a profile, a ranking, or an attendance mark.

## Shape

Two deployable units share one data contract:

- **Static browser application.** Runs every attendee-facing behavior in the browser. Model calls go to the attendee's own OpenAI or Anthropic key through a single inference gateway.
- **Scheduled fetcher.** A Lambda function that pulls public conference schedules and publishes them as static bundles.

The conference bundle is the contract between them. No runtime backend exists.

## Status

Briefs and architecture decisions are written. No TypeScript source exists yet, and the lifecycle verbs in `scripts/stack.sh` are not yet bound.

## Documentation

| Document | Purpose |
|----------|---------|
| [`docs/prd.md`](docs/prd.md) | Goals, non-goals, requirements |
| [`docs/system-design.md`](docs/system-design.md) | Module map, contracts, dependency policy |
| [`docs/adr/`](docs/adr/) | Architecture decisions |
| [`docs/ubiquitous-language.md`](docs/ubiquitous-language.md) | Domain vocabulary |
| [`CLAUDE.md`](CLAUDE.md) | Agent pipeline and contributor workflow |

## Development

```bash
scripts/gate.sh verify   # run the full quality gate: deps, format, lint, test, build
scripts/gate.sh list     # print the lifecycle verbs
```
