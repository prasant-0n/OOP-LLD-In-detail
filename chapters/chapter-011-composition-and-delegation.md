# Chapter 11 — Composition & Delegation

> **JavaScript OOP + LLD Mastery**
>
> Composition builds behavior by combining objects that own focused responsibilities. Delegation lets one object ask another object to perform work instead of inheriting that work through a class hierarchy.
>
> The central question is: **Who should own this responsibility, and who should collaborate with that owner?**

**Status:** `[ ] Not Started`

# 1. Learning Objectives

By the end of this chapter, you should be able to:

```text
[ ] define composition
[ ] define delegation
[ ] distinguish has-a from is-a
[ ] identify collaborators
[ ] explain dependency injection
[ ] distinguish dependency use from dependency ownership
[ ] reason about collaborator lifecycle
[ ] reason about shared mutable collaborators
[ ] explain collaboration direction
[ ] explain strategy composition
[ ] explain policy objects
[ ] explain adapter composition
[ ] explain decorator composition
[ ] explain facade-style composition
[ ] identify circular dependencies
[ ] identify god orchestrators
[ ] identify over-composition
[ ] compare composition with inheritance
[ ] design explicit dependency boundaries
[ ] implement compositional objects
[ ] debug delegation failures
[ ] connect composition to cohesion, coupling, SOLID and GRASP
```

# 2. Prerequisites

```text
Chapter 1 — JavaScript Object Model — Foundation
Chapter 2 — Object Identity, Equality & Mutability
Chapter 3 — Prototype Chain Deep Dive
Chapter 4 — Constructor Functions & Instance Construction
Chapter 5 — JavaScript Classes Internally
Chapter 6 — Class Fields & Initialization Semantics
Chapter 7 — Private State & Encapsulation
Chapter 8 — Abstraction & Stable Object Interfaces
Chapter 9 — Inheritance & Subtyping
Chapter 10 — Polymorphism & Dynamic Dispatch
```

# 3. What Is It?

Composition means building an object from other objects that provide parts of its behavior or state.

```js
class CheckoutService {
  constructor(paymentGateway, inventory, notifier) {
    this.paymentGateway = paymentGateway;
    this.inventory = inventory;
    this.notifier = notifier;
  }
}
```

Conceptually:

```text
CheckoutService
├── PaymentGateway
├── Inventory
└── Notifier
```

Delegation means one object asks another object to perform a responsibility:

```js
class CheckoutService {
  constructor(paymentGateway) {
    this.paymentGateway = paymentGateway;
  }

  pay(request) {
    return this.paymentGateway.charge(request);
  }
}
```

# 4. Why Does It Exist?

Inheritance can create:

```text
prototype coupling
constructor coupling
override hazards
fragile base classes
```

Composition makes many relationships explicit:

```text
Checkout
├── PaymentGateway
├── PricingPolicy
└── NotificationSender
```

Each collaborator can evolve independently when its contract is stable.

# 5. Mental Model

```text
             COMPOSITE OBJECT
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
      Gateway     Policy    Repository
          │         │         │
          ▼         ▼         ▼
      payment    decision  persistence
```

For every dependency ask:

```text
Who owns it?
Who creates it?
Who may replace it?
Who may mutate it?
How long does it live?
What contract does the composite require?
```

# 6. Core Rules

## Rule 1 — Composition Models Collaboration

Use composition for:

```text
has-a
uses-a
depends-on
delegates-to
contains
```

## Rule 2 — Responsibilities Stay With Their Owners

If a repository owns persistence behavior, a service should delegate persistence rather than duplicate driver-specific logic.

## Rule 3 — Dependencies Should Be Explicit

Prefer:

```js
constructor(repository)
```

over hidden global access or constructing infrastructure deep inside the object.

## Rule 4 — Dependency Injection Is a Composition Technique

External construction of collaborators separates:

```text
object behavior
from
dependency provisioning.
```

## Rule 5 — Composition Does Not Automatically Mean Low Coupling

A class with twenty concrete collaborators can still be highly coupled.

## Rule 6 — Delegation Preserves Contracts

A collaborator must satisfy the capability contract that the composite expects.

# 7. Syntax

## Constructor Injection

```js
class OrderService {
  constructor(repository) {
    this.repository = repository;
  }
}
```

## Delegation

```js
save(order) {
  return this.repository.save(order);
}
```

## Strategy Composition

```js
class PriceCalculator {
  constructor(strategy) {
    this.strategy = strategy;
  }

  calculate(order) {
    return this.strategy.calculate(order);
  }
}
```

