# 72-大模型基础与 Embedding 面试手册

> 目标：补齐 AI Agent / RAG / 大模型应用开发岗位里常见的模型基础追问。  
> 定位：不是算法研究岗深度，而是应用工程师必须能解释清楚的 Transformer、token、embedding、采样参数、上下文窗口、rerank、量化和微调基础。

适用场景：

- 一面问基础概念：Transformer、Attention、token、embedding 是什么。
- RAG 深挖：embedding 模型怎么选，向量相似度怎么算，rerank 为什么有效。
- 模型网关深挖：temperature、top_p、max_tokens、流式输出、上下文窗口怎么影响效果和成本。
- 算法应用岗：SFT、LoRA、DPO、RLHF、量化、蒸馏的基本边界。

配合文件：

- `04-RAG与向量数据库.md`：RAG 链路和向量数据库。
- `06-模型微调.md`：SFT/LoRA/PEFT。
- `31-P1标准回答-Prompt协议微调版.md`：Prompt、协议、微调标准回答。
- `60-技术选型与取舍防守手册.md`：RAG vs 微调、模型选择、成本取舍。
- `68-近期JD趋势与面试押题增量.md`：近期 JD 里的模型基础趋势。

---

## 1. 应用工程师要懂到什么程度

大模型应用开发不是要求你手写 Transformer 训练框架，但要能解释：

```text
模型为什么能生成文本
  -> token 怎么进入模型
  -> attention 怎么建模上下文
  -> embedding 为什么能做语义检索
  -> 参数怎么影响输出
  -> 为什么会幻觉
  -> 如何用 RAG / Tool / 微调 / 评测补工程短板
```

面试边界：

| 岗位 | 需要掌握 | 不一定要求 |
|---|---|---|
| AI 应用开发 | token、上下文、采样、embedding、RAG、推理参数 | 从零训练大模型 |
| RAG 工程师 | embedding、相似度、ANN、rerank、评测 | 复杂训练损失推导 |
| Agent 工程师 | LLM 推理不稳定、工具调用格式、上下文管理 | Transformer 细节实现 |
| 平台/网关 | 模型路由、token 成本、限流、流式、缓存、fallback | 模型预训练细节 |
| 算法应用 | SFT、LoRA、DPO、embedding/rerank 微调 | 大规模分布式训练优化 |

速背句：

> 应用工程师不用把模型当黑盒，但也不需要把自己伪装成预训练专家。重点是理解模型能力边界，并知道如何用检索、工具、评测、缓存、路由和安全策略把它变成可靠系统。

---

## 2. Transformer 和 Attention

### 2.1 Transformer 是什么

30 秒回答：

```text
Transformer 是当前大模型的主流基础架构，它用 self-attention 建模序列中 token 之间的关系，相比 RNN 更适合并行训练，也能处理长上下文。大语言模型通常基于 decoder-only Transformer，根据已有 token 预测下一个 token。
```

应用岗加一句：

```text
我不需要从零实现 Transformer，但理解它按 token 处理上下文，有助于解释上下文窗口、token 成本、注意力漂移和长文本效果下降。
```

### 2.2 Self-Attention 是什么

通俗解释：

```text
Self-attention 是让每个 token 在生成表示时，动态关注输入序列中其他 token。比如问题里的“它”指代什么，要根据上下文中哪些词更相关来判断。
```

关键点：

- Query、Key、Value 用来计算 token 之间的相关性。
- Attention 权重决定当前 token 更关注哪些上下文。
- Multi-head attention 让模型从多个角度建模关系。

面试追问：

> 为什么长上下文还是会效果变差？

回答：

```text
上下文窗口变长不等于模型能同等关注所有内容。长上下文会增加噪声、稀释关键信息，也会增加 token 成本和延迟。所以 RAG 里要做检索、rerank、去重、压缩和引用，而不是把所有文档都塞进 prompt。
```

