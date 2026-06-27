# Git Workflow

This repo uses a personal fork workflow:
- `origin` = personal fork (`git@github.com:0xemc/innatify.git`)
- `upstream` = source of truth (`git@github.com:Innatify/innatify.git`)

## Branches

- `main` — pure mirror of `upstream/main`. Never commit fork-only files here.
- `fork-ci` — the fork's **default branch**: `main` + one commit adding fork-only CI
  (`.github/workflows/pr-bot.yml` + `.github/actions/aider-run/`). Never base feature
  branches on it.

**Syncing with upstream**
```bash
git fetch upstream
git checkout main
git merge --ff-only upstream/main   # must fast-forward; main is a mirror
git push origin main
```
Don't use GitHub's "Sync fork" button on `fork-ci` — it offers to discard the bot commit.

**Feature branches**
```bash
git worktree add ../worktrees/my-feature -b my-feature origin/main
git push -u origin my-feature
```

## Fork-CI / PR bot flow

The aider PR bot (`pr-bot.yml`) lives only on `fork-ci`. Its two triggers resolve the
workflow file from different places:
- `pull_request_review` runs the workflow from the **PR merge ref** (head + base), so
  internal PRs must use `fork-ci` as their **base** for the review trigger to fire.
- `check_run` runs the workflow from the **default branch** (`fork-ci`), so it fires
  for checks on any PR in the fork.

Either way the bot files never enter feature-branch history, so PRs raised against
upstream are clean by construction.

1. **Open an internal PR** to iterate with the bot — base `fork-ci`, which is
   already the fork's default base:
   ```bash
   gh pr create
   ```
2. The bot reacts to submitted reviews and failed checks, committing fixes to the
   feature branch. To re-engage it, submit a new review or re-run a failed check.
3. **Raise the upstream PR** from the same branch when ready:
   ```bash
   gh pr create --repo Innatify/innatify --head 0xemc:my-feature --base main
   ```
4. Close (don't merge) the internal PR.

**Updating the bot** — commit on `fork-ci` (worktree at `../worktrees/fork-ci`) and
push; the workflow references its action by branch ref
(`0xemc/innatify/.github/actions/aider-run@fork-ci`), so it tracks the tip
automatically. To pull in upstream changes: merge `main` into `fork-ci`.
