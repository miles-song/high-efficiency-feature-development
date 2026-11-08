# Work-Package Orchestration

Use this reference when a feature spans distinct implementation packages or parallel contributors. One feature integrator owns the scope, shared interfaces, dependency map, integration, final evidence, and closure. Packages are internal execution units, not automatic pull-request boundaries. Obey all host repository rules, architecture constraints, and merge gates; skip orchestration ceremony for small, isolated fixes.

## Dependency and Ownership Map

Before launching parallel work, inspect existing code and contracts to map the smallest set of packages that deliver the complete user outcome:

- **Explicit file ownership:** Assign non-overlapping owned files or directories to each active package. If a shared file must change, assign a single owner or sequence edits through a deliberate handoff.
- **Stable interfaces:** Settle cross-package data shapes, signatures, error semantics, and state transitions up front. Do not let individual packages invent local contract substitutes.
- **Single shared-runtime owner:** When work touches a shared runtime, database, or environment, assign one owner for lifecycle, configuration, seed data, and recovery. Unrelated isolated unit or contract tests may still run concurrently.

## Contextual Concurrency and Scheduling

Match concurrency to actual reviewer, integration, and runtime capacity. Never fill worker slots just to use available concurrency, and avoid rigid state-machine tracking or fixed role counts.

- **Schedule only ready work:** Launch a package when what it actually consumes is settled: a usable interface or prerequisite, clear file ownership, and defined acceptance criteria. A predecessor package does not need to be fully finished if the consumed contract or artifact is already stable. If a package becomes blocked, pause it and record the specific dependency and concrete unblock condition rather than guessing.
- **Adapt scheduling to bottlenecks and risk:** Weigh active bottlenecks, integration risk, readiness, and available capacity rather than following a rigid queue. Generally favor critical-path or unblocking work, active integration defects, and acceptance or closure tasks over starting new packages, but never let a blocked high-priority item stall unrelated ready work when capacity is available.
- **Reuse context-carrying workers:** Route follow-up tasks, adjacent packages, and defect fixes to workers that already hold the relevant domain and codebase context. Avoid duplicate discovery audits and frequent status polling.
- **Pipeline verification and integrate when checks permit:** Start test harnesses, fixtures, and acceptance preparation as soon as contracts and prerequisites allow. Integrate completed packages into the combined candidate promptly once required package-level checks pass and the integration target is ready, running focused boundary checks so defects surface early.

## Task Brief

Provide each contributor a compact brief containing only the context required to execute and verify the package:

```text
Objective: package deliverable and how it advances the end-to-end user outcome
User flow: relevant trigger, expected behavior, and failure/recovery path
Owned files: exact files or directories this package is permitted to modify
Interface/dependencies: consumed contracts, upstream prerequisites, and handoff points
Verification: required unit/integration checks, fixtures, and runtime constraints
Result handoff: modified files/applicable revision, executed checks with outcomes, remaining integration obligations, or blocking dependency + concrete unblock condition
```

## Integrated Review Synchronization

Avoid fragmented package-by-package review gates. Use an early contract consultation only when a high-risk interface decision would otherwise cause expensive rework, and reuse that same reviewer later.

Once all packages are integrated into a stable combined candidate, conduct one complete Feature Review. When dependencies and runtime ownership permit, overlap this review with full CI runs and independent runtime acceptance execution, explicitly noting which verification evidence is already complete and which overlapping checks remain pending. Give the reviewer a concise context packet: the end-to-end user journey, locked contracts, unified diff, and distinct verification evidence (keeping unit, integration, CI, and any applicable UI or device results separate, and noting any pending checks).

The reviewer returns a single consolidated list of findings backed by concrete triggers and user, system, or maintenance/code-necessity impact, omitting purely stylistic nits. The same reviewer verifies the subsequent fix delta and its integration impact, while the integrator closes any previously pending CI or runtime evidence; restart a full review only if fixes introduce material scope or behavioral changes, or when local mandatory gates require it.

Optional supplementary reference: [Sentry's code-review skill](https://github.com/getsentry/skills/blob/c2f99a5b04b4cd992ec3022d7c2c3e23e938d241/skills/code-review/SKILL.md) offers general review heuristics when local guidelines leave room, without replacing the single integrated review model above.
