# Work-Package Orchestration and Reviewability Mechanics

Load this reference when selecting multi-agent execution, coordinating parallel waves across Features or work packages, or bounding review scope for a large candidate. Host `AGENTS.md` instructions, delegation limits, and branch protections always take precedence.

## Single-Agent vs Multi-Agent Decision Gate

Stay in single-agent mode when one author can hold the entire change cleanly: localized fixes, edits concentrated in the same files, or exploratory work with unsettled interfaces. Switch to multi-agent mode at Stage 3 only when all three conditions hold:

1. Consumed interfaces, data shapes, error semantics, and state ownership are settled.
2. At least two work packages (or independent Features) have disjoint writable paths and clear check commands.
3. Host rules permit delegation and active capacity remains to drain review, acceptance, and merge queues.

## Hierarchy, Ownership, and Dependency Map

- **Root-only dispatch:** One **primary integrator** owns decomposition, contract decisions, candidate integration, review/QA coordination, and target-branch closure. Dispatched contributors operate as **leaf workers** and must never spawn sub-workers, relax independent review, or self-authorize merges.
- **Compact dependency map:** Reuse the active task note or handoff record instead of creating a parallel tracking system. For each Feature and work package, record the integrator, package owner, writable paths, consumed contracts, shared-file sequence, runtime reservations, candidate ref, status, and immediate next action. Use explicit statuses: `ready`, `in_progress`, `awaiting_review_or_qa`, `ready_to_merge`, `merged`, `released`, or `blocked` (always naming the blocker, owner, and unblock condition).
- **Early consumer start:** A consumer package (client UI, caller logic, QA harness, or docs) may start before its producer finishes once schemas, state rules, and error contracts are fixed. Final boundary acceptance still waits for the integrated producer-and-consumer candidate at Stage 5 and Stage 6.
- **Shared-file handoff:** Give concurrent writers disjoint file scopes. Keep shared routers, registries, barrels, or ordered migrations with the integrator, or sequence them (`Package A -> integrator check -> Package B`).
- **Source isolation:** Isolate concurrent writers with separate worktrees, branches, or strict path scopes so uncommitted edits and snapshots never contaminate another worker's diff or test run. Disjoint owned paths or branch names alone do not isolate a build that reads the whole worktree. Once `C1` freezes, the exact source, build, and config inputs used by its review or QA cannot be edited underneath running checks. Continued parallel implementation must use an isolated checkout or snapshot alongside coordinated runtime reservations.
- **Runtime lifecycle vs reservations:** Assign **one shared-runtime lifecycle owner** for any shared dev server, database, container stack, seed state, or recovery routine. Separately track narrow **resource reservations** for exclusive conflicts: port bindings, interactive browser sessions, mobile/device emulators, destructive DB resets, or quiet CPU/GPU benchmark windows. Isolated unit suites, static analysis, and isolated test containers run concurrently outside those reservations.
- **Scoped failure handling:** When a package, check, or external gate fails, pause only the units that depend on that broken contract, artifact, or gate. Do not launch duplicate speculative implementations of a blocked prerequisite; redirect spare capacity to fixes, Stage 5-8 draining, or independent ready Features.

## Compact Leaf Task Brief

Send each leaf worker a self-contained brief without dumping unrelated chat history:

```text
Objective: concrete package deliverable and the single Feature outcome it serves
Business & test intent: user/caller scenario, failure/recovery expectations, and required suite-level scenario/gap notes
Exact writable paths: explicit files or directories writable by this worker (all other paths read-only)
Consumed interfaces & prerequisites: settled contracts, schemas, error codes, upstream refs, and non-goals
Allowed environment: permitted test/build commands, isolation mode, shared-runtime owner, and active resource reservations
Handoff output: base ref + checkpoint commit/patch, merge-base added+deleted counts by path category, exact checks run with pass/fail logs, and unresolved risks or blockers
Next closure owner: primary integrator (or named reviewer/QA leaf) and the immediate Stage 5/6 action after handoff
```

## Overlapping Execution Schedule and Bottleneck Control

