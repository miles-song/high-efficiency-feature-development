# Integration and Acceptance Testing

Use this reference when a feature crosses runtime boundaries or when correctness depends on persistence, state transitions, recovery, or real user interaction. Obey host repository rules for test environments, data isolation, and mandatory gates.

## Select and Design Integration Checks

Derive checks from consequential user-journey steps and state transitions rather than testing every permutation:

- **Declare boundaries:** State which collaborating layers are real in the check (such as client UI, API service, database, queue, or object storage) and which are mocked or excluded. If a collaborator central to the claim is mocked, name the mock and narrow the evidence claim accordingly.
- **Isolate fixtures and state:** Use deterministic inputs and isolated test data scoped to the run. Never rely on shared mutable records or execution order.
- **Explicit setup and teardown:** Define required service configuration, seed data, and starting state. Ensure teardown runs on both success and failure, removes only resources owned by the test, and verifies cleanup when residue could affect subsequent runs.
- **Assert cross-layer effects:** Do not stop at an HTTP status, dispatched event, or optimistic UI state. Verify the relevant authoritative effect across the boundary (such as emitted messages, downstream calls, or state changes). For features that persist state, query, reload, or reopen through the standard authoritative read API to prove persistence and recovery; inspect the underlying datastore directly only when permitted by local architectural and test boundaries.
- **Cover contract-specific failure modes:** Exercise error handling, unknown outcomes, retries, duplicate submissions, or cancellation only when the feature contract requires them. Keep retry clocks deterministic and assert that duplicate actions do not corrupt state or create unintended records.

### Example: Persisted-Record Save-and-Reopen Outline

Use a pattern like this when a feature creates or updates durable records (adapt setup and read paths to local architecture):

1. **Setup:** Create isolated test fixtures or seed an initial record through the supported service or test setup path (mocking only unrelated external collaborators).
2. **Action:** Submit a state-changing update through the target interface.
3. **Cross-boundary assertion:** Verify the response contract and the authoritative side effect (via the normal read API, or direct datastore inspection where locally permitted).
4. **Read-back / reopen:** Re-fetch or reopen the resource in a fresh context and assert that the persisted state returns or renders accurately.
5. **Teardown:** Remove only the test-owned records or resources in a guaranteed cleanup step.

## Execution Cadence

Pipeline verification to catch defects early without redundant full-suite runs:

- Run focused unit and narrow integration checks as soon as individual work packages or interfaces stabilize.
- Run broader integration, CI, and acceptance checks against the integrated candidate once dependencies meet.
- After a fix, rerun the affected checks and any integration path touched by the delta. Carry forward earlier results only when the underlying code, configuration, and environment remain unchanged; run broader regression checks if the fix alters shared contracts or risk.

## Keep Evidence Distinct

Do not conflate verification tiers or claim unrun checks when reporting closure:

- **Unit:** Isolated logic or component behavior with mocked collaborators.
- **Integration:** Specified real boundaries, isolated fixtures, guaranteed teardown of owned data, and authoritative cross-boundary assertions (including persistence/read-back when state is stored).
- **CI:** The exact automated suite and runner environment executed; passing CI does not imply browser or device acceptance unless those checks actually ran in that job.
- **Browser / Device Acceptance:** Select based on the affected user journey rather than the category of modified files. Run browser or device checks whenever a change alters an existing or new client-facing flow (including backend changes that alter client-visible behavior), and omit them for genuinely non-UI behavior. Exercise the agreed user flow in the real runtime, verifying visible outcomes, network requests, and persisted state on reload when applicable. Record browser and physical/emulated device evidence separately when both apply.

For every reported check, record the exact revision, target environment, command or user flow, data scope, pass/fail outcome, and artifact or log location, applying base/head and diff checks to the actual delivery heads rather than a dirty combined tree. When the user has authorized remote delivery, carry closure through local commit/submodule/dependency order, push/PR, required independent review and CI, merge and target-branch read-back, and any in-scope deployment or known-target check—distinguishing external deployment blockers from verified live updates. Redact credentials, tokens, and private data from all outputs and screenshots.

## Optional Further Reading

- For broader risk-based test design, see [Testified's test-case-design skill](https://github.com/testified-oss/agent-skills/blob/dce83769bc5d45b2150c1ac2e3c731db17830b29/skills/test-case-design/SKILL.md) as an optional supplement to local testing rules.
