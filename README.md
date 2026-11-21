# DeepAgents

A Jupyter walkthrough of [LangChain Deep Agents](https://github.com/langchain-ai/deepagents) on Groq, with Tavily for web search.

The notebook does four things:

- call a deep agent that can search the web
- compare that with a plain LangChain agent
- write a file in graph state, then on the real filesystem
- load project instructions from `agents.md`

Python 3.11 or newer. The lockfile pins `deepagents` 0.7.5, `langchain` 1.3.15, and `langchain-groq` 1.1.3.

## Quick start

```bash
uv sync
cp .env.example .env
```

Edit `.env` and add your keys. The file stays local. `.gitignore` already excludes it.

```text
GROQ_API_KEY=your-groq-key
TAVILY_API_KEY=your-tavily-key
```

```bash
uv run jupyter lab deepAgentsDemo/basicAgent.ipynb
```

Run the code cells from top to bottom. Later cells reuse `PROJECT_ROOT`, the Groq model, and `web_search` from the earlier cells.

Prefer pip:

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env
jupyter lab deepAgentsDemo/basicAgent.ipynb
```

`requirements.txt` lists the same packages without versions. `uv sync` follows `uv.lock` and is the more repeatable install.

## Repository

```text
deepAgentsDemo/
  basicAgent.ipynb          demo
  project/agents.md         Researcher and Todo Writer instructions
pyproject.toml              package name deepagents-app
uv.lock                     pinned dependencies
requirements.txt            unpinned install list
.env.example                GROQ_API_KEY and TAVILY_API_KEY
```

## Notebook tour

**Keys.** The first cell finds the folder that contains `pyproject.toml`, then loads `.env` from there. Jupyter can be started inside `deepAgentsDemo/` and the keys still resolve.

**Search tool.** `web_search` calls Tavily. `topic` is `general`, `news`, or `finance`. The default returns five results.

**Model.** `init_chat_model` uses Groq and the id `qwen/qwen3.6-27b`. If Groq rejects that id, replace it with a current tool-calling model on your account.

**Two agents, one model.**

| Name | Built with | Role in the notebook |
| --- | --- | --- |
| `simple_agent` | `langchain.agents.create_agent` | Constructed, then left unused |
| `deep_agent` | `deepagents.create_deep_agent` | System prompt `Act as a Researcher`, then asked “What is LangGraph?” |

The saved output for that question shows the model planning a search and calling `web_search`.

**State backend.** A new deep agent uses `StateBackend` and `MemorySaver`. Files it writes stay in the graph state under `result["files"]`. They are not written next to the notebook. The prompt asks it to create and edit `todo1.txt`. The saved run stored `/home/user/todo1.txt`. This agent does not receive `web_search`. Deep Agents supplies its own file tools, which is what the todo steps use. Pass `config={"configurable": {"thread_id": "state-backend-demo"}}` so the checkpointer can keep the thread.

**Filesystem backend.** `FilesystemBackend(root_dir=deepAgentsDemo, virtual_mode=True)` maps agent paths onto that directory. `/project/agents.md` is `deepAgentsDemo/project/agents.md` on disk.

**Memory.** `memory=["/project/agents.md"]` loads those instructions into the agent. The file defines two roles:

- **Researcher.** Prefer web search for current facts.
- **Todo Writer.** Create and edit todo files clearly.

The first deep agent does not load this file. Only the filesystem agent in the memory section does. Its system prompt is the one-line “Act as a Researcher”.

**Structured output.** The last section defines a Pydantic model, `researchFindings`, with `summary` and `findings`. No subagent is built from it, and the cell after it is empty.

## Model and keys

Groq model ids change. The notebook comments that `qwen/qwen3-32b` was not available, which is why it uses `qwen/qwen3.6-27b`.

Get a Groq key from the Groq console and a Tavily key from Tavily. Both are read only from the environment. Saved notebook output masks the Groq key as `SecretStr('**********')`. Re-run the notebook if you want fresh output. Those saved cells still mention the machine path from the earlier run.
