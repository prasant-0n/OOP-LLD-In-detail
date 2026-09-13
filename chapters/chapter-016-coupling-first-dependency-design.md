# Chapter 16 — Coupling-First Dependency Design

> **Part:** C — Responsibility-Driven Design  
> **Prerequisite:** Chapters 12–15  
> **Next:** Chapter 17 — Design Smells, Responsibility Drift & Refactoring  
> **Status:** `[+] Completed`

---

# Chapter Position

Chapter 12 introduced cohesion and coupling as central design forces.

Chapter 13 established responsibility ownership.

Chapter 14 formalized responsibility assignment through GRASP.

Chapter 15 showed how cohesion can guide decomposition.

This chapter focuses on the other half of that design equation:

> **Given cohesive responsibilities, who should depend on whom, how much should they know, and where should dependency boundaries live?**

The goal is not zero coupling.

The goal is:

```text
necessary collaboration
+
small dependency surface
+
stable direction
+
controlled change propagation
```

---

# 1. Learning Objectives

By the end of this chapter, you should be able to:

- define coupling precisely
- distinguish necessary coupling from accidental coupling
- explain data, stamp, control, common, content, temporal, and external coupling
- reason about fan-in and fan-out
- explain afferent and efferent coupling
- design dependency direction deliberately
- identify dependency cycles
- apply dependency inversion without interface-everywhere thinking
- use capability interfaces, ports, adapters, composition, and dependency injection appropriately
- identify coupling caused by globals, shared mutable state, framework objects, ORM models, schemas, provider SDKs, and event contracts
- reason about coupling in JavaScript and TypeScript
- analyze coupling in Node.js, browser applications, monorepos, modular monoliths, and distributed systems
- evaluate coupling against cohesion, performance, memory, security, reliability, observability, and operations
- refactor highly coupled components incrementally
- defend coupling decisions in LLD interviews and principal-level architecture reviews

---

# 2. Prerequisites

Recommended:

```text
Chapter 12 — Cohesion & Coupling
Chapter 13 — Responsibility-Driven Object Design
Chapter 14 — GRASP
Chapter 15 — Cohesion-First Object Decomposition
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

# 3. What Is Coupling?

**Coupling** describes how strongly one unit depends on another.

A unit may be:

```text
function
class
module
package
component
service
database
external provider
team
```

A practical question is:

> **What must change in A when B changes?**

The more unnecessary assumptions, knowledge, authority, or coordination cross the boundary, the stronger the coupling.

---

# 4. Coupling Is Not Automatically Bad

Useful systems necessarily contain dependencies.

```text
Order → OrderLine
CheckoutUseCase → OrderRepository
```

Those dependencies express meaningful collaboration.

The goal is:

```text
intentional
necessary
narrow
stable
understandable
```

not:

```text
zero dependencies
```

---

# 5. Coupling as Change Propagation

Consider:

```text
B changes
    ↓
Does A have to change?
    ↓
How many other units also change?
```

This produces a useful mental model:

```text
coupling risk
    ≈
knowledge + assumptions + change propagation
```

A stable boundary localizes change.

---

# 6. Coupling as Knowledge

A dependency is a form of knowledge.

Broad:

```ts
stripe.paymentIntents.create(...)
stripe.paymentIntents.confirm(...)
stripe.customers.create(...)
```

The caller must understand many provider concepts.

Narrow:

```ts
paymentGateway.charge(...)
```

The caller depends on one capability.

---

# 7. Coupling as Authority

Dependencies also grant authority.

Compare:

```ts
constructor(private readonly db: Database)
```

with:

```ts
constructor(
  private readonly orders: OrderRepository
) {}
```

The database grants broad authority.

The repository grants narrower authority.

Therefore:

```text
lower coupling
+
least authority
```

can reinforce each other.

---

# 8. Coupling Categories

Useful classical categories include:

```text
Data coupling
Stamp coupling
Control coupling
Common/global coupling
Content coupling
Temporal coupling
External coupling
```

These describe why dependency exists and where it becomes risky.

---

# 9. Data Coupling

Data coupling occurs when one unit passes only the information needed.

Example:

```ts
pricing.calculate(price, quantity);
```

The dependency is narrow and explicit.

This is often a relatively healthy form of coupling.

---

# 10. Stamp Coupling

Stamp coupling occurs when a whole structured object is passed even though only part is needed.

Example:

```ts
function sendWelcome(customer: Customer) {
  mailer.send(customer.email);
}
```

If the caller passes the entire customer while only `email` matters, the function knows more than necessary.

But do not reduce every object parameter to primitives mechanically.

A domain object may itself be the meaningful boundary.

---

# 11. Control Coupling

Control coupling occurs when one component tells another which behavior mode to use.

Example:

```ts
render(order, "PDF");
render(order, "CSV");
render(order, "JSON");
```

The caller now knows the callee's behavioral modes.

Potential alternatives:

```text
polymorphism
separate capability interfaces
separate operations
strategy objects
```

Use the simplest useful option.

---

# 12. Common / Global Coupling

Common coupling occurs through shared global state.

Examples:

```ts
globalThis.currentTenantId
```

or:

```ts
const globalState = {};
```

Many components can now affect one another indirectly.

This makes:

```text
ordering
ownership
testing
debugging
```

harder.

---

# 13. Content Coupling

Content coupling is strong dependency on another component's internal representation.

Bad:

```ts
order._state.status
```

or:

```ts
order.items.push(...)
```

when the collection is supposed to be owned by `Order`.

Semantic operations reduce this coupling:

```ts
order.confirm();
order.addLine(line);
```

---

# 14. Temporal Coupling

Temporal coupling exists when one operation must happen before another.

Example:

```ts
client.connect();
client.authenticate();
client.query();
```

The caller must know lifecycle ordering.

A higher-level abstraction can encapsulate sequencing:

```ts
client.executeAuthenticated(command);
```

when hiding the lifecycle is beneficial.

---

# 15. External Coupling

External coupling comes from direct dependence on:

```text
framework APIs
database schemas
SDKs
network protocols
file formats
OS facilities
cloud services
```

External dependencies cannot always be removed.

The design question is:

> Where should external coupling be contained?

---

# 16. Coupling Surface

**Coupling surface** is a practical way to think about how much of another component you must understand.

Large:

```text
provider request model
provider error types
provider lifecycle
provider SDK methods
provider configuration
```

Small:

```text
PaymentGateway.charge(input)
```

Smaller surfaces usually make change more local.

---

# 17. Dependency Surface Area

Review:

```text
number of dependencies
number of methods used
number of types imported
number of representations understood
number of lifecycle assumptions
number of failure modes understood
```

Count alone is insufficient.

---

# 18. Fan-Out

Fan-out is the number of outgoing collaborators.

Example:

```text
CheckoutUseCase
├── OrderRepository
├── Inventory
├── PaymentGateway
├── PricingPolicy
├── TaxPolicy
├── AuditLogger
└── NotificationSender
```

High fan-out can be valid for an orchestrator.

It can also reveal responsibility overload.

---

# 19. Fan-In

Fan-in is the number of incoming dependencies.

High fan-in can indicate:

```text
stable shared capability
```

but it can also create a central bottleneck.

Example:

```text
20 modules
    ↓
