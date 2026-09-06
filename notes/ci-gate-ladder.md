# The CI gate ladder

A new check earns authority in rungs. It runs advisory first, becomes required once its
signal is stable, and only then gets to stop a merge, from a job the consuming repository
owns. Skip a rung and the failure is predictable: the check blocks on its own bug, the team
routes around it, and within a month someone deletes it. The deletion will be correct.

## The failure class that creates the ladder

A gate that can fail a check on day one carries every bug it has into the merge path. A
timeout blocks. A malformed reply blocks. A missing secret on a fork blocks. None of those
is evidence about the change under review, and every one of them teaches the team that the
gate is an obstacle, not a measurement. The lesson generalizes past AI review: any check
that has not yet published its false-block rate has no business holding a merge.

## The three rungs

**Rung one: advisory.** The check runs on every pull request, posts what it found, and the
job passes no matter what. The point of this rung is the record it builds: how often the
check ran, what it said, how often it was wrong, and what it cost. Nothing about a merge
changes. A pull request with a loud advisory comment merges exactly as it did before.

**Rung two: required to run.** Branch protection requires the job to complete before a
merge, and the job still passes on every outcome. What this buys is presence: no pull
request merges without the check having looked, or having reported that it could not look.
"Did not review" and "found nothing" must render as two different states here, or rung two
hides the exact failure it exists to expose.

**Rung three: enforced, from the consumer's job.** The check publishes a decision. The
repository that lives with the consequences reads that decision in its own workflow and
fails its own job. That job, not the check, goes into branch protection. The kill switch
lives on the consumer's side as well, as a repository variable, so a bad night ends with a
setting change and not a release. The separation is the mechanism: the gate can be wrong
about a diff without being able to stop anyone, and the choice to enforce sits in a file its
owners control.

## When to climb

Climb on evidence, and name the evidence before you climb. Three questions, each with a
number:

1. How often did the check run, over how many pull requests, in what window?
2. How often would it have blocked something that should have merged? If that figure came
   from a benchmark, say what the benchmark cannot tell you.
3. How many findings did a human waive with written reasoning? Zero waivers on a check that
   has run fifty times means either a perfect tool or a team that stopped reading.

If any of the three has no number, the rung stays where it is.

## In public, on this portfolio

- **Rung one, twoseat.** twoseat reviews its own pull requests in advisory mode from
  `.github/workflows/ai-review.yml`. Nine merged pull requests through the v0.1.0 release
  carried an ai-review run (method: merged pull requests on `main` matched against the
  ai-review workflow's run list). Seven of them reached a seat: six came back with no
  findings and one ended `not-reviewed` on an unreadable reply; the other two merged
  before the seat existed and carry the pre-seat comment (method: the latest twoseat
  comment on each). Both counts as of 2026-09-06. A run is not a review, which is why the
  two counts are stated apart. The review job is not a required check. Requiring it would
  only guarantee that a comment exists before merge, which is rung two's exact value and a
  decision still open.
- **Rung one, flightdeck.** flightdeck consumes `Ap-Standard/twoseat@v0.1.0` from its own
  `.github/workflows/ai-review.yml`, comment-only. A second repository consuming the gate is
  what makes the "reviewed by" claim more than self-reference.
- **Rung three, not yet climbed.** twoseat publishes a `decision` output and a
  `blocking-findings` count on every run, and its policy document carries the workflow
  snippet a consumer would add to fail its own job. Neither repository has added it. The
  reason is the benchmark's own limit: the false-block table reads 0.0% at every confidence
  threshold because the seat reported no P1 on any of the 15 eligible synthetic cases in
  twoseat's published report (`bench/results/REPORT.md` at v0.1.0), which is one
  measurement printed three times, and the seat's record on live pull requests is short:
  one reported finding, a P2 on a 44-file pull request, read from twoseat issue #12 on
  2026-09-06, tracked in the open at twoseat issue #12. Enforcing a threshold the
  evidence cannot yet discriminate would be enforcement of an unmeasured thing. The enforce
  step is the v0.2 rung.

## What a ladder costs

Advisory runs spend money and produce comments nobody is required to read. The instrument
that makes the cost bearable is the same one that justifies the climb: a per-run cost figure
in the comment, beside the finding count. When the record shows real findings, waived
findings, and a cost you can defend, rung two is a settings change and rung three is a
ten-line job.

## Steal this

- [ ] Ship every new check advisory. The job passes on every outcome, including a crash.
- [ ] Render "did not run" and "found nothing" as different states in the comment, the
      outputs, and the labels.
- [ ] Publish the run count, the finding count, the waiver count, and the cost, with the
      window and the date.
- [ ] Require the job to run only after the record exists.
- [ ] Enforce from the consumer's workflow, reading a published decision. Put that job in
      branch protection, never the check itself.
- [ ] Put the kill switch on the consumer's side, as a variable, so a bad night needs no
      release.
- [ ] Write down which rung each check stands on and what number moves it.
