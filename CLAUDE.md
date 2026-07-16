# gmail-mcp-server

MCP server exposing Gmail (multi-account) to AI agents/assistants: read, write, archive, label, unsubscribe. TypeScript, built on `@modelcontextprotocol/sdk` + `googleapis`, served over Express.

## Commands

- `npm run dev` — run with hot reload (tsx watch)
- `npm run build` — compile TypeScript (tsc)
- `npm start` — run the compiled server (`dist/index.js`)

Source lives in `src/`.

## Model routing: Fable 5 orchestrator, subagent executors

This project uses a cost-optimized delegation pattern instead of running every step on the top-tier model:

- **Orchestrator (main conversation)** — run Claude Code with **Fable 5** selected (`/model`). Fable's job is judgment calls and delegation, not typing out boilerplate itself. It decides what needs deep reasoning vs. what's mechanical, and hands off accordingly.
- **`deep-reasoner` subagent** (pinned to **Opus**) — architecture/design tradeoffs, root-causing tricky bugs (esp. OAuth/token/multi-account edge cases), security-sensitive changes (credential handling, scopes), anything where a wrong call is expensive.
- **`fast-worker` subagent** (pinned to **Sonnet**) — implementation once the approach is decided: new tool handlers, boilerplate, tests, refactors, running the build.

Rules of thumb:
1. Default to `fast-worker` for any well-specified implementation task.
2. Escalate to `deep-reasoner` only for genuine judgment calls — not for volume.
3. The orchestrator (Fable) should delegate mechanical work rather than doing it inline, to conserve its budget for planning and review.
4. Run `/agents` to inspect, edit, or add subagents defined in `.claude/agents/`.

See `.claude/agents/deep-reasoner.md` and `.claude/agents/fast-worker.md` for the subagent definitions.