LegacyUtility
```

One change can propagate widely.

---

# 20. Afferent and Efferent Coupling

At module/package level:

```text
Afferent coupling
    incoming dependencies

Efferent coupling
    outgoing dependencies
```

A module with many dependents and few dependencies can be relatively stable.

A module with many outgoing dependencies may be more volatile.

These are diagnostic signals, not complete architecture metrics.

---

# 21. Dependency Direction

Direction matters.

Prefer:

```text
Domain
    ↓
stable capability
```

over:

```text
Domain
    ↓
volatile infrastructure
```

Example:

```text
Order
    ↓
PaymentGateway
    ↓
StripeAdapter
```

rather than:

```text
Order
    ↓
Stripe SDK
```

---

# 22. Dependency Inversion

Dependency inversion means high-level policy should not be forced to depend directly on low-level implementation details.

Use a stable boundary:

```ts
interface PaymentGateway {
  charge(input: ChargeInput): Promise<ChargeResult>;
}
```

Then:

```ts
class StripePaymentGateway implements PaymentGateway {
  ...
}
```

The dependency points toward a capability.

---

# 23. Dependency Inversion Is Not Interface Everywhere

Bad rule:

```text
Every class needs an interface.
```

Better:

```text
Create an abstraction when it:
    protects credible variation
    narrows a dependency
    expresses a stable capability
    establishes a useful architectural boundary
    improves substitutability
```

Otherwise direct dependency may be simpler.

---

# 24. Concrete Coupling

Example:

```ts
class CheckoutUseCase {
  constructor(
    private readonly stripe: StripeClient
  ) {}
}
```

The use case knows the provider.

This may be acceptable for a stable internal implementation.

It becomes expensive when:

```text
provider changes
multiple providers exist
testing requires provider setup
provider details leak into domain code
```

---

# 25. Capability Coupling

Better:

```ts
interface PaymentGateway {
  charge(input: ChargeInput): Promise<ChargeResult>;
}
```

The caller knows:

```text
what capability is available
```

without knowing:

```text
which provider
```

This usually creates a smaller dependency surface.

---

# 26. TypeScript Structural Typing

A narrow interface can be implemented by:

```text
class
object literal
fake
adapter
wrapper
```

Example:

```ts
const fakeGateway: PaymentGateway = {
  async charge(input) {
    return { id: "test" };
  }
};
```

This supports low-coupling test seams.

---

# 27. Runtime Reality

TypeScript interfaces do not exist at runtime.

Therefore type-level dependency inversion does not validate:

```text
HTTP payloads
queue messages
provider responses
untrusted input
```

Runtime validation remains a separate responsibility.

---

# 28. Module Coupling

Modules couple through more than imports.

Review:

```text
imports
exports
re-exports
shared state
side effects
initialization order
global configuration
module-level caches
```

---

# 29. Import Coupling

A module importing many unrelated capabilities may have high efferent coupling.

Example:

```ts
import { Stripe } from "...";
import { Redis } from "...";
import { S3 } from "...";
import { PrismaClient } from "...";
```

Review whether one responsibility really needs all of them.

---

# 30. Export Coupling

Every export invites consumers to depend on it.

Therefore:

```text
public API
    = potential future coupling
```

Export only meaningful stable capabilities.

---

# 31. Barrel Files

Barrels can simplify imports:

```ts
export * from "./orders";
export * from "./payments";
export * from "./inventory";
```

But broad barrels can increase:

```text
cycle risk
public API size
dependency ambiguity
```

Use them deliberately.

---

# 32. Side-Effect Coupling

This:

```ts
import "./register-all-handlers";
```

can make import order part of behavior.

Now merely importing a module performs work.

Explicit initialization may provide a clearer lifecycle.

---

# 33. Initialization Coupling

If module A assumes module B has already run:

```text
B must initialize before A
```

the dependency is implicit and temporal.

Make lifecycle assumptions explicit where practical.

---

# 34. Shared Mutable State

A common source of coupling:

```ts
const state = {
  user: null,
  cart: [],
  tenantId: null
};
```

Many modules mutate it.

Behavior depends on:

```text
who changed it
when
in what order
```

Clear ownership reduces this problem.

---

# 35. Node.js Global State

Avoid broad ambient state such as:

```ts
globalThis.currentTenantId = tenantId;
```

when explicit scoped dependencies can work.

Global state makes request isolation and reasoning harder.

---

# 36. Environment Coupling

Scattered access:

```ts
process.env.PAYMENT_PROVIDER
```

couples unrelated modules to process configuration.

Centralize validated configuration when useful.

---

# 37. Framework Coupling

If a domain object directly accepts:

```ts
Express.Request
```

or:

```ts
PrismaClient
```

the domain becomes coupled to the host framework.

Map external values into domain/application types.

---

# 38. ORM Coupling

ORM entities often expose storage-shaped representations.

Example:

```text
customer_id
created_at
status_code
```

while domain concepts may be:

```text
customerId
createdAt
status
```

Mapping can reduce representation coupling.

---

# 39. Database Schema Coupling

If many components directly query shared tables:

```text
A ─┐
B ─┼→ database schema
C ─┘
```

the schema becomes a broad coordination contract.

This can be practical, but it increases change coupling.

---

# 40. Shared Database as Organization-Level Coupling

A shared database can become:

```text
organizational API
```

where one team's table changes require multiple teams to coordinate.

Modular boundaries around persistence can reduce that coupling.

---

# 41. Provider SDK Coupling

Bad:

```text
Order → Provider SDK
Invoice → Provider SDK
Controller → Provider SDK
RefundService → Provider SDK
```

Better:

```text
PaymentGateway
    ↓
ProviderAdapter
```

The provider dependency becomes localized.

---

# 42. External API Coupling

Do not allow external provider models to spread through the domain:

```text
StripePaymentIntent
StripeError
StripeCustomer
```

unless those concepts are genuinely part of your domain.

Use translation where the boundary is valuable.

---

# 43. Anti-Corruption Layer

An anti-corruption layer can isolate another model:

```text
External Model
    ↓
Translator
    ↓
Internal Model
```

Useful across:

```text
legacy systems
bounded contexts
third-party APIs
acquired platforms
```

---

# 44. DTO Coupling

DTOs can decouple external representations:

```text
HTTP DTO
    ↓
Command
    ↓
