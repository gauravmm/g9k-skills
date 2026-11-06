---
name: minimal-risk-tests
description: Choose a small, risk-based test set for new code or prune an existing suite. Use for test retention criteria, redundant coverage, configuration checks, and capability boundaries. Includes an optional iterative audit for discovering repository-specific rules.
---

# Minimal risk-based tests

Choose the smallest useful test set for the application's contracts and accepted risks. The rules below are the core skill. Use the optional audit procedure only for a sustained pruning pass or to develop local testing policy.

## Core rules

### Establish the risk policy

- Use the user's requirements and project policy to identify protected failure classes and accepted risks. A request to reduce test count does not authorize every correctness, liveness, privacy or authorization tradeoff.
- Apply any requested reduction target across the agreed scope. Use it to prompt deeper review, not as a quota for each file. Do not invent a fixed reduction percentage.
- An observed regression is evidence of a failure, not a permanent retention requirement. Remove its unique detection only when the behavior is retired or the resulting risk is accepted.
- Name consequences precisely. Lost path confinement, credential isolation or output fencing is a security consequence, even when the affected code also formats or validates data.

### Test owned contracts

- Identify the owner of observable results, state transitions and effects. Include preconditions, invariants, failure and rollback behavior where applicable.
- Prefer a representative successful path and selected consequential edges. Branch reachability, input variety and scenario names alone do not earn tests.
- Keep consumer checks for independently breakable wiring, arguments, authorization or transaction placement. A helper test cannot detect a caller that omits the helper.
- Use the smallest scope that can detect the failure faithfully. Process restarts are necessary for some checkpoint and replay faults; repeated calls in one process do not establish crash safety.
- Combine related assertions about one operation or transition. Keep unrelated failure mechanisms separate. A boundary can span files when they share one state owner.
- Consider a small refactor toward a cohesive owner, purity or idempotence when it removes meaningful complexity. Preserve application behavior and durable identities; avoid abstractions created only for fixtures.

### Require marginal value

- For each additional case, name a plausible trigger, its consequence and the detection it adds beyond the retained set. Extra matrix combinations need a meaningful interaction to justify them.
- Give speculative coverage less weight when an identified ordinary check reliably exposes the fault. Do not assume rare paths will be noticed or equate visibility with low consequence.
- Classify each removal as redundant coverage, configuration/constant inspection, retired behavior, upstream responsibility, or accepted risk.
- Redundancy requires a retained witness matching the exercised path, boundary and assertion. Accepted risk requires naming the lost detection; it does not require an equivalent witness.
- Similar names, fixtures, line coverage or passing tests do not prove redundancy. A representative case may replace common coverage while leaving a distinct edge deliberately untested.

### Replace configuration pins with constraints

- Drop tests that merely pin chosen settings, validate a particular deployed configuration or restate inspectable constants.
- Enforce necessary validity relationships at runtime, at the owning boundary. Reuse existing validators. Use explicit validation for external input and safety constraints; assertions that can be disabled are insufficient for those boundaries.
- Do not replace arbitrary setting snapshots with runtime assertions that freeze the same values.
- Distinguish deployed settings from runtime user input, and constants from generated schemas or rendered output. Dropping dynamic checks accepts behavioral risk; it is not evidence that inspection alone suffices.

### Protect unavailable capabilities

- For each capability the application deliberately withholds, keep one focused witness for its effective boundary. Cover the entry points that could grant that capability, including independent actors or channels where relevant.
- Exercise actual constructed authority when a tool-list helper or configuration flag cannot detect an incorrectly privileged consumer. Require nonempty observations so an absent setup cannot make the check pass vacuously.
- Use doubles or traps that prevent real external effects. Where applicable, establish rejection before credentials, transport access or irreversible work.
- Source scans can detect application use of a forbidden dependency operation. They do not disable that operation in the dependency. Distinguish source absence, capability confinement and runtime rejection.
- Configuration-based blocking still needs its capability witness. Keep unrelated read, presentation and ownership contracts out of that test.

### Keep dependency coverage proportionate

- Test application adapters, configuration interpretation, authority and recovery boundaries. Do not recreate a dependency's general parser, validation, storage or retry suite.
- Assess the exact capability and local consequences. Popularity or import counts do not establish upstream coverage; unknown evidence remains unknown.
- Retain compatibility regressions according to the project's risk policy. A dependency executing a check does not make the application's configuration or wiring irrelevant.

### Measure and validate honestly

- Count collected parameter rows, test functions, lines and runtime separately. Moving cases into loops or unrelated assertions into a single test does not reduce behavioral complexity.
- Remove unused fixtures and helpers after pruning. Preserve helpers consumed outside the edited file.
- Run affected tests and the project's required checks. Collection errors, skipped checks and all-deselected runs do not count as successful execution.
- Report accepted detection losses with the measurements. A green reduced suite does not prove unchanged coverage.

## Optional: iterative pruning and local rule discovery

Use this procedure for a large suite or when local retention rules need evidence. It is unnecessary for an ordinary test addition or small edit.

1. Establish scope, baseline revision, collection counts and available validation commands. Inventory test files, helpers, integration/deployment tests and manual probes. Distinguish an inventory from semantic inspection; inspect manual probes without running live effects unless separately authorized.
2. Record the local risk policy and requested target. Start a repository-owned policy note for facts specific to that project: protected capabilities, accepted losses, state owners and required checks. Keep these facts out of the reusable skill.
3. Choose a small, cohesive tranche. Read every test body in that tranche and the relevant owner code. Select representative failure families, propose cuts and state the detection lost. A pilot should be small enough to revise cheaply.
4. Prune and combine within the agreed scope. Keep an existing protected witness until its replacement is validated. Run affected tests; distinguish environment restrictions from application failures.
5. Independently review proposed removals and replacements. When delegation is available and authorized, give workers disjoint file ownership and use separate reviewers within the user's concurrency limit. Review effective authority and risk classifications, not just passing results. A fresh review pass can substitute when separate reviewers are unavailable.
6. Synthesize each round before expanding. Turn concrete disagreements into candidate local rules. For each rule, record the triggering example, relevant counterexample, scope and how it changes retention decisions. Recheck earlier cuts if the refinement exposes a gap. Promote a rule to the reusable skill only when its reasoning applies beyond the project.
7. Try the revised rules on a fresh tranche without prescribing which tests to delete. Check whether another reviewer reaches compatible decisions. Refine ambiguous rules; do not accumulate exceptions merely to defend earlier cuts.
8. Save a concise per-file ledger: inspected scope, retained contracts, removal categories, accepted losses, review verdicts and validation. Keep inventoried, inspected, changed and validated statuses distinct. Record counts and unresolved work before handoff. Commit each coherent round when authorized and required checks are complete.
9. Repeat until the agreed scope has been inspected, including retained files. Reconcile the ledger with the inventory, run final required integration checks and report both reduction and coverage limits. Do not stop reviewing solely because the count target has been met.
