# 70-项目作品集与 Demo 证据包手册

> 目标：把 AI Agent / RAG / MCP / 大模型后端项目整理成面试可展示、可追问、可自证的作品集证据包。  
> 这份文件不鼓励包装虚假经历，而是帮助你把真实项目、练习项目、试点项目中的证据整理得更像工程交付。

适用场景：

- 面试官问“你这个项目有没有 Demo / README / 代码 / 截图 / 评测记录？”
- 简历写了项目，但担心被追问“怎么证明是你做的？”
- 没有大规模线上指标，只能用离线评测、Demo 反馈和工程证据防守。
- 想把企业知识库、客服 Agent、工具平台、模型网关项目放进作品集。

配合文件：

- `50-个人项目素材库模板.md`：填真实项目素材。
- `53-端到端项目蓝图库.md`：选择和改造项目蓝图。
- `54-项目指标与BadCase量化手册.md`：准备指标和 bad case。
- `58-简历逐行追问预案手册.md`：防守简历 bullet。
- `59-项目分轮次口播稿模板.md`：按轮次讲项目。
- `66-面试上场包与成品验收清单.md`：分层验收口径。

---

## 1. 什么是项目证据包

项目证据包不是把所有代码和截图塞给面试官，而是准备一组能支撑项目真实性和工程能力的材料。

最小证据包：

```text
README
  -> 架构图
  -> 核心链路说明
  -> Demo 截图/录屏
  -> API 示例
  -> 评测表
  -> bad case 记录
  -> 代码目录说明
  -> 脱敏说明
```

面试中的作用：

| 面试官担心 | 证据包如何回应 |
|---|---|
| 你是不是只调了 API | 展示模块拆分、接口、日志、评测和治理 |
| 项目是不是编的 | 展示 README、目录、截图、评测样例、bad case |
| 你到底做了什么 | 展示自己负责的模块和 commit/文档/接口说明 |
| 有没有工程能力 | 展示异常处理、限流、权限、日志、部署、测试 |
| 没有线上指标怎么办 | 展示离线评测、Demo 反馈、人工验收和下一步上线方案 |

原则：

1. 证据可以脱敏，但不能造假。
2. 不展示公司机密、客户数据、内部代码和未授权材料。
3. 没有线上指标就说没有，用离线评测和 Demo 反馈证明验证过程。
4. 证据包不需要很花哨，要能支撑追问。

---

## 2. 证据包目录模板

建议每个项目准备一个目录：

```text
project-portfolio/
  README.md
  architecture/
    system-overview.md
    rag-flow.md
    agent-flow.md
  demo/
    demo-script.md
    screenshots.md
    sample-questions.md
  api/
    openapi-summary.md
    request-response-examples.md
    tool-schema-examples.md
  evaluation/
    eval-set-sample.md
    metrics-summary.md
    badcase-log.md
  operations/
    logging-fields.md
    timeout-retry-fallback.md
    cost-latency-summary.md
  resume/
    bullet-version.md
    two-minute-pitch.md
    faq-defense.md
  privacy/
    desensitization-note.md
```

如果只剩 2 小时，保留最小版：

```text
README.md
architecture.md
demo-script.md
metrics-and-badcase.md
faq-defense.md
```

---

## 3. README 模板

一个好 README 要让面试官 1 分钟看懂：

- 这是解决什么问题。
- 用户是谁。
- 你做了哪一部分。
- 架构怎么走。
- 指标和 bad case 是什么。
- 生产化还差什么。

模板：

