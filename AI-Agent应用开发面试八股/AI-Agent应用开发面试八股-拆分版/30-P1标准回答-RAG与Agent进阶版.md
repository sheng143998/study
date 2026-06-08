# P1 标准回答：RAG 与 Agent 进阶版

> 用法：配合 `27-P0P1P2题库扩容版.md` 使用。  
> 本版覆盖 P1 的 101-135 题：RAG 进阶与 Agent 进阶。适合二面深挖、RAG/Agent 专项岗位、项目追问加分。

## 1. RAG 进阶：101-120

### 101. HyDE 是什么

30 秒版：

```text
HyDE 是 Hypothetical Document Embeddings。它先让模型根据用户问题生成一个假想答案或假想文档，再用这个假想文档做 embedding 去检索。它适合用户 query 很短、表达和文档差异较大的场景，但会增加模型调用成本，也可能引入偏差。
```

展开关键词：

- query expansion。
- 假想文档。
- 提升语义召回。
- 成本和偏差。

### 102. Multi-query retrieval 是什么

30 秒版：

```text
Multi-query retrieval 是让模型把原始问题改写成多个不同角度的 query，分别检索后合并结果。它能提升召回覆盖，适合用户问题含糊或表达单一的场景，但会增加检索成本和去重复杂度。
```

工程注意：

- 多 query 合并要去重。
- 可用 RRF 融合排序。
- 要控制 query 数量。

### 103. Query rewrite 怎么做

30 秒版：

```text
Query rewrite 是把用户原始问题改写成更适合检索的查询。常见做法包括补全多轮上下文、省略词消解、提取关键词、扩展同义词、拆分复杂问题。改写后最好保留原问题，避免改写偏离用户意图。
```

项目版：

```text
多轮客服场景里，用户说“这个怎么退”，需要结合历史把“这个”补成具体订单或商品。
```

### 104. 语义切分怎么做

30 秒版：

```text
语义切分不是固定 token 切，而是根据标题层级、段落边界、句子相似度或 embedding 变化来切。目标是让一个 chunk 内部语义相对完整，同时避免 chunk 过大带来噪声。
```

工程注意：

- 优先尊重文档结构。
- embedding 语义切分成本更高。
- 表格、代码块要特殊处理。

### 105. 表格文档如何切分和召回

30 秒版：

```text
表格不要简单按行打散。可以保留表头、单位、行列关系，把表格转成 Markdown 或结构化 JSON；按业务含义切分，并在元数据里保留页码、表名、列名。复杂表格可以单独走结构化查询或表格问答工具。
```

追问关键词：

- 表头继承。
- 单位和口径。
- Markdown table。
- SQL/结构化工具。

### 106. PDF OCR 错误怎么处理

30 秒版：

```text
OCR 错误会影响切分、embedding 和引用。处理方式包括选择更好的解析器/OCR、保留版面结构、清洗乱码、用规则修复常见错误、对低置信度区域标记或人工校验。关键文档不应完全依赖自动 OCR。
```

工程注意：

- OCR 置信度。
- 页码和坐标。
- 表格/扫描件特殊处理。

### 107. 代码文档 RAG 和普通文档有什么区别

30 秒版：

```text
代码文档更依赖结构和符号。切分时要保护函数、类、代码块、文件路径和接口定义；检索时关键词匹配很重要，因为函数名、错误码、类名需要精确匹配；答案要引用文件路径和代码位置，避免编造不存在的 API。
```

展开关键词：

- AST/符号索引。
- BM25 对函数名有效。
- repo path metadata。

### 108. 向量维度越高越好吗

30 秒版：

```text
不是。维度高可能表达能力更强，但存储、计算、索引和内存成本更高，也可能带来噪声。实际要根据 embedding 模型和评测集选择，看 Recall@K、延迟和成本。
```

追问关键词：

- 维度由模型决定。
- 不要盲目追高。
- 成本权衡。

### 109. cosine、dot product、L2 怎么选

30 秒版：

```text
要和 embedding 模型训练方式保持一致。cosine 看方向相似，dot product 受向量长度影响，L2 看空间距离。如果向量做了归一化，cosine 和 dot product 的排序可能接近。
```

工程注意：

- 看模型文档推荐。
- 用自己的评测集验证。

### 110. HNSW 原理是什么

30 秒版：

```text
HNSW 是一种基于多层图的近似最近邻索引。高层图用于快速跳转，底层图做精细搜索。它查询快、召回率高，但内存占用较大，构建和更新也有成本。
```

追问关键词：

- ANN。
- 多层图。
- recall/latency/memory trade-off。

### 111. IVF 和 PQ 是什么

30 秒版：

```text
IVF 是倒排文件索引，先把向量聚类到不同桶，查询时只搜索部分桶。PQ 是乘积量化，把向量压缩成更小编码，节省内存但可能损失精度。两者常用于大规模向量检索的性能和成本优化。
```

