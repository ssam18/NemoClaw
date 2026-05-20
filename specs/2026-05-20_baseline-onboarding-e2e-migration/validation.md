# Validation Plan: Baseline Onboarding E2E Scenario Migration

Generated from: `specs/2026-05-20_baseline-onboarding-e2e-migration/spec.md`
Test Spec: `specs/2026-05-20_baseline-onboarding-e2e-migration/tests.md`

## Overview

**Feature**: Migrate baseline onboarding legacy E2E assertions into NemoClaw's layered scenario framework with stable validation IDs and parity visibility.

**Available Tools**: Bash, Vitest, `npx tsx`, scenario runner scripts, suite runner scripts, GitHub CLI (`gh`) for PR/check validation.

## Completion Gate

Validation is complete only when both criteria are satisfied:

1. The implementation PR is opened and all added tests are passing.
2. Existing legacy onboarding E2E coverage is re-reviewed and shows 100% or greater parity in test coverage. For this plan, parity means every scoped legacy assertion is mapped, deferred with owner/runner/secret metadata, or retired with reviewer/date evidence, and migrated high-value assertions preserve or exceed legacy coverage.

## Coverage Summary

- Happy Paths: 8 scenarios
- Sad Paths: 5 scenarios
- Total: 13 scenarios

---

## Phase 1: Legacy Assertion Inventory and Classification - Validation Scenarios

### Scenario 1.1: Scoped Legacy Assertions Are Fully Classified [STATUS: passed] [VALIDATED: cef7748]
**Type**: Happy Path

**Given**: The four scoped legacy scripts exist and the parity inventory can be generated.
**When**: The parity map is updated for `test-full-e2e.sh`, `test-cloud-onboard-e2e.sh`, `test-cloud-inference-e2e.sh`, and `test-onboard-inference-smoke.sh`.
**Then**: Every scoped assertion is categorized as mapped, deferred, or retired with required metadata.

**Validation Steps**:
1. **Setup**: Bash: confirm all four scripts exist.
2. **Execute**: `npx tsx scripts/e2e/extract-legacy-assertions.ts --output /tmp/baseline-parity-inventory.json`.
3. **Verify**: `npx tsx scripts/e2e/check-parity-map.ts --strict` exits 0.

**Tools Required**: Bash, `npx tsx`.

### Scenario 1.2: Unknown or Unclassified Scoped Assertions Fail Validation [STATUS: passed] [VALIDATED: cef7748]
**Type**: Sad Path

**Given**: A scoped legacy assertion is missing from the parity map.
**When**: Strict parity validation runs.
**Then**: Validation fails and identifies the missing or uncategorized assertion.

**Validation Steps**:
1. **Setup**: Bash: create a temporary parity-map fixture omitting one scoped assertion.
2. **Execute**: `npx tsx scripts/e2e/check-parity-map.ts --strict` against the fixture.
3. **Verify**: Command exits nonzero and reports the omitted assertion.

**Tools Required**: Bash, `npx tsx`.

## Phase 2: Baseline Onboarding Primitive Library - Validation Scenarios

### Scenario 2.1: Helper Library Emits Stable PASS IDs With Mocked Context [STATUS: passed] [VALIDATED: cef7748]
**Type**: Happy Path

**Given**: A prepared `E2E_CONTEXT_DIR/context.env` and mocked `nemoclaw`/`openshell` binaries.
**When**: Helper-level tests execute CLI, sandbox, logs, and route assertions.
**Then**: The helper emits `PASS: validation.baseline_onboarding.*` IDs and exits 0.

**Validation Steps**:
1. **Setup**: Bash/Vitest: create temp context and mock PATH.
2. **Execute**: `npm test -- test/e2e/scenario-framework-tests/e2e-lib-helpers.test.ts`.
3. **Verify**: Test output includes stable PASS IDs and no setup/destructive commands are invoked.

**Tools Required**: Vitest, Bash.

### Scenario 2.2: Helper Failure Redacts Secrets and Emits Stable FAIL ID [STATUS: passed] [VALIDATED: cef7748]
**Type**: Sad Path