```markdown
# 项目名称

## 1. 项目背景

这个项目面向 [用户/业务方]，解决 [具体痛点]。原流程的问题是 [查找慢/问答不准/人工成本高/工具操作分散]，所以我们用 [RAG/Agent/Tool/MCP/模型网关] 做了一个 [系统/模块/原型/内部试点]。

## 2. 我的职责

- 主责：
- 参与：
- 联调：
- 没有负责：

## 3. 核心能力

- RAG：
- Agent：
- Tool Calling：
- 后端接口：
- 评测：
- 权限/审计：

## 4. 架构概览

用户请求
  -> API Gateway
  -> Auth / Tenant Filter
  -> Retriever / Agent Orchestrator
  -> Tool / LLM
  -> Citation / Validation
  -> Response / Trace / Feedback

## 5. 技术选型

| 模块 | 选择 | 原因 |
|---|---|---|
| 后端 | FastAPI | 支持异步和 SSE |
| 编排 | LangGraph / workflow | 需要状态和条件分支 |
| 检索 | hybrid search + rerank | 兼顾语义和关键词 |
| 工具 | OpenAPI / MCP adapter | 统一工具 schema |
| 评测 | golden set + bad case | 支持迭代验证 |

## 6. 指标和验证

- 离线问题集：
- Recall@K：
- 引用准确率：
- 平均延迟：
- 成本/延迟：p95_latency、first_token_latency、token_cost、cache_hit_rate、fallback_rate。
- 发布门禁快照：gate_id、dataset_version、judge_version、gate_decision(pass/block)、p95_latency_ms、first_token_latency_ms、cost_per_successful_task、safety_case_pass_rate、trace_id/change_id/approval_id。
- 工具调用成功率：
- 人工验收：

最小样例：

```text
gate_2026_06_demo：dataset=data_agent_v3，judge=judge_v2，decision=pass；
p95=1800ms<=2500ms，first_token=420ms<=800ms，cost_per_successful_task=0.012<=0.020；
safety_case_pass=38/38，trace_id=trace_demo_017，change_id=change_012，approval_id=approval_a2a_003。
```

## 7. 典型 bad case

| 现象 | 根因 | 修复 | 验证 |
|---|---|---|---|
| | | | |

## 8. 生产化边界

当前阶段：Demo / 内部试点 / 小范围上线 / 生产上线

已完成：
- 

可补充：
- 权限：
- 灰度：
- 监控：
- 成本：
- 回滚：

## 9. 脱敏说明

本项目展示材料已移除真实客户、真实业务数据、内部域名、密钥、token、账号和未公开接口。
```

---

## 4. Demo 演示脚本

Demo 不要只展示“问一句答一句”，要展示系统能力。

### 4.1 RAG Demo 脚本

```markdown
# RAG Demo 脚本

## 场景
用户要查询公司制度/产品文档/运维手册中的问题。

## 演示问题
1. 一个标准问题：能直接命中文档。
2. 一个表达不一致的问题：测试语义召回。
3. 一个需要引用的问题：测试 citation。
4. 一个无答案问题：测试拒答。
5. 一个权限问题：测试 tenant / ACL filter。

## 演示顺序
1. 上传或展示文档。
2. 展示 chunk / metadata。
3. 发起用户问题。
4. 展示召回片段。
5. 展示 rerank 后 topN。
6. 展示最终答案和引用。
7. 展示 trace / 日志 / 反馈。

## 口播
这个 Demo 不是只看模型回答，而是看 RAG 链路是否可解释。每次回答都能回到 chunk_id、doc_id 和 page，低置信度或无证据时不强行回答。
```

### 4.2 Agent Demo 脚本

```markdown
# Agent Demo 脚本

## 场景
用户希望 Agent 帮忙查询订单、生成工单、调用内部工具或完成多步流程。

## 演示任务
1. 只读查询：低风险工具。
2. 参数缺失：Agent 追问补槽。
3. 工具失败：timeout / retry / fallback。
4. 高风险写操作：二次确认或人工审批。
5. 多轮上下文：验证状态隔离。

## 演示顺序
1. 用户输入目标。
2. 展示 Agent state。
3. 展示 tool selection。
4. 展示 tool input schema。
5. 展示工具结果。
6. 展示失败分支或人工确认。
7. 展示最终回复和审计日志。

## 口播
我不会把高风险业务动作完全交给模型自由执行。模型负责理解和生成候选动作，真正执行前要经过 schema、权限、规则、确认和审计。
```

### 4.3 工具平台 / MCP Demo 脚本

```markdown
# 工具平台 Demo 脚本

## 场景
企业有多个业务系统，希望 Agent 可以统一调用工具。

## 演示内容
1. 注册一个 OpenAPI 工具。
2. 注册一个 MCP server 工具。
3. 展示 Tool Registry 字段。
4. 展示权限和风险等级。
5. 展示调用日志。
6. 展示版本升级。

## 口播
这个平台的重点不是让模型多调几个接口，而是统一工具契约、权限、版本、审计和错误处理。
```

### 4.4 实时语音 Agent Demo 脚本

