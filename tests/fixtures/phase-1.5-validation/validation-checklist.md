# Phase 1.5 Validation Checklist

Use this checklist during dry-run and brownfield validation after applying Phase 1.5 context asset changes. Each row maps one acceptance criterion to the specific observable behavior to confirm and the verification method to use.

**Validation method key**:
- **Gate**: automated gate test (shell test suite, `tests/sdl-workflow/`).
- **Dry-run**: run the named UV step against the fixture spec and observe agent output.
- **Brownfield**: full pipeline run on a real feature; confirm behavior in context.

---

## AC-01 — Tier 1 silent failure detection

**Criterion**: The test reviewer applies Tier 1 (silent failure detection) as a non-overridable hard gate. A spec whose testing strategy contains only error-absence assertions for user-facing behavior does not pass CP1.

| Step | Method | What to confirm |
|------|--------|-----------------|
| UV-1: Run `deficient-spec.md` through `/spec-review`. | Dry-run | The test reviewer output contains a Tier 1 finding. The finding must name the specific test entries that are error-absence assertions (e.g., "verify no console errors"). The output must include a hard-gate block — not an informational note. No override path is offered for this finding. |
| UV-1: Run `well-formed-spec.md` through `/spec-review`. | Dry-run | No Tier 1 finding is raised. The reviewer confirms behavioral assertions are present and linked to UV steps. |

**Pass condition**: deficient spec is blocked; well-formed spec is not. The word "override" must not appear in the Tier 1 section of the reviewer output for the deficient spec.

---

## AC-02 — Tier 2 structured "show your work" output

**Criterion**: The test reviewer applies Tier 2 criteria with structured output. The reviewer names the specific test covering each UV step and describes how that test would fail. For test-level adequacy, the reviewer cites the runtime-dependent indicator. Tier 2 findings accept overrides only from a defined reason category; freeform justification is rejected.

| Step | Method | What to confirm |
|------|--------|-----------------|
| UV-2: Run `deficient-spec.md` through `/spec-review`. | Dry-run | Tier 2 test-level adequacy finding is raised. The output names "Canvas API" (or `CanvasRenderingContext2D`) as the runtime-dependent indicator that triggered the flag. The output explains that jsdom stubs Canvas API calls, making the tests incapable of detecting rendering failures in real browsers. |
| UV-3: Run `well-formed-spec.md` through `/spec-review`. | Dry-run | The reviewer lists each UV step (`UV-1`, `UV-2`, `UV-3`) and names the specific test that covers it (e.g., "UV-1 → `chart-render.e2e.spec.js`"). For each mapping, the reviewer states how the test would fail if the behavior were absent (e.g., "pixel diff assertion would fail if canvas is blank"). |
| Override rejection: Submit a Tier 2 finding override with freeform prose ("this is fine because Canvas is hard to test"). | Dry-run | The reviewer rejects the override and requests a reason from the defined category list (e.g., `"runtime-unavailable"`, `"deferred-to-phase-N"`). |

**Pass condition**: reviewer output contains named UV-to-test mappings with failure mode descriptions; freeform override is rejected.

---

## AC-03 — Spec template includes UV steps and integration seam declaration sections

**Criterion**: The spec template includes a user verification steps subsection (action→outcome format) and an integration seam declaration subsection (checklist format). The `/spec` skill prompts for both. The test reviewer's seam declaration completeness criterion (Tier 2) flags a spec with module interactions but no seam declaration.

| Step | Method | What to confirm |
|------|--------|-----------------|
| UV-7: Run `deficient-spec.md` through `/spec-review`. | Dry-run | Tier 2 seam declaration completeness finding is raised. The output notes that the technical approach describes two interacting modules (`chart-controller.js` and `chart-renderer.js`) but no integration seam declaration section is present. |
| Authoring check: invoke `/spec` on a new feature prompt. | Dry-run | The skill explicitly prompts for (1) user verification steps in action→outcome format and (2) integration seam declarations in checklist format. Both prompts appear before the spec is finalized. |
| Structure check: open `well-formed-spec.md`. | Dry-run | Confirm the spec contains a "User verification steps" subsection with `UV-N: Action → Expected outcome` entries and an "Integration seam declaration" subsection with `- [ ] Seam N —` checklist entries. |

**Pass condition**: deficient spec triggers the seam completeness finding; well-formed spec does not; `/spec` skill prompts for both sections.

---

## AC-04 — Council review evaluates pattern consistency

**Criterion**: The council review evaluates whether the proposed approach follows existing patterns, integration points referenced in the spec exist in the code, and conventions tasks will need to follow are flagged for breakdown discovery.

