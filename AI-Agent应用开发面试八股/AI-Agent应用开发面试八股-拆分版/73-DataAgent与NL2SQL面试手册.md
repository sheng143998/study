# 73-Data Agent 与 NL2SQL 面试手册

> 目标：补齐自然语言查数、NL2SQL、数据分析 Agent、指标问答和图表生成相关面试题。  
> 核心观点：Data Agent 不是“让模型随便写 SQL”，而是一个受权限、指标口径、SQL 校验、查询成本和结果解释约束的数据应用系统。

适用场景：

- 面试官问：设计一个数据分析 Agent。
- 面试官问：自然语言转 SQL 怎么做？
- 面试官问：怎么防止危险 SQL、越权查库和扫全表？
- 面试官问：指标口径不一致怎么办？
- 面试官问：生成图表和结论怎么保证准确？
- 面试官问：怎么评估 NL2SQL / Data Agent 的效果？

配合文件：

- `09-大厂系统设计与项目表达.md`：已有数据分析 Agent 简版。
- `23-手撕题强化版.md`：SQL 只读校验和 limit 小题。
- `71-AI-Agent安全风控与合规面试手册.md`：Data Agent 安全边界。
- `54-项目指标与BadCase量化手册.md`：指标和 bad case。
- `67-系统设计变体与约束应对手册.md`：权限、高并发、成本、私有化等约束变体。

---

## 1. Data Agent 是什么

30 秒回答：

```text
Data Agent 是让用户用自然语言完成查数、分析和解释的数据应用。它通常会理解用户问题，选择数据源和指标口径，生成 SQL 或调用 BI/指标平台，校验查询安全，执行查询，生成图表和解释，并记录 trace 方便审计和复盘。
```

和普通 Chatbot 的区别：

| 普通 Chatbot | Data Agent |
|---|---|
| 主要回答文本 | 需要查询真实数据 |
| 可以只靠模型知识 | 必须接数据库/指标平台 |
| 错误影响较低 | 错误结论可能影响业务决策 |
| 主要看回答质量 | 还要看 SQL 正确性、权限、安全、口径 |

速背句：

> Data Agent 的难点不是生成 SQL 这一行，而是理解业务指标、选对表、控权限、防危险 SQL、解释结果并能审计。

---

## 2. 系统设计总架构

白板架构：

```text
用户问题
  -> 意图识别
  -> 权限识别
  -> 指标/口径解析
  -> Schema Linking
  -> SQL / Tool Plan 生成
  -> SQL AST 校验
  -> 查询执行
  -> 结果校验
  -> 图表生成
  -> 结论解释
  -> Trace / 审计 / 反馈
```

模块拆分：

| 模块 | 作用 |
|---|---|
| Intent Router | 判断是查数、解释指标、生成图表还是闲聊 |
| Metadata / Catalog | 表、字段、血缘、指标口径、样例值、owner |
| Schema Retriever | 根据问题召回相关表和字段 |
| Metric Resolver | 把“GMV/转化率/活跃用户”等映射到统一口径；字段包括 metric_id、version、owner、grain、formula、source_priority、deprecated_at |
| SQL Generator | 生成 SQL 或查询计划 |
| SQL Validator | AST 校验、只读、权限、limit、cost |
| Query Executor | 只读执行、超时、资源隔离 |
| Result Interpreter | 解释数据、生成摘要、提示不确定性 |
| Chart Builder | 选择图表类型并生成配置 |
| Audit / Trace | 记录问题、SQL、表、耗时、结果摘要、用户 |

面试开场：

```text
我会把 Data Agent 拆成两条线：一条是语义理解和 SQL 生成，另一条是数据治理和安全执行。前者决定能不能查对，后者决定能不能安全上线。
```

---

## 3. NL2SQL 核心流程

### 3.1 直接生成 SQL 的问题

低分做法：

```text
用户问题 -> LLM -> SQL -> 执行
```

风险：

- 选错表。
- 字段名幻觉。
- 指标口径错。
- 生成危险 SQL。
- 越权查库。
- 查询太慢。
- 结果解释过度。

### 3.2 更稳的流程

```text
用户问题
  -> 问题改写和槽位抽取
  -> 召回相关指标、表、字段和样例
  -> 生成 SQL 草稿
  -> SQL 语法检查
  -> AST 安全检查
  -> 权限和成本检查
  -> dry run / explain
  -> 执行
  -> 结果解释
```

关键点：

