# Python 手撕代码模板库

> 目标：把 AI Agent / RAG / Python 后端面试里常见的小工程题，整理成可现场默写的模板。  
> 这份文件不是刷算法大全，而是面向大模型应用开发岗位的高频代码表达。

## 1. 现场写代码的答题顺序

不要上来就写。先用 20 秒讲清楚：

```text
输入输出是什么
  -> 边界条件是什么
  -> 用什么数据结构
  -> 时间复杂度
  -> 再写代码
```

写完后再补 20 秒：

```text
这个版本能处理哪些情况
  -> 还有哪些生产环境增强
```

高分表达：

```text
我先写一个面试版，保证逻辑清楚；如果是生产环境，我会再补线程安全、过期清理、日志和指标。
```

证据字段口播：

```text
我会顺手报出 trace_id / risk_level / timeout_ms / retryable / owner_scope，因为这些字段能证明这段代码能进生产链路。
```

## 2. LRU Cache

适用场景：

- 缓存 embedding 结果。
- 缓存模型响应。
- 缓存用户最近会话。

核心思路：

- `OrderedDict` 维护访问顺序。
- `get` 命中后移动到末尾。
- `put` 超容量时弹出最久未使用。

```python
from collections import OrderedDict


class LRUCache:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.data = OrderedDict()

    def get(self, key: str):
        if key not in self.data:
            return None
        self.data.move_to_end(key)
        return self.data[key]

    def put(self, key: str, value) -> None:
        if key in self.data:
            self.data.move_to_end(key)
        self.data[key] = value
        if len(self.data) > self.capacity:
            self.data.popitem(last=False)
```

复杂度：

```text
get/put 平均 O(1)，空间 O(capacity)。
```

面试解释：

```text
这里用 OrderedDict 是为了同时维护 key-value 和访问顺序。生产环境里还要考虑 TTL、线程安全、缓存击穿和分布式缓存。
```

## 3. TTL Cache

适用场景：

- 缓存短时间内重复的大模型调用。
- 缓存工具查询结果。
- 缓存权限校验结果。

```python
import time


class TTLCache:
    def __init__(self, ttl_seconds: int):
        self.ttl = ttl_seconds
        self.data = {}

    def get(self, key: str):
        item = self.data.get(key)
        if item is None:
            return None
        value, expire_at = item
        if time.time() >= expire_at:
            self.data.pop(key, None)
            return None
        return value

    def set(self, key: str, value) -> None:
        self.data[key] = (value, time.time() + self.ttl)
```

可追问：

```text
如果数据量很大，不能只在 get 时懒清理，还需要后台定期清理或容量上限。
```

## 4. 滑动窗口限流

适用场景：

- 用户每分钟最多请求 N 次。
- 租户级大模型调用限流。
- Tool Calling 防刷。

```python
import time
from collections import defaultdict, deque


class SlidingWindowLimiter:
    def __init__(self, max_requests: int, window_seconds: int):
        self.max_requests = max_requests
        self.window = window_seconds
        self.events = defaultdict(deque)

    def allow(self, user_id: str) -> bool:
        now = time.time()
        q = self.events[user_id]

        while q and now - q[0] >= self.window:
            q.popleft()

        if len(q) >= self.max_requests:
            return False

        q.append(now)
        return True
```

复杂度：

```text
单次请求均摊 O(1)，每个用户最多保存窗口内请求数。
```

生产增强：

- 分布式环境用 Redis。
- 返回 `retry_after`。
- 可按用户、租户、模型、工具分别限流。

## 5. 令牌桶限流

适用场景：

- 允许短时间突发。
- 控制模型网关请求速率。

```python
import time


class TokenBucket:
    def __init__(self, capacity: int, refill_rate: float):
        self.capacity = capacity
        self.refill_rate = refill_rate
        self.tokens = capacity
        self.updated_at = time.time()

    def allow(self, cost: int = 1) -> bool:
        now = time.time()
        elapsed = now - self.updated_at
        self.updated_at = now
        self.tokens = min(self.capacity, self.tokens + elapsed * self.refill_rate)

        if self.tokens >= cost:
            self.tokens -= cost
            return True
        return False
```

面试解释：

```text
令牌桶适合允许突发流量；大模型场景里的 cost 可以不是 1，而是预计 token 数或工具风险权重。
```

## 6. TopK 小根堆

适用场景：

- 高频 query。
- top-k 相似文档。
- top-k 慢接口。

