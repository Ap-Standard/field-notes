# Post-merge verification

Merged is a claim, not a fact. Green CI proves the change passed its checks in a clean
environment before the merge. It proves nothing about what happened after: whether the
deploy ran, whether the migration applied, whether the feature behaves against production
data. A five-check ritual closes that gap after every deploying merge.

## The failure class that creates the ritual

Deploy webhooks drop silently. The default branch goes green, the merge lands, and
production keeps serving stale code until someone hits the missing behavior. Pre-merge CI
cannot catch this class of failure, because its checks end before deployment begins. The
lesson generalizes: every handoff between systems that "always works" needs one cheap
check that proves it worked this time.

## The ritual

Run after every merge that deploys. Five checks, in order, against production:

1. **Deployment status for the exact merged commit**, read from the deployment API, not
   inferred from a dashboard or a notification.
2. **Health endpoint** returns OK and reports the version or commit you just merged.
3. **Schema check**: the live database's migration version matches what the merge
   expects. A deploy that runs old migrations against new code fails in ways CI never saw.
4. **One feature probe**: exercise the changed behavior with a real request. Not a
   test suite, one request that would fail if the change were absent.
5. **Error-tracker sweep**: no new error class in the minutes after the deploy. New noise
   after a deploy belongs to that deploy until proven otherwise.

Every check is a single API call or a single request. The discipline is running them for
the boring merges too, because the webhook does not drop on the risky ones by appointment.

## What to automate, and what not to

Checks 1 through 3 script cleanly and belong in a post-deploy job that pages on mismatch.
Keep checks 4 and 5 manual at first: automating them early hides the failure mode they
exist to catch, because a passing synthetic probe and a passing real request fail
differently. Automate them after the ritual is habit, not instead of the habit.

## Steal this

- [ ] After each deploying merge: deployment status for the exact SHA, from the API.
- [ ] Health endpoint reports the expected version.
- [ ] Live schema version matches the merge's expectation.
- [ ] One real request exercises the changed behavior.
- [ ] Error tracker shows no new class within minutes of the deploy.
- [ ] Record the verification and a timestamp on the merge itself; merged is not done.
