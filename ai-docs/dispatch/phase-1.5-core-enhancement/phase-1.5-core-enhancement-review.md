# Phase 1.5: Core Enhancement — Spec Review

Perspectives: Architecture, Pragmatism, Quality, Advocacy

## Architectural Soundness

### Finding 1: task.json schema diverges from actual gate script architecture
**Severity**: blocking

The spec (section 4f) places the `category` field in `task.json` at the top level. The actual gate script (`task-reviewer-gate.sh`) does not read `task.json` — it parses YAML frontmatter from individual `task-NN-*.md` files. The spec describes an integration point that doesn't exist.

**Resolution**: Reconcile the `category` field placement with the actual gate architecture. Either add `category` as frontmatter in a designated task file or a separate manifest the gate reads, or modify the gate to consume `task.json`. The task-compilation.md documents a `task.json` schema, but the gate script doesn't use it — this is a pre-existing divergence that the spec must not compound.

### Finding 2: Test-level adequacy may be misclassified as Tier 1 (mechanical)
**Severity**: important

Silent failure detection is genuinely mechanical (pattern-match on assertion structure). Test-level adequacy requires judgment: determining whether behavior is "materially dependent" on a runtime environment different from the test environment is not binary. A React component that renders identically in jsdom for 95% of behavior but uses `getBoundingClientRect()` for one feature — is this "materially different"?

Tier 1 has no override path. False positives block with no recourse.

**Resolution**: Either move test-level adequacy to Tier 2 (overridable), or narrow it to a truly mechanical check with an enumerated list of runtime-dependent APIs (Canvas, WebGL, Web Audio, getBoundingClientRect, etc.) that trigger the flag.

### Finding 3: Baseline-snapshot storage, identity, and selective execution undefined
**Severity**: important

Section 4e defines the inter-wave regression check conceptually but omits: (1) what format the baseline is stored in, (2) where it's stored, (3) how "the same test" is identified across runs (rename-safe?), (4) whether baseline tests are run selectively or the full suite runs with results compared against the baseline list.

**Resolution**: The simplest sound approach: capture full test output at baseline, run the full suite after each wave, diff results. Any test that was "pass" at baseline and is now "fail" is a regression. This avoids selective execution entirely. Specify the storage location (e.g., `ai-docs/<feature>/baseline-snapshot.json`).

### Finding 4: Contract-staleness gap between breakdown and implementation
**Severity**: important

If Wave 1 modifies a file that Wave 2's task was compiled against, the Wave 2 task's interface contracts may be stale. The readiness check explicitly excludes re-verifying conventions ("trust the breakdown's verified contracts"). The inter-wave regression check catches this only if staleness causes a test failure.

**Resolution**: Add a narrow contract-staleness check to the readiness check: when a task's contracts reference a file modified by a prior wave (detectable from git diff), re-read the file and verify the specific conventions cited still hold. This is a targeted check, not a full codebase survey.

### Finding 5: Corrective workflow entry-point classification undefined
**Severity**: important

Section 4h says `/spec` recognizes corrective work, but no mechanism defines how (user statement? explicit flag? presence of failing tests?). How does the corrective classification propagate from spec to breakdown to gate? Fast-track bypasses the entire pipeline — this is a different path, not a parameter variation.

**Resolution**: Define the classification mechanism. Options: (a) a `workflow` field in spec frontmatter, (b) a flag on the `/spec` command (`/spec --corrective`), (c) a separate command (`/fix`). Pick one and trace it through every pipeline stage.

### Finding 6: CP2 UV-step fidelity check adds a new cross-reference without explicit checkpoint update
**Severity**: important

The new CP2 check requires cross-referencing UV steps in the spec → test entries in the testing strategy → test tasks in the breakdown. This is a new data flow not present in the current CP2 definition. The spec should explicitly state this is a new cross-reference operation and confirm the existing artifact set is sufficient.

## Over-engineering / Pragmatism

### Finding 7: Codebase-grounded compilation changes the breakdown agent's execution model
**Severity**: blocking (resolved by scoping)

