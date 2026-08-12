# CrewAI Automatic Deep Research

A modular, configuration-driven multi-agent deep research pipeline built with [CrewAI](https://www.crewai.com/). Given a query, it plans, gathers, verifies, and synthesizes web research into a final cited report — without any hardcoded prompts in code.

## How it works

`automatic_deep_research_modular.py` orchestrates four agents in a sequential crew:

| # | Agent | Role | Task |
|---|-------|------|------|
| 1 | **Research Planner** | Strategic analyst | Breaks the query into research topics, investigative questions, and success metrics |
| 2 | **Internet Researcher** | Digital sleuth | Executes the plan via Exa search + website scraping, mapping every fact to a source URL |
| 3 | **Fact Checker** | Meticulous auditor | Cross-references findings, flags inconsistencies/misinformation, closes gaps |
| 4 | **Report Writer** | Technical writer | Synthesizes verified data into a professional, cited executive report |

Agent and task instructions are **not hardcoded** — they're loaded from markdown files:

- `agent_definitions/` — role, goal, and backstory per agent
- `task_definitions/` — description and expected output per task

The `{user_query}` placeholder in task definitions is filled at runtime via `Crew.kickoff(inputs=...)`.

## Tools

- **Exa search** (`EXASearchTool`) — web search, configured via `EXA_BASE_URL`
- **Website scraping** (`ScrapeWebsiteTool`) — full-page content extraction

## Setup

```bash
# 1. Create environment (or use any Python 3.10+ env)
uv venv
source .venv/bin/activate

# 2. Install dependencies
uv add crewai crewai_tools exa_py python-dotenv

# 3. Configure API keys — copy example.env to .env and fill in real values
cp example.env .env
```

`.env` (never committed — see `.gitignore`):

```
OPENAI_API_KEY=...
OPENAI_API_BASE=https://api.openai.com/v1
EXA_API_KEY=...
EXA_BASE_URL=https://api.exa.ai/search
```

## Run

From the repository root:

```bash
python automatic_deep_research_modular.py
```

The final report is printed to stdout (and rendered as markdown in notebook contexts). To research a different topic, change the `query` variable in `automatic_deep_research_modular.py:137`.

## Project layout

```
.
├── automatic_deep_research_modular.py  # crew orchestration (agents, tasks, kickoff)
├── agent_definitions/                  # markdown: agent role/goal/backstory
├── task_definitions/                   # markdown: task description/expected output
├── utils.py                            # .env loading + API key accessors
├── patch.py                            # optional SSL-verification disable for lab/proxy envs
├── example.env                         # key template
└── .env                                # real keys (gitignored)
```

## Notes

- `patch.py` disables SSL verification — intended for constrained lab/proxy environments only; remove the `disable_ssl_verification()` call in `automatic_deep_research_modular.py:15` for production use.
- Agent runs are capped (`max_iter=2`, `max_rpm=10`) to keep API costs bounded.
