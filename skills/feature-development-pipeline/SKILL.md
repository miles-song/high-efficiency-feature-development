---
name: "feature-development-pipeline"
description: "Deliver cohesive domain modules incrementally with bounded PRs, module-level tests and review, followed by integration and end-to-end acceptance in single-agent or multi-agent mode."
---

# Feature Development Pipeline

This repository holds a high bar for code taste: modules keep one clear responsibility, control flow stays direct, abstractions and dependencies stay minimal, and tests verify real business behavior. Every agent delivering a Feature owns its correctness, reviewability, intent-aligned tests, and closure up to the authorized delivery boundary. An author's job does not end when coding stops or a pull request opens; the author remains accountable through review fixes, boundary verification, and landed merge confirmation. Responsibility still respects repository permissions: leaf contributors own their slices and fixes without relaxing independent review, while one primary integrator owns cross-slice integration and authorized target-branch merge.

Sustaining that standard at delivery speed creates four practical tensions. First, a real Feature often crosses UI, API, domain, and storage layers, yet bundling extra outcomes quickly makes diffs hard to review. Second, multi-agent parallelism stalls or collides whenever interfaces stay fluid or two writers edit the same shared files. Third, test suites lose value when they chase line counts or mirror private helpers instead of protecting user-visible behavior. Fourth, teams lose throughput when agents start new coding slices faster than they finish review, acceptance, and merge work.

This workflow resolves those tensions through strict Feature boundaries, explicit single-agent and multi-agent operating modes, an eight-stage lifecycle, honest diff accounting, and verified closure. Host `AGENTS.md` instructions, architecture documents, permission boundaries, test conventions, and branch protections always take precedence. Reuse confirmed designs and repository-required artifacts rather than creating parallel paperwork or reopening settled product decisions.

## Core Principles and Unit Boundaries

A **Feature** is a small, cohesive new or changed module within one domain. It completes one clear responsibility and a related set of user operations or caller behavior. A picker, an editor, or a domain query service can each be a Feature when it has a real consumer contract, complete behavior, and its own tests. A button, helper, state class, or test file does not become a Feature merely because it can be coded separately. An entire page, workflow, or product initiative is not automatically one Feature.

Choose this boundary **before implementation or delegation**. State the module's domain, entry or caller, complete responsibility, consumed and exposed contracts, companion tests, and intended merge order. Keep only the implementation and supporting changes needed for that module in its PR. It may cross UI, API, domain logic, and persistence layers when they serve the same responsibility; related modules connect through settled contracts rather than accumulating in a project-wide merge candidate. Do not implement the whole workflow first and plan to split it afterward.

1. **Cohesive domain module:** Ship a complete, bounded responsibility with its companion tests, without bundling adjacent modules or creating a PR for each implementation fragment.
2. **Reviewable scope:** Keep core decisions in a small, explicit file set. Set the PR boundary before coding, and rescope when it grows beyond the repository's review policy.
3. **Intent-aligned testing:** Give the module its own unit, component, service, or contract tests as appropriate. Test its business scenarios and failure rules; a later page-level test does not replace these tests.
4. **Incremental closure:** Review and merge ready modules in dependency order. After they land, test their assembled interactions and complete end-to-end acceptance for the affected user flow. Do not hold every module until the entire initiative is finished.

Keep the four execution units distinct:

- **Feature (delivery unit):** One cohesive domain module that satisfies its own behavior, tests, review, and merge gates.
- **Work package:** One internal implementation slice assigned to a single author; never open a separate PR per work package.
- **PR:** The repository delivery vehicle for a Feature, or coordinated repository PRs when one Feature spans repositories.
- **Checkpoint (internal only):** A commit or snapshot within an already bounded module. It neither creates another PR nor permits an oversized final PR.

## Single-Agent and Multi-Agent Execution Modes

Choose the execution mode at Stage 3 based on contract stability, file overlap, and host delegation rules.

### Single-Agent Mode

Use single-agent mode by default when edits are small or local, share core files, require step-by-step code discovery, or lack settled internal interfaces.

- One agent acts sequentially as planner, implementer, and integrator across Stages 1-5, 7, and 8.
- Run read-only searches or non-conflicting build, lint, and test commands in parallel only when ports, databases, and output paths stay disjoint.
- Low-risk localized fixes may compress Stages 1-3 into brief inline notes while preserving candidate freeze, required review, boundary checks, and merge read-back.
- Never relabel the implementer's own pass as an independent review; invoke the required review agent or human reviewer at Stage 6 and reuse them at Stage 7. When rules require no independent reviewer and delegation is unwarranted for a tiny low-risk edit, run a structured check labeled `self-review (no independent reviewer invoked)`.

