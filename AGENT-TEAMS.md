# Agent Teams: Multi-Agent Orchestration for Your Stack

A comprehensive guide to learning and implementing Claude's native agent teams feature within your existing toolkit ecosystem.

---

## Part 1: What Are Agent Teams?

### Definition

Agent teams are Claude's native multi-agent orchestration system. They allow you to define specialized agents within a *single Claude session* that:

- **Collaborate in parallel** on independent or sequenced tasks
- **Share context automatically** via unified session memory and Claude-Mem
- **Coordinate dependencies** ("Agent B waits for Agent A's output")
- **Maintain coherence** across specialized knowledge domains
- **Report back synchronously** to the parent session for review and iteration

### Core Benefit

Agent teams provide **structured parallelism** for multi-component features. The stack you've built (Superpowers, Claude-Mem, Context7, Trail of Bits) becomes a shared resource for all agents automatically:

- **Superpowers:** Methodology applies to each agent (TDD, verification)
- **Claude-Mem:** Agents share context, avoid redundant work
- **Context7:** All agents use current library docs
- **Trail of Bits:** Security checks run during implementation, not after

---

## Part 2: How Agent Teams Integrate with Your Stack

### Architecture: Agent Teams + Your Toolkit

```
YOUR SESSION (Interactive or Autonomous)
│
├─ Superpowers: Brainstorm → Plan
│  (You or lead agent designs the work)
│
└─ Agent Spawning Decision
   │
   ├─ SIMPLE TASK (single skill, <2 hours)
   │  └─→ Execute with Superpowers locally
   │
   └─ COMPLEX TASK (multi-domain, >2 hours)
      └─→ Spawn Agent Team
         │
         ├─ Agent 1: API Layer
         │  └─ Reads: Context7 (API docs), Claude-Mem (architecture)
         │  └─ Security: Trail of Bits (sharp-edges on HTTP handlers)
         │  └─ Testing: Superpowers (TDD)
         │  └─ Reports: "API interface complete, ready for Engine"
         │
         ├─ Agent 2: Core Engine (depends on Agent 1)
         │  └─ Reads: Agent 1's output, Context7 (library docs)
         │  └─ Waits: For Agent 1 to finish interface
         │  └─ Security: Trail of Bits (data flow analysis)
         │  └─ Reports: "Core logic complete, tests passing"
         │
         └─ Agent 3: Security Review (parallel to others)
            └─ Monitors: Changes from Agents 1–2
            └─ Skills: differential-review + insecure-defaults
            └─ Reports: "No security regressions detected"

         All agents log observations to Claude-Mem
         ↓
PARENT SESSION collects reports
├─ Verify: All tests pass?
├─ Review: Differential review shows acceptable blast radius?
├─ Integrate: Merge branches, resolve conflicts
└─ Commit: Record success in git + diary

FEEDBACK LOOP captures:
├─ When agent spawning was effective (team size, task complexity)
├─ Which agents ran in parallel vs. sequence
├─ Claude-Mem effectiveness (did agents avoid redundant work?)
├─ Time saved vs. sequential execution
└─ /reflect identifies patterns → improves future decisions
```

### Layer-by-Layer Integration

#### Superpowers + Agent Teams

**Superpowers provides the methodology; Agent Teams execute it.**

```
Single brainstorm/plan session:
  > Brainstorm feature spanning API + Engine + ML
  > Identify 3 semi-independent domains
  > Design interaction points (API interface, Engine contracts)

Spawn agent team:
  > Agent 1: Implement API (uses TDD via Superpowers)
  > Agent 2: Implement Engine (uses TDD, depends on Agent 1)
  > Agent 3: Implement ML (uses TDD, depends on Engine)

Each agent has access to:
  • Superpowers skills (test-driven-development, verification-before-completion)
  • CLAUDE.md rules (your team's methodology)
  • Project context (from parent session)
```

**Result:** Same structured methodology (brainstorm → plan → execute → verify) applied at both team and individual-agent levels.

---

#### Claude-Mem + Agent Teams

**Claude-Mem becomes the communication backbone.**

```
SESSION START:
  Claude-Mem injects: "Last session we decided API uses gin,
                       Engine uses tokio::sync::mpsc with buffer 1000,
                       ML uses scikit-learn for inference"

AGENT TEAM SPAWNS:
  Agent 1 (API):
    • Reads injected context
    • Implements using gin
    • Adds observation: "API interface defined: POST /predict accepts []float64"
    → Writes to Claude-Mem

  Agent 2 (Engine):
    • Reads Agent 1's observation from Claude-Mem
    • Reads injected context (tokio decision)
    • Implements using tokio::sync::mpsc
    • Validates against API interface
    → Writes to Claude-Mem

  Agent 3 (ML):
    • Reads Agents 1–2 observations
    • Reads injected context (scikit-learn)
    • Integrates models into Engine's pipeline
    → Writes to Claude-Mem

PARENT SESSION:
  Calls /reflect on agent observations
  → Learns: "Agent team parallelism saved 2 hours vs. sequential"
  → Updates CLAUDE.md: "For multi-component features >6 hours, spawn agent team"
  → Next session: Claude-Mem recommends agent spawning earlier
```

