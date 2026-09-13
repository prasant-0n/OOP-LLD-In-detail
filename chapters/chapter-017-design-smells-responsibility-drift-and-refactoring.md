# Chapter 17 — Design Smells, Responsibility Drift & Refactoring

> **Part:** C — Responsibility-Driven Design  
> **Prerequisite:** Chapters 12–16  
> **Next:** Chapter 18 — Designing Stable Contracts and Invariants  
> **Status:** `[+] Completed`

---

# Chapter Position

Chapter 12 introduced cohesion and coupling as central design forces.

Chapter 13 established responsibility ownership.

Chapter 14 formalized responsibility assignment through GRASP.

Chapter 15 showed how cohesion can guide decomposition.

Chapter 16 treated dependencies as deliberate design decisions.

This chapter focuses on the signals that tell you those decisions may be drifting.

> **A design smell is not automatically a defect. It is evidence that a design may be carrying avoidable cost.**

The central workflow is:

```text
Smell
  ↓
Underlying force
  ↓
Responsibility problem
  ↓
Change scenario
  ↓
Candidate refactoring
  ↓
Behavior preservation
  ↓
Dependency review
  ↓
Production verification
```

---

# 1. Learning Objectives

By the end of this chapter, you should be able to:

- define design smells and distinguish them from defects
- recognize responsibility drift
- diagnose cohesion, coupling, variation, representation, lifecycle, and boundary smells
- identify long methods, large classes, primitive obsession, data clumps, feature envy, inappropriate intimacy, message chains, middle men, speculative generality, service blobs, controller blobs, god objects, shotgun surgery, divergent change, dependency cycles, global state, leaky abstractions, and inappropriate inheritance
- select a refactoring based on the underlying force rather than the symptom
- use characterization tests and incremental refactoring safely
- apply Extract Method, Extract Class, Move Method, Move Field, value-object extraction, policy extraction, polymorphism, ports/adapters, and other common transformations
- reason about refactoring in dynamic JavaScript and statically typed TypeScript
- preserve contracts, invariants, runtime behavior, security, observability, and reliability while refactoring
- apply the approach to legacy systems, Node.js services, modular monoliths, distributed systems, and jewellery ERP workflows
- distinguish necessary technical debt from accidental design debt
- decide when a smell should intentionally remain
- defend refactoring priorities in LLD interviews and principal-level reviews

---

# 2. Prerequisites

Recommended:

```text
Chapter 12 — Cohesion & Coupling
Chapter 13 — Responsibility-Driven Object Design
Chapter 14 — GRASP
Chapter 15 — Cohesion-First Object Decomposition
Chapter 16 — Coupling-First Dependency Design
```

Also understand:

```text
objects
classes
modules
functions
closures
composition
polymorphism
encapsulation
TypeScript interfaces
dependency injection
```

---

# 3. Design Smells — What They Are

A design smell is a recurring structural symptom that suggests the design may become difficult to:

```text
understand
change
test
debug
evolve
operate securely
```

A smell is not automatically a defect.

Use:

```text
smell
  ↓
investigate
  ↓
identify underlying force
  ↓
refactor when justified
```

Do not refactor merely because a rule says something looks wrong.

---

# 4. Smell vs Defect

A defect means behavior is incorrect.

A smell suggests structural risk.

Example:

```text
Incorrect order total
    → defect

2,000-line OrderService
    → possible smell
```

A smell becomes important when it creates measurable or expected future cost.

---

# 5. Why Smells Matter in LLD

Low-level design is not only the ability to create objects.

It is also the ability to recognize when responsibility, cohesion, or dependency boundaries are degrading.

Smells reveal:

```text
unclear responsibility
weak cohesion
excessive coupling
variation leakage
representation leakage
duplicated knowledge
boundary erosion
```

---

# 6. Responsibility Drift

Responsibility drift occurs when a component gradually accumulates behavior unrelated to its original purpose.

Typical progression:

```text
Order
  ↓
add helper
  ↓
add payment call
  ↓
add email
  ↓
add reporting
  ↓
add cache
  ↓
God Object
```

The important observation is that drift is usually incremental.

---

# 7. Refactoring Mindset

Refactoring means improving internal structure while intentionally preserving externally observable behavior.

The safe loop is:

```text
characterize behavior
  ↓
small structural change
  ↓
test
  ↓
inspect dependency impact
  ↓
repeat
```

---

# 8. Refactoring Is Not Rewriting

A rewrite replaces large portions of a system.

A refactor changes structure incrementally.

Prefer refactoring when:

```text
behavior is mostly correct
change scope can be isolated
tests can characterize behavior
incremental migration is possible
```

---

# 9. Code Smells vs Design Smells

Local code smells:

```text
long method
magic number
duplicate branch
```

Structural design smells:

```text
god object
service blob
dependency cycle
shotgun surgery
inappropriate intimacy
```

The same underlying problem can appear at multiple levels.

---

# 10. Smell Severity

Prioritize smells using:

```text
customer impact
security impact
failure impact
change frequency
engineering cost
operational cost
```

One ugly helper used once may not matter.

A highly coupled component changed every week can be a major liability.

---

# 11. Smell Evidence

Before refactoring, gather evidence where available:

```text
change history
dependency graph
tests
production incidents
performance traces
security findings
ownership boundaries
```

Avoid architecture changes based only on aesthetics.

---

# 12. Long Method

A long method may contain multiple responsibility clusters.

Example:

```ts
async finalize() {
  validate();
  calculate();
  reserve();
  charge();
  save();
  notify();
}
```

Do not automatically split every call.

First classify each step as:

```text
domain decision
coordination
infrastructure
representation
```

---

# 13. Long Parameter List

Long parameter lists may indicate:

```text
missing concept
missing value object
missing command
missing cohesive context
```

But a parameter object can create stamp coupling.

Extract only when the parameters represent a meaningful stable concept.

---

# 14. Large Class

A large class is a smell when it contains multiple unrelated responsibility centers.

The key question is:

> Does this class still have one strong conceptual center?

A complex but cohesive state machine can legitimately be large.

---

# 15. Primitive Obsession

Primitive obsession appears when domain concepts are represented as raw primitives everywhere.

Examples:

```text
Money
Quantity
Percentage
EmailAddress
OrderStatus
TenantId
```

A value object can centralize invariants when the concept has enough behavior or validation to justify it.

---

# 16. Data Clumps

Data clumps are recurring groups of values:

```text
country
currency
taxRate
```

If the group has stable domain meaning, consider:

```text
TaxContext
```

Do not extract merely because fields happen to repeat.

---

