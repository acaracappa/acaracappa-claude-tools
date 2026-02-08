# Claude Code Toolkit

A curated, integrated toolkit that transforms Claude Code from a reactive coding assistant into a disciplined, memory-aware, security-conscious engineering system with a self-improving feedback loop.

---

## What This Is

Five development-layer tools and three feedback-layer tools, selected to work together without conflict. Each operates at a different layer of the Claude Code stack and solves a distinct problem.

## Architecture

```
┌───────────────────────────────────────────────────────────────────────────┐
│                           YOUR PROJECT REPO                               │
│                                                                           │
│  DEVELOPMENT LAYER (active during sessions)                               │
│  ┌───────────────┐ ┌──────────────┐ ┌────────────┐ ┌───────────┐          │
│  │  SUPERPOWERS  │ │  CLAUDE-MEM  │ │  CONTEXT7  │ │TRAIL OF   │          │
│  │  (Skills &    │ │  (Memory &   │ │  (Live     │ │  BITS     │          │
│  │   Workflow)   │ │   Context)   │ │   Docs)    │ │(Security) │          │
│  │  Plugin +     │ │  Plugin +    │ │  MCP       │ │ Plugin +  │          │
│  │  Skills +     │ │  Hooks +     │ │  Server +  │ │ Skills    │          │
│  │  Hooks +      │ │  Worker +    │ │  Plugin    │ │(on-demand)│          │
│  │  Commands     │ │  SQLite DB   │ │            │ │           │          │
│  └──────┬────────┘ └──────┬───────┘ └─────┬──────┘ └─────┬─────┘          │
│         │                 │               │              │                │
│         ▼                 ▼               ▼              ▼                │
│  ┌──────────────────────────────────────────────────────────────────┐     │
│  │                     CLAUDE CODE CLI / SDK                        │     │
│  │   (hooks merge, skills stack, MCP lazy-loads, security on-demand)│     │
│  └──────────────────────────────────────────────────────────────────┘     │
│                                                                           │
│  FEEDBACK LAYER (runs between sessions)                                   │
│  ┌───────────────┐ ┌──────────────┐ ┌──────────────┐                      │
│  │  TRANSCRIPTS  │ │ CLAUDE DIARY │ │  PLUNDERER   │                      │
│  │  (Preserve)   │ │  (Observe)   │ │  (Mine)      │                      │
│  │  JSONL → HTML │ │  /diary +    │ │  Prompt      │                      │
│  │  weekly cron  │ │  /reflect    │ │  patterns    │                      │
│  └───────────────┘ └──────────────┘ └──────────────┘                      │
└───────────────────────────────────────────────────────────────────────────┘
```

## Tool Summary

### Development Layer

| Tool | Layer | Concern | What It Does |
|------|-------|---------|--------------|
| **Superpowers** | Skills + Workflow | *How* Claude thinks and builds | Enforces brainstorm → plan → execute → verify methodology. TDD, subagents, worktrees, systematic debugging. |
| **Claude-Mem** | Memory + Context | *What* Claude remembers across sessions | Persistent cross-session memory via SQLite + vector embeddings. Privacy controls, configurable injection. |
| **Context7** | Knowledge + Docs | *What* Claude knows about your libraries | Fetches current, version-specific documentation from source repos at query time. Eliminates hallucinated APIs. |
| **Trail of Bits** | Security + Analysis | *How safe* Claude's output is | Professional-grade security skills: sharp-edges, differential-review, static-analysis, insecure-defaults, fix-review. |

### Feedback Layer

| Tool | Layer | Concern | What It Does |
|------|-------|---------|--------------|
| **Claude Diary** | Observation + Reflection | *What* Claude learns from sessions | Captures session observations, reflects across diary entries, auto-updates CLAUDE.md with learned rules. |
| **claude-code-transcripts** | Archival | *What's* preserved before auto-cleanup | Converts raw JSONL session files to browsable HTML archives. Safety net for historical data. |
| **Prompt Plunderer** | Pattern Mining | *How* you actually work | Extracts and analyzes prompting patterns, identifies repetitive workflows, generates slash commands. |

## How the Loop Closes

```
Development Layer runs sessions
    ↓ generates
Raw JSONL session data
    ↓ feeds
Feedback Layer (preserve → observe → mine → reflect)
    ↓ outputs
Updated CLAUDE.md rules + new slash commands
    ↓ feeds back into
Next Development Layer session (improved)
```

