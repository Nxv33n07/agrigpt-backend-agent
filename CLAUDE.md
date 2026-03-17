# AgriGPT Backend Agent — Claude Instructions

## Project Overview

FastAPI + LangGraph agent serving an agricultural chatbot.

- **Language**: Python
- **Entry point**: `app.py`
- **Framework**: FastAPI
- **LLM**: Google Gemini 2.5 Flash via LangChain
- **Tools**: Discovered dynamically from MCP servers (Alumnx, Vignan)
- **Memory**: MongoDB (`agrigpt.chats` collection)
- **Deploy**: EC2 via SSH

## How CI/CD Works

```
Developer writes code
  └── /qa (skill)         → Claude writes/updates tests → runs them → fixes failures

git push
  └── pre-push hook       → runs tests locally → blocks push if failing

PR opened
  ├── claude-review.yml   → Claude reviews code → posts findings as PR comment
  └── claude-test-gen.yml → Claude writes/updates tests → runs them → fixes failures → commits back

Merge to main
  └── deploy.yml          → SSH into EC2 → git pull → restart service
```

## Required GitHub Secrets

| Secret                    | Purpose                                          |
| ------------------------- | ------------------------------------------------ |
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude GitHub App auth (auto-set by App install) |
| `GITHUB_TOKEN`            | Auto-provided by GitHub Actions                  |
| `EC2_HOST`                | EC2 instance IP                                  |
| `EC2_USER`                | SSH username                                     |
| `EC2_SSH_KEY`             | Contents of `.pem` file                          |

## Onboarding (run once after cloning)

```bash
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
sh setup-hooks.sh
```

---

## Code Review Rules

When Claude reviews a PR, check for all of these:

1. No secrets or credentials hardcoded — all config via environment variables
2. No bare `except:` — always catch specific exception types
3. All FastAPI request/response bodies use Pydantic models
4. Async endpoints must not block — offload long work to `BackgroundTasks`
5. External service failures (MCP, MongoDB, LLM) must be caught and handled gracefully
6. No hardcoded URLs — all service URLs from environment variables
7. `global_tool_results` cleared at the start of every `/test/chat` request

---

## Test Strategy

> This is the single source of truth for how Claude writes tests.
> All test rules live here. Workflow files and the /qa skill read this section — they contain no test logic of their own.
> When the codebase changes, Claude updates this section automatically before writing tests.

### Step 1 — Detect the project

Before writing tests, Claude must:

1. Read all source files to understand what the project does
2. Identify the language (Python, TypeScript, Go, Java, etc.)
3. Identify the framework (FastAPI, Express, Spring, etc.)
4. Identify all functions, classes, endpoints, and modules that need testing
5. Check what test files already exist and what is already covered

### Step 2 — Choose the right test framework

| Language      | Framework      | Test runner command           |
| ------------- | -------------- | ----------------------------- |
| Python        | pytest         | `pytest tests/ -v --tb=short` |
| TypeScript/JS | Jest or Vitest | `npx jest` or `npx vitest`    |
| Go            | testing        | `go test ./...`               |
| Java          | JUnit          | `mvn test`                    |

For this project: **Python + pytest**

Required packages: `pytest pytest-asyncio httpx pytest-mock`

### Step 3 — What to test

For every function and endpoint that exists in the codebase, generate tests covering:

- **Happy path** — valid inputs produce correct outputs
- **Edge cases** — empty strings, `None`, missing optional fields, zero-length lists, boundary values
- **Error path** — dependencies fail (database down, external API unreachable, invalid input)

### Step 4 — Mocking rules

Never call real external services in tests. For this project:

| Dependency      | Mock target                                         |
| --------------- | --------------------------------------------------- |
| MongoDB         | `unittest.mock.patch("app.chat_sessions")`          |
| MCP HTTP calls  | `unittest.mock.patch("app.httpx.Client")`           |
| Gemini LLM      | `unittest.mock.patch("app.ChatGoogleGenerativeAI")` |
| LangGraph agent | `unittest.mock.patch("app.app_agent")`              |

Use `pytest-mock` (`mocker` fixture) or `unittest.mock.patch`.
Never use real environment variables — use `monkeypatch.setenv` or patch `os.getenv`.

### Step 5 — Standard fixtures

Always define these fixtures in `tests/test_app.py`:

```python
@pytest.fixture
def chat_id():
    return "test-chat-123"

@pytest.fixture
def phone():
    return "919999999999"
```

### Step 6 — Specific test cases for this project

> Claude updates this section whenever `app.py` changes — adding entries for new functions/endpoints, removing entries for deleted ones.

**`load_history`**

- Returns `[]` when `chat_id` not in MongoDB
- Correctly reconstructs `HumanMessage`, `AIMessage`, `SystemMessage`
- Ignores unknown roles silently

**`save_history`**

- Stores all messages when count ≤ 20
- Trims to last N human/AI pairs when count > 20
- Upserts correctly (creates on first call, updates on subsequent)
- Only sets `phone_number` when provided

**`extract_sources_from_tool_results`**

- Extracts `filename` from `sources[].filename` dict format
- Extracts plain string from `sources[]` list format
- Extracts `source` field from `results[].source` format
- Extracts `source`/`document`/`filename` from list-of-dicts (VignanUniversity style)
- Returns `[]` for empty input
- Falls back to tool name when no source field found

**`has_meaningful_tool_results`**

- Returns `True` for non-empty list result
- Returns `True` when `sources` list is non-empty
- Returns `True` when `information` string > 50 chars
- Returns `False` for error status
- Returns `False` for empty list

**`/webhook GET`**

- Returns `hub.challenge` when token matches
- Returns 403 when token mismatches

**`/webhook POST`**

- Returns `{"status": "ok"}` for non-text messages
- Returns `{"status": "ok"}` for empty messages array
- Parses phone number and body correctly for valid text message

**`/test/chat POST`**

- Returns `ChatResponse` with sources when tools return results
- Uses Gemini fallback when tools return no meaningful results
- Clears `global_tool_results` at start of each request
- Returns 500 on unhandled exception

**`/hello GET`** — returns `{"message": "Hello Claude!!"}`

**`/hi GET`** — returns `{"message": "Hi Claude"}`

### Step 7 — After writing tests

```bash
git add CLAUDE.md tests/
git commit -m "test: auto-generate/update tests via Claude [skip ci]"
git push
```

---

## Development Conventions

- Match existing code style exactly — no reformatting
- Commit prefix: `feat:` `fix:` `test:` `chore:`
- `global_tool_results` is module-level, cleared per request
- System prompt injected fresh each request, stripped before MongoDB save