# 17. Feature Envy

Feature envy occurs when behavior uses another object's data and structure more heavily than its own.

Example:

```ts
class DiscountService {
  calculate(order: Order) {
    return order.items.reduce((sum, item) => {
      return sum + item.price * item.quantity;
    }, 0);
  }
}
```

Ask:

```text
Who owns the decision?
Who has the necessary knowledge?
```

Then move behavior or introduce a policy when appropriate.

---

# 18. Inappropriate Intimacy

Two classes know too much about each other's internal details.

Examples:

```text
private representation assumptions
mutual field mutation
bidirectional helper dependence
internal data access
```

Use:

```text
encapsulation
semantic APIs
clear ownership
narrow capabilities
```

---

# 19. Message Chains

Deep navigation:

```ts
sale.customer.account.branch.taxProfile.rate
```

can couple callers to object topology.

A semantic operation may reduce that coupling:

```ts
sale.taxRate()
```

Do not treat every chain as a defect. Collections and fluent APIs can be intentionally chainable.

---

# 20. Middle Man

A middle man simply forwards calls:

```text
A → B → C
```

If B provides no meaningful responsibility, it may be unnecessary.

But B is justified if it provides:

```text
indirection
translation
security
policy
stability
```

---

# 21. Speculative Generality

Speculative generality creates abstractions for hypothetical requirements.

Examples:

```text
ITransportFactoryProvider
IAbstractOrderStrategy
```

before actual variation exists.

Prefer evidence-driven abstraction.

---

# 22. Dead Abstraction

An abstraction may become unnecessary when:

```text
only one implementation remains
no meaningful variation exists
clients gain no boundary value
```

Removing obsolete layers can be a valid refactoring.

---

# 23. Parallel Inheritance Hierarchies

If every new class in one hierarchy requires another class in a second hierarchy, you may have parallel hierarchies.

This often signals multiple variation dimensions that should be composed rather than represented through inheritance multiplication.

---

# 24. Switch Statements

Large switches can indicate behavioral variation.

Compare:

```text
small stable switch
```

with:

```text
large growing switch
many independent variants
repeated edits across callers
```

The latter may justify polymorphism.

---

# 25. Conditional Complexity

Nested conditionals can indicate:

```text
missing policy
missing state machine
missing value object
missing polymorphism
```

Refactor around the actual reason for complexity.

---

# 26. Boolean Parameter Smell

This:

```ts
generateReport(order, true);
```

can hide two meanings.

Prefer semantic operations when appropriate:

```ts
generatePdf(order);
generateCsv(order);
```

or a named options object when the flag is genuinely configuration.

---

# 27. Flag Arguments

Flags can create control coupling:

```ts
process(order, { force: true });
```

A flag is acceptable when it represents a stable option.

It is suspicious when it switches the fundamental responsibility of the function.

---

# 28. Optional Parameter Explosion

Many optional parameters can create hidden modes:

```text
null
undefined
default
override
special-case
```

Model meaningful states explicitly when complexity grows.

---

# 29. Null Checks Everywhere

Repeated null checks may reveal:

```text
missing state model
missing value object
weak boundary validation
implicit absence semantics
```

Do not replace every `null` with an arbitrary default.

---

# 30. Magic Numbers

Magic numbers often hide policy:

```ts
if (discount > 20) {
  ...
}
```

When the number has business meaning, name the rule:

```text
ManagerApprovalThreshold
```

---

# 31. Stringly Typed State

Arbitrary strings:

```ts
status = "approved";
```

can create invalid states.

Use constrained types, value objects, or semantic state transitions when lifecycle rules matter.

---

# 32. Enum Does Not Equal State Machine

An enum or union can constrain possible values.

It does not automatically enforce valid transitions.

Still define:

```text
state rules
transition behavior
invariants
```

where required.

---

# 33. Anemic Domain Model

An anemic model stores state while services contain most domain decisions.

It becomes a smell when:

```text
rules are duplicated
state transitions are weakly protected
business behavior is scattered
```

It can be appropriate for simple CRUD.

---

# 34. Service Blob

A service blob contains unrelated operations:

```text
BusinessService
  tax
  payment
  email
  inventory
  reporting
```

The fix is responsibility decomposition, not simply more service classes.

---

# 35. Controller Blob

A controller blob combines:

```text
transport
business logic
persistence
provider integration
```

Separate:

```text
controller → transport
use case → coordination
domain → business invariants
repository → persistence
adapter → external integration
```

---

# 36. God Object

A god object has:

```text
too much state
too many methods
too many collaborators
many reasons to change
large public surface
```

Do not refactor by method count.

First identify responsibility clusters.

---

# 37. God Module

A module can become a god object at package level:

```text
utils.ts
  pricing
  payment
  database
  auth
  email
```

Organize around coherent capabilities.

---

# 38. Shotgun Surgery

One conceptual change requires many small edits.

Example:

```text
tax rule change
  → controller
  → service
  → invoice
  → report
  → several tests
```

This suggests the rule lacks a coherent owner.

---

# 39. Divergent Change

One module changes for unrelated reasons:

```text
OrderService
  tax changes
  payment changes
  email changes
  reporting changes
```

This is a cohesion warning.

---

# 40. Change Amplification

Measure:

```text
one conceptual change
    ↓
number of components touched
```

High amplification is a valuable prioritization signal.

---

# 41. Dependency Cycle Smell

Example:

```text
A → B
B → C
C → A
```

Cycles make:

```text
initialization
change reasoning
testing
module ownership
```

more difficult.

---

# 42. Shared Mutable Singleton

Shared mutable state creates hidden coupling:

```text
module A
   ↓
shared singleton
   ↑
module B
```

Behavior depends on who changed state and when.

---

# 43. Global State Smell

Examples:

```ts
globalThis.currentTenantId = tenantId;
```

or broad application globals.

Prefer explicit/scoped dependencies when possible.

---

# 44. Service Locator Smell

Hidden dependency:

```ts
container.resolve(PaymentGateway)
```

Explicit dependency:

```ts
constructor(private readonly payment: PaymentGateway) {}
```

Constructor injection exposes coupling to review and tests.

---

# 45. Temporal Coupling Smell

If callers must remember:

```text
initialize()
authenticate()
start()
use()
stop()
```

the lifecycle may be too exposed.

Encapsulate ordering when it is not part of the caller's meaningful responsibility.

---

# 46. Content Coupling Smell

Depending on internal state:

```ts
payment._state.status = "CAPTURED";
```

turns implementation details into contracts.

Prefer:

```ts
payment.capture();
```

---

# 47. Broad Interface Smell

