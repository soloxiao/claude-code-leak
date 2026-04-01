# Claude Code 源码仓库功能总结

## 项目概述

本仓库包含 **Claude Code** 的完整客户端源代码 — Anthropic 官方的 AI 编程助手 CLI 工具。提供基于终端的交互式界面，让开发者通过对话与 Claude AI 协作完成代码阅读、编写、编辑和分析。

### 代码规模

| 指标 | 数值 |
|------|------|
| 源文件总数 | 1,904 |
| TypeScript/TSX 文件 | 1,884 |
| 总代码行数 | ~512,664 |
| 源码目录大小 | 33 MB |
| 顶层模块目录 | 36 个 |

---

## 技术栈

| 类别 | 技术 |
|------|------|
| 运行时 | Bun |
| UI 框架 | React + Ink（终端渲染器） |
| CLI 框架 | Commander.js |
| API 客户端 | @anthropic-ai/sdk |
| Schema 校验 | Zod v4 |
| MCP 集成 | @modelcontextprotocol/sdk |
| 特性开关 | GrowthBook + Bun 编译时 `feature()` |
| 状态管理 | 自定义 React Context Store（`useSyncExternalStore`） |

---

## 十大核心模块

### 1. 交互式 REPL

入口链路：`cli.tsx` → `main.tsx`（4683 行）→ `replLauncher.tsx`。负责 CLI 参数解析、会话初始化、认证、REPL 启动。UI 组件约 146 个文件。

### 2. 对话与查询引擎

`query.ts`（1729 行）+ `QueryEngine.ts`（1295 行）。管理消息构建、API 调用、工具执行、上下文自动压缩、Token/成本追踪。

### 3. 工具系统（43 个工具）

位于 `src/tools/` 下，包括：

| 类别 | 工具 |
|------|------|
| Shell 执行 | `BashTool`（含 AST 安全分析与沙箱）、`PowerShellTool` |
| 文件操作 | `FileReadTool`、`FileWriteTool`、`FileEditTool` |
| 搜索 | `GlobTool`、`GrepTool` |
| LSP 集成 | `LSPTool`（定义跳转、引用查找、悬停信息等） |
| Web 交互 | `WebFetchTool`、`WebSearchTool` |
| Agent 系统 | `AgentTool`（子代理派生：本地/远程/Fork/Worktree） |
| 任务管理 | `TaskCreateTool`、`TaskGetTool`、`TaskListTool`、`TaskUpdateTool`、`TaskStopTool`、`TaskOutputTool` |
| 团队协作 | `TeamCreateTool`、`TeamDeleteTool`、`SendMessageTool` |
| 计划模式 | `EnterPlanModeTool`、`ExitPlanModeTool` |
| Worktree | `EnterWorktreeTool`、`ExitWorktreeTool` |
| MCP | `MCPTool`、`ListMcpResourcesTool`、`ReadMcpResourceTool` |
| 定时任务 | `ScheduleCronTool`（CronCreate/Delete/List） |
| 远程触发 | `RemoteTriggerTool` |
| Notebook | `NotebookEditTool` |
| 用户交互 | `AskUserQuestionTool` |
| 其他 | `SkillTool`、`ConfigTool`、`SleepTool`、`BriefTool` |

### 4. 斜杠命令系统（100+ 命令）

位于 `src/commands/`，涵盖：

| 类别 | 命令示例 |
|------|----------|
| 会话管理 | `/resume`、`/session`、`/compact`、`/clear` |
| Git 集成 | `/commit`、`/commit-push-pr`、`/branch`、`/diff` |
| 配置 | `/config`、`/permissions`、`/theme`、`/color`、`/effort` |
| MCP 管理 | `/mcp` |
| 插件 | `/plugin`、`/reload-plugins` |
| 认证 | `/login`、`/logout`、`/install-github-app`、`/install-slack-app` |
| 代码审查 | `/review`、`/security-review` |
| 远程协作 | `/bridge`、`/teleport`、`/remote-setup` |
| 高级功能 | `/ultraplan`、`/thinkback`、`/plan`、`/tasks`、`/vim`、`/voice` |
| 实用工具 | `/doctor`、`/help`、`/status`、`/stats`、`/usage`、`/cost`、`/export`、`/copy` |

### 5. 多代理系统

"Swarm" 架构，支持子代理在独立 Git Worktree 中并行工作，提供团队创建/删除/消息传递。`src/coordinator/` 模块负责多代理编排。

### 6. 远程会话（Claude Code Remote）

`src/bridge/`（33 文件），基于 WebSocket + JWT 认证，支持远程容器中的会话管理与持久化。

### 7. 权限系统

精细化工具权限控制，Bash 命令 AST 级安全分析（`parseForSecurity`），策略限制与速率控制，权限拒绝追踪。

