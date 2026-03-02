# 🤖 Multi-Agent Workflow System

> **300-Character Description:**  
> Multi-agent orchestration system using LangGraph & LangChain. A Planner, Executor, Web Researcher, Chart Generator & Synthesizer collaborate via shared state to answer complex queries with real-time data and auto-generated visualizations.

---

## 📌 Overview

This project implements a **multi-agent orchestration system** that breaks complex user queries into structured execution plans and delegates tasks to specialized AI agents. Each agent has a distinct role — from fetching live web data to generating charts — and they collaborate through a shared state managed by **LangGraph**.

---

## 🏗️ Project Structure
```
multi-agent-workflow/
│
├── Multi Agent workflow.ipynb   # Main notebook — agents & graph execution
├── requirements.txt             # Pinned Python dependencies
├── README.md                    # Project documentation
└── .env                         # API keys (not committed to git)
```

---

## 🧠 System Architecture

The system follows a **Plan → Execute → Specialize → Synthesize** pattern:
```
User Query
    │
    ▼
┌─────────┐
│ Planner │  ── Generates a step-by-step JSON execution plan
└────┬────┘
     ▼
┌──────────┐
│ Executor │  ── Routes each step to the correct agent
└────┬─────┘
     ├─────────────────────────────────┐
     ▼                                 ▼
┌───────────────┐           ┌──────────────────┐
│ Web Researcher│           │ Chart Generator  │
│ (Tavily API)  │           │ (Matplotlib)     │
└───────┬───────┘           └────────┬─────────┘
        │                            │
        │                  ┌──────────────────┐
        │                  │ Chart Summarizer │
        │                  └────────┬─────────┘
        └──────────┬────────────────┘
                   ▼
          ┌─────────────┐
          │ Synthesizer │  ── Combines all outputs → Final Answer
          └─────────────┘
```

---

## 🤖 Agent Descriptions

| Agent | Role | Technology |
|-------|------|------------|
| **Planner** | Parses user query and generates a structured JSON plan with numbered steps and agent assignments | GPT-4o-mini |
| **Executor** | Reads the plan, routes each step to the right agent, manages `current_step` in state | LangGraph `Command` |
| **Web Researcher** | Performs live web searches and returns formatted results with source URLs | Tavily Search API |
| **Chart Generator** | Extracts numerical data from agent messages and renders a labeled bar chart | Matplotlib |
| **Chart Summarizer** | Reads chart output and produces 2–3 sentence key insights | GPT-4o-mini |
| **Synthesizer** | Aggregates all agent outputs into one coherent final answer | GPT-4o-mini |

---

## 🗂️ State Management

All agents communicate through a **shared State object** — no direct agent-to-agent coupling:
```python
class State(MessagesState):
    user_query: str           # The original user question
    enabled_agents: List[str] # Agents active for this query
    plan: List[Dict]          # Ordered execution steps
    current_step: int         # Tracks which step is currently running
    agent_query: str          # Task instruction for the current agent
    last_reason: str          # Executor's reasoning for routing decision
    replan_flag: bool         # Triggers planner to revise the plan
    replan_attempts: Dict     # Replanning count tracked per step
    final_answer: str         # Final synthesized response
```

> `State` inherits from `MessagesState`, which adds a shared `messages` list visible to all agents.

---

## 📦 Installation
```bash
# 1. Clone the repository
git clone <repo-url>
cd multi-agent-workflow

# 2. Create virtual environment
python -m venv venv
source venv/bin/activate       # macOS / Linux
venv\Scripts\activate          # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Set up API keys
# Create a .env file in the project root:
OPENAI_API_KEY=your-openai-api-key-here
TAVILY_API_KEY=your-tavily-api-key-here
```

---

## 🚀 Usage

Run notebook cells sequentially. Two example queries are included:

### Example 1 — Data Visualization
```python
query = "Chart the current market capitalization of the top 5 banks in the US?"
state = {
    "messages": [HumanMessage(content=query)],
    "user_query": query,
    "enabled_agents": ["web_researcher", "chart_generator", "chart_summarizer", "synthesizer"],
}
result = graph.invoke(state)
print(result["final_answer"])
```

### Example 2 — Regulatory Research
```python
query = "Identify current regulatory changes for the financial services industry in the US."
state = {
    "messages": [HumanMessage(content=query)],
    "user_query": query,
    "enabled_agents": ["web_researcher", "chart_generator", "chart_summarizer", "synthesizer"],
}
result = graph.invoke(state)
print(result["final_answer"])
```

> ⏳ Queries typically take **2–5 minutes** due to chained LLM + API calls.

---

## ⚙️ Adding a Custom Agent
```python
# Step 1 — Define the node function
def my_custom_agent(state: State) -> Command:
    result = do_something(state.get("agent_query", ""))
    return Command(
        update={"messages": [HumanMessage(content=result, name="my_agent")]},
        goto="executor"
    )

# Step 2 — Register in the graph
workflow.add_node("my_agent", my_custom_agent)

# Step 3 — Include in enabled_agents when running
"enabled_agents": ["web_researcher", "my_agent", "synthesizer"]
```

---

## 🛡️ Error Handling & Fallbacks

| Scenario | Fallback Behavior |
|----------|-------------------|
| Tavily API key missing | Uses mock data for 5 major US banks |
| LLM returns invalid JSON plan | Falls back to a default 2-step plan |
| Chart data extraction fails | Uses hardcoded default bank values |
| Graph PNG visualization fails | Prints list of registered node names |

---

## ⚠️ Known Limitations

- **Non-deterministic routing** — Planner may skip chart generation and go directly to Synthesizer; this is expected LLM behavior
- **`langgraph==0.1.0` unavailable on PyPI** — Use `langgraph>=0.2.x`; fully compatible with this codebase
- **API rate limits** — Each query chains multiple OpenAI + Tavily calls; monitor usage on heavy workloads

---

## 📚 Tech Stack

| Library | Version | Purpose |
|---------|---------|---------|
| `langchain` | 0.2.0 | Core agent & chain framework |
| `langchain-openai` | 0.1.7 | GPT-4o-mini LLM integration |
| `langchain-community` | 0.2.0 | Tavily search tool integration |
| `langgraph` | 0.1.0+ | Agent graph orchestration |
| `tavily-python` | 0.5.0 | Real-time web search |
| `matplotlib` | 3.9.2 | Chart generation |
| `pandas` | 2.2.3 | Data manipulation |
| `seaborn` | 0.13.2 | Enhanced plot styling |

---

## 📄 License

Educational implementation — part of a multi-agent systems course on LangChain and LangGraph. Free to use and adapt for learning purposes.

---

## 📚 Resources

- [LangGraph Docs](https://github.com/langchain-ai/langgraph)
- [LangChain Docs](https://docs.langchain.com/)
- [Tavily API](https://tavily.com/docs)
- [OpenAI API](https://platform.openai.com/docs)