A broad interface forces consumers to depend on unrelated operations.

Prefer focused capability interfaces when consumers genuinely have different needs.

---

# 48. Interface Pollution

Adding every future operation to one interface creates unnecessary dependency.

Keep contracts centered on actual responsibilities.

---

# 49. Leaky Abstraction

An abstraction leaks when clients must understand lower-level details.

Suspicious:

```ts
paymentGateway.charge({
  providerIntentMode: "special"
});
```

The interface may be exposing provider semantics rather than internal capability.

---

# 50. Leaky DTO

A DTO that mirrors database or provider representation throughout the system can create broad representation coupling.

Map at boundaries when representations evolve independently.

---

# 51. Inappropriate Inheritance

Inheritance is a smell when the subtype is not a true behavioral subtype or inherits irrelevant responsibility.

Ask:

```text
Is-a?
Substitutable?
Shared contract?
```

---

# 52. Fragile Base Class

A base class change unexpectedly breaks subclasses.

Signals:

```text
protected implementation details
implicit ordering
super calls
constructor side effects
overriding assumptions
```

Composition can often reduce this fragility.

---

# 53. Refused Bequest

A subclass inherits members it does not meaningfully need.

This can indicate a bad inheritance relationship.

---

# 54. Inheritance for Code Reuse

Code reuse alone is not sufficient justification for inheritance.

Use composition when the relationship is not truly substitutable.

---

# 55. Premature Abstraction

An abstraction introduced before the responsibility is understood may encode the wrong boundary.

Discover:

```text
responsibility
change
variation
```

before abstracting.

---

# 56. Premature Optimization

Architecture can become distorted by unmeasured performance assumptions:

```text
over-caching
premature batching
unnecessary async boundaries
complex memoization
```

Measure actual bottlenecks.

---

# 57. Over-Engineering

Solution complexity exceeds problem complexity.

Example:

```text
factory + builder + abstract factory
```

for:

```text
new User(name)
```

Use the simplest design that handles actual requirements.

---

# 58. Under-Engineering

Ignoring known complexity is also dangerous:

```text
direct provider calls everywhere
raw state mutation
shared database writes
implicit tenant state
```

The goal is appropriate complexity, not minimum complexity.

---

# 59. Abstraction Drift

A focused abstraction can accumulate unrelated methods over time.

Review long-lived interfaces and services periodically.

---

# 60. Boundary Erosion

A healthy architecture can erode through convenience shortcuts:

```text
domain → database
domain → provider SDK
controller → ORM internals
```

Architecture tests and review can stop gradual erosion.

---

# 61. Representation Leakage

When internal representation is exposed, it becomes a de facto API.

Semantic APIs preserve implementation freedom.

---

# 62. Knowledge Leakage

A component should not need to know details outside its responsibility.

Common leakage paths:

```text
deep property chains
provider error classes
ORM models
framework request objects
internal fields
```

---

# 63. Authority Leakage

Broad dependencies grant broad power.

Example:

```text
Database
```

instead of:

```text
OrderRepository
```

when only order persistence is needed.

---

# 64. Error Handling Blob

One component catching every error can become a responsibility blob.

Map errors near the boundary and let the application layer coordinate outcomes.

---

# 65. Retry Duplication

Repeated retry loops indicate missing cohesive resilience responsibility.

Consider:

```text
RetryPolicy
```

at the technical boundary.

---

# 66. Logging Everywhere

Scattered provider-specific logging can couple domain logic to infrastructure.

Prefer focused observability capabilities where needed.

---

# 67. Transaction Script

A transaction script places business decisions into a procedural workflow.

This can be valid for simple domains.

It becomes a smell when complex invariants are duplicated across scripts.

---

# 68. Transaction Script vs Anemic Model

These often appear together:

```text
data objects
    +
large procedural services
```

When rules grow, move cohesive invariant-heavy behavior toward the appropriate domain owner.

---

# 69. Refactoring Transaction Scripts

Do not convert every script into a class.

Move behavior when:

```text
rules are cohesive
state is owned
invariants are meaningful
multiple workflows reuse the decision
```

---

# 70. Smell Classification by Force

Classify a smell by underlying force:

```text
cohesion problem
coupling problem
ownership problem
variation problem
representation problem
lifecycle problem
coordination problem
boundary problem
```

This prevents symptom-driven fixes.

---

# 71. Smell Classification by Scope

Review smells at:

```text
method
class
module
package
service
architecture
organization
```

Do not respond to a local smell with an architectural rewrite automatically.

---

# 72. Smell Classification by Volatility

A volatile dependency becomes more dangerous when broadly exposed.

A useful review model is:

```text
volatility × dependency surface × change frequency
```

---

# 73. Smell Classification by Risk

Prioritize using:

```text
customer impact
security impact
failure impact
change frequency
engineering effort
```

Not every smell is equally important.

---

# 74. Refactoring Prioritization

| Priority | Meaning |
|---|---|
| P0 | correctness, security, reliability risk |
| P1 | frequent and expensive change |
| P2 | meaningful maintainability drag |
| P3 | cosmetic/low-impact cleanup |

Use business context.

---

# 75. Refactoring Economics

A refactor has:

```text
cost today
future benefit
migration risk
operational cost
```

Compare all four.

---

# 76. Opportunity Cost

Engineering time spent on structural cleanup competes with feature work.

Refactor where the structural improvement unlocks meaningful future work or reduces meaningful risk.

---

# 77. Risk Budget

Higher-risk refactors need stronger safeguards:

```text
tests
feature flags
canaries
rollback
metrics
```

Use a process proportional to impact.

---

# 78. Refactoring Safety Ladder

A useful progression:

```text
formatting
  ↓
Extract Method
  ↓
Move Method
  ↓
Extract Class
  ↓
introduce boundary
  ↓
change dependency direction
  ↓
change service boundary
```

Higher levels require stronger validation.

---

# 79. Semantic Preservation

A safe refactor should preserve intentional observable semantics:

```text
outputs
errors
state transitions
side effects
contracts
security behavior
```

Do not accidentally preserve bugs simply because they happen to exist; classify them deliberately.

---

# 80. Characterization Tests

Characterization tests capture current observable behavior before refactoring.

They are valuable when:

```text
legacy code has weak tests
behavior is unclear
edge cases are undocumented
```

They create a safety net for structural change.

---

# 81. Golden Master

For complex outputs, compare current and new results:

```text
reports
renderers
transformations
legacy calculations
```

Be careful: a known bug should not become a permanent contract accidentally.

---

# 82. Small Steps

Prefer:

```text
one structural move
    ↓
compile
    ↓
test
    ↓
commit
```

This makes review and rollback easier.

---

# 83. Extract Method

Extract Method when a block has:

```text
clear purpose
clear inputs
clear output
```

Name the responsibility semantically.

---

# 84. Extract Class

Extract Class when a group has its own:

```text
state
invariants
change reason
conceptual purpose
```

Do not extract merely to reduce line count.

---

# 85. Move Method

Move behavior toward the object with better knowledge or responsibility ownership.

Check:

```text
cohesion
coupling
polymorphism
invariants
```

---

# 86. Move Field

Move state when another object has clearer ownership.

Check effects on:

```text
construction
serialization
persistence
invariants
tests
```

---

# 87. Replace Primitive with Value Object

A raw primitive may become a domain concept:

```ts
class Money { ... }
```

when it needs:

```text
validation
arithmetic
comparison
invariants
```

---

# 88. Replace Conditional with Polymorphism

When behavior varies by type:

```text
conditional
  ↓
stable contract
  ↓
variants
```

This is useful when the variation is meaningful and likely to evolve.

---

# 89. Introduce Parameter Object

Use when a set of parameters forms a meaningful concept.

Example:

```ts
type TaxContext = {
  country: string;
  rate: number;
  currency: string;
};
```

Avoid universal context objects.

---

# 90. Encapsulate Collection

If the object owns a collection invariant:

```ts
order.addLine(line);
```

is safer than:

```ts
order.lines.push(line);
```

when external mutation would bypass rules.

---

# 91. Replace Data Value with Object

A raw value may become an object when behavior and invariants become meaningful.

Examples:

```text
Money
Percentage
DateRange
Quantity
EmailAddress
```

---

# 92. Null Object — Use Carefully

A Null Object can remove repetitive absence checks.

But it may hide meaningful absence.

Use only when "no value" has stable valid behavior.

---

# 93. Replace Type Code with State or Strategy

Type codes can remain simple.

When every type has substantial behavior, consider:

```text
state
strategy
polymorphism
```

---

# 94. Separate Query from Modifier

Combining reads and mutations can make behavior surprising.

Use explicit command/query semantics when that improves domain clarity.

---

# 95. Separate Construction from Behavior

Complex construction may deserve:

```text
Factory → construction
Entity → behavior
```

This separates lifecycle policy from domain lifecycle behavior.

---

# 96. Separate Policy from Entity

A rule that varies independently may belong in:

```text
Entity + Policy
```

instead of continuously growing the entity.

---

# 97. Separate Integration from Domain

External integration can be contained:

```text
Domain/Application
    ↓
Port
    ↓
Adapter
```

This reduces provider coupling.

---

# 98. Separate Reporting from Domain

Reporting often has a different:

```text
query model
change cadence
performance profile
```

Keep it separate when that provides real value.

---

# 99. Separate Audit from Core Behavior

Audit can be a focused capability:

```ts
interface AuditLogger {
  record(entry: AuditEntry): Promise<void>;
}
```

The domain state transition remains distinct from audit storage.

---

# 100. Separate Authorization from State Rules

Authorization asks:

```text
May actor perform this?
```

Domain rules ask:

```text
Is this state transition valid?
```

One operation can require both.

---

# 101. Separate Tenant Scope from Business Logic

Tenant isolation can cross:

```text
request context
authorization
application
repository
database
```

Do not rely on one hidden global.

---

# 102. Refactoring With Dependency Direction

After a refactor, verify:

```text
Does the dependency point toward a stable concept?
```

A refactor that improves class size while worsening dependency direction can be a net loss.

---

# 103. Refactoring With Tests

Tests should target behavior rather than implementation details.

Example:

```ts
order.cancel();
expect(order.status).toBe("CANCELLED");
```

is often more stable than testing private representation.

---

# 104. Architecture Tests

Executable architecture constraints can prevent drift:

```text
domain must not import infrastructure
ordering cannot import payment internals
reporting cannot mutate domain state
```

These rules turn architecture into enforceable behavior.

---

# 105. Refactoring and Version Control

Prefer structural commits that each represent one meaningful move:

```text
characterization tests
extract boundary
delegate
move tests
remove old path
```

---

# 106. Refactoring and Observability

Preserve or deliberately update:

```text
metrics
logs
traces
correlation IDs
```

A structurally cleaner system is not useful if production behavior becomes invisible.

---

# 107. Refactoring and Performance

After structural change, compare where relevant:

```text
latency
DB queries
allocations
memory
serialization
network calls
```

Do not assume cleaner structure means faster execution.

---

# 108. Refactoring and Security

Review:

```text
authority
secret access
tenant isolation
authorization
data exposure
```

A refactor can accidentally broaden permissions.

---

# 109. Refactoring and Reliability

Check:

```text
retry semantics
timeouts
transaction boundaries
idempotency
failure mapping
```

Moving responsibility can accidentally change failure behavior.

---

# 110. Refactoring and Concurrency

Moving state across objects can change synchronization requirements.

Review:

```text
ownership
atomicity
locks/versioning
shared mutable state
```

---

# 111. Refactoring and Distributed Systems

A local refactor becomes an architectural change if it crosses a process boundary.

Do not introduce network calls merely to achieve class-level separation.

---

# 112. Modular Monolith Refactoring

A modular monolith can be a valuable target:

```text
clear ownership
stable module APIs
no network cost
```

It can improve dependency structure before service extraction is considered.

---

# 113. Service Extraction Threshold

A cohesive module should not automatically become a microservice.

Also consider:

```text
independent scaling
independent deployment
team ownership
fault isolation
data ownership
```

and accept the network/operations cost.

---

# 114. Backward Compatibility During Refactoring

A facade can preserve callers:

```ts
class LegacyPaymentService {
  constructor(private readonly gateway: PaymentGateway) {}

  charge(input: ChargeInput) {
    return this.gateway.charge(input);
  }
}
```

This allows responsibility to move behind an existing contract.

---

# 115. Branch by Abstraction

Use:

```text
stable abstraction
    ↓
old implementation / new implementation
```

for high-risk migrations.

---

# 116. Smell-to-Principle Map

```text
God object
    → cohesion + responsibility

Feature envy
    → information expert

Shotgun surgery
    → ownership

Long switch
    → possible polymorphism

Provider leakage
    → protected variations

Cycle
    → dependency direction

Global state
    → explicit ownership

Service blob
    → cohesion
```

---

# 117. Smell-to-Refactoring Map

```text
Long Method
    → Extract Method

Large Class
    → Extract Class

Feature Envy
    → Move Method

Data Clump
    → Parameter Object / Value Object

Switch variation
    → Polymorphism

Provider coupling
    → Port + Adapter
```

