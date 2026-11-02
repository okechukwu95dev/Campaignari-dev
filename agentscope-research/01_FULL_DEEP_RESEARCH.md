# Claude Code “Brain UI”: Token Anatomy, Subagent Monitoring, and 3D Agent Workspaces

## What the tweet is actually pointing toward

The Spotify Shunt thread is useful because it exposes a deeper problem than “Claude is expensive”: **you normally cannot see, at a glance, what is occupying Claude Code’s working context, which operations are repeatedly inflating it, which subagents are consuming resources, and whether the expensive part of a task is actual reasoning or simply moving large amounts of text through the model.**

Spotify’s Shunt plugin attacks one specific part of that problem. Its `PreToolUse` hooks intercept large file reads—350 lines by default—and redirect the file to a cheaper worker instead of putting the entire file into Claude’s main context. Spotify reports 82–94% savings for the intercepted large-file scenarios in a 162,000-line Java monorepo, with a mean bulk-read reduction around 90%. The plugin explicitly does **not** delegate debugging, architecture, editing, or other tasks where the primary Claude model needs exact source context. The viral X posts accurately describe that mechanism, but the headline “90% token reduction” is easy to misread. Justin Johnson, whose English thread circulated widely, later tested Shunt’s routing criteria against 30 days of his own Claude Code history—574 sessions and roughly 174 million characters—and reported that the hooks would have intercepted only about **3.5% of his context**, with the Bash-read hook firing just five times. His conclusion was essentially that Spotify’s savings are real for the reads Shunt intercepts, but the fraction of your workload eligible for interception determines whether the technique matters. That makes your proposed “brain UI” arguably more useful as a first step than installing Shunt. Before optimizing, you want a visualization that answers:

- What is in this session’s effective context?
- Which prompt or tool call caused the context to jump?
- How much is fresh input versus cache read versus cache creation versus output?
- Which files are repeatedly being read?
- Which tools are returning oversized results?
- Which subagent consumed which tokens?
- How much context is being re-sent because of conversation history?
- What can be eliminated, isolated in a subagent, summarized, cached, or routed elsewhere?

Claude Code already records enough local information to build much of this. Its session transcripts under `~/.claude/projects/` contain per-message usage data including input, output, prompt-cache reads and prompt-cache creation, model identifiers, tool calls and timestamps; newer Claude Code layouts also keep subagent transcripts separately beneath the parent session. What does **not** exist is a literal view into Claude’s private internal state. A dashboard can reconstruct the observable context, messages, tool results, files, cache accounting and agent hierarchy. It cannot expose an exact semantic representation of “what Claude is thinking” or a faithful map of latent model state. Anthropic does expose thinking controls and token-related usage at the API/model level, but the local Claude Code usage records most third-party dashboards consume are centered on input, output, cache-read and cache-write accounting rather than a complete introspection feed of private model reasoning. ## What you can get with only a Claude Pro or Max account

A normal Claude Pro or Max subscription can authenticate Claude Code directly; an Anthropic Console/API account is not required just to use Claude Code. That distinction matters because many of the most useful monitoring tools do **not** call another model at all. They simply parse the transcripts Claude Code already writes locally. Therefore they do not inherently consume extra Claude tokens and do not require Spotify Portal, Gemini, OpenAI, or an Anthropic API key. The closest architecture to your idea is therefore:

```text
                    CLAUDE CODE SESSION
                           │
            ┌──────────────┴──────────────┐
            │                             │
      Main agent                    Subagents
            │                             │
            └──────────────┬──────────────┘
                           │
                     local JSONL logs
                           │
             ~/.claude/projects/...
                           │
             ┌─────────────┴─────────────┐
             │                           │
       "Brain" dashboard           Agent monitor
       context + tokens            hierarchy + status
             │                           │
             └─────────────┬─────────────┘
                           │
                   optional 3D UI
```

Because the observability layer is read-only, it can be almost free in token terms. The main cost is local CPU, memory and browser rendering rather than model usage. Tools that add LLM-generated summaries, titles or digests are different: those features can make additional model calls and should be disabled if the objective is pure measurement. One monitor, for example, can optionally generate titles through a background Haiku API call, while its basic transcript monitoring does not require that feature. Anthropic itself recommends subagents for exploratory work where preserving the parent context is valuable. Its Claude Code guidance specifically notes that subagents can investigate independently without filling the main context with all of their intermediate exploration. That means your idea naturally divides into **three different UIs** rather than one monolithic tool:

| Layer | What it should show | Can be local/no extra tokens? |
|---|---|---|
| Context brain | Current context occupancy, files, tools, conversation history | Yes |
| Cost brain | Input/output/cache creation/cache read, cost estimates, per-turn attribution | Yes |
| Agent lobby | Parent/subagent hierarchy, activity, current task, duration, context/cost | Yes |
| 3D office | Avatars representing those same agents | Yes, if visualization only |
| Semantic “reasoning brain” | Exact hidden reasoning/state | No |