**Result:** Agents don't rediscover decisions; they build on past context + each other's work.

---

#### Context7 + Agent Teams

**Each agent gets access to up-to-date library documentation.**

```
Agent 1 (API in Go):
  > Use context7 with /gin-gonic/gin for middleware composition
  → Context7 fetches current Gin docs (not hallucinated)
  → Implements correct middleware chain

Agent 2 (Engine in Rust):
  > Use context7 with /tokio-rs/tokio for bounded mpsc channels
  → Context7 fetches current Tokio docs
  → Implements backpressure handling correctly

Agent 3 (ML in Python):
  > Use context7 with /scikit-learn/scikit-learn for model integration
  → Context7 fetches current sklearn docs
  → Uses correct API for version in requirements.txt

BENEFIT: All agents reference CURRENT docs simultaneously.
No version mismatches, no hallucinated APIs, no "but I thought it worked that way."
```

**Result:** Multi-language, multi-library features with correct APIs across all agents.

---

#### Trail of Bits + Agent Teams

**Security checks run in parallel with implementation.**

```
Agent 1 (API):
  Implements: POST /predict endpoint
  Trail of Bits activates:
    • sharp-edges: "Validates input types, but missing rate limiting"
    • insecure-defaults: "Returns raw numpy arrays without sanitization"
  → Reports security gaps to parent

Agent 2 (Engine):
  Implements: tokio channels + data processing
  Trail of Bits activates:
    • differential-review: "Calculates blast radius of state changes"
    • sharp-edges: "Checks for unsafe blocks (found 1, justified)"
  → Reports findings to parent

Agent 3 (ML):
  Implements: model loading + inference
  Trail of Bits activates:
    • insecure-defaults: "Model files loaded from filesystem without verification"
    • sharp-edges: "No validation of model input dimensions"
  → Reports findings to parent

PARENT SESSION:
  Reviews all security findings
  Spawns quick Agent 4: Security Remediation
    • Adds rate limiting to API
    • Adds input/output sanitization
    • Adds model file integrity checks

  Runs differential-review on remediation
  → Confirms no new vulnerabilities introduced
```

**Result:** Security-first mindset applied at team level, not after-the-fact.

---

### The Feedback Loop With Agent Teams

```
Development Layer (Interactive + Autonomous)
    ↓ generates
Agent Team Sessions
    ├─ Agent 1 work: [JSONL + observations]
    ├─ Agent 2 work: [JSONL + observations]
    ├─ Agent 3 work: [JSONL + observations]
    └─ Parent coordination: [JSONL + team decisions]
    ↓ feeds
Feedback Layer captures:
    ├─ Claude Diary:
    │  ├─ When agent teams were spawned
    │  ├─ Task size/complexity metrics
    │  ├─ Dependencies between agents
    │  └─ Time saved vs. sequential execution
    │
    ├─ Transcripts:
    │  ├─ Full trace of parent session
    │  └─ Individual agent session artifacts
    │
    └─ Plunderer:
       ├─ "Spawn N agents for M-hour features"
       ├─ "Parallelizable tasks pattern recognition"
       └─ "Context7 library discovery across agents"
    ↓ feeds
/reflect (weekly)
    ├─ Pattern: "Feature with N semi-independent components → spawn N agents"
    ├─ Learning: "Claude-Mem effectiveness: 78% of agent context came from memory"
    ├─ Rule: "Always define dependency DAG before spawning agents"
    └─ Updates CLAUDE.md with new heuristics
    ↓ feeds
Next Agent Team Sessions
    ├─ Better spawning decisions (based on learned patterns)
    ├─ Faster agent startup (Claude-Mem provides context)
    └─ Improved coordination (based on past success/failure modes)
```

---

## Part 3: Learning Path for Agent Teams

### Prerequisites (Already Done)

You've completed:
- ✅ Superpowers brainstorm → plan → execute → verify
- ✅ Claude-Mem cross-session memory
- ✅ Context7 library documentation
- ✅ Trail of Bits security skills
- ✅ Feedback loop (diary, transcripts, plunderer)

**You're ready to learn agent teams right now.** No additional tooling needed.

---

### Level 1: Single Specialized Agent (30 minutes)

**Goal:** Understand how to request a specialized agent for a task.

#### Do This

1. Start a Claude Code session:
   ```bash
   claude code /path/to/project
   ```

