# Context Layer

> Current release: v1.0 — Flow and Pulse execution modes available

Runtime execution infrastructure for LLM systems.

## What it is

Context Layer is a runtime execution boundary between your application 
and the model provider. It controls how every LLM request is executed, 
not just what is sent.

It governs:
- whether execution proceeds
- what context the model receives
- how execution evolves over time

## Why Context Layer

Most multi-step LLM workflows fail not because the model underperforms, 
but because there is no infrastructure layer controlling execution across 
steps.

Tested on a strict multi-step workflow with GPT-4o mini:

| | Success Rate |
|---|---|
| Regular LLM calls | ~7% |
| With Context Layer | 70%+ |

Model capability wasn't the problem. Runtime execution was.

## How it works

- Constraints set in step 1 persist through step N
- Session state is managed at the infrastructure level, not the prompt level
- Model outputs are verified before execution continues
- Workflows run deterministically, not probabilistically

## Execution modes

### Flow
For multi-step workflows. Each `invokeCL()` call represents one workflow 
step. Context Layer manages state, enforces constraints, and verifies 
outputs across every step.

### Pulse
For chat-based and conversational systems. Runs outside of workflow 
session state — no step tracking, no workflow termination required.

---

## Quick start — Flow

The wrapper is copied into your codebase rather than installed as a 
package — this keeps the execution boundary explicit and under your 
control.

**Step 1 — provision execution authority**

Sign up at [cl.kaisek.com](https://cl.kaisek.com), create a project, 
and click Provision Execution Authority. The console generates your 
wrapper script and API key.

**Step 2 — set API key**
```bash
export CL_API_KEY="your_flow_api_key"
```

**Step 3 — import wrapper**
```javascript
const { invokeCL } = require("./flow-wrapper");
```

**Step 4 — run Flow execution**
```javascript
const res = await invokeCL("Generate invoice");
console.log(res.output);
```

**Step 5 — final step**
```javascript
const res = await invokeCL("Finalize invoice", { workflowEnd: true });

if (res.terminated) {
  console.log("Workflow completed");
}
```

**Stateless execution (optional)**

Runs outside of workflow session state — no step tracking, no 
termination required.
```javascript
await invokeCL("Quick validation", { stateless: true });
```

Note: `stateless: true` cannot be combined with `{ workflowEnd: true }`.

---

## Quick start — Pulse

**Step 1 — set API key**
```bash
export CL_API_KEY="your_pulse_api_key"
```

**Step 2 — import wrapper**
```javascript
const { invokeCL } = require("./pulse-wrapper");
```

**Step 3 — run Pulse execution**
```javascript
await invokeCL("User message");
```

---

## Runtime rules

- Flow steps execute sequentially
- Concurrent `invokeCL()` calls are rejected
- Direct model provider calls (OpenAI, Anthropic, etc.) are blocked
- Model selection is controlled by Context Layer, not the developer
- `stateless: true` runs outside of workflow session state and cannot 
  be combined with `{ workflowEnd: true }`

## Provider support

- OpenAI-compatible (OpenAI, Grok)
- Anthropic

Context Layer never accesses your LLM API keys directly.

## Prerequisites

- Node.js 18+
- An LLM provider API key
- A Context Layer account — [cl.kaisek.com](https://cl.kaisek.com)

## Learn more

- [Documentation](https://cl.kaisek.com/docs)
- [Runtime Contract](https://cl.kaisek.com/docs/runtime-contract)
- [Why Context Layer](https://cl.kaisek.com/why-context-layer)
