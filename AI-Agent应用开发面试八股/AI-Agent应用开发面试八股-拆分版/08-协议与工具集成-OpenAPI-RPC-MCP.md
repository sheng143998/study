# 协议与工具集成：OpenAPI、RPC、MCP

> 来源：由 `AI-Agent应用开发面试八股.md` 拆分整理。建议配合 `00-阅读索引.md` 使用。
> MCP / OpenAPI 工具安全、授权、roots、最小权限和审计深挖，见 `71-AI-Agent安全风控与合规面试手册.md`。

## 11. OpenAPI 八股

### 11.1 OpenAPI 是什么

OpenAPI 是一种描述 HTTP API 的规范，用机器可读的方式定义 API 的路径、方法、参数、请求体、响应、认证方式和 schema。

它常用于：

- 自动生成 API 文档。
- 生成客户端 SDK。
- 服务端代码生成。
- API 测试。
- 将 API 转成大模型工具 schema。

### 11.2 OpenAPI 核心结构

常见字段：

- openapi：规范版本。
- info：API 基本信息。
- servers：服务地址。
- paths：接口路径。
- operations：GET/POST 等操作。
- parameters：路径、查询、header 参数。
- requestBody：请求体。
- responses：响应。
- components：复用 schema、安全定义。
- security：认证要求。

### 11.3 OpenAPI 和 Tool Calling 的关系

OpenAPI 可以作为工具描述来源：

```text
OpenAPI spec
  -> 解析接口 path/method/schema
  -> 生成 tool name/description/parameters
  -> 模型选择工具
  -> 应用层调用 HTTP API
```

注意：

- 不能把所有 API 无脑暴露给模型。
- 需要筛选、改写 description、合并复杂参数。
- 要加权限、限流、审计。
- 对写操作要人工确认。

### 11.4 OpenAPI 高频题

Q：OpenAPI 和 Swagger 是什么关系？

A：Swagger 最初是 API 描述工具和规范名称，后来规范部分演进为 OpenAPI Specification；Swagger 现在常指相关工具生态。

Q：OpenAPI 能不能直接给 Agent 用？

A：可以作为工具生成来源，但通常需要做适配：过滤接口、简化 schema、补充自然语言描述、鉴权、参数校验和风险控制。

Q：REST API 的 GET 和 POST 有什么区别？

A：GET 通常用于读取，参数在 URL，应该是安全且幂等的；POST 通常用于创建或提交复杂操作，请求体承载数据，不一定幂等。实际还要遵守业务约定和 HTTP 语义。

---


## 12. RPC 八股

### 12.1 RPC 是什么

RPC 是 Remote Procedure Call，远程过程调用。它让调用远程服务像调用本地函数一样，隐藏网络通信、序列化和协议细节。

常见 RPC：

- gRPC。
- Thrift。
- Dubbo。
- JSON-RPC。
- XML-RPC。

### 12.2 RPC 和 REST 的区别

| 维度 | REST | RPC |
|---|---|---|
| 抽象 | 资源 | 方法/服务 |
| 常见协议 | HTTP/JSON | HTTP/2、TCP、自定义协议 |
| 接口风格 | GET /users/1 | GetUser(id) |
| 适合 | 公开 API、资源操作 | 内部服务、高性能调用 |
| 契约 | OpenAPI | Protobuf/IDL/JSON-RPC schema |

### 12.3 gRPC 的特点

- 基于 HTTP/2。
- 使用 Protobuf。
- 支持强类型 IDL。
- 支持双向流。
- 性能较好。
- 适合微服务内部通信。

### 12.4 JSON-RPC

JSON-RPC 是一种轻量级 RPC 协议，用 JSON 表示请求和响应。

