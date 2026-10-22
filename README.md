markdown
# LangGraph Human-in-the-Loop (HITL) Persona Agent
> An interactive, stateful agent workflow that dynamically generates domain-specific expert personas using Pydantic structured outputs and manages user review via LangGraph Human-in-the-Loop (HITL) interrupt checkpoints.
[![Python](https://img.shields.io/badge/Python-3.12+-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![LangGraph](https://img.shields.io/badge/LangGraph-Stateful%20Orchestration-1C3C3C)](https://www.langchain.com/langgraph)
[![LangChain](https://img.shields.io/badge/LangChain-LLM%20Workflows-1C3C3C)](https://www.langchain.com/)
[![OpenAI](https://img.shields.io/badge/OpenAI-gpt--4o-412991?logo=openai&logoColor=white)](https://platform.openai.com/)
[![Pydantic](https://img.shields.io/badge/Pydantic-v2-E92063?logo=pydantic&logoColor=white)](https://docs.pydantic.dev/)
[![uv](https://img.shields.io/badge/uv-Package%20Manager-DE5FE9)](https://docs.astral.sh/uv/)
---
## Overview
When conducting in-depth research or multi-agent simulations, asking an LLM for answers in a single generic prompt often yields biased or surface-level results. 
This project implements an interactive **Human-in-the-Loop Persona Agent** using **LangGraph**. Given any topic, the agent:
1. Automatically analyzes the topic and derives diverse, specialized expert analyst personas.
2. Enforces strict schema validation using **Pydantic** (`with_structured_output`).
3. **Pauses execution** at an interactive breakpoint using LangGraph's native `interrupt()` API.
4. Allows a human reviewer to either **approve** the personas or provide **editorial feedback** to regenerate them in a cyclical loop before proceeding.
---
## Core Capabilities (Resume Highlights)
- **Stateful Workflow Management**: Built a multi-agent orchestration graph using LangGraph state machines (`StateGraph`) with memory checkpointers (`MemorySaver`) to manage state transitions between generation and user approval steps.
- **Structured Persona Generation**: Utilized Pydantic schemas (`Analyst`, `Perspectives`) and LLM structured outputs to dynamically generate and validate tailored personas (Name, Role, Affiliation, and Focus Description).
- **Human-in-the-Loop Interruption**: Implemented state persistence and interrupt checkpoints in LangGraph (`interrupt()` and `Command(resume=...)`), allowing users to review, edit, or approve personas before execution finishes.
---
## Architecture Flow
```text
       ┌──────────────┐
       │    START     │
       └──────┬───────┘
              │ (topic, max_analysts)
              ▼
   ┌──────────────────────┐
   │   create_analysts    │◄─────────────────────────────┐
   │ (GPT-4o + Pydantic)  │                              │
   └──────────┬───────────┘                              │
              │                                          │
              ▼                                          │
   ┌──────────────────────┐                              │
   │    human_feedback    │                              │
   │ (LangGraph interrupt)│                              │
   └──────────┬───────────┘                              │
              │                                          │
              ▼                                          │
    [ User Review Step ]                                 │
              │                                          │
    Critique / Regenerate? ───► (feedback provided) ─────┘
              │
         (Approved)
              ▼
   ┌──────────────────────┐
   │        dummy         │
   └──────────┬───────────┘
              │
              ▼
       ┌──────────────┐
       │     END      │
       └──────────────┘

  

  
Project Structure

  
text
langGraph_assistant/
│
├── main.py                 # Interactive CLI demonstration of the HITL agent
├── langgraph.json          # LangGraph CLI and Studio configuration
├── pyproject.toml          # Project dependencies and metadata
├── uv.lock                 # Dependency lockfile
├── README.md               # Documentation
├── .env.example            # Environment variables template
│
└── src/
    ├── agent.py            # HITL Persona Agent graph definition & compilation
    └── utils/
        ├── objects.py      # Pydantic models (Analyst, Perspectives)
        ├── states.py       # Graph state definitions (GenerateAnalystsState)
        ├── nodes.py        # create_analysts, human_feedback implementations
        ├── edges.py        # should_continue conditional routing logic
        ├── models.py       # ChatOpenAI model initialization
        └── prompts.py      # System prompts for persona derivation and feedback
