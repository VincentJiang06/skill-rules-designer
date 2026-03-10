# iteration.md — Iteration Decision

## Step 1: Determine recommendation

Read `.forge/eval-report.md`. Apply this decision logic:

| Condition | Recommendation |
|-----------|---------------|
| Score ≥ 90 AND no fail cases | **Stop** — skill is production-ready |
| Issues are: param errors, format bugs, missing edge case handling, small logic gaps | **Debug** → small version (A₁) |
| Issues are: core workflow wrong, tool selection wrong, fundamental logic flaw, same-approach optimization has plateaued | **Redesign** → new major version (B) |

When in doubt between Debug and Redesign: pick Debug if the issues can be fixed
by changing < 10% of the content. Pick Redesign if fixing would require touching > 50%.

---

## Step 2: Present to user

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Round complete — [skill name] [current version]
Score: [X]/100  ([pass] pass / [fail] fail)

Issues:
  1. [Issue 1 — file + line if known]
  2. [Issue 2]
  3. [Issue 3]

Verdict: [one-sentence from evaluator]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Next step:

[[recommended]] [1] Debug small fix → version [A₁]
                    Fix the issues above, core logic unchanged (<10% edits)

               [2] Redesign → new version [B]
                    Rethink core workflow, new approach (>50% rewrite)

               [3] Stop — export report and finalize
```

Mark the recommended option with `[[recommended]]`. Always show all three.

---

## Step 3: Version naming reference

Current version naming state is tracked in `.forge/state.json`:

```json
{
  "major": "A",
  "minor": 0,
  "current_tag": "vA",
  "history": ["vA"]
}
```

**When user picks Debug (option 1):**
- Increment minor: A → A₁ → A₂ → A₃ (up to A₉, then suggest redesign)
- Branch name: `fix/v[major].[minor]` e.g. `fix/vA.1`
- Tag: `v[major].[minor]` e.g. `vA.1`
- Commit type: `fix`

**When user picks Redesign (option 2):**
- Increment major: A → B → C → D
- Branch name: `feat/v[major]` e.g. `feat/vB`
- Tag: `v[major]` e.g. `vB`
- Commit type: `feat`

Update `.forge/state.json` after each round.

---

## Step 4: After user chooses

**Option 1 or 2 (continue iterating):**

Tell the user exactly what to change:

```
Please make the following edits to your skill:

1. [File: rules/xxx.md, approximate location]
   Problem: [description]
   Suggested fix: [specific guidance]

2. [File: rules/yyy.md]
   Problem: [description]
   Suggested fix: [specific guidance]

When you're done editing, say "done" and I'll commit the new version and run the next round.
```

Wait for the user to say "done" (or similar). Then follow `rules/git-ops.md`.

**Option 3 (stop):**

Print final summary and the path to `.forge/eval-report.md`. Done.

---

## Notes

- Never auto-edit the user's skill files. Only provide guidance.
- Never proceed to Git commit until user explicitly confirms edits are complete.
- Never recommend Redesign on the first round unless the score is below 30.
