# Setup & Configuration

Get from zero to running. Install, configure, verify, and maintain the Claude Code Toolkit.

---

## Prerequisites

```bash
# Node.js v18+
node --version

# Bun (for Claude-Mem worker)
brew install oven-sh/bun/bun

# Python 3.8+
python3 --version

# Claude Code CLI
claude --version
```

---

## Development Layer (in Claude Code terminal)

```bash
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
```

## Feedback Layer (terminal)

```bash
# 5. Claude Diary
git clone https://github.com/rlancemartin/claude-diary ~/tools/claude-diary
cp ~/tools/claude-diary/commands/*.md ~/.claude/commands/
mkdir -p ~/.claude/hooks
cp ~/tools/claude-diary/hooks/pre-compact.sh ~/.claude/hooks/
chmod +x ~/.claude/hooks/pre-compact.sh
# Add PreCompact hook to ~/.claude/settings.json

# 6. Transcripts
pipx install claude-code-transcripts

# 7. Plunderer
git clone https://github.com/levindixon/claude-code-prompt-plunderer ~/tools/plunderer
```

---

## Configure Automation

```bash
mkdir -p ~/bin ~/claude-archives

# Add to ~/.claude/settings.json (PreCompact hook):
# {
#   "hooks": {
#     "PreCompact": [{
#       "matcher": "auto",
#       "hooks": [{
#         "type": "command",
#         "command": "bash ~/.claude/hooks/pre-compact.sh"
#       }]
#     }]
#   },
#   "cleanupPeriodDays": 90
# }

# Weekly automation script
cat > ~/bin/claude-weekly.sh << 'EOF'
#!/bin/bash
WEEK=$(date +%Y-%W)
ARCHIVE_DIR="$HOME/claude-archives/$WEEK"
mkdir -p "$ARCHIVE_DIR"
claude-code-transcripts all -o "$ARCHIVE_DIR" --include-agents --json
python3 ~/tools/plunderer/plunder_prompts.py --min-length 30 --max-length 2000 --output "$ARCHIVE_DIR/prompts.json"
find ~/.claude/projects/ -name "*.jsonl" -mtime +90 -delete
find ~/.claude/file-history/ -mtime +90 -delete
echo "$(date): archived week $WEEK" >> ~/claude-archives/archive.log
EOF

chmod +x ~/bin/claude-weekly.sh

# Add to crontab:
# 0 3 * * 0 ~/bin/claude-weekly.sh
# 0 9 * * * ~/bin/claude-reflect-check.sh
```

---

## Feedback Loop: Learning Between Sessions

The feedback layer captures session data and extracts learnings to improve the stack.

### How It Works

```
SESSION DATA FLOW
─────────────────

Your Development Layer Runs
    ↓ generates
Raw Session Data (JSONL)
    ↓ flows to
┌─────────────────────────────────────────────────────────┐
│ LAYER 1: PRESERVE (automated)                           │
│ claude-code-transcripts: JSONL → HTML before auto-delete│
│                                                         │
│ LAYER 2: OBSERVE (per-session)                         │
│ Claude Diary: /diary captures observations             │
│                                                         │
│ LAYER 3: MINE (automated)                              │
│ Prompt Plunderer: Extract user prompts, find patterns  │
│                                                         │
│ LAYER 4: REFLECT (weekly manual)                        │
│ /reflect: Read diary, find 2+ occurrence patterns,     │
│           update CLAUDE.md                              │
└─────────────────────────────────────────────────────────┘
    ↓ updates
CLAUDE.md + Slash Commands
    ↓ feeds into
Next Development Layer Session (improved)
```

### The Three Tools

**Claude Diary:**
- `/diary` — Fires during/after session, captures observations to `~/.claude/memory/diary/YYYY-MM-DD-session-N.md`
- `/reflect` — Run weekly, reads diary entries, finds patterns (2+ occurrences = pattern), auto-updates CLAUDE.md
- PreCompact hook — Auto-fires `/diary` before context compression on long sessions

**claude-code-transcripts:**
- Converts JSONL to browsable HTML with navigation, thinking traces, timestamps
- Modes: `local` (interactive), `all` (bulk), `json <file>` (specific)
- Preserves sessions before Claude Code's auto-cleanup (default 30 days)

**Prompt Plunderer:**
- Extracts user prompts from all JSONL files
- Filters by length, deduplicates, tracks frequency
- Outputs: `extracted_prompts.json` (structured) and readable markdown
- `prompts/slash_command_discovery.md` template generates ready-to-use slash commands

### Integration with Each Tool

