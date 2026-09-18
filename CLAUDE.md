# Project Overview
Grade students' homework submissions using concurrent **grader agent** (red annotations) + **independent checker agent** (single-pass blue verification). Agents work in parallel on different files.

# Workflow Rules
- **Only edit files in `graded-submissions/`**
- **No unnecessary intermediate files**: agents never create intermediate/temp files (e.g. `.json`, `.txt`) anywhere in the project folder — all extraction, verdicts, and comparisons stay in memory; the only files written are the final `_Graded.docx`/`_Graded.xlsx`
- **Submissions**: Accepts .doc, .docx, .pdf, .xlsx, and .xls files
- **Annotations**: Only annotate incorrect/incomplete answers (correct answers need no feedback). Placement convention (documents vs. spreadsheets), content extraction requirements (equations, spreadsheet tabs), and conceptual-question key-point checking are detailed in `grader.md` and summarized in `SKILL.md` — both grader and checker must follow the same methods
- **No scoring**: Agents never calculate or write a total score/grade (e.g. "8/10", "80%", a letter grade) — only per-question correct/incorrect/partial verdicts and explanations. Scoring is left entirely to the human instructor.
- **Verification**: Single-pass check by grader-checker (no correction loops)
  - If "Review Passed" → grading complete ✓
  - If "Review FAILED" → explicitly state every problem in blue text immediately below the mark, in the graded file itself (no separate report file)
- See SKILL.md for workflow details

# To Grade Submissions
Use the `/grading-instructions` skill for detailed workflow and invocation instructions.

# Project Structure
- `reference-solutions/` — Solution files (.doc, .docx, .pdf, .xlsx, .xls)
- `student-submissions/` — Student submissions (.doc, .docx, .pdf, .xlsx, or .xls, read-only)
- `graded-submissions/` — Output:
  - `[name]_Graded.docx` — annotated document (grader + checker marks), for .doc/.docx/.pdf input
  - `[name]_Graded.xlsx` — annotated workbook (grader + checker marks on a "Grading Summary" tab, cell-level annotations on original tabs), for .xlsx/.xls input
  - No separate report file is ever created — if verification fails, every problem is stated explicitly in blue text immediately below the "Review FAILED" mark in the graded file itself
- **Per-homework subfolders (optional)**: `reference-solutions/` and `student-submissions/` may each be flat or organized into subfolders (e.g. `HW1/`, `HW2/`), not always the same way on both sides. Agents must match a submission's subfolder to a solutions subfolder by exact name before grading (see `SKILL.md` Step 0), and output mirrors the input structure (e.g. `graded-submissions/HW1/`).
- `agents/` → `grader.md`, `grading-checker.md`
- `skills/grading-instructions/` → Full workflow


