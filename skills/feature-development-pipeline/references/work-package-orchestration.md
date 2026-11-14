# Work-Package Orchestration and Reviewability Mechanics

Load this reference when selecting multi-agent execution, coordinating parallel waves across Features or work packages, or bounding review scope for a large candidate. Host `AGENTS.md` instructions, delegation limits, and branch protections always take precedence.

## Single-Agent vs Multi-Agent Decision Gate

Stay in single-agent mode when one author can hold the entire change cleanly: localized fixes, edits concentrated in the same files, or exploratory work with unsettled interfaces. Switch to multi-agent mode at Stage 3 only when all three conditions hold:

1. Consumed interfaces, data shapes, error semantics, and state ownership are settled.
2. At least two work packages (or independent Features) have disjoint writable paths and clear check commands.
3. Host rules permit delegation and active capacity remains to drain review, acceptance, and merge queues.

## Hierarchy, Ownership, and Dependency Map

- **Root-only dispatch:** One **primary integrator** owns decomposition, contract decisions, candidate integration, review/QA coordination, and target-branch closure. Dispatched contributors operate as **leaf workers** and must never spawn sub-workers, relax independent review, or self-authorize merges.
- **Module boundaries before fan-out:** Define each Feature as one cohesive module in a domain, with its entry or caller, complete responsibility, contracts, own tests, and merge order. A module can contain several internal packages. Do not build the whole initiative first, create one combined merge candidate, and split it later; do not turn each helper or package into a PR.
- **Compact dependency map:** Reuse the active task note or handoff record instead of creating a parallel tracking system. For each module Feature and internal package, record ownership, writable paths, contracts, shared-file sequence, reservations, candidate ref, status, and next action. Distinguish `merged` modules from an assembled flow awaiting integration/E2E and from `released` behavior; name the owner and unblock condition for blocked work.
- **Early consumer start and ordered merge:** A consumer module or package may start before its producer finishes once contracts are fixed. A complete service module can merge with its own contract tests before a later view module when the merge preserves existing behavior. Once both land, verify their real integration and assembled user journey. Do not defer all module merges until one project-wide candidate exists.
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
| **Wave 3: Module candidate freeze** | **Serial gate:** required packages for module `F1` pass focused checks | Integrator assembles only `F1`, freezes exact candidate `C1`, and computes full diff counts; separately scoped `F2` continues implementation | Bounded module candidate `C1` and role-grouped diff disclosure |
| **Wave 4: Module review + QA** | **Serial gate:** `C1` frozen | One reviewer checks the complete module while its tests and mandatory CI run; verify any existing integrations it changes, without waiting for an unfinished future flow | Module review and merge-check evidence on `C1` |
| **Wave 5: Delta fix & reuse** | Actionable Stage 6 findings or check failures on `C1` | Original leaf owner (or integrator) applies minimal fix -> freezes `C2` -> reruns affected checks -> **same reviewer** inspects `C1..C2` delta while valid unchanged evidence stays attached | Approved `C2` candidate without full re-audit of untouched scope |
| **Wave 6: Incremental closure and integration** | **Serial gate:** module review, mandatory checks, dependencies, and merge authorization satisfied | Integrator merges `F1` and reads back the target while `F2` advances. Run cross-module checks as dependencies land, then full integration/E2E once the affected flow is assembled | Verified module merges plus separately reported assembled-flow and release acceptance |

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

Use roughly `1,000` handwritten production `added + deleted` lines as a **soft scope and closure checkpoint**, calibrated to risk rather than a universal Feature cap:

- Count core decisions and supporting handwritten production together; disclose tests, config, docs, and verified generated files separately.
- At the checkpoint, decide whether this module's complete responsibility is ready for its own tests, review, and merge. If adjacent responsibilities are accumulating, rescope them before adding more implementation.
- Small complete modules close earlier; never pad a Feature to reach the checkpoint. Apply the host's PR policy, and justify any larger cohesive scope through that policy rather than treating this checkpoint as an exception.
- Keep core decisions in a small, explicit file set. Check new large files for mixed responsibilities, keep localized fixes in existing large files tight, and prefer direct control flow over file splits, compressed code, or helper layers created only to shuffle counts.

### 3. Concrete Actions When a Module Outgrows Its PR Boundary

When the estimate or actual diff exceeds the agreed module boundary or host PR policy, stop adding implementation. Change the delivery boundary before continuing:

1. **Separate complete module responsibilities:** Identify the distinct component, editor, selector, service, or other domain responsibility and give it a real consumer contract, tests, and merge order. Keep its own cross-layer implementation together.
2. **Keep fragments internal:** Helpers, state classes, transport functions, and individual test files stay inside their owning module; they do not each receive a PR or full review cycle. Avoid speculative prerequisite modules with no confirmed consumer.
3. **Reduce unnecessary implementation:** Remove duplication or unrelated responsibilities without compressing readable code, moving business logic into excluded files, or cutting necessary tests to disguise scope. Report test and supporting churn separately.
4. **Merge progressively:** Freeze, review, test, and merge each bounded module, then verify the assembled interactions and full user flow. A series of review checkpoints is not an exception to PR size policy, and a large existing candidate is not the default merge unit.
