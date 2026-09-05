# Software Architecture Analysis and Refactoring Guide

## 1. Core Guidance

### 1.1 Data Structures Are a Physical Projection of the Business Model

Core data structures strongly influence the upper bound of implementation complexity.

A well-designed structure should naturally represent:
- the business entity;
- its lifecycle;
- valid states;
- invariants;
- ownership;
- allowed transitions.

Avoid weak, unstructured data bags whose semantics depend entirely on caller convention.

---

### 1.2 Design Patterns Are Remedies, Not Blueprints

A design pattern should address a specific source of coupling, duplication, state complexity, or variation.

Do not begin with:

> "Which design pattern can be used here?"

Begin with:

> "What concrete problem makes this code difficult to change, test, or reason about?"

Then determine whether a pattern is the smallest effective remedy.

---

### 1.3 Flow Is More Important Than Static Structure

Module organization exists to support data and control flow.

Stable architectures usually exhibit:
- mostly one-directional dependencies;
- clear ownership of mutable state;
- explicit transformation boundaries;
- localized side effects;
- minimal shared mutable state.

---

### 1.4 Runtime Behavior Matters as Much as Static Structure

A clean class diagram does not guarantee a robust architecture.

Review:
- transactions;
- retries;
- timeouts;
- idempotency;
- exceptions;
- concurrency;
- cache consistency;
- partial failures;
- resource cleanup.

---

# 2. Five-Dimension Architecture Review Framework

## Dimension 1: Dependency Topology and Boundaries

Focus on module-level architecture.

### Cyclic Dependencies

Look for:
- A imports B while B imports A;
- dependency cycles through several packages;
- lifecycle coupling caused by circular ownership.

Cycles make independent testing, replacement, and evolution difficult.

### High Fan-In / High Fan-Out

High fan-in:
- may indicate a stable shared abstraction;
- may also indicate an overly central data model or utility module.

High fan-out:
- may indicate orchestration responsibility;
- may also indicate a god module.

Evaluate both stability and responsibility, not only graph degree.

### Layer Inversion

Check whether domain logic directly depends on:
- database drivers;
- framework-specific objects;
- network clients;
- UI components;
- protocol implementations.

Infrastructure should generally depend on contracts defined near the core, rather than the core depending on concrete infrastructure.

### Blurred Module Boundaries

Watch for one module that simultaneously owns:
- business rules;
- workflow orchestration;
- persistence;
- protocol handling;
- UI formatting.

---

## Dimension 2: Data Flow and State Control

Trace representative use cases end to end.

### End-to-End Flow

Trace:

`input -> parse -> validate -> transform -> domain logic -> side effect -> persistence/output`

Capture:
- representations;
- conversions;
- mutations;
- ownership;
- error paths.

### Layer Model Isolation

Check whether one structure is reused as:
- API DTO;
- domain entity;
- database record.

Separation is valuable when these concerns evolve independently.

Do not split models mechanically in tiny systems where the cost exceeds the benefit.

### State Ownership

Every important mutable state should have a consistency owner.

Ownership answers:
- who creates it;
- who validates changes;
- who may mutate it;
- who resolves conflicting updates;
- when it expires.

"Single writer ownership" means single consistency responsibility, not necessarily one class or one thread.

### Side-Effect Boundary

Keep:
- database I/O;
- files;
- RPC;
- HTTP;
- messages;
- external commands

outside pure domain computation where practical.

---

## Dimension 3: Core Data Structure Audit

### Primitive Obsession

Symptoms:
- business identifiers as arbitrary strings;
- money as plain floats;
- state encoded by integers;
- nested dictionaries passed through many functions;
- repeated validation everywhere.

Prefer explicit types or value objects when they remove ambiguity or centralize invariants.

### Anemic vs Oversized Structures

Too light:
- fields exist but invariants do not;
- every caller revalidates the same assumptions.

Too heavy:
- one object contains every possible lifecycle field;
- many nullable fields;
- Boolean combinations encode hidden states.

### Lifecycle Representation

Use explicit lifecycle states when invalid combinations must be prevented.

For complex transitions, consider an FSM.

### Access Locality and Container Choice

Match storage to access pattern.

Examples:
- high-frequency lookup -> indexed map instead of repeated scans;
- bounded time-series append -> ring buffer/deque instead of expensive reallocations;
- hierarchy -> tree;
- dependency relation -> graph.

---

## Dimension 4: Pattern Recognition and Anti-Pattern Review

### Deep Branching

Do not automatically replace branches.

Use Strategy when:
- one variation axis exists;
- behaviors are substantial;
- implementations evolve independently.

Use State/FSM when:
- behavior depends on lifecycle state;
- transitions matter;
- invalid transitions should be prevented.

Keep a switch when:
- branches are simple;
- stable;
- easy to understand;
- unlikely to evolve independently.

### Hard-Coded Construction

Business logic should not usually create:
- concrete database clients;
- transport clients;
- vendor SDK clients.

Prefer dependency injection via constructor/function composition or a composition root.

A DI framework is optional.

### Communication Coupling

Direct chained notifications create:
- lifecycle coupling;
- deep call chains;
- difficult test setup.

Observer / Pub-Sub may help when reactions are truly independent.

### Pattern Abuse

Watch for:
- interface + one implementation with no variation pressure;
- empty factories;
- wrappers that only forward calls;
- generic abstractions that erase useful domain meaning.

---

## Dimension 5: Runtime Consistency and Failure Boundaries

### Transaction Boundaries

Clarify which operations must succeed atomically.

Avoid:
- huge transactions;
- holding database transactions across slow network calls;
- unclear multi-resource consistency.

### Exception Propagation

Avoid:
- swallowing errors;
- logging and continuing without a recovery model;
- leaking low-level implementation exceptions through every layer.

Translate errors where layer semantics differ.

### Timeout and Retry

Every remote call should have a deliberate timeout strategy.

Retries require:
- idempotency;
- bounded retry count;
- backoff where appropriate;
- awareness of duplicate side effects.

### Concurrency

Inspect:
- shared mutable state;
- lock ownership;
- read/write races;
- optimistic vs pessimistic concurrency;
- async task lifecycle.

### Cache Consistency

Define:
- source of truth;
- invalidation strategy;
- update path;
- acceptable staleness;
- fallback behavior.

The cache should not silently become a second authoritative datastore.

### Resource Lifecycle

Ensure clear ownership for:
- connections;
- files;
- sockets;
- threads;
- processes;
- tasks;
- GPU/device resources where relevant.

---

# 3. Smell-to-Remedy Matrix

| Architecture Smell | Typical Symptom | Data / Structure Problem | Likely Remedy |
|---|---|---|---|
| God class / procedural mudball | one module owns state, orchestration, persistence, and communication | unrelated lifecycle state mixed together | decompose responsibilities; Facade only for stable external entry; Mediator only if coordination complexity truly decreases |
| Branch explosion | giant switch or nested conditionals | magic values / implicit states | Strategy for independently evolving behaviors; State/FSM for lifecycle-driven behavior |
| Shotgun surgery | one field change touches parser, business logic, DB, API | no model boundaries | Adapter; explicit model conversion; contract isolation |
| Direct notification coupling | one change directly calls many downstream components | caller holds downstream structures | Observer / Pub-Sub with event objects |
| Infrastructure inversion | domain imports DB/network SDK | business state coupled to I/O representation | dependency inversion; ports/interfaces; composition root |
| Raw data propagation | dictionaries and arrays cross multiple layers | missing semantics and invariants | Value Objects / explicit domain objects |
| State combination explosion | many booleans define implicit state | invalid combinations possible | explicit states; FSM |
| Repeated side effects | many code paths write DB/send messages | unclear mutation responsibility | application-service boundary; explicit transaction/side-effect ownership |
| Cache as hidden truth source | code bypasses primary persistence | unclear authority | source-of-truth definition; invalidation/update policy |

---

# 4. Refactoring Rules

## Characterization Tests First

Before altering important structures, establish black-box tests that preserve observed behavior.

The purpose is not necessarily to prove that current behavior is correct.

The purpose is to detect unintended changes.

---

## Refactor According to the Main Complexity Source

Prefer data-model refactoring when representation is the dominant problem.

Prefer runtime-control refactoring when the dominant issue is:
- concurrency;
- transactions;
- retries;
- resource lifetime;
- error flow.

Do not apply a universal "data first" rule.

---

## Strangler-Style Migration

For legacy systems:

1. wrap unstable legacy boundaries;
2. migrate callers to new contracts;
3. replace implementation behind those contracts;
4. remove old code only after usage disappears.

Facade stabilizes a boundary.
Adapter reconciles incompatible interfaces.
Neither replaces real responsibility decomposition.

---

## Identify the Variation Axis Before Introducing Abstraction

Ask:
- what changes?
- how often?
- independently of what?
- what must remain stable?

If the answer is unclear, do not introduce a pattern yet.

---

## Make Side Effects Explicit

Avoid important side effects in:
- constructors;
- getters;
- mappers;
- generic helpers.

An interface should make it reasonably obvious when an operation changes external state.

---

## Keep Abstraction Proportional

Do not optimize for:
- number of interfaces;
- number of layers;
- number of patterns.

Optimize for:
- independent change;
- testability;
- local reasoning;
- clear ownership;
- reliable runtime behavior.

