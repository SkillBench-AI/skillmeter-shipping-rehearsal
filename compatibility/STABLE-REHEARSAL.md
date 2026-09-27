# Stable channel rehearsal

Target: `SkillBench-AI/skillmeter-shipping-rehearsal`. Synthetic data only.
This extends the existing shipping rehearsal; it does not replace its publisher
setup or authorize changes to the real marketplace. Keep all existing review,
queue and environment protections.

Policy and tests are copied unchanged from Codex marketplace PR103 commit
`e9f3acd53b9b78002f6cf446fb12b7764d0c19a9`. The promotion workflow adds only
sandbox repository/fixture guards. Refresh these copies if the reviewed source
changes. The new PR check executes policy tests; its success is not proof of
GitHub ref enforcement. The optional local Codex install test is skipped in
hosted CI because no Codex CLI is provisioned.

## Prerequisites

- Independent review and normal queue landing of this candidate.
- Existing sandbox publisher setup completed by its owner. App installation must
  remain limited to this repository; no production or developer credentials.
- `release-publisher` environment contains its App ID and key, requires independent
  approval, prevents self-review/admin bypass and allows only `main`.
- Add the two stable rulesets rendered by
  `node .github/scripts/stable-channel.cjs rules APP_ID`. Use the real sandbox
  App ID. Preserve existing main/queue/tag rules, including immutable tags.
- Produce a fresh successful synthetic release through the existing Release
  workflow. Do not reuse the old waiting demonstration run as acceptance.

## Hosted cases

Record the starting ref, exact candidate, run URL, expected outcome, observed
outcome and ending ref for each case. Stop on any unexpected result. Never put
credentials, private data or real telemetry in receipts.

| Case | Action | Required result |
|---|---|---|
| First promotion | Dispatch Promote stable with published tag/SHA, successful release run, reviewed synthetic evidence link and expected_stable=absent | Environment waits for independent approval; stable becomes exactly that release SHA |
| Unreleased main | Land another reviewed synthetic candidate; do not promote | stable remains unchanged |
| Missing/failed/mismatched evidence | Dispatch using missing release, failed run or a different tested SHA | Failure before ref mutation; stable unchanged |
| Stale old-ref input | Dispatch a valid release with the wrong expected_stable | Failure before mutation |
| Wrong branch | Attempt promotion dispatch from a non-main branch | Promotion job cannot execute |
| Older release | After promoting a greater version, attempt the older release | Failure; stable never moves backward |
| Concurrent promotion | Queue two valid promotions with the same expected old SHA | Serialized execution; at most one advances, the other holds on changed stable |
| Same-release retry | Dispatch the current release with its current stable SHA | No ref mutation; current SHA observed |
| Ordinary-user update/delete | Attempt each using an ordinary collaborator identity in this sandbox | GitHub denies both; stable unchanged |
| Publisher force/delete | Use the sandbox publisher under the approved test procedure to attempt a backward force update and deletion | GitHub denies both, despite App creation/update bypass |

Do not attempt destructive-denial cases against production. Review identity and
rule applicability before each sandbox attempt. A test unexpectedly permitted
is a failed protection boundary, not a reason to continue. Preserve the evidence
and repair through the established review process.

The test table is not an automated trigger. Publisher, review and runnable release
prerequisites must be completed first. No scheduled workflow is introduced.

## Local checks

```sh
GITHUB_REPOSITORY=SkillBench-AI/skillmeter-shipping-rehearsal GITHUB_SHA="$(git rev-parse HEAD)" node .github/scripts/rehearsal-fixture.cjs guard
node --test plugins/skillmeter/test/stable-channel.test.js
CODEX_MARKETPLACE_SMOKE=1 node --test plugins/skillmeter/test/stable-channel.test.js
```

After hosted acceptance, activate the real repository separately. Existing-user
state migration and desktop hook trust remain separate checks; a stable ref or
successful synthetic release does not establish either.
