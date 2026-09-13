# Chapter 18 — Designing Stable Contracts and Invariants

> **Status:** `[~] In Progress`  
> **Part:** C — Responsibility-Driven Design  
> **Primary theme:** Stable contracts, invariants, behavioral correctness, and evolvable object boundaries  
> **Tracks:** Track A — Core Theory · Track B — Implementation · Track C — Interview / Reasoning

---

## Chapter Map

A contract is an observable promise between collaborators. An invariant is what must remain true across valid states and valid transitions. This chapter moves from object-local validity to API, persistence, event, security, concurrency, and operational boundaries.

```text
object state
  → method contract
  → type contract
  → domain contract
  → API contract
  → persistence contract
  → event contract
  → security contract
  → operational contract
```

The central mental model is:

```text
contract   = observable promise
invariant  = what must remain true
implementation = mechanism
```

## 1. Learning Objectives

Master preconditions, postconditions, invariants, representation invariants, domain invariants, Design by Contract, Hoare-style reasoning, state machines, ownership, mutability, equality, serialization, idempotency, retries, concurrency, consistency, transactions, TypeScript runtime boundaries, API/event evolution, security, tenant isolation, contract testing, and principal-level trade-offs.

## 2. Prerequisites

Builds directly on the JavaScript object model, construction, classes, private state, abstraction, inheritance, polymorphism, composition, cohesion, coupling, responsibility-driven design, GRASP, and refactoring.

## 3. What Is a Contract?

A contract defines behavior a consumer can safely rely on. It can cover accepted inputs, outputs, state mutation, errors, ordering, timing, side effects, resource limits, authorization, consistency, and retry semantics. A method signature is only one part of the contract.

## 4. Why Stable Contracts Matter

Stable contracts isolate consumers from implementation change. They reduce change amplification, accidental coupling, integration surprises, migration cost, and incident blast radius.

## 5. Contract vs Implementation

Two implementations can differ completely while honoring the same contract:

```text
MemoryUserRepository ─┐
                      ├→ getById(id): User | null
SqlUserRepository ────┘
```

The storage mechanism is private; absence semantics and error behavior are observable.

## 6. Contract Layers

```text
type
behavior
 domain
 transport/API
 persistence
 events
 security
 operations
```

Do not confuse protocol syntax with business meaning. A JSON shape may be valid while still violating a domain invariant.

## 7. Preconditions

A precondition must hold before a successful operation.

```ts
withdraw(amount) {
  if (amount <= 0) throw new RangeError("amount must be positive");
  if (amount > this.balance) throw new InsufficientFundsError();
}
```

Typical preconditions include positive quantity, authorization, object lifecycle state, expected version, and required dependency readiness.

## 8. Postconditions

A postcondition describes successful outcome.

```text
oldBalance = B
withdraw(A)
success ⇒ newBalance = B - A
```

Postconditions should describe semantic results, not accidental implementation steps.

## 9. Invariants

An invariant is a property that remains true for every valid externally observable state.

Examples:

```text
balance >= 0
reservedQuantity >= 0
createdAt <= updatedAt
```

Every public mutation should preserve the invariant.

## 10. Design by Contract

Design by Contract centers on:

```text
requires  → preconditions
ensures   → postconditions
invariant → valid-state rules
```

JavaScript does not require a built-in formal contract language; these semantics can be implemented with ordinary language mechanisms and tests.

## 11. Hoare-Style Reasoning

Use the notation:

```text
{P} operation {Q}
```

For example:

```text
{balance >= amount ∧ amount > 0}
withdraw(amount)
{balance' = balance - amount}
```

This forces precise reasoning about state transitions.

## 12. Representation Invariants

A representation invariant constrains internal representation.

Example:

```text
for every map entry,
key === user.id
```

Representation invariants can change during refactoring as long as the domain and external contracts remain stable.

## 13. Domain Invariants

Domain invariants describe business truth independent of a particular representation.

Example:

```text
a serialized jewellery item cannot simultaneously be
AVAILABLE and SCRAPPED
```

Domain invariants are more durable than database or class-field details.

## 14. Construction Establishes Validity

Constructors and factories are invariant gates.

```ts
class EmailAddress {
  #value: string;

  constructor(value: string) {
    if (!value.includes("@")) throw new TypeError("invalid email");
    this.#value = value;
  }
}
```

Do not treat construction as mere allocation when validity matters.

## 15. Factory Contracts

Factories are useful when creation requires normalization, generated identity, dependency selection, configuration validation, or policy checks.

