# Claude Code 源码技术架构解析

## 一、项目概况

| 指标 | 数据 |
|------|------|
| 文件数量 | ~1,900 个文件 |
| 代码行数 | 512,000+ 行 |
| 语言 | TypeScript (strict) |
| 运行时 | Bun |
| UI 框架 | React + Ink (终端 UI) |

## 二、整体架构

```
src/
├── main.tsx                 # CLI 入口（Commander.js 解析器）
├── QueryEngine.ts           # 对话引擎核心（~46K 行）
├── Tool.ts                  # 工具类型定义（~29K 行）
├── commands.ts              # 命令注册中心（~25K 行）
├── query.ts                 # 查询执行循环
├── context.ts               # 系统/用户上下文构建
├── cost-tracker.ts          # Token 费用追踪
│
├── tools/                   # 40+ 内置工具实现
├── commands/                # 50+ 斜杠命令实现
├── components/              # 144 个 React/Ink UI 组件
├── hooks/                   # 80+ React Hooks
├── services/                # 外部服务集成层
├── bridge/                  # IDE 桥接层（VS Code/JetBrains）
├── coordinator/             # 多 Agent 协调器
├── skills/                  # 技能系统
├── plugins/                 # 插件系统
├── state/                   # 全局状态管理
├── vim/                     # Vim 模式
├── voice/                   # 语音输入
├── remote/                  # 远程会话
├── server/                  # 服务器模式
├── memdir/                  # 持久化记忆系统
├── tasks/                   # 任务管理
├── screens/                 # 全屏 UI（Doctor、REPL、Resume）
├── schemas/                 # Zod 配置校验
└── entrypoints/             # 多入口初始化逻辑
```

## 三、核心系统

### 3.1 工具系统（Tool System）

Claude Code 的所有能力通过工具（Tool）暴露给 LLM。每个工具都是自包含模块，定义输入 Schema、权限模型和执行逻辑。

| 工具分类 | 工具名 | 功能 |
|----------|--------|------|
| **文件操作** | FileReadTool | 读取文件（支持图片、PDF、Notebook） |
| | FileWriteTool | 创建/覆写文件 |
| | FileEditTool | 局部文件修改（字符串替换） |
| **搜索** | GlobTool | 文件模式匹配搜索 |
| | GrepTool | 基于 ripgrep 的内容搜索 |
| **执行** | BashTool | Shell 命令执行 |
| | PowerShellTool | Windows PowerShell 执行 |
| | REPLTool | 交互式 REPL |
| **网络** | WebFetchTool | URL 内容抓取 |
| | WebSearchTool | 网页搜索 |
| **Agent** | AgentTool | 子 Agent 生成 |
| | SendMessageTool | Agent 间消息传递 |
| | TeamCreateTool/TeamDeleteTool | 团队 Agent 管理 |
| **协议** | MCPTool | MCP 服务器工具调用 |
| | LSPTool | 语言服务器协议集成 |
| **任务** | TaskCreateTool/TaskUpdateTool | 任务创建与管理 |
| | TaskGetTool/TaskListTool | 任务查询 |
| **模式** | EnterPlanModeTool/ExitPlanModeTool | 计划模式切换 |
| | EnterWorktreeTool/ExitWorktreeTool | Git Worktree 隔离 |
| **其他** | SkillTool | 技能执行 |
| | ScheduleCronTool | 定时任务调度 |
| | SleepTool | 主动模式等待 |
| | SyntheticOutputTool | 结构化输出生成 |
| | ToolSearchTool | 延迟工具发现 |

### 3.2 命令系统（Command System）

src/commands/ 目录下有 50+ 个斜杠命令：

**核心开发命令：**
- `/commit` — Git 提交
- `/diff` — 查看变更
- `/review` — 代码审查
- `/compact` — 上下文压缩

**配置与管理：**
- `/config` — 设置管理
- `/mcp` — MCP 服务器管理
- `/memory` — 持久化记忆管理
- `/skills` — 技能管理
- `/hooks` — 钩子配置

