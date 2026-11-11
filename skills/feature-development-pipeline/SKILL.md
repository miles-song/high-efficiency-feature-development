---
name: feature-development-pipeline
description: Coordinate substantial cross-boundary features from user outcome and contracts through owned parallel execution, integrated review, real-boundary acceptance, and merge-ready evidence. Use when behavior spans collaborating components, services, or persisted state; skip for isolated, low-impact fixes.
---

# Feature Development Pipeline

Use this workflow for substantial features spanning collaborating components, services, or persisted state. Skip ceremony for isolated, low-impact fixes, which should go straight to targeted change and verification.

## Local Authority and Portability

Inspect the user's instructions and the repository's `AGENTS.md`, design documents, architecture, existing implementation, tests, runtime rules, permissions, and contribution or merge gates first. Host rules govern stack choices, environments, approvals, and required artifacts. Record decisions as settled only when confirmed; resolve contract-affecting ambiguities from existing sources or ask before starting dependent work.

## Workflow

1. **Define outcome and acceptance chain.** State the user outcome, entry point, primary flow, and relevant failure or recovery paths. Preserve any user-specified visible milestone order in the dependency and acceptance plan while allowing independent prerequisite or parallel work to proceed. For each consequential action, trace behavior across participating boundaries to the authoritative effect and any read-back or reload behavior. Convert this chain into observable acceptance checks mapped to concrete evidence.
2. **Settle contracts before fan-out.** Inspect current code and designs to lock down changed interfaces, data shapes, state ownership, persistence, retry, and error semantics. Update only the design artifacts required by the repository. Do not invent unresolved user-visible or cross-boundary behavior.
3. **Assign one integrator and explicit ownership.** One integrator owns scope, interface alignment, dependency order, integration, acceptance evidence, and closure. Split parallel work packages only across stable interfaces and non-overlapping file ownership; sequence shared files or unsettled contracts under a single owner at a time. Work packages are execution units, not automatic pull-request boundaries; before fan-out, identify coherent delivery units from host rules, ownership, and dependencies (for example, service and view layers when each holds complete operable responsibility, keeping companion tests, config, and docs with their unit per host policy) without forcing a PR per package or splitting helpers to hit a count. See [work-package orchestration](references/work-package-orchestration.md).
4. **Schedule by bottleneck and reuse context.** Dispatch only useful, ready work within actual integration, review, and runtime capacity—never fill concurrency slots just to keep workers busy. Apply these priorities as adaptive preferences based on actual bottlenecks, readiness, risk, and capacity rather than a rigid order that blocks useful work:

   - Critical-path and unblocking work
   - Integration defects
   - Acceptance and closure of integrated work
   - Newly ready packages

   Reuse workers that already hold domain or diff context instead of running duplicate audits or frequent status polling. Give any shared runtime environment a single owner for lifecycle, configuration, seed data, and recovery while allowing isolated tests to run concurrently.
5. **Pipeline validation and integrate promptly.** Run focused package checks as code lands, start CI and acceptance preparation as soon as dependencies allow, and integrate completed packages promptly so cross-boundary behavior is exercised early.
6. **Run one integrated Feature Review.** Once the combined candidate is stable and coherent, give a single reviewer the user journey, acceptance chain, contracts, full feature diff, and available test/runtime evidence—overlapping review with CI, runtime checks, and independent acceptance testing when practical. Evaluate correctness and concrete maintainability or code necessity concerns (unnecessary code or dependencies, duplication, excessive abstraction, tight coupling, and boundary violations) without preference-only style churn. Request consolidated findings stating trigger and user, system, or maintenance impact rather than per-file nit loops. Keep the same reviewer to verify fix deltas and integration impact; restart full review only after a material scope or behavior change, subject to mandatory local gates.
7. **Prove across real boundaries and close.** Verify the final candidate across the real participating boundaries required by the affected behavior and user journey: backend or contract changes that affect a client journey still require client-boundary verification even without UI code edits, while genuinely non-UI behavior requires no browser or device harness. Keep unit, integration, CI, browser, and device evidence distinct—mocks do not prove integration. Confirm the reviewed, tested, and delivered diff match; when remote delivery is authorized, reuse settled authorization without inferring extra permissions or bypassing mandatory gates to complete commit/push/PR, required review and CI, merge and target-branch read-back, and in-scope deployment verification (reporting external deployment blocks accurately). Report completed checks, actual evidence, and any material gaps without claiming unrun checks passed. See [integration and acceptance](references/integration-and-acceptance.md).

## Continuous Refinement

Adjust workflow habits only through narrow changes justified by observed bottlenecks or escaped defects. Avoid accumulating speculative checklists, extra gates, or parallel tracking artefacts.

## Optional External References

- [Sentry Code Review Skill](https://github.com/getsentry/skills/blob/c2f99a5b04b4cd992ec3022d7c2c3e23e938d241/skills/code-review/SKILL.md) (supplementary review heuristics)
- [Testified Test Case Design Skill](https://github.com/testified-oss/agent-skills/blob/dce83769bc5d45b2150c1ac2e3c731db17830b29/skills/test-case-design/SKILL.md) (supplementary test-design patterns)
