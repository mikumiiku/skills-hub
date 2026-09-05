---
name: architecture-refactoring-review
description: Systematically review software architecture, trace data flow, audit dependencies, data structures, state ownership, runtime consistency, and produce incremental refactoring plans without overengineering.
---

# Architecture Refactoring Review Skill

## Purpose

Use this skill to systematically analyze, review, and guide the refactoring of an existing software system.

This skill is intended for:
- architecture reviews;
- legacy-system refactoring;
- codebase smell analysis;
- dependency and module-boundary analysis;
- data-structure and state-ownership audits;
- runtime consistency and failure-boundary reviews;
- design-pattern selection based on observed problems;
- incremental refactoring planning.

The goal is not to maximize abstraction or apply as many design patterns as possible. The goal is to identify the real sources of complexity, explain their consequences, and propose the smallest effective architectural changes.

---

## Core Principles

1. **Data structures reflect the business model.**
   Prefer structures that explicitly represent business entities, lifecycle stages, invariants, and valid state transitions.

2. **Design patterns are remedies, not blueprints.**
   Never recommend a pattern merely because a code shape resembles a textbook example.

3. **Data and control flow matter more than static class diagrams.**
   Analyze how data enters the system, changes, causes side effects, and leaves the system.

4. **Runtime behavior matters as much as static structure.**
   Transactions, retries, timeouts, concurrency, caching, exceptions, and partial failures must be considered.

5. **Prefer minimum necessary complexity.**
   A flat and direct implementation is preferable when there is no real variation point or architectural pressure requiring abstraction.

---

## Mandatory Review Workflow

Perform the review in the following order unless the task explicitly requires a narrower scope.

### Step 1: Establish the System Boundary

Identify:
- major modules;
- entry points;
- external systems;
- persistence layers;
- network or protocol boundaries;
- UI or API boundaries;
- schedulers, workers, message queues, and background jobs.

Do not infer components that have not been observed in the codebase or supplied documentation.

Output a concise system-boundary summary before proposing architecture changes.

---

### Step 2: Build the Dependency Topology

Inspect module-level dependencies.

Look for:
- cyclic dependencies;
- high fan-in modules;
- high fan-out coordinators;
- business logic depending directly on infrastructure;
- modules with mixed responsibilities;
- package boundaries that do not match business boundaries.

Distinguish:
- **module-level dependency problems**, and
- **code-level implementation problems**.

Do not merge these two categories.

---

### Step 3: Perform an End-to-End Data-Flow Trace

Select one or more representative business scenarios and trace:

`input -> parsing -> validation -> transformation -> domain logic -> side effects -> persistence/output`

Record:
- data structures used at each stage;
- model conversions;
- mutation points;
- ownership transitions;
- external I/O;
- error paths.

Prefer real execution paths over isolated class-by-class inspection.

---

### Step 4: Audit Core Data Structures

Check whether core data structures accurately represent business semantics.

Look for:
- primitive obsession;
- deeply nested dictionaries/maps;
- magic strings or magic numbers;
- Boolean combinations used as implicit state machines;
- one oversized structure carrying fields for every lifecycle phase;
- DTO / Domain / PO model reuse across layers;
- missing invariant validation;
- inappropriate containers for the actual access pattern.

When a data structure is problematic, explain:
1. what business concept it is trying to represent;
2. why the current representation is weak;
3. what constraints are currently implicit;
4. what a better representation would make explicit.

---

### Step 5: Identify State Ownership and Side Effects

For each important mutable state, determine:
- who creates it;
- who owns consistency responsibility;
- who may mutate it;
- who may only read it;
- how updates are propagated.

Treat uncoordinated shared mutable state as high risk.

Also identify side effects:
- database writes;
- filesystem access;
- network calls;
- message publication;
- logging with business significance;
- external command execution;
- cache mutation.

Prefer side effects to be explicit at application or infrastructure boundaries.

---

### Step 6: Audit Runtime Consistency and Failure Boundaries

Inspect:
- transaction boundaries;
- exception propagation;
- timeout policies;
- retry policies;
- idempotency;
- concurrent writes;
- locking and race conditions;
- cache consistency;
- resource lifecycle;
- partial-failure behavior.

Do not assume the happy path is sufficient evidence of architectural correctness.

---

### Step 7: Classify Architecture Smells

Classify each issue into one or more categories:

- dependency topology;
- responsibility / module boundary;
- data structure;
- state ownership;
- control flow;
- side-effect coupling;
- runtime consistency;
- infrastructure coupling;
- unnecessary abstraction;
- testing difficulty.

For each issue, describe:

`Observed symptom -> Root cause -> Risk -> Smallest effective remedy`

Do not begin with a design-pattern name.

