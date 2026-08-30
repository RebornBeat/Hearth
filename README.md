<div align="center">

# Hearth

### The Control-Loop Harness — Enforcement Kernel, Control-Factor Substrate & Harness-Independent Benchmark

**Enforcement over artifacts. Measurement and enforcement are the same control path with different release authority.**

</div>

> **An MD checklist has zero power. A regressive control loop over that checklist has all
> the power. Without the loop, the checklist is a wish. With the loop, it is an enforced
> contract — the harness re-reads it, re-verifies it, and drags the model back on track
> based on findings.**

*(working name — rename freely; the identity is the contract, not the label)*

---

## Table of Contents

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
16. [Metrics, for free](#16-metrics-for-free)
17. [Overhead accounting](#17-overhead-accounting)
18. [The control-factor space — taxonomy & planes](#18-the-control-factor-space--taxonomy--planes)
19. [Benchmark: harness power === traversal power](#19-benchmark-harness-power--traversal-power)
20. [Control economics — cost to correction](#20-control-economics--cost-to-correction)
21. [Drift — detection, attribution, correction, recurrence](#21-drift--detection-attribution-correction-recurrence)
22. [The supervisor — the control plane](#22-the-supervisor--the-control-plane)
23. [The kernel contract — frozen](#23-the-kernel-contract--frozen)
24. [Quick start](#24-quick-start)
25. [Design invariants](#25-design-invariants)
26. [Limitations](#26-limitations)
27. [Prior art](#27-prior-art)
28. [Roadmap](#28-roadmap)
29. [FAQ](#29-faq)
30. [Glossary](#30-glossary)
31. [License](#31-license)

---

## 1. Why this exists

The execution stack today:

```
MODEL  →  HARNESS  →  TRAJECTORY  →  ENVIRONMENT
```

- Model benchmarks ask: *did you know the answer?*
- Agent benchmarks ask: *did you finish the task?*
- Trajectory evals ask: *how efficiently did you move?*
- Almost nothing asks: ***did your control actually hold?***

Two failures nobody handles together:

1. **Unenforced state.** Harnesses everywhere store state — plans, checklists, goals,
   recaps — and almost none of them *enforce* it. A stored plan that nothing re-verifies
   is a wish with good formatting.
2. **Configured ≠ demonstrated.** `max_iterations = 50` says nothing about capability.
   A harness *allowed* fifty iterations is not a harness that *demonstrates* fifty levels
   of useful traversal. Control capacity should be enforced and measured — not configured
   and assumed.

Hearth exists to close both gaps with one object.

**And the third gap — the reason Hearth cannot be baked into one harness:**

3. **No way to compare control across harnesses.** If the control machinery lives inside
   one agent, you can measure that agent. If it is external and adaptable, you can measure
   **the harness itself** — across Codex, Claude Code, ChatGPT, custom harnesses, browser
   agents, any host — under identical contracts, identical tick semantics, identical
   verdicts, identical trace, identical metrics. That is the difference between
   benchmarking a product and benchmarking a category.

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

**1. Native harness.**
Runs standalone. `/goal` → `/subtask` → execute → hooks → replay → verify → `/recap` → done.
A complete harness in itself.

**2. Meta harness.**
Sits **on top of** other harnesses via adapters. The host harness emits events; the
adapter maps them to canonical loop events; the enforcement core returns verdicts.
One interface for every host.

**3. Loop host — the vest.**
A **kit of flash-in control loops**: modality loops, tool loops, scenario loops,
verification loops, and anything you register. You should never be restricted to one
pass-through control loop — there are too many places loops belong, so this is a vest,
not a single strap.

### The five roles — what it is

> **The binding** — artifact to loop, always. Nothing stored unbound.
> **The machine** — re-reads, re-verifies, repairs, backtracks.
> **The vest** — flashes more control in without becoming a monolith.
> **The instrument** — measures the same thing it enforces.
> **The heart** — the organ other harnesses are missing; everything gets built around it.

The identities are how you deploy it. The roles are what it is. Both are true
simultaneously, and neither changes the other.

And the heart metaphor is load-bearing, not decorative: enforcement-over-state is the
organ most harnesses lack — which is exactly why other harnesses can be built *around*
it, integrate *with* it, and register *into* it without changing what it is. It adapts
cleanly into any harness, native or meta, as an add-on, a library, or an API.

### The two contributions

1. **The enforcement kernel** — binding obligations to loops, evaluating at host-event
   boundaries, gating through four verdicts.
2. **The canonical traversal graph** — enforcement history as a reusable substrate rather
   than something thrown away after a verdict. The trace is not a log; it is the
   measurement layer ([§15](#15-the-canonical-traversal-graph)) — and it is what makes
   the control-factor space measurable from one instrument
   ([§18](#18-the-control-factor-space--taxonomy--planes)).

### Why benchmarking tool and meta harness are one product

They are the same mechanism viewed from two directions:

> **Measurement and enforcement are the same control path with different release
> authority.**

- Do not release the verdict → **benchmarking** (observe and measure the control loop).
- Gate the verdict → **meta-harness** (participate in the control loop, enforce contracts).
- Escalate the verdict → **supervisor** (delegate release authority upward).

The distinction is not architectural. It is the degree of authority the deployment
grants Hearth. That is also why something good enough to benchmark control can enforce
control: it is the same engine, same events, same contracts, same trace — different
authority.

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
> for the control-factor space** ([§18](#18-the-control-factor-space--taxonomy--planes)).

Loops enforce factors. The taxonomy names what the loops measure. The trace records both.

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
| Architectural change | full dependency chain |
| Contract version change | full dependency chain |
| Critical failure | replay from last stable checkpoint |

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
rewinding (forward-repair only). The checkpoint's compensating-action descriptor and the
repair plan together declare exactly which phases execute. Declaring them separately is
what makes `BACKTRACK(d)` honest across heterogeneous hosts.

### Checkpoints

`BACKTRACK(d)` and "replay from last stable checkpoint" are defined against an explicit
checkpoint record. A checkpoint is:

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
│  │ · … flash in more, any time               │    │
│  └───────────────────────────────────────────┘    │
│                                                   │
│  ┌───────────────────────────────────────────┐    │
│  │ STATE STORE  (storage-agnostic)           │    │
│  │ · goals · subtasks · checklists · recaps  │    │
│  └───────────────────────────────────────────┘    │
│                                                   │
│  ┌───────────────────────────────────────────┐    │
│  │ TRACE OUT → canonical traversal graph     │    │
│  └───────────────────────────────────────────┘    │
└───────────────────────────────────────────────────┘
```

The state store holds the artifacts. The registry holds the loops. The core binds them.
The trace falls out.

---

## 6. The tick — the atomic unit

Hearth runs **on top of every prompt** a harness emits. Every prompt, tool call, or
state change is one **tick** — the atomic unit of enforcement, measurement, and trace.
There is no coarser or finer unit.

This makes "iteration" exact: **an iteration is one tick.** Every replay, every
verification, every trace record happens at tick boundaries. AITD is ticks per level —
the metric and the mechanism share a unit.

### Event identity vs iteration identity — stated precisely

Host events differ wildly in granularity: a prompt is heavy, a state change can be
tiny. Hearth resolves this by keeping two identities strictly separate:

- **Event identity** — every host event is exactly one tick. No coalescing, no splitting.
  One event in → one pipeline run → one tick record out. This is the invariant, and it
  holds regardless of event size.
- **Iteration identity** — a host "turn" or "step" may span *many* ticks. Hearth never
  counts host turns; every metric, including AITD, is counted in ticks.

The atomicity claim is therefore about the *pipeline*, not about events being equal in
size: **one event → one pipeline run → one tick record.** Granularity differences
between event classes are real, recorded (event class is a tick field), and never
allowed to blur the count.

### The per-tick pipeline

Fixed for every tick, native or meta, regardless of how many loops are loaded:

```
TICK
 1. INTERCEPT   host emits event (prompt / tool call / state change)
 2. RESOLVE     registry resolves which loops fire this tick
                (scope match + trigger match + composes_with check)
 3. STEP        each resolved loop steps — cheap checks first,
                deep replay only on trigger
 4. AGGREGATE   one verdict per tick — worst verdict wins:
                HALT > BACKTRACK(dmax) > REPAIR > CONTINUE
 5. EMIT        one trace record per loop + one tick record
 6. RELEASE     CONTINUE releases the tick to the host;
                anything else holds, per verdict
```

The pipeline never changes when loops are added. A new loop adds checks to step 3 —
never ticks, never stages. **Addition is monotonic.** Flash in a modality loop, a tool
gate, a scenario policy: same ticks, same rhythm, more eyes per tick. This is why
addition never destabilizes the host.

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
implements the same contract. This is why stacking stays clean as the registry grows:

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

Loops nest by **serialization level**. To treat AITD as a benchmark metric, what
constitutes a level must be locked, so it is:

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
in the canonical graph ([§15](#15-the-canonical-traversal-graph)), including revisits.
AITD is indexed by **serialization level**. The canonical graph records **traversal
depth**. Both are kept, neither substitutes for the other.

Precedence rules keep the stack coherent:

- **Outer loop owns ordering. Inner loops yield.**
- The **supervisory agent is the outermost loop** — it can raise replay depth or force a
  backtrack on anything below it.
- `composes_with` rejects incompatible stacks at registration time, not at 3 AM.

### Verdict aggregation — the tie-break, made explicit

When multiple loops fire on one tick, worst verdict wins:

```
HALT  >  BACKTRACK(d)  >  REPAIR  >  CONTINUE
```

When two loops return `BACKTRACK` with **different depths**, the aggregation rule is:

> **dmax wins.** The maximum required depth is the most conservative finding, and
> conservatism is the direction disagreement must degrade in. (A lesser depth is
> recorded in the trace as a dissenting finding — it is not lost, it is just not
> the gate.)

This is an invariant, not a convention
([Invariant #12](#25-design-invariants)).

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

There are three distinct powers, and they are not the same:

| Power | Meaning |
|---|---|
| **Observe** | Hearth sees what happened |
| **Intercept** | Hearth sees it *before* it happens |
| **Gate** | Hearth can prevent it from happening |

A browser extension may observe a conversation but be unable to prevent an action. A CLI
integration may intercept tool calls. A native adapter may have full synchronous gating.
Modes are chosen per deployment, within what the adapter's capabilities permit
([§10](#10-adapter-capabilities--the-declared-boundary)).

### The control path — five postures

The compliance ladder expands into the full control path. Every posture is the **same
pipeline** ([§6](#6-the-tick--the-atomic-unit)) with different release authority:

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
| **OBSERVE** | synchronous, step 3 bypassed | trace only | *what happened?* |
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
violations detected
violations ignored
violations self-corrected
violations propagated
eventual recovery
irreversible violations
supervision required
```

Enforced audit asks a different question:

> **How does the harness perform when a control layer actively prevents invalid
> progression?**

That quantifies controlled capability. Run both and the difference between them is
itself a benchmark — the **Hearth Delta**
([§19](#19-benchmark-harness-power--traversal-power)).

### The legacy ladder — mapped, not replaced

The earlier three-level compliance ladder remains valid as a shorthand for what a host
can honor, now derived from declared capabilities rather than assumed:

| Level | Name | Requires | Maps to |
|---|---|---|---|
| 2 | **ENFORCE** | gate + resume capabilities | ENFORCE / AUTO or SUPERVISED |
| 1 | **AUDIT** | intercept capability, no pause | AUDIT / SHADOW or ACTIVE |
| 0 | **OBSERVE** | observe capability only | OBSERVE |

A host that cannot pause still gets full shadow enforcement: the identical pipeline
runs asynchronously, every violation is recorded, and enforcement is carried out by the
outermost loop instead of inline. **No host is worth zero enforcement.** An
uncooperative host gets shadow enforcement — not none.

### Native, meta, host — unchanged

**Native:**

```
/goal → /subtask → execute → hooks → replay(R) → verify → /recap → next
```

Runs as its own harness. Nothing else required.

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

That's the entire integration surface. Same pattern for every host.

**Non-interference by design:** the surface is small (events + four verdicts + pause),
the contract is one, outer owns ordering, and Hearth reads ticks without rewriting them
([the non-rewrite guarantee](#6-the-tick--the-atomic-unit)).

**Host — other harnesses wear Hearth:**

External harnesses register **into** the vest as loop modules. Their native controls
become loop steps; their uniqueness is preserved behind the one contract. A harness with
a special planner, a special memory, a special tool router — register it, scope it, and
it stacks with everything else.

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

This is what prevents Hearth from slowly becoming "a framework that happens to support
Codex, Claude Code, ChatGPT, Blender, etc." Hearth stays the canonical protocol; hosts
are adapters; the benchmark data stays comparable because the canonical control
semantics stay fixed while the host implementation changes.

### The capability matrix

Every adapter declares exactly what it has:

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
the benchmark configuration** — it ships with every trace, so any comparison between
harnesses knows exactly what each adapter could and could not do.

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
    supervise: true        # escalates to supervisor surfaces
```

Hearth doesn't lie about the difference. It records it.

### Posture is derived from capabilities, validated at registration

`posture = f(capabilities, policy)`. A deployment requesting ENFORCE on an adapter
without `gate` + `resume` fails at registration with a precise message — not at 3 AM,
not silently. Requesting AUDIT/SHADOW always succeeds anywhere `observe` exists.

### The browser extension — a first-class adapter

The browser client is not a special product. It is an **observation/interception
boundary** speaking the same canonical event protocol:

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

Depending on what the browser environment permits, it maps into canonical events:

```text
conversation turn · model response · tool/action proposal · user intervention
approval/rejection · visible state transition · timing · attachments · navigation
```

Then the same control machinery applies. The benchmark isn't "a benchmark for ChatGPT."
It is:

> **A harness-independent control benchmark — and ChatGPT is one adapter.**

---

## 11. Flash control loops

A **flash loop** is a control loop packaged as a swappable module — flash it into the
vest at runtime, per modality, per tool, per scenario, per level.

First in, by design:

| Loop type | Scope | Enforces |
|---|---|---|
| **Modality loops** (first-class, first shipped) | `modality` | modality-specific ordering, horizons, and verification — vision, code, docs, audio, … |
| Tool loops | `tool` | per-tool constraints: read-before-write, confirmation gates, retry policy |
| **Tool gate** (`tool.gate`) | `tool` | **irreversibility gate** — actions tagged irreversible (delete, force-push, external send) require a fresh `CONTINUE` on the replay window before dispatch, or explicit supervisor confirmation |
| Scenario loops | `scenario` | per-task policies: research protocols, SWE-style verify-before-commit |
| Verification loops | `verification` | replay windows, audit passes, regression sweeps |
| **Yours** | anything | any loop matching the contract |

`tool.gate` is the concrete shape of a tool loop: one scope, one trigger, one check —
and it makes the difference between an agent that *did* something irreversible and one
that *verified before* doing it.

The registry is open. New modality, new tool, new scenario = new module, same interface,
same enforcement core, same trace format. The stack grows without degrading — addition
is monotonic, never a new cadence.

**Loops enforce factors.** Each loop is the executable embodiment of one or more control
factors ([§18](#18-the-control-factor-space--taxonomy--planes)) — `tool.gate` enforces
the *irreversibility* factor, `verification.replay` enforces the *replay* factor. The
taxonomy names what the loops measure; the loops are how factors get teeth.

---

## 12. The registries

The vest is a **routing table**, not a costume. Loops are addressable by **scope key**
and resolve at tick time.

### Structure

```
REGISTRY
├── harness registry     registered hosts: adapter, capabilities, posture,
│                        attached bundles
├── modality registry    loops keyed by modality scope     (first-class)
├── tool registry        loops keyed by tool scope
├── scenario registry    loops keyed by task/scenario scope
└── bundle registry      named sets of loops — kits
```

Per-harness, per-modality, per-tool, per-scenario — register a single control loop, or
register a whole set of control loops **around a registered harness**. Each harness is
unique; each modality is unique; the registry honors both without special cases.

### Resolution rules

A loop fires on a tick iff **all** hold:

1. **Scope match** — the tick's tags include the loop's scope key
   (`harness:`, `modality:`, `tool:`, `scenario:`)
2. **Trigger match** — the tick's event type matches the loop's trigger
3. **Composition holds** — the loop's `composes_with` permits the active set

Firing order is deterministic: declared priority, then registration order.
No nondeterministic stacks.

### Bundles — kits

A **bundle** is a named set of control loops, registered once and attached to any
registered harness:

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

### Portability

Because every loop speaks canonical events — never host events — **a bundle attached to
one harness attaches to any other harness unchanged.** Bundles are the unit of
ecosystem reuse: you share enforcement profiles across harnesses, not integrations per
harness. That is the flash economy.

### Where modality contracts bind

The per-modality registry is where multi-modal contracts land. A modality contract
binds to its modality loop, which binds to its registry scope — three names for one
chain, all resolved at tick time. No unbound contract, now at the routing layer too.

---

## 13. Harness registration

Register harnesses around Hearth, and harnesses into Hearth, from config — including
the capability declaration that makes every comparison honest:

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

loops:
  - id: modality.vision
  - id: tool.filesystem
  - id: tool.gate
  - id: scenario.swe-style

registered_harnesses:           # other harnesses, registered as loops
  - id: planner-x
    as: loop
    scope: scenario
  - id: memory-y
    as: loop
    scope: verification

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
control it.

---

## 14. Contracts

Hearth carries **multi-modal contracts** in the state store, each bound to a loop by
construction:

| Contract | Binds | Verified by |
|---|---|---|
| **Goal contract** | desired endpoint, constraints, completion conditions | master loop — every iteration |
| **Subtask contract** | scope, dependencies, verification conditions | orchestrator + verification loops |
| **Modality contract** | modality-specific constraints | modality loop |
| **Tool contract** | tool usage rules | tool loop |

A contract without a loop is a document. In Hearth, there is no way to store an unbound
contract — the storage layer and the loop registry are coupled by design. Every stored
obligation has a loop that drags execution back to it based on findings.

### Binding is structural, not conventional

"No unbound contract" is enforced **in the data model**, not merely by registration
discipline:

- The contract record's schema **requires** a `bound_loop` field — a foreign key into
  the loop registry.
- A write referencing a loop id that does not resolve **fails validation at write
  time**. The store physically cannot persist an unbound contract.
- Deleting or unloading a loop **fails while any contract references it** — obligations
  cannot become orphaned by the registry changing underneath them.

The strongest idea in the design should not depend on anyone remembering the rule.

### Contract versioning

Contracts are **versioned**. The version in force is recorded in every checkpoint
([§4](#4-the-core-loop)). A mid-run contract change is itself a **finding** — and it
raises R to the full dependency chain, reusing the existing dynamic-R rule rather than
inventing new machinery. The contract that changed, and everything downstream of it,
gets replayed before the run proceeds.

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

**Structural:**

```
Scene → Collection → Object → Mesh → Face
```

**Traversal:**

```
Collection → Object → Material → Object → Mesh → Face
        └────────── revisit ──────────┘
```

Traversal depth is 6 even where structural depth is 3. Revisits, replays, and
backtracks are preserved, not collapsed. This distinction is why "depth" in Hearth means
something no file tree can express — and why traversal depth (graph path length,
including revisits) and serialization level (registry nesting) are reported as separate
quantities ([§8](#8-stacking--precedence--levels-locked)).

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

The canonical schema never assumes the host's data model:

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
drift attribution ([§21](#21-drift--detection-attribution-correction-recurrence)),
benchmarks ([§19](#19-benchmark-harness-power--traversal-power)), the supervisor's view
([§22](#22-the-supervisor--the-control-plane)), and the control-factor space
([§18](#18-the-control-factor-space--taxonomy--planes)). The verdict consumes the tick;
the graph keeps everything.

---

## 16. Metrics, for free

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
  "depth_0": 12,
  "depth_1": 28,
  "depth_2": 25,
  "depth_3": 8,
  "max_depth": 3,
  "total_iterations": 73
}
```

AITD is reported as **distributions per level (p50 / p95), not just means** — a harness
that alternates between shallow and deep traversal is different from one that sits at
the average, and the mean hides that.

### The depth family — eight depths, one vector, never collapsed

Hearth distinguishes eight depths, and refuses to collapse them into one number:

| Depth | Meaning |
|---|---|
| **Traversal depth** | path length in the canonical graph, including revisits |
| **Serialization level** | registry nesting — what AITD indexes |
| **Replay depth (R)** | how many prior checkpoints each step re-verified |
| **Correction depth** | how far backward a correction propagated |
| **Backtrack depth** | checkpoints returned in a backtrack |
| **Dependency depth** | how deep the affected dependency chain reached |
| **Synchronization depth** | how far back state was re-synced before advancing |
| **Verification depth** | how deeply verification examined |

Reported per run as the **depth vector**:

```json
{
  "traversal": 41,
  "serialization": 3,
  "replay_used": [3, 3, 7, 3],
  "correction": [2, 5],
  "backtrack": [2],
  "dependency": 9,
  "synchronization": [4],
  "verification": [3, 3, 7]
}
```

The shape of the vector *is* the harness's control character: a harness that verifies
shallowly but synchronizes deeply is a different machine from one that does the reverse,
and no single number can express that difference
([Invariant #17](#25-design-invariants)). The economics of each depth behavior is priced
separately in [§20](#20-control-economics--cost-to-correction).

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

Run the identical workload through different harnesses and compare:

```
            D0    D1    D2    D3    TOTAL
Harness A   12    28    25     8       73
Harness B   12    17     6     0       35
Harness C   12    41    63    27      143
```

That is a concrete, structured, judge-free measurement of control-loop traversal
capacity — and it falls out of normal operation.

### The control profile — violations as a measurement set

Because every tick is verified, violations are counted, not imagined:

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

Because Hearth is external, the same task decomposes:

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

## 17. Overhead accounting

Per-tick enforcement has a cost, so it is measured like everything else. Enforcement is
never free, and pretending otherwise would break the honesty of the whole instrument.

| Metric | Meaning |
|---|---|
| `enforcement_overhead_per_tick` | latency added by steps 2–4 |
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

## 18. The control-factor space — taxonomy & planes

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

### The taxonomy — six control domains

```text
                         CONTROL FACTOR SPACE
                                  │
        ┌──────────────┬──────────┼──────────┬──────────────┐
        │              │          │          │              │
    PLANNING        TOOLING     STATE     RECOVERY     VERIFICATION   SUPERVISION
        │              │          │          │              │              │
  ┌─────┼─────┐   ┌────┼────┐  sync      detect      frequency      escalation
horizon  replan  order gate    stale-     correction  depth          intervention
dep.pres.        verify irrev.   state      depth       coverage       unresolved
                 …      checkpoint integrity replay eff. false-positive rate
```

| Domain | Factors |
|---|---|
| **Planning control** | horizon · replanning · dependency preservation |
| **Tool control** | ordering · gating · verification · irreversibility |
| **State control** | synchronization · stale-state detection · checkpoint integrity |
| **Recovery control** | detection latency · correction depth · replay efficiency · recovery success |
| **Verification control** | frequency · depth · coverage · false-positive burden |
| **Supervision control** | escalation rate · intervention rate · unresolved findings |

Each domain, each factor: **independently benchmarkable.**

### The ablation ladder — the core experimental design

This is the question the factor space exists to ask — about *every* factor:

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
the measurement of what that factor is actually *worth*.

### The factorial dataset — the benchmark output

Every observation is a point in the factor space:

```text
(model=M1,
 harness=H1,
 factor=tool.verify,
 level=L2,
 modality=code,
 tool=filesystem,
 R=3,
 checkpoint_policy=rolling,
 supervision=none)
        ↓
      outcome
        ↓
     trace
```

The dimensions:

```text
MODEL · HARNESS · CONTROL FACTOR · CONTROL LEVEL · MODALITY · TOOL · TASK
REPLAY DEPTH · CHECKPOINT DEPTH · VERIFICATION DEPTH · SUPERVISION
STATE-SYNC POLICY · MEMORY POLICY
```

Which means the benchmark database is not a leaderboard. It is a **factorial dataset** —
queryable, sliceable, recombinable. Any question of the form *"what does factor F
contribute at level L under posture P?"* is a query, not a new benchmark.

Factor definitions are registry objects — machine-readable, addressable like loops,
versioned like contracts — so the taxonomy itself is part of the infrastructure, not a
page in a paper ([Roadmap](#28-roadmap)).

---

## 19. Benchmark: harness power === traversal power

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

*(illustrative numbers — the point is the axis, not the values)*

The interesting result is never "more replay = better." It is the **Pareto frontier** —
the replay depth where the reliability gain stops being worth the cost. For Hearth,
adaptive R lands near full-trajectory reliability at a fraction of the cost.

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

Cross harnesses × models on the same sweep and you answer the question single-score
agent benchmarks cannot:

> **Is the model actually more capable — or is its harness giving it more effective
> traversal and control capacity?**

Capability should be reported at the model × harness configuration level. Hearth makes
that not just possible but automatic.

### Shadow vs enforced — the Hearth Delta

Run every harness twice — once shadow, once enforced — and the difference becomes a
first-class benchmark:

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

Harness A may have a much weaker native control loop but benefit enormously from
external enforcement. Harness B already has strong internal controls. Ordinary agent
benchmarks expose none of this — both might post similar single scores.

The **Hearth Delta** is the answer to: *how much of this harness's apparent capability
is actually external control?*

### Factor deltas — the Hearth Delta, decomposed

The aggregate delta is only the headline. Because every finding is attributed to a loop,
and every loop enforces a factor ([§11](#11-flash-control-loops),
[§18](#18-the-control-factor-space--taxonomy--planes)), the delta decomposes per factor:

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

Factors do not act independently. Paired and joint runs expose interactions:

```text
verification × replay          +17pp
checkpoint × state-sync        +14pp
tool-gate × verification       +12pp
```

An interaction larger than the sum of its parts means two factors compound — the harness
gets more from verification *when replay backs it* than either buys alone. This is how
the field moves from "agents need control" to **"these control factors matter, these
interact, these are dead weight for this harness"** — the factor ablation ladder
([§18](#18-the-control-factor-space--taxonomy--planes)) is the instrument.

### One model is enough — the expansion path

The machinery does not need a grid to be proven. It needs one pair:

```text
Model A + Harness A + Hearth
  → establish: trace → factors → interventions → correction → cost → counterfactuals
```

Once that works, the grid

```text
Model A × Harness A     Model B × Harness A     Model C × Harness A
Model A × Harness B     Model B × Harness B     Model C × Harness B
```

becomes primarily an **adapter and data-acquisition problem** — not a new benchmark
architecture. That is the payoff of Hearth being external: the same captured corpus
supports paired shadow/enforced runs per harness, Hearth as common referee, and "verify
against many" as a property of the data model rather than a re-engineering effort.

### The control profile — the behavioral fingerprint

Per harness, per run, per release:

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

The same harness may run autonomous or with human approval gates. Hearth benchmarks
**both**, and measures **how much supervision the harness actually needs**:

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

## 20. Control economics — cost to correction

Hearth's traces make the *economics* of control measurable — what correction costs,
when it is worth it, and what corners actually cost when they are cut. This is the
layer that turns control from a virtue into a priced quantity.

### The cost-to-correction decomposition

Every finding decomposes:

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
    detection cost
  + synchronization cost
  + rollback cost
  + replay cost
  + re-execution cost
  + downstream invalidation cost
```

And the law underneath all of it:

> **The cost depends on when you detect the problem.**

### The price of cutting corners

Consider the same defect under two verification policies:

```text
Correct control:                        Cheap control:

T1 ─ T2 ─ T3 ─ VERIFY ─ T4 ─ T5         T1 ─ T2 ─ T3 ─ T4 ─ T5 ─ T6 ─ T7 ─ VERIFY
                  ↑                                                         ↑
              catches error                                            catches same error
```

The cheap harness saved verification work. And paid:

```text
early detection:   correction depth 1 · replay 1 · downstream invalidation 1
late detection:    correction depth 5 · replay 5 · downstream invalidation 4
```

This converts a vague virtue into a measured quantity — the **economic value of
verification**:

> Not "did it verify?" but **"how much did insufficient verification eventually cost?"**

That is a substantially better benchmark question, and it is answerable only because
detection depth, replay, synchronization, and invalidation are all traced per tick.
Cost-cutting on control is no longer invisible — it is a debt, and the trace is the
ledger.

### Counterfactual back-simulation — one run, many policies

One real execution captures everything needed to ask:

> **What would have happened if the control policy had been different?**

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

The trace becomes a **counterfactual control laboratory** — the error-to-correction
relationship measured *with* intervention and *without* it, from the same recorded
behavior.

### Counterfactual validity — three tiers, declared

Counterfactual validity depends on what the recorded environment contains. Hearth
therefore tiers every counterfactual and records the tier with the result:

| Tier | Name | Question | Validity |
|---|---|---|---|
| 1 | **State-bound** | given captured state, would policy P have detected / prevented X? | computable from the capture — sound |
| 2 | **Checkpoint re-execution** | re-run from a checkpoint under policy P | sound when the environment is captured / deterministic from that point |
| 3 | **World-generation** | what would a *different model* have done downstream? | **not simulation** — requires live re-execution; the capture cannot know an unbuilt world |

No simulation pretends to be more valid than it is. **Tier is a field on every
counterfactual result** ([Invariant #18](#25-design-invariants)). This distinction is
what keeps the back-simulation laboratory scientifically honest instead of quietly
overclaiming.

### One capture, many policies — per-model, per-policy correction data

Because tier-1 counterfactuals are pure functions of the captured trace, **one recorded
run re-scores against many control policies** — and the paired shadow/enforced design
extends the same capture across harnesses where the capture supports it:

```text
                    REAL RUN DATA
                         │
                         ▼
                  CANONICAL TRACE
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
     Policy A         Policy B         Policy C
   correction       correction       correction
     profile          profile          profile
```

Shadow mode is what makes this honest: the harness behaves unassisted, Hearth records
what *would* have intervened — so the intervention / no-intervention contrast is
**measured, not assumed**. Per-policy correction cost, per-depth inefficiency, and the
price of skipped synchronization all fall out of the same paired records
([§19](#19-benchmark-harness-power--traversal-power)).

Detect, attribute, correct, measure, simulate, price — **one engine, six outputs.**

---

## 21. Drift — detection, attribution, correction, recurrence

Drift tracking is **not another subsystem**. It is a natural consequence of the core:
Hearth observes control behavior → deviations become findings → findings trigger the
existing verdict/repair/replay machinery → the resulting history is the benchmark.

### Drift is a time series

Instead of "Harness A = 72%," the same contracts re-run across harness releases give:

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

Because every tick is attributed to a scope and every finding to a loop, drift
decomposes for free:

```text
Harness A
│
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
drift detector is pointed at the harness; the loops *are* the detector. Drift
attribution and factor attribution are the same mechanism viewed at different
granularities ([§18](#18-the-control-factor-space--taxonomy--planes)).

### The drift lifecycle

```text
DETECT DRIFT
      ↓
ATTRIBUTE DRIFT
      ↓
CORRECT DRIFT        ← reuses REPAIR / BACKTRACK / replay — no new machinery
      ↓
MEASURE CORRECTION
      ↓
TRACK WHETHER DRIFT RETURNS
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

A new harness release regresses:

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

Detect, attribute, correct, measure, track recurrence — **one engine, five outputs.**

---

## 22. The supervisor — the control plane

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

When Hearth produces a non-CONTINUE verdict that policy routes upward, the supervisor:

- **Resolves `HALT`** — resume, repair, or abandon.
- **Approves or rejects `tool.gate`** — irreversible actions can require explicit
  supervisor confirmation.
- **Force-backtracks** any loop below it.
- **Raises R** on anything below it — between ticks.
- **Amends contracts** — as versioned changes, which automatically trigger full-chain
  replay ([§14](#14-contracts)).
- **Answers clarification requests** — when a loop's finding is ambiguous, the tick can
  surface a question, not just a verdict.

### Who the supervisor is

The supervisor does not have to be human:

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

And supervision is **measurable**:

### Supervision demand is a capability dimension

Two agents, same task:

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

## 23. The kernel contract — frozen

Everything in this document deploys or composes the following. This is the part every
feature must prove itself against, and it is frozen:

```text
Tick → Loop resolution → Step → Verdict aggregation → Checkpoint/repair → Trace → Release
```

The actual shape:

```text
                 CONTRACTS
                     │
                     ▼
HOST ──► TICK ──► RESOLVE ──► LOOPS ──► VERDICT
                     │                    │
                     │                    ▼
                     │              REPAIR / BACKTRACK
                     │                    │
                     ▼                    ▼
                  TRACE ◄──────────── CHECKPOINTS
                     │
                     ▼
                 METRICS / AITD / FACTORS
```

The frozen statements:

1. **The pipeline is fixed.** One event → one pipeline run → one tick record. Loops add
   checks; nothing adds ticks.
2. **The verdict protocol is closed.** `CONTINUE / REPAIR / BACKTRACK(d) / HALT` —
   worst verdict wins, `dmax` on competing backtracks, nothing else escapes the core.
3. **The capability boundary is declared.** Hearth defines what must be observable, what
   may be intercepted, what may be gated, and what verdicts mean; the adapter declares
   what the host actually provides; the declaration ships with every trace.
4. **The trace is the substrate.** Every tick is recorded into the canonical traversal
   graph; every metric — **including every control factor** — is derived from it;
   nothing is measured outside it.
5. **Release authority is the only variable.** Observe, audit (shadow/active), enforce
   (auto/supervised) — same path, different authority. Benchmarking and meta-harnessing
   are one mechanism.

Everything else — native mode, meta mode, host mode, modality loops, tool loops, scenario
loops, bundles, adapters, registries, the browser extension, the supervisor surfaces,
drift reporting, control economics, counterfactual simulation — is **how you deploy and
compose this kernel**.

---

## 24. Quick start

*(illustrative API — shape is stable, names may move)*

### Native

```python
from hearth import Hearth

h = Hearth(config="hearth.config.yaml")
h.goal("/goal: ship the refactor with zero regressions")
result = h.run()            # enforces ordering, replays, backtracks as findings demand

print(result.aitd)          # [12, 28, 25, 8]
print(result.verdicts)      # every CONTINUE / REPAIR / BACKTRACK / HALT, in order
```

### Meta — wrap an existing harness

```python
from hearth import Hearth, Adapter

h = Hearth(mode="meta", compliance="enforce", adapter=Adapter.for_host("my-agent-harness"))
h.attach_bundle("vision-swe", to="my-agent-harness")

result = h.run(host=MyAgentHarness())   # your harness runs; Hearth enforces
```

### Benchmark a public agent — shadow

```python
from hearth import Hearth, Adapter

h = Hearth(mode="meta", compliance="audit", audit_enforcement="shadow")
h.attach_adapter(Adapter.for_host("browser:chatgpt"))

result = h.run(workload="swe-bench-lite", seed=7)
print(result.control_profile)
# native_violation_rate, correction_rate, control_delta, supervisor_interventions, ...
```

The same run with `compliance="enforce"` gives the enforced profile — and the
difference is the Hearth Delta.

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

### Read the trace

```bash
hearth trace   --run R001 --format canonical-graph
hearth metrics --run R001 --metric aitd
hearth depths  --run R001                      # the full depth vector, all eight
hearth drift   --harness codex --baseline v1.0 --compare v1.4
hearth replay  --run R001 --depth 3            # reconstruct state at any traversal depth
```

### Factor analysis, economics, counterfactuals

```bash
hearth factors  --run R001 --report deltas        # per-factor shadow→enforced deltas
hearth cost     --run R001 --finding F12          # cost-to-correction decomposition
hearth simulate --run R001 --policy R=7 --tier state-bound   # back-simulation, tier declared
hearth ablate   --factor tool.verify --state absent --workload swe-bench-lite
```

---

## 25. Design invariants

1. **No artifact is trusted without a loop over it.**
2. **Every loop is a module.** One contract, no exceptions.
3. **Every iteration emits a trace record.** No silent steps.
4. **Outer owns ordering; inner yields.**
5. **Only the four verdicts escape the core.** Nothing else crosses the boundary.
6. **R is dynamic — and recorded per step.**
7. **Native, meta, host: zero architectural change between modes.**
8. **Hearth does not compete for "who is the agent" — it owns "did the stored
   obligations still hold."**
9. **Stopping is always a verdict, never a silent end.** Every loop has a soft budget;
   exhaustion surfaces as `HALT` to the supervisor — never a quiet fade.
10. **Hearth reads ticks; it never rewrites them.**
11. **The pipeline is fixed; loops add checks, never ticks.** Addition is monotonic.
12. **Worst verdict wins — and on competing `BACKTRACK` depths, `dmax` wins.** Many
    reviewers, one gate; disagreement degrades toward caution, never toward optimism.
    Dissenting findings are recorded, not discarded.
13. **Measurement and enforcement are the same control path with different release
    authority.** One engine; the deployment chooses the authority.
14. **Hearth never assumes how a host executes.** The adapter declares what the host
    actually provides — and the declaration is part of every trace and every benchmark
    configuration.
15. **No unbound contract is structurally representable.** Binding is enforced by the
    data model at write time, not by convention.
16. **Supervision demand is a first-class measurement.** How much external control a
    system requires to remain correct is a capability dimension, reported per run.
17. **The depth family is never collapsed.** Eight depths, one vector — collapsing them
    destroys exactly the information the benchmark exists to produce.
18. **Every counterfactual declares its tier.** State-bound, checkpoint re-execution, or
    world-generation — no simulation pretends to be more valid than it is.
19. **Every control factor is ablatable.** Any factor can be present, absent, weakened,
    strengthened, or enforced — and the ablation is itself a traced run.
20. **Hearth is the substrate, not a factor.** It measures and enforces the
    control-factor space; it is not one axis within it.

---

## 26. Limitations

Stated plainly, because a harness that demands honesty from its hosts should show some:

- **Adapters must be real.** The canonical graph is only as good as each host's adapter.
  Unmaintained adapters mean degraded traces, and degraded traces mean weaker replay.
- **Adaptive-R rules are policy.** The dynamic-R table is a sensible default, not a law
  of nature. It must be tuned per domain, and bad policy produces bad backtracks.
- **Replay costs tokens and latency.** Overhead is measured
  ([§17](#17-overhead-accounting)), not hidden. Cheap-tier checks are cheap; deep replays
  are not.
- **ENFORCE requires gate + resume capabilities.** Hosts without them degrade along the
  control path — to AUDIT (shadow/active) or OBSERVE. That is a documented degradation
  path, not a free pass, and the degradation itself is recorded in the trace.
- **Browser adapters are bounded by the environment.** What a web client can observe,
  intercept, or gate is limited by what the platform permits; capability declarations
  will honestly contain `partial` and `false`, and cross-host comparisons must respect
  them.
- **Supervisor quality bounds enforcement quality.** The outermost loop acts on
  findings; garbage findings produce garbage backtracks. An AI supervisor is itself a
  system under test — treat it as one.
- **Shadow findings are not prevention.** Shadow audit detects and records; it does not
  stop the violation from having happened. Irreversible damage in shadow mode is real
  damage — that is precisely the information shadow mode exists to capture.
- **Counterfactuals are tiered — and tier 3 is not simulation.** State-bound
  counterfactuals are computable from the capture; re-execution is sound only over
  captured / deterministic environments; world-generation (a different model producing a
  different downstream world) requires live runs. Every counterfactual result ships
  with its tier, and tier-3 claims must be backed by execution, not replay.
- **The factor space is combinatorial.** Factors × levels × postures × R × models ×
  harnesses explodes quickly. The taxonomy exists to make sampling principled, not
  exhaustive — reported results must state which cells of the factor matrix they cover.
- **Design-stage numbers are illustrative.** The R-sweep, Hearth-Delta, and factor-delta
  tables show the axes, not earned results. The values are claims to be earned by the
  benchmark suite ([§19](#19-benchmark-harness-power--traversal-power)), not facts yet.

---

## 27. Prior art

The full surveyed landscape — including hedged, verify-before-citing names — lives in
[`docs/PRIOR_ART.md`](docs/PRIOR_ART.md). The solidly citable pieces:

- **ReAct** — interleaved reasoning and acting; the base agent loop.
- **MPC / receding-horizon control** — plan, execute a portion, re-plan on new state.
- **Tree of Thoughts / search over reasoning paths** — branch, evaluate, backtrack.
- **Backtracking search** — return to an earlier decision point on downstream failure.
- **Checkpoint + replay engineering** — save points, revalidate from them.
- **Factorial experiment design** — factors, levels, interactions, ablation; the
  statistics the control-factor space imports.

What did not appear as one named object: enforcement-over-artifacts as the *identity*
of a harness, no-unbound-contracts by construction (structural, in the data model),
dynamic R as both first-class control and benchmark axis, dual wrap/be-wrapped under one
module contract and four verdicts, a flash vest of stackable loops with explicit
precedence, **the Hearth Delta as a benchmark of external-control contribution**,
**the control-factor space as a factorial benchmark substrate with ablation ladders and
interaction measurement**, **counterfactual back-simulation over control policy with
declared validity tiers**, **cost-to-correction as a decomposed benchmark metric**,
supervision demand as a capability dimension, and AITD plus the canonical traversal
graph as free measurement of harness power. The pieces existed. The binding did not.

---

## 28. Roadmap

- [ ] Reference adapters — files, shell, browser
- [ ] 3D adapters — Blender first (canonical graph is already software-agnostic)
- [ ] Browser extension adapter — ChatGPT / Claude / Z.ai / Grok clients
- [ ] Adapter capability validator — posture derivation + registration-time validation
- [ ] AITD reporter CLI + trace viewer (p50/p95 distributions per level)
- [ ] Control profile reporter — violation rates, control delta, supervision demand
- [ ] R-sweep benchmark suite — harness × model × replay depth
- [ ] Shadow/enforced paired benchmark runner — automatic Hearth Delta computation
- [ ] Drift time-series store — per-release baselines, attribution, recurrence tracking
- [ ] **Control-factor registry** — machine-readable factor taxonomy, addressable like loops
- [ ] **Factor ablation runner** — present / absent / weakened / strengthened / enforced sweeps
- [ ] **Counterfactual back-simulation engine** — tier-1 state-bound first; tier-2 over captured environments
- [ ] **Cost-to-correction reporter** — detection / sync / rollback / replay / re-execution / invalidation decomposition
- [ ] **Factor interaction analyzer** — pairwise and joint deltas from paired runs
- [ ] **Depth-vector reporter** — all eight depths per run, never collapsed
- [ ] Flash-loop registry — community loops, scoped and composable
- [ ] Bundle registry — portable enforcement profiles across harnesses
- [ ] Compliance-ladder adapters — AUDIT shadow runners for pause-less hosts
- [ ] Checkpoint store backends — snapshot refs + compensating-action descriptors
- [ ] Supervisor web app — findings inbox, clarification requests, contract amendments
- [ ] Supervisor mobile app — approvals, HALT resolution, escalation notifications
- [ ] Supervisor API — AI / hybrid / policy supervisor integrations
- [ ] Supervisory scheduler presets — periodic overview agent as outermost loop

---

## 29. FAQ

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
(Invariant #20). The factors are verification, gating, synchronization, replay,
escalation; Hearth is the instrument that prices them.

**What is a control factor?**
An independently addressable control behavior — verification, gating, synchronization,
replay, escalation — that can be present, absent, weakened, strengthened, or enforced,
produces measurable intervention, and is fully traced. Loops enforce factors; the
taxonomy names what the loops measure.

**Can one recorded run really benchmark many control policies?**
Tier-1 counterfactuals, yes — state-bound questions ("would R=7 have caught this?") are
computable from the capture. Re-running from checkpoints (tier 2) works over captured
environments. Asking what a *different model* would have done (tier 3) is not
simulation — that requires live execution, and Hearth says so rather than pretending
(Invariant #18).

**What does "cost to correction" include?**
Detection, synchronization, rollback, replay, re-execution, and downstream invalidation
— decomposed per finding. Its headline law: cost depends on *when* you detect the
problem, which is why "the economic value of verification" is a measurable quantity
here rather than a slogan.

**Why eight depths instead of one number?**
Because collapsing them destroys the measurement. Traversal, serialization, replay,
correction, backtrack, dependency, synchronization, and verification depth describe
different behaviors; the depth vector reports all eight, and the difference between
harnesses lives in its shape (Invariant #17).

**Does it replace my harness?**
No. It can wear your harness or be worn by it. Both directions are first-class
integrations. It does not compete for "who is the agent."

**Why not just build this into one agent?**
Because then you could only measure that agent. External means you can measure *the
harness* — across Codex, Claude Code, ChatGPT, custom hosts — under identical contracts,
ticks, verdicts, and metrics. Comparability is the reason for the architecture — and it
is also why one proven model/harness pair is enough to start: expansion becomes data
acquisition, not re-architecture.

**Why isn't the checklist the feature?**
Because a checklist is storage. The loop over it is the power. The checklist ships —
the loop is the product.

**Why "flash"?**
Loops are packaged to be flashed in and out of the vest at runtime — swappable,
stackable, per modality, tool, or scenario. You never lock into one pass-through loop,
and adding one never changes the cadence.

**What is a tick?**
The atomic unit. Every prompt, tool call, or state change the host emits is one tick,
and one tick runs the fixed six-step pipeline: intercept, resolve, step, aggregate,
emit, release. One event → one pipeline run → one tick record. A host "turn" may span
many ticks; Hearth counts ticks, never host turns.

**What if my harness can't pause?**
Declare its real capabilities. It gets AUDIT — shadow or active — the identical pipeline
in shadow, every violation recorded, reported to the supervisor, enforced from the
outermost loop where possible. No host is worth zero enforcement.

**What's the difference between shadow and active audit?**
Shadow records findings and lets the host continue. Active audit records findings *and*
files actionable ones the supervisor or outer loop can act on between ticks. Shadow
asks "what went wrong?" Active asks "what should be done about it?"

**What is the Hearth Delta?**
The measured difference between a harness's shadow (unassisted) success and its enforced
(controlled) success — reported alongside correction rate, and decomposable into
per-factor deltas and interaction effects. It answers: *how much of this harness's
apparent capability is external control — and which control is doing the work?*

**Who executes a repair?**
The loop that owns the failing artifact. Hearth schedules it and re-verifies with the
same R window that caught the finding.

**What happens when two loops disagree on the same tick?**
Worst verdict wins; on competing `BACKTRACK` depths, the maximum required depth wins
(`dmax`). Dissenting findings are recorded in the trace. Disagreement degrades toward
caution, never toward optimism.

**What is drift, and is it a separate system?**
Drift is the change in a harness's control behavior over releases — and it is not a
separate system. Detection, attribution, correction, measurement, and recurrence
tracking all reuse the existing tick → finding → verdict → repair → trace machinery.

**What's the difference between a temporary error and persistent drift?**
Recurrence. The same finding, repaired and returning across runs or releases, is
persistent drift — a structural control deficiency, not bad luck. `persistent_drift` and
`recurrence_count` are first-class metrics.

**Who is the supervisor — human or AI?**
Whatever the deployment makes it: human (mobile/web/API), AI (a supervised control loop
itself), hybrid (AI proposes, human confirms), or pure policy. Each is a benchmarkable
configuration, and supervision demand is measured for each.

**Can I use the browser extension with ChatGPT/Claude/Grok?**
That is the plan: the browser client is a first-class adapter — an observation and (where
the platform permits) interception boundary — speaking the same canonical protocol. The
benchmark is harness-independent; each public agent is one adapter.

**Why the name?**
A hearth is the heart of a structure — everything else gets built around it.
Enforcement-over-state is the organ other harnesses are missing.

**What does it measure?**
Whatever it enforces. Ticks per level, the eight-depth vector, blast radius, active
sets, traversal coverage, its own overhead — plus violation rates, control delta,
factor deltas, interactions, cost to correction, counterfactuals, supervision demand,
and drift. The instrument and the enforcer are the same object.

---

## 30. Glossary

| Term | Definition |
|---|---|
| **Control-Loop Harness** | master rolling-horizon loop + registry of sub loops; makes stored state enforceable |
| **Enforcement kernel** | the frozen core: binds obligations to loops, evaluates at host-event boundaries, gates through four verdicts, records the canonical trace |
| **Tick** | the atomic unit — one host event run through the fixed six-step pipeline; an iteration is one tick |
| **Event identity / iteration identity** | every event is exactly one tick; a host turn may span many ticks — metrics count ticks, never host turns |
| **Flash loop** | swappable control-loop module, flashed into the vest at runtime; the executable embodiment of one or more control factors |
| **The vest** | the loop registry — a kit, not a strap; a routing table of scope keys |
| **Bundle** | a named, portable set of control loops, attachable to any registered harness unchanged |
| **Scope key** | the address of a loop's jurisdiction — `harness:`, `modality:`, `tool:`, `scenario:` |
| **Serialization level** | a loop's nesting depth in the registry, declared at registration — fixed for the run; what AITD indexes |
| **Traversal depth** | path length in the canonical traversal graph, including revisits — distinct from serialization level |
| **Depth vector** | the eight depths — traversal, serialization, replay, correction, backtrack, dependency, synchronization, verification — reported per run, never collapsed |
| **Replay depth (R)** | how many prior checkpoints each step re-verifies; dynamic |
| **Forward horizon (H)** | how far ahead each step plans |
| **AITD** | Agentic Iterative Traversal Depth — ticks per serialization level, reported as p50/p95 distributions |
| **Blast radius** | downstream decisions invalidated by a correction |
| **Correction depth** | how far backward a correction propagated |
| **Checkpoint** | id + trace position + contract versions in force + state snapshot ref (or compensating-action descriptor) |
| **Verdict** | `CONTINUE` / `REPAIR` / `BACKTRACK(d)` / `HALT` |
| **Worst verdict wins** | per-tick aggregation — `HALT > BACKTRACK(dmax) > REPAIR > CONTINUE`; competing depths resolve to the maximum; dissents recorded |
| **Backtrack phases** | rewind state / repair consequences / replay decisions — declared separately in the repair plan |
| **Release authority** | what the deployment lets Hearth do with a verdict: trace it (observe), file it (audit), gate it (enforce), escalate it (supervise) |
| **Control path** | OBSERVE · AUDIT(SHADOW \| ACTIVE) · ENFORCE(AUTO \| SUPERVISED) — one pipeline, five postures |
| **Shadow audit** | evaluate without affecting execution — the raw-capability benchmark instrument |
| **Active audit** | evaluate, record, and file actionable findings for the supervisor / outer loop |
| **Hearth Delta** | enforced success − shadow success; the measured contribution of external control (reported in pp, distinct from correction rate) |
| **Correction rate** | proportion of behavior Hearth corrected — distinct from control delta |
| **Control profile** | the behavioral fingerprint: violation rates, repair/backtrack/escalation rates, persistent drift, recovery, overhead |
| **Control factor** | an independently addressable control behavior — present/absent/weakened/strengthened/enforced — producing measurable intervention, fully traced |
| **Control-factor space** | the full factor × level × posture × configuration space Hearth instruments; the benchmark's factorial substrate |
| **Substrate** | Hearth's role relative to the factor space — it measures and enforces factors; it is not one of them (Invariant #20) |
| **Ablation ladder** | the five states a factor is tested in: ABSENT / WEAKENED / PRESENT / STRENGTHENED / ENFORCED |
| **Factor delta** | the shadow→enforced success delta attributed to a single factor |
| **Interaction effect** | the measured joint deviation of two factors from the sum of their individual deltas — which controls compound |
| **Factorial dataset** | the benchmark output — every observation a point in the factor space, queryable rather than ranked |
| **Back-simulation** | counterfactual replay of a captured trace under a different control policy |
| **Counterfactual tier** | declared validity class of a simulation: state-bound / checkpoint re-execution / world-generation |
| **Cost to correction** | detection + synchronization + rollback + replay + re-execution + downstream invalidation, decomposed per finding |
| **Detection depth** | how long a defect survived before detection — the variable correction cost hinges on |
| **Economic value of verification** | the measured downstream cost avoided by verification at a given depth — pricing the corners cut |
| **Adapter capability** | declared host power: observe / intercept / gate / resume / inject / snapshot / compensate / supervise — `true`/`partial`/`false`, shipped with every trace |
| **Non-rewrite guarantee** | Hearth gates the release of ticks; it never edits them — diffable, therefore testable |
| **Canonical traversal graph** | software-agnostic record of every traversal event; the substrate for checkpoints, replay, drift, factors, economics, benchmarks, and supervision |
| **Adapter** | maps a native harness/environment onto the canonical schema |
| **Contract** | a stored obligation bound to a loop — structurally incapable of being stored unbound; always versioned |
| **Drift** | change in control behavior over releases — detect, attribute, correct, measure, track recurrence |
| **Persistent drift** | a finding that recurs after repair across runs/releases — a structural deficiency, not an error |
| **Supervision demand** | how much external control a system requires to remain correct — a first-class capability dimension |
| **Supervisor** | the control plane above the kernel — human, AI, hybrid, or policy; ultimate policy authority; every supervisor action is itself a traced tick |

---

## 31. License

MIT