# 8. Basic Examples

## Example 1 — Payment Delegation

```js
const payment = {
  charge(amount) {
    return `charged:${amount}`;
  },
};

class Checkout {
  constructor(payment) {
    this.payment = payment;
  }

  pay(amount) {
    return this.payment.charge(amount);
  }
}

console.log(new Checkout(payment).pay(100));
```

**Prediction**

```text
charged:100
```

**Trace**

```text
Checkout.pay
→ payment.charge
→ collaborator implementation
```

## Example 2 — Replaceable Policy

```js
const standard = {
  calculate(order) {
    return order.total;
  },
};

const discount = {
  calculate(order) {
    return order.total * 0.9;
  },
};
```

Both satisfy the same policy capability.

## Example 3 — Coordinating Collaborators

```js
class Checkout {
  constructor(payment, notifier) {
    this.payment = payment;
    this.notifier = notifier;
  }

  async complete(order) {
    await this.payment.charge(order.total);
    await this.notifier.send("Payment completed");
  }
}
```

Checkout coordinates; it does not own payment transport or notification transport.

# 9. Execution Walkthrough

```js
class Checkout {
  constructor(payment, notifier) {
    this.payment = payment;
    this.notifier = notifier;
  }

  async complete(order) {
    await this.payment.charge({ amount: order.total });
    await this.notifier.send("Payment completed");
  }
}
```

Flow:

```text
new Checkout(payment, notifier)
        ↓
store collaborator references
        ↓
complete(order)
        ↓
delegate charge
        ↓
payment implementation
        ↓
delegate notification
        ↓
notifier implementation
```

# 10. Internal Mechanics

Composition is an application-level design technique built from ordinary JavaScript mechanisms:

```text
objects
references
functions
classes
closures
modules
```

A delegation call is usually ordinary property lookup followed by function invocation:

```js
this.collaborator.method(...args);
```

There is no special ECMAScript runtime primitive called “composition”.

# 11. ECMAScript / Specification Semantics

ECMAScript specifies:

```text
objects
references
property access
calls
constructors
functions
```

It does not define composition or dependency injection as OOP keywords.

Therefore distinguish:

```text
language semantics
```
from:

```text
design technique.
```

# 12. Advanced Behavior

## 12.1 Composition vs Inheritance

Inheritance:

```text
Child
 ↓
Parent prototype
```

Composition:

```text
Object
├── collaborator A
├── collaborator B
└── collaborator C
```

Composition exposes relationships as dependencies instead of prototype ancestry.

## 12.2 Dependency Injection

```js
class OrderService {
  constructor(repository) {
    this.repository = repository;
  }
}
```

The service does not decide how the repository is provisioned.

## 12.3 Runtime Replacement

Collaborators are values and can technically be replaced. Whether replacement is allowed should be an explicit lifecycle/encapsulation decision.

## 12.4 Delegation vs Reimplementation

Prefer:

```js
return this.repository.save(order);
```

over duplicating persistence logic in the caller.

# 13. Dependency Ownership

Dependency usage and ownership are different.

Ask:

```text
Who created the dependency?
Who is responsible for its lifecycle?
Can multiple objects share it?
Is it request-scoped?
Is it transaction-scoped?
Is it application-scoped?
```

Example:

```text
DatabasePool → long-lived
Transaction  → short-lived
OrderService → may use both
```

The composite must not accidentally retain a short-lived dependency for the wrong lifetime.

# 14. Lifecycle Composition

Useful lifecycle categories:

```text
application
request
transaction
operation
object
```

A composed graph should respect these boundaries.

A request-scoped object stored inside a long-lived singleton is a warning sign unless the design intentionally manages that lifetime.

# 15. Shared Collaborators

Intentional sharing:

```text
Service A ──┐
            ├──→ Clock
Service B ──┘
```

This can be good for stateless or carefully controlled components.

Shared mutable collaborators create aliasing:

```text
Service A ──┐
            ├──→ mutable object
Service B ──┘
```

Now one service can affect what another observes.

# 16. Collaboration Direction

Prefer clear direction:

```text
higher-level policy
      ↓
capability
      ↓
implementation
```

Be cautious with:

```text
A → B
B → A
```

because cycles complicate construction, testing, and change propagation.

# 17. Circular Collaboration

```js
class A {
  constructor(b) {
    this.b = b;
  }
}

class B {
  constructor(a) {
    this.a = a;
  }
}
```

