# Idea Evaluation Pipeline

**Goal:** Take a research idea and iteratively evaluate, pivot, and defend it until it reaches top-3 finance journal quality.

---

## Pipeline Overview

```
1. EVALUATE ──► 2. REVIEW EVAL ──► Score >= 7? ──Yes──► 5. LIT REVIEW
                     │                  │
                     └── Repeat if      │ No
                         unfair         ▼
                                   3. PIVOT ──► 4. EVAL PIVOT
                                                      │
                                                      └── Repeat if dropped
                                                           (back to 3)
                                                           │
                                                           ▼ OK
                                                      5. LIT REVIEW
                                                           │
                                                      6. REVIEW LIT REVIEW
                                                           │
                                                      7. FINAL VERDICT
                                                           │
                                                      8. REVIEW FINAL VERDICT
                                                           │
                                                      Score >= 7? ──No──► Back to 3
                                                           │ Yes
                                                         DONE
```

---

## Step 1: EVALUATE IDEA

**Agent:** Opus
**Prompt file:** `prompt_ideas.txt`

```
You are a leading expert in economics and finance research. Your task is to
critically evaluate the potential of a given research idea, focusing primarily
on its marginal contribution to the field.

Criteria:
1. Novelty and Marginal Contribution
2. Methodological Innovation (exogenous variation?)
3. Theoretical or Empirical Advancement
4. Potential Impact
5. Feasibility and Data
6. Relevance and Timeliness

Rate 1-10. Max 250 words.
```

**Input:** Student's idea (using `idea_template.txt` format) + 3 closest papers with URLs
**Output:** `eval_[name]_idea[N].txt` — score + critique

---

## Step 2: REVIEW EVALUATION

**Agent:** Opus
**Prompt file:** `review_eval_prompt.txt`
**Purpose:** Check if the critique is fair, accurate, and well-reasoned

```
You are a senior finance professor. Review whether the following critique
is fair, accurate, and well-reasoned. Identify places where the critique
is too harsh, too lenient, factually wrong, or misses important points.

State whether you agree, partially agree, or disagree with the rating.
Max 200 words.
```

**Input:** Original idea + critique
**Output:** `review_[name]_idea[N].txt`
**Loop:** If review finds critique unfair → re-evaluate with corrections

---

## Step 3: PIVOT IDEA (if score < 7)

**Agent:** Opus
**Prompt file:** `pivot_prompt.txt`
**Purpose:** Reshape the idea to address weaknesses while keeping the core topic

```
You are a top finance professor helping a PhD student pivot their research
idea into a top-3 publishable paper.

The pivot should:
1. Keep the core topic
2. Add credible identification strategy
3. Sharpen theoretical prediction
4. Propose specific regressions and data
5. Articulate "new thing we learn" clearly

Max 400 words.
```

**Input:** Original idea + evaluation + review (full history)
**Output:** `pivot_idea[N].txt` (or `pivot_idea[N]_v2.txt` if iterating)

---

## Step 4: EVALUATE PIVOT

**Agent:** Opus
**Prompt file:** `prompt_ideas.txt` (same as Step 1)
**Purpose:** Re-score the pivoted idea using the same evaluation criteria

**Input:** Pivoted idea + original 3 closest papers
**Output:** `eval_pivot_idea[N].txt` (or `eval_pivot_idea[N]_v2.txt` if iterating)
**Loop:** If score dropped or stayed flat → back to Step 3 with full history (include why previous pivot failed)

---

## Step 5: LITERATURE REVIEW (Threat Search)

**Agent:** Sonnet (with WebSearch)
**Prompt file:** `lit_review_prompt.txt`

```
The student has already identified their 3 closest papers. Your job is to
find papers they MISSED — immediate threats to novelty.

1. Threat Search: papers testing similar mechanism, same ID strategy,
   recent SSRN/NBER working papers, adjacent fields
2. Summarize & Cite: full citation, 2-3 sentence threat summary,
   threat level (HIGH/MEDIUM/LOW)
3. Gap Analysis: is there still a clear gap?
4. Defensive Recommendations: how to sharpen against threats
```

**Input:** Pivoted idea + student's 3 cited papers
**Output:** `lit_review_pivot_idea[N].txt`

---

## Step 6: REVIEW & VERIFY LIT REVIEW

**Agent:** Opus (with WebSearch + Edit permissions)
**Prompt file:** `verify_lit_review_prompt.txt`
**Purpose:** Verify every citation is real, add URLs, remove hallucinated papers, correct threat levels

