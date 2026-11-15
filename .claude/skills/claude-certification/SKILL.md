---
name: claude-certification
description: Use when studying or practicing for the Claude Certified Architect (Foundations) exam or the Claude Certified Architect Professional (CCAR-P) exam across all domains, or when someone asks for certification practice questions, exam prep, or wants to test knowledge of Claude agentic systems, tool design, Claude Code, prompt engineering, context management, security architecture, evaluation design, or production API architecture.
---

# Claude Certified Architect — Exam Practice (Foundations & Professional)

## Language

**Default language is English.** Respond in English unless the user explicitly asks to switch to another language. This rule applies to all questions, feedback, and explanations throughout the session.

---

## Available Exams

| Exam | Code | Domains | Passing Score |
|---|---|---|---|
| Claude Certified Architect — Foundations | CCA-F | 5 domains | 720/1000 |
| Claude Certified Architect — Professional | CCAR-P | 5 domains | 750/1000 |

---

## CCA-F (Foundations) Domain Overview

| Domain | Topic | Weight |
|---|---|---|
| 1 | Agentic Architecture & Orchestration | 27% |
| 2 | Tool Design & MCP Integration | 18% |
| 3 | Claude Code Configuration & Workflows | 20% |
| 4 | Prompt Engineering & Structured Output | 20% |
| 5 | Context Management & Reliability | 15% |

