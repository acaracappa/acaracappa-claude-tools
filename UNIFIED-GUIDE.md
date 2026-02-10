# Claude Code Toolkit: Complete Guide

A curated, integrated toolkit that transforms Claude Code from a reactive coding assistant into a disciplined, memory-aware, security-conscious engineering system with a self-improving feedback loop.

---

## Architecture Overview

The toolkit operates in two layers: a development layer (active during sessions) and a feedback layer (running between sessions).

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           YOUR PROJECT REPO                             │
│                                                                         │
│  DEVELOPMENT LAYER (active during sessions)                             │
│  ┌──────────────┐ ┌──────────────┐ ┌────────────┐ ┌───────────┐        │
│  │ SUPERPOWERS  │ │ CLAUDE-MEM   │ │ CONTEXT7   │ │TRAIL OF   │        │
│  │ Skills &     │ │ Memory &     │ │ Live Docs  │ │ BITS      │        │
│  │ Workflow     │ │ Context      │ │            │ │ Security  │        │
│  └──────┬───────┘ └──────┬───────┘ └─────┬──────┘ └─────┬─────┘        │
│         │                │               │              │              │
│         └────────────────┼───────────────┼──────────────┘              │
│                          ▼               ▼                              │
│          ┌─────────────────────────────────────────┐                   │
│          │      CLAUDE CODE CLI / SDK              │                   │
│          │  (hooks, skills, MCP, security)         │                   │
│          └─────────────────────────────────────────┘                   │
│                                                                         │
│  FEEDBACK LAYER (runs between sessions)                                 │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐                   │
│  │ TRANSCRIPTS  │ │ CLAUDE DIARY │ │  PLUNDERER   │                   │
│  │ Preserve     │ │ Observe      │ │ Mine         │                   │
│  └──────────────┘ └──────────────┘ └──────────────┘                   │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Tool Summary

### Development Layer (4 tools)

| Tool | Concern | What It Does |
|------|---------|--------------|
| **Superpowers** | *How* you think and build | Enforces brainstorm → plan → execute → verify methodology. TDD, subagents, worktrees, systematic debugging. |
| **Claude-Mem** | *What* you remember | Persistent cross-session memory via SQLite + vector embeddings. Privacy controls, configurable injection. |
| **Context7** | *What* you know | Fetches current, version-specific documentation from source repos at query time. Eliminates hallucinated APIs. |
| **Trail of Bits** | *How safe* output is | Professional security skills: sharp-edges, differential-review, static-analysis, insecure-defaults, fix-review. |

### Feedback Layer (3 tools)

| Tool | Concern | What It Does |
|------|---------|--------------|
| **Claude Diary** | Observation + Reflection | Captures session observations, reflects across diary entries, auto-updates CLAUDE.md with learned rules. |
| **claude-code-transcripts** | Archival | Converts raw JSONL session files to browsable HTML archives. Safety net for historical data. |
| **Prompt Plunderer** | Pattern Mining | Extracts and analyzes prompting patterns, identifies repetitive workflows, generates slash commands. |

---

## Quick Install

### Prerequisites

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

### Development Layer (in Claude Code terminal)

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

### Feedback Layer (terminal)

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

### Configure Automation

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

## Learning Path: 11 Levels to Mastery

### Level 1: Installation & Smoke Test
**Goal:** All tools installed and not conflicting.

Verify: `/help` shows Superpowers, `curl http://localhost:37777/api/readiness` works, `claude mcp list` shows Context7, `/plugin list` shows Trail of Bits.

### Level 2: Superpowers — Following the Workflow
**Goal:** Complete one feature using brainstorm → plan → execute → verify end-to-end.

Don't skip any steps. Let Superpowers drive the methodology. Verify tests written in TDD style (tests before implementation).

### Level 3: Claude-Mem — Building Memory
**Goal:** Develop intuition for what Claude-Mem captures and how to query it.

Work 3–4 sessions across 2+ days. Observe what gets injected. Test `<private>` tags. Search past work with natural language. Adjust `maxTokens` setting (default 2250 usually right).

### Level 4: Context7 — Documentation on Demand
**Goal:** Make Context7 a reflex for any library interaction.

Practice three patterns: general mention, direct library ID, version-specific. Use the subagent pattern for long sessions. Build your library ID cheat sheet.

### Level 5: Trail of Bits — Security-First Development
**Goal:** Understand what Trail of Bits catches and integrate into workflow.

Trigger sharp-edges on sensitive code. Run insecure-defaults scans. Use differential-review on commits. Compare what Superpowers catches (structure) vs. Trail of Bits (security).

### Level 6: Agent Teams — First Autonomous Run
**Goal:** Successfully spawn and complete one multi-agent autonomous session.

Start small: 2–3 semi-independent components. Define dependencies. Watch agents respect ordering and use the stack tools (Context7, Trail of Bits). Verify tests pass.

### Level 7: Prompt Engineering for the Stack
**Goal:** Write prompts and CLAUDE.md that get maximum leverage from all tools.

Write project CLAUDE.md with methodology rules. Optimize agent team spawning prompts with clear success criteria. Learn Context7 patterns that work. Shape Claude-Mem deliberately at session end.

### Level 8: Multi-Component Orchestration
**Goal:** Spawn and coordinate parallel agent teams while maintaining coherence.

Pick real feature spanning 3+ components. Decompose into agent team structure with clear DAG. Spawn agents respecting dependencies. Verify agents use each other's output correctly. Run e2e tests.

### Level 9: Failure Recovery & Debugging
**Goal:** Diagnose and recover when the stack goes wrong.

Debug stuck agents, Claude-Mem worker issues, Context7 failures, Superpowers skill refusal, Trail of Bits not activating, context exhaustion. Know recovery playbook for each.

### Level 10: Custom Extensions & Optimization
**Goal:** Extend the stack with custom hooks, skills, configurations.

Write custom hooks and skills. Optimize token budget. Create pre-flight check scripts. Document agent team heuristics in CLAUDE.md.

### Level 11: Mastery — System Thinking
**Goal:** Operate the full stack as unified system. Complete multi-component feature end-to-end in one day.

**Morning (2h):** Design with Superpowers. **Midday (3–4h):** Spawn autonomous agents. **Afternoon (2h):** Security review + integration. **End of day:** Summarize for Claude-Mem. Feature deployed.

**Mastery indicators:**
- Install stack from memory in 15 minutes
- Explain each tool's architecture and failure modes
- Design using full Superpowers methodology
- Secure code using Trail of Bits integrated into workflow
- Spawn agent teams across 3+ components with coherent results
- Debug any tool failure in under 2 minutes
- Extend the stack with custom hooks and skills
- Optimize based on measured session profiles
- Teach someone else Level 0→6 in a week
- Ship 3–5x faster than without the stack
- Know when NOT to use tools

---

## Agent Teams: Multi-Agent Orchestration

Agent teams are Claude's native multi-agent orchestration system allowing specialized agents to collaborate in parallel within a single Claude session.

### Core Architecture

```
YOUR SESSION (interactive or autonomous)
│
├─ Superpowers: Brainstorm → Plan (you or lead agent designs)
│
└─ Agent Spawning Decision
   │
   ├─ SIMPLE TASK (single skill, <2 hours)
   │  └─→ Execute locally
   │
   └─ COMPLEX TASK (multi-domain, >2 hours)
      └─→ Spawn Agent Team
         │
         ├─ Agent 1: API Layer (no dependencies, start immediately)
         │  • Context7 for API docs
         │  • Trail of Bits security checks
         │  • Superpowers TDD
         │  • Reports: "API interface complete"
         │
         ├─ Agent 2: Core Engine (depends on Agent 1)
         │  • Waits for Agent 1
         │  • Uses Agent 1's interface
         │  • Context7 + Trail of Bits + TDD
         │  • Reports: "Core logic complete, tests passing"
         │
         └─ Agent 3: Review (parallel to others)
            • differential-review + insecure-defaults
            • Reports: "No security regressions"

PARENT SESSION
├─ Verify: All tests pass?
├─ Review: Blast radius acceptable?
├─ Integrate: Merge branches
└─ Commit: Record in git + diary
```

### Integration with Your Stack

**Superpowers:** Methodology applies to each agent (TDD, verification).

**Claude-Mem:** Agents share context, avoid redundant work. Session start injects: "API uses gin, Engine uses tokio, ML uses scikit-learn."

**Context7:** Each agent gets current library docs. Agent 1 uses gin docs, Agent 2 uses tokio docs, Agent 3 uses sklearn docs.

**Trail of Bits:** Security checks run in parallel with implementation. Issues caught early, not after-the-fact.

**Feedback Loop:** Captures when spawning was effective, time saved, decisions made → /reflect identifies patterns → CLAUDE.md updates → next session benefits.

### When to Spawn Agents

**✓ DO spawn when:**
- Feature spans 2–4 semi-independent components
- Components have clear interfaces/contracts
- Total work >3 hours
- Work can be parallelized
- Well-defined success criteria
- Security/quality review needed

**✗ DON'T spawn when:**
- Feature is single-component (<3 hours)
- Components are tightly coupled
- Still exploring design
- High-risk code (payment, auth, crypto) needing constant oversight
- Vague success criteria
- Constant real-time feedback needed

### Agent Team Prompt Template

```
I'm building [feature name] spanning [N] components/domains.

This feature breaks into semi-independent components:
- [Component 1]: [brief description]
- [Component 2]: [brief description]
- [Component 3]: [brief description]

Component dependencies:
- [Component 2] depends on [Component 1]
- [Component 3] depends on [Component 2]

Spawn [N] agents:

AGENT 1 ([Component Name]):
- Responsibility: [what this component does]
- Work: [specific tasks]
- Success criteria: [concrete: tests pass, API spec, etc.]
- No dependencies (can start immediately)
- Use Context7 for: [library]
- Use Trail of Bits for: [security concern]

AGENT 2 ([Component Name]):
- Responsibility: [what this component does]
- Work: [specific tasks]
- Success criteria: [concrete: tests pass, validates Agent 1, etc.]
- Depends on: Agent 1 (wait for [specific output])
- Use Context7 for: [library]
- Use Trail of Bits for: [security concern]

[Agent 3+ similarly]

All agents will:
- Use Superpowers test-driven-development
- Access CLAUDE.md for conventions
- Access Claude-Mem for cross-session context
- Report with concrete evidence when complete
```

### Common Patterns

**Pattern 1: Parallel Independent Agents**
All agents spawn at t=0, all complete at roughly t=T. Example: Add logging to module A, caching to module B, monitoring to module C.

**Pattern 2: Ordered Dependencies**
AGENT 1: Define API contracts → AGENT 2: Implement server (depends on Agent 1) → AGENT 3: Implement client (depends on Agent 1). Timeline: T_api + max(T_server, T_client) < T_api + T_server + T_client (fully sequential).

**Pattern 3: Parallel Review**
AGENTS 1-N: Implementation in parallel. AGENT N+1: Security review (parallel to others, monitors changes, runs Trail of Bits). AGENT N+2: Quality review (runs Superpowers verification).

**Pattern 4: Multi-Language Coordination**
Each agent specializes in one language/ecosystem. Agent 1: Go API. Agent 2: Python ML. Agent 3: React frontend. Agent 4: Integration testing cross-language contracts.

**Pattern 5: Autonomous Fire-and-Forget**
Morning: Brainstorm + plan. Afternoon: Spawn agents, go do something else. Evening: Review + merge.

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

## Open Source Projects Used

### Development Layer

| Tool | Author | Repository | License |
|------|--------|-----------|---------|
| **Superpowers** | Jesse Vincent (obra) | github.com/obra/superpowers-marketplace | — |
| **Claude-Mem** | TheDotMack | github.com/thedotmack/claude-mem | MIT |
| **Context7** | Upstash | github.com/upstash/context7 | Apache 2.0 |
| **Trail of Bits Skills** | Trail of Bits | github.com/trailofbits/skills | AGPL-3.0 |

### Feedback Layer

| Tool | Author | Repository | License |
|------|--------|-----------|---------|
| **Claude Diary** | Lance Martin (Anthropic) | github.com/rlancemartin/claude-diary | MIT |
| **claude-code-transcripts** | Simon Willison | github.com/simonw/claude-code-transcripts | Apache 2.0 |
| **Prompt Plunderer** | Levin Dixon | github.com/levindixon/claude-code-prompt-plunderer | MIT |

### Optional / Referenced

| Tool | Author | Repository |
|------|--------|-----------|
| **ccundo** | Anthropic community | github.com/anthropics/ccundo |
| **Semgrep** | Semgrep Inc. | github.com/semgrep/semgrep |

---

## Troubleshooting

### Agent Not Spawning

**Symptom:** Asked to spawn agents but it doesn't.

**Fixes:**
1. Use explicit phrasing: "Spawn N agents: Agent 1, Agent 2, Agent 3"
2. Make task larger (combine related work)
3. Check Claude Code version: `claude --version` (update if needed)

### Agents Working Sequentially, Not Parallel

**Symptom:** Agents should run parallel but work sequentially.

**Fixes:**
1. Remove "depends on" if agents are truly independent
2. Explicitly state: "These agents can work in parallel (no dependencies)"
3. Review what Agent A outputs that Agent B uses; remove dependency if none

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

## Integration Checklist

### Prerequisites
- [ ] Superpowers installed and practiced (Levels 1-5)
- [ ] Claude-Mem configured and working
- [ ] Context7 installed and MCP connected
- [ ] Trail of Bits skills installed
- [ ] Feedback loop running (diary + cron + /reflect)

### Learning
- [ ] Level 1: Installation smoke test
- [ ] Level 2: One feature with full Superpowers methodology
- [ ] Level 3: Cross-session memory working
- [ ] Level 4: Context7 documentation working
- [ ] Level 5: Trail of Bits security skills active
- [ ] Level 6: First multi-agent autonomous run
- [ ] Level 7: Prompt engineering for the stack
- [ ] Level 8: Multi-component agent teams
- [ ] Level 9: Failure recovery & debugging
- [ ] Level 10: Custom extensions
- [ ] Level 11: System mastery

### Integration
- [ ] Update CLAUDE.md with agent team heuristics
- [ ] Create /spawn-team slash command
- [ ] Test agent teams on real multi-component feature
- [ ] All stack tools (Context7, Trail of Bits, Claude-Mem) integrated

### Feedback
- [ ] Diary capturing decisions
- [ ] Claude-Mem storing patterns
- [ ] /reflect identifying effective patterns
- [ ] CLAUDE.md auto-updating

### Monitoring
- [ ] `claude-status` verifies stack health
- [ ] Claude-Mem observations for agent patterns
- [ ] Git history shows agent-generated commits
- [ ] Time savings measured vs. sequential execution

---

## Key Insights

**The Meta-Skill:** Delegation calibration — knowing how much autonomy to give Claude at each moment.

| Autonomy Level | When to Use | Tools Active |
|---|---|---|
| **Full manual** | Exploring, sensitive code | Superpowers + Context7 + Trail of Bits |
| **Supervised auto** | Known patterns, good tests | All five, you watching |
| **Unattended** | Mechanical tasks, well-specified | All five, you elsewhere |
| **Agent teams** | Multi-component sprints | Parallel agents in one session |

The master doesn't use more tools. The master uses the right tools at the right moment.

---

## Next Steps

1. **This week:** Complete Levels 1–2 (install + one feature)
2. **Next week:** Complete Levels 3–4 (memory + docs)
3. **Following week:** Integrate into Level 8 workflow with agent teams
4. **Ongoing:** Use `/reflect` to evolve patterns

Welcome to multi-agent orchestration. 🚀
