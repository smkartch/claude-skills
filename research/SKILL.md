---
name: research
description: Phase 1 of the agentic coding workflow. Conducts deep research on a topic using multiple parallel agents plus a critic pass, then produces a research document for human review. No code is written.
---

The user has asked you to research a topic in preparation for a coding task. Your job is to produce a thorough research document by orchestrating multiple agents — parallel researchers gather findings from different angles, then a critic agent pushes back on the synthesis before you finalize.

Do **not** propose solutions, write plans, or implement anything. Your only output is a research document.

## Step 1 — Confirm scope with the user

Before spawning agents, restate the topic in one or two sentences and list the angles you intend to research in parallel. Ask the user to confirm or adjust. Keep this short — a single message, not a back-and-forth.

## Step 2 — Spawn parallel research agents

Send a single message with multiple `Agent` tool calls (so they run concurrently). Use 2–3 agents — more is rarely worth the token cost.

Default angles (adjust to fit the topic):
- **Existing code & patterns** — what already exists in the codebase that touches this area; relevant files, line numbers, and patterns in use.
- **Dependencies & interfaces** — upstream/downstream callers, external services, contracts that constrain the design.
- **Constraints, edge cases, gotchas** — historical decisions, known bugs, performance/security/compliance constraints, anything subtle.

Brief each agent like a colleague who hasn't seen the conversation. Each prompt must be self-contained:
- State the overall research topic and why it matters.
- State the specific angle this agent owns and what's out of scope for them.
- Tell them to return a structured report (findings with file paths and line numbers, open questions, references) — not a recommendation.
- Cap their report length so synthesis stays manageable (e.g., "under 500 words").

Use `subagent_type: "Explore"` for read-only code surveys; use `general-purpose` when the agent needs broader reasoning.

## Step 3 — Synthesize a draft

Collect the agents' reports and write a draft document at:

```
<git-repo-root>/doc/work-sessions/<yyyy>/<yyyy-mm-dd_hh-mm-ss>-research-<terse-description>.md
```

- `<git-repo-root>` — from `git rev-parse --show-toplevel`
- `<yyyy-mm-dd_hh-mm-ss>` — from `date '+%Y-%m-%d_%H-%M-%S'`
- `<terse-description>` — short hyphen-separated topic slug (e.g. `user-auth-flow`)

### Document structure

1. **Topic** — one-sentence statement of what was researched.
2. **Context** — why this matters; what problem it relates to.
3. **Findings** — bulk of the document, organized by sub-topic. Include file paths and line numbers, dependencies and interfaces, constraints and gotchas, related patterns already in use.
4. **Open Questions** — things that remain unclear and should be resolved during planning.
5. **References** — external docs, files, or prior decisions consulted.

## Step 4 — Critic pass

Spawn one critic agent with the draft document. Brief it to:
- Identify gaps, unverified claims, and missing context.
- Flag findings that don't cite a file path or line number.
- Call out anything that reads like a recommendation rather than a fact (this is a research doc, not a proposal).
- Push back on conclusions that feel premature or under-evidenced.
- Return a punch list — under 300 words, prioritized by severity.

Use `subagent_type: "general-purpose"` and pass the draft path so the critic can read the full file.

## Step 5 — Revise

Apply valid critiques. If a critique points to a gap that needs more research, you can either:
- Investigate it yourself (preferred for narrow follow-ups), or
- Spawn a targeted research agent for it (only if the gap is substantial).

One critic round is usually enough. Run a second round only if the first round surfaced major issues — diminishing returns kick in fast.

## Step 6 — Hand off

Tell the user where the file was written and ask them to review it before proceeding to the proposal phase. Mention how many research agents ran and that the critic pass completed.

## Hard constraints

- No production or test code. No proposals. No implementation plans. Only a research document.
- Subagents do not talk to each other — every coordination step happens in the skill.
- Each subagent prompt must be fully self-contained.
