# Feedback Loop: Automation & Health Monitoring

The development layer (Superpowers, Claude-Mem, Context7, Trail of Bits) generates enormous amounts of session data but has no built-in mechanism to learn from it. Three feedback-layer tools fill that gap, each operating on a different layer of the data and at a different cadence. This document covers how they work, how to automate them, and how to monitor the health of the entire system.

---

## Part 1: The Three Feedback Tools

### Claude Diary (rlancemartin)

**What it is:** Two slash commands (`/diary` and `/reflect`) plus a PreCompact hook. No dependencies, no servers, no database — just markdown prompt files in `~/.claude/commands/`.

**How `/diary` works:**

1. Fires during a session (manually) or via PreCompact hook (automatically on long sessions before context compression)
2. Claude reflects on everything in context: messages, tool invocations, files modified, errors, solutions, decisions
3. Writes a structured markdown file to `~/.claude/memory/diary/YYYY-MM-DD-session-N.md`
4. Sections: task summary, work done, design decisions, user preferences, code review feedback, challenges, solutions, code patterns

Key detail: Diary uses a context-first approach — reads what's loaded in the session rather than parsing JSONL. The PreCompact hook is critical because it fires before context compression, capturing full session richness right before it would be lost.

**How `/reflect` works:**

1. Run after accumulating several diary entries (weekly is ideal)
2. Reads `processed.log` to skip already-analyzed entries
3. Performs pattern analysis across unprocessed entries
4. Checks for violations of existing CLAUDE.md rules (highest priority — these get strengthened)
5. Synthesizes across 6 categories: PR review feedback, persistent preferences, design decisions, anti-patterns, efficiency lessons, project patterns
6. Patterns appearing 2+ times = pattern, 3+ times = strong pattern
7. Auto-updates `~/.claude/CLAUDE.md` with new one-line bullet rules in imperative tone
8. Saves analysis to `~/.claude/memory/reflections/`
9. Logs processed entries to prevent duplicate analysis

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

### claude-code-transcripts (Simon Willison)

**What it is:** Python CLI converting raw JSONL session files to browsable, paginated HTML. Read-only archive layer — preserves but doesn't analyze or modify.

**How it works:**

1. Claude Code stores sessions as JSONL in `~/.claude/projects/<hash>/<session-id>.jsonl`
2. Renders them as multi-page HTML with navigation, thinking traces, tool calls, timestamps, git context
3. Three modes: `local` (interactive picker), `all` (bulk convert), `json <file>` (specific file)
4. Optional flags: `--gist`, `--json`, `--include-agents`, `--repo OWNER/NAME`

**Output structure:**

```
~/claude-archives/
├── index.html
├── project-foo/
│   ├── index.html
│   ├── session-abc123/
│   │   ├── index.html
│   │   ├── page-2.html
│   │   └── session.jsonl
```

**Critical function:** Preserves sessions before Claude Code's auto-cleanup (default 30 days) deletes them. Safety net ensuring Diary and Plunderer have historical data.

---

### Prompt Plunderer (levindixon)

**What it is:** Standalone Python script, zero dependencies beyond standard library. Extracts and analyzes prompting patterns.

**How it works:**

1. Scans `~/.claude/projects/` for all JSONL files
2. Extracts user messages (not Claude responses)
3. Filters by character length (configurable), deduplicates, tracks frequency
4. Outputs: `extracted_prompts.json` (structured) and `extracted_prompts.md` (human-readable)

**Core value:** The `prompts/slash_command_discovery.md` template analyzes extracted prompts, identifies repetitive patterns, generates ready-to-use slash command files for `~/.claude/commands/`.

---

## Part 2: How They Work Together

```
SESSION DATA FLOW
═════════════════

 ┌─────────────────────────────────────────────────────────┐
 │              YOUR DEVELOPMENT LAYER RUNS                 │
 │  Superpowers → Claude-Mem → Context7 → Trail of Bits    │
 │         (any session type: interactive or autonomous)     │
 └──────────────────────┬──────────────────────────────────┘
                        │ generates
                        ▼
 ┌──────────────────────────────────────────────────────────┐
 │              RAW SESSION DATA                            │
 │  ~/.claude/projects/<hash>/<session-id>.jsonl            │
 │  Every message, tool call, error, decision, token count  │
 │  ⚠️ AUTO-DELETED after 30 days (default)                 │
 └────────┬───────────────────┬───────────────────┬─────────┘
          │                   │                   │
          │ LAYER 1           │ LAYER 2           │ LAYER 3
          │ PRESERVE          │ OBSERVE           │ MINE
          │ (automated)       │ (per-session)     │ (automated)
          ▼                   ▼                   ▼
 ┌────────────────┐  ┌────────────────┐  ┌────────────────┐
 │  TRANSCRIPTS   │  │  CLAUDE DIARY  │  │    PLUNDERER   │
 │  Bulk JSONL    │  │  /diary during │  │  Extract user  │
 │  → HTML before │  │  or after each │  │  prompts, find │
 │  auto-cleanup  │  │  session (auto │  │  patterns &    │
 │  kills raw     │  │  via PreCompact│  │  frequencies   │
 │  data          │  │  hook)         │  │                │
 └────────┬───────┘  └────────┬───────┘  └────────┬───────┘
          │                   │                    │
          │                   │ LAYER 4            │
          │                   │ REFLECT (weekly)   │
          │                   ▼                    │
          │          ┌────────────────┐             │
          │          │   /reflect     │             │
          │          │  Reads diary,  │             │
          │          │  finds 2+      │             │
          │          │  occurrence    │             │
          │          │  patterns,     │             │
          │          │  checks rule   │             │
          │          │  violations    │◄────────────┤
          │          └────────┬───────┘  Plunderer  │
          │                   │         slash cmds  │
          │                   ▼         also go     │
          │          ┌──────────────────here────────┘
          │          │  UPDATED STACK CONFIG
          │          │  ~/.claude/CLAUDE.md
          │          │  ~/.claude/commands/*.md
          │          │  .claude/settings.json
          │          └──────────┬───────────────────┐
          │                     │ feeds back into    │
          │                     ▼                    │
          │          ┌─────────────────────────────┐ │
          │          │   NEXT STACK SESSION         │ │
          │          │   (improved by learnings)    │ │
          │          └─────────────────────────────┘ │
          │                                          │
          └── LAYER 5: HUMAN REVIEW (as-needed) ─────┘
              Browse archived transcripts to:
              • Investigate specific failures
              • Find exact prompt that worked
              • Share sessions for review
              • Audit autonomous runs
```

---

## Part 3: Integration With Each Development Tool

### → Superpowers

**Diary captures:** Whether Claude followed brainstorm → plan → execute → verify, or skipped steps.
**Reflect produces:** Rules like "Always run brainstorm phase before implementation on tasks >50 lines."
**Plunderer reveals:** If you frequently write "just implement X" without planning prompts, surfaces the pattern.
**Transcripts enable:** When verification catches a bug, browse the transcript for the full reasoning chain.

### → Claude-Mem

**Diary captures:** Which observations were injected and whether they were useful.
**Reflect produces:** Rules like "Claude-Mem: reduce injection count for doc-heavy sessions to 5."
**Plunderer reveals:** Repeated prompts like "ignore those old context notes" flag injection quality issues.
**Transcripts enable:** Audit exactly what was injected via MCP tool calls in the HTML archive.

### → Context7

**Diary captures:** Every MCP tool call to Context7, including failures and fallbacks.
**Reflect produces:** Rules like "Always use Context7 for fastapi docs, not training data" or "Pin library ID /tokio-rs/tokio."
**Plunderer reveals:** Manual "check the docs for X" prompts that should be Context7 automation.
**Transcripts enable:** Find every Context7 miss where Claude used stale training data instead of live docs.

### → Trail of Bits

**Diary captures:** When security skills fired and when they didn't but should have.
**Reflect produces:** Rules like "Always trigger differential-review on changes to auth/ or crypto/."
**Plunderer reveals:** Manual "run a security review" prompts that should trigger skills automatically.
**Transcripts enable:** Post-incident forensics — did Trail of Bits skills fire on the session that shipped the bug?

---

## Part 4: Automation Layers

Five layers, each building on the previous. Layer 0 is the baseline. Each subsequent layer adds monitoring, observability, or self-healing.

### Layer 0: Baseline (from Setup)

Already automated after following SETUP.md:

| Component | Mechanism | Trigger |
|-----------|-----------|---------|
| Diary capture (long sessions) | PreCompact hook | Auto on compaction |
| Session archival | `claude-weekly.sh` cron | Sunday 3am |
| Prompt extraction | `claude-weekly.sh` cron | Sunday 3am |
| Old JSONL cleanup | `claude-weekly.sh` cron | Sunday 3am |
| Reflect reminder | `claude-reflect-check.sh` cron | Daily 9am |
| CLAUDE.md versioning | Manual `claude-post-reflect.sh` | After `/reflect` |

Manual steps: `/reflect` weekly, `/diary` on short sessions, `/generate-commands` occasionally.

---

### Layer 1: Close the Manual Gaps

**Auto-diary on session exit (not just compaction)**

The PreCompact hook only fires on long sessions. Short-but-important sessions get nothing. Add a Stop hook that flags sessions above a threshold for diary:

```bash
#!/bin/bash
# ~/.claude/hooks/diary-on-exit.sh

SESSION_FILE=$(ls -t ~/.claude/projects/*/$(cat ~/.claude/last-session-id 2>/dev/null).jsonl 2>/dev/null | head -1)
if [ -n "$SESSION_FILE" ]; then
  MSG_COUNT=$(wc -l < "$SESSION_FILE")
  if [ "$MSG_COUNT" -ge 20 ]; then
    echo "DIARY_NEEDED" > ~/.claude/memory/.diary-pending
  fi
fi
```

```json
// Add to ~/.claude/settings.json
{
  "hooks": {
    "Stop": [{
      "matcher": "auto",
      "hooks": [{
        "type": "command",
        "command": "bash ~/.claude/hooks/diary-on-exit.sh"
      }]
    }]
  }
}
```

At next session start, Claude sees `@~/.claude/memory/.diary-pending` and you decide whether to run `/diary` on the previous session's JSONL.

**Auto-commit CLAUDE.md on change**

Instead of running `claude-post-reflect.sh` manually, use a filesystem watcher:

```bash
# macOS with fswatch
fswatch -o ~/.claude/CLAUDE.md | while read; do
  cd ~/.claude && git add CLAUDE.md && git diff --cached --quiet || \
    git commit -m "auto: $(date +%Y-%m-%d-%H%M)"
done
```

Or as a polling script:

```bash
#!/bin/bash
# ~/bin/claude-md-watcher.sh — run as background process or launchd

WATCHED="$HOME/.claude/CLAUDE.md"
LAST_HASH=$(md5sum "$WATCHED" 2>/dev/null | cut -d' ' -f1)

while true; do
  sleep 60
  CURRENT_HASH=$(md5sum "$WATCHED" 2>/dev/null | cut -d' ' -f1)
  if [ "$CURRENT_HASH" != "$LAST_HASH" ]; then
    cd ~/.claude
    git add CLAUDE.md
    CHANGES=$(git diff --cached --numstat CLAUDE.md | awk '{print $1"+"$2"-"}')
    git commit -m "auto: CLAUDE.md updated ($CHANGES) $(date +%Y-%m-%d-%H%M)"
    LAST_HASH="$CURRENT_HASH"
  fi
done
```

Every CLAUDE.md change is now tracked automatically — from `/reflect`, manual edits, or `/memory` commands.

---

### Layer 2: Observability Pipeline

**Token cost tracking**

Session JSONL contains token usage per message. Extract trends weekly:

```bash
#!/bin/bash
# ~/bin/claude-token-report.sh — add to weekly cron

REPORT="$HOME/claude-archives/token-report-$(date +%Y-%W).csv"
echo "date,project,input_tokens,output_tokens,cache_read,messages" > "$REPORT"

for PROJECT_DIR in ~/.claude/projects/*/; do
  PROJECT=$(basename "$PROJECT_DIR")
  for SESSION in "$PROJECT_DIR"*.jsonl; do
    [ -f "$SESSION" ] || continue
    jq -r 'select(.type == "assistant" and .usage) |
      [.timestamp, "'$PROJECT'", .usage.input_tokens, .usage.output_tokens,
       (.usage.cache_read_input_tokens // 0), 1] | @csv' \
      "$SESSION" 2>/dev/null >> "$REPORT"
  done
done

echo "=== Weekly Token Summary ==="
awk -F',' 'NR>1 {in+=$3; out+=$4; cache+=$5; msg+=$6}
  END {printf "Input: %d | Output: %d | Cache hits: %d | Messages: %d\n", in, out, cache, msg}' "$REPORT"
```

**Note:** The exact JSONL schema varies by Claude Code version. Before deploying, verify field names with `head -20 ~/.claude/projects/*/*.jsonl | jq '.type, .usage'` and adjust `jq` paths accordingly.

