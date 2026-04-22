# PromptFill Fork Development Workflow

This repository is maintained as a **custom fork** of:
- Fork remote (`origin`): `https://github.com/lxyer/PromptFill.git`
- Upstream remote (`upstream`): `https://github.com/TanShilongMario/PromptFill.git`

## Workflow Goal

Keep custom PromptFill development moving on a single stable mainline while merging upstream updates in **small, controllable batches**.

The rule of thumb is:

> Protect current custom development first. Pull upstream forward gradually. Never let upstream sync become a disruptive big-bang event.

## Branch Model

### Long-lived branches

#### `main`
- The only long-lived **custom development mainline**.
- Tracks `origin/main`.
- All of your own features should eventually land here.
- Treat this as the branch you can safely keep developing on.

#### `upstream-main`
- Local **read-only tracking branch** for `upstream/main`.
- Refresh it with `git fetch upstream`.
- Never do custom development on it.
- Never merge feature work into it.
- It exists only to show the current upstream base clearly.

### Short-lived branches

#### `feature/<topic>`
- Create from `main`.
- Use for normal secondary development.
- Merge back into `main` when complete.

Examples:
- `feature/prompt-history`
- `feature/share-ui`
- `feature/video-template-import`

#### `hotfix/<topic>`
- Create from `main` for urgent fixes.
- Merge back into `main` as soon as the fix is validated.

#### `sync/upstream-<date-or-topic>`
- Create from `main` when you want to intake upstream updates.
- Merge `upstream-main` into this branch.
- Resolve conflicts here, not directly on `main`.
- Validate locally.
- Merge back into `main` only after the sync looks safe.

Examples:
- `sync/upstream-2026-04-22`
- `sync/upstream-v1-2-sync`

## Daily Development Rules

### Normal feature work

```bash
git switch main
git pull origin main
git switch -c feature/<topic>
# develop
# commit

git switch main
git merge --no-ff feature/<topic>
git push origin main
```

Rules:
- Start features from `main`.
- Keep feature branches short-lived.
- Merge completed work back into `main`.
- Delete merged feature branches locally and remotely when no longer needed.

### Important constraints
- Do **not** develop directly on `upstream-main`.
- Do **not** rebase `main` onto `upstream/main`.
- Do **not** maintain multiple long-lived custom product branches unless the workflow changes intentionally later.
- Do **not** do large delayed upstream merges after long drift if small syncs are possible.

## Upstream Sync Procedure

Use this whenever you want to absorb upstream changes.

### 1) Refresh upstream tracking

```bash
git fetch upstream
git switch upstream-main
git reset --hard upstream/main
```

This keeps `upstream-main` as a clean mirror of upstream.

### 2) Start an intake branch from your current custom mainline

```bash
git switch main
git pull origin main
git switch -c sync/upstream-$(date +%Y-%m-%d)
```

### 3) Merge upstream into the sync branch

```bash
git merge upstream-main
```

If conflicts happen:
- resolve them on the `sync/upstream-*` branch
- prefer preserving your custom behavior on purpose
- only accept upstream behavior when it is clearly better or required
- commit the conflict resolution explicitly

### 4) Validate the sync branch

Minimum checks:
```bash
npm install
npm run build
```

If the repo has tests later, also run:
```bash
npm test
```

### 5) Land the sync safely

```bash
git switch main
git merge --no-ff sync/upstream-YYYY-MM-DD
git push origin main
```

Then delete the temporary sync branch if everything is stable.

## Conflict Policy

When upstream and custom work disagree:

1. **Protect active custom development first**.
2. Prefer **small repeated syncs** over a large risky sync.
3. If a sync becomes too disruptive, stop on the `sync/upstream-*` branch and continue normal development on `main`.
4. Retry upstream intake later after a smaller or more focused review window.

## Recommended Sync Cadence

Use a lightweight cadence:
- **Before starting a large new feature**: check whether upstream should be synced first.
- **At least every 1-2 weeks**: fetch upstream and evaluate drift.
- **Immediately for important upstream fixes**: security, data integrity, or core runtime breakages.

If upstream changes are mostly unrelated to your active work, merge them sooner.
If upstream changes would heavily disturb your current feature work, delay them and merge later in a dedicated sync pass.

## GitHub-side Recommendations

For the fork on GitHub:

- Keep `main` as the default branch.
- Treat `main` as the only long-lived branch that matters on the fork.
- Push `feature/*` branches only when you need backup, collaboration, or PR review.
- Avoid keeping stale remote feature branches around.
- You do **not** need to push `upstream-main`; it is primarily a local tracking branch.
- Consider enabling branch protection on `main` later if you want stricter safety.

## Quick Command Reference

### Initial setup
```bash
git remote -v
git remote add upstream https://github.com/TanShilongMario/PromptFill.git
git fetch upstream
git branch --track upstream-main upstream/main
```

### Start feature
```bash
git switch main
git pull origin main
git switch -c feature/<topic>
```

### Start upstream sync
```bash
git fetch upstream
git switch upstream-main
git reset --hard upstream/main
git switch main
git pull origin main
git switch -c sync/upstream-$(date +%Y-%m-%d)
git merge upstream-main
```

## Unsafe Operations to Avoid

Avoid these unless you intentionally want history surgery:

```bash
git rebase upstream/main main
git push --force origin main
git commit directly on upstream-main
```

## Success Criteria

This workflow is working if:
- upstream updates do not block your normal development
- custom features continue to land on one stable mainline
- conflicts remain limited and manageable
- syncing upstream feels like a controlled maintenance task, not a project reset