工程注意：

- IVF 控制搜索桶数。
- PQ 降低存储。
- 牺牲部分召回换性能。

### 112. FAISS 和 Milvus 区别是什么

30 秒版：

```text
FAISS 更像本地向量检索库，适合高性能实验或单机检索；Milvus 是服务化向量数据库，提供集合管理、元数据过滤、分布式、持久化、监控和运维能力，更适合生产 RAG。
```

追问关键词：

- library vs database。
- metadata。
- 多租户和服务化。

### 113. pgvector 适合什么场景

30 秒版：

```text
pgvector 适合数据规模中小、团队已经使用 PostgreSQL、希望关系数据和向量数据放在一起管理的场景。它运维简单，但超大规模、高 QPS、复杂 ANN 和多租户场景可能不如专门向量数据库。
```

工程注意：

- 快速落地。
- 事务和关系数据结合。
- 大规模要谨慎。

### 114. metadata filter 在召回前还是召回后

30 秒版：

```text
权限类 filter 最好在召回阶段就生效，避免无权限文档进入候选集。但具体实现取决于向量库能力。先召回后过滤可能导致 top_k 被无权限文档占满；先过滤再召回可能影响性能，需要根据过滤选择性调参。
```

项目版：

```text
tenant_id 这类强隔离条件要尽量前置，甚至用 collection 隔离。
```

### 115. 召回结果如何去重和合并

30 秒版：

```text
先按 doc_id、chunk_id 或内容 hash 去重，再合并 vector、BM25、多 query 的结果。排序融合可以用加权分数、归一化分数或 RRF。还可以合并相邻 chunk，减少上下文碎片。
```

工程注意：

- 去重后保留最高分和来源。
- RRF 对多路召回稳健。

### 116. 如何评估 citation accuracy

30 秒版：

```text
citation accuracy 评估答案引用是否真实支持回答。第一层检查引用编号是否来自本次上下文，第二层检查被引用片段是否包含答案依据，第三层人工或 LLM-as-judge 判断答案和证据是否一致。
```

追问关键词：

- 引用存在性。
- 引用支持性。
- groundedness。

### 117. LLM-as-judge 有什么问题

30 秒版：

```text
LLM-as-judge 成本低、扩展快，但会有模型偏差、不稳定、受 prompt 影响、可能偏好长答案或同源模型答案。高风险评测不能只依赖它，最好结合人工标注、规则指标和抽样复核。
```

工程注意：

- judge prompt 版本化。
- 多 judge 交叉。
- 人工校准集。

### 118. 如何构建 RAG 评测集

30 秒版：

```text
从真实用户问题、客服工单、搜索日志和 badcase 中抽样，给每个问题标注期望文档、答案要点和引用来源。评测集要覆盖高频、长尾、权限、无答案、过期文档和困难问题，并定期更新。
```

追问关键词：

- query。
- golden docs。
- expected answer。
- no-answer cases。

### 119. 知识库多租户怎么隔离

30 秒版：

```text
tenant_id 要贯穿文档、chunk、向量、关键词索引、缓存、日志和评测。强隔离可以按 tenant 建 collection 或 index；弱隔离用 metadata filter。缓存 key 和 trace 也必须包含租户和权限信息。
```

工程注意：

- 数据串租是高危问题。
- 日志也要隔离。
- 权限版本影响缓存。

### 120. RAG 如何支持多语言

30 秒版：

```text
可以选择多语言 embedding 模型，或先做 query/document 翻译，再检索。多语言场景要注意分词、术语对齐、跨语言召回和答案语言控制。评测集也要覆盖不同语言 query。
```

追问关键词：

- multilingual embedding。
- cross-lingual retrieval。
- terminology glossary。

## 2. Agent 进阶：121-135

### 121. Agent 的 memory 分哪几类

30 秒版：

```text
常见分短期记忆、长期记忆和结构化状态。短期记忆是当前会话上下文，长期记忆是跨会话保存的用户偏好或历史事实，结构化状态保存任务进度、工具结果、确认状态等可控字段。
```

工程注意：

- memory 不是越多越好。
- 敏感记忆要可删除。

### 122. 短期记忆和长期记忆怎么实现

30 秒版：

```text
短期记忆可以放在会话上下文、Redis 或数据库里，并做窗口裁剪和摘要。长期记忆可以放关系数据库或向量库，但要有权限、过期、用户可见和删除机制，不能无限沉淀敏感信息。
```

项目版：

```text
任务状态用结构化字段，不只靠聊天历史。
```

### 123. 如何压缩历史对话

30 秒版：

```text
可以保留最近 N 轮，把更早历史总结成摘要；也可以抽取结构化事实，如用户目标、约束、订单号、已确认动作。重要状态要结构化保存，避免摘要丢失关键信息。
```

追问关键词：