1. 不把全库 schema 全塞进 prompt。
2. 先根据问题召回相关表和字段。
3. 指标优先走指标平台或语义层。
4. SQL 执行前必须校验。
5. 复杂问题可以先让模型生成 plan，再生成 SQL。

---

## 4. Schema Linking

Schema Linking 是把用户问题中的业务词映射到数据库里的表、字段、指标和过滤条件。

例子：

```text
用户说：上周新用户付费转化率

可能映射：
新用户 -> user_register_time in last week
付费 -> payment_order.status = paid
转化率 -> paid_users / new_users
上周 -> date between ...
```

高分回答：

```text
NL2SQL 的关键不是让模型记住所有表结构，而是做好 schema linking。我要维护数据目录、表字段描述、指标口径、样例值和业务别名，先召回候选 schema，再让模型在候选范围内生成 SQL。
```

实现方式：

- 表名、字段名、字段描述入向量库。
- 指标口径单独维护。
- 业务别名词典。
- 样例值帮助判断字段。
- 常用查询模板。
- 数据 owner 和权限信息。

---

## 5. 指标口径治理

Data Agent 最容易翻车的是指标口径。

常见问题：

| 用户说法 | 可能歧义 |
|---|---|
| 活跃用户 | DAU、WAU、MAU、登录、访问、下单 |
| GMV | 支付金额、下单金额、退款后金额 |
| 转化率 | PV->下单、UV->支付、注册->付费 |
| 新用户 | 注册新、首单新、首次访问 |
| 留存 | 次日、7 日、自然日、滚动窗口 |

防守方案：

1. 建统一指标字典。
2. 指标有 owner、公式、时间粒度、适用场景。
3. 高歧义问题先澄清。
4. 重要指标优先走指标平台，不让模型自由拼公式。
5. 返回结果时展示口径说明。
6. 口径变更要版本化。

面试回答：

```text
我不会让模型自由定义 GMV 或转化率。核心指标应该来自指标平台或语义层，包含公式、过滤条件、时间粒度和 owner。用户问题有歧义时先澄清，返回结果时展示口径，避免模型查出一个看似正确但业务不可用的数。
```

---

## 6. SQL 安全

### 6.1 只读和危险语句

基础策略：

- 数据库账号默认只读。
- SQL AST 校验。
- 禁止 `insert/update/delete/drop/alter/truncate`。
- 禁止多语句执行。
- 禁止访问系统表和敏感表。
- 自动加 `limit`。
- 设置超时和扫描行数上限。

标准回答：

```text
SQL 不能由模型生成后直接执行。我要先用 parser 做 AST 校验，只允许 select/with，禁止写操作和危险函数；再做表字段白名单、行列权限、limit、超时和 explain cost 检查。执行账号也必须是只读账号。
```

### 6.2 权限控制

权限维度：

| 维度 | 示例 |
|---|---|
| 库权限 | 用户能访问哪些库 |
| 表权限 | 能访问订单表但不能访问薪资表 |
| 列权限 | 手机号、身份证、地址要脱敏 |
| 行权限 | 只能查自己部门/租户的数据 |
| 指标权限 | 某些经营指标只给管理层 |
| 操作权限 | 只能读，不能写 |

回答：

```text
权限不能只在前端控制。Data Agent 要把用户身份映射到库、表、列、行和指标权限，生成 SQL 前过滤 schema，执行前再校验 SQL 是否访问了无权限对象，结果返回前做脱敏和聚合。
```

### 6.3 查询成本控制

风险：

- 扫全表。
- 大 join。
- 无时间范围。
- group by 高基数字段。
- 频繁查询打爆数仓。

治理：

- 强制时间范围。
- 默认 limit。
- explain cost。
- 超时取消。
- 查询队列。
- 结果缓存。
- 热门指标预聚合。
- 高成本查询二次确认。

---

## 7. 结果解释和图表

### 7.1 结果解释风险

Data Agent 不是查出数据就结束，还要避免错误解释。

常见风险：

- 把相关性说成因果。
- 忽略样本量太小。
- 忽略异常值。
- 指标环比/同比口径错。
- 时间粒度不一致。
- 图表类型误导。

回答模板：

```text
结果解释阶段我会让模型基于查询结果和指标口径做总结，但不能过度推断。比如只能说“本周转化率下降”，不能直接说“因为活动质量变差”，除非有额外证据。关键分析要展示 SQL、口径、时间范围和样本量。
```

### 7.2 图表选择