**高级功能：**
- `/vim` — Vim 模式切换
- `/doctor` — 环境诊断
- `/cost` — 使用费用查看
- `/context` — 上下文可视化
- `/desktop` / `/mobile` — 跨平台交接

### 3.3 QueryEngine — 对话引擎核心

QueryEngine.ts（约 46,000 行）负责：

- **流式响应处理：**与 Anthropic API 的流式通信
- **工具调用循环：**LLM 输出 → 解析工具调用 → 执行工具 → 结果回传 → LLM 继续
- **Thinking 模式：**Extended Thinking 的启用与管理
- **重试逻辑：**API 错误的优雅降级与重试
- **Token 计数：**精确的 Token 使用量追踪

### 3.4 权限系统（Permission System）

src/hooks/toolPermission/ 实现多层权限模型：

```
工具调用请求 → 权限检查 → [自动通过 | 提示用户 | 拒绝]
```

权限模式：
- `default` — 默认模式，危险操作需确认
- `plan` — 计划模式，仅允许只读操作
- `auto` — 自动模式，信任所有操作
- `bypassPermissions` — 跳过权限（开发/测试用）

### 3.5 Bridge 系统 — IDE 集成

src/bridge/ 实现 CLI 与 IDE 扩展的双向通信：

- `bridgeMain.ts` — 桥接主循环
- `bridgeMessaging.ts` — 消息协议
- `bridgePermissionCallbacks.ts` — 权限回调
- `replBridge.ts` — REPL 会话桥接
- `jwtUtils.ts` — JWT 认证
- `sessionRunner.ts` — 会话执行管理

### 3.6 Agent 协调系统

src/coordinator/ 实现多 Agent 协调机制：

- **AgentTool：**生成子 Agent 处理独立子任务
- **SendMessageTool：**Agent 间通信
- **TeamCreateTool：**创建 Agent 团队并行处理
- `useSwarmInitialization.ts` / `useSwarmPermissionPoller.ts` — Swarm 模式初始化与权限轮询

## 四、服务层架构

src/services/ 外部依赖集成：

| 服务 | 说明 |
|------|------|
| `api/` | Anthropic API 客户端、文件 API、Bootstrap |
| `mcp/` | Model Context Protocol 服务器连接管理 |
| `oauth/` | OAuth 2.0 认证流程 |
| `lsp/` | Language Server Protocol 管理器 |
| `analytics/` | GrowthBook 特性标志与分析 |
| `plugins/` | 插件加载器 |
| `compact/` | 对话上下文压缩 |
| `extractMemories/` | 自动记忆提取 |
| `tokenEstimation.ts` | Token 数量估算 |
| `teamMemorySync/` | 团队记忆同步 |
| `policyLimits/` | 组织策略限制 |
| `remoteManagedSettings/` | 远程托管设置 |
| `MagicDocs/` | 智能文档 |
| `PromptSuggestion/` | 提示词建议 |
| `autoDream/` | 自动记忆处理 |

## 五、UI 组件体系

Claude Code 使用 React + Ink 构建终端 UI，src/components/ 下有 144 个 UI 组件。

**部分关键组件：**
- `App.tsx` — 应用根组件
- `CoordinatorAgentStatus.tsx` — 多 Agent 状态展示
- `ContextVisualization.tsx` — 上下文可视化
- `ContextSuggestions.tsx` — 上下文建议
- `DiagnosticsDisplay.tsx` — 诊断信息展示
- `AutoUpdater.tsx` — 自动更新
- `ConsoleOAuthFlow.tsx` — OAuth 认证流程
- `DevBar.tsx` — 开发者工具栏

**配套 React Hooks（80+）：**
- `useVimInput.ts` — Vim 模式输入处理
- `useVoice.ts` / `useVoiceIntegration.tsx` — 语音输入集成
- `useSwarmInitialization.ts` — Swarm 模式初始化
- `useScheduledTasks.ts` — 定时任务调度
- `useMemoryUsage.ts` — 内存使用监控
- `useTerminalSize.ts` — 终端尺寸自适应
- `useVirtualScroll.ts` — 虚拟滚动（性能优化）