### Multi-Agent Mode and Parallel Execution

Switch to multi-agent mode only when consumed interfaces are settled, at least two work packages or Features own disjoint paths, and host rules permit delegation.

- **Hierarchy:** One **primary integrator** owns Stages 1-3, Stage 5 integration, Stage 6 coordination, and Stage 8 closure; dispatched contributors are non-recursive **leaf workers**.
- **Parallel build waves:** Once schemas, state rules, and error contracts are fixed, producer leaves, consumer leaves, docs updates, and QA harnesses run concurrently on disjoint paths or worktrees.
- **Shared files and runtimes:** Keep shared routers, registries, or migration chains with the integrator or hand them off sequentially (`Package A -> integrator check -> Package B`).
- **Runtime ownership vs reservations:** Assign **one shared-runtime lifecycle owner** for shared servers or databases, and reserve narrow exclusive slots only for ports, browsers, devices, destructive DB resets, or benchmarks.
- **Overlapping verification and closure:** Frozen candidates run integrated review and CI/boundary checks in parallel while independent Features continue building, and ready Features merge without waiting on unrelated work.
- **Capacity discipline:** Honor user-requested concurrency within host limits, and reserve finishing capacity to drain Stages 5-8 before starting extra implementation slices. See [work-package orchestration](references/work-package-orchestration.md).

## Per-Feature Lifecycle (Stages 1-8)

Advance each Feature through Stages 1-8 on its own readiness; these stages govern each Feature rather than acting as a project-wide barrier.

| Stage | Entry / Input | Concrete Actions | Minimal Output / Evidence | Advance / Return Condition |
|---|---|---|---|---|
| **1. Behavior and acceptance definition** | User request, host rules, confirmed product or design docs | Identify the small domain module, its user entry or caller, complete operation, failure or recovery rules, and authoritative result. Separate module checks from later cross-module journeys. Preserve user-ordered visible milestones. | Module behavior statement and its testable acceptance chain | **Advance:** The responsibility is complete and bounded. **Stop/Ask:** Required behavior is ambiguous and unresolvable from repo sources. |
| **2. Code, design, and contract inspection** | Stage 1 module boundary, current codebase, existing schemas and tests | Inspect the module and its actual callers, data shapes, state transitions, errors, and merge/runtime rules. Settle consumed and exposed contracts and dependency order; update only repository-required design artifacts. | Domain and contract boundaries, existing or planned consumers, and required artifact updates | **Advance:** The module can be implemented and tested against settled contracts. **Return to 1:** Inspection exposes contradictory behavior assumptions. |
| **3. Scope, mode, and test-intent planning** | Settled contracts, candidate file map | Name the intended module PR and its integrator before coding. Choose execution mode, bound core files and expected implementation churn, assign disjoint paths, and plan module tests plus post-merge integration. Split adjacent modules by complete responsibilities when scope is wide. | Module/PR boundary, ownership and dependency map, test-intent plan, and merge order | **Advance:** The planned PR is cohesive and reviewable. **Rescope before 4:** It bundles modules, fragments helpers into PRs, or exceeds host review policy. |
| **4. Implementation and focused checks** | Stage 3 plan and settled prerequisites | Implement production changes and companion tests together. Keep localized fixes inside existing large files tight without forcing file-split churn. Run focused unit and narrow contract checks on touched paths. | Clean working diff or package checkpoint, suite-level intent notes, passing focused check logs | **Advance:** Assigned scope and focused checks pass. **Return to 2/3:** Contract gap appears or changes outgrow the Stage 3 review boundary. |
| **5. Integration and exact candidate freeze** | Completed packages for this module, current target branch | Integrate only this module's packages and shared-file handoffs. Update against the target branch if needed, freeze its exact candidate, and disclose every changed path and `merge-base` `added + deleted` lines by role. | Exact module candidate, full path disclosure, passing build and focused checks | **Advance:** Candidate builds and matches its planned PR boundary. **Return to 3/4:** It has accumulated another module or fails integration. |
| **6. Module Feature Review and merge checks** | Frozen module candidate, contracts, diff summary, test logs | Obtain one Feature Review of the complete module: behavior, domain boundaries, necessity, contracts, and test intent. Run module unit/component/service/contract tests and required CI. Verify affected existing integrations or client paths when the module changes them; do not require a future unassembled workflow as this module's merge gate. | Module review verdict and distinct test/CI evidence bound to its candidate | **Advance:** Module review and mandatory merge checks pass. **Go to 7:** Findings or failures remain. **Return to 2/3:** Contract or module boundaries are broken. |
| **7. Minimal finding fixes and delta recheck** | Stage 6 findings or failed check logs | Route minimal fixes to the original author or integrator, freeze the updated candidate, rerun affected checks and touched integration paths, and return the exact delta to the **same reviewer** while reusing valid unchanged evidence. | Delta diff, new candidate ID, rerun check logs, same-reviewer delta sign-off | **Advance:** Focused fixes preserving or restoring approved contracts and intended behavior pass affected checks, same-reviewer delta sign-off, and host review gates. **Return to 6 (full):** Fixes materially alter approved Feature scope, public contracts, intended behavior, or core risk invariants. |
| **8. Incremental merge and assembled acceptance** | Passed module Stage 6/7 gates, verified permissions and dependency order | Write business-focused commit/PR metadata without model/tool attribution. Publish and ordinarily merge the module when authorized, then read back the target commit. Test cross-module interactions as dependencies land; once the affected flow is assembled, run its full integration/end-to-end journey and authorized live read-back. Fix integration defects within the responsible module. | Verified module merge; separately recorded assembled integration/E2E and release results with owners for remaining work | **Module complete:** Own merge boundary is verified. **Flow complete:** Assembled acceptance and in-scope release pass. Keep these states distinct; do not hold ready modules behind unrelated work or call a module merge full-flow completion. |

