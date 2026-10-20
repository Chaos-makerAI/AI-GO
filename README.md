# MUC

**MUC is an Agent-native, multi-tenant CRM built around a controlled execution runtime.**

It turns a CRM task into an auditable execution run:

```text
CRM task / object
    -> context, memory, knowledge
    -> MUC AgentRuntime
    -> model + tools
    -> policy, permissions, approval, durable execution
    -> CRM state changes + trace + evaluation
```

The project is designed around a practical question: **how do we let an Agent
take useful CRM actions without losing control of scope, side effects, failure
semantics, or evidence?**

> The repository contains historical `pi-native-*` identifiers from the former
> project name. They are kept where they are protocol, migration, or compatibility
> identifiers; the product and GitHub-facing name is now **MUC**.

## What is MUC?

MUC is not only a chat interface or a prompt wrapper. CRM records are the source
of business facts; the Agent consumes context and knowledge, calls typed CRM
tools, produces structured outcomes, and leaves behind events and evidence that
can be inspected later.

The main execution surface is an Agent workbench with four built-in CRM roles:

- **CRM Intelligence** — assemble customer, conversation, opportunity, knowledge,
  and memory context with evidence.
- **Sales Operations** — investigate stalled opportunities and propose or apply
  bounded CRM updates.
- **Customer Communication** — draft grounded replies and surface commitments
  that need human review.
- **CRM Supervisor** — coordinate bounded specialist investigations and consolidate
  their results.

## My ownership

My interview-relevant responsibility covers **runtime, harness, and eval**:

### Runtime — the CRM-owned execution seam

- Defined the `AgentRuntime` contract so CRM domain code does not depend directly
  on Pi or a provider SDK.
- Built the Pi adapter boundary for messages, tool calls/results, usage, runtime
  events, cancellation, steering, and stop conditions.
- Kept provider governance separate from the Agent loop: model binding, budget,
  usage, and audit remain CRM concerns.
- Supports canonical `pi` execution and a `shadow` mode for planning/evaluation
  with non-read tools blocked before CRM handlers execute.

### Harness — turning model output into controlled work

- Carries organization scope, target scope, permissions, and tool effect classes
  through the whole run.
- Distinguishes `read`, `reversible_write`, `external`, and `irreversible`
  effects; external or irreversible actions can require human approval.
- Separates API admission from durable Worker execution with queueing, leases,
  cancellation, resume, retry, and explicit partial/reconciliation states.
- Preserves tool results, field changes, approvals, and audit events so an Agent
  cannot silently turn a suggestion into an unreviewed side effect.

### Eval — verifying more than “the model answered”

- Combines deterministic gates with an optional semantic Judge; semantic review
  cannot override a failed deterministic gate.
- Checks task completion, answer quality, knowledge grounding, tool reliability,
  policy compliance, efficiency, and collaboration quality.
- Verifies tool-event ordering, approval state, policy checks, evidence and
  provenance, structured results, and tool errors.
- Routes Judge failures to `needs_review` instead of manufacturing a pass, and
  keeps golden fixtures/regression cases close to the runtime contract.

## Why this project is technically interesting

1. **Runtime and provider are decoupled.** The CRM owns the contract; Pi owns
   the model/tool loop; provider governance remains outside the kernel.
2. **The harness is a safety boundary.** Permissions and side-effect policy are
   enforced around tools, not left to the model's wording.
3. **Execution is durable.** Long-running work has queue, lease, cancellation,
   resume, retry, and partial-failure semantics instead of a single request
   waiting on a model call.
4. **Evaluation is evidence-aware.** A fluent answer is not treated as a verified
   CRM outcome; the evaluator checks events, policy, approvals, and provenance.
5. **The system is inspectable.** Runs expose usage, tool lifecycle, state
   changes, stop reasons, and evaluation evidence for debugging and review.

## Architecture

```text
CRM task / object
        |
        v
Context builder: CRM facts + knowledge + memory + skills + prior runs
        |
        v
MUC AgentRuntime contract
        |
        v
Pi adapter / model provider
        |
        v
Harness: tenant scope + permissions + effects + approval + lease
        |
        +--> CRM tools / durable Worker / persisted events
        |
        v
Deterministic eval --> optional semantic review --> regression corpus
```

## Repository map

| Area | Purpose |
| --- | --- |
| [`lib/agent-runtime/`](lib/agent-runtime/) | CRM-owned runtime contract and Pi adapter |
| [`lib/ai/agents/`](lib/ai/agents/) | Tool effects, workbench leases, durable Agent turns |
| [`lib/agent-engine/queue/`](lib/agent-engine/queue/) | Queue and execution orchestration |
| [`lib/ai/evals/`](lib/ai/evals/) | Evaluation contracts, gates, Judge flow, and provenance |
| [`tests/agent-runtime/`](tests/agent-runtime/) | Runtime fixtures and golden evaluation cases |
| [`docs/architecture/`](docs/architecture/) | Architecture decisions and ownership boundaries |
| [`docs/harness-audit.md`](docs/harness-audit.md) | Harness failure/side-effect audit |

## Interview route

If you are reviewing this repository for the first time, these files show the
core design fastest:

1. [`lib/agent-runtime/types.ts`](lib/agent-runtime/types.ts) — the runtime seam.
2. [`lib/agent-runtime/pi/runtime.ts`](lib/agent-runtime/pi/runtime.ts) — Pi execution adapter.
3. [`lib/ai/agents/tool-effects.ts`](lib/ai/agents/tool-effects.ts) — side-effect taxonomy.
4. [`lib/ai/agents/workbench-job-lease.ts`](lib/ai/agents/workbench-job-lease.ts) — durable execution lease.
5. [`lib/ai/evals/evaluate-run.ts`](lib/ai/evals/evaluate-run.ts) — deterministic run checks.
6. [`lib/ai/evals/run-evaluation.ts`](lib/ai/evals/run-evaluation.ts) — evaluation orchestration and Judge handling.
7. [`lib/ai/evals/evidence-provenance.ts`](lib/ai/evals/evidence-provenance.ts) — evidence lineage.
8. [`docs/architecture/pi-native-runtime.md`](docs/architecture/pi-native-runtime.md) — runtime/provider boundary.

## Honest scope

MUC is an internal R&D / portfolio engineering project. The repository contains
implemented paths, tests, fixtures, and historical validation records, but those
are not automatically production business metrics. In particular, the project
does not claim that every channel, external message delivery path, or independent
business-outcome verification loop is complete or fully unattended. The code
keeps `partial`, `needs_review`, and reconciliation states explicit where the
system cannot prove success.

## Quickstart

```bash
pnpm install
pnpm dev
```

See [`docs/SETUP.md`](docs/SETUP.md) for environment and database setup. Useful
checks include:

```bash
pnpm typecheck
pnpm lint
pnpm test:e2e
```

## Further reading

- [Architecture overview](docs/architecture/README.md)
- [Agent services and integration boundaries](docs/integrations/agent-services.md)
- [Testing and validation records](docs/testing/)
- [LLM-oriented project summary](public/llms.txt)