```python
import heapq
from collections import Counter


def top_k_words(words: list[str], k: int) -> list[tuple[str, int]]:
    counter = Counter(words)
    heap = []

    for word, count in counter.items():
        heapq.heappush(heap, (count, word))
        if len(heap) > k:
            heapq.heappop(heap)

    return sorted([(word, count) for count, word in heap], key=lambda x: -x[1])
```

复杂度：

```text
统计 O(n)，堆维护 O(m log k)，m 是不同词数量。
```

## 7. Hybrid Search 合并去重

适用场景：

- 向量召回 + BM25 召回融合。
- 多路召回结果合并。

```python
from dataclasses import dataclass


@dataclass
class Doc:
    doc_id: str
    text: str
    score: float
    source: str


def merge_results(vector_docs: list[Doc], keyword_docs: list[Doc], top_k: int) -> list[Doc]:
    merged = {}

    for rank, doc in enumerate(vector_docs):
        score = 1.0 / (rank + 1)
        merged.setdefault(doc.doc_id, Doc(doc.doc_id, doc.text, 0.0, doc.source))
        merged[doc.doc_id].score += score

    for rank, doc in enumerate(keyword_docs):
        score = 1.0 / (rank + 1)
        merged.setdefault(doc.doc_id, Doc(doc.doc_id, doc.text, 0.0, doc.source))
        merged[doc.doc_id].score += score

    return sorted(merged.values(), key=lambda x: x.score, reverse=True)[:top_k]
```

面试解释：

```text
这里用了简化版 RRF 思路，不直接比较两路原始分数，因为向量分数和 BM25 分数分布不一定一致。
```

## 8. RAG Pipeline 骨架

适用场景：

- 手写一个简化 RAG 流程。
- 解释链路拆分。

```python
class SimpleRAG:
    def __init__(self, retriever, reranker, llm):
        self.retriever = retriever
        self.reranker = reranker
        self.llm = llm

    def answer(self, question: str) -> dict:
        candidates = self.retriever.search(question, top_k=20)
        docs = self.reranker.rank(question, candidates)[:5]

        if not docs:
            return {"answer": "根据现有资料无法确认。", "sources": []}

        context = "\n\n".join(f"[{i}] {doc.text}" for i, doc in enumerate(docs, 1))
        prompt = f"基于以下资料回答问题，无法确认就说无法确认。\n{context}\n\n问题：{question}"
        answer = self.llm.generate(prompt)

        return {
            "answer": answer,
            "sources": [{"doc_id": d.doc_id, "source": d.source} for d in docs],
        }
```

高分补充：

```text
生产环境还要补权限过滤、引用校验、无答案识别、日志 trace、缓存和评测。
```

RAG ACL 最小 trace 字段：`tenant_id, user_id, doc_acl_version, pre_filter_count, post_filter_count, citation_acl_pass`。

## 9. Tool Call 参数校验

适用场景：

- Function Calling 参数验证。
- MCP/OpenAPI 工具执行前校验。

```python
def validate_args(args: dict, schema: dict) -> tuple[bool, str]:
    required = schema.get("required", [])
    properties = schema.get("properties", {})

    for field in required:
        if field not in args:
            return False, f"missing required field: {field}"

    for key, value in args.items():
        if key not in properties:
            return False, f"unknown field: {key}"
        expected = properties[key].get("type")
        if expected == "string" and not isinstance(value, str):
            return False, f"{key} should be string"
        if expected == "integer" and not isinstance(value, int):
            return False, f"{key} should be integer"
        if expected == "boolean" and not isinstance(value, bool):
            return False, f"{key} should be boolean"

    return True, ""
```

面试解释：

```text
这是简化版 JSON Schema 校验。生产环境建议用成熟库，并且敏感参数不能相信模型生成，要从服务端上下文获取。
```

## 10. 工具调用重试退避

适用场景：

- 外部 API 临时失败。
- 模型供应商 429/5xx。

```python
import time


def retry_with_backoff(func, max_retries: int = 3, base_delay: float = 0.2):
    last_error = None
    for attempt in range(max_retries + 1):
        try:
            return func()
        except Exception as exc:
            last_error = exc
            if attempt == max_retries:
                break
            time.sleep(base_delay * (2 ** attempt))
    raise last_error
```

追问防守：

```text
只有幂等、可重试的错误才适合自动重试。对支付、提交、创建这类有副作用的工具，要用幂等 key 或人工确认。
```

