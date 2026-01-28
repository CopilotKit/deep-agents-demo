# Deep Research Assistant

A [CopilotKit](https://copilotkit.ai) Deep Agents demo showcasing planning, memory/files, subagent spawning, and generative UI using [Firecrawl](https://www.firecrawl.dev/) for web research.

## What This Demo Shows

This demo showcases all four key Deep Agents capabilities:

- **Planning (Todos)** - Visible research plan with status indicators (pending, in progress, completed)
- **Memory/Files** - Markdown files created by the agent, viewable in the workspace with download option
- **Subagent Delegation** - Researcher subagent handles deep-dive searches with activity tracking
- **Generative UI** - Rich tool call rendering with result summaries and expandable details
- **Web Research** - Firecrawl-powered search and page scraping for real-time information

## Architecture

```
[User asks research question]
        ↓
Next.js Frontend (CopilotChat + Workspace)
        ↓
CopilotKit Runtime → LangGraphHttpAgent
        ↓
Python Backend (FastAPI + AG-UI)
        ↓
Deep Agent (research_assistant)
    ├── write_todos (planning via middleware)
    ├── web_search (Firecrawl)
    ├── scrape_url (Firecrawl)
    ├── write_file (filesystem via middleware)
    └── task → researcher subagent
                 ├── web_search
                 └── scrape_url
```

## Project Structure

```
deep-research-v2/
├── src/                              # Next.js frontend
│   ├── app/
│   │   ├── layout.tsx               # CopilotKit provider
│   │   ├── page.tsx                 # Main page with useDefaultTool
│   │   ├── globals.css              # Glassmorphism styles
│   │   └── api/copilotkit/route.ts  # CopilotRuntime endpoint
│   ├── components/
│   │   ├── Workspace.tsx            # Research progress display
│   │   ├── ToolCard.tsx             # Generative UI for tools
│   │   └── FileViewerModal.tsx      # Markdown file viewer
│   └── types/
│       └── research.ts              # TypeScript types
│
├── agent/                           # Python backend
│   ├── main.py                      # FastAPI server + AG-UI
│   ├── agent.py                     # Deep Agent definition
│   ├── tools.py                     # Firecrawl tools
│   └── pyproject.toml               # Python dependencies
│
├── .env.example                     # Environment variables
└── README.md                        # This file
```

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `OPENAI_API_KEY` | Yes | - | [Get API key](https://platform.openai.com/api-keys) |
| `FIRECRAWL_API_KEY` | Yes | - | [Get API key](https://www.firecrawl.dev/app/api-keys) |
| `OPENAI_MODEL` | No | `gpt-4o` | Model to use (gpt-4o, gpt-4-turbo, etc.) |
| `LANGGRAPH_DEPLOYMENT_URL` | No | `http://localhost:8123` | Backend URL |
| `SERVER_HOST` | No | `0.0.0.0` | Backend host |
| `SERVER_PORT` | No | `8123` | Backend port |

## Setup & Installation

### Backend (Python)

```bash
cd agent
uv venv && source .venv/bin/activate
uv pip install -e .
```

Or with pip:

```bash
cd agent
python -m venv .venv && source .venv/bin/activate
pip install -e .
```

### Frontend (Node.js)

```bash
npm install
```

### Environment

Copy `.env.example` to `.env` in both the root directory and `agent/` directory, then fill in your API keys.

## Running Locally

**Terminal 1 - Backend:**
```bash
cd agent
source .venv/bin/activate
uvicorn main:app --port 8123
```

**Terminal 2 - Frontend:**
```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) and ask the assistant to research any topic.

## Key Patterns

### Frontend: useDefaultTool (not useCoAgent)

This demo uses local React state with `useDefaultTool` instead of `useCoAgent` to avoid type mismatches between Python's FilesystemMiddleware (Dict) and TypeScript (Array):

```typescript
const [state, setState] = useState<ResearchState>(INITIAL_STATE);

useDefaultTool({
  render: (props) => {
    // Update local state based on tool results
    if (name === "write_todos" && status === "complete") {
      setState(prev => ({ ...prev, todos: result.todos }));
    }
    return <ToolCard {...props} />;
  },
});
```

### Backend: Deep Agent with Subagent

```python
agent = create_deep_agent(
    name="research_assistant",
    model=ChatOpenAI(model="gpt-4o"),
    tools=[web_search, scrape_url],
    subagents=[{
        "name": "researcher",
        "description": "Deep-dive researcher for specific topics",
        "system_prompt": "You are a research specialist...",
        "tools": [web_search, scrape_url],
    }],
)
```

## Learn More

- [Deep Agents Documentation](https://docs.copilotkit.ai/langgraph/deep-agents)
- [Building Frontends for Deep Agents](https://www.copilotkit.ai/blog/how-to-build-a-frontend-for-langchain-deep-agents-with-copilotkit)
- [CopilotKit Documentation](https://docs.copilotkit.ai)
- [Firecrawl Documentation](https://docs.firecrawl.dev)

## License

MIT
