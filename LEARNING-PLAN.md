# Learning Plan: Path to Mastery

11 levels, each building on the last. Each level has specific exercises, success criteria, and a self-assessment marker.

**Time estimate:** 5–7 weeks at 1–2 hours/day of deliberate practice. You'll be productive from Level 2 onward — this isn't about waiting to use the tools, it's about deepening your command of them while shipping real work.

---

## Level 1: Installation & Smoke Test

**Goal:** All tools installed, verified, and not conflicting.

### Do This

1. Install all tools following `SETUP.md` (TL;DR section or full walkthrough)
2. Run each verification check:
   - `/help` shows Superpowers commands
   - `curl http://localhost:37777/api/readiness` returns JSON
   - `claude mcp list` shows Context7 connected
   - `/plugin list` shows Trail of Bits skills
   - `ralph --status` shows version
3. Start a fresh Claude Code session and type:
   ```
   Do you have superpowers? What do you remember from past sessions?
   Use context7 to look up gin middleware documentation.
   Then review the security of any file using your sharp-edges skill.
   ```
4. Confirm Claude acknowledges superpowers, references memory (or says none yet), fetches docs, and runs a security check

### Success Criteria

- [ ] All install commands completed without errors
- [ ] All verification checks pass
- [ ] Single integrated test prompt gets correct responses from all layers
- [ ] Trail of Bits skills appear in plugin list and activate when asked

### You've leveled up when...

You can install this stack from scratch on a clean machine in under 15 minutes without referencing docs.

---

## Level 2: Superpowers — Following the Workflow

**Goal:** Complete one full feature using the Superpowers methodology end-to-end.

### Do This

1. Pick a small, real feature. Something like "add a health check endpoint" or "create a config file parser."
2. Start Claude Code and say: "I want to build [feature]"
3. **Don't skip any steps.** Let Superpowers drive:
   - Brainstorming phase: answer Claude's clarifying questions honestly
   - Design review: read each section Claude presents, give real feedback
   - Plan review: check the implementation plan makes sense
   - Execution: let subagents do the TDD implementation
   - Verification: confirm Claude provides evidence of completion
4. After completion, inspect what happened:
   - What branch did the worktree skill create?
   - How many subagents were spawned?
   - Were tests written before implementation (red → green)?

### Study Material

