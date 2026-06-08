# P1 标准回答：Prompt、协议与微调版

> 用法：配合 `27-P0P1P2题库扩容版.md` 使用。  
> 本版覆盖 P1 的 136-180 题：Prompt / Structured Output、OpenAPI/RPC/MCP、微调。

## 1. Prompt / Structured Output：136-150

### 136. Prompt Engineering 是什么

30 秒版：

```text
Prompt Engineering 是通过设计模型输入来控制模型行为，包括角色、任务说明、上下文、示例、约束、输出格式和工具描述。它成本低、迭代快，但只是软约束，不能替代权限、安全和工具层校验。
```

追问关键词：

- system/developer/user。
- few-shot。
- structured output。
- Prompt 不是安全边界。

### 137. System、Developer、User Prompt 如何分工

30 秒版：

```text
System Prompt 定义最高层角色和安全边界，Developer Prompt 定义应用逻辑、格式、工具使用规则，User Prompt 是用户当前请求。生产中要把稳定规则放到高优先级指令里，把外部文档明确标记为上下文而不是指令。
```

工程注意：

- 外部检索内容不能当系统指令。
- Prompt 层级要清晰。

### 138. Few-shot 为什么有效

30 秒版：

```text
Few-shot 通过给模型示例，让模型模仿输入输出模式、风格和边界。它特别适合格式、分类、抽取和业务话术类任务。但示例过多会增加 token 成本，也可能让模型过拟合示例模式。
```

项目版：

```text
客服话术、JSON 输出、引用格式都可以用 few-shot 增强稳定性。
```

### 139. Chain-of-Thought 生产中怎么用

30 秒版：

```text
Chain-of-Thought 可以提升复杂推理，但生产中通常不直接暴露完整推理链。更常见做法是让模型内部推理，输出简洁依据、步骤摘要或最终结论，并避免泄露敏感推理过程。
```

工程注意：

- 不一定展示完整 CoT。
- 可要求 concise rationale。
- 高风险结论要证据校验。

### 140. Prompt injection 是什么

30 秒版：

```text
Prompt injection 是用户输入或外部文档试图覆盖系统指令，比如要求模型忽略规则、泄露密钥或调用危险工具。它常出现在用户消息、网页、文档和工具返回结果中。
```

追问关键词：

- direct injection。
- indirect injection。
- RAG 文档注入。

### 141. 如何防 prompt injection

30 秒版：

```text
不能只靠 Prompt 防。要把外部内容当数据而不是指令，工具层做权限和白名单，危险操作人工确认，检索结果隔离，输出过滤和日志审计。真正安全边界必须在代码和权限系统里。
```

项目版：

```text
RAG 文档中即使写着“忽略系统规则”，也只能作为文档内容，不应改变工具权限。
```

### 142. 如何让模型稳定输出 JSON

30 秒版：

```text
优先使用模型原生 structured output 或 tool/function calling；其次用 JSON schema、few-shot、低 temperature、输出解析、校验失败重试和 fallback。关键链路不能只靠自然语言要求“请输出 JSON”。
```

工程注意：

- schema validate。
- max retry。
- parser exception。

### 143. structured output 和 JSON repair 区别

30 秒版：

```text
structured output 是在模型生成阶段就约束输出结构，可靠性更高；JSON repair 是生成后发现格式错了再修复，是补救手段。生产中优先 structured output，repair 只能作为兜底。
```

追问关键词：

- generation-time constraint。
- post-processing。
- repair 可能引入语义变化。

### 144. temperature 对输出有什么影响

30 秒版：

```text
temperature 控制采样随机性。低 temperature 输出更稳定，适合结构化抽取、工具参数、RAG 问答；高 temperature 更发散，适合创意生成。生产中关键任务通常设低温，并配合 schema 校验。
```

工程注意：

- 不等于准确率开关。
- 不稳定问题还要靠约束和评测。

### 145. Prompt 如何版本化

30 秒版：

```text
把 Prompt 当配置和代码管理，每个 Prompt 有 name、version、适用场景、模型、变更记录和评测结果。trace 里记录 prompt_version，支持回放、灰度、A/B 和回滚。
```

项目版：

```text
RAG Prompt 改一行也可能影响拒答率和引用质量，所以必须版本化。
```

### 146. Prompt 如何做 A/B 测试

30 秒版：

```text
按用户、租户或流量比例分配不同 Prompt 版本，保持模型和检索策略尽量一致，比较答案质量、拒答率、引用准确率、用户反馈、延迟和 token 成本。需要支持快速回滚。
```

工程注意：

- 控制变量。
- 采样足够。
- badcase 对比。

### 147. 如何设计 RAG Prompt

30 秒版：

```text
RAG Prompt 要明确只能基于给定上下文回答，无证据就说不知道，保留引用格式，区分用户问题和检索文档。还要控制上下文长度，避免噪声 chunk 干扰模型。
```

常见结构：

```text
角色 -> 规则 -> 用户问题 -> Context -> 输出格式
```