```ts
class Money {
  private constructor(
    readonly amount: number,
    readonly currency: string,
  ) {}

  static create(amount: number, currency: string) {
    if (!Number.isFinite(amount)) throw new TypeError("amount must be finite");
    if (!currency) throw new TypeError("currency required");
    return new Money(amount, currency);
  }
}
```

## 16. Illegal States and Representation

A powerful design goal is:

```text
reduce representable invalid combinations
```

Discriminated unions, value objects, private state, and semantic commands can remove invalid states from the normal API surface.

## 17. TypeScript Is Not Runtime Validation

Types describe checked source programs. External JSON, database rows, environment variables, JavaScript callers, and third-party results still require runtime validation.

```ts
function handle(input: unknown) {
  // validate before treating input as a trusted domain object
}
```

## 18. `unknown` vs `any`

`unknown` preserves uncertainty and forces narrowing. `any` bypasses many compiler checks. `any` may be justified at tightly controlled compatibility boundaries, but it weakens contracts when used throughout a domain model.

## 19. `asserts` Functions

Assertion functions connect runtime checking with TypeScript control-flow analysis:

```ts
function assertPositive(value: number): asserts value is number {
  if (value <= 0) throw new RangeError("positive required");
}
```

The runtime implementation must actually enforce the promised condition.

## 20. Discriminated Unions

Model exclusive states explicitly:

```ts
type PaymentState =
  | { kind: "pending" }
  | { kind: "authorized"; authorizationId: string }
  | { kind: "captured"; captureId: string }
  | { kind: "failed"; reason: string };
```

This is stronger than a bag of booleans that allows contradictory combinations.

## 21. `satisfies`

`satisfies` checks that an expression conforms to a target type while preserving useful inferred detail. It is particularly useful for configuration tables and handler maps where exact literals matter.

## 22. `readonly` and `as const`

`readonly` restricts assignment through the type. `as const` preserves literal types and readonly properties. Neither automatically deep-freezes arbitrary runtime objects.

## 23. Branded Types

A branded type can represent a value that has passed a domain gate:

```ts
type PositiveQuantity = number & { readonly __brand: "PositiveQuantity" };
```

The brand is not validation; the validating constructor is what establishes the contract.

## 24. Null, Undefined, and Absence

Define whether each means:

```text
null      = explicitly absent
undefined = omitted / not supplied
missing   = property not present
empty     = present but empty
```

Do not leave these semantics to accident.

## 25. Error Contracts

Error behavior is observable. Stable APIs commonly define error category, error code, retryability, safe metadata, and translation rules.

A useful taxonomy is:

```text
validation
authorization
not found
conflict
business rule
transient infrastructure
permanent infrastructure
programmer defect
```

## 26. Command Contracts

Commands change state. Their contracts should define authorization, lifecycle preconditions, state transition, transaction scope, side effects, idempotency, and failure behavior.

## 27. Query Contracts

Queries should define ordering, filtering, pagination, consistency, freshness, absence semantics, and relevant resource limits. If order matters, make it explicit.

## 28. Lifecycle and Temporal Contracts

State machines make valid transitions explicit:

```text
DRAFT → CONFIRMED → PAID → SHIPPED
  └──────────────────────→ CANCELLED
```

A temporal contract can also state that one action must happen before another.

## 29. Ownership and Aliasing

Ask who owns a mutable value. Returning internal arrays or accepting external mutable objects can create aliases that bypass invariant-protecting methods.

Bad:

```ts
getLines() { return this.#lines; }
```

Safer options include snapshots, immutable values, or semantic query methods, chosen according to actual ownership and performance requirements.

## 30. Mutable vs Immutable Contracts

Immutability can reduce aliasing and simplify reasoning. Mutability can reduce allocation and suit aggregate state transitions. Choose deliberately based on identity, sharing, memory, concurrency, and ergonomics.

## 31. Equality and Identity Contracts

Define whether equality means reference, entity identity, structural equality, value equality, or business equality. Entity identity should not accidentally change during a lifecycle transition.

## 32. Serialization and DTO Contracts

A DTO is a transport representation; a domain object is an invariant-rich behavioral representation. Deserialization should transform untrusted representation into validated internal semantics rather than assuming JSON shape equals domain validity.

## 33. Transaction Contracts

A transaction should define what is atomic. A local database transaction does not automatically make a database update and external API call globally atomic.

## 34. Commit Points

When an async method resolves, define what “success” means:

```text
accepted by memory?
committed to database?
published to broker?
replicated?
```