### 2.3 Decoder-only、Encoder-only、Encoder-Decoder

| 架构 | 典型用途 | 例子 |
|---|---|---|
| Encoder-only | 理解、分类、embedding、rerank | BERT 类模型 |
| Decoder-only | 文本生成、对话、代码生成 | GPT / Llama / Qwen / DeepSeek 类 |
| Encoder-Decoder | 翻译、摘要、seq2seq | T5 类 |

面试回答：

```text
生成式大语言模型多是 decoder-only，适合 next token prediction；embedding 或 rerank 常用 encoder 或 cross-encoder 思路，更关注文本表示和 query-doc 匹配。
```

---

## 3. Token 和上下文窗口

### 3.1 Token 是什么

30 秒回答：

```text
Token 是模型处理文本的基本单位，可以是字、词、子词或符号。模型不是直接看中文句子，而是先分词成 token，再转成向量进行计算。输入和输出 token 数直接影响成本、延迟和上下文容量。
```

常见追问：

> token 越多越好吗？

回答：

```text
不是。token 多能放更多上下文，但也会增加成本和延迟，还可能引入噪声。RAG 中要控制 topK、chunk 长度、重复片段和历史对话，保证上下文最小充分。
```

### 3.2 上下文窗口

上下文窗口是模型一次请求能处理的最大 token 数，包括：

```text
system prompt
developer prompt
user question
retrieved context
chat history
tool results
expected output
```

工程影响：

- 窗口越大，成本越高。
- 长上下文不一定效果更好。
- 输出 token 也占预算。
- 多轮 Agent 容易把历史塞爆。

实践策略：

1. RAG 只放 topN 证据。
2. 对历史对话做摘要。
3. 工具结果只保留必要字段。
4. 长文档先检索再压缩。
5. 给 Agent 设置 token budget。

### 3.3 上下文溢出怎么办

标准回答：

```text
我会先做上下文预算，把 system、用户问题、检索片段、工具结果和输出分别设上限。RAG 侧控制 topK 和 chunk 长度，Agent 侧对历史做摘要，对工具结果做字段裁剪。如果仍然超限，就分阶段处理或让模型先生成中间摘要，而不是直接截断关键信息。
```

---

## 4. Embedding 基础

### 4.1 Embedding 是什么

30 秒回答：

```text
Embedding 是把文本、图片等对象映射成向量，使语义相近的内容在向量空间中距离更近。RAG 里会把文档 chunk 和用户 query 都转成 embedding，再用相似度检索相关内容。
```

面试高分点：

- embedding 适合语义相似，但不等于事实正确。
- 对专有名词、编号、代码、表格，BM25 可能更稳。
- embedding 模型要和语言、领域、任务匹配。
- embedding 质量要用 Recall@K、MRR、NDCG 评估。

### 4.2 向量相似度

常见度量：

| 度量 | 含义 | 注意 |
|---|---|---|
| Cosine similarity | 看方向相似度 | 常用于归一化 embedding |
| Dot product | 向量点积 | 受向量长度影响 |
| Euclidean distance | 欧氏距离 | 距离越小越相似 |

面试回答：

```text
相似度选择要和 embedding 模型训练方式、向量是否归一化、向量库索引支持有关。实际不只看理论，要用评测集比较 Recall@K 和排序质量。
```

### 4.3 Embedding 维度越高越好吗

回答：

```text
不一定。维度高可能表达能力更强，但存储、索引和计算成本更高，也可能引入噪声。应该看模型本身、业务数据和评测结果。大规模知识库还要考虑索引内存、召回延迟和成本。
```

### 4.4 Embedding 模型怎么选

选择维度：

1. 语言：中文、英文、多语言。
2. 领域：通用、法律、金融、代码、医学。
3. 任务：问答检索、相似文本、聚类、去重。
4. 质量：Recall@K、MRR、NDCG。
5. 成本：调用价格、延迟、吞吐、部署方式。
6. 工程：向量维度、批量能力、是否支持私有化。

