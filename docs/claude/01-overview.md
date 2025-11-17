# 项目概览 (Project Overview)

## 项目简介

**Agent Development Kit (ADK) for Go** 是一个灵活的、代码优先的 AI Agent 开发框架，用于构建、部署和编排 AI Agent。它将软件开发原则应用于 Agent 创建，专注于地道的 Go 语言实现，充分利用 Go 在并发和云原生应用方面的优势。

**核心特点：**
- 模型无关（针对 Gemini 优化，通过 `google.golang.org/genai`）
- 部署无关（强力支持容器化和 Google Cloud Run）
- 使用 Go 1.24.4+ 特性，包括迭代器 (`iter.Seq2`)
- Apache 2.0 许可证
- 与 adk-python 对齐（以 Python 版本为真理源）

---

## 目录结构与职责

### 核心公共 API 包

| 目录 | 主要职责 | 关键文件 |
|------|---------|----------|
| `agent/` | Agent 基础接口和构造器 | `agent.go`, `callbacks.go`, `context.go` |
| `agent/llmagent/` | 基于 LLM 的 Agent 实现，处理模型交互 | `llmagent.go`, `config.go` |
| `agent/remoteagent/` | 运行在远程服务器上的 Agent | `remoteagent.go` |
| `agent/workflowagents/` | Agent 编排模式（loop, parallel, sequential） | `loopagent/`, `parallelagent/`, `sequentialagent/` |
| `session/` | 会话管理和事件处理 | `session.go`, `event.go`, `service.go` |
| `session/database/` | 持久化会话存储（基于 GORM） | `database.go` |
| `runner/` | Agent 运行时，执行 Agent 并管理会话 | `runner.go` |
| `tool/` | 工具接口和实现 | `tool.go` |
| `tool/functiontool/` | 自定义函数工具 | `functiontool.go` |
| `tool/geminitool/` | Gemini 专用工具（GoogleSearch 等） | `googlesearch.go` |
| `tool/agenttool/` | Agent 委托工具 | `agenttool.go` |
| `tool/mcptoolset/` | Model Context Protocol 工具集 | `mcptoolset.go` |
| `tool/loadartifactstool/` | Artifact 加载工具 | `loadartifactstool.go` |
| `tool/exitlooptool/` | 循环退出工具（用于 LoopAgent） | `exitlooptool.go` |
| `server/` | 用于服务 Agent 的协议实现 | - |
| `server/adkrest/` | REST API 服务器 | `server.go` |
| `server/adka2a/` | Agent-to-Agent 协议服务器 | `server.go` |
| `model/` | LLM 模型接口 | `model.go` |
| `model/gemini/` | Gemini 模型实现 | `gemini.go` |
| `memory/` | 语义内存服务接口 | `memory.go` |
| `artifact/` | Artifact 存储接口 | `artifact.go` |
| `artifact/gcsartifact/` | Google Cloud Storage 实现 | `gcsartifact.go` |
| `telemetry/` | OpenTelemetry 集成 | `telemetry.go` |
| `util/` | 共享工具函数 | `instructionutil/` |

### 命令行工具

| 目录 | 主要职责 | 关键文件 |
|------|---------|----------|
| `cmd/launcher/` | 启动器框架，支持不同部署模式 | `full/launcher.go`, `prod/launcher.go` |
| `cmd/adkgo/` | CLI 工具 | `main.go` |

### 示例代码

| 目录 | 主要职责 | 关键文件 |
|------|---------|----------|
| `examples/quickstart/` | 简单的天气/时间 Agent 示例 | `main.go` |
| `examples/rest/` | REST API 服务器部署示例 | `main.go` |
| `examples/a2a/` | Agent-to-Agent 服务器示例 | `main.go` |
| `examples/web/` | Web UI 示例 | `main.go` |
| `examples/mcp/` | Model Context Protocol 集成示例 | `main.go` |
| `examples/vertexai/` | Vertex AI 集成示例 | `imagegenerator/main.go` |
| `examples/tools/` | 自定义工具示例 | `multipletools/`, `loadartifacts/` |
| `examples/workflowagents/` | 工作流 Agent 模式示例 | `loop/`, `parallel/`, `sequential/` |

### 内部包（非公共 API）

| 目录 | 主要职责 | 备注 |
|------|---------|------|
| `internal/agent/` | Agent 内部实现细节 | 包含 `parentmap`, `runconfig` |
| `internal/artifact/` | Artifact 内部实现 | - |
| `internal/cli/` | CLI 内部工具 | - |
| `internal/context/` | 上下文内部实现 | - |
| `internal/converters/` | 类型转换器 | - |
| `internal/httprr/` | HTTP 记录/重放工具 | 有独立的 LICENSE |
| `internal/llminternal/` | LLM 内部实现 | - |
| `internal/memory/` | Memory 内部实现 | - |
| `internal/sessioninternal/` | Session 内部实现 | - |
| `internal/sessionutils/` | Session 工具函数 | - |
| `internal/telemetry/` | Telemetry 内部实现 | - |
| `internal/testutil/` | 测试工具 | - |
| `internal/toolinternal/` | Tool 内部实现 | - |
| `internal/typeutil/` | 类型工具 | - |
| `internal/utils/` | 通用工具函数 | - |
| `internal/version/` | 版本管理 | - |