**Verification technique (transcript search)**: After running a council review on a spec that integrates with existing modules, search the full council transcript for the following phrases. Absence of all three indicates the criterion is not activating.

- `"pattern consistency"` — should appear when the reviewer checks whether the proposed approach follows existing module patterns.
- `"integration point existence"` — should appear when the reviewer verifies that integration points named in the spec (e.g., `store.getState()`, `CHART_MODE_CHANGED`) actually exist in the codebase.
- `"convention visibility"` — should appear when the reviewer flags conventions that breakdown tasks must follow (e.g., action type string format, context injection pattern).

| Step | Method | What to confirm |
|------|--------|-----------------|
| Brownfield run: submit `well-formed-spec.md` to council review against a real codebase that has a state store. | Brownfield | Search the council transcript for at least two of the three phrases above. If zero appear, the council criterion is not activating — investigate whether the skill instruction was applied. |

**Pass condition**: at least two of the three phrases appear in the council review transcript. All three should appear for a spec with meaningful integration surface.

---

## AC-05 — Breakdown produces codebase-grounded interface contracts

**Criterion**: Task instructions for shared-interface tasks include contracts (import convention, module type, key strings, wiring patterns). Orchestrator tasks are routed to Sonnet minimum with wiring checklists. Contracts match actual code — mismatches are flagged during compilation, not silently passed through.

**Verification technique (deliberate mismatch)**: Do not rely on a natural mismatch occurring. Before running UV-9, introduce a deliberate discrepancy between the spec and the actual code:

1. Edit `well-formed-spec.md` locally (do not commit): change the seam declaration to say the mode values are `"Space"` and `"Bar"` (capitalized strings with `"Space"` instead of `"line"`).
2. Ensure the actual store implementation uses `'line'` and `'bar'` (lowercase string literals).
3. Run the breakdown agent against the modified spec and the real codebase.
4. Confirm the breakdown agent flags the mismatch — output should identify that the spec specifies `"Space"` but the code uses `'line'`.

| Step | Method | What to confirm |
|------|--------|-----------------|
| UV-8: Run breakdown on `well-formed-spec.md` against a real codebase. | Brownfield | Task files for shared-interface tasks contain an interface contract section with at minimum: import convention (e.g., `import { store } from '../store/index.js'`), module type, key strings (`'line'`, `'bar'`, `'CHART_MODE_CHANGED'`), and wiring patterns. Orchestrator task is assigned Sonnet and includes a wiring checklist. |
| UV-9 (deliberate mismatch): Run the modified spec (with `"Space"` vs `'line'` discrepancy) through the breakdown agent. | Dry-run | The breakdown agent output contains an explicit flag identifying the spec-vs-code mismatch. The agent does not silently use the spec value or the code value — it reports the conflict and halts or escalates. |

**Pass condition**: UV-8 confirms contracts appear in task files; UV-9 deliberate mismatch is caught and reported.

---

## AC-06 — CP2 test reviewer verifies UV step coverage in breakdown

**Criterion**: The test reviewer at CP2 verifies that every UV step from the spec has at least one corresponding test task in the breakdown. A breakdown that omits test tasks for UV steps present in the approved testing strategy does not pass CP2.

| Step | Method | What to confirm |
|------|--------|-----------------|
| UV-4: Run a breakdown of `well-formed-spec.md` through CP2 task review. | Dry-run | The CP2 output explicitly lists each UV step (`UV-1`, `UV-2`, `UV-3`) and names the test task that covers it. Coverage is confirmed or flagged for each step — the reviewer does not skip UV steps silently. |
| Negative test: construct a breakdown that includes UV-3 in the spec's testing strategy but omits the `chart-legend-hover.e2e.spec.js` test task. Submit to CP2. | Dry-run | CP2 blocks the breakdown, citing the missing test task for UV-3. The output names the specific UV step that lacks coverage. |

**Pass condition**: complete coverage passes; missing UV-3 test task causes CP2 to block.

---

## AC-07 — Implementation agent readiness check

**Criterion**: Each implementation agent performs a readiness check before writing code: prior-wave files exist, paired test task's tests compile, and the build succeeds. On failure, the agent reports the mismatch without attempting implementation.

**Sabotage scenarios**: Validate all four sabotage scenarios in sequence. Each should be tested independently (restore state between tests).