## 11. Tool Call 去重

适用场景：

- 防止 Agent 重复调用同一工具。
- 防止用户重复提交。

```python
import hashlib
import json
import time


class ToolCallDeduper:
    def __init__(self, ttl_seconds: int = 60):
        self.ttl = ttl_seconds
        self.seen = {}

    def _key(self, tool_name: str, args: dict) -> str:
        raw = json.dumps({"tool": tool_name, "args": args}, sort_keys=True)
        return hashlib.sha256(raw.encode("utf-8")).hexdigest()

    def is_duplicate(self, tool_name: str, args: dict) -> bool:
        now = time.time()
        key = self._key(tool_name, args)

        expired = [k for k, expire_at in self.seen.items() if expire_at <= now]
        for k in expired:
            self.seen.pop(k, None)

        if key in self.seen:
            return True

        self.seen[key] = now + self.ttl
        return False
```

面试解释：

```text
这里用工具名和规范化参数生成 hash。生产环境可以放到 Redis，并结合 user_id、session_id 和 idempotency_key。
```

## 12. Agent 最大步数控制

适用场景：

- 防止 Agent 死循环。
- 控制成本。

```python
def run_agent(user_input: str, planner, tools: dict, max_steps: int = 5):
    state = {"input": user_input, "steps": [], "final": None}

    for _ in range(max_steps):
        action = planner.next_action(state)

        if action["type"] == "finish":
            state["final"] = action["answer"]
            return state

        if action["type"] == "tool":
            tool_name = action["tool"]
            if tool_name not in tools:
                state["steps"].append({"error": f"unknown tool: {tool_name}"})
                continue
            result = tools[tool_name](**action.get("args", {}))
            state["steps"].append({"tool": tool_name, "result": result})

    state["final"] = "任务步骤过多，已停止执行，请补充信息或转人工处理。"
    return state
```

高分补充：

```text
生产环境还要加重复动作检测、工具权限、超时、人工确认和 trace。
```

## 13. 引用编号校验

适用场景：

- RAG 答案必须带来源。
- 防止模型编造引用。

```python
import re


def validate_citations(answer: str, source_count: int) -> bool:
    refs = re.findall(r"\[(\d+)\]", answer)
    for ref in refs:
        idx = int(ref)
        if idx < 1 or idx > source_count:
            return False
    return True
```

追问防守：

```text
这个只能校验引用编号存在，不能证明语义一致。更严格时要检查答案中的关键结论是否能被对应 source 支撑。
```

## 14. Metric Contract Validator

适用场景：

- Data Agent / ChatBI 生成 SQL 前。
- 校验指标口径、权限、粒度和权威来源。

```python
def validate_metric_contract(metric: dict, scope: dict) -> bool:
    if metric.get("status") != "approved":
        return False
    if scope["role"] not in metric.get("allowed_roles", set()):
        return False
    if scope["tenant_id"] not in metric.get("allowed_tenants", set()):
        return False
    if metric.get("grain") not in {"day", "week", "month"}:
        return False
    return metric.get("source_priority") == "semantic_layer"
```

面试解释：

```text
Data Agent 不是把自然语言直接翻译成 SQL。指标口径、权限、时间粒度和权威来源要先过 Metric Contract，再进入 SQL 生成和 AST 校验。
```

## 15. SQL 只读校验

适用场景：

- Text-to-SQL Agent。
- 数据分析 Agent 工具调用前安全检查。

```python
import re


FORBIDDEN = {
    "insert", "update", "delete", "drop", "alter",
    "truncate", "create", "replace", "merge", "grant", "revoke",
}


def is_readonly_sql(sql: str) -> bool:
    normalized = re.sub(r"\s+", " ", sql.strip().lower())
    if not normalized.startswith("select"):
        return False
    tokens = set(re.findall(r"[a-z_]+", normalized))
    return not (tokens & FORBIDDEN)
```

高分补充：

```text
真实生产不能只靠正则，最好用 SQL parser，并结合表级权限、行级权限、LIMIT 和查询超时。
```

## 16. 自动添加 LIMIT

适用场景：

- 防止查询返回过多数据。
- 数据分析 Agent 兜底。

```python
import re


def ensure_limit(sql: str, limit: int = 100) -> str:
    stripped = sql.strip().rstrip(";")
    if re.search(r"\blimit\s+\d+\b", stripped, flags=re.IGNORECASE):
        return stripped + ";"
    return f"{stripped} LIMIT {limit};"
```

