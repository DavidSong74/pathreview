# PLAN.md — Issue #147: Resume section detection fails on text with leading whitespace

## Problem

`_detect_sections()` in `ingestion/parsers/resume_parser.py` (line 127) looks for
section headers ("Experience", "Education", "Skills", etc.) using regex patterns
anchored at the very start of a line:

```python
patterns = [
    rf"^{re.escape(section)}\s*$",
    rf"^{re.escape(section)}\s*[:|-]",
    rf"\n{re.escape(section)}\s*$",
    rf"\n{re.escape(section)}\s*[:|-]",
]
```

`^` and `\n` require the header text to begin at column 0 of a line. Text pulled
from PDFs (and any resume that isn't perfectly flush-left) commonly has leading
indentation, so a line like `"    Education:"` matches none of these patterns.
The result: `detected_sections` comes back empty even when the headers are
clearly present.

## Approach

Make the four regex patterns tolerant of optional leading whitespace by
inserting `\s*` right after the anchor, so indentation is allowed but not
required:

```python
patterns = [
    rf"^\s*{re.escape(section)}\s*$",
    rf"^\s*{re.escape(section)}\s*[:|-]",
    rf"\n\s*{re.escape(section)}\s*$",
    rf"\n\s*{re.escape(section)}\s*[:|-]",
]
```

This is a minimal, localized change (4 lines) — no change to the method's
signature, return type, or the calling code in `parse()`. Flush-left resumes
keep working exactly as before, since `\s*` matches zero whitespace too.

## Steps

1. Reproduce the bug with a test first (done): added
   `test_detect_sections_with_leading_whitespace` in
   `tests/unit/test_resume_parser.py`, confirmed it fails against the
   current code.
2. Apply the regex fix in `_detect_sections()` (`resume_parser.py:132-138`).
3. Re-run `test_detect_sections_with_leading_whitespace` and confirm it now
   passes.
4. Re-run the full test file (`pytest tests/unit/test_resume_parser.py -v`)
   and confirm the other tests the issue calls out as affected —
   `test_parse_single_column_resume_text`, `test_parse_resume_no_work_experience`,
   `test_detect_sections` — also pass, since their fixtures/inline text are
   themselves indented.
5. Run `make check` (ruff, black, mypy) to confirm the change is clean and
   doesn't introduce new type/lint issues.
6. Update `JOURNAL.md` with the outcome and open the PR referencing issue #147.

## Files touched

- `ingestion/parsers/resume_parser.py` — the actual fix, `_detect_sections()`
  (~4 lines changed).
- `tests/unit/test_resume_parser.py` — new test
  (`test_detect_sections_with_leading_whitespace`), already added.

No other files should need to change — no callers of `_detect_sections()`
depend on its exact regex behavior beyond "does it find the section," and the
method's signature (`text: str -> list[str]`) is unchanged.

## Risks / unknowns

- **Over-matching:** `\s*` after `^`/`\n` is intentionally permissive (any
  amount of whitespace, including none). Risk is low since section names are
  still matched exactly (`re.escape(section)`) — only leading whitespace
  before the name is now tolerated, not extra characters.
- **Overlap with an existing open PR:** issue #147 already has a linked PR
  (#178) proposing a similar fix (also widening `^`/`\n` to `^\s*`/`\n\s*`,
  and additionally in `_strip_markdown()`). My implementation will be written
  independently and only compared against #178 afterward as a sanity check,
  since the assignment is graded on my own artifacts — but there's a chance
  #178 merges first and this issue closes before my PR is up. Mitigation: if
  that happens, I'll pick a different open issue rather than treat this as
  wasted work, since the exercise (bug repro → fix → test) still counts.
- **Whitespace beyond spaces (tabs, non-breaking spaces):** `\s` in Python's
  `re` module already covers tabs and other whitespace characters by default,
  so no separate handling should be needed there — worth double-checking
  with a quick manual test if time allows, but not expected to be a real risk.