Do not let callers guess.

## 35. Idempotency Contracts

Idempotency means repeating the same logical request does not create an unintended additional effect. It does not mean “no side effects.”

For example, setting status to CANCELLED can be idempotent even though the first call mutates state.

## 36. Retry Safety

A timeout does not prove failure. The server may have completed the operation while the response was lost. Retry contracts therefore depend on idempotency, provider semantics, and correlation identifiers.

## 37. Idempotency Key Scope

Define the key namespace and reuse rules:

```text
per tenant
per account
per endpoint
per operation type
```

Reusing the same key with different logical input should be an explicit conflict, not ambiguous behavior.

## 38. Concurrency Contracts

For concurrent operations, state the required outcome. Example:

```text
Two callers reserve the final unit.
Exactly one reservation is accepted.
```

That requirement drives version checks, locking, conditional updates, or other mechanisms.

## 39. Optimistic Concurrency

A version contract can be:

```text
update succeeds only if storedVersion == expectedVersion
```

A stale writer receives a conflict and must reload before deciding whether to retry.

## 40. Consistency Contracts

Distinguish strong, eventual, read-your-writes, session, and other consistency expectations. A cache or replica should never silently imply a stronger freshness guarantee than it provides.

## 41. Cache Freshness

Cache behavior is a contract when consumers care about freshness.

```text
browsing price → small staleness acceptable
checkout price → authoritative read required
```

Define maximum tolerated staleness when correctness depends on it.

## 42. Time and Randomness Dependencies

Ambient `new Date()` and `Math.random()` create hidden dependencies. Inject clocks or random sources when deterministic testing and explicit temporal contracts matter.

## 43. Configuration Contracts

Configuration should be parsed, validated, and normalized at startup:

```text
environment
 → parse
 → validate
 → normalize
 → trusted Config
```

Do not defer invalid configuration errors until deep inside a request.

## 44. Database Constraints

Use database constraints as integrity backstops where appropriate:

```sql
CHECK (quantity >= 0)
UNIQUE (tenant_id, sku)
```

The database can protect against bugs, alternate writers, scripts, and races that application-only checks may miss.

## 45. Local vs Global Invariants

A local invariant can often be enforced inside one aggregate. A global invariant may require coordination, transactions, workflows, reconciliation, or an intentionally weaker consistency contract.

## 46. Aggregate Boundaries

Keep tightly coupled invariants inside a consistency boundary when possible. Splitting one atomic invariant across independent services introduces protocol and failure complexity.

## 47. Outbox Semantics

A common reliable pattern is:

```text
transaction:
  update domain state
  write outbox record

publisher:
  publish durable event
```

The semantics are “committed state plus durable publication intent,” not magic global atomicity.

## 48. Distributed Invariants

When an invariant crosses services, consider workflow/state machine orchestration, compensation, idempotency, eventual consistency, and reconciliation. Do not hide a distributed protocol inside a deceptively simple method name.

## 49. Event Contracts

Events are APIs for consumers. Define schema, meaning, identity, versioning, duplication, ordering, replay, and tenant context when relevant.

## 50. Event Identity and Delivery

A robust event may include:

```text
eventId
aggregateId
tenantId
occurredAt
type
schemaVersion
```

Only include fields whose semantics are meaningful and maintained.

## 51. Event Ordering

Ordering may be global, per partition, per aggregate, or best-effort. Promise the narrowest ordering guarantee consumers actually need.

## 52. Duplicate Delivery

At-least-once delivery means consumers may see the same logical event more than once. Consumers need deduplication or idempotent processing.

## 53. API Compatibility

A change can be breaking without changing TypeScript shape. Examples:

```text
null → exception
stable ordering → unspecified ordering
retry-safe → non-idempotent
same error code → different error code
```

These are semantic breaking changes.

## 54. Backward Compatibility

**Backward compatibility** means existing valid consumers can continue to rely on previously promised observable behavior while implementation evolves.

Use additive changes, adapters, compatibility shims, migrations, and explicit deprecation when risk warrants.

## 55. Contract Accretion

Undocumented behavior can become a de facto contract when consumers start relying on it. Inspect consumer code, tests, incidents, logs, and observed ordering to discover accidental promises.

## 56. Contract Surface Area

Contract surface includes:

```text
inputs + outputs + errors + ordering + timing + side effects
+ ownership + consistency + security + resource behavior
```

Minimize unnecessary observability to preserve implementation freedom.

## 57. Ports and Adapters