Domain
```

Use them when external representation and internal representation change for different reasons.

Do not create DTO copies without a boundary benefit.

---

# 45. Control Coupling Refactoring

Before:

```ts
render(order, "PDF");
render(order, "CSV");
render(order, "JSON");
```

Potential alternatives:

```ts
pdfRenderer.render(order);
csvExporter.export(order);
```

or a strategy when runtime selection is a real variation.

---

# 46. Stamp Coupling Refactoring

Before:

```ts
welcomeEmail.send(customer);
```

when only email is needed.

Potentially:

```ts
welcomeEmail.send(customer.email);
```

But if the semantic contract requires customer identity or more customer behavior, passing the object may still be correct.

Minimize unnecessary knowledge, not meaningful domain context.

---

# 47. Temporal Coupling Refactoring

Before:

```ts
client.connect();
client.authenticate();
client.query();
```

Potentially:

```ts
client.runAuthenticated(query);
```

This hides ordering that callers should not need to manage.

---

# 48. Common Coupling Refactoring

Before:

```ts
globalThis.currentTenantId
```

Potentially:

```ts
interface TenantContext {
  tenantId: string;
}
```

or explicit tenant scope in the use-case command.

The dependency becomes visible.

---

# 49. Content Coupling Refactoring

Before:

```ts
payment._state.status = "CAPTURED";
```

After:

```ts
payment.capture();
```

Representation becomes internal.

---

# 50. Coupling and Semantic APIs

Prefer:

```ts
order.cancel();
order.confirm();
order.addLine(line);
```

over:

```ts
order.status = "CANCELLED";
order.status = "CONFIRMED";
order.items.push(line);
```

when state transitions have rules.

Semantic APIs reduce representation coupling.

---

# 51. Coupling and Tell, Don't Ask

Ask-style:

```ts
if (order.status === "PENDING") {
  order.status = "CONFIRMED";
}
```

Tell-style:

```ts
order.confirm();
```

The caller now knows less about the order's internals.

---

# 52. Coupling and Law of Demeter

Risky:

```ts
sale.customer.account.branch.taxProfile.rate
```

Potentially better:

```ts
sale.taxRate();
```

The goal is reducing dependence on object topology.

Do not turn this into an absolute ban on method chaining.

---

# 53. Coupling and Information Expert

When behavior is placed near the information it needs, callers often need less knowledge.

This is why:

```text
Information Expert
```

and:

```text
Low Coupling
```

often reinforce one another.

---

# 54. Coupling and High Cohesion

Chapter 15 asked:

```text
What belongs together?
```

This chapter asks:

```text
What should depend on what?
```

Use both:

```text
cohesive responsibilities
+
controlled dependencies
```

---

# 55. Coupling and Polymorphism

Polymorphism can reduce control coupling:

```text
caller
    ↓
PaymentMethod
    ↓
Card / UPI / Cash
```

But too many abstractions can increase conceptual coupling.

Use it for meaningful variation.

---

# 56. Coupling and Indirection

Indirection changes:

```text
A → B
```

into:

```text
A → I ← B
```

This can reduce direct coupling.

But it introduces another component.

Evaluate total complexity.

---

# 57. Coupling and Protected Variations

Protected Variations identifies:

```text
volatile boundary
```

and creates:

```text
stable boundary
```

around it.

Examples:

```text
PaymentGateway
TaxPolicy
Clock
IdGenerator
```

---

# 58. Coupling and Pure Fabrication

Pure Fabrication can give technical responsibility a cohesive home:

```text
Repository
AuditLogger
EmailSender
ProviderAdapter
```

This keeps domain objects from depending directly on infrastructure.

---

# 59. Dependency Graph

Represent:

```text
A → B
```

as:

```text
A depends on B
```

Example:

```text
Controller
    ↓
UseCase
    ↓
Order
```

and:

```text
UseCase
 ├── Repository
 ├── Inventory
 └── PaymentGateway
```

The graph reveals dependency direction.

---

# 60. Dependency Graph Review

For every edge ask:

```text
Why does A need B?
What capability does B provide?
What does A know about B?
How stable is B?
How broad is B's API?
Can the dependency be narrowed?
Could direction be reversed?
Would an intermediary help?
```

---

# 61. Dependency Cycles

Cycle:

```text
A → B
B → C
C → A
```

Cycles increase:

```text
initialization complexity
testing difficulty
change propagation
conceptual load
```

Avoid unnecessary cycles.

---

# 62. JavaScript Module Cycles

Examples:

```text
order.ts → payment.ts
payment.ts → order.ts
```

ES modules have defined cycle semantics, but cycles still increase design complexity.

Use dependency direction to avoid unnecessary circular architecture.

---

# 63. Breaking a Cycle

Possible techniques:

```text
move coordination upward
extract shared stable concept
reverse dependency
introduce a capability/port
split responsibility
```

Do not introduce interfaces reflexively.

---

# 64. Dependency Stability

A stable dependency can safely have many consumers.

A volatile dependency should have fewer consumers or a protection boundary.

Example:

```text
PaymentGateway
    many consumers

StripeAdapter
    one infrastructure boundary
```

This limits provider-driven change.

---

# 65. Stable Abstractions

A useful boundary should itself be relatively stable.

If:

```text
PaymentGateway
```

changes whenever the provider changes, the abstraction failed to protect consumers.

The internal concept should be more stable than its implementation details.

---

# 66. Dependency Inversion Trade-Off

Adding:

```ts
interface PaymentGateway {}
```

costs:

```text
more code
more wiring
more navigation
```

Benefits:

```text
provider isolation
testing
change localization
clear capability
```

Make the trade-off explicit.

---

# 67. Constructor Injection

Constructor injection makes required dependencies visible:

```ts
class CheckoutUseCase {
  constructor(
    private readonly payment: PaymentGateway,
    private readonly orders: OrderRepository
  ) {}
}
```

This is usually easier to reason about than hidden lookups.

---

# 68. Service Locator Coupling

Weak:

```ts
const payment =
  container.resolve(PaymentGateway);
```

The object now depends on the container.

Its dependency graph is hidden.

Explicit injection usually communicates responsibility more clearly.

---

# 69. Composition Root

Concrete dependencies can be assembled in one composition area:

```text
main.ts
    StripePaymentGateway
        ↓
CheckoutUseCase
        ↓
Controller
```

The core model does not need to know the concrete wiring.

---

# 70. Node.js Coupling Sources

Common sources:

```text
process.env
globalThis
framework request/response
ORM objects
provider SDKs
singletons
module side effects
global event buses
```

Review these deliberately.

---

# 71. Browser Coupling Sources

Common sources:

```text
window
document
localStorage
fetch
framework component objects
global event buses
```

Move platform-specific behavior behind adapters when domain portability matters.

---

# 72. Platform Coupling

A pure domain function:

```ts
function calculateTax(...) {}
```

has low platform coupling.

A domain class calling:

```ts
window.localStorage.getItem(...)
```

has strong browser coupling.

Keep platform concerns at boundaries where possible.

---

# 73. Coupling and Testing

Broad dependencies increase setup cost.

If:

```text
Order.confirm()
```

requires:

```text
HTTP
database
Stripe
Redis
```

the object is likely over-coupled.

A focused domain object should generally have a much smaller dependency surface.

---

# 74. Coupling and Test Doubles

Narrow capability interfaces make simple fakes possible:

```ts
const fakeClock: Clock = {
  now: () => fixedDate
};
```

Large interfaces force large test doubles.

That is a practical coupling signal.

---

# 75. Interface Segregation Connection

Broad interfaces couple clients to operations they do not need.

Prefer:

```ts
interface PaymentCharger {
  charge(input: ChargeInput): Promise<ChargeResult>;
}
```

when charging is the actual capability.

---

# 76. Coupling and Least Authority

A narrow capability can reduce both:

```text
dependency surface
authority surface
```

Example:

```text
OrderRepository
```

is safer and clearer than:

```text
Database
```

when the component only needs order persistence.

---

# 77. Coupling and Security

Security-sensitive dependencies deserve special scrutiny.

Ask:

```text
Does this component receive secrets?
Does it receive write access?
Does it receive cross-tenant access?
Does it receive infrastructure administration capability?
```

Dependency design affects blast radius.

---

# 78. Coupling and Secrets

Avoid injecting:

```text
all application secrets
```

into components that need one credential.

Prefer scoped configuration or capabilities.

---

# 79. Coupling and Multi-Tenancy

Tenant context is a high-risk coupling source.

Avoid making tenant identity an uncontrolled ambient variable.

Possible layers:

```text
Request context
    ↓