| 问题 | 图表 |
|---|---|
| 趋势 | 折线图 |
| 分类对比 | 柱状图 |
| 占比 | 饼图/堆叠图 |
| 分布 | 直方图/箱线图 |
| 漏斗转化 | 漏斗图 |
| 地域分布 | 地图 |

图表输出要包含：

- chart_type。
- x/y 字段。
- 时间范围。
- 指标单位。
- 口径说明。
- 数据来源。

---

## 8. 评测体系

### 8.1 NL2SQL 评测

| 指标 | 含义 |
|---|---|
| SQL Exact Match | SQL 是否和标准答案一致 |
| Execution Accuracy | 执行结果是否正确 |
| Schema Linking Accuracy | 表字段是否选对 |
| Metric Accuracy | 指标口径是否正确 |
| Security Pass Rate | 是否通过安全校验 |
| Latency / Cost | 查询耗时和资源成本 |

高分回答：

```text
NL2SQL 不能只看生成 SQL 文本是否一致，因为等价 SQL 可能很多。更重要的是执行结果是否正确、schema linking 是否正确、指标口径是否符合业务定义，以及是否通过安全和成本校验。
```

### 8.2 评测集构造

问题类型：

1. 简单查询。
2. 多表 join。
3. 聚合统计。
4. 时间范围。
5. 指标口径。
6. 权限边界。
7. 危险 SQL。
8. 无法回答。
9. 图表生成。
10. 解释类问题。

样例：

```markdown
| 问题 | 期望表 | 期望字段 | 指标口径 | 是否可执行 | 预期结果 |
|---|---|---|---|---|---|
| 上周新用户付费转化率 | user, order | user_id, pay_time | paid_users/new_users | 是 | 数值 |
| 查所有用户手机号 | user | phone | 敏感字段 | 否 | 拒绝/脱敏 |
```

---

## 9. Bad Case

### 9.1 常见 bad case

| Bad case | 根因 | 修复 |
|---|---|---|
| 选错表 | schema 描述不清 | 补数据目录和样例值 |
| 指标口径错 | 没接指标平台 | 指标字典和语义层 |
| SQL 语法错 | 模型生成不稳定 | SQL parser + 修复重试 |
| 危险 SQL | 无 AST 校验 | 只读账号 + AST 白名单 |
| 查太慢 | 无 limit/时间范围 | explain cost + 超时 |
| 越权查数 | schema 未按权限过滤 | 权限下推到表/列/行 |
| 解释过度 | 模型过度推理 | 结论只基于数据和口径 |

### 9.2 Bad case 口播

```text
我们遇到过一个指标口径 bad case：用户问“本周转化率”，模型按访问到下单算，但业务实际看注册到支付。这个问题不是 SQL 语法错，而是指标语义错。后来我们把核心指标接入指标字典，包含公式、过滤条件和 owner；高歧义指标先澄清，结果里展示口径说明。验证上把这类问题加入评测集，看 metric accuracy。
```

---

## 10. 系统设计题模板

题目：

> 设计一个企业内部 Data Agent，用户可以用自然语言查业务数据、生成图表，并解释指标变化。

回答结构：

```text
1. 澄清需求
   - 数据源：数仓 / MySQL / BI / 指标平台
   - 用户：运营 / 产品 / 管理者
   - 权限：部门 / 租户 / 角色
   - 输出：表格 / 图表 / 解释

2. 核心链路
   - intent router
   - metadata catalog
   - schema linking
   - metric resolver
   - SQL generator
   - SQL validator
   - query executor
   - result interpreter

3. 治理
   - 只读账号
   - AST 校验
   - 权限下推
   - 查询成本控制
   - 日志审计

4. 评测
   - execution accuracy
   - schema linking accuracy
   - metric accuracy
   - security pass rate
   - latency / cost
```

白板图：

```text
User
  -> API / Auth
  -> Intent Router
  -> Schema Retriever + Metric Catalog
  -> SQL Generator
  -> SQL Validator / Permission / Cost
  -> Readonly Query Executor
  -> Result Interpreter / Chart Builder
  -> Audit Log / Feedback / Eval
```

---

## 11. 面试高频问答

### 11.1 NL2SQL 怎么做？

```text
我会先做问题理解和 schema linking，召回相关表、字段、指标和样例，再让模型在候选 schema 内生成 SQL。SQL 生成后要过语法检查、AST 安全检查、权限检查、limit 和 cost 检查，最后执行并解释结果。
```

### 11.2 怎么防止模型生成危险 SQL？