## The strongest “brain UI” tools I found

There are now enough Claude Code observability projects that the problem is no longer “does this exist?” but rather “which implementation is mature enough and conservative enough to run on a work machine?”

### Token Dashboard is closest to the token-forensics version of your idea

`nateherkai/token-dashboard` reads Claude Code JSONL files directly, keeps a local SQLite index and exposes a browser dashboard on `127.0.0.1`. Its documented views include per-prompt costs, per-session turn breakdowns, tool/file heatmaps, cache analytics, model/project comparisons, subagent attribution and rules that identify patterns such as repeated file reads or oversized tool responses. It states that the dashboard performs no telemetry, no API calls for user data and does not modify transcripts. Its Prompts view is particularly relevant to your request: an expensive prompt can be opened to inspect the assistant response, tool calls and the size of individual tool results. The Sessions view reconstructs the conversation turn by turn. This is much closer to “why did this particular question consume so much?” than a simple monthly token meter. For a work machine, one useful security property is that it defaults to `127.0.0.1`. Its documentation explicitly warns that binding it to `0.0.0.0` would expose prompt history to other machines on the network. **Verdict:** best current fit for retrospective token archaeology and prompt/file/tool attribution.

### Claude Code Monitor is closer to a live instrumentation console

`kchernev/claude-code-monitor` is more ambitious. It advertises live session monitoring, subagent tracking, workflow fan-outs, token composition, token velocity and API-equivalent cost derived from local Claude Code transcripts. Its token model distinguishes fresh input, re-sent context, cache reads and output, and it can group subagent runs by type with per-agent cost and duration. That “fresh versus re-sent” distinction is especially valuable. A raw `input_tokens` number alone cannot tell you whether a prompt itself was huge or whether the same accumulated context was being shipped through successive turns. A tool that reconstructs that difference comes much closer to showing “what part of the brain is taking up space.” It does have one optional network feature: the plan-limit card can contact Anthropic’s usage endpoint using Claude Code’s existing OAuth token. The project documents a `--no-network` mode that disables this and falls back to Claude Code’s cached data. **Verdict:** strongest candidate if you want one interface combining live context growth, cost and subagent activity.

### MixMe’s Claude Code Usage Dashboard is the lightweight accounting choice

`MixMe/claude-code-usage-dashboard` is deliberately simpler. It uses a Rust collector and static HTML output, with daily usage by model, token type, subagent, project, skill and session. It also maintains an archive so historical usage remains available even after old Claude Code transcripts disappear. Its documented token categories are exactly the core accounting fields most useful for a Pro/Max user:

```text
input
output
cache read
cache write
```

It then calculates API-equivalent cost rather than pretending that those dollar figures represent what a flat-rate Pro/Max subscriber was actually billed for each individual conversation. **Verdict:** lowest-complexity choice if you primarily care about longitudinal token/cost accounting.

### Agent Monitor is closer to the “open a subagent and look inside” idea

`karamanburak/agent-monitor` is a fully local live dashboard specifically for Claude Code sessions and subagents. It receives Claude Code lifecycle events through hooks, presents the agent hierarchy and trace waterfall, and binds its UI/API to localhost. Its documentation states that the event log stays locally. That is materially different from a passive billing dashboard. Hooks allow the UI to know that an agent just started a tool call instead of waiting for a transcript scan to notice after the fact. Claude Code officially supports lifecycle hooks such as `PreToolUse`, `PostToolUse`, `UserPromptSubmit` and `SessionStart`, which is the same extension mechanism Spotify uses for Shunt. Anthropic explicitly warns that Claude Code hooks execute commands with the user’s permissions and therefore should only be installed from sources you trust and inspect. That warning is especially important on a corporate machine. **Verdict:** closest fit for a live “mission control” view of subagents, but carries more installation/security surface than a pure transcript reader.

### PoolCoder and ccmon are good alternatives if you prefer terminal-first monitoring

PoolCoder tails Claude Code transcripts read-only and exposes current context size, token spend, cost, running subagents, workflow progress and plan limits through terminal/web/desktop variants. `ccmon` focuses specifically on listing, watching and attaching to Claude Code agents and subagents in a terminal UI. These are relevant at work because a TUI avoids running a large browser application or 3D renderer just to answer “which agent is doing what?”

## The 3D agent office is real, but separate from useful observability

I found two projects that match the Twitter clips much more closely than the generic “AI civilization” projects.

### Roost is almost exactly the visualization you described