| Tool | Diary Captures | Reflect Produces | Plunderer Reveals | Transcripts Enable |
|------|---|---|---|---|
| **Superpowers** | Skipped phases? | "Always brainstorm >50 lines" | "just implement X" patterns | Full reasoning chain |
| **Claude-Mem** | Useful injections? | "Reduce injection for doc-heavy" | "ignore old context" flags | View exact injections |
| **Context7** | MCP tool calls? | "Pin library ID /tokio" | Manual "check docs" prompts | Find Context7 misses |
| **Trail of Bits** | Skills fired? | "Always trigger differential on auth/" | Manual "run review" prompts | Security incident forensics |

---

## Automation & Health Monitoring

### Layer 0: Baseline (from Setup)

Already automated after install:
- Diary capture on long sessions (PreCompact hook)
- Weekly session archival (cron)
- Prompt extraction (cron)
- Old JSONL cleanup (cron)
- Reflect reminder (cron)

Manual steps: `/reflect` weekly, `/diary` on short sessions.

### Layer 1: Close Manual Gaps

Auto-diary on session exit + auto-commit CLAUDE.md on change.

### Layer 2: Observability

Extract token cost trends, diary accumulation metrics, CLAUDE.md growth tracking.

### Layer 3: Health Checks

Daily script checking: diary system, archives, CLAUDE.md health, tools configured, cron jobs registered. Failures trigger notifications.

### Layer 4: Dashboard

Single command (`claude-status`) shows all metrics at a glance with traffic-light indicators.

### Status Command

```bash
#!/bin/bash
# ~/bin/claude-status.sh

echo "╔══════════════════════════════════════════════╗"
echo "║          CLAUDE CODE STACK STATUS            ║"
echo "╚══════════════════════════════════════════════╝"
echo ""

# Sessions
RAW_SIZE=$(du -sh ~/.claude/projects/ 2>/dev/null | cut -f1)
SESSION_COUNT=$(find ~/.claude/projects/ -name "*.jsonl" 2>/dev/null | wc -l)
CLEANUP=$(jq -r '.cleanupPeriodDays // 30' ~/.claude/settings.json 2>/dev/null)
echo "📁 Sessions:  $SESSION_COUNT files | ${RAW_SIZE:-0} | cleanup: ${CLEANUP}d"

# Archives
ARCHIVE_SIZE=$(du -sh ~/claude-archives/ 2>/dev/null | cut -f1)
ARCHIVE_WEEKS=$(find ~/claude-archives/ -maxdepth 1 -type d | wc -l)
echo "📦 Archives:  ${ARCHIVE_SIZE:-0} | ${ARCHIVE_WEEKS} weeks"

# Diary
DIARY_TOTAL=$(find ~/.claude/memory/diary/ -name "*.md" 2>/dev/null | wc -l)
DIARY_PROCESSED=$(wc -l < ~/.claude/memory/diary/processed.log 2>/dev/null || echo 0)
DIARY_PENDING=$((DIARY_TOTAL - DIARY_PROCESSED))
if [ "$DIARY_PENDING" -ge 15 ]; then STATUS="🔴"; elif [ "$DIARY_PENDING" -ge 8 ]; then STATUS="🟡"; else STATUS="🟢"; fi
echo "📓 Diary:     $DIARY_TOTAL total | $DIARY_PENDING pending $STATUS"

# CLAUDE.md
RULE_COUNT=$(grep -c "^- " ~/.claude/CLAUDE.md 2>/dev/null || echo 0)
CLAUDE_BYTES=$(wc -c < ~/.claude/CLAUDE.md 2>/dev/null || echo 0)
CLAUDE_TOKENS=$((CLAUDE_BYTES / 4))
if [ "$CLAUDE_TOKENS" -ge 4000 ]; then STATUS="🔴"; elif [ "$CLAUDE_TOKENS" -ge 2000 ]; then STATUS="🟡"; else STATUS="🟢"; fi
echo "📜 CLAUDE.md: $RULE_COUNT rules | ~${CLAUDE_TOKENS} tokens $STATUS"

# Stack Tools
echo ""
echo "── Development Layer ──"
echo -n "  Superpowers:    "; [ -f ~/.claude/CLAUDE.md ] && echo "✅" || echo "❌"
echo -n "  Claude-Mem:     "; grep -q "claude-mem" ~/.claude/settings.json 2>/dev/null && echo "✅" || echo "⚠️"
echo -n "  Context7:       "; grep -q "context7" ~/.claude/settings.json 2>/dev/null && echo "✅" || echo "⚠️"
echo -n "  Trail of Bits:  "; grep -q "trail.of.bits\|trailofbits" ~/.claude/settings.json 2>/dev/null && echo "✅" || echo "⚠️"

echo ""
echo "── Feedback Layer ──"
echo -n "  Diary:          "; [ -f ~/.claude/commands/diary.md ] && echo "✅" || echo "❌"
echo -n "  Transcripts:    "; command -v claude-code-transcripts &>/dev/null && echo "✅" || echo "❌"
echo -n "  Plunderer:      "; [ -f ~/tools/plunderer/plunder_prompts.py ] && echo "✅" || echo "❌"

# Cron
echo ""
CRON_OK=true
crontab -l 2>/dev/null | grep -q "claude-weekly" || CRON_OK=false
if $CRON_OK; then echo "⏰ Cron:      all jobs 🟢"; else echo "⏰ Cron:      missing 🔴"; fi
```