### 8. 上下文管理

自动聚合 Git 上下文、CLAUDE.md 项目配置、Memory 持久化记忆。`src/services/compact/` 提供对话自动压缩。

### 9. 插件与技能系统

可扩展架构：插件热加载（`src/plugins/`）、内置 + MCP 扩展技能（`src/skills/`）、MCP 服务器连接管理（`src/services/mcp/`）。

### 10. 语音输入

语音转文字集成（`src/voice/`、`src/services/voice.ts`），通过 `feature('VOICE_MODE')` 特性开关控制。

---

## 目录结构

```
src/
├── entrypoints/       # 应用入口
├── main.tsx           # 主入口（4683 行）
├── query.ts           # 对话核心循环
├── QueryEngine.ts     # 查询编排引擎
├── Tool.ts            # 工具类型系统
├── tools/             # 43 个工具实现
├── commands/          # 100+ 斜杠命令
├── components/        # 146 个 UI 组件
├── hooks/             # 85 个 React Hooks
├── services/          # 后端服务（API、MCP、OAuth、分析等）
├── utils/             # 330+ 工具函数
├── state/             # 状态管理
├── bridge/            # 远程桥接
├── types/             # TypeScript 类型定义
├── coordinator/       # 多代理协调器
├── skills/            # 技能系统
├── plugins/           # 插件系统
├── screens/           # 顶级页面组件
├── server/            # 远程会话服务
├── remote/            # 直连功能
├── context/           # React Context
├── constants/         # 常量定义
├── migrations/        # 数据迁移
├── keybindings/       # 快捷键系统
├── vim/               # Vim 模式
├── voice/             # 语音输入
├── ink/               # Ink 扩展工具
├── memdir/            # 记忆目录管理
└── buddy/             # 伴侣精灵（动画角色）
```

---

## 特性开关

| 开关 | 功能 |
|------|------|
| `COORDINATOR_MODE` | 多代理协调器模式 |
| `VOICE_MODE` | 语音输入 |
| `BRIDGE_MODE` | 远程桥接模式 |
| `PROACTIVE` | 主动建议 |
| `AGENT_TRIGGERS` | 代理触发器 |
| `MONITOR_TOOL` | 监控工具 |
| `KAIROS` | Kairos 功能 |

---

## 安全机制

- Bash 命令 AST 解析与安全分析
- 沙箱执行适配器
- 只读校验
- 精细化权限控制与拒绝追踪
- Anthropic 内部功能通过 `process.env.USER_TYPE === 'ant'` 隔离

---

## 服务层详情（`src/services/`）

| 模块 | 功能 |
|------|------|
| `api/` | Claude API 客户端封装 |
| `oauth/` | OAuth 认证流程 |
| `mcp/` | MCP 服务器连接与管理 |
| `lsp/` | 语言服务器协议集成 |
| `compact/` | 对话自动压缩 |
| `analytics/` | 使用分析与追踪 |
| `plugins/` | 插件服务 |
| `SessionMemory/` | 会话记忆持久化 |
| `teamMemorySync/` | 团队记忆同步 |
| `extractMemories/` | 自动记忆提取 |
| `MagicDocs/` | 智能文档服务 |
| `PromptSuggestion/` | 提示建议 |
| `AgentSummary/` | 代理摘要 |
| `tools/` | 工具服务层 |
| `tokenEstimation.ts` | Token 估算 |
| `policyLimits/` | 策略与限制管理 |
| `remoteManagedSettings/` | 远程配置管理 |
| `settingsSync/` | 设置同步 |
| `tips/` | 提示与建议 |
| `voice.ts` | 语音服务 |
| `vcr.ts` | 会话录制/回放 |

---

## 其他特色模块

| 模块 | 路径 | 功能 |
|------|------|------|
| Buddy 精灵 | `src/buddy/` | 终端动画伴侣角色 |
| Vim 模式 | `src/vim/` | 完整 Vim 键绑定（motions/operators/text objects） |
| 输出样式 | `src/outputStyles/` | 可切换的输出显示模式 |
| 上游代理 | `src/upstreamproxy/` | HTTP 代理支持 |
| Native TS | `src/native-ts/` | 原生 TypeScript 工具 |
| Moreright | `src/moreright/` | 扩展权限模块 |
| 快捷键 | `src/keybindings/` | 可自定义快捷键系统 |
| 数据迁移 | `src/migrations/` | 配置/数据版本迁移 |

---

## 平台支持

Claude Code 可作为以下形态运行：
- **CLI 终端工具**（主要形态）
- **桌面应用**（Mac/Windows）
- **Web 应用**（claude.ai/code）
- **IDE 扩展**（VS Code、JetBrains）
- **Chrome 扩展集成**（`src/commands/chrome/`）
- **移动端**（`src/commands/mobile/`）
