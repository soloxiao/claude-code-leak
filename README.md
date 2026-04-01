# Claude Code Source Repository Summary

## Overview

This repository contains the complete client-side source code of **Claude Code** — Anthropic's official AI-powered coding assistant CLI tool. It provides a terminal-based interactive interface for developers to collaborate with Claude AI on reading, writing, editing, and analyzing code through conversation.

### Codebase Scale

| Metric | Value |
|--------|-------|
| Total source files | 1,904 |
| TypeScript/TSX files | 1,884 |
| Total lines of code | ~512,664 |
| Source directory size | 33 MB |
| Top-level module directories | 36 |

---

## Tech Stack

| Category | Technology |
|----------|------------|
| Runtime | Bun |
| UI Framework | React + Ink (terminal renderer) |
| CLI Framework | Commander.js |
| API Client | @anthropic-ai/sdk |
| Schema Validation | Zod v4 |
| MCP Integration | @modelcontextprotocol/sdk |
| Feature Flags | GrowthBook + Bun compile-time `feature()` |
| State Management | Custom React Context Store (`useSyncExternalStore`) |

---

## Ten Core Modules

### 1. Interactive REPL

Entry chain: `cli.tsx` → `main.tsx` (4,683 lines) → `replLauncher.tsx`. Handles CLI argument parsing, session initialization, authentication, and REPL startup. ~146 UI component files.

### 2. Conversation & Query Engine

`query.ts` (1,729 lines) + `QueryEngine.ts` (1,295 lines). Manages message construction, API calls, tool execution, automatic context compression, and token/cost tracking.

### 3. Tool System (43 Tools)

Located under `src/tools/`:

| Category | Tools |
|----------|-------|
| Shell Execution | `BashTool` (with AST security analysis & sandbox), `PowerShellTool` |
| File Operations | `FileReadTool`, `FileWriteTool`, `FileEditTool` |
| Search | `GlobTool`, `GrepTool` |
| LSP Integration | `LSPTool` (go-to-definition, find references, hover info, etc.) |
| Web Interaction | `WebFetchTool`, `WebSearchTool` |
| Agent System | `AgentTool` (subagent spawning: local/remote/fork/worktree) |
| Task Management | `TaskCreateTool`, `TaskGetTool`, `TaskListTool`, `TaskUpdateTool`, `TaskStopTool`, `TaskOutputTool` |
| Team Collaboration | `TeamCreateTool`, `TeamDeleteTool`, `SendMessageTool` |
| Plan Mode | `EnterPlanModeTool`, `ExitPlanModeTool` |
| Worktree | `EnterWorktreeTool`, `ExitWorktreeTool` |
| MCP | `MCPTool`, `ListMcpResourcesTool`, `ReadMcpResourceTool` |
| Scheduled Tasks | `ScheduleCronTool` (CronCreate/Delete/List) |
| Remote Triggers | `RemoteTriggerTool` |
| Notebook | `NotebookEditTool` |
| User Interaction | `AskUserQuestionTool` |
| Other | `SkillTool`, `ConfigTool`, `SleepTool`, `BriefTool` |

### 4. Slash Command System (100+ Commands)

Located in `src/commands/`:

| Category | Example Commands |
|----------|-----------------|
| Session Management | `/resume`, `/session`, `/compact`, `/clear` |
| Git Integration | `/commit`, `/commit-push-pr`, `/branch`, `/diff` |
| Configuration | `/config`, `/permissions`, `/theme`, `/color`, `/effort` |
| MCP Management | `/mcp` |
| Plugins | `/plugin`, `/reload-plugins` |
| Authentication | `/login`, `/logout`, `/install-github-app`, `/install-slack-app` |
| Code Review | `/review`, `/security-review` |
| Remote Collaboration | `/bridge`, `/teleport`, `/remote-setup` |
| Advanced Features | `/ultraplan`, `/thinkback`, `/plan`, `/tasks`, `/vim`, `/voice` |
| Utilities | `/doctor`, `/help`, `/status`, `/stats`, `/usage`, `/cost`, `/export`, `/copy` |

### 5. Multi-Agent System

"Swarm" architecture supporting subagents working in parallel within isolated Git worktrees, with team creation/deletion/messaging. The `src/coordinator/` module handles multi-agent orchestration.

### 6. Remote Sessions (Claude Code Remote)

`src/bridge/` (33 files). WebSocket + JWT authentication based, supporting session management and persistence in remote containers.

### 7. Permission System

Fine-grained tool permission control, Bash command AST-level security analysis (`parseForSecurity`), policy limits and rate control, permission denial tracking.

