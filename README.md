# Research Idea Evaluation Pipeline

A structured pipeline for iteratively evaluating, pivoting, and defending research ideas until they reach top-3 finance journal quality (JF, JFE, RFS) or top-5 economics journal quality.

Designed for PhD students in finance and economics. Works with any AI coding assistant (Claude Code, Cursor, Codex, Windsurf, etc.) or manually by copy-pasting prompts.

---

## How It Works

Your idea goes through up to 8 steps. The pipeline loops until the idea scores 7/10 or higher:

```
1. EVALUATE IDEA  →  2. REVIEW EVALUATION
                          │
                     Critique unfair? → Re-run Step 1
                          │
                     Score >= 7? ─── Yes ──→ 5. LITERATURE REVIEW
                          │                        ↓
                          No               6. VERIFY LIT REVIEW
                          ↓                        ↓
                     3. PIVOT IDEA          7. FINAL VERDICT
                          ↓                        ↓
                     4. EVALUATE PIVOT      8. REVIEW FINAL VERDICT
                          │                        │
                     Score < 7? → Back to 3   Score >= 7? → DONE
                     Score >= 7 → Step 5      Score < 7  → Back to 3
```

---

## Quick Start

### 1. Create your idea folder

```
mkdir my_idea
```

### 2. Add your idea file

Copy `idea_template.txt` into your folder and fill it in:

```
cp idea_template.txt my_idea/idea.txt
```

The template requires:
- Research question and hypothesis with expected sign
- Identification strategy (shock, instrument, or natural experiment)
- Specific data sources with variable names and sample periods
- 3 closest papers with full citations, URLs, and how your idea differs
- Proposed regression equation (if possible)

**The 3 closest papers are critical.** Pick the papers a referee would immediately cite against you — not broadly related work, but the papers that most directly threaten your novelty claim. Include URLs so the pipeline can verify them.

### 3. Run the pipeline

**With an AI coding tool:** Open this project in your tool and ask it to run the pipeline on your idea. The agent will read `AGENTS.md` (or `CLAUDE.md`) and follow the steps automatically.

**Manually:** Copy-paste each prompt file into your preferred LLM (Claude, GPT, etc.) along with the relevant input. Save each output to the correct filename. See the step-by-step guide below.

---

## Step-by-Step (Manual)

### Step 1: Evaluate Idea
- **Prompt:** `prompt_ideas.txt`
- **Input:** Your idea + 3 closest papers
- **Output:** Save as `my_idea/eval_my_idea_idea1.txt`
- **Model:** Use the strongest model available (Claude Opus, GPT-4, etc.)

### Step 2: Review Evaluation
- **Prompt:** `review_eval_prompt.txt`
- **Input:** Your idea + the evaluation from Step 1
- **Output:** Save as `my_idea/review_my_idea_idea1.txt`
- **Decision:** If the review finds the critique unfair, re-run Step 1 with corrections

### Step 3: Pivot Idea (if score < 7)
- **Prompt:** `pivot_prompt.txt`
- **Input:** Your idea + evaluation + review (full history)
- **Output:** Save as `my_idea/pivot_idea1.txt` (or `pivot_idea1_v2.txt`, `_v3.txt` if iterating)

### Step 4: Evaluate Pivot
- **Prompt:** `prompt_ideas.txt` (same as Step 1)
- **Input:** Your pivoted idea + original 3 closest papers
- **Output:** Save as `my_idea/eval_pivot_idea1.txt` (or `eval_pivot_idea1_v2.txt` if iterating)
- **Decision:** If score dropped or stayed flat, go back to Step 3

### Step 5: Literature Review (Threat Search)
- **Prompt:** `lit_review_prompt.txt`
- **Input:** Your pivoted idea + 3 cited papers
- **Output:** Save as `my_idea/lit_review_pivot_idea1.txt`
- **Model:** Use a model with web search (Claude Sonnet with WebSearch, Perplexity, etc.)
- **Important:** The prompt requires URLs for every cited paper to prevent hallucinated citations

