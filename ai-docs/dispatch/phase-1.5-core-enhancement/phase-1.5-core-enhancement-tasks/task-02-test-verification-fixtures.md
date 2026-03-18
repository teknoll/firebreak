---
id: T-02
type: test
wave: 1
covers: ["AC-01", "AC-02", "AC-03", "AC-04", "AC-05", "AC-06", "AC-07", "AC-09", "AC-10", "AC-11", "AC-12"]
files_to_create: ["../../tests/fixtures/phase-1.5-validation/deficient-spec.md", "../../tests/fixtures/phase-1.5-validation/well-formed-spec.md", "../../tests/fixtures/phase-1.5-validation/validation-checklist.md"]
completion_gate: "Fixture specs and validation checklist exist; checklist enumerates the specific observable behaviors to confirm during dry-run and brownfield validation"
---

## Objective

Creates verification fixtures for semi-manual validation of Phase 1.5 context asset changes — a deficient spec (triggers new criteria), a well-formed spec (passes new criteria), and a validation checklist mapping each AC to its observable verification step.

**File count justification**: 3 files in the same fixture directory forming a cohesive test artifact set. The deficient spec and well-formed spec are complementary (one triggers criteria, one passes) and the validation checklist maps both to specific ACs. Splitting these across tasks would break the logical unit.

## Context

Phase 1.5 modifies context assets (agent instructions, spec templates, workflow documentation). These changes cannot be unit tested — they are validated by running the pipeline and observing agent behavior. This task creates the artifacts needed for that validation: test specs with known deficiencies that should trigger the new criteria, and a checklist the human uses during dry-run and brownfield validation.

The deficient spec should have characteristics that trigger every new criterion: smoke-test-only e2e coverage (triggers Tier 1 silent failure), jsdom-only tests for browser-rendered features (triggers Tier 2 test-level adequacy), missing integration seam declarations (triggers Tier 2 seam declaration completeness), and no user verification steps.

The well-formed spec should include all new required sections correctly: UV steps in action→outcome format, integration seam checklist, runtime-precise values, and a testing strategy with UV-step-to-test mapping.

## Instructions

1. Create `tests/fixtures/phase-1.5-validation/deficient-spec.md`:
   - A minimal feature spec (all 9 sections present) for a hypothetical browser-rendered feature.
   - Testing strategy lists only jsdom unit tests. No e2e tests.
   - Tests include at least one assertion that is solely error-absence ("verify no console errors").
   - Technical approach describes two modules interacting but has no integration seam declaration section.
   - No user verification steps section.
   - This spec should trigger: Tier 1 (silent failure), Tier 2 (test-level adequacy, behavioral completeness, seam declaration completeness).

2. Create `tests/fixtures/phase-1.5-validation/well-formed-spec.md`:
   - Same hypothetical feature as the deficient spec, but correctly structured.
   - Includes UV steps (3 steps, action→outcome format).
   - Includes integration seam declaration (checklist format, 2 seams).
   - Testing strategy references UV steps in test entries.
   - Includes at least one e2e test targeting the real browser runtime.
   - Uses runtime-precise values (not conceptual shorthand).
   - This spec should pass all Tier 1 and Tier 2 criteria.

3. Create `tests/fixtures/phase-1.5-validation/validation-checklist.md`:
   - A structured checklist mapping each AC (AC-01 through AC-12) to:
     - The UV step that validates it
     - The specific observable behavior to confirm
     - Whether validation is automated (gate test), dry-run (run against fixture spec), or brownfield (full pipeline run)
   - This checklist is the human's guide during validation — it tells them exactly what to look for.
   - The checklist must include these specific verification techniques for hard-to-validate ACs:
     - **AC-04** (council pattern consistency): Specify transcript-search criteria — search the council review transcript for explicit mentions of "pattern consistency," "integration point existence," or "convention visibility." If none appear, the criterion is not activating.
     - **AC-05** (codebase-grounded compilation): Include instructions to create a deliberate spec-vs-code mismatch (e.g., spec says `"Space"` but code uses `' '`) and verify the breakdown agent flags it. Do not rely on a mismatch occurring naturally.
     - **AC-07** (readiness check): Enumerate sabotage scenarios for all 4 readiness check items: (1) delete a prior-wave file, (2) introduce a syntax error in a test file, (3) break the build, (4) modify a prior-wave file to change a convention. Verify the agent catches each.
     - **AC-11** (structured override fields): Specify that the reviewer's output must contain extractable discrete fields (criterion name, severity, rationale category) — not prose paragraphs. The human should confirm a simple grep or regex could extract override counts from the output.

## Files to create/modify

Create:
- `tests/fixtures/phase-1.5-validation/deficient-spec.md`
- `tests/fixtures/phase-1.5-validation/well-formed-spec.md`
- `tests/fixtures/phase-1.5-validation/validation-checklist.md`

## Test requirements

The fixtures themselves are the test artifacts. The deficient spec is designed to fail the updated test reviewer criteria. The well-formed spec is designed to pass. The validation checklist provides the structure for semi-manual confirmation.

## Acceptance criteria

AC-01 through AC-12: This task produces the artifacts needed to validate all ACs. The validation itself occurs during dry-run (after context asset edits) and brownfield testing.

## Model

Sonnet

## Wave

Wave 1