追问防守：

```text
这只是面试版。生产里要考虑子查询、不同数据库方言和 order by，最好用 SQL AST 处理。
```

## 17. 会话历史压缩

适用场景：

- 多轮对话上下文过长。
- Agent state 控制 token。

```python
def compact_history(messages: list[dict], summarizer, keep_last: int = 6) -> list[dict]:
    if len(messages) <= keep_last:
        return messages

    old = messages[:-keep_last]
    recent = messages[-keep_last:]
    text = "\n".join(f"{m['role']}: {m['content']}" for m in old)
    summary = summarizer(text)

    return [{"role": "system", "content": f"历史摘要：{summary}"}] + recent
```

面试解释：

```text
多轮对话不能无限塞上下文。可以保留最近消息，旧消息做摘要，但关键事实、用户偏好和工具执行结果最好结构化保存。
```

## 17.1 State Store Update

适用场景：

- 长任务 Agent。
- 多轮对话的结构化状态保存。

```python
SENSITIVE_KEYS = {"password", "token", "secret", "id_card"}


def update_state_store(state: dict, event: dict, keep_last: int = 6) -> dict:
    new_state = {**state, "version": state.get("version", 0) + 1}
    if event["type"] == "message":
        msgs = new_state.setdefault("recent_messages", [])
        msgs.append({"role": event["role"], "content": event["content"]})
        new_state["recent_messages"] = msgs[-keep_last:]
    if event["type"] == "tool_result":
        safe = {k: v for k, v in event["result"].items() if k not in SENSITIVE_KEYS}
        new_state.setdefault("tool_summaries", {})[event["tool_name"]] = safe
    return new_state
```

面试解释：

```text
摘要解决 token，state store 解决恢复和审计。审批状态、工具结果、权限范围不能只靠自然语言摘要。
```

## 18. 并发执行器

适用场景：

- 并发调用多个检索源。
- 并发调用多个工具。
- 批量 embedding。

```python
import asyncio


async def run_with_limit(tasks, limit: int):
    semaphore = asyncio.Semaphore(limit)

    async def wrapper(coro):
        async with semaphore:
            return await coro

    return await asyncio.gather(*(wrapper(task) for task in tasks), return_exceptions=True)
```

追问防守：

```text
return_exceptions=True 可以避免一个任务失败导致全部失败。生产环境要记录每个任务的错误，并对关键任务设置超时。
```

## 19. 异步超时控制

适用场景：

- 模型调用。
- 工具调用。
- 检索调用。

```python
import asyncio


async def call_with_timeout(coro, timeout_seconds: float):
    try:
        return await asyncio.wait_for(coro, timeout=timeout_seconds)
    except asyncio.TimeoutError:
        return {"ok": False, "error": "timeout"}
```

高分补充：

```text
超时后要看底层请求是否真的取消。对外部 HTTP 调用，还需要客户端层面的 timeout 配置。
```

## 20. SSE 生成器骨架

适用场景：

- FastAPI 流式输出。
- 大模型 token streaming。

```python
import json


async def sse_stream(llm_stream):
    try:
        async for chunk in llm_stream:
            data = json.dumps({"type": "delta", "content": chunk}, ensure_ascii=False)
            yield f"data: {data}\n\n"
        yield 'data: {"type": "done"}\n\n'
    except Exception as exc:
        data = json.dumps({"type": "error", "message": str(exc)}, ensure_ascii=False)
        yield f"data: {data}\n\n"
```

面试解释：

```text
SSE 是服务端持续推送文本事件。生产环境要补心跳、断连检测、首 token 延迟指标和敏感信息过滤。
```

高分追问：

```text
断线重连要支持 Last-Event-ID 或业务游标；服务端要发 heartbeat，客户端慢消费时要限制队列长度和 token buffer，避免一个连接拖垮 worker。
```

SSE 事件契约字段：`event_id, turn_id, event_type, trace_id, delta, token_delta, finish_reason, heartbeat, retry_after_ms, latency_ms, cancel_reason`。

## 21. JSON Repair 简化版

适用场景：

- 模型输出 JSON 偶尔带多余文本。
- Structured Output 兜底。

