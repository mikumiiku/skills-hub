# Architecture Review Report Template

## 1. System Snapshot

**System purpose:**  
<brief description>

**Major modules:**  
- <module>
- <module>

**External dependencies:**  
- <database / service / protocol / framework>

**Observed architecture style:**  
<description based only on evidence>

---

## 2. Key Data Flow

### Scenario: <business scenario>

`<input> -> <parse> -> <validate> -> <domain logic> -> <side effect> -> <output>`

### Data Representations

| Stage | Representation | Owner | Mutable? | Notes |
|---|---|---|---|---|
| Input | | | | |
| Domain | | | | |
| Persistence | | | | |

---

## 3. Architecture Findings

### Finding A1 — <title>

**Severity:** Critical / High / Medium / Low

**Evidence:**  
- `<file/module/function>`
- `<specific observed behavior>`

**Observed symptom:**  
<what is visibly wrong>

**Root cause:**  
<why the structure produces this problem>

**Why it matters:**  
<maintenance, reliability, testability, evolution impact>

**Recommended change:**  
<smallest effective remedy>

**Verification:**  
<test or architecture check>

---

## 4. Data Structure Findings

### Finding D1 — <title>

**Severity:**  

**Current representation:**  

**Business concept actually represented:**  

**Implicit constraints:**  

**Problem:**  

**Recommended representation:**  

**Verification:**  

---

## 5. State and Side-Effect Findings

### State: <name>

**Owner:**  
<module/component>

**Writers:**  
- ...

**Readers:**  
- ...

**Observed risks:**  
- ...

**Recommended ownership model:**  
...

### Side Effects

| Side Effect | Current Location | Appropriate Boundary? | Recommendation |
|---|---|---|---|
| DB write | | | |
| Network call | | | |
| Message publish | | | |

---

## 6. Runtime Consistency Findings

### Transactions

- <finding>

### Exceptions

- <finding>

### Timeout / Retry / Idempotency

- <finding>

### Concurrency

- <finding>

### Cache

- <finding>

### Resource Lifecycle

- <finding>

---

## 7. Recommended Refactoring Plan

### Phase 0 — Safety Baseline

- add characterization tests;
- define behavior that must remain unchanged.

### Phase 1 — Boundary Stabilization

- <adapter/facade/contract work>

### Phase 2 — Data Model Cleanup

- <model changes>

### Phase 3 — State and Side-Effect Ownership

- <ownership changes>

### Phase 4 — Runtime Consistency

- <transaction/retry/concurrency changes>

### Phase 5 — Structural Dependency Cleanup

- <dependency changes>

### Phase 6 — Remove Legacy Paths

- <deletion / cleanup>

---

## 8. Patterns Worth Using

### <Pattern>

**Problem it solves:**  
...

**Variation point:**  
...

**Why justified here:**  
...

---

## 9. Patterns Not Worth Using

### <Pattern>

**Why not:**  
- no real variation point;
- added abstraction exceeds expected benefit;
- current implementation is already sufficiently simple.

---

## 10. Deferred / No-Change Areas

### <Area>

**Decision:** Defer / No change

**Reason:**  
...

---

## 11. Final Priority Table

| Priority | Finding | Action | Risk if Deferred | Effort |
|---|---|---|---|---|
| P0 | | | | |
| P1 | | | | |
| P2 | | | | |

