# Idea Evaluation Pipeline — Claude Code

See `AGENTS.md` for full pipeline instructions.

## Quick Reference

This project runs a research idea evaluation pipeline for PhD students in finance/economics. The pipeline iterates through 8 steps until an idea scores >= 7/10.

### Prompt files (in project root):
- `prompt_ideas.txt` — Steps 1 & 4 (evaluate)
- `review_eval_prompt.txt` — Step 2 (review evaluation)
- `pivot_prompt.txt` — Step 3 (pivot)
- `lit_review_prompt.txt` — Step 5 (lit review, needs WebSearch)
- `verify_lit_review_prompt.txt` — Step 6 (verify citations, needs WebSearch + Edit)
- `final_verdict_prompt.txt` — Step 7 (final verdict)
- `review_final_verdict_prompt.txt` — Step 8 (review verdict)

### Running the pipeline:
1. Student creates a folder with their idea file and 3 closest papers
2. Run steps sequentially, appending results to a master `.md` file
3. Loop back to Step 3 if score < 7
4. Use background agents for parallelizing across multiple ideas
5. Steps 5 and 6 require WebSearch — use agents with web access

### Critical:
- **Never fabricate citations** — Steps 5 and 6 must use WebSearch to find and verify real papers
- **Always append to the master .md** after each step
- **Include full history** when running pivot prompts
