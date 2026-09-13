# Chapter 21 — Liskov Substitution Principle and Behavioral Subtyping

> **Status:** `[~] In Progress`  
> **Part:** D — Core Design Principles  
> **Primary theme:** Liskov Substitution Principle (LSP), behavioral subtyping, substitutability, and contract preservation  
> **Tracks:** Track A — Core Theory · Track B — Implementation · Track C — Interview / Reasoning

---

# 1. Learning Objectives

By the end of this chapter, you should be able to:

1. Define the Liskov Substitution Principle precisely.
2. Explain why syntactic inheritance is not sufficient for safe substitution.
3. Distinguish subtype compatibility from behavioral compatibility.
4. Reason about preconditions, postconditions, and invariants across subtype boundaries.
5. Explain why a subtype should not strengthen required inputs unexpectedly.
6. Explain why a subtype should not weaken guaranteed outputs unexpectedly.
7. Identify contract violations caused by exceptions, nullability, mutability, ordering, timing, and side effects.
8. Detect classic inheritance smells such as inappropriate inheritance, refused bequest, fragile base class, and type-checking branches.
9. Apply LSP to JavaScript and TypeScript despite structural typing.
10. Understand function parameter variance and return-type variance at a practical level.
11. Use covariance, contravariance, and invariance reasoning when designing APIs.
12. Recognize the difference between substituting a value, an object, a capability, or an entire service.
13. Explain how composition can avoid inheritance-based LSP failures.
14. Design stable domain contracts that preserve substitutability.
15. Test behavioral subtyping with contract tests, state-machine tests, property-based tests, and negative tests.
16. Analyze LSP in persistence, caching, concurrency, security, multi-tenancy, and distributed systems.
17. Refactor invalid inheritance hierarchies safely.
18. Defend an LSP decision at principal-engineer level.

---

# 2. Prerequisites

You should already understand:

```text
Chapter 18 → stable contracts and invariants
Chapter 19 → Single Responsibility Principle
Chapter 20 → Open/Closed Principle
```

Also useful:

```text
inheritance
polymorphism
composition
interfaces
TypeScript structural typing
function types
generic types
error handling
state machines
```

---

# 3. What Is the Liskov Substitution Principle?

The Liskov Substitution Principle states, in practical design language:

> **Objects of a subtype should be usable wherever objects of the supertype are expected without breaking the correctness of the program.**

This is the essence of **behavioral substitutability**.

The critical phrase is:

```text
without breaking correctness
```

A subtype may have:

```text
different implementation
different performance
different internal state
additional capabilities
```

while still being substitutable.

But it must preserve the meaningful contract of the base abstraction.

---

# 4. Why LSP Exists

Inheritance creates an expectation:

```text
Base type
   ↑
Subtype
```

Consumers reason using the base type.

If the subtype silently changes the meaning of operations, the abstraction is unsafe.

Example:

```ts
interface Account {
  withdraw(amount: Money): void;
}
```

If a subtype says:

```text
withdraw()
throws for every positive amount
```

when the base contract allows valid withdrawals, the subtype is not behaviorally substitutable.

---

# 5. Syntax Is Not Semantics

This can compile:

```ts
class SavingsAccount extends Account {}
```

That does not prove LSP.

Structural typing can also accept an object:

```ts
const account: Account = specialAccount;
```

The compiler may verify shape.

LSP asks whether:

```text
consumer assumptions
```

remain valid.

---

# 6. Type Compatibility vs Behavioral Compatibility

Type compatibility:

```text
method exists
parameter types line up
return type lines up
```

Behavioral compatibility:

```text
preconditions remain usable
postconditions remain valid
invariants remain preserved
errors remain compatible
side effects remain compatible
timing remains acceptable
security remains valid
```

LSP is mostly about the second category.

---

# 7. The Core Contract Model

For a base contract:

```text
{P} operation {Q}
```

a useful substitution rule is:

```text
Subtype must accept every valid base input
and produce behavior satisfying the base postconditions.
```

In simplified reasoning:

```text
subtype precondition
  should not be stronger

subtype postcondition
  should not be weaker
```

This is a practical summary, not the entire formal theory.

---

# 8. Preconditions and LSP

Suppose base contract allows:

```text
amount > 0
```

Subtype changes requirement to:

```text
amount >= 100
```

A caller that was valid under the base contract now fails.

The subtype strengthened the precondition.

That breaks substitution.

---

# 9. Postconditions and LSP

Suppose base contract says:

```text
save()
ensures entity is persisted
```

A subtype that returns successfully but only stores data in memory weakens the postcondition.

The consumer can no longer rely on the base promise.

---

# 10. Invariants and LSP

The subtype must preserve the meaningful invariants consumers expect from the base abstraction.

Example:

```text
Repository.getById()
```

base contract:

```text
returns a record belonging to the authorized tenant
```

A subtype that can return cross-tenant data breaks a critical invariant.

---

# 11. Exceptions and LSP

A subtype can violate substitution through errors.

Base:

```ts
read(id): User | null
```

Subtype:

```ts
read(id): User
```

and throws `NotFoundError` on absence.

Even if the TypeScript signature can be arranged to look compatible, the runtime semantics differ.

Error behavior is part of the contract.

---

# 12. Nullability and LSP

Changing:

```text
value may be absent
```

to:

```text
value must exist
```

can sometimes strengthen guarantees safely.

Changing in the opposite direction:

```text
base guarantees a value
subtype may return null
```

weakens the contract.

Subtyping must be evaluated in the direction of the promised behavior.

---

# 13. Return Types and LSP

A subtype may often return a more specific result when that still satisfies the base contract.

Conceptually:

```text
Base returns Animal
Subtype returns Dog
```

This is covariance.

The consumer asked for an `Animal` and receives a valid `Animal`.

---

# 14. Parameter Types and LSP

For method inputs, a subtype must not unexpectedly demand a narrower domain.

Conceptually:

```text
Base accepts Animal
Subtype accepts only Dog
```

A caller passing a valid `Cat` through the base contract can now fail.

This is why function parameter positions are associated with contravariant reasoning.

---

# 15. A Practical Rule

For a consumer:

```ts
function process(x: Base) {
  x.operation(input);
}
```

The subtype should behave correctly for every situation in which `Base` promised correctness.

Think:

```text
Base promise
    ↓
Subtype preserves promise
```

---

# 16. Behavioral Subtyping

Behavioral subtyping means:

```text
Subtype can stand in for Base
without forcing Base consumers to know the subtype.
```

If consumers need:

```ts
if (thing instanceof SpecialThing) ...
```

to use the abstraction correctly, the base contract may be inadequate.

---

# 17. Type Checks as LSP Smell

Example:

```ts
function render(shape: Shape) {
  if (shape instanceof Circle) ...
  if (shape instanceof Square) ...
}
```

This may indicate that the abstraction does not capture the behavior consumers need.

But exhaustive handling of a deliberately closed domain can be valid.

Again:

```text
smell ≠ proof
```

---

# 18. Classic Example — Rectangle/Square

The classic teaching example says:

```text
Rectangle
  width and height vary independently
```

while:

```text
Square
  width = height
```

If a consumer relies on independently setting width and height, substituting a square changes semantics.

The lesson is not merely:

```text
Square should not extend Rectangle.
```

The deeper lesson is:

> An inheritance relationship is invalid when the subtype cannot preserve the behavioral assumptions of the base abstraction.

---

# 19. The Real Question

Do not ask:

```text
Is Square mathematically a Rectangle?
```

Ask:

```text
Can Square safely satisfy the software contract of Rectangle?
```

Domain taxonomy and software substitutability are not identical.

---

# 20. Capability vs Identity

An object can have a conceptual identity relationship without being a valid subtype.

Example:

```text
Bird
Penguin
```

Biologically:

```text
Penguin is a Bird
```

But if:

```ts
interface Bird {
  fly(): void;
}
```

then:

```text
Penguin
```

cannot safely satisfy that abstraction.

The problem is the capability contract, not biology.

---

# 21. Better Modeling

Separate:

```text
Bird
Flyable
```

Then:

```ts
class Sparrow implements Bird, Flyable {}
class Penguin implements Bird {}
```

This is an example of designing abstractions around capabilities.

---

# 22. LSP and Interface Design

A poorly designed interface creates impossible subtypes.

Bad:

```ts
interface PaymentProvider {
  charge(): Promise<void>;
  refund(): Promise<void>;
  recurring(): Promise<void>;
}
```

A provider that cannot support recurring payments may be forced into fake behavior.

Better capability interfaces:

```text
Charger
Refunder
RecurringBilling
```

---

# 23. LSP and ISP

Interface Segregation Principle reduces LSP pressure.

A smaller interface asks less of implementations.

```text
smaller contract
→ fewer impossible substitutions
```

---

# 24. LSP and OCP

Open/Closed Principle depends on safe extension.

If every extension can violate the original contract, OCP is dangerous.

Therefore:

```text
OCP
  add implementation

LSP
  ensure implementation is substitutable
```

---

# 25. LSP and SRP

SRP gives focused responsibilities.

A subtype should preserve the contract of that focused responsibility.

If a subtype adds unrelated behavior or constraints, the abstraction may be poorly scoped.

---

# 26. LSP and DIP

Dependency Inversion encourages consumers to depend on abstractions.

LSP asks:

```text
Are implementations actually safe behind that abstraction?
```

DIP without LSP produces unsafe polymorphism.

---

# 27. LSP and Contracts

Chapter 18 established:

```text
contract = observable promise
```

LSP is fundamentally:

```text
preserve the observable promise across substitution
```

---

# 28. LSP and Invariants

Suppose:

```ts
interface Inventory {
  availableQuantity(): number;
  reserve(qty: number): void;
}
```

Base contract:

```text
availableQuantity() >= 0
reserve(qty) succeeds only when qty <= available
```

Every subtype must preserve those invariants.

---

# 29. LSP and State Machines

If base abstraction supports:

```text
A → B → C
```

a subtype that silently removes:

```text
B → C
```

may break consumers.

State transition semantics are part of substitutability.

---

# 30. Lifecycle Substitutability

A base object may promise:

```text
usable immediately after construction
```

A subtype requiring:

```text
initialize()
```

before every operation strengthens the lifecycle precondition.

Potential LSP violation.

---

# 31. Null Object and LSP

A Null Object can be substitutable when:

```text
“do nothing” is a valid implementation of the base contract.
```

It is not substitutable when:

```text
base contract requires a meaningful side effect.
```

---

# 32. Failure-as-Behavior

Failure is part of behavior.

If base says:

```text
missing file → FileNotFoundError
```

and subtype says:

```text
missing file → silently returns ""
```

the subtype changes semantics.

Even if both avoid crashing, substitution is broken.

---

# 33. Side Effects and LSP

Suppose:

```ts
interface Logger {
  log(message: string): void;
}
```

A subtype:

```text
sends the message externally to a public webhook
```

may technically satisfy the method shape.

But it may violate consumer assumptions about privacy, cost, timing, or side effects.

Contracts must define meaningful side-effect expectations.

---

# 34. Ordering and LSP

Base:

```text
listOrders()
```

may guarantee:

```text
createdAt descending
```

Subtype returning arbitrary order is behaviorally incompatible.

Ordering is often an overlooked LSP contract.

---

# 35. Timing and LSP

Base contract:

```text
operation completes within request deadline
```

Subtype that blocks for minutes may be behaviorally invalid even when values and types are correct.

Operational contracts can constrain substitutability.

---

# 36. Complexity and LSP

A subtype may be functionally correct but introduce unacceptable resource behavior.

Example:

```text
base lookup: O(1) expected
subtype lookup: O(n²)
```

If performance is contractual or operationally critical, substitution may fail.

Do not treat complexity as automatically contractual.

---

# 37. Memory and LSP

A subtype that retains all previously processed data may cause memory exhaustion while the base contract assumes bounded processing.

Again:

```text
only contractually relevant guarantees matter.
```

---

# 38. Security and LSP

A subtype that weakens authorization violates behavioral substitution.

Base:

```text
only current tenant may access resource
```

Subtype:

```text
allows global lookup
```

This is a serious LSP failure.

---

# 39. Tenant Isolation and LSP

If:

```ts
interface TenantRepository {
  findById(id: string): Promise<Entity | null>;
}
```

