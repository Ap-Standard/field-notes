# Case study: operating a production AI platform as a two-person team

## What it is

A multi-tenant AI platform in production since mid-2026: knowledge workflows and agent
runs for organizations, on a Python backend and a TypeScript frontend. I architect and
operate it as one half of a two-person team. The codebase, the clients, and the
infrastructure stay private. This case study shares the operating mechanisms and the
measured results, because the mechanisms are what transfer.

## Scale, and how each number is measured

- **2.5B tokens processed at a 97.8% cache hit rate.** Measured by token accounting
  instrumented at every model call site, stored per request, reconciled against provider
  billing. At a 75% hit rate the same volume would have cost roughly 2.7x more, computed
  from the cached-versus-uncached price ratio; cache behavior, not raw token count,
  decides the bill.
- **About 170 pull requests merged through the gated trunk in the platform's first three
  months.** Counted from the default branch's merge history at the time of writing.
- **Test suite grown from about 780 to about 1,500 backend tests plus about 250 frontend
  tests**, under a coverage ratchet in CI set just below the measured baseline, so
  coverage can only rise. Counted by the CI runners.
- **42 numbered, forward-only database migrations applied in production.** Counted from
  the migration chain.
- **7 required status checks on the default branch.** Each gate ran as informational
  first and graduated to required only after proving a stable signal.

## The operating model

Seven mechanisms carry the system. Each earns its place by a failure it prevents.

1. **Gated trunk.** No direct pushes to the default branch, for anyone, including the
   admin. Required checks are the only path in.
2. **Two-seat AI review.** One AI reviewer inside the authoring loop, plus a standing
   second seat from a different vendor filing severity-tagged findings on every PR, under
   a written degrade policy: if the seat is down or rate-limited, skip it and tag the PR,
   never block the loop. Detail: [two-seat AI code review](../notes/two-seat-ai-review.md).
3. **Dark launches.** Risky features merge disabled behind an environment flag plus a
   per-user entitlement, get enabled for one operator account, get verified live, then
   widen. Merge risk and launch risk stay two separate decisions.
4. **Migration discipline.** Additive first; constraints added invalid-then-validated so
   tables stay unlocked; check-then-insert races replaced with atomic upserts; every new
   table ships its access policies in the same migration.
5. **Post-merge verification.** Merged means verified live: deployment status for the
   exact commit, health endpoint, schema version, one feature probe, error-tracker sweep.
   Detail: [post-merge verification](../notes/post-merge-verification.md).
6. **A written build log.** Every working session ends with a committed log entry: what
   shipped, what broke, what the next session must know. The log is the program memory,
   and it is what makes handoffs between sessions verifiable instead of vibes.
7. **Reliability sweeps.** The error-tracker board is driven to zero on a named cadence.
   Every item is root-caused or explicitly accepted on the record; silence is not a state.

The same model scales up: a full architecture audit was sequenced into a remediation
program of file-disjoint PRs in dependency-ordered waves, executed wave by wave through
the same gated loop as feature work.

## A failure class worth naming

The class: a resource budget that no single component owns. Services that were each
individually correct held connection ceilings that summed past what the shared database
pooler actually granted, and the symptom (intermittent failures to obtain connections)
surfaced only under concurrent load. The fix: explicit per-service caps, sized so the sum
stays under the shared limit with headroom, recorded in configuration rather than tribal
memory. The verification: a load re-test, then an error-tracker sweep to zero on that
error class. The standing review question this failure earned: who owns the sum?

## What the discipline costs, and what it buys

The second review seat adds wall-clock and model spend to every PR; the cost scales with
diff size and model pricing, so instrument it at the call site and read your own number.
What it bought here, in its first week: a confirmed must-fix defect in input-handling code
that the authoring loop had passed, fixed and verified live within a day through the same
gated loop. The seat also files false positives. Those get waived in a comment with
written reasoning and stay on the record, because visible disagreement with your own
tooling is what makes the green checks believable.

## Limitations, and what does not transfer

- This is a two-person operation. Machine gates carry more of the review weight than they
  should at team scale. With a team, human review returns as the primary gate and AI
  seats become advisory.
- Per-user entitlement toggles for dark launches stop scaling once the operator can no
  longer hold the grant list in their head. Past that point you need cohorting and
  progressive rollout tooling.
- The single-operator reliability sweep works because one person holds the whole system
  in their head. Teams need ownership boundaries and paging, not a hero with a board.
- The build-log ritual is the part I would keep at any scale.