Mappings are starting points, not automatic transformations.

---

# 118. Smell Diagnosis Matrix

| Smell | Likely Force | First Question |
|---|---|---|
| God object | mixed responsibility | What are the clusters? |
| Service blob | weak cohesion | Who owns each decision? |
| Feature envy | misplaced behavior | Who has the knowledge? |
| Shotgun surgery | fragmented rule | Where is the owner? |
| Cycle | wrong direction | Who should coordinate? |
| Global state | implicit coupling | Can ownership be explicit? |
| Long switch | variation | Will variants grow? |
| Leaky abstraction | representation leakage | What stable capability is needed? |

---

# 119. Smell Review Workflow

```text
1. Name the smell.
2. Identify the underlying force.
3. Identify the affected responsibility.
4. Identify the change scenario.
5. Select the smallest useful refactoring.
6. Preserve behavior.
7. Test.
8. Review dependencies.
9. Measure production effects.
```

---

# 120. Principal-Level Smell Judgment

A principal engineer does not attempt to eliminate every smell.

Ask:

```text
What risk does it represent?
How often does it hurt?
What is the cost of leaving it?
What is the cost of fixing it?
What new complexity will refactoring introduce?
```

Sometimes the right answer is:

```text
accept the smell intentionally
```

and document why.

---

# 121. Intentional Smell

A smell can be deliberate.

Example:

```text
small stable switch
```

may be preferable to a strategy hierarchy when variation is limited.

---

# 122. Smell Acceptance Record

For an accepted smell, document:

```text
symptom
reason accepted
risk
trigger for revisit
owner
```

This turns accidental debt into managed debt.

---

# 123. Technical Debt vs Smell

A smell is evidence of possible structural cost.

Technical debt is broader: it describes the economic consequence of choosing a simpler or faster path now at a potential future cost.

Not every smell is technical debt.

Not all technical debt is visible as a classic smell.

---

# 124. Refactoring During Feature Work

Sometimes the safest refactor is made while implementing a feature because the relevant area is already under change.

Keep the structural scope focused.

Do not turn a feature ticket into an uncontrolled rewrite.

---

# 125. Boy Scout Principle — Carefully

Leaving a touched area clearer can be useful.

But:

```text
small improvement
    ≠
architecture rewrite
```

Stay within a controllable scope.

---

# 126. Code Review Questions

Reviewers should ask:

```text
What responsibility changed?
What dependencies changed?
What behavior is preserved?
What new abstraction was justified?
What risk moved?
```

---

# 127. Pairing and Complex Refactors

Complex responsibility migration benefits from a second set of eyes around:

```text
invariants
contracts
dependency direction
security
failure behavior
```

---

# 128. Commit Strategy

Useful commits are structurally meaningful:

```text
Add characterization tests
Extract payment port
Move provider adapter
Delegate legacy service
Remove obsolete dependency
```

This helps review and rollback.

---

# 129. Rollback Planning

For production refactors define:

```text
rollback point
success metric
failure signal
compatibility seam
```

Branch-by-abstraction and facades can lower rollback cost.

---

# 130. Feature Flags in Refactoring

Feature flags can separate:

```text
deployment
```

from:

```text
behavior rollout
```

Use them where runtime behavior is intentionally changing.

---

# 131. Canary Refactoring

For high-impact production changes, canary deployment can compare behavior on a limited traffic slice.

---

# 132. Shadow Execution

Shadowing can compare old and new results without using the new result directly.

Be careful with side effects, external calls, and duplicate writes.

---

# 133. Dual Write

Dual writes can help migration but create consistency and rollback complexity.

Use only when the migration requires it.

---

# 134. Data Migration Separation

Separate:

```text
structural code refactor
```

from:

```text
data/schema migration
```

when possible.

This reduces the number of simultaneous failure modes.

---

# 135. Schema Compatibility

During staged migrations, readers and writers may need temporary compatibility.

Design for rollback before removing old representations.

---

# 136. Event Compatibility

When refactoring event-driven responsibilities, preserve event contracts long enough to migrate consumers safely.

---

# 137. Distributed Ownership Migration

Moving state ownership between services is not a simple class refactor.

It can require:

```text
data migration
reconciliation
new contracts
operational agreements
consistency changes
```

---

# 138. Team Ownership Migration

Changing module responsibility can change team ownership.

Align code ownership with capability ownership where practical.

---

# 139. On-Call Implications

Refactoring can change failure boundaries.

Update:

```text
runbooks
metrics
alerts
rollback procedures
```

when operational ownership changes.

---

# 140. Dynamic JavaScript Reality

JavaScript can hide dependencies through:

```text
reflection
dynamic property access
string-based registration
plugins
dynamic imports
global state
```

Static search may therefore be incomplete.

Use runtime evidence and tests when needed.

---

# 141. TypeScript Reality

TypeScript improves static discoverability but does not remove runtime behavior.

External data still requires:

```text
runtime validation
mapping
error handling
```

---

# 142. Private Field Migration

Moving internal state into private fields can strengthen responsibility ownership.

Verify effects on:

```text
serialization
cloning
subclassing
testing
```

---

# 143. Prototype vs Instance Method Refactoring

Changing:

```ts
method() {}
```

to:

```ts
method = () => {}
```

can affect:

```text
allocation
memory
identity
this-binding
```

Do not treat this as behaviorally free.

---

# 144. Inheritance to Composition

When inheritance becomes fragile:

```text
extract capability
    ↓
compose collaborator
    ↓
delegate
    ↓
remove inheritance
```

Re-check substitutability and lifecycle semantics.

---

# 145. Factory Overgrowth

Factories can themselves become service blobs.

Split creation families or use registration when variants evolve independently.

---

# 146. Repository Overgrowth

A repository can become a dumping ground for unrelated queries.

Separate capabilities when ownership and change reasons are genuinely different.

---

# 147. Use-Case Overgrowth

A use case can accumulate too much business logic.

Keep orchestration but move invariant-heavy decisions toward domain owners.

---

# 148. Policy Overgrowth

A policy can become a god object.

Split by actual decision domain rather than line count.

---

# 149. Event Handler Overgrowth

A giant event handler that processes every event type can become low cohesion.

Split by meaningful event responsibility when useful.

---

# 150. Consumer Overgrowth

Message consumers should normally decode and dispatch.

Move complex workflows to use cases.

---

# 151. Job Overgrowth

A scheduler/job component containing unrelated tasks is often temporally grouped rather than functionally cohesive.

Split the tasks if their business and failure behavior differ.

---

# 152. Configuration Blob

One configuration object containing every setting can create broad coupling.

Use validated capability-specific configuration where meaningful.

---

# 153. Global Logger Smell

A global logger is convenient but creates implicit dependency.

Use explicit logging capability at the appropriate boundary when reviewability matters.

---

# 154. Global Clock Smell

Direct time access everywhere can make temporal behavior hard to test.

Use a Clock capability when deterministic time is a real requirement.

---

# 155. Global Randomness Smell

Direct randomness can complicate deterministic identity and tests.

Use an ID generator when identity generation is a meaningful boundary.

---

# 156. Hidden Cache Smell

A cache can silently become a source of truth.

Clarify:

```text
owner
scope
freshness
invalidation
fallback
```

---

# 157. Hidden Transaction Smell

A helper that silently starts and commits transactions may hide important ownership.

Make transaction responsibility explicit at the appropriate application boundary.

---

# 158. Hidden Authorization Smell

Authorization hidden inside generic helpers is difficult to audit.

Name permission responsibilities explicitly.

---

# 159. Hidden Retry Smell

Implicit retries can duplicate side effects.

Make:

```text
retryability
idempotency
backoff
```

explicit at technical boundaries.

---

# 160. Hidden Async Smell

Adding asynchronous boundaries everywhere can change failure, ordering, and transaction semantics.

Do not make domain behavior async without a real need.

---

# 161. Sync-to-Async Refactoring

When a responsibility becomes asynchronous, reassess:

```text
API contract
failure model
transaction boundaries
callers
tests
```

---

# 162. Refactoring to Events

Moving synchronous calls behind events changes timing and consistency.

Treat it as an architectural change, not merely a decoupling refactor.

---

# 163. Eventual Consistency Review

When introducing asynchronous behavior define:

```text
source of truth
delivery guarantees
retry behavior
idempotency
staleness
reconciliation
```

---

# 164. Function to Object Refactoring

Introduce an object when the responsibility gains meaningful:

```text
identity
state
lifecycle
polymorphism
collaboration
```

---

# 165. Object to Function Refactoring

Simplify an object into functions/modules when its identity, state, and lifecycle are no longer meaningful.

---

# 166. Function to Policy Refactoring

A function can become a policy when behavior needs:

```text
variation
configuration
polymorphism
injected capabilities
```

---

# 167. Policy to Function Refactoring

When variation disappears and the operation becomes simple and pure, simplifying back to a function can reduce unnecessary abstraction.

---

# 168. Entity to Value Object Refactoring

If identity and lifecycle stop mattering and value semantics become central, a concept may become a value object.

This is often a domain-model change, not merely code cleanup.

---

# 169. Value Object to Entity Refactoring

If identity and lifecycle become meaningful, a value concept may require an entity.

Migration must preserve data semantics.

---

# 170. Aggregate Refactoring

If several objects require one consistency boundary, an aggregate can make ownership explicit.

If an aggregate becomes a contention hotspot, investigate whether independent consistency boundaries exist.

---

# 171. Database Coupling Smell

Direct SQL in domain classes is often a boundary problem.

Consider repositories and adapters when persistence change or domain isolation justifies them.

---

# 172. Provider Coupling Smell

Provider-specific types spreading through the codebase indicate external coupling.

Contain them at adapters.

---

# 173. Security Smells as Priority Signals

Prioritize smells that:

```text
expand authority
bypass tenant isolation
bypass authorization
expose secrets
leak sensitive data
```

These are more than maintainability issues.

---

# 174. Performance Smells

Prioritize smells that create measurable:

```text
repeated queries
excessive allocations
network calls
serialization
CPU work
```

Profile before redesigning.

---

# 175. Memory Smells

Review:

```text
large object graphs
global references
unbounded caches
per-instance closures
```

Use profiling evidence.

---

# 176. Reliability Smells

Repeated technical failure handling can indicate missing cohesive infrastructure responsibilities.

Examples:

```text
retry policy
circuit breaker
timeout policy
reconciliation
```

---

# 177. Observability Smells

When one component owns unrelated behavior, telemetry becomes difficult to attribute.

Clear capability boundaries often improve diagnostics.

---

# 178. Testing Smells

If unit tests require:

```text
HTTP
DB
provider SDK
queue
cache
```

review coupling and responsibility placement.

---

# 179. Architecture Governance

Useful guardrails:

```text
architecture tests
import rules
package constraints
code ownership
ADR review
```

These keep refactoring gains from drifting away.

---

# 180. Documentation of Intentional Debt

Document accepted smells:

```text
what
why
risk
revisit trigger
owner
```

This prevents future engineers from rediscovering the same debate without context.

---

# 181. Legacy Code Reality

Legacy smells often encode history:

```text
deadlines
vendor constraints
old requirements
migration shortcuts
team changes
```

Respect the history while refactoring toward current requirements.

---

# 182. Legacy Service Refactoring Sequence

```text
characterization tests
    ↓
extract ports
    ↓
move domain decisions
    ↓
split technical responsibilities
    ↓
reduce globals
    ↓
enforce dependency direction
```

---

# 183. Legacy ORM Refactoring

When domain logic depends on ORM representation:

```text
ORM model
    ↓
mapper
    ↓
domain model
```

Migrate incrementally.

---

# 184. Provider Integration Refactoring

Use:

```text
PaymentGateway
    ↓
provider adapter
```

Then migrate callers away from provider-specific request and error types.

---

# 185. Service Blob Refactoring

Identify clusters:

```text
pricing
inventory
payment
notification
reporting
```

Move one cluster at a time.

---

# 186. God Object Refactoring

Start with:

```text
state
behavior
collaborators
change reasons
invariants
```

Extract the strongest boundary first.

---

# 187. Controller Blob Refactoring

Move:

```text
business decisions → domain/policy
persistence → repository
external calls → adapter
workflow → use case
```

Keep transport mapping in the controller.

---

# 188. Global Tenant State Refactoring

Replace broad ambient tenant state with explicit/scoped context and enforce persistence isolation where required.

---

# 189. Shared Cache Refactoring

Clarify:

```text
ownership
scope
consistency
invalidation
fallback
```

Then choose an explicit cache capability if needed.

---

# 190. Duplicate Rule Refactoring

Find repeated business decisions and move them to:

```text
entity
value object
policy
domain service
```

according to ownership.

---

# 191. Under-Abstracted Provider Refactoring

If a provider is volatile:

```text
direct SDK
    ↓
stable capability
    ↓
adapter
```

Contain provider semantics.

---

