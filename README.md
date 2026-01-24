# Claude Task Viewer

A real-time Kanban board for monitoring Claude Code tasks. See what Claude is working on, track dependencies between tasks, and add notes that Claude can read.

![Dark mode](screenshot-dark-v2.png)

![Light mode](screenshot-light-v2.png)

## Why Use This?

When Claude Code breaks down complex work into tasks, you get visibility into its thinking — but only in the terminal. Claude Task Viewer gives you a persistent, visual dashboard to:

- **See the big picture** — All your sessions and tasks in one place
- **Know what's happening now** — Live Updates show exactly what Claude is doing across all sessions
- **Understand task dependencies** — See which tasks are blocked and what's holding them up
- **Collaborate with Claude** — Add notes to tasks that Claude reads when it picks them up

## Key Features

### Interactive Task Management
Full CRUD operations on tasks:
- **Drag & drop** — Drag task cards between columns to change status instantly
- **Edit tasks** — Click the edit button or press `E` to modify task subject and description inline
- **Change status** — Use the status dropdown or drag & drop to move tasks between Pending, In Progress, and Completed
- **Create tasks** — Click the `+` button in any column or press `N` to create new tasks
- **Delete tasks** — Remove tasks with the delete button or press `D` (includes safety checks for dependencies)

### Task Reordering & Prioritization
- **Drag within columns** — Reorder tasks to set priority (position numbers shown on cards)
- **Dependency validation** — Warns when reordering conflicts with task dependencies
- **Visual feedback** — Position badges and smooth drag animations

### Session Management
Organize and customize your Claude Code sessions:
- **Rename sessions** — Give sessions meaningful names instead of UUIDs
- **Add descriptions** — Document what each session is working on
- **View project paths** — See the full filesystem path for each project
- **Git branch badges** — Display current branch for each session
- **Status indicators** — Visual activity status (green=active, yellow=inactive)
- **Configurable limits** — Show 10, 20, 50, or all sessions
- **Fuzzy search** — Search across session names, task descriptions, and project paths with instant filtering

### Live Updates
Real-time feed of all in-progress tasks across every session. See what Claude is actively working on without switching terminals. Each update shows the current action and which session it belongs to.

### Task Dependencies
Tasks can block other tasks. The viewer shows these relationships clearly — blocked tasks display what they're waiting on, and you can trace dependency chains to understand the critical path. The system prevents moving blocked tasks to in_progress and deleting tasks that block others.

### Notes
Add context to any task. Your notes are appended to the task description, so Claude sees them when it reads the task. Use this to clarify requirements, add constraints, or redirect work — all without interrupting Claude's flow.

### Keyboard Shortcuts
- `E` — Edit the currently selected task
- `D` — Delete the currently selected task (with confirmation)
- `N` — Create a new task
- `?` — Show keyboard shortcuts help
- `Esc` — Close detail panel or cancel edit mode

### Bulk Operations
- **Delete all tasks** — Delete all tasks in a session with dependency-aware sorting
- **Progress tracking** — Detailed feedback on deletion progress

### Project Filtering
Filter tasks by project using the dropdown. Working on multiple codebases? See only what's relevant. Combine with the session filter to show just active sessions for a specific project.

## Installation

### Quick start

```bash
npx claude-task-viewer
```

Open http://localhost:3456

### From source

```bash
git clone https://github.com/L1AD/claude-task-viewer.git
cd claude-task-viewer
npm install
npm start
```

## How It Works

Claude Code stores tasks in `~/.claude/tasks/`. Each session has its own folder:

```
~/.claude/tasks/
  └── {session-uuid}/
      ├── 1.json
      ├── 2.json
      └── ...
```

The viewer watches this directory and pushes updates via Server-Sent Events. Changes appear instantly — no polling, no refresh needed.

## Task Structure

```json
{
  "id": "1",
  "subject": "Implement user authentication",
  "description": "Add JWT-based auth with refresh tokens",
  "activeForm": "Setting up auth middleware",
  "status": "in_progress",
  "blocks": ["2", "3"],
  "blockedBy": []
}
```

- `activeForm` — What Claude is doing right now (shown in Live Updates)
- `blocks` / `blockedBy` — Task dependency relationships

## Configuration

```bash
# Custom port
PORT=8080 npx claude-task-viewer

# Open browser automatically
npx claude-task-viewer --open
```

## API

| Endpoint | Description |
|----------|-------------|
| `GET /api/sessions` | List all sessions with task counts (supports `?limit=N` parameter) |
| `GET /api/sessions/:id` | Get all tasks for a session |
| `PATCH /api/sessions/:id/metadata` | Update session metadata (custom name and description) |
| `GET /api/tasks/all` | Get all tasks across all sessions |
| `POST /api/tasks/:session` | Create a new task in a session |
| `PATCH /api/tasks/:session/:task` | Update a task (status, subject, description, order, dependencies) |
| `DELETE /api/tasks/:session/:task` | Delete a task (checks dependencies) |
| `POST /api/tasks/:session/:task/note` | Add a note to a task |
| `GET /api/events` | SSE stream for live updates |

## Roadmap

- **Shared task lists** — View tasks shared across multiple Claude Code sessions and subagents
- **Task creation** — Create tasks from the viewer that Claude picks up
- **CLI integration** — `claude-task-viewer add "Fix the bug"`
- **Desktop notifications** — Know when tasks complete
- **Export** — Push tasks to Linear, GitHub Issues, or Jira

[Open an issue](https://github.com/L1AD/claude-task-viewer/issues) with ideas or feedback.

## License

MIT
