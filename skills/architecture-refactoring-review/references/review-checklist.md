# Architecture Review Checklist

Use this file as an operational checklist during repository inspection.

## A. System Boundary

- [ ] Major modules identified
- [ ] Public entry points identified
- [ ] Persistence boundaries identified
- [ ] External APIs/services identified
- [ ] Background jobs/workers identified
- [ ] Message queues/event buses identified
- [ ] UI/API boundaries identified
- [ ] No unverified component assumptions introduced

## B. Dependency Topology

- [ ] Cyclic dependencies checked
- [ ] High fan-in nodes reviewed
- [ ] High fan-out nodes reviewed
- [ ] Business-core -> infrastructure dependencies checked
- [ ] Module responsibilities reviewed
- [ ] Package boundaries compared with business boundaries

## C. Data Flow

For at least one core business path:

- [ ] Input identified
- [ ] Parsing identified
- [ ] Validation identified
- [ ] Transformations identified
- [ ] Domain logic identified
- [ ] Persistence identified
- [ ] External side effects identified
- [ ] Error path identified
- [ ] Model conversions identified

## D. Data Structures

- [ ] Primitive obsession checked
- [ ] Magic strings/numbers checked
- [ ] Nested dictionaries/maps checked
- [ ] DTO / Domain / PO reuse checked
- [ ] Oversized lifecycle structures checked
- [ ] Boolean state combinations checked
- [ ] Invariant validation checked
- [ ] Container choice matches access pattern

## E. State Ownership

For each important mutable state:

- [ ] Creator identified
- [ ] Consistency owner identified
- [ ] Writers identified
- [ ] Readers identified
- [ ] Update propagation identified
- [ ] Invalid concurrent modification risk checked
- [ ] Lifetime / expiration identified

## F. Side Effects

- [ ] Database writes localized
- [ ] Network calls explicit
- [ ] Filesystem operations explicit
- [ ] Message publication explicit
- [ ] External commands explicit
- [ ] Constructors are side-effect safe
- [ ] Getters do not hide side effects
- [ ] Mapping functions do not hide side effects

## G. Runtime Consistency

- [ ] Transaction boundaries reviewed
- [ ] Long transactions checked
- [ ] External calls inside transactions checked
- [ ] Exception swallowing checked
- [ ] Error translation checked
- [ ] Timeout policies checked
- [ ] Retry policies checked
- [ ] Idempotency checked
- [ ] Race conditions checked
- [ ] Lock ownership checked
- [ ] Cache source of truth defined
- [ ] Cache invalidation reviewed
- [ ] Resource cleanup reviewed

## H. Control Flow and Patterns

- [ ] Large switches reviewed
- [ ] Nested conditionals reviewed
- [ ] Variation axis identified before recommending Strategy
- [ ] Lifecycle transitions identified before recommending State/FSM
- [ ] Hard-coded dependency creation checked
- [ ] Direct downstream notification coupling checked
- [ ] Facade use does not hide god objects
- [ ] Mediator does not become a god object
- [ ] Interfaces have a real variation/testing reason
- [ ] No pattern recommended solely for textbook conformity

## I. Refactoring Safety

- [ ] Characterization tests exist or are proposed
- [ ] Public behavior preservation defined
- [ ] Migration can be incremental
- [ ] Rollback path considered
- [ ] Verification method defined per step
- [ ] Unnecessary rewrites avoided

## J. Final Classification

Every finding should contain:

- [ ] Severity
- [ ] Evidence
- [ ] Symptom
- [ ] Root cause
- [ ] Risk
- [ ] Minimal remedy
- [ ] Verification method

Every recommendation should be placed into one of:

- [ ] Do now
- [ ] Do later
- [ ] No change required