**Given**: Context includes token-like values and a mocked command fails.
**When**: A baseline helper assertion fails.
**Then**: Output includes a stable `FAIL:` ID, actionable command snippet, and no secret value.

**Validation Steps**:
1. **Setup**: Vitest: temp context with secret-like variables and failing mock command.
2. **Execute**: Run helper failure test.
3. **Verify**: Assert nonzero exit, `FAIL: validation.baseline_onboarding.*`, and redaction.

**Tools Required**: Vitest, Bash.

## Phase 3: Suite Integration - Validation Scenarios

### Scenario 3.1: Baseline Suite Resolves In Scenario Plans [STATUS: passed] [VALIDATED: cef7748]
**Type**: Happy Path

**Given**: The baseline suite is defined and attached to supported scenarios.
**When**: Plan-only resolution runs for supported scenarios.
**Then**: Resolution succeeds and includes the expected baseline suite or launchable-compatible subset.

**Validation Steps**:
1. **Setup**: Bash: ensure scenario YAML and suite YAML are present.
2. **Execute**: `test/e2e/runtime/run-scenario.sh ubuntu-repo-cloud-openclaw --plan-only` and `test/e2e/runtime/run-scenario.sh brev-launchable-cloud-openclaw --plan-only`.
3. **Verify**: Both exit 0 and plan output includes expected suite steps.

**Tools Required**: Bash, scenario runner.

### Scenario 3.2: Unsupported Scenarios Do Not Receive Docker/Sandbox-Dependent Baseline Steps [STATUS: passed] [VALIDATED: cef7748]
**Type**: Sad Path

**Given**: macOS optional-Docker and negative preflight scenarios exist.
**When**: Their plans are resolved.
**Then**: Docker/sandbox-dependent baseline steps are absent.

**Validation Steps**:
1. **Setup**: Bash: identify macOS and negative preflight scenario IDs from `scenarios.yaml`.
2. **Execute**: Run `--plan-only` for those scenarios.
3. **Verify**: Output does not include incompatible baseline suite steps.

**Tools Required**: Bash, scenario runner.

### Scenario 3.3: Suite Runs Against Prepared Context Without Re-Onboarding [STATUS: passed] [VALIDATED: cef7748]
**Type**: Happy Path

**Given**: A mocked/prepared context and mocked external commands.
**When**: `run-suites.sh baseline-onboarding` executes.
**Then**: The suite passes and does not run install, onboard, destroy, rebuild, or rediscovery commands.

**Validation Steps**:
1. **Setup**: Bash: create temp `E2E_CONTEXT_DIR/context.env` and mock command directory.
2. **Execute**: `test/e2e/runtime/run-suites.sh baseline-onboarding` with the temp environment.
3. **Verify**: Exit 0, stable PASS IDs emitted, mock invocation log has no forbidden commands.

**Tools Required**: Bash, suite runner.

## Phase 4: Parity Map and Coverage Report Visibility - Validation Scenarios

### Scenario 4.1: Coverage Report Shows Baseline Onboarding Parity [STATUS: passed] [VALIDATED: cef7748]
**Type**: Happy Path

**Given**: Baseline parity entries are present with `gap_domain: baseline-onboarding`.
**When**: The coverage report is generated.
**Then**: The report shows mapped/deferred/retired counts for baseline onboarding.

**Validation Steps**:
1. **Setup**: Bash: ensure parity map and inventory fixture are available.
2. **Execute**: `test/e2e/runtime/coverage-report.sh`.
3. **Verify**: Output includes baseline onboarding domain and nonzero classified counts.

**Tools Required**: Bash, coverage-report script.

### Scenario 4.2: Deferred and Retired Entries Without Evidence Fail Review [STATUS: passed] [VALIDATED: cef7748]
**Type**: Sad Path

**Given**: A deferred entry lacks owner/runner/secret metadata or a retired entry lacks reviewer/date evidence.
**When**: Parity validation or focused metadata review runs.
**Then**: The entry is rejected before implementation is considered complete.

