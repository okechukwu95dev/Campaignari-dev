> **Public, anonymized version of the most recent Deep Research report | October 24, 2026**
>
> The source is the full report retrieved from the research file library in this conversation—not the earlier GitHub draft. This research analyzes public systems and generic examples only, **not** any employer's actual architecture. No confidential source code, channels, account names or production topology are included. Illustrative operational identifiers were generalized to **Output X**. Vendor capabilities and API prices reflect the research snapshot and should be checked against current primary documentation before use.
>
> **Internal agent:** Read [the dedicated handoff](../AGENT_HANDOFF.md). This is a read-only public reference: never send internal discoveries or private records back to this repository.

---

# Building an Internal AI SRE Investigation and Operational Intelligence Platform

The system you have been describing is real, and the industry is converging on a recognizable architecture for it.

The closest current term is **AI SRE**, sometimes **agentic SRE** or **AI-driven incident investigation**. Microsoft calls its product an SRE Agent, AWS now has a DevOps Agent, Google describes its Investigation Orchestrator as an “SRE in a box,” and Datadog describes Bits Investigation as an autonomous SRE agent. The terms are still settling, so I would not use “AI SRE” alone as the architectural description. A more exact description of what you are trying to build is:

> **An evidence-grounded operational intelligence and incident-investigation platform, driven by an AI SRE agent, backed by a living operational context graph, incident memory, code and document retrieval, governed operational tools, historical evaluations, and an optional isolated reproduction environment.**

That is substantially different from “a production support chatbot.”

The central result of this research is that **the model is not the hard part**. The hard part is the machinery around the model: determining what exists, resolving names, getting fresh evidence, keeping old knowledge from lying, deciding which tool to call, reducing huge tool outputs, tracking hypotheses, proving claims, managing permissions, testing changes against past incidents, and showing humans exactly what the system is doing. Recent engineering reports from incident.io, Datadog, AWS, Microsoft, Google, and others are converging strongly on these ideas. ## What you are actually trying to build

The simplest mental model is not “give an LLM access to Slack, GitHub and Datadog.”

Think of it as **four connected systems**.

| Part | Purpose |
|---|---|
| **Operational context layer** | Understand what things are and how they relate |
| **Investigation engine** | Form hypotheses, gather evidence, reject bad hypotheses, and continue |
| **Controlled tool layer** | Query real systems and, where permitted, perform bounded operations |
| **Human investigation workspace** | Let an engineer watch, inspect, challenge, approve, and navigate the investigation |

The first layer answers questions such as:

> What is “Output X”?  
> Is it a file, process, report, business stage, message type, or internal nickname?  
> Who produces it?  
> What has to happen before it exists?  
> Where is it delivered?  
> Which scheduler is involved?  
> Which code implements it?  
> What consumes it downstream?  
> What happens when it is late or empty?  
> Which team owns it?  
> Has this happened before?

The second layer answers:

> Given that Output X is missing today, what should I check first?  
> Which observations support each possible explanation?  
> Which observation could disprove an explanation?  
> Has anything changed recently?  
> Is this the same pattern as a previous incident?

The third layer answers:

> Am I permitted to query this database?  
> Can I inspect this Kubernetes namespace?  
> Can I read that object-storage location?  
> Can I only recommend a restart, or can I execute one?  
> Is an infrastructure-team approval required?

The fourth layer lets the human see all of that happen instead of receiving a mysterious paragraph saying, “The root cause appears to be X.”

This distinction is important. Rootly's 2026 reference architecture explicitly separates **evidence, inference, authorization, execution, and verification**. It also argues that different evidence types should be retrieved differently: telemetry through time-bounded queries, deployments/configuration through exact APIs, dependencies through graph/catalog traversal, ownership through authoritative systems, and runbooks/postmortems through document search. That is almost exactly the distinction your problem requires.

### The industry term and the more useful internal term

**AI SRE** is the recognizable market term. **Agentic SRE** is also used. AWS explicitly uses “agentic SRE” for an architecture connecting alerts, observability, GitHub, Slack, runbooks, custom tools, investigation, and remediation planning. For an internal project name or design document, I would call the underlying architecture an:

**Operational Investigation Engine**

with an:

**Operational Context Graph**

and an:

**Evidence Ledger**

Those three terms describe the actual engineering problem better than “bot.”

The “agent” is one component of the engine.

### This is not just RAG

A huge RAG database containing Slack, Notion, source code, logs, incidents and runbooks is **not** the right architecture.

RAG is useful for questions such as:

> Find the runbook explaining the overnight file process.

It is much worse for questions such as:

> Did job X run at 01:31 today?

or:

> Which commit is actually deployed in this environment?

or:

> Which service currently owns this Kubernetes pod?

or:

> What is downstream of this file?

Those facts should come from authoritative systems, APIs, deployment metadata and relationship graphs rather than semantic similarity. Rootly explicitly makes this distinction in its architecture guidance. Your system therefore needs **federated retrieval**, not “put everything into vectors.”

## What the strongest systems are doing now

The most useful research finding is that several teams independently arrived at a very similar loop.

```text
Incident / question
        |
        v
Identify entity + time + environment
        |
        v
Gather SMALL amount of orientation context
        |
        v
Generate competing hypotheses
        |
        v
Choose evidence that could prove/disprove each
        |
        v
Call targeted tools
        |
        v
Reduce raw results into structured findings
        |
        v
Update / reject / expand hypotheses
        |
        +-------------------------+
        |                         |
        v                         |
Need more evidence? -------------+
        |
        no
        v
Evidence-backed conclusion
        |
        v
Human decision / governed action
        |
        v
Verify outcome
        |
        v
Save useful reviewed knowledge
```

That loop is much closer to what experienced support engineers actually do than “search everything and ask the LLM what happened.”

### What incident.io learned

incident.io published one of the most valuable engineering descriptions I found in October 2026.

Their investigation system initially fans out across sources such as deployments, telemetry and Slack. It converts what it finds into findings, synthesizes hypotheses, and then uses another agent adversarially to try to disprove the current hypothesis. More important is what happened when they tried to make it reliable.

Their first implementation let the LLM write an essay about what it thought was happening. They report that it hallucinated too frequently. They progressively replaced that with:

structured findings → evidence-backed records → findings that new evidence can invalidate → incremental “waves” that record only what changed. That is extremely relevant to your design.

**Do not make the central memory object an AI-written narrative.**

Make it something closer to:

```text
Finding F-184

Claim:
    Today's expected output has not been observed.

Status:
    VERIFIED

Evidence:
    storage-query://...
    scheduler-run://...

Observed:
    2026-10-24 01:42 UTC

Scope:
    environment = ...
    process = ...
    output = ...

Supports:
    hypothesis H-3

Contradicts:
    hypothesis H-1

Last verified:
    2026-10-24 01:44 UTC
```

The prose explanation to humans can be generated from those records.

The records themselves should remain structured.

incident.io also reports that telemetry query results are compressed before they reach the investigation model: instead of dumping raw data, the system summarizes what was asked, why it was asked, and useful statistics. They specifically identify uncontrolled token consumption and excessive observability queries as production problems. This directly validates your concern about token usage.

### What Datadog learned

Datadog describes essentially the same change in approach.

Its agent forms a root-cause hypothesis, performs **targeted queries** to validate or reject it, and repeats. Datadog says this reduces noise compared with summarizing all available telemetry at once. Datadog also built a benchmark using **real incidents labeled by human responders**. Archived telemetry is replayed through the agent and results are compared with ground truth. That answers part of your annotation question.

