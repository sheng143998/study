# 项目指标与 Bad Case 量化手册

> 目标：把项目回答从“我做了优化”升级成“我知道怎么定义指标、怎么定位问题、怎么证明优化有效”。  
> 这份文件配合 `53-端到端项目蓝图库.md`、`41-面试官评分Rubric与回答自检表.md`、`42-项目深挖追问防守库.md` 使用。

面试里最容易露怯的不是不会说 RAG/Agent，而是被问到：

```text
你说效果提升了，指标怎么定义？
bad case 怎么分类？
怎么知道是召回问题还是模型问题？
线上怎么观测？
怎么证明不是偶然变好？
```

本文件专门解决这些问题。

## 1. 指标回答的通用框架

任何 AI Agent 应用都可以按四层指标回答：

```text
业务指标
  -> 用户是否真的被帮助

质量指标
  -> 模型/检索/工具结果是否正确

工程指标
  -> 延迟、稳定性、成本、并发是否可接受

治理指标
  -> 权限、安全、审计、可回滚是否可控
```

标准回答：

```text
我不会只用“感觉回答更好”来评估，而是拆成业务、质量、工程和治理四层。业务上看任务完成率、满意率、人工转接率；质量上看检索命中、答案忠实度、工具调用准确率；工程上看延迟、错误率、成本；治理上看越权、审计、回滚和 bad case 回流。
```

## 2. RAG 指标体系

### 2.1 检索层指标

| 指标 | 含义 | 面试怎么说 |
|---|---|---|
| Recall@K | 正确证据是否进入前 K 个结果 | 判断召回有没有漏掉答案 |
| Precision@K | 前 K 个结果里有多少相关 | 判断上下文是否被噪声污染 |
| MRR | 第一个正确结果排第几 | 判断正确证据是否靠前 |
| NDCG | 多个相关结果排序质量 | 适合多证据问题 |
| Hit Rate | 是否至少命中一个正确证据 | 粗粒度看检索覆盖 |

检索问题定位句：

```text
如果答案错了，我会先看正确证据有没有进 topK。没进 topK 是召回问题；进了但排序靠后是 rerank 或 embedding 区分度问题；排在前面但答案仍错，多半是 Prompt、上下文组织或模型忠实度问题。
```

### 2.2 生成层指标

| 指标 | 含义 | 面试怎么说 |
|---|---|---|
| Faithfulness | 答案是否忠于给定上下文 | 防幻觉核心指标 |
| Answer Relevance | 是否回答了用户问题 | 防答非所问 |
| Citation Accuracy | 引用是否支持答案 | 企业知识库必问 |
| Completeness | 是否覆盖关键要点 | SOP/政策类问题重要 |
| Refusal Accuracy | 无答案时是否拒答 | 防编造 |

生成问题定位句：

```text
如果正确证据已经在上下文里，但模型仍然答错，我会看三件事：上下文是否太长导致注意力分散，Prompt 是否明确要求基于证据回答，以及是否需要把答案拆成结构化字段或句子级引用。
```

### 2.3 RAG bad case 分类

| 一级分类 | 二级原因 | 典型表现 | 修复动作 |
|---|---|---|---|
| 文档问题 | 解析失败 | PDF 表格丢列、目录混入正文 | 优化 parser、OCR、表格解析 |
| 切分问题 | chunk 断裂 | 答案跨 chunk，召回片段不完整 | 标题路径、overlap、父子 chunk |
| 召回问题 | query 不匹配 | 用户问法和文档说法不同 | query rewrite、多查询、HyDE |
| 排序问题 | 噪声靠前 | 相关但不回答问题的片段排前 | rerank、metadata boost |
| 生成问题 | 幻觉 | 没依据仍然回答 | 强引用、拒答策略、后校验 |
| 权限问题 | 越权召回 | A 部门看到 B 部门文档 | 检索前 metadata filter |
| 更新问题 | 索引滞后 | 新文档查不到 | 增量索引、版本号、任务重试 |

### 2.4 RAG 评测集怎么构造

最小评测集字段：