| Sabotage | Method | What to confirm |
|----------|--------|-----------------|
| **(1) Delete a prior-wave file**: Before running a Wave 2 implementation task, delete the Wave 1 output file that the Wave 2 task depends on (e.g., delete `chart-renderer.js`). | Dry-run / UV-10 | The agent's readiness check reports that the prior-wave file is absent. The agent does not proceed to write any code. Output contains an explicit readiness check failure message naming the missing file. |
| **(2) Introduce a syntax error in a test file**: Before running an implementation task, add a syntax error (e.g., unclosed brace) to the paired test file (e.g., `chart-renderer.test.js`). | Dry-run / UV-10 | The agent's readiness check reports that the paired test file does not compile. The agent does not proceed to write implementation code. Output contains a readiness check failure message citing the compile error. |
| **(3) Break the build**: Before running an implementation task, introduce a breaking change in an unrelated file that causes `npm run build` (or equivalent) to fail. | Dry-run / UV-10 | The agent's readiness check detects the broken build. The agent does not proceed. Output contains a readiness check failure message citing build failure. |
| **(4) Modify a prior-wave file to change a convention**: Before running a Wave 2 task, edit the Wave 1 file to rename a key string (e.g., change `'CHART_MODE_CHANGED'` to `'SET_CHART_MODE'`). | Dry-run / UV-10 | The agent's readiness check detects the convention mismatch (the current prior-wave file no longer matches the interface contract recorded in the task). The agent does not proceed. Output names the changed convention. |

**Pass condition**: all four sabotage scenarios are caught. In every case, no implementation code is written and the agent's output contains a specific, named readiness check failure — not a generic error.

---

## AC-08 — Task reviewer gate accepts corrective and testing-infrastructure categories

**Criterion**: The task reviewer gate accepts `category: corrective` and `category: testing-infrastructure` in the task.json manifest and applies relaxed invariants. Default behavior for `category: feature` (or absent) is unchanged.

| Step | Method | What to confirm |
|------|--------|-----------------|
| UV-5a: Submit a task manifest with `category: corrective` where ACs are covered by the test task only (no paired implementation task). | Gate | Gate passes. The relaxed invariant for corrective tasks allows test-only AC coverage. |
| UV-5b: Submit a `category: corrective` manifest where ACs are covered by neither test nor implementation tasks. | Gate | Gate rejects. Even corrective tasks require at least one task covering each AC. |
| UV-5c: Submit a `category: testing-infrastructure` manifest with test-only ACs. | Gate | Gate passes. |
| UV-5d: Submit a `category: feature` manifest (or omit the category field) with test-only AC coverage. | Gate | Gate rejects. Existing behavior is preserved — feature tasks require the standard coverage invariant. |
| UV-5e: Submit a `category: corrective` manifest with mixed coverage (some ACs have both test+impl, some have test-only). | Gate | Gate passes. Mixed coverage within a corrective manifest is acceptable. |

**Pass condition**: all five gate tests pass (see `tests/sdl-workflow/test-task-reviewer.sh` for automated gate coverage).

---

## AC-09 — Baseline test guard on wave advancement

**Criterion**: Before Wave 1, the full test suite runs and passing tests are captured as baseline. After each wave, baseline tests run — any baseline regression blocks wave advancement. After the final wave, the full suite runs and all tests must pass.

| Step | Method | What to confirm |
|------|--------|-----------------|
| UV-11: Run a multi-wave implementation. Introduce a deliberate regression in Wave N that breaks one baseline test. Attempt Wave N+1 advancement. | Dry-run | Wave N+1 advancement is blocked. Output identifies the specific failing baseline test and the wave that introduced the regression. |
| Positive path: run a multi-wave implementation with no regressions. | Dry-run | Each wave advances without block. After the final wave, the full test suite runs and passes. |

**Pass condition**: regression blocks advancement; no-regression path completes cleanly.

---

## AC-10 — Corrective workflow documentation completeness

**Criterion**: The corrective workflow is documented with both modes (full diagnostic and fast-track), escalation criteria, the specific differences from the standard flow, and coverage of test-first ordering, context-independent diagnosis, iterative retest cycles, and ceremony-level variation.

| Step | Method | What to confirm |
|------|--------|-----------------|
| UV-12: Open `corrective-workflow.md` (produced by T-05 or equivalent) and review structure. | Dry-run | Document contains explicitly labelled sections for: (1) full diagnostic mode, (2) fast-track mode, (3) escalation criteria (conditions for moving from fast-track to full diagnostic), (4) differences from the standard spec-driven flow. Within the body, confirm coverage of: test-first ordering, context-independent diagnosis, iterative retest cycles, and ceremony-level variation within a corrective arc. |