A cycle can indicate:

```text
unclear ownership
missing mediator
third abstraction needed
incorrect dependency direction
```

Cycles are not automatically wrong, but they deserve explicit justification.

# 18. Strategy Composition

Strategy isolates a replaceable algorithm or policy:

```js
class Sorter {
  constructor(strategy) {
    this.strategy = strategy;
  }

  sort(items) {
    return this.strategy.sort(items);
  }
}
```

Use strategy when variation is genuinely independent and likely to change.

# 19. Policy Objects

Policies encapsulate decision rules:

```text
discount policy
tax policy
authorization policy
retry policy
pricing policy
```

Example:

```js
class Checkout {
  constructor(taxPolicy) {
    this.taxPolicy = taxPolicy;
  }

  calculateTax(order) {
    return this.taxPolicy.calculate(order);
  }
}
```

# 20. Adapter Composition

```text
Application
   ↓
PaymentGateway capability
   ↓
Adapter
   ↓
Provider SDK
```

The adapter owns translation between external and internal models.

# 21. Decorator Composition

```js
class LoggingRepository {
  constructor(repository, logger) {
    this.repository = repository;
    this.logger = logger;
  }

  async save(item) {
    this.logger.info("saving");
    return this.repository.save(item);
  }
}
```

Graph:

```text
LoggingRepository
      ↓
   Repository
```

The wrapper adds behavior without modifying the wrapped implementation.

# 22. Facade Composition

A facade coordinates several subsystem capabilities:

```js
class CheckoutFacade {
  constructor(payment, inventory, shipping) {
    this.payment = payment;
    this.inventory = inventory;
    this.shipping = shipping;
  }

  async checkout(order) {
    // coordinate subsystems
  }
}
```

A facade is useful when the client should not understand every subsystem.

# 23. Composite Object vs God Orchestrator

Composition becomes harmful when one object knows every subsystem:

```text
payment
inventory
pricing
tax
fraud
shipping
notifications
audit
analytics
```

A god orchestrator becomes a change hotspot and coupling hub.

Split coordination where there are natural boundaries.

# 24. Cohesion in Composition

A composite should have a coherent reason to coordinate its collaborators.

Good:

```text
Checkout
→ coordinates checkout completion
```

Suspicious:

```text
Checkout
→ also manages users, migrations, catalog indexing, reporting
```

# 25. Coupling in Composition

Bad:

```js
service.repository.client.connection.pool.query(...);
```

Better:

```js
service.repository.save(order);
```

The second depends on the repository capability rather than internal representation.

# 26. Stable Collaboration Interfaces

Prefer collaborators such as:

```text
PaymentGateway
OrderRepository
Notifier
Clock
IdGenerator
EventPublisher
```

The composite should depend on stable capabilities rather than concrete infrastructure details whenever meaningful variation exists.

# 27. Advanced Behavior — Dependency Lifetime

A dependency's lifetime should not exceed or conflict with the lifetime contract of the composite without an intentional ownership model.

For each collaborator record:

```text
scope
owner
cleanup
sharing policy
request/concurrency assumptions
```

# 28. Advanced Behavior — Immutable Collaborator References

In TypeScript:

```ts
class OrderService {
  constructor(
    private readonly repository: OrderRepository,
  ) {}
}
```

`readonly` expresses that the property should not be rebound through normal TypeScript checking. It does not mean the collaborator itself is deeply immutable.

# 29. Advanced Behavior — Composition as a Graph

Large systems are collaboration graphs:

```text
A → B
A → C
B → D
C → D
```

Watch for:

```text
high fan-in
high fan-out
cycles
hub classes
god objects
hidden globals
```

# 30. Edge Cases

```text
shared mutable collaborator
cyclic dependency
optional/null collaborator
runtime replacement
deep delegation chain
pass-through facade with no semantic value
hidden global dependency
short-lived dependency retained by long-lived object
```

# 31. Common Misconceptions

```text
"composition means zero coupling."
"dependency injection automatically creates good architecture."
"more collaborators means more modularity."
"delegation means the delegator has no responsibility."
"every behavior should become a strategy object."
"all shared collaborators should be singletons."
"composition and dependency injection are identical."
"bidirectional collaboration is always wrong."
"facades should expose every subsystem operation."
```

# 32. Common Mistakes

