# HermesX

<div align="center">

**桌面 Agent 工作台**

[![Version](https://img.shields.io/badge/version-0.1.2-blue)](package.json)
[![Tauri](https://img.shields.io/badge/tauri-2-ffc131)](src-tauri/Cargo.toml)
[![React](https://img.shields.io/badge/react-19-61dafb)](package.json)
[![TypeScript](https://img.shields.io/badge/typescript-6.0-3178c6)](package.json)
[![Rust](https://img.shields.io/badge/rust-1.95+-orange)](src-tauri/Cargo.toml)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

</div>

---

## 项目定位

HermesX 是一个基于 Tauri 的桌面端 AI 工作台，用来连接 Hermes 能力后端、本地 AI 模型配置、本机文件工具、任务执行面板、记忆系统、MCP 工具和桌面宠物反馈。

它的核心目标不是再做一个聊天框，而是把“对话、任务、工具、文件、权限、Agent 进度”放在同一个本机工作台里。远程 Agent 可以参与分析和执行，但本地文件访问、写入、命令执行和权限确认始终由 HermesX 控制。

当前项目处于早期桌面应用阶段，功能已经可用，但仍在快速迭代。README 以当前代码库实际能力为准，不把路线图或实验性想法写成已经完成的能力。

## 主要功能

### 会话工作台

- 支持直接向配置好的 AI 模型提问。
- 支持通过 `@agent` 指定一个 Agent，或通过 `@all` 派发给在线 Agent。
- 发送任务后会立即显示接单状态，避免长时间空白等待。
- 会话中可以携带图片和文件附件，附件会保存到 HermesX 本机数据目录。
- 对本地文件、目录、命令等需求会先经过 HermesX 的本机上下文和权限流程。

### Agent 协作

- 可连接多个 Hermes 实例，并聚合实例中的 Agent 配置。
- 会话工作台左侧展示实例、在线状态和 Agent 数量。
- 支持按实例/Agent 精确定位目标，避免同名 Agent 混淆。
- Agent 任务进度、日志和最终结果会回流到当前会话。
- 远程 Agent 请求本地工具时，HermesX 会在用户本机执行，并把结果回传。

### 本地工具与权限边界

HermesX 提供本机工具执行层，常用能力包括：

| 类别 | 能力 |
| --- | --- |
| 文件 | 读取文件、写入文件、读取目录、移动文件、创建目录、删除路径 |
| 搜索 | 文件名搜索、文本搜索、路径检查 |
| 命令 | 在当前工作区执行 shell 命令 |
| Git | 状态、diff、日志、提交、分支等操作 |
| 桌面 | 无障碍树快照、窗口列表 |
| MCP | 连接外部 MCP Server 并注册工具 |

安全策略采用本机优先的边界：

- 当前工作区是本地文件操作的默认上下文。
- 写入、删除、移动、命令执行等高风险操作需要确认。
- `.git`、`.env`、密钥文件、系统目录等敏感路径会被限制或阻止。
- 远程 Agent 不能把自己的服务器目录当成 HermesX 的本机工作区。

### 代码索引与记忆系统

记忆系统分为几类数据，不是单一的聊天记录表：

- **代码索引 / RAG**：对用户选择的工作区进行扫描、切块和 embedding 存储，供语义搜索和 Agent 对话召回相关代码片段。
- **经验教训**：从工具调用和任务结果中沉淀使用模式、失败信息和会话摘要。
- **项目知识**：记录项目相关约定、技术栈和常见错误。
- **用户/Agent 记忆快照**：供后续会话注入上下文。

在“设置 → 记忆系统 → 代码索引”中可以直接选择要索引的工作区，然后点击“建立索引”。索引不会在项目目录里生成文件，而是写入 HermesX 的本机 SQLite 数据库。

默认数据库位置为 Tauri 的应用本地数据目录：

```text
app_local_data_dir()/memory.db
```

Windows 上通常类似：

```text
C:\Users\<用户名>\AppData\Local\com.hermes.hermesx\memory.db
```

SQLite 启用了 WAL 模式，因此同目录下可能出现 `memory.db-wal` 和 `memory.db-shm`，这是正常现象。

### 工作流

HermesX 内置可视化工作流面板：

- 通过模板创建工作流。
- 在画布中查看和调整节点。
- 节点可以配置模型、Agent、输入、提示词、输出变量和输出路径。
- 执行进度在右侧抽屉中展示，不污染普通会话。
- 节点状态包括 `pending`、`running`、`completed`、`failed`、`skipped`。

### MCP 工具

设置中心包含 MCP 管理页，可添加、连接和停止 MCP Server。连接成功后，MCP 工具会进入 HermesX 的动态工具注册表，供本地执行层和 Agent 任务使用。

内置预设包括：

- Filesystem
- GitHub
- Fetch
- Memory
- PostgreSQL
- GSAP

### 桌面宠物

桌面宠物用于展示任务状态反馈：

- 支持独立透明置顶窗口。
- 支持角色选择、尺寸、位置和行为设置。
- 根据任务状态切换空闲、思考、工作、成功、失败等状态。
- 设置页可预览真实动画帧。

## 技术架构

HermesX 由 React 前端和 Tauri/Rust 后端组成。

```text
src/
  components/                 React UI
    Workspace.tsx             会话工作台
    CommandBar.tsx            输入框、模型选择、@agent 派发
    Sidebar.tsx               协作实例侧栏
    Settings.tsx              设置中心
    WorkflowBuilder.tsx       工作流画布
    settings/MemoryPanel.tsx  记忆系统与代码索引

  services/                   前端业务层
    providers.ts              AI Provider 调用
    toolRegistry.ts           工具定义
    toolExecutor.ts           工具执行与权限处理
    localToolAgent.ts         本地工具 Agent 循环
    ragEngine.ts              代码索引与语义搜索
    crossSessionLearning.ts   跨会话学习
    workflowEngine.ts         工作流执行
    mcp/                      MCP 客户端与管理器

  store/                      Zustand 状态
    useHermesXStore.ts        全局配置、工作区、记忆
    useConversationStore.ts   会话与消息
    useAgentConnectionStore.ts 实例连接状态
    useAgentExecutionStore.ts Agent 任务执行

src-tauri/
  src/
    main.rs                   Tauri 启动与命令注册
    commands/                 Tauri 命令
      fs.rs                   本地文件系统命令
      http.rs                 HTTP/SSE 代理
      memory.rs               记忆数据库命令
      data.rs                 本机数据目录与附件
      pty.rs                  终端会话
      watch.rs                文件监听
      pet.rs                  桌面宠物命令
    memory.rs                 SQLite 记忆存储
    data_dir.rs               应用数据目录管理
    mcp_server.rs             HermesX 本地 MCP 服务
    pty_manager.rs            PTY 管理
    watch_manager.rs          文件监听管理
```

## 技术背景与设计依据

HermesX 的设计不是从零发明一套概念，而是把桌面 Agent、编程 Agent、长期记忆、技能纪律、GUI 感知和本地安全边界等方向的成熟实践，整理到一个本机桌面工作台中。这里列出的项目不是依赖清单，也不是能力背书，而是说明 HermesX 在设计时重点关注过哪些问题，以及当前代码库如何落地这些问题。

### 桌面 Agent 与 GUI 感知

| 项目 | 关注点 | HermesX 中对应的问题 |
| --- | --- | --- |
| agent-desktop | 无障碍树遍历、结构化 UI 树、按需深入读取 | 桌面感知不应只依赖截图；HermesX 通过 `a11y_snapshot`、`a11y_list_windows` 暴露结构化桌面上下文。 |
| Cua | 后台桌面控制、避免打扰用户焦点 | HermesX 的桌面感知侧重读取和上下文提供，默认不接管鼠标、焦点和用户正在进行的操作。 |
| Microsoft UFO | 多 Agent 分工、Host/App 层级、Windows UI Automation | HermesX 的任务调度、工作流和 Agent 派发都围绕“拆解、分派、回流”组织。 |
| OpenAdapt | 桌面事件记录与回放 | HermesX 的文件监听、任务日志和执行面板关注可观察性与可追溯过程。 |
| OS-Copilot | OS 级 Agent、自改进循环 | HermesX 的经验教训、项目知识和任务历史用于让后续任务获得更好的本机上下文。 |
| AgentDesk | 隔离环境与远程桌面式 Agent 工作区 | HermesX 不把远程环境当成本机项目，而是把本地文件权限保留在桌面端。 |
| OpenPawz | Tauri 桌面 Agent、Fleet 式多 Agent 编排 | HermesX 采用 Tauri/Rust 桌面架构，并把多实例、多 Agent 聚合进会话工作台。 |

### 编程 Agent 与工具循环

| 项目 | 关注点 | HermesX 中对应的问题 |
| --- | --- | --- |
| Claude Code | 工具调用循环、上下文压缩、子任务委派 | HermesX 的本地工具 Agent、远程工具回传和上下文压缩都围绕“模型推理，本机执行”展开。 |
| Cline | IDE 内文件编辑、命令执行、用户确认 | HermesX 的文件读写、命令执行和授权卡片延续了“工具动作必须可见”的原则。 |
| Aider | 仓库结构理解、多文件编辑、Git 辅助 | HermesX 的项目上下文、Git 工具和代码索引都面向真实项目工作区。 |
| Goose | Rust Agent 框架、MCP/ACP 扩展 | HermesX 后端采用 Rust/Tauri，并通过 MCP 管理页接入外部工具。 |
| OpenHands | Agent 运行环境、技能与任务工作区 | HermesX 的执行面板、工作流画布和工具中继提供桌面端任务运行面。 |
| Continue | 模型切换、上下文提供者、IDE 辅助 | HermesX 的 Provider 管理、模型选择器和本机上下文采集面向多模型使用场景。 |

### 记忆、上下文与技能纪律

| 项目 | 关注点 | HermesX 中对应的问题 |
| --- | --- | --- |
| Mem0 | ADD-only 记忆、实体和时间信号、长期记忆检索 | HermesX 保留追加式记忆、经验教训和项目知识，不把历史经验粗暴覆盖成一条最新摘要。 |
| agentmemory | 类型化记忆、版本链、上下文优先级 | HermesX 的记忆条目包含类型、标签、强度、版本、TTL 和来源任务等字段。 |
| Letta | Agent 状态持久化、记忆块管理 | HermesX 将用户偏好、项目事实、经验教训和代码索引分开存储，避免所有上下文混在聊天记录里。 |
| screenpipe | 本地活动感知、长期记录、隐私边界 | HermesX 的日志、文件监听和桌面感知能力都优先落在本机数据目录和本机权限模型内。 |
| OmniParser | GUI 截图到结构化元素 | HermesX 当前主路径是无障碍树，视觉解析可作为未来补充，而不是替代本机结构化上下文。 |
| WindowsAgentArena | 桌面 Agent 评测维度 | HermesX 的执行面板和日志结构为后续任务质量评估保留了数据基础。 |
| obra/superpowers | Agent 工作流纪律、TDD、计划、审查 | HermesX 的技能系统支持按需加载流程技能，帮助 Agent 在复杂任务中保持执行纪律。 |
| everything-claude-code | 自调试、错误模式、验证闭环 | HermesX 内置 `agentDebugger`、`verifier`、错误模式和提交检查相关能力。 |

## 关键实现取舍

这一部分说明上述设计依据在 HermesX 当前代码中的落点。它不是路线图，而是当前实现中已经存在或正在使用的工程结构。

### Tauri 2 与 Rust 后端

HermesX 早期形态更接近 Web/Electron 式桌面应用，但当前主线已经转向 Tauri 2。这样做的目的不是追求“更酷的技术栈”，而是为了把本机文件、进程、SQLite、PTY、系统托盘、桌面宠物窗口和 HTTP/SSE 代理放在更明确的本机边界里。

当前后端主要模块：

- `commands/fs.rs`：本机文件系统读写、搜索、路径检查。
- `commands/http.rs`：模型请求和 SSE 请求代理，减少浏览器环境限制。
- `commands/memory.rs` 与 `memory.rs`：SQLite 记忆数据库、全文搜索、工具统计。
- `commands/pty.rs` 与 `pty_manager.rs`：终端会话。
- `commands/watch.rs` 与 `watch_manager.rs`：文件监听。
- `commands/data.rs` 与 `data_dir.rs`：应用本地数据目录、附件和文档存储。
- `mcp_server.rs`：HermesX 本地 MCP 服务。
- `pet.rs`：桌面宠物窗口和状态反馈。

前端通过 `runtime.ts` 暴露统一桥接接口。React 组件不直接关心 Tauri command 名称，而是使用 `window.hermesX` 下的桥接 API。

### 远程 Agent 与本机执行分离

HermesX 将远程 Hermes Agent 视为能力后端，而不是项目运行环境。这个边界非常重要：

- 远程 Agent 可以分析、规划和生成。
- 远程 Agent 不能把自己的服务器目录当成用户项目。
- 需要读取、搜索、写入、执行命令时，必须回到 HermesX 本机工具层。
- HermesX 根据工作区、授权记录和敏感路径规则决定是否执行。

对应代码路径：

- `useAgentConnectionStore.ts`：实例连接与 Agent 状态。
- `useAgentExecutionStore.ts`：Agent 任务创建、执行、失败和完成回流。
- `remoteAgentToolBridge.ts`：远程 Agent 请求本地工具时的桥接。
- `agentBriefing.ts`：派发前把本机边界、会话上下文、记忆和工具规则写入任务简报。

### 本地工具中继

HermesX 的工具体系不是分散在多个按钮里的辅助功能，而是统一注册、统一执行、统一预算、统一展示。

- `toolRegistry.ts` 描述工具能力、参数和权限等级。
- `toolExecutor.ts` 负责参数校正、权限判断、实际执行和结果富化。
- `toolBudget.ts` 控制工具结果长度，避免长文件或命令输出淹没上下文。
- `localToolAgent.ts` 让本地模型按“思考 → 工具调用 → 观察 → 继续”的循环处理本机任务。

这套结构让普通 AI 对话、远程 Agent 工具请求、工作流节点和 MCP 工具能尽量走同一个执行边界。

### 记忆系统与代码索引

HermesX 的记忆不是单一字段，而是多种数据的组合：

- IndexedDB / 持久化状态：应用配置、会话、用户/Agent 记忆快照。
- SQLite `memory.db`：代码索引、经验教训、会话摘要、项目知识、工具统计。
- 内存缓存：运行时索引、Agent 操作状态、任务结果缓存。

RAG 代码索引在 `ragEngine.ts` 中完成：

1. 扫描用户选择的工作区。
2. 忽略 `node_modules`、`.git`、`dist`、`build`、`target` 等目录。
3. 对支持的文本/代码文件切块。
4. 为 chunk 生成 embedding。
5. 通过 Tauri `memory_store` 写入 SQLite，类型为 `code_chunk`。

跨会话学习在 `crossSessionLearning.ts` 中完成，主要包括：

- 从工具调用中提取成功模式和失败教训。
- 保存会话摘要。
- 统计工具使用成功率。
- 在后续任务中检索相关历史经验。

### 上下文压缩与多 Agent 路由

长任务和多 Agent 协作最容易出现的问题是：日志越来越长，模型看到的上下文越来越混乱。HermesX 对此采用分层处理：

- 工具结果摘要化：长文件读取、命令输出和工具结果会被结构化裁剪。
- 会话上下文压缩：历史消息按预算压缩，避免无关内容持续占用窗口。
- 多 Agent 上下文路由：一个 Agent 不直接读取所有 Agent 的完整日志，而是看到与当前任务相关的摘要。

对应模块：

- `contextCompressor.ts`
- `multiAgentContext.ts`
- `conversationTimeline.ts`
- `agentBriefing.ts`

### 工作流执行隔离

工作流不是普通聊天的另一种展示形式，而是独立的任务执行面：

- `WorkflowBuilder.tsx` 提供模板、画布、节点配置和运行抽屉。
- `workflowEngine.ts` 按节点依赖执行 DAG。
- `useWorkflowStore.ts` 保存工作流状态。
- `useAgentExecutionStore.ts` 支持 `silent` 任务，让工作流结果回到工作流抽屉，而不是污染会话区。

这样普通会话保持清爽，工作流任务仍然可观察和可复盘。

### 技能系统

HermesX 的技能不是用户手动复制进 prompt 的长文本，而是通过技能目录和按需加载机制进入 Agent 工作流。

当前技能来源包括：

- 项目级技能目录。
- 用户级技能目录。
- 插件或运行时提供的技能。

主要相关模块：

- `skillsEngine.ts`：加载和构建技能目录。
- `skillRouting.ts`：任务与技能的匹配。
- `skillCurator.ts`：任务完成后的知识沉淀提示。
- `toolRegistry.ts` / `toolExecutor.ts`：提供 `hermesx_load_skill` 等工具入口。

技能的目标不是让 Agent “看起来更聪明”，而是让它在设计、实现、调试、验证、审查等流程中更稳定地遵守操作纪律。

## 数据存储

HermesX 会在 Tauri 应用本地数据目录下保存运行数据。实际目录可在应用内相关页面查看。

| 数据 | 存储位置 |
| --- | --- |
| 应用配置 | Tauri Store / HermesX 持久化配置 |
| 会话记录 | HermesX 本机持久化存储 |
| 附件、图片、生成文档 | 应用本地数据目录下的 `storage/images`、`storage/documents` |
| 代码索引、经验教训、项目知识 | `memory.db` SQLite 数据库 |
| 工具调用日志 | 应用内日志系统与部分 SQLite 统计 |

代码索引数据存入 `memories` 表，类型为 `code_chunk`。embedding 以 BLOB 形式写入数据库，不会写到被索引项目目录里。

## 快速开始

### 环境要求

- Node.js 20 或更新版本
- npm
- Rust 1.95 或更新版本
- 系统 WebView 运行环境

Windows 通常需要 Visual Studio Build Tools 和 Windows SDK。Linux 需要 WebKitGTK、GTK、AppIndicator 等 Tauri 依赖。

### 安装依赖

```bash
npm install
```

### 启动桌面开发环境

```bash
npm run dev
```

等价命令：

```bash
npm run tauri:dev
```

### 仅启动 Web 前端

```bash
npm run dev:web
```

这个模式适合调试前端界面。涉及 Tauri IPC、本地文件、PTY、桌宠窗口等能力时，应使用桌面开发环境。

## 构建

### 构建桌面应用

```bash
npm run build
```

或：

```bash
npm run tauri:build
```

### 构建前端资源

```bash
npm run build:web
```

构建产物位置由 Tauri 决定，常见路径包括：

```text
src-tauri/target/release/
src-tauri/target/release/bundle/
```

Windows NSIS 安装包通常位于：

```text
src-tauri/target/release/bundle/nsis/
```

## 测试与检查

```bash
npm run test
npm run build:web
```

Rust 检查：

```bash
cd src-tauri
cargo check
```

## 配置说明

### AI Provider

在设置中心配置 Provider，包括 OpenAI 兼容接口、Anthropic、Ollama 或自定义接口。会话工作台可以选择当前使用的模型。

### Hermes 实例

在连接设置中添加 Hermes 实例地址。实例连接成功后，其 Agent 会出现在协作侧栏，可通过 `@agent` 派发任务。

### 工作区

工作区是 HermesX 对本机项目的主要上下文。它影响：

- 文件读取和搜索的默认目录
- 命令执行的默认工作目录
- 代码索引的目标目录
- Agent 简报中的本机项目边界

工作区可以在会话输入栏的“目录”按钮设置，也可以在“记忆系统 → 代码索引”页中选择。

### MCP

在 MCP 设置页添加服务器命令、参数和环境变量。连接成功后工具会动态注册。需要访问外部服务的 MCP Server 可能要求额外环境变量或凭据。

## 常见问题

### 为什么代码索引显示 0？

常见原因：

- 尚未选择工作区。
- 选择的目录不是项目根目录。
- 目录内没有支持的文本或代码文件。
- 文件位于忽略目录中，例如 `node_modules`、`.git`、`dist`、`build`、`target`。
- embedding 或数据库写入失败。

支持的常见扩展包括：

```text
.ts .tsx .js .jsx .vue .svelte .rs .py .go .java .c .cpp .h .hpp
.css .scss .less .html .json .yaml .yml .md .txt .sh .bash .sql
```

### 索引后项目里为什么没有新文件？

这是正常的。索引数据写入 HermesX 的 `memory.db`，不会写入被索引的项目目录。

### `@agent` 和直接问 AI 有什么区别？

直接问 AI 使用当前选择的模型进行回复。`@agent` 会把任务派发给连接的 Hermes Agent，并附带 HermesX 当前会话、工作区、文件上下文和权限边界。Agent 需要本机文件或命令时，会通过 HermesX 请求本地工具执行。

### 什么时候需要授权？

读取未授权路径、写入文件、删除文件、移动路径、执行命令等操作可能需要确认。授权由 HermesX 控制，不由远程 Agent 自行决定。

### 可以只当普通 AI 聊天工具用吗？

可以。配置一个 AI Provider 后，可以直接在会话工作台提问；不连接 Hermes 实例也可以使用普通模型问答。

## 当前限制

- 部分桌面感知能力依赖操作系统支持和平台适配。
- 远程 Agent 能否稳定执行取决于对应 Hermes 实例的连接状态和接口能力。
- 代码索引目前以本地 SQLite 存储为主，适合项目级检索，不是通用大型向量数据库。
- 工作流、记忆、MCP 等模块仍在迭代，配置格式和界面可能继续调整。

## 贡献

欢迎提交 issue、建议和 pull request。提交代码前建议至少运行：

```bash
npm run build:web
```

涉及 Rust 后端时，请同时运行：

```bash
cd src-tauri
cargo check
```

## 许可证

MIT © [wwjiang0328](https://github.com/wwjiang0328)
