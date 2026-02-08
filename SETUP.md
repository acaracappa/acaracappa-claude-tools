# Setup Guide

Complete installation for the Claude Code Toolkit: five development-layer tools and three feedback-layer tools.

**Time estimate:** 20–30 minutes for a clean install. Prerequisites first, then tools in order.

---

## Prerequisites

```bash
# Node.js (required for Context7 MCP)
node --version  # v18+ required

# Bun (required for Claude-Mem worker)
# macOS:
brew install oven-sh/bun/bun
# Linux:
curl -fsSL https://bun.sh/install | bash

# Python 3 (required for Transcripts and Plunderer)
python3 --version  # 3.8+ required

# pipx (recommended for Transcripts install)
brew install pipx  # or: pip install pipx

# Claude Code CLI
claude --version  # Ensure Claude Code is installed and working
```

---

## Development Layer

### Step 1: Superpowers (Foundation Layer)

Transforms Claude Code from a reactive coder into a methodical engineer.

```bash
# In Claude Code terminal:
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace

# Restart Claude Code, then verify:
/help
# Should show:
#   /superpowers:brainstorm
#   /superpowers:write-plan
#   /superpowers:execute-plan
```

**What it provides:**

- **brainstorming** — Forces Claude to ask questions and explore alternatives before coding
- **using-git-worktrees** — Isolates work on new branches, verifies clean test baseline
- **test-driven-development** — RED → GREEN → REFACTOR on every implementation
- **subagent-driven-development** — Spawns subagents for tasks with two-stage review
- **systematic-debugging** — Four-phase debugging methodology, auto-activated on errors
- **verification-before-completion** — Blocks Claude from claiming "done" without evidence

**How it works at runtime:** On session start, Superpowers injects a bootstrap prompt that directs Claude to read the skill library. Claude discovers and uses skills contextually from that point forward.

---

### Step 2: Claude-Mem (Memory Layer)

Provides persistent memory across sessions.

```bash
# In Claude Code terminal:
/plugin marketplace add thedotmack/claude-mem
/plugin install claude-mem

# Restart Claude Code, then verify:
curl http://localhost:37777/api/readiness
# Should return JSON with status
```

**What it provides:**

- **5 lifecycle hooks** — SessionStart, UserPromptSubmit, PostToolUse, Stop, SessionEnd
- **Worker service** — Express API on port 37777, managed by Bun
- **SQLite database** at `~/.claude-mem/claude-mem.db` — stores compressed observations
- **Chroma vector embeddings** — semantic search across past sessions
- **Web viewer** at `http://localhost:37777` — browse memory in real-time
- **mem-search skill** — Claude queries past work with natural language
- **Privacy controls** — `<private>` tags exclude sensitive content

**Configuration:**

```json
// ~/.claude-mem/settings.json
{
  "workerPort": 37777,
  "logLevel": "info",
  "contextInjection": {
    "enabled": true,
    "maxTokens": 2250
  }
}
```

Reduce `maxTokens` to 1000 if working with large files and needing context headroom. Increase to 3000 for richer cross-session context.

**How it works with Superpowers:**

```
Session Start
  ├── Superpowers: Injects skill bootstrap prompt
  ├── Claude-Mem: Starts worker, injects relevant past context
User Prompt
  ├── Superpowers: Skills activate based on task context
  ├── Claude-Mem: Records prompt for observation
Post Tool Use
  ├── Claude-Mem: Captures tool output, compresses with AI
Session End
  ├── Claude-Mem: Generates session summary, stores in DB
```

---

### Step 3: Context7 (Knowledge Layer)

Fetches current, version-specific documentation from library source repos at query time. Eliminates hallucinated APIs and deprecated methods.

```bash
# Option A: MCP Server Only (minimal, always do this)
claude mcp add context7 -- npx -y @upstash/context7-mcp@latest

# Option B: Full Plugin (recommended, adds skills + agents + commands)
# In Claude Code terminal:
claude plugin marketplace add context7/context7-marketplace
claude plugin install context7@context7-marketplace

# Verify:
claude mcp list
# Should show: context7 - ✓ Connected
```