```python
import json


def parse_json_from_text(text: str):
    try:
        return json.loads(text)
    except json.JSONDecodeError:
        pass

    start = text.find("{")
    end = text.rfind("}")
    if start == -1 or end == -1 or end <= start:
        raise ValueError("no json object found")

    return json.loads(text[start:end + 1])
```

追问防守：

```text
这只是兜底解析。更推荐从模型侧使用 structured output 或 JSON mode，并在后端做 schema 校验。
```

## 22. Prompt 版本选择

适用场景：

- Prompt 平台。
- A/B 实验。
- 灰度发布。

```python
import hashlib


def choose_variant(user_id: str, variants: list[str]) -> str:
    digest = hashlib.md5(user_id.encode("utf-8")).hexdigest()
    bucket = int(digest[:8], 16)
    return variants[bucket % len(variants)]
```

面试解释：

```text
用稳定 hash 可以保证同一用户稳定落到同一版本，避免体验抖动。生产里还要支持流量比例、白名单和回滚。
```

## 23. 模型路由

适用场景：

- 简单问题走便宜模型。
- 复杂问题走强模型。
- 模型网关设计题。

```python
def route_model(request: dict) -> str:
    task_type = request.get("task_type")
    priority = request.get("priority", "normal")
    estimated_tokens = request.get("estimated_tokens", 0)

    if priority == "high":
        return "strong-model"
    if task_type in {"classification", "rewrite", "format"} and estimated_tokens < 1000:
        return "cheap-model"
    if estimated_tokens > 8000:
        return "long-context-model"
    return "default-model"
```

高分补充：

```text
真实路由要基于离线评测和线上指标，不是拍规则。要持续观察质量、延迟、成本和失败率。
```

生产追问：

```text
模型网关还要看 provider 健康度、error_rate、quota_left、p95_latency 和熔断状态。降级时记录 route_reason，优先从强模型降到稳定模型，再降到模板化兜底。
```

路由决策日志字段：`route_reason, policy_version, model_version, fallback_from, latency_budget_ms, cost_budget, safety_tier`。

## 24. 文档增量更新

适用场景：

- 知识库同步。
- 向量索引更新。

```python
import hashlib


def content_hash(text: str) -> str:
    return hashlib.sha256(text.encode("utf-8")).hexdigest()


def diff_docs(old_docs: dict[str, str], new_docs: dict[str, str]) -> dict:
    old_ids = set(old_docs)
    new_ids = set(new_docs)

    added = new_ids - old_ids
    deleted = old_ids - new_ids
    updated = {
        doc_id
        for doc_id in old_ids & new_ids
        if content_hash(old_docs[doc_id]) != content_hash(new_docs[doc_id])
    }

    return {"added": added, "updated": updated, "deleted": deleted}
```

面试解释：

```text
增量更新的关键是识别新增、修改和删除。生产环境还要处理版本、权限、索引原子切换和回滚。
```

## 25. 简单 BM25 骨架

适用场景：

- 解释关键词检索。
- Hybrid Search 面试题。

```python
import math
from collections import Counter


def bm25_score(query: list[str], doc: list[str], df: dict[str, int], total_docs: int) -> float:
    k1 = 1.5
    b = 0.75
    avgdl = 100
    tf = Counter(doc)
    score = 0.0
    dl = len(doc)

    for term in query:
        if term not in tf:
            continue
        idf = math.log((total_docs - df.get(term, 0) + 0.5) / (df.get(term, 0) + 0.5) + 1)
        numerator = tf[term] * (k1 + 1)
        denominator = tf[term] + k1 * (1 - b + b * dl / avgdl)
        score += idf * numerator / denominator

    return score
```

追问防守：

```text
这个是骨架，真实系统会预先构建倒排索引，不会每次遍历所有文档。
```

## 26. 面试前必背代码清单

P0：

- LRU Cache。
- 滑动窗口限流。
- TopK 小根堆。
- RAG Pipeline。
- Tool 参数校验。
- Agent 最大步数。
- Eval Gate 发布门禁。

P1：

- TTL Cache。
- 令牌桶。
- Hybrid Search 合并。
- 并发执行器。
- SSE 生成器。
- Metric Contract Validator。
- SQL 只读校验。
- A2A Task Contract 校验。

P2：

- Tool Call 去重。
- Prompt 版本选择。
- 模型路由。
- 文档增量更新。
- BM25 骨架。
- 会话历史压缩。
- State Store Update。
- Streaming Event Merger。
- Realtime Eval Gate。
- Tool Risk Classifier。
- Browser Action Guard。
- Trace To Eval Join。
- Network Sandbox Policy。
- Tool / Agent Retrieval。
- Memory Write Policy。
- Memory Delete Policy。
- Agent Job State Machine。