```markdown
# 实时语音 Agent Demo 脚本

## 场景
用户通过语音和 Agent 交互，Agent 需要理解、追问、调用工具并在高风险动作前确认。

## 演示任务
1. 正常语音问答：展示首响延迟和最终回答。
2. 用户打断：展示 barge-in 后停止当前输出。
3. ASR 低置信度：Agent 先澄清，不直接执行。
4. 高风险工具：提交、发送、删除等动作二次确认。
5. Trace 回放：展示 turn_id、partial/final transcript、confidence、tool_action、approval。

## 口播
实时语音 Demo 不是只证明能说话，而是证明低延迟、可打断、可确认和可回放。真正上线前，我会把 ASR 置信度、barge-in、工具动作、审批结果和失败降级都放进 trace。
```

补充展示：barge-in 不只是“前端停止播放”，还要证明后端取消语义正确。

```json
{"type":"assistant_audio_start","turn_id":"t2","response_id":"r8","seq":12}
{"type":"barge_in","turn_id":"t2","response_id":"r8","seq":13,"reason":"user_speech_detected"}
{"type":"response_cancelled","turn_id":"t2","response_id":"r8","seq":14}
{"type":"user_final_transcript","turn_id":"t3","text":"等等，先不要退款","confidence":0.93}
{"type":"tool_progress","run_id":"run_17","tool_name":"refund_order","status":"paused"}
{"type":"approval_required","run_id":"run_17","approval_id":"ap_09","risk_level":"high","action_summary":"退款 128 元"}
```

口播补充：

```text
实时语音里的打断要同时取消 TTS 播放、停止继续生成旧回答、避免旧工具动作继续执行；如果工具已经进入不可取消阶段，就进入 paused/approval_required，等待用户确认或人工接管。Demo 里我会展示 turn_id、response_id 和 seq，证明旧响应不会和新一轮用户意图串线。
```

### 4.5 Browser / Computer Use Demo 脚本

```markdown
# Browser Agent Demo 脚本

## 场景
Agent 在允许的网页里读取信息、填写表单或辅助完成流程。

## 演示任务
1. 只读浏览：展示 allowed_domains 和页面摘要。
2. 自动填写：展示 selector、输入字段和 action log。
3. 危险动作：submit / download / delete 触发确认。
4. 网页注入：页面诱导泄露信息时被策略拦截。
5. 回放审计：展示 step_id、url、selector、action、screenshot_id、approval_id。

## 口播
Browser Agent 的核心不是“自动点网页”，而是可控地观察和行动。网页内容只当 untrusted data，动作要经过域名白名单、动作白名单、风险确认和审计回放。
```

---

## 5. 截图和录屏清单

可以准备的截图：

| 截图 | 证明什么 | 脱敏注意 |
|---|---|---|
| 首页/问答页 | 项目不是纯口头描述 | 隐去用户和公司信息 |
| 文档管理页 | 有入库链路 | 隐去真实文档名 |
| 召回片段页 | 可解释检索 | 替换为示例文档 |
| 答案引用页 | citation 能落地 | 隐去内部链接 |
| 工具调用 trace | Agent 不是黑盒 | 隐去接口地址、token |
| Realtime event trace | 证明可打断、可确认、可回放 | 隐去录音、用户身份和原始语音 |
| Browser action replay | 证明 Computer Use 可审计 | 隐去 URL、cookie、账号和页面敏感字段 |
| LLMOps release diff | 证明灰度、门禁和回滚 | 隐去内部 app 名和租户名 |
| 评测表 | 有验证闭环 | 用样例问题替代真实问题 |
| 日志字段 | 有工程治理 | 隐去 user_id、tenant_id |
| 错误处理页 | 有失败兜底 | 隐去内部错误码映射 |

录屏建议：

```text
30 秒：只展示最核心闭环
90 秒：展示一次正常链路 + 一次 bad case/兜底
3 分钟：展示完整业务场景、trace、指标和边界
```

不要录：

- 真实客户数据。
- 内部生产环境地址。
- 密钥、token、账号。
- 未公开业务系统。
- 公司内部文档全文。

---

## 6. API 和 Tool Schema 证据

面试官问工程细节时，接口样例很有用。

### 6.1 RAG 问答接口示例