**Pass condition**: all four top-level sections present; all four body topics covered by searchable prose. Confirm by checking for the presence of the strings `"fast-track"`, `"full diagnostic"`, `"escalation"`, and `"test-first"` in the document.

---

## AC-11 — Structured override fields in test reviewer output

**Criterion**: The test reviewer output includes structured fields for override rationale and finding severity, enabling metric computation. Override frequency is captured and surfaced visibly.

**Verification technique (extractability check)**: The reviewer's output must contain discrete structured fields — not prose paragraphs — for each finding that supports override. A simple grep or regex must be able to extract override counts. Confirm by running:

```
grep -E "^severity:|^criterion:|^rationale-category:" <reviewer-output-file>
```

This command must return matches. If the fields exist only as prose (e.g., "The severity is medium because..."), the criterion is not met.

| Step | Method | What to confirm |
|------|--------|-----------------|
| Run `well-formed-spec.md` through `/spec-review` and capture the raw output to a file. | Dry-run | Run the grep command above against the captured output. Confirm `severity:`, `criterion:`, and `rationale-category:` fields are present as discrete lines or structured blocks — not embedded in prose. |
| Count override frequency: submit two specs with Tier 2 findings and one override each. | Dry-run | The pipeline summary output (or a log artifact) shows the override count (e.g., "2 Tier 2 overrides accepted in this run"). The count is visible in the output without manual parsing. |

**Pass condition**: grep command extracts structured fields; override count appears in pipeline summary.

---

## AC-12 — Permissions friction audit completed

**Criterion**: Static analysis of allowlist rules against pipeline tool call patterns produces a predicted prompt map. A `PreToolUse` logging hook is designed for empirical validation. The audit identifies the most frequent non-safety-critical prompt sources and candidates for helper-script encapsulation.

| Step | Method | What to confirm |
|------|--------|-----------------|
| UV-6: Run the static analysis tool against the current allowlist rules. | Gate / Dry-run | Output is a predicted prompt map that lists each pipeline tool call, whether it is in the allowlist, and whether it is predicted to trigger a prompt. The map is structured (not prose) — each row covers one tool call pattern. |
| Design review: locate the `PreToolUse` hook design artifact (produced by the audit task). | Dry-run | A hook design exists that describes: the hook trigger condition, the fields it logs (tool name, arguments, timestamp, pipeline stage), and how the log would be consumed for empirical validation. |
| Audit findings: read the audit output. | Dry-run | The audit names at least one frequent non-safety-critical prompt source and at least one candidate for helper-script encapsulation. Findings are specific (naming the tool call pattern) rather than generic. |

**Pass condition**: predicted prompt map exists with per-call-pattern rows; `PreToolUse` hook design exists; audit names specific findings.

---

## Summary table

| AC | Validation method | Fixture used | Key observable |
|----|-------------------|--------------|----------------|
| AC-01 | Dry-run (UV-1) | `deficient-spec.md` | Tier 1 hard-gate block in reviewer output; no override offered |
| AC-02 | Dry-run (UV-2, UV-3) | Both fixtures | UV-to-test mappings with failure mode descriptions; freeform override rejected |
| AC-03 | Dry-run (UV-7) | `deficient-spec.md` | Tier 2 seam completeness finding raised; `/spec` skill prompts for both sections |
| AC-04 | Brownfield | Real codebase spec | Transcript contains "pattern consistency", "integration point existence", or "convention visibility" |
| AC-05 | Dry-run + Brownfield (UV-8, UV-9) | `well-formed-spec.md` + deliberate mismatch | Contracts in task files; deliberate mismatch flagged explicitly |
| AC-06 | Dry-run (UV-4) | `well-formed-spec.md` | CP2 lists UV-to-test-task coverage; missing task triggers block |
| AC-07 | Dry-run (UV-10) | 4 sabotage scenarios | All 4 sabotage scenarios caught; no code written on failure |
| AC-08 | Gate (UV-5a–e) | Task manifests | All 5 gate test cases pass per `test-task-reviewer.sh` |
| AC-09 | Dry-run (UV-11) | Multi-wave run | Baseline regression blocks advancement; clean path completes |
| AC-10 | Dry-run (UV-12) | `corrective-workflow.md` | 4 sections present; 4 body topics covered; key strings grep-able |
| AC-11 | Dry-run | Any spec with Tier 2 finding | `severity:`, `criterion:`, `rationale-category:` fields grep-able from output |
| AC-12 | Gate / Dry-run (UV-6) | Allowlist + hook design | Predicted prompt map exists; hook design exists; specific findings named |
