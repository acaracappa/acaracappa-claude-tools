# Learning Plan: 11 Levels to Mastery

A progressive path from installation to full-stack mastery of the Claude Code Toolkit. Each level builds on the previous — don't skip ahead.

---

## Level 1: Installation & Smoke Test

**Goal:** All tools installed and not conflicting.

**Tasks:**
- Follow [SETUP.md](SETUP.md) to install all 7 tools
- Verify each tool is active

**Verify:** `/help` shows Superpowers, `curl http://localhost:37777/api/readiness` works, `claude mcp list` shows Context7, `/plugin list` shows Trail of Bits.

---

## Level 2: Superpowers — Following the Workflow

**Goal:** Complete one feature using brainstorm → plan → execute → verify end-to-end.

**Tasks:**
- Pick a small feature (new endpoint, utility function, UI component)
- Let Superpowers drive the methodology — don't skip any steps
- Write tests before implementation (TDD style)

**Verify:** Feature complete with tests written before implementation. Full brainstorm → plan → execute → verify cycle documented.

---

## Level 3: Claude-Mem — Building Memory

**Goal:** Develop intuition for what Claude-Mem captures and how to query it.

**Tasks:**
- Work 3–4 sessions across 2+ days
- Observe what gets injected at session start
- Test `<private>` tags for sensitive content
- Search past work with natural language queries
- Adjust `maxTokens` setting (default 2250 usually right)

**Verify:** Can query past session context. Understand what gets captured automatically vs. what needs explicit saving.

---

## Level 4: Context7 — Documentation on Demand

**Goal:** Make Context7 a reflex for any library interaction.

**Tasks:**
- Practice three patterns: general mention, direct library ID, version-specific
- Use the subagent pattern for long sessions
- Build your library ID cheat sheet

**Verify:** Never manually searching docs. Context7 used naturally in every session involving external libraries.

---

## Level 5: Trail of Bits — Security-First Development

**Goal:** Understand what Trail of Bits catches and integrate into workflow.

**Tasks:**
- Trigger sharp-edges on sensitive code (auth, crypto, input handling)
- Run insecure-defaults scans on configuration
- Use differential-review on commits
- Compare what Superpowers catches (structure) vs. Trail of Bits (security)

**Verify:** Security review is part of normal workflow, not an afterthought.

---

## Level 6: Agent Teams — First Autonomous Run

**Goal:** Successfully spawn and complete one multi-agent autonomous session.

**Tasks:**
- Start small: 2–3 semi-independent components
- Define dependencies between components
- Watch agents respect ordering and use the stack tools (Context7, Trail of Bits)
- Verify tests pass after agents complete

**Verify:** Agents completed work autonomously. Results are coherent and tests pass. See [AGENT-TEAMS.md](AGENT-TEAMS.md) for patterns and templates.

---

## Level 7: Prompt Engineering for the Stack

**Goal:** Write prompts and CLAUDE.md that get maximum leverage from all tools.

**Tasks:**
- Write project CLAUDE.md with methodology rules
- Optimize agent team spawning prompts with clear success criteria
- Learn Context7 patterns that work
- Shape Claude-Mem deliberately at session end

**Verify:** CLAUDE.md actively improves session quality. Agent prompts produce consistent results.

---

## Level 8: Multi-Component Orchestration

**Goal:** Spawn and coordinate parallel agent teams while maintaining coherence.

**Tasks:**
- Pick real feature spanning 3+ components
- Decompose into agent team structure with clear DAG
- Spawn agents respecting dependencies
- Verify agents use each other's output correctly
- Run e2e tests

**Verify:** Multi-agent feature delivered with all components integrating correctly. See [AGENT-TEAMS.md](AGENT-TEAMS.md) for orchestration patterns.

---

## Level 9: Failure Recovery & Debugging

**Goal:** Diagnose and recover when the stack goes wrong.

**Tasks:**
- Debug stuck agents, Claude-Mem worker issues, Context7 failures
- Handle Superpowers skill refusal, Trail of Bits not activating
- Manage context exhaustion
- Know recovery playbook for each failure mode

**Verify:** Can diagnose and fix any tool failure in under 2 minutes. See [SETUP.md](SETUP.md#troubleshooting) for common fixes.

---

## Level 10: Custom Extensions & Optimization

**Goal:** Extend the stack with custom hooks, skills, configurations.

**Tasks:**
- Write custom hooks and skills
- Optimize token budget
- Create pre-flight check scripts
- Document agent team heuristics in CLAUDE.md

**Verify:** Stack customized to your workflow. Token usage optimized. Custom tools integrated.

---

## Level 11: Mastery — System Thinking

**Goal:** Operate the full stack as unified system. Complete multi-component feature end-to-end in one day.

**Sample day:**
- **Morning (2h):** Design with Superpowers
- **Midday (3–4h):** Spawn autonomous agents
- **Afternoon (2h):** Security review + integration
- **End of day:** Summarize for Claude-Mem. Feature deployed.

### Mastery Indicators

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

## Next Step

When you reach Level 6, dive into [AGENT-TEAMS.md](AGENT-TEAMS.md) for orchestration patterns and templates.

## All Docs

- [README.md](README.md) — Toolkit overview and architecture
- [SETUP.md](SETUP.md) — Installation, configuration, and troubleshooting
- [AGENT-TEAMS.md](AGENT-TEAMS.md) — Multi-agent orchestration guide
- [STATUSLINE-CONFIG.md](STATUSLINE-CONFIG.md) — Claude Code statusline setup
