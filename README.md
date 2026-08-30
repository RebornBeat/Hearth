Here it is — the full README, complete, nothing omitted.

````markdown
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
6. [The module contract](#6-the-module-contract)
7. [Stacking & precedence](#7-stacking--precedence)
8. [Three modes](#8-three-modes)
9. [Flash control loops](#9-flash-control-loops)
10. [Harness registration](#10-harness-registration)
11. [Contracts](#11-contracts)
12. [The canonical traversal graph](#12-the-canonical-traversal-graph)
13. [Metrics, for free](#13-metrics-for-free)
14. [Benchmark: harness power === traversal power](#14-benchmark-harness-power--traversal-power)
15. [Quick start](#15-quick-start)
16. [Design invariants](#16-design-invariants)
17. [Roadmap](#17-roadmap)
18. [FAQ](#18-faq)
19. [Glossary](#19-glossary)
20. [License](#20-license)

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

It is three things at once, all simultaneously true:

### 1. Native harness
Runs standalone. `/goal` → `/subtask` → execute → hooks → replay → verify → `/recap` → done.
A complete harness in itself.

### 2. Meta harness
Sits **on top of** other harnesses via adapters. The host harness emits events; the
adapter maps them to canonical loop events; the enforcement core returns verdicts.
One interface for every host.

### 3. Loop host — the vest
A **kit of flash-in control loops**: modality loops, tool loops, scenario loops,
verification loops, and anything you register. You should never be restricted to one
pass-through control loop — there are too many places loops belong, so this is a vest,
not a single strap.

And one metaphor that is load-bearing, not decorative: it is the **heart**.
Enforcement-over-state is the organ most harnesses lack — which is exactly why other
harnesses can be built *around* it, integrate *with* it, and register *into* it without
changing what it is. It adapts cleanly into any harness, native or meta, as an add-on,
a library, or an API.

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
| Critical failure | replay from last stable checkpoint |

### The verdict vocabulary

Everything the enforcement core says is one of four things. Nothing else escapes it:

| Verdict | Meaning |
|---|---|
| `CONTINUE` | verified — advance |
| `REPAIR` | fix in place — re-verify before advancing |
| `BACKTRACK(d)` | return *d* checkpoints, repair, re-advance |
| `HALT` | stop — surface to supervisor or operator |

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

## 6. The module contract

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

---

## 7. Stacking & precedence

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

Every loop reports its iteration count per level. The enforcement structure *is* the
instrumentation structure.

---

## 8. Three modes

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

### Host — other harnesses wear Hearth

External harnesses register **into** the vest as loop modules. Their native controls
become loop steps; their uniqueness is preserved behind the one contract. A harness with
a special planner, a special memory, a special tool router — register it, scope it, and
it stacks with everything else.

**Both directions are first-class.** Hearth wraps harnesses. Harnesses wrap Hearth.
It is the heart either way.

---

## 9. Flash control loops

A **flash loop** is a control loop packaged as a swappable module — flash it into the
vest at runtime, per modality, per tool, per scenario, per level.

First in, by design:

| Loop type | Scope | Enforces |
|---|---|---|
| **Modality loops** (first-class, first shipped) | `modality` | modality-specific ordering, horizons, and verification — vision, code, docs, audio, … |
| Tool loops | `tool` | per-tool constraints: read-before-write, confirmation gates, retry policy |
| Scenario loops | `scenario` | per-task policies: research protocols, SWE-style verify-before-commit |
| Verification loops | `verification` | replay windows, audit passes, regression sweeps |
| **Yours** | anything | any loop matching the contract |

The registry is open. New modality, new tool, new scenario = new module, same interface,
same enforcement core, same trace format. The stack grows without degrading.

---

## 10. Harness registration

Register harnesses around Hearth, and harnesses into Hearth, from config:

```yaml
# hearth.config.yaml
harness:
  mode: meta                    # native | meta | host
  host:
    id: my-agent-harness
    adapter: adapters/my_agent_harness.py

loops:
  - id: modality.vision
  - id: tool.filesystem
  - id: scenario.swe-style

registered_harnesses:           # other harnesses, registered as loops
  - id: planner-x
    as: loop
    scope: scenario
  - id: memory-y
    as: loop
    scope: verification
```

Every harness is unique. The adapter exists precisely to honor that: it preserves each
harness's native control surface and maps it onto the one module contract. Nothing about
your harness needs to change to be controlled — and nothing about Hearth changes to
control it.

---

## 11. Contracts

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

---

## 12. The canonical traversal graph

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

## 13. Metrics, for free

The enforcement core is also the instrument. Every verdict, every iteration, emits a
record. Nothing extra to wire — **the thing that enforces traversal is the same thing
that measures it.**

### Primary metric: AITD — Agentic Iterative Traversal Depth

The vector of iteration counts per serialization level:

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

### Also emitted, per depth

| Metric | Meaning |
|---|---|
| `replay_depth_used` | R applied at that step |
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

## 14. Benchmark: harness power === traversal power

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

## 15. Quick start

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

h = Hearth(mode="meta", adapter=Adapter.for_host("my-agent-harness"))
h.register_loop("modality.vision")
h.register_loop("tool.filesystem")

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
heath replay   --run R001 --depth 3          # reconstruct state at any traversal depth
```

---

## 16. Design invariants

1. **No artifact is trusted without a loop over it.**
2. **Every loop is a module.** One contract, no exceptions.
3. **Every iteration emits a trace record.** No silent steps.
4. **Outer owns ordering; inner yields.**
5. **Only the four verdicts escape the core.** Nothing else crosses the boundary.
6. **R is dynamic — and recorded per step.**
7. **Native, meta, host: zero architectural change between modes.**

---

## 17. Roadmap

- [ ] Reference adapters — files, shell, browser
- [ ] 3D adapters — Blender first (canonical graph is already software-agnostic)
- [ ] AITD reporter CLI + trace viewer
- [ ] R-sweep benchmark suite — harness × model × replay depth
- [ ] Flash-loop registry — community loops, scoped and composable
- [ ] Supervisory scheduler presets — periodic overview agent as outermost loop

---

## 18. FAQ

**Is this another agent framework?**
No. It is the control layer frameworks lack. Run it native, or wrap yours — it works
either way, which is the point.

**Does it replace my harness?**
No. It can wear your harness or be worn by it. Both directions are first-class
integrations.

**Why isn't the checklist the feature?**
Because a checklist is storage. The loop over it is the power. The checklist ships —
the loop is the product.

**Why "flash"?**
Loops are packaged to be flashed in and out of the vest at runtime — swappable,
stackable, per modality, tool, or scenario. You never lock into one pass-through loop.

**Why the name?**
A hearth is the heart of a structure — everything else gets built around it.
Enforcement-over-state is the organ other harnesses are missing.

**What does it measure?**
Whatever it enforces. Iterations per level, replay depth used, blast radius, active
sets, traversal coverage. The instrument and the enforcer are the same object.

---

## 19. Glossary

| Term | Definition |
|---|---|
| **Control-Loop Harness** | master rolling-horizon loop + registry of sub loops; makes stored state enforceable |
| **Flash loop** | swappable control-loop module, flashed into the vest at runtime |
| **The vest** | the loop registry — a kit, not a strap |
| **Replay depth (R)** | how many prior checkpoints each step re-verifies; dynamic |
| **Forward horizon (H)** | how far ahead each step plans |
| **AITD** | Agentic Iterative Traversal Depth — iteration counts per serialization level |
| **Blast radius** | downstream decisions invalidated by a correction |
| **Verdict** | `CONTINUE` / `REPAIR` / `BACKTRACK(d)` / `HALT` |
| **Canonical traversal graph** | software-agnostic record of every traversal event |
| **Adapter** | maps a native harness/environment onto the canonical schema |
| **Contract** | a stored obligation bound to a loop — never stored unbound |
| **Supervisor** | the outermost loop — raises R, forces backtracks, audits progress |

---

## 20. License

MIT
````

Two notes, both quick:

1. **Name.** I used *Hearth* (heart + the thing everything is built around) as the working name — swap it anywhere without touching a single concept. The identity is the contract, not the label.
2. **No omission check.** Everything from the thread is in: native/meta/host, the vest, flash loops, modality-first, master + sub loops, multi-modal contracts, harness registration in both directions, uniqueness-per-harness adapters, the artifact-is-not-a-control principle, dynamic R table, verdicts, precedence rules, canonical graph with all 13 fields, structural-vs-traversal parent, AITD, the R-sweep benchmark, and the supervisor as outermost loop.

Want me to split this into README + CONTRIBUTING + spec pages next, or tighten any single section first?
