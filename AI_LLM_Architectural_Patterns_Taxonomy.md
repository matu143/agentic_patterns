# AI / LLM Architectural Patterns — Taxonomy

Author: Mauricio Trigo  
Purpose: Organize the most common architectural patterns used in LLM systems, agentic systems, and multi-agent applications.

---

# 1. What counts as an architectural pattern?

An architectural pattern defines **how control, responsibility, coordination, and capabilities are distributed** inside an AI system.

Not everything is a pattern.

Useful distinction:

- **Pattern** = structure of coordination or execution
- **Mechanism** = implementation vehicle used by a pattern
- **Capability package** = reusable specialized behavior or knowledge bundle

Example:

- `subagent delegation` → pattern
- `delegation tool` → mechanism
- `skill` → capability package or modular specialization pattern

---

# 2. Classification of the terms you mentioned

## Agent as Tool

Type: Architectural pattern

Definition:

A specialized agent is wrapped and exposed like a callable tool for a main agent.

Use when:

- you want centralized control
- you want the supervisor to own the main workflow
- the sub-agent should not directly converse with the user

Notes:

This is closely related to the **subagents pattern** in LangChain.

---

## Subagent Delegation

Type: Architectural pattern

Definition:

A main agent delegates subtasks to specialized agents.

Use when:

- tasks span multiple expertise areas
- one agent alone becomes too overloaded
- specialized reasoning is needed by domain

Notes:

This is a core multi-agent pattern and appears in both LangChain and Anthropic workflows.

---

## Delegation Tools

Type: Mechanism

Definition:

Tools used by an agent to delegate work to another agent or worker.

Use when:

- delegation is implemented through tool calls
- the supervisor uses a formal interface to call specialists

Notes:

This is usually not the pattern itself, but the **mechanism that implements delegation**.

---

## Skills

Type: Capability modularization pattern

Definition:

Reusable specialized instruction bundles, sometimes including scripts and resources, loaded on demand when relevant.

Use when:

- you want progressive disclosure of specialized knowledge
- you want reusable prompt modules
- you want a cleaner alternative to giant system prompts

Notes:

Anthropic defines Skills as folders with instructions, scripts, and resources that Claude can load when needed. LangChain also documents a **skills pattern** as progressive disclosure of specialized prompts and domain knowledge.

---

## Orchestrator

Type: Architectural pattern

Definition:

A central controller coordinates workers, agents, steps, and outputs.

Use when:

- workflow control must remain centralized
- multiple workers are involved
- planning and aggregation happen at the top

Notes:

Anthropic explicitly describes orchestrator-worker style systems in its agent guidance and research system. LangChain's supervisor pattern is a close equivalent.

---

# 3. Core pattern families

---

## A. Single-Agent Patterns

These patterns use one main agent, sometimes with tools, memories, or modular prompts.

### 1. Tool-Calling Agent Loop

Definition:

The agent iteratively decides whether to answer directly or call a tool.

Flow:

User Input  
→ Agent Reasoning  
→ Tool Call  
→ Tool Result  
→ Agent Continues  
→ Final Answer

Use when:

- the task is dynamic
- tool use is needed
- the flow does not need many specialized agents

---

### 2. Skills Pattern

Definition:

A single agent loads specialized instructions and knowledge only when needed.

Use when:

- you want modular prompting
- you want reusable expertise packs
- you want to reduce giant static prompts

References:

LangChain documents this pattern explicitly, and Anthropic's Skills system follows the same idea.

---

### 3. State Machine Agent

Definition:

A single agent changes its instructions and tools depending on the current task state.

Use when:

- behavior changes across phases
- customer support or guided flows are needed
- a deterministic progression is useful

References:

LangChain's handoff tutorial explicitly describes a state machine implementation using tool calls and dynamic configuration.

---

## B. Multi-Agent Coordination Patterns

These patterns distribute work across multiple agents.

### 4. Supervisor / Orchestrator Pattern

Definition:

A central agent coordinates specialized worker agents.

Use when:

- workflow must remain under central control
- specialized workers are useful
- aggregation and consistency matter

References:

LangChain's supervisor pattern and Anthropic's orchestrator-worker framing both support this architecture.

---

### 5. Subagents Pattern

Definition:

A main agent delegates tasks to domain-specific subagents.

Use when:

- domains are clearly separable
- specialists do not need user-facing interaction
- centralized workflow control is still desired

References:

LangChain documents this explicitly.

---

### 6. Agent as Tool

Definition:

A special case of subagents where the subagent is invoked exactly like a tool.

Use when:

- you want strict contracts around delegation
- you want simple integration with a supervisor agent
- you want clean I/O boundaries

---

### 7. Handoffs Pattern

Definition:

Control moves from one agent to another rather than staying with a permanent supervisor.

Use when:

- responsibility should pass between agents
- different phases require different personas or toolsets
- conversation ownership changes over time

References:

LangChain documents handoffs as a distinct multi-agent pattern and shows two implementation approaches.

---

### 8. Router Pattern

Definition:

A routing layer decides which model, agent, or workflow should handle the request.

Use when:

- requests vary by category
- explicit routing logic is needed
- cost/latency specialization matters

References:

LangChain documents router separately and contrasts it with subagents.

