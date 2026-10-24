# The Contrarian and the Explorer: Two Oversight Roles for an Operational Investigation Team

**Additional user requirement | 24 October 2026 | Public, anonymized research and internal-agent design note**

> Companion to [full research](DEEP_RESEARCH_REPORT.md), [multi-agent workbench](MULTI_AGENT_OPERATIONS_WORKBENCH.md), and [internal discovery handoff](../AGENT_HANDOFF.md).
>
> These are **capability roles to test**, not necessarily two always-running LLM instances. The actual target organization and technologies are deliberately unspecified. The internal agent must obtain permission and establish the true environment. Never publish private code, architecture, incidents, credentials, logs, human identities or source references to this public repository.

## 1. The design problem

Specialists are optimized for familiar domains. That efficiency introduces two risks:

- **Premature agreement / anchoring:** Specialists find something plausible, then treat the first explanation as established.
- **Local-search trap:** Specialists follow known playbooks and repeatedly query their favored system even when the true cause is elsewhere, or give up once their script runs out.

The user wants **two additional recognizable masters in the game-like lobby** who challenge the team's *investigation process*, not merely its technical answers:

1. **Contrarian (Skeptic / Red Team / Falsifier):** protects truth and calibration.
2. **Explorer (Curiosity / Investigative Coach / Opportunity Scout):** protects breadth, curiosity, and useful persistence.

Their purpose is not to fight for airtime. Both must propose **specific evidence-based actions with bounded cost**.

There is real precedent for a *Contrarian*: incident.io publicly describes an adversarial agent that attempts to disprove an investigation hypothesis, then updates the structured findings. Anthropic describes evaluator–optimizer loops, but warns of increased inference cost. The exact *Explorer* role below is a proposed design extension that must be evaluated; do not present it as an established vendor standard.

Sources:
- incident.io, *Building Investigations: what it takes to build an AI SRE*, October 1, 2026: https://incident.io/blog/building-investigations-what-it-takes-to-build-an-ai-sre
- Anthropic, *Building effective agents*, December 19, 2024: https://www.anthropic.com/research/building-effective-agents
- Anthropic, *Common architecture patterns*: https://claude.com/resources/guides/building-effective-ai-agents/common-architecture-patterns

## 2. Distinct responsibilities

| | Contrarian | Explorer |
|---|---|---|
| Fundamental question | "What makes us think this is true? How could it be wrong?" | "What might we be overlooking? What is the best new check?" |
| Trigger | Root-cause claim, risky proposed action, contradiction, suspiciously high confidence, recent-change attribution | Investigation stalls, existing playbook fails, unexplained symptom, missing cross-domain evidence, human requests deeper analysis |
| Input | Claims, hypotheses, evidence ledger and a small set of source pointers | Current investigation state, paths already checked, unresolved questions and unused capabilities |
| Output | Specific counter-hypotheses, falsifying tests, contradictory observations, unsupported claims | A prioritized list of novel diagnostic actions with expected information value and query cost |
| Success | Prevents a false conclusion or strengthens a correct one with decisive evidence | Discovers a better next step that improves eventual accuracy or time to useful evidence |
| Failure mode | Contradicts everything or demands impossible proof | Recommends endless queries, speculative tangents or unsafe experimentation |

Neither may fabricate a source, overrule a permissions gate, or insist that continued investigation is always beneficial.

## 3. Example: missing expected output

A user reports: "Output X has not arrived."

The domain masters establish that an expected output was not observed. A build master finds a deployment nearby in time. The coordinator proposes: **"The deployment caused the output failure."**

The **Contrarian** objects to the causal jump:
- "A nearby deployment is a correlation, not a demonstrated cause."
- "Did the generating process fail *before* or *after* deployment?"
- "Did other runs under the same version succeed?"
- "Could input data, schedule conditions, delivery or naming explain the symptom?"
- "What one inexpensive observation would disprove the deployment hypothesis?"

The **Explorer** points out a possibly unexamined path:
- "The team has confirmed the destination is empty but has not established whether the output was expected on this business date."
- "Check the authoritative schedule condition and expected filename convention before opening thousands of more log records."
- "This costs one narrowly scoped read-only query and could eliminate multiple hypotheses."