A port defines application-facing semantics. An adapter translates provider-specific protocols into that stable contract.

```text
domain
  ↓
PaymentGateway
  ↓
provider adapter
  ↓
external provider
```

## 58. Adapter Semantic Loss

If a provider distinguishes three failure categories while the application exposes one, the semantic loss should be intentional and documented. Do not accidentally collapse information consumers need.

## 59. Repository Contracts

Define repository semantics beyond CRUD:

```text
identity
absence
consistency
transaction expectations
concurrency
error taxonomy
```

A memory repository and a SQL repository should pass the same behavioral contract tests where the abstraction claims substitutability.

## 60. Tenant Isolation

For a multi-tenant system, a critical invariant is:

```text
a caller may only observe or mutate resources within authorized tenant scope
```

Preserve tenant context across requests, services, repositories, caches, events, and storage.

## 61. Authorization Contracts

Authentication identifies a caller. Authorization determines whether a requested transition is permitted. Sensitive operations should expose the required authority boundary rather than relying on arbitrary property mutation.

## 62. Audit Contracts

Where auditability is required, define what must be captured, such as actor, before value, after value, time, reason, and affected resource. Required audit history is a contract, not optional debug logging.

## 63. Performance Contracts

Performance can be contractual when callers rely on it. Watch for accidental N+1 database calls, unbounded scans, excessive copying, unbounded queues, and impossible strict-complexity promises.

## 64. Resource Contracts

Define relevant limits: page size, upload size, retries, memory, open connections, API quota, and queue depth. Resource constraints preserve system-level invariants.

## 65. Timeout and Deadline Contracts

External calls should have intentional timeout semantics. A request deadline can be propagated downstream so retries do not silently consume the entire remaining budget.

## 66. Cancellation Contracts

Define what cancellation means. Is work stopped? Are resources released? Are partial side effects rolled back, compensated, or left committed? An abort signal alone does not answer those questions.

## 67. Operational Contracts and SLOs

Availability, latency, and failure budgets are operational expectations. They are useful contracts when measured and engineered intentionally, not when written as unsupported promises.

## 68. Backpressure

For queues and streams define what occurs when producers outrun consumers:

```text
buffer | block | reject | drop | shed load
```

Hidden backpressure behavior often becomes an incident under load.

## 69. Security Failure Semantics

Error detail can leak resource existence, identifiers, permissions, or provider behavior. A secure contract exposes enough information to act without unnecessarily revealing protected information.

## 70. Fail Closed vs Fail Open

For authorization, an unknown permission state will often need to fail closed. For availability-sensitive best-effort features, fallback may be preferable. The correct choice depends on security and business risk.

## 71. Graceful Degradation

Classify dependencies as critical, important, optional, or best-effort. Define what remains functional when each dependency is unavailable.

## 72. Bulkheads and Circuit Breakers

Resource isolation and circuit breaking change observable failure behavior. Document the fallback/error contract rather than treating these mechanisms as invisible implementation details.

## 73. Pure-ish Functions

Pure calculations reduce contract surface because their output depends on explicit input rather than hidden time, randomness, global state, or ambient configuration. Use them for normalization and calculation when practical.

## 74. Dependency Injection

Injection makes required semantics visible:

```ts
class PriceCalculator {
  constructor(private readonly taxPolicy: TaxPolicy) {}
}
```

The dependency is now part of the design rather than hidden global context.

## 75. Contract Documentation

Document:

```text
inputs
success
failure
mutation
side effects
ordering
retryability
security
consistency
resource limits
```

Strong documentation is backed by implementation and tests.

## 76. Executable Examples

Turn critical examples into automated tests where practical. An executable example reduces documentation drift and provides evidence of the intended contract.

## 77. Contract Tests

A contract test focuses on behavior rather than implementation:

```ts
async function repositoryContract(makeRepo: () => UserRepository) {
  const repo = makeRepo();
  // create
  // load by id
  // verify absence
  // verify error semantics
}
```

Run the same semantic suite against in-memory and real implementations when they claim substitutability.

## 78. Property-Based Testing

Property-based testing validates general properties over many generated inputs.

Examples:

```text
normalize(normalize(x)) == normalize(x)
money + zero == money
valid transition ⇒ invariant remains true
```

## 79. State-Machine Testing

Generate valid and invalid command sequences over lifecycle states. After every accepted transition, assert invariants. This is powerful for reservations, payments, order states, and workflows.

## 80. Characterization and Golden Master

