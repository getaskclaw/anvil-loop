---
name: anvil-loop-development
description: "Use when a change needs hardening beyond a happy-path pass: implement, test, get independent break/review, convert real findings into tests or doc fixes, and repeat until verified with no blockers."
version: 1.1.1
author: Ash
license: MIT
metadata:
  hermes:
    tags: [development, tdd, review, iteration, hardening, docs]
    related_skills: [test-driven-development, systematic-debugging, requesting-code-review]
---

# Anvil Loop Development

## Definition

An **anvil loop** is a hardening workflow.

Do not stop at “it works once.” Put the change on the anvil, hit it with tests and independent review, fix what bends, then hit it again until nothing important moves.

Short form:

```text
contract → implement → verify → break/review → fix → re-verify → re-break/review → stop
```

## Use When

Use an anvil loop for changes where a normal happy-path pass can hide real risk:

- deploy procedures, migrations, backups, auth, credentials, rollback, or source-of-truth changes
- CLI/tools work, filesystem/process/log handling, reports, snapshots, diffs, generated artifacts
- code touching edge cases, compatibility behavior, or production runtime paths
- docs/runbooks with commands someone may copy-paste
- user says “anvil-loop it,” “break-test it,” or “review very carefully”

Usually do **not** use it for tiny typo-only edits unless the typo changes a command, path, credential warning, or operational meaning.

## The Loop

1. **State the contract**
   - What should be true after the change?
   - What must not happen?
   - What are the dangerous assumptions?

2. **Implement the smallest useful change**
   - Avoid speculative fixes.
   - Keep unrelated edits out.

3. **Verify directly**
   - Code: run tests, build, lint/type checks if relevant.
   - Docs: smoke-check headings, code fences, paths, command safety, and secret leakage.

4. **Get independent break/review**
   - Use a fresh reviewer context when possible.
   - Ask for blockers, contradictions, unsafe steps, missing edge cases, and misleading wording.

5. **Classify findings**
   - **Blocker:** must fix before shipping.
   - **Non-blocker:** useful, but safe to ship without it.
   - **Accepted tradeoff:** limitation is real and documented.
   - **Deferred:** belongs in a later issue/task.

6. **Convert real findings into evidence first**
   - Code finding → failing test or minimal repro.
   - Doc/runbook finding → exact bad line, missing warning, or unsafe command example.
   - Do not patch blindly from vibes.

7. **Fix only the proven problem**
   - Small, targeted fix.
   - Do not widen the blast radius.

8. **Re-run verification and re-review**
   - Re-run the relevant checks after any patch, including doc-only patches.
   - If behavior, commands, safety wording, or generated artifacts changed, run the full relevant suite/checks again.
   - If a reviewer found blockers, get a fresh review after fixes.

9. **Stop only when the stop conditions are met**

## Stop Conditions

Stop the anvil loop when all are true:

- the stated contract is satisfied
- tests/build/smoke checks pass
- independent review has no blockers
- known limitations are documented or explicitly deferred
- no unverified last-minute patch remains
- the final artifact matches actual runtime/source-of-truth behavior

If you make a patch after the last green run, the state is no longer verified. Re-run the checks.

## Roles

Keep roles separate:

- **Builder:** makes the change.
- **Breaker/reviewer:** tries to find what is wrong, unsafe, misleading, or untested.
- **Fixer:** turns confirmed findings into tests/doc fixes and patches them.

One agent can perform builder and fixer work, but should not pretend self-review is independent. Use a subagent/fresh context for breaker review when available.

If no independent reviewer is available, label the pass as **self-review** and do not claim the anvil loop is complete.

## For Code Changes

Good pattern:

```text
write failing test → implement minimum fix → run targeted test → run full relevant suite → independent review → add regression tests for findings → repeat
```

Check for:

- compatibility with old fields/APIs/data
- edge cases and malformed input
- race/churn behavior for filesystem/process/log code
- generated artifacts matching actual behavior
- failures returning the right exit code/status
- security issues: secrets, command injection, unsafe deserialization, credential leaks

## For Docs and Runbooks

Docs can be anvil-looped too. For operational docs, review is often more important than prose polish.

Check for:

- clear source-of-truth: repo vs runtime vs backup vs mirror
- local path vs remote path explicitly named
- commands fail closed: `set -euo pipefail`, clean-tree checks, dry-runs before writes
- no credentials, tokens, private keys, Basic Auth strings, or credentialed URLs
- destructive or irreversible steps clearly gated
- backup/restore/rollback path documented when relevant
- deploy separated from git push when runtime does not auto-deploy
- stale hostnames/remotes called out as stale instead of silently reused

Run smoke checks against the target doc, not this skill file. This skill intentionally contains sample secret-pattern strings.

Lightweight Markdown smoke checks:

These checks are only a starting gate. They do **not** replace full secret scanning, command review, or operational review.

```bash
DOC=${DOC:-README.md} python3 - <<'PY'
import os
from pathlib import Path
p = Path(os.environ['DOC'])
s = p.read_text()
fence = chr(96) * 3
assert s.count(fence) % 2 == 0, 'unbalanced code fences'
for bad in ['password=', 'token=', 'PRIVATE KEY', 'https://user:pass@']:
    assert bad not in s, f'possible secret: {bad}'
print('md_smoke_ok')
PY
```

## Reviewer Prompt Template

Use something like this for independent review:

```text
Review this artifact with blocker priority. Include non-blockers only if they affect safe shipping or future maintenance.

Contract:
- <what must be true>
- <what must not happen>

Inspect:
- <files/paths>

Look for:
- incorrect behavior
- unsafe commands or migration steps
- stale source-of-truth claims
- missing credential warnings
- edge cases not covered by tests
- wording that could make an operator do the wrong thing

Return:
- blockers
- non-blockers
- verdict
```

## Finding Handling Template

For each real finding, record:

```text
Finding: <short name>
Severity: blocker | non-blocker | tradeoff | deferred
Evidence: <test failure, repro, line number, or command output>
Fix: <what changed>
Verification: <test/check/re-review result>
```

## Common Pitfalls

1. **Calling a normal review an anvil loop**
   - One read-through is not an anvil loop. The loop requires fix + re-check + re-review when real findings appear.

2. **Self-reviewing and calling it independent**
   - Use a fresh reviewer/subagent where possible.

3. **Implementing without a failing test or exact doc evidence**
   - This creates guess-fixes and new regressions.

4. **Stopping after a last-minute edit**
   - Last edit invalidates previous verification. Re-run checks.

5. **Mixing unrelated changes**
   - The loop gets muddy if the artifact contains unrelated edits.

6. **Docs that describe intent instead of reality**
   - Verify live paths, remotes, service names, timers, and runtime behavior before documenting them.

## Minimal Checklist

- [ ] Contract stated
- [ ] Intended files only changed
- [ ] Tests/build or Markdown smoke checks passed
- [ ] Independent review completed
- [ ] Blockers fixed with evidence
- [ ] Checks re-run after fixes
- [ ] Re-review completed if blockers were found
- [ ] Final state and limitations reported plainly
