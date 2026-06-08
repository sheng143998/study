# Agent 基础与生产治理

> 来源：由 `AI-Agent应用开发面试八股.md` 拆分整理。建议配合 `00-阅读索引.md` 使用。

## 2. AI Agent 基础八股

### 2.1 什么是 Agent

标准回答：

Agent 是一种以目标为导向的软件系统，它使用大模型进行推理、规划和决策，并通过工具调用、记忆、环境反馈来完成多步骤任务。与普通 Chatbot 相比，Agent 不只是回答问题，还能根据状态选择动作、调用外部系统、观察结果并继续迭代。

面试可展开：

- LLM：负责理解、推理、规划、生成。
- Tools：外部能力，例如搜索、数据库、业务 API、代码执行。
- Memory/State：保存对话历史、任务进度、中间结果。
- Planner：把目标拆解成步骤。
- Executor：执行工具调用或子任务。
- Observer：读取工具结果和环境反馈。
- Guardrails：权限、安全、格式、预算、循环控制。

### 2.2 Agent 和普通 LLM 调用有什么区别

普通 LLM 调用：

- 输入 prompt，输出文本。
- 一次性或少量多轮。
- 不主动访问外部系统。
- 状态主要由上下文窗口承载。

Agent：

- 以任务目标为中心。
- 能动态选择工具。
- 能多步执行和反思。
- 需要显式状态管理。
- 需要错误恢复和终止条件。
- 更强调可观测性和安全边界。

### 2.3 Agent 的典型循环是什么

经典循环：

```text
Receive goal
  -> Plan
  -> Select action/tool
  -> Execute tool
  -> Observe result
  -> Update state
  -> Decide continue or finish
```

ReAct 模式：

```text
Thought -> Action -> Observation -> Thought -> Action -> Observation -> Final Answer
```

工程化回答：

现在很多生产系统不会让模型无限自由 ReAct，而是用状态机/工作流约束 Agent，例如 LangGraph 中的节点、边、条件分支、checkpoint、human-in-the-loop。

### 2.4 Agent 常见架构模式

1. 单 Agent + 工具集
   - 简单客服、知识库问答、SQL 查询。

2. Planner-Executor
   - Planner 负责拆任务，Executor 负责执行。
   - 适合长任务、复杂任务。

3. Router Agent
   - 先判断意图，再路由到 RAG、工具、人工客服、普通问答。

4. Multi-Agent
   - 多个专职 Agent 协作，例如 Researcher、Coder、Reviewer。
   - 难点是协调、收敛、成本和稳定性。

5. Workflow-constrained Agent
   - 用图或状态机限制流程。
   - 更适合生产。

### 2.5 Agent 为什么容易不稳定

原因：

- 模型输出具有概率性。
- 工具描述不清导致误调用。
- 工具返回噪声大。
- 多步任务误差累积。
- 上下文过长导致遗忘或注意力漂移。
- 缺少终止条件导致循环。
- 权限和数据边界没有硬约束。

治理手段：

- 工具 schema 严格化。
- 使用 structured output。
- 设置最大步数、超时、token budget。
- 对工具做权限、幂等、审计。
- 将自由 Agent 改成受控工作流。
- 加评测集和线上 trace。
- 对高风险动作加人工确认。

### 2.6 Agent 项目最常被追问什么

高频追问：

- 为什么要用 Agent，不用固定流程行不行？
- Agent 的状态放在哪里？
- 怎么防止工具乱调？
- 怎么防止死循环？
- 怎么处理工具调用失败？
- 怎么评估 Agent 是否完成任务？
- 怎么做多轮上下文压缩？
- 怎么控制成本？
- 怎么接入权限系统？
- 如果模型输出 JSON 格式错了怎么办？
- Agent 线上链路怎么 trace？

---


## 16. 高频追问速查

### 16.1 为什么不用纯 Prompt

因为纯 Prompt 只能影响模型行为，不能提供实时/私有知识，不能真正执行外部动作，也不能作为安全边界。企业级应用需要 RAG、工具调用、权限系统和观测系统配合。

### 16.2 为什么不用纯微调

因为企业知识频繁变化，微调成本高、更新慢、不可引用溯源。微调更适合稳定任务模式和输出风格，知识接入优先 RAG。

### 16.3 为什么不用纯 RAG

RAG 解决知识问题，但不能执行动作。需要查询实时订单、发邮件、改配置、创建工单时，需要 Tool Calling 或 Agent。

### 16.4 Agent 为什么要加状态机

自由 Agent 灵活但不可控。状态机/图工作流能显式约束流程、分支、循环和终止条件，便于测试、恢复和审计。

### 16.5 线上如何降延迟

- embedding 缓存。
- query rewrite 缓存。
- 检索和 rerank 并行优化。
- 减少 top_k。
- 使用轻量 reranker。
- prompt 压缩。
- 模型流式输出。
- 工具调用异步并行。
- 热点问题缓存。
- 模型分级路由。

### 16.6 线上如何降成本

- 小模型处理简单任务。
- 大模型只处理复杂任务。
- 缓存。
- Prompt 压缩。
- 控制上下文长度。
- 批量 embedding。
- 减少无效工具调用。
- 离线预计算。
- 对长任务设置 token budget。

### 16.7 如何做可观测性

记录：

- request_id / user_id / session_id。
- prompt 版本。
- 模型名称。
- token 输入输出。
- 延迟。
- 检索 query。
- 召回 chunk id。
- rerank 分数。
- tool call 参数和结果摘要。
- 错误栈。
- 最终答案。
- 用户反馈。

注意：

- 敏感信息脱敏。
- 日志按租户隔离。
- 支持 trace 回放。

### 16.8 如何做灰度

- 按用户/租户/流量比例灰度。
- Prompt 版本灰度。
- 模型版本灰度。
- 检索策略灰度。
- A/B 测试。
- 指标监控。
- 支持快速回滚。

### 16.9 如何处理模型输出不稳定

- structured output。
- JSON schema。
- Pydantic 校验。
- 输出解析失败重试。
- 降低 temperature。
- Few-shot 示例。
- 缩小任务范围。
- 后处理校验。

### 16.10 如何处理长上下文

- 检索式上下文选择。
- 摘要压缩。
- 滑动窗口。
- 层级记忆。
- 重要信息结构化存储。
- 不把无关历史全塞给模型。

### 16.11 Memory Governance 怎么做

长期记忆不是“用户说过什么都存”。生产里我会把记忆分成短期上下文、结构化任务状态和长期用户偏好三类，并给长期记忆加写入策略。

可写：

- 用户明确确认的偏好。
- 非敏感、低风险、对后续体验有帮助的信息。
- 有来源、时间和版本的事实。

不可写：

- 密码、token、身份证、银行卡、病历等敏感信息。
- 未确认的模型猜测。
- 工具返回里的隐私字段。
- 已过期或用户要求删除的信息。

治理字段：

```text
memory_id
user_id / tenant_id
source
content
category
sensitivity
ttl
created_at
updated_at
delete_status
confidence
```

面试口播：

```text
Memory 的核心不是存得越多越好，而是可控、可删、可审计。写入前先判断来源、敏感级别、用户是否确认和 TTL；读取时按租户、权限和任务相关性过滤；用户要求删除时要从主存储、向量索引、缓存和评测样本里同步处理，并保留脱敏审计记录。
```

---


