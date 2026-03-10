# module-debug.md — Single Module Debug

Use this mode when the user wants to test one specific rules file in isolation,
without running the full skill test suite.

---

## Step 1: Select target module

List all available rules files in the target skill's `rules/` directory.
Ask the user to pick one:

```
Which module do you want to debug?

  [1] rules/phase-extract.md
  [2] rules/phase-synthesize.md
  [3] rules/phase-expand.md
  ... (list all found)
```

---

## Step 2: Select dependencies (optional)

Ask:
```
Does this module depend on any other modules to work correctly?
Select any dependencies to load alongside it (or press Enter to skip):

  [ ] rules/templates.md
  [ ] rules/phase-extract.md
  ... (list all except selected module)
```

The selected module + checked dependencies will be the full context injected into the test agent.

---

## Step 3: Define targeted test cases

Generate or ask for 3–5 test cases that are **specific to the selected module's function**.

Examples:
- If testing an extraction module → test cases should be input documents that need extraction
- If testing an output format module → test cases should check format compliance
- If testing error handling in a module → test cases should have edge/invalid inputs

Ask:
```
Describe 3–5 test inputs that specifically exercise [module name].
(Or press Enter to use auto-generated cases based on the module's content.)
```

If auto-generating, read the module file and infer what it's supposed to do,
then create test inputs that directly probe its logic.

---

## Step 4: Execute

For each test case, spawn one Agent with:
- **Instructions:** selected module content + any dependency module contents
- **Input:** test case prompt

Save results to `.forge/module-debug/[module-slug]/case-[N].md`

Print one line per case as it completes:
```
✓ Case [N] done — [pass|fail|warn]
```

---

## Step 5: Quick evaluation

After all cases, do a focused evaluation (do NOT use the full evaluator.md flow):

Read all case results and in one LLM call:
```
Given the module's instructions and these test results,
identify:
1. What this module does correctly
2. What it fails to handle
3. The single most important thing to fix (specific line or rule if possible)
```

Print the result directly. No score, no report file — just clear feedback.

---

## Step 6: Git commit (optional)

Ask:
```
Would you like to commit the current state of [module file] to Git?
```

If yes, after user confirms their edits are done:
- Follow the commit portion of `rules/git-ops.md`
- Use commit message: `fix([module-slug]): [one-line description of what was fixed]`
- Do NOT branch — commit directly if on a development branch, or create `fix/[module-slug]` if on main

---

## Notes

- Module debug does not update `.forge/state.json` — it's a lightweight, standalone check
- Never load the full skill context in module debug mode — defeats the purpose of isolation
- If the user wants to run a full test after module debug, return to `rules/intake.md`
