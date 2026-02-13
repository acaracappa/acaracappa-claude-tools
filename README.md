# Claude Code Toolkit

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

## Getting Started

1. **Install everything:** See [SETUP.md](SETUP.md) for prerequisites, installation, automation, and troubleshooting
2. **Learn the stack:** Follow the 11-level progression in [LEARNING-PLAN.md](LEARNING-PLAN.md)
3. **Scale up:** When ready for multi-agent work, see [AGENT-TEAMS.md](AGENT-TEAMS.md)
4. **Configure your statusline:** See [STATUSLINE-CONFIG.md](STATUSLINE-CONFIG.md)

---

## All Docs

| Doc | Purpose |
|-----|---------|
| **[SETUP.md](SETUP.md)** | Installation, configuration, automation, troubleshooting |
| **[LEARNING-PLAN.md](LEARNING-PLAN.md)** | 11-level progressive mastery path |
| **[AGENT-TEAMS.md](AGENT-TEAMS.md)** | Multi-agent orchestration patterns and templates |
| **[STATUSLINE-CONFIG.md](STATUSLINE-CONFIG.md)** | Claude Code statusline setup |
