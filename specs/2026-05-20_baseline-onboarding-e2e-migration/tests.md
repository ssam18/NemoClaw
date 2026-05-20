# Test Specification: Baseline Onboarding E2E Scenario Migration

Generated from: `specs/2026-05-20_baseline-onboarding-e2e-migration/spec.md`

## Phase 1: Legacy Assertion Inventory and Classification - Test Guide

**Existing Tests to Modify:**
- `test/e2e/scenario-framework-tests/e2e-legacy-assertion-inventory.test.ts`
  - Verify scoped legacy PASS/FAIL strings are extractable.
- `test/e2e/scenario-framework-tests/e2e-parity-map.test.ts`
  - Verify mapped/deferred/retired metadata requirements.

**New Tests to Create:**
1. `baseline_inventory_should_include_scoped_legacy_scripts`
   - **Input**: The four scoped legacy scripts.
   - **Expected**: Inventory contains assertions for each script.
   - **Covers**: Every scoped script has a parity-map entry plan.
2. `baseline_parity_entries_should_classify_all_scoped_assertions`
   - **Input**: `parity-map.yaml` plus generated inventory.
   - **Expected**: No scoped assertion is missing or uncategorized.
   - **Covers**: mapped/deferred/retired classification.
3. `baseline_mapped_assertions_should_use_stable_validation_ids`
   - **Input**: Baseline mapped entries.
   - **Expected**: IDs match `validation.baseline_onboarding.*` and include layer/domain/owner.
   - **Covers**: Stable ID and metadata requirements.

**Test Implementation Notes:**
- Reuse existing parity-map validation helpers where possible.
- Do not require live E2E execution for schema metadata tests.

## Phase 2: Baseline Onboarding Primitive Library - Test Guide

**Existing Tests to Modify:**
- `test/e2e/scenario-framework-tests/e2e-lib-helpers.test.ts`
  - Add shell helper sourceability and behavior tests.

**New Tests to Create:**
1. `baseline_helper_should_source_under_strict_shell_options`
   - **Input**: `set -euo pipefail; source baseline_onboarding.sh`.
   - **Expected**: Exit code 0.
   - **Covers**: Shellcheck-friendly helper structure.
2. `baseline_helper_should_load_required_context_keys`
   - **Input**: Temp `context.env` with sandbox/provider/route keys.
   - **Expected**: Helper exports/reads normalized values successfully.
   - **Covers**: Context-first contract.
3. `baseline_helper_should_fail_when_required_context_missing`
   - **Input**: Temp `context.env` missing sandbox identity.
   - **Expected**: Stable `FAIL:` line and nonzero exit.
   - **Covers**: Failure behavior.
4. `baseline_helper_should_redact_sensitive_context_in_failures`
   - **Input**: Context with token-like values and failing mocked command.
   - **Expected**: Output omits secret values.
   - **Covers**: Secret redaction.
5. `baseline_cli_assertions_should_use_mocked_binaries`
   - **Input**: PATH containing mock `nemoclaw` and `openshell`.
   - **Expected**: `PASS: validation.baseline_onboarding.*` for CLI assertions.
   - **Covers**: CLI/OpenShell primitives.
6. `baseline_sandbox_assertions_should_report_stable_ids`
   - **Input**: Mock `nemoclaw list/status/logs` responses.
   - **Expected**: PASS/FAIL IDs for sandbox listed/status/logs.
   - **Covers**: Sandbox primitives.

**Test Implementation Notes:**
- Use temporary directories for `E2E_CONTEXT_DIR` and mock PATH.
- Assert no helper invokes install, onboard, destroy, rebuild, or global discovery commands.

## Phase 3: Suite Integration - Test Guide

**Existing Tests to Modify:**
- `test/e2e/scenario-framework-tests/e2e-suite-runner.test.ts`
- `test/e2e/scenario-framework-tests/e2e-scenario-resolver.test.ts`
- `test/e2e/scenario-framework-tests/e2e-scenario-schema.test.ts`

**New Tests to Create:**
1. `baseline_suite_should_be_defined_with_ordered_steps`
   - **Input**: `validation_suites/suites.yaml`.
   - **Expected**: `baseline-onboarding` exists with CLI, sandbox-state, and route/smoke steps.
   - **Covers**: Suite definition.
2. `baseline_steps_should_source_baseline_helper_and_context_only`
   - **Input**: Step script contents.
   - **Expected**: Scripts source helper/context and avoid setup/destructive commands.
   - **Covers**: No setup side effects.
3. `affected_scenarios_should_resolve_plan_only`
   - **Input**: `ubuntu-repo-cloud-openclaw`, `brev-launchable-cloud-openclaw`.
   - **Expected**: `run-scenario.sh <id> --plan-only` exits 0 and includes appropriate suites.
   - **Covers**: Plan-only preservation and scenario attachment.