```json
{
  "query": "如何申请报销？",
  "tenant_id": "demo_tenant",
  "user_id": "demo_user",
  "top_k": 5,
  "stream": true
}
```

返回示例：

```json
{
  "answer": "根据制度文档，报销需要提交发票和审批单。",
  "citations": [
    {
      "doc_id": "policy_demo",
      "chunk_id": "chunk_012",
      "page": 3,
      "score": 0.86
    }
  ],
  "trace_id": "trace_demo_001",
  "latency_ms": 1280
}
```

面试口播：

> 我会让接口返回 trace_id 和 citations，因为线上排障和答案可信度都依赖这些字段。

### 6.2 Tool schema 示例

```json
{
  "name": "query_order_status",
  "description": "查询订单状态，只读工具",
  "risk_level": "read_only",
  "input_schema": {
    "type": "object",
    "properties": {
      "order_id": {
        "type": "string",
        "description": "订单 ID"
      }
    },
    "required": ["order_id"]
  },
  "auth": {
    "scope": "order:read"
  },
  "timeout_ms": 3000
}
```

高风险工具要额外准备：

```json
{
  "name": "refund_order",
  "risk_level": "high_risk_write",
  "requires_confirmation": true,
  "idempotency_key_required": true,
  "audit_required": true
}
```

### 6.3 A2A 委派 Task Contract 示例

多 Agent 项目不要只展示“上游 Agent 调了下游 Agent”，要展示委派合同、身份传递、scope 校验和 handoff audit。

```json
{
  "task_id": "task_demo_20260608_001",
  "correlation_id": "trace_demo_001",
  "caller_agent": "customer_service_agent",
  "delegate_agent": "refund_policy_agent",
  "user_subject": {
    "tenant_id": "demo_tenant",
    "user_id": "demo_user",
    "role": "support_operator"
  },
  "delegated_identity": "on_behalf_of_user",
  "capability": "refund_policy_check",
  "scope": ["order:read", "refund:policy_check"],
  "deadline_at": "2026-06-08T10:30:00+08:00",
  "risk_level": "read_only",
  "input": {
    "order_id": "order_demo_001",
    "question": "是否满足退款条件"
  },
  "result_schema": {
    "type": "object",
    "required": ["decision", "reason_codes", "evidence_refs"],
    "properties": {
      "decision": { "enum": ["eligible", "ineligible", "need_human"] },
      "reason_codes": { "type": "array", "items": { "type": "string" } },
      "evidence_refs": { "type": "array", "items": { "type": "string" } }
    }
  },
  "handoff_audit": {
    "handoff_decision": "accepted",
    "scope_check": "pass",
    "capability_check": "pass",
    "deadline_check": "pass",
    "approval_id": null
  }
}
```

面试口播：

> A2A 委派我不会只传一句自然语言任务。上游 Agent 必须带 task_id、调用方身份、被委派 capability、scope、deadline 和 result_schema。下游 Agent 先做 capability/scope/deadline 校验，结果必须按 schema 返回，handoff 决策写入审计。这样能避免 confused deputy，也能追责是哪一次委派越权。

---

## 7. 评测证据包

没有真实线上流量时，最重要的是把离线评测做扎实。

### 7.1 RAG 评测表

```markdown
| 问题 | 标准证据 | 召回 topK 是否命中 | 答案是否正确 | 引用是否支持 | bad case 类型 |
|---|---|---|---|---|---|
| 如何申请报销？ | policy_demo p3 | 是 | 是 | 是 | - |
| 离职后社保怎么处理？ | hr_demo p7 | 否 | 否 | 否 | 召回失败 |
```

指标：

- Recall@K：正确证据是否进入候选。
- Answer Accuracy：答案是否正确。
- Citation Accuracy：引用是否支持答案。
- Faithfulness：答案是否基于证据。
- No-answer Accuracy：无资料时是否拒答。

### 7.2 Agent 评测表

```markdown
| 任务 | 期望工具 | 实际工具 | 参数是否正确 | 是否成功 | 失败原因 | 是否需要人工 |
|---|---|---|---|---|---|---|
| 查询订单状态 | query_order_status | query_order_status | 是 | 是 | - | 否 |
| 申请退款 | refund_order | refund_order | 是 | 待确认 | 高风险写操作 | 是 |
```

指标：

