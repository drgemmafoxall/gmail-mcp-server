---
name: delegate-work
description: Consult before starting any non-trivial coding task in this repo — new tool handlers, bug fixes, refactors, tests, or architecture questions. Decides whether to delegate to a cheaper pinned subagent (fast-worker/Sonnet, deep-reasoner/Opus) instead of doing the work inline on the orchestrator model, to cut token spend. Skip only for single-line trivial edits where delegation overhead exceeds the savings.
---

# Delegate work to save tokens

This repo runs Claude Code with a top-tier model (Fable 5) as the orchestrator. The orchestrator's job is classification, planning, and review — not typing out implementation. Every non-trivial unit of work should be delegated to whichever subagent in `.claude/agents/` is cheapest for the job.

## Decision procedure

1. **Single trivial edit?** (rename, one-line fix, typo, formatting) — do it directly. Delegation overhead isn't worth it for volume this small.
2. **Genuine judgment call?** (architecture/design tradeoffs, ambiguous root cause — especially OAuth/token/multi-account edge cases — or security-sensitive changes to credentials/scopes) — delegate to `deep-reasoner`.
3. **Everything else** (new tool handlers, boilerplate, tests, refactors once the approach is decided, running the build) — delegate to `fast-worker`.
4. **Multiple independent subtasks?** — launch them as parallel Agent calls in one message rather than one at a time.

## How to delegate

Call the Agent tool with `subagent_type: fast-worker` or `subagent_type: deep-reasoner`. Brief it like a colleague with no context:
- concrete file paths / line numbers already identified
- what's already been tried or ruled out
- a clear definition of done

Don't hand an open-ended "figure this out" prompt to `fast-worker` — that belongs to `deep-reasoner` or the orchestrator itself. Don't hand mechanical volume work to `deep-reasoner` — that's what `fast-worker` is for.

## After delegation

Review the subagent's actual diff before reporting the task done. A subagent's summary describes what it intended to do, not necessarily what it did.