### 148. 如何设计 Tool Calling Prompt

30 秒版：

```text
Tool Calling Prompt 要说明工具使用边界：什么时候该调用、什么时候不该调用、参数从哪里来、不要猜测缺失参数。高风险动作只能生成待确认操作，不能直接执行。
```

工程注意：

- 工具描述比总 Prompt 更关键。
- 工具权限仍在后端。

### 149. 如何处理模型拒答过多

30 秒版：

```text
先判断拒答是否合理。如果是检索不足导致，就优化召回和上下文；如果 Prompt 过严，就调整无答案判断条件；如果模型对风险过敏，可以给正反例。不能为了降低拒答率让模型无依据硬答。
```

指标：

- no-answer rate。
- answer relevance。
- hallucination rate。

### 150. 如何处理模型过度自信

30 秒版：

```text
要求模型基于证据回答、输出引用、无证据拒答，并对低置信检索结果触发拒答或人工。后处理可以做引用校验和 groundedness 检查，高风险场景用规则或人工复核。
```

工程注意：

- 不要让模型自己随便报置信度。
- 结合检索分数和证据强度。

## 2. 协议与平台：151-165

### 151. OpenAPI 是什么

30 秒版：

```text
OpenAPI 是描述 HTTP API 的规范，用机器可读格式定义路径、方法、参数、请求体、响应 schema 和认证方式。它可以用于生成文档、SDK、测试，也可以作为 Agent 工具定义的来源。
```

### 152. OpenAPI 和 Swagger 关系是什么

30 秒版：

```text
Swagger 最初是 API 描述规范和工具生态，后来规范部分演进为 OpenAPI Specification。现在 Swagger 更多指相关工具生态，OpenAPI 指标准规范。
```

### 153. OpenAPI 的核心字段有哪些

30 秒版：

```text
核心包括 openapi、info、servers、paths、parameters、requestBody、responses、components、schemas 和 security。Agent 工具生成时最关注 path、method、参数 schema、响应和认证。
```

### 154. REST 和 RPC 区别是什么

30 秒版：

```text
REST 面向资源，通过 HTTP 方法操作资源，比如 GET /users/1；RPC 面向过程或方法，比如 GetUser(id)。REST 常用于开放 API，RPC 常用于内部高性能服务通信。
```

追问关键词：

- resource vs method。
- HTTP/JSON vs gRPC/Thrift。

### 155. gRPC 有什么特点

30 秒版：

```text
gRPC 基于 HTTP/2 和 Protobuf，支持强类型 IDL、多语言代码生成、流式通信和较高性能，常用于微服务内部通信。缺点是浏览器直接调试和可读性不如 HTTP/JSON。
```

### 156. Protobuf 有什么优缺点

30 秒版：

```text
Protobuf 优点是体积小、序列化快、强 schema、多语言生成；缺点是可读性不如 JSON，调试门槛高，字段演进要遵守兼容规则。
```

### 157. JSON-RPC 是什么

30 秒版：

```text
JSON-RPC 是一种轻量级远程过程调用协议，用 JSON 表示 method、params、id 和 result/error。MCP 的消息层使用 JSON-RPC 风格交互。
```

### 158. MCP 是什么

30 秒版：

```text
MCP 是 Model Context Protocol，是 AI 应用连接外部工具、资源和 Prompt 的标准协议。它让不同 AI Host/Client 能用统一方式发现和调用 MCP Server 暴露的能力。
```

### 159. MCP 的 Host、Client、Server 是什么

30 秒版：

```text
Host 是承载 AI 应用的环境，比如 IDE、桌面客户端或 Web Agent；Client 是 Host 内部和 Server 通信的协议客户端；Server 负责暴露 tools、resources、prompts 等能力。
```

### 160. MCP tools/resources/prompts 区别

30 秒版：

```text
tools 是可执行动作，比如查数据库、创建工单；resources 是可读取上下文，比如文件、文档、数据；prompts 是可复用 Prompt 模板。三者分别对应动作、数据和模板。
```

### 161. MCP 和 OpenAPI 的关系是什么

30 秒版：

```text
OpenAPI 描述 HTTP API，MCP 面向 AI 应用标准化工具和上下文连接。MCP 不是 OpenAPI 的替代品，MCP Server 内部可以调用 OpenAPI 描述的 API，并包装成 MCP tools。
```

### 162. MCP 和 LangChain Tool 的区别是什么

30 秒版：

```text
LangChain Tool 是框架内部工具抽象，主要服务 LangChain 应用；MCP Tool 是协议层能力，可以被不同 MCP Host 使用。两者可以互相适配，把 MCP tool 接入 LangChain，或把 LangChain tool 包装成 MCP server。
```

### 163. MCP 安全怎么做

30 秒版：

```text
MCP 本身只是协议，不自动解决所有安全问题。要在 Host、Client、Server 和底层工具层做授权、工具白名单、roots 限制、参数校验、用户确认、审计和敏感信息脱敏。
```