- Task Success Rate。
- Tool Selection Accuracy。
- Parameter Accuracy。
- Step Count。
- Human Handoff Rate。
- Tool Error Rate。

### 7.3 模型网关评测表

```markdown
| 业务 | 模型 | 成本 | P95 | 成功率 | fallback 率 | 备注 |
|---|---|---|---|---|---|---|
| FAQ | small_model | 低 | 800ms | 96% | 1% | 简单问题 |
| 合同问答 | strong_model | 高 | 2600ms | 91% | 4% | 高风险 |
```

指标：

- 单次请求成本。
- TTFT / P95 / P99。
- 成功率。
- fallback 率。
- 限流率。
- 用户满意度或人工评分。

### 7.4 Data Agent 评测表

```markdown
| question | metric_id | SQL_readonly_pass | ACL_pass | result_verified | chart_explanation_pass |
|---|---|---|---|---|---|
| 上周各渠道转化率是多少？ | conversion_rate_v3 | 是 | 是 | 是 | 是 |
```

端到端证据样例：

```json
{
  "trace_id": "trace_data_agent_041",
  "question": "为什么上周 GMV 比前一周下降了？按渠道拆一下并画图",
  "intent": "attribution_analysis",
  "metric_contract": {
    "metric_id": "gmv_paid_amount",
    "version": "v5",
    "owner": "trade_data_team",
    "time_window": "2026-05-25~2026-05-31",
    "baseline_window": "2026-05-18~2026-05-24"
  },
  "tool_plan": [
    "resolve_metric_contract",
    "generate_sql",
    "validate_sql_ast",
    "execute_readonly_query",
    "build_chart",
    "generate_grounded_explanation"
  ],
  "sql_validation": {
    "readonly_pass": true,
    "multi_statement_pass": true,
    "acl_pass": true,
    "tenant_filter_injected": true,
    "time_range_required": true,
    "limit_injected": true,
    "explain_scan_rows": 238000,
    "cost_pass": true
  },
  "result_check": {
    "sample_size": 18420,
    "result_reconciled_with_metric_platform": true,
    "null_rate_checked": true,
    "outlier_checked": true
  },
  "chart_contract": {
    "chart_type": "bar",
    "x": "channel",
    "y": "gmv_delta_contribution",
    "unit": "CNY",
    "must_show_metric_version": true,
    "chart_explanation_pass": true
  },
  "explanation_guardrail": {
    "allowed_claim": "渠道 A 对 GMV 下降贡献最大",
    "blocked_claim": "渠道 A 是 GMV 下降的根本原因",
    "reason": "只有分解贡献度，没有实验或外部事件证据"
  },
  "final_answer_evidence": {
    "shown_to_user": ["metric_id", "version", "time_window", "sql_summary", "chart", "sample_size", "confidence"],
    "next_query": "继续按品类和新老用户拆分渠道 A"
  }
}
```

面试口播：

```text
这个 Demo 不是只证明能查数，而是证明“查数 -> 归因 -> 图表 -> 解释”每一步都有证据。尤其解释阶段只说贡献度和候选原因，不把相关性包装成因果。
```

### 7.5 发布门禁快照

面试官继续追问“你怎么证明这次迭代可以上线”时，不要只说“跑过评测”。用一张门禁快照把质量、延迟、成本、安全和回滚证据串起来。

```markdown
| gate_id | change_id | dataset_version | judge_version | decision | block_reason | rollback_version |
|---|---|---|---|---|---|---|
| gate_2026_06_demo | change_012 | data_agent_v3 | judge_v2 | pass | - | prompt_v11 |
```

| 门禁项 | 通过标准 | 阻断例子 | 可展示证据 |
|---|---|---|---|
| 质量 | 核心 case 通过，关键 bad case 不回归 | RAG 引用不支持答案、Data Agent 指标口径错 | eval summary、失败 case |
| 延迟 | p95 / first_token 不超过预算 | 首 token 超预算，SSE 卡住无进度事件 | latency report、event trace |
| 成本 | cost_per_successful_task 在预算内 | 重试或上下文过长导致单任务成本飙升 | cost report、token breakdown |
| 安全 | 越权、危险工具、注入样例通过 | 未授权 SQL、退款/删除无需确认 | safety case、approval log |
| 可回滚 | 有 rollback_version 和 owner | 只改 prompt 无版本记录 | change record、rollback plan |