Reserve finishing capacity (at least one active slot or the primary integrator's immediate attention) to drain the oldest unit in Stages 5-8 before starting new coding packages.

| Wave | Prerequisite / Serial Gate | Safe Parallel Work | Output Needed Next |
|---|---|---|---|
| **Wave 1: Contract & plan** | Stage 1 behavior clear | Primary integrator settles Stage 2 contracts and Stage 3 ownership/review boundary; optional read-only repo/test inspection leaves | Settled contracts, disjoint file map, agreed review boundary, and test-intent plan |
| **Wave 2: Parallel build & prep** | **Serial gate:** consumed contracts and file map settled | Producer leaf(s), contract-consuming client/caller leaf(s), companion docs/config updates, and integration/QA fixture or harness preparation | Verified package checkpoints and ready acceptance harness |
| **Wave 3: Candidate freeze** | **Serial gate:** required packages for Feature `F1` pass focused checks | Primary integrator integrates `F1`, resolves shared-file wiring, freezes exact candidate `C1`, and computes `merge-base` diff counts; independent Feature `F2` continues Wave 2 | Frozen candidate `C1` and full role-grouped diff disclosure |
| **Wave 4: Review + QA overlap** | **Serial gate:** exact candidate `C1` frozen | One integrated reviewer inspects `C1` while CI, integration suites, and permitted browser/device acceptance run against `C1` under runtime reservations; `F2` integrates in parallel | Consolidated review result + distinct runtime/CI evidence bound to `C1` |
| **Wave 5: Delta fix & reuse** | Actionable Stage 6 findings or check failures on `C1` | Original leaf owner (or integrator) applies minimal fix -> freezes `C2` -> reruns affected checks -> **same reviewer** inspects `C1..C2` delta while valid unchanged evidence stays attached | Approved `C2` candidate without full re-audit of untouched scope |
| **Wave 6: Independent closure** | **Serial gate:** `C1`/`C2` review + mandatory checks pass and merge is authorized | Primary integrator merges `F1`, reads back target branch, and runs in-scope release check while `F2` continues in Wave 3-5 | Verified `F1` closure (`merged` / `released`) with `F1` PR(s) closed; remaining active units retain named owners and immediate next actions |

## Reviewability Mechanics and Oversize Response

### 1. Reproducible Merge-Base Accounting

Measure scope from the target branch merge-base to the exact candidate:

1. Resolve `BASE=$(git merge-base <target-branch> <candidate-ref>)`.
2. Run `git diff --numstat "$BASE" <candidate-ref>` and sum `added + deleted` lines per file; never report net delta (`added - deleted`) or latest-commit-only diffs.
3. Disclose every changed path across five buckets:
   - **Core decision files:** domain invariants, state transitions, auth/permissions, billing/payments, concurrency/locking, schema/query logic, and external or cross-service contracts.
   - **Supporting handwritten production files:** framework wiring, transport DTO plumbing, straightforward UI bindings, and config glue.
   - **Handwritten test files:** unit, integration, contract, and browser/device suites.
   - **Supporting config and docs:** documentation, environment templates, and non-code metadata.
   - **Verified generated, vendor, lock, or binary paths:** excluded from handwritten totals only after listing each path, its generator or origin, and verifying zero hand-edited logic.

Never classify handwritten branching, business defaults, retry policies, or permission checks as `generated` or `mechanical adapter`.

### 2. Risk Calibration and Provisional Planning Heuristics

At Stage 3, agree an explicit review boundary covering the expected core decision files, critical invariants, and, when useful, a rough handwritten diff estimate. Calibrate that boundary primarily by semantic risk: state-machine changes, contract surface, cross-directory spread, and sensitive logic in auth, permissions, billing, idempotency, concurrency, or migrations. A 15-line permission or payment-retry edit still requires thorough failure-mode review.

Treat numeric thresholds solely as **initial team planning heuristics** (provisional checkpoints calibrated to risk, not researched reviewer-accuracy facts and not universal Feature caps):

- Treat roughly `6-8` core decision files or `500-800` handwritten production `added + deleted` lines as a checkpoint to verify that one review pass remains bounded.
- Treat roughly `400-500` lines in a **new** handwritten file, or a second unrelated responsibility in an existing file, as a cohesion check.
- Keep localized bug fixes or small behavior updates inside existing large files tight; never split legacy files merely to satisfy file-count or line-count heuristics.
- Prefer direct control flow and clear module ownership over compressed one-liners or helper layers created only to shuffle line counts across files.

### 3. Concrete Actions When a Candidate Outgrows a Flat Pass

When a Stage 3 estimate or Stage 5 diff crosses the agreed review boundary, stop expanding the flat pass and never claim an unbounded `atomic feature` waiver. Apply these responses in order while keeping business code and companion tests in the same Feature PR:

1. **Split independent outcomes or groundwork:** Extract any second user-visible behavior or standalone prerequisite into its own Feature with its own tests and merge cycle.
2. **Trim repetitive test setup:** Consolidate duplicated test scaffolding into shared builders, fixtures, or parameterized cases without weakening scenario coverage.
3. **Use bounded internal review packets:** Partition an indivisible cross-layer Feature into ordered commit or snapshot checkpoints reviewed sequentially by the **same reviewer**, without opening per-packet PRs.
4. **Run final integrated verification:** Freeze the complete Feature candidate, disclose full `merge-base` counts, verify cross-packet interactions, and pass full Stage 6 boundary acceptance before merge.