2. Try this prompt (exact phrasing matters for agent spawning):
   ```
   I need to add comprehensive error handling to src/handlers/auth.go.

   Spawn a specialized agent to:
   - Review current error handling patterns
   - Identify error-prone paths
   - Add structured logging with context IDs
   - Write tests for error cases

   Report back with:
   - Changed files
   - Test results
   - Security implications (use Trail of Bits sharp-edges)
   ```

3. Observe:
   - Does Claude spawn a visible agent?
   - Can you see the agent's work in real-time?
   - Does it report back to you?
   - Does it use your stack tools (Context7, Trail of Bits)?

#### Success Criteria

- [ ] Agent spawns and reports starting
- [ ] Agent completes task and returns with evidence
- [ ] Trail of Bits skill activated on error handling code
- [ ] You understand the agent's decision-making process

#### Key Insight

A single specialized agent can be more focused than having yourself juggle multiple concerns in a single session. The agent inherits your stack context automatically.

---

### Level 2: Two-Agent Coordination (45 minutes)

**Goal:** Learn how to express dependencies between agents.

#### Do This

1. Start with a feature that spans two domains:
   ```
   I'm building a feature: WebSocket real-time updates.

   It involves:
   - Server: adding WebSocket endpoint (Go + gorilla/websocket)
   - Client: adding reconnection logic (React)

   Spawn TWO agents:

   AGENT 1 (Backend):
   - Design WebSocket protocol (message types, error codes)
   - Implement Server endpoint
   - Write integration tests
   - Report: "Protocol spec and server ready"

   AGENT 2 (Frontend, depends on Agent 1):
   - Wait for Agent 1's protocol spec
   - Implement client with reconnection + heartbeat
   - Write integration tests against server
   - Report: "Client complete, e2e tests passing"

   Both agents should use Context7 for library docs and Trail of Bits for security review.
   ```

2. Observe:
   - Does Agent 1 finish before Agent 2 starts?
   - Does Agent 2 reference Agent 1's work?
   - Are both agents using your stack tools?
   - What gets logged to Claude-Mem?

3. After completion, check:
   ```bash
   # Review Claude-Mem observations
   curl http://localhost:37777/api/observations

   # Look for entries like:
   # "Agent 1 defined WebSocket protocol with X message types"
   # "Agent 2 implemented client with Y reconnection strategies"
   ```

#### Success Criteria

- [ ] Agent 1 completes first
- [ ] Agent 2 waits for Agent 1's output (not starting in parallel)
- [ ] Both agents reference shared code/docs correctly
- [ ] Claude-Mem captures dependency relationship
- [ ] Tests for both components pass

#### Key Insight

Dependencies matter. Expressing them explicitly (vs. hoping agents coordinate) prevents wasted work and contradictory implementations.

---

### Level 3: Three-Agent Parallelization (1 hour)

**Goal:** Launch parallel agents that have no dependencies.

#### Do This

1. Pick a feature with three truly independent components:
   ```
   Feature: Comprehensive observability dashboard

   Three independent components:
   - Metrics collector (backend)
   - Dashboard UI (frontend)
   - Alert rule engine (backend service)

   These can be built in parallel because:
   - Each has its own test suite
   - They integrate via API contracts (already defined)
   - No code dependencies between them

   Spawn THREE agents IN PARALLEL:

   AGENT 1 (Metrics Collector):
   - Implement Prometheus endpoint
   - Add instrumentation hooks
   - Write unit + integration tests

   AGENT 2 (Dashboard UI):
   - Fetch /metrics endpoint specification
   - Build React dashboard consuming metrics
   - Write component + integration tests

   AGENT 3 (Alert Rules):
   - Implement CEL-based rule engine
   - Add rule evaluation + notification
   - Write rule test suite

   All three can start immediately (no ordering required).
   ```

2. Time the execution:
   ```bash
   # Before spawning agents:
   time_start=$(date +%s)

   # [Spawn agent team]

   # After agents complete:
   time_end=$(date +%s)
   duration=$((time_end - time_start))

   echo "Agent team completed in ${duration}s"
   ```

3. Compare to sequential:
   - Estimate: "If I did these 3 sequentially, ~${duration}s × 3 ≈ $(($duration * 3))s"
   - Actual savings: Sequential would take ~$(($duration * 3))s
   - Parallelism win: ~$(($duration * 2))s saved

4. Review Claude-Mem for parallelism patterns:
   ```bash
   curl http://localhost:37777/api/observations | jq '.[] | select(.text | contains("Agent")) | .text'
   ```

#### Success Criteria

- [ ] All 3 agents start immediately (not sequentially)
- [ ] All 3 agents complete in roughly the same time (not serialized)
- [ ] Each agent's tests pass independently
- [ ] You can measure actual time savings vs. sequential
- [ ] Claude-Mem captures parallelism decisions

