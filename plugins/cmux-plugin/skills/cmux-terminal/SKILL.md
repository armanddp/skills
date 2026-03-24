---
name: cmux-terminal
description: cmux terminal app management for workspaces, split panes, browser automation, notifications, sidebar status, and multi-agent workflows. Use when working with cmux, terminal workspaces, split panes, embedded browser, notifications, sidebar metadata, or orchestrating multiple Claude Code sessions. Also use when the user wants to open a browser, show progress, send notifications, or manage terminal layout.
---

# cmux Terminal Management Skill

You are an expert at using cmux — a native macOS terminal app purpose-built for AI coding agent workflows. cmux provides GPU-accelerated terminal rendering, workspaces, split panes, an embedded scriptable browser, notifications, and sidebar metadata — all controllable via CLI and Unix socket.

## Concepts & Hierarchy

cmux uses a four-level hierarchy:

```
Window > Workspace (sidebar tab) > Pane (split region) > Surface (tab within pane)
```

- **Window**: Top-level OS window
- **Workspace**: A sidebar tab grouping related work (like tmux sessions)
- **Pane**: A split region within a workspace (horizontal/vertical splits)
- **Surface**: A terminal or browser tab within a pane

Surfaces can be **terminal** or **browser** type. Each surface has a unique ID.

## Environment Detection

cmux sets these environment variables in spawned shells:

```bash
TERM_PROGRAM=ghostty
CMUX_SOCKET_PATH=/path/to/cmux.sock
CMUX_WORKSPACE_ID=<uuid>
CMUX_SURFACE_ID=<uuid>
```

To check if running inside cmux:
```bash
cmux ping  # Returns "PONG" if cmux is running
```

## CLI Reference

All commands support `--json` for machine-readable output. Use `--socket PATH` to target a specific socket.

### Workspace Management

```bash
# List all workspaces
cmux list-workspaces --json

# Create a new workspace
cmux new-workspace

# Switch to a workspace
cmux select-workspace --workspace <id>

# Get the active workspace
cmux current-workspace

# Close a workspace
cmux close-workspace --workspace <id>
```

### Split Panes

```bash
# Create a split (directions: left, right, up, down)
cmux new-split right
cmux new-split down

# List all surfaces in current workspace
cmux list-surfaces

# Focus a specific surface
cmux focus-surface --surface <id>
```

### Sending Input

```bash
# Send text to the focused surface
cmux send "ls -la"

# Send a keypress
cmux send-key enter
cmux send-key tab
cmux send-key escape
cmux send-key up
cmux send-key down

# Send to a specific surface (by ID)
cmux send-surface --surface <id> "npm test"
cmux send-key-surface --surface <id> enter
```

### Notifications

Use notifications to alert the user about task completion, errors, or important events.

```bash
# Send a notification
cmux notify --title "Build Complete" --body "All tests passed" --subtitle "Project X"

# List all notifications
cmux list-notifications

# Clear all notifications
cmux clear-notifications
```

### Sidebar Status & Progress

The sidebar shows metadata next to each workspace tab — use it to communicate state at a glance.

```bash
# Set a status pill (key-value with icon and color)
cmux set-status "build" "passing" --icon "checkmark.circle.fill" --color "#34D399"
cmux set-status "env" "staging" --icon "server.rack" --color "#FBBF24"
cmux set-status "branch" "feature/auth" --icon "arrow.triangle.branch" --color "#60A5FA"

# Clear a status entry
cmux clear-status "build"

# List all status entries
cmux list-status

# Set a progress bar (0.0 to 1.0)
cmux set-progress 0.75 --label "Running tests..."

# Clear the progress bar
cmux clear-progress
```

**Common status icons** (SF Symbols):
- `checkmark.circle.fill` — success/passing
- `xmark.circle.fill` — failure/error
- `arrow.triangle.branch` — git branch
- `server.rack` — environment/server
- `gearshape` — config/settings
- `bolt.fill` — active/running
- `clock` — waiting/scheduled
- `exclamationmark.triangle.fill` — warning

### Sidebar Log

Append structured log entries to the sidebar for real-time activity tracking.

```bash
# Log a message (levels: info, progress, success, warning, error)
cmux log "Starting deployment..." --level progress --source "deploy"
cmux log "Tests passed" --level success --source "test"
cmux log "Missing env var" --level warning --source "config"
cmux log "Build failed: exit 1" --level error --source "build"

# List recent log entries
cmux list-log --limit 20

# Clear all log entries
cmux clear-log
```

### Sidebar State Dump

```bash
# Get all sidebar metadata (status, progress, logs) at once
cmux sidebar-state
```

### Utility

```bash
# Check if cmux is running
cmux ping

# List all available methods and access mode
cmux capabilities

# Show focused context (window/workspace/pane/surface IDs)
cmux identify
```

## Browser Automation

cmux embeds a full scriptable browser (Playwright-powered). Open it in a split or standalone.

