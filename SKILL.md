---
name: python-tests
description: Write new Python tests with pytest, or review and verify existing ones. Use when asked to write, add, fix, check, review or verify tests in a Python project.
---

# Python tests

Goal: the smallest set of tests that would catch realistic bugs. Quality over count.
Coverage percentage is not a goal.

## Mode
Each run is exactly one mode:
- **Write**: create new tests or fix existing ones.
- **Review**: review and verify existing tests. No tests are written or changed.
Pick the mode from the request. If it is unclear, ask before doing anything.
Don't switch modes mid-run; suggest the other mode in the report instead.

## Before starting (both modes)
- Read the project's CLAUDE.md for test specifics (how to run, fixtures, test data).
  If they are missing and you had to figure them out, propose adding them there in the report.
- Read the code under test, existing tests and conftest.py. Follow the existing layout
  and style. Existing conftest.py fixtures take precedence over the rules below.
- Find how tests are run: project CLAUDE.md, pyproject.toml, Makefile, tox.ini, CI config.
  Use the project's environment. Don't `pip install` anything; ask if a dependency is missing.

## What deserves a test
- Public behavior other code relies on (the contract, not the implementation).
- Non-trivial logic: parsing, mapping, branching, calculations, rule matching.
- Each fixed bug: one regression test reproducing it.
- Edge cases from real data: if sample data or fixtures exist, base edge cases on them.
  Don't invent.

## What does not
- Getters, constants, dataclass fields, simple delegation.
- Behavior of Python, pytest, or third-party libraries.
- Cases already covered by another test with a different input value only.
- Private helpers already exercised through the public function.

## Writing rules
- Plain pytest functions; `parametrize` instead of copy-pasted tests.
- Small, realistic inputs inline; a new fixture only when shared by 3+ tests.
- Assert on concrete values, not just "not None" or "no exception".
- Mock only at boundaries. Never mock the code under test.
  - Files: `tmp_path`. Never real project paths, never a mocked `open`.
  - Env vars, attributes, config: `monkeypatch`.
  - Network, DB, external services: `unittest.mock` (`pytest-mock` only if the project already uses it).
- Deterministic: no network, no `sleep`, fixed time and random seeds.

## Production code safety (both modes)
- Production code must end the run unchanged. Never fix it and never refactor it
  to make it testable; propose such changes in the report instead.
- If a test reveals a bug in the code under test: stop and report it
  (input, expected, actual, why you believe it is a bug). Never adjust the assertion
  to match the wrong behavior. Keep the reproducing test and ask how to proceed.

## Verifying that tests can fail
For each test, state in one sentence which change in the code would make it fail.
Only where this is not obvious, check it by temporarily breaking the code:
1. Copy the production file to the scratchpad before touching it.
2. Make the minimal break, run only the affected test, confirm it fails.
3. Restore the file from the copy and confirm with `diff` that it is identical.
   Restore before breaking anything else.
4. At the end, confirm that every temporarily changed file matches its backup,
   and state this check explicitly in the report.
A test that passes on broken code gets fixed (Write) or reported as Weak (Review).

## Write mode
1. Propose a plan: list of cases, one line each with the bug it catches.
   Wait for approval. Start small (typically 3-8 cases per module).
2. Write the tests following the writing rules.
3. Run the new test file, then the whole suite to make sure nothing else broke.
4. Verify that the tests can fail.
5. Report: number of tests, what each protects, verification result,
   production code check, bugs found, anything left untested on purpose.

## Review mode
Report, don't rewrite. No new or changed tests.
1. Run the existing tests and note failures.
2. Read each test and verify that it can fail (see above).
3. Report findings grouped:
   - Wrong: asserts behavior that contradicts the spec, docstring or clear intent;
     a passing test that cements a bug. Report it as a suspected bug.
   - Redundant: duplicates or same-path variations → suggest deleting/merging.
   - Weak: tautological asserts, asserts only on mocks, can't fail.
   - Brittle: tied to implementation details, order, timing, sleep; flaky.
   - Missing: important behavior or known bug without a test.
   Include the verification result and the production code check.
Suggesting deletions is a valid and welcome outcome. Fixes happen only in a separate
Write run after approval.

## Housekeeping
- Backups, outputs and reports go to the scratchpad, never the project.
- No changes to pytest/coverage config without approval.