```text
首先数据库账号只读；其次 SQL 进入执行前用 parser/AST 校验，只允许 select/with，禁止写操作、多语句和危险函数；再做表字段白名单、行列权限、limit、超时和 explain cost。不能只靠 prompt 要求模型别写危险 SQL。
```

代码面/深挖时可以口述这个伪代码：

```python
def validate_sql(sql, user_ctx, catalog):
    ast = parse_sql(sql)

    if ast.has_multiple_statements():
        raise Reject("multi statement")
    if ast.root_type not in {"select", "with"}:
        raise Reject("readonly only")
    if ast.has_write_operation() or ast.has_dangerous_function():
        raise Reject("dangerous sql")

    tables = ast.tables()
    columns = ast.columns()
    if not catalog.allow_tables(user_ctx, tables):
        raise Reject("table permission denied")
    if not catalog.allow_columns(user_ctx, columns):
        raise Reject("column permission denied")

    ast = ensure_tenant_filter(ast, user_ctx.tenant_id)
    ast = ensure_time_range(ast, default_days=30)
    ast = ensure_limit(ast, max_limit=1000)

    cost = explain_cost(ast.to_sql())
    if cost.scan_rows > 1_000_000 or cost.timeout_risk:
        raise NeedApproval("high cost query")

    return ast.to_sql(), {"tables": tables, "columns": columns, "cost": cost}
```

速背句：

```text
代码主干不是“正则拦截 delete”，而是 parser -> AST -> 只读白名单 -> 表字段权限 -> tenant/time/limit 注入 -> explain cost -> 只读执行。
```

### 11.3 指标口径不一致怎么办？

```text
核心指标不能让模型自由定义，要接指标平台或语义层，维护指标公式、过滤条件、时间粒度、owner 和版本。用户问题有歧义时先澄清，返回结果时展示口径说明。
```

### 11.4 如何评估 Data Agent？

```text
分几层评估：schema linking 是否选对表字段，SQL 执行结果是否正确，指标口径是否正确，安全校验是否通过，查询耗时和成本是否可接受，结果解释是否忠于数据。不能只看模型回答像不像。
```

### 11.5 怎么处理模型解释错？

```text
解释阶段要绑定 SQL、结果、口径和时间范围。模型不能做没有证据的因果推断。高风险结论要给出不确定性，必要时让用户查看 SQL 和数据明细，或转人工分析。
```

### 11.6 Data Agent 和 RAG 什么关系？

```text
Data Agent 可以用 RAG 检索数据字典、指标口径、表字段说明和历史查询样例，但最终数值要来自数据库或指标平台。RAG 帮助理解 schema 和业务语义，SQL/工具负责查真实数据。
```

---

## 12. 公司风格下的回答重点

| 公司/方向 | 重点 |
|---|---|
| 百度 | 搜索/RAG + Data Agent，强调 schema linking、trace、效果评估 |
| 阿里/阿里云 | 指标平台、数据治理、多租户、权限、审计、平台化 |
| 字节 | 指标实验、查询性能、成本、A/B 分析、快速迭代 |
| 腾讯 | 业务用户体验、结果可信、权限和转人工 |
| 美团/京东 | 经营指标、交易数据、客服/运营查数、安全和稳定 |
| ToB/云厂商 | 私有化、客户数据边界、审计、可交付 |

---

## 13. 项目表达模板

如果你做过数据相关项目：

```text
这个项目不是让模型直接连库裸查，而是做了受控 Data Agent。用户问题先做意图识别和 schema linking，相关表字段来自数据目录和指标字典；SQL 生成后经过 AST 只读校验、权限校验、limit 和 explain cost；执行结果再生成表格、图表和解释。我们评估时看 execution accuracy、metric accuracy、security pass rate 和查询延迟。
```

如果没做过完整 Data Agent：

```text
我没有完整上线过 Data Agent，但做过 RAG/Tool/后端相关能力。按我的理解，Data Agent 的生产难点主要是指标口径、SQL 安全、权限和结果解释。真正上线时我会先接指标平台和数据目录，只读执行，做 AST 校验和安全评测，再逐步开放复杂分析能力。
```

---

## 14. 上场前检查

- [ ] 能解释 Data Agent 和普通 Chatbot 区别。
- [ ] 能画 NL2SQL 核心链路。
- [ ] 能解释 schema linking。
- [ ] 能解释指标口径治理。
- [ ] 能说 SQL AST 校验、只读账号、limit、超时、cost。
- [ ] 能讲表/列/行/指标权限。
- [ ] 能解释结果解释和图表生成风险。
- [ ] 能说 execution accuracy、metric accuracy、security pass rate。
- [ ] 能准备 1 个指标口径或 SQL 安全 bad case。