**Validation Steps**:
1. **Setup**: Bash: create temporary invalid parity-map fixtures.
2. **Execute**: Run strict parity validation and/or focused Vitest metadata test.
3. **Verify**: Nonzero failure identifies the incomplete evidence.

**Tools Required**: Bash, `npx tsx`, Vitest.

## Phase 5: Integration Verification - Validation Scenarios

### Scenario 5.1: Added Tests Pass Locally and in PR Checks [STATUS: passed] [VALIDATED: cef7748]
**Type**: Happy Path

**Given**: Implementation changes and tests have been pushed to a PR.
**When**: Local validation commands and GitHub PR checks complete.
**Then**: All added tests pass and the PR remains open.

**Validation Steps**:
1. **Setup**: Git/GitHub CLI: open the implementation PR.
2. **Execute**: Run targeted local tests plus `gh pr checks --watch` for the PR.
3. **Verify**: All added tests pass locally and in PR checks.

**Tools Required**: Bash, Vitest, `npx tsx`, `gh`.

### Scenario 5.2: PR Is Not Accepted When Added Tests Fail [STATUS: passed] [VALIDATED: cef7748]
**Type**: Sad Path

**Given**: The implementation PR exists but any added test fails.
**When**: Completion is evaluated.
**Then**: Validation remains incomplete and the failing test is treated as a blocker.

**Validation Steps**:
1. **Setup**: GitHub CLI: locate PR check results.
2. **Execute**: `gh pr checks` for the implementation PR.
3. **Verify**: Any failed added-test check blocks completion.

**Tools Required**: `gh`.

## Phase 6: Clean the House - Validation Scenarios

### Scenario 6.1: Final Diff Is Scoped and Clean [STATUS: passed] [VALIDATED: cef7748]
**Type**: Happy Path

**Given**: Implementation and validation are complete.
**When**: Final hygiene checks run.
**Then**: No temporary files remain, docs are updated only if conventions changed, and the diff is scoped to the E2E migration.

**Validation Steps**:
1. **Setup**: Bash: inspect changed and untracked files.
2. **Execute**: Run formatting/lint checks for touched files.
3. **Verify**: `git status --short` contains only intentional changes before commit/PR.

**Tools Required**: Bash, project lint/format tools.

### Scenario 6.2: Legacy Onboarding Coverage Re-Review Shows 100%+ Parity [STATUS: passed] [VALIDATED: cef7748]
**Type**: Happy Path

**Given**: The implementation PR is open and all added tests pass.
**When**: Existing legacy onboarding E2E coverage is re-reviewed against the updated scenario suite and parity map.
**Then**: Coverage is 100% or greater parity: every scoped legacy assertion is mapped, deferred with metadata, or retired with evidence, and high-value mapped checks are not reduced relative to legacy coverage.

**Validation Steps**:
1. **Setup**: Generate or inspect the current legacy assertion inventory for the four scoped scripts.
2. **Execute**: Compare the inventory to the updated parity map and suite assertion IDs.
3. **Verify**: No scoped assertion is uncategorized; mapped/deferred/retired counts equal total scoped assertions; document the re-review in the PR test plan.

**Tools Required**: Bash, `npx tsx`, coverage-report script, GitHub PR description/checklist.

## Summary

| Phase | Happy | Sad | Total | Passed | Failed | Pending |
|-------|-------|-----|-------|--------|--------|---------|
| Phase 1 | 1 | 1 | 2 | 2 | 0 | 0 |
| Phase 2 | 1 | 1 | 2 | 2 | 0 | 0 |
| Phase 3 | 2 | 1 | 3 | 3 | 0 | 0 |
| Phase 4 | 1 | 1 | 2 | 2 | 0 | 0 |
| Phase 5 | 1 | 1 | 2 | 2 | 0 | 0 |
| Phase 6 | 2 | 0 | 2 | 2 | 0 | 0 |
| **Total** | **8** | **5** | **13** | **13** | **0** | **0** |

## Approval Status

APPROVED by supplied task criteria, contingent on the Completion Gate above remaining present in the plan.
