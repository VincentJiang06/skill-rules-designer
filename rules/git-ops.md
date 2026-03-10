# git-ops.md — Git Operations Standard

All Git operations in skill-forge follow standard GitHub Flow.
No custom logic. No force pushes. No history rewrites.

---

## Version naming reference

| User sees | Git tag | Git branch |
|-----------|---------|------------|
| A (initial) | `vA` | `main` |
| A₁ | `vA.1` | `fix/vA.1` |
| A₂ | `vA.2` | `fix/vA.2` |
| B | `vB` | `feat/vB` |
| B₁ | `vB.1` | `fix/vB.1` |
| C | `vC` | `feat/vC` |

---

## Operation: Start a new version branch

Called after user confirms they are about to edit their skill for a new version.

```bash
# For Debug small version (e.g. A → A₁):
git -C [target_path] checkout -b fix/vA.1

# For Redesign major version (e.g. A → B):
git -C [target_path] checkout -b feat/vB
```

Tell the user: "Branch created. Make your edits, then say 'done'."

---

## Operation: Commit a completed version

Called after user says "done" with their edits.

```bash
# Stage only the skill files (not logs/results)
git -C [target_path] add SKILL.md rules/

# Commit with Conventional Commits format
git -C [target_path] commit -m "[type]([scope]): [description]"
```

**Commit message rules:**

| Version type | `type` | `scope` | Example |
|-------------|--------|---------|---------|
| Debug fix | `fix` | affected module name | `fix(phase-2): handle empty input edge case` |
| Redesign | `feat` | main changed component | `feat(workflow): replace linear flow with conditional branching` |
| Docs only | `docs` | file changed | `docs(skill): update description and trigger conditions` |

If multiple files were changed, use the most-changed module as scope.
Description: one line, present tense, lowercase, no period.

---

## Operation: Run tests on the new branch

After committing, run the full test suite on the new version (follow `rules/test-runner.md`).

---

## Operation: Merge passing version to main

Called only after test pass (score ≥ threshold or user decides to accept).

```bash
git -C [target_path] checkout main
git -C [target_path] merge --no-ff [branch_name] -m "merge: accept [version_tag]"
git -C [target_path] tag [version_tag]
```

Example for merging A₁:
```bash
git -C [target_path] checkout main
git -C [target_path] merge --no-ff fix/vA.1 -m "merge: accept vA.1"
git -C [target_path] tag vA.1
```

**Always use `--no-ff`** to preserve the branch history in the graph.

---

## Operation: Discard failing version

Called when test fails and user decides not to keep this version.

```bash
git -C [target_path] checkout main
git -C [target_path] branch -d [branch_name]
```

Tell the user: "Branch [name] deleted. No changes to main."

---

## Operation: Show version history

Called when user asks to see the full iteration history.

```bash
git -C [target_path] log --oneline --graph --all
git -C [target_path] tag -l
```

---

## Forbidden operations (never execute)

```bash
git push --force          # NEVER
git rebase                # NEVER (rewrites history)
git commit --amend        # NEVER (on pushed commits)
git reset --hard          # only with explicit user confirmation
git push origin main      # only with explicit user confirmation
```

---

## .gitignore for target skill

When initializing Git, create a `.gitignore` in the target skill directory with:

```
.DS_Store
.forge/results/
.forge/module-debug/
```

The `.forge/eval-report.md` and `.forge/state.json` ARE committed (they're part of the version record).
