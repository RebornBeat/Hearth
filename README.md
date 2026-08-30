<div align="center">

# Hearth

### The Control-Loop Harness

**Enforcement over artifacts.**

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
3. [What it is](#3-what-it-is)
4. [The core loop](#4-the-core-loop)
5. [Architecture](#5-architecture)
6. [The tick — the atomic unit](#6-the-tick--the-atomic-unit)
7. [The module contract](#7-the-module-contract)
8. [Stacking & precedence](#8-stacking--precedence)
9. [Three modes, one compliance ladder](#9-three-modes-one-compliance-ladder)
10. [Flash control loops](#10-flash-control-loops)
11. [The registries](#11-the-registries)
12. [Harness registration](#12-harness-registration)
13. [Contracts](#13-contracts)
14. [The canonical traversal graph](#14-the-canonical-traversal-graph)
15. [Metrics, for free](#15-metrics-for-free)
16. [Overhead accounting](#16-overhead-accounting)
17. [Benchmark: harness power === traversal power](#17-benchmark-harness-power--traversal-power)
18. [Quick start](#18-quick-start)
19. [Design invariants](#19-design-invariants)
20. [Limitations](#20-limitations)
21. [Prior art](#21-prior-art)
22. [Roadmap](#22-roadmap)
23. [FAQ](#23-faq)
24. [Glossary](#24-glossary)
25. [License](#25-license)

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

## 3. What it is

One sentence:

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
                HALT > BACKTRACK(d) > REPAIR > CONTINUE
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

## 8. Stacking & precedence

Loops nest by **serialization level**:

```
L0  orchestrator (master loop)
L1    modality loops
L2      tool / scenario loops
L3        verification loops
```

Precedence rules keep the stack coherent:

- **Outer loop owns ordering. Inner loops yield.**
- The **supervisory agent is the outermost loop** — it can raise replay depth or force a
  backtrack on anything below it.
- `composes_with` rejects incompatible stacks at registration time, not at 3 AM.

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

## 9. Three modes, one compliance ladder

### Native

```
/goal → /subtask → execute → hooks → replay(R) → verify → /recap → next
```

Runs as its own harness. Nothing else required.

### Meta — Hearth wears another harness

```
HOST HARNESS ──native events──► ADAPTER ──canonical loop events──►
ENFORCEMENT CORE ──verdict──► HOST HARNESS
```

A host harness has exactly three obligations:

1. Emit events (step done, tool call, file write, state change).
2. Accept verdicts (`CONTINUE | REPAIR | BACKTRACK(d) | HALT`).
3. Pause on `REPAIR` / `BACKTRACK` / `HALT` until released.

That's the entire integration surface. Same pattern for every host.

**Non-interference by design:** the surface is small (events + four verdicts + pause),
the contract is one, outer owns ordering, and Hearth reads ticks without rewriting them
([the non-rewrite guarantee](#6-the-tick--the-atomic-unit)).

### Host — other harnesses wear Hearth

External harnesses register **into** the vest as loop modules. Their native controls
become loop steps; their uniqueness is preserved behind the one contract. A harness with
a special planner, a special memory, a special tool router — register it, scope it, and
it stacks with everything else.

**Both directions are first-class.** Hearth wraps harnesses. Harnesses wrap Hearth.
It is the heart either way.

### The compliance ladder

Obligation 3 is not always available. Hosts differ in what they can honor, so
compliance is a **declared spectrum**, not a hidden assumption. The per-tick pipeline
is identical at every level — only its timing and gate change:

| Level | Name | Host behavior | Pipeline timing | Gate at step 6 |
|---|---|---|---|---|
| 2 | **ENFORCE** | pauses on non-CONTINUE | synchronous | blocks the tick on non-CONTINUE |
| 1 | **AUDIT** | cannot pause | asynchronous (shadow) | records the verdict, reports to supervisor |
| 0 | **OBSERVE** | trace-only | synchronous, step 3 bypassed | trace only |

A host that cannot pause still gets **full shadow enforcement**: the identical pipeline
runs asynchronously, every violation is recorded, and enforcement is carried out by the
outermost loop instead of inline. **No host is worth zero enforcement.** An
uncooperative host gets shadow enforcement — not none.

### The ownership boundary

> **Hearth does not compete for "who is the agent."
> It owns "did the stored obligations still hold."**

It does not need to own planning, tools, memory, or the host's special logic. It needs
only events in, verdicts out, and a hold when the verdict is not CONTINUE. That is why
it can sit beside other harnesses as a double-checker and enforcer — without fighting
them for ownership of tools or personality.

---

## 10. Flash control loops

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

---

## 11. The registries

The vest is a **routing table**, not a costume. Loops are addressable by **scope key**
and resolve at tick time.

### Structure

```
REGISTRY
├── harness registry     registered hosts: adapter, compliance level,
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

## 12. Harness registration

Register harnesses around Hearth, and harnesses into Hearth, from config:

```yaml
# hearth.config.yaml
harness:
  mode: meta                    # native | meta | host
  compliance: enforce           # enforce | audit | observe
  host:
    id: my-agent-harness
    adapter: adapters/my_agent_harness.py

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

## 13. Contracts

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

### Contract versioning

Contracts are **versioned**. The version in force is recorded in every checkpoint
([§4](#4-the-core-loop)). A mid-run contract change is itself a **finding** — and it
raises R to the full dependency chain, reusing the existing dynamic-R rule rather than
inventing new machinery. The contract that changed, and everything downstream of it,
gets replayed before the run proceeds.

---

## 14. The canonical traversal graph

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
something no file tree can express.

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

---

## 15. Metrics, for free

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

---

## 16. Overhead accounting

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

## 17. Benchmark: harness power === traversal power

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

Cross harnesses × models on the same sweep and you answer the question single-score
agent benchmarks cannot:

> **Is the model actually more capable — or is its harness giving it more effective
> traversal and control capacity?**

Capability should be reported at the model × harness configuration level. Hearth makes
that not just possible but automatic.

---

## 18. Quick start

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
hearth replay  --run R001 --depth 3          # reconstruct state at any traversal depth
```

---

## 19. Design invariants

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
12. **Worst verdict wins.** Many reviewers, one gate — disagreement degrades toward
    caution, never toward optimism.

---

## 20. Limitations

Stated plainly, because a harness that demands honesty from its hosts should show some:

- **Adapters must be real.** The canonical graph is only as good as each host's adapter.
  Unmaintained adapters mean degraded traces, and degraded traces mean weaker replay.
- **Adaptive-R rules are policy.** The dynamic-R table is a sensible default, not a law
  of nature. It must be tuned per domain, and bad policy produces bad backtracks.
- **Replay costs tokens and latency.** Overhead is measured
  ([§16](#16-overhead-accounting)), not hidden. Cheap-tier checks are cheap; deep replays
  are not.
- **ENFORCE requires a pause-capable host.** Hosts that cannot pause degrade to AUDIT
  (shadow) or OBSERVE — enforcement quality drops with the ladder. That is a documented
  degradation path, not a free pass.
- **Supervisor quality bounds enforcement quality.** The outermost loop acts on
  findings; garbage findings produce garbage backtracks.
- **Design-stage numbers are illustrative.** The R-sweep table shows the axis, not
  earned results. The values are claims to be earned by the benchmark suite
  ([§17](#17-benchmark-harness-power--traversal-power)), not facts yet.

---

## 21. Prior art

The full surveyed landscape — including hedged, verify-before-citing names — lives in
[`docs/PRIOR_ART.md`](docs/PRIOR_ART.md). The solidly citable pieces:

- **ReAct** — interleaved reasoning and acting; the base agent loop.
- **MPC / receding-horizon control** — plan, execute a portion, re-plan on new state.
- **Tree of Thoughts / search over reasoning paths** — branch, evaluate, backtrack.
- **Backtracking search** — return to an earlier decision point on downstream failure.
- **Checkpoint + replay engineering** — save points, revalidate from them.

What did not appear as one named object: enforcement-over-artifacts as the *identity*
of a harness, no-unbound-contracts by construction, dynamic R as both first-class
control and benchmark axis, dual wrap/be-wrapped under one module contract and four
verdicts, a flash vest of stackable loops with explicit precedence, and AITD plus the
canonical traversal graph as free measurement of harness power. The pieces existed.
The binding did not.

---

## 22. Roadmap

- [ ] Reference adapters — files, shell, browser
- [ ] 3D adapters — Blender first (canonical graph is already software-agnostic)
- [ ] AITD reporter CLI + trace viewer (p50/p95 distributions per level)
- [ ] R-sweep benchmark suite — harness × model × replay depth
- [ ] Flash-loop registry — community loops, scoped and composable
- [ ] Bundle registry — portable enforcement profiles across harnesses
- [ ] Compliance-ladder adapters — AUDIT shadow runners for pause-less hosts
- [ ] Checkpoint store backends — snapshot refs + compensating-action descriptors
- [ ] Supervisory scheduler presets — periodic overview agent as outermost loop

---

## 23. FAQ

**Is this another agent framework?**
No. It is the control layer frameworks lack. Run it native, or wrap yours — it works
either way, which is the point.

**Does it replace my harness?**
No. It can wear your harness or be worn by it. Both directions are first-class
integrations. It does not compete for "who is the agent."

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
emit, release. Iteration = tick. Metric = tick. Trace = tick.

**What if my harness can't pause?**
Declare `compliance: audit`. The identical pipeline runs in shadow — every violation
recorded, reported to the supervisor, enforced from the outermost loop. No host is
worth zero enforcement.

**Who executes a repair?**
The loop that owns the failing artifact. Hearth schedules it and re-verifies with the
same R window that caught the finding.

**What happens when two loops disagree on the same tick?**
Worst verdict wins: `HALT > BACKTRACK(d) > REPAIR > CONTINUE`. Many reviewers, one
gate. Disagreement degrades toward caution, never toward optimism.

**Why the name?**
A hearth is the heart of a structure — everything else gets built around it.
Enforcement-over-state is the organ other harnesses are missing.

**What does it measure?**
Whatever it enforces. Ticks per level, replay depth used, blast radius, active sets,
traversal coverage, its own overhead. The instrument and the enforcer are the same
object.

---

## 24. Glossary

| Term | Definition |
|---|---|
| **Control-Loop Harness** | master rolling-horizon loop + registry of sub loops; makes stored state enforceable |
| **Tick** | the atomic unit — one host event run through the fixed six-step pipeline; an iteration is one tick |
| **Flash loop** | swappable control-loop module, flashed into the vest at runtime |
| **The vest** | the loop registry — a kit, not a strap; a routing table of scope keys |
| **Bundle** | a named, portable set of control loops, attachable to any registered harness unchanged |
| **Scope key** | the address of a loop's jurisdiction — `harness:`, `modality:`, `tool:`, `scenario:` |
| **Replay depth (R)** | how many prior checkpoints each step re-verifies; dynamic |
| **Forward horizon (H)** | how far ahead each step plans |
| **AITD** | Agentic Iterative Traversal Depth — ticks per serialization level, reported as p50/p95 distributions |
| **Blast radius** | downstream decisions invalidated by a correction |
| **Correction depth** | how far backward a correction propagated |
| **Checkpoint** | id + trace position + contract versions in force + state snapshot ref (or compensating-action descriptor) |
| **Verdict** | `CONTINUE` / `REPAIR` / `BACKTRACK(d)` / `HALT` |
| **Worst verdict wins** | per-tick aggregation rule — `HALT > BACKTRACK(d) > REPAIR > CONTINUE` |
| **Compliance level** | declared host capability — ENFORCE (pause) / AUDIT (shadow) / OBSERVE (trace) |
| **Non-rewrite guarantee** | Hearth gates the release of ticks; it never edits them — diffable, therefore testable |
| **Canonical traversal graph** | software-agnostic record of every traversal event |
| **Adapter** | maps a native harness/environment onto the canonical schema |
| **Contract** | a stored obligation bound to a loop — never stored unbound, always versioned |
| **Supervisor** | the outermost loop — the only time-based loop; raises R, forces backtracks, audits progress |

---

## 25. License

MIT
