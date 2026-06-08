# P0 标准回答：LangChain/LangGraph 与后端工程版

> 用法：配合 `27-P0P1P2题库扩容版.md` 使用。  
> 本版覆盖 P0 的 71-100 题：LangChain / LangGraph、FastAPI、SSE、asyncio、限流、缓存、trace、成本和稳定性。

## 1. LangChain / LangGraph：71-85

### 71. LangChain 是什么

30 秒版：

```text
LangChain 是一个大模型应用开发框架或组件库，封装了模型调用、Prompt、Retriever、VectorStore、Tool、Agent、OutputParser、Callback 和 Runnable 等模块，常用于快速搭建 RAG、工具调用和 Agent 应用。
```

追问关键词：

- 组件库，不是模型。
- 适合快速集成。
- 生产中不要黑盒依赖。

### 72. LangChain 的核心组件有哪些

30 秒版：

```text
核心组件包括 ChatModel/LLM、PromptTemplate、OutputParser、Document Loader、Text Splitter、Embedding、VectorStore、Retriever、Tool、Agent、Callback/Tracing，以及 Runnable/LCEL。
```

追问关键词：

- RAG：loader、splitter、embedding、vectorstore、retriever。
- Agent：tool、agent executor、memory/state。
- 观测：callback/tracing。

### 73. Chain 和 Agent 的区别是什么

30 秒版：

```text
Chain 是固定流程，步骤由开发者预先定义；Agent 是动态决策，模型根据输入和中间结果决定是否调用工具、调用哪个工具以及什么时候结束。Chain 更稳定，Agent 更灵活但更难控。
```

项目版：

```text
生产里我会优先用固定 workflow，只有确实需要动态工具选择或多步规划时才引入 Agent。
```

### 74. Retriever 和 VectorStore 区别是什么

30 秒版：

```text
VectorStore 是向量存储和相似度搜索后端，比如 Milvus、FAISS、pgvector；Retriever 是应用层检索接口，可以封装向量检索、BM25、混合检索、metadata filter 和 rerank 等逻辑。
```

追问关键词：

- VectorStore 偏存储。
- Retriever 偏检索策略。

### 75. LCEL/Runnable 是什么

30 秒版：

```text
LCEL 是 LangChain Expression Language，Runnable 是可组合执行单元。它可以把 prompt、model、parser、retriever 等用管道组合，例如 `prompt | model | parser`，并支持 async、stream、batch 和并行组合。
```

追问关键词：

- 组合清晰。
- 支持流式。
- 比传统 Chain 更灵活。

### 76. OutputParser 有什么用

30 秒版：

```text
OutputParser 用来把模型输出解析成结构化结果，比如 JSON、列表、Pydantic 对象或枚举。它能降低下游处理复杂度，但仍要配合 schema 校验、重试和 fallback，因为模型可能输出非法格式。
```

项目版：

```text
工具调用和工单生成场景里，我会用 structured output 或 parser 把输出转成后端可校验对象。
```

### 77. LangChain Memory 生产中有什么坑

30 秒版：

```text
框架内置 Memory 适合 Demo，但生产中不能只把历史对话堆进上下文。要做会话持久化、摘要压缩、结构化状态、权限隔离、过期策略和用户可删除能力。
```

追问关键词：

- 上下文窗口有限。
- 敏感信息。
- 长期记忆要可控。

### 78. LangGraph 是什么

30 秒版：

```text
LangGraph 是 LangChain 生态里用于构建有状态 Agent 工作流的框架。它把流程表示成图，State 保存状态，Node 表示处理步骤，Edge 表示控制流，适合复杂、多步骤、需要恢复和人工介入的生产 Agent。
```

追问关键词：

- StateGraph。
- 有状态。
- workflow-constrained agent。

### 79. LangGraph 和 AgentExecutor 有什么区别

30 秒版：

```text
AgentExecutor 更像封装好的工具调用循环，适合简单 Agent；LangGraph 把状态、节点、边、条件分支、循环和终止显式表达出来，更适合复杂生产流程，比如人工确认、失败恢复和多 Agent 协作。
```

项目版：

```text
如果只是查天气，用 AgentExecutor 就够；如果有 RAG、工具、确认、回滚和审计，我会用 LangGraph 或自研状态机。
```