One manual step per week: run `/reflect` (~10 minutes). Everything else is automated via cron and hooks.

## Token Budget

| Component | Session Start Cost | Runtime Cost (per use) |
|-----------|-------------------|----------------------|
| Superpowers bootstrap | ~500–1,000 tokens | Skills loaded on-demand |
| Claude-Mem injection | ~2,250 tokens (configurable) | Async via worker |
| Context7 MCP | **0 tokens** (lazy-loads) | ~500–2,000 per doc query |
| Trail of Bits skills | **0 tokens** (on-demand) | ~300–500 per skill activation |
| **Total at start** | **~2,750–3,250 tokens** | — |

This is ~1.5% of the 200k context window.

## Potential Conflicts & Mitigations

**Hook ordering:** Superpowers and Claude-Mem both register SessionStart hooks. Claude Code merges them additively — both fire, order is non-deterministic but irrelevant since they operate independently.

**Context7 + Superpowers:** No conflict. Context7 is MCP-based (tool calls), Superpowers is skill-based (prompt injection). When brainstorming needs API info, Claude naturally invokes Context7. They compose, not compete.

**Trail of Bits + Superpowers:** Complementary. Superpowers provides methodology; Trail of Bits provides security analysis within that workflow. During verification-before-completion, Trail of Bits skills contribute security evidence.

**Bun dependency:** Claude-Mem requires Bun. Context7 requires Node.js (for npx). Both are likely already installed.

**VSCode extension caveat:** Claude-Mem hooks may not trigger in the VSCode extension (known issue). All tools work reliably with the CLI version.

## Project Files

| File | Purpose |
|------|---------|
| `README.md` | This file — architecture overview and tool summary |
| `SETUP.md` | Installation guide for all eight tools |
| `LEARNING-PLAN.md` | 11-level mastery plan with exercises and success criteria |
| `FEEDBACK-LOOP.md` | Feedback loop automation, health monitoring, and dashboard scripts |

## Open Source Projects Used

This toolkit integrates the following open source projects. All credit to their respective authors and contributors.

### Development Layer

| Tool | Author | Repository | License |
|------|--------|-----------|---------|
| **Superpowers** | Jesse Vincent (obra) | [github.com/obra/superpowers-marketplace](https://github.com/obra/superpowers-marketplace) | — |
| **Claude-Mem** | TheDotMack | [github.com/thedotmack/claude-mem](https://github.com/thedotmack/claude-mem) | MIT |
| **Context7** | Upstash | [github.com/upstash/context7](https://github.com/upstash/context7) | Apache 2.0 |
| **Context7 Plugin** | Context7 | [github.com/context7/context7-marketplace](https://github.com/context7/context7-marketplace) | — |
| **Trail of Bits Skills** | Trail of Bits | [github.com/trailofbits/skills](https://github.com/trailofbits/skills) | AGPL-3.0 |
| **Ralph** | Frank Bria | [github.com/frankbria/ralph-claude-code](https://github.com/frankbria/ralph-claude-code) | MIT |

### Feedback Layer

| Tool | Author | Repository | License |
|------|--------|-----------|---------|
| **Claude Diary** | Lance Martin (Anthropic) | [github.com/rlancemartin/claude-diary](https://github.com/rlancemartin/claude-diary) | MIT |
| **claude-code-transcripts** | Simon Willison | [github.com/simonw/claude-code-transcripts](https://github.com/simonw/claude-code-transcripts) | Apache 2.0 |
| **Prompt Plunderer** | Levin Dixon | [github.com/levindixon/claude-code-prompt-plunderer](https://github.com/levindixon/claude-code-prompt-plunderer) | MIT |

### Optional / Referenced

| Tool | Author | Repository | Description |
|------|--------|-----------|-------------|
| **ccundo** | Anthropic community | [github.com/anthropics/ccundo](https://github.com/anthropics/ccundo) | Surgical undo for Claude Code file operations |
| **Semgrep** | Semgrep Inc. | [github.com/semgrep/semgrep](https://github.com/semgrep/semgrep) | Static analysis engine used by Trail of Bits skills |

> **Note:** Licenses listed are as of the time of writing. Check each repository for current licensing terms before use. Entries marked "—" had no license file visible at the time of review.
