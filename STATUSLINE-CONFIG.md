# MCP Statusline Configuration

Configure Claude Code's statusline to display real-time information from MCP servers and custom metrics.

---

## Quick Setup

The statusline config is stored in `~/.claude/statusline.json`:

```json
{
  "left": [
    { "type": "mode" },
    { "type": "branch" },
    { "type": "status" }
  ],
  "right": [
    { "type": "mcp-health" },
    { "type": "token-count" },
    { "type": "memory" }
  ],
  "refresh_interval_ms": 1000
}
```

---

## Statusline Components

### Left Side (Primary Info)

| Component | Type | Shows |
|-----------|------|-------|
| Mode | `mode` | Current mode (brainstorming, planning, executing, etc.) |
| Branch | `branch` | Active git branch name |
| Status | `status` | File changes, uncommitted work |

### Right Side (Metrics & Status)

| Component | Type | Shows |
|-----------|------|-------|
| MCP Health | `mcp-health` | Connected MCP servers (Context7, claude-mem, etc.) |
| Token Count | `token-count` | Current session token usage |
| Memory | `memory` | Claude-Mem observations injected |
| Time | `time` | Current time |
| Python Version | `python-version` | Active Python version |

---

## Full Configuration Example

```json
{
  "left": [
    {
      "type": "mode",
      "format": "[{mode}]"
    },
    {
      "type": "branch",
      "format": "({branch})"
    },
    {
      "type": "status",
      "format": "{changes}"
    }
  ],
  "right": [
    {
      "type": "mcp-health",
      "format": "MCPs: {healthy}/{total}",
      "show_failed": true
    },
    {
      "type": "token-count",
      "format": "tokens: {used}/{total}"
    },
    {
      "type": "memory",
      "format": "mem: {observations} obs"
    },
    {
      "type": "time",
      "format": "%H:%M"
    }
  ],
  "refresh_interval_ms": 1000,
  "colors": {
    "mode": "blue",
    "branch": "green",
    "mcp_connected": "green",
    "mcp_failed": "red",
    "token_warning": "yellow",
    "token_critical": "red"
  },
  "thresholds": {
    "token_warning_percent": 50,
    "token_critical_percent": 75
  }
}
```

---

## Component Details

### Mode (`type: "mode"`)

Shows your current Superpowers phase or custom mode.

```json
{
  "type": "mode",
  "format": "[{mode}]",
  "colorize": true
}
```

**Shows:** `[brainstorming]`, `[planning]`, `[executing]`, `[verifying]`

### Branch (`type: "branch"`)

Displays active git branch from your project worktree.

```json
{
  "type": "branch",
  "format": "({branch})",
  "max_length": 20,
  "truncate_indicator": "…"
}
```

**Shows:** `(feature/auth-tokens)`, `(main)`

### Status (`type: "status"`)

Indicates uncommitted changes, staged files, or other git status.

```json
{
  "type": "status",
  "format": "{changes}",
  "show_ahead_behind": true,
  "show_untracked": true
}
```

**Shows:** `↑2↓1` (2 commits ahead, 1 behind), `M` (modified), `?` (untracked)

### MCP Health (`type: "mcp-health"`)

Shows connected MCP servers and their status. Critical for seeing what tools are available.

```json
{
  "type": "mcp-health",
  "format": "MCPs: {healthy}/{total}",
  "show_names": false,
  "show_failed": true,
  "compact": true
}
```

**Shows:** `MCPs: 4/4` (all 4 MCPs healthy), `MCPs: 3/4 ⚠️ context7` (1 failed)

**Default MCPs to monitor:**
- `context7` — Documentation fetching
- `claude-mem` — Cross-session memory
- `superpowers` — Methodology skills
- `trail-of-bits` — Security analysis

### Token Count (`type: "token-count"`)

Displays current session token usage and remaining context window.

```json
{
  "type": "token-count",
  "format": "tokens: {used}/{total}",
  "show_percentage": true,
  "color_thresholds": {
    "warning": 50,
    "critical": 75
  }
}
```

**Shows:** `tokens: 45k/200k (22%)`, `tokens: 155k/200k (78%) 🔴`

### Memory (`type: "memory"`)

Shows Claude-Mem observations injected and available cross-session context.

```json
{
  "type": "memory",
  "format": "mem: {observations} obs ({tokens} tokens)",
  "show_confidence": true
}
```

**Shows:** `mem: 12 obs (2.2k tokens)`, `mem: 0 obs (first session)`

### Time (`type: "time"`)

Current time (useful for long sessions).

```json
{
  "type": "time",
  "format": "%H:%M:%S",
  "update_interval_ms": 1000
}
```

**Shows:** `14:32:15`

### Python Version (`type: "python-version"`)

Active Python interpreter (useful when running Trail of Bits or Plunderer).

```json
{
  "type": "python-version",
  "format": "py{version}",
  "show_path": false
}
```

**Shows:** `py3.11`, `py3.10.5`

---

## Color Schemes

### Predefined Schemes

```json
{
  "color_scheme": "dark",  // or "light", "auto"
  "colors": {
    "mode": "blue",
    "branch": "green",
    "status_clean": "green",
    "status_dirty": "yellow",
    "status_untracked": "gray",
    "mcp_connected": "green",
    "mcp_failed": "red",
    "mcp_warning": "yellow",
    "token_normal": "white",
    "token_warning": "yellow",
    "token_critical": "red",
    "memory_high": "green",
    "memory_medium": "yellow",
    "memory_low": "red"
  }
}
```

### Custom Colors

