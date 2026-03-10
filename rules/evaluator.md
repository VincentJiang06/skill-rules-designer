# evaluator.md — Two-Layer Evaluation

Evaluate all results from `.forge/results/`. Two layers in strict order.

---

## Layer 1: Local rules check (0 LLM tokens)

Read each result file. For every case, check these criteria mechanically:

| Check | Pass condition | Weight |
|-------|---------------|--------|
| Completed | Agent produced output (not empty, not error) | required |
| Format match | Output structure matches what skill description promises | 20 pts |
| Step coverage | Key phases/steps mentioned in SKILL.md are present | 20 pts |
| No hallucinated tools | Only uses tools declared in SKILL.md or standard Claude tools | 15 pts |
| Handles the input | Output is relevant to the test case input (not generic) | 15 pts |

Mark each check: ✅ pass / ❌ fail / ⚠️ warn

**If any case has status `error` (agent failed):** note it, skip Layer 1 checks for that case, count as 0 pts.

Compute a preliminary score per case (out of 70 pts max from Layer 1).

---

## Layer 2: Batch LLM evaluation (1 call)

Assemble ONE prompt containing:
- The target skill's description (from SKILL.md frontmatter)
- All test case results (title + output, trimmed to ~500 chars each)
- Layer 1 scores

Ask in a single call:
```
For each test case, evaluate:
1. Semantic quality (0–15): Is the output genuinely useful and correct?
2. Logic coherence (0–15): Does the reasoning make sense end-to-end?

Then give:
- Overall score /100
- Top 3 specific issues (each with: description + which rules file is likely responsible)
- One-sentence verdict: which issue matters most?
```

**Key rule: batch all cases into one call. Never call LLM per case.**

---

## Step 3: Compile final report

Write to `.forge/eval-report.md`:

```markdown
# Evaluation Report — [skill name] [version]
Date: [date]
Test cases: [N] ([X] pass / [Y] fail / [Z] warn / [W] error)

## Score: [total]/100
- Layer 1 (structural): [X]/70
- Layer 2 (semantic):   [Y]/30

## Issues found
1. [Issue description] — likely in: [rules/filename.md] line ~[N]
2. [Issue description] — likely in: [rules/filename.md]
3. [Issue description]

## Verdict
[One sentence from LLM: the single most important thing to fix]

## Per-case results
| Case | L1 score | L2 score | Status |
|------|----------|----------|--------|
| ...  | ...      | ...      | ...    |
```

Then follow `rules/iteration.md`.