**Stop and return rules:** Pause affected work immediately when destructive or external authorization is missing, user requirements conflict, or live side effects cannot be verified. Never treat author `STOP`, green unit tests, or an open pull request as Feature completion.

## Reviewability and Diff Accounting

Measure review scope from `git merge-base <target> <candidate>` using `added + deleted` lines, never net delta or latest-commit diffs. Disclose every changed path across five categories:

1. **Core decision files:** Domain rules, state transitions, auth/permissions, billing/payments, concurrency, persistence schemas/queries, and external or cross-service contracts.
2. **Supporting handwritten production code:** Framework wiring, routes, transport mapping, UI bindings, and types; never disguise business logic or defaults as an `adapter` or `generated`.
3. **Handwritten tests:** Unit, integration, contract, and end-to-end suites shipped in the same Feature.
4. **Supporting config and docs:** Required configuration, migration metadata, and documentation.
5. **Verified non-handwritten files:** Lockfiles, snapshots, vendored files, binaries, and codegen outputs, excluded from handwritten counts only after listing each path, its source, and zero manual logic edits.

Judge review difficulty first by domain risk, state transitions, contract surface, and file spread; a 20-line permission or payment edit still needs deep invariant review. Treat rough size numbers only as **initial team planning heuristics**, never as researched reviewer-accuracy facts or universal Feature caps. Recheck scope when core decision files exceed roughly `6-8` files, handwritten production churn exceeds `500-800` `added + deleted` lines, or a new file passes `400-500` lines. Apply the host repository's PR size policy to the entire proposed PR. Keep small fixes inside existing large files localized and companion tests with their code. When scope grows, reshape the module and its contracts before continuing; sequential review checkpoints do not make an oversized PR acceptable. See [work-package orchestration](references/work-package-orchestration.md).

## Test Completeness, Commit Hygiene, and Closure

Begin every new or materially expanded test file or suite with a short note naming covered scenarios, protected behavior or failure modes, and the prior coverage gap. Assert observable contracts, persisted read-backs, and recovery rules rather than private helper calls, keeping independent tests isolated and declaring preconditions for any stateful chain. Run real client acceptance in the browser, CLI, SDK, or device runtime whenever backend or contract changes alter client-observable behavior, even with zero UI edits.

Write commit subjects, commit bodies, and PR summaries around the business change and validation evidence. Omit model names and automated-generation or tool attribution and provenance trailers from commit and PR metadata. Preserve normal Git author identity, legitimate human credit, and product model or provider identifiers in code and APIs. Bind verification logs to the exact candidate, complete authorized ordinary merge and target-branch read-back, coordinate coupled user-visible changes, and report `merged`, `released`, and `blocked` separately. Close finished Feature PRs promptly while keeping named owners and immediate next actions on any remaining work; see [integration and acceptance](references/integration-and-acceptance.md).

## Optional External References

- [Sentry Code Review Skill](https://github.com/getsentry/skills/blob/c2f99a5b04b4cd992ec3022d7c2c3e23e938d241/skills/code-review/SKILL.md) (supplementary review heuristics)
- [Testified Test Case Design Skill](https://github.com/testified-oss/agent-skills/blob/dce83769bc5d45b2150c1ac2e3c731db17830b29/skills/test-case-design/SKILL.md) (supplementary test-design patterns)