#### Key Insight

Parallel agents are the whole point. You save time by executing independent work simultaneously, not by having smarter individual agents.

---

### Level 4: Agent Teams + Feedback Loop (1.5 hours)

**Goal:** Close the loop — learn when agent spawning was effective.

#### Do This

1. Complete a multi-agent feature from start to finish:
   - Interactive session: brainstorm + plan (you or lead agent)
   - Autonomous: spawn agent team for implementation
   - Verification: review, merge, commit

2. At session end, run:
   ```
   We just completed [feature] using a 3-agent team.

   Summarize for my memory:
   - Feature scope (lines of code, components touched)
   - Agent team composition (3 agents: [names])
   - Duration (actual time from spawn to completion)
   - Estimated sequential duration (if done without parallelism)
   - Time saved from parallelism
   - Was agent spawning the right call? Why or why not?
   - What would improve next time?
   ```

3. Check Claude-Mem captures this:
   ```bash
   curl http://localhost:37777/api/observations | tail -20
   ```
   Should see recent entries about the agent team session.

4. Next session, check what Claude-Mem remembers:
   ```
   What do you remember about how we handled multi-component features?
   ```
   Should reference the agent team pattern from previous session.

5. After accumulating 3–4 agent team sessions, run `/reflect`:
   ```bash
   /reflect
   ```
   Check the reflection output for patterns like:
   - "Agent teams effective for features with N independent components"
   - "Parallel agent execution saved ~X% time on multi-component work"
   - Updated CLAUDE.md with new rules

#### Success Criteria

- [ ] Completed 1 multi-agent feature end-to-end
- [ ] Session captured in diary + Claude-Mem
- [ ] Claude-Mem successfully remembers agent team pattern in next session
- [ ] `/reflect` identifies agent team effectiveness pattern
- [ ] CLAUDE.md updated with new heuristic

#### Key Insight

The feedback loop turns agent teams from a cool feature into a learned practice. Over time, Claude automatically gets better at deciding when to spawn agents.

---

### Level 5: Integration with Your Existing Workflow (2 hours)

**Goal:** Make agent teams part of your daily practice, not a separate skill.

#### Do This

1. **Update your CLAUDE.md with agent team heuristics:**
   ```markdown
   ## Multi-Component Features

   - For features spanning N independent components (N ≥ 2):
     - Single brainstorm/plan session
     - Spawn N-agent team for implementation
     - Each agent uses Superpowers (TDD, verification)
     - All agents share Claude-Mem + Context7 + Trail of Bits

   - Estimate parallelism savings:
     - 2 agents: ~1.5x faster than sequential
     - 3 agents: ~2.5x faster than sequential
     - 4+ agents: diminishing returns (coordination overhead)

   - Dependency DAG:
     - Explicit ordering prevents wasted work
     - Define contracts before agent spawning
     - Use "wait for Agent X" syntax
   ```

2. **Create a slash command for agent team spawning:**
   ```markdown
   <!-- ~/.claude/commands/spawn-team.md -->
   ---
   description: Spawn a multi-agent team for a feature
   ---

   To spawn an agent team:

   1. Have completed brainstorming + planning
   2. Identify N independent components or domains
   3. Define dependencies between agents (DAG)
   4. For each agent, specify:
      - Component/domain
      - Success criteria (tests passing, etc.)
      - Dependencies (if any)
      - Stack tools to use (Context7, Trail of Bits, etc.)

   Example:
   ```
   Spawn THREE agents:

   AGENT 1 (API Layer):
   - Define REST endpoints [success: tests pass, OpenAPI generated]
   - No dependencies

   AGENT 2 (Database):
   - Implement persistence [success: all queries working]
   - Depends on Agent 1 (knows API contracts)

   AGENT 3 (Security Review):
   - Review changes from Agents 1-2 [success: no regressions]
   - Runs in parallel (uses differential-review + Trail of Bits)
   ```

   All agents will automatically:
   - Access your CLAUDE.md methodology
   - Use Claude-Mem for cross-agent context
   - Call Context7 for library docs
   - Trigger Trail of Bits on sensitive code
   ```

3. **Use agent teams for multi-component work:**

   ```bash
   # Single session, single prompt:
   claude code ~/project

   > I'm building feature X spanning API, Engine, ML.
   > Spawn three agents:
   > - Agent 1: API (defines contracts)
   > - Agent 2: Engine (depends on Agent 1)
   > - Agent 3: ML (depends on Agent 2)
   > Coordinate via Claude-Mem, report when done.
   ```

4. **Test the integration:**
   - Use agent teams on your next multi-component feature
   - Verify all stack tools activate (Context7, Trail of Bits)
   - Check Claude-Mem for captured decisions
   - Run `/reflect` after 3–4 features
   - Review updated CLAUDE.md