```json
{
  "colors": {
    "mode": "#0088ff",
    "branch": "#00aa00",
    "mcp_connected": "#00ff00",
    "mcp_failed": "#ff0000"
  }
}
```

---

## Practical Configurations

### Minimal (Just Essentials)

```json
{
  "left": [
    { "type": "mode" },
    { "type": "status" }
  ],
  "right": [
    { "type": "mcp-health" }
  ],
  "refresh_interval_ms": 2000
}
```

### Developer (Comprehensive)

```json
{
  "left": [
    { "type": "mode" },
    { "type": "branch" },
    { "type": "status" }
  ],
  "right": [
    { "type": "mcp-health", "show_names": true },
    { "type": "token-count", "show_percentage": true },
    { "type": "memory" },
    { "type": "time" }
  ],
  "refresh_interval_ms": 1000,
  "thresholds": {
    "token_warning_percent": 50,
    "token_critical_percent": 75
  }
}
```

### Agent Teams (Multi-Agent Focus)

```json
{
  "left": [
    { "type": "mode" },
    { "type": "branch" }
  ],
  "right": [
    { "type": "mcp-health", "show_failed": true },
    { "type": "token-count", "show_percentage": true },
    { "type": "memory", "show_confidence": true },
    { "type": "time", "format": "%H:%M" }
  ],
  "refresh_interval_ms": 500,
  "alerts": {
    "mcp_failure": true,
    "token_critical": true,
    "memory_injection_failed": true
  }
}
```

### Minimal Token (Low-Overhead)

```json
{
  "left": [
    { "type": "mode" }
  ],
  "right": [
    { "type": "token-count" }
  ],
  "refresh_interval_ms": 5000
}
```

---

## Advanced Features

### Conditional Display

Show/hide components based on conditions:

```json
{
  "components": [
    {
      "type": "token-count",
      "show_when": "token_percent > 50"
    },
    {
      "type": "memory",
      "show_when": "observations_available > 0"
    },
    {
      "type": "mcp-health",
      "show_when": "any_mcp_failed"
    }
  ]
}
```

### Alerts

Trigger notifications when thresholds are crossed:

```json
{
  "alerts": {
    "token_critical": {
      "threshold_percent": 80,
      "message": "Context window filling up!"
    },
    "mcp_failure": {
      "enabled": true,
      "message": "MCP {name} disconnected"
    },
    "memory_injection_failed": {
      "enabled": true,
      "message": "Claude-Mem injection failed"
    }
  }
}
```

### Aliases

Create short aliases for long modes:

```json
{
  "aliases": {
    "mode": {
      "brainstorming": "🧠",
      "planning": "📋",
      "executing": "⚙️",
      "verifying": "✓",
      "autonomous": "🤖"
    }
  }
}
```

---

## Troubleshooting

### MCP Health Shows "Failed"

**Symptom:** `MCPs: 3/4 ⚠️ context7`

**Fixes:**
1. Check MCP is running: `claude mcp list`
2. Restart Claude Code
3. Reinstall if needed: `claude mcp add context7 -- npx -y @upstash/context7-mcp@latest`

### Token Count Not Updating

**Symptom:** Token count freezes or doesn't refresh

**Fixes:**
1. Increase `refresh_interval_ms` (slower updates consume fewer tokens)
2. Check if in compaction state (token counting may be paused)
3. Verify `~/.claude/statusline.json` is valid JSON

### Memory Shows 0 Observations

**Symptom:** `mem: 0 obs` even after multiple sessions

**Fixes:**
1. Check Claude-Mem running: `curl http://localhost:37777/api/readiness`
2. Verify observations captured: `sqlite3 ~/.claude-mem/claude-mem.db "SELECT COUNT(*) FROM observations;"`
3. Check `contextInjection.enabled` in `~/.claude-mem/settings.json`

### Statusline Flickers

**Symptom:** Statusline updates too frequently, distracting

**Fix:** Increase `refresh_interval_ms`:
```json
{
  "refresh_interval_ms": 2000  // Update every 2 seconds instead of 1
}
```

---

## Integration with UNIFIED-GUIDE.md

The statusline is a **real-time diagnostic tool** for the entire toolkit:

- **Mode display** — Shows which Superpowers phase you're in
- **MCP health** — Confirms Context7, Claude-Mem, Trail of Bits are connected
- **Token count** — Warns when context window is filling (watch for critical threshold)
- **Memory observations** — Confirms Claude-Mem is actively learning across sessions

**Recommended setup:** Use the "Developer (Comprehensive)" config above. It gives you full visibility into the stack's health without overwhelming the display.

---

## Example Statusline States

### Healthy Session
```
[planning] (feature/agent-teams) M    MCPs: 4/4    tokens: 45k/200k (22%)    mem: 8 obs (2.2k tokens)    14:32
```

### Warning: Context Filling Up
```
[executing] (main) ↑2    MCPs: 4/4    tokens: 155k/200k (78%) 🟡    mem: 12 obs (2.5k tokens)    14:45
```

### Error: MCP Failed
```
[brainstorming] (debug/context7) M    MCPs: 3/4 ⚠️ context7    tokens: 30k/200k (15%)    mem: 0 obs    14:20
```

### Multi-Agent Run
```
[autonomous] (parallel-agents) ↑5↓2    MCPs: 4/4    tokens: 120k/200k (60%)    mem: 15 obs (3.1k tokens)    15:00
```

---

## See Also

- `UNIFIED-GUIDE.md` — Main toolkit documentation
- `~/.claude/statusline.json` — Your statusline config
- `claude mcp list` — Check connected MCPs
- `curl http://localhost:37777/api/readiness` — Check Claude-Mem worker status