---

### Step 8: Select Patterns Only After the Variation Point Is Clear

Before introducing a pattern, answer:

- What changes frequently?
- What must remain stable?
- Are there multiple real implementations?
- Do those implementations evolve independently?
- Is the current coupling already causing maintenance or test cost?

Pattern guidance:

- **Strategy**: use when multiple algorithms/behaviors evolve independently.
- **State / FSM**: use when behavior depends on lifecycle state and transitions are explicit.
- **Adapter**: use to isolate external protocols, legacy interfaces, or incompatible representations.
- **Facade**: use to stabilize an external boundary, not to hide an internal god object.
- **Observer / Pub-Sub**: use when many consumers react independently to an event and direct notification coupling is costly.
- **Factory Method / composition root**: use when concrete dependency creation must be isolated from business logic.
- **Value Object**: use when primitive values carry business meaning or invariants.
- **Mediator**: use cautiously; it must reduce coordination complexity rather than become a new god object.

---

### Step 9: Produce an Incremental Refactoring Plan

Prefer Strangler-style migration over large rewrites.

A typical safe order is:

1. establish characterization tests;
2. stabilize external contracts;
3. introduce adapters/facades where needed;
4. clarify data models and conversions;
5. establish state ownership;
6. isolate side effects;
7. repair runtime consistency issues;
8. remove cyclic or inverted dependencies;
9. introduce patterns only where justified;
10. delete obsolete paths after migration.

Every proposed step should explain:
- scope;
- expected benefit;
- risk;
- prerequisites;
- how to verify behavior preservation.

---

## Review Modes

### Review Mode

Use when the user asks to:
- analyze architecture;
- find smells;
- evaluate code quality;
- suggest refactoring;
- produce a review report.

Rules:
- do not modify code unless explicitly requested;
- prioritize evidence;
- cite concrete files/modules/functions when available;
- classify findings by severity and urgency.

### Refactor Mode

Use when the user asks to modify the codebase.

Rules:
1. establish characterization tests first when behavior is insufficiently protected;
2. change one architectural pressure point at a time;
3. keep public behavior stable unless the user explicitly requests behavior changes;
4. run tests after each logical stage;
5. avoid broad rewrites when smaller migration steps are possible.

---

## Severity Levels

Use the following default classification:

### Critical
Likely to cause:
- data corruption;
- unrecoverable inconsistency;
- severe production instability;
- security boundary violations;
- widespread architectural deadlock.

### High
Likely to:
- block independent module evolution;
- produce repeated regressions;
- make testing or deployment substantially difficult;
- cause significant runtime inconsistency.

### Medium
Creates:
- recurring maintenance cost;
- unnecessary coupling;
- duplicated logic;
- fragile data conventions.

### Low
Primarily:
- readability;
- local simplification;
- minor abstraction cleanup;
- non-urgent maintainability concerns.

Do not inflate severity merely because a textbook smell exists.

---

## Hard Constraints

The following rules are mandatory.

- Do not recommend Strategy merely because a `switch` exists.
- Do not recommend State merely because an enum exists.
- Do not recommend Facade as a substitute for decomposing internal responsibilities.
- Do not recommend Mediator if it simply centralizes all logic into a new god object.
- Do not introduce interfaces solely for hypothetical future implementations.
- Do not treat DTO, Domain Entity, and persistence model separation as mandatory when the system is sufficiently small and change pressure does not justify it.
- Do not invent architecture problems in code that has not been inspected.
- Do not propose a rewrite unless incremental migration is demonstrably more dangerous or impractical.
- Do not optimize for design-pattern count.
- Do not confuse dependency inversion with dependency injection frameworks.
- Do not hide side effects inside getters, constructors, mapping functions, or generic utility helpers.
- Do not change public behavior silently during structural refactoring.

---

## Required Output Structure

Unless the user asks for another format, use:

1. **System Snapshot**
2. **Key Data Flow**
3. **Architecture Findings**
4. **Data Structure Findings**
5. **State / Side-Effect Findings**
6. **Runtime Consistency Findings**
7. **Recommended Refactoring Plan**
8. **Patterns Worth Using**
9. **Patterns Not Worth Using**
10. **Deferred / No-Change Areas**

For each finding, use:

- **Severity**
- **Evidence**
- **Why it matters**
- **Root cause**
- **Recommended change**
- **Verification**

---

## Reference Files

Read these when needed:

- `references/architecture-refactoring-guide.md`
  - detailed architecture methodology;
  - conceptual explanations;
  - smell-to-remedy mapping.

- `references/review-checklist.md`
  - concise operational checklist for codebase inspection.

- `references/report-template.md`
  - reusable output template for architecture review reports.