**Capability comparison:**

| Capability | MCP Only | Full Plugin |
|-----------|----------|-------------|
| `resolve-library-id` tool | ✅ | ✅ |
| `query-docs` tool | ✅ | ✅ |
| Auto-triggering skill (no "use context7" needed) | ❌ | ✅ |
| `docs-researcher` subagent (isolated context) | ❌ | ✅ |
| `/context7:docs` slash command | ❌ | ✅ |

**Usage patterns:**

```bash
# Explicit invocation (MCP-only install)
"use context7 to show me how to set up middleware in gin v1.10"
"use context7 with /tokio-rs/tokio for async channel patterns"

# Implicit invocation (full plugin)
"implement the WebSocket handler using gorilla/websocket"
# → Context7 auto-detects library mention, fetches docs

# Subagent pattern (full plugin, good during long sessions)
"spawn docs-researcher to look up rust sqlx connection pooling"
```

**Optional — API key for higher rate limits:**

```bash
claude mcp add --header "CONTEXT7_API_KEY: YOUR_KEY" \
  --transport http context7 https://mcp.context7.com/mcp
```

Free key available at `context7.com/dashboard`.

---

### Step 4: Trail of Bits Security Skills (Security Layer)

Professional-grade security analysis from one of the most respected audit firms in the industry. Skills activate on-demand when Claude touches security-relevant code.

```bash
# In Claude Code terminal:
/plugin marketplace add trailofbits/skills

# Install the five core plugins:
/plugin install sharp-edges@trailofbits/skills
/plugin install differential-review@trailofbits/skills
/plugin install static-analysis@trailofbits/skills
/plugin install insecure-defaults@trailofbits/skills
/plugin install fix-review@trailofbits/skills

# Verify:
/plugin list
# Should show all five plugins
```

**What each plugin does:**

| Plugin | What It Does |
|--------|-------------|
| **sharp-edges** | Identifies error-prone APIs, dangerous configs, footgun designs |
| **differential-review** | Security-focused code review with git history analysis + blast radius estimation |
| **static-analysis** | CodeQL + Semgrep + SARIF parsing for automated vulnerability scanning |
| **insecure-defaults** | Detects hardcoded secrets, default credentials, weak crypto |
| **fix-review** | Verifies fix commits address findings without introducing new bugs |

**How it works at runtime:** Skills are on-demand — zero tokens at idle, activate only when Claude touches relevant code. During Superpowers' verification-before-completion phase, Trail of Bits skills naturally contribute security evidence.

**Additional plugins (install later as needed):**

- `testing-handbook-skills` — Application Security Testing Handbook
- `variant-analysis` — Find similar vulnerabilities via pattern-based analysis
- `audit-context-building` — Line-by-line semantic analysis for deep security reviews
- `semgrep-rule-creator` — Write custom Semgrep rules for your vulnerability patterns
- `constant-time-analysis` — Detect compiler-induced timing side-channels

**Note:** Static analysis with Semgrep requires `pip install semgrep`. The skill will tell you if it's missing.

---

## Feedback Layer

### Step 5: Claude Diary (Observation + Reflection)

Two slash commands and a PreCompact hook. No dependencies, no servers, no database — just markdown prompts.

```bash
# Clone and install
git clone https://github.com/rlancemartin/claude-diary ~/tools/claude-diary
cp ~/tools/claude-diary/commands/*.md ~/.claude/commands/
mkdir -p ~/.claude/hooks
cp ~/tools/claude-diary/hooks/pre-compact.sh ~/.claude/hooks/
chmod +x ~/.claude/hooks/pre-compact.sh

# Configure PreCompact hook in ~/.claude/settings.json:
```

Add to `~/.claude/settings.json`:

```json
{
  "hooks": {
    "PreCompact": [{
      "matcher": "auto",
      "hooks": [{
        "type": "command",
        "command": "bash ~/.claude/hooks/pre-compact.sh"
      }]
    }]
  }
}
```

```bash
# Verify:
ls ~/.claude/commands/diary.md ~/.claude/commands/reflect.md
```

**What it provides:**

