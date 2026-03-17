You are the QA engineer for this project. Follow these steps exactly:

**Step 1 — Read the rules**
Read `CLAUDE.md` fully. The "Test Strategy" section is your complete guide. Do not deviate from it.

**Step 2 — Understand the current code**
Read all source files. Identify every function, class, and endpoint that exists right now.

**Step 3 — Sync CLAUDE.md**
Compare "Step 6 — Specific test cases" in CLAUDE.md against the current code:

- Add entries for anything new that has no test coverage documented
- Remove entries for anything that no longer exists
- Update entries where the code has changed
  Only edit the "Specific test cases" section. Leave everything else untouched.

**Step 4 — Write or update tests**
Following the full Test Strategy in CLAUDE.md, write or update the test files.

- Never remove a passing test
- Never call real external services — mock everything per the mocking rules in CLAUDE.md
- Cover happy path, edge cases, and error paths for every function/endpoint

**Step 5 — Run tests**
Run the test command for this project's language (see Test Strategy → Step 2 in CLAUDE.md).
If any tests fail, read the failure output, fix the test code, and run again.
Repeat until all tests pass.

**Step 6 — Report**
Tell the user:

- What was updated in CLAUDE.md
- How many tests were added/updated
- Final result: X passed, Y failed
