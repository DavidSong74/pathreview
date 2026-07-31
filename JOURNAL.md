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

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/DavidSong74/pathreview/commit/d943c775799fb5a2dbcbba31c8c244183ad13f04

**Reproduction summary:**
I wrote a new test, `test_detect_sections_with_leading_whitespace`, that feeds `_detect_sections()` resume text with indented section headers and asserts that Experience, Education, and Skills all still get detected. Running it against the current code confirms the bug: `detected_sections` comes back empty because the regex patterns are anchored with `^`/`\n` and don't allow for the leading whitespace.

**PLAN.md link:** https://github.com/DavidSong74/pathreview/blob/fix/147-resume-section-whitespace/PLAN.md

**Walkthrough video (recommended):** [link to your Loom video, ≤2 min — recommended, not graded]

**Blockers or open questions:**
None currently. One thing I'm keeping an eye on: issue #147 already has an open PR (#178) proposing a similar fix. I'm implementing independently and treating #178 only as a reference to sanity-check against afterward, per the course guidance that grading is based on my own artifacts — but if #178 merges before I submit, I may need to pick a different issue.

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
Implemented the fix from PLAN.md step 2: in `_detect_sections()` (`ingestion/parsers/resume_parser.py:132-137`), added `\s*` right after each `^`/`\n` anchor in the four section-header regex patterns, so a leading indentation before a header (e.g. `"    Education:"`) no longer prevents a match. The reproduction test from Week 7/8 (`test_detect_sections_with_leading_whitespace`) now passes, along with the three other tests noted in PLAN.md as failing for the same underlying reason (`test_parse_single_column_resume_text`, `test_parse_resume_no_work_experience`, `test_detect_sections`).

Before touching anything I ran the full `tests/unit` suite and `ruff`/`black`/`mypy` on the file to record the baseline: 54 unit tests failing pre-existing (unrelated modules — faithfulness checker, PII scrubber, review service, etc.), plus two pre-existing failures in `test_resume_parser.py` itself (`test_parse_markdown_resume`, `test_strip_markdown_syntax` — a `_strip_markdown()` header-regex bug, not the section-detection bug I'm fixing) and two pre-existing lint issues (unsorted imports, a `B904` bare `raise` in `_parse_pdf`). After my change: 50 unit-suite failures (the 4 I fixed are gone, nothing else changed) and the same 2 pre-existing `test_resume_parser.py` failures, confirmed unaffected by diffing against a stashed copy of the original code. I also fixed the pre-existing `B904` issue (`raise ... from e`) since the pre-commit hook wouldn't let me commit otherwise — one-line, unrelated to the section-detection logic itself.

**Next steps:**
Push the branch, open a draft PR against `ascherj/pathreview` referencing issue #147 (filling in `.github/PULL_REQUEST_TEMPLATE.md` and documenting the pre-existing failures per the course guidance), get peer/mentor feedback in Slack, then mark it ready for review and finish Check-in 2.

**Blockers:**
None. Still watching whether PR #178 (an existing fix for the same issue) merges before mine is up — if it does I'll switch to a different open issue, per the plan noted in Week 8's blockers.

---

### Check-in 2 (end of week)

**PR link:** [link to your submitted pull request]

**Branch:** `fix/147-resume-section-whitespace`

**What you built:**
Fixed `_detect_sections()` in `ingestion/parsers/resume_parser.py` so section-header regexes tolerate leading whitespace (`^\s*`/`\n\s*` instead of bare `^`/`\n`), so indented resume text — common when text is extracted from PDFs — is parsed the same as flush-left text instead of returning an empty `detected_sections` list.

**Tests added or updated:**
`tests/unit/test_resume_parser.py` — added `test_detect_sections_with_leading_whitespace` (indented `Experience`/`Education`/`Skills` headers must still be detected). No other test files needed changes; three existing tests in this file that were failing for the same root cause now pass without modification.

**Self-review confirmation:** [x] make check passes  [x] make test-unit passes
*(Both pass modulo the pre-existing failures documented above and in the PR description — my change introduces no new failures. Verified specifically via `test_detect_sections_with_leading_whitespace` in `tests/unit/test_resume_parser.py:148-163` — the bug-reproduction test from Week 7/8, which failed against the original code and now passes against this fix.)*

**Draft PR feedback received from:** [name or Slack handle, or "none"]