口播：

```text
我不会把一次 prompt 或工具 schema 修改直接推上去。每次变更会绑定 change_id，跑固定 dataset 和历史 bad case，看质量、延迟、成本和安全门禁；只要核心 case 失败、越权非 0、成本/延迟超预算，或者没有回滚版本，就 block。
```

---

## 8. Bad Case 证据包

每个项目至少准备 3 个 bad case。

模板：

```markdown
## Bad Case 名称

### 现象

用户问：

系统回答：

问题影响：

### 定位

- trace_id：
- 召回片段：
- tool call：
- 日志字段：
- 评测样例：

### 根因

- 解析：
- 切分：
- 召回：
- rerank：
- prompt：
- 工具：
- 权限：
- 模型：

### 修复

1.
2.
3.

### 验证

- 修复前：
- 修复后：
- 是否加入回归集：

### 面试口播

这个 bad case 的价值是让我发现问题不在模型本身，而在 [具体链路]。修复后我没有只看单个样例，而是把它加入回归集，避免后续版本再退化。
```

---

## 9. 代码目录说明

即使不能展示完整代码，也要能讲清目录结构。

RAG 项目目录示例：

```text
app/
  api/
    chat.py
    documents.py
  ingestion/
    parser.py
    chunker.py
    embedding_worker.py
  retrieval/
    hybrid_retriever.py
    reranker.py
    citation_checker.py
  services/
    llm_client.py
    prompt_builder.py
  eval/
    rag_eval.py
    badcase_runner.py
  infra/
    logging.py
    config.py
    rate_limit.py
```

Agent 项目目录示例：

```text
app/
  graph/
    state.py
    nodes.py
    edges.py
  tools/
    registry.py
    schemas.py
    executor.py
  guardrails/
    auth.py
    risk.py
    confirmation.py
  traces/
    tracer.py
    audit_log.py
  api/
    agent.py
```

口播：

> 代码结构上我会把 API、编排、工具、治理和评测拆开。这样面试官追问某个问题时，我能定位到具体模块，而不是只说一个大函数。

---

## 10. 脱敏和合规边界

可以展示：

- 自己写的非敏感代码片段。
- 脱敏后的接口样例。
- 自己整理的架构图。
- 示例文档和模拟数据。
- 离线评测表结构。
- 已公开或个人项目的 README。

不要展示：

- 公司内部源码。
- 真实客户数据。
- 内部接口域名。
- 生产日志原文。
- 密钥、token、cookie。
- 未公开业务策略。
- 同事或客户个人信息。

如果面试官要求看细节，可以说：

```text
真实业务数据和内部代码我不能直接展示，但我可以用脱敏结构说明。我这里准备了接口字段、链路图、评测表结构和 bad case 归因方式，能说明我是怎么做和怎么排查的。
```

这句话很重要：既保护合规，也展示工程成熟度。

---

## 11. 面试问法和回答

### 11.1 你有 Demo 吗？

回答：

```text
有一个脱敏版 Demo / 原型。我主要用它展示完整链路，不展示真实业务数据。RAG 部分可以看到文档入库、召回片段、引用和评测；Agent 部分可以看到工具选择、参数校验、失败分支和审计日志。
```

### 11.2 你这个是不是只做了 Demo？

回答：

```text
这个项目目前是 [Demo/内部试点/小范围上线] 阶段，我不会把它夸大成大规模生产。它已经验证了 [核心链路/评测/权限/日志/工具调用]，离生产还需要补 [灰度/监控/成本/回滚/更完整权限]。我准备这个项目时，也重点整理了从 Demo 到生产要补的工程项。
```

### 11.3 怎么证明是你做的？

回答：

```text
我可以从三个层面说明。第一是我负责的模块边界，比如 [chunk/rerank/tool schema/FastAPI/SSE]；第二是具体实现细节，比如接口字段、状态字段、日志字段；第三是问题处理记录，比如某个 bad case 的 trace、根因和修复验证。
```

### 11.4 没有线上指标怎么办？

回答：

```text
没有大规模线上流量我会如实说明。我主要用离线评测和小范围试用反馈验证，包括问题集、正确证据、Recall@K、引用准确率、人工验收和 bad case 回归。如果后续上线，会补用户反馈、无答案率、P95、token 成本和转人工率。
```