工具风险分类面试先答四格：`read_only` 自动，`low_risk_write` 要幂等，`high_risk_write` 要确认，`external_effect` 要审计和限流。

## 26.1 Eval Gate 发布门禁

适用场景：

- RAG / Agent 发布前回归。
- Prompt、模型、检索配置、工具 schema 改动后的质量门禁。

```python
def eval_gate(results: list[dict], thresholds: dict) -> tuple[bool, list[dict]]:
    if not results:
        return False, [{"metric": "dataset", "reason": "empty"}]

    total = len(results)
    p95_index = max(0, (95 * total + 99) // 100 - 1)
    metrics = {
        "task_success": sum(r["task_success"] for r in results) / total,
        "faithfulness": sum(r["faithfulness"] for r in results) / total,
        "safety_pass": sum(r["safety_pass"] for r in results) / total,
        "p95_latency_ms": sorted(r["latency_ms"] for r in results)[p95_index],
        "avg_cost": sum(r["cost"] for r in results) / total,
    }

    failed = []
    for name, threshold in thresholds.items():
        value = metrics[name]
        ok = value <= threshold if name.startswith("p95_") or name.startswith("avg_cost") else value >= threshold
        if not ok:
            failed.append({"metric": name, "value": value, "threshold": threshold})

    return len(failed) == 0, failed
```

追问防守：

```text
门禁不是追求指标越多越好，而是覆盖质量、安全、成本和延迟四类核心风险。失败样本要回流到 bad case set，不能只看平均分。
```

15 秒模板：

```text
我不会只看平均分，会把 quality / latency / cost / safety 四类 gate 都过一遍，任一核心指标回退就禁止灰度或自动回滚。
```

LLM-as-Judge 校准：

```text
Judge 本身也要版本化：记录 judge_version、prompt_version、抽检比例和人工复核结果。关键集可以用多 judge 一致性或人工金标校准，防止阈值漂移和偏置样本把发布门禁带偏。
```

## 26.2 A2A Task Contract 校验

适用场景：

- 多 Agent 委派任务。
- 防止能力、权限、交付物和超时边界不清。

```python
REQUIRED = {"task_id", "from_agent", "to_agent", "capability", "scope", "result_schema"}


def validate_a2a_contract(contract: dict, allowed_capabilities: set[str]) -> bool:
    if REQUIRED - contract.keys():
        return False
    if contract["capability"] not in allowed_capabilities:
        return False
    scope = contract["scope"]
    if not scope.get("tenant_id") or not scope.get("permission_token"):
        return False
    return True
```

追问防守：

```text
A2A 解决 Agent 间任务通信，不替代鉴权和审计。生产里还要校验 deadline、approval_id、result_schema 和责任归属。
```

## 26.3 Streaming Event Merger

适用场景：

- 语音 / 实时 / 多模态 Agent。
- 合并 partial transcript、final transcript、barge-in 和高风险动作确认。

```python
def merge_event(state: dict, event: dict) -> dict:
    turn = state.setdefault(event["turn_id"], {"partial": "", "final": "", "cancelled": False})

    if event["type"] == "barge_in":
        turn["cancelled"] = True
        turn["partial"] = ""
    elif not turn["cancelled"] and event["type"] == "partial":
        turn["partial"] = event["text"]
    elif not turn["cancelled"] and event["type"] == "final":
        turn["final"] = event["text"]
        turn["partial"] = ""
    elif event["type"] == "tool_action":
        turn["confirm_required"] = event.get("risk_level") == "high"

    return state
```

追问防守：

```text
实时 Agent 要记录 turn_id、ASR 置信度、打断、最终转写和确认状态，否则线上很难复盘误触发和错执行。
```

## 26.4 Realtime Eval Gate

适用场景：

- 实时语音 / 多模态 Agent 发布前回归。
- 检查首响延迟、打断、误触发和高风险确认。