Read these skill files directly (they're in your plugin cache after install):

```
~/.claude/plugins/cache/Superpowers/skills/brainstorming/SKILL.md
~/.claude/plugins/cache/Superpowers/skills/test-driven-development/SKILL.md
~/.claude/plugins/cache/Superpowers/skills/subagent-driven-development/SKILL.md
~/.claude/plugins/cache/Superpowers/skills/verification-before-completion/SKILL.md
```

### Success Criteria

- [ ] Completed one feature using full brainstorm → plan → execute → verify pipeline
- [ ] Feature has tests written in TDD style (tests existed before implementation)
- [ ] Code is on an isolated branch created by the worktree skill
- [ ] Can articulate what each phase does and why it exists

### You've leveled up when...

You instinctively feel wrong when Claude starts coding without brainstorming first. The methodology becomes your default, not a conscious choice.

---

## Level 3: Claude-Mem — Building Memory

**Goal:** Develop intuition for what Claude-Mem captures, how to query it, and how privacy controls work.

### Do This

1. Work with Claude Code for 3–4 sessions across 2+ days on the same project
2. At the start of each session, observe what Claude-Mem injects:
   - Open the web viewer at `http://localhost:37777`
   - Note what observations were captured from the previous session
   - Pay attention to what Claude "already knows" when you start
3. Test privacy controls:
   ```
   <private>My API key is PKTEST123456</private>
   Now configure the client with the key I just gave you.
   ```
   Check the web viewer — the API key should NOT appear in stored observations
4. Test memory search:
   ```
   Search your memory for what we worked on last week.
   ```
5. Experiment with `maxTokens` setting:
   - Set it to 500, restart, start a session — notice reduced context injection
   - Set it to 3000, restart — notice richer context injection
   - Find your sweet spot (default 2250 is usually right)

### Study Material

```bash
# Examine the database directly
sqlite3 ~/.claude-mem/claude-mem.db ".tables"
sqlite3 ~/.claude-mem/claude-mem.db "SELECT * FROM observations ORDER BY created_at DESC LIMIT 10;"
```

### Success Criteria

- [ ] Can explain what Claude-Mem captured from a given session by checking the web viewer
- [ ] Verified that `<private>` tags properly exclude sensitive data
- [ ] Used mem-search to query past work successfully
- [ ] Adjusted maxTokens and observed the effect on session starts

### You've leveled up when...

You naturally use `<private>` tags around secrets without thinking about it, and you notice when Claude-Mem's context injection is helping Claude skip redundant exploration.

---

## Level 4: Context7 — Documentation on Demand

**Goal:** Make Context7 a reflex for any library interaction, and understand its limits.

### Do This

1. **Explicit usage** — Practice the three invocation patterns:
   ```
   # Pattern 1: General mention
   "use context7 to show me tokio channel examples"

   # Pattern 2: Direct library ID
   "use context7 with /gin-gonic/gin for middleware chaining"

   # Pattern 3: Version-specific
   "use context7 for scikit-learn 1.4 cross-validation"
   ```

2. **Auto-triggering** (full plugin only) — Just mention libraries naturally:
   ```
   "Implement the WebSocket handler using gorilla/websocket"
   ```
   Watch the skill auto-trigger — Claude should fetch docs without "use context7"

3. **Subagent pattern** — During a long session, offload doc lookups:
   ```
   "spawn docs-researcher to look up rust sqlx connection pooling"
   ```
   This runs in isolated context, doesn't pollute your main session

4. **Test the limits:**
   - Try a very new or obscure library that probably isn't indexed
   - Note how Context7 handles it (should say "library not found")
   - Fall back to regular web search for unindexed libraries

5. **Build your library ID cheat sheet:**
   Identify the 10 libraries you use most and find their Context7 IDs. Pin them in your project CLAUDE.md.

### Success Criteria

- [ ] Used all three invocation patterns successfully
- [ ] Observed auto-triggering work (or manually triggered if MCP-only install)
- [ ] Used docs-researcher subagent during a long session
- [ ] Found one library that ISN'T indexed and know the fallback
- [ ] Have a personal cheat sheet of your top 10 library IDs

### You've leveled up when...

You never write "how do I use X in library Y" — you just reference the library and Context7 handles it. Your code uses correct APIs on the first try consistently.

---

## Level 5: Trail of Bits — Security-First Development

**Goal:** Understand how Trail of Bits skills activate, what they catch, and how to incorporate security checks into your daily workflow.

### Do This

1. **Understand what you installed:**
   ```
   sharp-edges        → error-prone APIs, footgun designs
   differential-review → blast radius of code changes
   static-analysis    → CodeQL + Semgrep automated scanning
   insecure-defaults  → hardcoded secrets, weak crypto, default creds
   fix-review         → verify fixes don't introduce new bugs
   ```

2. **Trigger sharp-edges deliberately.** Open a file that handles sensitive operations (auth, payments, external API calls):
   ```
   Review src/handlers/auth.go using your sharp-edges skill.
   What error-prone patterns or dangerous API usage do you find?
   ```

3. **Trigger insecure-defaults.** Scan for credential issues:
   ```
   Scan the entire config/ and internal/auth/ directories
   for insecure defaults using your insecure-defaults skill.
   ```

4. **Test differential-review on a real PR.** Make a change, commit it:
   ```
   Use differential-review to analyze my last commit.
   What's the blast radius? What security implications exist?
   ```

5. **Run static-analysis with Semgrep:**
   ```
   Run Semgrep on the codebase to find security vulnerabilities.
   Focus on injection flaws, authentication issues, and crypto misuse.
   ```
   Note: Requires Semgrep installed (`pip install semgrep`).

6. **Compare Trail of Bits vs. Superpowers review** on the same file:
   - Superpowers catches structural issues (naming, testing, architecture)
   - Trail of Bits catches security issues (unsafe patterns, credential exposure, crypto misuse)
   - They're complementary

### Study Material

- Trail of Bits skills repo: `https://github.com/trailofbits/skills`
- Trail of Bits Testing Handbook: `https://appsec.guide`

### Success Criteria

- [ ] Successfully triggered sharp-edges on a real file and understood the findings
- [ ] Ran insecure-defaults scan and addressed or acknowledged all findings
- [ ] Used differential-review on a real commit and understood the blast radius report
- [ ] Attempted a Semgrep scan (or noted Semgrep isn't installed yet)
- [ ] Can articulate the difference between what Superpowers catches vs. Trail of Bits
- [ ] No false sense of security — you understand these are aids, not guarantees

### You've leveled up when...

You instinctively ask for a security review after modifying auth, external API, or sensitive business logic code. Trail of Bits skills feel like a natural part of the development flow, not an afterthought.

---

## Level 6: Ralph — First Autonomous Run

**Goal:** Successfully complete one unattended Ralph loop (5–10 iterations) that ships working code.

### Do This

1. **Start small.** Pick something mechanical:
   - "Add unit tests for all functions in `pkg/utils/`"
   - "Create CRUD API endpoints for the users model"
   - "Add error handling and logging to all external API calls"

2. **Set up the task:**
   ```bash
   cd ~/projects/myapp
   ralph-setup add-tests
   # Edit .ralph/PROMPT.md with clear, specific requirements
   # Edit .ralph/fix_plan.md with a checklist of tasks
   ```

3. **Configure conservatively:**
   ```bash
   # .ralphrc
   CLAUDE_MAX_CALLS_PER_HOUR=20
   CLAUDE_TIMEOUT_MINUTES=15
   MAX_CONSECUTIVE_TEST_LOOPS=3
   ```

4. **Run with monitoring:**
   ```bash
   # Terminal 1
   ralph --monitor

   # Terminal 2 (optional)
   ralph-monitor
   ```

5. **Watch the first 2–3 iterations** before walking away. Verify:
   - Is Superpowers' methodology activating?
   - Is Claude-Mem injecting relevant context?
   - Is Context7 being used for library references?
   - Are Trail of Bits skills firing on security-sensitive files?

6. **After completion**, review:
   - `git log` for commits made during the run
   - `.ralph/fix_plan.md` — all items should be `[x]`
   - Test results — all passing?
   - Claude-Mem web viewer — what was captured?

### Success Criteria

- [ ] Completed 5+ Ralph iterations without manual intervention
- [ ] All fix_plan.md items marked complete
- [ ] Code compiles and tests pass
- [ ] Ralph exited cleanly (exit detection worked)
- [ ] Can read ralph-monitor output and understand what happened

### You've leveled up when...

You trust Ralph enough to start a run, go make dinner, and come back to merged code. But you still review before pushing.

---

## Level 7: Prompt Engineering for the Stack

**Goal:** Write prompts, CLAUDE.md files, and Ralph configs that get maximum leverage from all five tools.

### Do This

1. **Write a project CLAUDE.md** that references the toolkit:
   ```markdown
   # Project Context

   ## Architecture
   - [describe your components/services]

   ## Development Methodology
   - Always use Superpowers brainstorming before implementing features
   - Use TDD (red → green → refactor) for all new code
   - Use Context7 for all library API references — don't guess
   - Run Trail of Bits sharp-edges on code touching [sensitive paths]
   - Use differential-review on all PRs touching security-sensitive paths

   ## Conventions
   - [your language/framework conventions]

   ## Security-Sensitive Paths (always trigger Trail of Bits review)
   - [list your sensitive directories]
   ```

2. **Optimize .ralph/PROMPT.md** for autonomous runs:
   - Be specific about what "done" means
   - Reference which test suites must pass
   - Tell Claude which Context7 library IDs to use
   - Set explicit boundaries ("do NOT modify config/production.yaml")

3. **Write better brainstorming prompts:**
   Instead of: "Add WebSocket support"
   Write: "Add WebSocket support for real-time updates. The API uses gin. The frontend connects from React. Handle reconnection, heartbeats, and graceful shutdown."

4. **Learn Context7 prompt patterns that work:**
   - ❌ "How do I use tokio?" (too vague)
   - ✅ "use context7 with /tokio-rs/tokio for bounded mpsc channel with backpressure"

5. **Shape Claude-Mem deliberately** at session end:
   ```
   Before we end: remember that we decided to use tokio::sync::mpsc
   with a buffer of 1000, and we chose JSON over protobuf for the
   wire format because the frontend team prefers it.
   ```

### Success Criteria

- [ ] CLAUDE.md written and tested (Claude references it during sessions)
- [ ] .ralph/PROMPT.md that leads to successful autonomous runs
- [ ] Brainstorming prompts consistently trigger deeper exploration
- [ ] Context7 queries return targeted, useful documentation
- [ ] Deliberate session-end summaries improve next-session context

### You've leveled up when...

You think about prompts as infrastructure, not throwaway text. Your CLAUDE.md and Ralph configs are versioned in git and evolve with the project.

---

## Level 8: Multi-Component Orchestration

**Goal:** Run parallel Ralph instances across components while maintaining coherence through Claude-Mem.

### Do This

1. **Create component-specific Ralph configs** for different parts of a polyglot or multi-service project

2. **Run parallel Ralph instances:**
   ```bash
   # Terminal 1: API layer
   cd ~/project/api && ralph --monitor

   # Terminal 2: Core engine
   cd ~/project/engine && ralph --monitor

   # Terminal 3: ML pipeline
   cd ~/project/ml && ralph --monitor
   ```

3. **Observe cross-component memory:** After one Ralph run finishes, start a session in a different component directory. Does Claude-Mem inject relevant context about the other component's changes?

4. **Handle cross-component dependencies:** Run dependent components in sequence (e.g., establish an interface first, then build the consumer)

5. **Coordinate via git:** After parallel runs complete, merge branches and resolve interface conflicts

### Success Criteria

- [ ] Successfully ran 2+ Ralph instances in parallel on different components
- [ ] Cross-component changes are coherent
- [ ] Git history shows clean, parallel development branches
- [ ] Claude-Mem context from one component informs work in another

### You've leveled up when...

You routinely break sprints into parallel Ralph runs and merge the results. Your throughput on multi-component features is 3–5x what it was doing them serially.

---

## Level 9: Failure Recovery & Debugging

**Goal:** Diagnose and recover when the stack goes wrong.

### Do This

1. **Debug a stuck Ralph loop:** Create a fix_plan.md with an impossible task, watch Ralph iterate, identify the stuck point, intervene mid-run

2. **Debug Claude-Mem issues:**
   ```bash
   curl http://localhost:37777/api/readiness
   sqlite3 ~/.claude-mem/claude-mem.db "PRAGMA integrity_check;"
   sqlite3 ~/.claude-mem/claude-mem.db \
     "SELECT created_at, summary FROM observations ORDER BY created_at DESC LIMIT 5;"
   ```

3. **Debug Context7 failures:** What happens when it's down? (Falls back to training data.) What about a bad library ID? (`claude mcp list` to check health, remove and re-add if needed.)

4. **Debug Superpowers skill refusal:** Claude jumping straight to code without brainstorming. Fix: "Use your brainstorming skill before writing any code."

5. **Debug Trail of Bits skill issues:**
   - Skills not firing? Be explicit: "Run your sharp-edges skill on the file you just modified."
   - False positives? Learn to distinguish (test file with hardcoded creds = fine; production config = not)
   - Missing tools? `static-analysis` requires Semgrep locally. Install with `pip install semgrep`.
   - Plugin not loaded? Check `/plugin list`, reinstall if missing.

6. **Debug context exhaustion:** Long session → compaction → signal loss. Mitigation: `/clear` and start fresh (Claude-Mem re-injects context).

### Success Criteria

- [ ] Identified and resolved a stuck Ralph loop
- [ ] Debugged a Claude-Mem worker issue
- [ ] Handled a Context7 outage gracefully
- [ ] Caught Superpowers skill refusal and corrected it
- [ ] Diagnosed a Trail of Bits skill not activating and fixed it
- [ ] Managed context exhaustion in a long session

### You've leveled up when...

When something goes wrong, you diagnose it in under 2 minutes. You have a mental model of each tool's failure modes and know the recovery playbook for each.

---

## Level 10: Custom Extensions & Optimization

**Goal:** Extend the stack with custom hooks, skills, and configurations tailored to your workflow.

### Do This

1. **Write a custom hook:**
   ```json
   {
     "event": "PostToolUse",
     "command": "bash -c 'if echo \"$TOOL_INPUT\" | grep -q \"production\"; then echo \"⚠️ PRODUCTION file detected\"; fi'",
     "description": "Warn on production file modifications"
   }
   ```

2. **Write a custom skill:**
   ```markdown
   <!-- .claude/skills/my-safety/SKILL.md -->
   ---
   name: my-project-safety
   description: "Safety checks for sensitive paths"
   ---

   When modifying code in [sensitive path]:
   1. NEVER modify production config files
   2. ALWAYS add logging with request IDs for audit trail
   3. Use Context7 for API reference
   4. Run Trail of Bits sharp-edges on modified files
   ```

3. **Optimize token budget:** Profile typical sessions, adjust Claude-Mem's `maxTokens` based on actual usage

4. **Build Ralph templates** for recurring task types

5. **Create a pre-flight check script** (see `FEEDBACK-LOOP.md` for a comprehensive version)

### Success Criteria

- [ ] Custom hook running and triggering on relevant events
- [ ] Custom skill written, tested, and used during sessions
- [ ] Token budget optimized based on measured usage patterns
- [ ] Ralph template for a recurring task type
- [ ] Pre-flight check script works reliably

### You've leveled up when...

You're extending the tools to fit your workflow rather than fitting your workflow to the tools. Your `.claude/` directory is a curated environment, not a default config.

---

## Level 11: Mastery — System Thinking

**Goal:** Operate the full stack as a unified system with intuitive command of every component.

### The Level 11 Test

Complete this scenario end-to-end in a single day:

**Scenario:** A new feature that spans multiple components.

1. **Morning (Interactive, 2 hours):** Design with Superpowers brainstorming, Context7 for current API docs, Claude-Mem for past architectural decisions. Produce a design document and implementation plan.

2. **Midday (Autonomous, 3–4 hours):** Set up parallel Ralph instances per component. Launch with `ralph --monitor`. Go do something else.

3. **Afternoon (Security Review + Integration, 2 hours):** Review output, run Trail of Bits differential-review on each component, merge branches, run integration tests, use Superpowers verification for final check.

4. **End of Day:** Summarize decisions for Claude-Mem. Feature is merged, tested, and deployable.

### Mastery Indicators

- [ ] **Install** the full stack from memory on a new machine in 15 minutes
- [ ] **Explain** each tool's architecture, hook points, and failure modes
- [ ] **Design** features using full Superpowers methodology without skipping steps
- [ ] **Secure** code using Trail of Bits skills integrated into workflow
- [ ] **Run** parallel Ralph instances across 3+ components and merge coherent results
- [ ] **Debug** any tool failure in under 2 minutes
- [ ] **Extend** the stack with custom hooks, skills, and configurations
- [ ] **Optimize** token budgets based on measured session profiles
- [ ] **Teach** someone else to get from Level 0 to Level 6 in a week
- [ ] **Ship** multi-component features 3–5x faster than without the stack
- [ ] **Know** when NOT to use a tool

### The Meta-Skill

The real Level 11 insight is **delegation calibration** — knowing how much autonomy to give Claude at each moment:

| Autonomy Level | When to Use | Tools Active |
|---------------|------------|--------------|
| **Full manual** | Exploring new territory, sensitive code | Superpowers + Context7 + Trail of Bits |
| **Supervised auto** | Known problem patterns, good test coverage | All five, you watching |
| **Unattended** | Mechanical tasks, well-specified work | All five, you elsewhere |
| **Parallel batch** | Multi-component sprints | Multiple Ralph instances |

The master doesn't use more tools. The master uses the right tools at the right moment.

---

## Quick Reference: Daily Workflow

```
Morning startup:
  $ claude-status               # Check stack health (see FEEDBACK-LOOP.md)
  $ claude                      # Start session
  > What do you remember?       # Check Claude-Mem context
  > Let's brainstorm [feature]  # Superpowers drives design

Implementation:
  > /superpowers:execute-plan   # TDD with subagents
  > (Context7 auto-fetches docs as needed)
  > (Trail of Bits fires on security-sensitive changes)
  > (Claude-Mem captures observations)

Autonomous work:
  $ ralph-setup [task]          # Define the work
  $ ralph --monitor             # Let it run
  $ ralph-monitor               # Check progress

Security review:
  > Review my last commit with differential-review
  > Run sharp-edges on internal/auth/

Recovery:
  $ ccundo list                 # See recent operations
  $ ccundo undo                 # Surgical revert if needed
  $ curl localhost:37777/api/readiness  # Check Claude-Mem

End of day:
  > Remember: we decided [X] because [Y], next session
  > we should start with [Z].   # Shape Claude-Mem capture

Weekly:
  $ /reflect                    # Review diary entries, update CLAUDE.md (~10 min)
```
