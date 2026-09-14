# Project Overview
Grade students' homework submissions using concurrent **grader agent** (red annotations) + **independent checker agent** (blue verification) with automatic correction loop (max 3 iterations). Agents work in parallel on different files.

# Workflow Rules
- Never edit raw files in `student-submissions/` — always work on duplicates
- **Correction loop**: max 3 iterations (after iteration 3 with remaining issues → submission marked "review FAILED" and skipped)
- See SKILL.md for completion marks and workflow details

# To Grade Submissions
Use the `/grading-instructions` skill for detailed workflow and invocation instructions.

# Project Structure
- `reference-solutions/` — Solution files
- `student-submissions/` — Original submissions (read-only)
- `graded-submissions/` — Output: `[name]_Graded.docx`, `[name]_verdicts.json`, `[name]_check_report.json`
- `.claude/agents/` → `grader.md`, `grading-checker.md`
- `.claude/skills/grading-instructions/` → Full workflow


