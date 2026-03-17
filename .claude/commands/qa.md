Run: git diff --name-only HEAD

Only read the files that changed. Do not read unchanged files.

Then follow these steps:

1. Read CLAUDE.md — only the "Test Strategy" section
2. For each changed file, identify only the functions/endpoints that were added or modified
3. Update only the relevant entries in "Step 6 — Specific test cases" in CLAUDE.md (add/update/remove entries for what changed — nothing else)
4. In tests/test_app.py, add or update only the tests for the changed functions/endpoints — do not rewrite the whole file
5. Run: pytest tests/ -v --tb=short
6. If failures, fix only the failing tests and run again
7. Report: which functions were tested, how many tests added/updated, pass/fail count