authorization
    ↓
use-case scope
    ↓
repository enforcement
    ↓
database enforcement
```

---

# 80. Coupling and Tenant Isolation

A tenant-aware repository can provide a narrower capability:

```text
TenantScopedOrderRepository
```

rather than giving every component arbitrary database access.

Defense in depth may require multiple enforcement layers.

---

# 81. Coupling and Reliability

Every dependency imports possible failure.

```text
PaymentGateway
    timeout
    rejection
    provider outage

Repository
    connection failure
```

Keep failure translation near the boundary.

---

# 82. Coupling and Resilience

Technical wrappers can isolate failure behavior:

```text
RetryPolicy
CircuitBreaker
TimeoutPolicy
Bulkhead
```

The domain need not understand the implementation of those mechanisms.

---

# 83. Coupling and Transactions

Transaction boundaries can create intentional coupling among operations.

Example:

```text
TransferMoneyUseCase
    ├── withdraw
    ├── deposit
    └── save
```

Before splitting a workflow, understand consistency requirements.

---

# 84. Coupling and Concurrency

Shared mutable state increases coupling between concurrent operations.

Prefer:

```text
Inventory.reserve(...)
```

over direct mutation of:

```text
availableQuantity
```

Technical atomicity may then be enforced with:

```text
transaction
lock
atomic update
optimistic version
```

---

# 85. Coupling and Distributed Systems

Network dependency is stronger than local object dependency.

A service call introduces:

```text
latency
timeouts
serialization
partial failure
versioning
observability
```

Do not equate:

```text
distributed
```

with:

```text
decoupled
```

---

# 86. Coupling and Microservices

Microservices can reduce:

```text
code ownership coupling
```

while increasing:

```text
network coupling
contract coupling
deployment coupling
operational coupling
consistency coupling
```

Evaluate the full system.

---

# 87. Coupling and Modular Monoliths

A modular monolith can provide strong dependency boundaries without network costs.

Example:

```text
Ordering
Payments
Inventory
```

communicating through stable module APIs.

This is often an excellent coupling trade-off.

---

# 88. Coupling and Events

Events reduce direct dependency:

```text
Order
   → OrderConfirmed
```

instead of:

```text
Order
   → NotificationService
```

But event schemas become contracts.

Events trade some direct coupling for:

```text
asynchronous consistency complexity
```

---

# 89. Event Coupling

Consumers may depend on:

```text
event fields
event semantics
ordering
delivery guarantees
timing
version
```

So events are not "free decoupling."

They create a different coupling surface.

---

# 90. Orchestration vs Choreography

Orchestration:

```text
UseCase
 ├── Inventory
 ├── Payment
 └── Repository
```

Choreography:

```text
A emits event
    ↓
B reacts
    ↓
C reacts
```

Orchestration makes dependency explicit.

Choreography can reduce direct references but creates implicit event coupling.

---

# 91. Coupling and Bounded Contexts

A bounded context should avoid depending directly on another context's internal model.

Use:

```text
API
events
adapters
anti-corruption layers
```

where appropriate.

---

# 92. Coupling and Anti-Corruption Layer

An anti-corruption layer absorbs:

```text
schema differences
terminology
workflow differences
error models
```

This prevents external concepts from becoming internal coupling.

---

# 93. Coupling and Legacy Systems

Use:

```text
NewDomain
    ↓
LegacyGateway
    ↓
LegacySystem
```

rather than spreading legacy concepts through every module.

---

# 94. Coupling and API Versioning

Provider schemas can change.

A stable internal capability can isolate:

```text
Provider v1
Provider v2
```

behind adapters.

---

# 95. Coupling and Schema Evolution

Repositories and mappers can contain database representation changes:

```text
Domain
    ↓
Repository
    ↓
DB mapping
```

The domain model does not need to mirror the schema.

---

# 96. Coupling and Caches

Caches introduce coupling through:

```text
key format
invalidation
freshness
serialization
consistency
```

A cache interface can contain these concerns.

---

# 97. Coupling and Read Models

A read model can decouple reporting from operational domain models:

```text
Dashboard
    ↓
SalesReadModel
```

rather than:

```text
Dashboard
    ↓
10 repositories
```

The trade-off is duplicated/derived data.

---

# 98. Coupling and Reporting

Reporting may legitimately combine multiple domains.

That does not mean every domain entity should know reporting queries.

Keep read-side composition separate when useful.

---

# 99. Coupling and ORM Lazy Loading

Deep navigation may hide persistence behavior:

```ts
order.customer.account.branch
```

What looks like object access may trigger database queries.

That creates hidden data-access coupling.

---

# 100. Coupling and N+1

```ts
orders.map(
  order => order.customer.name
);
```

may create repeated queries depending on persistence mapping.

Dependency design must consider runtime data access, not only type relationships.

---

# 101. Coupling and Serialization

Cross-process boundaries require:

```text
serialization
validation
versioning
mapping
```

These are part of the total dependency cost.

---

# 102. Coupling and Error Models

Provider-specific errors should usually stop at the adapter boundary.

Example:

```text
StripeError
    ↓
PaymentProviderAdapter
    ↓
PaymentFailure
```

This keeps internal code independent of provider error classes.

---

# 103. Coupling and Logging

Direct dependence on one logger implementation can couple domain code to infrastructure.

Use a logging capability only where logging is genuinely needed.

---

# 104. Coupling and Time

Direct `new Date()` calls can create implicit time dependence.

Where deterministic testing or time policies matter:

```ts
interface Clock {
  now(): Date;
}
```

makes the dependency explicit.

---

# 105. Coupling and Randomness

Direct:

```ts
Math.random()
```

can make identity generation nondeterministic.

Use:

```ts
interface IdGenerator {
  generate(): string;
}
```

when the variation matters.

---

# 106. Coupling and Configuration

Scattered configuration access couples modules to environment structure.

Centralized configuration can provide:

```text
validation
stable shape
security
testability
```

---

# 107. Coupling and Feature Flags

Direct provider coupling:

```ts
launchDarkly.isEnabled(...)
```

can spread through business modules.

A capability abstraction can isolate the provider when multiple implementations or testing require it.

---

# 108. Coupling and Public API

Every exported symbol can become a dependency.

Therefore:

```text
export surface
    → future coupling surface
```

Keep exports intentional.

---

# 109. Coupling and Representation Independence

This:

```ts
order.status = "CANCELLED";
```

couples clients to internal representation.

This:

```ts
order.cancel();
```

couples clients to a semantic responsibility.

The semantic contract is more stable.

---

# 110. Coupling and Change Scenarios

For every dependency ask:

```text
What if B changes?
```

Examples:

```text
Stripe field changes
    → adapter

Tax formula changes
    → policy

DB implementation changes
    → repository

HTTP payload changes
    → controller/mapper
```

Good boundaries localize credible change.

---

# 111. Dependency Risk Matrix

| Dependency | Stability | Surface | Failure Impact | Risk |
|---|---|---|---|---|
| OrderLine | high | small | low | low |
| Clock | high | tiny | low | low |
| Payment SDK | volatile | broad | high | high |
| Shared DB schema | variable | broad | high | high |
| Tax Policy | variable | focused | medium | medium |
| Global mutable state | unclear | implicit | high | very high |

---

# 112. Dependency Review Worksheet

```text
Component:
________________

Dependency:
________________

Capability needed:
________________

Knowledge crossing:
________________

Authority crossing:
________________

Stability:
________________

Surface size:
________________

Failure impact:
________________

Change impact:
________________

Can it be narrowed?
________________

Should direction change?
________________

Would an adapter/port help?
________________

Is the abstraction worth its cost?
________________
```

---

# 113. Coupling Refactoring Workflow

```text
1. Draw dependency graph.
2. Identify high-risk edges.
3. Identify volatile dependencies.
4. Identify broad interfaces.
5. Identify shared mutable state.
6. Identify cycles.
7. Narrow capabilities.
8. Reverse dependency where appropriate.
9. Introduce ports/adapters when justified.
10. Re-test.
11. Re-check change propagation.
```

---

# 114. Extract a Port

Before:

```ts
class CheckoutUseCase {
  constructor(
    private readonly stripe: StripeClient
  ) {}
}
```

After:

```ts
interface PaymentGateway {
  charge(input: ChargeInput): Promise<ChargeResult>;
}
```

The port describes the stable capability.

---

# 115. Extract an Adapter

```ts
class StripePaymentGateway implements PaymentGateway {
  constructor(
    private readonly stripe: StripeClient
  ) {}

  async charge(input: ChargeInput) {
    const result = await this.stripe.charge({
      amount: input.amount,
      currency: input.currency
    });

    return mapStripeResult(result);
  }
}
```

Stripe coupling is localized.

---

# 116. Narrow an Interface

Before:

```ts
interface PaymentSystem {
  charge();
  refund();
  listCustomers();
  exportReports();
  sendNotifications();
}
```

Potentially:

```ts
interface PaymentCharger {
  charge(input: ChargeInput): Promise<ChargeResult>;
}
```

Consumers now depend on less.

---

# 117. Reverse a Dependency

Before:

```text
Domain → Infrastructure
```

Potentially:

```text
Application/Domain → Port
Infrastructure → Port implementation
```

This is dependency inversion.

---

# 118. Move Coordination Upward

If:

```text
A → B
B → A
```

because both coordinate a workflow, move sequencing upward:


```text
UseCase
 ├── A
 └── B
```

This can restore a directional graph.

---

# 119. Extract a Stable Concept

If two components depend on each other because they need one shared concept:

```text
A ↔ B
```

consider extracting:

```text
stable shared concept
```

But avoid a giant "common" package.

---

# 120. Break Global State

Before:

```ts
globalThis.tenantId = tenantId;
```

After:

```ts
type TenantContext = {
  tenantId: string;
};
```

Pass it explicitly or scope it through a carefully designed application context.

---

# 121. Remove Content Coupling

Before:

```ts
payment._state.status = "CAPTURED";
```

After:

```ts
payment.capture();
```

The caller no longer knows representation.

---

# 122. Remove Control Coupling

Before:

```ts
render(order, "PDF");
```

Potentially:

```ts
pdfRenderer.render(order);
```

The PDF capability owns its operation.

---

# 123. Remove Temporal Coupling

Before:

```ts
db.connect();
db.begin();
db.query();
db.commit();
```

Potentially:

```ts
unitOfWork.run(async () => {
  ...
});
```

The caller no longer manages transaction ordering manually.

---

# 124. Coupling and Parameter Objects

A parameter object:

```ts
pricing.calculate({
  price,
  quantity,
  currency
});
```

can improve readability.

But broad parameter objects can create stamp coupling.

Make input types correspond to the capability.

---

# 125. Coupling and Events

Events can reduce direct dependency, but create contract coupling:

```text
Producer
    ↓
Event contract
    ↓
Consumers
```

Version and validate events deliberately.

---

# 126. Coupling and Service Boundaries

Moving from in-process:

```text
A → B
```

to network:

```text
A → HTTP → B
```

may reduce source-code coupling while increasing:

```text
operational coupling
failure coupling
latency
contract coupling
```

Evaluate total cost.

---

# 127. Coupling and Microservice Myth

This is false:

```text
microservices = low coupling
```

A tightly synchronized set of services can be highly coupled.

A modular monolith can have cleaner boundaries.

---

# 128. Modular Monolith Coupling

A modular monolith can enforce:

```text
Ordering → Pricing
```

without permitting:

```text
Pricing → Ordering internals
```

Use package/module boundary rules.

---

# 129. Tooling to Enforce Dependency Direction

Possible tools:

```text
ESLint import rules
dependency-cruiser
Nx module boundaries
TypeScript project references
package dependency rules
architecture tests
```

Tools enforce a decision.

They do not decide what the correct architecture is.

---

# 130. Architecture Tests

Examples:

```text
domain must not import infrastructure
payments must not import ordering internals
reporting must not mutate domain objects
```

Executable architecture reduces dependency drift.

---

# 131. Build Coupling

High module coupling can increase:

```text
build graph size
incremental compilation
test startup
bundle work
```

Dependency boundaries can therefore improve both architecture and developer experience.

---

# 132. Bundle Coupling

Browser imports can pull unrelated code into bundles.

Cohesive packages and narrow exports can improve tree-shaking.

---

# 133. Runtime Start-Up Coupling

Large side-effect import graphs can slow application startup and make initialization order difficult.

Explicit composition reduces ambiguity.

---

# 134. Memory Coupling

Broad singletons and caches can retain large object graphs.

Avoid global references merely for convenience.

---

# 135. Security Coupling

A broad dependency can increase privilege:

```text
Database
```

versus:

```text
OrderRepository
```

Prefer the minimum capability.

---

# 136. Failure Domains

Each dependency edge can define a failure boundary.

```text
Checkout
    ↓
PaymentGateway
```

means payment failure can propagate into checkout.

Define:

```text
timeout
retry
fallback
compensation
```

where appropriate.

---

# 137. Coupling and Operational Complexity

Remote dependencies add:

```text
deployment
monitoring
on-call
networking
security
versioning
```

A source-level decoupling win may become an operations-level cost.

---

# 138. Coupling and Developer Experience

Too much coupling:

```text
One provider change breaks everything.
```

Too much indirection:

```text
One operation requires opening seven files.
```

Good architecture balances:

```text
change isolation
+
cognitive simplicity
```

---

# 139. Architecture Decision Record

For significant dependency boundaries, capture:

```text
Context
Decision
Alternatives
Benefits
Costs
Consequences
```

Example:

```text
Decision:
Payment provider isolated behind PaymentGateway.

Why:
provider replacement and provider-independent tests.

Cost:
port + adapter + wiring.
```

---

# 140. Coupling and Documentation

Document non-obvious boundaries.

Example:

```text
Order does not call the payment provider directly because
provider-specific changes must remain outside the domain model.
```

Good documentation prevents accidental architecture regression.

---

# 141. Coupling and Team Boundaries

Dependency direction affects teams.

If:

```text
Ordering
```

depends directly on:

```text
Payment implementation
```

then payment changes may require ordering changes.

Stable capability boundaries reduce cross-team synchronization.

---

# 142. Coupling and Ownership

A useful question:

> Who owns the change?

If one team owns:

```text
Payment provider integration
```

another team should ideally depend on a stable capability rather than the provider implementation.

---

# 143. Coupling and Releases

A highly coupled system may require synchronized releases.

Stable interfaces can allow:

```text
independent deployment
```

only when the runtime architecture actually supports it.

Do not assume an interface guarantees independent deployment.

---

# 144. Coupling and Compatibility

Compatibility may be:

```text
source compatibility
runtime compatibility
API compatibility
schema compatibility
event compatibility
```

The boundary type determines the relevant form.

---

# 145. Coupling and Versioning Strategy

For volatile external contracts:

```text
internal stable model
    ↓
versioned adapter
```

can localize change.

---

# 146. Coupling and Migration

An incremental migration can use:

```text
new capability
    ↓
legacy adapter
```

then gradually replace the old implementation.

---

# 147. Coupling and Strangler Refactoring

Safe sequence:

```text
identify boundary
    ↓
introduce capability
    ↓
wrap old implementation
    ↓
move callers
    ↓
verify behavior
    ↓
replace old implementation
    ↓
remove legacy dependency
```

---

# 148. Coupling and Branch-by-Abstraction

```text
StablePort
├── OldImplementation
└── NewImplementation
```

Consumers depend on the stable port while implementation changes.

---

# 149. Coupling and Backward Compatibility

A facade can preserve old callers:

```ts
class LegacyPaymentService {
  constructor(
    private readonly gateway: PaymentGateway
  ) {}

  charge(input: ChargeInput) {
    return this.gateway.charge(input);
  }
}
```

The internal dependency can evolve.

---

# 150. Coupling and Refactoring Safety

When reducing coupling:

```text
preserve behavior
preserve contracts
preserve invariants
preserve observable semantics
```

unless behavior change is intentional.

---

# 151. Coupling Audit Exercise

Inspect an existing project and record:

```text
top 10 dependencies
top 5 fan-out components
top 5 high fan-in components
all module cycles
all globals
all provider SDK imports
all ORM imports from domain code
all shared mutable state
```

Classify each dependency.

---

# 152. Dependency Graph Exercise

Given:

```text
OrderController
 → OrderService
 → Stripe
 → Prisma
 → Mailer

Order
 → Prisma

Invoice
 → Prisma
 → Mailer
```

Identify:

```text
framework coupling
provider coupling
persistence coupling
coordination coupling
```

Redraw a better graph.

---

# 153. Coupling Refactoring Exercise

Given:

```ts
class Order {
  constructor(
    private readonly db: Database,
    private readonly stripe: StripeClient,
    private readonly mailer: Mailer
  ) {}
}
```

Move toward:

```text
Order
    domain behavior

OrderRepository
    persistence

PaymentGateway
    payment

NotificationSender
    notification

UseCase
    coordination
```

---

# 154. Implementation Exercise — Ports

Implement:

```ts
interface PaymentGateway {
  charge(input: ChargeInput): Promise<ChargeResult>;
}

interface OrderRepository {
  findById(id: string): Promise<Order | null>;
  save(order: Order): Promise<void>;
}
```

Then implement:

```text
CheckoutUseCase
StripePaymentGateway
InMemoryOrderRepository
```

Prove the use case can be tested without the provider SDK.

---

# 155. Implementation Exercise — Narrow Capability

Replace:

```ts
SystemManager
```

with:

```ts
interface SalesReader {
  getSalesForPeriod(
    range: DateRange
  ): Promise<SaleSummary[]>;
}
```

Measure:

```text
methods exposed
authority granted
knowledge required
test setup
```

---

# 156. Implementation Exercise — Cycle Removal

Create:

```text
Order ↔ Payment
```

Then refactor:

```text
CheckoutUseCase
 ├── Order
 └── PaymentGateway
```

Explain why the cycle disappears.

---

# 157. Track A — Core Theory Retrieval

Without notes, explain:

```text
Why is zero coupling impossible?
What is data coupling?
What is stamp coupling?
What is control coupling?
What is common coupling?
What is content coupling?
What is temporal coupling?
What is external coupling?
What are fan-in and fan-out?
What are afferent and efferent coupling?
```

---

# 158. Track B — Implementation Retrieval

Given:

```text
Controller
 → Prisma
 → Stripe
 → Mailer
```

redraw it using:

```text
Controller
UseCase
Order
Repository
PaymentGateway
NotificationSender
```

Then justify every dependency edge.

---

# 159. Track C — Interview Retrieval

Question:

> What would you inspect first in a highly coupled codebase?

Strong answer:

> "I would draw the dependency graph, identify broad and volatile dependencies, inspect shared mutable state and cycles, and examine which dependencies have the highest change propagation, failure impact, or security authority. Then I would narrow capabilities and correct dependency direction incrementally."

---

# 160. Interview — Is Low Coupling Always Good?

Strong answer:

> "Not by itself. Extreme decoupling can create fragmentation, indirection and cognitive load. I want necessary dependencies to stay explicit and meaningful while accidental dependency is reduced."

---

# 161. Interview — Does Every Class Need an Interface?

Strong answer:

> "No. An abstraction should earn its cost by representing a meaningful capability, protecting credible variation, improving dependency direction, or establishing a useful boundary."

---

# 162. Interview — Shared Database

Strong answer:

> "A shared database is not inherently wrong. It becomes risky when many components treat the schema as a public coordination contract and schema changes create broad coupling. In a modular monolith it can be practical if ownership boundaries remain explicit."

---

# 163. Interview — Do Microservices Reduce Coupling?

Strong answer:

> "They can reduce some source-code and ownership coupling while adding network, contract, operational and consistency coupling. I evaluate the full dependency system rather than assuming physical distribution means decoupling."

---

# 164. Interview — High Fan-Out

Strong answer:

> "High fan-out is a signal, not automatically a problem. A use-case orchestrator may legitimately depend on several focused capabilities. I inspect whether those dependencies support one coherent responsibility and whether independent change reasons are overwhelming the component."

---

# 165. Interview — Dependency Inversion

Strong answer:

> "Higher-level policy should not be forced to depend directly on lower-level implementation details. Both can depend on a stable capability, with the implementation satisfying that capability."

---

# 166. Predict-the-Design

### Scenario 1

```text
Order directly imports Stripe SDK.
```

Likely issue:

```text
provider/external coupling
```

### Scenario 2

```text
A service has 18 constructor dependencies.
```

Review:

```text
fan-out
cohesion
orchestration
```

### Scenario 3

```text
A module reads global tenant state.
```

Likely issue:

```text
implicit common coupling
```

### Scenario 4

```text
Two modules import each other.
```

Likely issue:

```text
dependency cycle
```

### Scenario 5

```text
Controller mutates ORM entity fields directly.
```

Likely issue:

```text
transport + persistence + domain coupling
```

---

# 167. Code Review Exercise

Review:

```ts
class OrderController {
  constructor(
    private readonly prisma: PrismaClient,
    private readonly stripe: StripeClient,
    private readonly mailer: Mailer
  ) {}