You do need labeling.

But the highest-value labeling is probably **not manually tagging millions of Slack messages**.

The high-value labeling is:

> Here are real incidents.  
> Here was the information available.  
> Here was the real problem.  
> Here was the affected component.  
> Here was the fix.  
> Here were misleading signals.  
> Here is what a competent investigation should have discovered.

### What Microsoft is doing

Azure SRE Agent now exposes **deep investigations as a hypothesis tree**. Hypotheses can be validating, validated, invalidated, or inconclusive, and validated hypotheses can create deeper branches. Microsoft's current architecture also exposes connectors, memory/knowledge, skills, code execution and different run modes rather than treating the agent as a single prompt. That is important for your desired UI.

The investigation should look less like:

```text
AI:
I think the file transfer failed.
```

and more like:

```text
Missing Output X
│
├── H1: upstream generation failed
│   ├── schedule executed       VERIFIED
│   ├── producer started        VERIFIED
│   └── producer completed      FALSE
│
├── H2: transfer failed
│   ├── source artifact exists  UNKNOWN
│   └── transfer error          NOT CHECKED
│
├── H3: destination problem
│   └── expected path empty     VERIFIED
│
└── H4: recent change caused failure
    ├── deployment at 22:17     VERIFIED
    └── causal relationship     UNKNOWN
```

The human should watch that tree evolve.

### What Google is doing

Google's October 2026 Investigation Orchestrator explicitly describes **parallelized hypothesis evaluation** and separate quick versus deep investigation strategies. It combines observability data, diagnostic procedures, topology and source-code information. This reinforces an important design choice:

**Depth should be adaptive.**

Most incidents should not trigger the maximum possible investigation.

A cheap first pass may discover enough evidence.

Only ambiguous or important incidents should cause a much deeper search.

### What AWS is doing

AWS's 2026 architecture is particularly close to your scattered-tool scenario. Its documented example connects monitoring, Splunk, GitHub and Slack; receives incident webhooks; retrieves deployment history; analyzes logs and metrics; correlates operational behavior with deployments; and uses Skills/runbooks to guide investigations. AWS also separates two forms of learned knowledge:

**Skills** contain procedural knowledge: how an investigation should be performed. **Memories** contain synthesized informational knowledge about an environment. AWS added a “reflections” memory mechanism in September 2026 so human corrections to incorrect investigations can influence future investigations. That distinction is useful enough to copy conceptually:

```text
FACT / MEMORY
"Process X normally creates artifact Y."

SKILL / PROCEDURE
"When Y is missing, first establish whether X ran.
If X ran, establish whether Y was created.
Only then investigate transfer/delivery."
```

One should not be confused with the other.

## Knowledge, indexing, graphs, annotation and keeping everything fresh

This is where I would change the original mental model most strongly.

You do **not** want one giant “company knowledge base.”

You want several kinds of knowledge that behave differently.

### Keep the authoritative data where it already lives

A strong design looks more like this:

| Information | Preferred source |
|---|---|
| Current logs/metrics/traces | Observability system |
| Current scheduler state | Scheduler |
| Current objects/files | Storage system |
| Current transfer state | Transfer system |
| Current database state | Database through approved queries |
| Current Kubernetes state | Kubernetes API through existing permissions |
| Code | Git repository/code search |
| Commit/deployment relationships | VCS + deployment system |
| Runbooks/docs | Document search |
| Historical discussions | Chat search |
| Ownership | Existing catalog/ownership source |
| Stable relationships | Operational context graph |
| Previous investigation lessons | Incident memory |
| Troubleshooting methods | Skills/playbooks |

The agent asks each system the question that system is good at answering. This is the same broad principle found in current AI SRE reference architectures. This also reduces duplication and stale data.

### Build an entity registry before a giant graph

One of the most dangerous problems will be names.

Suppose:

```text
output-x
Output X
output-x-file
output_x
overnight-output-x
business phrase "Output X"
```

all refer to the same thing.

Or perhaps they do **not**.

Your agent needs an **entity-resolution layer**.

Rootly specifically warns about the case where different service names refer to the same service and recommends resolving identifiers before correlating evidence. Your eventual entity catalog might contain things like:

```text
Service
Application
Process
Business process
Job
Schedule
File / artifact
Storage location
Transfer endpoint
Queue / topic
Event
API
Database
Table
Repository
Code module
Library
Deployment
Environment
Feature
Team
Alert
Dashboard
Runbook
Incident
External dependency
```

This list is deliberately open.

I would **not** tell your internal agent that those are the entities your organization definitely has. Its job should be to discover the actual vocabulary.

### The important thing is relationships

For an operational investigator, relationships are often more useful than tags.

Compare:

```text
tags = ["batch", "overnight", "file"]
```

with:

```text
Job A --PRODUCES--> Output X
Schedule B --TRIGGERS--> Job A
Source C --INPUT_TO--> Job A
Output X --DELIVERED_VIA--> Transfer D
Output X --CONSUMED_BY--> Job E
Job E --PRODUCES--> Report F
Repository G --IMPLEMENTS--> Job A
Deployment H --DEPLOYED--> Service A
Team I --OWNS--> Job A
```

The second structure lets you answer:

> What breaks downstream if Output X is absent?

That is the capability you were describing.

OpenLineage already demonstrates this concept for data workflows by modelling datasets, jobs and runs and weaving individual runtime observations into a lineage graph. Backstage demonstrates another part of it: a software catalog containing services, libraries, pipelines, ownership metadata and relationships among catalog entities. DataHub demonstrates upstream/downstream impact analysis over a lineage graph. None of those products alone is the system you described. Their data models are useful prior art.

Your graph is broader. It may need to connect **business concepts, application concepts, data movement, infrastructure and code**.

### Every relationship needs provenance

This is one of the most important design decisions in this report.

Do not store only:

```text
Output X -> consumed by -> Process Q
```

Store something conceptually like:

```text
subject:
    Output X

relationship:
    consumed_by

object:
    Process Q

status:
    verified

source:
    code reference / runtime observation / documentation / human confirmation

confidence:
    ...

environment:
    ...

valid_from:
    ...

last_verified_at:
    ...

evidence_refs:
    [...]
```

The graph UI can then visually distinguish:

**Verified relationship**  
Authoritative evidence supports it.

**Observed relationship**  
The system actually saw it happen at runtime.

**Inferred relationship**  
The agent thinks it exists but has not verified it.

**Human-declared relationship**  
An engineer explicitly supplied it.

**Stale relationship**  
It used to be valid but has not recently been confirmed.

**Disputed relationship**  
Current evidence conflicts.

Without this, an AI-generated graph can become a very attractive map of incorrect assumptions.

### You probably do not need a graph database initially

A graph **model** is valuable.

A graph **database product** is an implementation decision.

For a prototype, an entity table plus a relationship table may be entirely sufficient:

```text
entities
relationships
evidence
incidents
findings
hypotheses
skills
memories
```

The internal agent should examine the scale and query patterns before selecting a specialized graph database.

That also keeps the project smaller.

### Do not GraphRAG the entire company on day one

Microsoft's GraphRAG documentation says its richer standard graph extraction is expensive and estimates graph extraction at roughly **75% of indexing cost** in that pipeline; its cheaper approach trades richness for noisier extraction. That reinforces a better order:

```text
Authoritative relationships first
        |
        v
Relationships extractable deterministically from code/config
        |
        v
Runtime-observed relationships
        |
        v
Existing human-maintained metadata
        |
        v
LLM-extracted candidate relationships from prose
        |
        v
Human verification where important
```

Do not pay an LLM to “understand everything” when a configuration file, scheduler API, deployment manifest or runtime trace already states the relationship exactly.

### Freshness is a first-class problem

A stored fact becoming wrong can be more dangerous than having no fact.

incident.io describes exactly this problem: a saved telemetry query may become wrong after a service rename or log-format change. Their system periodically re-runs knowledge and intentionally rewrites or forgets memories that have drifted. So every persistent fact should eventually have something like:

```text
created_at
observed_at
last_verified_at
source_version
valid_from
valid_until
freshness_policy
```

Freshness can be maintained with a mixture of:

```text
Webhooks/events
    Git changes
    deployments
    incidents
    document updates

Incremental polling
    scheduler metadata
    catalogs
    APIs without webhooks

Runtime observations
    traces
    jobs
    files
    message flow

Periodic validation
    important stored memories
    saved queries
    inferred relationships
```

This is much better than periodically re-embedding the entire company.

### Do you need annotation, labels and tags?

**Yes. But not in the way you were imagining.**

You need three kinds of labeling.

| Kind | Purpose | Human work |
|---|---|---:|
| Metadata | Make things identifiable | Mostly automatic |
| Relationship verification | Make the system map trustworthy | Selective |
| Incident ground truth | Measure whether the investigator is correct | Important |

I would **not** start with people manually tagging thousands of Slack messages.

Start by automatically extracting metadata already present in systems.

Then create a relatively small, carefully reviewed **historical incident benchmark**.

Datadog built its own evaluation around real incidents labeled by responders. incident.io independently arrived at historical backtesting. They freeze an old incident at the point in time when it happened, run the agent using only information that would have existed then, and compare its hypothesis with the historical result. They specifically warn about accidentally leaking future information such as the fix PR, later postmortem or later Slack diagnosis into the test. For your prototype, my recommendation would be to begin with perhaps **20–50 diverse, well-understood historical cases**, not thousands. That number is my proposed starting size, not an industry requirement.

Each case might capture:

```text
Incident
Initial symptom
Affected environment
Known affected entities
Evidence available at T0
Actual root cause
Root-cause entity
Useful investigation steps
Misleading signals
Actual remediation
Downstream impact
Human confidence in the label
```

That dataset is probably more valuable than enormous amounts of generic annotation.

## The investigation engine and how to keep token use low

This is the part I would optimize most aggressively.

### Do not give the model everything

The naive architecture is:

```text
Alert
  |
  +--> Fetch all logs
  +--> Fetch all Slack
  +--> Fetch all GitHub
  +--> Fetch all dashboards
  +--> Fetch all documentation
  +--> Fetch all Kubernetes data

Put everything into enormous prompt

Ask AI what happened
```

This is exactly what you should avoid.

Datadog's documented design instead forms hypotheses and performs targeted queries to validate or reject those hypotheses. Anthropic calls the broader discipline **context engineering**: curating the useful information that occupies the model's finite working context rather than assuming more context is always better. Its documentation also warns that recall can degrade as context grows. ### A good investigation loop

I would make the harness operate approximately like this.

```text
STEP A
Resolve incident identity

What happened?
When?
Which environment?
Which known entity/entities?

        |
        v

STEP B
Retrieve orientation context

Entity definition
Owner
Known dependencies
Recent changes
Relevant historical cases

        |
        v

STEP C
Generate a SMALL hypothesis set

H1
H2
H3

        |
        v

STEP D
For each hypothesis ask:

"What evidence would discriminate this
from the alternatives?"

        |
        v

STEP E
Execute narrow queries

        |
        v

STEP F
Reduce tool results

Raw:
12,000 log lines

Model receives:
42 occurrences
First: 01:03:11
Last: 01:14:52
3 representative samples
Affected run IDs
Link to full query

        |
        v

STEP G
Update evidence ledger

        |
        v

STEP H
Prune / expand hypotheses

        |
        v

STEP I
Stop when:

sufficient evidence exists
OR
no useful check remains
OR
budget exhausted
OR
permission boundary reached
```

That final stopping rule matters.

An autonomous agent without a query budget can endlessly search.

Rootly recommends bounding tool calls by time window, result size, timeout and query budget. ### The “internal chain of thought” should become explicit operational artifacts

You have repeatedly described your team's “chain of thought” when troubleshooting.

You do **not** need to store some giant hidden free-form reasoning transcript.

What you actually need to capture is:

```text
Observed symptom
        ↓
Possible explanation
        ↓
Why that explanation was considered
        ↓
Check performed
        ↓
Evidence returned
        ↓
Interpretation
        ↓
Hypothesis strengthened / weakened
        ↓
Next check
```

Call this an **investigation trace**, **decision trace**, or **hypothesis trace**.

That is much easier to audit and much cheaper to store than an unrestricted internal monologue.

Microsoft's current deep-investigation interface similarly presents a visible hypothesis tree with states such as validated, invalidated and inconclusive. The organization's troubleshooting knowledge should eventually be represented as a mixture of:

```text
skills
runbooks
decision rules
saved queries
historical cases
known relationships
known failure patterns
human corrections
```

The model then supplies flexible reasoning between those pieces.

### How it goes “off script”

You specifically do not want:

```text
IF missing file:
    always do checks 1, 2, 3, 4.
```

That cannot handle novel problems.

But you also do not want:

```text
AI has shell access.
Good luck.
```

The useful middle ground is:

```text
Known procedure provides starting strategy

        +

Agent may select other approved tools

        +

Every new check must answer a hypothesis

        +

Tool and query budget is bounded

        +

Evidence changes the next move

        +

Actions remain permission controlled
```

Anthropic distinguishes **workflows**, where predefined code controls the path, from **agents**, where the model dynamically chooses its process and tools. It recommends starting with simple composable patterns rather than unnecessary complexity. Your application is likely a **hybrid**:

deterministic workflow around the outside, dynamic investigation inside it.

### Tool design matters enormously

Do not expose only primitive functions such as:

```text
execute_shell(command)
execute_sql(query)
get_http(url)
```

where possible.

Give the model higher-level investigation tools:

```text
get_recent_deployments(service, time_range)

query_logs(
    service,
    environment,
    start,
    end,
    search_expression,
    max_results
)

get_schedule_history(schedule_id, date)

inspect_artifact(location, expected_pattern, date)

get_runtime_dependencies(entity, time_range)

search_incident_history(entity, symptom)

search_code_references(entity)

get_kubernetes_events(workload, namespace, time_range)
```

The actual names depend on your real environment.

Anthropic's agent tooling research emphasizes choosing the correct tool abstractions, clear namespaces, useful tool descriptions, meaningful results, and token-efficient responses. MCP may be one useful connector protocol. It lets agents discover and invoke externally exposed tools, but the specification explicitly treats tool access as a security-sensitive capability and recommends authorization for sensitive or enterprise resources. MCP is therefore **a connector mechanism, not the architecture and not the permission system**.

### Do not start with five agents talking to each other

Parallel specialists sound attractive:

```text
Datadog agent
GitHub agent
Kubernetes agent
Slack agent
Database agent
Supervisor agent
Critic agent
```

Eventually that may be useful.

It is not where I would start when token efficiency is your highest priority.

Anthropic reports that in its own research system, agents used roughly four times the tokens of ordinary chat interactions and its multi-agent architecture roughly fifteen times as many tokens as chats. That measurement comes from a research workload rather than SRE, but it is a strong warning that multi-agent parallelism is not free. Start closer to:

