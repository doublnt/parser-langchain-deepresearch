# Repository Guidelines

## Project Structure & Module Organization
- `research_agent.py` is the orchestration hub: it wires LangGraph, LangChain, DeepAgents middleware, and all sub-agents (“initial scout” → “report merger”). Treat this file as the canonical source for graph changes.
- `tools.py` contains shared utilities: Tavily-powered `WebSearch/WebFetch`, `GetCurrentDate`, and `get_indicator_tools()` (wrappers around `langchain_tools.py` Mai SDK + remote OHLC fetcher).
- `langchain_tools.py` houses LangChain `BaseTool` classes (`indicator_parser`, `indicator_calculator`, etc.) and the HTTP market-data loader. Keep the DLL `lib_hevo_parser.dll` alongside it.
- Prompts live in `prompts/`. New stages should follow the `*_agent.md` naming pattern and be referenced via `load_prompt`.
- Runtime artifacts are written to `research_assets/` (ignored). Configuration and dependencies sit in `env.example`, `langgraph.json`, and `requirements.txt`.

## Build, Test, and Development Commands
- `uv venv --python 3.12 && source .venv/bin/activate` — create/activate the recommended virtual environment.
- `uv pip install -r requirements.txt` — install LangGraph CLI, DeepAgents, Tavily client, Mai SDK bindings (`cffi`), and HTTP fetch deps (`requests`).
- `uv run langgraph dev --config langgraph.json` — launch the LangGraph Dev UI at `http://127.0.0.1:2024` for local experimentation with the full agent graph.
- Manual tool smoke tests: `uv run python - <<'PY' ... fetch_market_data('AAPL')` ensures remote OHLC fetching/dll integration still works.

## Coding Style & Naming Conventions
- Python 3.12+, 4-space indentation, snake_case identifiers. Keep docstrings/comments bilingual only when the surrounding file already mixes CN/EN.
- New LangChain tools should inherit `BaseTool`, expose a Pydantic `args_schema`, and return JSON via `create_success_response`.
- Prompts should stay in Markdown files; avoid inline mega-strings.

## Testing Guidelines
- No automated suite yet. Validate changes by running `uv run langgraph dev` and walking through a representative research task (check `research_assets/` outputs).
- When editing `langchain_tools.py`, manually call the relevant tool or `fetch_market_data` via `uv run python` to confirm DLL + HTTP paths still function.

## Commit & Pull Request Guidelines
- Use concise, imperative commit messages (e.g., “Add indicator plot tool”); mention issue IDs in the body when applicable.
- PRs should describe which agent stages/tools/prompts changed, outline manual test steps (LangGraph run, tool invocation), and attach screenshots/log snippets if the output format changed.
- Never commit `.env`, API keys, or generated `research_assets/` files; document new environment settings in `env.example` instead.
