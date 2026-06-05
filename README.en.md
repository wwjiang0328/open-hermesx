# HermesX

<div align="center">

**Agent Command Center · Native Desktop AI Workbench**

[![Version](https://img.shields.io/badge/version-0.1.0-blue)](package.json)
[![Rust](https://img.shields.io/badge/rust-1.96-orange)](src-tauri/Cargo.toml)
[![TypeScript](https://img.shields.io/badge/typescript-6.x-3178c6)](package.json)
[![Tauri](https://img.shields.io/badge/tauri-2.0-ffc131)](src-tauri/Cargo.toml)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

[中文](./README.md)

</div>

---

## Current Technical State

HermesX has moved from “feature collection” toward a cleaner desktop workbench architecture. The recent updates focus on information structure, workflow execution isolation, settings density, and desktop pet presentation.

### Settings Center

Settings is now a dedicated two-column surface:

```text
Settings menu (left) -> active settings panel (right)
```

- **Core**: Agent Instances, AI Models, MCP Tools
- **Personalization**: Desktop Pet, Workflow
- **System**: Memory, Built-in Assistants, Logs, Backup/Restore, About
- Menu rows now show only icon + label, without explanatory subtext.
- All return actions use “Back to Workbench”.
- Opening Settings no longer shows the collaboration/sidebar navigation.

### Workflow Panel

Workflow editing is embedded inside Settings instead of taking over the whole app.

```text
Template cards
      ↓
DAG execution canvas
      ↓
Right-side run drawer
```

- Template cards are shown above the canvas.
- Clicking a node opens a configuration panel.
- Nodes can configure model, Agent, input, prompt, output variable, output path, and template-specific parameters.
- Running a workflow opens a right-side progress drawer with `pending / running / completed / failed / skipped` states.
- Workflow tasks execute silently with `dispatcher: workflow`; results are not written into the conversation workbench.
- Node dragging is clamped to the visible canvas bounds.

### Desktop Pet Panel

The pet settings page now behaves like a real responsive panel:

- Sunny, Kage, and yomi are shown as three horizontal cards.
- The panel uses the full available Settings width instead of a fixed max width.
- The lower area previews common animation states: idle, stretch, jump, working, success, and celebration.
- Previews reuse the real 8x9 WebP spritesheets used by the desktop pet window.

### Workbench Cleanup

The workspace was simplified into operation-first headers:

- Left header: `Collaboration Instances`
- Right header: `Conversation Workbench`
- Explanatory copy was removed from the main chrome.
- Header separator lines are aligned across left and right panels.

---

## One Sentence

**HermesX is not another AI chat box. It is a desktop conductor for local context, remote Hermes agents, AI models, workflow execution, permissions, memory, and a small animated companion.**

---

## Technical Adaptations

HermesX is built by adapting proven Agent patterns to a Tauri-native desktop runtime.

### 1. Tauri 2 Native Runtime

- Rust backend for file proxying, HTTP/SSE proxying, file watching, PTY sessions, desktop pet window control, and local commands.
- React + TypeScript frontend with Zustand stores and Vite.
- Tauri IPC replaces the previous Electron shell path.
- WebView and desktop integrations stay local-first.

### 2. Conductor Mode

HermesX does not try to become one more Agent. It coordinates Agents.

```text
HermesX
  ├─ Hermes backend instances
  ├─ Agent profiles
  ├─ local tool relay
  ├─ permissions and path approval
  ├─ workflow execution
  └─ memory and settings
```

Key properties:

- `@agent` dispatch and `@all` broadcast from the workbench.
- Multiple backend instances are aggregated into one collaboration surface.
- Remote Agents reason; HermesX executes local tools and applies permission boundaries.
- Conversation workbench and workflow runtime are now separate execution surfaces.

### 3. Local Tool Relay

Remote Agents can request tools, but the desktop app remains the execution boundary.

| Category | Examples | Boundary |
|---|---|---|
| Files | `read_file`, `write_file`, `patch_file`, `read_path` | approved local paths |
| Search | `search_files`, `find_files`, `list_dir` | local filesystem |
| Commands | `run_command` | confirmation required |
| Git | `git_status`, `git_diff` | local repo |
| Desktop | `a11y_snapshot`, `a11y_list_windows` | OS UI tree |
| Skills | `hermesx_load_skill` | bundled skill content |

Dangerous actions are routed through explicit confirmation instead of being executed automatically.

### 4. Workflow Runtime

The current workflow runtime is a visual DAG editor plus a silent execution layer.

- `WorkflowBuilder.tsx` owns template cards, node cards, node configuration, canvas drag logic, and the run drawer.
- `useWorkflowStore.ts` stores the active template, canvas transform, selected node, and execution object.
- `workflowEngine.ts` walks the graph from the trigger node, updates node states, and stores node outputs in execution context.
- `useAgentExecutionStore.ts` supports silent workflow tasks so workflow output stays inside the workflow drawer.
- Completed Agent task results are consumed by workflow execution through an in-memory result cache rather than by reading conversation messages.

This separation prevents workflow runs from polluting normal conversations.

### 5. Settings Information Architecture

Settings is intentionally compact and non-explanatory:

| Group | Entries |
|---|---|
| Core | Agent Instances, AI Models, MCP Tools |
| Personalization | Desktop Pet, Workflow |
| System | Memory, Built-in Assistants, Logs, Backup/Restore, About |

The UI goal is that the menu behaves like navigation, not documentation.

### 6. Desktop Pet

The desktop pet is a task-state feedback surface, not only decoration.

- Three characters: Sunny, Kage, and yomi.
- Each animated character uses an 8-column by 9-row WebP spritesheet.
- The separate Tauri pet window can be transparent and always-on-top.
- The settings page previews real animation frames so users can inspect the available states.

### 7. Memory And Context

HermesX keeps memory local-first:

- ADD-only memory concepts inspired by Mem0-style write semantics.
- Typed memory categories such as pattern, preference, architecture, bug, workflow, and fact.
- Agent briefings can inject relevant memory before dispatch.
- Conversation context, Agent logs, and project profiles are separated rather than collapsed into one chat transcript.

---

## Frontend / Backend Structure

```text
src-tauri/
  src/
    commands/           Tauri commands for files, HTTP, pet, PTY, watch, data
    accessibility/      OS accessibility tree adapters
    proxy.rs            local file proxy
    pet.rs              desktop pet manager
    pty_manager.rs      PTY session manager

src/
  components/
    Workspace.tsx       conversation workbench
    Sidebar.tsx         collaboration instances
    Settings.tsx        settings center
    WorkflowBuilder.tsx visual workflow canvas, node config, run drawer
    SimplePetTab.tsx    desktop pet cards and frame previews
    DesktopPet.tsx      embedded/windowed pet renderer
  services/
    workflowEngine.ts   DAG execution engine
    toolExecutor.ts     local tool execution
    toolRegistry.ts     tool registry
    providers.ts        model provider routing
    contextCompressor.ts context compression
    projectMemory.ts    project learning
  store/
    useWorkflowStore.ts workflow state
    useAgentExecutionStore.ts Agent task execution and silent result cache
    useHermesXStore.ts settings and config
```

---

## Development

Install dependencies:

```bash
npm install
```

Start the web frontend:

```bash
npm run dev:web
```

Start the Tauri desktop app:

```bash
npm run dev
```

Build the frontend:

```bash
npm run build:web
```

Build the desktop app:

```bash
npm run tauri:build
```

Run tests:

```bash
npm run test
```

---

## License

MIT. See [LICENSE](./LICENSE).