4. `baseline_suite_should_not_attach_to_unsupported_scenarios`
   - **Input**: macOS/negative preflight scenarios.
   - **Expected**: Docker/sandbox-dependent baseline suite absent.
   - **Covers**: Platform/runtime constraints.

**Test Implementation Notes:**
- Prefer schema/resolver tests over live cloud runs.
- If the suite must split, update tests to assert split names and attachments.

## Phase 4: Parity Map and Coverage Report Visibility - Test Guide

**Existing Tests to Modify:**
- `test/e2e/scenario-framework-tests/e2e-parity-map.test.ts`
- `test/e2e/scenario-framework-tests/e2e-coverage-report.test.ts`

**New Tests to Create:**
1. `baseline_parity_map_should_pass_strict_validation`
   - **Input**: Updated `parity-map.yaml`.
   - **Expected**: `npx tsx scripts/e2e/check-parity-map.ts --strict` passes.
   - **Covers**: Metadata and classification validity.
2. `baseline_coverage_report_should_show_gap_domain_summary`
   - **Input**: Parity map with baseline entries.
   - **Expected**: Coverage output includes baseline onboarding mapped/deferred/retired counts.
   - **Covers**: Coverage visibility.
3. `baseline_deferred_entries_should_include_runner_or_secret_requirement`
   - **Input**: Deferred scoped entries.
   - **Expected**: Owner plus runner/secret metadata present.
   - **Covers**: Deferred evidence.
4. `baseline_retired_entries_should_include_reviewer_and_date`
   - **Input**: Retired scoped entries.
   - **Expected**: Reviewer and approved date present.
   - **Covers**: Retirement evidence.

## Phase 5: Integration Verification - Test Guide

**Existing Tests to Run:**
- Scenario framework Vitest files touched by implementation.
- Plan-only runs for affected scenarios.
- `run-suites.sh baseline-onboarding` with mocked/prepared context.

**New Tests to Create:**
1. `baseline_suite_should_execute_against_mock_context`
   - **Input**: Prepared `context.env` and mocked CLI/OpenShell commands.
   - **Expected**: Suite exits 0 and emits stable PASS IDs.
   - **Covers**: End-to-end suite execution without live cloud dependency.
2. `baseline_suite_failure_should_emit_stable_fail_id`
   - **Input**: Mock command failure.
   - **Expected**: Suite exits nonzero with `FAIL: validation.baseline_onboarding.*`.
   - **Covers**: Observable failure reporting.

**Validation Commands:**
- `npm test -- test/e2e/scenario-framework-tests/e2e-lib-helpers.test.ts`
- `npm test -- test/e2e/scenario-framework-tests/e2e-suite-runner.test.ts`
- `npm test -- test/e2e/scenario-framework-tests/e2e-scenario-resolver.test.ts`
- `npx tsx scripts/e2e/check-parity-map.ts --strict`
- `test/e2e/runtime/run-scenario.sh ubuntu-repo-cloud-openclaw --plan-only`
- `test/e2e/runtime/run-scenario.sh brev-launchable-cloud-openclaw --plan-only`

## Phase 6: Clean the House - Test Guide

**Existing Tests to Run:**
- Formatting/lint checks for touched shell, YAML, and TypeScript files.
- Final `git status --short` inspection.

**New Tests to Create:**
1. `baseline_migration_should_leave_no_temporary_files`
   - **Input**: Repository tree after implementation.
   - **Expected**: No debug/temp files under `test/e2e` or spec directory.
   - **Covers**: Cleanup acceptance criteria.
2. `baseline_docs_should_reference_new_suite_if_conventions_changed`
   - **Input**: `test/e2e/docs/README.md` when suite conventions changed.
   - **Expected**: Documentation reflects new stable-ID/parity expectations.
   - **Covers**: Contributor-facing docs.

## Cross-Phase Acceptance Tests

1. `baseline_validation_complete_only_after_pr_and_tests_green`
   - **Input**: PR URL/check status for the implementation branch.
   - **Expected**: PR is open and all added tests pass in CI/local validation.
   - **Covers**: User-supplied validation criterion #1.
2. `baseline_legacy_e2e_parity_should_be_at_least_100_percent`
   - **Input**: Re-reviewed legacy onboarding coverage inventory vs scenario/parity map.
   - **Expected**: Existing legacy onboarding E2E coverage has 100% or greater parity: every legacy assertion is mapped, deferred with metadata, or retired with evidence; mapped high-value coverage is not lower than legacy baseline.
   - **Covers**: User-supplied validation criterion #2.