  async finalize(req: Request) {
    const order = await this.prisma.order.findUnique(...);

    if (order.status === "PENDING") {
      order.status = "CONFIRMED";
    }

    await this.stripe.charge(...);
    await this.prisma.order.update(...);
    await this.mailer.send(...);

    return order;
  }
}
```

Identify:

```text
content coupling
external coupling
framework coupling
persistence coupling
responsibility overload
change propagation
```

Then redesign the dependency graph.

---

# 168. Debugging Coupling Problems

Symptoms:

```text
provider change breaks domain
    → external coupling

schema change breaks many modules
    → database coupling

global mutation changes unrelated behavior
    → common coupling

mutual imports
    → cyclic dependency

huge mocks
    → broad coupling

deep object navigation
    → structural coupling
```

Treat these as signals.

---

# 169. Debugging Procedure

```text
1. Reproduce the failure/change.
2. Draw the dependency edge.
3. Identify knowledge crossing.
4. Identify authority crossing.
5. Identify volatility.
6. Identify coupling category.
7. Narrow capability.
8. Correct dependency direction.
9. Add boundary only if justified.
10. Re-test.
11. Re-evaluate the full graph.
```

---

# 170. Mastery Exercise — System

Design:

```text
Ordering
Pricing
Inventory
Payments
Reporting
```

Constraints:

```text
payment provider may change
tax rules may change
reporting is read-heavy
inventory is authoritative
tenant isolation is mandatory
```

Produce:

```text
dependency graph
ports
adapters
public APIs
change boundaries
security boundaries
```

---

# 171. Mastery Exercise — Jewellery ERP

For:

```text
Tenant
Branch
Customer
Product
InventoryItem
Sale
SaleLine
Payment
Invoice
PricingPolicy
TaxPolicy
SaleRepository
PaymentGateway
AuditLogger
FinalizeSaleUseCase
```

produce:

```text
1. dependency graph
2. stable vs volatile components
3. high-risk edges
4. capability interfaces
5. adapters
6. repository boundaries
7. tenant/security boundaries
8. transaction boundaries
9. event alternatives
10. module ownership
```

---

# 172. Principal-Level Review

Evaluate important dependencies using:

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
Is the dependency necessary?
Is it explicit?
Is it narrow?
Is direction correct?
Is it stable?
What failure does it import?
What authority does it grant?
What changes if it changes?
```

---

# 173. Dependency Decision Algorithm

```text
1. Identify needed capability.
2. Identify actual knowledge required.
3. Remove unnecessary knowledge.
4. Identify authority.
5. Identify volatility.
6. Prefer direct dependency when stable and cheap.
7. Introduce a port when variation/boundary justifies it.
8. Use an adapter for translation/provider isolation.
9. Keep orchestration at the workflow boundary.
10. Validate the resulting graph.
```

---

# 174. Coupling Risk Pyramid

```text
Lower risk
    pure local function

    local object collaboration

    stable module capability

    stable library dependency

    framework dependency

    shared schema contract

    volatile provider SDK everywhere

    global mutable state
Higher risk
```

This is heuristic.

The riskiest dependencies are typically:

```text
implicit
broad
volatile
high-authority
failure-prone
```

---

# 175. Coupling and Change Amplification

Measure:

```text
one conceptual change
    ↓
number of components touched
```

If one provider change touches ten modules, review the boundary.

---

# 176. Coupling and Blast Radius

A broad dependency can increase failure blast radius.

```text
Provider
    ↓
20 modules
```

localization can reduce impact.

---

# 177. Coupling and Failure Domains

For:

```text
Service A → Service B
```

consider:

```text
timeout
retry
circuit
fallback
compensation
monitoring
```

Dependency reduction and failure isolation are related but separate goals.

---

# 178. Coupling and Distributed Transactions

Splitting one local transaction across services can replace:

```text
local atomicity
```

with:

```text
eventual consistency
sagas
compensation
reconciliation
```

Therefore reducing code coupling may increase consistency complexity.

---

# 179. Coupling and Caching Consistency

A cache boundary may reduce database load but introduces:

```text
freshness assumptions
invalidation coupling
consistency complexity
```

Include those costs in architecture decisions.

---

# 180. Coupling and Read Models

Read models can decouple queries from write-side structures.

But they introduce:

```text
projection lifecycle
event delivery
staleness
rebuild complexity
```

---

# 181. Coupling and Event Contracts

An event contract is a dependency.

Versioning should consider:

```text
backward compatibility
consumer assumptions
schema evolution
ordering
delivery
```

---

# 182. Coupling and Operational Ownership

Ask:

```text
Who owns retries?
Who owns reconciliation?
Who owns cache invalidation?
Who owns schema migration?
Who owns audit?
```

Operational responsibilities also create dependency boundaries.

---

# 183. Coupling and Team Coordination

Every cross-team dependency is a potential coordination requirement.

Stable interfaces can reduce:

```text
synchronized implementation changes
```

but cannot eliminate business coordination.

---

# 184. Coupling and Release Coordination

If several modules must release together, the dependency may be stronger than it appears.

Stable backward-compatible contracts can loosen release coupling.

---

# 185. Coupling and Build Graphs

Dependency graphs affect:

```text
compile times
test startup
incremental builds
bundle generation
```

---

# 186. Coupling and Bundle Size

A broad import graph may make front-end bundles larger.

Cohesive modules and narrow exports can improve tree-shaking.

---

# 187. Coupling and Startup

Side-effect-heavy modules can increase startup cost and create lifecycle coupling.

Explicit composition reduces ambiguity.

---

# 188. Coupling and Memory Retention

Singletons and shared caches can keep dependency graphs alive longer than expected.

Review lifecycle and ownership.

---

# 189. Coupling and Observability

Dependency boundaries should be visible in telemetry:

```text
checkout
inventory.reserve
payment.charge
repository.save
```

This makes failure localization easier.

---

# 190. Coupling and Audit

Audit boundaries can be explicit:

```ts
interface AuditLogger {
  record(entry: AuditEntry): Promise<void>;
}
```

The domain/application layer depends on the capability, not storage details.

---

# 191. Coupling and Security Audit

For each dependency:

```text
Does it read secrets?
Can it write data?
Can it cross tenant boundaries?
Can it call external systems?
```

Narrow capabilities reduce privilege.

---

# 192. Coupling and Compliance

Compliance boundaries may require:

```text
audit
approval
tenant isolation
retention
segregation of duties
```

Keep these responsibilities explicit.

---

# 193. Coupling and Reconciliation

Payment reconciliation is a separate responsibility:

```text
Provider records
    vs
internal records
```

Do not make the order entity own operational reconciliation.

---

# 194. Coupling and Background Jobs

A job should invoke a cohesive capability:

```text
ExpireReservationsJob
    ↓
ExpireReservationsUseCase
```

rather than containing all business logic.

---

# 195. Coupling and Schedulers

Scheduler responsibility:

```text
when
```

Task responsibility:

```text
what
```

This prevents scheduling infrastructure from becoming business logic.

---

# 196. Coupling and Workflow Engines

A workflow engine owns:

```text
sequence
recovery
workflow state
```

It does not automatically own every domain rule.

---

# 197. Coupling and Domain Events

A domain event can reduce direct dependency between producers and consumers.

But:

```text
event schema
```

becomes a new contract.

Use that trade-off deliberately.

---

# 198. Coupling and Domain Boundaries

A domain object should not become coupled to:

```text
transport
database
provider SDK
monitoring vendor
```

unless the domain genuinely requires the capability.

