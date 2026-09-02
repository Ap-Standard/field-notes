# Two-seat AI code review

Two AI reviewers from two different vendors hold standing review seats on every pull
request. One reviews inside the authoring loop. The second is a different model with
different failure modes, run against the same diff, filing findings tagged by severity
with a confidence level. A finding is closed only by disposition: fixed, or waived in a
comment with written reasoning.

## Why two seats, and why different vendors

A model reviewing code written in its own authoring loop shares that loop's blind spots:
same training, same prompt context, same failure modes. A second seat from a different
vendor decorrelates the errors. The design expects disagreement between the seats, and the
disagreement is the product: findings that survive both seats deserve attention, and
findings one seat raises alone get a human disposition on the record.

## The degrade policy, written before the first outage

A review gate and an authorization control need opposite defaults. Authorization fails
closed: no identity, no access. A review seat fails open: if the model is down,
rate-limited, or over budget, skip the seat, tag the PR as unreviewed by that seat, and
never block the loop. A team that lets a vendor outage freeze its merge queue will delete
the gate, and the deletion will be correct. Write the degrade policy the day you add the
seat.

## What it costs

Cost scales with diff size and model pricing. Instrument tokens per review at the call
site and post the number in the PR comment, so the cost sits next to the value it buys.

## Evidence from production

Running this pattern on a production platform, the second seat produced a confirmed
must-fix defect in input-handling code within its first week, one the authoring loop had
passed; the fix merged and verified live within a day through the same gated loop it
guards. The same seat files false positives, which get waived with written reasoning and
left on the record. Both halves matter. A gate with no waived findings has either a
perfect tool or a team that stopped reading the findings.

## Steal this

- [ ] Pick a second model from a different vendor than your authoring model.
- [ ] Define severity tags, and decide what blocks (high-confidence critical findings)
      versus what only comments.
- [ ] Write the degrade policy before the first outage: skip the seat, tag the PR,
      never block the loop.
- [ ] Require written disposition of every finding: fixed, or waived with reasoning.
- [ ] Label and count gate bypasses; publish the count where the team can see it.
- [ ] Keep a fixed set of seeded diffs with known defects as a seat-acceptance benchmark,
      and re-run it whenever a seat's model version changes.

I am building a runnable, measured implementation of this pattern in the open at
[twoseat](https://github.com/Ap-Standard/twoseat).
