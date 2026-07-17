## Week 7 — Issue selection

**Issue link:** https://github.com/DavidSong74/pathreview/tree/fix/147-resume-section-whitespace

**Issue title:** Resume section detection fails on text with leading whitespace

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The problem here is that the resume section's detection system fails on text that input whitespaces in the beginning. This is probably because of a parsing error, since it is waiting for digits/alphabets to be inputted as the beginning letters. However, since an unexpected input is detected, the section detection system fails. 

**Branch name:** fix/147-resume-section-whitespace

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger