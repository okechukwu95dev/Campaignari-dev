# Claude Dreams and memory consolidation: verified findings and project implications

Research date: 2026-11-02. Scope: generic development tooling. No internal repositories, credentials, session transcripts or business information are included.

## Executive conclusion

Add **Memory Lab** as an optional, explicitly reviewed module of AgentScope. It should turn completed investigation sessions into evidence-linked, versioned *candidate* knowledge rather than silently rewriting `CLAUDE.md` or the agent's standing instructions. The first version can operate fully locally on recorded transcripts and structured, deterministic extractors; an optional Claude Code synthesis pass uses the user's existing plan quota. Do not assume 'dreaming' is free. Measure whether memory reduces repeated investigation work and factual errors on held-out tasks.

## Three things called 'Claude Dreams' must not be confused

| System | Verified basis | Availability / cost | Data handling and behavior |
| --- | --- | --- | --- |
| **Anthropic Managed Agents Dreams** | Official platform documentation | Research preview, API / Managed Agents, requires access and beta headers `managed-agents-2026-04-01` + `dreaming-2026-04-21`; separate from the ordinary Claude Code subscription | Consumes an existing memory store and 1-100 managed-agent sessions; produces a **new** memory store, leaving input unchanged; supports inspect/review/adopt/discard. Docs explicitly say **not zero-data-retention eligible** and **not HIPAA eligible**. |
| **Claude Code local auto memory** | Official Claude Code documentation | Available in ordinary local Claude Code sessions; no separate paid service required. Model-backed memory operations may consume plan allowance. | Automatic project-scoped markdown notes in `~/.claude/projects/<project>/memory/`; `MEMORY.md` is auto-loaded up to first 200 lines or 25KB, and linked topic files load on demand. `/memory` reviews/toggles. |
| **AutoDream internal/local consolidation** | Reverse-engineering reports and user issue reports; not fully specified in current official memory documentation | Reported background consolidation mechanism in some Claude Code versions. Exact triggers/commands/version behavior **not a stable documented public contract**. | Reports describe consolidation of older memory from past sessions, but reliability issues and inconsistent `/dream` availability have been filed. Do not depend on an undocumented trigger. |

**Important documentation update:** Current Claude Code docs say `AGENTS.md` can be loaded natively in supported newer versions when no applicable project `CLAUDE.md` is present. Older articles claiming Claude Code never reads `AGENTS.md` are version-dependent and outdated. Use `CLAUDE.md` for universally applicable Claude-specific instructions, or configure the project instruction loading policy explicitly.

## Official Dreams API details

The managed-agent Dream API takes an input `memory_store` and 1-100 managed-agent sessions, runs a model-driven asynchronous consolidation, and creates a distinct output memory store. It includes usage reporting and an event-traceable underlying session; it does not train model weights. It is not equivalent to a local Claude Code plugin or `CLAUDE.md` edit. Its research-preview data handling restrictions make it an inappropriate starting point for a regulated-workflow experiment until compliance review.

## Local 'dreaming' alternatives

1. **Microsoft SkillOpt-Sleep** — https://github.com/microsoft/SkillOpt/tree/main/plugins/claude-code . Documented pipeline: harvest local transcripts -> mine recurring tasks -> replay tasks using a model/backend -> propose bounded edits to `CLAUDE.md`/`SKILL.md` -> score on held-out tasks -> stage and require explicit adopt (auto adopt is opt-in). Interesting because it tries to prove an improvement, not just compress text. **Caveat:** replay and synthesis involve model calls, consume subscription quota or API budget, and read sensitive transcript history. Inspect code and data flow before considering work use.
2. **cg1262/dreaming-skill** — https://github.com/cg1262/dreaming-skill . Non-destructive local skill for Claude Code/Codex that reads session history + memory, writes a timestamped proposal and literal diff, and requires manual promotion. **Caveat:** model synthesis is neither free nor experimentally validated merely because it produces a good-looking memory file.
3. **Claude Code native auto memory** — most conservative first choice. `/memory` lets the user inspect current memory, and `CLAUDE.md` remains the appropriate place for hand-curated, enduring instructions. No additional plugin is necessary to start.

## Failure modes to design against

- **Stale facts**: an old project assumption can be repeated for weeks. Include observed-at dates and independently verified evidence; never overwrite current architecture from a vague summary.
- **Cross-project contamination**: enforce repo/workspace scope and source provenance. A shared home-directory session collection is not automatically a single knowledge domain.
- **Truncation and inflated permanent context**: `MEMORY.md` has a first-200-lines/25KB load cap; keep it a sparse pointer index to topical notes. Make current index size, load budget, and age visible.
- **Hallucinated confidence**: each new candidate fact must cite source session IDs and source file/tool references where appropriate; label conclusions unverified until evaluated.
- **Unsafe promotion**: conversational observations are not security policies. Rules, permissions, infrastructure operations and credentials require human and/or institutional review. Memory instructions do not replace technical permission enforcement.
- **Secrets in transcripts**: the collector must not export content, code fragments or credentials to a public dashboard. Prefer metadata-only indexing, explicit redaction and no network access.
- **Memory poisoning**: log data and agent text are untrusted inputs. Treat synthesized memories as proposals, not automatically executable instructions.
- **Cost illusion**: a dream may save future tokens while costing more than it saves today; a measured benefit needs follow-up tasks.
- **Low usage does not mean low value**: infrequently used safety instructions may be critical. Never prune solely by low access frequency.