```python
def realtime_eval_gate(events: list[dict], thresholds: dict) -> tuple[bool, list[dict]]:
    turns = {}
    for e in events:
        turn = turns.setdefault(e["turn_id"], {"start": None, "first": None, "barge": 0, "unsafe": 0, "confirm": 0})
        if e["type"] == "user_start":
            turn["start"] = e["ts_ms"]
        elif e["type"] == "assistant_first_token" and turn["first"] is None:
            turn["first"] = e["ts_ms"]
        elif e["type"] == "barge_in":
            turn["barge"] += 1
        elif e["type"] == "unsafe_trigger":
            turn["unsafe"] += 1
        elif e["type"] == "confirm_required":
            turn["confirm"] += 1

    total = max(len(turns), 1)
    latencies = [t["first"] - t["start"] for t in turns.values() if t["start"] is not None and t["first"] is not None]
    metrics = {
        "avg_first_token_ms": sum(latencies) / max(len(latencies), 1),
        "unsafe_trigger_rate": sum(t["unsafe"] for t in turns.values()) / total,
        "confirm_rate": sum(t["confirm"] for t in turns.values()) / total,
    }
    failed = [{"metric": k, "value": metrics[k], "threshold": v} for k, v in thresholds.items() if metrics[k] > v]
    return len(failed) == 0, failed
```

追问防守：

```text
实时 Agent 不能只看答案正确率，还要看首响延迟、打断恢复、误触发率和高风险动作确认，否则文本评测通过也可能在线上出事故。
```

## 26.5 Tool Risk Classifier

适用场景：

- Function calling / MCP / A2A 委派前的统一风险判断。
- 决定是否需要用户确认、幂等键和审计日志。

```python
HIGH_RISK_TOOLS = {"payment", "delete_file", "send_email", "execute_command"}


def classify_tool_risk(call: dict) -> dict:
    name = call["tool_name"]
    args = call.get("args", {})
    high = name in HIGH_RISK_TOOLS or name.startswith(("create_", "update_", "delete_", "send_"))
    if args.get("contains_secret"):
        high = True
    return {
        "risk_level": "high" if high else "low",
        "require_confirm": high,
        "audit_required": True,
        "need_idempotency_key": high,
    }
```

追问防守：

```text
工具风险分类要在应用层做，覆盖副作用、敏感字段、外部系统、幂等和审计。MCP/A2A 只提供连接或通信协议，不替代风险治理。
```

## 26.6 Browser Action Guard

适用场景：

- Computer Use / Browser Agent。
- 自动填表、网页操作、RPA、测试/运维自动化。

```python
from urllib.parse import urlparse


CONFIRM_ACTIONS = {"submit", "download", "payment", "delete", "send_message"}


def guard_browser_action(action: dict, allowed_domains: set[str]) -> bool:
    domain = urlparse(action["url"]).netloc
    if domain not in allowed_domains:
        return False
    if action.get("contains_secret"):
        return False
    if action["type"] in CONFIRM_ACTIONS and not action.get("approval_id"):
        return False
    if action.get("confidence", 1.0) < 0.8:
        return False
    return True
```

追问防守：

```text
Browser Agent 的动作策略要在应用层执行，不能交给模型自觉。跨域、下载、提交、付款、删除、发送消息、低置信度和敏感字段都要阻断或确认。
```

## 26.7 Trace To Eval Join

适用场景：

- 线上 bad case 回流。
- RAG / Agent 发布前回归。

```python
def trace_to_eval_case(trace: dict, expected: str, failure_type: str) -> dict:
    failed = [s for s in trace["spans"] if s.get("status") == "failed"]
    span = failed[0] if failed else trace["spans"][-1]
    return {
        "case_id": f"case_{trace['trace_id']}",
        "trace_id": trace["trace_id"],
        "input": trace["user_input"],
        "actual": trace.get("final_answer"),
        "expected": expected,
        "failure_type": failure_type,
        "failed_node": span["node"],
        "span_id": span["span_id"],
    }
```

追问防守：

```text
trace 负责定位哪一步错，eval case 负责防止下次再错。二者用 trace_id/case_id 关联，才能形成线上问题到离线回归的闭环。
```

## 26.8 Network Sandbox Policy

适用场景：

- Browser / Computer Use Agent。
- 代码执行、终端执行、自动化运维。

```python
import ipaddress
from urllib.parse import urlparse


def is_private_ip(ip: str) -> bool:
    addr = ipaddress.ip_address(ip)
    return addr.is_private or addr.is_loopback or addr.is_link_local


def allow_request(req: dict, policy: dict) -> bool:
    domain = urlparse(req["url"]).netloc
    if domain not in policy["allowed_domains"]:
        return False
    if req.get("resolved_ip") and is_private_ip(req["resolved_ip"]):
        return False
    if req.get("credential_scope") not in policy["allowed_credential_scopes"]:
        return False
    return True
```