implicitly guarantees tenant scoping, every implementation must preserve it.

A cached subtype must not accidentally use:

```text
id
```

without:

```text
tenantId
```

in its cache key.

---

# 40. Cache Substitutability

Caching creates common LSP failures.

Base repository:

```text
fresh enough result
```

Cached repository:

```text
returns stale result indefinitely
```

If freshness matters to consumers, the cached implementation is not behaviorally equivalent.

---

# 41. Persistence Substitutability

Base repository:

```text
save is durable after success
```

In-memory implementation:

```text
save succeeds but data disappears on process restart
```

This may be valid for tests only if the test contract explicitly differs.

Do not pretend a fake has production durability semantics unless the contract does not include durability.

---

# 42. Fake Implementations and LSP

Test doubles are often intentionally weaker.

A fake can be useful if:

```text
the missing behavior is outside the test's contract.
```

But tests can become misleading if the fake violates important semantics.

---

# 43. Contract Testing

For all implementations:

```text
same contract suite
```

Example:

```ts
function repositoryContract(
  create: () => UserRepository
) {
  const repo = create();

  // shared behavior assertions
}
```

This provides executable evidence of LSP.

---

# 44. Consumer Contract

A consumer defines what it actually relies upon.

Suppose:

```ts
function checkout(gateway: PaymentGateway) {}
```

Its real contract may require:

```text
charge is idempotent
errors are classified
result is authoritative
```

Not merely:

```text
charge() exists.
```

---

# 45. Consumer-Driven LSP

Ask:

> What assumptions does the consumer make about the base abstraction?

Then test whether all implementations preserve those assumptions.

---

# 46. Substitutability Boundary

Substitutability is always relative to:

```text
a contract
```

Not:

```text
all possible behaviors
```

A subtype may intentionally add capabilities.

It must preserve base expectations.

---

# 47. Stronger Guarantees

A subtype can sometimes provide a stronger guarantee.

Example:

```text
Base:
  getUser() may return cached data up to 5 seconds stale.

Subtype:
  getUser() is always authoritative.
```

This may preserve substitutability because the subtype satisfies the weaker base guarantee.

---

# 48. Stronger Preconditions

Usually unsafe:

```text
Base:
  accepts any positive quantity.

Subtype:
  only accepts multiples of 10.
```

The subtype cannot safely reject inputs the base allowed.

---

# 49. Stronger Postconditions

Often safe:

```text
Base:
  result is a User.

Subtype:
  result is an AdminUser.
```

provided the stronger result still satisfies the base contract.

---

# 50. Weaker Postconditions

Unsafe:

```text
Base:
  save means durable persistence.

Subtype:
  save means local cache update.
```

The subtype no longer guarantees what the base promised.

---

# 51. Invariant Strength

A subtype may impose stronger internal invariants.

But callers using the base contract must not become unable to invoke valid base operations.

This is a subtle boundary.

---

# 52. Subtype-Specific Invariants

Example:

```text
Base Product:
  price >= 0

Subtype LuxuryProduct:
  price >= 100000
```

If base consumers can legitimately create a product at price 5000, the subtype is not substitutable as a general `Product`.

The subtype may need a different abstraction or be used under a narrower contract.

---

# 53. Narrower Domain vs Subtype

A type can be a legitimate **specialized domain** without being a subtype.

Use:

```text
specialized value
```

rather than:

```text
subclass
```

when substitution does not hold.

---

# 54. Inheritance for Reuse

A common mistake:

```text
extends
```

is used only to reuse code.

Code reuse does not prove subtype semantics.

Composition often better expresses reuse.

---

# 55. Inheritance for Representation

Another smell:

```text
subclass only to inherit fields
```

If behavioral substitutability does not exist, inheritance is misleading.

---

# 56. Inheritance for Convenience

Avoid:

```text
I need these five helper methods
therefore extends Base.
```

This creates accidental coupling to future base behavior.

---

# 57. Fragile Base Class

Changes to a base class can silently break subclasses.

Examples:

```text
new invariant
new hook ordering
new method override
new default behavior
```

LSP makes this risk visible.

---

# 58. Template Method and LSP

Template Method relies on subclass hooks.

Base class must specify:

```text
hook preconditions
hook postconditions
ordering
allowed side effects
```

Otherwise subclasses can break the algorithm's assumptions.

---

# 59. Constructor Substitutability

A base construction contract may imply:

```text
new Base(config)
```

works for all valid configurations.

Subclass constructors with additional mandatory constraints can undermine substitutability depending on how the objects are constructed.

Prefer factories or explicit creation contracts when necessary.

---

# 60. Static Factory and LSP

Factories can select implementations without exposing inheritance details:

```ts
function createPaymentGateway(config): PaymentGateway
```

The returned object must still satisfy the behavioral contract.

Factory abstraction does not remove LSP requirements.

---

# 61. Structural Typing and Accidental Subtypes

In TypeScript:

```ts
type Reader = {
  read(id: string): Promise<User>;
};
```

An object with that method may be assignable.

But semantic substitution depends on behavior.

Structural typing makes accidental acceptance easy.

---

# 62. TypeScript's Structural Model

TypeScript focuses heavily on:

```text
shape
```

LSP focuses on:

```text
behavior
```

Therefore good TypeScript design needs:

```text
type modeling
+
runtime contract modeling
+
tests
```

---

# 63. Function Variance Intuition

Consider:

```ts
type Consumer<T> = (value: T) => void;
```

A consumer of a broader type can often handle values a narrower consumer cannot.

Thus function parameter positions have contravariant intuition.

---

# 64. Covariance Intuition

```ts
type Producer<T> = () => T;
```

A producer of a narrower subtype can often be used where a producer of a broader type is expected.

This is covariance.

---

# 65. Invariance Intuition

A mutable container:

```ts
Box<T>
```

may need invariance because both reading and writing occur.

If:

```text
Box<Dog>
```

were treated as:

```text
Box<Animal>
```

a consumer might write a `Cat` into a box that expects only Dogs.

---

# 66. Why Variance Matters to LSP

Variance controls whether substitutions are type-safe at the function/type level.

Behavioral subtyping extends beyond compiler variance.

You need both:

```text
type-level compatibility
+
semantic compatibility
```

---

# 67. Method Bivariance Caveat

TypeScript has special assignability behavior in some method and callback positions.

Do not use compiler acceptance as proof of semantic substitutability.

Understand the actual variance and runtime behavior of the API.

---

# 68. Mutable Collections and LSP

A base interface:

```ts
interface Inventory {
  items: Item[];
}
```

allows a subtype to expose mutation.

A subtype returning a read-only view may be incompatible if callers rely on mutation.

Conversely, a subtype exposing more mutation can violate invariants.

Collection mutability is part of the contract.

---

# 69. Readonly as Substitutability Tool

A read-only base contract can narrow responsibilities:

```ts
interface Catalog {
  readonly items: readonly Product[];
}
```

Implementations can provide stronger internal mutation protection without exposing it.

---

# 70. LSP and Equality

If base objects promise:

```text
value equality
```

a subtype changing equality semantics can break collections and callers.

Identity/equality is part of behavior.

---

# 71. LSP and Hash/Key Behavior

A subtype used as a key must preserve assumptions around:

```text
equality
hashing/canonical key
stability
```

where the environment defines such contracts.

---

# 72. LSP and Serialization

If a base object is serializable:

```text
serialize()
```

a subtype should not silently produce a shape that consumers cannot interpret.

Serialization compatibility is a behavioral contract.

---

# 73. LSP and Deserialization

A subtype-specific deserializer should not accept less input than the base contract unless consumers already know they are using the subtype.

---

# 74. LSP and Error Taxonomy

If base consumers expect:

```text
NotFound
Conflict
Transient
```

a subtype should not collapse everything into:

```text
UnknownError
```

unless the base contract only promised generic failure.

---

# 75. LSP and Retryability

If base operation is retry-safe:

```text
subtype must remain retry-safe.
```

If subtype introduces duplicate effects, substitution is broken.

---

# 76. LSP and Idempotency

Example:

```text
Base PaymentGateway:
  same idempotency key → same logical outcome
```

A subtype that charges twice for repeated requests violates the contract.

---

# 77. LSP and Concurrency

Base repository:

```text
optimistic version check
```

Subtype:

```text
last write wins
```

This can break callers that rely on conflict detection.

Concurrency semantics are substitutability concerns.

---

# 78. LSP and Atomicity

Base:

```text
operation is atomic
```

Subtype:

```text
partially applies updates before failing
```

The subtype weakens the contract.

---

# 79. LSP and Consistency

Base:

```text
read-your-writes
```

Subtype:

```text
may return stale replica data immediately after write
```

Not substitutable if consumers rely on read-your-writes semantics.

---

# 80. LSP and Event Ordering

Base publisher:

```text
EventA before EventB
```

Subtype:

```text
EventB before EventA
```

Consumers may break.

---

# 81. LSP and Delivery Guarantees

Base:

```text
at-least-once
```

Subtype:

```text
at-most-once
```

is a semantic change.

The direction of guarantee matters.

---

# 82. LSP and Performance

Performance only affects LSP when the base contract or surrounding operational requirements include it.

Do not invent a performance contract accidentally.

---

# 83. LSP and Timeouts

If base API guarantees deadline-aware behavior, subtype must respect it.

An extension that ignores cancellation can violate the operational contract.

---

# 84. LSP and Cancellation

Base:

```ts
operation(signal)
```

consumer expects cancellation.

Subtype ignores:

```ts
signal
```

and continues consuming resources.

This can violate substitutability even when result values are correct.

---

# 85. LSP and Resource Ownership

Base contract:

```text
caller owns returned stream and can close it
```

Subtype returns:

```text
shared global stream that caller must not close
```

Ownership semantics differ.

---

# 86. LSP and Thread/Concurrency Safety

In JavaScript server runtimes, “thread safety” often appears as:

```text
concurrent requests
shared mutable state
async interleaving
worker communication
```

A subtype must preserve any concurrency guarantees of the base abstraction.

---

# 87. Singleton Subtype Risk

Base is stateless/request-safe.

Subtype stores:

```ts
currentRequest
```

on instance fields.

A shared singleton subtype can leak state across requests.

Behavioral substitution fails under concurrency.

---

# 88. LSP and Tenant Context

Base service:

```text
every operation uses request tenant
```

Subtype reads a global:

```text
currentTenant
```

and can race across requests.

This is a subtle but severe substitutability failure.

---

# 89. LSP and Authorization

Base:

```text
caller cannot update records outside scope
```

Subtype:

```text
administrator bypass accidentally applied to all callers
```

The subtype weakens security guarantees.

---

# 90. LSP and Information Disclosure

Even if operation succeeds correctly, a subtype returning extra sensitive data can violate the base security contract.

More information is not always a stronger postcondition.

---

# 91. LSP and Audit

Base:

```text
every financial update produces an audit record
```

Subtype:

```text
updates data without audit
```

The behavior is not substitutable.

---

# 92. LSP and Observability

Base service may guarantee:

```text
correlation ID propagated
```

Subtype drops it.

Tracing behavior can be part of an operational contract where required.

---

# 93. LSP and Logging Side Effects

A subtype logging secrets that base implementations do not is a behavioral/security difference.

Side-effect freedom or data-handling constraints can be contractual.

---

# 94. LSP and Environment Dependencies

Base:

```text
uses injected Clock
```

Subtype:

```text
calls system time directly
```

Tests may fail to control behavior.

If deterministic time is part of the contract, substitution breaks.

---

# 95. LSP and Randomness

Same principle for injected randomness.

A subtype that uses nondeterministic global randomness may violate reproducibility assumptions.

---

# 96. LSP and Determinism

Base:

```text
same input → same output
```

Subtype introduces:

```text
random result
```

This is not substitutable when determinism is part of the contract.

---

# 97. LSP and Caching

A subtype may add caching without breaking LSP if:

```text
freshness
ordering
mutation visibility
identity
```

remain compatible.

Caching is safe only when semantic guarantees are preserved.

---

# 98. LSP and Cache Invalidation

If base says:

```text
after update(), get() reflects new state
```

cached subtype must invalidate or bypass stale cache appropriately.

---

# 99. LSP and Persistence Transactions

Base repository:

```text
save(order)
  commits all required state atomically
```

Subtype:

```text
writes order
then line items
without transaction
```

If partial persistence is observable, substitution fails.

---

# 100. LSP and Database Isolation

Base may rely on:

```text
serializable / repeatable read / optimistic locking
```

A subtype using weaker semantics can break consumers.

---

# 101. LSP and Search Providers

Base:

```text
search returns all matching records within documented consistency
```

Subtype:

```text
search index omits recently created records
```

This is valid only if eventual consistency is part of the base contract.

---

# 102. LSP and API Pagination

Base:

```text
cursor remains valid under normal concurrent insertions
```

Subtype page-based pagination may violate the contract.

Pagination is not merely return shape.

---

# 103. LSP and Ordering

Any consumer relying on:

```text
stable ordering
```

will break when a subtype does not preserve it.

---

# 104. LSP and Limits

Base:

```text
maximum page size = 100
```

Subtype silently limits:

```text
10
```

This may strengthen an operational restriction and can violate substitution if callers are allowed to request 50 by the base contract.

---

# 105. LSP and Rate Limits

Base provider:

```text
100 calls/minute
```

Subtype:

```text
5 calls/minute
```

The subtype may not be substitutable where callers rely on base capacity.

Capacity guarantees become contractual only when consumers truly rely on them.

---

# 106. LSP and Quotas

Similar reasoning applies to:

```text
storage quota
memory quota
external API quota
```

---

# 107. LSP and Network Failures

A subtype can expose provider-specific failures differently.

Stable abstraction should normalize meaningful error behavior.

---

# 108. LSP and Adapters

Adapters are often safer than inheritance when integrating external systems.

```text
ProviderAAdapter implements PaymentGateway
ProviderBAdapter implements PaymentGateway
```

Each can preserve the stable contract independently.

---

# 109. LSP and Anti-Corruption Layer

An adapter can translate external semantics:

```text
provider errors
provider states
provider IDs
```

into stable application semantics.

This protects the substitution boundary.

---

# 110. LSP and Domain Services

A domain service implementation can be substituted when:

```text
domain contract remains stable
```

even if algorithms differ.

---

# 111. LSP and Strategy

Strategy implementations are classic LSP subjects:

```text
TaxPolicy
DiscountPolicy
ShippingPolicy
```

Each implementation must satisfy the same behavioral expectations.

---

# 112. LSP and Policy Preconditions

A strategy that rejects valid base inputs is not substitutable.

For example:

```text
TaxPolicy accepts all supported jurisdictions
```

Subtype supports only one.

It is not a general `TaxPolicy` unless the contract says it is a specialized capability.

---

# 113. LSP and Result Types

A stable result union helps make allowed outcomes explicit:

```ts
type ChargeResult =
  | { kind: "approved"; transactionId: string }
  | { kind: "declined"; reason: string }
  | { kind: "retryable"; retryAfterMs: number };
```

A subtype must return valid members of that contract.

---

# 114. LSP and Exceptions vs Results

Results make some semantic differences visible to the type system.

Exceptions require stronger documentation and tests.

Both can support LSP.

---

# 115. LSP and Nullability in TypeScript

Do not rely solely on compiler assignability.

A subtype can preserve a return type while changing runtime absence behavior in ways callers did not expect.

---

# 116. LSP and `unknown`

External inputs should be validated before reaching a polymorphic domain abstraction.

Otherwise different implementations may interpret malformed values inconsistently.

---

# 117. LSP and Branded Types

Branded types can make valid input domains clearer:

```ts
type PositiveQuantity = number & {
  readonly __brand: "PositiveQuantity";
};
```

Subtypes receiving already-validated domain values should not re-interpret them inconsistently.

---

# 118. LSP and Discriminated Unions

A closed union can prevent invalid substitution paths by making all valid states explicit.

---

# 119. LSP and `readonly`

Read-only contracts can make substitutability easier by preventing implementations from exploiting writable references unexpectedly.

---

# 120. LSP and `satisfies`

`satisfies` can check structural conformance while preserving the precise source type.

It does not prove behavioral substitutability.

---

# 121. LSP and Generics

Generic abstractions can create variance issues.

Example:

```ts
interface Producer<T> {
  produce(): T;
}
```

A producer of a subtype may be safe where a producer of a base type is expected.

---

# 122. LSP and Generic Consumers

```ts
interface Consumer<T> {
  consume(value: T): void;
}
```

The direction differs because the type appears in input position.

Use variance intuition to reason about what substitutions are safe.

---

# 123. LSP and Mutable Generic Containers

Mixed read/write containers are harder to substitute safely.

This is why read-only views can simplify type relationships.

---

# 124. LSP and `Array<T>`

Arrays are mutable.

Therefore substitution assumptions around:

```text
Array<Subtype>
Array<Base>
```

must be handled carefully.

Prefer:

```ts
readonly T[]
```

when consumers only need reading.

---

# 125. LSP and Function Parameters

Given:

```ts
type Handler<T> = (value: T) => void;
```

A handler that can accept every `Animal` can also handle `Dog`.

A handler that accepts only `Dog` cannot safely replace one expected to handle any `Animal`.

---

# 126. LSP and Function Returns

Given:

```ts
type Factory<T> = () => T;
```

A factory returning `Dog` can be used where `Animal` is expected.

---

# 127. LSP and Optional Methods

An optional method can weaken a contract.

If consumers expect:

```ts
refund()
```

but the subtype may omit it:

```ts
refund?: ...
```

the base abstraction should not promise universal refund support.

---

# 128. LSP and Capability Interfaces

Prefer:

```ts
Charger
Refunder
```

when capabilities differ.

This reduces substitutability pressure.

---

# 129. LSP and Partial Implementations

A partial implementation can be valid if the base contract explicitly represents:

```text
unsupported capability
```

For example:

```ts
Result<
  RefundResult,
  UnsupportedCapability
>
```

The contract must make that possibility explicit.

---

# 130. LSP and “Not Supported”

A subtype throwing:

```text
NotSupportedError
```

may be valid only when the base contract permits unsupported operations.

Otherwise it violates substitution.

---

# 131. LSP and Feature Flags

A subtype behind a feature flag that changes semantics can violate substitution if consumers are unaware.

Feature rollout should not silently weaken the base contract.

---

# 132. LSP and Configuration

Configuration-driven implementations remain substitutable only if every configuration mode preserves the contract.

---

# 133. LSP and Optional Configuration

Do not silently create modes where:

```text
base guarantees invariant A
config disables A
```

unless the abstraction contract explicitly allows the weaker mode.

---

# 134. LSP and Environment

A subtype depending on environment variables unavailable to the base implementation can change failure semantics.

Initialization contracts must remain compatible.

---

# 135. LSP and Time Zones

A date service base contract may specify:

```text
all returned instants are UTC
```

A subtype returning branch-local times breaks the contract.

---

# 136. LSP and Units

Base:

```text
weight is grams
```

Subtype:

```text
returns kilograms
```

This is a classic semantic substitution failure.

---

# 137. LSP and Money

Base:

```text
Money.amount is in minor currency units
```

Subtype:

```text
uses floating-point major units
```

Even if the type is `number`, semantics differ.

---

# 138. LSP and Rounding

Base:

```text
round half up
```

Subtype:

```text
banker's rounding
```

This can be a behavioral incompatibility when consumers rely on exact financial semantics.

---

# 139. LSP and Precision

A subtype using lower precision can weaken guarantees.

Financial abstractions should define numeric precision explicitly.

---

# 140. LSP and State Identity

A subtype should preserve identity semantics expected by consumers.

If base:

```text
entity identity remains stable
```

subtype should not regenerate ID unexpectedly.

---

# 141. LSP and Equality

A subtype changing equality from:

```text
value equality
```

to:

```text
reference equality
```

can break sets, maps, caches, and comparisons.

---

# 142. LSP and Serialization Identity

Serialized IDs must remain stable if consumers depend on them.

---

# 143. LSP and Event Identity

If base publisher guarantees:

```text
eventId stable across retries
```

subtype generating a new event ID every retry violates idempotency assumptions.

---

# 144. LSP and Duplicate Events

Consumer code relying on deduplication requires stable identity behavior from every implementation.

---

# 145. LSP and Workflow State

A subtype workflow must preserve allowed state transitions.

If base supports:

```text
PENDING → APPROVED
```

subtype silently jumps:

```text
PENDING → SETTLED
```

it may bypass required invariants.

---

# 146. LSP and Invariant Enforcement

A subtype must not bypass invariant-protecting methods by mutating shared representation.

---

# 147. LSP and Encapsulation

Base:

```ts
#state
```

subtype may implement separate storage, but exposed behavior must remain consistent.

Private representation can differ safely.

---

# 148. LSP and Hooks

Base hooks should define:

```text
when called
what state exists
what return means
what errors are allowed
```

Otherwise subclasses can accidentally violate lifecycle assumptions.

---

# 149. Fragile Hook Contract

A base method:

```ts
process() {
  this.before();
  this.execute();
  this.after();
}
```

is fragile if subclasses override methods and invalidate:

```text
state
ordering
exceptions
```

Template Method requires disciplined contracts.

---

# 150. LSP and `super`

A subtype override that forgets:

```ts
super.execute()
```

can bypass base invariant behavior.

Never assume inheritance automatically preserves lifecycle.

---

# 151. LSP and Override Rules

Before overriding a base method ask:

```text
What does the base promise?
What does the consumer assume?
Which invariants does the base establish?
Which side effects does it guarantee?
```

---

# 152. LSP and Protected Members

Protected mutable state is a fragile contract.

Subclasses can accidentally rely on internal representation.

Prefer protected behavior hooks over exposing too much mutable state.

---

# 153. LSP and Final Methods

When a method's contract must not be altered by subclasses, composition or architectural enforcement may be safer than inheritance.

JavaScript/TypeScript does not provide a universal `final` method mechanism comparable to some languages.

Use design boundaries intentionally.

---

# 154. LSP and Sealed Hierarchies

Sometimes you want:

```text
closed set of subclasses
```

especially for domain state modeling.

A closed union may communicate this more directly in TypeScript.

---

# 155. LSP and Discriminated State

Instead of:

```ts
class BaseOrder {}
class PaidOrder extends BaseOrder {}
class ShippedOrder extends BaseOrder {}
```

you may prefer:

```ts
type OrderState =
  | { kind: "paid"; ... }
  | { kind: "shipped"; ... };
```

when states are values rather than substitutable behaviors.

---

# 156. State Objects vs Type States

State-pattern objects can be useful when behavior changes substantially.

But each state object must satisfy its interface contract.

---

# 157. LSP and Strategy State

A state strategy may have different allowed operations.

If the base contract claims all operations are available, state-specific strategies can violate it.

Use narrower interfaces or explicit result states.

---

# 158. LSP and Visitor

Visitor patterns rely on a stable set of variants.

Adding new variants can require consumer modification.

This is a closed-world trade-off, not automatically an LSP failure.

---

# 159. Open vs Closed Subtyping

Open-world subtype hierarchy:

```text
new implementations expected
```

Closed-world union:

```text
all variants controlled centrally
```

Choose based on domain evolution.

---

# 160. LSP and Exhaustiveness

Exhaustive handling can protect correctness for closed sets.

Open polymorphism shifts correctness toward shared contracts and tests.

---

# 161. LSP and Composition

Composition avoids forcing an identity relationship.

Instead:

```ts
class Penguin {
  constructor(
    private readonly birdBehavior: BirdBehavior
  ) {}
}
```

The model can choose capabilities explicitly.

---

# 162. Composition for Reuse

If the goal is:

```text
reuse algorithm
```

composition is often safer than inheritance.

---

# 163. Composition for Policy

Policies can be injected:

```ts
class Order {
  constructor(
    private readonly pricing: PricingPolicy
  ) {}
}
```

No subtype hierarchy is required.

---

# 164. Composition and Substitutability

Composition can reduce LSP failures because each collaborator has a smaller contract.

---

# 165. LSP and Delegation

Delegation allows an object to expose only capabilities it can genuinely support.

---

# 166. Wrapper vs Subtype

A wrapper can adapt behavior:

```text
LegacyProvider
  ↓
PaymentGatewayAdapter
```

without pretending:

```text
LegacyProvider IS-A PaymentGateway
```

internally.

---

# 167. Adapter as Semantic Firewall

The adapter translates:

```text
legacy semantics
→ stable semantics
```

and prevents invalid behavior from leaking into the core.

---

# 168. LSP and Inheritance Smells

Watch for:

```text
instanceof branches
empty overrides
methods throwing NotSupported
ignored methods
surprising side effects
stronger validation
weaker guarantees
state bypass
subclass-specific preconditions
```

These are common indicators.

---

# 169. Empty Override

Example:

```ts
class ReadOnlyRepository extends Repository {
  save() {}
}
```

If base promises persistence, this is a contract violation.

---

# 170. Throwing Override

```ts
class ImmutableAccount extends Account {
  withdraw() {
    throw new Error("not supported");
  }
}
```

Valid only if the base contract already allows that operation to be unsupported.

---

# 171. Changing Error Type

```text
Base:
  InvalidInputError

Subtype:
  RangeError
```

This can break consumers that classify errors.

---

# 172. Changing Return Meaning

```text
Base:
  Promise<User | null>

Subtype:
  Promise<User> that returns placeholder user
```

This violates absence semantics.

---

# 173. Placeholder Values

Returning:

```ts
{
  id: "unknown"
}
```

instead of:

```text
null
```

may look convenient but silently changes the contract.

---

# 174. Ignoring Input

A subtype that silently ignores required base inputs can violate postconditions.

---

# 175. Side-Effect Amplification

Base:

```text
read()
```

Subtype:

```text
read() + writes audit state + sends network request
```

This may be incompatible if consumers rely on read purity or low latency.

---

# 176. Side-Effect Removal

Base:

```text
save() publishes event after commit.
```

Subtype:

```text
save() persists only.
```

If event publication is contractual, substitution fails.

---

# 177. Ordering Changes

Base:

```text
save → event
```

Subtype:

```text
event → save
```

can cause consumers to observe impossible states.

---

# 178. Transaction Changes

Base:

```text
transactional update
```

Subtype:

```text
partial writes
```

breaks atomicity.

---

# 179. Concurrency Changes

Base:

```text
optimistic conflict detection
```

Subtype:

```text
last write wins
```

weakens conflict semantics.

---

# 180. Security Changes

Base:

```text
tenant scope mandatory
```

Subtype:

```text
global data access
```

is a catastrophic substitution failure.

---

# 181. LSP Refactoring Strategy

When an inheritance relationship is invalid:

```text
1. identify base contract
2. identify violating subtype behavior
3. identify actual shared capability
4. extract focused interface
5. migrate consumer
6. replace inheritance with composition/adapter
7. preserve contract
8. test all implementations
```

---

# 182. Replace Inheritance with Composition

Before:

```ts
class Square extends Rectangle {}
```

After:

```ts
class Square {
  // square-specific invariant
}
```

If shared behavior exists:

```text
GeometryCalculator
```

can be composed.

---

# 183. Extract Capability Interface

Before:

```ts
interface Bird {
  fly(): void;
}
```

After:

```ts
interface Bird {}
interface Flyable {
  fly(): void;
}
```

Implement only applicable capabilities.

---

# 184. Extract Read/Write Interfaces

Before:

```ts
interface Repository {
  read();
  save();
}
```

If some implementations are read-only:

```ts
interface Reader {}
interface Writer {}
```

This reduces impossible subtype contracts.

---

# 185. Narrow Consumer Dependencies

If a consumer only needs:

```ts
read()
```

depend on:

```ts
Reader
```

not:

```ts
Repository
```

This can remove the need for unsupported methods.

---

# 186. LSP and Interface Segregation Refactor

Typical sequence:

```text
broad abstraction
→ identify actual consumer capabilities
→ split interfaces
→ migrate consumers
→ simplify implementations
```

---

# 187. LSP and Open/Closed Refactor

A valid extension seam should satisfy:

```text
new implementation
+
same contract
```

not:

```text
new implementation
+
special cases in every consumer
```

---

# 188. LSP Contract Matrix

Create:

| Contract | Base | Subtype A | Subtype B |
|---|---|---|---|
| valid inputs | ? | ? | ? |
| result | ? | ? | ? |
| errors | ? | ? | ? |
| ordering | ? | ? | ? |
| idempotency | ? | ? | ? |
| security | ? | ? | ? |

Any weakening needs deliberate review.

---

# 189. LSP Test Matrix

Test:

```text
happy path
boundary input
invalid input
missing data
failure
retry
concurrency
security
ordering
timeouts
```

across all implementations.

---

# 190. Contract Test Example

```ts
function repositoryContract(
  create: () => UserRepository
) {
  // same assertions
}
```

Run against:

```text
memory
SQL
cache
remote service
```

---

# 191. Property-Based LSP Testing

Generate valid base inputs:

```text
for every generated valid input
```

assert each subtype satisfies the same properties.

This is powerful when the input space is large.

---

# 192. State-Machine LSP Testing

Generate command sequences and assert:

```text
all implementations produce equivalent allowed state transitions
```

where equivalence is defined by the contract.

---

# 193. Differential Testing

Run:

```text
base/reference implementation
subtype implementation
```

on the same generated inputs.

Compare:

```text
observable outcome
```

This is useful for deterministic algorithms and adapters.

---

# 194. Differential Testing Caveat

Different implementations may legally produce:

```text
different internal traces
different performance
```

while still satisfying the same contract.

Compare semantics, not implementation details.

---

# 195. Mutation Testing for LSP

Intentionally mutate an implementation:

```text
remove validation
change error
skip update
alter ordering
```

A strong contract suite should catch the violation.

---

# 196. Mocking and LSP

Mocks can accidentally define unrealistic contracts.

A mock returning:

```text
success for everything
```

may hide a subtype that fails under real conflicts.

Prefer contract-compliant fakes.

---

# 197. LSP and Integration Tests

Adapters should be tested against real provider behavior where important.

A compile-safe adapter can still violate the domain contract.

---

# 198. LSP and Golden Masters

Golden masters can preserve behavior across implementation swaps.

Review whether captured behavior is contractual or accidental.

---

# 199. LSP and Characterization Tests

Before changing an inheritance hierarchy:

```text
capture existing consumer-observable behavior
```

Then refactor and compare.

---

# 200. LSP and Error Tests

Test exact categories:

```text
NotFound
Conflict
Unauthorized
Transient
Validation
```

when categories are contractual.

---

# 201. LSP and Security Tests

Verify:

```text
cross-tenant access denied
unauthorized mutation denied
sensitive field not exposed
```

across every subtype.

---

# 202. LSP and Performance Tests

Only test performance as LSP behavior when:

```text
the contract actually requires it
```

Otherwise performance is an implementation concern.

---

# 203. LSP and Resource Tests

For streaming or plugin abstractions, test:

```text
close
cancel
dispose
```

semantics across implementations.

---

# 204. LSP and Lifecycle Tests

For lifecycle interfaces:

```text
start
stop
restart
failure during start
failure during stop
```

must behave according to the same contract.

---

# 205. LSP and Async Contracts

Promises have behavioral semantics:

```text
resolve value
reject error
settlement timing
cancellation
side-effect completion
```

A subtype must preserve required parts.

---

# 206. LSP and “Await Means Complete”

If base contract says:

```text
await save() means durable commit is complete
```

subtype cannot resolve before commit merely because the method type is:

```ts
Promise<void>
```

---

# 207. LSP and Streams

Stream interfaces introduce:

```text
backpressure
close
error
ordering
partial data
```

All are part of substitutability.

---

# 208. LSP and Iterators

An iterator contract includes:

```text
done
value
ordering
side effects
termination
```

A subtype producing duplicate or reordered items can be behaviorally incompatible.

---

# 209. LSP and Generators

Generators may encode:

```text
lazy evaluation
resource lifetime
exception behavior
```

Substitutable implementations must respect the contract.

---

# 210. LSP and Async Iteration

Async iterators add:

```text
await
backpressure
cancellation/return
errors
```

Do not compare them only by method shape.

---

# 211. LSP and Event Emitters

A subtype changing:

```text
event name
payload
ordering
duplication
```

breaks subscribers.

---

# 212. LSP and Domain Events

Event type is a behavioral contract.

A subtype should preserve stable facts when implementing an event publisher abstraction.

---

# 213. LSP and Repository Caches

A cached repository must preserve:

```text
identity
freshness
tenant scope
mutation visibility
```

according to the base contract.

---

# 214. LSP and Read Models

A read-model implementation can be substitutable if the base contract permits:

```text
eventual consistency
```

Otherwise it cannot replace an authoritative repository.

---

# 215. LSP and Search

Search providers can differ internally:

```text
SQL
Elasticsearch
in-memory
```

but all must preserve:

```text
filter semantics
ordering
pagination
freshness guarantees
```

defined by the contract.

---

# 216. LSP and External Providers

Adapters are safe substitutable implementations only if:

```text
amounts
currencies
errors
retry
idempotency
```

are normalized correctly.

---

# 217. LSP and Payment

A payment gateway contract might require:

```text
same idempotency key
→ no duplicate charge
```

Every provider adapter must preserve this, even when providers differ.

---

# 218. LSP and Refunds

A refund provider that supports only partial refunds cannot be a general implementation of:

```text
full + partial refund gateway
```

unless the contract explicitly models capability differences.

---

# 219. LSP and Capability Negotiation

Instead of invalid substitution:

```text
Gateway
```

can expose explicit capabilities:

```text
Charger
Refunder
PartialRefunder
```

Consumers depend only on what they need.

---

# 220. LSP and Multi-Tenant ERP

A stable inventory contract should include:

```text
tenant scope
branch scope
authorization
concurrency
```

Every implementation must preserve those semantics.

---

# 221. LSP and Branch Scope

If base says:

```text
branch inventory query only returns branch-owned stock
```

subtype returning organization-wide stock is not substitutable.

---

# 222. LSP and Pricing

A base:

```ts
PricingPolicy
```

may guarantee:

```text
price >= cost
```

if that is business-required.

A subtype returning below cost violates the contract.

---

# 223. LSP and Tax

Tax policy implementations can differ by jurisdiction, but each must follow the stable input/result contract.

---

# 224. LSP and Jewellery Weights

Base:

```text
weight in grams
```

subtype:

```text
weight in troy ounces
```

is a semantic contract violation without explicit conversion.

---

# 225. LSP and Purity

If:

```text
purity represented in fineness
```

a subtype returning karat number under the same type is not substitutable.

---

# 226. LSP and Inventory Reservation

Base:

```text
reserve succeeds only when stock is available.
```

Subtype:

```text
allows negative inventory
```

breaks the invariant.

---

# 227. LSP and Sale Workflow

Base:

```text
sale cannot finalize without required payment evidence.
```

Subtype:

```text
finalizes immediately
```

weakens the workflow invariant.

---

# 228. LSP and Audit

Base:

```text
financial mutation produces audit record.
```

Subtype:

```text
no audit
```

is not substitutable.

---

# 229. LSP and Approval Policies

Base:

```text
high-value sale requires manager approval.
```

Subtype:

```text
skips approval
```

breaks business/security behavior.

---

# 230. LSP and Role-Based Access

Base:

```text
only permitted roles may execute adjustment.
```

Subtype:

```text
allows any authenticated user.
```

breaks authorization contract.

---

# 231. LSP and Tenant Cache Keys

Base:

```text
get(tenantId, id)
```

cached subtype key:

```text
id
```

Two tenants share the same ID.

Identify the violated invariant.

---

# 232. LSP and Event Payloads

Base event:

```text
tenantId required
```

subtype publisher omits it.

Downstream tenant-aware consumers can no longer safely operate.

---

# 233. LSP and Observability Context

Base:

```text
correlationId propagated
```

subtype adapter drops it.

Debugging guarantees degrade.

---

# 234. LSP and Operational Guarantees

Substitutability may include:

```text
timeouts
resource limits
health checks
shutdown semantics
```

when consumers depend on them.

---

# 235. LSP and Health Checks

Base provider:

```text
healthCheck() accurately reports readiness.
```

Subtype always returns `true`.

Operational orchestration can make unsafe routing decisions.

---

# 236. LSP and Startup Contracts

Base:

```text
start() resolves only when ready.
```

Subtype resolves when initialization begins.

Consumers may send traffic too early.

---

# 237. LSP and Shutdown Contracts

Base:

```text
stop() resolves after resources are released.
```

