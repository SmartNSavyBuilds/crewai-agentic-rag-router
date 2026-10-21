# Agentic RAG: Router-Retriever System

Multi-agent system built with CrewAI that routes a user's question to either a PDF vector search or a live web search, then returns a grounded, source-cited answer.

## Project files

- `agentic_rag.ipynb` - the notebook implementing the full system
- `pdfs/` - domain-specific PDF(s) used by the PDF search tool

## Setup

1. Add 1+ domain-specific PDF(s) to `pdfs/`.
2. Set `OPENAI_API_KEY` and `TAVILY_API_KEY` in the environment (get a Tavily key at https://app.tavily.com).
3. Run `agentic_rag.ipynb` top to bottom.

## System architecture

- **Router Agent** - reads the incoming question and decides whether it should be answered from the static PDF or from a fresh web search. Outputs a routing decision plus a short justification.
- **Retriever Agent** - reads the router's decision, calls the matching tool (`PDFSearchTool` or `TavilySearchTool`), and produces the final answer grounded in the retrieved content, citing its source.
- **Coordination flow** - CrewAI's `Process.sequential` passes the router's task output as context into the retriever's task, so the retriever agent's decision is driven by the router rather than making an independent choice.

## Logic and responsibilities

On a PDF question, the router chooses `source=pdf`, and the retriever uses `search-a-pdf-tool` to find passages related to the question. On a time-sensitive web question, the router chooses `source=web`, and the retriever uses Tavily search to gather current information before producing an answer.

## Challenges and trade-offs

The sequential workflow makes the routing decision easy to trace: the retriever receives the router’s decision before selecting a search tool. The trade-off is added latency because the router and retriever run one after the other. Only the selected retrieval tool runs, so the system does not pay for both PDF and web searches on every question. A question that could be answered from either source may still be routed incorrectly; clearer routing instructions or an ambiguity rule could improve consistency.

Running PDF and web searches in parallel might reduce retrieval time, but would use both tools unnecessarily and would not follow the current router-first flow. The web demo was cancelled during execution, so I do not yet have a successful run to compare its latency or results with the PDF path.
