# field-notes

Production mechanisms from operating an AI platform end to end, and from enterprise
program delivery. Each note is one to two pages, mechanism-first, and ends with a
checklist you can apply the same day. Steal them, with attribution.

Claude drafts these notes from my working notes. Every claim, number, and sanitization
decision is mine to defend.

## Contents

| Note | Implemented in |
| --- | --- |
| Case study: [Operating a production AI platform as a one-person team](case-study/production-ai-platform.md) | A private platform. Mechanisms only, per [decision 0003](https://github.com/Ap-Standard/Ap-Standard/blob/main/docs/decisions/0003-sanitization-and-disclosure-policy.md) |
| Note: [Two-seat AI code review](notes/two-seat-ai-review.md) | [twoseat v0.1.0](https://github.com/Ap-Standard/twoseat/releases/tag/v0.1.0): one seat, benchmarked before it was trusted |
| Note: [Post-merge verification](notes/post-merge-verification.md) | The `## Verified` section in twoseat release notes, counted nightly by [flightdeck](https://ap-standard.github.io/flightdeck/) |
| Note: [The CI gate ladder](notes/ci-gate-ladder.md) | twoseat's advisory [ai-review.yml](https://github.com/Ap-Standard/twoseat/blob/main/.github/workflows/ai-review.yml): nine merged pull requests ran it and seven reached a seat, as of 2026-09-06 (methods in the note); flightdeck's [ai-review.yml](https://github.com/Ap-Standard/flightdeck/blob/main/.github/workflows/ai-review.yml) at `@v0.1.0`, comment-only |

More notes land through reviewed pull requests; the queue lives in this repo's issues.

## What you will not find here

No client names, no employer stories, no infrastructure identifiers, and no numbers
without a stated measurement method or cited public provenance. The disclosure policy that
governs every file:
[decision 0003](https://github.com/Ap-Standard/Ap-Standard/blob/main/docs/decisions/0003-sanitization-and-disclosure-policy.md).
The portfolio program these notes belong to:
[charter](https://github.com/Ap-Standard/Ap-Standard/blob/main/docs/charter.md).

## License

Prose: [CC BY 4.0](LICENSE). Code snippets embedded in notes: additionally usable under
[Apache-2.0](LICENSE-CODE).
