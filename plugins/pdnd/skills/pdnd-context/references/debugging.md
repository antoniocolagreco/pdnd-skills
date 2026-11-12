# Debugging And Bug Fixes

Before changing production code, reproduce the bug with a failing test or deterministic check.

## Steps

1. Understand the reported behavior, expected behavior, actor, endpoint, and package.
2. Find the closest test surface in the owning package.
3. Write the smallest test that proves the bug.
4. Run it. Confirm it fails for the reported reason.
5. Find the root cause from the failing path.
6. Fix the root cause with the smallest change.
7. Rerun the test and the targeted checks.
8. Report what failed, why, what changed, and what remains unverified.

If the user asked only for investigation, stop after step 5.

## Choosing The Reproducer

- If QA sees the bug through `backend-for-frontend`, trace the request from the BFF and reproduce the bug in the layer that runs the faulty logic.
- The reproducer must execute that logic. An API test that mocks the faulty service method cannot prove the bug.
- If the exact path cannot be automated, write the closest deterministic check and say what stays manual.
- Do not add a new test harness, Vitest config, or helper layer without asking.

## Where To Look

- `*-process`: router authorization, service flow, validators, errors, `toEvent.ts`.
- Readmodel bug: event payload, writer, SQL, Drizzle schema, splitter, aggregator, read service.
- Contract bug: OpenAPI, generated clients, protobuf, converters, BFF mapping.
- Regression: changed paths first, but still prove the bug.

## Known Causes

**Old risk analysis version served after a merge:** rulesets in `packages/commons/src/risk-analysis/rules/` are static and evaluated at startup. There is no cache to clear. Check the `version` in `GET /purposes/riskAnalysis/latest`, then ask for a restart of the `purpose-process` pod before searching the code.

## Security Bugs

Check access control, tenant and ownership checks, privilege escalation, data exposure, input handling, and unbounded work. For a confirmed issue, report severity, attack path, fix, and the regression test or why none is possible.

## Rules

- Report only what a failing test, a check, or a concrete trace proves.
- Fix the cause, not the symptom. No test-only patches.
- No refactors during a bug fix.
- Do not change the failing test to make it pass, unless the test is what is wrong and you say so.
