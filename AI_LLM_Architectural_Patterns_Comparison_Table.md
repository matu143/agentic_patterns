# AI / LLM Architectural Patterns — Comparison Table

Author: Mauricio Trigo  
Purpose: Compare the most common AI/LLM architectural patterns by problem, tradeoffs, and LangGraph implementation strategy.

---

## Quick reading rule

- **Pattern** = how control and work are distributed
- **Mechanism** = how that pattern is technically implemented
- **Capability package** = reusable specialization that plugs into a pattern

---

| Pattern | Type | What problem it solves | Pros | Cons | Best for | LangGraph implementation idea |
|---|---|---|---|---|---|---|
| **Tool-calling agent loop** | Single-agent pattern | Lets one agent decide when to answer vs when to call tools | Simple mental model, flexible, fast to prototype | Can become messy with too many tools, weaker specialization boundaries | Small-to-medium agents, assistants, RAG with a few tools | One main agent node with tool bindings and loop until stop condition |
| **Skills** | Capability modularization pattern | Avoids giant monolithic prompts by loading specialized knowledge only when needed | Prompt modularity, progressive disclosure, easier maintenance, lighter than subagents | Less hard isolation than full subagents, can become prompt spaghetti if unmanaged | Coding assistants, domain packs, reusable expertise modules | `load_skill()` tool, state flag for active skills, dynamic prompt/context injection |
| **State-machine agent** | Single-agent control pattern | Changes behavior by phase or task state without needing multiple free-floating agents | Deterministic transitions, easier governance, good for workflows | Less flexible than full dynamic delegation | Customer support, guided flows, multi-step business processes | StateGraph with explicit state variable and transition edges |
| **Router** | Coordination pattern | Sends a request to the right specialist or workflow using explicit routing logic | Clean dispatch, easy to reason about, can be rule-based or LLM-based | Usually stateless unless you add state, weaker for long multi-turn orchestration | Multi-source KB, intent classification, domain dispatch | Router node that classifies input and routes to subgraph/agent nodes |
| **Subagents** | Multi-agent pattern | Splits work across specialists while keeping central control in a supervisor | Strong specialization, context isolation, centralized control, parallelizable | More calls, more orchestration overhead, more moving parts | Personal assistants, enterprise agents, multi-domain tasks | Supervisor node calling specialized subagents as tools or subgraphs |
| **Agent as tool** | Multi-agent pattern | Treats a specialist agent as a callable tool with a clean contract | Easy integration into a supervisor, strong boundaries, reusable agent units | Still inherits multi-agent latency/cost, can hide complexity behind "tool" label | Research agent, legal reviewer, SQL agent, domain specialists | Wrap subagent invocation in a tool function and bind to main agent |
| **Handoffs** | Multi-agent or dynamic-config pattern | Transfers control from one agent/configuration to another | Good when ownership changes by phase, natural for workflows with distinct personas | Can complicate state and ownership tracking | Customer support, onboarding, multi-phase task flows | Middleware-based dynamic config or agent subgraphs with explicit handoff edges |
| **Prompt chaining** | Workflow pattern | Breaks a difficult task into clean sequential steps | Higher reliability, simpler subtasks, easy gating between steps | Higher latency, brittle if decomposition is poor | Document generation, transformation pipelines, structured extraction | Linear graph: step A → validation/gate → step B → step C |
| **Parallelization** | Workflow pattern | Runs independent subtasks simultaneously to reduce total time or widen coverage | Speed, breadth, good for search and comparison tasks | Merge/synthesis step can get tricky, extra coordination needed | Multi-source retrieval, research, independent analyses | `Send` / parallel branches to worker nodes, then aggregation node |
| **Evaluator-optimizer** | Workflow / quality pattern | Adds a critique or judge step to improve output quality | Better reliability, catches bad outputs, useful for refinement | More cost/latency, can overcomplicate simple tasks | High-value outputs, compliance, report generation, code review | Generator node → evaluator node → retry/refine loop |
| **Plan-and-execute** | Workflow / orchestration pattern | Separates planning from execution so big tasks become manageable | Better decomposition, clearer control, easier monitoring | Planner can hallucinate bad plans, extra overhead | Long-horizon tasks, research, automation, batch workflows | Planner node produces plan in state, executor nodes consume plan steps |
| **Orchestrator / Supervisor** | Multi-agent coordination pattern | Keeps one central brain coordinating multiple workers | Clear governance, centralized memory, easier policy control | Bottleneck risk, added latency, more complex supervisor prompt | Enterprise workflows, assistants with many domains, compliance systems | Supervisor node + worker nodes or subagents, aggregation in supervisor |
| **Retrieval-Augmented Generation (RAG)** | Knowledge pattern | Grounds answers in external documents | Better factuality, fresher knowledge, supports citations | One-shot retrieval often misses nuance, retrieval quality is everything | QA over docs, enterprise search, policy assistants | Retriever node → reranker node → answer node |
| **Retrieval-Augmented Agent** | Agentic knowledge pattern | Allows iterative search, rewrite, rerank, and synthesis rather than one-shot retrieval | More robust than static RAG, better for ambiguous or hard questions | More complexity, more cost, harder debugging | Deep research, compliance, evidence packs, exploratory QA | Agent loop with retrieval tools plus query rewrite / rerank / synthesize nodes |
| **Knowledge routing** | Retrieval coordination pattern | Routes questions to the right corpus/index/source | Improves relevance and reduces noisy retrieval | Requires index/source taxonomy and routing logic | Multi-corpus systems, Slack + Notion + GitHub + docs | Router node chooses source-specific retriever subgraph |
| **Working memory** | Memory pattern | Maintains short-lived task context through a workflow | Improves coherence during current task | Can bloat quickly if unmanaged | Any multi-step task | Keep working state in graph state schema |
| **Episodic memory** | Memory pattern | Stores previous task trajectories or outcomes for reuse | Learns from prior executions, useful for repeated tasks | Retrieval and curation become their own problem | Recurrent workflows, assistants with repeated jobs | Save execution summaries/episodes to store, retrieve before planning |
| **Semantic / long-term memory** | Memory pattern | Stores durable facts, concepts, and distilled knowledge across time | Better personalization and continuity | Risk of stale or polluted memory | User memory, project memory, domain knowledge memory | Vector store or structured memory store queried before/during execution |
| **Human-in-the-loop** | Control/safety pattern | Inserts human approval or review before risky actions | Strong safety, trust, governance | Slower, adds operational friction | Emails, purchases, destructive actions, compliance decisions | Interrupts / approval nodes before side-effect tools |
| **Guardrails / policy checks** | Safety pattern | Enforces constraints before or after model outputs/actions | Better compliance and reliability | Can become rigid, needs maintenance | Regulated domains, enterprise apps | Validator nodes, middleware, output schema checks, tool permission checks |
| **Critic / reflection loop** | Quality pattern | Lets the system inspect and improve its own output | Often boosts quality without humans | Can spiral into overthinking or wasted calls | Writing, synthesis, code, high-stakes reasoning | Answer node → critic node → revise node with stop criteria |