```text
[ ] constructing infrastructure inside domain objects unnecessarily
[ ] exposing collaborators publicly without a contract reason
[ ] passing concrete infrastructure types everywhere
[ ] creating tiny objects with no semantic boundary
[ ] creating god orchestrators
[ ] creating cycles without justification
[ ] sharing mutable collaborators accidentally
[ ] ignoring collaborator lifecycle
[ ] making every variation a strategy class
[ ] delegating without a clear capability contract
```

# 33. Comparison With Related Concepts

| Concept | Core idea |
|---|---|
| Composition | Build behavior from collaborators |
| Delegation | Ask another object to perform work |
| Inheritance | Specialize/reuse through hierarchy |
| Strategy | Inject replaceable behavior/policy |
| Policy | Encapsulate a decision rule |
| Adapter | Translate one interface into another |
| Decorator | Wrap and extend behavior |
| Facade | Simplify subsystem coordination |
| Dependency Injection | Provide collaborators externally |
| Aggregation | Whole-part relation with independent lifetime |
| Ownership | Lifecycle/control responsibility |

# 34. Performance Considerations

Composition can add:

```text
references
wrapper objects
method-call indirection
mapping
validation
```

Usually the important cost is not one extra function call but the total behavior around it.

For hot in-memory paths, measure.

Do not assume:

```text
composition = slow
inheritance = fast
```

# 35. Memory Considerations

A composite retains references to its collaborators.

A long-lived composite can therefore keep alive:

```text
caches
connections
large graphs
closures
buffers
```

Trace object lifetime:

```text
root
 ↓
composite
 ↓
collaborator
 ↓
resource graph
```

# 36. Security Considerations

Composition is also a capability-design mechanism.

Prefer:

```text
ReportReader
```

over:

```text
unrestricted database connection
```

Give an object the minimum capability required for its responsibility.

This supports least-authority design.

# 37. Production Usage

A production composition boundary should make these explicit:

```text
dependency contract
ownership
lifecycle
mutation policy
failure behavior
observability expectations
```

Before adding a collaborator, ask:

```text
Why does this object need it?
What responsibility does it provide?
Who owns it?
How long does it live?
Can the object function without it?
```

# 38. Implementation From Scratch

## Exercise 1 — Checkout Composition

Build:

```text
Checkout
PaymentGateway
Inventory
Notifier
```

Requirements:

```text
Checkout coordinates
Each collaborator owns one responsibility
Dependencies are injected
```

## Exercise 2 — Pricing Strategies

Implement:

```text
NoDiscount
PercentageDiscount
FlatDiscount
```

and inject them into `PricingEngine`.

## Exercise 3 — Repository Decorators

Compose:

```text
LoggingRepository
CachingRepository
Repository
```

## Exercise 4 — Payment Adapter

Build a provider adapter so application code knows only the internal payment contract.

## Exercise 5 — Dependency Graph

Draw:

```text
OrderService
PricingPolicy
OrderRepository
Clock
IdGenerator
EventPublisher
```

Mark:

```text
ownership
lifetime
direction
```

# 39. Debugging Exercises

## Debug 1 — Concrete Dependency

```js
class OrderService {
  constructor(prisma) {
    this.prisma = prisma;
  }

  save(order) {
    return this.prisma.order.create({ data: order });
  }
}
```

Identify the technology leak and redesign the dependency boundary.

## Debug 2 — Hidden Construction

```js
class EmailService {
  constructor() {
    this.client = new SomeEmailSdk();
  }
}
```

Explain the testing and replacement consequences.

## Debug 3 — Mutable Collaborator Leak

```js
class Service {
  constructor(config) {
    this.config = config;
  }

  getConfig() {
    return this.config;
  }
}
```

Decide whether the reference should escape.

## Debug 4 — Cyclic Dependency

```js
class A {
  constructor(b) {
    this.b = b;
  }
}

class B {
  constructor(a) {
    this.a = a;
  }
}
```

Consider:

```text
third abstraction
mediator/event
dependency inversion
factory-managed lifecycle
```

# 40. Code Review Exercise

Review:

```js
class CheckoutService {
  constructor(
    database,
    redis,
    stripe,
    sendgrid,
    logger,
    metrics,
    featureFlags,
    audit,
    shipping,
    tax,
    fraud,
  ) {}
}
```

Evaluate:

```text
1. Is this one cohesive responsibility?
2. Which dependencies are infrastructure concerns?
3. Which capabilities can be narrowed?
4. Are any lifetimes mixed?
5. Is there a hidden god orchestrator?
6. Which policies should become separate collaborators?
```

# 41. Interview Questions