标准回答：

```text
我不会只看模型榜单。会先用业务问题集和标准证据构造评测集，对比不同 embedding 在 Recall@K、MRR、延迟和成本上的表现。如果领域词很多，还会考虑混合检索或领域 embedding 微调。
```

### 4.5 Embedding 微调

适合：

- 通用 embedding 对领域术语不敏感。
- query 和文档表达差异大。
- 有 query-doc 正负样本。
- 检索召回是主要瓶颈。

不适合：

- 数据很少且标注不稳。
- 文档质量本身很差。
- 问题主要在 rerank 或生成。
- 只是想让模型记住新知识。

---

## 5. Rerank 基础

### 5.1 Rerank 是什么

30 秒回答：

```text
Rerank 是在粗召回之后，用更强但更慢的模型重新排序候选文档。向量召回适合快速找一批可能相关的 chunk，reranker 同时看 query 和文档内容，能更精细判断相关性。
```

### 5.2 Bi-encoder vs Cross-encoder

| 模型 | 方式 | 优点 | 缺点 |
|---|---|---|---|
| Bi-encoder | query 和 doc 分别编码成向量 | 快，可预计算 | 匹配精度较弱 |
| Cross-encoder | query 和 doc 一起输入模型评分 | 精度高 | 慢，不能大规模预计算 |

RAG 常见组合：

```text
Bi-encoder / BM25 粗召回 topK
  -> Cross-encoder / LLM rerank topN
  -> LLM 生成答案
```

### 5.3 什么时候不用 rerank

回答：

```text
如果召回候选很少、延迟要求极严、问题很简单，或者向量召回已经足够稳定，可以先不上 rerank。rerank 能提升相关性，但会增加延迟和成本，需要用评测集和 P95 延迟一起衡量。
```

---

## 6. 生成参数

### 6.1 Temperature

含义：

```text
temperature 控制采样随机性。越低输出越稳定，越高输出越发散。
```

使用建议：

| 场景 | 建议 |
|---|---|
| 知识库问答 | 低 temperature |
| 结构化输出 | 低 temperature |
| 代码生成 | 较低 temperature |
| 文案创作 | 可适当提高 |
| 多样化 brainstorming | 可提高 |

面试回答：

```text
企业问答和工具调用我倾向低 temperature，因为可控和稳定更重要。创意类任务可以提高，但也要配合评测和安全边界。
```

### 6.2 Top-p / Top-k

解释：

- top_p：从累计概率达到 p 的候选 token 中采样。
- top_k：只从概率最高的 k 个候选 token 中采样。

回答：

```text
它们都影响输出多样性。生产中不会盲目调大，而是根据任务稳定性要求、格式约束和评测结果设置。
```

### 6.3 Max tokens

max tokens 控制最大输出长度。

工程意义：

- 控制成本。
- 避免模型输出过长。
- 防止 Agent 单步占用过多预算。
- 和上下文窗口一起决定请求能否成功。

### 6.4 Stop sequence

stop sequence 用于遇到特定字符串时停止生成。

常见用途：

- 防止模型输出多余段落。
- 控制多轮格式。
- 工具调用或结构化输出时截断。

---

## 7. 幻觉和事实性

### 7.1 为什么会幻觉

原因：

- 模型本质是按概率生成，不保证事实正确。
- 训练数据里可能有错误或过期信息。
- Prompt 问题超出模型知识范围。
- RAG 没召回正确证据。
- 上下文太长或噪声太多。
- 模型为了满足用户问题而过度补全。

### 7.2 工程治理

| 手段 | 作用 |
|---|---|
| RAG | 给模型提供外部证据 |
| Tool | 查询实时系统或数据库 |
| Citation | 让答案可追溯 |
| No-answer | 无证据时拒答 |
| Verifier | 校验答案是否被证据支持 |
| Human review | 高风险场景人工确认 |
| Eval set | 持续评估幻觉和 bad case |