- `/diary` — Captures structured session observation to `~/.claude/memory/diary/`
- `/reflect` — Reads accumulated diary entries, finds patterns, updates `~/.claude/CLAUDE.md` with learned rules
- PreCompact hook — Auto-fires `/diary` before context compression on long sessions

**Output structure:**

```
~/.claude/memory/
├── diary/
│   ├── 2026-02-01-session-1.md
│   ├── 2026-02-05-session-1.md
│   └── processed.log
├── reflections/
│   └── 2026-02-reflection-1.md
```

---

### Step 6: claude-code-transcripts (Archival)

Converts raw JSONL session files into browsable, paginated HTML.

```bash
pipx install claude-code-transcripts

# Verify:
claude-code-transcripts --help
```

**Usage:**

```bash
# Interactive picker for recent sessions
claude-code-transcripts local

# Bulk convert all sessions
claude-code-transcripts all -o ~/claude-archives/ --include-agents --json

# Convert specific file
claude-code-transcripts json ~/.claude/projects/<hash>/<session>.jsonl
```

**Optional flags:** `--gist` (publish to GitHub Gist), `--json` (archive raw source alongside HTML), `--include-agents` (capture subagent sessions), `--repo OWNER/NAME` (auto-link git commits).

---

### Step 7: Prompt Plunderer (Pattern Mining)

Standalone Python script, zero dependencies beyond standard library.

```bash
git clone https://github.com/levindixon/claude-code-prompt-plunderer ~/tools/plunderer

# Verify:
python3 ~/tools/plunderer/plunder_prompts.py --help
```

**Usage:**

```bash
# Extract prompts
python3 ~/tools/plunderer/plunder_prompts.py --min-length 50 --max-length 2000

# Generate slash commands from patterns
claude "$(cat ~/tools/plunderer/prompts/slash_command_discovery.md)"
```

---

## Automation Setup

### Cron Jobs

```bash
mkdir -p ~/bin ~/claude-archives
```

**Weekly archive + cleanup + prompt extraction:**

```bash
#!/bin/bash
# ~/bin/claude-weekly.sh

WEEK=$(date +%Y-%W)
ARCHIVE_DIR="$HOME/claude-archives/$WEEK"
mkdir -p "$ARCHIVE_DIR"

# Archive sessions to HTML before auto-cleanup can delete them
claude-code-transcripts all -o "$ARCHIVE_DIR" --include-agents --json

# Extract prompts for pattern analysis
python3 ~/tools/plunderer/plunder_prompts.py \
  --min-length 30 --max-length 2000 \
  --output "$ARCHIVE_DIR/prompts.json"

# Clean old raw JSONL (already preserved in archive)
find ~/.claude/projects/ -name "*.jsonl" -mtime +90 -delete
find ~/.claude/file-history/ -mtime +90 -delete

echo "$(date): archived week $WEEK" >> ~/claude-archives/archive.log
```

```bash
chmod +x ~/bin/claude-weekly.sh
```

**Daily reflect reminder:**

```bash
#!/bin/bash
# ~/bin/claude-reflect-check.sh

DIARY_DIR="$HOME/.claude/memory/diary"
PROCESSED="$DIARY_DIR/processed.log"

TOTAL=$(find "$DIARY_DIR" -name "*.md" 2>/dev/null | wc -l)
PROCESSED_COUNT=$(wc -l < "$PROCESSED" 2>/dev/null || echo 0)
UNPROCESSED=$((TOTAL - PROCESSED_COUNT))

if [ "$UNPROCESSED" -ge 8 ]; then
  echo "⚠️ $UNPROCESSED unprocessed diary entries. Run /reflect." \
    > ~/.claude/memory/reflect-reminder.md

  # macOS notification (optional)
  osascript -e "display notification \"$UNPROCESSED diary entries waiting\" \
    with title \"Claude: Run /reflect\"" 2>/dev/null
fi
```

```bash
chmod +x ~/bin/claude-reflect-check.sh
```

**Register cron jobs:**

```bash
crontab -e
# Add:
# 0 3 * * 0 ~/bin/claude-weekly.sh
# 0 9 * * * ~/bin/claude-reflect-check.sh
```