### 80. LangGraph 的 State、Node、Edge 是什么

30 秒版：

```text
State 是图运行时共享状态，比如 messages、intent、retrieved_docs、tool_results、step_count；Node 是一个处理步骤，比如检索、调用工具、校验；Edge 是节点之间的流转关系，Conditional Edge 可以根据状态动态路由。
```

追问关键词：

- State 要结构化。
- Node 单一职责。
- Edge 控制流程。

### 81. Conditional Edge 怎么用

30 秒版：

```text
Conditional Edge 用来根据当前状态决定下一步。例如 intent 是知识问答就去 retrieve，intent 是订单查询就去 call_tool，工具失败次数过多就去 fallback，高风险操作就去 human_review。
```

项目版：

```text
客服 Agent 中我会用条件边区分 RAG、只读工具、写操作确认和转人工。
```

### 82. checkpoint 有什么用

30 秒版：

```text
checkpoint 用来保存图执行过程中的状态快照，支持长任务恢复、人工审批后继续执行、故障恢复、调试回放和多轮会话持久化。
```

追问关键词：

- long-running task。
- human-in-the-loop。
- replay/debug。

### 83. human-in-the-loop 怎么实现

30 秒版：

```text
在高风险节点前插入人工确认节点，把待执行动作、参数、风险说明写入状态并中断流程。用户或人工审核确认后，更新 approval_status，再从 checkpoint 恢复继续执行。
```

项目版：

```text
退款、删数据、发邮件、改配置这类工具都应该进入 human_review。
```

### 84. LangGraph 如何防死循环

30 秒版：

```text
通过显式图结构限制循环路径，并在 state 里维护 step_count、error_count、last_tool_calls。条件边根据最大步数、重复调用、错误次数决定继续、结束、fallback 或转人工。
```

追问关键词：

- max steps。
- timeout。
- token budget。
- fallback。

### 85. 多 Agent 如何用 LangGraph 表达

30 秒版：

```text
可以把不同 Agent 设计成图中的节点或子图，由 supervisor/router 节点根据状态调度。关键是定义共享状态、每个 Agent 的输入输出 schema、终止条件和冲突处理，而不是让多个 Agent 自由聊天。
```

项目版：

```text
比如 Researcher、Tool Executor、Reviewer、Writer 可以是不同节点，最终由 supervisor 汇总和判断是否结束。
```

## 2. 后端工程：86-100

### 86. FastAPI 为什么适合大模型应用

30 秒版：

```text
FastAPI 支持 async、高性能、类型注解、Pydantic 校验和自动 OpenAPI 文档，适合封装模型调用、RAG 检索、工具服务和 SSE 流式接口。
```

追问关键词：

- async IO。
- Pydantic。
- OpenAPI。
- StreamingResponse。

### 87. SSE 流式输出怎么实现

30 秒版：

```text
后端返回 `text/event-stream`，用 async generator 接收模型流式 token/chunk，并按 `data: ...\n\n` 格式持续 yield。结束时发送 done 事件，异常时发送 error 事件，同时处理客户端断连。
```

工程注意：

- 代理层缓冲。
- 超时。
- trace。
- token 统计。

### 88. SSE 和 WebSocket 怎么选

30 秒版：

```text
如果只是服务端向客户端单向推送模型 token，SSE 更简单，基于 HTTP，断线重连也方便。如果需要双向实时通信，比如多人协作或实时控制，WebSocket 更合适。
```

追问关键词：

- SSE 单向。
- WebSocket 双向。
- LLM chat streaming 通常 SSE 足够。

### 89. asyncio 适合什么场景

30 秒版：

```text
asyncio 适合 IO 密集型并发，比如调用模型 API、向量库、数据库、Rerank 服务和外部工具。它在等待 IO 时让出事件循环，提高吞吐，但不适合 CPU 密集计算。
```

项目版：

```text
RAG 在线链路里，向量检索、BM25、部分只读工具可以异步并行。
```

### 90. GIL 是什么

30 秒版：

```text
GIL 是 CPython 的全局解释器锁，使同一进程中通常只有一个线程执行 Python 字节码。它限制 CPU 密集型多线程性能，但 IO 密集型任务仍能通过线程或 asyncio 提升吞吐。
```

追问关键词：