---

## Selection heuristics

### Use **Router** when:
- you want explicit dispatch logic
- routing can happen in one step
- you need control over preprocessing or parallel source selection

### Use **Subagents** when:
- multiple specialist domains exist
- you want centralized supervision
- the main agent should keep conversation ownership

### Use **Handoffs** when:
- ownership should move between phases
- one configuration or agent should "take over" from another
- the conversation flow naturally changes roles

### Use **Skills** when:
- one agent needs many specializations
- you want modular prompt packs
- full subagents would be overkill

### Use **Prompt chaining** when:
- the task decomposes cleanly into fixed steps
- you prefer predictability over autonomy

### Use **Evaluator-optimizer** when:
- output quality matters more than raw speed
- a first draft is rarely enough

### Use **Retrieval-augmented agent** when:
- one-shot RAG is too weak
- the task requires search, rewrite, compare, rerank, and synthesize

---

## LangGraph-oriented mental model

### Minimal stack
- **StateGraph** for workflow state
- **Router node** for dispatch
- **Supervisor node** for orchestration
- **Worker subgraphs** for specialized execution
- **Interrupt nodes** for HITL
- **Validation nodes** for guardrails
- **Memory store** for long-term context

---

## Example composed architecture

User Query  
→ Router  
→ Supervisor  
→ Subagents  
→ Skills loaded on demand  
→ Retrieval loop  
→ Evaluator  
→ Human approval if risky  
→ Final Answer

---

## Important distinction

A production system often combines several patterns.

Example:

- Router for intent/domain dispatch
- Supervisor for orchestration
- Subagents for specialization
- Skills for lightweight modular knowledge
- Retrieval for grounding
- Evaluator for quality
- Human-in-the-loop for risky actions

That is not "one architecture pattern".

That is a **composed AI architecture** built from multiple patterns.