# Anvil Loop

A hardening workflow for changes that need more than a happy-path pass.

```text
contract → build → verify → break/review → fix → re-verify → re-review → stop
```

Use it when a change touches risk: deploys, migrations, backups, auth, credentials, rollback, CLI/tools, filesystem/process handling, generated artifacts, or operational docs people may copy-paste.

## What it means

Do not stop at “it works once.”

1. State the contract.
2. Build the smallest useful change.
3. Verify directly.
4. Get independent break/review.
5. Classify findings.
6. Convert real findings into tests, repros, or exact doc evidence.
7. Fix only proven problems.
8. Re-run checks and re-review.
9. Stop only when there are no blockers.

If no independent reviewer is available, label it as self-review and do not claim the anvil loop is complete.

## Hermes skill

Canonical skill file:

`skills/software-development/anvil-loop-development/SKILL.md`

For a local Hermes install, copy it to:

`~/.hermes/skills/software-development/anvil-loop-development/SKILL.md`

There is intentionally no root `SKILL.md`, to avoid duplicate drift.

## Minimal checklist

- [ ] Contract stated
- [ ] Intended files only changed
- [ ] Tests/build or Markdown smoke checks passed
- [ ] Independent review completed
- [ ] Blockers fixed with evidence
- [ ] Checks re-run after fixes
- [ ] Re-review completed if blockers were found
- [ ] Final state and limitations reported plainly

## License

MIT
