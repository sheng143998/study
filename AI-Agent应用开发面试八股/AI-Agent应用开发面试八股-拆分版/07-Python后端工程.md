# Python 后端工程

> 来源：由 `AI-Agent应用开发面试八股.md` 拆分整理。建议配合 `00-阅读索引.md` 使用。

## 5. Python 高频八股

AI Agent 应用开发岗位通常不是纯 Python 语法题，而是结合工程能力问：并发、异步、数据处理、Web API、类型、异常、性能、部署。

### 5.1 Python 中 list、tuple、dict、set 的区别

| 类型 | 是否有序 | 是否可变 | 是否可重复 | 典型用途 |
|---|---|---|---|---|
| list | 是 | 是 | 是 | 动态数组 |
| tuple | 是 | 否 | 是 | 不可变记录 |
| dict | 是，3.7+ 保持插入顺序 | 是 | key 不重复 | 键值映射 |
| set | 通常不关心顺序 | 是 | 否 | 去重、集合运算 |

追问：

- dict 为什么快？
  - 基于哈希表，平均 O(1) 查找。
- key 为什么必须可哈希？
  - key 的 hash 值和相等性必须稳定。

### 5.2 深拷贝和浅拷贝

浅拷贝：

- 只复制外层容器。
- 内部引用对象仍共享。

深拷贝：

- 递归复制内部对象。
- 成本更高，可能遇到循环引用。

```python
import copy

a = [[1], [2]]
b = copy.copy(a)
c = copy.deepcopy(a)
```

### 5.3 生成器和迭代器

迭代器：

- 实现 `__iter__` 和 `__next__`。
- 可逐个返回元素。

生成器：

- 使用 `yield` 创建。
- 惰性计算，节省内存。

应用：

- 流式读取大文件。
- 流式返回模型输出。
- 分批处理 embedding。

### 5.4 装饰器

装饰器是接收函数并返回新函数的高阶函数，常用于日志、鉴权、重试、缓存、耗时统计。

```python
from functools import wraps

def log_time(fn):
    @wraps(fn)
    def wrapper(*args, **kwargs):
        print("start")
        return fn(*args, **kwargs)
    return wrapper
```

AI 应用中的场景：

- 给模型调用加重试。
- 给工具调用加权限校验。
- 记录 trace。
- API 限流。

### 5.5 GIL 是什么

GIL 是 CPython 的全局解释器锁，它让同一进程内同一时刻通常只有一个线程执行 Python 字节码。

影响：

- CPU 密集型任务多线程收益有限。
- IO 密集型任务多线程仍有价值，因为等待 IO 时会释放 GIL。
- CPU 密集型可用 multiprocessing、C 扩展、NumPy、Rust/Go 服务等方案。

面试回答：

LLM 应用多数是 IO 密集型，例如 HTTP 调模型、查数据库、调向量库，所以 asyncio 或线程池能提升吞吐。但本地 embedding、大规模文档解析、rerank 模型推理可能是 CPU/GPU 密集，需要进程池或独立推理服务。

### 5.6 asyncio、线程、进程怎么选

| 场景 | 方案 |
|---|---|
| 大量 HTTP 请求、数据库请求 | asyncio |
| 阻塞 SDK、少量 IO 并发 | ThreadPoolExecutor |
| CPU 密集处理 | ProcessPoolExecutor / multiprocessing |
| GPU 推理 | 独立推理服务 / batch |
| FastAPI 高并发接口 | async + 连接池 + 限流 |

常见追问：

Q：asyncio 为什么能提高并发？

A：它是协作式并发，在单线程事件循环中，当任务遇到 await IO 时让出执行权，事件循环调度其他任务执行，从而提高 IO 密集场景吞吐。

Q：asyncio 能提升 CPU 密集任务吗？

A：不能明显提升，因为 CPU 计算不会主动让出事件循环，还会阻塞其他协程。CPU 密集任务应放进进程池、线程池或独立服务。

### 5.7 Python 中 `is` 和 `==`

- `is` 比较对象身份，即是否同一个对象。
- `==` 比较值是否相等，调用 `__eq__`。

### 5.8 Python 可变默认参数陷阱

错误：

```python
def append_item(x, items=[]):
    items.append(x)
    return items
```

正确：

```python
def append_item(x, items=None):
    if items is None:
        items = []
    items.append(x)
    return items
```

原因：

默认参数在函数定义时求值，只会创建一次。

### 5.9 FastAPI 在 AI 应用中的高频问题

Q：FastAPI 为什么适合 LLM 应用？

A：支持 async、高性能、类型注解、自动 OpenAPI 文档、Pydantic 数据校验，适合封装模型调用、RAG、工具服务和 MCP/OpenAPI 网关。

Q：如何实现流式输出？

A：可以用 `StreamingResponse` 或 SSE，把模型增量 token/chunk 逐步返回给前端。要注意断连处理、超时、异常、日志和 token 统计。

生产级 SSE 不只是把 token `yield` 出去，还要定义事件协议：

| 事件 | 触发时机 | 必带字段 | 前端处理 |
|---|---|---|---|
| `message_start` | 一轮回答开始 | event_id、turn_id、trace_id | 初始化消息气泡和计时 |
| `partial` | 模型增量输出 | event_id、sequence、delta | 追加 token / 文本片段 |
| `tool_progress` | 工具开始、重试、完成 | tool_name、status、latency_ms | 展示进度，不让用户以为卡死 |
| `approval_required` | 高风险动作待确认 | action_id、risk_level、summary | 暂停执行，等待用户/人工确认 |
| `final` | 回答完成 | finish_reason、usage、latency_ms | 收束消息，记录 token 和耗时 |
| `error` | 模型/工具/服务异常 | error_code、retryable、message | 展示可恢复提示或降级 |
| `heartbeat` | 长任务无输出时 | event_id、trace_id | 保持连接，检测断线 |

最小字段：

```text
event_id, turn_id, sequence, event_type, trace_id, timestamp,
delta, tool_name, status, approval_id, finish_reason,
latency_ms, token_usage, retry_after_ms
```

生产细节：

- 断线重连：支持 `Last-Event-ID` 或业务游标，避免重复显示 token。
- 慢客户端：限制每连接队列长度，超过阈值丢弃低价值 progress 或主动断开。
- client disconnect：服务端检测断连后取消下游模型流和未确认工具。
- backpressure：模型输出、工具进度和日志写入要解耦，不能让前端慢消费拖垮 worker。
- 安全过滤：进入 SSE 前做敏感字段脱敏，工具原始结果不要直接推给前端。
- 指标：记录 first_token_latency、tokens_per_second、stream_error_rate、client_disconnect_rate。

60 秒回答：

```text
我会把 SSE 当成事件协议，而不是简单流式字符串。每个事件有 event_id、turn_id、sequence 和 trace_id，支持 partial、tool_progress、approval_required、final、error 和 heartbeat。生产上要处理断线重连、慢客户端 backpressure、client disconnect 取消下游任务，以及敏感信息过滤和首 token 延迟指标。
```

Q：怎么处理高并发模型调用？

A：连接池、异步客户端、限流、队列、批处理、缓存、超时、熔断、重试、降级模型、隔离租户 quota。

---


