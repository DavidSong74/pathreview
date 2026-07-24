## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/147

**Issue title:** Resume section detection fails on text with leading whitespace

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
`_detect_sections()` in `ingestion/parsers/resume_parser.py` looks for section headers (like "Education" or "Skills") using regex patterns anchored to the start of a line — `^section` or `\nsection`. Text pulled from PDFs frequently keeps its original indentation, so a header line like `"    Education:"` never matches any of those patterns. The result is that `detected_sections` comes back completely empty for indented resumes, even though the headers are clearly present to a human reader. A successful fix makes the section-header regexes tolerant of leading whitespace so indented resumes are parsed the same as flush-left ones, and adds a test that locks in the whitespace case so it can't silently regress.

**"Is this right for me?" checklist reasoning:**
- *Understanding:* I can restate the bug without re-reading it — the detector's regexes assume section headers start at column 0, so indentation preserved from PDF extraction causes false negatives. "Done" looks like: parsing an indented resume text (e.g. `"\n    Education:\n    - B.S. CS\n"`) returns `detected_sections` containing `Education` instead of `[]`.
- *Tier fit:* Tagged `tier-1` / `good first issue` on GitHub, and it matches that description — the whole fix lives in one method (`_detect_sections`, `resume_parser.py:127`), no cross-module or API changes needed. This is my first contribution to this codebase, so Tier 1 is the right level to start at rather than reaching for Tier 2/3.
- *Codebase readiness:* I've read `_detect_sections()` (`ingestion/parsers/resume_parser.py:127-146`) and confirmed the four regex patterns are all anchored with `^` or `\n` with no whitespace allowance — reproducing the bug is as simple as passing indented text through `.parse()`. I've also read `tests/unit/test_resume_parser.py`, including `test_detect_sections` (line 126) and the two tests the issue calls out by name, so I know the fixture/assertion style to follow for the new test.
- *Scope and time:* The issue thread is heavily claimed (~25 comment claims), which is normal/non-exclusive per the course guidance, but I did check that a PR (#178) already proposes a fix — I'm treating that only as a reference to compare my own implementation against, not something to copy, since the grade is based on my own artifacts. The actual change is small (the issue's own linked PR is +5/-5 lines), so 3-6 hours for implementation, a new whitespace test, and verifying the two existing tests it's blocking is realistic well before the Week 9 deadline. No blockers or dependencies are noted on the issue.

**Branch name:** fix/147-resume-section-whitespace

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

## Week 7 — Reproducing the bug

1. Added a new test, `test_detect_sections_with_leading_whitespace`, in `tests/unit/test_resume_parser.py` (lines 146-163). It gives `_detect_sections()` resume text where the section headers ("Experience:", "Education:", "Skills:") are indented, and checks that all three sections still get detected.
2. Ran it with `.venv/bin/pytest tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections_with_leading_whitespace -v` to reproduce the bug. The test fails right now, which confirms the bug: when the headers are indented, `_detect_sections()` finds nothing.