```text
ONE investigation orchestrator
        |
        +--> many deterministic tools
        |
        +--> optional cheap reducers
        |
        +--> optional challenger only for difficult/high-risk conclusions
```

Then add additional agents only where evaluation proves they help.

### Large tool libraries should be loaded on demand

Do not put 600 tool descriptions into every prompt.

Anthropic has documented environments where tool definitions and intermediate tool results can consume tens of thousands of tokens before meaningful work starts, and recommends on-demand tool discovery/loading and filtering results before they reach the model. Your tool hierarchy could therefore look like:

```text
storage.*
scheduler.*
telemetry.*
code.*
deploy.*
kubernetes.*
database.*
messaging.*
docs.*
incidents.*
sandbox.*
```

At first the agent sees the namespaces.

It loads individual tools when necessary.

### Compress before the expensive model sees data

A raw query might generate 100,000 lines.

The high-quality reasoning model should see:

```text
query
time window
total matches
important dimensions
first/last occurrence
change from baseline
representative examples
anomalies
pointer to complete source
```

not 100,000 lines.

incident.io describes essentially this technique for telemetry. This is one of the highest-value token optimizations available.

### Use model routing

Do not use the strongest model for every operation.

A reasonable conceptual router is:

| Task | Model need |
|---|---|
| Classify incoming alert | Cheap |
| Extract identifiers | Cheap |
| Reformat tool results | Cheap/deterministic |
| Deduplicate logs | Code, no model when possible |
| Retrieve graph neighbors | No model |
| Run database query | No model |
| Compare timestamps | Code |
| Summarize narrow evidence | Cheap |
| Generate competing root causes | Stronger |
| Resolve contradictory evidence | Stronger |
| High-impact final assessment | Stronger |
| Produce UI text | Cheap |

This is where the economics become surprisingly favorable.

### Is roughly \$2,000/month realistic?

Assuming by “\$2,000 token budget” you mean approximately **\$2,000 per month in model/API spend**, yes: that is enough for a serious internal prototype if you engineer the retrieval properly.

As of October 24, 2026, OpenAI lists GPT-6 Luna at \$0.10 per million input tokens and \$0.50 per million output tokens, GPT-6.1 Sol at \$2 and \$10, and GPT-6 Astra at \$10 and \$50 respectively on its model/pricing pages. Pricing and processing modes can change, so this is a snapshot rather than a long-term budget commitment. For illustration:

| Job | Assumption | Approx. model cost |
|---|---:|---:|
| Cheap triage | 20k Luna input + 2k output | **\$0.003** |
| Substantial investigation | 200k Sol input + 20k output | **\$0.60** |
| Very deep investigation | 500k Astra input + 50k output | **\$7.50** |

Those are intentionally simple token calculations and do **not** include search/tool charges, retries, cache writes, databases, compute, observability costs or infrastructure.

An artificial monthly workload of:

```text
5,000 cheap triages        ≈ $15
1,000 substantial cases   ≈ $600
100 very deep cases        ≈ $750
                              ------
                              $1,365
```

would still be below \$2,000 in those raw model-token assumptions.

That is not a prediction of your actual workload. It demonstrates something important:

**Model tokens are probably not the limiting resource for the prototype.**

Access, connectors, correctness, evaluations and engineering time are more likely to be the bottlenecks.

Even text embeddings themselves are inexpensive at current list prices: OpenAI currently lists `text-embedding-3-small` at \$0.02 per million input tokens, which means embedding 100 million tokens once would be roughly \$2 in model charges before storage and surrounding infrastructure. The expensive failure mode is repeatedly sending huge irrelevant contexts through strong reasoning models.

Prompt caching can further reduce repeated-input cost where a stable prefix recurs across investigations. OpenAI's current caching guidance specifically supports keeping stable context reusable while placing frequently changing material later in the prompt. ## Production access, sandbox reproduction and the human workspace

Your clarification about infrastructure permissions makes the architecture cleaner rather than weaker.

The system should have **three operational zones**.

```text
+-----------------------------------------------------+
| PRODUCTION                                          |
|                                                     |
| Observe using only existing authorized permissions |
| Logs / metrics / storage / scheduler / etc.         |
|                                                     |
| Primarily READ                                      |
+-------------------------+---------------------------+
                          |
                          | evidence
                          v
+-----------------------------------------------------+
| INVESTIGATION ENGINE                                |
|                                                     |
| Context + graph + hypotheses + evidence + history   |
|                                                     |
| May PROPOSE production actions                      |
+-------------------------+---------------------------+
                          |
              +-----------+-----------+
              |                       |
              v                       v
+-------------------------+   +-----------------------+
| HUMAN / INFRA APPROVAL  |   | SANDBOX / LOWER ENV  |
|                         |   |                       |
| Prod-controlled action  |   | Broad experimental   |
| if authorized           |   | freedom where allowed|
+-------------------------+   +-----------------------+
```

Kubernetes itself supports fine-grained RBAC and recommends granting users and service accounts the minimum permissions they require, preferably with namespace-level scope where possible. A Kubernetes client does not create additional authority beyond the credentials presented to the Kubernetes API. So if your team can inspect workloads but cannot deploy or restart them, the agent should have the same boundary.

It can say:

```text
Evidence suggests workload X should be inspected/restarted.

Agent authorization:
NO WRITE PERMISSION

Required owner:
Infrastructure team

Prepared request:
[exact resource]
[why]
[evidence]
[expected result]
[risk]
[verification step]
```

That is far better than granting the AI elevated credentials merely because the AI is useful.

### Production actions should not come directly from the model

This is one of the strongest conclusions across the research.

Rootly's architecture explicitly places a policy gateway between model recommendations and production execution. Approved operations should be deterministic workflows with preconditions, postconditions, stop conditions and rollback. So the eventual architecture should be:

```text
Model:
"Restarting worker X may be appropriate."

        ↓

Policy system:
Is this agent allowed?
Is this human allowed?
Correct environment?
Correct resource?
Incident still open?
Action approved?
Blast radius acceptable?

        ↓

Deterministic workflow:
Perform exact operation

        ↓

Independent verification:
Did the actual service recover?
```

Not:

```text
LLM → unrestricted shell
```

### Your sandbox idea is very good

The investigation system should eventually be able to say:

> Production evidence suggests failure X.
>
> I cannot safely experiment on production.
>
> I can reconstruct the relevant conditions in the development sandbox and test the hypothesis there.

But “recreate production” should have levels.

#### Low-fidelity reproduction

Run the relevant function/process with a controlled input.

#### Dependency reproduction

Create temporary database, queue, storage or service dependencies using containers/emulators.

Testcontainers is specifically built around disposable test dependencies and portable integration-test environments. #### Service reproduction

Run the relevant service or collection of services locally.

#### Temporary Kubernetes reproduction

Where useful and supported, instantiate an isolated cluster or namespace containing the relevant subset.

#### Internal lower environment

Deploy through the organization's normal development/test pathway if the team already has one.

#### High-fidelity ephemeral environment

Provision a temporary environment from infrastructure definitions where your organization actually permits that.

Terraform **could** be one implementation mechanism in that final category. It should not be hard-coded into the architecture until the internal environment is inspected.

The reproduction record should contain something like:

```text
Production evidence
        ↓
Candidate failure model
        ↓
Sandbox recipe
        ↓
Code revision
Configuration approximation
Synthetic/sanitized input
Mocked dependencies
Replay events
Injected fault
        ↓
Observed sandbox result
        ↓
Reproduced?
YES / NO / PARTIAL
        ↓
Confidence in applicability to production
```

This distinction matters because the sandbox will rarely be perfectly identical to production.

The agent must be allowed to say:

> I reproduced the same error under these conditions.

or:

> I could not reproduce it.

or:

> I reproduced a similar symptom, but the environment differs in these ways.

That is much more trustworthy than calling everything a “digital twin.”

### The interface you described should be part of the core product

A good investigation workspace could look conceptually like:

```text
┌───────────────────────────────┬─────────────────────────────────────┐
│ SYSTEM / DEPENDENCY MAP       │ LIVE INVESTIGATION                  │
│                               │                                     │
│ Source A                      │ 01:32 Identified Output X entity          │
│    │                          │ 01:33 Checked expected schedule     │
│    v                          │ 01:33 Schedule ran                  │
│ Job B                         │ 01:34 Checking producer logs         │
│    │                          │ 01:35 Producer failed               │
│    v                          │ 01:35 H1 confidence increased        │
│ [Output X]                          │ 01:36 Checking recent changes        │
│    │                          │                                     │
│    +----> Consumer C          │                                     │
│    |                          │                                     │
│    +----> Consumer D          │                                     │
├───────────────────────────────┼─────────────────────────────────────┤
│ HYPOTHESES                    │ EVIDENCE                            │
│                               │                                     │
│ H1 Producer failed       78%  │ [scheduler run]                    │
│ H2 Transfer failed       18%  │ [log query]                        │
│ H3 Destination error      4%  │ [storage check]                    │
│                               │ [deployment event]                 │
├───────────────────────────────┴─────────────────────────────────────┤
│ SANDBOX EXPERIMENTS / PROPOSED ACTIONS                              │
└─────────────────────────────────────────────────────────────────────┘
```

The exact percentages may not even be desirable initially; categorical confidence may be safer unless the numbers are calibrated.

Clicking **Output X** should show:

```text
Definition
Aliases
Entity type
Owner

Expected behavior
Expected arrival/completion window

UPSTREAM
What triggers it
What produces it
What inputs are required

TRANSPORT / LOCATION
Where it moves
Where it lands

DOWNSTREAM
What consumes it
What stops if it is unavailable

IMPLEMENTATION
Relevant repositories
Relevant modules
Relevant configuration

OPERATIONS
Related monitors
Related dashboards
Related schedules
Related runbooks

HISTORY
Previous incidents
Known failure modes

PROVENANCE
Where every relationship came from
Last verification
```

OpenLineage, Backstage and DataHub already demonstrate parts of this relationship/impact model in narrower domains. The important new piece for your system is connecting those concepts to an active incident investigator.

### Show investigation trace, not mysterious private reasoning

Humans need to see:

```text
What question the agent asked
Which tool it used
What scope it queried
What evidence returned
What conclusion changed
Which hypothesis was rejected
What it intends to check next
```

That is the useful transparency.

Datadog now exposes an Agent Trace view for investigation reasoning, while Microsoft exposes the hypothesis tree itself. The goal is not to display pages of model monologue. It is to display an **auditable operational decision trail**.

## The architecture I would actually recommend

There are several legitimate ways to approach this.

### Knowledge-first

Build the company map first.

```text
Docs
Code
Services
Files
Jobs
Owners
Dependencies
Incidents
        ↓
Comprehensive knowledge system
        ↓
Agent
```

**Strength:** excellent browsing and explanations.

**Weakness:** large upfront effort; knowledge becomes stale; you can spend months cataloging before solving a useful incident.

### Graph-first

Attempt to create the complete operational graph immediately.

```text
Everything → extraction → giant graph → investigator
```

**Strength:** potentially excellent relationship traversal.

**Weakness:** relationship extraction, entity resolution and freshness become a project of their own. Full text-to-graph approaches can also add significant indexing cost. Microsoft's GraphRAG documentation is a useful warning here. ### Skill-first

Encode human troubleshooting processes.

```text
Missing artifact?
    ↓
Skill A

Scheduler problem?
    ↓
Skill B

API failure?
    ↓
Skill C
```

**Strength:** cheap, controlled, explainable.

**Weakness:** unfamiliar incidents may fall outside the procedures.

AWS's current DevOps Agent design provides an example of treating investigation procedures as modular Skills. ### Incident-first

Do very little upfront indexing.

When an incident occurs:

```text
Resolve the problem
        ↓
Search only relevant sources
        ↓
Build temporary context
        ↓
Investigate
```

**Strength:** fast to prototype, lower cost, produces value immediately.

**Weakness:** repeatedly rediscovers facts unless useful knowledge is retained.

### The architecture I recommend: incident-first hybrid

For what you have described, I would start here:

```text
                    INCIDENT / HUMAN QUESTION
                              |
                              v
                    +-------------------+
                    | ENTITY RESOLVER   |
                    +-------------------+
                              |
                              v
                 +-------------------------+
                 | CONTEXT BUILDER         |
                 +-------------------------+
                    /       |        \
                   /        |         \
                  v         v          v
           Context Graph   Search    Live APIs
                  \         |          /
                   \        |         /
                    +-------+--------+
                            |
                            v
                 +-------------------------+
                 | INVESTIGATION HARNESS   |
                 |                         |
                 | hypotheses              |
                 | planning                |
                 | tool selection          |
                 | stopping rules          |
                 +------------+------------+
                              |
                  +-----------+-----------+
                  |                       |
                  v                       v
          +---------------+       +---------------+
          | TOOL GATEWAY  |       | SKILLS        |
          +---------------+       +---------------+
                  |
          +-------+------+------+--------+---------+
          |       |      |      |        |         |
        obs.    code   sched. storage   DB     other systems
          \       |      |      |        |         /
           +------+------+------+--------+--------+
                          |
                          v
                   RESULT REDUCERS
                          |
                          v
                   EVIDENCE LEDGER
                          |
               +----------+-----------+
               |                      |
               v                      v
       HYPOTHESIS UPDATE        LIVE HUMAN UI
               |
               v
          CONCLUSION
               |
        +------+------+
        |             |
        v             v
     MEMORY       ACTION PROPOSAL
                      |
                      v
               POLICY / APPROVAL
                      |
              +-------+-------+
              |               |
              v               v
          Production        Sandbox
          workflow          experiment
```

This preserves your whole vision without requiring you to build the whole vision before it becomes useful.

### The persistent data layer can remain surprisingly small

A first implementation might need conceptual records resembling:

```text
entities
aliases
relationships

incidents
hypotheses
findings
evidence_refs

skills
memories

tool_runs
investigation_runs

evaluations
historical_cases
```

The actual logs remain in the logging platform.

The actual code remains in Git.

The actual metrics remain in the observability system.

The actual Slack history remains in Slack.

Your system stores **references, extracted high-value facts and relationships**, not unnecessary copies of everything.

That helps with cost, freshness, storage, access control and deletion requirements.

### Code understanding should also be layered

Do not assume “index the entire Git repository into embeddings” means the agent understands the software.

Use progressively stronger mechanisms:

```text
Repository metadata
        ↓
File/path search
        ↓
Text/code search
        ↓
Manifest/dependency extraction
        ↓
Symbol/reference navigation where supported
        ↓
Targeted semantic retrieval
        ↓
Agent reads only relevant code
```