### Opening & Navigation

```bash
# Open URL in a new browser split
cmux browser open "https://localhost:3000"

# Open in a specific split direction
cmux browser open-split "https://example.com" right

# Navigate current browser to a new URL
cmux browser navigate "https://example.com/dashboard"

# Back/forward/reload
cmux browser back
cmux browser forward
cmux browser reload

# Get current URL
cmux browser url
```

### Waiting for State

Always wait for pages/elements to be ready before interacting:

```bash
# Wait for page load
cmux browser wait --load-state networkidle

# Wait for a selector to appear
cmux browser wait --selector "#login-form" --timeout-ms 5000

# Wait for text to appear
cmux browser wait --text "Welcome back"

# Wait for URL to contain a string
cmux browser wait --url-contains "/dashboard"

# Wait for a JS function to return true
cmux browser wait --function "() => document.readyState === 'complete'"
```

### DOM Interaction

```bash
# Click, double-click, hover
cmux browser click "#submit-btn"
cmux browser dblclick ".editable-cell"
cmux browser hover ".tooltip-trigger"

# Type text (simulates keystrokes) vs fill (sets value instantly)
cmux browser type "#search" "query text"
cmux browser fill "#email" "user@example.com"

# Press a key
cmux browser press "Enter"

# Check/uncheck checkboxes
cmux browser check "#agree-terms"
cmux browser uncheck "#newsletter"

# Select from dropdown
cmux browser select "#country" "US"

# Scroll element into view
cmux browser scroll-into-view "#footer"

# Focus an element
cmux browser focus "#input-field"
```

### Inspecting the Page

```bash
# Get accessibility snapshot (best for understanding page structure)
cmux browser snapshot
cmux browser snapshot --compact          # Minimal output
cmux browser snapshot --interactive      # Only interactive elements
cmux browser snapshot --selector "#main" # Scope to element
cmux browser snapshot --max-depth 3      # Limit nesting

# Get page title
cmux browser get title

# Get text content of an element
cmux browser get text "#message"

# Get HTML of an element
cmux browser get html "#container"

# Get input value
cmux browser get value "#email-input"

# Get attribute value
cmux browser get attr "#link" "href"

# Count matching elements
cmux browser get count ".list-item"

# Get bounding box
cmux browser get box "#element"

# Get computed styles
cmux browser get styles "#element" "color,background-color"

# Check element state
cmux browser is visible "#modal"
cmux browser is enabled "#submit"
cmux browser is checked "#checkbox"
```

### Finding Elements

```bash
# Find by various strategies (returns selector for chaining)
cmux browser find role "button"
cmux browser find text "Sign In"
cmux browser find label "Email"
cmux browser find placeholder "Search..."
cmux browser find testid "submit-btn"
cmux browser find title "Close dialog"
cmux browser find alt "Logo image"

# Positional
cmux browser find first ".card"
cmux browser find last ".card"
cmux browser find nth ".card" 2

# Highlight element (visual debugging)
cmux browser highlight "#target"
```

### Screenshots

```bash
# Take a screenshot and save to file
cmux browser screenshot --out /tmp/screenshot.png
```

### JavaScript Execution

```bash
# Evaluate JavaScript in the page
cmux browser eval "document.title"
cmux browser eval "window.scrollTo(0, document.body.scrollHeight)"

# Add a script to the page
cmux browser addscript "console.log('injected')"

# Add an init script (runs before page scripts on every navigation)
cmux browser addinitscript "window.__test = true"

# Add custom CSS
cmux browser addstyle "body { outline: 1px solid red; }"
```

### Cookies & Storage

```bash
# Get/set/clear cookies
cmux browser cookies get
cmux browser cookies set '{"name":"token","value":"abc123","domain":"localhost"}'
cmux browser cookies clear

# Local/session storage
cmux browser storage local get "authToken"
cmux browser storage local set "debug" "true"
cmux browser storage session clear
```

### Browser Tabs

```bash
# List browser tabs
cmux browser tab list

# Open new tab
cmux browser tab new "https://example.com"

# Switch tab
cmux browser tab switch <index>

# Close tab
cmux browser tab close <index>
```

### State Persistence

```bash
# Save browser state (cookies, storage, etc.)
cmux browser state save "my-session"

# Load saved state
cmux browser state load "my-session"
```

### Dialogs & Frames

```bash
# Handle browser dialogs (alert, confirm, prompt)
cmux browser dialog accept
cmux browser dialog dismiss

# Switch to iframe
cmux browser frame "#iframe-selector"

# Switch back to main frame
cmux browser frame main
```

### Console & Errors

```bash
# List console messages
cmux browser console list
cmux browser console clear

# List JavaScript errors
cmux browser errors list
```

## Common Workflows

### Multi-Agent Layout

Set up a workspace with multiple Claude Code sessions working in parallel:

```bash
# Create splits for parallel agents
cmux new-split right    # Side-by-side
cmux new-split down     # Stack below

# Get surface IDs
cmux list-surfaces --json

# Send commands to specific surfaces
cmux send-surface --surface <id1> "claude 'implement auth module'"
cmux send-surface --surface <id2> "claude 'write tests for auth'"
```

### Development Dashboard

Open your app alongside a browser for live testing:

```bash
# Start dev server in current surface
cmux send "npm run dev"

# Open browser in a split to preview
cmux browser open-split "http://localhost:3000" right

# Set status to show what's running
cmux set-status "server" "running" --icon "bolt.fill" --color "#34D399"
cmux set-status "url" "localhost:3000" --icon "globe" --color "#60A5FA"
```

### Task Progress Tracking

Use sidebar progress and logs to show long-running task status:

```bash
# Start a task
cmux set-progress 0.0 --label "Deploying..."
cmux log "Starting deployment" --level progress --source "deploy"

# Update as work progresses
cmux set-progress 0.33 --label "Building..."
cmux log "Docker build complete" --level success --source "deploy"

cmux set-progress 0.66 --label "Pushing..."
cmux log "Image pushed to registry" --level success --source "deploy"

# Complete
cmux set-progress 1.0 --label "Done"
cmux log "Deployment successful" --level success --source "deploy"
cmux notify --title "Deploy Complete" --body "Production updated"

# Clean up after a moment
cmux clear-progress
```

### QA Testing with Browser

Use the embedded browser to test a web application:

```bash
# Open the app
cmux browser open "http://localhost:3000"
cmux browser wait --load-state networkidle

# Take a baseline screenshot
cmux browser screenshot --out /tmp/before.png

# Interact with the page
cmux browser fill "#username" "testuser"
cmux browser fill "#password" "testpass"
cmux browser click "#login-btn"

# Wait for navigation
cmux browser wait --url-contains "/dashboard"
cmux browser wait --selector ".dashboard-content"

# Verify state
cmux browser get text ".welcome-message"
cmux browser screenshot --out /tmp/after-login.png

# Check for errors
cmux browser errors list
cmux browser console list
```

### Reading Terminal Output

```bash
# Read text from a specific surface (useful to check command output)
cmux read-text --surface <id>
```

## Socket API

For programmatic control, cmux exposes a Unix socket (JSON protocol):

```bash
# Socket path (from capabilities or env)
SOCKET="/Users/$USER/Library/Application Support/cmux/cmux.sock"

# Send a command
echo '{"method":"system.ping","params":{}}' | nc -U "$SOCKET"

# Example: create a notification
echo '{"method":"notification.create","params":{"title":"Hello","body":"World"}}' | nc -U "$SOCKET"
```

Methods mirror CLI commands: `workspace.list`, `workspace.create`, `surface.split`, `surface.send_text`, `notification.create`, `browser.navigate`, etc.

## Keyboard Shortcuts Reference

| Action | Shortcut |
|---|---|
| New workspace | Cmd+N |
| Jump to workspace 1-8 | Cmd+1 through Cmd+8 |
| Last workspace | Cmd+9 |
| Close workspace | Cmd+Shift+W |
| Rename workspace | Cmd+Shift+R |
| New surface tab | Cmd+T |
| Close surface | Cmd+W |
| Split right | Cmd+D |
| Split down | Cmd+Shift+D |
| Focus split (directional) | Opt+Cmd+Arrow |
| Open browser | Cmd+Shift+L |
| Browser address bar | Cmd+L |
| Browser dev tools | Opt+Cmd+I |
| Notification panel | Cmd+Shift+I |
| Latest unread notification | Cmd+Shift+U |
| Settings | Cmd+, |

## Configuration

cmux reads Ghostty config from `~/.config/ghostty/config` or `~/Library/Application Support/com.mitchellh.ghostty/config`.

Supported config keys: `font-family`, `font-size`, `theme`, `background`, `foreground`, `cursor-color`, `selection-background`, `selection-foreground`, `unfocused-split-opacity`, `unfocused-split-fill`, `split-divider-color`, `scrollback-limit`, `working-directory`.

In-app settings (Cmd+,) control:
- **Theme mode**: system / light / dark
- **Automation mode**: off / cmuxOnly / allowAll
- **Browser link behavior**: how links open

## Safety Guidelines

- Always use `--json` when parsing output programmatically
- Use surface IDs (from `list-surfaces --json`) to target specific terminals — never assume focus
- When sending commands to surfaces, include `enter` key if execution is needed:
  ```bash
  cmux send-surface --surface <id> "npm test"
  cmux send-key-surface --surface <id> enter
  ```
- Check `cmux ping` before running workflows — cmux may not be running
- Browser automation requires a browser surface to be open first — use `cmux browser open` before other browser commands
- Be careful with `cmux browser eval` — only run trusted JavaScript
- Prefer `cmux browser snapshot --interactive` over full snapshots when you only need actionable elements
