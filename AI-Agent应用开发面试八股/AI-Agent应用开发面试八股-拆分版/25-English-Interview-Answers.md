# English Interview Answers for AI Agent Application Development

> Usage: Use this file for English technical interviews, overseas teams, or English resume project discussions. Each answer is written in practical interview English, followed by short Chinese notes.

## 1. Self Introduction

### 60-second version

```text
Hi, my main focus is AI application development, especially RAG systems, AI agents, tool calling, and backend engineering for LLM-powered products.

I have worked on an enterprise knowledge assistant project that integrates document parsing, vector retrieval, RAG-based question answering, tool calling, permission control, and observability.

Technically, I am familiar with Python, FastAPI, LangChain, LangGraph, vector databases, prompt engineering, tool calling, and integration protocols such as OpenAPI, RPC, and MCP.

What I care about is not only building a demo, but making the system reliable in production, including retrieval quality, hallucination control, tool safety, latency, cost, tracing, and evaluation.
```

中文提示：

- 重点突出方向：RAG、Agent、Tool Calling、后端。
- 不要只说会框架，要说生产治理。

## 2. What Is RAG?

### Short answer

```text
RAG stands for Retrieval-Augmented Generation. It retrieves relevant information from an external knowledge base and provides it as context to the language model, so the model can generate answers grounded in external evidence.

It is useful for private knowledge, frequently updated information, and scenarios where citation and traceability are required.
```

### Expanded answer

```text
A typical RAG system has two pipelines.

The offline pipeline includes document ingestion, parsing, cleaning, chunking, embedding, indexing, and metadata storage.

The online pipeline includes query rewriting, retrieval, metadata filtering, reranking, context construction, LLM generation, and citation validation.

In production, RAG quality depends heavily on document quality, chunking strategy, embedding model, hybrid retrieval, reranking, prompt design, and evaluation.
```

中文提示：

- 离线链路和在线链路一定要讲。
- 最后补生产因素，显得不是只背定义。

## 3. How Would You Improve Poor Retrieval Quality?

```text
I would not start by simply increasing top_k. I would first build a bad case set and identify whether the problem is missing recall, poor ranking, or the model not using the retrieved evidence.

Then I would check the pipeline step by step: document parsing quality, chunking strategy, metadata completeness, embedding model suitability, query rewriting, hybrid search, reranking, and permission filters.

For enterprise knowledge bases, I usually prefer hybrid retrieval, because vector search is good at semantic matching, while BM25 is better for exact terms such as error codes, product names, policy IDs, and function names.

Finally, I would evaluate the changes with metrics such as Recall@K, MRR, NDCG, citation accuracy, and human review.
```

中文提示：

- 不要只说 top_k。
- 一定要说 badcase、链路排查、hybrid、rerank、评估。

## 4. Why Can RAG Still Hallucinate?

```text
RAG reduces hallucination, but it does not eliminate it.

There are two main reasons. First, the retrieval stage may fail to retrieve the right evidence, or may return noisy and irrelevant chunks. Second, even if the evidence is retrieved, the model may not faithfully use it during generation.

To mitigate this, I would improve retrieval quality, use reranking, design prompts that require evidence-based answers, ask the model to say "I don't know" when there is no supporting context, validate citations, and use groundedness checks or human review for high-risk scenarios.
```

中文提示：

- RAG 不能消灭幻觉。
- 分检索阶段和生成阶段讲。

## 5. What Is an AI Agent?

```text
An AI agent is a goal-oriented LLM application that can reason, plan, call tools, read external context, maintain state, and update its next action based on observations.

Compared with a normal chatbot, an agent is not limited to generating text. It can interact with external systems and complete multi-step tasks.

In production, I would not let the agent act freely without constraints. I would add tool schemas, permission checks, maximum step limits, timeouts, idempotency, audit logs, fallback strategies, and human confirmation for high-risk actions.
```

中文提示：

- Agent = 目标驱动 + 工具 + 状态 + 反馈。
- 生产里强调约束。

## 6. How Do You Prevent an Agent from Running Forever?

```text
I would use multiple safeguards.

First, set a maximum number of steps and a token budget. Second, keep step_count and error_count in the agent state. Third, detect repeated tool calls with the same arguments. Fourth, use timeouts for tool execution. Fifth, route the workflow to fallback or human review when the agent cannot make progress.

If the workflow is complex, I would prefer a graph-based design such as LangGraph, where loops, conditional edges, and termination conditions are explicit.
```

中文提示：

- max steps 只是其中一层。
- 提 LangGraph 显式状态图。

## 7. What Is Tool Calling?

```text
Tool calling allows the model to choose an external function or API and generate structured arguments for it.

The model should not directly execute the tool. The application layer should validate the arguments, check permissions, execute the tool, handle errors, and return the observation back to the model.

For high-risk operations such as sending emails, deleting data, submitting orders, or issuing refunds, I would require user confirmation, idempotency keys, and audit logs.
```

中文提示：

- 模型只生成意图和参数。
- 后端负责执行、安全和审计。

## 8. How Would You Design Tool Schemas?

```text
I would make the tool name short and specific, write a clear description about when to use and when not to use the tool, and define a strict argument schema.

I prefer structured parameters, required fields, enums when possible, and stable return formats. For dangerous tools, I would add a risk level and require explicit confirmation before execution.

A vague tool description can easily cause wrong tool selection, so tool design is part of prompt engineering and system design.
```