典型请求：

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "search",
    "arguments": {
      "query": "LangGraph"
    }
  },
  "id": 1
}
```

MCP 的消息层使用 JSON-RPC 2.0。

### 12.5 RPC 在 AI Agent 中的应用

- Agent 编排服务调用模型网关。
- 工具服务内部调用。
- RAG 服务调用 embedding/rerank 服务。
- 高性能检索服务。
- MCP client/server 的底层通信可基于 JSON-RPC。

### 12.6 RPC 高频题

Q：为什么内部服务更常用 RPC？

A：RPC 有强契约、序列化效率高、调用方式统一、适合服务间通信。比如 gRPC 支持 Protobuf、HTTP/2、多语言代码生成和流式传输。

Q：RPC 有什么问题？

A：远程调用不是本地调用，会有网络延迟、超时、重试、服务不可用、版本兼容、限流熔断等问题。不能因为语法像本地函数就忽略分布式系统复杂性。

---


## 13. MCP 八股

### 13.1 MCP 是什么

MCP 是 Model Context Protocol，模型上下文协议。它是一个开放协议，用于标准化 AI 应用与外部工具、数据源、上下文资源和 Prompt 模板之间的连接方式。

可以类比：

- OpenAPI 标准化 HTTP API 描述。
- LSP 标准化编辑器和语言服务。
- MCP 标准化 AI 应用和上下文/工具服务。

### 13.2 MCP 解决什么问题

没有 MCP 时：

- 每个 AI 应用都要单独适配每个工具。
- 工具接入方式不统一。
- 权限、发现、调用、上下文管理碎片化。

有 MCP 后：

- 工具和资源以标准协议暴露。
- AI 客户端可以发现和调用 MCP server 能力。
- 一个 MCP server 可被多个客户端复用。
- 更容易构建可组合的 Agent 工具生态。

### 13.3 MCP 基本架构

```text
MCP Host
  例如 Claude Desktop、IDE、Agent 应用

MCP Client
  Host 内部的协议客户端，和 server 建立连接

MCP Server
  暴露 tools/resources/prompts 等能力

Data Sources / Tools
  文件系统、数据库、Git、浏览器、SaaS、内部 API
```

### 13.4 MCP 核心能力

| 能力 | 说明 |
|---|---|
| Tools | 可被模型调用的动作，例如查数据库、发请求 |
| Resources | 可读取的上下文资源，例如文件、文档、数据 |
| Prompts | 可复用的 Prompt 模板 |
| Sampling | Server 请求 Host 调用模型生成 |
| Roots | Host 告诉 Server 可访问的根目录边界 |
| Elicitation | Server 请求用户补充信息 |

### 13.5 MCP 通信协议

MCP 使用 JSON-RPC 2.0 风格消息，支持不同传输方式，例如：

- stdio：本地进程通信。
- Streamable HTTP：远程服务通信。

面试回答：

MCP 不是某个模型的专属插件系统，而是 AI 应用和上下文/工具服务之间的协议。它定义了能力发现、工具调用、资源读取、Prompt 获取等交互方式。

### 13.6 MCP、OpenAPI、RPC 的关系

```text
OpenAPI
  描述 HTTP API
  可用于生成 Agent tools

RPC
  远程调用风格/协议族
  MCP 使用 JSON-RPC 消息格式

MCP
  面向 AI 应用的上下文和工具连接协议
  可以把 OpenAPI/RPC/数据库/文件系统包装成 MCP server
```

一句话：

OpenAPI 描述 API，RPC 负责远程调用抽象，MCP 负责让 AI 应用以统一方式发现和使用工具/资源/Prompt。

### 13.7 MCP 和 LangChain Tool 的区别

LangChain Tool：

- 框架内部工具抽象。
- 主要在 LangChain 应用内使用。

MCP Tool：

- 协议层工具暴露方式。
- 可以被不同 MCP Host/Client 使用。
- 更强调跨应用、跨工具生态的标准化。

关系：

- 可以把 MCP server 暴露的 tool 接入 LangChain/LangGraph。
- 也可以把已有 LangChain 工具包装成 MCP server。

### 13.8 MCP 高频题

Q：MCP 是不是 OpenAPI 的替代品？

A：不是。OpenAPI 主要描述 HTTP API；MCP 面向 AI 应用的上下文和工具连接。MCP server 内部可以调用 OpenAPI 描述的 API。

Q：MCP 是不是只支持工具调用？

A：不是。MCP 还包括 resources、prompts、sampling、roots、elicitation 等能力。

Q：MCP 为什么适合 Agent？

A：Agent 需要动态发现和调用工具、读取外部上下文、复用 Prompt。MCP 把这些能力标准化，降低不同工具源接入 Agent 的成本。

Q：MCP 的安全边界在哪里？

A：MCP 提供协议层能力，但安全需要 Host、Client、Server 和底层工具共同实现，包括 roots 限制、用户确认、鉴权、权限隔离、审计和敏感数据保护。

---