```text
question
gold_doc_id
gold_chunk_ids
reference_answer
question_type
difficulty
tenant/permission_tag
```

题型比例建议：

- 40% 高频 FAQ。
- 20% 政策/SOP 细节。
- 15% 多文档综合。
- 10% 需要拒答的问题。
- 10% 权限隔离问题。
- 5% 新文档/更新类问题。

回答模板：

```text
评测集不是只写一些简单 FAQ。我会按真实线上问题构造，至少包含高频问题、细节问题、多文档综合、无答案拒答和权限隔离几类。每题标正确证据 chunk 和参考答案，这样能同时评估检索和生成。
```

## 3. Agent 指标体系

### 3.1 核心指标

| 指标 | 含义 | 适用场景 |
|---|---|---|
| Intent Accuracy | 意图识别准确率 | 客服、流程 Agent |
| Tool Selection Accuracy | 工具选择是否正确 | 多工具 Agent |
| Argument Accuracy | 工具参数是否正确 | Tool Calling |
| Task Success Rate | 任务是否完成 | 业务流程 |
| Step Success Rate | 每一步节点成功率 | LangGraph/工作流 |
| Human Handoff Rate | 人工接管率 | 高风险场景 |
| Unsafe Action Rate | 错误执行/越权执行率 | 写操作场景 |
| Recovery Rate | 失败后恢复成功率 | 工具超时/异常 |
| Human Takeover Success | 用户/坐席接管后是否完成任务 | Browser/实时/高风险 Agent |

### 3.2 Agent bad case 分类

| 分类 | 典型表现 | 定位方式 | 修复动作 |
|---|---|---|---|
| 意图错 | 用户要退款，Agent 查物流 | 看 intent trace | 增加分类样本、澄清问题 |
| 工具错 | 该查订单却查库存 | 看 tool_call trace | 工具描述去重、router |
| 参数错 | order_id 填成用户 ID | 看 schema validation | Pydantic、slot filling |
| 状态丢 | 多轮追问忘记上下文 | 看 session state | checkpoint、状态持久化 |
| 重复执行 | 工单创建两次 | 看 idempotency key | 幂等键、状态机 |
| 越权执行 | 查询别人订单 | 看 auth log | 工具网关权限校验 |
| 不会停 | 循环调用工具 | 看 step count | 最大步数、终止条件 |
| 过度自信 | 低置信度仍执行 | 看 confidence/risk | 人工确认、风险分级 |
| 浏览器误点 | DOM/截图理解错，点击了错误按钮 | 看 action trace、URL、元素 selector、截图 | 动作白名单、关键动作确认、低置信度接管 |
| 网页注入 | 页面文本诱导 Agent 泄露信息或执行危险动作 | 看 page_content、tool decision、risk log | 页面内容当 untrusted data、动作策略硬约束 |

### 3.3 Agent 线上观测字段

建议每轮会话记录：

```text
trace_id
session_id
user_id
tenant_id
intent
agent_version
prompt_version
state_before
tool_name
tool_input
tool_output_summary
tool_latency
tool_status
risk_level
human_handoff
observation_source
action_policy
approval_id
final_answer
user_feedback
```

不要记录：

- 明文密码。
- 完整身份证/手机号等敏感信息。
- 超出业务必要范围的隐私字段。

面试表达：

```text
Agent 的观测重点不是只看最终回答，而是要记录每一步的状态、工具选择、参数、工具结果、风险等级和人工接管。这样 bad case 才能回放，不然线上错了只知道用户不满意，不知道哪一步错。
```

### 3.4 Trace Span 与 Eval Join

面试官问“trace 具体怎么落地”时，可以用 span tree 说明，不要只列字段。

| 证据 | 字段 | 证明什么 | 最小样例 |
|---|---|---|---|
| Trace Span | trace_id、span_id、parent_span_id、node、model/tool、latency_ms、token、error_code | 能把一次请求拆成可回放链路 | `root -> retriever -> rerank -> llm -> tool` |
| Eval Dataset Join | trace_id、case_id、dataset_version、judge_version、labeler、failure_type | bad case 能进入回归评测 | `trace_001 -> case_032 -> fail: tool_argument` |
| Release Evidence | change_id、prompt/model/tool/retrieval version、eval_gate、rollback_version、owner | 变更可灰度和回滚 | `change_017 -> prompt_v12 -> pass -> 10% gray` |

