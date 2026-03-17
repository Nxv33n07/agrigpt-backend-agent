Analyze all Python files in this project to understand the current code.

Then generate or update `tests/test_app.py` following the **Test Strategy** section in `CLAUDE.md` exactly.

Steps to follow:

1. Read `CLAUDE.md` — follow the Test Strategy section as the single source of truth
2. Read `app.py` — identify every function and endpoint that exists right now
3. Compare with `tests/test_app.py` if it exists — find any functions/endpoints that have no test or whose tests are outdated
4. Write or update `tests/test_app.py` so every function and endpoint is covered
5. Also create `tests/__init__.py` if it does not exist (empty file)
6. Run the tests: `pytest tests/ -v --tb=short`
7. If any tests fail, read the failure output, fix `tests/test_app.py`, and run again
8. Report: how many tests were added/updated, how many pass, how many fail

Rules:

- Never remove existing passing tests
- Never call real external services — mock everything as described in CLAUDE.md
- Never ask the user what to test — decide based on what exists in the code