- sliding window。
- summary memory。
- structured state。

### 124. 结构化状态和自然语言摘要怎么结合

30 秒版：

```text
自然语言摘要适合保留背景和对话语义，结构化状态适合保存可执行信息，例如 tool_results、approval_status、step_count、pending_action。生产中两者结合，关键控制流依赖结构化状态。
```

工程注意：

- 不让模型从摘要里猜关键状态。
- 状态字段用于路由和安全。

### 125. Agent 如何做任务规划

30 秒版：

```text
可以用 Planner-Executor：Planner 根据目标生成步骤，Executor 逐步执行工具并根据 observation 更新状态。计划要有结构化输出、最大步骤限制和可执行性检查。
```

追问关键词：

- plan schema。
- step validation。
- observation。

### 126. 计划失败后如何重规划

30 秒版：

```text
先判断失败原因：工具不可用、参数错误、权限不足、信息缺失还是目标变化。把失败 observation 写入状态，让 Planner 基于当前状态生成修正计划。连续失败要停止或转人工。
```

工程注意：

- 不要无限重规划。
- error_count。
- fallback。

### 127. 多 Agent 如何避免互相发散

30 秒版：

```text
要有 supervisor 控制，不让多个 Agent 自由聊天。每个 Agent 单一职责，输出结构化结果，限制轮数和成本，冲突由 reviewer 或 supervisor 解决，最终由一个节点负责收敛。
```

追问关键词：

- role boundary。
- schema。
- max rounds。
- convergence。

### 128. Supervisor Agent 是什么

30 秒版：

```text
Supervisor Agent 是调度者，负责根据任务状态选择调用哪个子 Agent 或工具，并判断是否继续、重试或结束。它不一定执行具体任务，而是控制流程和收敛。
```

项目版：

```text
研究助手里 supervisor 可以调度 searcher、reader、critic、writer。
```

### 129. Agent 如何做权限边界

30 秒版：

```text
权限边界不能靠 prompt。运行时只给 Agent 暴露当前用户可用工具和资源，工具执行前再做后端鉴权，RAG 检索带 metadata filter，高风险操作需要确认和审计。
```

工程注意：

- least privilege。
- tool whitelist。
- resource filter。

### 130. Agent 如何接入人工审核

30 秒版：

```text
在高风险动作前插入 human_review 节点，把待执行动作、参数、证据和风险说明展示给用户或审核人。确认后更新 approval_status，从 checkpoint 恢复继续执行。
```

追问关键词：

- interrupt。
- approval_status。
- checkpoint。

### 131. Agent 如何做成本预算

30 秒版：

```text
给每次任务设置 token budget、最大步骤数、最大工具调用次数和最大运行时间。每步记录已用 token 和成本，超过预算时降级、停止或请求用户确认继续。
```

工程注意：

- 多 Agent 成本容易失控。
- 简单任务路由小模型。

### 132. Agent 如何支持中断和恢复

30 秒版：

```text
需要把状态持久化，比如 messages、task_id、current_node、tool_results、pending_action、approval_status。中断时保存 checkpoint，恢复时从当前节点和状态继续，而不是重新跑完整任务。
```

项目版：

```text
human-in-the-loop 和长任务都依赖 checkpoint。
```

### 133. Agent 如何处理工具返回的脏数据

30 秒版：

```text
工具结果要先经过 schema 校验、字段脱敏、错误码转换和摘要压缩，再返回给模型。不能把异常栈、敏感字段或大量无关数据直接塞进上下文。
```

工程注意：

- tool result sanitizer。
- structured observation。
- max length。

### 134. Agent 如何处理用户意图变化

30 秒版：

```text
每轮输入都要重新做意图识别或状态校验。如果用户目标变化，需要取消或挂起当前 pending_action，清理不再有效的计划，并重新路由。高风险动作执行前必须确认最新意图。
```

追问关键词：

- intent drift。
- pending action invalidation。
- user confirmation。

### 135. Agent 如何做线上 A/B

30 秒版：

```text
可以灰度不同 prompt、模型、工具描述、检索策略和 workflow。指标不只看最终满意度，还要看任务成功率、工具调用准确率、平均步骤数、人工介入率、延迟、成本和安全事件。
```

工程注意：

- 按用户/租户分流。
- 版本可回滚。
- trace 可对比。

## 3. 复习建议

这份 P1 不建议死背。建议按三条主线记：

```text
RAG 进阶：
  query 变换 -> 切分/索引 -> 多路召回 -> 排序/引用 -> 评测/多租户

Agent 进阶：
  memory/state -> planning -> supervisor -> permission -> human review -> budget/recovery

工程边界：
  所有智能行为都要有权限、预算、终止、审计和评测。
```

一句话收束：

```text
P1 题的重点不是名词解释，而是能把技术点放回生产系统里讲清楚它解决什么问题、引入什么成本、如何评估和兜底。
```