**The foundations exam rewards:**
- Deterministic solutions over probabilistic ones when stakes are high
- Proportionate fixes (don't over-engineer)
- Root cause tracing (identify WHERE in the pipeline the failure originates)
- Low-effort, high-leverage fixes as first steps

---

## CCAR-P (Professional) Domain Overview

| Domain | Topic | Weight |
|---|---|---|
| 1 | Advanced Agentic Architecture & Reliability | 27% |
| 2 | Production API & Performance Architecture | 20% |
| 3 | Security Architecture & Trust Engineering | 18% |
| 4 | Evaluation Design & Production Quality | 20% |
| 5 | Advanced Tool & Integration Architecture | 15% |

**The professional exam rewards:**
- Architectural trade-off reasoning (knowing when each pattern applies)
- Defence-in-depth thinking (structural defences over prompt-based)
- Measurement before optimisation
- Root cause isolation in distributed systems
- Proportionate fixes (simplest intervention that solves the root cause)

---

## Session Format

### On Session Start

1. Greet the user in their language.
2. Ask: **"Which exam are you preparing for? (1) CCA-F — Foundations or (2) CCAR-P — Professional?"**
3. Once the exam is selected, ask: **"¿Prefieres el _Modo Explicación_ para estudiar un dominio o el _Modo Examen_ para probar tus conocimientos? / Do you prefer _Explanation Mode_ to study a domain or _Exam Mode_ to test your knowledge?"**
4. Route to the appropriate domain files based on exam choice (see Domain File Routing below).

### Domain File Routing

**For CCA-F (Foundations):**
- Explanation Mode: read `./resources/foundations/CCA_Domain*.md` for the selected domain
- Exam Mode: generate questions using the CCA-F key concepts (Domains 1–5 below)

**For CCAR-P (Professional):**
- Explanation Mode: read `./resources/professional/CCAR-P_Domain*.md` for the selected domain
- Exam Mode: generate questions using the CCAR-P key concepts (Domains 1–5 below)

### Explanation Mode

1. Explain to the user the purpose of the mode and the selected exam.
2. Ask: **"¿Cual dominio quieres estudiar? / Which domain do you want to study?"**
3. Read the corresponding domain file from the routing table above and follow its instructions.
4. Provide structured summaries, production examples and highlight any "exam traps".
5. The user is allowed to ask questions; identify which domain the question belongs to and answer following the domain file's instructions.

### Exam Mode

1. Explain to the user the purpose of the mode and the selected exam.
2. Ask: **"¿Quieres practicar un dominio específico o todos los dominios? / Do you want to practice a specific domain or all domains?"**
3. Ask how many questions they want (default: 15).
4. Begin immediately — no teaching upfront.

### Question Format

- Present ONE question at a time
- Scenario-based: describe a real production situation, then ask what to do
- Options A, B, C, D — one correct, three plausible distractors
- Wait for the user to answer before showing the next question
- Show question number, domain, and exam: e.g., **Question 3/15 — CCAR-P Domain 2**

### After Each Answer

Show:
1. **CORRECT ✅** or **INCORRECT ❌**
2. **Why the correct answer is right** (2–3 sentences, grounded in the concept)
3. **Why each wrong answer is wrong** — one sentence per distractor explaining the specific flaw

### After All Questions

Show:
- Final score (X/Y)
- List of questions answered incorrectly
- Offer to re-drill weak domains

---

## CCA-F — Key Concepts by Domain

### Domain 1: Agentic Architecture & Orchestration (27%)

#### Key concepts to test

**Agentic loop (1.1)**
- Correct termination: check `stop_reason` field — `"end_turn"` = done, `"tool_use"` = continue
- Anti-pattern 1: checking `if response.content[0].type == "text"` — wrong because model can return text AND tool_use simultaneously
- Anti-pattern 2: checking for natural language signals ("I'm done") — ambiguous and unreliable
- Anti-pattern 3: arbitrary iteration caps as primary stopping mechanism
- Tool results MUST be appended to conversation history before next turn

**Multi-agent orchestration (1.2)**
- Hub-and-spoke: coordinator at centre, subagents never communicate directly
- Critical isolation: subagents do NOT inherit coordinator's history. Everything must be explicit in their prompt.
- Narrow decomposition failure: root cause is always the coordinator's decomposition logic, NOT downstream agents
- The exam tests tracing failures to their origin

**Subagent invocation (1.3)**
- Coordinator's `allowedTools` must include "Task" to spawn subagents
- Parallel spawning: emit multiple Task calls in ONE coordinator response (faster)
- `fork_session`: independent branches from shared baseline
- Context passing must include structured metadata (source URLs, doc names, page numbers) for attribution

**Workflow enforcement (1.4)**
- Prompt-based guidance = probabilistic (non-zero failure rate)
- Programmatic hooks/gates = deterministic (works every time)
- Rule: financial, security, compliance → programmatic. Style/formatting → prompt is fine.
- Handoff summaries must be self-contained (human agent has no transcript access)

**SDK Hooks (1.5)**
- PostToolUse: intercepts AFTER execution, BEFORE model processes result → normalise data formats
- Tool call interception: intercepts BEFORE execution → block/redirect based on business rules
- A single compliance failure → use hook, never a prompt

**Task decomposition (1.6)**
- Fixed sequential (prompt chaining): predetermined steps, reliable, cannot adapt
- Dynamic adaptive: generates subtasks from discoveries, flexible, less predictable
- Attention dilution: too many files in one pass → inconsistent depth. Fix: per-file passes + separate cross-file integration pass

**Session management (1.7)**
- `--resume`: use when context is still valid
- `fork_session`: use for divergent approaches from same baseline
- Fresh start + summary injection: use when files changed or context is stale
- After code modifications: inform agent of SPECIFIC file changes, not re-explore everything

---

### Domain 2: Tool Design & MCP Integration (18%)

#### Key concepts to test

**Tool descriptions (2.1)**
- Descriptions are THE mechanism for tool selection (not supplementary)
- Minimal descriptions ("Retrieves customer information") cause misrouting between similar tools
- Fix for misrouting: expand descriptions. NOT few-shot examples (wrong root cause), NOT routing classifiers (over-engineered first step)
- Good description includes: purpose, input format, example queries, edge cases, explicit "use THIS vs THAT" boundaries
- System prompt keyword conflicts can override well-written descriptions

**Error handling (2.2)**
- Four error categories: Transient (retry), Validation (fix input), Business (not retryable, alternative workflow), Permission (escalate)
- Critical distinction: access failure (tool couldn't reach source → consider retry) vs valid empty result (source reached, no matches → do NOT retry)
- Error metadata: `errorCategory`, `isRetryable` boolean, human-readable description

**Tool distribution (2.3)**
- Optimal: 4-5 tools per agent, scoped to role
- `tool_choice` options: `"auto"` (default), `"any"` (must call a tool), `{"type": "tool", "name": "X"}` (force specific tool)
- Scoped cross-role tools: give synthesis agent a constrained verify_fact instead of routing through coordinator for 85% simple lookups

**MCP configuration (2.4)**
- Project-level: `.mcp.json` in repo → version-controlled, shared with team
- User-level: `~/.claude.json` → personal, NOT shared
- `${GITHUB_TOKEN}` syntax in `.mcp.json` keeps credentials out of version control
- Use existing community servers first; build custom only for team-specific workflows

**Built-in tools (2.5)**
- Grep: searches file CONTENTS. Use to find function callers, imports, error messages
- Glob: matches file PATHS. Use to find files by extension or naming pattern
- Edit: targeted modification with unique text anchors. Fallback: Read + Write for full file
- Exploration order: Grep entry points → Read to follow imports. Do NOT read all files upfront.

---

### Domain 3: Claude Code Configuration & Workflows (20%)

#### Key concepts to test

**CLAUDE.md hierarchy (3.1)**
- User-level (`~/.claude/CLAUDE.md`): only YOU, not version-controlled, NOT shared via git
- Project-level (`.claude/CLAUDE.md` or root `CLAUDE.md`): everyone, version-controlled
- Directory-level: applies when working in that specific directory only
- Exam trap: new team member not getting instructions → root cause is user-level instead of project-level config
- `/memory` command: debugging tool for inconsistent behaviour across sessions
- `@import` syntax for modular organisation; `.claude/rules/` for topic-specific rule files

**Custom commands and skills (3.2)**
- `.claude/commands/` = project-scoped, shared via git
- `~/.claude/commands/` = personal, not shared
- Skill frontmatter: `context: fork` (isolated sub-agent, verbose output stays contained), `allowed-tools` (restricts tools), `argument-hint` (prompts for parameters)
- Skills = on-demand, task-specific. CLAUDE.md = always-loaded, universal standards. Never mix these.

**Path-specific rules (3.3)**
- `.claude/rules/` files with YAML frontmatter `paths: ["**/*.test.tsx"]`
- Advantage over directory CLAUDE.md: glob patterns match files across the ENTIRE codebase
- Loads ONLY when editing matching files → token efficiency

**Plan mode vs direct execution (3.4)**
- Plan mode when: large-scale changes, multiple valid approaches, architectural decisions, multi-file modifications (45+ files)
- Direct execution when: well-understood single-file bug fix, clear limited scope
- Explore subagent: isolates verbose discovery from main conversation, returns summaries
- Common pattern: plan mode for investigation → direct execution for implementation

**Iterative refinement (3.5)**
- Concrete input/output examples beat prose descriptions every time (2-3 examples)
- Test-driven iteration: write tests first, share failures
- Interview pattern: have Claude ask questions before implementing
- Batch feedback when fixes interact; sequential when issues are independent

**CI/CD integration (3.6)**
- `-p` flag: non-interactive mode. Without it, CI job hangs waiting for input. THIS IS TESTED.
- `--output-format json --json-schema`: machine-parseable structured findings
- Same session that generated code is LESS effective at reviewing it (retains reasoning context)
- Re-reviews: include prior findings, instruct to report ONLY new/unaddressed issues

---

### Domain 4: Prompt Engineering & Structured Output (20%)

#### Key concepts to test

**Explicit criteria (4.1)**
- Vague: "be conservative." Specific: "Flag only when claimed behaviour contradicts actual code. Skip style preferences."
- High false positive rates in ONE category destroy trust in ALL categories
- Fix: temporarily disable high-FP categories while improving prompts for them
- Severity calibration requires actual CODE EXAMPLES, not prose descriptions

**Few-shot prompting (4.2)**
- Most effective technique for consistency — beats more instructions or confidence thresholds
- Deploy when: inconsistent formatting, inconsistent judgment on ambiguous cases, extraction misses existing information
- Construct: 2-4 examples, each showing REASONING for choice over plausible alternatives
- Enables generalisation to novel patterns, not just memorisation

**Structured output with tool_use (4.3)**
- `tool_use` with JSON schema eliminates syntax errors. Prompt-based JSON can produce malformed output.
- `tool_use` does NOT prevent: semantic errors (line items not summing to total), field placement errors, fabrication
- `tool_choice "any"`: must call a tool, chooses which (guaranteed structured output, unknown doc types)
- Schema: use optional/nullable fields to PREVENT fabrication. "unclear" enum for ambiguous cases. "other" + detail string for extensible categories.

**Validation-retry loops (4.4)**
- Retry with: original document + failed extraction + specific validation error
- Effective for: format mismatches, structural errors, misplaced values
- NOT effective for: information genuinely absent from source
- `detected_pattern` fields: track what triggered each finding, enables systematic prompt improvement

**Batch processing (4.5)**
- Batch API: 50% cost savings, up to 24-hour window, no latency SLA, does NOT support multi-turn tool calling
- Synchronous API: blocking workflows (pre-merge checks, developer waits for result)
- Batch API: latency-tolerant workflows (overnight reports, weekly audits)
- Exam trap: using batch for everything. Keep blocking workflows synchronous.

**Multi-instance review (4.6)**
- Same session reviewing its own output = less effective (retains reasoning context, less likely to question decisions)
- Independent instance without prior context catches more subtle issues
- Per-file local passes + separate cross-file integration pass = prevents attention dilution
- Confidence-based routing: low-confidence findings → human review. Calibrate with labelled validation sets.

---

### Domain 5: Context Management & Reliability (15%)

#### Key concepts to test

**Context preservation (5.1)**
- Progressive summarisation trap: compresses specific values ("$247.83 for order #8891") into vague summaries
- Fix: persistent "case facts" block with transactional facts. Include in every prompt. Never summarise it.
- "Lost in the middle" effect: models process beginning and end reliably, middle may be missed. Fix: key summaries at the START, explicit section headers.
- Trim verbose tool results to relevant fields BEFORE appending to context
- Upstream agent optimisation: return structured key facts, not verbose reasoning chains

**Escalation and ambiguity (5.2)**
- Valid triggers: explicit human request (honour IMMEDIATELY, no investigation first), policy gaps, inability to make progress
- Invalid triggers: sentiment/frustration, self-reported confidence scores
- Frustration nuance: if issue is straightforward + customer is frustrated → acknowledge + offer resolution. Only escalate if customer REITERATES human preference after you offer help.
- Ambiguous customer match: ask for additional identifiers. Do NOT select based on heuristics.

**Error propagation (5.3)**
- Structured error context: failure type, what was attempted, partial results, alternative approaches
- Anti-pattern 1: silent suppression (empty result marked success) → prevents any recovery
- Anti-pattern 2: workflow termination on single failure → throws away partial results
- Access failure vs valid empty result (same as Domain 2 — reinforced across domains)
- Coverage annotations: note which findings are well-supported vs which areas have gaps

**Codebase exploration (5.4)**
- Context degradation: agent references "typical patterns" instead of specific classes → context filled with discovery output
- Mitigation: scratchpad files (write findings to file), subagent delegation, summary injection before next phase, `/compact`
- Crash recovery: each agent exports structured state to manifest file; coordinator loads manifest on resume

**Human review and confidence (5.5)**
- 97% overall accuracy can hide 40% error rate on a specific document type
- Always validate by document type AND field segment before automating
- Stratified random sampling of high-confidence extractions detects novel error patterns
- Field-level confidence calibration with labelled validation sets

**Information provenance (5.6)**
- Each finding needs: claim + source URL + document name + relevant excerpt + publication date
- Two conflicting credible sources: annotate BOTH with attribution. Do NOT arbitrarily select one.
- Temporal awareness: different dates explain different numbers (not contradictions)
- Rendering: financial data → tables, news → prose, technical findings → structured lists

---

## CCAR-P — Key Concepts by Domain

### Domain 1: Advanced Agentic Architecture & Reliability (27%)

**Fault-tolerant loops (1.1)**
- Retry budgets with exponential backoff + jitter; finite, not infinite
- Circuit-breaker pattern: after N consecutive failures, open circuit and degrade gracefully
- `max_tokens` stop reason = dangerous truncation; never surface partial output as final result
- Resume from last checkpoint, not from scratch; preserve completed tool results in persistent store

**Hierarchical agent networks (1.2)**
- Context budget allocation: explicitly limit context passed to each subagent
- Scope partitioning: coordinator assigns non-overlapping scopes; overlapping subtasks cause duplicate work
- Cross-agent isolation: subagent outputs are UNTRUSTED inputs to the coordinator
- Implicit dependency detection: tasks sharing an output dependency must be sequenced, not parallelised

**Agent memory architectures (1.3)**
- Working memory: active context (finite, managed)
- Episodic memory: persistent session summaries injected as structured prefix
- Semantic memory: vector-searchable knowledge base; retrieved on-demand, never dumped wholesale
- Anti-pattern: injecting the full episodic log into every session context

**Checkpointing and crash recovery (1.4)**
- Checkpoint at stage boundaries, not individual tool calls
- Store structured state, not raw conversation history
- Resumption must skip completed stages (idempotency at stage boundaries)
- Never re-run stages with side effects (payments sent, emails dispatched)

**Timeout and resource management (1.5)**
- Per-operation timeouts at the tool level (HTTP timeout alone does not catch internal hangs)
- Cascading timeout failure: downstream timeouts must be shorter than upstream timeouts
- Token budget as primary resource limit; wall-clock timeout as safety net only

**Parallel execution strategies (1.6)**
- Fan-out/fan-in: latency = max(subagents), not sum
- Speculative execution: launch multiple approaches, use first to succeed, cancel rest
- Concurrency gate (semaphore/token bucket): prevents rate limit saturation from simultaneous fan-out
- Recursion depth limit: prevents exponential spawning from dynamic subagent creation

**Observability and debugging (1.7)**
- Correlation IDs propagated through all subagent calls
- Structured log events, NOT raw conversation histories (PII risk + noise)
- Span-based tracing at agent boundaries
- Alert on statistical anomalies (token usage, tool call counts, duration) vs baseline

---

### Domain 2: Production API & Performance Architecture (20%)

**Prompt caching strategy (2.1)**
- Cache hit requires EXACT same prefix; any change to preceding tokens invalidates everything after
- Stable content first (system prompt, tools, static docs), dynamic content last (user query, timestamps)
- Dynamic values in system prompt = cache miss on every request
- Warm-up requests at deployment time to populate cache before real traffic

**Context window management (2.2)**
- Progressive summarisation: compress OLD turns into "prior context" block; never drop initial task
- Rolling window anti-pattern: dropping oldest messages (FIFO) loses original task specification
- "Lost in the middle": place critical instructions at START and END, not middle
- Pre-plan context budget across pipeline stages before building

**Streaming architecture (2.3)**
- Never execute tool calls on partial `input_json_delta` events; accumulate until `content_block_stop`
- Backpressure: bounded queues with consumer-side flow control
- PostToolUse hooks fire after execution, before result enters streaming context

**Batch API and throughput (2.4)**
- 50% cost reduction, up to 24-hour window, no latency SLA, no multi-turn support
- Use for latency-tolerant, high-volume, single-turn tasks; NEVER for blocking workflows
- Results arrive out of submission order; always key by `custom_id`

**Token optimisation (2.5)**
- Output tokens cost ~5x input tokens; verbosity drives most cost
- Prune tool descriptions to only tools needed in each pipeline stage
- Measurement before optimisation; define quality floor before cutting costs
- Model downgrade: always prototype and measure quality on a sample first

**Rate limiting and backpressure (2.6)**
- Exponential backoff with jitter; fixed delay creates thundering herd on retry
- TPM limit hit = documents too large; chunk first
- Bounded queues with backpressure; unbounded queues grow until OOM

---

### Domain 3: Security Architecture & Trust Engineering (18%)

**Prompt injection defences (3.1)**
- Structural defence: XML tag delimiters between instructions and user content
- Indirect injection via retrieved documents: validate tool outputs before passing to next agent
- Least-privilege: agents with no write tools cannot exfiltrate via tool calls
- Canary tokens in system prompt detect unexpected echoing

**PII handling (3.2)**
- PII minimisation: request minimum PII required for the task
- Tokenisation: replace real PII with tokens in working context; store map in secure store
- Never log raw conversation histories containing PII
- Output filtering as last-resort safeguard, not primary defence

**Trust boundaries (3.3)**
- Trust hierarchy: system prompt > user message > tool output > subagent result
- Treat subagent outputs as user-level, not system-level content
- Validate ALL data crossing a trust boundary (schema + range + anomaly)
- Encryption alone does not solve trust; authentication + validation also required

**Permission models (3.4)**
- Per-tenant scoped credentials in multi-tenant systems (not shared admin credentials)
- Dynamic permission scoping at session start by operator, not requestable by agent
- Permission escalation attempts = security signal; block and log
- Three levels: tool-level, data-level, action-level

**Audit logging (3.5)**
- Log: structured events, pseudonymised IDs, tool calls (which tool + params), timestamps
- Never log: raw user messages with PII, full conversation histories, credentials
- Immutable append-only audit logs (tamper-evident)
- GDPR right to erasure: impossible if raw PII is in append-only logs; prevent at ingestion

**Red-teaming (3.6)**
- Five attack categories: direct jailbreak, indirect jailbreak, injection via tool output, data exfiltration, privilege escalation
- Automated red-teaming at scale (separate adversary Claude instance); 50 manual prompts is insufficient for production
- Canary evaluation: embed known test cases in production traffic; alert on regression
- Red-team suite grows continuously as new attack vectors are discovered

---

### Domain 4: Evaluation Design & Production Quality (20%)

**Eval design methodology (4.1)**
- Gold standard dataset required for accuracy measurement; without it, only consistency is measurable
- Segment accuracy by input type; overall accuracy hides segment failures
- Minimum 50–100 examples per segment; smaller sets have too-wide confidence intervals
- Eval contamination: eval examples must not appear in prompts

**LLM-as-judge (4.2)**
- Judge must receive: original input + agent output + evaluation criteria + scoring rubric
- Never use same model AND same system prompt as both agent and judge (shared biases)
- Calibrate judge against human-labelled set; below 80% agreement = unreliable
- Positional bias: run A vs B AND B vs A comparisons; average scores

**Production monitoring (4.3)**
- Monitor p95/p99 latency, not just p50
- Canary requests: synthetic test cases with known expected outputs embedded in production
- Alert taxonomy: P1 (safety failures), P2 (quality regressions), P3 (efficiency drift)
- Output length distribution shift = signal for prompt injection or verbosity regression

**A/B testing (4.4)**
- Minimum weekly cycle to cover full input distribution
- Pre-define rollback trigger before the test begins
- Measure multiple metrics simultaneously (quality, cost, latency)
- Gradual rollout: 1% → 5% → 20% → 50% → 100% with monitoring between stages

**Failure taxonomy (4.5)**
- Hallucination → add citation requirement + validate against source
- Non-compliance → clarify instruction + add few-shot examples
- Tool misuse → improve tool descriptions + explicit routing instruction
- Truncation → increase max_tokens + instruct model to signal if truncated
- Refusal → check if system prompt triggers safety filters unintentionally

**Quality-cost trade-offs (4.6)**
- Optimisation ladder: verbosity → prune tools → prompt caching → batch API → model switch
- Never switch models without first measuring quality on a sample at the new model
- Routing patterns: cheap classifier → route complex queries to capable model
- Quality floor defined before optimisation begins; never trade below it

---

### Domain 5: Advanced Tool & Integration Architecture (15%)

**MCP server design (5.1)**
- Strict schemas: `additionalProperties: false`, all required fields declared
- Idempotency keys on all write operations; retries must be safe
- Document full error contract in tool descriptions; undocumented errors cause poor handling
- Versioning: breaking schema changes require version bump; non-breaking additions do not
- Tool discoverability: include real input examples in descriptions

**Tool composition patterns (5.2)**
- Atomic tools: one operation per tool; no "mega tools"
- PostToolUse hooks for format transformation between tools (not wrapper tools)
- Stateful chains: agent manages sequencing and precondition logic, not embedded in tools
- Exam trap: embedding precondition checks inside tools makes them non-atomic

**Event-driven integration (5.3)**
- Use event timestamps, not arrival order, to sequence events
- Idempotency keys for exactly-once processing of webhook events
- Dead letter queues for events that fail after N retries
- Pull vs push: push (webhook) has lower latency; pull (polling) is simpler

**Cross-system data consistency (5.4)**
- Saga pattern: sequence of local transactions with compensating transactions on failure
- Two-phase commit alternative for cross-system operations
- Read-your-writes consistency: know when your own write may not be immediately visible
- Cache invalidation on write, or TTL appropriate to staleness tolerance

**Tool performance profiling (5.5)**
- Measure first; identify the bottleneck tool before optimising
- Cache stable-output tools only; never cache live data tools (balance, inventory)
- Parallelisation ceiling: total latency = latency of slowest critical-path tool
- Dependency graph: map before parallelising; tools with shared dependencies cannot run in parallel

---

## CCA-F Question Generation Guidelines

When generating CCA-F questions:

1. **Always scenario-based** — describe a real production situation (agent behaving unexpectedly, CI breaking, misrouted tool call, etc.)
2. **One clearly correct answer** — grounded in the concepts above
3. **Three plausible distractors** — use the specific wrong patterns the exam favours:
   - The probabilistic fix when a deterministic one is needed
   - The downstream component when the coordinator is the root cause
   - The over-engineered solution when a simple fix is correct
   - Conflating access failure with valid empty result
   - User-level config when project-level is needed
4. **Cover all domains proportionally** over a full session: ~4 D1, ~3 D2, ~3 D3, ~3 D4, ~2 D5 per 15 questions
5. **Vary difficulty**: some questions test direct recall, others require multi-step reasoning

## CCAR-P Question Generation Guidelines

When generating CCAR-P questions:

1. **Always scenario-based** — enterprise or production context, higher complexity than CCA-F
2. **One clearly correct answer** — often requires trade-off reasoning, not just pattern recall
3. **Three plausible distractors** using the specific wrong patterns:
   - Prompt-based fix when a structural/programmatic fix is needed (security domain)
   - Optimisation without measurement first
   - Model switch before exhausting lower-risk options
   - Cache applied to live data
   - Shared credentials in a multi-tenant context
   - Full conversation history logged (PII risk)
   - Eval score aggregated without per-segment breakdown
4. **Cover all domains proportionally**: ~4 D1, ~3 D2, ~3 D3, ~3 D4, ~2 D5 per 15 questions
5. **Professional-level difficulty**: favour questions requiring multi-step reasoning and trade-off analysis

---

## CCA-F Distractor Bank

| Situation | Correct answer | Common distractors |
|---|---|---|
| High-stakes compliance failure | Programmatic hook/gate | Enhanced system prompt, few-shot examples, routing classifier |
| Agent terminates prematurely | Check `stop_reason` | Check content type, add iteration cap, parse natural language |
| Multi-agent coverage gap | Coordinator decomposition | Web search subagent, synthesis subagent, retrieval subagent |
| Tool misrouting | Expand tool descriptions | Add routing classifier, merge tools, add few-shot |
| New team member not getting instructions | Move to project-level CLAUDE.md | Update user-level, add to skills, use /memory |
| CI pipeline hangs | Add `-p` flag | Add `--timeout`, add `--non-interactive`, use batch API |
| Tool returns empty array, agent retries | Treat as valid empty result, no retry | Retry with backoff, escalate immediately, check permissions |
| Synthesis report has no attribution | Fix context passing (add structured metadata) | Fix synthesis agent prompt, fix web search agent, fix coordinator |

## CCAR-P Distractor Bank

| Situation | Correct answer | Common distractors |
|---|---|---|
| Agent pipeline crashes mid-run, loses work | Resume from checkpoint (skip completed stages) | Restart from scratch, replay conversation history, re-run all stages |
| Cache miss on every request despite stable prompt | Move dynamic content (timestamp) out of system prompt | Increase cache TTL, switch to larger model, use Batch API |
| Rate limit 429 from burst of parallel subagents | Add concurrency gate (semaphore) with exponential backoff + jitter | Increase rate limit quota, reduce number of subagents, use fixed delay retry |
| Prompt injection via retrieved document | Structural defence (XML delimiters + validate tool outputs) | Prompt instruction to ignore injections, refuse all document processing |
| Multi-tenant agent using admin DB credentials | Per-tenant scoped credentials | Encrypt queries, add prompt instructions, use read-only credentials |
| PII in audit logs (GDPR erasure problem) | Strip PII before logging (structured events only) | Delete audit logs on request, encrypt PII in logs |
| 95% overall accuracy but 65% on key segment | Segment eval by input type, fix coverage | Switch to larger model, add more training examples, increase temperature |
| Order created in system A; system B fails | Saga pattern with compensating transaction (cancel system A) | Two-phase commit, retry system B indefinitely, ignore system B failure |