```text
1. What is composition?
2. What is delegation?
3. How does composition differ from inheritance?
4. What is dependency injection?
5. What is dependency ownership?
6. Why should dependencies be explicit?
7. How can a composition design still be tightly coupled?
8. What is a strategy object?
9. What is a policy object?
10. What is an adapter?
11. What is a decorator?
12. What is a facade?
13. What is collaborator lifecycle?
14. Why are circular dependencies risky?
15. What is a god orchestrator?
16. How does composition affect cohesion and coupling?
17. When is inheritance preferable?
18. How does composition support dependency inversion?
19. How can composition implement least authority?
20. How would you design a compositional LLD solution?
```

# 42. Predict-the-Output Exercises

## Exercise A

```js
const payment = {
  charge(amount) {
    return `charged:${amount}`;
  },
};

class Checkout {
  constructor(payment) {
    this.payment = payment;
  }

  pay(amount) {
    return this.payment.charge(amount);
  }
}

console.log(new Checkout(payment).pay(100));
```

## Exercise B

```js
const first = {
  calculate(order) {
    return order.total;
  },
};

const second = {
  calculate(order) {
    return order.total * 0.9;
  },
};

class Pricing {
  constructor(strategy) {
    this.strategy = strategy;
  }

  total(order) {
    return this.strategy.calculate(order);
  }
}

const order = { total: 100 };

console.log(new Pricing(first).total(order));
console.log(new Pricing(second).total(order));
```

## Exercise C

```js
class Decorator {
  constructor(inner) {
    this.inner = inner;
  }

  run() {
    return `before-${this.inner.run()}-after`;
  }
}

const inner = {
  run() {
    return "work";
  },
};

console.log(new Decorator(inner).run());
```

## Exercise D

```js
const shared = { value: 1 };

class A {
  constructor(shared) {
    this.shared = shared;
  }
}

const a = new A(shared);
shared.value = 2;

console.log(a.shared.value);
```

# 43. Mastery Exercises

## Level 1 — Understand

Explain:

```text
composition
delegation
collaborator
dependency
ownership
lifecycle
```

## Level 2 — Explain

Compare composition and inheritance using:

```text
coupling
lifecycle
replacement
state ownership
testability
```

## Level 3 — Predict

Trace:

```text
delegation
shared collaborators
strategies
decorators
adapters
```

## Level 4 — Implement

Build:

```text
Checkout
PricingEngine
Repository decorator
Payment adapter
```

## Level 5 — Debug

Find:

```text
hidden dependency
cyclic dependency
mutable collaborator leak
god orchestrator
```

## Level 6 — Apply

Design a `JewelleryOrderService` using:

```text
OrderRepository
PricingPolicy
InventoryService
Clock
IdGenerator
EventPublisher
```

Keep domain rules close to their owners.

## Level 7 — Compare

Compare:

```text
inheritance
composition
delegation
strategy
decorator
adapter
facade
```

## Level 8 — Defend

Answer:

> Why is composition often a better default than inheritance for application-level LLD?

Connect:

```text
changeability
explicit dependencies
ownership
lifecycle
testing
substitutability
coupling.
```

# 44. Key Takeaways

```text
1. Composition builds objects from collaborators.
2. Delegation keeps a responsibility with another object.
3. Composition naturally models has-a/uses-a relationships.
4. Dependency injection makes composition explicit.
5. Composition does not eliminate coupling; contracts control coupling.
6. Collaborator ownership and lifetime must be deliberate.
7. Shared mutable collaborators introduce aliasing risk.
8. Clear collaboration direction improves reasoning.
9. Strategy objects isolate replaceable algorithms/policies.
10. Adapters translate external implementations.
11. Decorators add behavior through wrapping.
12. Facades coordinate subsystems for clients.
13. Too many collaborators can indicate a god orchestrator.
14. Cyclic composition deserves explicit justification.
15. Stable capability contracts make composition more useful.
16. Composition supports dependency inversion and least authority.
17. Inheritance remains useful for genuine subtype relationships.
18. Good composition creates explicit, testable collaboration boundaries.
```

# 45. Concept Connections

## Depends On

```text
Chapter 1 — JavaScript Object Model — Foundation
Chapter 2 — Object Identity, Equality & Mutability
Chapter 3 — Prototype Chain Deep Dive
Chapter 4 — Constructor Functions & Instance Construction
Chapter 5 — JavaScript Classes Internally
Chapter 6 — Class Fields & Initialization Semantics
Chapter 7 — Private State & Encapsulation
Chapter 8 — Abstraction & Stable Object Interfaces
Chapter 9 — Inheritance & Subtyping
Chapter 10 — Polymorphism & Dynamic Dispatch
```