Today the breakdown agent is a compiler: read spec, produce tasks. Codebase-grounded compilation changes it to: read spec, read source files, verify conventions, resolve mismatches, potentially halt for spec revision. Three gaps: (a) no concrete flow for "mismatch detected, return to spec" within the `/breakdown` skill, (b) for greenfield work most files don't exist yet — the value is primarily brownfield, (c) reading 10-20 files during compilation changes the cost profile.

**Resolution**: Scope codebase-grounded compilation to files that already exist (brownfield and existing files in greenfield). For greenfield tasks creating new files, verified interface contracts from the spec are sufficient. Add a concrete flow for mismatch resolution within the breakdown skill. Acknowledge the context-window cost.

### Finding 8: AC-11 metrics are premature — keep override frequency, defer the rest
**Severity**: important

Three agents independently flagged this. Defect escape rate and revision yield require longitudinal data across multiple pipeline runs. Override frequency is trivially captured by the override mechanism itself. The spec's own validation says "verify metrics are being captured" — making it a documentation task disguised as measurement.

**Resolution**: Rewrite AC-11 to require: (a) the test reviewer output includes structured fields for override rationale and finding severity (so metrics CAN be computed), (b) override frequency is captured and surfaced. Defer defect escape rate and revision yield to a phase with enough brownfield data to define baselines.

### Finding 9: Spec template enforcement mechanism unclear
**Severity**: important

AC-03 says specs lacking integration seam declarations don't pass the spec gate. But the spec gate is currently "all 9 sections present and non-empty." The new subsections (UV steps within testing strategy, seams within technical approach) require subsection-level checking. Is this enforced by the gate script or by the `/spec` skill following the guide?

**Resolution**: Clarify whether enforcement is script-based (gate script updated to check for subsection markers) or instruction-based (the `/spec` skill follows the guide). If script-based, add the gate script update to documentation impact.

### Finding 10: Gate script test plan needs specific edge cases
**Severity**: important

The testing strategy says "cover the new `category: corrective` and `category: testing-infrastructure` manifest values" without naming the cases. Needed: AC covered by test task only (valid in corrective), AC covered by neither (invalid in all categories), corrective manifest with mixed coverage, testing-infrastructure with test-only ACs.

### Finding 11: Implementation readiness check should specify test-task skip
**Severity**: informational

Item 2 of the readiness check ("tests compile") applies to implementation tasks consuming test tasks, not to test tasks creating tests. Add: "Test tasks skip item 2."

### Finding 12: Readiness check and regression check sequencing implicit
**Severity**: informational

The inter-wave regression check runs between waves (wave-advancement gate). The readiness check runs per-task within a wave (pre-implementation gate). This sequencing is logical but not stated.

**Resolution**: Add a sentence making the ordering explicit.

## Testing Strategy and Impact

### Finding 13: Seam declaration completeness not evaluated by the reviewer
**Severity**: important

The test reviewer validates whether declared seams have test coverage, but does not evaluate whether the declaration is *complete* relative to the technical approach. If a seam is undeclared, there is nothing to evaluate. Bugs 3 and 4 from the retrospective would only be caught if the spec author declared the relevant seams.

**Resolution**: Add guidance in 4a or 4b stating that the test reviewer at CP1 should also evaluate whether the spec's technical approach describes module interactions missing from the integration seam declaration.

### Finding 14: 10 of 12 ACs have no validation until brownfield runs
**Severity**: important

For context asset changes (markdown instruction files), pipeline operation is realistically the only validation. But "validated through pipeline operation" is observation during usage, not a test.

**Resolution**: Add a "dry run" validation step: after modifying each context asset, run one existing spec (e.g., a Space Invaders feature spec with known deficiencies) through the affected checkpoint to confirm new criteria activate. Even a single pass through CP1 with a smoke-test-only spec would confirm Tier 1 and Tier 2 criteria fire.

### Finding 15: Corrective Tier 2 "show your work" needs a third variant
**Severity**: important

The spec handles two cases: proposed new tests (standard) and existing failing tests (corrective). Missing: existing passing tests that must remain passing after the fix. A corrective spec may include UV steps for regression protection — these are covered by tests that already pass and need no new work.

**Resolution**: Add: "UV-N is covered by [existing test], which currently passes. This test must continue to pass after the fix."

### Finding 16: Missing integration seam for corrective workflow entry point
**Severity**: important

The corrective workflow documentation depends on the `/spec` skill recognizing corrective work. This is an integration seam not declared in the spec's own integration seam section.