**Diary accumulation metrics**

```bash
#!/bin/bash
DIARY_DIR="$HOME/.claude/memory/diary"
PROCESSED="$DIARY_DIR/processed.log"

TOTAL=$(find "$DIARY_DIR" -name "*.md" 2>/dev/null | wc -l)
PROCESSED_COUNT=$(wc -l < "$PROCESSED" 2>/dev/null || echo 0)
UNPROCESSED=$((TOTAL - PROCESSED_COUNT))
THIS_WEEK=$(find "$DIARY_DIR" -name "*.md" -mtime -7 2>/dev/null | wc -l)

echo "Diary: $TOTAL total | $UNPROCESSED unprocessed | $THIS_WEEK this week"
```

**CLAUDE.md growth tracking**

```bash
#!/bin/bash
cd ~/.claude
RULE_COUNT=$(grep -c "^- " CLAUDE.md 2>/dev/null || echo 0)
FILE_SIZE=$(wc -c < CLAUDE.md 2>/dev/null || echo 0)
TOKEN_EST=$((FILE_SIZE / 4))
COMMITS=$(git log --oneline CLAUDE.md 2>/dev/null | wc -l)
LAST_CHANGE=$(git log -1 --format="%ar" CLAUDE.md 2>/dev/null || echo "never")

echo "CLAUDE.md: $RULE_COUNT rules | ~${TOKEN_EST} tokens | $COMMITS revisions | last changed $LAST_CHANGE"
```

This matters because CLAUDE.md competes for context window. If it grows past ~2000 tokens from accumulated reflect rules, it's eating working context. You need to know when to prune.

---

### Layer 3: Health Checks

Things that silently break and you'd never notice:

```bash
#!/bin/bash
# ~/bin/claude-health.sh — run daily via cron, output to log + notify on failure

HEALTH_LOG="$HOME/claude-archives/health.log"
FAILURES=0
WARNINGS=0
TIMESTAMP=$(date '+%Y-%m-%d %H:%M')

check_pass() { echo "[$TIMESTAMP] ✅ $1" >> "$HEALTH_LOG"; }
check_warn() { echo "[$TIMESTAMP] ⚠️  $1" >> "$HEALTH_LOG"; ((WARNINGS++)); }
check_fail() { echo "[$TIMESTAMP] ❌ $1" >> "$HEALTH_LOG"; ((FAILURES++)); }

echo "[$TIMESTAMP] === Health Check ===" >> "$HEALTH_LOG"

# ── DIARY SYSTEM ──

if [ -f ~/.claude/commands/diary.md ] && [ -f ~/.claude/commands/reflect.md ]; then
  check_pass "Diary commands present"
else
  check_fail "Diary commands missing from ~/.claude/commands/"
fi

if grep -q "pre-compact\|PreCompact" ~/.claude/settings.json 2>/dev/null; then
  check_pass "PreCompact hook configured"
else
  check_fail "PreCompact hook not in settings.json — diary won't auto-capture"
fi

RECENT_DIARIES=$(find ~/.claude/memory/diary/ -name "*.md" -mtime -7 2>/dev/null | wc -l)
if [ "$RECENT_DIARIES" -gt 0 ]; then
  check_pass "Diary entries generated this week ($RECENT_DIARIES)"
else
  check_warn "No diary entries in 7 days — hook may not be firing"
fi

TOTAL_DIARIES=$(find ~/.claude/memory/diary/ -name "*.md" 2>/dev/null | wc -l)
PROCESSED=$(wc -l < ~/.claude/memory/diary/processed.log 2>/dev/null || echo 0)
BACKLOG=$((TOTAL_DIARIES - PROCESSED))
if [ "$BACKLOG" -lt 15 ]; then
  check_pass "Reflect backlog manageable ($BACKLOG entries)"
elif [ "$BACKLOG" -lt 30 ]; then
  check_warn "Reflect backlog growing ($BACKLOG entries) — run /reflect"
else
  check_fail "Reflect backlog critical ($BACKLOG entries) — stale learnings"
fi

# ── ARCHIVE SYSTEM ──

if command -v claude-code-transcripts &>/dev/null; then
  check_pass "claude-code-transcripts installed"
else
  check_fail "claude-code-transcripts not found — sessions won't be archived"
fi

THIS_WEEK=$(date +%Y-%W)
if [ -d "$HOME/claude-archives/$THIS_WEEK" ]; then
  check_pass "Weekly archive exists for $THIS_WEEK"
else
  check_warn "No archive for week $THIS_WEEK — cron may have failed"
fi

ARCHIVE_SIZE=$(du -sm ~/claude-archives/ 2>/dev/null | cut -f1)
if [ "${ARCHIVE_SIZE:-0}" -lt 5000 ]; then
  check_pass "Archive size OK (${ARCHIVE_SIZE:-0}MB)"
else
  check_warn "Archive growing large (${ARCHIVE_SIZE}MB) — consider pruning old weeks"
fi

# ── RAW SESSION DATA ──

CLEANUP_DAYS=$(jq -r '.cleanupPeriodDays // 30' ~/.claude/settings.json 2>/dev/null)
if [ "$CLEANUP_DAYS" -ge 60 ]; then
  check_pass "Cleanup retention: ${CLEANUP_DAYS} days"
else
  check_warn "Cleanup retention only ${CLEANUP_DAYS} days — sessions may be lost before archive"
fi

RAW_SIZE=$(du -sm ~/.claude/projects/ 2>/dev/null | cut -f1)
if [ "${RAW_SIZE:-0}" -lt 3000 ]; then
  check_pass "Raw session data: ${RAW_SIZE:-0}MB"
else
  check_warn "Raw session data: ${RAW_SIZE}MB — cleanup may not be running"
fi

# ── CLAUDE.MD HEALTH ──

if [ -f ~/.claude/CLAUDE.md ]; then
  check_pass "~/.claude/CLAUDE.md exists"
else
  check_fail "~/.claude/CLAUDE.md missing"
fi

CLAUDE_MD_SIZE=$(wc -c < ~/.claude/CLAUDE.md 2>/dev/null || echo 0)
CLAUDE_MD_TOKENS=$((CLAUDE_MD_SIZE / 4))
if [ "$CLAUDE_MD_TOKENS" -lt 2000 ]; then
  check_pass "CLAUDE.md size OK (~${CLAUDE_MD_TOKENS} tokens)"
elif [ "$CLAUDE_MD_TOKENS" -lt 4000 ]; then
  check_warn "CLAUDE.md growing (~${CLAUDE_MD_TOKENS} tokens) — review for pruning"
else
  check_fail "CLAUDE.md too large (~${CLAUDE_MD_TOKENS} tokens) — eating context window"
fi

if (cd ~/.claude && git status &>/dev/null); then
  LAST_COMMIT=$(cd ~/.claude && git log -1 --format="%ar" CLAUDE.md 2>/dev/null)
  check_pass "CLAUDE.md git tracking active (last commit: $LAST_COMMIT)"
else
  check_warn "CLAUDE.md not git-tracked — no change history"
fi

RULE_COUNT=$(grep -c "^- " ~/.claude/CLAUDE.md 2>/dev/null || echo 0)
UNIQUE_RULES=$(grep "^- " ~/.claude/CLAUDE.md 2>/dev/null | sort -u | wc -l)
DUPES=$((RULE_COUNT - UNIQUE_RULES))
if [ "$DUPES" -eq 0 ]; then
  check_pass "No duplicate rules in CLAUDE.md ($RULE_COUNT rules)"
else
  check_warn "$DUPES duplicate rules in CLAUDE.md — prune manually"
fi

# ── PLUNDERER ──

if [ -f ~/tools/plunderer/plunder_prompts.py ]; then
  check_pass "Prompt Plunderer installed"
else
  check_warn "Plunderer not found at ~/tools/plunderer/"
fi

# ── CRON HEALTH ──

CRON_WEEKLY=$(crontab -l 2>/dev/null | grep -c "claude-weekly")
CRON_REFLECT=$(crontab -l 2>/dev/null | grep -c "claude-reflect-check")
if [ "$CRON_WEEKLY" -gt 0 ] && [ "$CRON_REFLECT" -gt 0 ]; then
  check_pass "Cron jobs registered (weekly + reflect-check)"
elif [ "$CRON_WEEKLY" -gt 0 ]; then
  check_warn "Weekly cron OK but reflect-check cron missing"
else
  check_fail "Cron jobs not registered — automation not running"
fi

# ── V3 STACK TOOLS ──

check_pass "Superpowers: CLAUDE.md-driven (always active)"

if grep -q "claude-mem" ~/.claude/settings.json 2>/dev/null || \
   grep -q "claude-mem" .claude/settings.json 2>/dev/null; then
  check_pass "Claude-Mem: configured in settings"
else
  check_warn "Claude-Mem: not found in settings.json"
fi

if grep -q "context7" ~/.claude/settings.json 2>/dev/null || \
   grep -q "context7" .claude/settings.json 2>/dev/null; then
  check_pass "Context7: configured in settings"
else
  check_warn "Context7: not found in settings.json"
fi

if grep -q "trail.of.bits\|trailofbits" ~/.claude/settings.json 2>/dev/null || \
   grep -q "trail.of.bits\|trailofbits" .claude/settings.json 2>/dev/null; then
  check_pass "Trail of Bits: configured in settings"
else
  check_warn "Trail of Bits: not found in settings.json"
fi

# ── SUMMARY ──

echo "" >> "$HEALTH_LOG"
echo "[$TIMESTAMP] Summary: $FAILURES failures, $WARNINGS warnings" >> "$HEALTH_LOG"
echo "" >> "$HEALTH_LOG"

tail -30 "$HEALTH_LOG"

# Notify on failures
if [ "$FAILURES" -gt 0 ]; then
  osascript -e "display notification \"$FAILURES failures in Claude stack\" \
    with title \"⚠️ Claude Health Check\"" 2>/dev/null

  echo "⚠️ Stack health: $FAILURES failures. Run \`bash ~/bin/claude-health.sh\` for details." \
    > ~/.claude/memory/health-alert.md
fi
```

