# Claude Code — Leaked Source

**The full source code of Anthropic's Claude Code CLI, leaked on March 31, 2026**

[TypeScript](#tech-stack)  
[Bun](#tech-stack)  
[React + Ink](#tech-stack)  
[Files](#directory-structure)  

---

## How It Leaked

[Chaofan Shou (@Fried_rice)](https://x.com/Fried_rice) discovered that the published npm package for Claude Code included a `.map` file referencing the full, unobfuscated TypeScript source — downloadable as a zip from Anthropic's R2 storage bucket.

> **"Claude code source code has been leaked via a map file in their npm registry!"**
>
> — [@Fried_rice, March 31, 2026](https://x.com/Fried_rice/status/2038894956459290963)

---

### One-liner setup (from source)

```bash
  npm install && npm run build
```

---

## Directory Structure

```
src/
├── main.tsx                 # Entrypoint — Commander.js CLI parser + React/Ink renderer
├── QueryEngine.ts           # Core LLM API caller (~46K lines)
├── Tool.ts                  # Tool type definitions (~29K lines)
├── commands.ts              # Command registry (~25K lines)
├── tools.ts                 # Tool registry
├── context.ts               # System/user context collection
├── cost-tracker.ts          # Token cost tracking
│
├── tools/                   # Agent tool implementations (~40)
├── commands/                # Slash command implementations (~50)
├── components/              # Ink UI components (~140)
├── services/                # External service integrations
├── hooks/                   # React hooks (incl. permission checks)
├── types/                   # TypeScript type definitions
├── utils/                   # Utility functions
├── screens/                 # Full-screen UIs (Doctor, REPL, Resume)
│
├── bridge/                  # IDE integration (VS Code, JetBrains)
├── coordinator/             # Multi-agent orchestration
├── plugins/                 # Plugin system
├── skills/                  # Skill system
├── server/                  # Server mode
├── remote/                  # Remote sessions
├── memdir/                  # Persistent memory directory
├── tasks/                   # Task management
├── state/                   # State management
│
├── voice/                   # Voice input
├── vim/                     # Vim mode
├── keybindings/             # Keybinding configuration
├── schemas/                 # Config schemas (Zod)
├── migrations/              # Config migrations
├── entrypoints/             # Initialization logic
├── query/                   # Query pipeline
├── ink/                     # Ink renderer wrapper
├── buddy/                   # Companion sprite (Easter egg 🐣)
├── native-ts/               # Native TypeScript utils
├── outputStyles/            # Output styling
└── upstreamproxy/           # Proxy configuration
```

---

## Architecture

### 1. Tool System

> `src/tools/` — Every tool Claude can invoke is a self-contained module with its own input schema, permission model, and execution logic.


| Tool                                     | Description                               |
| ---------------------------------------- | ----------------------------------------- |
| **File I/O**                             |                                           |
| `FileReadTool`                           | Read files (images, PDFs, notebooks)      |
| `FileWriteTool`                          | Create / overwrite files                  |
| `FileEditTool`                           | Partial modification (string replacement) |
| `NotebookEditTool`                       | Jupyter notebook editing                  |
| **Search**                               |                                           |
| `GlobTool`                               | File pattern matching                     |
| `GrepTool`                               | ripgrep-based content search              |
| `WebSearchTool`                          | Web search                                |
| `WebFetchTool`                           | Fetch URL content                         |
| **Execution**                            |                                           |
| `BashTool`                               | Shell command execution                   |
| `SkillTool`                              | Skill execution                           |
| `MCPTool`                                | MCP server tool invocation                |
| `LSPTool`                                | Language Server Protocol integration      |
| **Agents & Teams**                       |                                           |
| `AgentTool`                              | Sub-agent spawning                        |
| `SendMessageTool`                        | Inter-agent messaging                     |
| `TeamCreateTool` / `TeamDeleteTool`      | Team management                           |
| `TaskCreateTool` / `TaskUpdateTool`      | Task management                           |
| **Mode & State**                         |                                           |
| `EnterPlanModeTool` / `ExitPlanModeTool` | Plan mode toggle                          |
| `EnterWorktreeTool` / `ExitWorktreeTool` | Git worktree isolation                    |
| `ToolSearchTool`                         | Deferred tool discovery                   |
| `SleepTool`                              | Proactive mode wait                       |
| `CronCreateTool`                         | Scheduled triggers                        |
| `RemoteTriggerTool`                      | Remote trigger                            |
| `SyntheticOutputTool`                    | Structured output generation              |


### 2. Command System

> `src/commands/` — User-facing slash commands invoked with `/` in the REPL.


| Command              | Description             |     | Command   | Description       |
| -------------------- | ----------------------- | --- | --------- | ----------------- |
| `/commit`            | Git commit              |     | `/memory` | Persistent memory |
| `/review`            | Code review             |     | `/skills` | Skill management  |
| `/compact`           | Context compression     |     | `/tasks`  | Task management   |
| `/mcp`               | MCP server management   |     | `/vim`    | Vim mode toggle   |
| `/config`            | Settings                |     | `/diff`   | View changes      |
| `/doctor`            | Environment diagnostics |     | `/cost`   | Check usage cost  |
| `/login` / `/logout` | Auth                    |     | `/theme`  | Change theme      |
| `/context`           | Context visualization   |     | `/share`  | Share session     |
| `/pr_comments`       | PR comments             |     | `/resume` | Restore session   |
| `/desktop`           | Desktop handoff         |     | `/mobile` | Mobile handoff    |


### 3. Service Layer

> `src/services/` — External integrations and core infrastructure.


| Service                  | Description                                    |
| ------------------------ | ---------------------------------------------- |
| `api/`                   | Anthropic API client, file API, bootstrap      |
| `mcp/`                   | Model Context Protocol connection & management |
| `oauth/`                 | OAuth 2.0 authentication                       |
| `lsp/`                   | Language Server Protocol manager               |
| `analytics/`             | GrowthBook feature flags & analytics           |
| `plugins/`               | Plugin loader                                  |
| `compact/`               | Conversation context compression               |
| `extractMemories/`       | Automatic memory extraction                    |
| `teamMemorySync/`        | Team memory synchronization                    |
| `tokenEstimation.ts`     | Token count estimation                         |
| `policyLimits/`          | Organization policy limits                     |
| `remoteManagedSettings/` | Remote managed settings                        |


### 4. Bridge System

> `src/bridge/` — Bidirectional communication layer connecting IDE extensions (VS Code, JetBrains) with the CLI.

Key files: `bridgeMain.ts` (main loop) · `bridgeMessaging.ts` (protocol) · `bridgePermissionCallbacks.ts` (permission callbacks) · `replBridge.ts` (REPL session) · `jwtUtils.ts` (JWT auth) · `sessionRunner.ts` (session execution)

### 5. Permission System

> `src/hooks/toolPermission/` — Checks permissions on every tool invocation.

Prompts the user for approval/denial or auto-resolves based on the configured permission mode: `default`, `plan`, `bypassPermissions`, `auto`, etc.

### 6. Feature Flags

Dead code elimination at build time via Bun's `bun:bundle`:

```typescript
import { feature } from 'bun:bundle'

const voiceCommand = feature('VOICE_MODE')
  ? require('./commands/voice/index.js').default
  : null
```

Notable flags: `PROACTIVE` · `KAIROS` · `BRIDGE_MODE` · `DAEMON` · `VOICE_MODE` · `AGENT_TRIGGERS` · `MONITOR_TOOL`

---

## Key Files


| File             | Lines | Purpose                                                                                |
| ---------------- | ----- | -------------------------------------------------------------------------------------- |
| `QueryEngine.ts` | ~46K  | Core LLM API engine — streaming, tool loops, thinking mode, retries, token counting    |
| `Tool.ts`        | ~29K  | Base types/interfaces for all tools — input schemas, permissions, progress state       |
| `commands.ts`    | ~25K  | Command registration & execution with conditional per-environment imports              |
| `main.tsx`       | —     | CLI parser + React/Ink renderer; parallelizes MDM, keychain, and GrowthBook on startup |


---

## Tech Stack


| Category          | Technology                                                              |
| ----------------- | ----------------------------------------------------------------------- |
| Runtime           | [Bun](https://bun.sh)                                                   |
| Language          | TypeScript (strict)                                                     |
| Terminal UI       | [React](https://react.dev) + [Ink](https://github.com/vadimdemedes/ink) |
| CLI Parsing       | [Commander.js](https://github.com/tj/commander.js) (extra-typings)      |
| Schema Validation | [Zod v4](https://zod.dev)                                               |
| Code Search       | [ripgrep](https://github.com/BurntSushi/ripgrep) (via GrepTool)         |
| Protocols         | [MCP SDK](https://modelcontextprotocol.io) · LSP                        |
| API               | [Anthropic SDK](https://docs.anthropic.com)                             |
| Telemetry         | OpenTelemetry + gRPC                                                    |
| Feature Flags     | GrowthBook                                                              |
| Auth              | OAuth 2.0 · JWT · macOS Keychain                                        |


---

## Design Patterns

**Parallel Prefetch** — Startup optimization

MDM settings, keychain reads, and API preconnect fire in parallel as side-effects before heavy module evaluation:

```typescript
// main.tsx
startMdmRawRead()
startKeychainPrefetch()
```

**Lazy Loading** — Deferred heavy modules

OpenTelemetry (~~400KB) and gRPC (~~700KB) are loaded via dynamic `import()` only when needed.

**Agent Swarms** — Multi-agent orchestration

Sub-agents spawn via `AgentTool`, with `coordinator/` handling orchestration. `TeamCreateTool` enables team-level parallel work.

**Skill System** — Reusable workflows

Defined in `skills/` and executed through `SkillTool`. Users can add custom skills.

**Plugin Architecture** — Extensibility

Built-in and third-party plugins loaded through the `plugins/` subsystem.

---

## Disclaimer

This repository archives source code leaked from Anthropic's npm registry on **2026-03-31**. All original source code is the property of [Anthropic](https://www.anthropic.com). This is not an official release and is not licensed for redistribution.

---

