# Intent-Aligned Testing, Real-Boundary Acceptance, and Closure

Load this reference when planning test suites (Stage 3), writing companion tests (Stage 4), running integration and client acceptance (Stages 5-7), or closing merges (Stage 8). Repository test conventions, environment rules, and branch protections always take precedence.

## Suite-Level Test Intent and Completeness

Tests are complete when they protect the Feature's business behavior, caller contracts, and failure or recovery rules, not when they chase arbitrary line-coverage percentages.

- **Mandatory suite or file intent note:** Start every new or materially expanded test file or top-level `describe`/suite block with a concise 2-4 line note:
  1. **Business scenarios:** Which user or caller workflows and inputs this suite exercises.
  2. **Protected behavior or failure mode:** Which invariant, state transition, permission rule, or regression fails if this suite breaks.
  3. **Existing coverage gap:** Why existing suites did not already protect this behavior (for example, mocked storage hid reload bugs or a new role rule arrived).
- **Extend matching suites first:** Add cases to established suite files and reuse fixtures or parameterization when boundaries fit, avoiding cloned test files or line-by-line assertion comments.
- **Assert authoritative outcomes:** Check returned contracts, user-visible states, emitted domain events, and persisted read-backs rather than private helper calls or internal execution order.
- **Cover real failure and recovery rules:** Test validation rejections, permission denials, retry or idempotency rules, partial-failure rollback, and reload consistency when relevant to this Feature's actual contract. Skip permutations with no distinct rule or failure modes outside that contract, such as permission or payment checks on a simple UI edit.
- **Ship tests with the Feature:** Keep unit, contract, and required integration or acceptance tests in the same Feature delivery unit rather than deferring them to follow-up PRs.

## Honest Mocks, Isolation, and Stateful Chains

- **Declare real vs mocked boundaries:** In Stage 6-8 reports, name which collaborators ran for real (client, API, DB, queue, cache, auth) and which external services were stubbed. Never describe a mock-backed test run as end-to-end or real-boundary acceptance.
- **Isolate independent tests:** Scope records, files, clocks, and accounts per test run, and clean up created state in teardown on both pass and fail.
- **State preconditions for stateful chains:** When an acceptance journey intentionally spans ordered steps (such as create -> approve -> reopen), document initial preconditions, run namespaces, and step transitions. Inspect shared environments for pre-existing records before mutating state, and never replay one-time creation steps blindly against dirty fixtures.

## Verification Tiers and Exact Candidate Binding

Keep the four verification tiers distinct in all Stage 6-8 evidence reports:

1. **Unit and narrow contract checks:** Fast isolated logic, schema, and edge-case checks run in Stage 4 and after every fix.
2. **Real-boundary integration checks:** Multi-layer checks across live service, storage, or queue boundaries that confirm authoritative persistence and read-back.
3. **CI pipeline checks:** Required repository runner jobs; green CI counts as browser or device proof only when that job actually executed those real client flows.
4. **Real client or QA boundary acceptance:** Required whenever changes alter client-observable behavior, including backend schema, validation, permission, pagination, error, or timing updates with zero UI files touched. Exercise the real browser, CLI, SDK, or mobile/device runtime when client behavior is affected; skip UI harnesses for purely internal non-client changes.

Bind every check record to the exact candidate ID, `merge-base` ref, environment, command or user journey, fixture scope, real vs mocked boundaries, verdict, and log path. Reuse prior evidence after a Stage 7 fix or clean rebase only when untouched code, build inputs, config, environment, and fixtures remain verifiably identical. Rerun every check affected by the delta, and retain failed-run logs when diagnosing flakes or regressions instead of retrying silently until green.

## Commit Hygiene, Authorized Merge, and Closure States

The primary integrator owns final handoffs until each Feature reaches its authorized delivery boundary:

1. **Clean commit and PR metadata:** Describe the observable business change, key design rationale, and validation summary in commit subjects, commit bodies, and PR descriptions. Omit model names and automated-generation or tool attribution and provenance trailers from commit and PR metadata. Preserve normal Git author identity, legitimate human credit, and product model or provider identifiers in source code and APIs.
2. **Pre-merge gate check:** Confirm Stage 6/7 review sign-off, required CI and boundary evidence on the exact candidate, target-branch sync, and any main-branch auto-deploy effect.
3. **Ordinary authorized merge:** When remote push, PR, and merge actions are authorized, publish the Feature PR, satisfy required branch gates, and run the standard repository merge. Never bypass branch protections, skip required reviews or CI, or assume external merge or deploy permissions that were not granted.
4. **Independent vs coupled publication:** Merge each independent Feature once its own gates pass, even while other Features remain in implementation or review. When coupled changes across layers or repositories would expose broken intermediate behavior alone, coordinate feature flags, merge order, or release windows before publication.
5. **Target-branch read-back:** Fetch and inspect the landed target-branch commit (plus any package or submodule pins) immediately after merge; local branch state or `PR mergeable` never suffices.
6. **Release check and terminal states:** Complete any authorized in-scope live verification, close the finished Feature's PR(s), and report status precisely:
   - **`merged`:** Landed and read back on the target branch; live deployment is out of scope or not yet executed.
   - **`released`:** Merged on the target branch **and** verified live in the target runtime environment.
   - **`blocked`:** Paused at a missing external gate, environment fault, or ungranted permission, naming the blocker, owner, and immediate unblock action while unrelated work continues.

When other Features remain active, keep their named owners and next actions explicit in the handoff note, and redact secrets, tokens, cookies, and private data from artifacts.

## Optional Further Reading

- [Testified Test Case Design Skill](https://github.com/testified-oss/agent-skills/blob/dce83769bc5d45b2150c1ac2e3c731db17830b29/skills/test-case-design/SKILL.md) (optional supplement for boundary and scenario selection)
