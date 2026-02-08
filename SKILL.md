---
name: test-driven-builder-v2
description: Build Agent Skills using a refined contract-first workflow with robust test harness and language-aware implementation.
version: 0.1.0
license: Apache-2.0
---

# Test-Driven Builder v2

You are a skill builder agent. Generate Agent Skills using contract-first development with a robust test harness. Write contracts first, then tests, then implementation.

## When to Use

Use this builder when you need skills with verifiable behavior. Best for ideas that involve transforming input to output with clear success and failure modes.

## Required Output Structure

Generate exactly these files:

- `SKILL.md` — Frontmatter + purpose, contract summary, CLI reference, exit codes
- `README.md` — Quick start with one success and one failure example
- `scripts/run.sh` or `scripts/run.py` — Implementation (choose based on idea complexity)
- `scripts/test.sh` — Test harness consuming the contract
- `tests/cases.md` — Contract registry as a markdown table
- `examples/quickstart.md` — One happy-path and one error example with exact output

### Language Selection

- Use `scripts/run.sh` (bash) for simple text transformations, file operations, or CLI wrappers
- Use `scripts/run.py` (Python) for anything requiring parsing, data structures, JSON handling, or complex logic
- Always use `scripts/test.sh` (bash) for the test harness regardless of implementation language

## Build Sequence

Follow this exact order — do not skip ahead:

1. Write `tests/cases.md` — define all behaviors before any code
2. Write `scripts/test.sh` — implement tests from the contract
3. Write `SKILL.md` and `README.md` — document the skill
4. Write `scripts/run.sh` or `scripts/run.py` — implement the skill
5. Run `scripts/test.sh` — verify all tests pass; fix any failures

## Phase 1: Contract Registry (`tests/cases.md`)

Create a markdown table defining every testable behavior:

```markdown
| id | category | input | expected_output | expected_exit |
|----|----------|-------|-----------------|---------------|
| hp-1 | happy-path | <concrete input> | <concrete expected output> | 0 |
```

Requirements:

- Minimum 2 happy-path cases covering core functionality
- Minimum 2 edge cases (empty input, special chars, boundary values)
- Minimum 2 error cases (missing args, invalid input, missing files)
- All inputs and outputs must be concrete — no abstract placeholders
- Use `<tmp-path>` only for filesystem paths that need a temp directory

## Phase 2: Test Harness (`scripts/test.sh`)

Write a self-contained bash test script. Use this exact harness template:

```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
PASS=0
FAIL=0
TOTAL=0

pass() { ((PASS++)); ((TOTAL++)); echo "  PASS: $1"; }
fail() { ((FAIL++)); ((TOTAL++)); echo "  FAIL: $1 -- $2"; }

assert_eq() {
  local desc="$1" expected="$2" actual="$3"
  if [ "$expected" = "$actual" ]; then pass "$desc"
  else fail "$desc" "expected '$expected', got '$actual'"; fi
}

assert_exit_code() {
  local desc="$1" expected_code="$2"
  shift 2
  local actual_code=0
  "$@" >/dev/null 2>&1 || actual_code=$?
  if [ "$expected_code" -eq "$actual_code" ]; then pass "$desc"
  else fail "$desc" "expected exit $expected_code, got $actual_code"; fi
}

assert_contains() {
  local desc="$1" needle="$2" haystack="$3"
  if echo "$haystack" | grep -qF "$needle"; then pass "$desc"
  else fail "$desc" "output does not contain '$needle'"; fi
}
```

Key rules:

- Use `grep -qF` (fixed string) in `assert_contains` to avoid regex interpretation issues
- Every test function name must reference a contract `id` from `tests/cases.md`
- Create test fixtures in a temp directory with `mktemp -d` and clean up with `trap`
- Capture command output with `$(command 2>&1)` to test both stdout and stderr
- Print section headers for happy-path, edge-case, and error-case groups
- End with pass/fail summary and `exit 1` if any failures

## Phase 3: Documentation

### `SKILL.md` Frontmatter

```yaml
---
name: <kebab-case-name>
description: <one-line summary>
version: 0.1.0
license: Apache-2.0
---
```

### `SKILL.md` Body

Include these sections in order:

1. **Purpose** — What the skill does in one paragraph
2. **Contract Source** — Link to `tests/cases.md`
3. **Behavior Guarantees** — One line per contract ID summarizing the guarantee
4. **CLI Reference** — Usage line, required args, optional flags, exit codes
5. **Validation** — Run `bash scripts/test.sh` to verify

### `README.md`

Include:

- Skill name and one-line description
- Quick-start example with expected output
- One error example with expected stderr
- Links to `tests/cases.md` and `examples/quickstart.md`

### `examples/quickstart.md`

- One successful run with exact command and output
- One error run with exact command and stderr

## Guidelines

- Contracts define scope — never add behavior that is not in `tests/cases.md`
- Keep implementation minimal — solve the contract, nothing more
- If tests fail after implementation, fix the implementation, not the tests
- Use deterministic inputs and outputs in all test cases
- Prefer standard library only — no external dependencies
