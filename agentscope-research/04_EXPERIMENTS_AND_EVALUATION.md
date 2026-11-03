# AgentScope controlled evaluation: efficiency and memory

Goal: establish whether the observability project can identify savings, and whether a reviewed memory-consolidation workflow improves later Claude Code sessions without damaging correctness. This document describes experiments, **not measured results**.

## Experiment A: token-routing and context economization

Conditions, randomized across identical fresh git clones and fresh session histories:

- **A0**: stock Claude Code running the task; no new hooks, no delegation constraint.
- **A1**: stock Claude Code plus an explicit, narrowly scoped Haiku bulk-reader subagent (subscription quota consumed as normal). No Spotify infrastructure.
- **A2**: Shunt-inspired routing for large *read-only* exploration with an approved hook, only in a personal/synthetic environment; never on work without review. Measure both parent and worker usage. This is not the actual Spotify plugin unless Portal/AiKA is configured.

Use at least five scenarios x three replicates per condition: locate root cause, cross-service file flow, identify test coverage gap, explain a failing test, generate a small additional test with acceptance checks. The corpus must contain both qualifying large-file reads and ordinary short-file tasks to avoid selection bias.

Collect in machine-readable result records: task and fixture revision, random seed/order, model(s), version, subscription/API mode, requests, input/output/cache read/cache creation tokens, worker tokens, context size, API-equivalent price-table version, runtime, tool calls, duplicate file reads, tests passed, issue root-cause correctness, editor change quality, deviations and failure notes. Report medians plus ranges; no claim of monetary 'billing savings' under a flat subscription.

Calculations:

- `total_api_equiv_usd = Σ_requests (uncached_input*rate_in + output*rate_out + cache_reads*rate_cache_read + cache_writes_by_ttl*rate_cache_write) / 1e6` using each request's model-specific rates.
- `quality_adjusted_cost = total_api_equiv_usd / number_of_successful_tasks` (only when denominator > 0).
- `speed = correct_tasks / wall_clock_hour`; `per_task_time = elapsed seconds`.
- `context_reduction` is measured at comparable checkpoints, distinct from summed request token reduction.

Use a simple blind rubric for correctness. If optimization increases errors, a lower token count is not a success. Do not assume Spotify's 90% intercepted-read number generalizes to total session cost.

## Experiment B: memory consolidation versus baseline

Use the same synthetic repository across sequential sessions and simulate genuine evolving facts: a path was moved, a job renamed, a previous fix didn't work, a new configuration rule becomes current. Construct an initial history set (training) and unseen follow-up tasks (evaluation). Avoid leaking evaluation answers into memory synthesis prompts.

- **B0**: default auto memory only.
- **B1**: indexed and reviewed factual notes, no LLM-generated consolidation.
- **B2**: propose dream-style consolidation then approve only verified factual changes, preserve original memory and log every diff.

Evaluate **grounded retrieval precision**, **fact freshness**, **duplicate-rate after consolidation**, **contradictory memory rate**, **time-to-root-cause**, **number of resumed investigations that repeat solved work**, **runtime**, **token-class consumption**, **context overhead of loaded memory**, and **total improvement from memory**. Include an adversarial task where an outdated memory contradicts current repo evidence; expected behavior is to trust current verified state. Include a rare but critical safety constraint that must not be pruned.

A practical success gate for a proof-of-concept: zero known secret leakage, no regression in correctness on held-out tasks, an auditable explanation for every promoted memory, and a demonstrated reduction in redundant exploration or wall time. Do not assert numerical targets as outcomes.

## Cost/privacy safeguards

The monitor itself must not call Claude or any other LLM. For a dream experiment, use a fixed model call budget, preview of submitted context and documented use of subscription quota. Disable unknown background replay. Default all experimentation to synthetic sessions. Do not upload internal source, chats, tools, files, or memory to a public Github repository, a third-party vendor or an LLM API outside approved controls.