# 192. Over-Abstracted System Refactoring

Good refactoring can remove abstraction:

```text
merge trivial wrapper
remove dead interface
remove unnecessary factory
simplify dependency graph
```

---

# 193. Refactoring Verification

After a structural change verify:

```text
behavior
contracts
invariants
dependency direction
performance
memory
security
observability
reliability
```

---

# 194. Production Refactoring Checklist

```text
[ ] Smell identified from evidence.
[ ] Underlying force identified.
[ ] Responsibility owner reviewed.
[ ] Change scenario identified.
[ ] Current behavior characterized.
[ ] Migration is incremental.
[ ] Dependency direction reviewed.
[ ] Tests updated.
[ ] Performance checked where relevant.
[ ] Security checked.
[ ] Tenant isolation checked.
[ ] Reliability checked.
[ ] Observability preserved.
[ ] Rollback understood.
[ ] Old path removed when safe.
```

---

# 195. LLD Interview — What Is a Smell?

Strong answer:

> "A smell is a recurring design symptom that suggests future maintenance cost, such as weak cohesion, excessive coupling, or unclear responsibility. It is not automatically a defect; I investigate the underlying force and choose an appropriate refactoring only when the risk justifies it."

---

# 196. LLD Interview — Refactor a God Object

Strong answer:

> "I inventory responsibilities, state, invariants, collaborators, and change reasons. I cluster them semantically, identify natural owners, extract incrementally behind compatibility seams, and verify behavior after each step. I do not split solely by line count."

---

# 197. LLD Interview — Is a Large Class Always Bad?

Strong answer:

> "No. Size is a signal, not a verdict. I care about responsibility cohesion, change reasons, coupling, invariant ownership, and testability."

---

# 198. LLD Interview — Is a Service Class Bad?

Strong answer:

> "No. A cohesive application service can be a good orchestration boundary. The smell appears when the service becomes the default home for unrelated business rules or technical concerns."

---

# 199. LLD Interview — Should Every Switch Become Polymorphism?

Strong answer:

> "No. I use polymorphism when behavior is genuinely variable and the abstraction localizes meaningful change. A small stable switch can be clearer and cheaper."

---

# 200. LLD Interview — Refactoring Without Tests

Strong answer:

> "For legacy code I first establish characterization tests around important observable behavior. Then I make small structural changes, continuously verify behavior, and use integration or golden-master techniques where focused unit tests are difficult."

---

# 201. LLD Interview — When Not to Refactor

Strong answer:

> "When the smell creates little current cost, the area is stable, refactoring risk is high, or the change would distract from higher-value product or reliability work. I would document the accepted trade-off if the debt is intentional."

---

# 202. Predict the Smell — Service Blob

```text
OrderService changes whenever:
payment provider changes,
tax rules change,
email templates change,
order state rules change.
```

Likely:

```text
service blob
+
divergent change
```

First response:

```text
responsibility inventory
```

---

# 203. Predict the Smell — Global Tenant State

```text
20 modules directly mutate global tenant state.
```

Likely:

```text
common/global coupling
```

Potential direction:

```text
explicit context
+
enforced scope
```

---

# 204. Predict the Smell — Provider Leakage

```text
Five classes depend on Stripe response objects.
```

Likely:

```text
provider/model leakage
```

Potential response:

```text
PaymentGateway
+
provider adapter
```

---

# 205. Predict the Smell — Variation Leakage

```text
Every new shipping type requires editing five switches.
```

Likely:

```text
variation leakage
+
shotgun surgery
```

Consider a stable capability and polymorphism.

---

# 206. Predict the Smell — Large but Cohesive

```text
Order
    2,000 lines
    one complex state machine
```

Question:

```text
Is it actually low cohesion?
```

Do not answer from line count alone.

---

# 207. Predict the Smell — Over-Abstraction

```text
One-line helper
+ three interfaces
+ two factories
```

Likely:

```text
over-abstraction
```

Simplify if the boundaries provide no value.

---

# 208. Predict the Smell — Controller Blob

```text
Controller
 → database
 → provider SDK
 → domain mutation
```

Likely:

```text
controller blob
+
boundary erosion
```

---

# 209. Predict the Smell — Cyclic Dependencies

```text
Order ↔ Payment
```

Likely:

```text
dependency cycle
```

Investigate whether a use case should coordinate the relationship.

---

# 210. Predict the Smell — Feature Envy

```text
A method repeatedly reads five fields from another object and makes all the decisions.
```

Likely:

```text
feature envy
+
knowledge leakage
```

---

# 211. Predict the Smell — Anemic Model

```text
Entities contain fields only.
All business decisions live in services.
```

Likely:

```text
anemic domain model
```

Review invariant ownership.

---

# 212. Mastery Exercise — Smell Audit

Take an existing project and produce:

```text
top 10 design smells
underlying force
affected responsibility
change scenario
risk
candidate refactoring
evidence
```

Do not refactor solely because a smell is present.

---

# 213. Mastery Exercise — God Service

Start with:

```text
EcommerceManager
    users
    orders
    tax
    payment
    email
    reports
    inventory
    audit
```

Produce:

```text
responsibility matrix
cohesion clusters
dependency graph
target boundaries
migration sequence
tests
```

---

# 214. Mastery Exercise — Jewellery ERP

Audit a jewellery ERP workflow containing:

```text
SaleService
InventoryService
PaymentService
InvoiceService
TenantContext
AuditLogger
```

Look for:

```text
shared mutable state
provider leakage
tenant coupling
service blobs
duplicate rules
transaction coupling
```

Then propose the smallest safe refactoring.

---

# 215. Mastery Exercise — No Reference

Given:

```text
One legacy class calculates price,
reserves stock,
charges payment,
generates invoices,
sends email,
and exports reports.
```

Without notes:

```text
identify smells
identify forces
assign responsibility
select refactorings
define migration order
preserve behavior
defend trade-offs
```

---

# 216. Track A — Retrieval

Explain without notes:

```text
What is a design smell?
What is responsibility drift?
Why can god objects be expensive?
What is shotgun surgery?
What is divergent change?
What is feature envy?
Why is speculative generality risky?
Why can over-abstraction be harmful?
```

---

# 217. Track B — Implementation

Refactor a service blob through:

```text
characterization tests
Extract Method
Move Method
Extract Class
Introduce Port
Introduce Adapter
remove legacy path
```

After each step, run tests.

---

# 218. Track C — Interview

Practice:

> "How do you know when not to refactor?"

A strong answer considers:

```text
business impact
change frequency
risk
cost
team context
operational constraints
```

---