口播：

```text
我会把线上 trace 和离线 eval 打通。线上出错先通过 trace_id 找到失败 span，再把样本沉淀成 case_id，绑定 dataset_version 和 judge_version。下一次 prompt、模型、检索或工具 schema 变更时，eval gate 必须跑这些 bad case，避免修完一次又回归。
```

发布变更单和回滚单模板见 `32-P2系统设计平台加分版.md` 的 `8.1/8.2`；本文件只负责说明指标和证据如何进入变更验收。

### 3.5 Eval Gate / 发布门禁细节

如果面试官继续问“什么叫 gate 通过”，要能说出阈值、阻断条件和证据字段。不要只回答“我们跑了评测”。

门禁输入：

```text
change_id
dataset_version
judge_version
model/prompt/tool/retrieval version
traffic_stage(replay/shadow/canary/prod)
owner
rollback_version
```

门禁表：

| 门禁维度 | Pass 条件 | Block 条件 | 需要留下的证据 |
|---|---|---|---|
| 核心质量 | P0 case 全通过，P1 低于阈值的失败有解释 | 核心业务 case 失败、历史 bad case 回归 | case_id、failure_type、judge_reason |
| RAG 可信度 | 引用支持答案，无答案能拒答 | citation 不支持结论、无证据仍回答 | citation_doc_id、faithfulness、refusal_accuracy |
| Agent 工具 | 工具选择和参数正确，高风险动作需确认 | 错工具、错参数、重复执行、无需确认就写操作 | tool_name、schema_error、approval_id、idempotency_key |
| Data Agent | SQL 只读、指标口径正确、权限通过 | 写 SQL、越权查库、metric_id/version 不匹配 | metric_id、sql_readonly_pass、acl_pass |
| 实时体验 | first_token 和 p95 在预算内，SSE 有进度事件 | 首响超预算、长任务无 partial/tool_progress | turn_id、event_id、first_token_latency_ms |
| 成本预算 | cost_per_successful_task 不超预算 | 重试、上下文膨胀或强模型路由导致成本超阈值 | token_cost、cache_hit_rate、fallback_rate |
| 安全合规 | 注入、越权、敏感信息样例通过 | unauthorized_tool_rate 非 0、敏感字段输出 | safety_case_id、risk_level、audit_id |
| 回滚准备 | 有 rollback_version、owner、灰度比例 | 无版本记录、无法定位最近变更 | rollback_version、owner、gray_percent |

最小样例：

```text
gate_id=gate_2026_06_demo
change_id=change_012
dataset_version=data_agent_v3
judge_version=judge_v2
decision=block
block_reason=first_token_latency_ms 920 > budget 800; case_041 metric_id mismatch
rollback_version=prompt_v11
```

60 秒回答：

```text
我会把发布门禁拆成质量、工具、Data Agent、实时体验、成本、安全和回滚几类。每次变更先绑定 change_id，跑固定 dataset 和历史 bad case。只要 P0 case 失败、越权非 0、高风险动作缺审批、成本/延迟超预算，或者没有 rollback_version，就不放量。
```

## 4. Tool Calling 与 MCP 指标体系

### 4.1 工具层指标

| 指标 | 含义 |
|---|---|
| Tool Call Success Rate | 工具调用成功率 |
| Schema Validation Error Rate | 参数校验失败率 |
| Permission Deny Rate | 权限拒绝率 |
| Timeout Rate | 工具超时率 |
| Retry Success Rate | 重试后成功率 |
| Idempotency Conflict Rate | 幂等冲突率 |
| Unsafe Call Blocked | 高风险调用拦截数 |
| Tool Reuse Count | 工具被多少 Agent 复用 |

### 4.2 工具 bad case