---

## 15. 最终速背

## 16. 语义层、指标平台与 Data Agent 生产化追问

近期 Data Agent / ChatBI / BA Agent 类 JD 会把“自然语言查数”继续追到语义层和指标治理。核心不是模型会不会写 SQL，而是能不能保证查出来的是同一个业务口径。

生产化链路：

```text
用户问题
  -> 意图识别
  -> 指标/维度/时间范围抽取
  -> 语义层匹配
  -> schema linking
  -> SQL 生成
  -> AST 安全校验
  -> 权限过滤
  -> 查询执行
  -> 结果解释
  -> 图表生成
  -> 口径和 SQL 展示
```

| 追问 | 高分回答要点 |
|---|---|
| 指标口径冲突怎么办？ | 指标平台/语义层是权威来源，模型只做匹配和澄清，不临时发明口径 |
| 表很多怎么选表？ | 先按业务域、指标、维度、时间字段召回候选 schema，再让模型在候选集里选择 |
| 用户问题不完整怎么办？ | 先澄清时间范围、指标定义、过滤条件，不直接生成 SQL |
| 如何防止“看起来对”的错图？ | 图表必须带 SQL、指标口径、时间范围、过滤条件和不确定性说明 |
| 如何上线灰度？ | 先只读、只查白名单指标，人工审核通过后逐步开放更多数据域 |

项目证据槽：

```text
指标口径来源：
schema 数量和业务域：
SQL 安全校验规则：
权限模型：
典型 bad case：
评测指标：
图表可信度校验：
```

### 16.1 Metric Contract 模板

面试官追问“指标层具体怎么建”时，不要只说“接指标平台”，要能给出合同字段。

```markdown
# Metric Contract

- metric_id:
- metric_name:
- owner:
- business_domain:
- formula:
- filters:
- time_grain:
- dimensions:
- version:
- permission:
- sample_sql:
- test_cases:
  - input:
  - expected_sql:
  - expected_result:
- change_log:
```

最小样例：

```text
metric_id: pay_conversion_rate
formula: paid_users / new_users
time_grain: day/week
dimensions: channel, region
permission: product_manager_read
sample_sql: select paid_users / nullif(new_users, 0) ...
test_cases: 上周新用户付费转化率 -> 必须使用注册到支付口径
```

完整样例：

```yaml
metric_id: pay_conversion_rate
metric_name: 新用户付费转化率
business_domain: growth
owner: growth_data_pm
version: v3
status: active
description: 在指定时间窗内，完成注册的新用户中产生首笔支付的比例。
formula: paid_new_users / nullif(registered_new_users, 0)
numerator:
  table: dws_user_payment_day
  field: paid_new_users
  filters:
    - is_new_user = 1
    - first_pay_time between window_start and window_end
denominator:
  table: dws_user_register_day
  field: registered_new_users
  filters:
    - register_time between window_start and window_end
time_grain: day/week/month
dimensions:
  - channel
  - region
  - campaign_id
default_time_window: last_7_days
permission:
  roles: [growth_pm, data_analyst]
  row_scope: tenant_id + region_scope
  sensitive_columns: []
source_priority:
  - metric_platform
  - semantic_layer
  - warehouse_view
sql_policy:
  readonly: true
  max_rows: 1000
  timeout_ms: 3000
  require_partition_filter: true
chart_policy:
  default_chart: line
  allow_chart: [line, bar, table]
  must_show: [metric_id, version, time_window, filters]
test_cases:
  - input: 上周各渠道新用户付费转化率
    expected_metric_id: pay_conversion_rate
    expected_grain: week
    expected_dimensions: [channel]
    expected_sql_contains: [dws_user_payment_day, dws_user_register_day, is_new_user]
    expected_chart: bar
change_log:
  - v2 -> v3: 分母从访问用户改为注册新用户；2026-05-18 生效
```

为什么这个细节重要：

```text
如果没有 Metric Contract，模型可能生成一条能跑的 SQL，但业务口径是错的。Contract 把公式、时间粒度、维度、权限、SQL 约束、图表约束和测试用例固定下来，让 Data Agent 只做匹配和解释，不临时发明指标。
```

两个高频 bad case：