OpenTelemetry's semantic conventions already support VCS revision identifiers in CI/CD metadata, creating one possible way of connecting operational observations with source revisions where the environment emits that information. The important concept is:

```text
runtime entity
    ↔ deployed version
    ↔ deployment
    ↔ commit
    ↔ changed files
    ↔ relevant implementation
```

Once that relationship exists, the agent does not need to repeatedly search the entire repository wondering which code is running.

### The context graph should grow through actual use

This is the elegant part.

The first time someone asks:

> Why didn't Output X arrive?

the system may know little.

It investigates and discovers:

```text
Output X is produced by X.
X runs after Y.
Y depends on Z.
Output X is consumed by Q.
```

Those relationships become **candidate knowledge**.

Evidence or humans verify them.

Next time, the investigation begins with more knowledge.

This creates a flywheel:

```text
Incident
   ↓
Investigation
   ↓
Useful discovery
   ↓
Verification
   ↓
Operational knowledge
   ↓
Faster next investigation
```

That is much better than trying to model the entire organization before launch.

### The system also needs controlled forgetting

Knowledge should not only accumulate.

It must be able to delete or downgrade:

```text
obsolete aliases
old services
invalid queries
superseded runbooks
incorrect relationships
human-corrected diagnoses
```

incident.io's description of actively revalidating and forgetting stale memory is strong evidence that this problem appears in real deployments. ### Security must exist outside the prompt

Another subtle issue is that Slack, docs, tickets and even source-code comments are **untrusted content** from an agent-security perspective.

A retrieved message saying:

```text
restart-prod-now
```

must not become authorization to restart production.

MCP's current security specification warns that tools can represent arbitrary code execution paths and require appropriate access controls. Rootly's AI SRE architecture likewise treats retrieved content as evidence rather than authority and places permissions and policy outside the model. This is why:

```text
retrieval
reasoning
authorization
execution
```

must be separate components.

## What is realistic for one person and what should happen first

A narrow version is realistic for one strong engineer.

A mature organization-wide autonomous SRE is not a small side project.

incident.io says its mature system has taken a dedicated team about two years and remains ongoing. That is one vendor's experience rather than a universal engineering estimate, but it demonstrates how much work appears after the prototype: backtesting, knowledge drift, token control, observability load, trust, security and evaluation. I would separate your project into capability levels.

| Level | Capability | Solo feasibility |
|---|---|---|
| **Discovery** | Understand sources, entities, permissions and workflows | High |
| **Read-only investigator** | Query several systems and produce cited findings | High |
| **Context graph** | Explain upstream/downstream relationships | High to moderate |
| **Historical evaluation** | Replay labeled old incidents | Moderate |
| **Live investigation UI** | Show graph, evidence and hypotheses | Moderate |
| **Skills/memory** | Learn reusable procedures and facts | Moderate |
| **Sandbox reproduction** | Reconstruct selected failure classes | Highly environment-dependent |
| **Production write actions** | Controlled remediation | Organizational/security project |
| **Broad autonomous SRE** | Reliable across most operational failures | Large project |

A reasonable **planning estimate**, not a promise, is:

**Initial discovery: roughly one to two weeks.**  
The internal agent examines actual repositories, docs, integrations, existing tooling, access boundaries and representative incidents.

**Narrow proof of concept: roughly three to six weeks.**  
One or two incident classes, perhaps three or four read-only evidence sources, structured findings, simple hypothesis engine and a basic relationship view.

**Useful internal pilot: perhaps two to three months.**  
More incident types, historical benchmark, memory, knowledge freshness, stronger UI, more connectors and real usage.

**Broader operational platform: ongoing engineering.**

The \$2,000 monthly model budget is compatible with the first several stages if the system is designed around targeted retrieval rather than brute-force context.

### The first success criterion should not be “autonomous root cause”

A much better first milestone is:

> Given one real operational report, identify what the reported thing actually is, construct a reliable picture of its immediate operational neighborhood, retrieve relevant current evidence, show several testable explanations, and let an engineer inspect every claim.

That alone would remove a huge amount of tab-opening and tribal knowledge.

### Measure the system from the beginning

I would track:

| Measurement | Question |
|---|---|
| Root-cause correctness | Did it identify the actual problem? |
| Evidence precision | Did cited evidence really support the claim? |
| Evidence coverage | Did it overlook critical evidence? |
| Time to useful context | How quickly did it make the incident understandable? |
| Hypothesis quality | Were the proposed explanations plausible and testable? |
| False-confidence rate | Did it assert things unsupported by evidence? |
| Tool-call count | How much investigation work was performed? |
| Tokens per investigation | How expensive was it? |
| Query volume | Did it abuse observability/data systems? |
| Stale-memory rate | How often was stored knowledge outdated? |
| Human correction rate | How frequently did engineers override it? |
| Permission violations | Must remain zero |
| Reproduction success | Could the reported behavior be recreated in the sandbox? |

Agent evaluations matter because nondeterministic tool-using systems can take different valid paths on repeated runs. Anthropic's 2026 agent-evaluation guidance specifically treats evaluation as a core engineering discipline rather than a final QA step. ### The decisive conclusion

Do not start with:

> “How do I index the whole company?”

Start with:

> **“How do I make one investigation trustworthy, measurable and cheap?”**

Then let successful investigations teach the system what deserves to become persistent knowledge.

The resulting progression is:

```text
READ
↓
UNDERSTAND
↓
RELATE
↓
INVESTIGATE
↓
PROVE
↓
REMEMBER
↓
VISUALIZE
↓
REPRODUCE
↓
RECOMMEND
↓
CONTROLLED ACTION
```

That is the path I see repeatedly in the strongest systems and engineering write-ups reviewed here. ## Internal-agent research and discovery handoff

The internal agent should **not** be told to implement a graph database, choose an orchestration framework, deploy infrastructure or index everything yet.

It knows things I do not know: the actual repositories, naming conventions, technologies, deployment model, internal documentation, permissions and business vocabulary.

Its first task should therefore be **environment discovery plus architecture selection**.

The following is the handoff I would put in the repository.