### 11.5 你能展示代码吗？

回答：

```text
如果是个人项目或可公开代码，我可以展示 README 和核心模块。如果是公司项目，我不能展示内部源码，但可以讲脱敏后的目录结构、接口设计、关键状态字段和异常处理逻辑。
```

---

## 12. 作品集按公司风格调整

| 公司/方向 | 作品集重点 | 不要突出 |
|---|---|---|
| 腾讯 | 用户体验、bad case、线上兜底、反馈闭环 | 只展示框架名 |
| 阿里/阿里云 | 平台化、多租户、权限、工具注册、灰度治理 | 单点脚本 Demo |
| 百度 | RAG/搜索、效果评估、trace、后端稳定性 | 只讲前端演示 |
| 字节 | 指标、实验、性能成本、代码实现、快速迭代 | 长篇背景故事 |
| 美团/京东 | 业务流程、客服/交易风险、ROI、人工接管 | 只讲模型效果 |
| 云厂商/ToB | 私有化、合规、审计、交付、运维 | 忽略数据边界 |
| 初创/AI 产品 | MVP、端到端交付、Demo 到生产清单 | 过重平台架构 |

---

## 13. 上场前证据包验收

至少准备 1 个项目证据包，最好准备 2 个：

```text
主项目：最贴合 JD
备项目：能展示后端/工具/平台/代码能力
```

验收表：

| 项目 | raw_score | grade | display | close_level | A 档表现 | B 档风险 | C 档风险 |
|---|---|---|---|---|---|---|---|
| README | 1-5 | A/B/C | 作品集展示 | 关闭/待复测/高风险 | 背景、职责、架构、指标、边界清楚 | 有说明但偏散 | 只有一句项目介绍 |
| 架构图 | 1-5 | A/B/C | 作品集展示 | 关闭/待复测/高风险 | 能讲入口、核心链路、治理、兜底 | 只有主链路 | 没有图 |
| Demo | 1-5 | A/B/C | 作品集展示 | 关闭/待复测/高风险 | 能展示正常链路和异常兜底 | 只能展示正常问答 | 没有可演示内容 |
| API/schema | 1-5 | A/B/C | 作品集展示 | 关闭/待复测/高风险 | 有请求、响应、字段解释 | 只有口头描述 | 说不清接口 |
| 评测 | 1-5 | A/B/C | 作品集展示 | 关闭/待复测/高风险 | 有样例、指标、bad case | 只有主观评价 | 没有验证 |
| 代码目录 | 1-5 | A/B/C | 作品集展示 | 关闭/待复测/高风险 | 能说明模块职责 | 只知道大概 | 说不清自己写哪 |
| 脱敏 | 1-5 | A/B/C | 作品集展示 | 关闭/待复测/高风险 | 合规边界清楚 | 有意识但没准备话术 | 乱展示敏感信息 |
| 口播 | 1-5 | A/B/C | 作品集展示 | 关闭/待复测/高风险 | 2 分钟能讲清 | 需要提示 | 讲散 |

口径说明：作品集验收也使用 `41-面试官评分Rubric与回答自检表.md` 的 `raw_score / grade / display / close_level`，避免和模拟面评分拆成两套标准。

最低上场标准：

- [ ] 1 个 README。
- [ ] 1 张架构图。
- [ ] 1 个 Demo 脚本。
- [ ] 3 个接口或 schema 样例。
- [ ] 1 张评测表。
- [ ] 3 个 bad case。
- [ ] 1 份脱敏说明。
- [ ] 1 个 2 分钟口播。

---

## 14. 最终速背

1. 项目证据包的目标不是炫耀，而是证明“项目真实、我做过、能追问、懂边界”。
2. README 要讲背景、职责、架构、指标、bad case 和生产化边界。
3. Demo 要展示正常链路、异常兜底、trace 和评测，不只是问答效果。
4. 没有线上指标可以讲离线评测、小范围反馈和下一步上线指标，不能编造生产数据。
5. 公司项目必须脱敏，不能展示内部源码、客户数据、token 和真实生产日志。
6. 面试官问“是不是 Demo”时，要诚实说明阶段，并补“从 Demo 到生产还要补什么”。
7. 最强证据不是截图，而是你能讲清接口字段、状态字段、日志字段、bad case 根因和修复验证。