| 问题 | 表现 | 修复 |
|---|---|---|
| 工具描述不清 | 模型不知道什么时候调用 | 补充 use_when / do_not_use_when |
| schema 太宽 | 参数乱填也能过 | 收紧 enum、format、required |
| 工具粒度太大 | 一个 execute 覆盖所有操作 | 拆成业务语义明确的小工具 |
| 错误码不清 | 模型无法处理失败 | 规范 error_code 和可恢复错误 |
| 写操作无确认 | 误删/误提交 | risk_level + confirm |
| 无审计 | 无法追责和复盘 | tool_call_log + trace_id |

### 4.3 OpenAPI 转 Tool 的评估清单

```text
工具名是否表达业务动作？
描述是否说明什么时候用？
输入字段是否有类型、范围、枚举？
输出字段是否解释业务含义？
错误码是否可被 Agent 理解？
是否标了风险等级？
是否有权限要求？
是否支持幂等？
是否能 mock？
是否能审计？
```

## 5. 模型网关指标体系

### 5.1 平台指标

| 指标 | 说明 |
|---|---|
| Request QPS | 请求规模 |
| P50/P95/P99 Latency | 延迟分布 |
| First Token Latency | 流式首 token 延迟 |
| Error Rate | 调用错误率 |
| Timeout Rate | 超时率 |
| Fallback Rate | 降级/备用模型比例 |
| Token Usage | 输入/输出 token |
| Cost Per Request | 单请求成本 |
| Cost By App/Tenant | 按应用/租户拆成本 |
| Prompt Version Impact | Prompt 版本对质量/成本影响 |

### 5.1.1 Agent SLO / Alerting

平台岗和 ToB 岗常追问“线上怎么知道坏了”。不要只说 trace，要能说 SLO、告警和事故处理。

| SLO 类别 | 指标 | 告警例子 |
|---|---|---|
| 质量 | task_success_rate、tool_call_accuracy、faithfulness | 某 prompt_version 成功率下降 5% |
| 稳定性 | p95_latency、timeout_rate、provider_error_rate、queue_lag | p95 超阈值 10 分钟 |
| 成本 | cost_per_successful_task、token_budget_overrun | 单成功任务成本上涨 30% |
| 安全 | unsafe_action_rate、unauthorized_tool_rate、memory_write_reject_rate | 高风险动作拦截突增或越权调用非 0 |

告警处理链：

```text
Alert
  -> 按 tenant / model_version / prompt_version / tool_version 聚合
  -> 找最近 change_id
  -> 关联 trace 和 eval case
  -> 决策：降级 / 回滚 / 限流 / 人工接管
  -> 复盘进入 bad case 和变更单
```

60 秒回答：

```text
我会把 Agent 观测分成质量、稳定性、成本和安全四类 SLO。告警不是只看单次失败，而是按租户、模型版本、prompt_version 和 tool_version 聚合，触发后先定位最近变更和失败 span，再决定降级、回滚、限流还是人工接管。事故样本要进入 eval set，避免同类问题复发。
```

### 5.2 成本异常排查

排查顺序：

```text
总成本上涨
  -> 请求量是否上涨
  -> 单请求 token 是否变长
  -> 输出长度是否失控
  -> 重试率是否上升
  -> 是否切到了更贵模型
  -> Prompt/RAG 上下文是否变长
  -> 是否有循环调用或异常任务
```

回答模板：

```text
我会按 app、tenant、model、prompt_version 拆成本。先区分是请求量上涨还是单次 token 变长，再看重试率、fallback、上下文长度和 Prompt 版本变更。如果是 RAG 上下文过长，就做上下文压缩和 topK 调整；如果是异常循环调用，就按 trace 定位并加最大步数和限流。
```

## 6. A/B 测试与灰度

### 6.1 什么时候需要 A/B

需要：

- 换 embedding 模型。
- 调整 chunk 策略。
- 上线新的 rerank。
- 改 Prompt 主模板。
- 切换 LLM。
- 改 Agent 工具选择策略。

不一定需要：

- 明显 bug 修复。
- 日志字段增加。
- 后台配置页修改。
- 只影响内部测试环境的改动。

### 6.2 灰度维度

