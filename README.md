# Context Layer

> Current release: v2.2 — Persistent Constraint State + Unified Execution + MCP Interface

Runtime execution infrastructure for reliable LLM systems.

---

## What it is

Context Layer is a runtime execution boundary between your application and the model provider. It controls how every LLM request is executed, not just what is sent.

It is not a framework. It does not orchestrate tools or manage agents. It governs execution: whether a request is admitted, what context the model receives, whether constraints are enforced, and whether execution should continue after the model responds.

At its core, Context Layer separates:

* **generation** (what the model produces)
* **enforcement** (what is allowed to pass)

---

## Why Context Layer

Most multi-step LLM workflows fail not because the model underperforms, but because nothing governs execution across steps.

* Constraints set in step 1 degrade by step N
* Outputs are accepted without verification
* Session state lives inside prompts and drifts
* Small inconsistencies accumulate into failure

The model is being asked to do two jobs:

* generate output
* maintain execution integrity

It was built for the first one.

---

### Proof (same model, same prompts)

On strict multi-step workflows:

|                          | Success Rate |
| ------------------------ | ------------ |
| Direct GPT-4o mini calls | 16.4%        |
| With Context Layer       | 97.1%        |

Same model. Same prompts. Different infrastructure.

---

## What changed in v2.2

### Persistent constraint state

Constraint Activation Engine (CAE) is now **Redis-backed**.

* Constraint state survives process restarts
* Consistent across multiple workers
* No loss of enforcement state within a session

Constraint behavior itself is unchanged.
v2.2 makes it **reliable under real-world conditions**.

---

### Unified execution interface (MCP)

Context Layer exposes a unified execution endpoint for MCP-based integrations:

POST /api/execute

* Used by MCP to route execution internally
* Supports both Flow and Pulse modes via a single entry point
* Centralizes validation and execution control for external clients

Note:

* Wrapper-based integrations continue to use `/api/flow` and `/api/pulse`
* `/api/execute` is the MCP-facing execution surface

---

### Usage types (Agent vs Workflow)

Execution behavior is now determined by usage type:

* **Agent mode** — continuous execution, no explicit termination
* **Workflow mode** — explicit lifecycle with `workflowEnd` and `stateless`

In Agent mode:

* `workflowEnd` is not allowed
* `stateless` is not allowed

This enables external agents (via MCP) to run flows without managing lifecycle manually.

---

### MCP interface (external execution)

Context Layer is now accessible via Model Context Protocol (MCP).

* Transport: Server-Sent Events (SSE)
* Endpoint: `https://mcp.cl.kaisek.com/sse`

This enables:

* Direct integration with external agents
* Execution without embedding a wrapper
* A standardized runtime interface

---

## How it works

* Constraints are converted into executable checks (via CAE)
* Constraint state is maintained outside the prompt
* Each step is verified before execution continues
* Violations trigger repair or escalation
* Execution is controlled deterministically

Constraint enforcement is not advisory.
It is enforced by the runtime.

---

## Execution modes

### Flow

For multi-step workflows.

Each `invokeCL()` call represents one workflow step.
Context Layer manages constraint enforcement, state, and verification across the entire workflow.

Flow now supports continuous execution across multiple calls.

When used in Agent mode (via MCP or wrapper), sessions persist automatically and do not require explicit termination.

---

### Pulse

For chat-based and conversational systems.

Runs outside of workflow session state.

* no step tracking
* no workflow termination required

---

## Quick start

### Flow (wrapper-based)

Use the Context Layer wrapper inside your codebase.

---

**Step 1: provision execution authority**

Sign up at [cl.kaisek.com](https://cl.kaisek.com), create a project,
and provision execution authority.

---

**Step 2: set API key**

```id="flowkey"
export CL_API_KEY="your_flow_api_key"
```

---

**Step 3: import wrapper**

```id="flowimport"
const { invokeCL } = require("./flow-wrapper");
```

---

**Step 4: run Flow execution**

```id="flowrun"
const res = await invokeCL("Generate invoice");
console.log(res.output);
```

---

**Step 5: final step (Workflow mode only)**

```id="flowend"
const res = await invokeCL("Finalize invoice", { workflowEnd: true });

if (res.terminated) {
  console.log("Workflow completed");
}
```

---

**Stateless execution (Workflow mode only)**

```id="flowstateless"
await invokeCL("Quick validation", { stateless: true });
```

Note:

* `stateless: true` cannot be combined with `{ workflowEnd: true }`
* Both are rejected in Agent mode

---

### Pulse (chat / conversational)

Use Pulse for chat-based or conversational systems.

---

**Step 1: set API key**

```id="pulsekey"
export CL_API_KEY="your_pulse_api_key"
```

---

**Step 2: import wrapper**

```id="pulseimport"
const { invokeCL } = require("./pulse-wrapper");
```

---

**Step 3: run Pulse execution**

```id="pulserun"
await invokeCL("User message");
```

---

## MCP (no wrapper)

Call Context Layer via MCP (recommended for agent integrations).

---

**Connect to:**

```id="mcpurl"
https://mcp.cl.kaisek.com/sse
```

---

**Available tool:**

```id="mcptool"
send_message(
  message?: string,
  step_description?: string,
  state?: object,
  context?: string,
  constraint?: string,
  workflow_end?: boolean,
  stateless?: boolean
)
```

---

### Input modes

**1. Plain message**

```id="mcpplain"
send_message(message="Generate invoice")
```

---

**2. Structured input (State Bridge)**

```id="mcpstructured"
send_message(
  step_description="Generate invoice",
  state={ amount: 100, currency: "USD" },
  context="Checkout flow",
  constraint="Amount must be valid"
)
```

Structured input is converted into a natural-language message before execution.

---

### Flags

* `workflow_end` — terminates workflow (Workflow mode only)
* `stateless` — executes outside workflow (Workflow mode only)

Rules:

* Cannot use both together
* Both are rejected in Agent mode

---

### Usage

* Connect via any MCP-compatible client
* Send a message using `send_message`
* Context Layer handles execution, enforcement, and response

No wrapper required.

---

## Authority reports

When a workflow is terminated (explicitly or via inactivity), Context Layer generates an authority report.

* Stored for 30 days
* Automatically cleaned up after expiry
* Available for inspection and debugging

---

## Session lifecycle

In Agent mode:

* Sessions persist across calls
* Automatically terminate after inactivity (~2 hours)
* Finalization triggers authority report generation

---

## Runtime rules

* Flow steps execute sequentially
* Concurrent `invokeCL()` calls are rejected
* Direct model provider calls are blocked
* Model selection is controlled by Context Layer
* Stateless execution cannot be combined with workflow termination

---

## Provider support

* OpenAI-compatible (OpenAI, Grok)
* Anthropic

Context Layer never accesses your LLM API keys directly.

---

## Prerequisites

* Node.js 18+
* An LLM provider API key
* A Context Layer account: [cl.kaisek.com](https://cl.kaisek.com)

---

## Learn more

* [Documentation](https://cl.kaisek.com/docs)
* [Runtime Contract](https://cl.kaisek.com/docs/runtime-contract)
* [Why Context Layer](https://cl.kaisek.com/why-context-layer)