#### Success Criteria

- [ ] CLAUDE.md updated with agent team heuristics
- [ ] Custom slash command created and tested
- [ ] Level 8 workflow integrated with agent teams
- [ ] First multi-agent feature completed using new approach
- [ ] All stack tools (Context7, Trail of Bits, Claude-Mem) integrated

#### Key Insight

Agent teams become your default pattern for multi-component work, not a special feature you learn separately. They're woven into your methodology (Superpowers), your memory (Claude-Mem), your docs (Context7), and your security (Trail of Bits).

---

## Part 4: Common Patterns & Recipes

### Pattern 1: Independent Parallel Agents

**When to use:** Components with no interdependencies.

```
AGENT 1: Feature A
AGENT 2: Feature B
AGENT 3: Feature C

All spawn at t=0
All complete at roughly t=T
Timeline: T (vs. 3T if sequential)
```

**Example:** Add logging to module A, add caching to module B, add monitoring to module C.

---

### Pattern 2: Ordered Dependencies

**When to use:** Sequential work with clear handoffs.

```
AGENT 1: Define API contracts
  Report: REST endpoints specified, OpenAPI generated

AGENT 2: Implement server (depends on Agent 1)
  Input: Agent 1's API spec
  Report: Server running, tests passing

AGENT 3: Implement client (depends on Agent 1)
  Input: Agent 1's API spec
  Report: Client working, e2e tests passing

Timeline: T_api + T_server + T_client (agents 2 & 3 in parallel)
         = T_api + max(T_server, T_client)
         < T_api + T_server + T_client (fully sequential)
```

---

### Pattern 3: Parallel Review

**When to use:** Security/quality concerns during implementation.

```
AGENT 1-N: Implementation agents
  (do work in parallel)

AGENT N+1: Security Review (parallel to others)
  (monitors changes, runs Trail of Bits)
  Report: No regressions, security findings [list]

AGENT N+2: Quality Review (parallel to others)
  (runs Superpowers verification, tests)
  Report: All tests passing, coverage >X%
```

**Benefit:** Catch issues early; don't wait until the end to discover security gaps.

---

### Pattern 4: Multi-Language Coordination

**When to use:** Polyglot projects (Go API, Python ML, React frontend).

```
AGENT 1: Go API
  Uses Context7 with /gin-gonic/gin
  Uses Trail of Bits for HTTP security

AGENT 2: Python ML
  Uses Context7 with /scikit-learn/scikit-learn
  Uses Trail of Bits for data handling

AGENT 3: React Frontend
  Uses Context7 with /react/react
  Uses Trail of Bits for XSS/injection

AGENT 4: Integration
  Tests cross-language API contracts
  Validates type coercion (JSON, protobuf, etc.)
  Uses differential-review for blast radius
```

**Benefit:** Each agent specializes in one language/ecosystem; Claude-Mem prevents version/API mismatches.

---

### Pattern 5: Autonomous Agent Team (Fire-and-Forget)

**When to use:** Well-defined work, high confidence in completion.

```
Morning: Brainstorm + plan (interactive)
  Produce: Design doc, task breakdown, success criteria

Afternoon: Spawn autonomous agent team
  Agents run without supervision
  Claude handles coordination automatically
  You come back later to review

Evening: Review + merge
  Check: Tests pass? Security review done? Code quality OK?
  Integrate: Merge branches, commit results
```

**Setup:**
```
cd ~/project
claude code . << 'EOF'
Read docs/plan-YYYY-MM-DD.md

Spawn autonomous agent team:
- Agent 1: [task 1]
- Agent 2: [task 2]
- Agent 3: [task 3]

Report when complete with evidence (test output, commit hashes).
EOF

# [While agents run, go do something else]

# Later:
claude code . << 'EOF'
Review the work from the agent team that just ran.
Are all tests passing? Security review clean? Ready to merge?
EOF
```

---

## Part 5: Integration Patterns with Your Stack

### With Superpowers

```
Superpowers:brainstorm (interactive)
  ↓
Superpowers:write-plan (interactive)
  ↓
Spawn agent team (autonomous)
  Each agent uses Superpowers:test-driven-development
  Each agent uses Superpowers:verification-before-completion
  ↓
Parent session: Superpowers:requesting-code-review
  Review agent team output
  Verify completion criteria met
```

---

### With Claude-Mem

```
Session 1: Work on feature A with agents
  Agents create observations in Claude-Mem:
    - "API uses gin with custom middleware"
    - "Engine uses tokio::sync with buffer 1000"
    - "Decision: JSON over protobuf (frontend team preference)"

Session 2: Work on feature B
  Claude-Mem injects past decisions
  New agent team sees: "Use gin, tokio, JSON"
  Avoids re-deciding, builds on past context
  Adds new observations: "ML agents prefer scikit-learn for inference"

/reflect (weekly)
  Finds pattern: "Multi-component features use agent teams"
  Finds pattern: "Decision consistency improves with Claude-Mem"
  Updates CLAUDE.md with heuristics
```