```
Review this lit review for accuracy. For EVERY cited paper:

1. VERIFY: Search Google Scholar / SSRN for the exact title and authors.
   - If found: add the URL next to the citation
   - If NOT found: mark as [UNVERIFIED] or remove if clearly fabricated
2. CHARACTERIZE: Is the paper correctly summarized? Is the threat level appropriate?
3. MISSING: Did the search miss obvious categories or key papers?
4. GAP ANALYSIS: Is the surviving gap convincing given verified threats?

You MUST edit the lit review file directly to:
- Add URLs for every verified paper (e.g., [Google Scholar link] or [SSRN link])
- Remove or flag fabricated/unverifiable citations
- Correct any mischaracterized threat levels
- Add any missing papers you find during verification

Then write a summary of changes to the review file.
```

**Input:** Lit review output file (with edit access)
**Output:**
- Modified `lit_review_pivot_idea[N].txt` (URLs added, fakes removed)
- `review_lit_review_idea[N].txt` (summary of changes + gap assessment)

---

## Step 7: FINAL VERDICT

**Agent:** Opus
**Prompt file:** `final_verdict_prompt.txt`
**Purpose:** Synthesize everything into a final assessment

```
Given the full history below (original idea, evaluation, review, pivot,
pivot evaluation, lit review threats, and defensive recommendations),
provide a final verdict:

1. Final score (1-10) with justification
2. Top 3 remaining threats and how to address each
3. Suggested title and one-paragraph abstract
4. Recommended target journal
5. Key risk: what could kill this paper?
```

**Input:** Full history (all previous outputs)
**Output:** `final_verdict_idea[N].txt`

---

## Step 8: REVIEW FINAL VERDICT

**Agent:** Opus
**Prompt file:** `review_final_verdict_prompt.txt`
**Purpose:** Verify the final verdict is fair, accurate, and that the score is justified given the full evidence

```
You are a senior finance professor reviewing a final verdict on a student's
research idea. You have the full history: original idea, evaluation, review,
pivot(s), pivot evaluation(s), lit review, and lit review review.

Assess whether:
1. The final score is consistent with the evidence presented
2. The threat assessment is neither too harsh nor too lenient
3. The suggested title and framing are well-positioned
4. The preconditions/next steps are actionable and sufficient
5. Any important concerns were missed or underweighted

State whether you agree, partially agree, or disagree with the verdict.
If score < 7, recommend whether to pivot again or adjust the current approach.
Max 300 words.
```

**Input:** Full history + final verdict
**Output:** `review_final_verdict_idea[N].txt`
**Decision:** If score < 7 → back to Step 3 with full history for another pivot

---

## File Organization

```
IdeaEvaluation/
├── pipeline.md                    ← this file
├── pipeline_diagram.png           ← visual flowchart
├── prompt_ideas.txt               ← Step 1 & 4: evaluation prompt
├── review_eval_prompt.txt         ← Step 2: review evaluation prompt
├── pivot_prompt.txt               ← Step 3: pivot/spinoff prompt
├── lit_review_prompt.txt          ← Step 5: lit review prompt
├── verify_lit_review_prompt.txt   ← Step 6: verify lit review prompt
├── final_verdict_prompt.txt       ← Step 7: final verdict prompt
├── review_final_verdict_prompt.txt ← Step 8: review final verdict prompt
├── idea_template.txt              ← required input format for ideas
│
├── [student_name]/
│   ├── [student_name].md          ← master file (full history)
│   ├── [original_submission].pdf
│   ├── [original_submission].txt
│   ├── eval_[name]_idea1.txt           ← Step 1
│   ├── review_[name]_idea1.txt        ← Step 2
│   ├── pivot_idea1.txt                ← Step 3
│   ├── pivot_idea1_v2.txt             ← Step 3 (iteration)
│   ├── eval_pivot_idea1.txt           ← Step 4
│   ├── eval_pivot_idea1_v2.txt        ← Step 4 (iteration)
│   ├── lit_review_pivot_idea1.txt     ← Step 5
│   ├── review_lit_review_idea1.txt    ← Step 6
│   ├── final_verdict_idea1.txt        ← Step 7
│   └── review_final_verdict_idea1.txt ← Step 8
```

---

## Agent Configuration

| Step | Agent | Model | WebSearch | Background |
|------|-------|-------|-----------|------------|
| 1. Evaluate | Opus | claude-opus-4-6 | No | Yes (parallel) |
| 2. Review | Opus | claude-opus-4-6 | No | Yes (parallel) |
| 3. Pivot | Opus | claude-opus-4-6 | No | Yes (parallel) |
| 4. Eval Pivot | Opus | claude-opus-4-6 | No | Yes (parallel) |
| 5. Lit Review | Sonnet | claude-sonnet-4-6 | **Yes** | Yes (parallel) |
| 6. Review Lit | Opus | claude-opus-4-6 | **Yes** | Yes (parallel) |
| 7. Final Verdict | Opus | claude-opus-4-6 | No | Yes (parallel) |
| 8. Review Verdict | Opus | claude-opus-4-6 | No | Yes (parallel) |

**All steps can be parallelized across students** (launch N agents simultaneously).