```text
# Operational Investigation Platform — Discovery Assignment

## Mission

Investigate this organization's real engineering and production-support
environment and determine how an evidence-grounded AI operational investigation
system could be built here.

Do not assume that any example technology, entity, protocol, architecture,
database, cloud provider, scheduler, queue, storage system, observability
platform, infrastructure tool, or deployment model mentioned in this document
exists internally.

Discover the actual environment from available evidence.

The eventual goal is an internal system capable of helping a human understand
and investigate operational problems across scattered sources while keeping
model/token usage low.

The target is not a chatbot.

The target is an operational investigation engine with:

- entity understanding
- upstream/downstream dependency understanding
- live operational evidence retrieval
- code/change correlation
- hypothesis-driven investigation
- evidence-backed conclusions
- reusable operational memory
- reusable troubleshooting skills
- historical evaluation
- human-visible investigation state
- strict permission boundaries
- optional controlled sandbox reproduction where feasible

Do not make production changes as part of this discovery.

Do not introduce new infrastructure merely because this document mentions it.

Do not commit to a graph database, vector database, agent framework, MCP,
Terraform, Kubernetes, a specific LLM, or any other implementation technology
until the real requirements justify it.

## Core problem

Operational troubleshooting knowledge is distributed.

A human responder may need to move among several systems to understand a single
problem.

Depending on the actual environment, evidence might live in some combination of:

- monitoring/observability
- alerts
- application logs
- infrastructure logs
- traces
- metrics
- scheduler/orchestrator systems
- object/file storage
- file-transfer systems
- queues/topics/event systems
- databases
- source repositories
- deployment history
- configuration
- feature flags
- Kubernetes/container systems
- infrastructure systems
- documentation
- chat systems
- incident history
- tickets
- dashboards
- local developer tooling
- external services
- business-operated processes

These are examples only.

Discover which sources really exist.

## Desired user experience

Consider a generic report such as:

    "The Output X did not arrive."

"Output X" is only an illustrative placeholder.

The eventual system should not merely search for the word.

It should determine, from internal evidence:

    What is Output X?

    Is it a file?
    A report?
    A message?
    A business process?
    A job?
    A stage?
    An internal nickname?
    Something else?

Once identified, the user should eventually be able to inspect:

    definition
    aliases
    owner
    expected behavior
    expected timing

    upstream dependencies
    producer
    triggers
    required inputs

    transport/delivery path where applicable

    downstream consumers
    downstream business or technical impact

    relevant implementation
    repository/code references
    relevant configuration

    deployments and recent changes

    observability references
    schedules
    runbooks

    related historical incidents
    known failure patterns

Every relationship should have provenance.

The system must distinguish:

    verified facts
    runtime observations
    human-declared facts
    inferred relationships
    unknowns
    stale information
    contradicted information

Never silently turn an inference into a fact.

## Investigation model

Study whether the environment can support an investigation loop like:

    1. Identify the incident scope.

    2. Resolve the affected entity or entities.

    3. Establish:
       - environment
       - relevant time range
       - ownership
       - expected behavior

    4. Retrieve a small amount of orientation context.

    5. Generate a small set of competing, testable hypotheses.

    6. For each hypothesis determine:
       - evidence supporting it
       - evidence contradicting it
       - evidence still missing
       - the cheapest/safest next check that could distinguish it
         from competing hypotheses

    7. Run targeted read-only checks using authorized tools.

    8. Reduce large tool outputs before they enter expensive model context.

    9. Record evidence as structured findings.

    10. Update, reject, or deepen hypotheses.

    11. Stop when:
        - evidence is sufficient,
        - further investigation has low expected value,
        - a configured resource/query/token budget is reached,
        - permission prevents further investigation,
        - or human assistance is required.

    12. Produce an evidence-backed conclusion or explicitly report
        that the cause remains inconclusive.

    13. Save only reviewed/reusable knowledge.

Do not design the system around unconstrained free-form model reasoning.

Prefer a visible investigation trace containing:

    hypothesis
    query/check
    tool
    source
    result
    evidence
    conclusion change
    next action

## Evidence model

Evaluate a structured evidence record resembling:

    evidence_id
    incident_id

    claim

    status:
        verified
        supported
        contradicted
        unknown

    source_type
    source_reference
    source_timestamp
    retrieval_timestamp

    environment
    entity_ids

    query/check performed

    supporting_evidence
    contradicting_evidence

    hypothesis_ids

    confidence or evidence quality

Do not treat generated prose as the primary system of record.

The human explanation may be generated from structured records.

## Entity and relationship discovery

Inspect the real environment and identify candidate entity classes.

Do not assume the following list is complete or correct.

Possible examples include:

    application
    service
    business process
    job
    schedule
    artifact/file
    storage location
    queue/topic
    event
    database
    table
    API
    repository
    code module
    dependency/library
    deployment
    environment
    infrastructure resource
    feature
    team
    alert
    incident
    runbook
    external dependency

Determine the actual internal vocabulary.

Identify alias problems.

Examples:

    service-a
    service_a
    old-service-name
    nickname used by operations

may or may not refer to the same thing.

Determine whether a canonical identifier can be built.

Study candidate relationships such as:

    produces
    consumes
    triggers
    requires
    depends_on
    calls
    publishes
    subscribes_to
    reads_from
    writes_to
    transfers_to
    scheduled_by
    implemented_by
    deployed_from
    configured_by
    owned_by
    monitored_by
    observed_on
    changed_by
    affected_by

Again, these are examples.

Discover the relationships that actually matter internally.

For every persisted relationship, consider storing:

    source
    evidence
    verification state
    environment
    observed time
    last verified time
    validity interval

## Knowledge architecture

Do not assume that all internal information belongs in one RAG/vector index.

Classify each source by the correct retrieval mechanism.

Examples:

    current telemetry
        -> targeted live query

    deployment/change records
        -> exact lookup

    scheduler state
        -> scheduler API/query

    current storage state
        -> storage query

    dependency relationships
        -> graph/catalog lookup

    code
        -> code/search/index tools

    ownership
        -> authoritative ownership system where available

    documentation/runbooks
        -> lexical + semantic search

    historical conversations
        -> scoped search

    prior incidents
        -> incident retrieval

Determine which data must be copied/indexed and which should remain at source.

Prefer source references over unnecessary duplication.

## Freshness

Determine how each important source changes.

For every source, identify whether freshness could be maintained using:

    webhooks
    events
    change feeds
    repository commits
    deployment events
    scheduled polling
    runtime observations
    periodic validation

The eventual system should recognize that stored knowledge can become stale.

Investigate a memory lifecycle:

    discovered
    candidate
    verified
    active
    stale
    contradicted
    superseded
    forgotten

Determine which memories can be automatically revalidated.

## Annotation and labeling

Do not begin a broad manual annotation program.

First determine which metadata and relationships can be extracted
deterministically from existing systems.

Identify which relationships can be learned from:

    source code
    configuration
    manifests
    scheduler definitions
    runtime telemetry
    deployment metadata
    existing catalogs
    documentation

LLM extraction from prose should create candidate facts, not automatically
trusted facts.

Human review should focus on high-value relationships and evaluation cases.

## Historical evaluation dataset

Find a small representative set of past operational incidents.

Prefer diversity over volume initially.

For each candidate incident determine whether we can reconstruct:

    original symptom
    evidence available at incident start
    affected entities
    relevant timeline
    actual root cause
    resolution
    misleading evidence
    downstream effects
    responder investigation path

Watch for future-data leakage.

Do not allow an evaluation agent to see:

    later postmortems
    fix commits that were not available yet
    later messages revealing root cause
    later documentation

unless they were actually available at the point being simulated.

Design an initial backtesting strategy.

Propose evaluation metrics including:

    root-cause correctness
    evidence correctness
    evidence completeness
    hypothesis quality
    time to useful context
    tool-call efficiency
    tokens per investigation
    stale-knowledge errors
    human correction frequency
    permission violations
    sandbox reproduction accuracy

## Procedures and skills

Study how humans currently troubleshoot recurring incident classes.

Look for repeated sequences such as:

    check X
    then Y
    if result A, inspect Z
    otherwise inspect Q

Do not blindly convert every historical sequence into a rule.

Determine which parts are:

    stable procedure
    useful heuristic
    one-time workaround
    historical accident
    obsolete behavior

Propose a "skill" format capable of storing:

    purpose
    trigger conditions
    applicable entities/environments
    required tools
    prerequisite evidence
    suggested checks
    branching conditions
    stopping conditions
    escalation conditions
    known limitations
    owner
    version
    last verification date

Skills should guide the agent while still allowing investigation outside the
skill when evidence justifies it.

## Token and cost discipline

Token efficiency is a primary architectural requirement.

Measure it from the first prototype.

Do not load:

    entire repositories
    entire log streams
    entire chat histories
    entire documentation collections
    every available tool definition

into model context.

Investigate:

    progressive retrieval
    targeted queries
    lexical search
    semantic retrieval only where useful
    reranking
    tool discovery on demand
    deterministic filtering
    result reduction
    caching
    reusable structured facts
    incremental indexing
    model routing
    explicit tool budgets
    explicit token budgets
    investigation stopping rules

For a large result set, return a small structured summary plus a source
reference rather than the complete payload when possible.

Determine which tasks require a strong reasoning model and which can use:

    deterministic code
    SQL
    graph queries
    search
    small/cheap models

## Code and deployment correlation

Determine whether the real environment permits the following chain to be built:

    runtime entity
        ->
    deployed version
        ->
    deployment event
        ->
    commit/revision
        ->
    changed files/modules
        ->
    relevant implementation

Inspect how deployments are currently recorded.

Determine whether commit/revision metadata reaches runtime telemetry.

Identify existing dependency manifests and lock files.

Study existing code-search/navigation capabilities before proposing a new index.

Do not assume that embedding the entire repository is sufficient.

## Human-visible investigation workspace

Design concepts for an interface where a human can watch an investigation.

The interface should eventually expose:

    affected entity

    dependency map

    upstream entities

    downstream entities / likely blast radius

    current hypotheses

    validated hypotheses

    rejected hypotheses

    inconclusive hypotheses

    live investigation events

    tool/query used

    evidence returned

    links to authoritative source

    recent code/config/deployment changes

    prior incidents

    proposed checks

    permission boundaries

    sandbox experiments

    proposed actions

The human must be able to inspect why the system believes a relationship or
claim.

Do not expose hidden model monologue as the product.

Expose the auditable investigation state and evidence trail.

Consider visual states for graph relationships:

    VERIFIED
    OBSERVED
    INFERRED
    HUMAN-CONFIRMED
    STALE
    DISPUTED

## Permissions and production safety

Inventory the actual permissions available to the intended users.

Separate:

    what can be observed
    what can be queried
    what can be executed
    what requires infrastructure/security/other-team approval

Do not assume that access through Kubernetes tooling permits deployment or
mutation.

The eventual agent must operate within the same authorization boundaries as
its user/service identity.

A model recommendation is not authorization.

Production writes, if ever supported, must pass through deterministic policy
and workflow controls outside the model.

Consider an autonomy ladder:

    OBSERVE
        retrieve evidence

    RECOMMEND
        recommend next action

    PREPARE
        prepare exact action/request

    EXECUTE WITH APPROVAL
        authorized human approves deterministic workflow

    NARROW POLICY-BASED EXECUTION
        only for explicitly approved low-risk failure modes

The discovery phase must remain read-only.

## Infrastructure-team boundary

Determine which operations only an infrastructure or platform team can
perform.

For those operations, design an escalation artifact containing:

    requested action
    exact target
    why it is requested
    supporting evidence
    expected result
    known risk
    verification procedure
    rollback considerations

The investigation should remain useful even when the agent cannot itself
perform the next operation.

## Sandbox and incident reproduction

Discover what lower/development/sandbox environments actually exist.

Do not assume Terraform, Kubernetes, containers, or any specific technology is
available.

Determine whether selected failures could be reconstructed using some
combination of:

    unit/process-level reproduction
    synthetic inputs
    sanitized historical inputs
    containers
    service mocks
    local infrastructure
    emulated external services
    replayed messages/events
    lower environments
    temporary namespaces/clusters
    ephemeral infrastructure
    fault injection

If infrastructure-as-code exists and users are permitted to create isolated
resources, evaluate whether it can support temporary reproduction
environments.

The agent should record environment differences and reproduction confidence.

Possible outcomes must include:

    REPRODUCED
    PARTIALLY REPRODUCED
    NOT REPRODUCED
    CANNOT REPRODUCE WITH AVAILABLE ACCESS

Never claim sandbox behavior proves production causality without supporting
evidence.

Do not automatically copy production secrets or sensitive production data into
a lower environment.

## Architecture alternatives

Compare at minimum:

    A. Incident-first architecture

    B. Knowledge/catalog-first architecture

    C. Graph-first architecture

    D. Skill/runbook-first architecture

    E. Hybrid incident-first + thin context graph + skills + evidence ledger

Do not choose based on fashion.

Choose based on observed internal requirements.

Specifically determine whether a dedicated graph database is actually needed
or whether a relational entity/relationship representation is sufficient for
the first implementation.

Likewise determine whether a dedicated vector database is required or whether
existing search/index systems already solve the problem.

## Initial implementation candidate

Evaluate, but do not blindly adopt, a minimal architecture shaped like:

    incident/question
        ->
    entity resolver
        ->
    context builder
        ->
    hypothesis planner
        ->
    tool router
        ->
    targeted queries
        ->
    result reducers
        ->
    evidence ledger
        ->
    hypothesis updater
        ->
    human-visible investigation
        ->
    reviewed memory

with independent components for:

    operational context graph
    skills
    historical evaluation
    policy/permissions
    sandbox execution

## Discovery deliverable

Produce a detailed report.

The report must include:

### Environment inventory

What systems actually exist.

### Information-source inventory

What each system knows.

### Permission matrix

What the intended agent identity can read/write and what requires another
team.

### Current troubleshooting workflow

Reconstruct several real human investigation paths.

### Entity vocabulary

What nouns actually matter internally.

### Relationship vocabulary

What dependency types actually matter.

### Identifier problems

Aliases, inconsistent names, missing environment identifiers, etc.

### Existing metadata

What relationships can already be obtained without AI.

### Existing search/index capabilities

Avoid rebuilding tools that already exist.

### Historical incident availability

Whether useful backtesting is possible.

### Sandbox capability

What can realistically be reproduced outside production.

### Knowledge freshness model

How important sources can remain current.

### Token-cost model

Estimate retrieval and model consumption for several representative
investigations.

### Architecture comparison

Compare candidate designs against the actual environment.

### Recommended narrow prototype

Select one useful incident/problem class.

Choose the minimum set of connectors needed.

Define success/failure criteria.

### Evaluation plan

Define historical cases and metrics before broad rollout.

### Risks and unknowns

Explicitly state what has not yet been established.

## Evidence requirements for this discovery

Every important conclusion must cite internal evidence where possible:

    repository path
    code symbol
    configuration path
    documentation page
    API/resource
    deployment metadata
    dashboard
    query
    incident
    historical message
    runbook

Separate:

    VERIFIED FACT
    INFERENCE
    RECOMMENDATION
    UNKNOWN

Do not present guesses as architecture facts.

## Final constraint

Do not optimize for impressive autonomy.

Optimize for:

    correctness
    evidence
    inspectability
    low token consumption
    low query volume
    freshness
    permission safety
    usefulness to a human responder

The first successful system should make a human investigation substantially
faster and easier to understand.

Autonomy can be earned later.
```

That assignment preserves the full scope you laid out without forcing an architecture on an environment I have not seen.

The major conclusion from the research is not that you need a giant knowledge graph, a giant labeling project, or a swarm of agents.

It is this:

> **Build a small, evidence-driven investigation harness. Let it query authoritative systems selectively. Give operational entities stable identities. Preserve useful relationships in a thin context graph. Store procedures separately from facts. Turn raw evidence into structured, reversible findings. Backtest against real historical incidents. Show the investigation live to the human. Give production only the permissions the human already has. Give a controlled sandbox more freedom. Make every expensive model call earn its tokens.**

That is the architecture most consistent with both your requirements and the lessons now being published by teams actually building AI SRE systems in production.