Register the daily health check:

```cron
0 10 * * * ~/bin/claude-health.sh
```

Make Claude aware of health alerts at session start — add to `~/.claude/CLAUDE.md`:

```markdown
@~/.claude/memory/health-alert.md
@~/.claude/memory/reflect-reminder.md
```

---

### Layer 4: Dashboard (Single-Glance Status)

```bash
#!/bin/bash
# ~/bin/claude-status.sh — run anytime for quick status

echo "╔══════════════════════════════════════════════╗"
echo "║          CLAUDE CODE STACK STATUS            ║"
echo "╚══════════════════════════════════════════════╝"
echo ""

# ── Sessions ──
RAW_SIZE=$(du -sh ~/.claude/projects/ 2>/dev/null | cut -f1)
SESSION_COUNT=$(find ~/.claude/projects/ -name "*.jsonl" 2>/dev/null | wc -l)
CLEANUP=$(jq -r '.cleanupPeriodDays // 30' ~/.claude/settings.json 2>/dev/null)
echo "📁 Sessions:  $SESSION_COUNT files | ${RAW_SIZE:-0} | cleanup: ${CLEANUP}d"

# ── Archives ──
ARCHIVE_SIZE=$(du -sh ~/claude-archives/ 2>/dev/null | cut -f1)
ARCHIVE_WEEKS=$(find ~/claude-archives/ -maxdepth 1 -type d | wc -l)
LAST_ARCHIVE=$(ls -td ~/claude-archives/20*/ 2>/dev/null | head -1 | xargs basename 2>/dev/null)
echo "📦 Archives:  ${ARCHIVE_SIZE:-0} | ${ARCHIVE_WEEKS} weeks | latest: ${LAST_ARCHIVE:-none}"

# ── Diary ──
DIARY_TOTAL=$(find ~/.claude/memory/diary/ -name "*.md" 2>/dev/null | wc -l)
DIARY_PROCESSED=$(wc -l < ~/.claude/memory/diary/processed.log 2>/dev/null || echo 0)
DIARY_PENDING=$((DIARY_TOTAL - DIARY_PROCESSED))
DIARY_WEEK=$(find ~/.claude/memory/diary/ -name "*.md" -mtime -7 2>/dev/null | wc -l)
if [ "$DIARY_PENDING" -ge 15 ]; then STATUS="🔴"; elif [ "$DIARY_PENDING" -ge 8 ]; then STATUS="🟡"; else STATUS="🟢"; fi
echo "📓 Diary:     $DIARY_TOTAL total | $DIARY_PENDING pending | $DIARY_WEEK this week $STATUS"

# ── Reflections ──
REFLECT_COUNT=$(find ~/.claude/memory/reflections/ -name "*.md" 2>/dev/null | wc -l)
LAST_REFLECT=$(ls -t ~/.claude/memory/reflections/*.md 2>/dev/null | head -1 | xargs basename 2>/dev/null)
echo "🔍 Reflects:  $REFLECT_COUNT total | latest: ${LAST_REFLECT:-never}"

# ── CLAUDE.md ──
RULE_COUNT=$(grep -c "^- " ~/.claude/CLAUDE.md 2>/dev/null || echo 0)
CLAUDE_BYTES=$(wc -c < ~/.claude/CLAUDE.md 2>/dev/null || echo 0)
CLAUDE_TOKENS=$((CLAUDE_BYTES / 4))
GIT_COMMITS=$(cd ~/.claude && git log --oneline CLAUDE.md 2>/dev/null | wc -l)
LAST_EDIT=$(cd ~/.claude && git log -1 --format="%ar" CLAUDE.md 2>/dev/null || echo "untracked")
if [ "$CLAUDE_TOKENS" -ge 4000 ]; then STATUS="🔴"; elif [ "$CLAUDE_TOKENS" -ge 2000 ]; then STATUS="🟡"; else STATUS="🟢"; fi
echo "📜 CLAUDE.md: $RULE_COUNT rules | ~${CLAUDE_TOKENS} tokens | $GIT_COMMITS commits | $LAST_EDIT $STATUS"

# ── Slash Commands ──
CMD_COUNT=$(find ~/.claude/commands/ -name "*.md" 2>/dev/null | wc -l)
echo "⚡ Commands:  $CMD_COUNT total"

# ── Cron ──
CRON_OK=true
crontab -l 2>/dev/null | grep -q "claude-weekly" || CRON_OK=false
crontab -l 2>/dev/null | grep -q "claude-reflect-check" || CRON_OK=false
crontab -l 2>/dev/null | grep -q "claude-health" || CRON_OK=false
if $CRON_OK; then echo "⏰ Cron:      all jobs registered 🟢"; else echo "⏰ Cron:      missing jobs 🔴"; fi

# ── Stack Tools ──
echo ""
echo "── Development Layer ──"
echo -n "  Superpowers:    "; [ -f ~/.claude/CLAUDE.md ] && echo "✅ active" || echo "❌ no CLAUDE.md"
echo -n "  Claude-Mem:     "; grep -q "claude-mem" ~/.claude/settings.json 2>/dev/null && echo "✅ configured" || echo "⚠️  not in settings"
echo -n "  Context7:       "; grep -q "context7" ~/.claude/settings.json 2>/dev/null && echo "✅ configured" || echo "⚠️  not in settings"
echo -n "  Trail of Bits:  "; grep -q "trail.of.bits\|trailofbits" ~/.claude/settings.json 2>/dev/null && echo "✅ configured" || echo "⚠️  not in settings"

echo ""
echo "── Feedback Layer ──"
echo -n "  Diary:          "; [ -f ~/.claude/commands/diary.md ] && echo "✅ installed" || echo "❌ missing"
echo -n "  Transcripts:    "; command -v claude-code-transcripts &>/dev/null && echo "✅ installed" || echo "❌ missing"
echo -n "  Plunderer:      "; [ -f ~/tools/plunderer/plunder_prompts.py ] && echo "✅ installed" || echo "❌ missing"

# ── Last Health Check ──
echo ""
LAST_HEALTH=$(tail -1 ~/claude-archives/health.log 2>/dev/null)
echo "🏥 Last health: ${LAST_HEALTH:-never run}"
```