---

## 构建与运行方式

### 构建

```bash
# 构建所有包
go build -mod=readonly -v ./...

# 验证依赖
go mod tidy -diff
```

### 测试

```bash
# 运行所有测试
go test -mod=readonly -v ./...

# 带竞态检测的测试（nightly 模式）
go test -race -mod=readonly -v -count=1 -shuffle=on ./...

# 运行特定测试
go test -mod=readonly -v ./path/to/package -run TestName
```

### 代码质量检查

```bash
# 运行 linter（需要 golangci-lint v2.3.1+）
golangci-lint run

# 项目使用 .golangci.yml 配置，包含：
# - goimports 格式化
# - goheader linter（强制 Apache 2.0 许可证头）
# - 自定义 staticcheck 配置
```

### 运行示例

```bash
# 通用模式
go run ./examples/<example_name>/main.go [options]

# Quickstart 示例（不同启动器）
go run ./examples/quickstart/main.go help          # 显示可用选项
go run ./examples/quickstart/main.go console       # 控制台模式
go run ./examples/quickstart/main.go restapi       # REST API 服务器
go run ./examples/quickstart/main.go a2a           # Agent-to-Agent 服务器
go run ./examples/quickstart/main.go webui         # Web UI

# 其他示例
go run ./examples/tools/multipletools/main.go      # 多工具示例
go run ./examples/workflowagents/loop/main.go      # 循环 Agent 示例
go run ./examples/rest/main.go                     # REST 服务器示例
```

### 安装依赖

```bash
# 添加 ADK Go 到你的项目
go get google.golang.org/adk
```

---

## 外部依赖

### 核心依赖

| 依赖 | 用途 | 版本 |
|------|------|------|
| `google.golang.org/genai` | Gemini 模型集成（主要 LLM） | v1.20.0 |
| `cloud.google.com/go/storage` | Google Cloud Storage（Artifact 存储） | v1.56.1 |
| `gorm.io/gorm` | ORM 框架（Session 数据库） | v1.31.0 |
| `gorm.io/driver/sqlite` | SQLite 驱动（Session 持久化） | v1.6.0 |
| `github.com/a2aproject/a2a-go` | Agent-to-Agent 协议 | v0.3.0 |
| `github.com/modelcontextprotocol/go-sdk` | Model Context Protocol | v0.7.0 |

### 可观测性与遥测

| 依赖 | 用途 |
|------|------|
| `go.opentelemetry.io/otel` | OpenTelemetry 集成 |
| `go.opentelemetry.io/otel/sdk` | OpenTelemetry SDK |
| `go.opentelemetry.io/otel/trace` | 分布式追踪 |

### 工具与框架

| 依赖 | 用途 |
|------|------|
| `github.com/gorilla/mux` | HTTP 路由（REST API） |
| `github.com/spf13/cobra` | CLI 框架 |
| `github.com/google/uuid` | UUID 生成 |
| `golang.org/x/sync` | 并发控制 |

### 数据库

- **SQLite**（默认）：用于本地开发和测试
- **可扩展**：通过 GORM 支持其他数据库（PostgreSQL, MySQL 等）

### 外部服务

| 服务 | 用途 | 必需性 |
|------|------|--------|
| **Gemini API** | LLM 模型服务 | 必需（运行 LLM Agent） |
| **Google Cloud Storage** | Artifact 存储 | 可选（使用 GCS Artifact） |
| **Vertex AI** | Vertex AI 集成 | 可选（使用 Vertex AI 特性） |

---

## 新手阅读顺序

### 第一步：理解项目基础
1. **README.md** - 了解项目简介和核心特性
2. **CLAUDE.md** - 了解项目架构、开发规范和常用命令
3. **CONTRIBUTING.md** - 了解贡献指南和与 adk-python 的对齐关系

### 第二步：通过示例学习
4. **examples/quickstart/main.go** - 最简单的 Agent 示例，了解基本用法
   - 查看如何创建 Agent
   - 理解 Launcher 的不同模式（console, restapi, a2a, webui）
