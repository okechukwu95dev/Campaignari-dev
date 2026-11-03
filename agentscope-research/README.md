# AgentScope research and internal-agent handoff

Prepared 2026-11-02. Published as six research/engineering documents in `okechukwu95dev/Campaignari-dev/agentscope-research/` on the repository's `main` branch. This is a **public** repository. All material here describes generic development tooling and synthetic examples; do not add internal company material.

Read in this order:

1. `01_FULL_DEEP_RESEARCH.md` — **complete, unchanged** prior Deep Research report: Spotify Shunt, local context/accounting dashboards, subagent monitors, work safety, 3D office options, recommendations.
2. `02_CLAUDE_DREAMS_MEMORY_RESEARCH.md` — new, source-linked research distinguishing Managed Agent Dreams, Claude Code auto memory/AutoDream, and local SkillOpt-Sleep/dreaming-skill. Adds concrete Memory Lab feature and safeguards.
3. `03_AGENT_SCOPE_ENGINEERING_HANDOFF.md` — buildable architecture, security requirements, event schema, milestones, agent implementation tasks and definition of done.
4. `04_EXPERIMENTS_AND_EVALUATION.md` — reproducible A/B benchmarks for token optimizations and memory quality. No fabricated measurements.
5. `05_SOURCE_INDEX.md` — official docs, source repositories, issue reports and confidence notes.

## Mission for the coding agent

Build a **generic**, local-only, read-only Claude Code observability system. The first demo uses synthetic sessions with parallel agents and a failure investigation. Let an engineer inspect the session's observable context, per-request input/output/cache token classes, agent hierarchy and costly tool outputs. Add a staged, evidence-linked memory-consolidation prototype that cannot rewrite live instructions without explicit approval.

## Safety constraints

Never copy internal company repositories, source, screenshots, issue trackers, session transcripts, endpoint names, customer data or secrets into a personal or public GitHub repo. Do not run external LLM calls or model-backed sleep/replay without explicit review and a quota budget. Do not bind any transcript-viewing HTTP service to a public interface. Do not install unreviewed Claude Code hooks. Any visualization of 'thinking' is a metaphor for observable agent activity, not a literal internal model state.

## Current repository and implementation boundary

These documents are published **for reading and planning**, not an instruction to integrate AgentScope into Campaignari application code. Build AgentScope as an isolated prototype using synthetic inputs. If the work evolves to a corporate environment, migrate the approved implementation plan to a company-approved private repository before using internal data.

## Agent reading checklist

- Read **all five numbered documents**, not just `01_FULL_DEEP_RESEARCH.md`. Total is roughly 55 KB of substantive Markdown, plus this README.
- Use `01` and `05` for sourced market/tool landscape; `02` for memory design; `03` for architecture and MVP tasks; `04` for controlled evaluation and acceptance criteria.
- Distinguish directly observed billing fields from estimated per-file/tool attribution, and observational monitoring from a claim about private model reasoning.
- Do not assume these research files contain a working AgentScope implementation; they are a specification and sources for the implementation.
- Implement the **read-only, local-first dashboard** before adding hooks, model-backed memory synthesis or 3D visuals. Test on synthetic fixtures.
- Report the MVP code changes and remaining gaps in a PR; do not deploy to a corporate workstation without security approval.
