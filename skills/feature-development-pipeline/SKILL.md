---
name: "feature-development-pipeline"
description: "Run single-feature delivery from behavior definition through reviewable implementation, intent-aligned testing, real-boundary acceptance, and authorized merge closure in single-agent or multi-agent mode."
---

# Feature Development Pipeline

This repository holds a high bar for code taste: modules keep one clear responsibility, control flow stays direct, abstractions and dependencies stay minimal, and tests verify real business behavior. Every agent delivering a Feature owns its correctness, reviewability, intent-aligned tests, and closure up to the authorized delivery boundary. An author's job does not end when coding stops or a pull request opens; the author remains accountable through review fixes, boundary verification, and landed merge confirmation. Responsibility still respects repository permissions: leaf contributors own their slices and fixes without relaxing independent review, while one primary integrator owns cross-slice integration and authorized target-branch merge.

Sustaining that standard at delivery speed creates four practical tensions. First, a real Feature often crosses UI, API, domain, and storage layers, yet bundling extra outcomes quickly makes diffs hard to review. Second, multi-agent parallelism stalls or collides whenever interfaces stay fluid or two writers edit the same shared files. Third, test suites lose value when they chase line counts or mirror private helpers instead of protecting user-visible behavior. Fourth, teams lose throughput when agents start new coding slices faster than they finish review, acceptance, and merge work.

This workflow resolves those tensions through strict Feature boundaries, explicit single-agent and multi-agent operating modes, an eight-stage lifecycle, honest diff accounting, and verified closure. Host `AGENTS.md` instructions, architecture documents, permission boundaries, test conventions, and branch protections always take precedence. Reuse confirmed designs and repository-required artifacts rather than creating parallel paperwork or reopening settled product decisions.

## Core Principles and Unit Boundaries

A **Feature** is one new or modified business behavior observable by a user or external caller. It may cross UI, API, domain logic, persistence, configuration, and companion tests when every touched layer serves that single behavior. Do not bundle independent business outcomes or opportunistic cleanups in one Feature for coding convenience.

1. **Single Feature:** Scope each delivery unit to one observable new or changed business behavior and ship its companion tests together.
2. **Reviewable scope:** Keep core business decisions in a small, explicit file set, and agree a concrete review boundary before implementation expands.
3. **Intent-aligned testing:** Tie every new or materially changed test suite to business scenarios, protected failure modes, and real runtime boundaries.

Keep the four execution units distinct:

- **Feature (delivery unit):** One coherent business change that satisfies its own acceptance chain and merge gates.
- **Work package:** One internal implementation slice assigned to a single author; never open a separate PR per work package.
- **PR:** The repository delivery vehicle for a Feature, or coordinated repository PRs when one Feature spans repositories.
- **Review packet (internal only):** A commit or snapshot checkpoint used to review a large indivisible Feature in bounded passes without opening per-packet PRs.

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
| **1. Behavior and acceptance definition** | User request, host rules, confirmed product or design docs | State the single observable new or changed behavior, entry trigger, primary flow, failure or recovery behavior, and authoritative persistence or client effect. Preserve any user-ordered visible milestones. | Concise behavior statement and observable acceptance chain mapped to verification tiers | **Advance:** Outcome and acceptance checks are explicit. **Stop/Ask:** User-visible behavior is ambiguous and unresolvable from repo sources. |
| **2. Code, design, and contract inspection** | Stage 1 acceptance chain, current codebase, existing schemas and tests | Inspect affected modules, data shapes, state transitions, error/retry semantics, and merge/runtime rules. Settle external or cross-package contracts; update only repository-required design artifacts. | Settled contract summary, touched boundaries, and required artifact updates | **Advance:** Consumed contracts and state ownership are settled. **Return to 1:** Inspection exposes contradictory behavior assumptions. |
| **3. Scope, mode, and test-intent planning** | Settled contracts, candidate file map | Select single-agent or multi-agent mode, name the integrator, agree a review boundary (expected decision paths, key invariants, and rough diff size if useful), assign disjoint paths, and plan test suites with scenario and gap notes. Define a split or internal packet plan before coding if scope is wide. | Mode choice, ownership/dependency map, agreed review boundary, test-intent plan, and split/packet plan if needed | **Advance:** File ownership is conflict-free and the review boundary is bounded. **Rescope before 4:** Independent outcomes are bundled or core logic is scattered without need. |
| **4. Implementation and focused checks** | Stage 3 plan and settled prerequisites | Implement production changes and companion tests together. Keep localized fixes inside existing large files tight without forcing file-split churn. Run focused unit and narrow contract checks on touched paths. | Clean working diff or package checkpoint, suite-level intent notes, passing focused check logs | **Advance:** Assigned scope and focused checks pass. **Return to 2/3:** Contract gap appears or changes outgrow the Stage 3 review boundary. |
| **5. Integration and exact candidate freeze** | Completed implementation slice(s), current target branch | Integrate slices, complete shared-file handoffs, rebase or merge against the target branch if needed, freeze an exact commit or content snapshot, and disclose all changed paths and `merge-base` `added + deleted` lines by role. | Exact candidate ID, full path disclosure by role (including verified generated/lock/binary files), passing build and smoke checks | **Advance:** Candidate builds cleanly and diff accounting matches the review plan. **Return to 3/4:** Integration fails or unsplit scope bloats a flat review pass. |
| **6. Integrated Feature Review and boundary/QA acceptance** | Frozen candidate, acceptance chain, diff summary, Stage 4-5 logs | Obtain one integrated Feature Review covering correctness, domain invariants, module responsibility, code necessity, and test intent (using planned packets plus final integrated sign-off when needed). Run required CI, integration, and real client/runtime checks in parallel when safe. | Consolidated review verdict and distinct unit, integration, CI, and client/runtime evidence bound to the candidate ID | **Advance:** Review and all mandatory boundary checks pass on the exact candidate. **Go to 7:** Actionable findings or check failures remain. **Return to 2/3:** Broken contract or mixed Features appear. |
| **7. Minimal finding fixes and delta recheck** | Stage 6 findings or failed check logs | Route minimal fixes to the original author or integrator, freeze the updated candidate, rerun affected checks and touched integration paths, and return the exact delta to the **same reviewer** while reusing valid unchanged evidence. | Delta diff, new candidate ID, rerun check logs, same-reviewer delta sign-off | **Advance:** Focused fixes preserving or restoring approved contracts and intended behavior pass affected checks, same-reviewer delta sign-off, and host review gates. **Return to 6 (full):** Fixes materially alter approved Feature scope, public contracts, intended behavior, or core risk invariants. |
| **8. Authorized merge, read-back, and closure** | Passed Stage 6/7 gates, verified delivery permissions | Write business-focused commit/PR messages (omitting model, generation-tool, or automated-generation attribution and provenance trailers while keeping normal Git author identity and human credit), publish/update the Feature PR if authorized, satisfy branch checks, run ordinary merge in safe order, read back the target branch, and complete any in-scope live check. | Target-branch commit read-back, distinct `merged` / `released` / `blocked` status, and named owner + next action for any blocked or remaining unit | **Complete:** Authorized boundary verified on target (and live environment if in scope). **Hold blocked step only:** External permission or gate missing; keep unrelated ready units moving. |