## 六、技术栈

| 类别 | 技术 | 选型理由 |
|------|------|----------|
| 运行时 | Bun | 极快的启动速度 + 原生 TypeScript 支持 |
| 语言 | TypeScript (strict) | 类型安全，512K 行代码必须强类型 |
| 终端 UI | React + Ink | 声明式 UI，组件复用，开发效率高 |
| CLI 解析 | Commander.js | 成熟的 CLI 参数解析框架 |
| Schema 校验 | Zod v4 | 运行时类型校验 + TypeScript 类型推断 |
| 代码搜索 | ripgrep | 极快的代码搜索引擎 |
| 协议 | MCP SDK + LSP | 标准化的工具/语言服务器协议 |
| API | Anthropic SDK | 官方 SDK |
| 遥测 | OpenTelemetry + gRPC | 标准化可观测性 |
| 特性标志 | GrowthBook | A/B 测试与灰度发布 |
| 认证 | OAuth 2.0 + JWT + macOS Keychain | 多层认证体系 |

### Bun 运行时优势

1. **启动速度：**比 Node.js 快数倍，CLI 工具对冷启动时间敏感
2. **原生 TypeScript：**无需编译步骤
3. **Bundle 特性标志：**`bun:bundle` 的 `feature()` 机制实现编译时死代码消除

```typescript
import { feature } from 'bun:bundle'

// 未启用的功能代码在构建时被完全移除
const voiceCommand = feature('VOICE_MODE')
  ? require('./commands/voice/index.js').default
  : null
```

已知特性标志：`PROACTIVE`、`KAIROS`、`BRIDGE_MODE`、`DAEMON`、`VOICE_MODE`、`AGENT_TRIGGERS`、`MONITOR_TOOL`。

### React 终端 UI 优势

1. **声明式渲染：**降低复杂状态管理（权限弹窗、多 Agent 状态、流式输出）的复杂度
2. **组件复用：**144 个组件之间存在大量复用关系
3. **Hooks 生态：**80+ 自定义 Hooks 实现关注点分离
4. **Bridge 复用：**同一套组件逻辑可以通过 Bridge 在 IDE 中复用

## 七、启动优化

```typescript
// main.tsx — 并行预取策略
startMdmRawRead()        // 预读 MDM 设置
startKeychainPrefetch()  // 预取 Keychain 凭据
```

- **并行预取：**MDM 设置、Keychain、API 预连接和 GrowthBook 初始化全部并行执行
- **懒加载：**重模块（OpenTelemetry ~400KB、gRPC ~700KB）通过动态 `import()` 延迟加载

## 八、隐藏特性

### 隐藏命令

- `good-claude` — 给 Claude 正向反馈
- `bughunter` — 自动化 Bug 猎手
- `chrome` — Chrome 浏览器集成
- `btw` — 内部调试命令
- `ant-trace` — Anthropic 内部追踪
- `mock-limits` — 模拟速率限制（测试用）
- `heapdump` — 堆内存快照

### 特殊模块

- **Buddy 系统：**src/buddy/ — 伴侣精灵功能
- **autoDream：**src/services/autoDream/ — 自动记忆处理服务

## 九、核心设计理念

### 1. 工具即能力（Tools as Capabilities）

LLM 的所有交互能力通过标准化 Tool 接口暴露，每个工具都是自描述的（Schema + 权限 + 执行逻辑）。

### 2. Agent 是一等公民

Agent 从架构层面就是核心概念，子 Agent 生成、Agent 间通信、团队协作都有专门工具和服务支撑。

### 3. 声明式优于命令式

从 React/Ink 的 UI 渲染到 Zod Schema 的类型校验，再到工具定义的声明式接口，整个项目倾向于声明式编程范式。

### 4. 性能是设计约束

Bun 运行时、并行预取、懒加载、编译时死代码消除——这些优化是技术选型阶段的决策，而非事后补丁。