---

### With Context7

```
Agent 1 (Go):
  "Implement API middleware using context7 with /gin-gonic/gin"
  → Fetches current Gin docs

Agent 2 (Rust):
  "Use context7 with /tokio-rs/tokio for channel bounds"
  → Fetches current Tokio docs

Agent 3 (Python):
  "Use context7 for /scikit-learn/scikit-learn model integration"
  → Fetches current sklearn docs

Guarantee: All agents working with CURRENT library versions
No version mismatches, no hallucinated APIs.
```

---

### With Trail of Bits

```
During implementation:
  Agent 1: Implements API endpoint
    Trail of Bits sharp-edges: "Missing input validation"
    Reports: "Fix needed: add request body validation"

  Agent 2: Implements data processing
    Trail of Bits differential-review: "State mutation in critical path"
    Reports: "Review: blast radius analysis"

  Agent 3: Implements security review
    Trail of Bits insecure-defaults: "Model files loaded without integrity check"
    Reports: "Issue: add model verification"

Parent session:
  Collects all findings
  Spawns remediation agent
  Runs differential-review on fixes
  Confirms no new vulnerabilities
```

---

### With Feedback Loop

```
Development Layer (Interactive + Autonomous)
    ↓ generates
Agent Team Sessions (new signal!)
    ├─ When agents were spawned (task size, complexity)
    ├─ Agent team composition (2–4 agents typical)
    ├─ Parallelism achieved (actual vs. theoretical)
    ├─ Time saved vs. sequential
    └─ Context7 + Trail of Bits + Claude-Mem usage
    ↓ feeds
Claude Diary:
    └─ Captures: "Used 3-agent team for feature X, saved ~2 hours"

Transcripts:
    └─ Preserves: Full session trace of parent + agent decisions

Plunderer:
    └─ Extracts: "Spawn agent patterns" → suggests future agent team use

/reflect:
    ├─ Pattern: "N-component features → spawn N agents"
    ├─ Learning: "Agent teams 2.3x faster than sequential"
    ├─ Update CLAUDE.md: "For multi-component >3 hours, spawn agents"
    └─ Next session: Claude auto-suggests agent teams earlier
```

---

## Part 6: Common Mistakes & How to Avoid Them

### Mistake 1: Spawning Too Many Agents

**Problem:**
```
Spawn 7 agents for a single feature
Each agent now has to coordinate with 6 others
Communication overhead explodes
Agents spend more time waiting than working
```

**Solution:**
- Keep agent team size ≤ 4 for most features
- Group related work under a single agent
- Example: "One agent for all API endpoints" not "one agent per endpoint"

---

### Mistake 2: Not Defining Dependencies

**Problem:**
```
Spawn Agent B (depends on Agent A)
But don't tell Claude "Agent B depends on Agent A"
Agent B starts immediately, uses stale/missing contracts
Agent B's work becomes invalid when Agent A finishes
Wasted effort
```

**Solution:**
```
AGENT 1: API Contracts
  Define: POST /predict, GET /status
  Report: "Contracts defined"

AGENT 2: Server (DEPENDS ON AGENT 1)
  Wait: For Agent 1 to finish
  Input: Agent 1's contracts
  Implement: Handlers matching contracts
  Report: "Server running"
```

---

### Mistake 3: Ignoring Claude-Mem Context

**Problem:**
```
Agent 1 discovers: "We need a specific version of library X"
Adds to code: `"x": "^1.2.3"`

Agent 2 doesn't see this decision
Adds to its code: `"x": "^2.0.0"` (breaking change)

Agents conflict
Integration fails
```

**Solution:**
- Make decisions explicit at session start
- Use Claude-Mem by running "Remember: we use library version X.Y.Z"
- Agents read Claude-Mem context automatically
- Consistency enforced

---

### Mistake 4: Weak Success Criteria

**Problem:**
```
Agent 1: "Implement API"
  What counts as "done"? Undefined.
  Agent might leave incomplete tests, missing docs, TODO comments

Agent 2: Depends on Agent 1
  Tries to integrate against incomplete API
  Failure
```

**Solution:**
```
Agent 1: "Implement API"
  Success criteria:
    - All endpoints have tests (100% coverage for routes)
    - OpenAPI spec generated
    - Handlers validate input per spec
    - Rate limiting configured
  Report: "API complete per spec [link], ready for Agent 2"
```

---

### Mistake 5: Not Using Trail of Bits During Parallelism