**Stop and return rules:** Pause affected work immediately when destructive or external authorization is missing, user requirements conflict, or live side effects cannot be verified. Never treat author `STOP`, green unit tests, or an open pull request as Feature completion.

## Reviewability and Diff Accounting

Measure review scope from `git merge-base <target> <candidate>` using `added + deleted` lines, never net delta or latest-commit diffs. Disclose every changed path across five categories:

1. **Core decision files:** Domain rules, state transitions, auth/permissions, billing/payments, concurrency, persistence schemas/queries, and external or cross-service contracts.
2. **Supporting handwritten production code:** Framework wiring, routes, transport mapping, UI bindings, and types; never disguise business logic or defaults as an `adapter` or `generated`.
3. **Handwritten tests:** Unit, integration, contract, and end-to-end suites shipped in the same Feature.
4. **Supporting config and docs:** Required configuration, migration metadata, and documentation.
5. **Verified non-handwritten files:** Lockfiles, snapshots, vendored files, binaries, and codegen outputs, excluded from handwritten counts only after listing each path, its source, and zero manual logic edits.

Judge review difficulty first by domain risk, state transitions, contract surface, and file spread; a 20-line permission or payment edit still needs deep invariant review. Treat rough size numbers only as **initial team planning heuristics** for a bounded review pass, never as researched reviewer-accuracy facts or universal Feature caps. Recheck scope when core decision files exceed roughly `6-8` files, handwritten production churn exceeds `500-800` `added + deleted` lines, or a new file passes `400-500` lines. Keep small fixes inside existing large files localized without forcing file-split churn, keep companion tests with their code, and never grant an unbounded `atomic feature` waiver. When a diff outgrows a bounded flat pass, split genuinely independent outcomes, trim repetitive test setup, or run internal review packets plus final integrated verification; see [work-package orchestration](references/work-package-orchestration.md).

## Test Completeness, Commit Hygiene, and Closure

Begin every new or materially expanded test file or suite with a short note naming covered scenarios, protected behavior or failure modes, and the prior coverage gap. Assert observable contracts, persisted read-backs, and recovery rules rather than private helper calls, keeping independent tests isolated and declaring preconditions for any stateful chain. Run real client acceptance in the browser, CLI, SDK, or device runtime whenever backend or contract changes alter client-observable behavior, even with zero UI edits.

Write commit subjects, commit bodies, and PR summaries around the business change and validation evidence. Omit model names and automated-generation or tool attribution and provenance trailers from commit and PR metadata. Preserve normal Git author identity, legitimate human credit, and product model or provider identifiers in code and APIs. Bind verification logs to the exact candidate, complete authorized ordinary merge and target-branch read-back, coordinate coupled user-visible changes, and report `merged`, `released`, and `blocked` separately. Close finished Feature PRs promptly while keeping named owners and immediate next actions on any remaining work; see [integration and acceptance](references/integration-and-acceptance.md).

## Optional External References

- [Sentry Code Review Skill](https://github.com/getsentry/skills/blob/c2f99a5b04b4cd992ec3022d7c2c3e23e938d241/skills/code-review/SKILL.md) (supplementary review heuristics)
- [Testified Test Case Design Skill](https://github.com/testified-oss/agent-skills/blob/dce83769bc5d45b2150c1ac2e3c731db17830b29/skills/test-case-design/SKILL.md) (supplementary test-design patterns)