追问防守：

```text
网络沙箱要防 SSRF、内网探测、生产环境误访问和凭据外泄。审计日志至少记录 url、resolved_ip、credential_scope、decision、reason 和 trace_id。
```

## 26.9 Tool / Agent Retrieval

适用场景：

- 工具很多，避免 tool overload。
- 多 Agent 平台做能力发现和委派。

```python
def rank_tools(query_caps: set[str], tools: list[dict], tenant_id: str) -> list[dict]:
    result = []
    for tool in tools:
        matched = len(query_caps & set(tool["capabilities"]))
        if matched == 0:
            continue
        if tenant_id not in tool["tenant_scope"] and "*" not in tool["tenant_scope"]:
            continue
        risk_penalty = {"low": 0.0, "medium": 0.2, "high": 0.5}[tool["risk_level"]]
        score = matched + tool.get("recent_success_rate", 0.5) - risk_penalty
        result.append((score, tool))
    return [tool for score, tool in sorted(result, key=lambda x: x[0], reverse=True)]
```

追问防守：

```text
不要把 100 个工具全塞给模型。先按 capability、tenant scope、risk level、recent success 和 latency 做检索排序，再把少量候选暴露给模型。
```

## 26.10 Memory Write Policy

适用场景：

- 个性化 Agent。
- 客服长期偏好。
- 长任务状态和用户画像。

```python
SENSITIVE = {"password", "token", "id_card", "bank_card", "medical"}


def memory_write_policy(memory: dict, scope: dict) -> str:
    if memory["category"] in SENSITIVE:
        return "reject"
    if memory["tenant_id"] != scope["tenant_id"]:
        return "reject"
    if memory.get("source") not in {"user_explicit", "tool_verified"}:
        return "reject"
    if memory.get("ttl_days", 0) <= 0:
        return "reject"
    if memory["category"] == "preference" and not memory.get("user_confirmed"):
        return "require_confirm"
    return "allow"
```

追问防守：

```text
长期记忆要有写入策略、读取权限、TTL 和删除链路。不要把模型猜测、敏感信息或跨租户数据写进长期记忆。
```

## 26.11 Memory Delete Policy

适用场景：

- 用户要求删除长期记忆。
- 合规要求 TTL 到期或撤回授权。

```python
def memory_delete_plan(memory: dict, requester: dict) -> tuple[bool, list[str]]:
    if memory["tenant_id"] != requester["tenant_id"]:
        return False, ["tenant mismatch"]
    if memory["user_id"] != requester["user_id"] and requester.get("role") != "admin":
        return False, ["permission denied"]
    return True, [
        f"delete_primary:{memory['memory_id']}",
        f"tombstone_vector:{memory['memory_id']}",
        f"purge_cache:{memory['memory_id']}",
        f"rebuild_summary:{memory['user_id']}",
        f"audit_delete:{memory['memory_id']}",
    ]
```

追问防守：

```text
删除记忆不能只删主库。长期记忆可能进入向量索引、缓存和摘要，要同步失效并保留脱敏审计。
```

## 26.12 Agent Job State Machine

适用场景：

- 报告生成、批量文档处理、自动化运维、代码执行等长任务 Agent。

```python
TRANSITIONS = {
    "pending": {"running", "cancelled"},
    "running": {"waiting_approval", "succeeded", "failed", "cancel_pending"},
    "waiting_approval": {"running", "cancel_pending", "failed"},
    "cancel_pending": {"cancelled", "failed"},
    "failed": {"pending"},
}


def move(job: dict, to_status: str) -> dict:
    if to_status not in TRANSITIONS.get(job["status"], set()):
        raise ValueError("invalid transition")
    return {**job, "status": to_status}
```

追问防守：

```text
长任务不要占着 HTTP 请求硬跑。提交返回 job_id，后台 worker 执行，每步写 checkpoint；取消要在安全点生效，高风险写操作需要补偿或人工接管。
```

## 27. 最后速记

```text
代码题不是只写出结果，而是展示工程意识：

边界条件
  -> 数据结构
  -> 复杂度
  -> 异常处理
  -> 生产增强
```

一句话：

```text
AI 应用开发的手撕题，面试官更想看你能不能把算法小题讲成工程组件。
```