---

# 199. Coupling and API Design

Good APIs minimize:

```text
representation knowledge
```

and maximize:

```text
semantic capability
```

Example:

```ts
inventory.reserve(request);
```

rather than:

```ts
inventory.available -= request.quantity;
```

---

# 200. Coupling and Abstraction Quality

A bad abstraction can be more coupled than a direct dependency.

If:

```text
PaymentInterface
```

exposes every provider-specific concept, the boundary is fake.

A good abstraction represents internal needs, not external SDK structure.

---

# 201. Coupling and Principal Judgment

For every abstraction, ask:

```text
What change does this protect?
What dependency does it remove?
What knowledge does it hide?
What authority does it narrow?
What complexity does it add?
```

If the answers are weak, direct code may be better.

---

# 202. Design Review Example

Bad:

```text
Order
 ├── StripeClient
 ├── PrismaClient
 ├── Mailer
 ├── Redis
 └── Logger
```

Better:

```text
FinalizeOrderUseCase
 ├── OrderRepository
 ├── PaymentGateway
 ├── NotificationSender
 └── AuditLogger

Order
 └── domain behavior
```

Now technical dependencies are moved toward focused boundaries.

---

# 203. Design Review — Trade-Off

The second design introduces:

```text
more abstractions
more wiring
more types
```

But can provide:

```text
provider isolation
testability
least authority
clear responsibilities
lower change propagation
```

Judge it in context.

---

# 204. Final Mental Model

When reviewing a dependency, ask:

```text
What capability do I need?
        ↓
What knowledge do I actually need?
        ↓
What authority do I need?
        ↓
Can the capability be narrower?
        ↓
How stable is the dependency?
        ↓
What changes if it changes?
        ↓
What failures cross the boundary?
        ↓
Should I depend directly?
        ↓
Should I use a port?
        ↓
Should I use an adapter?
        ↓
What is the runtime cost?
        ↓
What is the operational cost?
        ↓
What is the simplest safe graph?
```

---

# 205. Final Design Principle

> **Keep necessary dependencies explicit and meaningful; minimize unnecessary knowledge, authority, volatility, and change propagation across boundaries.**

Coupling is not the enemy.

**Accidental coupling is.**

---

# 206. Completion Criteria

```text
[ ] I can define coupling.
[ ] I can distinguish necessary and accidental coupling.
[ ] I can explain data coupling.
[ ] I can explain stamp coupling.
[ ] I can explain control coupling.
[ ] I can explain common/global coupling.
[ ] I can explain content coupling.
[ ] I can explain temporal coupling.
[ ] I can explain external coupling.
[ ] I understand fan-in and fan-out.
[ ] I understand afferent and efferent coupling.
[ ] I can reason about dependency direction.
[ ] I can apply dependency inversion.
[ ] I can avoid interface-everywhere thinking.
[ ] I can identify dependency cycles.
[ ] I can reason about shared mutable state.
[ ] I can reduce provider coupling.
[ ] I can reduce ORM/database coupling.
[ ] I can design narrow capabilities.
[ ] I can use ports and adapters appropriately.
[ ] I can analyze distributed coupling.
[ ] I can apply coupling analysis to security and multi-tenancy.
[ ] I can evaluate performance and memory trade-offs.
[ ] I can draw and review dependency graphs.
[ ] I can refactor highly coupled components.
[ ] I can apply coupling analysis to jewellery ERP.
[ ] I can defend dependency decisions in an interview.
```

---

# 207. Revision / Retrieval Record

```text
Date:
________________

Highest-risk dependency:
________________

Coupling category:
________________

Current direction:
________________

Desired direction:
________________

Abstraction considered:
________________

Benefit:
________________

Cost:
________________

Final decision:
________________
```

---

# 208. Track A — Retrieval Record

```text
Date:
________________

Weakest concept:
________________

Most useful coupling category:
________________

Dependency cycle found:
________________

Volatile dependency found:
________________

Security implication:
________________
```

---

# 209. Canonical References and Source Discipline

### ECMAScript

Use the ECMAScript specification for language-level claims:

https://tc39.es/ecma262/

### TypeScript

Use official TypeScript documentation:

https://www.typescriptlang.org/docs/

### Object-Oriented Design

Use established object-oriented design literature for coupling, cohesion, responsibility assignment, modularity, and dependency design.

### Architecture

For:

```text
dependency inversion
ports and adapters
bounded contexts
application services
repositories
distributed systems
service boundaries
```

treat the discussion as architectural guidance rather than JavaScript language semantics.

### Runtime

For:

```text
V8 optimization
module loading
allocation
bundling
startup behavior
```

identify implementation-specific or tooling-specific claims clearly.

---

# 210. Source Discipline Rules

```text
ECMAScript
    → language semantics

TypeScript
    → type/compiler semantics

Node/browser documentation
    → host/runtime behavior

Architecture literature
    → architecture guidance

Project requirements
    → project-specific constraints
```

Do not use a framework convention as proof of correct dependency direction.

---

# 211. Concept Connections

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
SOLID
    ↓
Dependency Inversion
    ↓
Design Patterns
    ↓
DDD / Architecture
    ↓
Persistence
    ↓
Transactions
    ↓
Concurrency
    ↓
Resilience
    ↓
Distributed Systems
    ↓
Enterprise LLD
```

Core pair:

```text
Cohesion
    → What belongs together?

Coupling
    → What should depend on what?
```

---

# 212. Chapter Completion Snapshot

```text
Chapter: 16
Title: Coupling-First Dependency Design

Theory:
[+] Coupling
[+] Data coupling
[+] Stamp coupling
[+] Control coupling
[+] Common/global coupling
[+] Content coupling
[+] Temporal coupling
[+] External coupling
[+] Fan-in
[+] Fan-out
[+] Afferent coupling
[+] Efferent coupling
[+] Dependency direction
[+] Dependency inversion

Design:
[+] Capability boundaries
[+] Ports and adapters
[+] Dependency graphs
[+] Cycle analysis
[+] Shared-state analysis
[+] Change propagation
[+] Abstraction budget
[+] Architecture decision records
[+] Incremental refactoring

JavaScript / TypeScript:
[+] Modules
[+] Imports/exports
[+] Barrel files
[+] Side effects
[+] Globals
[+] process.env
[+] Interfaces
[+] Structural typing
[+] Dependency injection
[+] Composition root

Production:
[+] Provider coupling
[+] ORM/database coupling
[+] API coupling
[+] Event coupling
[+] Distributed coupling
[+] Modular monolith
[+] Microservice trade-offs
[+] Multi-tenancy
[+] Security
[+] Reliability
[+] Observability
[+] Transactions
[+] Concurrency
[+] Performance
[+] Memory
[+] Operations

Interview:
[+] Question bank
[+] Predict-the-design
[+] Code review
[+] Dependency graph exercises
[+] Jewellery ERP coupling audit
[+] Principal-level decision framework
```

# Chapter 16 — Completion Statement

Coupling-first dependency design teaches you to treat every dependency as a deliberate design decision. The objective is not an architecture with no arrows. The objective is a dependency graph in which arrows represent necessary capabilities, point toward stable concepts, expose little unnecessary knowledge or authority, and localize credible change and failure.

Next chapter: **Design Smells, Responsibility Drift & Refactoring**.