**Sample output:**

```
╔══════════════════════════════════════════════╗
║          CLAUDE CODE STACK STATUS            ║
╚══════════════════════════════════════════════╝

📁 Sessions:  47 files | 312M | cleanup: 90d
📦 Archives:  1.2G | 8 weeks | latest: 2026-06
📓 Diary:     23 total | 6 pending | 4 this week 🟢
🔍 Reflects:  3 total | latest: 2026-02-reflection-3.md
📜 CLAUDE.md: 34 rules | ~850 tokens | 5 commits | 3 days ago 🟢
⚡ Commands:  12 total
⏰ Cron:      all jobs registered 🟢

── Development Layer ──
  Superpowers:    ✅ active
  Claude-Mem:     ✅ configured
  Context7:       ✅ configured
  Trail of Bits:  ✅ configured

── Feedback Layer ──
  Diary:          ✅ installed
  Transcripts:    ✅ installed
  Plunderer:      ✅ installed

🏥 Last health: [2026-02-08 10:00] Summary: 0 failures, 0 warnings
```

---

## Part 5: Alert Reference

### 🔴 Critical (Something Is Broken)

| Alert | Cause | Fix |
|-------|-------|-----|
| Diary commands missing | Files deleted or never installed | `cp ~/tools/claude-diary/commands/*.md ~/.claude/commands/` |
| PreCompact hook not configured | settings.json missing hook entry | Add hook block to `~/.claude/settings.json` |
| claude-code-transcripts missing | pipx uninstalled or PATH issue | `pipx install claude-code-transcripts` |
| CLAUDE.md missing | Deleted or never created | `claude /init` or create manually |
| CLAUDE.md too large (>4000 tokens) | Reflect adding without pruning | Review and consolidate rules, remove stale entries |
| Cron jobs missing | crontab cleared or never set | Re-add entries from SETUP.md |
| Reflect backlog critical (>30) | Haven't run `/reflect` in weeks | Run `/reflect` immediately |