标准回答：

```text
幻觉不能只靠调 prompt 解决。知识类问题用 RAG 和引用，实时问题用工具查询，高风险结论用规则或人工审核。评估上要看 faithfulness、citation accuracy、no-answer accuracy 和 bad case 回归。
```

---

## 8. 微调基础补充

### 8.1 预训练、SFT、对齐

| 阶段 | 作用 |
|---|---|
| 预训练 | 学语言规律、世界知识和通用能力 |
| SFT | 学会按指令完成任务 |
| RLHF / DPO | 让输出更符合人类偏好和安全策略 |
| LoRA / PEFT | 用较低成本适配任务或领域 |

### 8.2 RAG vs 微调

简洁回答：

```text
知识更新、私有知识、引用溯源和权限隔离优先 RAG；输出格式、风格、固定任务模式和工具调用格式稳定性可以考虑微调。微调不是 RAG 的替代，而是解决不同问题。
```

### 8.3 微调为什么不能解决所有问题

```text
微调不能保证事实实时更新，也不能天然解决权限、引用和工具执行安全。它还能引入过拟合、遗忘和评测偏差。生产里要结合 RAG、工具、规则、评测和灰度。
```

---

## 9. 量化、蒸馏和部署基础

### 9.1 量化是什么

30 秒回答：

```text
量化是用更低精度表示模型权重或激活，比如从 FP16 降到 INT8/INT4，以减少显存和推理成本。代价是可能损失一定效果，需要评测确认。
```

适合：

- 私有化部署。
- 成本敏感。
- 推理资源有限。
- 可接受轻微效果损失。

### 9.2 蒸馏是什么

```text
蒸馏是用大模型作为 teacher，训练更小的 student 模型学习它的输出或能力，用于降低推理成本和延迟。
```

### 9.3 私有化模型部署要考虑什么

```text
要考虑模型大小、显存、并发、量化、推理框架、上下文长度、吞吐、延迟、日志脱敏、权限、升级和回滚。不能只说把模型下载到服务器。
```

---

## 10. 模型网关里的基础概念

模型网关要理解的模型基础：

| 概念 | 网关里的作用 |
|---|---|
| context window | 判断请求能否进入某模型 |
| input/output token | 成本统计和限额 |
| temperature | 任务稳定性配置 |
| streaming | 降低首 token 体验延迟 |
| model version | 灰度和回滚 |
| embedding model | 检索质量和成本 |
| rerank model | 排序质量和延迟 |
| fallback model | 高可用和质量兜底 |

标准回答：

```text
模型网关不只是转发请求，还要理解模型能力边界。比如上下文窗口决定能处理多长输入，token 决定成本，temperature 影响稳定性，模型版本影响灰度和回滚，fallback 要提前评测质量差异。
```

---

## 11. 面试高频问答

### 11.1 Transformer 为什么适合大模型？

```text
Transformer 用 self-attention 建模 token 之间的关系，训练时比 RNN 更容易并行，也能扩展到大规模数据和参数。大语言模型通常基于 decoder-only Transformer 做 next token prediction。
```

### 11.2 Embedding 和大模型生成有什么区别？

```text
Embedding 模型把文本转成向量，用于相似度检索、聚类、去重；生成模型根据上下文生成 token。RAG 里 embedding 负责找证据，LLM 负责基于证据组织答案。
```

### 11.3 为什么向量召回还要 BM25？

```text
向量召回擅长语义相似，但对编号、专有名词、代码、精确关键词可能不如 BM25。混合检索能兼顾语义和词面匹配，实际效果要用评测集验证。
```

### 11.4 TopK 越大越好吗？

```text
不是。TopK 大可能提高召回覆盖，但也会引入噪声、增加 rerank 和 token 成本，甚至降低生成质量。要结合 Recall@K、citation accuracy、P95 延迟和成本调优。
```

