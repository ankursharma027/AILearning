---
title: "LangGraph: Handling Memory in AI Agents"
date: 2026-04-19
categories:
  - AI
tags:
  - langgraph
  - langchain
  - agents
  - memory
  - llm
excerpt: "A deep dive into short-term and long-term memory patterns in LangGraph — from in-memory state to persistent checkpointers."
---

## Types of Memory in LangGraph

LangGraph distinguishes between two kinds of memory:

| Type | Scope | Persists? |
|---|---|---|
| **Short-term (In-graph state)** | Within one graph run | No |
| **Long-term (Checkpointer)** | Across multiple runs / conversations | Yes |

---

## Short-Term Memory — Graph State

The simplest form of memory is the **state object** passed between nodes. Every node reads from and writes to it.

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, END
import operator

class AgentState(TypedDict):
    messages: Annotated[list, operator.add]  # append-only list
    summary: str
    turn_count: int
```

Using `Annotated[list, operator.add]` tells LangGraph to **append** new messages rather than replace them — key for conversation history.

---

## Long-Term Memory — Checkpointers

Checkpointers save the graph state to storage so it can be resumed later, enabling multi-turn conversations.

### In-Memory Checkpointer (dev/testing)

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import StateGraph, END

memory = MemorySaver()

graph = (
    StateGraph(AgentState)
    .add_node("agent", agent_node)
    .add_edge("agent", END)
    .compile(checkpointer=memory)
)
```

Invoke with a `thread_id` to maintain separate conversation threads:

```python
config = {"configurable": {"thread_id": "user-123"}}

# First turn
graph.invoke({"messages": ["Hello!"]}, config=config)

# Second turn — remembers the first
graph.invoke({"messages": ["What did I just say?"]}, config=config)
```

### SQLite Checkpointer (local persistence)

```python
from langgraph.checkpoint.sqlite import SqliteSaver

with SqliteSaver.from_conn_string("memory.db") as checkpointer:
    graph = builder.compile(checkpointer=checkpointer)
    graph.invoke({"messages": ["Hi"]}, config={"configurable": {"thread_id": "abc"}})
```

### PostgreSQL Checkpointer (production)

```python
from langgraph.checkpoint.postgres import PostgresSaver

DB_URI = "postgresql://user:password@localhost:5432/mydb"

with PostgresSaver.from_conn_string(DB_URI) as checkpointer:
    checkpointer.setup()  # creates tables on first run
    graph = builder.compile(checkpointer=checkpointer)
```

---

## Conversation Summarization

For long conversations, the message list grows too large for the LLM context window. Summarize older messages to compress history.

```python
from langchain_anthropic import ChatAnthropic
from langgraph.graph import StateGraph, END
from typing import TypedDict, Annotated
import operator

llm = ChatAnthropic(model="claude-sonnet-4-6")

class State(TypedDict):
    messages: Annotated[list, operator.add]
    summary: str

def should_summarize(state: State) -> str:
    if len(state["messages"]) > 10:
        return "summarize"
    return "chat"

def summarize_node(state: State) -> State:
    existing = state.get("summary", "")
    prompt = f"Existing summary: {existing}\n\nNew messages:\n"
    prompt += "\n".join(str(m) for m in state["messages"][-6:])
    prompt += "\n\nWrite an updated summary."

    summary = llm.invoke(prompt).content

    # Keep only the last 2 messages, drop the rest
    return {
        "summary": summary,
        "messages": state["messages"][-2:]
    }

builder = StateGraph(State)
builder.add_node("chat", chat_node)
builder.add_node("summarize", summarize_node)
builder.set_entry_point("chat")
builder.add_conditional_edges("chat", should_summarize, {
    "summarize": "summarize",
    "chat": END
})
builder.add_edge("summarize", END)
```

---

## Accessing & Inspecting State

Read the saved state for a thread at any point:

```python
# Get current state
state = graph.get_state(config)
print(state.values)          # state dict
print(state.next)            # next node(s) to run

# Get full history
for snapshot in graph.get_state_history(config):
    print(snapshot.values["messages"])
```

Rewind to a past checkpoint:

```python
# Grab a past snapshot
history = list(graph.get_state_history(config))
past_config = history[2].config   # 3rd checkpoint back

# Resume from that point
graph.invoke(None, config=past_config)
```

---

## External / Semantic Memory (Long-Term Knowledge)

For facts that should persist beyond a single conversation, store them in a vector database and retrieve them as context.

```python
from langchain_community.vectorstores import Chroma
from langchain_anthropic import AnthropicEmbeddings

vectorstore = Chroma(embedding_function=AnthropicEmbeddings())

def memory_node(state: State) -> State:
    query = state["messages"][-1]
    docs = vectorstore.similarity_search(query, k=3)
    context = "\n".join(d.page_content for d in docs)
    return {"messages": [f"[Context from memory]: {context}"]}
```

---

## Quick Reference

| Use Case | Solution |
|---|---|
| Pass data between nodes | `StateGraph` with typed state |
| Multi-turn conversation | `MemorySaver` + `thread_id` |
| Persist across restarts (local) | `SqliteSaver` |
| Persist across restarts (prod) | `PostgresSaver` |
| Long conversation compression | Summarization node |
| Domain knowledge retrieval | Vector store + RAG node |
| Time-travel / debugging | `get_state_history()` |

---

## Resources

- [LangGraph Persistence Docs](https://langchain-ai.github.io/langgraph/concepts/persistence/)
- [LangGraph How-to: Memory](https://langchain-ai.github.io/langgraph/how-tos/memory/manage-conversation-history/)
- [LangGraph Checkpointers Reference](https://langchain-ai.github.io/langgraph/reference/checkpoints/)