When refactoring legacy code, characterize current observable behavior before changing implementation. Golden-master comparisons can expose accidental contract changes, but they may also preserve accidental behavior unless reviewed.

## 81. Mutation Testing

Mutation testing intentionally breaks implementation, such as changing `>` to `>=`. Strong contract tests should fail for meaningful mutations. This evaluates whether tests actually protect the intended behavior.

## 82. Debugging by Contract Violation

Trace the first semantic break:

```text
bad input
 → missing precondition
 → invalid transition
 → broken invariant
 → later symptom
 → incident
```

The final exception may be far from the root contract violation.

## 83. Refactoring — Value Objects

Replace repeated primitive clusters with invariant-bearing types such as Money, PositiveQuantity, EmailAddress, or UserId. This reduces duplicated validation and semantic ambiguity.

## 84. Refactoring — Replace Flags With States

Replace contradictory boolean combinations with explicit variants:

```ts
type OrderState =
  | { kind: "draft" }
  | { kind: "paid" }
  | { kind: "shipped" }
  | { kind: "cancelled" };
```

## 85. Refactoring — Hide Collections

Replace writable collection exposure with snapshot/query semantics or domain mutation methods that preserve invariants.

## 86. Refactoring — Introduce Version Contracts

Add `expectedVersion` when stale writers can overwrite newer state. Decide what the caller should do after conflict rather than silently overwriting.

## 87. Refactoring — Semantic Commands

Prefer:

```text
cancel()
markPaid(captureId)
reserve(quantity)
```

over generic:

```text
setStatus(...)
```

when transitions have domain rules or side effects.

## 88. Jewellery ERP — Inventory Invariants

Typical domain candidates include:

```text
quantity >= 0
grossWeight >= netWeight
netWeight >= 0
stoneWeight >= 0
purity within allowed domain range
```

The actual business rules remain the source of truth.

## 89. Jewellery ERP — Reservation Contract

A reservation operation may require:

```text
authorized tenant/branch
item exists
quantity > 0
quantity <= available
current version
idempotency key when retriable
```

A successful transition should update all related inventory state consistently.

## 90. Jewellery ERP — Sale Contract

A sale can require sellability, stock authority, pricing permission, tax correctness, payment semantics, auditability, and branch/tenant scope. Avoid collapsing all of this into a generic `update()` method.

## 91. Principal Question — Who Owns the Invariant?

Ask:

> Which boundary has enough information and authority to enforce this rule atomically and reliably?

The answer may be a domain aggregate, application service, database, security boundary, or distributed workflow.

## 92. Principal Question — How Strong Should the Guarantee Be?

Do not promise stronger semantics than the architecture can enforce.

```text
stronger guarantee
  → more implementation cost
  → more testing/operations
  → more future migration constraints
```

## 93. Principal Question — Contract Blast Radius

Evaluate consumer count, persisted data, event consumers, security impact, financial impact, performance impact, and operational impact before changing a critical contract.

## 94. Principal Question — Contract Complexity Budget

Every guarantee creates cost. Prefer a small set of high-value promises that can be enforced and observed reliably rather than maximal documentation with weak enforcement.

## 95. Principal Question — Stable Boundary

A healthy boundary allows you to replace:

```text
class
ORM
storage engine
cache
provider
message broker
framework
```

without changing business meaning unnecessarily.

## 96. API Evolution Workflow

Use:

```text
observe → characterize → define target → compatibility bridge
→ migrate → measure → remove bridge
```

This is safer than a synchronized “flag day” migration in large systems.

## 97. Backward Compatibility Principle

Backward compatibility is not only syntax compatibility. Preserve previously valid observable semantics unless a deliberate breaking change is announced and migrated.

## 98. Deprecation Contracts

A deprecation should state the old contract, replacement, migration path, and transition period. Deprecation without an exit strategy becomes permanent complexity.

## 99. Feature Flag Contracts

Feature flags can create multiple simultaneous contracts. Define cohort behavior, data compatibility, telemetry, and removal criteria.

## 100. Contract Review Checklist

Before shipping a change ask:

```text
What are the inputs?
What is success?
What is absence?
What are the errors?
What mutates?
What side effects occur?
What is ordered?
What can be stale?
Can it be retried?
What happens under concurrency?
Who is authorized?
What is the tenant scope?
What is atomic?
What is observed?
What breaks if this changes?
```

## 101. Code Review Exercise

Review:

```ts
async reserve(itemId: string, quantity: number) {
  const item = await db.items.findOne({ id: itemId });
  if (!item) return false;
  item.quantity -= quantity;
  await db.items.save(item);
  return true;
}
```

