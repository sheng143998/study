# LangChain 与 LangGraph

> 来源：由 `AI-Agent应用开发面试八股.md` 拆分整理。建议配合 `00-阅读索引.md` 使用。

## 3. LangChain 高频八股

### 3.1 LangChain 是什么

LangChain 是一个用于开发 LLM 应用的框架，提供模型调用、Prompt 模板、输出解析、Retriever、Tool、Agent、Chain、Callback、Tracing 等组件。它的价值是把大模型应用常见模块标准化，便于快速搭建 RAG、Agent、结构化输出、工具调用等系统。

面试回答要点：

- LangChain 不是模型本身。
- 它是应用层框架。
- 适合快速集成多个模型、检索器、工具、记忆和链路。
- 生产系统中需要关注可控性、性能、依赖复杂度，不应盲目套框架。

### 3.2 LangChain 的核心组件

| 组件 | 作用 |
|---|---|
| ChatModel / LLM | 模型调用封装 |
| PromptTemplate / ChatPromptTemplate | Prompt 模板化 |
| OutputParser | 解析模型输出 |
| Retriever | 检索相关文档 |
| Document Loader | 加载 PDF、网页、Markdown、数据库等数据 |
| Text Splitter | 文档切分 |
| Embeddings | 文本向量化 |
| VectorStore | 向量存储与检索 |
| Tool | 可被 Agent 调用的外部能力 |
| Agent | 根据模型决策选择工具 |
| Callback / Tracing | 链路观测 |
| Runnable / LCEL | 可组合执行单元 |

### 3.3 LCEL 是什么

LCEL 是 LangChain Expression Language，用于把 Prompt、Model、Parser、Retriever 等组件用管道方式组合成 Runnable。

典型形式：

```python
chain = prompt | model | parser
```

优点：

- 组合清晰。
- 支持流式、批处理、异步。
- 方便插入并行步骤、映射、fallback。
- 便于观测和复用。

常见追问：

- LCEL 和传统 Chain 有什么区别？
  - LCEL 更偏组合式、声明式。
  - 传统 Chain 往往封装较重。
  - 新版本 LangChain 更推荐 Runnable 体系。

### 3.4 LangChain Agent 的基本原理

LangChain Agent 通常包含：

- LLM：决定下一步。
- Tools：可调用工具列表。
- Prompt：描述任务、工具和输出格式。
- AgentExecutor：执行模型决策与工具调用循环。
- Memory/State：保存上下文。

基本流程：

```text
用户输入
  -> Agent prompt
  -> LLM 判断是否调用工具
  -> 调用工具
  -> 把观察结果放回上下文
  -> LLM 继续决策或输出最终答案
```

### 3.5 LangChain 的 Tool 怎么定义

工具一般需要：

- name：工具名，模型用来选择。
- description：工具描述，影响模型是否会调用。
- args_schema：参数 schema，约束输入。
- function：实际执行逻辑。

面试重点：

- 工具描述必须准确，避免互相重叠。
- 参数 schema 要尽量结构化。
- 工具需要超时、重试、异常处理。
- 工具调用要记录日志。
- 高风险工具要鉴权或人工确认。

### 3.6 LangChain 适合什么，不适合什么

适合：

- 快速验证 LLM 应用。
- 组合 RAG、工具调用、模型、输出解析。
- 多模型、多向量库、多数据源集成。
- 原型到中小规模生产。

不适合或要谨慎：

- 极致性能要求。
- 链路必须完全可控。
- 团队希望最小依赖。
- 框架抽象影响排查问题。
- Agent 流程很复杂时，应该考虑 LangGraph 或自研状态机。

### 3.7 LangChain 面试高频题

Q：LangChain 里的 Chain 和 Agent 区别？

A：Chain 是预定义流程，步骤相对固定；Agent 是由模型根据输入和中间结果动态决定下一步，包括是否调用工具、调用哪个工具、何时结束。Chain 更稳定可控，Agent 更灵活但不确定性更高。

Q：Retriever 和 VectorStore 的关系？

A：VectorStore 是存储和相似度查询的后端，例如 Milvus、Pinecone、Chroma、FAISS；Retriever 是面向应用层的检索接口，可以封装向量检索、关键词检索、混合检索、过滤、rerank 等逻辑。

Q：OutputParser 有什么用？

A：把模型输出解析成结构化结果，例如 JSON、Pydantic 对象、列表、枚举。它能降低下游处理复杂度，但仍需要错误修复和重试，因为模型可能输出非法格式。

