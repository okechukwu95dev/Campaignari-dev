# AgentScope: engineering handoff to a coding agent

Last updated: 2026-11-02. Build a generic, local-first open-source prototype against a **synthetic** demo repo, not an internal company codebase. No existing project secrets or data are necessary.

## Product thesis

A read-only flight recorder and observability console for Claude Code. Given one or more Claude Code sessions, explain the *observable* context, token economics, tool timeline, subagent execution graph and repeated investigation waste. Add a reviewed memory-consolidation experiment (Memory Lab) as a second-phase feature.

Out of scope: viewing private model thoughts, recording a literal latent 'brain', control over coding agents, cloud upload of transcripts, automatic instruction rewriting, unrestricted hooks/MCP orchestration and actual cost savings claims without benchmarking. Optional 3D office is a separate visual renderer, not a source of truth.

## MVP user story

An engineer runs a synthetic multi-service repo task: 'An expected inbound file is missing; why didn't downstream reconciliation complete?' Claude spawns three investigators. AgentScope draws a parent/child graph and live activity, lets a user select a child to see its tool events, files and token counts, shows context hot spots, and replays why a route was useful or wasteful. AgentScope itself performs **no model calls**.

## Security constraints (mandatory)

- Read-only transcript ingestion. Do not mutate `~/.claude`, edit user settings, create hooks, run shell commands from UI, or invoke MCP tools.
- Bind to `127.0.0.1`. Default to no outbound network calls or telemetry; disable automatic link previews in transcript views. Explicitly avoid logging OAuth/API tokens.
- Support a `metadataOnly` mode: no prompt text, code, file contents or tool outputs persisted or served. Redact or hash paths by default; sensitive transcript content is untrusted.
- No external GitHub sync on work machines. Never commit real transcripts, CLAUDE local memory, application source fragments, environment files or credentials.
- Separate only public synthetic demo data from internal datasets; check organizational policy before running a third-party parser on corporate transcripts.
- Strictly distinguish 'measured', 'derived' and 'estimated' fields in schema and UI.
- Show local process memory/CPU overhead; provide a low-frequency polling option, no GPU requirement for core MVP.

## Architecture

```text
Claude Code sessions + subagent transcripts (~/.claude/projects/.../*.jsonl)
      | approved local read-only collector
      v
JSONL line parser -> normalizer -> event deduplication -> usage attribution
      |                       |
      v                       v
SQLite (optional)      in-memory live updates (SSE on localhost)
      |                       |
      +-----------> React/TS dashboard <----------+
                       | live sessions / graph / turns / token brain
                       +-- Memory Lab (phase 2: read-only inventory + staged proposals)
                       +-- optional Three.js agent-office view (phase 3)
```

Use a small Node.js + TypeScript service, React frontend, AG Grid for large event tables, and React Flow for execution topology. SQLite optional in early MVP (memory only / indexed JSONL offsets acceptable). Browser accesses only localhost. Make a generic synthetic incident app. No dependence on Spotify Portal.

## Canonical event model

```ts
export type EvidenceClass = 'measured' | 'derived' | 'estimated';
export type AgentScopeEvent = {
  id: string;                         // stable hash of source + file offset + message identity
  sessionId: string;
  parentSessionId?: string;
  agentId?: string;
  parentAgentId?: string;
  timestamp: string;
  eventType: 'user'|'assistant'|'tool_start'|'tool_result'|'subagent_start'|'subagent_end'|'compaction'|'system';
  model?: string;
  tool?: { name: string; callId?: string; outputBytes?: number; filePathHash?: string };
  usage?: {
    input: number;
    output: number;
    cacheRead: number;
    cacheWrite: number;
    thinking?: number; // null unless explicitly available in a supported record
    evidence: EvidenceClass;
  };
  rawTranscriptLocation: { sourceFile: string; line: number };
  redactedSummary?: string;
};
```

## Token economics rules

- Use transcript usage fields as the **source of truth** for per-request token classes, not `characters / 4` when exact fields exist.
- Beware streaming/update records repeating the same message/usage ID. Never sum duplicate snapshots as separate invocations; dedupe by stable message ID + request identity and use the latest complete usage record for that request.
- Track model separately: pricing varies with model, date, cache-read vs cache-creation (possibly 5-min/1-hour variants), input and output. Use a versioned pricing table and label all dollar values **API-equivalent estimates** for Pro/Max/Enterprise seats; actual subscription task dollars are not observable.
- Sum *usage per request* across an agent, not successive estimates of current context size. Context occupancy at a point in time is not the same number as cumulative tokens burned.
- Repeated reads/file and tool attribution are reconstructed estimates, not provider-certified prices of that file. Label them accordingly, expose the estimation method and avoid false precision.
- Do not claim to expose private reasoning text. Show any actual recorded thinking-token count only when present; otherwise say 'not available'.
- Prompt-cache read/write values are separate classes with different economics; never show a single 'tokens used' pie as a proxy for cost or context.