Subtype resolves early.

Process orchestration may fail.

---

# 238. LSP and Backpressure

Base stream:

```text
respects consumer backpressure
```

subtype floods memory.

This can be a reliability contract violation.

---

# 239. LSP and Retry Policy

Base:

```text
retryable errors clearly classified.
```

subtype marks permanent failures retryable.

The consumer's retry behavior becomes unsafe.

---

# 240. LSP and Circuit Breakers

A subtype bypassing the base circuit breaker may create operational instability.

---

# 241. LSP and Timeouts

A subtype ignoring a caller-provided deadline can cause resource exhaustion.

---

# 242. LSP and Cancellation

A subtype ignoring cancellation can continue after the consumer believes the operation has ended.

---

# 243. LSP and Memory Leaks

A subtype retaining references beyond the lifecycle expected by the base contract can break long-running systems.

---

# 244. LSP and Hidden Global State

Global state is dangerous because substitution can depend on:

```text
which implementation ran before
```

rather than only on its explicit inputs.

---

# 245. LSP and Deterministic Tests

A subtype using hidden global state may pass individual tests but fail when run concurrently.

---

# 246. LSP and Isolation

An implementation is easier to substitute when behavior depends primarily on:

```text
explicit inputs
declared dependencies
stable state
```

rather than ambient context.

---

# 247. LSP and Dependency Injection

Injected dependencies should themselves be contract-compatible.

A chain of unsafe substitutions compounds risk.

---

# 248. LSP and Mock Injection

Tests that inject unrealistic mocks can hide LSP violations.

---

# 249. LSP and Contract-Centered Tests

Test consumers against the abstraction, then run the same tests for every implementation.

---

# 250. LSP and Reference Implementations

A simple reference implementation can provide a semantic baseline.

Use it carefully:

```text
different optimization
different persistence
```

may still satisfy the contract without matching internals.

---

# 251. LSP and Differential Semantics

Compare:

```text
observable output
state transition
error category
```

not:

```text
internal calls
private state
exact timing
```

unless those are contractual.

---

# 252. LSP and Logging

Internal logs usually are not substitutability requirements unless operational behavior depends on them.

---

# 253. LSP and Metrics

Metrics may be contractual if:

```text
SLO
billing
compliance
capacity planning
```

depends on them.

Otherwise do not over-constrain implementations.

---

# 254. LSP and Monitoring

Health and readiness semantics are often operational contracts.

---

# 255. LSP and API Clients

If a client depends on:

```text
HTTP status
error code
pagination
retry header
```

all compatible server implementations must preserve them.

---

# 256. LSP and HTTP Semantics

Changing:

```text
404
```

to:

```text
200 with empty object
```

can break clients even if the JSON looks reasonable.

---

# 257. LSP and GraphQL

Changing nullability:

```text
field!
→
field
```

weakens guarantees.

The reverse may break clients that send or expect null.

Schema compatibility is part of behavioral substitutability.

---

# 258. LSP and Event Schema

Subtype publishers must emit schemas compatible with the base event contract.

---

# 259. LSP and Persistence Schema

A repository implementation may map different database schemas while preserving domain behavior.

Schema differences are internal when correctly adapted.

---

# 260. LSP and ORM Differences

Different ORM implementations are substitutable only if behavior, not model shape, remains compatible.

---

# 261. LSP and Transactions

A subtype using a different transaction strategy can still be valid if callers see the same atomic behavior.

Implementation diversity is allowed.

---

# 262. LSP and Eventual Consistency

A subtype can be eventually consistent only when the base contract already permits that level of staleness.

---

# 263. LSP and Stronger Consistency

A stronger consistency implementation is often safe when it preserves all base-observable guarantees.

The reverse may not be safe.

---

# 264. LSP and Performance Optimizations

An optimization is safe if it preserves observable contract.

Caching:

```text
optimization
```

becomes a behavior issue only when it changes promised semantics.

---

# 265. LSP and Lazy Loading

Lazy loading can change:

```text
when errors occur
when network calls occur
performance
```

Consumers must not rely on behavior the subtype no longer preserves.

---

# 266. LSP and Eager Loading

Conversely, eager loading can increase cost or trigger errors earlier.

Timing and side effects can matter.

---

# 267. LSP and Resource Limits

If base contract limits:

```text
memory
payload size
concurrency
```

subtype must respect them where they are promised.

---

# 268. LSP and Security Boundaries

An adapter can be functionally correct yet unsafe if it bypasses security context.

Always test authorization separately.

---

# 269. LSP and Least Privilege

Different implementations should receive only the permissions required by the contract.

---

# 270. LSP and Secrets

A subtype should not expose secrets through methods that the base abstraction does not promise.

---

# 271. LSP and Data Minimization

Returning more data than expected can violate privacy/security even if the result is otherwise valid.

---

# 272. LSP and Audit Logs

Audit events may become part of the observable contract when required for compliance.

---

# 273. LSP and Multi-Branch ERP

A branch-specific implementation may be substitutable only for a contract scoped to that branch.

Do not substitute it for a globally capable repository unless the contract says so.

---

# 274. LSP and Specialized Services

A specialized class may be valid without being a subtype.

Example:

```text
BranchInventory
```

instead of:

```text
Inventory
```

if it has narrower semantics.

---

# 275. LSP and Domain Naming

Names such as:

```text
SpecializedRepository
RestrictedRepository
ReadOnlyAccount
```

should trigger the question:

```text
Is this truly a subtype or a different capability?
```

---

# 276. LSP and Refused Bequest

A subtype inherits methods it cannot meaningfully implement.

Example:

```ts
class ReadOnlyDocument extends MutableDocument {
  save() {
    throw new Error("unsupported");
  }
}
```

This is a strong LSP signal.

---

# 277. LSP and Inappropriate Intimacy

Subclasses reaching deeply into base internals creates coupling and fragile contracts.

---

# 278. LSP and Fragile Base

A base class change can alter subclass behavior unexpectedly.

Prefer composition when invariants are complex.

---

# 279. LSP and Temporal Coupling

Subclass hooks may rely on call order.

Document lifecycle contracts explicitly.

---

# 280. LSP and Protected State

Protected mutable fields increase the number of ways subclasses can violate base invariants.

---

# 281. LSP and Inheritance Depth

Deep inheritance increases the number of contracts a subtype must preserve.

A shallow hierarchy is easier to reason about.

---

# 282. LSP and Multiple Inheritance

JavaScript classes support only one class `extends`, though composition and mixins can emulate multiple behavior sources.

Mixins still require behavioral contract analysis.

---

# 283. LSP and Mixins

A mixin can introduce:

```text
methods
state
assumptions
```

without an explicit subtype contract.

Document required host capabilities.

---

# 284. LSP and Traits

Trait-like composition can be safer when capabilities are independent.

But semantic conflicts still need resolution.

---

# 285. LSP and Decorators

Decorators can alter:

```text
errors
timing
side effects
```

and therefore can break substitution if they change the base contract.

---

# 286. LSP and Middleware

Middleware can change behavior of every implementation.

Ordering and side effects matter.

---

# 287. LSP and Interceptors

An interceptor that retries a non-idempotent operation can make an otherwise valid implementation unsafe.

---

# 288. LSP and Resilience Wrappers

Retries, timeouts, circuit breakers, and fallbacks must preserve the intended contract.

---

# 289. LSP and Fallback Semantics

A fallback implementation must produce an outcome acceptable under the base contract.

Returning approximate data where authoritative data is required can violate substitution.

---

# 290. LSP and Graceful Degradation

Degraded behavior may be valid only when the base contract explicitly allows it.

---

# 291. LSP and Nullability Revisited

The absence semantics of:

```text
null
undefined
empty
exception
```

must remain stable.

---

# 292. LSP and String Semantics

A subtype that normalizes strings differently can break case sensitivity, whitespace, Unicode, or ordering assumptions.

Semantic normalization is part of contracts.

---

# 293. LSP and Unicode

Identifier or text-processing abstractions should specify normalization semantics where exact behavior matters.

---

# 294. LSP and Locale

A subtype using a different locale for comparison or formatting may be incompatible.

---

# 295. LSP and Currency Locale

Financial and regional formatting differences should not leak into a stable monetary contract.

---

# 296. LSP and Date Formatting

A base formatter promising ISO output cannot be substituted by one returning locale-specific output.

---

# 297. LSP and Time Representation

Machine-time contracts should use explicit semantics.

---

# 298. LSP and Object Identity

A subtype that clones objects instead of preserving identity can break consumers relying on identity semantics.

---

# 299. LSP and Mutation

A mutable base object replaced by an immutable subtype may or may not be substitutable.

If the base contract allows callers to mutate state, immutability changes the capability contract.

Prefer read/write interfaces that reflect actual expectations.

---

# 300. LSP and Readonly Upgrade

If consumers only read, making implementations immutable is usually safe.

This illustrates why narrow interfaces help.

---

# 301. Implementation — Base Contract

```ts
interface PaymentGateway {
  charge(input: ChargeInput): Promise<ChargeResult>;
}
```

Suppose the contract is:

```text
valid positive amount
supported currency
idempotency key required
retryable failures classified
approved result contains provider-independent transaction identity
```

---

# 302. Implementation — Good Adapter

```ts
class ProviderGateway implements PaymentGateway {
  constructor(
    private readonly provider: ProviderClient
  ) {}

  async charge(input: ChargeInput): Promise<ChargeResult> {
    // validate/translate
    // normalize provider response
    // preserve idempotency
    // map errors
    // return stable result
    throw new Error("example");
  }
}
```

The implementation details are provider-specific.

The contract is not.

---

# 303. Implementation — Bad Adapter

```ts
class BadGateway implements PaymentGateway {
  async charge(input: ChargeInput) {
    if (input.amount < 500) {
      throw new Error("minimum is 500");
    }

    return { providerSpecificId: "x" };
  }
}
```

Potential violations:

```text
stronger input precondition
wrong result semantics
provider leakage
```

---

# 304. Implementation — Readonly Capability

```ts
interface ProductReader {
  getById(id: ProductId): Promise<Product | null>;
}

class CachedProductReader implements ProductReader {
  // ...
}
```

The narrower contract reduces unnecessary LSP obligations.

---

# 305. Implementation — Capability Split

```ts
interface ReservationReader {
  getReservation(id: ReservationId): Promise<Reservation | null>;
}

interface ReservationWriter {
  create(...): Promise<Reservation>;
}
```

A read-only implementation can now safely satisfy the reader contract.

---

# 306. Implementation — Result Contract

```ts
type ReserveResult =
  | { kind: "reserved"; reservationId: string }
  | { kind: "insufficient"; available: number }
  | { kind: "conflict" };
```

All implementations must preserve this vocabulary.

---

# 307. Implementation — Contract Test

```ts
async function reservationContract(
  create: () => ReservationService
) {
  const service = create();

  // same semantic tests for every implementation
}
```

---

# 308. Implementation — Invariant Check

```ts
function assertInventoryInvariant(
  available: number,
  reserved: number
) {
  if (available < 0) {
    throw new Error("available quantity cannot be negative");
  }

  if (reserved < 0) {
    throw new Error("reserved quantity cannot be negative");
  }
}
```

A subtype must preserve the invariant.

---

# 309. Implementation — State Machine

```ts
type OrderState =
  | { kind: "draft" }
  | { kind: "confirmed" }
  | { kind: "paid" }
  | { kind: "shipped" }
  | { kind: "cancelled" };
```

When the state set is intentionally closed, explicit state modeling may be safer than inheritance.

---

# 310. Implementation — Strategy Contract

```ts
interface ShippingPolicy {
  calculate(input: ShippingInput): Money;
}
```

Good implementation:

```ts
class FlatRateShipping implements ShippingPolicy {
  calculate(input: ShippingInput): Money {
    return input.currency.amount(100);
  }
}
```

The implementation must still satisfy all domain semantics.

---

# 311. Implementation — Fake Repository

```ts
class InMemoryUserRepository implements UserRepository {
  // preserve absence semantics
  // preserve identity
  // preserve tenant isolation
  // preserve contract-level failures
}
```

Tests should not assume the fake has database durability unless that is explicitly part of the contract.

---

# 312. Implementation — Adapter Around Legacy