Roost describes itself as a **live 3D office for Claude Code agents**. It reads local Claude Code transcripts and renders up to four agents as animated workers at desks. Active agents sit and type; idle agents walk around, use the water cooler or couch; offline agents leave the office. Their monitors display short activity labels derived from current transcript events. Its architecture is unusually appropriate for work use because it is deliberately read-only. The server exposes GET routes only, does not let the visualization send commands to agents, reduces transcript content to short activity labels, applies redaction before serialization, binds to localhost by default and advertises no telemetry or external calls. The visual layer is React/Three.js and polls/streams agent state on roughly a 2.5-second cycle. It has a fallback card UI if WebGL is unavailable and a performance watchdog that reduces graphical quality on weaker hardware. This is therefore **not just a fake Sims demo**. It is actually wired to Claude Code transcript activity. It does not, however, provide the detailed token brain you want. It should be thought of as a visualization frontend that could sit beside a real observability dashboard. ### Agent Office goes much further, and that is precisely why I would not start with it at work

Agent Office creates a shared 3D office where workers can run Claude Code, Codex and other coding agents. Each worker has a visible terminal; users can walk to desks, inspect terminals, assign tasks, manage worktrees and PRs, and even allow agents to hire/message other agents through MCP. It supports Claude subscription authentication as well as API-key authentication. That is much closer to a **real multi-agent operating environment** than a monitor. Consequently it also has a much larger trust boundary: terminals are interactive, repositories can be cloned, GitHub credentials can be made available to the environment, agents can create worktrees and PRs, and remote/team deployment is supported. Its own documentation notes that users sharing the office can potentially use the GitHub credential passed into it and recommends finer-grained tokens or disabling GitHub credential sharing where appropriate. For a personal sandbox this is interesting. For a locked-down corporate workstation, it is categorically different from a read-only transcript viewer.

**Verdict:** impressive, but overpowered for your monitoring objective.

## What the “brain” can and cannot measure accurately

The most important distinction is between **token accounting** and **context attribution**.

Claude’s underlying usage schema can expose token accounting fields such as normal input, output, cache-read input and cache-creation input. Server-side tools can also have separate request counters. Anthropic’s own examples expose those categories directly in the API usage object. Claude Code transcript analyzers can therefore calculate totals such as:

```text
turn 42
────────────────────────────────
fresh input             4,812
cache creation          9,104
cache read            118,330
output                   2,941
────────────────────────────────
approx API-equiv cost     $...
```

What becomes harder is answering:

```text
Of those 118,330 cached tokens:

31,402 = repository files
18,223 = tool definitions
12,890 = CLAUDE.md + instructions
44,281 = conversation history
11,534 = previous tool results
```

That level of attribution generally has to be reconstructed from transcript structure rather than read directly from a provider-supplied billing field. This is why different dashboards may disagree about the apparent cost of a file, prompt, skill or tool despite agreeing on the underlying token totals. The local transcript itself is also an implementation detail whose format has changed over time rather than a stable public analytics API. Subagents are easier to isolate because newer Claude Code versions maintain separate subagent transcript files, allowing dashboards to attribute their activity independently from the parent session. There is another important accounting trap: **cache tokens are not equivalent to uncached input tokens economically.** A high cache-read number may represent a large amount of context but be substantially cheaper than repeatedly ingesting fresh uncached content. Consequently, a UI should visualize at least two dimensions:

```text
SPACE                     COST
How much context?         How expensive was it?
────────────────         ─────────────────────
cache read  65%          cache read    22%
fresh input 15%          fresh input   31%
cache write 12%          cache write   27%
output       8%          output        20%
```

Treating all token classes as equivalent would produce a misleading “brain.” Anthropic’s API exposes the categories separately specifically because caching changes how prompt processing is accounted for. The same applies to reasoning. Current Claude models can use adaptive thinking, and Anthropic documents that higher effort can increase thinking and latency. That does **not** make a local Claude Code dashboard a debugger for Claude’s hidden reasoning process. The useful observable is how much output/model effort was consumed and what actions resulted—not a browsable private chain of thought. ## What is acceptable to run at work

For a work machine, I would distinguish **read-only observability** from **agent-control infrastructure**.

| Tool | Extra model calls required | Reads source/prompt contents | Hooks | Network by default | Local resource load | Work-machine assessment |
|---|---:|---:|---:|---:|---:|---|
| MixMe Usage Dashboard | No | Local transcripts | No | None documented | Very low | Best conservative choice |
| Token Dashboard | No | Yes, transcript contents | No | Localhost only | Low | Strong |
| Claude Code Monitor | No for core monitoring | Yes | No for core transcript monitoring | Optional usage request | Low | Strong with `--no-network` |
| ccmon | No | Local transcripts/process state | No | Local | Very low | Strong |
| Agent Monitor | No | Events/transcripts | Yes | Localhost | Low–medium | Review hooks before use |
| Roost | No | Reads transcripts but emits labels | No required for basic operation | Localhost | Medium/GPU | Safe design, mostly decorative |
| Agent Office | Can run subscription workers | Full repo/terminals | Multiple integrations | Local/server capable | High | Not my first choice at work |