### 11.5 Temperature 怎么设？

```text
知识问答、结构化输出、工具调用场景要低 temperature，保证稳定；创意写作可以适当提高。生产里不是凭感觉调，而是看格式正确率、人工评分、幻觉率和一致性。
```

### 11.6 长上下文模型是不是可以不用 RAG？

```text
不一定。长上下文能放更多内容，但成本高、延迟高、噪声多，也不能天然解决权限、更新、引用和评测。企业知识库仍然需要 RAG 做检索、过滤、引用和权限治理。
```

### 11.7 为什么模型会幻觉？

```text
模型按概率生成文本，不保证事实正确。知识过期、问题超范围、上下文噪声、召回错误都会导致幻觉。工程上用 RAG、工具查询、引用校验、无答案策略和评测集治理。
```

### 11.8 LoRA 为什么省资源？

```text
LoRA 冻结原模型权重，只训练低秩适配矩阵，训练参数少、显存需求低、adapter 可切换，所以比全量微调成本低。
```

---

## 12. 公司风格下的模型基础

| 公司/方向 | 容易问的模型基础 |
|---|---|
| 腾讯 | embedding 召回质量、答案可信、成本和用户体验 |
| 阿里/阿里云 | 模型网关、上下文窗口、token 成本、私有化部署、LoRA |
| 百度 | RAG/搜索、embedding/rerank、模型效果评估 |
| 字节 | 模型参数、成本延迟、实验评测、工程优化 |
| 云厂商/ToB | 模型部署、量化、私有化、数据边界 |
| 算法应用岗 | SFT、LoRA、DPO、embedding/rerank 微调 |

---

## 13. 项目表达模板

如果项目里涉及模型基础，可以这样讲：

```text
这个项目里我没有只把模型当 API 调用。RAG 侧我关注 embedding 模型是否适合中文和领域术语，用 Recall@K 和 MRR 评估；生成侧我控制上下文 token、temperature 和引用约束；平台侧记录 input/output token、latency、model_version 和成本。遇到召回不准时，会先区分是 chunk、embedding、query rewrite、topK 还是 rerank 问题。
```

如果面试官问你模型底层不深怎么办：

```text
我不是预训练方向，但作为应用工程师，我会理解模型的关键边界：token 成本、上下文窗口、采样稳定性、embedding 检索质量、幻觉和评测。我的重点是把这些边界转成工程策略，比如 RAG、工具查询、缓存、模型路由和安全兜底。
```

---

## 14. 上场前检查

- [ ] 能用 30 秒解释 Transformer 和 self-attention。
- [ ] 能解释 token、上下文窗口和成本关系。
- [ ] 能解释 embedding、向量相似度和 embedding 模型选型。
- [ ] 能解释 rerank 为什么提升排序。
- [ ] 能解释 temperature/top_p/max_tokens。
- [ ] 能解释幻觉为什么发生，以及工程治理手段。
- [ ] 能解释 RAG vs 微调。
- [ ] 能解释 LoRA、量化、蒸馏的基本用途。
- [ ] 能把模型基础落到自己的项目里。

---

## 15. 最终速背

1. Transformer 用 attention 建模 token 关系，LLM 通常预测下一个 token。
2. Token 决定上下文容量、成本和延迟。
3. 长上下文不等于不用 RAG，企业场景还要权限、引用、更新和评测。
4. Embedding 做语义表示，RAG 用它找证据，不保证事实正确。
5. 向量召回适合语义，BM25 适合关键词和编号，混合检索常更稳。
6. Rerank 用更强模型精排候选，但会增加延迟和成本。
7. Temperature 越低越稳定，知识问答和工具调用通常用低值。
8. 幻觉要靠 RAG、工具、引用、拒答、评测和人工兜底治理。
9. 微调适合格式、风格和稳定任务，不适合频繁更新知识。
10. 模型网关要理解 token、上下文、版本、成本、fallback 和路由。