A specialist executes the chosen permitted check; the evidence ledger is updated. The agents do not win debates by confidence or eloquence; **observations resolve disputes**.

## 4. An output contract instead of an endless conversation

**Contrarian review (compact structured record):**

```json
{
  "role": "contrarian",
  "target_claim_id": "claim-7",
  "verdict": "unsupported | supported | contradicted | insufficient_evidence",
  "strongest_counterargument": "A deploy preceded the symptom, but no failure was linked to its revision.",
  "contradictory_evidence_refs": [],
  "missing_evidence": ["execution-revision metadata"],
  "discriminating_check": {
    "question": "Were failed and successful runs using different revisions?",
    "source": "authorized deployment/run metadata",
    "read_only": true,
    "estimated_cost_class": "low"
  },
  "recommended_next_step": "test_once"
}
```

**Explorer proposal (compact structured record):**

```json
{
  "role": "explorer",
  "investigation_state": "stalled",
  "blind_spot": "The incident's expected behavior has not been verified.",
  "candidate_check": "Read the authoritative output schedule and eligibility conditions for the specific date.",
  "hypotheses_distinguished": ["not_expected", "generation_failed", "delivery_failed"],
  "estimated_query_cost_class": "low",
  "expected_information_value": "high",
  "evidence_needed": ["authoritative schedule definition"],
  "permissions_required": "existing read-only",
  "safety_notes": "No state changes; do not fetch sensitive payload content.",
  "decision": "suggest_to_coordinator"
}
```

These are illustrative schemas. The UI should show what is known and why a check matters, **not an inaccessible hidden chain of thought**. Results need pointers, timestamps, environment scope and per-role token/tool usage.

## 5. How to implement this without wasteful Claude usage

### Model A — Two invisible checks inside the coordinator (**baseline**)

Before a final cause claim, the coordinator runs a short deterministic rubric:
1. Is the object and expected behavior established?
2. Which evidence directly supports the claim?
3. What would falsify it?
4. Are contrary signals ignored or inaccessible?
5. Is cause being confused with timing/correlation?
6. What remains unknown?
7. What would another query reveal that changes the decision?

When the investigation stalls, run a different short rubric:
1. Have we verified assumptions about expected behavior?
2. Are we stuck in one source or repeating a check?
3. Which untested, authorized source might distinguish current hypotheses?
4. Is there a cheaper query than another model call?
5. Is new evidence likely enough to justify cost and latency?
6. If not, stop and escalate.

Advantages: negligible architectural overhead and no required extra agent session. Limitation: the same model can share its own blind spots.

### Model B — On-demand specialist evaluation (**recommended to benchmark**)

Keep the Contrarian and Explorer visible as permanent lobby characters, but launch a **short, isolated Claude review** only on meaningful triggers. Give each role **a compact evidence packet**, not the full investigation or raw logs. Demand the schemas above and a hard output-token cap. No unrestricted tools by default.

The coordinator may accept, reject or request a single permitted check. Prevent automatic role-to-role endless debate.

Advantages: independent perspective with bounded additional cost. Limitation: another model call may still share biases; an evaluator is not ground truth.

### Model C — Continuous autonomous debate (**research-only until justified**)

Independent long-running agents challenge every finding and propose ongoing research. This may help selected complex incidents but increases token, query and latency costs. It can create superficial disagreements and speculative detours. Prove superiority on historical cases before considering it.

**Default:** show the characters always; invoke their reasoning only when needed. Persistent persona in the interface **does not mean persistent model compute**.

## 6. When to activate each role

**Contrarian events:**
- Before calling something the root cause, especially on moderate/high-impact incidents.
- Before recommending a production-affecting action.
- When a model claims a change *caused* an outage based only on timestamps.
- When different specialists disagree or evidence quality is poor.
- When humans flag an apparently wrong conclusion.
- Optionally a low-cost single-pass review on otherwise routine incidents, evaluated against added latency.

**Explorer events:**
- No important new finding after N scoped checks, with N configurable and measured.
- A standard playbook ended without resolving the issue.
- Significant unknowns remain despite a plausible narrative.
- All active specialists are focused on the same system even though the incident crosses boundaries.
- The investigation is blocked by permission or missing data; propose an escalation or sanctioned sandbox test instead of bypassing permission.
- A human presses **"Challenge conclusion"** or **"Explore further"**.