Q：Callback/Tracing 有什么用？

A：用于记录每一步调用，包括 prompt、模型响应、token、延迟、工具调用、异常。生产中用于排查问题、成本统计、评测和回放。

Q：LangChain 的 Memory 生产中怎么做？

A：不能只依赖框架内存对象。生产中通常把短期上下文放请求态或缓存，把长期记忆放数据库/向量库，并做摘要压缩、权限隔离、过期策略和用户可删除能力。

---


## 4. LangGraph 高频八股

### 4.1 LangGraph 是什么

LangGraph 是 LangChain 生态中用于构建有状态、多步骤、可控制 Agent 工作流的框架。它把 Agent 流程表示为图：节点代表计算步骤，边代表控制流，状态在节点之间传递。

适合场景：

- 多轮、多步骤 Agent。
- 需要分支、循环、人工确认。
- 需要持久化状态和恢复。
- 需要更强可控性，而不是完全自由的 AgentExecutor。

### 4.2 LangGraph 为什么比普通 Agent 更适合生产

普通 Agent 的问题：

- 循环逻辑隐式在 AgentExecutor 里。
- 终止条件不直观。
- 状态和分支难管理。
- 出错恢复困难。

LangGraph 的优势：

- 流程显式图结构。
- 状态 schema 明确。
- 支持条件边。
- 支持 checkpoint。
- 支持 human-in-the-loop。
- 支持中断、恢复、回放。
- 更容易做测试和观测。

### 4.3 LangGraph 核心概念

| 概念 | 说明 |
|---|---|
| State | 图运行时共享状态 |
| Node | 一个处理步骤，例如调用模型、工具、检索器 |
| Edge | 节点之间的执行顺序 |
| Conditional Edge | 根据状态动态路由 |
| START / END | 图的开始和结束 |
| Checkpointer | 保存状态快照，支持恢复 |
| Interrupt | 暂停执行，等待人工输入 |
| Command | 控制图跳转和状态更新 |

### 4.4 LangGraph 的状态怎么设计

状态一般设计为 TypedDict / Pydantic 模型：

```python
from typing import TypedDict, Annotated

class AgentState(TypedDict):
    messages: list
    task_id: str
    retrieved_docs: list
    tool_results: dict
    final_answer: str | None
    error_count: int
```

设计原则：

- 状态字段要服务于流程控制。
- 不要把所有大对象都塞进状态。
- 大文档、文件、工具结果可以存外部存储，只在状态中存引用。
- 对 messages 做压缩或裁剪。
- 高风险动作需要记录确认状态。

### 4.5 LangGraph 的节点怎么设计

常见节点：

- intent_router：意图识别。
- retrieve：知识检索。
- rerank：重排序。
- generate_answer：生成答案。
- call_tool：工具调用。
- validate_output：校验输出。
- human_review：人工确认。
- summarize_memory：上下文压缩。
- error_handler：异常处理。

节点设计原则：

- 单一职责。
- 输入输出明确。
- 可单元测试。
- 节点内部异常要转成状态。
- 不要让一个节点承担完整业务流程。

### 4.6 LangGraph 面试高频题

Q：LangGraph 和 LangChain AgentExecutor 的区别？

A：AgentExecutor 更像一个封装好的循环执行器，适合简单工具调用 Agent；LangGraph 用图显式表达状态、节点、边和条件路由，更适合复杂、多步骤、需要持久化和人工介入的生产 Agent。

Q：LangGraph 如何防止 Agent 死循环？

A：可以通过图结构限制循环路径，设置最大迭代次数、状态中的 error_count/step_count，条件边判断终止，工具调用超时，以及失败后进入 fallback 或 human_review 节点。

Q：LangGraph 中 checkpointer 有什么用？

A：保存图执行过程中的状态快照，支持长任务恢复、人工审批后继续执行、故障恢复、调试回放和多轮会话持久化。

Q：human-in-the-loop 怎么做？

A：在高风险节点前插入 interrupt/human_review 节点，把待确认动作、参数、风险提示写入状态，等待人工确认后再继续执行。常见于发邮件、下单、改数据库、转账、删文件等操作。

Q：多 Agent 协作如何用 LangGraph 表达？

A：可以把不同 Agent 作为节点，或者把每个 Agent 子图化，再由 supervisor/router 节点根据状态调度。关键是定义共享状态、路由条件、终止条件和冲突处理。

---