### CLAUDE.md Version Tracking

```bash
cd ~/.claude && git init && git add CLAUDE.md && git commit -m "initial"
```

Make Claude see the reflect reminder by adding to `~/.claude/CLAUDE.md`:

```markdown
@~/.claude/memory/reflect-reminder.md
```

### Slash Command Generator

Create `~/.claude/commands/generate-commands.md`:

```markdown
---
description: Analyze prompt patterns and generate slash commands from repetitive workflows
---

Read the most recent prompt analysis file from ~/claude-archives/ (find the latest prompts.json).

Analyze the extracted prompts for:
1. Prompts that appear 3+ times across sessions
2. Prompts that follow a consistent pattern (same structure, different parameters)
3. Multi-step sequences that always appear together

For each pattern found:
1. Generate a slash command .md file with a descriptive name
2. Include a description frontmatter field
3. Use $ARGUMENTS for parameterized parts
4. Write the file to ~/.claude/commands/

Report what was created. Do not overwrite existing commands.
```

### Session Retention

Extend the default 30-day cleanup period to give archives time to run. Add to `~/.claude/settings.json`:

```json
{
  "cleanupPeriodDays": 90
}
```

---

## Quick Install (TL;DR)

```bash
# === Development Layer (in Claude Code terminal) ===

# 1. Superpowers
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace

# 2. Claude-Mem
/plugin marketplace add thedotmack/claude-mem
/plugin install claude-mem

# 3. Context7
claude mcp add context7 -- npx -y @upstash/context7-mcp@latest
claude plugin marketplace add context7/context7-marketplace
claude plugin install context7@context7-marketplace

# 4. Trail of Bits
/plugin marketplace add trailofbits/skills
/plugin install sharp-edges@trailofbits/skills
/plugin install differential-review@trailofbits/skills
/plugin install static-analysis@trailofbits/skills
/plugin install insecure-defaults@trailofbits/skills
/plugin install fix-review@trailofbits/skills

# 5. Restart Claude Code

# === Feedback Layer (in regular terminal) ===

# 6. Claude Diary
git clone https://github.com/rlancemartin/claude-diary ~/tools/claude-diary
cp ~/tools/claude-diary/commands/*.md ~/.claude/commands/
mkdir -p ~/.claude/hooks
cp ~/tools/claude-diary/hooks/pre-compact.sh ~/.claude/hooks/
chmod +x ~/.claude/hooks/pre-compact.sh
# Add PreCompact hook to ~/.claude/settings.json (see Step 5 above)

# 7. Transcripts
pipx install claude-code-transcripts

# 8. Plunderer
git clone https://github.com/levindixon/claude-code-prompt-plunderer ~/tools/plunderer

# 9. Automation
mkdir -p ~/bin ~/claude-archives
# Copy claude-weekly.sh and claude-reflect-check.sh to ~/bin/
chmod +x ~/bin/claude-*.sh
crontab -e  # Add cron entries

# 10. CLAUDE.md tracking
cd ~/.claude && git init && git add CLAUDE.md && git commit -m "initial"

# 11. Session retention
# Add "cleanupPeriodDays": 90 to ~/.claude/settings.json
```

---

## Verification Checklist

```bash
# Development Layer
/help                                          # Superpowers commands visible
curl http://localhost:37777/api/readiness       # Claude-Mem worker running
claude mcp list                                # Context7 connected
/plugin list                                    # Trail of Bits skills listed

# Feedback Layer
ls ~/.claude/commands/diary.md                  # Diary commands installed
which claude-code-transcripts                   # Transcripts CLI available
python3 ~/tools/plunderer/plunder_prompts.py --help  # Plunderer works

# Automation
crontab -l | grep claude                       # Cron jobs registered
ls ~/bin/claude-weekly.sh ~/bin/claude-reflect-check.sh  # Scripts exist
cd ~/.claude && git log --oneline              # Git tracking active

# Integration test (in Claude Code):
# "Do you have superpowers? What do you remember?
#  Use context7 to look up gin middleware docs.
#  Then review the security of any file using sharp-edges."
```
