# intake.md — Entry Point

## Step 1: Read the target skill

Ask the user for the path to the skill they want to test.

Read the following from that path:
- `SKILL.md` (required — abort if missing)
- All files under `rules/` (list them, note which exist)

Summarize what you found:
```
Target skill: [name] v[version]
Rules modules: [list of files found]
Description: [first 2 lines of description]
```

## Step 2: Git status check

Check if the target skill directory already has a Git repository:

```bash
git -C [target_path] rev-parse --is-inside-work-tree 2>/dev/null
```

**If no Git repo exists:**
```bash
git -C [target_path] init
git -C [target_path] add .
git -C [target_path] commit -m "init: initial version vA"
git -C [target_path] tag vA
```

Tell the user: "Git repository initialized. Initial version tagged as vA."

**If Git repo already exists:**
Show current branch and last commit:
```bash
git -C [target_path] branch --show-current
git -C [target_path] log --oneline -5
```

Ask: "Continue from the current state above?"

## Step 3: Mode selection

Present two options:

```
How would you like to test this skill?

[1] Full test — run all test cases against the complete skill
[2] Module debug — test a single rules/*.md file in isolation
```

**If [1]:** Proceed to `rules/test-runner.md`
**If [2]:** Proceed to `rules/module-debug.md`

## Abort conditions

- `SKILL.md` not found at the given path → stop and tell the user
- Target path does not exist → stop and tell the user
- Git init fails (permissions issue etc.) → warn the user, proceed without Git