Questions:

```text
negative quantity?
tenant filter?
insufficient stock?
concurrency?
version?
retry?
transaction?
error category?
meaning of false?
```

## 102. Predict-the-Output — Shared Mutation

```ts
const state = { count: 0 };
function increment(s: { count: number }) { s.count++; }
increment(state);
console.log(state.count);
```

Prediction: `1`.

The contract lesson is ownership and mutation: the same object is observed through multiple references.

## 103. Predict-the-Output — Getter Snapshot

```ts
class Counter {
  #value = 0;
  get value() { return this.#value; }
  increment() { this.#value++; }
}
const c = new Counter();
const a = c.value;
c.increment();
console.log(a, c.value);
```

Prediction: `0 1`.

## 104. Predict-the-Output — Shallow Copy

```ts
const items = [{ quantity: 1 }];
const copy = [...items];
copy[0].quantity = 99;
console.log(items[0].quantity);
```

Prediction: `99`. The outer array is copied; nested object identity is shared.

## 105. Implementation From Scratch — Invariant Helper

```ts
export function invariant(
  condition: unknown,
  message: string,
): asserts condition {
  if (!condition) throw new Error(`Invariant violation: ${message}`);
}
```

Use it for trusted internal assumptions and critical invariant checks; do not treat an assertion helper as a substitute for external-input validation.

## 106. Implementation From Scratch — Result

```ts
type Result<T, E> =
  | { ok: true; value: T }
  | { ok: false; error: E };
```

Use explicit result types where expected failure is part of normal control flow and callers benefit from typed branching.

## 107. Implementation From Scratch — PositiveInteger

```ts
class PositiveInteger {
  private constructor(readonly value: number) {}

  static create(value: number) {
    if (!Number.isInteger(value) || value <= 0) {
      throw new RangeError("positive integer required");
    }
    return new PositiveInteger(value);
  }
}
```

## 108. Implementation From Scratch — State Machine

Define legal transitions first, then encode them with explicit state and semantic commands. After each successful command, assert the aggregate invariant.

## 109. Implementation From Scratch — Tenant-Aware Repository

Construct repositories with authorized tenant scope or an equivalent trusted context so that every read/write is automatically constrained. Do not make tenant filtering an optional convention at every call site.

## 110. Implementation From Scratch — Idempotent Command

Store an idempotency record keyed by the defined scope. On replay with the same logical request, return the existing outcome; on the same key with incompatible input, return a stable conflict.

## 111. Debugging — Negative Stock

The symptom is negative quantity. The first violation may be missing quantity preconditions or a race that allowed two reservations to observe the same availability. Fix the earliest violated contract, not only the final bad row.

## 112. Debugging — Double Charge

A timeout is mistaken for failure. Add a provider-safe idempotency strategy and a reconciliation path so a lost response does not imply a second charge.

## 113. Debugging — Cross-Tenant Read

Missing tenant scope is a security invariant violation. Check repository filters, cache keys, authorization context, and event payloads.

## 114. Debugging — Lost Update

A stale writer overwrites newer state because no version contract exists. Add optimistic concurrency or an intentional serialization strategy.

## 115. Debugging — Leaked Collection

A caller mutates an internal array directly. The object loses control over invariant-preserving transitions. Restore ownership through snapshots, immutable values, or semantic operations.

## 116. Testing — Preconditions

Test normal values, exact boundaries, just-inside and just-outside boundaries, invalid types, missing values, and unauthorized callers.

## 117. Testing — Postconditions

Assert semantic state relations, not accidental implementation details. For a transfer, test conservation of total balance and correct account deltas.

## 118. Testing — Invariants

Run invariant checks after successful state transitions. For high-risk domains, test both expected paths and generated command sequences.

## 119. Testing — Contract Matrix

Apply the same semantic suite to:

```text
in-memory implementation
integration implementation
cached implementation
provider adapter
```

This detects behavioral drift between implementations that claim the same port contract.

## 120. Testing — Mutation Resistance

A strong suite should fail when meaningful rules are weakened, such as changing `>` to `>=`, removing a version check, or skipping an authorization branch.

## 121. Interview — Invariant vs Preconditions

A precondition is operation-specific. An invariant is a valid-state property that persists across operations. A postcondition describes a successful result.

## 122. Interview — Why Interfaces Are Not Enough

An interface mainly expresses shape. Behavioral contracts include mutation, errors, ordering, retryability, security, consistency, and side effects that a simple method signature may not encode.