| Bad Case | 现象 | 修复 | 评测 |
|---|---|---|---|
| 同名指标冲突 | “转化率”在运营和交易团队口径不同 | Metric Contract 加 owner、domain、version，高歧义先澄清 | metric accuracy |
| 图表口径错但 SQL 能跑 | SQL 正确返回了数，但图表按错误时间粒度展示 | 图表配置绑定 time_grain、filters、sample_size 和口径说明 | chart grounding pass rate |

### 16.2 归因分析 Agent 题

面试官问“用户问为什么 GMV 下降，Data Agent 怎么做”时，不要直接让模型编原因。

回答框架：

```text
先锁定指标口径和时间窗
  -> 找基线和对照周期
  -> 按维度拆分：渠道、地区、品类、用户层级、流量来源
  -> 检测异常贡献度
  -> 生成候选原因，不做过度因果判断
  -> 输出 SQL、口径、图表、候选解释、置信度和下一步验证
```

最小输出：

| 字段 | 含义 |
|---|---|
| metric_contract | 使用哪个指标口径和版本 |
| baseline_window | 对照周期 |
| breakdown_dims | 拆分维度 |
| contribution | 每个维度对下降的贡献 |
| candidate_reason | 候选原因 |
| confidence | 置信度，不等于因果结论 |
| next_query | 下一步验证 SQL 或分析 |

端到端样例：

| 阶段 | 输入/输出 | 证据字段 | 防错点 |
|---|---|---|---|
| 查数 | “为什么上周 GMV 下降？” -> 先确认 GMV 口径和时间窗 | metric_id、version、window | 口径不清先澄清 |
| 基线 | 查询上周、本周、去年同期或前 4 周均值 | baseline_window、compare_window | 避免只看单点波动 |
| 归因 | 按渠道、地区、品类、用户层级拆贡献 | breakdown_dim、contribution_pct | 只说贡献，不直接说因果 |
| 图表 | 生成趋势图 + 贡献度柱状图 | chart_type、grain、filters | 图表必须展示口径和时间范围 |
| 解释 | 输出候选原因、置信度、下一步验证 | confidence、next_query | 需要实验或外部证据才说因果 |

示例输出：

```text
结论：按 gmv_v2 口径，本周 GMV 环比下降 8.4%。贡献拆分显示华东区下降贡献 46%，其中 paid_search 渠道贡献 31%。这只能说明相关性，候选原因可能是投放减少、库存不足或活动结束。下一步需要查 paid_search spend、库存缺货率和活动日历。
```

60 秒回答：

```text
归因分析 Agent 不能直接回答“因为某某原因”。我会先确认 GMV 口径和时间窗，再做同比/环比基线，按渠道、地区、品类、用户层级等维度拆分贡献度。输出时必须带 SQL、口径、样本量、候选原因和置信度，并明确这是相关性分析，真正因果还要结合实验、投放、库存、活动和外部事件验证。
```

## 17. Data Agent 面经高频 Bad Case 补丁

| Bad Case | 根因 | 修复 |
|---|---|---|
| SQL 能跑但口径错 | 模型选错指标或聚合方式 | 引入语义层，指标必须由权威口径映射 |
| 选错表 | schema 太多，表描述不清 | 表/字段增加业务描述，先召回候选 schema |
| 越权查数据 | 只做了 SQL 语法校验 | 行列权限、租户过滤、敏感字段脱敏 |
| 图表误导 | 图表类型不适合或时间粒度错 | 图表规则、时间粒度校验、展示 SQL 和口径 |
| 成本过高 | 大表全表扫或 join 过多 | limit、cost estimate、超时、预聚合表 |
| 用户问题模糊 | 缺时间范围或过滤条件 | 先澄清，不直接查询 |

面试收束句：

```text
Data Agent 的关键不是让模型自由查库，而是让模型在语义层、权限层和 SQL 安全层约束下完成查数，并把 SQL、口径、时间范围和图表解释一起交付给用户。
```

1. Data Agent 不是直接让模型写 SQL，而是查数、口径、安全、执行、解释的完整系统。
2. NL2SQL 的关键是 schema linking 和指标口径，不只是 SQL 语法。
3. 核心指标优先走指标平台或语义层，不让模型自由定义。
4. SQL 执行前必须 AST 校验、只读、权限、limit、超时和 cost 检查。
5. 权限要做到库、表、列、行、指标多层控制。
6. 查询结果要脱敏、聚合，避免泄露明细。
7. 结果解释不能过度因果推断，要绑定 SQL、口径、时间范围和样本量。
8. 评测要看 execution accuracy、schema linking accuracy、metric accuracy、安全通过率和延迟成本。