## Proposed AgentScope Memory Lab

### Collection (deterministic; no model calls)

- Index only local, approved paths; do not ingest whole corporate chat archives by default.
- Build an episode record for each session: timestamps, project, root request, files/tools touched, major decisions, tests, failures, verified outcome, token/latency summary and transcript pointers.
- Preserve source provenance as immutable hashes/IDs. Scrub secret-like values before storing optional textual excerpts.
- Detect repeated failures, repeated instructions, contradicted paths/configuration, forgotten decisions, and repeated file-search trajectories by deterministic signatures before any LLM call.

### Candidate memory schema

```json
{
  "id": "mem_c72a",
  "scope": {"repository": "demo/synthetic-app", "agent_role": "investigator"},
  "kind": "project_fact",
  "statement": "Scheduler config points to inbound-file readiness check before reconciliation",
  "confidence": "verified",
  "observed_at": "2026-11-02T12:00:00Z",
  "expires_after": null,
  "evidence": [{"session_id":"demo-0007","event_id":"event-102","file":"config/schedule.yaml"}],
  "conflicts_with": [],
  "sensitivity": "internal",
  "status": "proposed",
  "version": 1
}
```

Use `kind` from a strict vocabulary, e.g. decision, preference, verified project fact, troubleshooting runbook, correction, reference pointer. Keep a field for the validation procedure. Do not promote arbitrary Markdown into a live instruction file.

### Consolidation (optional, quota-consuming)

- On demand or on a user-approved schedule, cluster related episode records. Model calls, if enabled, receive only the minimum approved metadata and redacted extracts.
- Generate *proposed* candidates: add, merge, supersede, archive, and why. A model may suggest a change but cannot directly modify the live memory.
- Verify syntactic facts (file exists, command valid, current config value, provenance points to a real event) using deterministic checks and a read-only sandbox.
- Display a before/after diff, source evidence, contradictions, freshness, context-token delta and downstream test results.
- Human approve/reject/rollback. Keep the previous memory version and append an audit event.

### Context budget and retrieval

`MEMORY.md` should remain a short **index**, never a transcript dump. Use topic files and on-demand retrieval. UI should show:

- Always-loaded memory size, in bytes and estimated tokens, with warnings near 200 lines / 25KB.
- Last updated, conflict count, proposed versus approved count, and evidence links.
- Whether each entry was *actually recalled* in later sessions, not merely stored.
- Saved work on the next similar task: lower exploration time, fewer repeated reads, fewer corrections, and quality-adjusted tokens.

### Quality gate

Benchmark three setups on a synthetic repo with 8-12 recurring incident types:

- A: default Claude Code memory, unchanged.
- B: memory index + reviewed factual notes.
- C: B plus proposed dream consolidation accepted only after validation.

For each setup run multiple fresh sessions on held-out incidents not used in creating the memory. Measure resolution accuracy, unsupported assertions, time to correct root cause, tool calls, cache classes, input/output tokens, context bytes and estimated API-equivalent cost. Evaluate memory *precision* (supported facts), *recall* (relevant facts surfaced), *staleness* (contradictory facts carried forward), *privacy violations* (must be zero), and score differences with uncertainty intervals. Avoid using a single anecdote as proof.

## Work-machine recommendation

Start with local, read-only metadata for analysis plus human-authored project instructions. Do **not** enable the Managed Agents Dreams API for internal data by default: it is an API research preview with explicit data-handling limitations. Do not install automatic transcript-harvesting, replaying or self-editing skills on a managed work machine without review. If the team values memory synthesis, build the review and evidence layer first, then introduce an opt-in local consolidation backend behind a strict allowlist.

## Source references

- Anthropic official Claude Code memory: https://code.claude.com/docs/en/memory
- Anthropic official Managed Agents Dreams: https://platform.claude.com/docs/en/managed-agents/dreams
- Anthropic official Claude Code plan usage: https://support.claude.com/en/articles/14552983-models-usage-and-limits-in-claude-code
- Microsoft SkillOpt-Sleep source: https://github.com/microsoft/SkillOpt/blob/main/plugins/claude-code/skills/skillopt-sleep/SKILL.md
- Microsoft SkillOpt-Sleep design: https://github.com/microsoft/SkillOpt/blob/main/docs/superpowers/specs/2026-06-07-skillopt-sleep-claude-code-plugin-design.md
- Local dreaming skill: https://github.com/cg1262/dreaming-skill
- Reverse-engineering report (unofficial): https://github.com/agenmod/decode-claude-code
- Claude Code issue: `/dream` missing in some versions: https://github.com/anthropics/claude-code/issues/38426
- Claude Code issue: inaccurate and opaque auto-dream outputs: https://github.com/anthropics/claude-code/issues/38493
- Claude Code issue: memory index truncation: https://github.com/anthropics/claude-code/issues/57574
- Claude Code issue: stale auto memory: https://github.com/anthropics/claude-code/issues/85075
- Claude Code issue: project scoping/cross-contamination: https://github.com/anthropics/claude-code/issues/91738
- Claude Code issue: access-aware pruning: https://github.com/anthropics/claude-code/issues/40806
- Claude Code issue: promote essential rules into persistent instructions: https://github.com/anthropics/claude-code/issues/59904