### Finding 17: The spec's own testing strategy does not follow the UV-step mapping format it prescribes
**Severity**: blocking

The spec requires test entries to reference UV steps ("E2e test: fire projectile — covers UV-1"). The spec's own testing strategy references ACs, not UV steps. The spec does not eat its own dog food.

**Resolution**: Restructure the testing strategy's test entries to reference UV steps, not ACs.

## Test Strategy Review

**Result**: FAIL

9 ACs lack concrete test descriptions sufficient to produce test tasks. The Tier 2 bulk deferral ("validated through pipeline operation") is a single vague sentence covering 10 ACs without naming inputs, observable outputs, or failure modes. UV steps partially compensate but only cover 5 of 12 ACs.

ACs without concrete test coverage: AC-03, AC-04, AC-05a, AC-05b, AC-05c, AC-07, AC-09, AC-10, AC-11.

The test reviewer identified specific test descriptions needed for each — see Finding 17 and the test reviewer's individual defect reports.

## User Impact / Scope Creep

### Finding 18: Scope split consideration
**Severity**: important (dissenting — Advocate rates blocking)

The Advocate recommends splitting into Phase 1.5a (testing quality: 4a, 4b, 4e) and Phase 1.5b (breakdown, corrective, operational: 4c, 4d, 4f, 4g, 4h). Rationale: testing quality improvements are highest-leverage and should ship first.

The Builder explicitly assessed "this phase is not over-engineered — the changes are proportionate." The Architect and Guardian did not flag scope. This is a dissenting view, not consensus.

The Advocate's core point is valid: if the phase takes long to ship, the highest-impact changes (test reviewer + spec template) are delayed. Whether this warrants a split depends on implementation timeline.

### Finding 19: 3-8 UV step target needs flexibility
**Severity**: important

The "3-8 steps per feature" target has no empirical basis. Infrastructure or internal features may have fewer. Rigid floor of 3 could force artificial UV steps.

**Resolution**: Frame as "typically 3-8 for user-facing features" and allow fewer with documented rationale for infrastructure or internal work.

### Finding 20: Permissions audit (4g) is operationally independent
**Severity**: important

Both Builder and Advocate flag this. The audit produces no immediate friction reduction, has no dependency on other sections, and could run as a separate tiny spec in parallel.

**Resolution**: Consider extracting 4g into its own spec. Alternatively, keep it but acknowledge it's a parallel workstream, not a prerequisite for the testing quality improvements.

## Threat Model Determination

**Security-relevant characteristics**: This feature modifies context assets (markdown instruction files) and one gate script. No authentication, data handling, trust boundary changes, or external API interaction. The override mechanism is a workflow control, not a security control, in the current solo-developer context.

**Decision**: Threat model not needed. No new trust boundaries, no data handling changes, no external API interaction. Security considerations from the override mechanism are documented in the council's prior analysis (structured reason categories, audit trail, override frequency monitoring).

## Testing Strategy

**New tests needed**: Updated gate test for `category` field values. Dry-run validation of context asset changes against existing specs (recommended by Finding 14). UV steps UV-1 through UV-6 provide concrete validation for 5 ACs; 7 ACs need additional concrete test descriptions.

**Existing tests impacted**: `tests/sdl-workflow/test-task-reviewer.sh` — add cases for category field and edge cases per Finding 10. `tests/fixtures/tasks/` — add corrective and testing-infrastructure fixture sets.

**Test infrastructure changes**: New test fixtures for category types. No framework changes.

## Revision Log

First review — no prior revisions.

## Summary

- **3 blocking findings**: task.json schema mismatch (F1), codebase-grounded compilation flow (F7, resolved by scoping), spec testing strategy doesn't follow its own prescribed format (F17)
- **14 important findings**: tier classification, baseline mechanics, contract staleness, corrective entry point, CP2 cross-reference, AC-11 metrics, spec gate enforcement, gate test cases, seam completeness, dry-run validation, Tier 2 third variant, missing seam declaration, UV step flexibility, permissions audit independence
- **2 informational findings**: readiness check test-task skip, sequencing clarification
- **1 dissenting view**: Advocate recommends scope split; Builder, Architect, Guardian disagree
