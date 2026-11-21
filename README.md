# DeepAgents

A small Jupyter demo of [LangChain Deep Agents](https://github.com/langchain-ai/deepagents) on Groq, with Tavily web search. The notebook builds a plain LangChain agent and a deep agent, then shows two file backends and project memory.

The lockfile pins `deepagents` 0.7.5, `langchain` 1.3.15, and `langchain-groq` 1.1.3. Python 3.11 or newer is required.

## Layout

| Path | What it is |
| --- | --- |
| `deepAgentsDemo/basicAgent.ipynb` | The demo. Run the cells from top to bottom. |
| `deepAgentsDemo/project/agents.md` | Memory file loaded by the filesystem agent. It defines a Researcher and a Todo Writer. |
| `pyproject.toml` | Project metadata and dependencies for `uv`. |
| `uv.lock` | Pinned dependency set. |
| `requirements.txt` | The same packages without pins, for a plain `pip` install. |
| `.env.example` | The two API keys the notebook expects. |

## What the notebook does

1. Loads `GROQ_API_KEY` and `TAVILY_API_KEY` from a `.env` file at the project root. The root is the folder that contains `pyproject.toml`, so the notebook still works when Jupyter's working directory is `deepAgentsDemo/`.
2. Defines `web_search`, a Tavily tool. The topic can be `general`, `news`, or `finance`.
3. Builds a Groq chat model with `init_chat_model`. The model id in the notebook is `qwen/qwen3.6-27b`. Groq model ids change; if that id is rejected, swap it for a current tool-calling model on your Groq account.
4. Builds two agents with that model and `web_search`:
   - `simple_agent` from `langchain.agents.create_agent`
   - `deep_agent` from `deepagents.create_deep_agent`, with the system prompt `Act as a Researcher`
5. Asks the deep agent `What is LangGraph?`. The saved output shows the model planning a search and calling `web_search`.
6. **State backend.** A second deep agent uses `StateBackend` and an in-memory `MemorySaver`. Files it writes live in the graph state, not on disk. The prompt asks it to create and edit `todo1.txt`. The saved run stored `/home/user/todo1.txt` in `result["files"]`.
7. **Filesystem backend.** `FilesystemBackend` is rooted at `deepAgentsDemo/` with `virtual_mode=True`. Paths the agent sees, including `/project/agents.md`, are virtual paths under that root.
8. **Memory.** `create_deep_agent(..., memory=["/project/agents.md"])` loads the Researcher and Todo Writer instructions from `agents.md` into the agent.

The last section, "Structured Output with SubAgents", only declares a Pydantic model, `researchFindings`, with `summary` and `findings`. No subagent is created, and the following cell is empty. `simple_agent` is constructed and never called.

## Run it

From this folder:

```bash
uv sync
cp .env.example .env
```

Put your keys in `.env`:

```text
GROQ_API_KEY=your-groq-key
TAVILY_API_KEY=your-tavily-key
```

`.env` is gitignored. Then:

```bash
uv run jupyter lab deepAgentsDemo/basicAgent.ipynb
```

Run every code cell in order. Later cells use `PROJECT_ROOT`, `model`, and the search tool from the earlier cells.

Without `uv`:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
jupyter lab deepAgentsDemo/basicAgent.ipynb
```

On Windows, activate the venv with `.venv\Scripts\activate`.

## Notes from the code

- The notebook used to point at `/Users/varun.patil/Desktop/DeepAgents`. Those paths now resolve from the project root, so a clone runs on another machine.
- The first deep agent does not receive `agents.md`. Only the filesystem agent in the memory section does.
- The state-backend agent does not get `web_search`. It uses the filesystem tools that Deep Agents adds on its own, which is what the todo example needs.
- Saved cell outputs are from a previous run. The Groq key in that output is already masked. Re-run the notebook to refresh them.