### 164. 如何把内部 API 包装成 MCP server

30 秒版：

```text
先筛选适合暴露的 API，再定义 MCP tool 的 name、description、参数 schema 和返回结构。Server 内部调用原有 OpenAPI/RPC 服务，执行前做鉴权、限流、参数校验和审计，高风险写操作加确认。
```

### 165. 工具平台如何做多租户

30 秒版：

```text
工具平台要按 tenant_id 隔离工具白名单、权限、配额、日志、密钥和审计。运行时根据用户和租户加载可用工具，缓存和 trace 也要包含租户信息，避免工具和数据串租。
```

## 3. 微调：166-180

### 166. SFT 是什么

30 秒版：

```text
SFT 是 supervised fine-tuning，监督微调。它用高质量输入输出样本继续训练模型，让模型更符合特定任务、格式、风格或领域话术。
```

### 167. LoRA 是什么

30 秒版：

```text
LoRA 是低秩适配微调方法。它冻结原模型参数，只在部分线性层旁边训练低秩矩阵，显著降低训练显存和参数量，适合参数高效微调。
```

### 168. PEFT 为什么省资源

30 秒版：

```text
PEFT 只训练少量额外参数或部分参数，而不是全量更新模型，所以显存、训练时间和存储成本更低。LoRA 是最常见的 PEFT 方法之一。
```

### 169. DPO 是什么

30 秒版：

```text
DPO 是 Direct Preference Optimization，直接偏好优化。它用偏好数据对模型进行对齐，让模型更倾向于被偏好的回答，常作为比 RLHF 更简单的偏好优化方法。
```

### 170. RLHF 和 DPO 区别是什么

30 秒版：

```text
RLHF 通常包括奖励模型和强化学习优化流程，复杂度较高；DPO 直接用偏好对优化模型，不需要显式训练奖励模型和 RL 阶段，流程更简单稳定。
```

### 171. 微调数据怎么准备

30 秒版：

```text
高质量比数量更重要。数据要来自真实任务，格式一致，覆盖高频、长尾、边界和困难样本，去重、脱敏，划分训练/验证/测试集，并保留 baseline 做对比。
```

### 172. 如何防止微调过拟合

30 秒版：

```text
控制训练轮数和学习率，使用验证集早停，保证数据多样性，避免重复样本，保留独立测试集。还要比较泛化场景，防止模型只记住训练格式。
```

### 173. 微调后怎么评估

30 秒版：

```text
看任务指标、格式正确率、人工评分、和 baseline 的 A/B 对比，也要测安全、幻觉、拒答率、延迟和成本。不能只看训练 loss。
```

### 174. 微调能解决幻觉吗

30 秒版：

```text
不能完全解决。微调可以改善模型行为和格式，但事实正确性仍然需要 RAG、工具查询、引用校验和评估。把频繁变化知识写进微调不是好选择。
```

### 175. embedding 模型可以微调吗

30 秒版：

```text
可以。领域 query-doc 对或正负样本可以微调 embedding，提升特定领域召回。但要用检索指标验证，比如 Recall@K、MRR、NDCG，并注意不要损伤通用语义能力。
```

### 176. reranker 可以微调吗

30 秒版：

```text
可以。用 query、相关文档、不相关文档构造训练数据，微调 reranker 能提升排序质量，尤其适合领域术语和业务规则复杂的知识库。
```

### 177. 多个 LoRA adapter 怎么管理

30 秒版：

```text
按任务、领域或租户管理 adapter，记录 base model、adapter version、训练数据、评测结果和适用场景。推理时按任务路由加载对应 adapter，也可以在需要时合并权重。
```

### 178. 微调和 Prompt 优化怎么选

30 秒版：

```text
先 Prompt，因为成本低、迭代快；如果需求稳定、Prompt 很长仍不稳定，并且有高质量样本，再考虑微调。微调适合长期稳定的格式、风格和任务模式。
```

### 179. 微调和 RAG 如何结合

30 秒版：

```text
RAG 提供最新和可溯源知识，微调让模型更会遵循业务格式、风格或使用检索上下文。也可以微调 embedding 或 reranker 提升检索质量。
```

### 180. 什么时候不该微调

30 秒版：

```text
知识频繁变化、数据少且质量差、需求还不稳定、只是 Prompt 没调好、或者只是想解决安全权限问题时，不该优先微调。安全和权限必须靠系统层实现。
```

## 4. 复习建议

这部分可以按三句话记：

```text
Prompt：
  快速控制行为，但不是安全边界。

协议：
  OpenAPI 描述 API，RPC 负责远程调用，MCP 面向 AI 应用连接工具和上下文。

微调：
  缺知识优先 RAG，行为和格式稳定后再考虑微调。
```

面试收束句：

```text
这些技术不是互相替代，而是分层协作：Prompt 控制输入输出，RAG 提供知识，Tool/MCP 连接外部能力，微调改善稳定行为，权限和安全由工程系统兜底。
```

