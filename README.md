# High-Efficiency Feature Development

High-Efficiency Feature Development is a portable workflow for incremental software delivery. It scopes each Feature as a small, cohesive module within one domain, with clear contracts, its own tests, and a reviewable PR. Ready modules merge progressively; integration and end-to-end checks then verify the assembled user flow.

## The Challenge

Parallel agents can generate work faster than integration capacity, and this can shift the bottleneck from implementation to verification. When the volume of incoming patches exceeds the speed of review and end-to-end validation, repeated setup, redundant code analysis, and lost context consume valuable time and tokens, potentially hindering the project’s overall velocity rather than accelerating it.

## Field Observations

When multi-agent delivery stalls near the finish line, five failure patterns typically appear:

- **Implementation-fragment PRs:** Opening separate PRs for helpers, state classes, or test files multiplies review overhead without delivering a complete module responsibility.
- **Lost cross-boundary context:** Independent workers make incompatible assumptions across pages, APIs, state transitions, or persistence layers.
- **Duplicate reviews on shifting candidates:** Running full-scope audits repeatedly before underlying branches stabilize wastes context and produces contradictory feedback.
- **Green mocks without an operable flow:** Unit suites pass against synthetic stubs while the assembled application fails on real state transitions or persistence.
- **Integration and merge queues:** Completed fragments sit waiting for conflict resolution, rebasing, and combined verification while new branches continue to open.

Progress includes tested, reviewed modules landing on the target branch, followed by verified integration and complete user flows. These are distinct milestones. A helper-level patch is not automatically a Feature, and a finished module need not wait for the entire initiative before it can merge.

## Pipeline Architecture

The workflow separates internal execution parallelism from integration and review boundaries.

```mermaid
flowchart LR
    A[Domain & Module Contracts] --> B[Module PR Boundaries]
    B --> C1[Package A: Owned Files]
    B --> C2[Package B: Owned Files]
    C1 --> D[Bounded Module Candidate]
    C2 --> D
    B --> E[Acceptance & Runtime Prep]
    D --> F[Module Review & Own Tests]
    F --> G[Incremental Merge & Read-back]
    G --> H[Assembled Integration & End-to-End]
    E --> H
```

### 1. Outcome Definition and Single Integrator Ownership
Work begins by inspecting the domain, existing modules, contracts, and repository rules. Before implementation, the integrator identifies complete module responsibilities and their intended PR boundaries, tests, and merge order. The integrator owns both incremental merge closure and later assembled-flow acceptance.

### 2. Execution Packages vs. Delivery Boundaries
Parallel packages are internal execution units. A Feature PR contains one cohesive domain module and its necessary tests, config, and docs. A picker, editor, or service with a complete responsibility and real consumer contract can be a Feature; individual buttons, helpers, state classes, and test files stay inside their module. Settle interfaces and file ownership before parallel work. Do not implement a whole workflow first and defer PR decomposition until the end.

### 3. Capacity-Aware Scheduling and Worker Reuse
The integrator maintains adaptive scheduling preferences informed by bottlenecks, readiness, risk, and capacity. Prioritize critical path unblocking, integration defects, and acceptance or closure tasks—but blocked urgent work must not prevent useful, independent parallel progress. Concurrency tracks actual reviewer and runtime capacity; workers are never spawned merely to fill slots. Context-carrying workers are reused across related follow-ups rather than replaced with fresh duplicate audits or noisy status polling.

### 4. Shared Runtime Discipline and Pipelining
Isolated unit and contract checks may run concurrently inside individual packages. However, any shared runtime environment—such as an application server, database, simulator, or seeded test environment—has a single owner responsible for lifecycle, configuration, test data, and recovery. Focused tests, CI checks, and acceptance preparation are pipelined as soon as dependencies permit. CI and actual independent runtime acceptance can overlap with the Feature Review, not just preparation.

## Feature Review

Each bounded module receives one Feature Review when its implementation and companion tests are coherent:

- **Full module context:** The reviewer evaluates the module's complete operation, domain boundaries, consumer contracts, implementation, and own tests together. Related modules remain separate delivery units; sequential review checkpoints do not justify an oversized PR.
- **Consolidated actionable findings:** Feedback is grouped into concrete defects stating exact trigger conditions and user or system impact.
- **Targeted delta verification:** After fixes land, the same reviewer inspects the fix delta and its integration impact. A full review restart occurs only when scope or behavior changes materially, or when local mandatory gates require one.

## Integration and Acceptance Evidence

A module reaches its merge milestone after its own tests, review, mandatory checks, authorized merge, and target-branch read-back. As modules land, test their real interactions; once the affected flow is assembled, run full integration, end-to-end acceptance, and authorized live verification. Module merges and whole-flow release are reported separately. Evidence categories stay distinct:

- **Unit evidence:** Fast logic and edge-case checks in isolation.
- **Integration evidence:** Real module, service, storage, or API contract interaction.
- **CI evidence:** Automated repository build, lint, type, and regression gates.
- **Browser or device evidence:** Real UI rendering, navigation, and interaction proof, selected from the affected user journey. Backend changes affecting clients may require client acceptance, but isolated non-UI behavior does not require UI tests.

## Portability and Host Governance

This repository provides a portable coordination layer without hardcoding project names, local paths, API hosts, credentials, model routing, fixed worker counts, numeric PR caps, branch policies, or stack-specific rules. At runtime, user instructions and the host repository's `AGENTS.md`, architecture docs, runtime procedures, contribution guidelines, permissions, and mandatory merge gates always take precedence.

### Repository Contents

- `skills/feature-development-pipeline/SKILL.md` — Core workflow entry point, triage rules, and decision guardrails.
- `skills/feature-development-pipeline/references/work-package-orchestration.md` — Detailed guidance on file ownership, scheduling priorities, worker reuse, and shared-runtime control.
- `skills/feature-development-pipeline/references/integration-and-acceptance.md` — Rules for early code integration, consolidated Feature Review, delta re-review, and boundary-appropriate test design and evidence.

## Installation and Usage

Clone this repository and link or copy the nested skill directory into your local Codex skills path:

```bash
git clone https://github.com/miles-song/high-efficiency-feature-development.git
mkdir -p ~/.codex/skills
ln -s "$(pwd)/high-efficiency-feature-development/skills/feature-development-pipeline" ~/.codex/skills/feature-development-pipeline
```

If you prefer a standalone copy instead of a symlink, run `cp -R` with the same source and destination paths.

Invoke the workflow in Codex by referencing `$feature-development-pipeline` in your prompt when planning or executing multi-step features. Leave `$feature-development-pipeline` uninvoked for small, single-file, low-risk fixes where direct execution is faster and clearer.

## Iterating on the Workflow

Refine this workflow only through narrow, targeted adjustments backed by observed delivery bottlenecks or escaped late-stage defects. Avoid accumulating speculative checklists, rigid ceremony, or parallel status-tracking bureaucracy, and evaluate changes against actual integration smoothness and defect discovery.

## Optional Supplementary References

Teams looking for standalone guidance on general code review heuristics or test-case design techniques may consult these external skills (linked for optional reference only; neither is bundled nor required by this workflow):

- [Sentry Code Review Skill](https://github.com/getsentry/skills/blob/c2f99a5b04b4cd992ec3022d7c2c3e23e938d241/skills/code-review/SKILL.md)
- [Testified Test Case Design Skill](https://github.com/testified-oss/agent-skills/blob/dce83769bc5d45b2150c1ac2e3c731db17830b29/skills/test-case-design/SKILL.md)

## License

Released under the [MIT License](LICENSE).
