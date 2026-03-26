# OpenSage Replication

Python reimplementation of **OpenSage: Self-programming Agent Generation Engine** based on [arXiv:2602.16891](https://arxiv.org/pdf/2602.16891).

## Overview

OpenSage is the first Agent Development Kit (ADK) that enables LLMs to automatically create agents with self-generated topology and toolsets while providing comprehensive and structured memory support.

### Key Innovations

| Component | Description |
|-----------|-------------|
| **Self-Generated Agent Topology** | AI creates vertical (sequential) and horizontal (parallel ensemble) agent structures at runtime |
| **Dynamic Tool Synthesis** | Agents write and register new Python tools on the fly |
| **Hierarchical Graph-Based Memory** | Short-term + long-term memory stored as queryable directed graphs |

## Architecture

```
opensage/
├── core/
│   ├── agent.py        # SageAgent: main agentic loop with tool use
│   └── engine.py       # OpenSage: top-level ADK entry point
├── llm/
│   ├── base.py         # Abstract LLMClient interface
│   └── claude.py       # Anthropic Claude implementation
├── memory/
│   ├── graph.py        # GraphMemory: directed graph store
│   ├── hierarchical.py # HierarchicalMemory: task-structured API
│   └── memory_agent.py # MemoryAgent: background maintenance
├── tools/
│   ├── base.py         # Tool and ToolResult dataclasses
│   ├── executor.py     # ExecutionEnvironment (subprocess/Docker)
│   ├── manager.py      # ToolManager: register, execute, AI-create
│   └── se_toolkit/     # Software engineering domain tools
└── topology/
    ├── vertical.py     # Sequential sub-agent decomposition
    └── horizontal.py   # Parallel ensemble with synthesis
```

## Quick Start

```python
from opensage import OpenSage

# Initialize the engine (requires ANTHROPIC_API_KEY env var)
engine = OpenSage(model="claude-opus-4-5", verbose=True)

# Solve a task — the agent decides its own topology
result = engine.solve("Fix the bug in fibonacci.py")
print(result)
```

### Forced Topologies

```python
# Vertical: decompose into sequential sub-tasks
result = engine.solve(task, topology="vertical")

# Horizontal: run parallel ensemble and synthesize
result = engine.solve(task, topology="horizontal")
```

### Direct Agent API

```python
from opensage import SageAgent
from opensage.tools.manager import ToolManager
from opensage.tools.se_toolkit import get_se_toolkit
from opensage.memory.hierarchical import HierarchicalMemory
from opensage.llm.claude import ClaudeClient

llm = ClaudeClient(model="claude-opus-4-5")
tool_manager = ToolManager()
tool_manager.register_many(get_se_toolkit())
memory = HierarchicalMemory(agent_id="my_agent")

agent = SageAgent(
    name="my_agent",
    system_prompt="You are a software engineer.",
    llm=llm,
    tool_manager=tool_manager,
    memory=memory,
    enable_topology=True,  # Enables create_sub_agent, create_tool, etc.
)

result = agent.run("Analyze and fix the bug in sorting.py")
```

## Features

### Self-Generating Agent Topology

- **`create_sub_agent`**: Dynamically create a specialized sub-agent at runtime
- **`invoke_sub_agent`**: Run a previously created sub-agent on a task
- **`use_ensemble`**: Run N agents in parallel with different strategies

### Dynamic Tool Synthesis

- **`create_tool`**: Write and register a new Python tool from AI-generated code
- Hierarchical file-system-based tool organization
- Tool-specific sandboxed execution via subprocesses

### Hierarchical Memory System

- **Short-term**: Graph of task → sub-task → observation chains
- **Long-term**: Persistent knowledge across tasks
- **Memory agent**: Background summarization, deduplication, pruning
- Keyword + importance-weighted retrieval

### SE Toolkit (Domain-Specific Tools)

| Tool | Description |
|------|-------------|
| `read_file` | Read files with optional line ranges |
| `write_file` | Write/overwrite files |
| `run_command` | Execute shell commands |
| `run_python` | Execute Python code in sandboxed subprocess |
| `search_code` | Regex search across source files |
| `list_files` | List files in directory |
| `apply_patch` | Apply unified diff patches |
| `run_tests` | Run pytest/unittest test suites |
| `analyze_code` | Syntax/structure analysis of Python code |

## Installation

```bash
pip install -r requirements.txt
```

Set your API key:
```bash
export ANTHROPIC_API_KEY=your_key_here
```

## Running Tests

```bash
pytest tests/ -v
```

Tests use mock LLM objects and do not require an API key.

## Paper Reference

> **OpenSage: Self-programming Agent Generation Engine**  
> Hongwei Li, Zhun Wang, Qinrun Dai, et al.  
> arXiv:2602.16891, February 2026  
> https://arxiv.org/pdf/2602.16891