```ts
class LegacyAdapter implements UserRepository {
  constructor(
    private readonly legacy: LegacyClient
  ) {}

  async getById(id: string): Promise<User | null> {
    const result = await this.legacy.lookup(id);
    return result ?? null;
  }
}
```

The adapter translates rather than leaking legacy semantics.

---

# 313. Debugging Exercise 1 — Strengthened Precondition

Base:

```ts
interface Processor {
  process(value: number): number;
}
```

Subtype:

```ts
process(value) {
  if (value < 100) throw new Error();
  return value * 2;
}
```

Ask:

```text
What did the subtype assume that the base never promised?
```

Answer:

```text
minimum value 100
```

---

# 314. Debugging Exercise 2 — Weakened Postcondition

Base:

```text
save()
  success means durable persistence
```

Subtype:

```text
success means local cache write
```

Identify the weakened postcondition.

---

# 315. Debugging Exercise 3 — Error Contract

Base:

```text
missing user → null
```

Subtype:

```text
missing user → throw
```

Identify the semantic substitution failure.

---

# 316. Debugging Exercise 4 — Tenant Cache

Base:

```text
find(tenantId, id)
```

Cache subtype key:

```ts
`${id}`
```

Two tenants share the same ID.

Identify the violated invariant.

---

# 317. Debugging Exercise 5 — Retry

Base:

```text
same idempotency key is retry-safe
```

Subtype:

```text
creates a second charge
```

Identify the contract violation.

---

# 318. Debugging Exercise 6 — Ordering

Base:

```text
list()
  ordered by createdAt descending
```

Subtype:

```text
returns database natural order
```

Identify why the consumer can break.

---

# 319. Debugging Exercise 7 — Read Only

```ts
class ReadOnlyOrderRepository extends OrderRepository {
  save() {
    throw new Error("not supported");
  }
}
```

Determine whether inheritance is appropriate.

---

# 320. Debugging Exercise 8 — Side Effect

```ts
class CustomerReader {
  async getCustomer() {
    // read customer
    // unexpectedly sends welcome email
  }
}
```

Identify the hidden contract change if the base abstraction promises a read-only operation.

---

# 321. Debugging Exercise 9 — Timeout

Base:

```text
operation respects AbortSignal
```

Subtype ignores the signal.

Explain the resource and lifecycle consequences.

---

# 322. Debugging Exercise 10 — Authorization

Base:

```text
delete()
  allowed only with correct permission
```

Subtype:

```text
delete()
  skips policy check
```

This is an LSP and security failure.

---

# 323. Code Review Exercise

Review:

```ts
interface Account {
  withdraw(amount: number): void;
}

class NormalAccount implements Account {
  withdraw(amount: number) {
    if (amount <= 0) throw new Error();
  }
}

class LockedAccount implements Account {
  withdraw(amount: number) {
    throw new Error("account locked");
  }
}
```

Ask:

```text
Is LockedAccount a valid Account subtype?
```

The answer depends on whether the base contract permits permanently unavailable withdrawals.

If the interface promises only the operation shape, the semantic contract is under-specified.

---

# 324. Code Review Exercise — Better Contract

```ts
interface WithdrawableAccount {
  canWithdraw(amount: Money): boolean;
  withdraw(amount: Money): Result<void, WithdrawalError>;
}
```

Now the domain can explicitly represent:

```text
locked
insufficient funds
invalid amount
success
```

The contract becomes more honest.

---

# 325. Code Review Exercise — Bird Hierarchy

Bad:

```ts
interface Bird {
  fly(): void;
}

class Penguin implements Bird {
  fly() {
    throw new Error("cannot fly");
  }
}
```

Better:

```ts
interface Bird {}

interface FlyingBird extends Bird {
  fly(): void;
}
```

The capability contract becomes accurate.

---

# 326. Code Review Exercise — Repository

```ts
interface Repository<T> {
  get(id: string): Promise<T>;
  save(entity: T): Promise<void>;
}

class ReadOnlyRepository<T> implements Repository<T> {
  async get(id: string) { /* ... */ }

  async save(entity: T) {
    throw new Error("not supported");
  }
}
```

Design a safer abstraction.

---

# 327. Code Review Exercise — Cache

```ts
class CachedRepository implements Repository<Order> {
  async get(id: string) {
    return cache.get(id);
  }

  async save(order: Order) {
    await database.save(order);
  }
}
```

What if:

```text
cache is stale after save
```

Does the implementation satisfy the repository contract?

---

# 328. Code Review Exercise — Payment

```ts
class GatewayB implements PaymentGateway {
  async charge(input) {
    return provider.charge(input);
  }
}
```

Provider errors leak directly.

Determine whether behavioral compatibility exists.

---

# 329. Code Review Exercise — Collection

```ts
interface Cart {
  items: Item[];
}
```

Subtype returns:

```text
readonly Item[]
```

Ask:

```text
Can consumers mutate items under the base contract?
```

If yes, a read-only subtype may not be substitutable.

Better:

```ts
interface Cart {
  readonly items: readonly Item[];
}
```

---

# 330. Code Review Exercise — Lifecycle

```ts
interface Client {
  request(): Promise<Response>;
}

class LazyClient implements Client {
  async request() {
    if (!this.connected) {
      await this.connect();
    }
    return this.send();
  }
}
```

Could this be substitutable?

Potentially yes, if the base contract does not require:

```text
pre-connected state
```

The implementation detail itself is not an LSP problem.

---

# 331. Predict-the-Output Exercise 1

```ts
class Base {
  get(): number {
    return 1;
  }
}

class Derived extends Base {
  get(): number {
    return 2;
  }
}

const value: Base = new Derived();
console.log(value.get());
```

### Prediction

```text
2
```

Runtime dispatch selects the subtype implementation.

LSP asks whether that implementation preserves the base contract.

---

# 332. Predict-the-Output Exercise 2

```ts
class Base {
  run(x: number) {
    return x + 1;
  }
}

class Derived extends Base {
  run(x: number) {
    if (x < 0) throw new Error("negative");
    return x + 1;
  }
}

const runner: Base = new Derived();
console.log(runner.run(1));
```

### Prediction

```text
2
```

But:

```ts
runner.run(-1);
```

reveals the strengthened precondition.

---

# 333. Predict-the-Output Exercise 3

```ts
class Base {
  value() {
    return 10;
  }
}

class Derived extends Base {
  value() {
    return 20;
  }
}

const x: Base = new Derived();
console.log(x.value());
```

### Prediction

```text
20
```

A different result is not automatically an LSP violation.

The question is whether 20 remains within the base's promised result semantics.

---

# 334. Predict-the-Output Exercise 4

```ts
const handlers: Array<(x: number) => number> = [
  x => x + 1,
  x => x * 2,
];

console.log(handlers[1](3));
```

### Prediction

```text
6
```

Function substitution is governed by both type compatibility and behavioral contracts.

---

# 335. Predict-the-Output Exercise 5

```ts
interface Reader {
  read(): string;
}

const reader: Reader = {
  read() {
    return "ok";
  },
};

console.log(reader.read());
```

### Prediction

```text
ok
```

The object satisfies the structural contract.

Behavioral correctness still requires semantic agreement.

---

# 336. Mastery Exercise 1 — Detect LSP Violations

Find five inheritance relationships in a codebase.

For each:

```text
base contract
subtype assumptions
consumer assumptions
substitution result
```

Classify:

```text
valid
invalid
underspecified
```

---

# 337. Mastery Exercise 2 — Capability Refactor

Refactor:

```ts
Bird
FlyableBird
SwimmingBird
```

so impossible subtype implementations disappear.

---

# 338. Mastery Exercise 3 — Repository Contract

Define:

```text
Reader
Writer
TransactionalWriter
TenantReader
```

Only allow implementations to claim capabilities they actually provide.

---

# 339. Mastery Exercise 4 — Payment Providers

Create:

```text
PaymentGateway
ProviderAAdapter
ProviderBAdapter
FakeGateway
```

All must satisfy:

```text
idempotency
error semantics
currency semantics
amount semantics
authorization
audit
```

---

# 340. Mastery Exercise 5 — Cache Substitutability

Build:

```text
DatabaseReader
CachedReader
```

Define:

```text
freshness contract
invalidation contract
tenant-scoped key
fallback behavior
```

Then prove whether the cache is substitutable.

---

# 341. Mastery Exercise 6 — Jewellery Inventory

Design:

```text
InventoryRepository
BranchInventoryRepository
GlobalInventoryRepository
```

Determine which are genuine subtypes and which should be separate abstractions.

---

# 342. Mastery Exercise 7 — Pricing

Create:

```text
PricingPolicy
RetailPricing
WholesalePricing
BranchPricing
PromotionPricing
```

Determine whether:

```text
each is a subtype
```

or whether some are composable policy components.

---

# 343. Mastery Exercise 8 — State Machine

Create:

```text
OrderState
```

and compare:

```text
inheritance state objects
vs
discriminated union
```

Defend one design.

---

# 344. Mastery Exercise 9 — Contract Testing

Write a reusable contract suite for:

```text
InMemoryRepository
SqlRepository
CachedRepository
RemoteRepository
```

Ensure all implementations preserve observable semantics.

---

# 345. Mastery Exercise 10 — Inheritance Removal

Take a problematic hierarchy.

Replace it with:

```text
composition
interfaces
adapters
capability types
```

without changing intended consumer behavior.

---

# 346. Principal Exercise — Payment Gateway

A new provider has:

```text
minimum charge amount
provider-specific timeout
different refund semantics
```

Decide whether it can implement the existing contract.

If not:

```text
change the abstraction
add capability interface
adapt semantics
or reject the provider.
```

Do not weaken the contract merely to force substitution.

---

# 347. Principal Exercise — Inventory

One provider uses:

```text
eventual consistency
```

while current contract requires:

```text
read-your-writes
```

Should it implement the existing repository?

Answer:

```text
not without changing or narrowing the contract.
```

---

# 348. Principal Exercise — Tenant Cache

A global cache implementation improves latency but uses:

```text
itemId
```

without tenant ID.

Determine:

```text
performance benefit
security risk
contract violation
correct cache key
```

Security takes precedence.

---

# 349. Principal Exercise — Read-Only Repository

A consumer only reads.

Instead of:

```ts
Repository
```

depend on:

```ts
Reader
```

This can turn an impossible subtype into a valid implementation.

---

# 350. Principal Exercise — Payment Result

Provider returns:

```text
APPROVED
DECLINED
PENDING
```

Base abstraction only models:

```text
success/failure
```

Should `PENDING` be:

```text
error
success
new union member
```

Decide from the domain contract, not provider shape.

---

# 351. Principal Decision Framework

Evaluate every subtype using:

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

Then ask:

```text
What base assumptions does the consumer make?
Which subtype behaviors differ?
Are those differences allowed by the contract?
```

---

# 352. LSP Scorecard

Review:

```text
Input compatibility
Output compatibility
Invariant preservation
Error compatibility
Mutation compatibility
Ordering compatibility
Timing compatibility
Resource compatibility
Security compatibility
Concurrency compatibility
Operational compatibility
```

A scorecard is evidence, not a formal proof.

---

# 353. LSP Review Questions

```text
Can every valid base call be made?
Does every successful subtype call satisfy base postconditions?
Are errors compatible?
Are side effects compatible?
Are lifecycle assumptions compatible?
Are invariants preserved?
Is security preserved?
Is retry semantics preserved?
```

---

# 354. LSP Mastery Gate

```text
Understand
  ↓
Explain
  ↓
Predict
  ↓
Implement
  ↓
Debug
  ↓
Apply
  ↓
Compare
  ↓
Defend
```

Reading alone is not mastery.

You must be able to show why a subtype is safe, not merely point to `extends` or `implements`.

---

# 355. Common Misconception — “Is-A Means Subtype”

False.

Domain identity does not guarantee behavioral substitutability.

---

# 356. Common Misconception — “Compiles Means LSP”

False.

Compilation mainly checks type/structural constraints.

Behavior remains a runtime and contract question.

---

# 357. Common Misconception — “A Subtype Can Reject Anything It Wants”

False.

Rejecting inputs previously valid under the base contract can violate substitutability.

---

# 358. Common Misconception — “Subtypes Must Be Identical”

False.

A subtype can optimize, add stronger guarantees, and implement different algorithms.

It must preserve the base's required semantics.