The security issue with the passive dashboards is not primarily token cost. It is **source-code and prompt confidentiality**. A Claude Code transcript can contain code, shell output, file paths, prompts, tool results and potentially secrets accidentally printed by commands. Any dashboard that reads `~/.claude/projects/` therefore has access to sensitive material even if it never transmits it. Token Dashboard explicitly warns against exposing its local server to the LAN; Roost deliberately converts raw events into labels and redacts serialized values; Claude Code Monitor provides a no-network mode. Hooks deserve greater scrutiny because Anthropic’s own documentation warns that they execute arbitrary commands under the current user account. Spotify’s Shunt relies on hooks precisely because this gives it the power to intercept and block Claude’s tool calls. For a corporate environment with no formal approval for third-party agent tooling, the lowest-risk architecture is therefore:

```text
Claude Code
    │
    ├── unchanged
    │
    └── read-only local transcript
             │
             └── local parser
                   │
                   └── localhost/static UI
```

No MCP server, no Claude hook, no API key, no second model, no external telemetry and no command/control capability.

## The setup I would choose for your exact goal

Your desired UI is actually the combination of **Claude Code Monitor or Token Dashboard plus Roost**, not Shunt.

The practical stack would be:

```text
┌──────────────────────────────────────────────────────┐
│                   SESSION BRAIN                      │
│                                                      │
│ Context: 71%                                         │
│ ███████████████████████████████░░░░░░░░░░░░         │
│                                                      │
│ Context composition        Token economics           │
│ Files        38%           Cache read        58%      │
│ Tool output  24%           Fresh input       18%      │
│ History      17%           Cache creation    14%      │
│ Instructions 11%           Output            10%      │
│ Other        10%                                    │
│                                                      │
│ Biggest objects                                      │
│ PaymentService.java          21,440 tokens           │
│ npm test output              17,901 tokens           │
│ previous grep results        11,203 tokens           │
│ CLAUDE.md                     4,110 tokens           │
└──────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────┐
│                   AGENT LOBBY                        │
│                                                      │
│ Main Claude       working       $1.82    ctx 71%     │
│ ├─ explore-auth   complete      $0.21    18k tok     │
│ ├─ find-tests     working       $0.14    11k tok     │
│ └─ inspect-db     waiting       $0.09     7k tok     │
│                                                      │
│ click agent → transcript / tools / files / tokens    │
└──────────────────────────────────────────────────────┘
```

The first panel is already technically achievable from the data being collected by the better monitoring tools, although no project I found presents **all** of those dimensions in exactly this integrated form. Token Dashboard has the strongest prompt/file/tool forensic views; Claude Code Monitor has the strongest live token/context/subagent story; Agent Monitor adds richer real-time agent traces. Roost can provide the optional “Sims office” view without becoming the control plane. That separation is desirable: observability remains factual and lightweight while the 3D representation is merely another visualization of the same session state. I would rank the current options for your use case as follows:

1. **Claude Code Monitor** — best approximation of a live brain + subagent monitor in one program. It exposes context/token composition, agents and cost while remaining primarily transcript-driven. Use its no-network mode on a work machine. 2. **Token Dashboard** — best forensic view of *why* a particular prompt/session became expensive, especially tool results and repeated file reads. Fully local architecture and no model calls. 3. **MixMe Usage Dashboard** — best lightweight audit/accounting layer when minimal dependencies matter more than flashy live visualization. 4. **Agent Monitor** — strongest live subagent-oriented interface, but I would inspect and approve its hook scripts before placing it in a corporate development environment. 5. **Roost** — the best match to the 3D Twitter aesthetic while remaining read-only; useful as an ambient second-screen display rather than the primary diagnostic tool. 6. **Agent Office** — much more powerful than necessary. It is an orchestration environment, not merely a monitor, so the security and operational surface is significantly larger. The key finding is that **you do not need another model, an API account, Spotify Portal, or a token-consuming Claude skill to build most of the “brain UI.”** Claude Code is already leaving an observable event trail on disk. The safest and cheapest architecture is to treat that trail as telemetry and visualize it externally. Shunt then becomes a second-stage experiment rather than the starting point. Once the dashboard shows that, for example, 35% of expensive uncached context is coming from large exploratory file reads, there is evidence for routing those reads to an isolated subagent or cheaper worker. If instead the dashboard shows that your dominant cost is cached conversation history, test output, MCP schemas, or repeated subagent exploration, Shunt would be optimizing the wrong component. Spotify’s own design and the follow-up measurements from the viral thread support exactly that interpretation: **measure the shape of your workload before adopting the routing optimization.**