**Problem:**
```
Agents 1-2 implement in parallel
Security review comes after (Agent 3)
By then, 2 agents' worth of mistakes already committed
Harder to fix
```

**Solution:**
```
Agent 3: Security Review (PARALLEL to Agents 1-2)
  Monitors: Git commits from Agents 1-2 in real-time
  Triggers: Trail of Bits differential-review on each commit
  Reports: "Issue found by Agent 1, fix needed"

Agents can iterate while security feedback is fresh
Final merge is clean
```

---

## Part 7: When NOT to Use Agent Teams

Agent teams are powerful, but not always the right choice.

### Use Manual Superpowers Instead When:

1. **Exploring new territory**
   - You don't know the design yet
   - Need back-and-forth refinement
   - Can't clearly specify agent tasks

2. **High-risk code**
   - Payment processing, auth, cryptography
   - Need constant human oversight
   - Can't justify autonomous execution

3. **Single-component work**
   - Task is small (<2 hours)
   - Task is single-domain
   - No parallelism benefits

4. **Real-time feedback needed**
   - Debugging complex issues
   - Teaching Claude new domain
   - Making architectural decisions

---

### Use Subagent-Driven Development (Supervised) Instead When:

1. **Staged delegation within single session**
   - You design, then delegate implementation
   - You review each agent's output before next phase
   - Not fully autonomous, but structured

---

## Part 8: Integration Checklist

Use this checklist to integrate agent teams into your stack:

### Prerequisites
- [ ] Superpowers installed and practiced (Levels 1-5)
- [ ] Claude-Mem configured and working (Level 3)
- [ ] Context7 installed and MCP connected (Level 4)
- [ ] Trail of Bits skills installed (Level 5)
- [ ] Feedback loop running (diary + cron + /reflect)

### Learning
- [ ] Level 1: Spawn single agent for specialized task
- [ ] Level 2: Spawn two agents with dependencies
- [ ] Level 3: Spawn three+ agents in parallel
- [ ] Level 4: Complete feature using agent teams + feedback loop
- [ ] Level 5: Integrate agent teams into daily workflow

### Integration
- [ ] Update CLAUDE.md with agent team heuristics
- [ ] Create /spawn-team slash command
- [ ] Update Level 8 (multi-component) workflow
- [ ] Test agent teams on real multi-component feature
- [ ] Test integration on real multi-component feature

### Feedback
- [ ] Diary capturing agent team decisions
- [ ] Claude-Mem storing agent patterns
- [ ] /reflect identifying effective agent team use
- [ ] CLAUDE.md auto-updating with learned heuristics
- [ ] Next session benefits from agent team learnings

### Monitoring
- [ ] Run `claude-status` to verify stack health
- [ ] Check Claude-Mem observations for agent patterns
- [ ] Review git history for agent-generated commits
- [ ] Track time savings vs. sequential execution

---

## Part 9: Quick Reference

### Agent Team Prompt Template

```
I'm building [feature name] spanning [N] components/domains.

This feature breaks into these semi-independent components:
- [Component 1]: [brief description of responsibility]
- [Component 2]: [brief description of responsibility]
- [Component 3]: [brief description of responsibility]

Component dependencies:
- [Component 2] depends on [Component 1] (requires: [interface/output])
- [Component 3] depends on [Component 2] (requires: [interface/output])

Spawn [N] agents to build this in parallel:

AGENT 1 ([Component Name]):
- Responsibility: [what this component does]
- Specific work:
  * [Task 1]
  * [Task 2]
  * [Task 3]
- Success criteria: [concrete: tests pass, API spec generated, etc.]
- No dependencies (can start immediately)
- Use Context7 for: [library name]
- Use Trail of Bits for: [security concern]

AGENT 2 ([Component Name]):
- Responsibility: [what this component does]
- Specific work:
  * [Task 1]
  * [Task 2]
  * [Task 3]
- Success criteria: [concrete: tests pass, validates against Agent 1, etc.]
- Depends on: Agent 1 (wait for [specific output])
- Use Context7 for: [library name]
- Use Trail of Bits for: [security concern]

AGENT 3 ([Component Name]):
- Responsibility: [what this component does]
- Specific work:
  * [Task 1]
  * [Task 2]
  * [Task 3]
- Success criteria: [concrete: tests pass, e2e validated, etc.]
- Depends on: Agent 2 (wait for [specific output])
- Use Context7 for: [library name]
- Use Trail of Bits for: [security concern]

All agents will:
- Use Superpowers test-driven-development (write tests first, then implementation)
- Access CLAUDE.md for project conventions and methodology
- Access Claude-Mem for cross-session context and learned patterns
- Report back with concrete evidence when complete (test results, git commits)

Coordinate and report when all agents finish their work.
```

---