---

# 359. Common Misconception — “Exceptions Don't Count”

False.

Failure behavior is observable.

---

# 360. Common Misconception — “Only Return Values Matter”

False.

Ordering, side effects, timing, security, and resource behavior can matter.

---

# 361. Common Misconception — “LSP Applies Only to Classes”

False.

It applies to:

```text
interfaces
functions
strategies
services
repositories
adapters
protocol implementations
```

---

# 362. Common Misconception — “Inheritance Is Required”

False.

Structural subtyping and interface implementation can also create substitution relationships.

---

# 363. Common Misconception — “Composition Automatically Solves LSP”

False.

Collaborators still need compatible contracts.

---

# 364. Common Misconception — “More Specific Means Better”

Not always.

A narrower input domain can make a subtype less substitutable.

---

# 365. Common Mistake — Overriding to Disable

```ts
override save() {
  throw new Error();
}
```

This is usually a strong LSP smell.

---

# 366. Common Mistake — Returning Placeholder Data

Changing:

```text
not found
```

into:

```text
fake record
```

can corrupt downstream semantics.

---

# 367. Common Mistake — Tightening Validation

Subtype:

```text
minimum = 100
```

Base:

```text
minimum = 1
```

The subtype silently narrows the valid input space.

---

# 368. Common Mistake — Weakening Errors

Subtype swallows errors and returns success.

This can violate failure contracts.

---

# 369. Common Mistake — Ignoring Side Effects

Read-like methods that mutate state create hidden incompatibility.

---

# 370. Common Mistake — Ignoring Security

Cross-tenant access is not merely an implementation issue.

It is a broken behavioral contract.

---

# 371. Common Mistake — Ignoring Timing

Synchronous-looking or deadline-sensitive APIs can be broken by slow implementations.

---

# 372. Common Mistake — Ignoring Concurrency

A stateful singleton subtype can violate assumptions under concurrent requests.

---

# 373. Common Mistake — Inheritance for Reuse

Code reuse is not evidence of substitutability.

---

# 374. Common Mistake — Giant Base Class

A large base class creates many obligations for every subtype.

Prefer focused abstractions.

---

# 375. Common Mistake — Interface Too Broad

Impossible implementations are often a sign of an abstraction failure.

Apply ISP.

---

# 376. Common Mistake — Testing Only the Base

A subtype needs contract evidence.

---

# 377. Common Mistake — Testing Only Happy Paths

LSP failures often appear in:

```text
invalid input
failure
retry
concurrency
security
```

---

# 378. Common Mistake — Mock Everything

Mocks can hide semantic differences.

Use contract-compliant fakes.

---

# 379. Common Mistake — Ignoring Operational Guarantees

Timeouts, cancellation, resource ownership, and lifecycle can be part of substitution.

---

# 380. Common Mistake — Forcing Every Provider into One Interface

Different capabilities may require different interfaces.

---

# 381. Common Mistake — Weakening Base Contract to Fit One Subtype

Do not reduce the abstraction's guarantees merely to accommodate an implementation.

Reconsider the abstraction.

---

# 382. Common Mistake — Over-Generalizing

A generic base with every possible feature creates fragile subtype obligations.

---

# 383. LSP and Refactoring Safety

A safe hierarchy refactor should preserve:

```text
consumer-visible contract
```

while changing:

```text
implementation relationship
```

---

# 384. LSP Refactoring Checklist

```text
□ capture current contract
□ identify all consumers
□ identify subtype-specific constraints
□ classify violations
□ split capabilities
□ introduce adapters if needed
□ migrate consumers
□ run contract tests
□ verify security
□ verify concurrency
□ verify transactions
```

---

# 385. LSP and Branch by Abstraction

Introduce:

```text
stable capability
```

between consumers and legacy hierarchy.

Migrate implementations behind it.

---

# 386. LSP and Strangler Refactoring

Replace problematic subtype behavior incrementally:

```text
legacy subtype
→ adapter
→ stable contract
→ new implementation
```

---

# 387. LSP and Compatibility Layer

Compatibility adapters can preserve the old contract while the internal model changes.

---

# 388. LSP and Deprecation

Deprecate unsafe hierarchy relationships and migrate consumers to capability-based interfaces.

---

# 389. LSP and API Versioning

If semantics genuinely change:

```text
new contract
```

may be better than pretending the implementation is still a valid subtype.

---

# 390. LSP and Contract Evolution

When the base contract changes, re-evaluate every subtype.

Adding a new required method or strengthening a guarantee can invalidate existing implementations.

---

# 391. LSP and Semantic Versioning

For public libraries, changes that make previously valid subtypes invalid are compatibility concerns.

Document them as breaking changes where appropriate.

---

# 392. LSP and Plugin Ecosystems

Public plugin contracts require:

```text
behavioral compatibility
versioning
contract tests
security
lifecycle
```

---

# 393. LSP and Plugin Security

A plugin satisfying the interface but escaping its sandbox is not an acceptable implementation.

Security is part of operational substitutability.

---

# 394. LSP and Third-Party Implementations

If consumers can implement your interface, publish:

```text
behavioral documentation
examples
contract tests
failure semantics
```

---

# 395. LSP and Contract Suites as API Assets

A shared contract test suite can become part of the SDK.

This turns tribal knowledge into executable behavior.

---

# 396. LSP and Reference Documentation

Document:

```text
valid input range
result semantics
error semantics
side effects
ordering
timing
security
retry
```

Do not document only method signatures.

---

# 397. LSP and Observability Evidence

Monitor all implementations:

```text
error rates
latency
retry rates
contract violations
security failures
```

---

# 398. LSP and Canary Testing

Introduce a new implementation for a small traffic cohort.

Compare semantic outcomes before full rollout.

---

# 399. LSP and Shadow Testing

Run a new implementation alongside the old implementation without exposing its result.

Compare:

```text
observable semantics
```

where safe.

---

# 400. LSP and Production Divergence

A subtype can pass unit tests but fail in production due to:

```text
concurrency
timeouts
tenant mix
cache staleness
provider behavior
```

Production-like contract testing matters.

---

# 401. LSP and Chaos Testing

For critical implementations inject:

```text
timeouts
duplicates
partial failures
cancellations
dependency outages
```

and verify contract preservation.

---

# 402. LSP and Fuzzing

Generate edge inputs to discover strengthened hidden preconditions.

---

# 403. LSP and Security Fuzzing

Try:

```text
cross-tenant IDs
authorization changes
malformed tokens
unexpected roles
```

across implementations.

---

# 404. LSP and Data Races

Generate concurrent requests to implementations with shared mutable state.

---

# 405. LSP and Invariant Monitoring

Runtime invariant monitors can detect implementations drifting from base behavior.

---

# 406. LSP and Error Telemetry

Track errors by implementation:

```text
implementationId
errorKind
retryable
```

This identifies contract divergence.

---

# 407. LSP and Performance Telemetry

Track implementation-specific latency while preserving the shared semantic contract.

---

# 408. LSP and Memory Telemetry

Long-lived implementations should be monitored for retention and leaks.

---

# 409. LSP and Resource Exhaustion

Every implementation should respect resource contracts:

```text
timeouts
concurrency limits
payload limits
memory
```

where specified.

---

# 410. LSP and Backpressure

Streaming implementations must preserve backpressure semantics.

---

# 411. LSP and Cancellation

Cancellation must be tested because async cleanup is implementation-dependent.

---

# 412. LSP and Lifecycle

Substitutability includes:

```text
construct
initialize
start
operate
stop
dispose
```

when the abstraction defines those phases.

---

# 413. LSP and Object Lifetime

Subtype lifetime must not exceed or undercut assumptions of the base contract in unsafe ways.

---

# 414. LSP and Dependency Lifetime

A singleton subtype with request-scoped dependencies can violate runtime semantics.

---

# 415. LSP and Framework Containers

Dependency injection containers may resolve different implementations behind one token.

That makes LSP critical at runtime.

---

# 416. LSP and NestJS

A NestJS provider implementing an interface can be structurally replaceable, but behavior must still preserve the application's contract.

Framework injection does not prove LSP.

---

# 417. LSP and Express/Fastify

Middleware or handlers can be substituted only when:

```text
request
response
next/error
timing
side effects
```

contracts remain compatible.

---

# 418. LSP and Event Handlers

Event consumers implementing a handler interface must preserve:

```text
acknowledgement
failure
retry
idempotency
```

semantics.

---

# 419. LSP and Queues

A consumer implementation changing:

```text
ack before processing
```

to:

```text
ack after processing
```

can alter delivery guarantees.

---

# 420. LSP and Jobs

A job handler must preserve:

```text
idempotency
retryability
failure classification
timeout
```

expected by the scheduler.

---

# 421. LSP and Schedulers

A scheduler implementation that changes:

```text
at-most-once
```

to:

```text
at-least-once
```

must not be presented as the same contract without acknowledging changed semantics.

---

# 422. LSP and Time-Based Contracts

Cron-like jobs can differ in clock handling.

Timezone semantics matter if they are part of the abstraction.

---

# 423. LSP and Randomness

A deterministic implementation can often substitute for a nondeterministic one if the base contract permits deterministic outcomes.

The reverse may not be safe when determinism is required.

---

# 424. LSP and Environment

Implementations that depend on local environment state can violate portability assumptions.

---

# 425. LSP and External HTTP Clients

A subtype changing retry policy or error mapping can alter observable behavior.

---

# 426. LSP and SDK Wrappers

SDK-specific semantics should be normalized behind the stable domain contract.

---

# 427. LSP and Financial Systems

Financial substitutions require special care:

```text
precision
rounding
currency
idempotency
authorization
audit
durability
```

---

# 428. LSP and Jewellery ERP Financials

A pricing or billing implementation must preserve:

```text
money unit
tax semantics
rounding
discount rules
audit
tenant/branch scope
```

---

# 429. LSP and Inventory Financial Coupling

Inventory and billing may interact, but neither should weaken the other's contract when substituted.

---

# 430. LSP and Multi-Tenant Pricing

A tenant-specific implementation is not automatically a subtype of a global pricing policy.

It must preserve the expected scope.

---

# 431. LSP and Branch Overrides

A branch pricing adapter can safely implement a broader policy only if its behavior remains valid for all inputs the broader contract allows.

Otherwise use a narrower interface.

---

# 432. LSP and Domain Boundaries

Often the right answer is:

```text
do not subtype.
```

Model a separate concept and compose it.

---

# 433. LSP and Principal Judgment

The most important question is:

> **What assumptions does the consumer have permission to make from the base contract?**

Then verify that every implementation honors those assumptions.

---

# 434. Principal Judgment — Do Not Force Substitution

If a subtype cannot satisfy the contract:

```text
change the abstraction
or
do not treat it as a subtype.
```

Do not weaken a useful abstraction just to preserve a hierarchy.

---

# 435. Principal Judgment — Model Capabilities

When capabilities differ:

```text
Reader
Writer
Refundable
Chargeable
Streamable
```

can be better than one giant base type.

---

# 436. Principal Judgment — Contracts Before Hierarchies

Do not start with:

```text
Class A extends Class B
```

Start with:

```text
What behavior must consumers rely on?
```

Then choose:

```text
interface
composition
inheritance
adapter
union
function
```

---

# 437. Principal Judgment — Stronger Is Not Always Safer

A subtype with stronger requirements can be less substitutable.

A stronger guarantee is often safe.

A stronger requirement often is not.

---

# 438. Principal Judgment — Preserve Meaning

The exact internal implementation may change.

The meaning of the operation should remain stable.

---

# 439. Principal Judgment — Semantic Minimum

Define the smallest contract common to valid implementations.

Then expose specialized capabilities separately.

---

# 440. Principal Judgment — Operational Reality

A subtype is not truly substitutable if production conditions make its behavior violate critical system guarantees.

Review:

```text
timeouts
concurrency
security
resource limits
observability
```

when relevant.

---

# 441. Principal Scenario — Payment Provider

Base:

```text
charge(amount, currency, key)
```

New provider:

```text
minimum amount 500
```

Existing base consumers may submit 100.

Decision:

```text
not a general subtype
unless the contract is narrowed or the provider is adapted.
```

---

# 442. Principal Scenario — Cached Repository

Base:

```text
read-your-writes
```

Cache:

```text
stale for 30 seconds
```

Decision:

```text
not substitutable under the existing contract.
```

---

# 443. Principal Scenario — Read-Only

Base:

```text
Reader
```

Implementation:

```text
read-only database
```

Decision:

```text
valid subtype.
```

The issue disappears when the contract is correctly scoped.

---

# 444. Principal Scenario — Bird

Base:

```text
Bird
```

if it promises only bird behavior:

```text
eat()
```

Penguin can substitute.

If it promises:

```text
fly()
```

the abstraction is wrong.

---

# 445. Principal Scenario — Inventory

Base:

```text
reserve() never creates negative availability.
```

Subtype:

```text
temporary negative stock allowed.
```

Decision:

```text
not substitutable unless negative stock is explicitly permitted by the shared contract.
```

---

# 446. Principal Scenario — Event Publisher

Base:

```text
events carry tenantId and stable eventId.
```

Subtype omits both.

Decision:

```text
invalid implementation.
```

---

# 447. Principal Scenario — Payment Result

Base:

```text
success means final settlement.
```

Subtype returns success for:

```text
PENDING
```

Decision:

```text
semantic contract violation.
```

---

# 448. Principal Scenario — Analytics

Base:

```text
eventual consistency allowed.
```

Subtype with stronger consistency:

```text
authoritative
```

Decision:

```text
potentially valid, because stronger behavior preserves the weaker guarantee.
```

Still review cost, ordering, and error semantics.

---

# 449. Principal Scenario — Authorization

Base:

```text
caller cannot access cross-tenant data.
```

Subtype:

```text
global admin access always enabled.
```

Decision:

```text
invalid unless the contract explicitly includes that authority model.
```

---

# 450. Principal Scenario — Performance

Base:

```text
no explicit latency guarantee.
```

Subtype is:

```text
10x slower
```

Decision:

```text
not automatically an LSP violation.
```

But it may still be operationally unacceptable.

---

# 451. Principal Scenario — Error Handling

Base:

```text
conflict errors are retry/reload signals.
```

Subtype maps conflict to generic failure.

Decision:

```text
contract weakened.
```

---

# 452. Principal Scenario — Cancellation

Base:

```text
AbortSignal cancels work and releases resources.
```

Subtype ignores signal.

Decision:

```text
contract violation.
```

---

# 453. Principal Scenario — Tenant Cache

Base:

```text
get(tenantId, productId)
```

subtype key:

```text
productId
```

Decision:

```text
security-critical LSP violation.
```

---

# 454. Retrieval Drill

Without notes, explain:

```text
LSP
behavioral subtype
precondition rule
postcondition rule
invariant preservation
covariance
contravariance
invariance
capability interface
contract test
```

---

# 455. Retrieval Drill — Spot the Violation

For each scenario, answer:

```text
valid subtype
invalid subtype
underspecified base contract
```

Scenarios:

```text
stronger return guarantee
narrower accepted inputs
different error code
different ordering
different authorization
different timeout
different cache freshness
```

---

# 456. Implementation Drill

Implement:

```text
PaymentGateway
```

with three providers.

Write one shared contract suite.

Break each provider intentionally once.

Confirm the suite catches the violation.

---

# 457. Debugging Drill

Take a failing production scenario:

```text
same interface, different behavior
```

Find the hidden contract dimension:

```text
error
ordering
freshness
tenant
idempotency
transaction
```

---

# 458. Comparison Drill

Compare:

```text
inheritance
composition
adapter
capability interface
discriminated union
function parameter
```

Choose which best expresses each domain scenario.

---

# 459. Defense Drill

Explain:

> “We rejected this provider as an implementation of the interface.”

Your answer should include:

```text
which contract it violates
why that matters
what adaptation could fix it
why weakening the base contract is or is not acceptable
```

---

# 460. Final Mental Model

```text
Base abstraction
    ↓
consumer assumptions
    ↓
behavioral contract
    ↓
subtype
    ↓
must preserve
    ├─ valid inputs
    ├─ promised outputs
    ├─ invariants
    ├─ failure semantics
    ├─ security
    ├─ state transitions
    └─ relevant operational guarantees
```

And:

```text
type compatibility
    ≠
behavioral substitutability
```

---

# 461. Final Master Rule

> **A subtype is valid only when every important assumption a consumer may make from the base contract remains true after substitution.**

---

## 462. Completion Criteria

Do not mark this chapter mastered until you can:

- Define LSP in behavioral terms.
- Explain why inheritance syntax is not enough.
- Distinguish type compatibility from behavioral compatibility.
- Explain precondition and postcondition constraints.
- Explain invariant preservation.
- Reason about exceptions, nullability, ordering, side effects, timing, and resource semantics.
- Apply covariance and contravariance intuitively.
- Identify invalid inheritance.
- Refactor broad interfaces into capability contracts.
- Replace unsafe inheritance with composition or adapters.
- Build shared contract tests.
- Apply LSP to repositories, caches, payment gateways, event publishers, workflows, and plugins.
- Detect tenant isolation and authorization violations.
- Analyze concurrency, idempotency, transaction, and consistency semantics.
- Apply LSP to jewellery ERP examples.
- Defend subtype decisions at principal level.

Status:

```text
[ ] Not Started
[~] In Progress
[?] Needs Revision
[+] Completed
[*] Mastered
```

---

# Chapter 21 — Key Takeaways

```text
LSP is about behavioral substitutability.

A subtype must preserve the meaningful contract of the base abstraction.

TypeScript structural compatibility does not prove LSP.

Subtype preconditions should not become unexpectedly stronger.

Subtype postconditions should not become unexpectedly weaker.

Invariants must remain valid.

Exceptions are part of behavioral contracts.

Ordering can be part of a contract.

Timing can be part of a contract.

Security is part of substitutability when authorization is promised.

Concurrency and transaction semantics can be contractual.

Idempotency can be contractual.

Cache freshness can be contractual.

Composition often avoids invalid inheritance.

Capability interfaces reduce impossible subtype obligations.

A narrower abstraction is often better than a fake universal subtype.

Do not weaken a useful base contract merely to fit one implementation.

The correct abstraction is the one whose implementations can honestly preserve consumer assumptions.
```

---

# Chapter 21 — Concept Connections

Backward connections:

```text
Chapter 18 → Stable Contracts and Invariants
Chapter 19 → Single Responsibility Principle
Chapter 20 → Open/Closed Principle
```

Directly connected concepts:

```text
Inheritance
Polymorphism
Composition
Delegation
Protected Variations
Interface Segregation
Dependency Inversion
TypeScript variance
Design by Contract
Contract Testing
State Machines
Capability Modeling
```

Forward connections:

```text
Chapter 22 → Interface Segregation Principle
Chapter 23 → Dependency Inversion Principle
Chapter 24 → SOLID in JavaScript/TypeScript
Chapter 25 → DRY, KISS, YAGNI
Chapter 26 → Law of Demeter
Chapter 27 → Tell, Don't Ask
Chapter 28 → Command-Query Separation
Chapter 29 → Stable Dependencies
Chapter 30 → Stable Abstractions
```

---

# Chapter 21 — Dependency Graph

```text
Contract
    ↓
Consumer Assumptions
    ↓
Base Abstraction
    ↓
Subtype
    ↓
Behavioral Compatibility
    ↓
Invariant Preservation
    ↓
Safe Polymorphism
    ↓
Extensible System
```

---

# Chapter 21 — Revision / Retrieval Record

## Session Record

```text
Status:
[~] In Progress

Can define:
- LSP
- behavioral subtyping
- precondition
- postcondition
- invariant preservation

Can reason about:
- covariance
- contravariance
- mutable containers
- capability interfaces
- inheritance smells

Needs implementation:
- contract-tested subtypes
- capability refactoring
- adapter-based substitution
- tenant-safe repositories
```

## Retrieval Template

```text
Date:
Concept:
Can define:
Can explain:
Can predict:
Can implement:
Can debug:
Can compare:
Can defend:
Evidence:
Next review:
```

---

# Chapter 21 — Completion Snapshot

## Core Theory

```text
[+] LSP
[+] behavioral subtyping
[+] type vs behavioral compatibility
[+] precondition preservation
[+] postcondition preservation
[+] invariant preservation
[+] capability modeling
[+] covariance
[+] contravariance
[+] invariance
```

## Design

```text
[+] composition
[+] delegation
[+] adapters
[+] capability interfaces
[+] refactoring inheritance
[+] contract-centered design
```

## Production

```text
[+] persistence
[+] caching
[+] concurrency
[+] transactionality
[+] idempotency
[+] security
[+] tenant isolation
[+] observability
[+] lifecycle
```

## Principal Judgment

```text
[+] reject false subtyping
[+] narrow abstractions
[+] preserve consumer assumptions
[+] avoid weakening contracts
[+] distinguish semantic vs implementation differences
[+] test behavior, not only structure
```

---

# Chapter 21 — Canonical Source Discipline

When verifying this chapter:

```text
TypeScript type/variance behavior
  → current TypeScript documentation and compiler behavior

JavaScript runtime behavior
  → ECMAScript semantics

Design-principle framing
  → canonical design-principle literature and established software-design sources

HTTP/database/runtime semantics
  → authoritative protocol or runtime documentation

Domain behavior
  → actual application/domain requirements
```

Do not claim that TypeScript assignability proves behavioral substitutability.

---

# Chapter 21 — Compact Mental Checklist

```text
□ What does the base contract promise?
□ What inputs are valid?
□ What outputs are guaranteed?
□ Which errors are expected?
□ Which invariants exist?
□ What side effects are allowed?
□ What ordering is promised?
□ What timing is promised?
□ What security is promised?
□ What consistency is promised?
□ What retry/idempotency semantics exist?
□ What lifecycle is promised?
□ Can every implementation preserve these assumptions?
□ Would composition be safer?
□ Should the abstraction be narrower?
```

---

# Chapter 21 — Standard Template Crosswalk

| Standard Area | Covered By |
|---|---|
| Learning Objectives | Section 1 |
| Prerequisites | Section 2 |
| What Is It? | Sections 3–7 |
| Why Does It Exist? | Sections 4–5 |
| Mental Model | Sections 15, 460 |
| Core Rules | Sections 8–50 |
| Syntax / TypeScript | Sections 61–126 |
| Basic Examples | Sections 18–23, 301–312 |
| Runtime / Operational Semantics | Sections 200–270 |
| Advanced Behavior | Sections 51–453 |
| Edge Cases | Sections 71–107, 115–178, 231–270 |
| Common Misconceptions | Sections 355–364 |
| Common Mistakes | Sections 365–382 |
| Comparisons | Sections 54–60, 121–130, 154–160 |
| Performance | Sections 36, 82, 202, 264 |
| Memory | Sections 37, 243, 267 |
| Security | Sections 38–40, 89–94, 133–136, 200–230 |
| Production Usage | Sections 200–450 |
| Implementation From Scratch | Sections 301–312 |
| Debugging | Sections 313–322 |
| Code Review | Sections 323–330 |
| Interview Questions | Embedded throughout the chapter's reasoning sections; retrieval should be practiced from the core questions |
| Predict-the-Output | Sections 331–335 |
| Mastery | Sections 336–350 |
| Completion | Section 462 |
| Key Takeaways | Later section |
| Concept Connections | Later section |
| Revision Record | Later section |

---

# Chapter 21 — Final Retrieval Card

```text
LSP
  behavioral substitutability

TYPE COMPATIBILITY
  structural/compiler-level fit

BEHAVIORAL COMPATIBILITY
  consumer assumptions remain valid

PRECONDITION
  what inputs are allowed

POSTCONDITION
  what successful execution guarantees

INVARIANT
  what must remain true

COVARIANCE
  narrower valid outputs can often substitute for broader outputs

CONTRAVARIANCE
  broader accepted inputs can often substitute for narrower input domains

INVARIANCE
  no safe substitution in either direction for a mutable mixed-use type

CAPABILITY INTERFACE
  exposes only behavior an implementation can honestly guarantee

CONTRACT TEST
  executable evidence that implementations preserve semantic expectations
```

---

# Chapter 21 — One-Line Mastery Test

> **Can you explain exactly why a consumer is safe or unsafe when a different implementation replaces the expected abstraction, and can you redesign the abstraction when the subtype cannot honestly preserve the contract?**