---

## Daily Workflow

```
Morning startup:
  $ claude-status              # Check stack health
  $ claude                     # Start session
  > What do you remember?      # Check Claude-Mem context
  > Let's brainstorm [feature] # Superpowers drives design

Implementation:
  > Use Superpowers execution
  > Context7 auto-fetches docs
  > Trail of Bits fires on sensitive code
  > Claude-Mem captures observations

Autonomous multi-component work:
  $ claude code ~/project
  > Spawn N agents: [design]
  > [Agents run autonomously]
  > [Go do something else]

Security review:
  > Review last commit with differential-review
  > Run sharp-edges on internal/auth/

End of day:
  > Remember: we decided [X] because [Y]
  > Next session: start with [Z]

Weekly:
  $ /reflect                    # Review diary, update CLAUDE.md (~10 min)
```

---

## Token Budget

| Component | Session Start Cost | Runtime Cost (per use) |
|-----------|-------------------|----------------------|
| Superpowers bootstrap | ~500–1,000 tokens | Skills loaded on-demand |
| Claude-Mem injection | ~2,250 tokens (configurable) | Async via worker |
| Context7 MCP | **0 tokens** (lazy-loads) | ~500–2,000 per doc query |
| Trail of Bits skills | **0 tokens** (on-demand) | ~300–500 per skill activation |
| **Total at session start** | **~2,750–3,250 tokens** | — |

This is ~1.5% of the 200k context window.

---

## Potential Conflicts & Mitigations

**Hook ordering:** Superpowers and Claude-Mem both register SessionStart hooks. Claude Code merges them additively — both fire independently.

**Context7 + Superpowers:** No conflict. Context7 is MCP-based, Superpowers is skill-based. They compose naturally.

**Trail of Bits + Superpowers:** Complementary. Superpowers provides methodology; Trail of Bits provides security analysis within that workflow.

**Bun dependency:** Claude-Mem requires Bun. Context7 requires Node.js. Both likely already installed.

**VSCode extension caveat:** Claude-Mem hooks may not trigger in VSCode extension (known issue). Works reliably with CLI version.

---

## Troubleshooting

### Claude-Mem Not Shared Across Agents

**Symptom:** Agent 2 doesn't see observations from Agent 1.

**Fixes:**
1. Check: `curl http://localhost:37777/api/readiness`
2. Verify agents are in same parent session (not separate CLI calls)
3. Verify `settings.json` has claude-mem configured

### Trail of Bits Not Activating

**Symptom:** Asked agents to use sharp-edges but they don't.

**Fixes:**
1. Check: `/plugin list` (reinstall if missing)
2. Explicitly ask: "Run your sharp-edges skill on [file]"
3. Ensure code matches skill's trigger conditions (e.g., error handling for sharp-edges)

### Debug Stuck Agent Teams

Debug with:
```bash
curl http://localhost:37777/api/readiness
sqlite3 ~/.claude-mem/claude-mem.db "PRAGMA integrity_check;"
sqlite3 ~/.claude-mem/claude-mem.db "SELECT created_at, summary FROM observations ORDER BY created_at DESC LIMIT 5;"
```

### Reflect Backlog Growing

When diary entries accumulate without `/reflect` running, alerts trigger. Run `/reflect` to analyze patterns and update CLAUDE.md. Then system learns and improves.

---

## Statusline Configuration

See [STATUSLINE-CONFIG.md](STATUSLINE-CONFIG.md) for Claude Code statusline setup.

---

## Next Step

Once installed, start the [LEARNING-PLAN.md](LEARNING-PLAN.md) at Level 1 to verify everything works.

## All Docs

- [README.md](README.md) — Toolkit overview and architecture
- [LEARNING-PLAN.md](LEARNING-PLAN.md) — 11-level mastery path
- [AGENT-TEAMS.md](AGENT-TEAMS.md) — Multi-agent orchestration guide
- [STATUSLINE-CONFIG.md](STATUSLINE-CONFIG.md) — Claude Code statusline setup
