# Agent Teams: Multi-Agent Orchestration

Agent teams are Claude's native multi-agent orchestration system allowing specialized agents to collaborate in parallel within a single Claude session.

---

## Core Architecture

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

---

## Integration with Your Stack

**Superpowers:** Methodology applies to each agent (TDD, verification).

**Claude-Mem:** Agents share context, avoid redundant work. Session start injects: "API uses gin, Engine uses tokio, ML uses scikit-learn."

**Context7:** Each agent gets current library docs. Agent 1 uses gin docs, Agent 2 uses tokio docs, Agent 3 uses sklearn docs.

**Trail of Bits:** Security checks run in parallel with implementation. Issues caught early, not after-the-fact.

**Feedback Loop:** Captures when spawning was effective, time saved, decisions made → /reflect identifies patterns → CLAUDE.md updates → next session benefits.

---

## When to Spawn Agents

### DO spawn when:
- Feature spans 2–4 semi-independent components
- Components have clear interfaces/contracts
- Total work >3 hours
- Work can be parallelized
- Well-defined success criteria
- Security/quality review needed

### DON'T spawn when:
- Feature is single-component (<3 hours)
- Components are tightly coupled
- Still exploring design
- High-risk code (payment, auth, crypto) needing constant oversight
- Vague success criteria
- Constant real-time feedback needed

---

## Agent Team Prompt Template

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

---

## Common Patterns

### Pattern 1: Parallel Independent Agents

All agents spawn at t=0, all complete at roughly t=T. Example: Add logging to module A, caching to module B, monitoring to module C.

### Pattern 2: Ordered Dependencies

AGENT 1: Define API contracts → AGENT 2: Implement server (depends on Agent 1) → AGENT 3: Implement client (depends on Agent 1). Timeline: T_api + max(T_server, T_client) < T_api + T_server + T_client (fully sequential).

### Pattern 3: Parallel Review

AGENTS 1-N: Implementation in parallel. AGENT N+1: Security review (parallel to others, monitors changes, runs Trail of Bits). AGENT N+2: Quality review (runs Superpowers verification).

### Pattern 4: Multi-Language Coordination

Each agent specializes in one language/ecosystem. Agent 1: Go API. Agent 2: Python ML. Agent 3: React frontend. Agent 4: Integration testing cross-language contracts.

### Pattern 5: Autonomous Fire-and-Forget

Morning: Brainstorm + plan. Afternoon: Spawn agents, go do something else. Evening: Review + merge.

---

## Cost/Benefit Analysis

### When Agent Teams Pay Off

| Scenario | Time Without Agents | Time With Agents | Savings |
|----------|-------------------|-----------------|---------|
| 3 independent modules | 6–9 hours (sequential) | 2–3 hours (parallel) | ~60–70% |
| API + client + tests | 4–6 hours | 2–3 hours | ~50% |
| Implementation + security review | 5–7 hours | 3–4 hours | ~40% |

### When Agent Teams Cost More

- **Tightly coupled code:** Agents block on each other, losing parallelism
- **Exploratory work:** Design isn't clear enough for agents to execute independently
- **Small tasks (<2 hours):** Overhead of spawning + coordination exceeds time saved
- **High-risk code:** Constant oversight needed defeats autonomous execution

### Token Costs

Agent teams consume more tokens than single-agent work:
- Each agent maintains its own context window
- Shared context (CLAUDE.md, Claude-Mem) is loaded per agent
- Expect 2–4x total token usage for 2–4 agents
- ROI comes from wall-clock time savings, not token efficiency

### Decision Rule

**Spawn agents when:** `(number_of_components >= 2) AND (components_are_independent) AND (total_work > 3_hours) AND (success_criteria_are_concrete)`

If any condition fails, execute sequentially.

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

---

## All Docs

- [README.md](README.md) — Toolkit overview and architecture
- [SETUP.md](SETUP.md) — Installation, configuration, and troubleshooting
- [LEARNING-PLAN.md](LEARNING-PLAN.md) — 11-level mastery path
- [STATUSLINE-CONFIG.md](STATUSLINE-CONFIG.md) — Claude Code statusline setup