### 🟡 Warning (Degraded, Not Broken)

| Alert | Cause | Fix |
|-------|-------|-----|
| No diary entries in 7 days | Not using Claude Code, or hook broken | Check if you've had compactable sessions; verify hook |
| Reflect backlog growing (15–30) | Normal if busy, concerning if chronic | Schedule `/reflect` this week |
| No archive this week | Cron failed or machine was off | Run `~/bin/claude-weekly.sh` manually |
| Archive growing large (>5GB) | Long history, no pruning | Delete archives older than N months |
| CLAUDE.md growing (2000–4000 tokens) | Normal accumulation | Review during next `/reflect` for consolidation |
| Duplicate rules | Reflect added similar rules across runs | Manual dedup during review |
| Cleanup retention too low (<60d) | Sessions deleted before archive | Increase `cleanupPeriodDays` in settings.json |
| Raw data large (>3GB) | Cleanup not running or heavy usage | Verify `cleanupPeriodDays`; run manual cleanup after archive |
| Stack tool not in settings | MCP not configured for this context | Check project-level vs global config |

---

## Part 6: What Each Tool Cannot Do

| Capability | Diary | Transcripts | Plunderer |
|-----------|-------|-------------|-----------|
| Capture in-session context | ✅ | ❌ | ❌ |
| Preserve full session history | ❌ | ✅ | ❌ |
| Auto-fire during long sessions | ✅ (hook) | ❌ (cron) | ❌ (cron) |
| Pattern detection across sessions | ✅ (/reflect) | ❌ | ✅ (frequency) |
| Update CLAUDE.md automatically | ✅ | ❌ | ❌ |
| Generate new slash commands | ❌ | ❌ | ✅ |
| Human-browsable archives | ❌ | ✅ | ❌ |
| Work after sessions are deleted | ❌ | ✅ (archived) | ❌ |
| Analyze prompting style/evolution | ❌ | ❌ | ✅ |
| Post-incident forensics | ❌ | ✅ | ❌ |

**Diary** captures *what happened* and *why* → feeds into CLAUDE.md rules.
**Transcripts** preserves *everything* before deletion → safety net and audit trail.
**Plunderer** reveals *how you work* → automates repetitive patterns into slash commands.

Without Diary: your stack never learns from its own sessions.
Without Transcripts: sessions vanish after 30–90 days and you lose the ability to investigate.
Without Plunderer: you keep manually typing prompts that should be one-keystroke commands.

---

## Part 7: Automation Summary

```
AUTOMATION LAYERS
═════════════════

Layer 0: Baseline
├── PreCompact hook → diary auto-capture
├── Weekly cron → archive + extract + cleanup
├── Daily cron → reflect reminder
└── Manual: /reflect, /generate-commands

Layer 1: Close gaps
├── Stop hook → flag sessions needing diary
├── fswatch → auto-commit CLAUDE.md changes
└── Removes: manual post-reflect git commit

Layer 2: Observability
├── Token cost extraction → weekly CSV
├── Diary accumulation metrics
└── CLAUDE.md growth tracking

Layer 3: Health checks
├── Daily health check script → log + alerts
├── Claude reads alert files at session start
├── Covers: hooks, cron, disk, tools, config
└── macOS notifications on failures

Layer 4: Dashboard
├── Single-command status view (claude-status)
├── Traffic-light indicators (🟢🟡🔴)
└── All metrics in one glance

MANUAL TOUCHPOINTS:
1. /reflect (weekly, ~10 min) — intentionally manual
2. claude-status (as-needed, instant)
3. /generate-commands (occasional, ~2 min)
```