## 123. Interview — Why TypeScript Does Not Prove JSON

Types do not validate runtime data arriving from outside the checked program. Schema validation or equivalent runtime checks are needed at trust boundaries.

## 124. Interview — Why Setters Can Be Dangerous

Setters can enable invalid transitions because they expose state assignment without enforcing related preconditions, authorization, audit, concurrency, and side-effect rules.

## 125. Interview — Why Events Are APIs

Consumers depend on event fields and semantics without sharing the producer's internal implementation. Event changes therefore require compatibility discipline.

## 126. Interview — Why Idempotency Matters

Retries happen because networks fail. A timeout can happen after a successful side effect. Idempotency makes the repeated logical request safe.

## 127. Interview — Exactly Once

Treat exactly-once as a strong claim that needs architectural evidence. A common practical design is at-least-once delivery with idempotent consumers and durable deduplication.

## 128. Interview — Principal-Level Guarantee

Ask whether a guarantee is valuable to consumers, enforceable by the architecture, observable in production, affordable, and compatible with future change.

## 129. Mastery Exercise — Money

Implement immutable Money with currency checks, explicit precision and rounding, value equality, and contract tests.

## 130. Mastery Exercise — Inventory

Implement reserve, release, sell, and adjust. Protect non-negative quantity, tenant isolation, optimistic versioning, error semantics, and retriable requests.

## 131. Mastery Exercise — Lifecycle

Implement DRAFT, CONFIRMED, PAID, SHIPPED, CANCELLED with explicit allowed transitions and invariant checks after every valid transition.

## 132. Mastery Exercise — Repository Contract

Write one behavioral test suite and run it against memory and database implementations. Document which consistency and transaction semantics the interface intentionally does and does not promise.

## 133. Mastery Exercise — Provider Adapter

Normalize two hypothetical payment providers with different statuses, units, and errors into one application-facing contract. Identify information that is intentionally preserved and intentionally discarded.

## 134. Mastery Exercise — API Evolution

Evolve an API from `getUser(id)` to `getUser(query)` while preserving valid old consumers through an explicit migration strategy. Identify every semantic compatibility risk.

## 135. Principal Scenario — Final Item Reservation

Two actors attempt to reserve one unit simultaneously. Design the invariant, authority boundary, concurrency strategy, idempotency semantics, error contract, and audit evidence. Defend why your mechanism is sufficient rather than merely common.

## 136. Principal Scenario — Payment Timeout

A provider times out after the request is sent. Define the contract for client response, idempotency key, reconciliation, retry, provider status lookup, and eventual user-visible state.

## 137. Principal Scenario — Tenant Isolation

A user has access to branch A and requests branch B. Define authorization, repository scoping, cache-key isolation, event tenant propagation, and audit requirements. Treat isolation as a system invariant, not only a controller check.

## 138. Principal Scenario — Cache Freshness

Browsing may tolerate stale prices while checkout cannot. Split the read contracts so each operation has a freshness level appropriate to its correctness requirements.

## 139. Principal Scenario — Event Replay

A consumer receives the same OrderCreated event twice. Define event identity, deduplication, idempotent processing, ordering assumptions, and replay behavior.

## 140. Anti-Pattern — Validate Only in Controller

It fails when background jobs, scripts, repositories, or alternate entry points bypass the controller. Core domain invariants need enforcement closer to domain truth.

## 141. Anti-Pattern — TypeScript Will Catch It

It will not catch arbitrary invalid runtime JSON, database corruption, stale state, authorization mistakes, or provider semantic differences.

## 142. Anti-Pattern — Database Is Enough

Database constraints are excellent backstops but cannot replace every business semantic or distributed workflow contract.

## 143. Anti-Pattern — Generic Error Everywhere

Generic errors force callers to infer retryability, conflict, validation, and authorization behavior from unstable text.

## 144. Anti-Pattern — Generic Patch

`update(patch)` can permit forbidden fields and invalid state jumps. Sensitive transitions often deserve semantic commands.

## 145. Anti-Pattern — Hidden Global Context

Global tenant, user, clock, and configuration state weaken explicit contracts and make behavior harder to test and reason about.

## 146. Anti-Pattern — Accidental Contracts

Unspecified ordering, exact error text, cache freshness, or side effects can become de facto API promises once consumers rely on them.

## 147. Contract Review — Correctness

Can the implementation preserve the stated invariant for every accepted transition, including concurrent and retried cases?