### 8. Context Management

Automatic aggregation of Git context, CLAUDE.md project configuration, and persistent Memory system. `src/services/compact/` provides automatic conversation compression.

### 9. Plugin & Skill System

Extensible architecture: hot-reloading plugins (`src/plugins/`), built-in + MCP-extended skills (`src/skills/`), MCP server connection management (`src/services/mcp/`).

### 10. Voice Input

Speech-to-text integration (`src/voice/`, `src/services/voice.ts`), controlled via `feature('VOICE_MODE')` feature flag.

---

## Directory Structure

```
src/
├── entrypoints/       # Application entry points
├── main.tsx           # Main entry (4,683 lines)
├── query.ts           # Core conversation loop
├── QueryEngine.ts     # Query orchestration engine
├── Tool.ts            # Tool type system
├── tools/             # 43 tool implementations
├── commands/          # 100+ slash commands
├── components/        # 146 UI components
├── hooks/             # 85+ React hooks
├── services/          # Backend services (API, MCP, OAuth, analytics, etc.)
├── utils/             # 330+ utility functions
├── state/             # State management
├── bridge/            # Remote bridging
├── types/             # TypeScript type definitions
├── coordinator/       # Multi-agent coordinator
├── skills/            # Skill system
├── plugins/           # Plugin system
├── screens/           # Top-level page components
├── server/            # Remote session server
├── remote/            # Direct connection features
├── context/           # React Context
├── constants/         # Constants
├── migrations/        # Data migrations
├── keybindings/       # Keybinding system
├── vim/               # Vim mode
├── voice/             # Voice input
├── ink/               # Ink extension utilities
├── memdir/            # Memory directory management
└── buddy/             # Buddy sprite (animated character)
```

---

## Feature Flags

| Flag | Description |
|------|-------------|
| `COORDINATOR_MODE` | Multi-agent coordinator mode |
| `VOICE_MODE` | Voice input |
| `BRIDGE_MODE` | Remote bridge mode |
| `PROACTIVE` | Proactive suggestions |
| `AGENT_TRIGGERS` | Agent triggers |
| `MONITOR_TOOL` | Monitor tool |
| `KAIROS` | Kairos feature |

---

## Security Mechanisms

- Bash command AST parsing & security analysis
- Sandbox execution adapters
- Read-only validation
- Fine-grained permission control & denial tracking
- Anthropic internal features isolated via `process.env.USER_TYPE === 'ant'`

---

## Service Layer Details (`src/services/`)

| Module | Description |
|--------|-------------|
| `api/` | Claude API client wrapper |
| `oauth/` | OAuth authentication flow |
| `mcp/` | MCP server connection & management |
| `lsp/` | Language Server Protocol integration |
| `compact/` | Automatic conversation compression |
| `analytics/` | Usage analytics & tracking |
| `plugins/` | Plugin services |
| `SessionMemory/` | Session memory persistence |
| `teamMemorySync/` | Team memory synchronization |
| `extractMemories/` | Automatic memory extraction |
| `MagicDocs/` | Intelligent documentation service |
| `PromptSuggestion/` | Prompt suggestions |
| `AgentSummary/` | Agent summaries |
| `tools/` | Tool service layer |
| `tokenEstimation.ts` | Token estimation |
| `policyLimits/` | Policy & limit management |
| `remoteManagedSettings/` | Remote configuration management |
| `settingsSync/` | Settings synchronization |
| `tips/` | Tips & suggestions |
| `voice.ts` | Voice service |
| `vcr.ts` | Session recording/playback |

---

## Additional Modules

| Module | Path | Description |
|--------|------|-------------|
| Buddy Sprite | `src/buddy/` | Animated terminal companion character |
| Vim Mode | `src/vim/` | Full Vim keybindings (motions/operators/text objects) |
| Output Styles | `src/outputStyles/` | Switchable output display modes |
| Upstream Proxy | `src/upstreamproxy/` | HTTP proxy support |
| Native TS | `src/native-ts/` | Native TypeScript utilities |
| Moreright | `src/moreright/` | Extended permission module |
| Keybindings | `src/keybindings/` | Customizable keybinding system |
| Migrations | `src/migrations/` | Configuration/data version migrations |

---

## Platform Support

Claude Code runs in the following forms:
- **CLI terminal tool** (primary form)
- **Desktop app** (Mac/Windows)
- **Web app** (claude.ai/code)
- **IDE extensions** (VS Code, JetBrains)
- **Chrome extension integration** (`src/commands/chrome/`)
- **Mobile** (`src/commands/mobile/`)