**Never activate** simply because an agent is idle or because a conversation could be longer.

## 7. Priority of checks and stopping rule

Represent a suggested check conceptually as:

`value = expected ability to distinguish current hypotheses or identify actionable missing evidence`

`cost = tokens + telemetry load + latency + human review + security/operational risk`

Prefer high **value per permitted cost**, not the most exciting or unusual query.

An Explorer must not make unsupported guesses merely to be creative. When several options look unpromising, it should return "No useful further check within the current budget."

Stop or escalate when:
- Evidence is sufficient to support the user-facing conclusion.
- The next check cannot materially distinguish hypotheses.
- Query/token/time allowance is exhausted.
- Permissions are unclear.
- The remaining action belongs to the infrastructure or another authorized team.
- Unsafe replication would be required.

**No permanent debates, no automatic approval of tool actions, no forcing artificial consensus.** If the experts disagree after allowed checks, the UI shows the disagreement and its specific missing evidence.

## 8. Two different graphs to show humans

The UI may include:

**Investigation/team graph:** who asked whom, whether an expert is working, task dependencies, challenges and check proposals. The Contrarian and Explorer appear as distinct lobby cards with clear "Challenging" or "Exploring" status.

**Operational dependency graph:** what the reported object means, verified producers, consumers, inputs, triggers, live observations, code/deployment relations, downstream impact and evidence provenance.

Do not conflate "the Explorer asked the deployment master" with "the service is technically dependent on the deployment service."

Suggested UI widgets/cards:
- **Challenge raised:** "Recent deployment is only temporally correlated." → "View evidence" → "Run low-risk check".
- **Blind spot detected:** "Expected arrival date not verified." → "Proposed read-only schedule lookup".
- **Unresolved debate:** each claim and counterclaim points to sources and shows exactly what evidence is missing.
- **Budget:** cumulative model tokens, model cost (if authorized/billed), tool calls and estimated next-check cost.

## 9. Benchmark these roles; do not assume they improve anything

Run the same permitted historical incident set with these variants:

A. One coordinator + domain tool packs.

B. A + deterministic contrarian/explorer checklists.

C. B + triggered isolated contrarian Claude calls.

D. C + triggered isolated explorer Claude calls.

E. Fully persistent multi-agent debate (optional stress-test only).

Measure:
- Correctness and evidence precision.
- False attribution to recent commits/config/deployments.
- Contradictions identified that actually matter.
- Improved diagnosis of incidents outside known playbooks.
- Useful next-check rate versus speculative query rate.
- Time to useful context, total latency, per-incident token cost and telemetry query volume.
- Frequency of abstention, permission-safe escalation and unsafe action proposals.
- Whether roles merely repeat one another or actually change the outcome.

Be particularly skeptical of LLM-as-judge agreement as a metric. Use reviewed incident outcomes and time-correct source evidence; do not let the critic read future fix commits or postmortems unavailable during the original incident.

## 10. Assignment extension for the authorized internal agent

After reading the latest [internal-agent handoff](../AGENT_HANDOFF.md) and [workbench design](MULTI_AGENT_OPERATIONS_WORKBENCH.md):

1. Document the actual pattern of premature conclusions, blind spots and known troubleshooting dead ends in a narrow permission-approved sample.
2. Identify what a deterministic evidence-quality checklist can catch *without* invoking an additional model.
3. Define small, bounded trigger conditions for Contrarian and Explorer roles. Identify contexts where a role adds cost but no benefit.
4. Design a synthetic UI mock showing coordinator + domain specialists + Contrarian + Explorer, including observable findings, challenges and authorized test proposals.
5. Implement only an approved, sandboxed or synthetic benchmark of the four cheapest role arrangements above before requesting more infrastructure.
6. Require evidence-backed claims, cost counters and strict stopping rules. Do not automate production changes on the strength of a model debate.
7. Return recommendations, measured or explicitly estimated model-token impact, the most common false positive patterns, and whether these roles merit independent Claude sessions.

**Core design principle:** The Contrarian should ask, *"How might our explanation fail?"* The Explorer should ask, *"What evidence have we not thought to seek?"* The engineer and the empirical evidence remain the final authority.