## 148. Contract Review — Performance

Does enforcement introduce an accidental N+1 path, excessive copying, unbounded work, or a hidden latency regression?

## 149. Contract Review — Security

Can a caller bypass authorization or tenant scope through alternate paths, caches, events, or bulk operations?

## 150. Contract Review — Memory

Does the ownership model create unnecessary copies, retained references, or duplicated cached state?

## 151. Contract Review — Reliability

What happens after timeout, dependency failure, duplicate delivery, partial commit, or process restart?

## 152. Contract Review — Observability

Can metrics, logs, and traces demonstrate that important contracts are holding? Do violations become actionable signals?

## 153. Contract Review — Maintainability

Is the contract precise enough that another engineer can change implementation without guessing which behaviors are relied upon?

## 154. Contract Review — Scalability

Which invariants become expensive to enforce as data and concurrency grow? Can the contract be preserved without global coordination?

## 155. Contract Review — Future Change

Which future implementations does the contract permit, and which does it accidentally prohibit?

## 156. Completion Gate — Understand

Define contract, precondition, postcondition, representation invariant, and domain invariant without notes.

## 157. Completion Gate — Explain

Explain why TypeScript types do not prove runtime input validity, why events are APIs, and why idempotency matters under retries.

## 158. Completion Gate — Predict

Predict behavior for aliasing, shallow copies, private state, lifecycle transitions, and repeated commands before execution.

## 159. Completion Gate — Implement

Build an invariant helper, value object, state machine, tenant-aware repository, and idempotent command processor.

## 160. Completion Gate — Debug

Trace an incident back to the first violated contract instead of only fixing the final symptom.

## 161. Completion Gate — Apply

Use the framework on a realistic jewellery ERP inventory, reservation, sale, and payment workflow.

## 162. Completion Gate — Compare

Compare alternatives such as Result vs exception, mutable vs immutable, optimistic vs pessimistic concurrency, local vs global consistency, and adapter vs vendor leakage.

## 163. Completion Gate — Defend

Defend your choices with correctness, performance, memory, security, reliability, maintainability, scalability, observability, developer experience, operational complexity, and future change.

## 164. Revision / Retrieval Record

```text
Understand → Explain → Predict → Implement → Debug → Apply → Compare → Defend
```

Reading alone does not mark mastery.

## 165. Canonical References and Source Discipline

Use the ECMAScript specification for language semantics; TypeScript documentation/compiler behavior for TypeScript; protocol standards for HTTP; database documentation for engine-specific transaction and locking semantics; and actual application/domain documentation for business invariants. Distinguish standardized behavior from implementation-specific behavior and provider behavior from domain semantics.

## 166. Completion Snapshot

```text
[+] contracts and invariants
[+] preconditions and postconditions
[+] Design by Contract / Hoare-style reasoning
[+] representation vs domain invariants
[+] runtime validation and TypeScript limits
[+] ownership, aliasing, and mutability
[+] lifecycle, temporal, and protocol contracts
[+] idempotency, retry, concurrency, and consistency
[+] transaction, event, API, and persistence contracts
[+] security and tenant isolation
[+] contract testing and state-machine testing
[+] principal-level design judgment
```

## 167. Principal Decision Framework

```text
Correctness
Performance
Memory
Security
Reliability
Maintainability
Scalability
Observability
Developer Experience
Operational Complexity
Future Change
```

For every important guarantee ask: what consumer problem does it solve, can the architecture enforce it, what evidence proves it, what does it cost, and what future change does it constrain?

## 168. Compact Mental Checklist

```text
□ What must callers provide?
□ What may callers observe?
□ What must always remain true?
□ Where is each invariant enforced?
□ What happens on absence?
□ What happens on failure?
□ Can the operation be retried?
□ What happens under concurrency?
□ What can be stale?
□ Who owns mutable data?
□ What is the side-effect contract?
□ What is the authorization contract?
□ What is atomic?
□ What happens after timeout?
□ How does the contract evolve?
□ What evidence proves the contract?
```

## 169. Final Mental Model

```text
contract = observable promise
invariant = what must remain true
implementation = mechanism

stable design
  = small meaningful contract surface
  + explicit invariants
  + strong enforcement
  + clear failure semantics
  + safe evolution
```

## 170. Master Rule

> **Design the smallest contract that expresses the real semantic promise, then enforce the invariants at the boundary where you have enough authority to keep that promise true.**

**Next chapter:** Continue from stable contracts into deeper TypeScript-level design principles and reusable object-design decisions.