---

## C. Workflow Patterns

These patterns decompose work into stages or branches.

### 9. Prompt Chaining

Definition:

One LLM step feeds the next in a predefined sequence.

Use when:

- the task is easy to split into clean substeps
- reliability matters more than flexibility
- each step simplifies the next

References:

Anthropic lists prompt chaining as a core workflow pattern.

---

### 10. Parallelization

Definition:

Multiple workers or branches process subtasks simultaneously.

Use when:

- tasks can be decomposed independently
- speed matters
- synthesis can happen afterward

References:

Anthropic includes parallelization among its recommended agent workflow patterns.

---

### 11. Evaluator-Optimizer

Definition:

One component produces an output and another evaluates, critiques, or improves it.

Use when:

- quality control is important
- hallucination risk must be reduced
- iterative refinement helps

References:

Anthropic explicitly includes evaluator-optimizer as a reusable workflow pattern.

---

### 12. Plan-and-Execute

Definition:

One module plans the task, then one or more executors perform the steps.

Use when:

- tasks are complex and benefit from decomposition
- planning should be separate from execution
- execution can be monitored or rerouted

Notes:

This is a practical derivative of orchestrator and workflow decomposition patterns.

---

## D. Retrieval and Knowledge Patterns

These patterns add external knowledge to reasoning.

### 13. Retrieval-Augmented Generation (RAG)

Definition:

The model retrieves external documents before answering.

Use when:

- the answer depends on external knowledge
- freshness matters
- citations or grounding are required

---

### 14. Retrieval-Augmented Agent

Definition:

An agent does not just retrieve once; it can iteratively rewrite queries, rerank, search again, and synthesize.

Use when:

- static RAG is too weak
- search is multi-step
- the system must explore and refine

References:

Anthropic's multi-agent research system contrasts static RAG with iterative research and coordinated search.

---

### 15. Knowledge Routing

Definition:

The system routes different queries to different knowledge sources or indexes.

Use when:

- multiple corpora exist
- domain separation matters
- retrieval quality depends on source choice

References:

LangChain's routed knowledge base tutorial shows this idea.

---

## E. Memory Patterns

These patterns govern persistence across turns or tasks.

### 16. Working Memory Pattern

Definition:

The agent maintains short-lived context for the current task.

Use when:

- temporary scratch context is needed
- the current task has multiple steps

---

### 17. Episodic Memory Pattern

Definition:

The system stores past interactions or completed trajectories for reuse.

Use when:

- repeated tasks occur
- prior outcomes matter
- agent improvement benefits from history

---

### 18. Semantic / Long-Term Memory Pattern

Definition:

The system stores stable facts, concepts, and distilled knowledge over time.

Use when:

- the agent should remember durable user/project knowledge
- context windows alone are not enough

---

## F. Control, Safety, and Governance Patterns

These patterns reduce risk or add supervision.

### 19. Human-in-the-Loop

Definition:

A human reviews or approves specific steps before they execute.

Use when:

- actions are high risk
- external side effects happen
- legal/compliance concerns exist

References:

LangChain provides human-in-the-loop middleware for risky tool actions.

---

### 20. Guardrail / Policy Check Pattern

Definition:

A validation layer checks constraints before or after model actions.

Use when:

- outputs must obey rules
- tool calls must be constrained
- compliance matters

---

### 21. Critic / Reflection Loop

Definition:

The system inspects its own output and attempts to improve it.

Use when:

- tasks benefit from self-review
- the first answer is often incomplete
- synthesis quality matters

This often overlaps with evaluator-optimizer.

---

# 4. Quick taxonomy by level

## True architectural patterns

- orchestrator / supervisor
- subagents
- handoffs
- router
- agent as tool
- prompt chaining
- parallelization
- evaluator-optimizer
- plan-and-execute
- retrieval-augmented agent
- human-in-the-loop

---

## Mechanisms

- delegation tools
- tool calls
- middleware
- prompt templates
- dynamic configuration
- graph edges and state transitions

---

## Capability packaging patterns

- skills
- prompt modules
- domain packs
- reusable reasoning packs

---

# 5. Recommended mental model

Use this rule:

- **Pattern** = who controls what, and how work is divided
- **Mechanism** = how the pattern is technically implemented
- **Capability package** = reusable expertise module

Examples:

- orchestrator = pattern
- delegation tool = mechanism
- skill = capability package
- subagent = architectural unit
- router = coordination pattern

---

# 6. Minimal list every AI engineer should know

If you want a compact set of patterns worth mastering first:

1. Tool-calling agent loop
2. Router
3. Orchestrator / Supervisor
4. Subagents
5. Handoffs
6. Skills
7. Prompt chaining
8. Parallelization
9. Evaluator-optimizer
10. Human-in-the-loop
11. Retrieval-augmented agent
12. Plan-and-execute

---

# 7. Final insight

The biggest source of confusion in agentic systems is mixing up:

- architecture
- orchestration
- prompting
- tools
- reusable capabilities

A strong system usually combines several patterns at once.

Example:

Router  
→ Orchestrator  
→ Subagents  
→ Skills  
→ Retrieval  
→ Evaluator  
→ Human approval

That is not one pattern.

That is an **AI architecture composed of multiple patterns**.