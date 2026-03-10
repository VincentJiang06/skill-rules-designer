# test-runner.md — Sequential Test Execution

## Step 1: Load test cases

**Source priority (in order):**

1. A `test-cases.md` or `evals/evals.json` file in the target skill directory — use it directly
2. User provides test cases now (ask them to describe 3–5 scenarios)
3. No input — offer built-in templates (see below)

**Built-in template categories (offer as checklist):**

| # | Category | What it tests |
|---|----------|---------------|
| T1 | Happy path | Standard input, expects correct full output |
| T2 | Edge input | Empty, minimal, or boundary input |
| T3 | Tool call correctness | Does the skill invoke the right tools with right params? |
| T4 | Output format | Does output match the expected format/structure? |
| T5 | Error handling | Missing input, ambiguous request, partial data |

Ask: "Which categories do you want to include? (default: T1 T2 T3)"

## Step 2: Confirm before running

Show a summary:
```
Ready to run [N] test cases against: [skill name]
Estimated cost: ~[N × 2000] tokens

Proceed? [Y/n]
```

## Step 3: Execute sequentially

For each test case, in order:

```
Spawning agent for case [N]: [case title]...
```

Spawn one Agent with:
- **Instructions:** full content of target SKILL.md + all rules/*.md files concatenated
- **Input:** the test case prompt

Wait for the agent to complete. Save the output to:
```
.forge/results/case-[N]-[slug].md
```

File format:
```markdown
# Test Case [N]: [title]
**Status:** pass | fail | warn
**Input:** [the test case prompt]
**Output:** [agent output — full text]
**Notes:** [any observations]
```

After each case, print one line:
```
✓ Case [N] done — [pass|fail|warn]
```

## Step 4: Summary before evaluation

After all cases complete:
```
Test run complete.
Results: [X] pass  [Y] fail  [Z] warn
Saved to .forge/results/

Proceeding to evaluation...
```

Then follow `rules/evaluator.md`.

## Notes

- Do NOT judge quality during test-runner. Only collect outputs.
- If an agent call times out or errors, mark that case as `error` and continue with remaining cases.
- Never re-run a case automatically. If a case errors, note it and move on.