## Builds Toward

```text
Chapter 12 — Cohesion & Coupling
Chapter 13 — Object Responsibilities
Chapter 14 — Dependency Direction
GRASP
SOLID
Strategy Pattern
Decorator Pattern
Adapter Pattern
Facade Pattern
Mediator
Factory
Builder
Dependency Injection
DDD services
application services
aggregate collaboration
```

## Related Concepts

```text
delegation
dependency injection
strategy
policy
adapter
decorator
facade
aggregation
ownership
capabilities
lifecycle
```

## Why This Chapter Matters

Inheritance explains specialization.

Composition explains collaboration.

Real application LLD is dominated by collaboration graphs, so composition is a core design skill rather than an alternative afterthought.

# 46. Completion Criteria

Mark:

```text
[+] Completed
```

when you can:

```text
define composition
define delegation
inject dependencies
design collaborator contracts
reason about lifecycle
compare composition with inheritance
```

Mark:

```text
[*] Mastered
```

when you can identify:

```text
responsibility owner
required collaborators
capability contracts
dependency lifetime
collaboration direction
shared-state risks
god-orchestrator risks
```

without relying on composition as a style preference.

Reading alone does not mark mastery.

# 47. Revision / Retrieval Record

```md
# Chapter 11 — Revision / Retrieval Record

## Attempt
- Date:
- Duration:
- Status before:
- Status after:

## Composition
- What does the composite own?
- What is delegated?

## Collaborators
- Collaborator:
- Responsibility:
- Contract:
- Lifetime:
- Owner:

## Coupling
- Where is the design coupled?
- Which dependencies are concrete?

## Lifecycle
- Application:
- Request:
- Transaction:
- Operation:

## Smells
- God orchestrator:
- Cycle:
- Hidden dependency:
- Over-composition:

## Alternatives
- Inheritance:
- Composition:
- Strategy:
- Adapter:
- Decorator:

## Weak Areas
-

## Debugging Mistakes
-

## Interview Questions Missed
-

## Predict-the-Output Mistakes
-

## Design Insights
-

## Next Review
-
```

# 48. Canonical References and Source Discipline

Primary language source:

```text
ECMAScript Language Specification
https://tc39.es/ecma262/
```

Useful type references:

```text
TypeScript Handbook — Classes
https://www.typescriptlang.org/docs/handbook/2/classes.html

TypeScript Handbook — Type Compatibility
https://www.typescriptlang.org/docs/handbook/type-compatibility.html
```

Source discipline:

```text
JavaScript mechanics
→ ECMAScript

TypeScript static behavior
→ TypeScript documentation

composition/design quality
→ responsibility, contract, cohesion, coupling, lifecycle

performance
→ measurement + workload evidence
```

# 49. Completion Snapshot

```text
Chapter: 011
Title: Composition & Delegation

Theory         [ ]
Implementation  [ ]
Debugging       [ ]
Code Review     [ ]
Interview       [ ]
Prediction      [ ]
Mastery         [ ]

Overall: [ ] Not Started
```

# Final Mental Model

A composition graph is:

```text
                    Composite
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      Gateway        Policy      Repository
          │            │            │
       payment      decisions   persistence
```

For every edge, ask:

```text
1. Why does this dependency exist?
2. What responsibility does it provide?
3. What contract does the composite rely on?
4. Who owns the collaborator?
5. What is its lifetime?
6. Is sharing intentional?
7. Can the dependency be replaced safely?
8. Is the collaboration direction clear?
9. Is the composite becoming a god object?
10. Would another boundary make the graph clearer?
```

# Principal Design Principle

> **Prefer composition when responsibilities vary independently: make collaborators explicit, give each object ownership of a coherent responsibility, and keep collaboration contracts smaller than the implementations behind them.**

# Track Mapping

```text
Track A — Core Theory
    composition
    delegation
    dependency injection
    collaborator contracts
    ownership
    lifecycle
    cohesion/coupling
    collaboration graphs

Track B — Implementation
    checkout composition
    strategy/policy objects
    adapter
    decorator
    facade
    dependency graphs

Track C — Interview / Reasoning
    composition vs inheritance
    dependency direction
    god orchestrator diagnosis
    lifecycle reasoning
    shared-state analysis
    capability boundaries
```