- 按租户。
- 按部门。
- 按用户比例。
- 按问题类型。
- 按应用。
- 按 Prompt 版本。
- 按模型版本。

### 6.3 实验记录模板

```text
实验名称：
变更内容：
实验假设：
实验对象：
流量比例：
开始时间：
结束时间：
核心指标：
护栏指标：
结果：
是否全量：
回滚条件：
bad case：
复盘：
```

护栏指标例子：

- 错误率不能升高。
- P95 延迟不能超过阈值。
- 成本不能超过阈值。
- 投诉/差评不能升高。
- 高风险工具误调用为 0。

## 7. Bad Case 复盘模板

### 7.1 单条复盘

```text
case_id：
发生时间：
用户问题：
期望答案/动作：
实际答案/动作：
影响范围：
问题分类：
链路定位：
根因：
临时止血：
长期修复：
验证指标：
是否进入评测集：
负责人：
```

### 7.2 归因树

```text
bad case
  -> 输入问题
      -> 用户表达模糊
      -> 缺少上下文
      -> prompt injection
  -> 数据问题
      -> 文档缺失
      -> 文档过期
      -> 解析错误
  -> 检索问题
      -> chunk 不合理
      -> embedding 不匹配
      -> rerank 失败
  -> 编排问题
      -> 状态丢失
      -> 分支错误
      -> 工具选择错误
  -> 工具问题
      -> 参数错误
      -> 权限错误
      -> 超时失败
  -> 模型问题
      -> 幻觉
      -> 不遵循格式
      -> 拒答不准
```

### 7.3 面试回答模板

```text
我会先把 bad case 归因到链路节点，而不是直接说模型不好。比如 RAG 问答错了，先看文档是否存在，再看解析和 chunk，再看正确证据是否进入 topK，再看 rerank 排序，再看 Prompt 和生成是否忠实。每个 bad case 修完后要进入回归评测集，防止同类问题反复出现。
```

## 8. 面试中可以报的“合理指标”

如果你没有真实线上指标，可以诚实说“这是离线验证或 Demo 指标”。不要编造生产数据。

### 8.1 Demo/课程/个人项目

可以说：

```text
这个项目还没有真实线上流量，所以我没有伪造业务指标。我主要做了离线评测：构造了几十到几百条问题，标注正确文档片段，用 Recall@K、引用准确率和人工评分验证改动。
```

### 8.2 实习/内部试点

可以说：

```text
当时是内部试点阶段，样本量不算特别大，所以我更关注趋势而不是绝对数。我们主要看命中率、人工反馈、响应延迟和 bad case 类型，发现问题后迭代 chunk、rerank 和 Prompt。
```

### 8.3 生产项目

可以说：

```text
生产上我会同时看业务指标和护栏指标。业务指标包括任务完成率、满意率、人工转接率；护栏指标包括错误率、P95 延迟、成本、越权调用和高风险误执行。任何 Prompt 或模型变更都要先灰度观察。
```

## 9. 指标不要怎么说

不要说：

```text
准确率提升了很多。
模型效果挺好。
用户反馈不错。
我们优化了 Prompt，所以幻觉少了。
```

要说：

```text
我把效果拆成检索和生成两部分看。优化前正确证据经常进不了 top5，主要是 query 和文档表达不一致；后来加了 query rewrite 和混合召回，离线 Recall@5 有明显提升。生成侧又加了引用约束和无依据拒答，用 bad case 集回归验证，幻觉类问题减少。
```

## 10. 一页速背

```text
RAG 指标：
Recall@K / MRR / NDCG / Faithfulness / Citation Accuracy / Refusal Accuracy

Agent 指标：
Intent Accuracy / Tool Selection / Argument Accuracy / Task Success / Human Handoff / Unsafe Action

工具指标：
Schema Error / Permission Deny / Timeout / Retry / Idempotency / Audit

模型网关指标：
P95 / First Token / Error Rate / Fallback / Token / Cost / Prompt Version

bad case 归因：
输入 -> 数据 -> 检索 -> 编排 -> 工具 -> 模型 -> 治理

万能回答：
先定位链路，再给修复动作，最后放进评测集回归。
```