## UI views and acceptance criteria

1. **Session list**: project, start/end, root prompt metadata, status, model, subagent count, requests, token classes, estimate. Status must be derived, never invented.
2. **Context Brain**: current window occupancy where available; estimated category composition (instructions, history, file reads, tool output) with provenance and per-item inspect; distinguish unknown/unattributed portion.
3. **Token Costs**: chronological per-turn input/output/cache read/cache write; model prices versioned, API-equivalent estimate not billable subscription spend. Drill into dominant tools and size of outputs.
4. **Agent Lobby**: parent/child graph, active/ended, name, last action, elapsed time, requests, cost; click to open agent-specific timeline and transcript pointer. Support multiple parallel children.
5. **Replay**: deterministic timeline playback of a recorded fixture with play/pause/scrub, no model requests.
6. **Hotspots**: list repeated file reads, oversized tool output, repeated errors, excessive speculative branches; suggestions must be explicitly labeled heuristics.
7. **Memory Lab (phase 2)**: candidates with evidence/session refs, conflicts, freshness, context-budget size, diff, proposed vs approved vs archived status. No auto promotion.
8. **3D office (optional)**: reuse same events in simplified 3D avatars. Do not let animations imply an agent is thinking/acting if no telemetry indicates that. Disable by default.

## Implementation milestones

- M0: create synthetic JSONL fixtures: one root, three subagents, duplicate streaming usage, empty usage, old/new transcript format, malformed line, huge tool output, tool failure, compaction event, and secrets. Snapshot expected summaries.
- M1: parser and deduplication library; unit tests for token totals, current context distinction, event order, trace IDs.
- M2: localhost API, `/health`, `/sessions`, `/sessions/:id/events`, `/sessions/:id/agents`, `/sessions/:id/usage` and server-sent event updates. Validate schema; restrict CORS.
- M3: React UI + React Flow + token charts + AG Grid; replay fixture and one real **personal** test session.
- M4: benchmark harness for baseline vs selective Haiku exploration vs Shunt-inspired routing; correctness gate and cost/time stats.
- M5: Memory Lab proposals + reversible promotion + change ledger + held-out test score.
- M6: optional Roost-style 3D visualization *only if* the core monitor works and resource usage is acceptable.

## Engineering tasks for the internal coding agent

- Read `01_FULL_DEEP_RESEARCH.md` and `02_CLAUDE_DREAMS_MEMORY_RESEARCH.md` first. Verify linked upstream repositories/official docs yourself before adopting source code.
- Initialize a standalone TypeScript monorepo under a distinct path; keep this research bundle separate from company code.
- First implement `packages/ingest` with a file reader that never writes to inputs, injectable fixture providers and a normalized event schema.
- Implement a deduped per-request accounting engine. Provide tests proving cache reads/writes aren't counted as fresh input, and message snapshots aren't counted twice.
- Add a security test ensuring a sentinel string in raw tool output never reaches metadata-only API responses, logs, storage or UI.
- Add a replayable deterministic synthetic incident fixture.
- Build dashboard with a functional agent graph and drilldown. Keep 3D entirely optional.
- Document measured-vs-estimated provenance for every number displayed.
- Add a one-command local demo script and verify zero outbound calls. Collect CPU/RAM readings on Windows and Linux/WSL; don't invent benchmark numbers.
- Present a PR-style architecture memo, all tests, a limitations section and screenshots generated from synthetic data.

## Definition of done for first team demo

One command boots a local dashboard with an entirely synthetic but realistic three-agent debugging replay. The user can locate the highest-cost tool output and show which agent generated it, view actual/estimated token-class accounting with provenance, see a repeated read, and inspect a proposed memory fix without affecting the agent. No network request, API key, extra LLM call or real company transcript is required to replay the fixture.

## Evaluation and ecosystem references

- https://github.com/kchernev/claude-code-monitor
- https://github.com/nateherkai/token-dashboard
- https://github.com/MixMe/claude-code-usage-dashboard
- https://github.com/karamanburak/agent-monitor
- https://github.com/sorryhumans/roost
- https://github.com/AgentSystemLabs/agent-office
- https://github.com/spotify/portal-ai-plugins/tree/main/plugins/shunt
- https://code.claude.com/docs/en/memory
- https://platform.claude.com/docs/en/managed-agents/dreams