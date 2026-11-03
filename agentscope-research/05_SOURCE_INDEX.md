# Source index and confidence classification (2026-11-02)

## Primary / official

- Anthropic Claude Code memory: https://code.claude.com/docs/en/memory — `CLAUDE.md`/`AGENTS.md`, auto memory, scope, 200 lines / 25KB index limits, subagent behavior, audit and settings.
- Anthropic Managed Agents Dreams: https://platform.claude.com/docs/en/managed-agents/dreams — research-preview API, 1-100 sessions, cloned output store, usage and data-handling restrictions.
- Anthropic Claude Code official CLI reference: https://code.claude.com/docs/en/cli-reference
- Anthropic Claude Code best practices: https://www.anthropic.com/research/claude-code-best-practices
- Anthropic subscription usage: https://support.claude.com/en/articles/14552983-models-usage-and-limits-in-claude-code
- Spotify engineering claim: https://engineering.atspotify.com/2026/9/portal-by-spotify-cut-my-claude-code-token-usage-by-90
- Spotify Shunt actual plugin: https://github.com/spotify/portal-ai-plugins/tree/main/plugins/shunt

## Original social threads (not independently verified in full)

- https://x.com/mylifcc/status/2116337357133615224
- https://x.com/builderleader/status/2096587825445474560

## Third-party tool repositories: read their current licenses and code before use

- https://github.com/kchernev/claude-code-monitor — live read-only observability, optional usage-network endpoint; `--no-network` documented.
- https://github.com/nateherkai/token-dashboard — local prompt/file/tool analytics and SQLite index.
- https://github.com/MixMe/claude-code-usage-dashboard — Rust/static usage reporting.
- https://github.com/karamanburak/agent-monitor — hooks plus real-time agent trace; hooks require security review.
- https://github.com/kkilictepe/PoolCoder — transcript-based monitoring.
- https://github.com/adamclark64/ccmon — subagent-focused terminal UI.
- https://github.com/sorryhumans/roost — 3D read-only visual office.
- https://github.com/AgentSystemLabs/agent-office — 3D interactive orchestration; larger trust boundary.
- https://github.com/emrehangorgec/tokenbill — context attribution estimates, CLI.

## Memory consolidation and quality-gating research

- https://github.com/microsoft/SkillOpt/blob/main/plugins/claude-code/skills/skillopt-sleep/SKILL.md — staged, held-out-validated sleep pipeline; optional model-backed replay.
- https://github.com/microsoft/SkillOpt/blob/main/docs/superpowers/specs/2026-06-07-skillopt-sleep-claude-code-plugin-design.md — detailed design.
- https://github.com/cg1262/dreaming-skill — non-destructive manual local consolidation skill.
- https://github.com/agenmod/decode-claude-code — unofficial reverse engineering of AutoDream; not a stable product specification.

## Reports of unresolved/version-specific issues (treat as reports, not validated platform behavior)

- https://github.com/anthropics/claude-code/issues/38426 — `/dream` command inconsistent/missing.
- https://github.com/anthropics/claude-code/issues/38493 — concerns about auto-dream accuracy and auditability.
- https://github.com/anthropics/claude-code/issues/57574 — `MEMORY.md` truncation at index cap.
- https://github.com/anthropics/claude-code/issues/85075 — stale memories.
- https://github.com/anthropics/claude-code/issues/91738 — possible cross-project memory scoping issues in affected versions.
- https://github.com/anthropics/claude-code/issues/40806 — value of access-aware pruning.
- https://github.com/anthropics/claude-code/issues/59904 — distinction between auto memory and persistent instructions.

## Evidence caution

The imported original deep research is a saved research report, not a reproducible data export. It contains claims derived from public repository READMEs and third-party reports. Revalidate releases and source code before adopting dependencies or their security claims. Several cited token figures are estimates, not independently audited cost measures. Never conflate API-equivalent price, subscription quota, cumulative tokens, current context size and actual billing. No hidden chain-of-thought access is implied.