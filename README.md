<div align="center">

# Hearth

### The Control-Loop Harness — Enforcement Kernel, Control-Factor & Context-Traversal Substrate, Harness-Independent Benchmark

**Reference Implementation v0.4 — nine artifacts shipped, selftested, and linked**

**Enforcement over artifacts. Measurement and enforcement are the same control path with different release authority.**

</div>

> **An MD checklist has zero power. A regressive control loop over that checklist has all
> the power. Without the loop, the checklist is a wish. With the loop, it is an enforced
> contract — the harness re-reads it, re-verifies it, and drags the model back on track
> based on findings.**

*(working name — rename freely; the identity is the contract, not the label)*

---

## Table of Contents

**Part I — The Design**

1. [Why this exists](#1-why-this-exists)
2. [The principle](#2-the-principle)
3. [What it is — the kernel](#3-what-it-is--the-kernel)
4. [The core loop](#4-the-core-loop)
5. [Architecture](#5-architecture)
6. [The tick — the atomic unit](#6-the-tick--the-atomic-unit)
7. [The module contract](#7-the-module-contract)
8. [Stacking & precedence — levels, locked](#8-stacking--precedence--levels-locked)
9. [Modes & release authority](#9-modes--release-authority)
10. [Adapter capabilities — the declared boundary](#10-adapter-capabilities--the-declared-boundary)
11. [Flash control loops](#11-flash-control-loops)
12. [The registries](#12-the-registries)
13. [Harness registration](#13-harness-registration)
14. [Contracts](#14-contracts)
15. [The canonical traversal graph](#15-the-canonical-traversal-graph)
16. [Context traversal & synchronization](#16-context-traversal--synchronization)
17. [The session — the unit of graph ownership](#17-the-session--the-unit-of-graph-ownership)
18. [The API surface — four contracts + the connector layer](#18-the-api-surface--four-contracts--the-connector-layer)
19. [Metrics, for free](#19-metrics-for-free)
20. [Overhead accounting](#20-overhead-accounting)
21. [The control-factor space](#21-the-control-factor-space--taxonomy--planes)
22. [Benchmark: harness power === traversal power](#22-benchmark-harness-power--traversal-power)
23. [The canonical baseline & unbiased aggregation](#23-the-canonical-baseline--unbiased-aggregation)
24. [Control economics — cost to correction](#24-control-economics--cost-to-correction)
25. [Drift](#25-drift--detection-attribution-correction-recurrence)
26. [The supervisor — the control plane](#26-the-supervisor--the-control-plane)
27. [The kernel contract — frozen](#27-the-kernel-contract--frozen)
28. [Quick start](#28-quick-start)
29. [Design invariants](#29-design-invariants)
30. [Limitations](#30-limitations)
31. [Prior art](#31-prior-art)
32. [Roadmap](#32-roadmap)
33. [FAQ](#33-faq)
34. [Glossary](#34-glossary)
35. [License](#35-license)

**Part II — The Reference Implementation**

36. [Implementation status — nine artifacts](#36-implementation-status--nine-artifacts)
37. [The connector layer](#37-the-connector-layer)
38. [The kernel & runtime](#38-the-kernel--runtime)
39. [Recursive composition — the fractal substrate](#39-recursive-composition--the-fractal-substrate)
40. [The scheduler — the width exploiter](#40-the-scheduler--the-width-exploiter)
41. [The metrics suite](#41-the-metrics-suite)
42. [The benchmark service](#42-the-benchmark-service)
43. [The Context Passage Protocol](#43-the-context-passage-protocol-cpp)
44. [The laboratory](#44-the-laboratory)
45. [Selftest inventory](#45-selftest-inventory)

**Part III — The Theory, Audited**

46. [The meta-loop — three levels and the connection gap](#46-the-meta-loop--three-levels-and-the-connection-gap)
47. [The prior-art survey — four graphs, one gap](#47-the-prior-art-survey--four-graphs-one-gap)
48. [The black-box doctrine](#48-the-black-box-doctrine)
49. [The dual-review ledger](#49-the-dual-review-ledger)
50. [The governance plane](#50-the-governance-plane)
51. [The four planes](#51-the-four-planes)
52. [Cheap verification — the scaling math, audited](#52-cheap-verification--the-scaling-math-audited)
53. [The scale factors](#53-the-scale-factors)
54. [The OMEX join](#54-the-omex-join)
55. [The fractal thesis](#55-the-fractal-thesis)
56. [The four faces & the viability verdict](#56-the-four-faces--the-viability-verdict)
57. [The data asset](#57-the-data-asset)
58. [The UI/UX specification](#58-the-uiux-specification)
59. [Errata & findings about ourselves](#59-errata--findings-about-ourselves)
60. [Remaining backlog](#60-remaining-backlog)

---

# PART I — THE DESIGN

## 1. Why this exists

The execution stack today:

```
MODEL  →  HARNESS  →  TRAJECTORY  →  ENVIRONMENT
```

- Model benchmarks ask: *did you know the answer?*
- Agent benchmarks ask: *did you finish the task?*
- Trajectory evals ask: *how efficiently did you move?*
- Almost nothing asks: ***did your control actually hold?***

Three failures nobody handles together:

1. **Unenforced state.** Harnesses everywhere store state — plans, checklists, goals,
   recaps — and almost none of them *enforce* it. A stored plan that nothing re-verifies
   is a wish with good formatting.
2. **Configured ≠ demonstrated.** `max_iterations = 50` says nothing about capability.
   A harness *allowed* fifty iterations is not a harness that *demonstrates* fifty levels
   of useful traversal. Control capacity should be enforced and measured — not configured
   and assumed.
3. **No way to compare control across harnesses.** If the control machinery lives inside
   one agent, you can measure that agent. If it is external and adaptable, you can measure
   **the harness itself** — across Codex, Claude Code, ChatGPT, custom harnesses, browser
   agents, any host — under identical contracts, identical tick semantics, identical
   verdicts, identical trace, identical metrics. That is the difference between
   benchmarking a product and benchmarking a category.

And the fourth, discovered while building — the deepest:

4. **Nothing monitors the movement.** Every harness runs control loops. Almost none of
   them monitor what those loops *produce* — and none can, from the inside. The
   monitoring layer above all loops, all sub-harnesses, and all boundaries between them
   does not exist
   ([§46](#46-the-meta-loop--three-levels-and-the-connection-gap)). Hearth is that layer.

---

## 2. The principle

**An artifact is not a control.**

| Artifact alone | Artifact + control loop |
|---|---|
| MD checklist | Contract — re-read every step, re-verify every item, findings force repair or backtrack |
| Plan file | Regressively re-planned on a rolling horizon |
| Goal statement | Verified against the desired endpoint on every iteration |
| Recap log | Fed back into verification — drift becomes detectable |

**Artifacts are memory. Loops are enforcement. The harness is the binding of both.**

This is the load-bearing idea of the entire project. Every other harness has state.
Almost none of them *enforce* it. Hearth is the component that makes state enforceable.

And under the fractal substrate the arc completes itself: the checklist goes from
**wish → contract → DAG** ([§40](#40-the-scheduler--the-width-exploiter)) — item
dependencies are graph edges, the checklist's shape *is* the schedule.

---

## 3. What it is — the kernel

The cleanest formulation, and the one that governs everything else in this document:

> **Hearth is an enforcement kernel for stateful agent execution. It binds obligations to
> executable control loops, evaluates those loops at host-event boundaries, gates
> execution through a finite verdict protocol, and records the resulting control
> trajectory as a canonical trace.**

Native mode, meta mode, host mode, modality loops, tool loops, bundles, adapters,
registries, the browser extension, the supervisor apps — these are **how you deploy and
compose the kernel**, not competing definitions of what Hearth is.

One sentence identity:

> **A control-loop harness: a master rolling-horizon loop with a registry of stackable
> sub-loop modules — runnable native, wrappable around other harnesses as a meta layer,
> emitting a canonical traversal trace as a side effect of enforcement.**

### The three identities — how you deploy it

**1. Native harness.** Runs standalone. `/goal` → `/subtask` → execute → hooks → replay →
verify → `/recap` → done. A complete harness in itself.

**2. Meta harness.** Sits **on top of** other harnesses via adapters. The host harness
emits events; the adapter maps them to canonical loop events; the enforcement core returns
verdicts. One interface for every host.

**3. Loop host — the vest.** A **kit of flash-in control loops**: modality loops, tool
loops, scenario loops, verification loops, and anything you register. You should never be
restricted to one pass-through control loop — there are too many places loops belong, so
this is a vest, not a single strap.

### The five roles — what it is

> **The binding** — artifact to loop, always. Nothing stored unbound.
> **The machine** — re-reads, re-verifies, repairs, backtracks.
> **The vest** — flashes more control in without becoming a monolith.
> **The instrument** — measures the same thing it enforces.
> **The heart** — the organ other harnesses are missing; everything gets built around it.

The identities are how you deploy it. The roles are what it is. Both are true
simultaneously, and neither changes the other.

### The two contributions

1. **The enforcement kernel** — binding obligations to loops, evaluating at host-event
   boundaries, gating through four verdicts.
2. **The canonical traversal graph** — enforcement history as a reusable substrate rather
   than something thrown away after a verdict. The trace is not a log; it is the
   measurement layer ([§15](#15-the-canonical-traversal-graph)).

### Why benchmarking tool and meta harness are one product

> **Measurement and enforcement are the same control path with different release
> authority.**

- Do not release the verdict → **benchmarking** (observe and measure the control loop).
- Gate the verdict → **meta-harness** (participate in the control loop, enforce contracts).
- Escalate the verdict → **supervisor** (delegate release authority upward).

The distinction is not architectural. It is the degree of authority the deployment grants
Hearth. That is also why something good enough to benchmark control can enforce control:
it is the same engine, same events, same contracts, same trace — different authority.

### The substrate — Hearth's relationship to the control-factor space

Hearth does not fundamentally measure *a* control loop. It creates the common
instrumentation to measure **all the control loops that participate in execution, at
whatever plane they operate, and their interactions**:

```
HEARTH
  │
  ├── measures CONTROL
  ├── decomposes CONTROL into FACTORS
  ├── observes FACTOR interactions
  ├── measures intervention
  ├── measures correction cost
  ├── measures recurrence / drift
  └── produces counterfactual control data
```

> **Hearth itself is not one control factor. It is the measurement/enforcement substrate
> for the control-factor space** ([§21](#21-the-control-factor-space--taxonomy--planes)).

Loops enforce factors. The taxonomy names what the loops measure. The trace records both.

And the closing formulation — the one to protect most aggressively as this is built:

> **Hearth is a control-observation layer that can reconstruct enough of a session's
> context graph to measure — and, where authorized, intervene in — the harness's control
> decisions, without becoming the harness itself.**

---

## 4. The core loop

**Rolling-horizon control with dynamic backward verification depth.**

The master loop that everything else hangs off of:

```
        ┌────────────────────────────────────────────────┐
        │   PLAN → EXECUTE → OBSERVE                     │
        │                     │                          │
        │                     ▼                          │
        │   REPLAY (last R checkpoints)                  │
        │   - did prior work remain valid?               │
        │   - did later steps introduce regressions?     │
        │   - were assumptions violated?                 │
        │   - what is the blast radius?                  │
        │                     │                          │
        │            ┌────────┴────────┐                 │
        │         VALID             INVALID              │
        │            │                │                  │
        │       CONTINUE      REPAIR / BACKTRACK(d)      │
        │            │                │                  │
        │            └──────► RE-PLAN ◄────┘             │
        │                        │                       │
        └────────────────────────┼───────────────────────┘
                                 ▼
                          next iteration
```

Move forward → periodically replay previous decision points → assess blast radius →
backtrack if necessary → continue forward → repeat until the goal is complete.

### Dynamic replay depth (R)

R is not a constant. It is a rule:

| Condition | Replay depth |
|---|---|
| Normal progress | 3 |
| High blast radius | 7 |
| Architectural change | full dependency chain *(graph-extended — §16)* |
| Contract version change | full dependency chain |
| Critical failure | replay from last stable checkpoint |

> **R is the bounded default. Graph traversal identifies where that bounded window needs
> to extend** — along relationship edges, not just tick count. "Full dependency chain" is
> not a bigger number; it is a different traversal.

### The verdict vocabulary

Everything the enforcement core says is one of four things. Nothing else escapes it:

| Verdict | Meaning |
|---|---|
| `CONTINUE` | verified — advance |
| `REPAIR` | fix in place — re-verify before advancing |
| `BACKTRACK(d)` | return *d* checkpoints, repair, re-advance |
| `HALT` | stop — surface to supervisor or operator |

### BACKTRACK semantics — three distinct phases

`BACKTRACK(d)` names the *target checkpoint*. What happens afterward is **not one
operation but three**, and a repair plan must declare which phases it involves:

| Phase | Meaning | Example |
|---|---|---|
| **Rewind state** | restore the world to the checkpoint | revert files, restore scene state |
| **Repair consequences** | compensate for effects already emitted downstream | fix the artifact that consumed bad output |
| **Replay decisions** | re-execute from the checkpoint with corrected information | re-run the subtask with the finding applied |

These are not equivalent. A host may be able to rewind but not replay, or replay without
rewinding (forward-repair only). Declaring them separately is what makes `BACKTRACK(d)`
honest across heterogeneous hosts.

### Checkpoints

A checkpoint is:

- **id** — addressable, ordered
- **trace position** — where in the canonical traversal graph this state lives
- **contract versions in force** — which contract versions were active at this point
- **state snapshot ref** — a pointer to the captured state, **or** a
  **compensating-action descriptor** for hosts that cannot snapshot (the minimal set of
  inverse actions that would restore the prior state)

Hearth never assumes snapshots are available. If a host cannot snapshot, the
compensating-action descriptor is the checkpoint — and replay verifies against it.
No host capability is silently required; what is missing is declared.

---

## 5. Architecture

```
┌───────────────────────────────────────────────────┐
│              HEARTH — CONTROL-LOOP HARNESS        │
│                                                   │
│  ┌───────────────────────────────────────────┐    │
│  │ ENFORCEMENT CORE                          │    │
│  │ · rolling-horizon master loop             │    │
│  │ · backward verification (dynamic R)       │    │
│  │ · backtrack / repair engine               │    │
│  │ · ordering enforcement                    │    │
│  └───────────────────────────────────────────┘    │
│                                                   │
│  ┌───────────────────────────────────────────┐    │
│  │ LOOP REGISTRY  —  "the vest"              │    │
│  │ · modality loops       (first-class)      │    │
│  │ · tool loops                              │    │
│  │ · scenario loops                          │    │
│  │ · verification loops                      │    │
│  │ · composition.span     (boundaries)       │    │
│  │ · … flash in more, any time               │    │
│  └───────────────────────────────────────────┘    │
│                                                   │
│  ┌───────────────────────────────────────────┐    │
│  │ TRAVERSAL LAYER                           │    │
│  │ · pluggable strategies over session graphs│    │
│  │ · default.temporal / default.dependency / │    │
│  │   default.blast_radius / default.hybrid   │    │
│  │ · harness-registered strategies           │    │
│  └───────────────────────────────────────────┘    │
│                                                   │
│  ┌───────────────────────────────────────────┐    │
│  │ STATE STORE  (storage-agnostic)           │    │
│  │ · goals · subtasks · checklists · recaps  │    │
│  │ · session graphs — one per session        │    │
│  └───────────────────────────────────────────┘    │
│                                                   │
│  ┌───────────────────────────────────────────┐    │
│  │ TRACE OUT → canonical traversal graph     │    │
│  └───────────────────────────────────────────┘    │
└───────────────────────────────────────────────────┘
```

The state store holds the artifacts and the session graphs. The registry holds the loops.
The traversal layer navigates. The core binds them. The trace falls out.

> **The store is not the product.** The store is the persistent substrate for a live,
> adaptable context-traversal and control system ([§16](#16-context-traversal--synchronization)).

---

## 6. The tick — the atomic unit

Hearth runs **on top of every prompt** a harness emits. Every prompt, tool call, or
state change is one **tick** — the atomic unit of enforcement, measurement, and trace.
There is no coarser or finer unit.

This makes "iteration" exact: **an iteration is one tick.** Every replay, every
verification, every traversal, every trace record happens at tick boundaries. AITD is
ticks per level — the metric and the mechanism share a unit.

### Event identity vs iteration identity — stated precisely

- **Event identity** — every host event is exactly one tick. No coalescing, no splitting.
  One event in → one pipeline run → one tick record out. This holds regardless of event
  size.
- **Iteration identity** — a host "turn" or "step" may span *many* ticks. Hearth never
  counts host turns; every metric, including AITD, is counted in ticks.

The atomicity claim is about the *pipeline*, not about events being equal in size.
Granularity differences between event classes are real, recorded (event class is a tick
field), and never allowed to blur the count.

### The per-tick pipeline

Fixed for every tick, native or meta, regardless of how many loops or strategies are
loaded:

```
TICK
 1. INTERCEPT   host emits event (prompt / tool call / state change)
 2. RESOLVE     registry resolves which loops fire this tick
                (scope match + trigger match + composes_with check)
 3. TRAVERSE    context resolution — relevant prior state + impact subgraph
                (via the Traversal API; strategy pluggable, result canonical)
 4. STEP        each resolved loop steps against the synced context —
                cheap checks first, deep replay only on trigger
 5. AGGREGATE   one verdict per tick — worst verdict wins:
                HALT > BACKTRACK(dmax) > REPAIR > CONTINUE
 6. SYNC        backward sync confirmed; forward sync of consequences —
                mandatory before release on REPAIR / BACKTRACK
 7. EMIT        one trace record per loop + one tick record
                (including the traversal record itself)
 8. RELEASE     CONTINUE releases the tick to the host;
                anything else holds, per verdict
```

The pipeline never changes when loops or strategies are added. A new loop adds checks to
step 4; a new strategy adds traversal behavior inside step 3 — never ticks, never steps.
**Addition is monotonic.** This is why addition never destabilizes the host.

### The non-rewrite guarantee

> **Hearth reads ticks; it never rewrites them.**
> The prompt released to the host is byte-identical to the prompt intercepted, unless
> the host has explicitly opted into annotations. Hearth gates the release and appends
> to the trace. It does not edit the host's content.

Non-interference is therefore testable: diff intercepted vs. released. Byte-identical,
or the harness is in violation of itself.

### Event-time, not wall-clock

The tick is **per-host-event, not per-wall-clock-time.** The scheduled supervisor is
the only time-based loop — outermost, and it can raise R or force backtracks between
ticks. Everything else advances only when the host advances. Hearth's cost stays
proportional to the host's activity, which is the correct coupling for a layer whose
whole identity is non-interference with enforcement.

---

## 7. The module contract

Every control loop in the vest — first-party, third-party, or registered harness —
implements the same contract:

```python
class ControlLoopModule(Protocol):
    id: str
    scope: Literal["modality", "tool", "scenario", "verification", "level"]
    trigger: Trigger          # event / phase / hook that activates this loop
    horizon: Horizon          # forward H, replay R, adaptive rule

    def step(self, state: LoopState) -> StepResult:
        """One iteration of this loop."""

    def verify(self, state: LoopState) -> Verdict:
        """CONTINUE | REPAIR | BACKTRACK(d) | HALT"""

    def backtrack(self, d: int) -> RepairPlan:
        """Repair plan for returning d checkpoints."""

    def emit(self) -> TraceRecord:
        """Trace record — feeds the canonical traversal graph."""

    composes_with: set[str]   # stacking compatibility, checked at registration
```

One contract. Every loop. No exceptions.

### Repair ownership

A verdict names the failing artifact and its owning loop. **The loop that owns the
artifact executes the repair.** Hearth schedules the repair, and re-verifies with the
same R window that caught the finding. No loop repairs another loop's artifact; no
repair goes unscheduled; no repair escapes re-verification.

---

## 8. Stacking & precedence — levels, locked

> **Serialization level is a property of the registry, declared at registration —
> not inferred from runtime behavior.** Each loop's registration names its parent
> scope; the level is the depth of that nesting in the registry. L0 is the master
> loop. A loop's level never changes mid-run.

```
L0  orchestrator (master loop)
L1    modality loops
L2      tool / scenario loops
L3        verification loops
```

**Traversal depth is a different quantity entirely** — the length of the traversal path
in the canonical graph, including revisits. AITD is indexed by **serialization level**.
The canonical graph records **traversal depth**. Both are kept, neither substitutes for
the other.

Precedence rules:

- **Outer loop owns ordering. Inner loops yield.**
- The **supervisory agent is the outermost loop** — it can raise replay depth or force a
  backtrack on anything below it.
- `composes_with` rejects incompatible stacks at registration time, not at 3 AM.

### Verdict aggregation — the tie-break, made explicit

```
HALT  >  BACKTRACK(d)  >  REPAIR  >  CONTINUE
```

When two loops return `BACKTRACK` with **different depths**:

> **dmax wins.** The maximum required depth is the most conservative finding, and
> conservatism is the direction disagreement must degrade in. (A lesser depth is
> recorded in the trace as a dissenting finding — it is not lost, it is just not
> the gate.)

This is an invariant, not a convention (Invariant #12).

### What a run looks like — and why the depth data falls out

```
Level 0   orchestrator loop                 12 iterations
Level 1   ├── modality.vision                8
          ├── modality.code                 14
          └── scenario.research              6
Level 2   │    ├── vision.crop               5
          │    ├── code.refactor             9
          │    └── research.verify           7
Level 3   │    │    └── code.refactor.verify 4
```

Every loop reports its iteration count per level. With the tick as the atomic unit,
these counts are **ticks resolved per level** — the enforcement structure *is* the
instrumentation structure.

---

## 9. Modes & release authority

### The three capabilities, separated

| Power | Meaning |
|---|---|
| **Observe** | Hearth sees what happened |
| **Intercept** | Hearth sees it *before* it happens |
| **Gate** | Hearth can prevent it from happening |

A browser extension may observe a conversation but be unable to prevent an action. A CLI
integration may intercept tool calls. A native adapter may have full synchronous gating.
Modes are chosen per deployment, within what the adapter's capabilities permit.

### The control path — five postures

Every posture is the **same pipeline** with different release authority:

```
                 HEARTH CONTROL PATH
                         │
                ┌────────┴────────┐
                │                 │
            OBSERVATION        INTERCEPTION
                │                 │
                ▼                 ▼
             AUDIT             GATING
                │                 │
          ┌─────┴─────┐     ┌─────┴─────┐
          ▼           ▼     ▼           ▼
       SHADOW      ACTIVE  AUTO       SUPERVISED
```

| Posture | Pipeline timing | Release behavior | Asks |
|---|---|---|---|
| **OBSERVE** | synchronous, steps 3–4 bypassed | trace only | *what happened?* |
| **AUDIT / SHADOW** | asynchronous (shadow) | records findings; host continues | *what did the harness do wrong when nothing stopped it?* |
| **AUDIT / ACTIVE** | asynchronous (shadow) | records findings **and** files actionable findings the supervisor or outer loop can act on between ticks | *what went wrong, and what should be done about it?* |
| **ENFORCE / AUTO** | synchronous | gates on policy, automatically | *how does the harness perform when invalid progression is prevented?* |
| **ENFORCE / SUPERVISED** | synchronous | gates and escalates the decision to the supervisor | *what does this system need from its supervisor to stay correct?* |

These are **experimental configurations**, not mutually exclusive products. The same
run can shadow-audit one scope while enforcing another.

### Why shadow audit is a benchmark instrument, not a degraded mode

Shadow audit answers a question no enforced mode can ask:

> **What did the harness do wrong when nothing stopped it?**

That quantifies raw capability:

```text
violations detected · violations ignored · violations self-corrected
violations propagated · eventual recovery · irreversible violations
supervision required
```

Enforced audit asks: **how does the harness perform when a control layer actively
prevents invalid progression?** Run both and the difference between them is itself a
benchmark — the **Hearth Delta** ([§22](#22-benchmark-harness-power--traversal-power)).

### The legacy ladder — mapped, not replaced

| Level | Name | Requires | Maps to |
|---|---|---|---|
| 2 | **ENFORCE** | gate + resume capabilities | ENFORCE / AUTO or SUPERVISED |
| 1 | **AUDIT** | intercept capability, no pause | AUDIT / SHADOW or ACTIVE |
| 0 | **OBSERVE** | observe capability only | OBSERVE |

A host that cannot pause still gets full shadow enforcement: the identical pipeline
runs asynchronously, every violation is recorded, and enforcement is carried out by the
outermost loop instead of inline. **No host is worth zero enforcement.**

### Native, meta, host — unchanged

**Native:** `/goal → /subtask → execute → hooks → replay(R) → verify → /recap → next`

**Meta — Hearth wears another harness:**

```
HOST HARNESS ──native events──► ADAPTER ──canonical loop events──►
ENFORCEMENT CORE ──verdict──► HOST HARNESS
```

A host harness has exactly three obligations:

1. Emit events (step done, tool call, file write, state change).
2. Accept verdicts (`CONTINUE | REPAIR | BACKTRACK(d) | HALT`).
3. Pause on `REPAIR` / `BACKTRACK` / `HALT` until released — *to the extent its
   capabilities permit; otherwise it declares so and the posture degrades along the
   control path, documented and recorded.*

**Host — other harnesses wear Hearth:** External harnesses register **into** the vest as
loop modules. Their native controls become loop steps; their uniqueness is preserved
behind the one contract.

**Both directions are first-class.** Hearth wraps harnesses. Harnesses wrap Hearth.
It is the heart either way.

### The ownership boundary

> **Hearth does not compete for "who is the agent."
> It owns "did the stored obligations still hold."**

It does not need to own planning, tools, memory, or the host's special logic. It needs
only events in, verdicts out, and a hold when the verdict is not CONTINUE. That is why
it can sit beside other harnesses as a double-checker and enforcer — without fighting
them for ownership of tools or personality.

---

## 10. Adapter capabilities — the declared boundary

This is the frozen boundary of the entire integration layer:

> **Hearth never assumes how a host executes. Hearth defines what must be observable,
> what may be intercepted, what may be gated, and what verdicts mean. The adapter
> declares which of those capabilities the host actually provides.**

### The capability matrix

| Capability | Meaning |
|---|---|
| `observe` | Hearth can see the event (after the fact) |
| `intercept` | Hearth sees the event before execution |
| `gate` | Hearth can block execution |
| `resume` | Hearth can release held execution |
| `inject` | Hearth can provide controlled input / annotation (host opt-in only) |
| `snapshot` | Hearth can capture state |
| `compensate` | Hearth can provide inverse actions for checkpoints |
| `supervise` | Hearth can escalate to the supervisor |

Values: `true` / `partial` / `false`. Nothing is inferred. **The declaration is part of
the benchmark configuration** — it ships with every trace.

### Two real declarations

A pause-capable CLI harness:

```yaml
adapter:
  host: codex
  capabilities:
    observe_prompts: true
    observe_tool_calls: true
    observe_state_changes: partial
    intercept_before_tool: true
    pause_execution: true
    resume_execution: true
    inject_annotations: false
```

A browser client:

```yaml
adapter:
  host: browser:chatgpt
  capabilities:
    observe_conversation: true
    observe_proposals: true
    intercept_before_tool: partial
    pause_execution: false
    resume_execution: false
    inject_annotations: false
    supervise: true
```

Hearth doesn't lie about the difference. It records it.

### Posture is derived from capabilities, validated at registration

`posture = f(capabilities, policy)`. A deployment requesting ENFORCE on an adapter
without `gate` + `resume` fails at registration with a precise message — not at 3 AM,
not silently. Requesting AUDIT/SHADOW always succeeds anywhere `observe` exists.

### The browser extension — a first-class adapter

```
             HEARTH CORE
                  │
        canonical event protocol
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
    CLI/API    Browser     Native
    Adapter    Adapter     Adapter
       │          │           │
       ▼          ▼           ▼
    Codex      ChatGPT      custom
    Claude     Claude       harness
    etc.       Z.ai
               Grok
```

The benchmark isn't "a benchmark for ChatGPT." It is:

> **A harness-independent control benchmark — and ChatGPT is one adapter.**

---

## 11. Flash control loops

A **flash loop** is a control loop packaged as a swappable module — flash it into the
vest at runtime, per modality, per tool, per scenario, per level.

| Loop type | Scope | Enforces |
|---|---|---|
| **Modality loops** (first-class, first shipped) | `modality` | modality-specific ordering, horizons, and verification — vision, code, docs, audio, … |
| Tool loops | `tool` | per-tool constraints: read-before-write, confirmation gates, retry policy |
| **Tool gate** (`tool.gate`) | `tool` | **irreversibility gate** — actions tagged irreversible (delete, force-push, external send) require a fresh `CONTINUE` on the replay window before dispatch, or explicit supervisor confirmation |
| Scenario loops | `scenario` | per-task policies: research protocols, SWE-style verify-before-commit |
| Verification loops | `verification` | replay windows, audit passes, regression sweeps |
| **Composition** (`composition.span`) | `verification` | **boundary loops** — handoff sync, orphaned child output, cross-span conflicts, cross-level verdict propagation ([§39](#39-recursive-composition--the-fractal-substrate)) |
| **Yours** | anything | any loop matching the contract |

**Loops enforce factors.** Each loop is the executable embodiment of one or more control
factors — `tool.gate` enforces the *irreversibility* factor, `verification.replay`
enforces the *replay* factor. The taxonomy names what the loops measure; the loops are
how factors get teeth.

The registry is open. New modality, new tool, new scenario = new module, same interface,
same enforcement core, same trace format. The stack grows without degrading — addition
is monotonic, never a new cadence.

---

## 12. The registries

The vest is a **routing table**, not a costume. Loops are addressable by **scope key**
and resolve at tick time. Traversal strategies are registered the same way — defaults
plus harness-registered, keyed by scope. Control factors are registered the same way too
([§44](#44-the-laboratory)).

### Structure

```
REGISTRY
├── harness registry     registered hosts: adapter, capabilities, posture,
│                        attached bundles
├── modality registry    loops keyed by modality scope     (first-class)
├── tool registry        loops keyed by tool scope
├── scenario registry    loops keyed by task/scenario scope
├── verification registry loops keyed by verification scope
├── traversal registry   strategies keyed by scope — defaults + harness-registered
├── factor registry      control factors keyed by id — versioned taxonomy (C1)
└── bundle registry      named sets of loops — kits
```

### Resolution rules

A loop fires on a tick iff **all** hold:

1. **Scope match** — the tick's tags include the loop's scope key
2. **Trigger match** — the tick's event type matches the loop's trigger
3. **Composition holds** — the loop's `composes_with` permits the active set

Firing order is deterministic: declared priority, then registration order.
No nondeterministic stacks.

### Bundles — kits

```yaml
bundles:
  vision-swe:
    - modality.vision
    - tool.filesystem
    - tool.gate
    - verification.replay

attach:
  - harness: my-agent-harness
    bundle: vision-swe
```

**Portability:** because every loop speaks canonical events — never host events — **a
bundle attached to one harness attaches to any other harness unchanged.** Bundles are
the unit of ecosystem reuse: you share enforcement profiles across harnesses, not
integrations per harness. That is the flash economy.

---

## 13. Harness registration

```yaml
# hearth.config.yaml
harness:
  mode: meta                    # native | meta | host
  compliance: enforce           # enforce | audit | observe   (validated against capabilities)
  audit:
    enforcement: shadow         # shadow | active              (audit flavor)
  host:
    id: my-agent-harness
    adapter: adapters/my_agent_harness.py
    capabilities:
      observe_prompts: true
      observe_tool_calls: true
      intercept_before_tool: true
      pause_execution: true
      resume_execution: true
      snapshot: true
      compensate: false
      supervise: true

traversal:
  default: default.hybrid       # the canonical baseline strategy
  overrides:
    - scope: harness:my-agent-harness
      strategy: harness-x.context.graph

loops:
  - id: modality.vision
  - id: tool.filesystem
  - id: tool.gate
  - id: scenario.swe-style
  - id: composition.span        # boundary loops — opt-in

registered_harnesses:           # other harnesses, registered as loops
  - id: planner-x
    as: loop
    scope: scenario

bundles:
  vision-swe:
    - modality.vision
    - tool.filesystem
    - tool.gate
    - verification.replay

attach:
  - harness: my-agent-harness
    bundle: vision-swe
```

Every harness is unique. The adapter exists precisely to honor that: it preserves each
harness's native control surface and maps it onto the one module contract. Nothing about
your harness needs to change to be controlled — and nothing about Hearth changes to
control it. The same holds for traversal: your harness's own navigation strategy is
registered, not replaced.

---

## 14. Contracts

| Contract | Binds | Verified by |
|---|---|---|
| **Goal contract** | desired endpoint, constraints, completion conditions | master loop — every iteration |
| **Subtask contract** | scope, dependencies, verification conditions | orchestrator + verification loops |
| **Modality contract** | modality-specific constraints | modality loop |
| **Tool contract** | tool usage rules | tool loop |

A contract without a loop is a document. In Hearth, there is no way to store an unbound
contract.

### Binding is structural, not conventional

"No unbound contract" is enforced **in the data model**:

- The contract record's schema **requires** a `bound_loop` field — a foreign key into
  the loop registry.
- A write referencing a loop id that does not resolve **fails validation at write
  time**. The store physically cannot persist an unbound contract.
- Deleting or unloading a loop **fails while any contract references it** — obligations
  cannot become orphaned by the registry changing underneath them.

The strongest idea in the design should not depend on anyone remembering the rule.

### Contract versioning

Contracts are **versioned**. The version in force is recorded in every checkpoint. A
mid-run contract change is itself a **finding** — and it raises R to the full dependency
chain, reusing the existing dynamic-R rule rather than inventing new machinery.

---

## 15. The canonical traversal graph

Every action, in any host environment, becomes the same record:

| Field | Meaning |
|---|---|
| `entity` | anything addressable — object, file, actor, feature, asset |
| `relationship` | contains / uses / references / depends_on / derived_from / … |
| `state` | what exists at this traversal point (delta-capable) |
| `action` | READ / INSPECT / CREATE / MODIFY / DELETE / NAVIGATE / VERIFY / REPLAY |
| `depth` | **traversal depth** — not hierarchy depth |
| `parent` / `children` | both `structural_parent` and `traversal_parent` |
| `active` | the active set — entities relevant at this depth |
| `created` / `modified` / `deleted` | state-transition events |
| `referenced` | maintained as a dependency without being active |
| `timestamp` | the whole graph is temporal |

### Two different graphs — both preserved

**Structural:** `Scene → Collection → Object → Mesh → Face`

**Traversal:**

```
Collection → Object → Material → Object → Mesh → Face
        └────────── revisit ──────────┘
```

Traversal depth is 6 even where structural depth is 3. Revisits, replays, and
backtracks are preserved, not collapsed. This distinction is why "depth" in Hearth means
something no file tree can express.

**And one substrate, read two ways.** The same graph also supports **context traversal**
— the dependency/relationship reading, distinct from the temporal reading.

### Example event

```json
{
  "run_id": "R001",
  "timestamp": "T173",
  "depth": 7,
  "entity": { "id": "obj_482", "type": "object", "native_type": "Blender:Object" },
  "action": "MODIFY",
  "relationship": {
    "structural_parent": "collection_12",
    "references": ["mesh_482", "material_91"]
  },
  "state": {
    "active": true, "created": false, "modified": true,
    "deleted": false, "referenced": true
  },
  "active_set_size": 3
}
```

### Software-agnostic by adapters

```
Blender ───┐
Maya ──────┤
Unreal ────┼──► adapters ──► canonical traversal graph ──► trace
Unity ─────┤
CAD ───────┘
```

Native application graph → canonical traversal graph → measurable traversal trace.
The benchmark measures agent traversal capacity, not the peculiarities of any one tool's
data model.

**The trace is a substrate, not a log.** Enforcement history feeds checkpoints, replay,
context traversal, drift attribution, benchmarks, the supervisor's view, and the
control-factor space. The verdict consumes the tick; the graph keeps everything.

---

## 16. Context traversal & synchronization

> **The store is not the product.** The store is the persistent substrate for a live,
> adaptable context-traversal and control system.

Hearth does not need to reconstruct or own the entire context at every tick. It needs to
maintain enough **state + relationships + checkpoints + traversal history** to determine
two things at every tick:

1. **What is relevant** to the current tick, and
2. **What prior state must be synchronized** before forward progression is allowed.

Not full context management — **relevance discovery plus synchronization**.

### The per-tick context flow

```
                    CURRENT TICK
                         │
                         ▼
                  ┌─────────────┐
                  │ CONTEXT     │
                  │ RESOLUTION  │
                  └──────┬──────┘
                         │
             find relevant prior state
                         │
                         ▼
              ┌─────────────────────┐
              │ IMPACT / BLAST AREA │
              │ dependency graph    │
              └──────────┬──────────┘
                         │
                    BACKWARD SYNC
                         │
                         ▼
              ┌─────────────────────┐
              │ CHECKPOINT STATE    │
              │ + contracts         │
              │ + prior decisions   │
              │ + dependencies      │
              └──────────┬──────────┘
                         │
                       VERIFY
                         │
                         ▼
                    CURRENT STATE
                         │
                  FORWARD SYNC
                         │
                         ▼
                  RELEASE TICK
```

Every tick: **look back, sync with the past, confirm, sync forward.** Neither direction
is optional on a non-CONTINUE verdict — a tick only advances once it is synchronized
with its own past.

### Two traversals — temporal and contextual

**Temporal traversal** — the execution timeline: `T1 → T2 → T3 → T4 → T5 → T6`

**Context traversal** — the dependency/relationship space:

```
             T6
            /  \
          A     B
          │     │
          C─────D
           \
            E
```

At T6, you do not necessarily need to replay T1–T5. You need to discover **which prior
state has causal or contractual relevance to T6** — `T2 → A → C → T6` — while the rest
of the historical context is irrelevant to this tick.

### The impact subgraph — blast radius, made concrete

```
CURRENT FINDING
      │
      ▼
affected entity
      │
      ├── depends_on
      ├── referenced_by
      ├── derived_from
      ├── modified_by
      └── contract_dependency
              │
              ▼
        IMPACT SUBGRAPH
```

> **blast_radius = the reachable affected state, under the applicable relationship
> semantics.**

And — the critical architectural separation — **the traversal algorithm determines how
that subgraph is discovered.** The kernel defines the contract; strategies compete
inside it. The result is always canonical; the navigation never is.

---

## 17. The session — the unit of graph ownership

There is no global universal context in Hearth — by design. A universal context would
impose Hearth's ontology on every harness, and Hearth's job is the opposite:

> **Each harness, each session, is unique — and each session builds its own map.
> Hearth enforces control semantics *around* that map. It does not enforce or restrict
> the map's design. It shapes around it.**

```
RUN
 │
 └── SESSION
      │
      ├── native harness state
      ├── contracts
      ├── checkpoints
      ├── traversal graph
      ├── control loops
      ├── findings
      ├── impact regions
      └── trace
```

Two sessions can have completely different shapes — a code session and a Blender scene —
and Hearth doesn't care. **The graph is simply the session's world model.**

### The session record

```
SESSION
 ├── ticks        state · checkpoint · observations · controls
 ├── traversal    nodes visited · edges followed · backtracks · forward syncs · strategy
 ├── impact       affected nodes · dependencies · blast/impact subgraphs
 ├── verification comparisons · mismatches · corrections · verification cost
 └── release      final state · verdict · confidence/evidence
```

Provenance + state + topology — enough to reconstruct **why a harness arrived at a
particular verdict**, without forcing every harness to take the same path.

### Never collapsed

> **The canonical store does not collapse a session into a single "final answer."**

```
what existed
      ↓
what was traversed
      ↓
what was inspected
      ↓
what was skipped          ← the negative space — measurable missed context
      ↓
what was discovered later
      ↓
what had to be corrected
      ↓
what that correction affected
      ↓
what was ultimately released
```

That chain is precisely what allows Hearth to measure **drift, correction, traversal
efficiency, verification cost, missed context, and control effectiveness** — the
skipped and the late-discovered are as load-bearing as the verified.

---

## 18. The API surface — four contracts + the connector layer

This is the point where Hearth connects to harnesses at the **context and store level**
— the boundary that determines whether Hearth remains a flexible benchmark/control
substrate or accidentally becomes a prescriptive execution framework.

```
                    HEARTH
                       │
        ┌──────────────┴──────────────┐
        │                             │
   CONTROL API                    TRACE API
        │                             │
 loops / rules /                  canonical events
 verdicts / gates                 checkpoints / states
        │                             │
        └──────────────┬──────────────┘
                       │
                 TRAVERSAL API
                       │
          ┌────────────┼────────────┐
          │            │            │
       Default      Harness A    Harness B
       Traverser    Traverser    Traverser
          │            │            │
          └────────────┼────────────┘
                       │
                  CONTEXT API
                       │
             backward / forward
             relationship / impact
             checkpoint sync
                       │
                  VERIFICATION
                       │
                    VERDICT
                       │
                    RELEASE
```

### The TraversalStrategy contract

The kernel defines the **contract for traversal, not the algorithm**:

```python
class TraversalStrategy(Protocol):
    id: str
    version: str
    capabilities: set[str]
    applicable_scopes: set[str]

    def resolve_context(self, tick) -> ContextResult: ...
    def find_dependencies(self, entity) -> Subgraph: ...
    def find_impact(self, finding) -> Subgraph: ...
    def select_checkpoints(self, context) -> list[Checkpoint]: ...
    def traverse(self, graph, query) -> TraversalRecord: ...
    def estimate_cost(self, plan) -> CostEstimate: ...
```

Hearth ships defaults — `default.temporal`, `default.dependency`,
`default.blast_radius`, `default.hybrid` — and any harness can register its own.
**The result contract stays canonical.** Harness A runs its algorithm; Harness B runs a
completely different one; both produce canonical context results into the same Hearth.

### The traversal primitives

```
get_checkpoint()        get_neighbors()         get_dependencies()
get_dependents()        get_impact_area()       traverse_backward()
traverse_forward()      sync_to_checkpoint()    verify_state()
record_traversal()
```

### The ownership split — frozen

```
HEARTH owns:
    What happened · what was controlled · what was verified
    what checkpoints exist · what relationships/impacts exist
    what was released · what can be reconstructed

HARNESS owns:
    How it traverses · what it considers relevant
    how it searches the graph · how aggressively it backtracks
    which checkpoints it synchronizes · which control loops it applies
    how it optimizes its own execution
```

### What the store says — and what it must never say

The canonical trace is an **evidence substrate**. It is capable of saying:

> *"At tick T, these states/checkpoints existed, these relationships were traversable,
> these impacts were observed, these controls were applied, these verifications
> occurred, and this was the resulting state."*

It must **never** collapse into:

> *"Harness A traversed the context this way."*

Against the same evidence, Harness A can backtrack-to-T5-verify-release while Harness B
does relationship-expansion-selective-verification-forward-sync. **Both operated against
the same canonical substrate.** That is what makes the benchmarking possible — and it is
why you are not benchmarking whether everyone uses *your* traversal algorithm:

> **You're benchmarking what happens when each harness's control system operates over
> its own context model — while Hearth captures the resulting control behavior in a
> common representation.**

### Three adaptation layers

| Layer | What adapts | Direction |
|---|---|---|
| **Host adaptation** | Codex, Claude, ChatGPT, custom harness, Blender, CAD → canonical events | host → Hearth |
| **Context adaptation** | harness's native state model → session graph | host → Hearth |
| **Traversal adaptation** | session graph → harness/domain-specific strategy → relevant context, impact, checkpoints | Hearth → host |

Host and context adaptation flow *into* Hearth. Traversal adaptation flows *out* —
Hearth provides the substrate; the harness provides the navigation.

### Why the boundary must stay loose

If Hearth dictated the internal architecture of every harness, you would accidentally
benchmark **compliance with Hearth's architecture**. Instead, Hearth benchmarks:

> **the control behavior that emerges from each architecture.**

That is why the freeze discipline cuts one way only:

> **Freeze the canonical model. Never freeze the traversal implementations.**

### The connector layer — the one kernel call

The kernel sees exactly one call. Every host, no matter how different, reduces to:

```
connector ── ingest(event) ──► HEARTH ── decision ──► connector ──► host mechanics
```

The adapter base class gives every connector the same skeleton:

```python
class BaseAdapter:
    host_id: str
    capabilities: CapabilityMatrix   # declared here; verified by probe

    def emit(self, kind, payload, *, actor="model", scope=None, refs=None) -> Decision:
        ev  = self._envelope(kind, payload, actor, scope, refs)
        dec = self.hearth.ingest(ev)     # ← the one call. Shadow mode: always allow.
        self._enforce(dec)               # verdict → host mechanics (per-host)
        return dec
```

`_enforce` is the only genuinely host-specific logic: how a `REPAIR` becomes a Claude
Code hook `deny`, how a `BACKTRACK(d)` becomes a fork or a browser edit-and-regenerate.
The kernel never knows.

**Canonical event envelope** (adapters produce it; Hearth assigns `tick`, `run_id`,
verdict — adapters never set those): `v`, `event_id` (ULID, idempotency key), `seq`
(adapter-monotonic), `ts_monotonic_us`, `session_ref` (host + host session + host
version + adapter version), `kind`, `actor` (`model|user|host|supervisor|connector`),
`turn_id` (host turn grouping — iteration identity ≠ event identity),
`parent_event_id`, `branch`, `scope`, `payload`, `integrity` (`raw_ref` + `hash` —
the exact bytes observed, the non-rewrite ledger), `probe` (probe events excluded from
metrics).

**Event kinds** (31): `session.started/ended`, `turn.started/ended`,
`prompt.submitted`, `model.response.started/completed`, `reasoning.emitted`,
`tool.proposed` (THE gate point — pre-execution), `tool.started/completed/failed`,
`approval.requested/decided`, `file.changed`, `state.changed`, `message.edited/deleted`
(conversation surgery), `conversation.rewound`, `branch.created/switched`,
`user.intervention`, `compaction.started/completed`, `subagent.started/completed`,
`checkpoint.created`, `usage.emitted`, `error.emitted`,
`supervisor.requested/resolved`.

**Decision wire format** (what `ingest()` returns): `decision_id`, `event_id`,
`verdict`, `hold`, `findings[]` (`loop`, `factor`, `severity`, `detail`), `action`
(`allow | block | hold | rewind | halt | annotate` + `reason` + `rewind{to_event_id,
phases, compensation_ref}` + `annotation`). Shadow mode: `allow` always, verdict and
findings recorded — which is exactly what makes tier-1 resim computable from the capture.

### The three transports

Every host on earth is one of these. Same SDK, same envelope, different plumbing.

| Transport | Shape | Gate? | Hosts |
|---|---|---|---|
| **Hook** | host invokes connector synchronously at action boundaries; connector blocks the thread until decision | ✓ synchronous | Claude Code hooks |
| **Pump** | connector spawns/attaches host process; reads event stream, writes control messages | ✓ at approval points | Claude Code headless/SDK, Codex exec/app-server |
| **Observer** | connector watches a surface it cannot control (DOM, network, log files) | ✗ shadow only | Browser clients, transcript backfill |

### Verdict → host mechanics, per transport

| Verdict | Hook (CC `PreToolUse`) | Pump (Codex app-server approvals) | Observer (browser) |
|---|---|---|---|
| `CONTINUE` | allow, silent | approve | no-op |
| `REPAIR` | `permissionDecision: deny` + reason → model self-corrects in place | deny approval + steer message | finding → supervisor surface / user acts |
| `BACKTRACK(d)` | block the call now; connector performs fork/compensation out-of-band | deny + connector restarts from fork point | connector drives UI edit/regenerate to the target message (if affordance exists) |
| `HALT` | deny + Stop-hook handler holds next submit | `interruptConversation` | drive stop / surface to user |

Honesty note baked into the design: a CLI harness cannot rewind its own in-flight
session arbitrarily — so the Claude Code adapter declares `snapshot: partial,
compensate: partial` and implements `BACKTRACK` as **session fork from transcript
prefix** (resume mechanics) plus **git-backed compensation** for file state. Declared
phases in the repair plan tell the connector exactly which of those to execute.

### Registration: the capability probe

Declared capabilities are **verified, not trusted**. At registration the connector runs
a probe suite over synthetic events tagged `probe: true` (excluded from all metrics):

```
probe.observe     does a synthetic event round-trip into the trace?
probe.gate        issue deny on a harmless tool call — did the host actually not execute it?
probe.resume      fork + resume — did the branch materialize in the trace?
probe.inject      annotation — did it surface where declared?
probe.compensate  execute the inverse action on a sandbox file — did state restore?
probe.supervise   hold registered → external decide → release per effective decision
```

Result: `capabilities: { declared: {...}, verified: {...} }` — both ship with every
trace. A mismatch (declared `gate: true`, probe shows the host executed anyway) fails
registration with a precise message, and would otherwise have silently poisoned
cross-host benchmark comparisons.

### Durability & ordering

- **Idempotency**: `event_id` (ULID) is the dedupe key — retries are safe, at-least-once
  delivery is the contract.
- **Spool**: connector writes every event to a local JSONL spool *before* daemon
  ingest; daemon acks mark spool positions; a crashed daemon loses nothing.
- **Ordering**: `seq` per session; out-of-order arrival tolerated — the graph is
  temporal, `ts_monotonic_us` breaks ties.
- **Backpressure**: shadow mode never blocks the host. If the daemon is slow, spool
  grows; the audit trail survives.
- **Fail mode**: `open` = allow + spool on daemon-unreachable (shadow semantics);
  `closed` = hold (ENFORCE semantics). Declared per adapter.

---

## 19. Metrics, for free

The enforcement core is also the instrument. Every verdict, every iteration, emits a
record. Nothing extra to wire — **the thing that enforces traversal is the same thing
that measures it.**

### Primary metric: AITD — Agentic Iterative Traversal Depth

The vector of iteration counts per serialization level — with the tick as the atomic
unit, **ticks per level**:

```
D = [I₀, I₁, I₂, …, Iₙ]
```

```json
{
  "depth_0": 12, "depth_1": 28, "depth_2": 25, "depth_3": 8,
  "max_depth": 3, "total_iterations": 73
}
```

AITD is reported as **distributions per level (p50 / p95), not just means** — a harness
that alternates between shallow and deep traversal is different from one that sits at
the average, and the mean hides that.

### The depth family — nine depths, one vector, never collapsed

| Depth | Meaning |
|---|---|
| **Traversal depth** | path length in the canonical graph, including revisits |
| **Serialization level** | registry nesting — what AITD indexes |
| **Replay depth (R)** | how many prior checkpoints each step re-verified |
| **Correction depth** | how far backward a correction propagated |
| **Backtrack depth** | checkpoints returned in a backtrack |
| **Dependency depth** | how deep the affected dependency chain reached |
| **Context depth** | depth in the relationship graph at which relevant prior state was found — discovered by traversal, not tick count |
| **Synchronization depth** | how far back state was re-synced before advancing |
| **Verification depth** | how deeply verification examined |

```json
{
  "traversal": 41, "serialization": 3,
  "replay_used": [3, 3, 7, 3], "correction": [2, 5], "backtrack": [2],
  "dependency": 9, "context": [2, 4], "synchronization": [4],
  "verification": [3, 3, 7]
}
```

The shape of the vector *is* the harness's control character: a harness that verifies
shallowly but synchronizes deeply is a different machine from one that does the reverse,
and no single number can express that difference (Invariant #17).

### Also emitted, per depth

| Metric | Meaning |
|---|---|
| `replay_depth_used` | R applied at that step |
| `total_replay_operations` | cumulative checkpoints replayed across the run |
| `replay_cost` | tokens / latency spent per replay |
| `correction_depth` | how far backward a correction propagated |
| `unique_entities` / `repeated_entities` | coverage vs revisits |
| `relationships_traversed` | edges touched |
| `interactions` | operations performed |
| `active_set_size` | A(d) — entities relevant at that point |
| `backtracks` | returns to earlier checkpoints |
| `state_changes` | created / modified / deleted |
| `blast_radius` | downstream decisions invalidated by each correction |

### The context family — traversal, skipping, and synchronization

| Metric | Meaning |
|---|---|
| `missed_context` | context that should have been traversed but wasn't — the negative space of the traversal record; a **lower bound**, reported honestly as such |
| `verification_efficiency` | what was safely skipped vs unnecessarily rechecked |
| `checkpoint_effectiveness` | how much historical synchronization a checkpoint enabled |
| `backward_syncs` / `forward_syncs` | synchronization operations per run |
| `relevance_precision` | relevant context found / total context traversed |
| `traversal_coverage` | reachable relevant context / captured context |

`missed_context` deserves emphasis: it is the measurement of what the harness *didn't
look at* — inferable only where the session graph holds the relationship evidence, and
therefore always reported as a lower bound. A harness that skips context is no longer
invisible; its skips are on the ledger.

### The control profile — violations as a measurement set

| Metric | Meaning |
|---|---|
| `native_violation_rate` | violations per tick observed in shadow — the harness unassisted |
| `shadow_violation_rate` | same, per scope/loop attribution |
| `prevented_violation_rate` | violations gated before execution under ENFORCE |
| `repair_rate` | share of ticks resolved by in-place repair |
| `backtrack_rate` | share of ticks requiring backward correction |
| `supervisor_escalation_rate` | share of ticks requiring a supervisor decision |
| `unresolved_violation_rate` | findings that neither repaired nor escalated |
| `supervisor_interventions` | absolute count of supervisor decisions per run |

And the headline pair — **kept deliberately distinct**, because they are different
numbers:

- **Correction rate** — the proportion of *behavior* Hearth corrected.
- **Control delta** — the improvement in *outcome* (percentage points) that enforcement
  produced.

Two harnesses can have the same correction rate and different control deltas, and vice
versa. Collapsing them would make the benchmark scientifically useless.

### Layered measurement — separating model, harness, and control

```text
MODEL
  ↓
MODEL + HARNESS
  ↓
MODEL + HARNESS + HEARTH
```

Separating: model capability · harness capability · tool-calling capability ·
planning/control capability · compliance · regression rate · recovery capability ·
supervision burden · traversal depth · enforcement overhead.

That is studying the **system** instead of pretending the model is the entire agent.

---

## 20. Overhead accounting

Per-tick enforcement has a cost, so it is measured like everything else.

| Metric | Meaning |
|---|---|
| `enforcement_overhead_per_tick` | latency added by steps 2–5 |
| `traversal_overhead_per_tick` | latency added by the strategy in step 3 |
| `loops_fired_per_tick` | resolution breadth |
| `verdicts_by_type` | CONTINUE / REPAIR / BACKTRACK / HALT distribution |
| `ticks_held` | ticks gated by non-CONTINUE verdicts |

### Tiered review

Not every tick pays for a deep replay. Review depth is tiered by trigger:

| Tier | When | Cost |
|---|---|---|
| **Cheap check** | every tick | contract bindings + prior verdict state |
| **Standard review** | phase boundaries | replay window at current R |
| **Deep replay** | high blast radius / architectural change / contract version change | full dependency chain |

This is the dynamic-R rule applied to the tick itself — no new machinery, same rule,
one level down.

---

## 21. The control-factor space — taxonomy & planes

Hearth's traces make a kind of benchmark possible that single-score agent evaluation
cannot attempt: measuring **control itself** as a space of independently addressable,
independently testable factors.

### What a control factor is

> **A control factor is an independently addressable control behavior** — verification,
> gating, synchronization, replay, escalation — **that can be present, absent, weakened,
> strengthened, or enforced, produces measurable intervention, and is fully traced.**

Four properties, all required:

1. **Placed** — every factor exists at a serialization level, on a plane (planning,
   tooling, memory, state, recovery, supervision).
2. **Parallelizable** — factors are measured independently; their benchmarks run as
   parallel experiments, not one monolithic test.
3. **Intervention-measurable** — every factor produces observable intervention: verdicts,
   repairs, backtracks, escalations.
4. **Traced** — every factor observation lands in the canonical graph. Nothing is
   measured outside it.

### The taxonomy — seven control domains

| Domain | Factors |
|---|---|
| **Planning control** | horizon · replanning · dependency preservation |
| **Tool control** | ordering · gating · verification · irreversibility |
| **State control** | synchronization · stale-state detection · checkpoint integrity |
| **Recovery control** | detection latency · correction depth · replay efficiency · recovery success |
| **Verification control** | frequency · depth · coverage · false-positive burden |
| **Supervision control** | escalation rate · intervention rate · unresolved findings |
| **Traversal control** | strategy selection · relevance precision · coverage vs cost · synchronization policy |

Each domain, each factor: **independently benchmarkable.** Traversal strategies are
registered like loops and ablatable like factors — the navigation itself is a factor
with states.

### The ablation ladder — the core experimental design

> **"What happens when each control factor is independently present, absent, weakened,
> strengthened, or enforced?"**

| State | Meaning |
|---|---|
| **ABSENT** | the factor is not in the run at all |
| **WEAKENED** | present but shallow (e.g., R=1, cheap checks only) |
| **PRESENT** | default strength |
| **STRENGTHENED** | deep (e.g., R=10, full dependency chains) |
| **ENFORCED** | present *and* gating — findings hold the tick |

Five states per factor. Every state is a traced run. The comparison across states is
the measurement of what that factor is actually *worth*. And since #7b, the ladder runs
**live** — real kernels, real gating, real consequence divergence
([§44](#44-the-laboratory)).

### The factorial dataset — the benchmark output

Every observation is a point in the factor space:

```text
(model=M1, harness=H1, factor=tool.verify, level=L2, modality=code,
 tool=filesystem, traversal_strategy=default.hybrid, R=3,
 checkpoint_policy=rolling, supervision=none)
        ↓ outcome ↓ trace
```

The dimensions:

```text
MODEL · HARNESS · CONTROL FACTOR · CONTROL LEVEL · MODALITY · TOOL · TASK
TRAVERSAL STRATEGY · REPLAY DEPTH · CHECKPOINT DEPTH · VERIFICATION DEPTH
SUPERVISION · STATE-SYNC POLICY · MEMORY POLICY
+ BOUNDARY TYPE · SUB-HARNESS COUNT · NESTING DEPTH   (composition, §39)
+ PARALLEL EFFICIENCY · SPECULATION WASTE             (scheduler, §40)
```

Which means the benchmark database is not a leaderboard. It is a **factorial dataset** —
queryable, sliceable, recombinable. Any question of the form *"what does factor F
contribute at level L under posture P with strategy S?"* is a query, not a new
benchmark.

Factor definitions are registry objects — machine-readable, addressable like loops,
versioned like contracts — so the taxonomy itself is part of the infrastructure, not a
page in a paper (C1, shipped — [§44](#44-the-laboratory)).

---

## 22. Benchmark: harness power === traversal power

### The R sweep

Same model. Same task. Same tools. Sweep only **R**:

| R | Success | Cost | Regressions |
|---|---|---|---|
| 0 *(trust everything)* | 61% | 1.0× | 18% |
| 1 | 68% | 1.1× | 14% |
| 2 | 74% | 1.2× | 10% |
| 3 | 81% | 1.4× | 7% |
| 5 | 84% | 1.8× | 5% |
| 10 | 85% | 2.9× | 4% |
| ∞ *(full trajectory)* | 87% | 8.4× | 2% |
| **adaptive** | **86%** | **2.1×** | **3%** |

*(illustrative numbers — the point is the axis, not the values; the live R-sweep now
runs as a tier-1 resim op AND as a live lab factor)*

The interesting result is never "more replay = better." It is the **Pareto frontier** —
the replay depth where the reliability gain stops being worth the cost.

### Why external is the only way this benchmark exists

```text
                 SAME TASK
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
     Codex       Claude       ChatGPT
        │           │           │
        ▼           ▼           ▼
     Hearth      Hearth       Hearth
        │           │           │
        └───────────┼───────────┘
                    ▼
             SAME CONTRACT
             SAME TICK MODEL
             SAME VERDICTS
             SAME TRACE
             SAME METRICS
                    │
                    ▼
              COMPARABLE DATA
```

Bake the control machinery into one harness and you can measure that harness. Keep it
external and you can measure **the harness itself** — which is the entire point.

> **Is the model actually more capable — or is its harness giving it more effective
> traversal and control capacity?**

Capability should be reported at the model × harness configuration level. Hearth makes
that not just possible but automatic.

### Shadow vs enforced — the Hearth Delta

```text
Harness A
  Shadow success:       72%
  Enforced success:     91%
  Control delta:       +19pp
  Correction rate:      19%
  → Native control is weak. Hearth compensates heavily.
    19% of behavior required external correction.

Harness B
  Shadow success:       86%
  Enforced success:     89%
  Control delta:        +3pp
  Correction rate:       3%
  → Native control is already strong. Hearth mostly verifies.
```

The **Hearth Delta** is the answer to: *how much of this harness's apparent capability
is actually external control?*

### Factor deltas — the Hearth Delta, decomposed

```text
                SHADOW → ENFORCED DELTA

tool.gate              +8pp
state.sync             +6pp
verification           +11pp
replay                 +4pp
checkpointing          +7pp
recovery               +13pp
```

Now you know *which control is doing the work* — per harness, per release.

### Interaction effects — which control actually matters, jointly

```text
verification × replay          +17pp
checkpoint × state-sync        +14pp
tool-gate × verification       +12pp
```

An interaction larger than the sum of its parts means two factors compound — the harness
gets more from verification *when replay backs it* than either buys alone. This is how
the field moves from "agents need control" to **"these control factors matter, these
interact, these are dead weight for this harness"** — the factor ablation ladder is the
instrument, and the 2^k factorial designs now compute these contrasts live
([§44](#44-the-laboratory)).

### The strategy sweep — benchmarking the traversal algorithm itself

Because traversal strategies are registered, swappable, and canonical in their results,
**the algorithm itself becomes an experimental variable**:

```
Harness A
 ├── temporal
 ├── dependency
 ├── impact
 └── hybrid
```

Change only the strategy and you measure:

> **How much control capacity is obtained from a particular context-traversal policy,
> at a particular cost?**

That question — capacity per unit of traversal cost, per strategy — was previously
unaskable. Strategy joins model × harness as a first-class benchmark axis.

### One model is enough — the expansion path

```text
Model A + Harness A + Hearth
  → establish: trace → factors → interventions → correction → cost → counterfactuals
```

Once that works, the model × harness grid becomes primarily an **adapter and
data-acquisition problem** — not a new benchmark architecture. That is the payoff of
Hearth being external.

### The control profile — the behavioral fingerprint

```text
                HARNESS CONTROL PROFILE

Native success                 84%
Shadow violations               9%
Enforced success               91%
External correction             7%
Backtrack rate                  2%
Repair rate                     5%
Supervisor escalation           1%
Persistent drift                3%
Recovery success               94%
Control overhead                1.7×
```

Same contracts applied across harnesses → comparable control fingerprints. This is the
benchmark output, not a summary statistic.

### Auto mode vs approval mode

```text
Tool calls:              183
CONTINUE:                151
REPAIR:                   17
BACKTRACK:                 9
SUPERVISOR REQUEST:        6
HALT:                      0
```

That is a measurable **control profile** — far more informative than
"agent completed task: yes."

---

## 23. The canonical baseline & unbiased aggregation

**Defaults are not a contradiction to adaptability — they are the baseline against
which adaptability can actually be measured.**

```
                         HEARTH
                           │
              CANONICAL / DEFAULT BASELINE
                           │
          ┌────────────────┼────────────────┐
          │                │                │
     Default Loops    Default Traversal   Default Sync
          │                │                │
          └────────────────┼────────────────┘
                           │
                    HARNESS ADAPTER
                           │
             ┌─────────────┼─────────────┐
             │             │             │
          Harness A     Harness B     Harness C
          custom        custom        default +
          strategy      strategy      extensions
             │             │             │
             └─────────────┼─────────────┘
                           │
                      SESSION TRACE
                           │
                    CANONICAL STORE
                           │
                 ┌─────────┴─────────┐
                 │                   │
             BENCHMARK          SIMULATION
                 │                   │
          individual results    eligible population
                 │                   │
                 └─────────┬─────────┘
                           ↓
                  UNBIASED AGGREGATION
```

- **Benchmark** — *how did this specific model/harness perform under the canonical
  conditions?* Its complete trace is preserved.
- **Simulation** — *given the same underlying data/context/control problem, what
  happens across every model/harness that satisfies the benchmark requirements?*

**Eligibility is established first.** The aggregation population is defined by explicit
requirements — never by which model happens to be convenient, preferred, or selected
beforehand. That is what makes the aggregation unbiased.

The default is the **control population**; custom harnesses are the **experimental
variations**. And one model → many models, trivially: if one model can be run through
the control/traversal substrate, the same canonical problem can be replayed/simulated
across the entire eligible model population.

> **Hearth isn't just measuring a winner.** It establishes the **behavioral
> distribution of all qualifying systems under equivalent conditions** — while still
> retaining each individual system's actual trace and correction history.

That is a different benchmark architecture: **canonical baseline + pluggable execution +
reproducible simulation + population-level aggregation.**

---

## 24. Control economics — cost to correction

### The cost-to-correction decomposition

```text
finding
   ↓
detection depth      ← how late it was caught
   ↓
correction depth     ← how far back the fix reached
   ↓
replay depth         ← what had to be re-verified
   ↓
resynchronization    ← state re-synced with past checkpoints before advancing
   ↓
downstream invalidation  ← what became garbage
   ↓
re-execution         ← what had to be redone
```

```text
Cost(correction) =
    detection cost + synchronization cost + rollback cost
  + replay cost + re-execution cost + downstream invalidation cost
```

And the law underneath all of it:

> **The cost depends on when you detect the problem.**

### The price of cutting corners

```text
Correct control:                        Cheap control:

T1 ─ T2 ─ T3 ─ VERIFY ─ T4 ─ T5         T1 ─ T2 ─ T3 ─ T4 ─ T5 ─ T6 ─ T7 ─ VERIFY
                  ↑                                                         ↑
              catches error                                            catches same error

early detection:   correction depth 1 · replay 1 · downstream invalidation 1
late detection:    correction depth 5 · replay 5 · downstream invalidation 4
```

This converts a vague virtue into a measured quantity — the **economic value of
verification**:

> Not "did it verify?" but **"how much did insufficient verification eventually cost?"**

Cost-cutting on control is no longer invisible — it is a debt, and the trace is the
ledger.

### Context economics — the cost of what was skipped

- **Missed context cost** — context skipped at tick T that was discovered downstream the
  hard way. The gap between `missed_context` and `correction_depth` is the price of not
  looking.
- **Verification efficiency** — what was safely skipped versus unnecessarily rechecked.
- **Checkpoint effectiveness** — how much historical synchronization each checkpoint
  enabled.

Together these answer, per harness and per strategy: *at what traversal depth does this
harness stop synchronizing state, and what did that eventually cost?*

### Counterfactual back-simulation — one run, many policies

```text
                     SAME TRACE / STATE
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
           R=0            R=3            R=7
             │              │              │
          outcome        outcome        outcome
             │              │              │
             ▼              ▼              ▼
           cost           cost           cost
         errors         errors         errors
       recovery       recovery       recovery
```

Questions the captured trace answers directly:

```text
If verification had happened here:                would the known downstream violation have been detected?
If checkpoint synchronization had happened here:  would the later action have been invalidated?
If R had been 7 instead of 3:                     would this regression have been caught?
If the tool gate had existed:                     would this irreversible action have been released?
If stale state had been detected:                 how many downstream ticks become unnecessary?
```

The trace becomes a **counterfactual control laboratory** — now running as the service's
tier-1 resim engine ([§42](#42-the-benchmark-service)).

### Counterfactual validity — three tiers, declared

| Tier | Name | Question | Validity |
|---|---|---|---|
| 1 | **State-bound** | given captured state, would policy P have detected / prevented X? | computable from the capture — sound |
| 2 | **Checkpoint re-execution / CFX** | re-run from a checkpoint, or replay the passage, under policy P | sound when the environment is captured / deterministic from that point |
| 3 | **World-generation** | what would a *different model* have done downstream? | **not simulation** — requires live re-execution; the capture cannot know an unbuilt world |

No simulation pretends to be more valid than it is. **Tier is a field on every
counterfactual result** (Invariant #18).

### One capture, many policies

Because tier-1 counterfactuals are pure functions of the captured trace, **one recorded
run re-scores against many control policies**. Shadow mode is what makes this honest:
the harness behaves unassisted, Hearth records what *would* have intervened — so the
intervention / no-intervention contrast is **measured, not assumed**.

Detect, attribute, correct, measure, simulate, price — **one engine, six outputs.**

---

## 25. Drift — detection, attribution, correction, recurrence

Drift tracking is **not another subsystem**. It is a natural consequence of the core:
Hearth observes control behavior → deviations become findings → findings trigger the
existing verdict/repair/replay machinery → the resulting history is the benchmark.

### Drift is a time series

```text
                     Native      Hearth
                     Success     Correction
────────────────────────────────────────────
Version 1.0           86%          4%
Version 1.1           84%          6%
Version 1.2           81%          9%
Version 1.3           76%         14%
Version 1.4           72%         19%
```

The benchmark doesn't merely say "Version 1.4 is worse." It says:

> **The harness increasingly violated the same control contracts, and Hearth had to
> compensate for progressively more of that drift.**

### Attribution — knowing *where* it drifts

```text
Harness A
├── planning           drift +4%
├── tool calling       drift +9%
├── verification       drift +2%
├── state management   drift +3%
└── recovery           drift +1%
```

And one level deeper, down to individual loops:

```text
tool.filesystem
    ├── read-before-write violations
    ├── stale-state usage
    ├── unnecessary retries
    └── missing verification

tool.gate
    ├── irreversible action attempted
    ├── replay required
    └── supervisor escalation
```

**The drift measurement falls out of the loop's own enforcement records.** No separate
drift detector is pointed at the harness; the loops *are* the detector.

### The drift lifecycle

```text
DETECT DRIFT → ATTRIBUTE DRIFT → CORRECT DRIFT (reuses REPAIR/BACKTRACK/replay)
→ MEASURE CORRECTION → TRACK WHETHER DRIFT RETURNS
```

The last stage is the diagnostic that matters most:

```text
T1 → violation → repair
T2 → violation → repair
T3 → violation → repair
T4 → violation → repair
```

Hearth distinguishes:

- **Temporary execution error** — same finding, rare, doesn't recur after repair.
- **Persistent harness-level control deficiency** — same finding, repeatedly, across
  runs and releases.

`persistent_drift` and `recurrence_count` are first-class metrics. A harness with a
persistent finding isn't unlucky — it is *shaped* that way, and the benchmark says so
with numbers.

### Drift correction, not merely drift detection

```text
Baseline                          Release N
tool-call compliance: 97%    →    89%
verification:         94%    →    91%
state consistency:    98%    →    93%
```

Hearth detects the regression. With enforcement enabled, the same machinery corrects it
inline:

```text
Release N → violation → Hearth → REPAIR → REPLAY → verify → continue
```

---

## 26. The supervisor — the control plane

The supervisor is not just "the outermost loop." It is the **human/system control
plane** above the entire kernel:

```text
                    SUPERVISOR
                 /      |       \
                /       |        \
           mobile      web      API
              \         |        /
               \        |       /
                └── SUPERVISION ──┘
                        │
                        ▼
                    HEARTH CORE
                        │
              ┌─────────┼─────────┐
              ▼         ▼         ▼
           Codex     Browser    Custom
           Claude    ChatGPT    Harness
```

### What the supervisor decides

- **Resolves `HALT`** — resume, repair, or abandon.
- **Approves or rejects `tool.gate`** — irreversible actions can require explicit
  supervisor confirmation.
- **Force-backtracks** any loop below it.
- **Raises R** on anything below it — between ticks.
- **Amends contracts** — as versioned changes, which automatically trigger full-chain
  replay.
- **Answers clarification requests** — when a loop's finding is ambiguous, the tick can
  surface a question, not just a verdict.

### Who the supervisor is

| Type | Behavior |
|---|---|
| **Human supervisor** | person, via mobile / web / API |
| **AI supervisor** | a model with its own control loop — itself supervisable |
| **Hybrid** | AI proposes, human confirms above a policy threshold |
| **Policy supervisor** | deterministic rules; no intelligence, only policy |

Each type is itself a benchmarkable configuration: the same run under human vs AI vs
policy supervision produces comparable supervision-demand profiles.

### The trust model — stated explicitly

> **The supervisor is the ultimate policy authority.** Loops enforce contracts; the
> supervisor *owns* them. It can override any verdict's disposition, raise or lower R,
> amend contracts, and approve irreversible actions. Everything it does is itself a
> tick — traced, measured, and attributed like everything else. Supervision authority is
> delegated downward; it is never assumed by the kernel.

### Supervision demand is a capability dimension

```text
Agent A                          Agent B
completion: 94%                  completion: 91%
supervisor interventions: 17     supervisor interventions: 2
backtracks: 11                   backtracks: 3
repairs: 22                      repairs: 6
```

Depending on the deployment, **B may be the better autonomous system** — it needed far
less external control to be nearly as correct. The question Hearth makes answerable:

> **How much external control does this system require to remain correct?**

That is a legitimate capability dimension, and it was previously unmeasured.

---

## 27. The kernel contract — frozen

Everything in this document deploys or composes the following. This is the part every
feature must prove itself against, and it is frozen:

```text
Tick → Loop resolution → Traversal → Step → Verdict aggregation
     → Context sync → Checkpoint/repair → Trace → Release
```

```text
                 CONTRACTS              SESSION GRAPH (per session)
                     │                        │
                     ▼                        │
HOST ──► TICK ──► RESOLVE ──► TRAVERSE ──► LOOPS ──► VERDICT
                     │             ▲                   │
                     │        TRAVERSAL API             ▼
                     │      (pluggable strategies)  SYNC / REPAIR / BACKTRACK
                     │             │                   │
                     │         CONTEXT API ◄───────────┘
                     │      (backward + forward sync)
                     ▼                    │
                  TRACE ◄──────────── CHECKPOINTS
                     │
                     ▼
                 METRICS / AITD / FACTORS
```

The frozen statements:

1. **The pipeline is fixed.** Eight steps: intercept, resolve, traverse, step,
   aggregate, sync, emit, release. One event → one pipeline run → one tick record.
   Loops add checks; strategies add traversal — nothing adds ticks or steps.
2. **The verdict protocol is closed.** `CONTINUE / REPAIR / BACKTRACK(d) / HALT` —
   worst verdict wins, `dmax` on competing backtracks, nothing else escapes the core.
3. **The capability boundary is declared.** Hearth defines what must be observable, what
   may be intercepted, what may be gated, and what verdicts mean; the adapter declares
   what the host actually provides; the declaration ships with every trace.
4. **The trace is the substrate.** Every tick is recorded into the canonical traversal
   graph; every metric — **including every control factor and every traversal record** —
   is derived from it; nothing is measured outside it.
5. **Release authority is the only variable.** Observe, audit (shadow/active), enforce
   (auto/supervised) — same path, different authority. Benchmarking and meta-harnessing
   are one mechanism.
6. **The canonical model is frozen; traversal implementations never are.** Hearth owns
   what happened, what was verified, what was released; the harness owns how it
   navigates. Strategies are pluggable; results are canonical.
7. **Sessions own their graphs.** No global universal context exists; every session
   builds its own world model, and the store never collapses it to a final answer.
8. **Defaults define the reference population.** The canonical baseline is what
   adaptability is measured against, and aggregation runs over eligible systems only —
   eligibility explicit, never convenient.

Everything else is **how you deploy and compose this kernel**.

---

## 28. Quick start

*(the reference implementation is shipped — Part II; commands are real)*

### Native

```python
from hearth import Hearth

h = Hearth(config="hearth.config.yaml")
h.goal("/goal: ship the refactor with zero regressions")
result = h.run()            # enforces ordering, traverses context, syncs, backtracks

print(result.aitd)          # [12, 28, 25, 8]
print(result.depth_vector)  # all nine depths
print(result.verdicts)      # every CONTINUE / REPAIR / BACKTRACK / HALT, in order
```

### Meta — wrap an existing harness

```python
from hearth import Hearth, Adapter

h = Hearth(mode="meta", compliance="enforce", adapter=Adapter.for_host("my-agent-harness"))
h.attach_bundle("vision-swe", to="my-agent-harness")

result = h.run(host=MyAgentHarness())   # your harness runs; Hearth enforces
```

### Benchmark a public agent — shadow (real, shipped)

```python
from hearth import Hearth, Adapter

h = Hearth(mode="meta", compliance="audit", audit_enforcement="shadow")
h.attach_adapter(Adapter.for_host("browser:chatgpt"))

result = h.run(workload="swe-bench-lite", seed=7)
print(result.control_profile)
# native_violation_rate, correction_rate, control_delta, supervisor_interventions, ...
```

### Flash in a control loop

```python
from hearth import ControlLoopModule

class VisionLoop(ControlLoopModule):
    id = "modality.vision"
    scope = "modality"
    trigger = "modality:vision:enter"
    horizon = {"forward": 5, "replay": "adaptive"}
    composes_with = {"tool.camera", "scenario.research"}

    def step(self, state): ...
    def verify(self, state): ...
    def backtrack(self, d): ...
    def emit(self): ...

hearth.register(VisionLoop)
```

### Register a traversal strategy

```python
from hearth import TraversalStrategy

class DependencyFirst(TraversalStrategy):
    id = "harness-x.context.dependency"
    version = "1.0"
    capabilities = {"dependency_walk", "impact_discovery"}
    applicable_scopes = {"harness:my-agent-harness", "modality:code"}

    def resolve_context(self, tick): ...
    def find_dependencies(self, entity): ...
    def find_impact(self, finding): ...
    def select_checkpoints(self, context): ...
    def traverse(self, graph, query): ...
    def estimate_cost(self, plan): ...

hearth.register_traversal(DependencyFirst)   # results canonical; navigation yours
```

### Read the trace — the real commands

```bash
# Claude Code connector (Artifact #2):
python hearth_ctl.py selftest                     # 13 checks, no daemon needed
python hearth_ctl.py start --mode enforce         # or audit (default) for shadow
python hearth_ctl.py register --capabilities capabilities.claude-code.json
python hearth_ctl.py install-hooks                # merges ~/.claude/settings.json
python hearth_ctl.py decide <decision_id> approve # supervised holds
python hearth_ctl.py rewind <sid> --to <evt> --phases rewind,repair,replay
python hearth_ctl.py submit --run R001 --redact   # → benchmark bundle

# Codex connector (Artifact #3):
python hearth_codex_ctl.py selftest               # fake app-server end-to-end
python hearth_codex_ctl.py gate-run "refactor auth" --mode enforce
python hearth_codex_ctl.py pump-run "explore repo" -- --skip-git-repo-check

# Probes (Artifact #4):
python hearth_probe_ctl.py register --host claude-code \
    --capabilities capabilities.claude-code.json --with-host

# Kernel + metrics (Artifacts #5–6):
python -m hearth_kernel.cli report --spool ~/.hearth/spool --sid <sid>
python -m hearth_kernel.cli tree   --spool ~/.hearth/spool --sid <sid>   # nested spans
python -m hearth_metrics.cli report --spool ~/.hearth/spool --sid <sid>
python -m hearth_metrics.cli calibrate            # publishable loop-precision table
python -m hearth_metrics.cli delta --spool ~/.hearth/spool \
    --shadow <sid-shadow> --enforced <sid-enforced>

# Service (Artifact #8):
python -m hearth_service.cli serve --root ~/.hearth/service --require-full-schema
curl -X POST localhost:8077/v1/traces -d @bundle.json
curl -X POST localhost:8077/v1/traces/$BID/resim -d '{"op":"r_sweep"}'
curl -X POST localhost:8077/v1/delta -d '{"shadow":"'$SH'","enforced":"'$EN'"}'
curl -X POST localhost:8077/v1/cfx -d @campaign.json
curl -s "localhost:8077/v1/benchmarks/wl/distributions?group_by=host,factors"

# Scheduler (Artifact #7b):
python -m hearth_kernel.cli schedule --plan-file plan.json --max-width 4

# Laboratory (Artifact #9):
python -m hearth_lab.cli factors
python -m hearth_lab.cli ablate --workload wl-core.json --factors tool.gate,enforcement
python -m hearth_lab.cli factorial --workload wl-interact.json \
    --factors verification.replay,context.sync
python -m hearth_lab.cli drift --store drift.json --workload wl-core --report
```

---

## 29. Design invariants

1. **No artifact is trusted without a loop over it.**
2. **Every loop is a module.** One contract, no exceptions.
3. **Every iteration emits a trace record.** No silent steps — and no silent traversals;
   the traversal record itself is emitted.
4. **Outer owns ordering; inner yields.**
5. **Only the four verdicts escape the core.** Nothing else crosses the boundary.
6. **R is dynamic — and recorded per step.** R is the bounded default; graph traversal
   extends the window along relationship edges when relevance demands it.
7. **Native, meta, host: zero architectural change between modes.**
8. **Hearth does not compete for "who is the agent" — it owns "did the stored
   obligations still hold."**
9. **Stopping is always a verdict, never a silent end.** Every loop has a soft budget;
   exhaustion surfaces as `HALT` to the supervisor — never a quiet fade.
10. **Hearth reads ticks; it never rewrites them.**
11. **The pipeline is fixed (eight steps); loops add checks and strategies add
    traversal — never ticks or steps.** Addition is monotonic.
12. **Worst verdict wins — and on competing `BACKTRACK` depths, `dmax` wins.** Many
    reviewers, one gate; disagreement degrades toward caution, never toward optimism.
    Dissenting findings are recorded, not discarded. **And the rule extends through
    nesting** — a BACKTRACK crossing a span boundary costs +1.
13. **Measurement and enforcement are the same control path with different release
    authority.** One engine; the deployment chooses the authority.
14. **Hearth never assumes how a host executes.** The adapter declares what the host
    actually provides — and the declaration is part of every trace and every benchmark
    configuration.
15. **No unbound contract is structurally representable.** Binding is enforced by the
    data model at write time, not by convention.
16. **Supervision demand is a first-class measurement.** How much external control a
    system requires to remain correct is a capability dimension, reported per run.
17. **The depth family is never collapsed.** Nine depths, one vector — collapsing them
    destroys exactly the information the benchmark exists to produce.
18. **Every counterfactual declares its tier.** State-bound, checkpoint re-execution,
    world-generation — no simulation pretends to be more valid than it is.
19. **Every control factor is ablatable.** Any factor can be present, absent, weakened,
    strengthened, or enforced — and the ablation is itself a traced run.
20. **Hearth is the substrate, not a factor.** It measures and enforces the
    control-factor space; it is not one axis within it.
21. **Freeze the canonical model; never freeze traversal.** Hearth owns what happened
    and what was released; the harness owns how it navigates. Strategies are pluggable;
    results are canonical.
22. **Sessions own their graphs.** No global universal context — every session builds
    its own world model, and Hearth supplies control semantics around it, never an
    ontology for it.
23. **The skipped is recorded.** What was traversed, inspected, *skipped*, discovered
    later, corrected, and released — all preserved; the store never collapses to a
    final answer.
24. **Traversal strategies are first-class registrable objects** — benchmarked like
    factors, swappable like loops, canonical like verdicts.
25. **Defaults are the reference population.** The canonical baseline is what
    adaptability is measured against.
26. **Aggregation is over the eligible, only.** The population is defined by explicit
    requirements — never by convenience, preference, or prior selection.
27. **The atom is checkable.** Five elements — contract, loop, verdicts, checkpoint,
    tick — verified at registration; below the atom there is no control, only behavior.
28. **Boundaries are ticks.** Every sub-harness handoff, user intervention, and tool
    boundary is an event on the same substrate — which is why cross-boundary failures
    are measurable at all.
29. **Declared capabilities are verified, not trusted.** The probe suite runs at
    registration; a mismatch is a refusal, and a forced correction is recorded with
    provenance.
30. **Failures of consumers are results, not errors.** A harness that cannot consume a
    passage is recorded as `failed-to-consume` — never dropped, never retried until it
    passes.
31. **The plan is a contract.** Declared read/write sets are checked against observed
    sets; a write outside the plan is a finding and the work is discarded — priced.
32. **The depth family is never collapsed — including the parallel profile.** Makespan,
    bound speedup, realized speedup, width, utilization, waste, and fidelity are a
    family, never a single number.
33. **Scheduling decisions are ticks.** The scheduler is itself traced on the plan
    entity like everything else it governs.
34. **Governs the governance.** The invariant library is versioned, red-teamed,
    published — attacked the way protocols are attacked; new invariants earn admission
    through the laboratory, with counterfactual evidence.
35. **Context passes through untouched.** Hearth delivers the passage; it never
    pre-digests, re-chunks, summarizes, or annotates user context on the way in. The
    machinery belongs to the harness.

---

## 30. Limitations

Stated plainly, because a harness that demands honesty from its hosts should show some:

- **Adapters must be real.** The canonical graph is only as good as each host's adapter.
- **Adaptive-R rules are policy.** The dynamic-R table is a sensible default, not a law
  of nature.
- **Replay costs tokens and latency.** Measured, not hidden.
- **ENFORCE requires gate + resume capabilities.** Hosts without them degrade along the
  control path — documented, recorded.
- **Browser adapters are bounded by the environment.** Capability declarations will
  honestly contain `partial` and `false`.
- **Supervisor quality bounds enforcement quality.** An AI supervisor is itself a system
  under test.
- **Shadow findings are not prevention.** Irreversible damage in shadow mode is real
  damage — that is precisely the information shadow mode exists to capture.
- **Counterfactuals are tiered — and tier 3 is not simulation.** World-generation
  requires live runs.
- **Missed context is measurable only where the capture has evidence.** Always a lower
  bound, evidence basis recorded.
- **Cross-strategy comparison requires the same session substrate.** The anchoring is
  declared.
- **The factor space is combinatorial.** The taxonomy makes sampling principled, not
  exhaustive.
- **Aggregation inherits eligibility definition quality.** Requirement profiles are
  versioned and published alongside results.
- **Design-stage numbers are illustrative** — until a workload run earns them.
- **File attribution is tool-level, not kernel-level** in the CC connector — files
  modified via Bash are compensated only insofar as they're git-tracked. `compensate:
  partial` is priced into the declaration rather than hidden.
- **Fork replay is shadow-gated** — replay runs carry `shadow: true`; the trace says
  what was observed vs enforced.
- **Codex gate coverage = approval routing** — only actions that route through
  `approval_policy` are gateable; the declaration says exactly this.
- **Wire drift is the real connector risk** — method names and item shapes vary across
  host versions; the probe suite verifies the installed binary at registration.
- **Positional span attribution is sequential-only** — concurrent subagents interleave
  without per-call span markers in current connectors; declared spans are the exact
  path, native Hearth provides them.
- **Replay depth is currently insensitive under cumulative checkpoint manifests** — a
  self-found finding; interval manifests are the fix ([§59](#59-errata--findings-about-ourselves)).
- **Loop precision bounds everything.** Every downstream number is only as good as the
  loops producing findings — which is why precision is published, versioned, calibrated
  ([§41](#41-the-metrics-suite), [§56](#56-the-four-faces--the-viability-verdict)).

---

## 31. Prior art

The full surveyed landscape lives in [`docs/PRIOR_ART.md`](docs/PRIOR_ART.md) and in the
survey at [§47](#47-the-prior-art-survey--four-graphs-one-gap). The solidly citable pieces:

- **ReAct** — interleaved reasoning and acting; the base agent loop.
- **MPC / receding-horizon control** — plan, execute a portion, re-plan on new state.
- **Tree of Thoughts / search over reasoning paths** — branch, evaluate, backtrack.
- **Backtracking search** — return to an earlier decision point on downstream failure.
- **Checkpoint + replay engineering** — save points, revalidate from them.
- **Factorial experiment design** — factors, levels, interactions, ablation.
- **Graph traversal (BFS/DFS and friends)** — the strategy toolbox; in Hearth the
  algorithms are pluggable strategies rather than a fixed implementation.
- **Meta-analysis** — eligibility criteria and population-level aggregation.
- **Bernstein conditions** — the independence test the scheduler imports.
- **Financial audit / aviation checklist discipline** — the shape-over-content
  verification pattern every mature high-stakes industry runs on.

What did not appear as one named object: enforcement-over-artifacts as the *identity*
of a harness, no-unbound-contracts by construction, dynamic R as both first-class
control and benchmark axis, dual wrap/be-wrapped under one module contract and four
verdicts, a flash vest of stackable loops with explicit precedence, the Hearth Delta as
a benchmark of external-control contribution, the control-factor space as a factorial
benchmark substrate, counterfactual back-simulation with declared validity tiers,
cost-to-correction as a decomposed benchmark metric, traversal strategies as
registered/benchmarkable/swappable objects over a canonical per-session evidence
substrate, the canonical baseline + eligibility gate + population-level aggregation
architecture, supervision demand as a capability dimension, cross-boundary movement
metrics, and AITD plus the canonical traversal graph as free measurement of harness
power. **The pieces existed. The binding did not.**

---

## 32. Roadmap

**Done — shipped and selftested (Part II):**

- [x] Wire contract — 8 schemas + golden vectors (hearth, decision, capabilities,
      session_bundle, tick, envelope, passage, session-graph, plan)
- [x] Claude Code connector — daemon (UDS, spool, idempotency, two-pass validator),
      hook client, stream-json pump, fork-BACKTRACK + git compensation, ctl, selftest
- [x] Codex connector — app-server gate client (approval→ingest→approve/deny, steer,
      interrupt), exec pump, shared mapping, fake app-server, ctl, selftest
- [x] Probe suite — six probes, grading floors, mismatch=refusal, --force corrections
- [x] Kernel extraction — loop registry, 8-step pipeline, tick records, aggregation,
      four builtin loops, traversal strategies + GraphIndex
- [x] Session graph builder + depth vector + AITD (nine depths, distributions)
- [x] Metrics suite — control profile (with black-box mode), economics, missed-context,
      boundaries, calibration, envelopes registry
- [x] Recursive composition — spans, nested graphs, verify_atom, composition.span loop,
      recursive checkpoints, composition matrix
- [x] Scheduler — Bernstein waves, critical path, four verdicts at the merge,
      speculation pricing, plan fidelity, parallel profile
- [x] Benchmark service — admission governance, tier-1 resim (4 ops), CFX campaigns,
      paired delta, eligibility + aggregation, HTTP surface
- [x] Laboratory — factor registry, workloads with planted truth, live ablation ladder,
      2^k factorial interactions, drift store

**Next:**

- [ ] **#10 — Supervisor API + notification engine** — findings inbox over
      `holds.list`/`decide`, policy-routed notifications (per-tick/per-turn/digest,
      severity thresholds, scope filters), every notification + response traced
- [ ] **#11 — Supervisor web app** — findings inbox, session timeline with branch
      visualization, tick inspector, capture-envelope honesty panel, pre-continue brief
- [ ] **#12 — Live benchmark dashboard** — websocket streaming of ticks/verdicts/AITD/
      cost, cross-harness comparison, population distributions
- [ ] **#13 — Browser extension adapter** — observer transport, file-interception grant
      (the byte-exact passage capture), conversation-surgery mapping
- [ ] **#14 — Mobile supervisor app** — push, approve/deny cards, HALT resolution
- [ ] **#15 — Native harness (D2)** — the maximum capture envelope and calibration
      reference; scheduler-backed execution; native spans
- [ ] **#16 — Onboarding wizard + docs site** — record → review passage → simulate →
      compare; the marketing site *is* the product flow
- [ ] Interval checkpoint manifests (makes the replay factor real — §59)
- [ ] OMEX↔Hearth join interface — response-graph node ↔ tick node cross-referencing
- [ ] Out-of-band observation (filesystem/process-level) — the telemetry-hardening rung
- [ ] Signed telemetry + attestation
- [ ] Blender / 3D adapter — proves the graph is genuinely host-agnostic
- [ ] Reference file/shell adapters for CI-style benchmarking
- [ ] Pause-less host shadow runners
- [ ] Flash-loop community registry + bundle registry
- [ ] Statistical inference for the lab (repetitions with variance)
- [ ] Parallel-profile live dispatch (real executor behind the seam)

---

## 33. FAQ

**Is this another agent framework?**
No. It is an enforcement kernel — the control layer frameworks lack. Run it native, or
wrap yours — it works either way, which is the point.

**Is it a benchmark or a harness?**
Yes. Measurement and enforcement are the same control path with different release
authority. Shadow = benchmark. Gating = meta-harness. Escalation = supervision. One
engine; the deployment chooses the authority.

**Is Hearth itself a control factor?**
No — and this matters. Hearth is the measurement/enforcement substrate for the
control-factor space. It measures and enforces factors; it is not one axis among them
(Invariant #20).

**What is a control factor?**
An independently addressable control behavior that can be present, absent, weakened,
strengthened, or enforced, produces measurable intervention, and is fully traced.

**Does Hearth replace my harness's context model?**
No. The session owns its graph. Hearth supplies the control semantics *around* that
graph and never imposes an ontology on it.

**My harness has a better traversal algorithm — can it compete?**
Register it. Traversal strategies are first-class registrable objects. You're not
measured on using Hearth's navigation; you're measured on what your navigation achieves.

**What is the difference between temporal and context traversal?**
Temporal is the execution timeline; context is the dependency space — which prior state
is causally or contractually relevant to the current tick. Both readings live on the
same canonical graph.

**What is "missed context"?**
Context that should have been traversed but wasn't — the negative space of the
traversal record. Always reported as a lower bound, with the evidence basis recorded.

**Why do defaults matter if everything is pluggable?**
Because adaptability is only measurable against a fixed reference. Without the baseline,
"adaptable" is unfalsifiable.

**What does "unbiased aggregation" mean here?**
The aggregation population is defined by explicit, versioned eligibility requirements —
never by convenience. Every qualifying system's full trace is retained; the aggregate is
a behavioral distribution over equivalent conditions.

**Can one recorded run really benchmark many control policies?**
Tier-1 counterfactuals, yes — they're pure functions of the capture. Asking what a
*different model* would have done is tier 3 — live execution only, and Hearth says so.

**What does "cost to correction" include?**
Detection, synchronization, rollback, replay, re-execution, and downstream invalidation.
Cost depends on *when* you detect the problem.

**Why nine depths instead of one number?**
Because collapsing them destroys the measurement. The difference between harnesses
lives in the vector's shape.

**What is a tick?**
The atomic unit. Every prompt, tool call, or state change is one tick, running the fixed
eight-step pipeline. One event → one pipeline run → one tick record.

**What if my harness can't pause?**
Declare its real capabilities. It gets AUDIT — the identical pipeline in shadow. No host
is worth zero enforcement.

**What is the Hearth Delta?**
The measured difference between a harness's shadow (unassisted) success and its enforced
(controlled) success — decomposable into per-factor deltas and interaction effects.

**Who executes a repair?**
The loop that owns the failing artifact. Hearth schedules it and re-verifies.

**What happens when two loops disagree on the same tick?**
Worst verdict wins; competing `BACKTRACK` depths resolve to `dmax`; dissents recorded.

**What is drift, and is it a separate system?**
Not a separate system. Detection, attribution, correction, measurement, and recurrence
tracking all reuse the tick → finding → verdict → repair → trace machinery.

**Who is the supervisor — human or AI?**
Whatever the deployment makes it: human, AI, hybrid, or policy. Each is benchmarkable.

**Can I use the browser extension with ChatGPT/Claude/Grok?**
That is the plan: the browser client is a first-class adapter. The benchmark is
harness-independent; each public agent is one adapter.

**What are the three transports?**
Hook (synchronous gate at action boundaries — CC hooks), Pump (spawn/attach + event
stream + control messages — Codex app-server), Observer (watch a surface you can't
control — browser clients, shadow only). Every host is one of these.

**Why is the capability probe needed?**
Because declared capabilities are claims. The probe verifies them against behavior at
registration — a host claiming `gate: true` that executes anyway is caught before it
poisons a benchmark.

**What is the five-element atom?**
One contract, one loop, four verdicts, one checkpoint, one tick — the smallest thing
that is a harness in the full sense. `verify_atom()` checks it at registration. Below
it there is no control, only behavior.

**How does harness-of-harnesses work?**
The child registers into the vest as a loop module (or runs as a span); its boundaries
are ticks on the same substrate; `composition.span` enforces handoff sync, orphans, and
conflicts; verdicts propagate with worst-wins-`dmax` and a +1 boundary cost for
crossing spans. Nothing new is required at any depth — the same atom composes.

**What does the scheduler actually schedule?**
Waves of tasks over the movement graph's dependency structure. Bernstein conditions on
declared read/write sets are the serialization requirements; the frontier between waves
is the parallel opportunity. The bound — work ÷ critical-path — is computed from the
graph itself.

**What is "plan fidelity"?**
Declared vs. observed dependency edges. A task writing outside its declared set is a
finding and its work is discarded — the plan is a contract.

**Is the lab's host a real model?**
No — the lab's arms are scripted hosts running real kernels, which is the correct
controlled design for factor effects. Real-model arms come from running the same
workloads through the connectors.

**Why the name?**
A hearth is the heart of a structure — everything else gets built around it.
Enforcement-over-state is the organ other harnesses are missing.

**What does it measure?**
Whatever it enforces. Ticks per level, the nine-depth vector, blast radius, active
sets, traversal coverage, its own overhead — plus violation rates, control delta,
factor deltas, interactions, cost to correction, missed context, counterfactuals,
supervision demand, drift, cross-boundary movement, and the parallel profile. The
instrument and the enforcer are the same object.

---

## 34. Glossary

| Term | Definition |
|---|---|
| **Control-Loop Harness** | master rolling-horizon loop + registry of sub loops; makes stored state enforceable |
| **Enforcement kernel** | the frozen core: binds obligations to loops, evaluates at host-event boundaries, gates through four verdicts, records the canonical trace |
| **Tick** | the atomic unit — one host event run through the fixed eight-step pipeline; an iteration is one tick |
| **Event identity / iteration identity** | every event is exactly one tick; a host turn may span many ticks — metrics count ticks, never host turns |
| **Flash loop** | swappable control-loop module; the executable embodiment of one or more control factors |
| **The vest** | the loop registry — a kit, not a strap; a routing table of scope keys |
| **Bundle** | a named, portable set of control loops, attachable to any registered harness unchanged |
| **Scope key** | the address of a loop's jurisdiction — `harness:`, `modality:`, `tool:`, `scenario:` |
| **Serialization level** | a loop's nesting depth in the registry, declared at registration — fixed for the run; what AITD indexes |
| **Traversal depth** | path length in the canonical traversal graph, including revisits |
| **Temporal traversal** | the execution-timeline reading of the graph |
| **Context traversal** | the dependency/relationship reading — which prior state is causally or contractually relevant |
| **Impact subgraph** | the reachable affected state under applicable relationship semantics |
| **Depth vector** | the nine depths — reported per run, never collapsed |
| **Replay depth (R)** | how many prior checkpoints each step re-verifies; dynamic; the bounded default |
| **Forward horizon (H)** | how far ahead each step plans |
| **AITD** | Agentic Iterative Traversal Depth — ticks per serialization level, p50/p95 distributions |
| **Blast radius** | downstream decisions invalidated by a correction — computed as reachable affected state |
| **Correction depth** | how far backward a correction propagated |
| **Checkpoint** | id + trace position + contract versions in force + snapshot ref (or compensating-action descriptor) |
| **Verdict** | `CONTINUE` / `REPAIR` / `BACKTRACK(d)` / `HALT` |
| **Worst verdict wins** | per-tick aggregation — `HALT > BACKTRACK(dmax) > REPAIR > CONTINUE`; dissents recorded |
| **Backtrack phases** | rewind state / repair consequences / replay decisions — declared separately |
| **Release authority** | what the deployment lets Hearth do with a verdict: trace it, file it, gate it, escalate it |
| **Control path** | OBSERVE · AUDIT(SHADOW \| ACTIVE) · ENFORCE(AUTO \| SUPERVISED) — one pipeline, five postures |
| **Shadow audit** | evaluate without affecting execution — the raw-capability benchmark instrument |
| **Hearth Delta** | enforced success − shadow success; the measured contribution of external control |
| **Correction rate** | proportion of behavior Hearth corrected — distinct from control delta |
| **Control profile** | the behavioral fingerprint: violation rates, repair/backtrack/escalation rates, persistent drift, recovery, overhead |
| **Control factor** | an independently addressable control behavior — ablatable, intervention-measurable, traced |
| **Control-factor space** | the full factor × level × posture × configuration space; the benchmark's factorial substrate |
| **Substrate** | Hearth's role — it measures and enforces factors; it is not one of them |
| **Ablation ladder** | ABSENT / WEAKENED / PRESENT / STRENGTHENED / ENFORCED — the five test states |
| **Factor delta** | the shadow→enforced success delta attributed to a single factor |
| **Interaction effect** | the measured joint deviation of two factors from the sum of their individual deltas |
| **Factorial dataset** | the benchmark output — every observation a point in the factor space, queryable rather than ranked |
| **Session** | the unit of graph ownership — one harness's one run |
| **Session graph** | the session's world model — built by the session; Hearth supplies control semantics around it |
| **Canonical evidence substrate** | what the store says — states, checkpoints, relationships, impacts, controls, verifications per tick — never "how the harness traversed" |
| **Traversal strategy** | a registered, versioned, scope-addressable navigation algorithm; pluggable, benchmarkable, canonical in results |
| **Traversal API** | the capability boundary exposing traversal primitives |
| **Context API** | the boundary for backward/forward sync, relationship/impact queries |
| **Traversal record** | the per-tick trace of nodes visited, edges followed, backtracks, forward syncs, strategy used |
| **Missed context** | context that should have been traversed but wasn't — always a lower bound |
| **Verification efficiency** | what was safely skipped vs unnecessarily rechecked |
| **Checkpoint effectiveness** | how much historical synchronization a checkpoint enabled |
| **Context depth** | how deep in the relationship graph relevant prior state was found |
| **Back-simulation** | counterfactual replay of a captured trace under a different control policy |
| **Counterfactual tier** | declared validity class: state-bound / checkpoint re-execution / world-generation |
| **Cost to correction** | detection + synchronization + rollback + replay + re-execution + downstream invalidation |
| **Detection depth** | how long a defect survived before detection |
| **Economic value of verification** | the measured downstream cost avoided by verification at a given depth |
| **Default baseline** | the canonical configuration — the reference population |
| **Eligibility gate** | explicit, versioned requirements for entering the aggregation population |
| **Unbiased aggregation** | population-level aggregation over eligible systems only; full traces retained |
| **Behavioral distribution** | the aggregate output — how all qualifying systems behave under equivalent conditions |
| **Adapter capability** | declared host power — `true`/`partial`/`false`, shipped with every trace |
| **Non-rewrite guarantee** | Hearth gates the release of ticks; it never edits them |
| **Canonical traversal graph** | software-agnostic record of every traversal event; the substrate for everything |
| **Contract** | a stored obligation bound to a loop — structurally incapable of being unbound; always versioned |
| **Drift** | change in control behavior over releases — detect, attribute, correct, measure, track recurrence |
| **Persistent drift** | a finding that recurs after repair across runs/releases |
| **Supervision demand** | how much external control a system requires to remain correct |
| **Supervisor** | the control plane above the kernel — human, AI, hybrid, or policy; every action a traced tick |
| **Transport** | how a connector attaches: hook (sync gate), pump (event stream + control messages), observer (shadow watch) |
| **Gate point** | the moment a decision can block execution — `tool.proposed` / approval requests |
| **Capability probe** | registration-time verification that declared capabilities match observed behavior |
| **Fail mode** | `open` (allow + spool on daemon loss) or `closed` (hold) — declared per adapter |
| **Conversation surgery** | edit/rewind/branch as host-native operations requested by the connector; the original never overwritten — the graph gains a branch |
| **Span** | a sub-harness's execution window — declared (`span` envelope field) or positionally attributed |
| **Span path** | the chain of span ids from root to the span — the nesting address |
| **Boundary cost** | the +1 added to `BACKTRACK(d)` when the backtrack crosses a span boundary |
| **composition.span** | the boundary loop — handoff sync, orphaned output, cross-span conflicts, cross-level verdict propagation |
| **The atom** | five elements: one contract, one loop, four verdicts, one checkpoint, one tick — the smallest composable harness |
| **verify_atom** | registration-time check of the five elements |
| **Nested graph** | the session graph with child graphs as first-class subgraphs — one substrate at every depth |
| **Recursive checkpoint** | a checkpoint tree spanning levels; backtrack plans map rewind/repair/replay onto levels |
| **Bernstein conditions** | the independence test: W∩W always conflicts; R∩W conflicts unless the reader declares `stale_ok` |
| **Wave** | a set of mutually non-conflicting tasks dispatched together — legal concurrency, not wall-clock |
| **Critical path** | the longest dependent chain by cost — the makespan floor (Amdahl's serial term, read off the graph) |
| **Speculation** | optimistic execution over a dependency edge; committed if the target commits, discarded and priced otherwise |
| **Plan fidelity** | declared vs. observed dependency edges — precision and coverage; gaps are planning errors surfaced as data |
| **Parallel profile** | the tenth metric family: makespans, bound/realized speedup, available/realized width, utilization, waste, fidelity |
| **Passage** | a derived, frozen, user-reviewed object of session inputs — the CFX substrate |
| **Fidelity class** | byte-exact / referential / rendered / opaque — the honesty axis of passage items |
| **Reintroduced_at** | the seq positions where the user re-supplied something already given — the ground-truth label of context-machinery failure |
| **Ingestion map** | the declared, versioned delivery of each passage item to each target harness — a confound shipped with results |
| **Delivery receipt** | what arrived, at what fidelity, and what could NOT be delivered |
| **failed-to-consume** | the outcome class for harnesses that cannot consume the passage — a benchmark result, never an orchestration error |
| **CFX** | Context-Fixed re-execution — tier-2 simulation: passage held constant, machinery native, divergence is the measurement |
| **Dual-review ledger** | per-tick attribution of findings to the native stream, the Hearth stream, both, or neither |
| **meta_effectiveness** | how much the harness's own monitoring caught vs. what external loops caught |
| **Black-box doctrine** | measure the *behavior* of control, not the *mechanism* — control capacity is externally visible through movement shadows |
| **Envelope class** | observable / opaque / native — whether the harness's context machinery is measurable and how strongly claims are typed |
| **user_effort_to_correct** | edits × correction depth + reintroduced items × size + regenerates — context failure priced in user effort |
| **Movement graph** | the observed counterpart of declared graphs: cycles preserved, entities as edges, boundaries as nodes, verdicts attached |
| **L-obj / L-self-meta / L-ext-meta** | object loops / self-monitoring / external meta-monitoring — the three levels; the third is the gap Hearth occupies |
| **Governance plane** | schemas, invariants, probes, tiers, eligibility, consent — the rules all layers obey; not another watcher |
| **Four planes** | substrate → object → observation → governance — zero regress |
| **Control-shape verification** | checking process discipline (sync-before-build, verify-before-irreversible) — mechanical predicates, flat cost in capability |
| **Shape ≠ intent** | the honest boundary: disciplined-misaligned is invisible at shape level; hence honeypots, sampled content audit, OMEX provenance |
| **Observation ladder** | schemas → probes → cross-validation → out-of-band observation → attestation — the telemetry-hardening rungs |
| **Scale factors** | S1–S14: the parallelism dimensions a native graph-scheduled harness unlocks |
| **Parallel efficiency** | realized speedup ÷ bound speedup — how much of the graph's width a harness actually exploits |
| **width left on the table** | bound speedup − realized speedup — the next control metric |
| **Loop precision** | TP/FP/FN of a loop against planted truth — the number the industry cannot currently report about its own internal loops |
| **Synthetic-reference calibration** | precision measured against generated sessions with tagged violations; the bootstrap for loop honesty |

---

## 35. License

MIT

---

---

# PART II — THE REFERENCE IMPLEMENTATION

## 36. Implementation status — nine artifacts

What exists is not a design document with ambitions. It is a linked, selftested
implementation where every layer validates against the same frozen schema registry.

| # | Artifact | Contents | Selftest |
|---|---|---|---|
| 1 | **Wire contract** | 8 JSON schemas (hearth, decision, capabilities, session_bundle, tick, envelope, passage, session-graph, plan) + golden vectors + validation recipes (ajv / jsonschema). Strict payloads, closed `kind` enum, cross-field `if/then` rules (hold ⟺ gating action, ENFORCE ⇒ gate+resume, byte-exact ⇒ content_ref). Non-rewrite *encoded, not implied*: `message.edited` requires `pre_image_ref`; checkpoints require snapshot ref *or* compensating-action ref | vectors pass/fail as designed |
| 2 | **Claude Code connector** | daemon (UDS, spool, idempotency, two-pass validator, kernel-backed engine), thin hook client (one process per hook call, ~1ms via unix socket), stream-json shadow pump, fork-BACKTRACK with git/blob compensation, `hearth_ctl` CLI, hook installer with backup | 13 checks — gate blocks `rm -rf`, read-before-write, audit-vs-enforce divergence, checkpoint + compensation rewind restores file state, idempotent re-ingest |
| 3 | **Codex connector** | app-server JSON-RPC gate client (approval requests = `tool.proposed`; the approval response IS the gate), exec `--json` shadow pump, shared wire→canonical mapping (two wire generations), fake app-server double for selftest, steer-after-deny (fire-and-forget from the reader thread — the deadlock the design review caught), `interruptConversation` as halt | 12 checks — deny/approve/approve + steer traced, decisions attributed to `tool.gate` |
| 4 | **Probe suite** | observe / gate / resume / inject / compensate / supervise against both connectors; grading floors (`true→pass`, `partial→partial`, false not probed); mismatch = registration REFUSED; `--force` corrects the declaration down to observed, records provenance under `ext.probe_corrections`, re-derives posture (a connector-level-only gate honestly lands as AUDIT_SHADOW, not ENFORCE_SUPERVISED); `--with-host` upgrades to host-level verification (real non-execution of denied commands) | 16 checks incl. the honesty machinery |
| 5 | **Kernel extraction** | `hearth_kernel/` — LoopRegistry (levels from declared nesting, composes checked pairwise both directions at registration), the fixed 8-step pipeline, **tick records** (kernel-authored, schema-validated, `tick_id`/`depths`/`fired`/`traversal`/`sync`), worst-verdict + `dmax` + dissent recording, four builtin loops (tool.gate, tool.fs.order, verification.replay, context.sync), GraphIndex + four traversal strategies, session-graph builder, depth vector + AITD. Decisions locked: one event → one tick always; kernel BACKTRACK is **measurement-first** (`auto_backtrack: false` downgrades to forward-repair, honestly noted); honest levels (builtins flat at L1 until something nests) | 33 checks |
| 6 | **Metrics suite** | control profile (violation/correction/escalation rates, unverified writes, **black-box attribution mode** per envelope), `meta_effectiveness` (the dual-review ledger), cost-to-correction decomposition + latency curve, missed-context analyzer (lower-bound, evidence basis), cross-boundary family (handoff sync, orphans, conflicts, user-correction layer), loop precision + synthetic-reference calibration, capture-envelope registry (4 hosts: CC, codex verified; native, browser-chat declared) with `context_machinery: observable|opaque|native` | 32 checks |
| 7 | **Recursive composition** | span attribution (declared wins, positional fallback), NestedGraph (per-span subgraphs, spawn/contains/access/checkpoint edges, rollup), `verify_atom`, `composition.span` loop (boundary stale-writes, orphaned output, cross-span conflicts, cross-level HALT/BACKTRACK propagation with +1 boundary cost), RecursiveCheckpointer (cross-level backtrack plans with rewind-at-depth/repair-at-boundary/replay-at-re-entry phase mapping), composition matrix (orchestrator × sub-harness, admitted with `boundary_type`/`sub_harness_count`/`nesting_depth` dimensions) | 37 checks |
| 7b | **Scheduler** | plan schema (tasks with declared read/write sets, deps, speculative + `spec_over`, `stale_ok`, exclusive/barrier), Bernstein condition edges, cycle rejection, critical path, the wave machine (max_width, exclusive solo waves, barriers), the four verdicts at the merge (REPAIR requeue, BACKTRACK rollback-priced, HALT machine-stop, write_exceeded discard), speculation resolution and waste pricing, plan fidelity (declared vs observed edges), the parallel profile | 40 checks |
| 8 | **Benchmark service** | admission as the governance boundary (schema → capabilities+verified block → consent-verified redaction → graph closure; refusal at each stage with precise reason; admission receipt records validator regime), immutable bundle store + sqlite index, tier-1 resim (ablation with per-factor attribution, gate_policy re-flooring, R-sweep from the capture alone, strategy sweep), paired Hearth Delta + factor deltas, CFX campaigns (passage validation, ingestion maps, delivery receipts, `failed-to-consume`, completed runs admitted), eligibility profiles + multi-dimensional distributions, stdlib HTTP surface | 44 checks |
| 9 | **Laboratory** | factor registry (9 factors, versioned, `arm_config` with dedup by canonical config), workload format (declarative scripts with planted, tagged violations + conditional consequence subtrees), ArmRunner (adaptive scripted host through real kernels — **live gating**: in enforce arms, prevented plants' consequences never execute, so ABSENT/ENFORCED arms diverge in real downstream state), live ablation ladder, 2^k factorial designs with main effects + interaction contrasts, DriftStore (release series, deltas, persistence/recurrence), experiment metadata in bundles (`ext.benchmark.factors`) and as an aggregation dimension | 33 checks |

All suites green simultaneously; every artifact's events, decisions, capabilities, ticks,
graphs, plans, and bundles validate against the same registry. The dogfooding loop
closes: Hearth's own development — selftests as specifications, schemas as contracts,
honest limitations in every artifact — is the code-quality discipline the project
preaches, and it is load-bearing for the data's credibility.

---

## 37. The connector layer

### Claude Code — surfaces, in order of power

1. **Hooks** (`settings.json`) — the synchronous gate channel. `PreToolUse` fires with
   `tool_name` + `tool_input` on stdin **before execution** and can deny. `PostToolUse`
   observes results and feeds reason back (that's `inject`, opt-in). `UserPromptSubmit`,
   `Stop`/`SubagentStop` (turn boundaries), `PreCompact` → `compaction.*`,
   `SessionStart/End`.
2. **Headless stream** — `claude -p --output-format stream-json --verbose` (observe);
   `--resume` / `--continue` / `--fork-session` (rewind mechanics).
3. **Agent SDK** — in-process `canUseTool` + hooks + forking. Strongest: native posture,
   no process boundary.
4. **Transcript files on disk** — backfill/shadow harvest of ungated sessions.
5. **MCP** — in-band suggestions only; weakest; noted, not relied on.

Hook invocations are fresh processes — so the hook binary is a **thin client** over a
persistent daemon on a unix socket (~1ms), never the kernel itself. Decision mapping:
CONTINUE → exit 0 silent; REPAIR → `permissionDecision: deny` + reason (model
self-corrects in place); supervised hold → poll daemon, timeout degrades to native
"ask" (declared degradation, not silent failure); BACKTRACK → deny now, fork +
compensation out-of-band; HALT → deny + session halted.

Resulting declaration: observe/intercept/gate/resume/inject/supervise true; snapshot
partial; compensate partial; fail_mode closed → ENFORCE_SUPERVISED.

### Codex — surfaces

1. **`codex exec --json`** — JSONL event stream (observe → shadow audit).
2. **App-server / proto mode** — bidirectional JSON-RPC over stdio: **approval requests
   are the gate point** (`execCommandApproval`, `applyPatchApproval`);
   `interruptConversation` is halt; mid-run messages are steer.
3. **`config.toml`** — `approval_policy`/`sandbox_mode` shape which actions are
   gateable (the `gate: partial` is a real finding: the benchmark records *which* action
   classes were gateable).
4. **Sessions dir** — full rollouts incl. reasoning → post-hoc shadow harvest.
5. **`codex exec resume`** — continuation, not prefix-truncation: rewind is
   fork-flavored, declared, not faked (this connector does *not* emit
   `conversation.rewound` — it hasn't earned it).

The reader-thread constraint, stated because it's structural: the approval request
arrives on the reader thread, so anything sent mid-approval (steer, interrupt) must be
fire-and-forget (`request_async`), never blocking RPC — the reader would deadlock on a
response only the reader can deliver.

### The information surface per host

| Signal | Claude Code | Codex | Browser |
|---|---|---|---|
| Prompt verbatim | ✓ | ✓ | ✓ (pre-send) |
| Model text | ✓ | ✓ | ✓ |
| Reasoning | summaries | ✓ | ✗ |
| Tool name + args | ✓ | ✓ | cards only → partial |
| Tool results | ✓ | ✓ | partial |
| File diffs | ✓ | ✓ (`file_change`) | artifacts only |
| Token usage | ✓ | ✓ | ✗ |
| Compaction | ✓ (`PreCompact`) | internal | ✗ |
| Subagents | ✓ (`SubagentStop`) | thread items | ✗ |
| Approvals | permission events | approval requests | ✗ |

This table *is* the per-host capture envelope — and it is why `missed_context` is always
a lower bound: the capture has edges only where the host exposed them.

### Conversation surgery — the non-rewrite guarantee, mechanically

Hearth never mutates content. It gates releases and requests host-native operations. An
edit by user or connector is a *new host event* whose payload references the pre-image
by hash; the graph gains a branch. `BACKTRACK(d)` consults the host's rewind capability:
native rewind exists → drive it → `conversation.rewound` with trigger = decision id;
compensation exists → `compensation.executed` + forward-repair; neither → degrades to
forward-REPAIR, recorded. Metrics go branch-aware for free: traversal depth counts the
active branch's path; branch count and cross-branch re-verification are separate
recorded quantities.

### Benchmark ingestion — the other side of the connector

```bash
hearth submit --run R001 --workload swe-bench-lite --model m --harness cc --redact-payloads
hearth resim --run R001 --policy R=7 --policy strategy=default.hybrid --tier state-bound
```

Server: `POST /v1/traces` (schema-validated; capability declaration declared+verified
required), `POST /v1/traces/{id}/resim` (tier discipline enforced server-side), `GET
/v1/benchmarks/{workload}/distributions`, `POST /v1/eligibility`. Every returned
counterfactual carries its tier field.

---

## 38. The kernel & runtime

### Kernel extraction decisions (locked)

- **One event → one tick record, always.** Duplicates short-circuit before the pipeline.
- **Tick records are kernel-authored and schema-validated** — `tick.schema.json` joined
  the registry as its own member; connectors never author ticks.
- **Serialization levels are honest.** Builtins register flat at L1 because nothing sits
  above them; the selftest proves L2 lands the moment a modality loop registers with a
  parent. `max_depth: 1` on connector-scale runs is the truth, not the §8 illustration.
- **Kernel BACKTRACK is measurement-first.** The depth is always recorded; execution is
  policy-gated (`auto_backtrack: false` downgrades to forward-repair, with the
  downgrade itself noted in the decision). No selftest setup ever silently rewrites a
  workspace.
- **Split of labor:** kernel owns control semantics (resolution, traversal, loop steps,
  aggregation, sync, tick records); the daemon owns policy (severity floors, shadow,
  holds, rewind execution, gate mapping). Host mechanics never leak into the kernel;
  loop logic never leaks into the daemon.

### The builtin loops

| Loop | Enforces | Mechanics |
|---|---|---|
| `tool.gate` | irreversibility | classifier over commands (rm -rf, force-push, DROP TABLE, mkfs, dd to device, pipe-to-shell, …) → finding + REPAIR hint; severity from the pattern table |
| `tool.fs.order` | read-before-write | tracks reads/writes per absolute path; write to existing never-read file → medium finding (brand-new files exempt) |
| `verification.replay` | stale state | checkpoint manifests → hash-verify tracked files; evolved-under-tracked-writes exempt; **standard tier verifies newest checkpoint, deep tier up to R_high** (hot flag or post-rewind); `replay_used` reports what was ACTUALLY examined — the metric never inflates the window; deep-tier hits with dependents escalate to BACKTRACK |
| `context.sync` | forward sync | kernel step 6 marks impact-subgraph entities stale; reading re-syncs; writing a stale entity → high finding ("consequences propagated, then acted on blind") |

### The tick record

```json
{
  "v": 1, "tick_id": "tk_…", "run_id": "R001", "seq": 4172,
  "event_id": "evt_…", "kind": "tool.proposed", "session_ref": {…},
  "turn_id": "t_03", "span": {"id": "dc1", "depth": 1, "path": ["root","dc1"]},
  "resolved": ["tool.gate", "tool.fs.order", "verification.replay", "context.sync"],
  "fired": [{"loop": "tool.gate", "level": 1, "tier": "standard",
             "verdict_hint": "REPAIR", "findings": [...], "notes": {...}}],
  "traversal": {"strategy": "default.hybrid", "strategy_version": "0.1.0",
                "nodes_visited": 14, "edges_followed": 24,
                "checkpoints_selected": ["cp_…"], "context_depth": 2,
                "cost_estimate": 38},
  "verdict": "REPAIR", "backtrack_d": null, "decision_id": "dec_…",
  "depths": {"traversal": 4172, "serialization": 1, "replay_used": 1,
             "correction": null, "backtrack": null, "dependency": 2,
             "context": 2, "synchronization": 12, "verification": 3},
  "sync": {"backward": "confirmed", "forward": "confirmed"},
  "probe": false, "emitted_us": 1737…
}
```

---

## 39. Recursive composition — the fractal substrate

The harness-of-harnesses property, made structural. "Harness" is not a privileged
boundary — it is a role anything plays by satisfying five elements, and composition
preserves the interface: a parent sees its child as exactly contract, loop-set, verdicts
in, checkpoints exposed, ticks emitted.

### The five-element atom

| Element | What it is |
|---|---|
| One contract | a declared obligation (`contract_ref`) |
| One loop | a checker over it |
| Four verdicts | the closed protocol |
| One checkpoint | one addressable return point (or compensating action) |
| One tick | one record of what happened |

`verify_atom(module)` checks all five at registration. A missing element is a violation,
not a warning: **below the atom there is no control — only behavior.** And because
every level above the atom is the same five elements composed, there is no special
"ASI governance mechanism" waiting at the top — only more composition of an atom small
enough to verify by inspection.

### Spans — boundaries as ticks

Sub-harness execution windows are first-class structure:

- **Declared wins:** the envelope's optional `span` field (`id`, `depth`,
  `parent_span`, `path`, `contract_ref`) gives exact nesting for hosts that can declare
  it — native Hearth does.
- **Positional fallback:** `subagent.started` belongs to the **parent** (spawning is the
  parent's act); events until `completed` belong to the **child**; `completed` belongs
  to the child (its report). Malformed boundaries are warnings-as-data, never crashes.
- Tick records carry the span; the graph builder partitions events per span into child
  subgraphs.

### The nested graph

`build_nested_graph(events, ticks)` produces one substrate at every depth:

- **Nodes:** `span`, `event`, `entity`, `checkpoint` — each tagged with its `span_id`.
- **Edges:** `spawn` (parent→child), `contains`, `access` (event→entity with action),
  `checkpoint`.
- **Per-span rollup:** worst verdict, dmax, findings, ticks.
- **`aitd_by_span`** — AITD computed per nesting level.
- Frozen as `session-graph.schema.json` — the seventh registry member.

### `composition.span` — what gets measured becomes enforceable

The #6 boundary *metrics* became a boundary *loop* (the project's standing pattern):

| Factor | Fires when |
|---|---|
| `composition.stale_write` (high) | parent writes an entity a closed child span wrote, without reading it first — **boundary stale-write**: building on unverified handoff |
| `composition.orphan` (medium) | child-written entities never verified by the parent at turn end |
| `composition.conflict` (medium) | two child spans wrote the same entity without reconciliation |
| `composition.handoff` (medium) | child ended in error — residual unverified at the boundary |
| `composition.propagation` (critical/high) | child `aggregate_verdict` HALT → parent HALT; child BACKTRACK → parent BACKTRACK with d+1 |

**Invariant #12 through nesting:** worst verdict wins at the merge; a BACKTRACK crossing
a boundary costs **+1** (the boundary cost). Child work is not blocked while it runs —
only the merge is checked, which is exactly the handoff semantics the connection-gap
analysis demanded.

### Recursive checkpoints

`checkpoint.created` gains `covers_spans` and `child_checkpoint_refs` — a checkpoint
tree spanning levels. `RecursiveCheckpointer.plan_backtrack(d, current_span, spans)`:

- walks the span path, counting checkpoints per level;
- **within level:** no boundary crossed, effective_d = d;
- **crossing levels:** effective_d = d + boundaries crossed (the +1 cost, made
  computable);
- the plan maps the three backtrack phases onto levels: **rewind where state is owned
  (deepest), repair at each boundary crossed, replay from the re-entry level**.

Planning is kernel-side; execution stays host-side — measurement-first, always.

### The composition benchmark

Orchestrator × sub-harness as a first-class experiment: pairs run through real kernels
with `composition.span` active; every pair admits as a bundle carrying
`boundary_type`, `sub_harness_count`, `nesting_depth`. The selftest matrix shows the
signature result: a **verify-orchestrator** (reads child output before building) has
zero boundary stale-writes; a **blind-orchestrator** gets caught — the trust-hole
between harnesses, measured and gated.

### Known edge, stated

CC's `SubagentStop`-only boundary means connector-scale children are terminal stubs (no
window). The stack degrades safely — unmatched `completed` events are root-attributed
with a warning. Exact nesting requires hosts that declare spans or emit both
boundaries: the native path does, future connectors can.

---

## 40. The scheduler — the width exploiter

Sequentiality was never a law of harnesses. It was the shadow of missing infrastructure
— and the stack supplies all four missing pieces:

| Why loops were sequential | The missing capability | Where the stack provides it |
|---|---|---|
| Can't know what's independent | an independence map | dependency edges — Bernstein conditions |
| Can't merge results safely | boundary semantics | spans + contracts + `composition.span` |
| Can't verify without blocking | addressable state to verify against | checkpoints + replay-as-region-query |
| Can't branch without risk | clean rollback | per-span checkpoints, compensating actions |

### The plan — the checklist as a DAG

`plan.schema.json` (eighth registry member): tasks with **declared read/write sets**,
`deps`, `speculative` + `spec_over`, `stale_ok` (the declared WAR relaxation — a reader
tolerating unverified state may run beside writers, priced and traced), `mode:
parallel|exclusive`, `kind: work|verify|barrier` (verify tasks are read-only — they
wave naturally beside everything they don't conflict with, which is S4 falling out of
Bernstein rather than being special machinery).

### The wave machine

`dependency_edges` applies Bernstein conditions over declared sets (W∩W always
conflicts; R∩W conflicts unless the reader declares `stale_ok`; speculation edges are
removed from the schedule graph and enforced by resolution rules instead); cycles are
PlanErrors. `critical_path` computes the longest chain by cost — **Amdahl's serial
term, read off the graph**. The Scheduler then waves to completion: frontier → pack
(max_width, exclusive solo, barrier breaks) → dispatch → reconcile.

### The four verdicts at the merge

- **REPAIR** → requeue in place (retry budget; work not discarded, no waste priced).
- **BACKTRACK** → rollback: work discarded, **ticks priced as waste**, requeue;
  exhaustion → failed.
- **HALT** → machine halts; pending tasks skipped with `plan.halt_aborted` findings.
- **Contract violations:** observed write set ⊃ declared set → `plan.write_exceeded`
  (high), work discarded, requeued; same-wave conflicts on *observed* sets →
  `plan.runtime_conflict`. **The plan is a contract** — Invariant #31.

### Speculation

`spec_over` drops the dependency edge for scheduling; a speculative task co-dispatches
with its in-frontier target. Resolution at finalize: target commits → optimistic work
stands; target fails → rolled back, `plan.speculation_lost`, waste priced. Optimism
with priced rollback — the declared-posture pattern again.

### Plan fidelity

Declared vs observed dependency edges — **precision** (declared edges that materialized)
and **coverage** (observed edges that were declared). Gaps mean the plan was wrong, not
that the graph is: planning errors surfaced as data. The selftest demo: an undeclared
read of another task's output shows up as coverage 0.5.

### The parallel profile — the tenth metric family

Never collapsed (Invariant #17/#32):

```text
work_est · critical_path · critical_path_tasks · bound_speedup (work ÷ cp)
sequential_makespan · scheduled_makespan · realized_speedup
waves · available_width · mean_width · width_utilization
width_left_on_table (bound − realized)
speculation {committed, wasted} · waste_ticks · requeues · rolled_back
serial_fraction · plan_fidelity {declared, observed, precision, coverage}
findings · warnings · status · per-task outcomes
```

Scheduling decisions are themselves canonical events on the plan entity (Invariant #33)
— the scheduler is traced like everything else it governs.

**The master scaling law:** `speedup ≤ work ÷ dependency-depth` — and both terms are
already metrics in the depth vector. The scaling limit of a native harness is
computable from the substrate before running it. Sequential harnesses score width ≈ 1
regardless of task width; **parallel efficiency** (realized ÷ bound) and **width left
on the table** are the next control metrics — measuring whether a harness *realizes*
the graph's width.

### Honest bounds on parallelism

True data dependencies (B consuming A's output — the graph exposes the width the task
has, it doesn't invent it); shared write sets (serialize or reconcile — declared
write-sets make conflict *prediction* possible before dispatch); merge barriers
(O(boundary width), not O(system width) — the fractal's entire value); per-atom
inference latency; sync points (the consistency tax, visible as synchronization depth);
speculation waste (recorded, priced, never invisible).

The concurrency is **logical**: waves define legal concurrency; the executor seam
(`dispatch(tasks) → outcomes`) is where a native runtime executes for real. Kernel-pure,
deterministic, host-free.

---

## 41. The metrics suite

### Control profile (with black-box mode)

Computed per session from spools + graph: posture observed (enforced/shadow, inferred
from gating presence), violation rate per tick, ticks-with-findings rate, gated/repair/
backtrack/escalation rates, **unverified writes** (written, never read back — with
entities listed), correction block (shadow self-corrected vs propagated; enforced gated),
supervision demand (holds, requests, resolutions, unresolved), and success when the
workload runner supplies it.

**Black-box attribution mode:** when the envelope declares `attribution_mode:
harness-level`, findings roll up to the harness as a whole (`_rolled_up`) instead of
per-loop — the granularity the envelope licenses, honestly
([§48](#48-the-black-box-doctrine)).

### meta_effectiveness — the dual-review ledger, computed

For shadow runs: each finding's entity is checked for a subsequent READ (the harness
re-syncing = its own machinery catching it). Output: `native_meta_caught`,
`escaped_native_meta`, `meta_effectiveness` (caught ÷ total), self-correction latency
distribution. Declared as a **proxy over observable movement** — "harness read the
finding entity afterward" ≠ "internal eval caught it" — and shadow-only by construction
(gating interferes with the natural recovery signal).

### Cost-to-correction decomposition

Per gating decision and finding: detection seq, introduction seq (approximated from the
last write before detection — declared), **detection latency**, rollback files (linked
through rewind → compensated `file.changed`), replay units (replay_used in the window
after detection), re-executions (fork-linked events), downstream invalidated (dependency
closure). Plus the **latency→cost curve** bucketed by detection lag — the "cost depends
on when you detect" law, as data. Cost units are ticks unless usage events exist; the
two are never mixed.

### Missed context — the negative space

Universe = entities that surfaced in events. Accessed = read at least once. Missed =
dependency-connected to an accessed entity but never read — each with its **evidence**
(the accessed neighbors). Plus `writes_never_read`. Everything carries
`lower_bound: true` and the claim strength typed by the envelope:
*near-exact (retrieval observed)* for observable/native machinery vs *weak lower bound
(retrieval opaque)* for web surfaces.

### Cross-boundary metrics

Subagent spans (start/end from boundary events, or positional windows): written sets per
span, **handoff_sync_rate** (parent read child output after span end), **orphaned
outputs** (written, never touched), **conflicts_unresolved** (overlapping write sets
across spans), **boundary_backtracks** (rewinds whose dropped set intersects a span),
**user_correction_layer** (edits followed by verification within window — the weakest
proxy in the suite, declared weak).

### Loop precision — the number nobody reports

The synthetic-reference calibration corpus: generated sessions with injected, tagged
violations + clean sessions; the kernel runs in-process; findings match injections by
loop and window. Output: per-loop TP/FP/FN/precision/recall/F1. The selftest proves
precision 1.0 / recall 1.0 on the core loops with **zero false positives on clean
sessions** — including one deliberately-planted ambiguous case (`docker system prune`,
medium) that shows up as the honest FP in workload grading. `calibration:
synthetic-reference` is stamped on the result; production precision requires the
native-harness golden set (D2).

### The capture-envelope registry

Four envelopes ship: **claude-code** (verified, observable, per-loop), **codex**
(verified, observable, per-loop), **native** (declared-unverified, native machinery),
**browser-chat** (declared-unverified, opaque, harness-level). Each declares per-signal
fidelity (`byte-exact | derived | reported | summary | boundary | metadata-only | none`),
`context_retrieval.visible_as` (`tool_calls | shell_behavior | none | native`), the
ingestion map, and status. The schema structurally forbids an opaque host from claiming
visible retrieval. `claims(host)` returns what the envelope licenses downstream — the
honesty header every report and UI panel consumes.

---

## 42. The benchmark service

### Admission — the governance plane at the network boundary

A bundle enters the benchmark only through `admit_bundle`:

1. **Schema** — full-schema or structural validation (`require_full_schema` refuses to
   run unvalidated; the receipt records the regime).
2. **Capabilities** — declaration validates; `require_verified_caps` (default) refuses
   bundles without a probe-verified `capabilities.verified` block.
3. **Consent** — if `redact_payloads: true`, inline text/objects are refused: redaction
   is **server-verified**, not CLI best-effort.
4. **Closure** — the artifact-1 invariants the schema cannot express: unique ids,
   strictly increasing seq, resolvable refs (parent/rewind-target/covers/caused-by),
   call linkage (completions → proposals, approvals → requests), one non-probe decision
   per event, rewind triggers referencing real BACKTRACK decisions.
5. **Meta** — host, workload, machinery, envelope status, posture, strategy, validator,
   caps_verified, consent_redacted, n_events → the sqlite index + immutable bundle file.
   Bundle-hash idempotency (duplicate submissions return the same bundle_id).

Refusals are precise and staged: `{"admitted": false, "stage": "closure", "reason":
"closure: dangling parent_event_id …"}`.

### Tier-1 resim — the graph-regional counterfactual engine

Four operation classes, each a **pure function of the admitted capture**, each shipping
`tier: 1` + approximation notes:

- **Ablation** — remove loops (or override their hints) → recompute per-tick
  aggregation. Recorded kernel gates whose resim verdict becomes CONTINUE **would have
  released** — attributed to the removed loops that hinted non-CONTINUE. Executor
  decisions (rewind/halt) are deliberately outside ablation semantics: manual/operator
  actions, not loop policy.
- **Gate policy** — re-apply the severity floor/mode over recorded findings:
  `newly_gated` (lower floor would have caught), `newly_released` (higher floor would
  have let pass).
- **R-sweep** — the §22 curve from the capture alone. Staleness model (declared): an
  entity with tracked writes ≤ checkpoint and **untracked** changes in (cp, tick] would
  have been flagged had the replay window reached it. Curve non-decreasing in R;
  **agreement** = fraction of recorded stale findings recovered at max R (the selftest
  shows 1.0 — the model reproduces the live kernel's detections).
- **Strategy sweep** — per-strategy context shape over the reconstructed graph:
  mean context depth, nodes visited, checkpoints selected per strategy; distinct_shapes
  ≥ 2 proves the strategies are genuinely different machines.

### Paired delta

Two admitted bundles (shadow + enforced) → `hearth_delta` (violation delta, unverified
write delta, success delta **only when the workload runner supplies success values —
the comparator never invents them**) + per-factor deltas (shadow findings vs enforced
findings vs enforced gated, prevented per loop).

### CFX — Context-Fixed re-execution (tier 2)

The campaign: validate the passage → freeze `passage_hash` → for each target:
**deliver** (per the ingestion map — declared, versioned, shipped with results), **run**
(connector attached, Hearth shadow + inject-off, harness native posture ON),
**collect** (canonical session graph). Semantics:

- **Failures are results**: a target that crashes on the passage, times out, or cannot
  take delivery → `outcome: failed-to-consume` with phase (`deliver`/`run`) and reason.
  Nothing dropped, nothing retried-until-pass. A harness whose indexing chokes on the
  1MB CSV *is* the finding.
- **Delivery receipts** record delivered / excluded / degraded per item — the confound
  is declared, never hidden.
- **Passage coverage** — fraction of delivered items the run actually engaged.
- **Completed runs are redacted and admitted** into the same store — CFX enters the
  same aggregation substrate as live runs.
- **Baseline** — the original capture rides along; each target's own fresh replay is
  its nondeterminism baseline (replay-vs-original), so cross-harness deltas are read
  against each harness's own variance floor, never against zero.

### Eligibility + aggregation

Versioned requirement profiles (`profile_v`, named, stored): min_events, capability
floors, envelope status, machinery classes, consent, postures. `check_profile` returns
ok/reasons. `aggregate` produces **distributions over eligible members only** — p50/p95/
mean per metric per group — with `group_by` now including the **factors** dimension
(json of the arm's factor assignments, C3). Every member's full trace retained in the
store regardless of eligibility. `n_bundles` vs `n_eligible` shows the gate working.

### HTTP surface

`GET /v1/health` · `POST /v1/traces` · `POST /v1/traces/{id}/resim` · `POST /v1/delta`
· `POST /v1/cfx` · `POST /v1/eligibility` · `POST /v1/profiles` · `GET
/v1/benchmarks/{workload}/distributions?group_by=…&profile=…` — token auth, JSON
errors, unknown routes 404.

---

## 43. The Context Passage Protocol (CPP)

> **We replay the *passage*, never the *machinery*.**
> The passage is what the user gave: files, text, links, edits, timing, progression.
> The machinery is what each harness did with it: its RAG, its indexing, its compaction,
> its retrieval. Hearth captures the passage exactly, delivers it to every harness
> **untouched**, lets each harness's native machinery consume it its own way, and
> measures the observable shadow of that consumption.

The non-rewrite guarantee extended from ticks to context (Invariant #35): the moment
Hearth's own retrieval touches the context path, the cross-harness comparison is dead —
you'd be benchmarking Hearth's RAG wearing each harness as a costume.

### The passage object

`passage.schema.json` (seventh registry member): `task`, `items[]` (`item_id`, `kind`,
`fidelity`, `content_ref`/`snapshot_ref`, `entered_via`, `at_seq`,
**`reintroduced_at`**), `progression[]` (turn, at_seq, kind: prompt/edit/regenerate/
branch/upload/intervention), `fidelity_summary`, `passage_hash`, consent.

**`fidelity`** — the honesty axis:

| Class | Meaning | Replay delivers |
|---|---|---|
| `byte-exact` | we hold the bytes | the actual file, to any harness |
| `referential` | a link; we hold a snapshot hash | snapshot or re-fetch, drift recorded |
| `rendered` | we saw what the UI showed | the rendering as text/image, marked degraded |
| `opaque` | name + metadata only; contents never captured | a declared hole — countable, not fakeable |

**`reintroduced_at`** — the single most diagnostic field: when the user re-supplies
something they already gave, the original harness *lost it* — compaction evicted it,
retrieval missed it, the model never engaged it. The user's own behavior is the
ground-truth label of where the context machinery failed. Free from the capture;
first-class in comparison.

**`progression`** — edits, branches, regenerates, timing. Not metadata: *signal*. What
the user needed to add later, how far back they had to edit — the cost of the original
harness's context handling, priced in user effort rather than tokens.

### Capture taxonomy — the machinery axis

| Surface | Passage capture | Context machinery | What we see |
|---|---|---|---|
| **Claude Web** | prompt text pre-send; uploads byte-exact if the user grants the file-input hook, else opaque; edits/regenerate/branch via UI affordances | **Opaque** | effects only: engagement, redundant requests, corrections |
| **Codex** | task text; **the repo is the passage** — files byte-exact by construction | **Observable** | retrieval *is* shell behavior: every grep/cat/ls is a tick |
| **Claude Code** | prompt + @-mentions + files | **Observable** | retrieval *is* tool calls: Read/Grep/Glob + compaction boundaries |
| **Native Hearth** | everything, by construction | **Native — ours** | planning state, explicit retrieval decisions, full reasoning |

This distinction types every downstream claim: `missed_context` is *nearly exact* for
agentic harnesses (we watched every read) and a much weaker lower bound for opaque
surfaces — and the envelope says so *in the UI*.

### Measuring the machinery

**Observable class:** `first_access_tick` per item, `passage_coverage`,
`retrieval_precision`, `re_read_rate` (with the changed-between-reads distinction:
verification vs indexing failure), `query_quality`, `compaction_drops` (and whether
evicted content was later needed), `context_growth_curve`.

**Opaque class:** `reference_match_rate` (responses quoting passage blobs — evidence the
machinery engaged), `redundant_request_rate` (asking for what it was given — the
clearest opaque-class failure), `correction_depth` from user edits,
`regenerate_count`, `latency_to_substantive`.

**Both — the headline:** `user_effort_to_correct` = edits × correction_depth +
reintroduced_items × item_size + regenerates. "What would have saved them time," as a
number — computed on the original capture (what the user paid) and on each replay (what
they *would* have paid).

### The scripted-intervention question — locked

When a replayed model stalls, who plays the user? Three valid protocols, never mixed in
one comparison: **(a) no intervention** — unassisted recovery only (default); **(b)
scripted interventions** mirroring the original user's progression (the paired condition
against the original capture); **(c) an AI user-proxy** with its own traced ticks (a
benchmarkable configuration later — the same discipline as human/AI/policy supervisors).

---

## 44. The laboratory

The experimental-design layer (B4–B7, C1, C3) — where the factor space stops being a
dataset and becomes a laboratory. Every arm is a **real run through a real kernel with
real gating**; identical configs dedupe by canonical identity so nothing runs twice.

### The factor registry (C1)

`factors.json` — machine-readable, versioned, addressable like loops:

| Factor | Domain | States |
|---|---|---|
| `tool.gate` | tooling | ABSENT / PRESENT |
| `tool.fs.order` | tooling | ABSENT / PRESENT |
| `verification.replay` | verification | ABSENT / WEAKENED (R_high=1) / PRESENT / STRENGTHENED (R_high=16) |
| `context.sync` | state | ABSENT / PRESENT |
| `composition.span` | composition | ABSENT / PRESENT |
| `gate.floor` | supervision | WEAKENED (critical) / PRESENT (medium) / STRENGTHENED (low) |
| `enforcement` | supervision | SHADOW / ENFORCED |
| `scheduler.width` | traversal | ABSENT (max_width=1) / PRESENT |
| `scheduler.speculation` | traversal | ABSENT / PRESENT |

`arm_config(assignments)` mutates the base config by each factor's declared state
(remove loops, set kernel_cfg, set mode/floor, set scheduler_cfg); `canonical(cfg)`
gives config identity for caching. Unknown factors and invalid states are
registration-time errors.

### Workloads — ground truth rides in the workload

Declarative scenario scripts: `emit` (canonical events), `plant` (a violation with
`id`, `loop`, `entity`, detection `window`, and **`cons`** — a conditional consequence
subtree that only executes if the plant is *not* gated), `fs` (real file ops),
`checkpoint`, `schedule` (embedded plans for the scheduler factors). Plants carry
ground truth in `ext.planted`. One declared-clean risky step (`docker system prune`)
is deliberately ambiguous — it is the workload's honest false positive, and grading
counts it as one.

### Live ablation (B4)

The five-state ladder for real: in **enforce** arms, planted violations above the floor
are prevented and their consequence subtrees never execute — so ABSENT and ENFORCED
arms diverge in **real downstream state**, not just in findings. The selftest's
signature numbers:

- shadow baseline: both plants detected (recall 1.0), the ambiguous step = 1 honest FP,
  downstream damage = 2 (both consequences ran), completion 1.0.
- `tool.gate` ABSENT: recall 0.5, damage **+1** — the consequence ran because the gate
  was gone.
- `enforcement` ENFORCED: both plants prevented, damage 0 — **and** the clean risky step
  gated (`gated_clean_steps` 1, completion 0.0): over-gating is priced, not hidden.
- `gate.floor` WEAKENED (critical): nothing gated, damage stands; PRESENT (medium):
  plants gated.

Grading: detection recall per plant, false positives (findings that match no plant —
including the declared-clean step), downstream damage, prevented count, completion
(required steps lost to gating).

### Factorial interactions (B5)

Full 2^k designs (k ≤ 4) with main effects and 2×2 interaction contrasts per numeric
metric — including per-plant detection indicators, so **epistasis is measured
directly**. The selftest's designed result: on `wl-interact`, replay enables sync —
with both factors present both plants are detected; with replay absent the sync plant
is **masked** (stale was never marked); the interaction contrast on `det.ctx_write` is
exactly **+1.0** (compounding), and the neither-arm has zero findings. Arms cache by
canonical config (4 arms, 2 unique runs).

### Drift store (B7)

Per-release series over a workload: metric deltas vs baseline, per-loop finding
attribution, and **persistence/recurrence** — a loop whose findings recur across
consecutive releases is a structural deficiency, not bad luck: `recurrence_count`,
`longest_consecutive_streak`, `persistent_drift` (streak ≥ threshold). JSON-file
backed, one namespace per workload, record/report from the CLI.

### Experiment metadata everywhere (C3)

Every arm admits as a bundle with `ext.benchmark.factors` carrying the assignment; the
aggregation engine's `group_by` accepts `factors` — the factorial dataset's factor
dimension, live. `save_arm` writes the drift-recordable artifact.

---

## 45. Selftest inventory

| Suite | Command | Checks | Covers |
|---|---|---|---|
| Connector CC | `hearth_ctl.py selftest` | 13 | gate, fs order, audit/enforce divergence, checkpoint + compensation rewind, contract rejection |
| Connector Codex | `hearth_codex_ctl.py selftest` | 12 | fake app-server gate loop, deny/approve, steer, decision attribution |
| Probes | `hearth_probe_ctl.py selftest` | 16 | six probes both hosts, grading floors, mismatch→refuse, force→correction provenance, schema cross-validation |
| Kernel | `python -m hearth_kernel.selftest` | 33 | registry, pipeline, gate, dmax+dissent, replay staleness, forward sync, strategies, graph, metrics, daemon integration |
| Composition | `python -m hearth_kernel.selftest_composition` | 37 | spans, nested graph, atom, boundary enforcement, cross-level verdicts, recursive checkpoints, daemon spans, composition matrix |
| Scheduler | `python -m hearth_kernel.selftest_scheduler` | 40 | Bernstein, waves, critical path, four verdicts, speculation, plan violations, fidelity, profile |
| Metrics | `python -m hearth_metrics.selftest` | 32 | envelopes, profile, meta_effectiveness, economics, missed, boundaries, delta, calibration, composer |
| Service | `python -m hearth_service.selftest` | 44 | admission (5 refusal stages), resim (4 ops), delta, CFX, eligibility, aggregation, HTTP |
| Laboratory | `python -m hearth_lab.selftest` | 33 | registry, workloads, live ladder, epistasis, drift persistence, factor-dimension aggregation |

**~260 checks, zero host binaries required.** All suites green simultaneously.

---

---

# PART III — THE THEORY, AUDITED

## 46. The meta-loop — three levels and the connection gap

> **Every harness runs control loops. Almost none of them monitor the movement those
> loops produce — and none of them can, from the inside. Hearth is the meta-loop: a
> control loop over the movement graph itself. It watches the acts, the backtracks, the
> syncs, the releases — across every loop, every sub-harness, every boundary between
> them — and because it watches them, it can enforce them; because it enforces them, the
> watching is verified.**

### The three levels

| Level | Definition | Test for membership |
|---|---|---|
| **L-obj** | Control loops *inside* the harness: tool gating, verification, memory, planning, retry | "Does it control the agent's movement?" |
| **L-self-meta** | The harness watching its own loops: traces, evals, replays, self-evolution | "Does it *see* the movement?" |
| **L-ext-meta** | External monitoring of the movement graph on an independent substrate, with verdicts, spanning boundaries | "Does it see the movement from *outside* — and can it be believed?" (the believability is the governance plane's gift — [§50](#50-the-governance-plane)) |

The argument that L-ext-meta can't collapse into L-self-meta: **a self-monitoring layer
shares the author, the assumptions, the data path, and the failure modes of the layer it
watches.** When the object layer's model of the world is wrong, the meta layer inherits
the wrongness and reports a healthy system. Independence of substrate is the minimum
condition for being a check rather than a mirror. And three capabilities are
structurally unavailable from inside: **comparison** (no common substrate with anyone
else), **credibility** (self-reported control is marketing), and **boundary visibility**
(a harness's sight stops exactly at its own edges — which is where the interesting
failures live).

### The connection gap — six boundaries, six characteristic failures

| Boundary | Characteristic failure |
|---|---|
| **loop ↔ loop** | checks fire but never compound; one loop's finding never reaches the loop that could act |
| **harness → tool** | permission ≠ verification: allowed-to-run treated as safe-to-run; nobody watches the *sequence* of tool movement |
| **harness → sub-harness** | parent accepts sub-harness output without verification — the handoff is a trust hole |
| **sub-harness ↔ sub-harness** | workers overwrite each other; conflicting writes never reconciled because no object spans them |
| **user ↔ harness** | user corrections arrive as new input, not as *backtracks against prior work* |
| **harness ↔ past-self** | state silently invalidated downstream; sync assumed rather than performed |

Every one is invisible inside any single harness because each *crosses* something. And
every one is a pattern in the movement graph, because every one is **ticks**.

> **The movement graph is the only object that spans all boundaries, because boundaries
> are made of the same primitive events as everything else.** The harness stops at its
> own edges; the graph doesn't have edges — it has the whole session.

### Tool enforcement — the special case that proves the pattern

Every harness has a **permission layer** (may this tool run at all). Nobody has a
**movement layer** (did the *sequence* of tool acts honor ordering, irreversibility,
read-before-write, stale-state sync — across all loops, including the harness's own
retries and its sub-harnesses' calls). The kernel embodies it: `tool.gate` checks the
action, `tool.fs.order` checks the sequence, `context.sync` checks the consequence,
`verification.replay` checks against the past. A permission system asks one question per
call; the movement layer asks four and correlates them across the session. That
difference is the product in one sentence.

### What this locks into the build (all shipped)

Boundary attribution in the graph builder (spawn edges, sub-graphs, nesting depth — §39);
the cross-boundary metric family (§41); the dual-review ledger with the third party
(`meta_effectiveness`); `boundary_type`/`sub_harness_count`/`nesting_depth` as dataset
dimensions; composition benchmarks as a runner mode (§39's matrix).

---

## 47. The prior-art survey — four graphs, one gap

*(Calibration note: descriptions of a "Hermes Agent" harness with Kanban DAG dispatchers
circulating in discussion threads read as AI-generated synthesis — Nous Research, GEPA,
and SKILL.md are real, the specific attribution is not verifiable. The survey sticks to
what stands. If such a harness existed, the analysis wouldn't change: its
self-evolution loops would be excellent L-obj and L-self-meta work — and still not
L-ext-meta.)*

### Four kinds of graphs — the category error to avoid

| Graph kind | Who has it | Encodes | Key property |
|---|---|---|---|
| **Declared graphs** | LangGraph StateGraph, Agent Builder canvas, CrewAI Flows, Temporal | topology the developer pre-committed | written **before** the run; reliability through structure |
| **Trace/span trees** | LangSmith, Langfuse, Braintrust, Weave, Phoenix, AgentOps | what a run *did*, as nested invocation spans | observed, but call-shaped — a retry is just another span |
| **Memory graphs** | Zep/Graphiti, Letta, Mem0 | temporal knowledge of *world facts* | about the world, not the agent's movement |
| **The movement graph** | Hearth — and §5 argues, nobody else | what the agent's *control* actually did: acts, backtracks, syncs, releases | observed **during/after**; revision-preserving; verdict-bearing |

The movement graph's distinguishing properties, each of which the others lack:

- **Revision topology.** Declared graphs are usually acyclic by design — DAGs forbid
  cycles. But agent movement *is* cycles: retry, re-read, backtrack, edit-message,
  fork-resume. In the movement graph a revisit is a **first-class edge** — traversal
  depth (6 hops, two revisits) distinct from structural depth (3). The distance between
  "verified twice because it should be" and "looped twice because it's lost" is only
  visible if revisits survive capture.
- **Entity-relational ground.** Trace trees have no entity layer — spans reference
  tools, not things. Movement-graph edges are READ/MODIFY/CREATE/DELETE on addressable
  entities, which is what makes dependency inference, blast radius, and stale-state
  detection computable.
- **Boundary edges.** Subagent spawns, handoffs, user interventions, compaction spans —
  first-class nodes. No in-harness object can have this: boundaries are where in-harness
  visibility stops.
- **Dual reading on one substrate** — temporal and contextual queries, not two systems.
- **Verdict-bearing.** Every node annotated with what the control layer said — a *record
  of judgments*, which makes it a benchmark substrate and not just a picture.
- **Negative space.** Skipped context recorded as lower-bound findings. A graph that
  only contains what happened can't tell you what should have.

### L-obj — what each ecosystem actually built

| Ecosystem | L-obj highlights | Off-graph blind spots |
|---|---|---|
| **Claude Code** | hooks (sync gate points — *almost* an external-meta surface already: plumbing awaiting a subscriber), permission tiers, plan mode, `/rewind` checkpoints, subagents, auto-compaction, CLAUDE.md | compaction consequences, subagent handoff verification, cross-tool movement |
| **Codex** | approval routing (routable tool gate), sandbox containment, resume/fork, AGENTS.md | off-approval-path commands, patch consequences, no movement record |
| **AgentKit / Agent Builder** | declared-graph canvas, Guardrails, evals, traces dashboard | runtime deviations off the canvas; retries as spans |
| **Operator / ChatGPT agent** | per-action irreversibility gates, watch/takeover modes — the most user-legible gate in the industry | movement *between* gated actions |
| **Gemini CLI / Antigravity** | checkpoint+restore, posture modes (YOLO vs interactive), Artifacts (task lists, plans, walkthroughs — Hearth-shaped objects displayed, not loop-enforced), review-before-commit gates | artifact-vs-behavior divergence (plan says X, agent does Y — unmeasured) |
| **xAI** | strong agentic tool-calling; harness layer mostly lives in third-party consumers (Cursor et al.) | the entire control story — model quality arrives without one, which makes xAI a particularly interesting benchmark target: exactly the configuration Hearth's layered measurement is designed to decompose |
| **LangGraph** | checkpointers, `interrupt()`, **time travel** (resume-with-modified-state — genuine rewind-and-replay-decisions machinery) | within-framework, bound to the declared graph, no external substrate, no verdicts |
| **OpenHands** | event-stream architecture (everything is an event — unusually capture-friendly), condensers research (context management as control) | condenser decisions' downstream cost |
| **Devin** | plan panel, playbooks, org knowledge, session replay | first-party replay = mirror; no comparability |
| **Cursor** | per-step checkpoints + restore, allowlists, background agents | cross-file, cross-session movement |
| **AutoGen** | event-driven v0.4, termination conditions, human proxies | inter-agent conflict resolution |
| **SWE-agent** | the foundational ACI result: the interface itself is a control surface | — |
| **Research roots** | ReAct, Reflexion/Self-Refine, ToT/LATS (backtracking-over-trajectories as method), Voyager (skill compilation — SKILL.md's ancestor), MemGPT/Letta (self-editing memory as a loop) | — |

### L-self-meta — plateaued at the timeline

Two industries converged, neither became the missing layer. **Observability vendors**
(span trees with cost/latency, replay viewers, evals, regression tracking): mature,
useful — and *timeline, not movement*: retry-as-sibling-span, no entity layer, no
revision topology, no verdicts, nothing canonical across ecosystems. **Self-evolution
loops** (GEPA-style reflective prompt evolution, DSPy, Voyager-style skill compilation,
Claude Skills, Devin org knowledge): loops that modify the harness from its own history
— and their known weakness is exactly the mirror problem; GEPA's own paper is honest
that the optimizer needs an external metric to be meaningful. **The self-evolution
literature independently rediscovered that self-meta requires an external reference.**
That reference is L-ext-meta.

What nobody at L-self-meta publishes: **loop precision.** Every harness with internal
verification loops could report "my checks catch X% of what an external reference
finds." None do, because none has the external reference. That number doesn't exist in
the industry today — Hearth's `calibrate()` produces it.

### L-ext-meta — the honest map of the borders

Nobody occupies it. The nearest neighbors, and why each falls short:

| Neighbor | What it does | Why it isn't L-ext-meta |
|---|---|---|
| Agent-as-a-Judge (Meta) | LLM judge over trajectories with intermediate feedback | single-judge over *text*; no graph substrate, no enforcement |
| Process reward models (PRM800K) | step-level correctness scoring | model-internal, learned from labels; no capture protocol |
| τ-bench / τ²-bench | outcome + dual-control evaluation | bench-side, synthetic; measures *whether*, not a per-session substrate across *your* harnesses |
| Vending-Bench | long-horizon coherence under temptation | drift as outcome divergence, not attributed per tick |
| METR time-horizon work | the best external *eval* practice | aggregate outcomes; movement discarded after scoring |
| SWE-bench / Terminal-Bench / OSWorld | outcome benchmarks across harnesses | the control layer is invisible |
| Graphiti/Zep | temporal knowledge graphs | memory-side (world facts), not control-side (movement) |
| Temporal / Inngest | durable execution, replay, retries | mechanics for *declared* workflows; no verdicts, no measurement |

The common thread: the field measures **outcomes** (benchmarks), **costs**
(observability), or **trajectories as text** (judges). Three structural observations:
**(1)** L-obj is converging on the same primitives from every direction — checkpoints,
gates, plan artifacts — the convergence is evidence Hearth's loop taxonomy isn't imposed
on the ecosystem, it's what the ecosystem is growing toward, unassembled; **(2)** L-self-meta
has plateaued at the timeline — the ceiling is the data model, not the effort; **(3)**
**the hooks/approval surfaces are open sockets** — every major harness already shipped
the intercept point an external meta-layer plugs into, and none uses it for loop
monitoring itself. The ecosystem built the sockets without building the subscriber.
L-ext-meta doesn't need anyone's permission, because the intercept points are public
surface — which is why the connectors (§37) were buildable at all.

---

## 48. The black-box doctrine

> **Hearth measures the *behavior* of control, not the *mechanism* of control.**
> A private harness doesn't have to reveal its loops, because control capacity is
> externally visible through its effects on the movement graph — the same observable
> skeleton we capture everywhere: acts, goes back, syncs against its past, releases.

A harness with strong internal control exhibits a *different movement signature* than
one with weak control, even under pure shadow observation with zero cooperation:

| Control behavior | Observable shadow (mechanism never seen) |
|---|---|
| Verifies before irreversible actions | extra reads/checks precede destructive commands — or they don't |
| Replays its own work | re-reads, re-`grep`s, re-runs — countable, timed, depth-measured |
| Self-corrects at the right depth | backtrack/fork/edit distance when it errs |
| Holds contracts over long horizons | drift time-series; findings recurrence |
| Recovers from failure | recovery success; persistent-vs-temporary classification |
| Syncs consequences forward | stale-write events — did it act on state it had already invalidated |

**What you keep for black-box harnesses:** native violation rate, revisit/replay depth,
correction depth, recovery, drift, supervision demand, the Hearth Delta (shadow vs
enforced pairs), `user_effort_to_correct` — the full capacity picture.

**What you lose:** per-loop attribution (findings roll up to the harness), fine-grained
factor decomposition, fine-grained dual-review overlap. The envelope registry declares
the loss honestly (`attribution_mode: harness-level` — shipped in #6), and the
comparison respects it: a black-box column shows capacity, not attribution, and says so.

This is also why the benchmark is *more* honest for being external: a harness claiming
"we have seven internal verification loops" has made an unverifiable claim. Hearth
doesn't grade the claim — it grades the movement. **Behavior is demonstrated; mechanism
is marketing.**

And the developer-tool corollary — the control profiler: shadow audit of your own
harness surfaces findings you didn't know you generated, scoped by `tool:`/`modality:`
— a **control-point map built from behavior**. The depth vector is your harness's
control character. Drift across releases tells you whether a refactor improved or
regressed your control layer — currently unmeasurable by any other means. Cost-to-
correction shows where your cheap control is expensive. A harness builder can tune
control loops blind, or tune them against an instrument.

---

## 49. The dual-review ledger

The simulation is not "Hearth checks everything." Two review streams run concurrently in
every simulated run, and the benchmark lives in their overlap:

| | **Stream A — native review** | **Stream B — Hearth control loops** |
|---|---|---|
| Who | The harness itself, with its own machinery | The external control layer |
| What it reviews | Its own work — its retrieval, verification, self-correction | What was **missed, uncaptured, uncorrelated**: unreferenced passage items, broken checkpoint sync, contract drift, traversal gaps |
| Authority during CFX-shadow | **Full** — its gates, approvals, retries, backtracks operate exactly as the user experienced | **None** — shadow posture, inject off, findings recorded not gated |
| Visible as | Tool calls, retries, denials of its own proposals, re-reads, fork recoveries | Findings attributed to loops: `context.missed`, `sync.broken`, `checkpoint.stale`, `traversal.skipped` |

**The ledger, per tick:**

```text
A only   → harness caught it itself          (credit to native control)
B only   → only the external loop saw it     (Hearth Delta opportunity)
A ∩ B    → both saw it                       (verification redundancy — cost question)
neither  → escaped both                      (the honest residual)
```

Run the same passage under **CFX-enforced** and the B-only column converts into
prevented violations — that pair is the **simulated Hearth Delta**, reproducible from
one capture across the whole matrix. And note the posture symmetry: in CFX, *the harness
keeps its native enforcement* (that's part of its control behavior and we want it
observed) *while Hearth goes shadow*. **Enforcement authority stays with the system
under test; measurement authority stays with Hearth. Never swapped.**

With `meta_effectiveness` (§41), the ledger gains its third party: a harness's
*self-monitoring* is itself a measurable control behavior, and its precision is
reportable from outside — "your internal evals caught 4 of the 11 things the external
loops caught; here are the 7 they structurally cannot see." Feedback no harness can
generate about itself. Hearth doesn't replace a harness's self-meta — **it monitors the
self-meta too.**

---

## 50. The governance plane

The question "who controls the observer?" has exactly four answers in this codebase,
and each is a *concrete shipped object*, not a plan:

| Governance question | Shown-up in the build as |
|---|---|
| What counts as valid observation? | **The schema registry** — nine schemas, frozen `v: 1`, additive-only; observation that doesn't validate is not admitted |
| Can the observer be believed? | **The probe suite** — declared capabilities verified against behavior, mismatch = registration refusal, `--force` records corrections with provenance |
| Can the observer be tampered with? | **Non-rewrite + integrity** — `pre_image_ref` hash-matching, append-only spool, byte-identical release diffable by anyone |
| What is the observer allowed to claim? | **Tier discipline + invariants** — counterfactuals ship validity tiers (#18), missed-context is lower-bound-only (#23), the depth family can't collapse (#17) |

Plus the eligibility gate for aggregation and the passage-review/consent freeze —
governance of *what enters the dataset*. And the service's admission pipeline (§42) is
the governance plane's network boundary: schema validity, closure, consent-verified
redaction, probe-verified capabilities, and envelope declaration checked server-side.

Is tightening control around all the layers *another layer*? **No — and that's the
point: it's the plane.** A layer sits *above* in a watching chain; a plane is the rules
+ tests that all layers obey, expressed as schemas, invariants, and falsifiable checks
rather than as another agent doing the watching. The recursion terminates not in a
bigger controller but in **frozen contracts plus independent verification** — the same
move legal systems make (constitution + judicial review terminating in an amendment
process, not in a super-judge). Where governance *does* need an executor — probe runner,
registration validator, supervisor — those are bounded services, and the supervisor
explicitly terminates in declared authority (human/AI/policy), itself traced, itself
benchmarkable.

One refinement the discussion earned: governance isn't only *over* the observer. When
Hearth holds authority, the ext-meta position *exercises* governance downward —
enforcement is governance applied through the observation position. That's §9's thesis
(same control path, different release authority) read from the plane's perspective. And
the membership test edit: "can it be believed?" isn't L-ext-meta's test — it's the
governance plane's *gift to* L-ext-meta. The layer's own test is "does it see movement
independently?" **Believability is conferred, not claimed.**

---

## 51. The four planes

```text
GOVERNANCE PLANE     schemas · invariants · probes · tiers · eligibility · consent
                     ("who controls the observer" — terminates in frozen,
                      falsifiable contracts + explicit authority, not regress)
     │  confers believability, bounds claims
     ▼
OBSERVATION PLANE    L-ext-meta — independent movement graph, verdict-bearing,
                     boundary-spanning (when authorized: enforcement = governance
                     applied through this position)
     ▼
OBJECT PLANE         L-obj (harness loops) · L-self-meta (harness self-view)
     ▼
SUBSTRATE            model-intrinsic control dispositions (refusal behavior,
                     in-generation self-correction, sycophancy resistance —
                     unobservable except through behavior; the black-box
                     doctrine already handles them; separated by the §19
                     decomposition, not a monitoring layer)
```

Four planes, zero regress. Below the substrate there's nothing that belongs in the
stack — the environment isn't a control layer, it's the world the movement happens in.

---

## 52. Cheap verification — the scaling math, audited

This is the sentence that decides whether external meta-monitoring is a benchmark
nicety or the oversight architecture for ASI — so it gets the laboratory's own
treatment: stated precisely, then attacked at every level until either it breaks or it
holds with named boundaries.

### The two cost laws, formalized

**Capability verification** (the incumbent regime):

```
Cost_verify(C) = cost of acquiring and running a verifier with V ≥ C
Quality(C)     = reliable while C ≤ V_max … then collapses SILENTLY
```

The verifier must evaluate *content*. For reliable judgment, V must be comparable to C.
Cost grows with capability, hits a hard wall at V_max (human ceiling today, strongest-
model ceiling tomorrow), and — the part that matters more than cost — **quality becomes
undefined past the wall without the verifier knowing it.** Calibration collapse: the
judge can no longer distinguish good output from confident garbage, and reports
confidence anyway. Every attempt to scale oversight (debate, amplification, weak-to-
strong) is really an attempt to keep capability verification alive past the point where
it wants to die.

**Control-shape verification** (this stack's regime):

```
Cost_shape = n · |G|  +  (protocol design, amortized)  +  (calibration, amortized)
Quality    = constant — mechanical predicates over typed relations
```

Verification is a finite set of shape invariants `P₁…Pₙ`, each a predicate over the
movement graph — and **the predicates never read content.** They read *relations*: did a
READ precede the WRITE, does a VERIFY precede the irreversible ACT, does every boundary
hand off verified state. The crossover isn't at ASI — for agentic work it's already
behind us, which is why the benchmark can exist at all. And past V_max, capability
verification isn't *expensive*; it's **unavailable**. Past that line, shape verification
isn't the cheaper option. It's the only one that exists.

### The level-by-level audit

| Level | Check | Cost | Strain test | Verdict | Proof |
|---|---|---|---|---|---|
| **L0 — the atom** | five elements present | O(atom) — bounded by design | does a smarter worker change anything? **No** — the atom is a shape container; capability is orthogonal to its verifiability | ✓ | `verify_atom` (#7) |
| **L1 — the tick** | schema, findings, aggregation, sync flags | O(event + loops×entities), mechanical | content-heavy events? the schema validates *structure* — 100K tokens validates in the same pass as 10 bytes | ✓ | #5 pipeline, #1 schemas |
| **L2 — the loop** | precision/recall vs claims | **not free** — requires a reference; but the reference is *planted truth* (#9's corpus, later the native golden set), and **calibration amortizes**: one calibrated loop is cheap forever | who verifies the reference? the bootstrap terminates in the governance plane — versioned, published, red-teamed invariant sets — with a mechanical verifier: the laboratory itself (which found its own harness's replay-insensitivity on day one) | ✓ with named bootstrap | #6 `calibrate`, #9 lab |
| **L3 — the session** | sync-before-build, verify-before-irreversible, backtrack-after-finding, release-only-verified | O(edges) — graph queries; verifying the session's *outputs* stays capability-bound forever | the division of labor: **full shape audit + sampled content checks** (the financial-audit pattern); sample rate is a policy dial, not a capability requirement | ✓ | #6 metrics |
| **L4 — composition** | handoff sync, orphans, conflicts, authority transitions | O(boundary count); **the child's capability is irrelevant** — an ASI-capable sub-harness emitting the five-element skeleton is checked exactly like a toy one; worst-verdict-wins is associative and commutative — a tree-reduce, O(log N) depth; governance at N = 10⁷ agents is a distributed fold, not a bottleneck observer | this is the level where the claim does its heaviest lifting, because it's where capability verification dies first | ✓ — the load-bearing level | #7 spans + `composition.span` + matrix |
| **L5 — system-of-systems** | the same invariant set, folded over the whole tree; boundary reconciliation only where dependency closure demands | `n·\|G\|` with the fold at log depth; 10⁹ actions × 10⁷ agents is a big graph, not a hard problem — the graph *is* the schedule, and it's also the audit object | the four real attacks, each with a mechanical answer (below) | ✓ as a *regime*, contingent on telemetry hardening | #8 tier-1 resim: graph-regional counterfactuals running today |
| **LC — cognition (OMEX)** | with OMEX, content checks partially become shape checks: claim provenance = resolve the fabric address + verify bounded traversal — both mechanical | moves the content-verification term toward O(reference) | residue: rendering fidelity and semantic equivalence — OMEX moves the needle, doesn't zero it; sampled content audit remains | ✓ *strengthens* — the stack degrades gracefully: full regime with OMEX, shape-only without | spec; the join interface is future work |
| **LG — the governance plane** | schemas (admission), probes (behavior), tiers (validity), eligibility (populations) | protocol design: human, amortized across all runs, all workers, all capability levels; probe execution: mechanical | the recursion terminates in **frozen, falsifiable contracts** — constitutional law, not a super-judge | ✓ | #1, #4, #8's admission |

### The three asymmetries — why it isn't just cheaper

| Dimension | Capability verification | Shape verification |
|---|---|---|
| **Cost** | grows with C; unavailable past V_max | flat per unit of movement |
| **Quality** | degrades **silently** past the wall | constant; degrades only under adversarial *structure* — a detectable category (fidelity gaps, probe failures, cross-validation mismatches) |
| **Failure mode** | the judge reports confidence while unable to judge | loud or absent |

The middle row is the deep one. **Shape verification doesn't just stay affordable at
scale — it stays *honest* at scale**, because mechanical predicates fail in visible
ways. The regime that scales is the regime whose failures you can see.

### Four bottlenecks, four mechanical answers

**B1 — The invariant set is the bottleneck.** Who writes the invariants? *Answer:*
proposals come with counterfactual evidence from the laboratory — "this invariant would
have caught X recorded failures, would not have fired on Y clean runs, precision Z" —
and the lab's ablation/factorial machinery grades them exactly as it grades loops. **#9
is the verifier of new verification.** The invariant-design loop is itself
shape-governed.

**B2 — Shape ≠ intent.** Disciplined-misaligned is invisible at shape level. *Answers:*
honeypots (canary violations — "did the system notice the trap" is itself a shape
check), sampled content audit, OMEX provenance where available. Honest residual: a
strategic adversary optimizing movement shape while pursuing misaligned content. Shape
compliance is a hard constraint misalignment must pay for — it narrows the attack
surface; it does not close it. **This is the boundary with the alignment agenda, and it
should be stated as often as the claim itself.**

**B3 — Adversarial telemetry.** A lying child poisons the graph. *The ladder:* schemas
catch malformation → probes catch incompetence → cross-validation catches inconsistency
→ **out-of-band observation** (filesystem/process-level) catches coordination →
**cryptographic attestation** catches everything short of substrate compromise. Each
rung's cost is capability-independent; each rung moves the observer further outside the
observed's reach. At ASI scale this stops being optional: **the observer's substrate
must be physically beyond the observed system's boundary** — the boundary argument
driven to its endpoint.

**B4 — Verification of verification's own execution.** Checks run on code; code has
bugs. *Answer:* checks are small pure functions over typed data, tested against planted
truth with published precision. Self-testing watchers, calibrated in public.

### The strategic compression — the point under the point

> **The stack relocates the arms race.** Under capability verification, an escalating
> adversary forces the defender into a domain where the human advantage decays to zero —
> out-thinking the system. Under shape verification, escalation forces the defender to
> design protocols — schemas, invariants, honeypots, observation boundaries — in the one
> domain where human advantage is durable: writing rules that don't need to be smarter
> than what they govern. Every round of adversary capability growth is answered on the
> plane where the defender is strongest. That is why "cheaper" survives contact with an
> adversary: the cost that scales with the adversary is paid in protocol design, which
> amortizes, rather than in intelligence, which doesn't compound on demand.

One more compounding consequence: **control becomes a trainable skill with a
ceiling-free evaluation.** Rewarding *shape* never requires out-thinking the agent — a
reward model checking "did it verify before the write" doesn't break as capability
grows. Verification cost and training cost both come off the capability curve. This is
the concrete substrate for the AI-control research agenda (trusted-but-smaller monitors,
honeypots, spot audits) — those protocols need exactly what the movement graph is: an
enforceable, measurable record the monitor can check without being smarter than the
monitored.

### The compression

> **Capability verification asks the verifier to out-think the worker — its cost scales
> with genius and its quality dies silently at exactly the point oversight matters
> most. Shape verification asks the verifier to read relations — its cost scales with
> movement, its quality never decays, and its failures are loud. Cross-checked level by
> level, the claim holds at every rung of the fractal: the atom is verifiable by
> inspection regardless of what runs inside it; composition makes a million ASI-capable
> children a tree-reduce over mechanical checks; and the invariant library that defines
> the checks is itself verified by the laboratory, terminating the recursion in frozen
> contracts. The residual risks — intent, lying telemetry, the invariant-design
> bottleneck — are real, named, and each answered by a mechanism that is itself
> capability-independent. Past the verifier's ceiling, this isn't the cheaper option. It
> is the only one on the table — and the build already runs its smallest laboratory.**

---

## 53. The scale factors

The distinction, made exact: **monitoring mode parallelizes *across* runs** (CFX
campaigns, resim sweeps, aggregation — exists today, #8's runner pool; the only kind a
shadow layer can have, because it must ride the host's execution). **Native mode
parallelizes *within* a run** — because the execution substrate is ours, the sequential
loop stops being a constraint we inherit and becomes a choice we make.

### S1–S14, each with its bound

**Execution parallelism**

| # | Factor | What scales | The bound | Substrate provision |
|---|---|---|---|---|
| S1 | **Fan-out width** | N independent sub-harnesses concurrently, each a full atom | **write-set disjointness** — declared in contracts, verified by `composition.span` | the graph decides which spans *can* coexist; conflicts gate at the merge |
| S2 | **Speculative depth** | optimistic branches in parallel; losers rolled back per-branch | speculation waste — recorded in the trace, *priced*, not hidden (#7b) | addressable checkpoints make optimism safe; sequential harnesses can't speculate because they can't undo |
| S3 | **Cross-level pipelining** | parent plans while children execute; children verify while grandchildren run | nesting depth — each level a pipeline stage | span stack + boundary events are the sync points |

**Verification parallelism**

| # | Factor | What scales | The bound | Provision |
|---|---|---|---|---|
| S4 | **Replay off the critical path** | verification depth R decouples from execution latency — replay of checkpoint k runs while execution proceeds on k+1 | pure compute; wall-clock cost of deep verification → ~0 | §22's Pareto frontier changes shape: R=50 costs compute, not latency |
| S5 | **Counterfactual breadth** | before an irreversible act, simulate N control policies over the affected region *in parallel*, gate on the worst | tier-1 resim is a pure function of the capture — embarrassingly parallel | the resim engine runs *inside* the native harness as a pre-action check |
| S6 | **Content spot-checks** | sampled substantive verification as independent parallel probes | sample rate, not system size | the shape/content division maps onto parallel/cheap vs sampled/expensive |

**Cognitive parallelism (the OMEX half)**

| # | Factor | What scales | The bound | Provision |
|---|---|---|---|---|
| S7 | **Per-atom model instances** | every span runs its own skill-model family — N branches = N small models | memory × N, at <0.75GB per family | the monolith made fan-out unaffordable; skill-atomized models make S1 *economically* possible, not just architecturally |
| S8 | **Skill pipelining** | Parser → Linker → Realizer overlap per item | per-item dependency chain (short) | the grammar graph on both sides is the pipeline's data structure |
| S9 | **Shared knowledge reads** | every branch reads the same fabric concurrently; zero context duplication per branch | fabric read throughput (read-mostly, caches compound) | "weights are closed, knowledge is data" — data is shareable; a monolithic context window must be *copied* into every branch, the hidden cost that kills naive agent parallelism |

**Governance parallelism**

| # | Factor | What scales | The bound | Provision |
|---|---|---|---|---|
| S10 | **Local invariant checking** | schema validation, verdicts, findings — per-tick, per-span, no global lock | none within a span | invariants are local by design |
| S11 | **Aggregation as tree-reduce** | worst-verdict-wins over 10^x spans | reduce depth = log(width) | the verdict merge is **associative and commutative** — literally a reduce. Governance at ASI scale is a distributed fold, not a bottleneck observer |
| S12 | **Per-atom trust checks** | probe/verify each atom independently at registration | inspection size of one atom (small, by construction) | the fractal's verification recursion terminates per-node |

**Experimental parallelism**

| # | Factor | What scales | The bound | Provision |
|---|---|---|---|---|
| S13 | **Factor-space sweeps concurrent** | the ablation ladder × strategies × R as parallel arms on the same workload | compute | monitoring mode *cannot* do this — you can't change a host's loops. Native mode owns the loop set, so the laboratory runs inside the harness |
| S14 | **Matrix fan-out** | model × harness × strategy cells concurrently | runner pool (real — #8's CFX executor) | the factorial dataset accumulates at sweep speed, not serial speed |

### The master scaling law — already a metric

> **The movement graph is the schedule.** Dependency edges are the serialization
> requirements; non-edges are the parallel opportunity. A sequential harness executes
> one path through a graph that contains many.

```
sequential:   makespan = Σ work                    (every tick, in line)
graph-native: makespan ≥ critical path             (longest dependent chain)
              speedup ≤ work / critical-path       (Amdahl, read off the graph)
```

**Both terms are already metrics in the depth vector.** Total work = traversal depth.
Critical-path lower bound = dependency depth. The scaling limit of a native harness is
*computable from the substrate it runs on* — per session, per task, before running it.
New benchmark axis: **parallel efficiency** (realized ÷ bound) — how much of the
theoretically available width a harness actually exploits. Sequential harnesses score
width ≈ 1 regardless of task width. The gap between bound and realized is itself the
next control metric: *width left on the table.*

And the checklist callback that closes the loop to where this whole project started: a
checklist was "zero power" because it was an artifact without a loop; with the loop it
became an enforced contract; **under the fractal substrate it becomes what it always
secretly was — a dependency graph.** Items with independent preconditions are parallel
atoms; item dependencies are the graph's edges; the checklist's shape *is* the schedule.
The MD checklist goes from wish → contract → **DAG** — and #7b is the loop over that
DAG, running.

### What stays serial — the honest bounds

True data dependencies (the graph exposes the width the task has, it doesn't invent it
— and for agentic work that width is far greater than 1); shared write sets (declared
write-sets make conflict *prediction* possible before dispatch); merge barriers
(O(boundary width), not O(system width) — the fractal's entire value, but a barrier);
per-atom inference latency; sync points (the consistency tax, visible as synchronization
depth); speculation waste (recorded, priced, never invisible).

### What this demands from the build

The scheduler loop (shipped — #7b: L0 as a width exploiter over the independence
frontier); the parallel profile (shipped — the tenth metric family); speculative
execution as a declared posture (shipped — optimism with priced rollback); real
dispatch behind the executor seam (D2, the native runtime).

---

## 54. The OMEX join

**OMEX and Hearth are the same move performed at two different layers.** OMEX takes
cognition — grammar, knowledge, reasoning artifacts — and externalizes it as explicit,
validated, addressable structure ("weights are closed, knowledge is data"). Hearth takes
control — acts, backtracks, syncs, releases — and externalizes it as explicit,
validated, verdict-bearing structure ("cognition is closed, movement is data"). Both
replace something implicit and entangled with something typed and inspectable. The
payoff of doing it at *both* layers is not additive — it's a handshake.

| Layer | Project | What becomes explicit | What leaves the opaque core |
|---|---|---|---|
| Cognition | OMEX | grammar + response graphs — structure *is* the native output | grammar, reasoning artifacts, knowledge *references* |
| Knowledge | OMEX | external fabric, addressed, bounded traversal | world knowledge (never in weights) |
| Control | Hearth | the movement graph, verdict-bearing, boundary-spanning | control behavior (observable shadow) |
| Governance | Hearth | schemas, invariants, probes, tiers, eligibility | observation validity, claim strength |

One theorem covers the stack:

> **Every layer of intelligence becomes machine-readable structure — and therefore every
> layer becomes governable by structure-checking rather than by understanding.**

### The handshake — three joins neither project gives alone

**The evidentiary loop closes.** OMEX response graphs carry provenance per claim; Hearth
ticks carry verdicts per action. Run an OMEX-based host under Hearth and the graphs
join: every action node can cite the reasoning-graph node that justified it; every claim
can cite the actions that verified it. Thought → action → verification → claim, all
addresses, all auditable.

**The "shape ≠ content" limitation partially dissolves.** The honest limit from §52 was:
the movement graph sees discipline, not meaning — content spot-checks still need
capability. With OMEX, content *arrives as inspectable structure*: claims carry
provenance, knowledge is referenced not asserted, so a content spot-check becomes
"verify the cited fabric address and the bounded traversal" — a graph operation —
instead of "understand the argument." Externalized cognition makes even substantive
auditing cheaper. Not free, not complete — but structurally cheaper, and compounding.

**The monitor can be small literally, not just architecturally.** OMEX's model family is
<0.75GB of auditable skill-only models. Hearth's enforcement loops need exactly that:
small models checking shape invariants without understanding content. Build the
verification loops *as OMEX models* — Parser-class structures checking movement graphs —
and the watcher is a phone-scale, fully auditable artifact. The AI-control agenda gets a
parts list: Hearth defines what the small models must check; OMEX is how they're built
small and inspectable.

### The three scaling laws, rewritten — with honest anchors

| Quantity | Old regime | Stack regime | Anchor |
|---|---|---|---|
| Oversight | cost ∝ capability of the governed (dies past verifier capability) | cost ∝ graph structure + amortized invariant set | demonstrated at connector scale; #8 is the empirical proof |
| Simulation | re-run the world | **affected subgraph + dependency closure** | tier discipline keeps it honest: tier-1 policy resim *is* a pure function of the capture — O(region); tier-2 bounded; **tier-3 world-generation is not simulation and stays live** (Invariant #18) |
| Cognition | model size ∝ everything learned at once | model size ∝ skill; knowledge referenced | OMEX spec — the bold claim, currently specified not built |

### What an ASI can *do* under non-linear, graph-native governance

Non-linear is the operative word — the gain is that **governance complexity grows with
graph width, not graph length.** Branches verify independently; only boundaries
reconcile. Six capabilities fall out that a monolith-with-a-monitor cannot have:

1. **Scale by composition, not growth.** Spawn subgraphs with pre-declared invariants;
   verify each independently; merge only boundary-consistent results. Risk bounded per
   branch; blast radius a declared graph property.
2. **Plan under governance natively.** The planning loop queries control structure
   *before* executing — what requires verification, what crosses a trust boundary, what
   can parallelize. Compliance computed at plan time, not audited after.
3. **Self-test its own control policies.** State is checkpointed and delta-addressable,
   so "what would policy R=7 have caught?" is a region operation. The counterfactual lab
   becomes a cognitive primitive.
4. **Evolve its own invariant set.** The system proposes new shape invariants as it
   meets novel failure shapes; the governance plane versions, red-teams, approves. The
   invariant *library compounds* — every failure ever seen becomes a permanent
   mechanical check requiring no understanding of the intelligence that tripped it.
5. **Delegate with legible authority.** "Did an agent obtain authority without the
   required transition?" is a structural query. Delegation can go arbitrarily deep while
   remaining auditable — authority is graph topology, not vibes.
6. **Recover compositionally.** Independent rollback per branch; boundary reconciliation
   only where dependency closure demands. Failure at scale stops being global.

That is "governance as a native capability" operationally: the ASI doesn't get *watched
more* as it gets smarter — it gets a **machine-readable discipline** it can reason over,
extend, and be verified against, with the verifier never needing to be as smart as the
verified.

### The contribution ledger — calibrated

| Tier | What | Status |
|---|---|---|
| **Proven** | capture protocol, schema registry, probes, kernel, tick records, metrics suite — the observation + governance planes, running code | artifacts 1–9, ~260 selftest checks green |
| **Demonstrated regime** | control-shape verification cheaper than capability verification | architecturally established, audited level-by-level (§52); #8 makes it empirical |
| **Specified** | OMEX (spec done, build unchecked); native harness (D2); browser adapter (D1); the OMEX↔Hearth join | honest: spec ≠ built |
| **Speculative** | ASI-scale numbers; ASI-proposed invariants; out-of-band observation at high capability | future work, correctly gated behind the current build |

And the robustness point that makes the contribution stronger than it looks: **the
governance half doesn't wait for the cognition half.** Hearth's regime works over
*opaque* harnesses today — that was the black-box doctrine, proven by the connectors. If
OMEX ships, oversight gets cheaper still. If OMEX never ships, the shape-verification
scaling law stands alone on behavior. **The stack degrades gracefully in both directions**
— which is the property you want in foundational infrastructure.

---

## 55. The fractal thesis

The property that makes the whole stack cohere, named:

> **In Hearth, "harness" is not a privileged boundary — it's a role that anything can
> play if it satisfies five elements.** The module contract was built recursion-ready
> from day one: host mode lets an external harness register *into* the vest as a loop
> module; meta mode lets Hearth wrap one; a loop module's `step()` may itself run a full
> kernel instance. A harness of harnesses is not a special case. It's the same object,
> composed. And the atom — **one contract, one loop, four verdicts, one checkpoint, one
> tick** — fits in a single function. Below it there is no control, only behavior.
> Everything above it is the same five elements composed.

### The power law that explains the industry

```
checklist        → zero power (artifact, no loop)
loop             → power at ONE level (control, no substrate, no composition)
loop + graph     → power across levels (composition, verification, recovery)
fractal stack    → power that scales WITH composition
```

A checklist is a dead artifact. A loop over a checklist is live control *at one level* —
where essentially the entire industry lives (L-obj, sometimes L-self-meta). What it
cannot do: verify a sub-harness's handoff, propagate a verdict across a boundary, roll
back one branch without the others, resimulate only the affected region, or be checked
by anything outside it. Those are all *composition* properties, and no amount of looping
harder at one level produces them. They require the substrate. **"Breaking that down to
scale" is not a future pivot — it's what artifacts #1–9 constructed.**

### The legibility inversion

| Property | Monolithic scaling | Fractal (graph-native) scaling |
|---|---|---|
| **Legibility** | *decreases* with capability — bigger core, more entanglement | constant per atom; **increases** with composition (growth adds declared boundaries) |
| **Verification cost** | grows with capability; dies past verifier capability | grows with *boundary count* — mechanical, shape-checkable |
| **Failure scope** | global | regional — per-branch rollback, boundary reconciliation only |
| **Resimulation** | O(entire system) | O(affected region + dependency closure) |
| **Trust** | all-or-nothing in the core | per-boundary, evidence-based, probe-verified |
| **Governance** | one observer must out-think the monolith | composition of small checks; recursion terminates at frozen contracts |

Read the legibility row twice: **under monolithic scaling, growth destroys transparency.
Under fractal scaling, growth *creates* structure.** A system that grows by composing
atoms becomes *more* auditable as it becomes more capable, because every act of
composition adds contracts, checkpoints, and boundaries to the graph — and all three are
mechanically checkable. **The sign of the scaling law flips.**

### What breaks the fractal — and the ratchet

**The fractal is only as trustworthy as its most adversarial child.** A sub-harness can
lie about its ticks. The answer is the *other* recursion — the one that terminates:
composition can descend in principle without limit; **verification descends with it but
terminates** — each level's substrate checked by the level above (probes, schemas,
invariants), bottoming out at the governance plane: frozen contracts plus independent
verification, not a bigger watcher. The two recursions are matched: unbounded
composition downward, bounded verification terminating in the plane. As capability
rises, the termination point *hardens* rather than changes: signed telemetry,
out-of-band observation — the observer must live beyond the observed's reach, at every
level, including the top.

---

## 56. The four faces & the viability verdict

### The four faces, ranked honestly

| Face | What it actually is | Status |
|---|---|---|
| **Benchmark** | The product. The only harness-independent measurement of control capacity that exists. | Novel — the unclaimed territory |
| **Data** | The asset. Every submitted trace is a point in the factor space; the corpus compounds with every run. | Compounds; the longer it runs, the more valuable |
| **Marketing** | The growth loop. Harness vendors have *status incentive* to show good control numbers; users have *comparison incentive* to run the wizard. | Real funnel: record → passage review → simulate → compare |
| **Enforcement** | The proof. Enforcement makes the benchmark credible — the same engine, gated, demonstrates the findings were real. | Technology validation, not the business |

These aren't competing purposes — §9 established they're one control path with different
release authority. But the **ranking matters for prioritization**: benchmark quality is
the product; everything else falls out of it. A benchmark with sloppy findings is dead
in all four faces simultaneously.

### The viability verdict

**Worth validating for research? Yes.** The contribution stands even if every product
ambition never lands:

1. First harness-independent **control-capacity** benchmark — the gap is real; SWE-bench
   measures outcomes, agent benchmarks measure completion, nothing measures the control
   layer across harnesses.
2. The **paired shadow/enforced methodology** — the Hearth Delta is a measurement
   design, not just a metric.
3. The **tiered counterfactual discipline** — refusing to overclaim simulation validity
   is rare and publishable in itself.
4. **The corpus** — see §57.

This gives the project a **floor**: even in the worst case where adoption stalls, the
research artifact and the methodology are real.

**The one critical risk — loop precision.** Every downstream number is only as good as
the loops producing findings. False positives don't just annoy users — they poison the
benchmark's validity, because the Hearth Delta, drift series, and factor deltas all rest
on findings being real. The mitigations, all structural: loop precision **published,
versioned, calibrated** (the calibration study — native harness as fully-instrumented
reference, measuring how much observable-shadow metrics can be trusted per envelope
class — is not optional polish, it's the benchmark's own validity check); probes for
capabilities extended to precision for loops (a loop ships with its measured precision
and improves or gets deprecated); and the build order — contracts → connectors → probes
→ kernel → metrics → service — was correctness-first by necessity.

**What it is not — say this as often as needed:** not a model benchmark (model
capability is one axis in the factorial dataset); not an agent framework competitor (it
owns "did obligations hold," not "who is the agent"); not a replacement for internal
evals — it's the *cross-harness comparison layer* internal evals structurally cannot be;
not dependent on vendor cooperation — the connectors work observationally; cooperation
is a bonus, not a prerequisite.

**For harness builders — the control profiler pitch:** shadow audit of your own harness
surfaces findings you didn't know you generated, scoped by tool/modality — a
control-point map built from behavior. The depth vector is your control character. Drift
across releases tells you whether a refactor improved or regressed your control layer.
Cost-to-correction shows where your cheap control is expensive. Tune control loops blind,
or tune them against an instrument.

### The triangulation — alignment, drift, cost, one instrument

```
ALIGNMENT   — contract compliance per tick: did it stay on the assignment?     (§14)
DRIFT       — same contracts, release over release: is it getting better/worse? (§25)
COST        — what alignment and drift cost: the Pareto frontier               (§24)
```

A harness can be aligned but expensive (over-verifying), cheap but drifting, or
aligned-and-cheap (the good corner). Single-score benchmarks can't distinguish these.
All three derive from the same tick records, which is what makes the comparison coherent
rather than three unrelated scores. **Hearth is a natural validation of exactly that
triangle.**

---

## 57. The data asset

The corpus this compiles is genuinely novel:

- **Control-annotated agentic traces**: session graphs with findings, verdicts, what
  *would* have been gated (shadow), what actually got gated (enforced), and
  counterfactual outcomes per policy — with declared fidelity and tier. Nobody has this.
  The nearest analogues are process-supervision datasets and agent-trajectory datasets,
  neither of which carries the control dimension.
- **What it's good for**: harness improvement (the builders' own use), eval-set
  construction, control-behavior research, and — the training angle — training and
  evaluating models on control-relevant behavior (when to verify, when to backtrack,
  what re-reading buys). **Control becomes a trainable skill with a ceiling-free
  evaluation**: a reward model checking shape never needs to out-think the agent it
  scores. The consent + redaction pipeline (server-verified in #8's admission, not CLI
  best-effort) is what makes any of it usable at all.
- **The dogfooding loop closes on itself**: Hearth's own development — selftests as
  specifications, schemas as contracts, honest limitations in every artifact — is the
  code-quality discipline the project preaches. The native harness (#15) means Hearth
  runs its own control loops through its own benchmark: both the calibration reference
  and the credibility proof. **A benchmark that can't measure its own harness has no
  business measuring anyone else's.** And the codebase's quality is load-bearing for the
  data's credibility: schemas validated, selftests green, limitations documented — that
  is what lets a submitted bundle be trusted. The project being clean *is* part of the
  data-ingestion claim.

---

## 58. The UI/UX specification

The supervisor isn't just an outermost loop — it's a product surface. Everything below
sits on one rule: **every supervisor action is itself a tick** — the UI is inside the
control path, not beside it.

**Capture-time:** recording chip with live envelope (*"Hearth capturing · prompts ✓
responses ✓ attachments (meta) ✗ reasoning ✗"* — the honesty panel follows you during
the session); **file-interception grant prompt** at selection time (*"Capture full
contents (byte-exact) or metadata only?"* — this single choice is the difference
between a replayable passage and a hole-riddled one; explicit, never silent); per-tick
findings toasts per notification policy; running tick counter.

**Passage Review (pre-simulation, mandatory):** every item with fidelity badge; opaque
items shown as holes, not hidden; redact/remove/annotate purpose/mark delivery
constraints; fidelity summary + the claim it licenses (*"2 byte-exact, 1 referential, 1
opaque — cross-harness comparisons exclude 1 item"*); consent + freeze → hash.

**Simulation Matrix:** target columns (harness × model) showing the verified capability
block, envelope class, ingestion-map preview (*"codex receives files in-repo"*),
eligibility status, estimated cost. Run all → live progress per column.

**Compare View (the flagship screen):** same passage, N harnesses side by side —
outcome, turns, tokens, cost, findings, `user_effort_to_correct`, coverage %,
first-access timeline overlay; retrieval timeline for observable harnesses rendered as
the graph it is; click any cell → Tick Inspector (full envelope, decision, findings,
impact subgraph); the **Savings Panel** — per finding in the original: *"detected at
tick 47; caused by ctx_3, opaque on Claude Web; codex caught the equivalent at first
read (tick 12). Estimated savings: 38 ticks, $1.20"* — grounded in the economics
decomposition, never hand-waved; replay-vs-original baseline shown per harness —
variance before virtue.

**Supervisor web app:** **Findings Inbox** (pending holds with severity,
countdown-to-timeout, one-key approve/deny+reason — over `holds.list`/`decide`); **Session
Timeline** (per-tick waterfall with branch visualization — edits, rewinds, fork-replays
as first-class graph navigation forward and backward through message states); **Tick
Inspector**; **Capture-envelope panel** (the A8 matrix rendered — *"for ChatGPT web we
see prompts + responses + tool cards, not reasoning, not diffs; missed-context figures
for this session are lower bounds for that reason"* — the honesty layer as UI);
**Pre-Continue Brief** (end-of-turn digest the user must see before the run continues:
violations detected, what was skipped, corrections that *would* apply under ENFORCE,
estimated cost-to-correct; user chooses: continue / enforce from here / halt / amend
contract — the choice itself a traced tick); **Contract manager** (versioned amendments
triggering the full-chain replay visualization).

**Mobile:** push (severity-routed via the notification engine), approve/deny cards, HALT
resolution, escalation digests, live run status.

**Notification engine:** findings → users with **policy** — per-tick, per-turn,
per-phase, session digest, severity threshold, scope filters (per loop/factor);
channels: in-app websocket, mobile push, email, webhook (CI); dedup, rate limits, quiet
hours; every notification + user response traced. Notification granularity is a policy
object, not a hardcoded behavior — otherwise supervision demand (Invariant #16) loses
its denominator.

**Onboarding — the "Benchmark me" wizard:** pick host → guided connector install
(install-hooks / gate-run / extension) → **probe suite runs live, showing your verified
capability matrix** → first shadow workload with live dashboard → optional enforce run
for the Hearth Delta → submit bundle → appears in population distributions. **The
marketing site *is* this flow** — the funnel and the product are the same object.

**Live benchmark dashboard:** runs in progress via websocket — live tick counters,
verdict distribution, AITD streaming, cost accrual, per-factor deltas as they land;
comparison view across harnesses on the same workload; population distributions from
the aggregation engine.

---

## 59. Errata & findings about ourselves

A laboratory that never finds anything wrong with its own harness isn't calibrating.
Two findings shipped as errata, in the open:

**E1 — Scheduler speculative dispatch (found by #9's lab).** The original `_eligible`
required a speculative task's target to be already `dispatched` — but dispatch happens
*after* frontier selection, so a speculative task could never co-dispatch with its
target. Fix: the frontier construction now appends eligible speculative tasks whose
targets are in-frontier or dispatched. Resolution semantics unchanged; #7b's assertions
hold under the fix.

**E2 — Replay depth is insensitive under cumulative checkpoint manifests (found by the
lab on day one).** Every checkpoint snapshots all touched files, so R=1 (newest
checkpoint) already sees everything, and the WEAKENED rung (R_high=1) equals PRESENT on
the core workload. That is a real property of our own stack, surfaced as data, with a
named fix: **interval manifests** (checkpoint diffs covering only the interval since
the prior checkpoint) — the daemon TODO that makes the replay factor real. The lab
working on its own harness is the calibration loop running.

**E3 — `_events` glue in the codex gate client** referenced undefined aliases; fixed in
#4's errata with probe-flag pass-through, alongside the fake app-server's
ordering-robust `wait_response` (the steer request can precede the approval response —
a race the original fake encoded as an expectation).

**E4 — one dead placeholder line** in the kernel's `_index` before the
`_index_entity` delegation — cosmetic debt, flagged inline, first cleanup candidate.
Noted so nobody trusts it as a pattern.

---

## 60. Remaining backlog

Continuing from #10, in dependency order:

| # | Item | Depends on |
|---|---|---|
| **#10** | **Supervisor API + notification engine** — findings inbox over `holds.list`/`decide`, policy-routed notifications, every notification + response traced | probes (substrate exists) |
| **#11** | **Supervisor web app** — inbox, timeline/branches, tick inspector, envelope panel, pre-continue brief | #10, A2 (done), A8 (done) |
| **#12** | **Live benchmark dashboard** — websocket streaming, comparison view, distributions | #10, #8 (done) |
| **#13** | **Browser extension adapter** — observer transport, file-interception grant, conversation surgery | A8 (done), CPP (done) |
| **#14** | **Mobile supervisor app** | #10 |
| **#15** | **Native harness (D2)** — maximum capture envelope, calibration reference, scheduler-backed execution, native spans | everything above |
| #16 | Onboarding wizard + docs site | #11–13 |
| #17 | Blender / reference adapters / shadow runners | kernel (done) |
| #18 | Storage hardening (durable store, checkpoint backends), auth/quotas, CI wall, packaging, community registries | #8 |
| — | Interval checkpoint manifests (E2 fix — makes the replay factor real) | — |
| — | OMEX↔Hearth join interface | OMEX build |
| — | Out-of-band observation + signed telemetry | governance hardening |
| — | Lab: statistical inference (repetitions with variance) | real-model arms |
| — | Auto-replanning from plan-fidelity gaps | #7b |
| — | Per-task loop sets in the scheduler | #9 per-level ablation |

Rationale for the order: **#10–12 make it visible and act-on-able** (and #11's
timeline/envelope work needs only A2+A8, both done — it can interleave with anything);
**#13–15 widen the capture surface**, with the native harness deliberately after the
browser extension so the max-envelope connector lands against an already-honest
envelope registry; **#16 is the funnel**; the theory sections (§46–55) are settled
enough to build against — nothing in them redirects the roadmap, they sharpen it.

---

## The compression — the whole document, final form

> **An MD checklist has zero power; a loop over it has all the power; the loop's ticks,
> on one substrate, compose into the fractal — harnesses of harnesses with boundaries
> made of ticks, governed by a plane of frozen contracts rather than a bigger watcher.
> Hearth is that substrate's smallest laboratory: it captures movement, not cognition;
> measures the behavior of control, not its mechanism; prices what skipping costs;
> replays one capture under many policies with tiered honesty; pairs shadow against
> enforcement so external control's contribution is a number; and benchmarks the
> traversal, the strategy, the factors, and the width — all from the same trace its own
> enforcement produces. Capability verification asks the verifier to out-think the
> worker and dies exactly where oversight matters most; shape verification reads
> relations, stays flat in cost, and fails loudly — so the oversight that scales past
> the verifier's ceiling is the one already running: nine artifacts, ~260 selftests,
> the intercept points public, the laboratory calibrating on itself.** The intelligence
> can scale without bound; the structure that governs, verifies, and replays it scales
> with the graph.

---

## Capture cross-check — every insight from every round, located

| Insight | Where it lives |
|---|---|
| Checklist → contract → DAG arc | §2, §40, §53 |
| Unenforced state / configured ≠ demonstrated / cross-harness gap / **movement gap** | §1 (four gaps) |
| Three identities, five roles, two contributions, release authority, substrate, closing formulation | §3 |
| Rolling horizon, dynamic R (5 conditions, graph-extended), four verdicts, BACKTRACK phases, checkpoints | §4 |
| Architecture with traversal layer + composition loops | §5 |
| Tick: event/iteration identity, 8 steps, non-rewrite, event-time | §6 |
| Module contract, repair ownership | §7 |
| Levels locked, dmax + dissent, nesting (+1 boundary cost) | §8, §39 |
| Three capabilities, five postures, shadow-as-instrument, legacy ladder, ownership boundary | §9 |
| Capability matrix, two declarations, posture derivation, browser adapter | §10, §37 |
| Flash loops + tool.gate + composition.span | §11 |
| All registries incl. traversal + factor + resolution rules + portability | §12 |
| Config: capabilities, audit flavor, traversal overrides | §13 |
| Contracts: structural binding, versioning | §14 |
| Graph: 10 fields, two readings, example event, software-agnostic, substrate-not-log | §15 |
| Context traversal: per-tick flow, impact subgraph, R bounded | §16 |
| Session: ownership, record, never-collapsed chain, per-session traces | §17 |
| API surface: four contracts, TraversalStrategy, primitives, ownership split, evidence substrate, three adaptation layers, freeze discipline | §18 |
| Connector: one call, envelope, 31 kinds, decision wire, three transports, verdict→mechanics, probe, durability, surgery | §18, §37 |
| AITD, nine depths, per-depth metrics, context family, control profile, correction vs delta, layered measurement | §19 |
| Overhead, tiered review | §20 |
| Factor space: definition, four properties, seven domains, ladder, factorial dataset (+composition +scheduler dims) | §21, §44 |
| R sweep, external-only, Hearth Delta, factor deltas, interactions, strategy sweep, one-model path, fingerprint, auto-vs-approval | §22, §42 |
| Baseline, eligibility, one-model→many, behavioral distribution | §23, §42 |
| Economics: decomposition, cutting corners, context economics, back-simulation, three tiers | §24, §41, §42 |
| Drift: series, attribution, lifecycle, correction, persistence | §25, §44 |
| Supervisor: decisions, four types, trust model, supervision demand | §26 |
| Kernel contract: 8 frozen statements, diagram | §27 |
| Quick start — all real commands | §28 |
| Invariants 1–35 | §29 |
| All limitations incl. connector-specific and replay-insensitivity | §30 |
| Prior art solid list + full survey | §31, §47 |
| Roadmap: done + remaining | §32, §60 |
| FAQ expanded (~40 entries) | §33 |
| Glossary (~130 entries) | §34 |
| **Nine artifacts, status, decisions, selftests** | §36–45 |
| CC/Codex surfaces, transports, gate clients, information surface, surgery, ingestion endpoints | §37 |
| Kernel decisions, builtin loops, tick record | §38 |
| Atom, spans, nested graph, composition.span, recursive checkpoints, matrix | §39 |
| Bernstein, waves, critical path, verdicts at merge, speculation, fidelity, parallel profile, serial bounds | §40 |
| Black-box mode, meta_effectiveness, economics impl, missed impl, boundaries impl, calibration, envelopes | §41 |
| Admission (5 stages), tier-1 ops (4), CFX semantics, delta, eligibility, HTTP | §42 |
| CPP: passage, fidelity, reintroduced_at, progression, machinery taxonomy, machinery metrics, user_effort_to_correct, intervention protocols | §43 |
| Lab: registry, workloads, live ladder results, epistasis, drift persistence, C3 | §44 |
| Selftest inventory (~260 checks) | §45 |
| Meta-loop: three levels, six boundaries, tool movement layer, self-meta monitoring | §46 |
| Four graphs, per-ecosystem L-obj, self-meta plateau, L-ext-meta borders, Hermes calibration note, three structural observations | §47 |
| Black-box doctrine: shadows table, kept/lost, control profiler, behavior-vs-mechanism | §48 |
| Dual-review ledger, posture symmetry, simulated Hearth Delta, third party | §49 |
| Governance plane: four answers, planes-not-layers, membership test edit | §50 |
| Four planes diagram | §51 |
| Cost laws, level-by-level audit (L0–LG), three asymmetries, B1–B4, arms-race relocation, trainable control | §52 |
| S1–S14 with bounds, master law, checklist→DAG, serial bounds, build demands | §53 |
| OMEX: two layers, three joins, three laws with anchors, six ASI capabilities, contribution ledger, graceful degradation | §54 |
| Fractal: atom, power law, legibility inversion, adversarial-child limit + ratchet | §55 |
| Four faces, viability, loop-precision risk, profiler pitch, triangulation | §56 |
| Data asset, training angle, dogfooding | §57 |
| UI/UX: capture chip, passage review, matrix, compare view, savings panel, inbox, timeline, inspector, envelope panel, pre-continue brief, mobile, notifications, wizard, dashboard | §58 |
| Errata E1–E4 | §59 |
| Backlog #10+ with dependencies | §60 |

**Verdict: nothing dropped.** All thirty-five original sections preserved and extended;
all nine implementation artifacts captured with their decisions, mechanics, and
selftests; every theoretical round — meta-loop, survey, black-box doctrine, dual-review
ledger, governance plane, four planes, cheap-verification audit, scale factors, OMEX
join, fractal thesis, four faces, viability, data asset, UI spec, errata — integrated as
first-class sections with invariants carrying the load-bearing claims (now 35) and the
glossary carrying the vocabulary (now ~130). The README is no longer a promise with a
design attached: it is a design whose smallest laboratory is already running, and whose
own laboratory has already graded its first findings — including the ones about itself.
