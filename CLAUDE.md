# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

A Claude Code **skill** for studying and practicing the Claude Certified Architect (Foundations) exam. It is invoked via:

```
/claude-certification
```

There is no build, lint, or test pipeline — this is purely a markdown-based skill consumed by Claude Code's skill system.

## Skill Architecture

```
.claude/skills/claude-certification/
├── SKILL.md              # Orchestration: session modes, question generation rules, distractor bank
└── resources/
    ├── CCA_Domain1.md    # Agentic Architecture & Orchestration (27%)
    ├── CCA_Domain2.md    # Tool Design & MCP Integration (18%)
    ├── CCA_Domain3.md    # Claude Code Configuration & Workflows (20%)
    ├── CCA_Domain4.md    # Prompt Engineering & Structured Output (20%)
    └── CCA_Domain5.md    # Context Management & Reliability (15%)
```

**SKILL.md** is the orchestration layer — it defines the two session modes (Explanation, Exam), language preference handling, per-domain question ratios (~4 D1, ~3 each D2/D3/D4, ~2 D5 per 15 questions), and reusable distractor patterns.

**Domain files** each follow an identical internal structure:
1. Teaching Structure (depth-adaptive concept explanations with production examples)
2. Task Statements (5–7 concepts each with: explanation → exam traps → check questions → connection forward)
3. Practice Scenarios
4. Domain Completion (practice exam spec + build exercises)

## Exam Content Constraints

When modifying or adding content, maintain these properties that the exam rewards:

- **Deterministic over probabilistic** — when stakes are high, deterministic solutions are always preferred
- **Proportionate fixes** — the simplest fix that solves the root cause, not an over-engineered rewrite
- **Root cause tracing** — questions require identifying *where* in the pipeline a failure originates, not just symptoms
- **Low-effort/high-leverage** — exam questions test whether candidates pick minimal interventions first

## Distractor Patterns

The shared distractor bank in SKILL.md defines canonical wrong-answer patterns used across all domains:
- Probabilistic fix when deterministic is needed
- Fixing a downstream component when the coordinator is the root cause
- Over-engineering a solution that requires a simple config change
- Common MCP/tool configuration mistakes

When writing new practice questions, distractors must be *plausible* (reflect real misunderstandings) not obviously wrong.