5. **examples/tools/** - 学习如何创建和使用自定义工具
6. **examples/workflowagents/** - 了解 Agent 编排模式

### 第三步：核心概念与接口
7. **agent/agent.go** - Agent 基础接口，理解 `Agent` 接口定义
   - `Agent.Run()` 返回事件迭代器
   - `SubAgents()` 层级委托
8. **session/session.go** - Session 和 Event 的概念
   - Session 如何跟踪用户-Agent 交互
   - Event 的不可变性和流式处理
9. **tool/tool.go** - Tool 接口，理解工具如何扩展 Agent 能力
10. **runner/runner.go** - Runner 如何在 Session 中执行 Agent

### 第四步：深入理解实现
11. **agent/llmagent/llmagent.go** - LLM Agent 的实现细节
    - 如何与 LLM 模型交互
    - 工具调用机制
12. **agent/workflowagents/** - 工作流 Agent 的实现
    - LoopAgent：循环执行直到满足退出条件
    - ParallelAgent：并发执行多个 Sub-Agent
    - SequentialAgent：顺序执行 Sub-Agent
13. **model/gemini/gemini.go** - Gemini 模型集成

### 第五步：服务器与部署
14. **server/adkrest/server.go** - REST API 服务器实现
15. **server/adka2a/server.go** - Agent-to-Agent 协议服务器
16. **cmd/launcher/** - 启动器框架
    - `full/launcher.go`：完整启动器（支持所有模式）
    - `prod/launcher.go`：生产启动器（仅 REST 和 A2A）

### 第六步：高级特性
17. **memory/memory.go** - 语义内存服务
18. **artifact/artifact.go** - Artifact 存储
19. **telemetry/telemetry.go** - OpenTelemetry 集成
20. **session/database/database.go** - Session 持久化

### 推荐阅读路径总结

```
README.md → CLAUDE.md → examples/quickstart
    ↓
agent/agent.go → session/session.go → tool/tool.go → runner/runner.go
    ↓
agent/llmagent/ → agent/workflowagents/
    ↓
server/ → cmd/launcher/ → model/gemini/
    ↓
memory/ → artifact/ → telemetry/ → session/database/
```

---

## 架构核心概念

### Agent 执行模型

```
Runner (运行时)
  ├─ Session (会话管理)
  │   ├─ State (可变状态：key-value 存储)
  │   └─ Events (不可变事件序列)
  ├─ Agent (核心抽象)
  │   ├─ Run() → iter.Seq2[*Event, error]
  │   ├─ SubAgents (层级委托)
  │   └─ Callbacks (前置/后置回调)
  └─ Services (可选服务)
      ├─ SessionService (必需)
      ├─ ArtifactService (可选)
      └─ MemoryService (可选)
```

### 事件驱动架构

- **Event**：不可变的交互记录（用户输入、模型响应、函数调用等）
- **EventActions**：可变操作（修改 State、转移控制、执行工具等）
- **iter.Seq2**：Go 1.24+ 迭代器，支持流式处理和背压控制

### Launcher 模式

- **Full Launcher** (`full.NewLauncher`): 支持 console、REST API、A2A、WebUI
- **Prod Launcher** (`prod.NewLauncher`): 仅支持 REST API 和 A2A（生产环境）

---

## 代码风格与约定

### 许可证头
所有新 Go 文件必须包含 Apache 2.0 许可证头（由 `goheader` linter 强制执行）。

### 风格指南
- 遵循 [Google Go Style Guide](https://google.github.io/styleguide/go/index)
- 使用 `goimports` 格式化
- 清晰、描述性的测试名称
- 优先使用表驱动测试
- 保持 PR 小而专注（每个 PR 一个关注点）
- 为所有导出的类型、函数和常量添加 godoc 注释

### 测试要求
- 为所有新功能和 bug 修复添加单元测试
- 覆盖边界情况、错误条件和典型用例
- 测试应快速、隔离、使用 mock/fixture（无外部依赖）
- 对于 Agent 变更，在 PR 中提供手动 E2E 测试证据

---

## 对齐 adk-python

根据 CONTRIBUTING.md，**adk-python 是真理源**。实现功能时：
1. 检查 adk-python 的参考实现
2. 尽可能保持 API 兼容性
3. 适配 Go 习惯用法（例如，迭代器代替异步生成器）
4. 验证行为与 Python 版本一致

---

## 相关项目

- **adk-python**: Python 版本（真理源）
- **adk-java**: Java 版本
- **adk-web**: Web 版本
- **adk-samples**: 生产级示例（本仓库示例仅为最小测试）
- **adk-docs**: 官方文档 - https://google.github.io/adk-docs/

---

## 注意事项

1. **internal/httprr** 有自己的许可证（见 `internal/httprr/LICENSE`）
2. 本仓库的示例是最小测试，生产级示例见 [adk-samples](https://github.com/google/adk-samples)
3. 官方文档：https://google.github.io/adk-docs/
4. Go 版本要求：**1.24.4+**（使用迭代器特性）

---

## 下一步

- **02-entrypoint.md**: 程序入口与启动流程
- **03-callchains.md**: 核心调用链
- **04-modules.md**: 模块依赖与数据流
- **05-architecture.md**: 系统架构