### Step 6: Review & Verify Literature Review
- **Prompt:** `verify_lit_review_prompt.txt`
- **Input:** The lit review from Step 5
- **Output:** Edit the lit review file (add URLs, remove fakes) + save summary as `my_idea/review_lit_review_idea1.txt`
- **Model:** Must have web search access (strongest model + web search recommended)
- **Important:** This step catches hallucinated citations. Every paper must be verified via Google Scholar or SSRN. Remove any paper that cannot be found.

### Step 7: Final Verdict
- **Prompt:** `final_verdict_prompt.txt`
- **Input:** Full history (all previous outputs)
- **Output:** Save as `my_idea/final_verdict_idea1.txt`

### Step 8: Review Final Verdict
- **Prompt:** `review_final_verdict_prompt.txt`
- **Input:** Full history + final verdict
- **Output:** Save as `my_idea/review_final_verdict_idea1.txt`
- **Decision:** If score < 7, go back to Step 3 with full history

---

## Scoring Guide

| Score | Meaning | Action |
|-------|---------|--------|
| 1-3 | Low potential, lacks novelty | Major pivot or new idea needed |
| 4-6 | Moderate potential, needs work | Pivot to strengthen ID strategy, sharpen contribution |
| 7-8 | Good potential, publishable with refinement | Proceed to execution |
| 9-10 | Excellent potential, highly novel | Proceed — rare, don't expect this |

**Target: 7/10** to proceed. A score of 6-6.5 after multiple pivots may indicate the idea has hit its ceiling for the current topic. Consider:
- Trying a different idea from your original proposals
- Accepting a realistic target journal (JFQA, JFI, JHE, etc.) instead of top-3
- A more fundamental rethink of the mechanism or setting

---

## File Organization

```
IdeaEvaluation/
├── README.md                       ← this file
├── AGENTS.md                       ← instructions for AI agents
├── CLAUDE.md                       ← Claude Code specific
├── pipeline.md                     ← detailed pipeline documentation
├── pipeline_diagram.png            ← visual flowchart
│
├── prompt_ideas.txt                ← Step 1 & 4: evaluation prompt
├── review_eval_prompt.txt          ← Step 2: review evaluation
├── pivot_prompt.txt                ← Step 3: pivot/spinoff
├── lit_review_prompt.txt           ← Step 5: literature review
├── verify_lit_review_prompt.txt    ← Step 6: verify citations
├── final_verdict_prompt.txt        ← Step 7: final verdict
├── review_final_verdict_prompt.txt ← Step 8: review final verdict
│
├── my_idea/                           ← your idea folder
│   ├── idea.txt                       ← your research idea + 3 closest papers
│   ├── my_idea.md                     ← master file (built up through pipeline)
│   ├── eval_my_idea_idea1.txt         ← Step 1 output
│   ├── review_my_idea_idea1.txt       ← Step 2 output
│   ├── pivot_idea1.txt                ← Step 3 output
│   ├── pivot_idea1_v2.txt             ← Step 3 (iteration)
│   ├── eval_pivot_idea1.txt           ← Step 4 output
│   ├── eval_pivot_idea1_v2.txt        ← Step 4 (iteration)
│   ├── lit_review_pivot_idea1.txt     ← Step 5 output
│   ├── review_lit_review_idea1.txt    ← Step 6 output
│   ├── final_verdict_idea1.txt        ← Step 7 output
│   └── review_final_verdict_idea1.txt ← Step 8 output
```

---

## Tips

- **The 3 closest papers matter.** If you pick papers that are too distant, the evaluation will be too generous. Pick the papers a referee would immediately cite against you.
- **Don't skip Step 6 (verification).** AI models hallucinate citations. Every paper in your lit review must have a verifiable URL.
- **Read the lit review threats yourself.** The pipeline identifies threats, but only you can judge whether a threat is truly fatal or can be addressed.
- **A pivot is not a failure.** Most ideas need 1-2 pivots. The pipeline is designed for iteration.
- **If stuck at 6.5 after 3+ pivots,** the idea may have hit its ceiling. That's useful information — better to learn it now than after a year of data work.

---

## Requirements

- Access to a strong LLM (Claude Opus, GPT-4, or equivalent)
- Web search access for Steps 5 and 6 (Claude with WebSearch, Perplexity, or similar)
- No coding required — this is a prompt-based pipeline
