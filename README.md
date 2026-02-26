# TaskFlow

A full-stack task management app built with [Tova](https://github.com/paiml/tova-lang) — a modern language that transpiles to JavaScript. TaskFlow demonstrates how to build a production-style CRUD application using Tova's `server {}` and `browser {}` blocks, SQLite, and TailwindCSS.

## Prerequisites

- [Bun](https://bun.sh/) v1.0+ (runtime)
- [Tova](https://github.com/paiml/tova-lang) compiler

Install Bun:

```bash
curl -fsSL https://bun.sh/install | bash
```

Clone and set up the Tova compiler:

```bash
git clone https://github.com/paiml/tova-lang.git
cd tova-lang
```

## Quick Start

```bash
cd taskflow

# Start the dev server (auto-compiles + live reload)
bun run /path/to/tova-lang/bin/tova.js dev src

# Open http://localhost:3000
```

The dev server compiles both `.tova` files, starts the HTTP server on port 3000, serves the browser client, and watches for file changes with live reload.

## Project Structure

```
taskflow/
├── tova.toml           # Project config (name, entry dir, port)
├── package.json        # NPM metadata
├── .gitignore          # Ignores node_modules, .tova-out, *.db
├── src/
│   ├── server.tova     # Server block: database, models, functions, routes
│   └── app.tova        # Browser block: state, components, UI
└── taskflow.db         # SQLite database (auto-created on first run)
```

## What You'll Learn

TaskFlow is designed as a learning resource for Tova. Each concept below links to where it appears in the source code.

### 1. Server Block Basics (`src/server.tova`)

The `server {}` block defines your backend — database, models, functions, and routes — all in one file.

**Database config:**

```tova
server {
  db {
    driver: "sqlite"
    path: "taskflow.db"
  }
}
```

**Models** auto-generate CRUD methods (`.all()`, `.find()`, `.create()`, `.update()`, `.delete()`, `.where()`):

```tova
model Project {
  name: String
  color: String
  icon: String
  created_at: String
}
```

**Server functions** are automatically exposed as RPC endpoints to the browser:

```tova
fn list_projects() -> [Project] {
  Project.all()
}

fn create_project(name: String, color: String, icon: String) -> Project {
  Project.create({
    name: name,
    color: color,
    icon: icon,
    created_at: Date.new().toISOString()
  })
}
```

**Routes** map HTTP methods and paths to functions:

```tova
route GET    "/api/projects"       => list_projects
route POST   "/api/projects"       => create_project
route PUT    "/api/projects/:id"   => update_project
route DELETE "/api/projects/:id"   => delete_project
```

**Guard clauses** for early returns:

```tova
fn update_project(id: Int, name: String, color: String, icon: String) {
  proj = Project.find(id)
  guard proj != nil else { return nil }
  // ... proceed with update
}
```

### 2. Browser Block Basics (`src/app.tova`)

The `browser {}` block defines your frontend — reactive state, components, and UI.

**Reactive state** — assignments automatically trigger re-renders:

```tova
browser {
  state projects = []
  state tasks = []
  state current_view = "dashboard"
  state search_query = ""
}
```

**Effects** run on mount (like `useEffect` with empty deps):

```tova
effect {
  projects = server.list_projects()
  tasks = server.list_tasks()
  stats = server.get_stats()
}
```

**Server RPC calls** — call server functions directly from browser code:

```tova
fn handle_task_submit() {
  if form_title != "" {
    server.create_task(form_project_id, form_title, ...)
    refresh_tasks()
  }
}
```

### 3. Components

Components are declared with `component Name(props)` and use JSX-like syntax:

```tova
component StatCard(label, value, color_class) {
  <div class="bg-white rounded-xl p-4 shadow-sm border border-gray-100">
    <p class="text-xs font-medium text-gray-500">"{label}"</p>
    <p class="text-2xl font-bold mt-1 {color_class}">"{value}"</p>
  </div>
}
```

**Using components:**

```tova
<StatCard label="Total Tasks" value={stats.total} color_class="text-gray-900" />
```

**Conditional rendering** — use `if`/`else` directly inside JSX:

```tova
if current_view == "dashboard" {
  <DashboardView />
} else {
  <TaskListView />
}
```

**Loops** — use `for` inside JSX:

```tova
for task in get_filtered_tasks() {
  <div class="mb-3">
    <TaskCard task={task} />
  </div>
}
```

**Event handlers** — use `on:click`, `on:input`, `on:change`:

```tova
<button on:click={fn() handle_status_toggle(task)}>
  "Click me"
</button>

<input on:input={fn(e) search_query = e.target.value} />
```

### 4. Pattern Matching

Tova's `match` expression is used throughout for status/priority logic:

```tova
fn handle_status_toggle(task) {
  next = match task.status {
    "backlog"     => "todo"
    "todo"        => "in_progress"
    "in_progress" => "done"
    "done"        => "backlog"
    _             => "todo"
  }
  server.update_task_status(task.id, next)
}
```

### 5. List Comprehensions

Filter and transform arrays inline:

```tova
fn get_stats() {
  all_tasks = Task.all()
  todo_val = len([t for t in all_tasks if t.status == "todo"])
  high_val = len([t for t in all_tasks if t.priority == "high" or t.priority == "urgent"])
  // ...
}
```

### 6. Key Tova Syntax Patterns

| Pattern | Tova | Notes |
|---------|------|-------|
| Immutable binding | `x = 10` | Cannot be reassigned |
| Mutable binding | `var x = 10` | Can be reassigned |
| Chained conditionals | `if ... elif ... else` | NOT `else if` |
| Lambda | `fn(x) body` | NOT `fn(x) => body` |
| String interpolation | `"{variable}"` | Inside JSX text |
| Dynamic classes | `class="base {if cond { 'active' } else { '' }}"` | Inline conditionals |
| Constructor call | `Date.new()` | Maps to `new Date()` |

## Features

- **Dashboard** with 8 stat cards (total, in-progress, completed, overdue, backlog, to-do, high priority, urgent)
- **Project management** — create, edit, delete projects with color picker
- **Task CRUD** — full create, read, update, delete with modal forms
- **Status workflow** — click status circles to advance: Backlog → To Do → Active → Done
- **Filtering** — filter by status, priority, and text search (all client-side, instant)
- **Bulk actions** — "Mark All Done" per project, "Clear Completed" globally
- **Overdue highlighting** — red left border on overdue tasks
- **Sidebar navigation** — Dashboard, All Tasks, per-project views with task counts

## API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/projects` | List all projects |
| POST | `/api/projects` | Create a project |
| PUT | `/api/projects/:id` | Update a project |
| DELETE | `/api/projects/:id` | Delete project + its tasks |
| GET | `/api/tasks` | List all tasks |
| POST | `/api/tasks` | Create a task |
| PUT | `/api/tasks/:id` | Update a task |
| PUT | `/api/task-status/:id` | Quick status change |
| DELETE | `/api/tasks/:id` | Delete a task |
| DELETE | `/api/tasks/completed` | Bulk delete completed tasks |
| POST | `/api/tasks/mark-done` | Mark all tasks in a project as done |
| GET | `/api/stats` | Get dashboard statistics |

## Exercises

Try extending TaskFlow to practice Tova:

1. **Add task sorting** — sort by due date, priority, or creation date using a `sort_by` dropdown
2. **Add a task detail view** — click a task title to see a full detail panel
3. **Add task comments** — create a `Comment` model with `task_id`, `text`, `created_at`
4. **Add drag-and-drop** — reorder tasks within the list
5. **Add a `security {}` block** — add JWT authentication with protected routes
6. **Deploy to edge** — add an `edge {}` block targeting Cloudflare Workers

## Troubleshooting

**SQLite disk I/O error**: If you get `SQLITE_IOERR_SHORT_READ`, delete the database files and restart:

```bash
rm -f taskflow.db taskflow.db-shm taskflow.db-wal
bun run /path/to/tova-lang/bin/tova.js dev src
```

**Port already in use**: Kill existing processes on port 3000:

```bash
lsof -ti :3000 | xargs kill -9
```

**Compilation warnings about undefined types**: Warnings like `'Project' is not defined` are expected — model types are generated at runtime by the ORM layer. The app runs correctly despite these warnings.
