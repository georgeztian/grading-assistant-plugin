# Grading Assistant

An AI-agent-based system for grading student homework submissions inside [Claude Code]. Two independent agents — a **grader** and a **checker** — extract content from submissions and solution files, compare answers, and produce annotated copies with color-coded feedback, without ever touching the original student files. **The agents only flag incorrect/incomplete answers and explain why — they never subtract points or compute a score.** Turning per-question verdicts into a final grade (point deductions, weighting, totaling) is left entirely to the human instructor. **No intermediate files** (e.g. `.json`, `.txt`) are ever created in the project folder — the only files written are the final `_Graded.docx`/`_Graded.xlsx` copies in `graded-submissions/`.

## How it works

1. **Grader agent** extracts every piece of content from a submission and its matching solution file (text, equations, embedded images, spreadsheet tabs — see [Extraction challenges handled](#extraction-challenges-handled)), compares each answer against the solution, and produces an annotated copy. **Only incorrect or incomplete answers get feedback** — correct answers are left untouched. When grading is done, the grader marks the file **"Grading Completed."**
2. **Checker agent** independently re-grades the same submission from scratch — without looking at the grader's annotations first — then compares its own verdicts against the grader's. It adds a final **"Review Passed"** or **"Review FAILED"** mark. If it finds discrepancies, it states every problem explicitly in blue text immediately below the "Review FAILED" mark, in that same graded file — no separate report file is created.

The two agents can run concurrently across a batch of submissions since each works on its own file. **Concurrency is capped at 10 agents of the same type running at once** — at most 10 grader agents and, separately, at most 10 checker agents may run simultaneously. For a batch larger than 10 submissions, the next agent launches as soon as an earlier one of the same type finishes.

## Folder structure

```
reference-solutions/      # Answer key file(s) — .doc, .docx, .pdf, .xlsx, or .xls
student-submissions/      # Student submissions (same file types) — read-only, never modified
graded-submissions/       # Output only — annotated copies (any check discrepancies are stated inline, no separate report files)
agents/                   # grader.md, grading-checker.md — the two agents' full logic
skills/grading-instructions/  # SKILL.md — orchestration workflow
CLAUDE.md                 # Quick-reference project rules
```

### Optional per-homework subfolders

Both `reference-solutions/` and `student-submissions/` can either be flat, or organized into per-assignment subfolders (e.g. `HW1/`, `HW2/`) — independently on each side, and mixed layouts are fine (some assignments flat, others subfoldered, at the same time).

- A submission in `student-submissions/HW1/` is matched to `reference-solutions/HW1/` by **exact folder name** — `HW1` will not match `Homework 1` or `hw1_solutions`.
- A flat submission is matched to a flat solutions file by content (subject/case name, file type), since filenames aren't required to match — if more than one flat solutions file could plausibly apply, the system asks rather than guessing.
- If no confident match is found, the system stops and asks rather than guessing.
- Output mirrors the input: a submission from `HW1/` produces its graded file in `graded-submissions/HW1/`.

## Supported file types

| Input | Output |
|---|---|
| `.doc`, `.docx`, `.pdf` | `[name]_Graded.docx` |
| `.xlsx`, `.xls` | `[name]_Graded.xlsx` (`.xls` is always upgraded to `.xlsx`, since the legacy format can't reliably round-trip formatting) |

## Reading the annotations

**Documents (.docx output):**
- Red text is inserted immediately below each incorrect/incomplete answer, formatted as `**INCORRECT**: [why, and what the correct answer is]` or `**INCOMPLETE**: [what's missing]`.
- `Grading Completed` appears in red at the end of the document once the grader is done.
- `Grading Completed | Review Passed` or `Grading Completed | Review FAILED` is added in **blue** by the checker. If it's a FAILED review, every discrepancy is stated explicitly in blue text immediately below that mark, in the same document — no separate file.

**Spreadsheets (.xlsx output):**
- Each incorrect/incomplete answer **cell itself** is highlighted using Excel's standard "Bad" style (light red fill, dark red font) — the value/formula is never changed, only its formatting.
- An explanation is written in red text into the closest empty cell to it (right, then below, then further out) — existing cell content is never overwritten.
- A dedicated **"Grading Summary"** tab (added as the last sheet) carries `Grading Completed` (red) and, once checked, `Review Passed`/`Review FAILED` (blue). If FAILED, every discrepancy is stated explicitly in blue text in the rows immediately below that mark, on the same tab — no separate file.

**Conceptual / short-answer / essay questions:** for written-sentence answers, the system doesn't just judge overall direction — it checks the student's answer against every key term or point the solution's explanation relies on. If the answer is coherent but missing a specific point, it's marked **INCOMPLETE** and every missing point is named explicitly (never a vague "explanation incomplete").

If nothing is annotated and the mark is "Review Passed," every question was answered correctly.

**No total score:** neither agent calculates or writes a total score/grade (e.g. "8/10", "80%", a letter grade), and neither subtracts points per question — they only record a per-question correct/incorrect/partial verdict plus an explanation. Deciding how much each wrong or incomplete answer costs, and adding it all up into a final grade, is left entirely to the human instructor.

## How to use it

### Prerequisites
- [Claude Code] with access to this repository.
- **Recommended model: Claude Sonnet or higher** (e.g. Sonnet 5, Opus 5). Grading requires careful multi-step content extraction (equations, embedded images, spreadsheet formulas) and nuanced comparison against a solution key — lower-tier models are more prone to missed or inaccurate verdicts.
- Python 3 with the relevant libraries available (`python-docx`, `pypdf`, `openpyxl`, `xlrd<2.0`, `lxml`; optionally `PyMuPDF`/`pdf2image` for PDF rendering) — the agents will use `pip install` as needed.
- A `.doc`-to-`.docx` converter for legacy Word files: `LibreOffice` (headless) or `pandoc`, either of which also handles equation-to-LaTeX conversion for `.docx`.

### 1. Add your files
- Put the answer key in `reference-solutions/`.
- Put student submissions in `student-submissions/`.
- Optionally organize either folder into per-homework subfolders (see above).

### 2. Ask Claude to grade
Just ask, in plain language, for example:
- "grade all student submissions"
- "grade student-submissions/HW1/john_doe.docx against reference-solutions/HW1/answer_key.docx"

This runs the `/grading-instructions` skill, which will:
1. Inspect the folder structure and match each submission to its solution file (asking you to confirm if a match is ambiguous).
2. Run the grader agent on each submission (respecting the concurrency cap above).
3. Run the checker agent once a submission is marked "Grading Completed" (same cap, tracked separately).
4. Report a Pass/Fail summary for each file.

### 3. Review the output
- Open the corresponding file in `graded-submissions/`.
- Read the red annotations for what was marked wrong and why.
- Check the blue mark at the end (or on the "Grading Summary" tab) for the checker's final verdict.
- If it says **Review FAILED**, read the specific discrepancies stated in blue text immediately below that mark in the same file, and decide manually how to proceed — the system will not auto-correct or re-grade on its own.

## Important: this is AI-assisted grading

The grader and checker are independent, but both are AI agents interpreting free-form student work — treat "Review Passed" as a strong second opinion, not an infallible verdict. Spot-check a sample of graded files yourself, especially for high-stakes grading, ambiguous/creative answers, or anything the checker flagged as low-confidence or ambiguous.

## Where to look for more detail

- [`agents/grader.md`](agents/grader.md) — full grader logic and extraction methods
- [`agents/grading-checker.md`](agents/grading-checker.md) — full checker logic
- [`skills/grading-instructions/SKILL.md`](skills/grading-instructions/SKILL.md) — orchestration workflow, folder-matching rules, discrepancy types
- [`CLAUDE.md`](CLAUDE.md) — quick-reference project rules
