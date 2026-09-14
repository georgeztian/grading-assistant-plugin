---
name: grading-instructions
description: grading instructions to grade students' homework submissions
---

# Concurrent Grading Workflow

Orchestrates parallel **grader agent** (red annotations) + **independent checker agent** (blue verification) with automatic correction loop (max 3 iterations).

## Concurrent Workflow Overview

Both agents work independently on different files:
- **Grader**: processes submission queue, one at a time
- **Checker**: processes any submission with "Grading Completed" mark, one at a time
- When checker finds issues, grader corrects that submission while continuing with next submission
- Agents can be invoked concurrently (in separate processes/sessions)

## Grader: Grade Submission

**Invoke** for each submission:
- `submission_file`: path in `student-submissions/`
- `solutions_file`: path in `reference-solutions/`
- `output_dir`: `graded-submissions/`
- `mode`: "initial" (default) or "correction"
- For corrections: add `check_report` and `iteration` (1-3; max 3)

**Output**: `_Graded.docx` (with "Grading Completed" mark) + `_verdicts.json`

## Checker: Verify Grading

**Invoke** for any submission with "Grading Completed" mark:
- `submission_file`: path in `student-submissions/`
- `graded_file`: `_Graded.docx` in `graded-submissions/`
- `verdicts_file`: `_verdicts.json` in `graded-submissions/`
- `solutions_file`: path in `reference-solutions/`
- `iteration`: 1-3 (default 1; max 3)

**Output**: "Review Passed" mark (✓ complete) OR `_check_report.json` (needs correction)

## Correction Loop (MAX 3 ITERATIONS — NO FURTHER RETRIES AFTER ITERATION 3)

**If "Review Passed":** ✓ Complete

**If discrepancies (iterations 1-2):**
1. Invoke `grader` with `mode="correction"`, `check_report`, `iteration+1`
2. Grader fixes issues, resubmits
3. Invoke `checker` with `iteration+1` to re-verify
4. Go back to step 1 if still discrepancies and iteration < 3

**If discrepancies at iteration 3:** Checker adds "review FAILED" mark → **STOP** (no more retries allowed)

### Discrepancy Types & Severity

| Type | Meaning | Severity |
|------|---------|----------|
| `verdict_mismatch` | Grader/checker disagree on correct/incorrect | high |
| `missed_question` | Grader skipped a question | high |
| `annotation_placement` | Feedback not immediately below answer | medium |
| `explanation_error` | Explanation unclear/incomplete/wrong | medium |
| `incomplete_coverage` | Multi-part question not fully addressed | medium |
