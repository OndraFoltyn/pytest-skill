# pytest-skill

Claude Code skill `python-tests` for writing and reviewing pytest tests.

Goal: the smallest set of tests that catches realistic bugs, not coverage.
The skill has two modes:

- **Write**: proposes test cases first (one line each: which bug it catches), writes
  them after approval, and verifies each test can fail.
- **Review**: reports wrong, redundant, weak, brittle and missing tests. It does not
  change anything.

In both modes production code stays unchanged. Bugs found by tests are reported,
not fixed.

## Install

The repository root is the skill directory, so clone it straight into the skills folder:

```bash
git clone https://github.com/OndraFoltyn/pytest-skill.git ~/.claude/skills/python-tests
```

Update:

```bash
git -C ~/.claude/skills/python-tests pull
```

For a single project only, clone into `<project>/.claude/skills/python-tests` instead.
The skill is available from the next Claude Code session.

## Global rules (recommended)

The skill loads only when you work with tests. These rules apply always, so add them
to `~/.claude/CLAUDE.md`:

```markdown
## Tests

- Fewer, meaningful tests beat many shallow ones. No tests written just for coverage.
- Every test must catch a realistic bug. If you can't name the bug, don't write the test.
- Before writing tests, list the proposed cases (one line each: what it protects against) and wait for approval.
- Don't test trivial code, constants, library/framework behavior or pure wiring.
- No new test dependencies or config changes without asking.
- Project-specific test details (how to run, fixtures, test data, module quirks) belong in the project's CLAUDE.md, not in the python-tests skill.
```

## Project-specific details

Keep how to run tests, fixtures, test data and module quirks in the project's
`CLAUDE.md`. The skill reads them from there.