### Heuristics for Agent Team Decisions

```
DO spawn agents when:
✓ Feature spans 2-4 semi-independent components
✓ Components have clear interfaces/contracts
✓ Total work >3 hours
✓ Work can be parallelized
✓ You have well-defined success criteria
✓ Security/quality review needed

DON'T spawn agents when:
✗ Feature is single-component (<3 hours)
✗ Components are tightly coupled
✗ You're still exploring design
✗ High-risk code (payment, auth, crypto)
✗ Success criteria are vague
✗ You need constant real-time feedback
```

---

### Integration Points (Copy-Paste)

**Add to your CLAUDE.md:**
```markdown
## Agent Teams for Multi-Component Work

When building features spanning N independent or semi-ordered components:

1. Brainstorm + plan (interactive, with Superpowers)
2. Spawn N-agent team (autonomous):
   - Each agent specializes in one component
   - Define dependencies explicitly (DAG)
   - All agents use: TDD, Context7, Trail of Bits, Claude-Mem
3. Parent session: Verify results, merge, integrate

Time savings:
- 2 agents: ~1.5x faster than sequential
- 3 agents: ~2.5x faster than sequential
- 4+ agents: diminishing returns (coordination overhead)

Libraries + versions (for Claude-Mem):
- [your choices here]
```

**Create ~/.claude/commands/spawn-team.md:**
```markdown
---
description: Spawn multi-agent team for complex features
---

Use this command when you have a multi-component feature ready for autonomous build.

Required: Completed brainstorm + plan document
Pattern: Define agents, dependencies, success criteria
Result: Autonomous parallel implementation

[Copy template from Part 9 above]
```

---

## Part 10: Troubleshooting

### Agent Not Spawning

**Symptom:** You ask Claude to spawn agents, but it doesn't.

**Causes:**
1. Claude doesn't recognize the spawn request (phrasing matters)
2. Task is too simple (Claude decides single agent better)
3. Agent teams feature not available (old Claude Code version)

**Fix:**
1. Use explicit phrasing: "Spawn N agents: Agent 1, Agent 2, Agent 3"
2. Make task larger (combine related work)
3. Check Claude Code version: `claude --version` (update if needed)

---

### Agents Working in Sequence, Not Parallel

**Symptom:** Agents A, B, C should run parallel but Agent B waits for A to finish.

**Causes:**
1. You specified "Agent B depends on Agent A" (creates ordering)
2. Claude misinterpreted task as sequential
3. Agents have subtle dependency you didn't notice

**Fix:**
1. Remove "depends on" if agents are truly independent
2. Explicitly state: "These 3 agents can work in parallel (no dependencies)"
3. Review what Agent A outputs that Agent B uses; if nothing, remove dependency

---

### Claude-Mem Not Shared Across Agents

**Symptom:** Agent 2 doesn't see observations from Agent 1.

**Causes:**
1. Claude-Mem worker not running (webhook down)
2. Agents in separate sessions (they should be in same session!)
3. Claude-Mem not configured in settings.json

**Fix:**
1. Check: `curl http://localhost:37777/api/readiness`
2. Verify agents are in same parent session (not spawned as new CLI calls)
3. Verify `settings.json` has claude-mem configured

---

### Trail of Bits Not Activating in Agents

**Symptom:** You ask agents to use sharp-edges, but they don't.

**Causes:**
1. Skills not installed (`/plugin list` doesn't show them)
2. Agent prompt doesn't explicitly request the skill
3. Code doesn't match skill's trigger conditions

**Fix:**
1. Check: `/plugin list` (reinstall if missing)
2. Explicitly ask: "Run your sharp-edges skill on [file]"
3. Ensure code does match concern (e.g., error handling for sharp-edges)

---

## Conclusion: Why Agent Teams Matter

Agent teams are the primary mechanism for autonomous multi-component work in your stack. They close the loop on your feedback system:
- Superpowers methodology scales to multi-agent teams
- Claude-Mem makes agents smarter by sharing context
- Context7 keeps all agents using current library docs
- Trail of Bits applies security-first thinking to all agents
- Feedback loop learns when agent spawning works best

**Over weeks of use:**
- Claude learns your decomposition patterns
- /reflect identifies what makes good multi-component work
- CLAUDE.md evolves with agent team heuristics
- Each new feature spawns agents more effectively than the last

Agent teams are the primary mechanism for orchestrating autonomous multi-component work in Claude Code.

---

## Next Steps

1. **This week:** Complete Levels 1–2 (single + two-agent basics)
2. **Next week:** Complete Levels 3–4 (parallelism + feedback loop)
3. **Following week:** Integrate into Level 8 workflow with agent teams
4. **Ongoing:** Use `/reflect` to evolve agent team patterns

Welcome to multi-agent orchestration. 🚀