中文提示：

- name、description、args_schema、return_schema。
- description 影响模型选工具。

## 9. LangChain vs LangGraph

```text
LangChain is more like a component library for LLM applications. It provides abstractions for models, prompts, retrievers, vector stores, tools, agents, output parsers, callbacks, and runnable chains.

LangGraph is designed for stateful and controllable agent workflows. It represents the workflow as a graph, where nodes are computation steps, edges define control flow, and state is passed between nodes.

For simple tool-calling agents, LangChain AgentExecutor may be enough. For complex production workflows with branching, loops, checkpoints, human-in-the-loop, and recovery, I would prefer LangGraph.
```

中文提示：

- LangChain 是组件库。
- LangGraph 是状态图工作流。

## 10. What Is MCP?

```text
MCP stands for Model Context Protocol. It is an open protocol that standardizes how AI applications connect to external tools, resources, and prompt templates.

It is not a replacement for OpenAPI. OpenAPI describes HTTP APIs, RPC is a remote procedure call abstraction, while MCP is designed for AI applications to discover and use tools and context in a standardized way.

An MCP server can wrap internal APIs, databases, file systems, SaaS tools, or knowledge bases and expose them as tools or resources to AI clients.
```

中文提示：

- MCP = AI 应用连接工具/资源/Prompt 的协议。
- OpenAPI/RPC/MCP 分层讲。

## 11. RAG vs Fine-tuning

```text
If the problem is missing or frequently updated knowledge, I would choose RAG first.

If the problem is unstable behavior, output format, style, or task pattern, and we have high-quality training data, then fine-tuning may be useful.

RAG keeps knowledge external and traceable, while fine-tuning changes the model behavior. In many production systems, they can be combined: RAG provides up-to-date knowledge, and fine-tuning improves instruction following or domain-specific response style.
```

中文提示：

- 缺知识 RAG。
- 行为/格式/风格微调。

## 12. How Would You Design an Enterprise Knowledge Assistant?

```text
I would design it with five layers.

The first layer is the API and access layer, including authentication, rate limiting, tenant identification, and streaming response.

The second layer is the offline ingestion pipeline, including document parsing, cleaning, chunking, embedding, indexing, and metadata management.

The third layer is the online retrieval pipeline, including query rewriting, hybrid retrieval, metadata filtering, reranking, context construction, and citation validation.

The fourth layer is the agent and tool layer, where read-only tools can be executed automatically, while high-risk write operations require confirmation, idempotency, and audit logs.

The fifth layer is observability and evaluation, including trace logs, token cost, latency, retrieval metrics, answer quality, user feedback, and bad case analysis.
```

中文提示：

- 五层架构，适合系统设计题。

## 13. How Do You Handle Permission Control in RAG?

```text
Permission control should happen before the model sees the context.

During document ingestion, each chunk should carry metadata such as tenant_id, department, role, ACL, document version, and source. During retrieval, the system should build a metadata filter based on the user identity and apply it to both vector search and keyword search.

Post-generation filtering is not enough, because once the model has seen unauthorized context, the data may already be leaked.

Caches and logs also need tenant and permission isolation.
```

中文提示：

- 权限必须在检索阶段。
- 缓存和日志也要隔离。

## 14. How Do You Reduce Latency and Cost?

```text
I would first use tracing to identify where the latency comes from: query rewriting, retrieval, reranking, tool calls, or model generation.

Then I would optimize layer by layer. For latency, I would use async calls, parallel retrieval, caching, smaller top_k, lightweight rerankers, streaming responses, and timeouts for slow tools.

For cost, I would use model routing, prompt compression, context deduplication, query caching, embedding caching, token budgets, and tenant-level quotas.

For simple tasks, I would route to smaller and cheaper models. Large models should be reserved for complex reasoning or high-value tasks.
```

中文提示：

- 先 trace 再优化。
- latency 和 cost 分开讲。

## 15. How Do You Talk About a Project That Was Not Fully Launched?

```text
I would be honest and avoid exaggerating production metrics.

I would say that the project completed an end-to-end prototype and offline validation, including document ingestion, retrieval, reranking, answer generation, citation validation, tool calling, and tracing.

For evaluation, I built a small test set with representative questions and expected documents, and measured retrieval quality, answer relevance, citation accuracy, and human feedback.

If the project were to be launched, the next steps would be permission hardening, online evaluation, gray release, cost monitoring, and human fallback.
```

中文提示：

- 没上线就诚实说。
- 强调端到端验证和下一步上线条件。

## 16. Useful English Phrases

### Engineering trade-offs

```text
There is a trade-off between recall quality, latency, and cost.
I would not rely only on prompts for safety.
This should be enforced at the application layer.
The model should propose the action, but the system should control execution.
I would start with a controlled workflow before allowing more autonomous behavior.
```

### Project explanation

```text
The main challenge was not calling the model, but making the system reliable.
We split the problem into retrieval quality, generation faithfulness, and tool safety.
We used traces and bad cases to drive iteration.
The system was designed to be observable, recoverable, and auditable.
```

### When you do not know

```text
I have not implemented this exact part in production, but I understand the design considerations.
If I were to design it, I would start with a minimal reliable workflow and then add evaluation and observability.
I would need to validate this with experiments and production traces.
```