- CPU 密集用 multiprocessing/独立服务。
- LLM API 调用多是 IO 密集。

### 91. CPU 密集任务和 IO 密集任务怎么处理

30 秒版：

```text
IO 密集任务用 asyncio、线程池和连接池，比如模型 API、数据库和向量库调用。CPU 密集任务用进程池、任务队列、独立推理服务或 GPU batch，比如本地 embedding、大规模解析和 rerank 推理。
```

工程注意：

- 不要阻塞事件循环。
- 重任务异步化。

### 92. 模型调用超时怎么办

30 秒版：

```text
要设置请求超时、重试和降级策略。读请求可以有限重试，写工具不能简单重试。模型超时后可以降级到小模型、返回部分结果、转人工或提示稍后重试，并记录 trace。
```

追问关键词：

- timeout。
- retry with backoff。
- fallback。
- idempotency for writes。

### 93. 如何做限流

30 秒版：

```text
按用户、租户、接口、模型和工具做多维限流。常见算法有固定窗口、滑动窗口、令牌桶。大模型应用还要按 token quota 和并发数限制，避免成本失控和上游被打满。
```

项目版：

```text
企业多租户场景下，不能只做全局限流，要防止单个租户抢占资源。
```

### 94. 如何做缓存

30 秒版：

```text
可缓存 embedding、query rewrite、检索结果、rerank 结果和热点问答。缓存要设置 TTL，并根据数据更新、权限版本、prompt 版本和模型版本失效。
```

追问关键词：

- 缓存命中率。
- 热点 query。
- 失效策略。

### 95. 缓存 key 如何避免权限泄漏

30 秒版：

```text
缓存 key 不能只用用户 query，必须包含 tenant_id、user_id 或权限版本、retrieval_strategy_version、prompt_version、model_name 等信息。否则不同权限用户可能命中同一答案，造成数据泄漏。
```

工程注意：

- RAG answer cache 风险高。
- 可以优先缓存无权限敏感的中间结果。

### 96. 如何做降级

30 秒版：

```text
降级可以分层做：大模型不可用时切小模型或备用模型，rerank 慢时跳过 rerank，工具超时时返回可解释错误，RAG 失败时走关键词搜索或人工兜底，高峰期限制长任务。
```

追问关键词：

- graceful degradation。
- fallback。
- circuit breaker。

### 97. 如何做 trace

30 秒版：

```text
每次请求记录 request_id、user_id、session_id、prompt 版本、模型、token、延迟、query rewrite、召回 chunk、rerank 分数、tool call、错误和最终答案。trace 用于排查、评估、成本统计和 badcase 回放。
```

安全注意：

- 敏感字段脱敏。
- 按租户隔离。

### 98. token 成本怎么统计和优化

30 秒版：

```text
统计每次请求的 input token、output token、模型单价、工具调用次数和总成本。优化上可以做模型路由、上下文压缩、减少 top_k、缓存、Prompt 精简、限制最大输出和 Agent step budget。
```

项目版：

```text
简单问题走小模型，复杂任务再走大模型，是最常见的成本优化。
```

### 99. 如何做灰度发布

30 秒版：

```text
灰度可以按用户、租户、流量比例或白名单发布。大模型应用要灰度 Prompt、模型版本、检索策略、rerank 模型和工具 schema，并监控效果、延迟、成本、错误率和用户反馈，支持快速回滚。
```

追问关键词：

- A/B。
- prompt version。
- model version。
- rollback。

### 100. 如何处理上游模型不稳定

30 秒版：

```text
要做超时、重试、熔断、备用模型、模型路由、错误码归一化和降级。关键业务不能强依赖单一模型供应商。还要监控模型延迟、失败率、输出格式错误率和成本波动。
```

工程注意：

- fallback model。
- circuit breaker。
- structured output 校验。

## 3. 口述训练方式

建议这样练 71-100：

```text
LangChain/LangGraph：
  先讲它是什么，再讲为什么生产中需要显式状态和工作流。

后端工程：
  先讲实现方法，再补线上治理：超时、限流、缓存、降级、trace、权限。
```

最重要的收束句：

```text
大模型应用本质上还是一个后端系统，只是多了模型的不确定性、token 成本和工具调用风险，所以工程治理比单纯调 API 更重要。
```