# 219. Principal Decision Framework

Evaluate a refactoring using:

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
What problem does the smell represent?
How often does it matter?
What is the cost of leaving it?
What is the cost of fixing it?
What new complexity will the refactor create?
Can it be incremental?
How will behavior be verified?
```

---

# 220. Completion Criteria

```text
[ ] I can define a design smell.
[ ] I can distinguish smell from defect.
[ ] I can identify responsibility drift.
[ ] I can diagnose cohesion smells.
[ ] I can diagnose coupling smells.
[ ] I can diagnose variation leakage.
[ ] I can diagnose representation leakage.
[ ] I can diagnose lifecycle/temporal coupling.
[ ] I can identify service blobs.
[ ] I can identify controller blobs.
[ ] I can identify god objects.
[ ] I can identify feature envy.
[ ] I can identify shotgun surgery.
[ ] I can identify divergent change.
[ ] I can identify dependency cycles.
[ ] I can identify global/shared-state coupling.
[ ] I can choose refactorings from underlying forces.
[ ] I can write characterization tests.
[ ] I can refactor incrementally.
[ ] I can preserve behavior and contracts.
[ ] I can review security during refactoring.
[ ] I can review reliability and transactions.
[ ] I can evaluate performance and memory effects.
[ ] I can reason about JavaScript runtime behavior.
[ ] I can use TypeScript effectively.
[ ] I can apply the method to a jewellery ERP.
[ ] I can defend a refactoring decision in an interview.
```

---

# 221. Revision / Retrieval Record

```text
Date:
________________

Top smell identified:
________________

Underlying force:
________________

Responsibility affected:
________________

Refactoring selected:
________________

Evidence:
________________

Risk:
________________

Result:
________________
```

---

# 222. Canonical References and Source Discipline

### ECMAScript

Use the ECMAScript specification for JavaScript language semantics:

https://tc39.es/ecma262/

### TypeScript

Use official TypeScript documentation:

https://www.typescriptlang.org/docs/

### Refactoring

Martin Fowler, *Refactoring: Improving the Design of Existing Code*, is a primary reference for named refactoring techniques and the broader discipline of behavior-preserving structural improvement.

### Object-Oriented Design

Craig Larman, *Applying UML and Patterns*, is useful for responsibility assignment, GRASP, and object design reasoning.

### Runtime distinction

For:

```text
V8 optimization
module loading
allocation
bundling
```

identify implementation-specific claims separately from language guarantees.

### Architecture distinction

Treat:

```text
repositories
ports/adapters
application services
bounded contexts
modular boundaries
```

as architectural guidance rather than ECMAScript semantics.

---

# 223. Canonical Source Discipline

Use:

```text
ECMAScript specification
    → language semantics

TypeScript documentation
    → compiler/type semantics

Node/browser documentation
    → host/runtime behavior

Design literature
    → refactoring and architecture guidance

Project requirements
    → project-specific constraints
```

Do not use a framework convention as proof that a responsibility belongs in a particular layer.

---

# 224. Concept Connections

```text
Chapter 12
    Cohesion + Coupling
        ↓
Chapter 13
    Responsibility assignment
        ↓
Chapter 14
    GRASP
        ↓
Chapter 15
    Cohesion-first decomposition
        ↓
Chapter 16
    Coupling-first dependency design
        ↓
Chapter 17
    Smell detection + refactoring
        ↓
Chapter 18
    Stable contracts + invariants
        ↓
SOLID / Patterns / DDD
        ↓
Production LLD
```

Core progression:

```text
observe symptom
    ↓
identify force
    ↓
identify responsibility issue
    ↓
choose smallest useful refactoring
    ↓
verify behavior
    ↓
measure production outcome
```

---

# 225. Final Mental Model

When you notice a smell, do not ask first:

```text
"Which refactoring should I apply?"
```

Ask:

```text
What is wrong or expensive?
        ↓
What responsibility is unclear?
        ↓
What knowledge is leaking?
        ↓
What dependency is unnecessary?
        ↓
What changes together?
        ↓
What varies independently?
        ↓
What invariant lacks an owner?
        ↓
What failure/security risk exists?
        ↓
What is the smallest safe structural change?
        ↓
How will I verify it?
```

---

# 226. Final Design Principle

> **Treat smells as evidence, not verdicts: identify the underlying design force, make responsibility and dependency boundaries explicit, refactor incrementally, and verify the result against behavior, security, reliability, performance, and future change.**

---

# 227. Chapter Completion Snapshot

```text
Theory:
[+] Design smells
[+] Responsibility drift
[+] Cohesion smells
[+] Coupling smells
[+] Variation smells
[+] Representation leakage
[+] Authority leakage
[+] Lifecycle/temporal coupling
[+] Global/shared state
[+] Dependency cycles

Major smells:
[+] Long Method
[+] Long Parameter List
[+] Large Class
[+] Primitive Obsession
[+] Data Clumps
[+] Feature Envy
[+] Inappropriate Intimacy
[+] Message Chains
[+] Middle Man
[+] Speculative Generality
[+] Anemic Domain Model
[+] Service Blob
[+] Controller Blob
[+] God Object
[+] Shotgun Surgery
[+] Divergent Change
[+] Leaky Abstraction
[+] Inappropriate Inheritance

Refactoring:
[+] Characterization tests
[+] Golden master
[+] Extract Method
[+] Extract Class
[+] Move Method
[+] Move Field
[+] Value Objects
[+] Polymorphism
[+] Parameter Objects
[+] Encapsulated Collections
[+] Policy extraction
[+] Ports/adapters
[+] Incremental migration
[+] Branch by abstraction
[+] Backward compatibility

JavaScript / TypeScript:
[+] Dynamic runtime concerns
[+] Module cycles
[+] Private state
[+] Structural typing
[+] Runtime vs compile-time boundaries
[+] Prototype/inheritance considerations

Production:
[+] Security
[+] Multi-tenancy
[+] Performance
[+] Memory
[+] Reliability
[+] Observability
[+] Transactions
[+] Concurrency
[+] Distributed systems
[+] Migration
[+] Rollback

Interview:
[+] Smell diagnosis
[+] Refactoring strategy
[+] Principal-level trade-offs
[+] Predict-the-smell
[+] Jewellery ERP audit
```

# Chapter 17 — Completion Statement

A mature engineer does not ask how to eliminate every smell. The goal is to understand what structural cost a smell represents, whether that cost matters now, and what smallest safe change improves ownership, cohesion, coupling, testability, security, reliability, and future change without creating a worse problem.

Next chapter: **Designing Stable Contracts and Invariants**.
