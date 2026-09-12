# Chapter 12 — Cohesion & Coupling

> **JavaScript OOP + LLD Mastery**
>
> Cohesion and coupling are two of the most useful lenses for evaluating object design.
>
> **Cohesion asks:** “How strongly do the responsibilities inside this module or object belong together?”
>
> **Coupling asks:** “How strongly does this module depend on other modules?”
>
> Strong LLD does not mean “minimum dependencies” or “maximum abstraction.” It means placing responsibilities and dependencies so that change remains local, contracts stay understandable, and collaboration stays intentional.

**Status:** `[ ] Not Started`

---

# 1. Learning Objectives

By the end of this chapter, you should be able to:

```text
[ ] define cohesion
[ ] define coupling
[ ] distinguish high cohesion from low cohesion
[ ] distinguish low coupling from high coupling
[ ] explain why cohesion and coupling must be considered together
[ ] identify coincidental cohesion
[ ] identify logical cohesion
[ ] identify temporal cohesion
[ ] identify procedural cohesion
[ ] identify communicational cohesion
[ ] identify sequential cohesion
[ ] identify functional cohesion
[ ] recognize cohesion problems in classes and modules
[ ] identify dependency types
[ ] identify data coupling
[ ] identify control coupling
[ ] identify stamp coupling
[ ] identify common/global coupling
[ ] identify content coupling conceptually
[ ] identify temporal dependency coupling
[ ] explain fan-in and fan-out
[ ] explain afferent and efferent coupling
[ ] explain change amplification
[ ] explain dependency direction
[ ] identify coupling through concrete implementation details
[ ] identify coupling through shared mutable state
[ ] identify coupling through inheritance
[ ] identify coupling through public data structures
[ ] identify temporal coupling
[ ] identify god objects
[ ] identify feature envy
[ ] identify shotgun surgery
[ ] identify divergent change
[ ] use cohesion/coupling as refactoring criteria
[ ] compare object, module and service boundaries
[ ] reason about trade-offs instead of using slogans
[ ] debug design failures caused by poor boundaries
[ ] implement and refactor a low-coupling design
[ ] defend architecture decisions using change scenarios
```

# 2. Prerequisites

Required:

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
Chapter 11 — Composition & Delegation
objects
classes
modules
interfaces
composition
delegation
dependencies
```

# 3. What Is It?

## Cohesion

Cohesion describes how strongly the responsibilities inside a unit belong together.

A highly cohesive object might represent:

```text
BankAccount
```

and own:

```text
deposit
withdraw
balance
```

because these operations all relate to the same concept.

A low-cohesion class might contain:

```text
database access
email sending
PDF generation
discount calculation
CSV parsing
password hashing
```

with little semantic unity.

---

## Coupling

Coupling describes how strongly one unit depends on another.

High coupling can mean:

```text
implementation dependency
state dependency
ordering dependency
inheritance dependency
technology dependency
```

A class that reaches deeply into another class's internals is strongly coupled.

# 4. Why Does It Exist?

Software changes.

Suppose one requirement changes:

```text
tax rules
```

Ideally:

```text
tax-related code changes
```

rather than:

```text
Order
Checkout
Invoice
Controller
Repository
Notification
Report
```

all changing because each contains tax details.

Cohesion helps place related responsibility together.

Coupling helps control how far a change spreads.

A useful goal:

```text
high cohesion
+
appropriate/controlled coupling
→
localized change
```

# 5. Mental Model

Think of a system as a graph:

```text
           ┌──────────────┐
           │   Module A   │
           └──────┬───────┘
                  │
                  ▼
           ┌──────────────┐
           │   Module B   │
           └──────┬───────┘
                  │
                  ▼
           ┌──────────────┐
           │   Module C   │
           └──────────────┘
```

Inside each module:

```text
cohesion
```

asks:

```text
Do these responsibilities belong together?
```

Across modules:

```text
coupling
```

asks:

```text
How much does A need to know about B?
```

The ideal is not:

```text
zero coupling.
```

The ideal is:

```text
meaningful dependencies
with stable contracts
and clear responsibility ownership.
```

# 6. Core Rules

## Rule 1 — High Cohesion Means Responsibilities Belong Together

Do not interpret high cohesion as:

```text
few methods.
```

A class can have many methods and still be highly cohesive if they serve one clear responsibility.

---

## Rule 2 — Low Coupling Means Fewer Unnecessary Dependencies

A module can legitimately depend on several collaborators.

The question is:

```text
Are these dependencies necessary and stable?
```

---

## Rule 3 — Coupling Is Not Automatically Bad

A service that depends on:

```text
Clock
Repository
PaymentGateway
```

may be well-designed.

The coupling is explicit and responsibility-driven.

---

## Rule 4 — Hidden Coupling Is More Dangerous

Examples:

```text
global variables
shared mutable state
implicit ordering
prototype mutation
framework globals
database assumptions
```

---

## Rule 5 — Cohesion and Coupling Trade Off

Splitting everything into tiny classes can:

```text
reduce local cohesion
increase collaboration complexity.
```

Combining everything can:

```text
increase local convenience
reduce cohesion
increase coupling.
```

Good LLD balances both.

---

## Rule 6 — Change Scenarios Are Better Than Slogans

Ask:

```text
What changes next?
Who should change?
Who should not change?
```

Use those answers to judge boundaries.

# 7. Syntax

Cohesion/coupling are design properties rather than JavaScript syntax.

Example:

```js
class Order {
  addItem(item) {}
  removeItem(id) {}
  calculateTotal() {}
}
```

Compared with:

```js
class Order {
  addItem(item) {}
  sendEmail() {}
  renderPdf() {}
  connectDatabase() {}
  parseCsv() {}
}
```

The second has weaker conceptual cohesion.

# 8. Basic Examples

## Example 1 — Cohesive Object

```js
class Money {
  constructor(amount, currency) {
    this.amount = amount;
    this.currency = currency;
  }

  add(other) {
    // money-specific behavior
  }

  equals(other) {
    // money-specific equality
  }
}
```

Responsibilities are centered on:

```text
Money as a value concept.
```

---

## Example 2 — Low Cohesion

```js
class UserManager {
  createUser() {}
  sendWelcomeEmail() {}
  generatePdfReport() {}
  migrateDatabase() {}
  calculateTax() {}
}
```

These operations have different reasons to change.

---

## Example 3 — Hidden Coupling

```js
class OrderService {
  calculate(order) {
    return globalConfig.taxRate * order.total;
  }
}
```

The service appears to have one explicit input but is coupled to:

```text
globalConfig
```

This is hidden dependency coupling.

# 9. Execution Walkthrough

Consider:

```js
class CheckoutService {
  constructor(payment, inventory, notifier) {
    this.payment = payment;
    this.inventory = inventory;
    this.notifier = notifier;
  }

  async checkout(order) {
    await this.inventory.reserve(order);
    await this.payment.charge(order.total);
    await this.notifier.send("completed");
  }
}
```

Dependencies:

```text
CheckoutService
├── Inventory
├── Payment
└── Notifier
```

This is not automatically high coupling.

Evaluate:

```text
Does Checkout need these responsibilities?
Are contracts stable?
Does each collaborator expose only required capability?
Would changing notifier internals change Checkout?
```

The quality depends on the answers.

# 10. Internal Mechanics

Cohesion and coupling are not ECMAScript runtime concepts.

They emerge from:

```text
references
imports
calls
shared state
inheritance
module boundaries
data structures
```

JavaScript does not enforce:

```text
high cohesion
low coupling.
```

They are engineering properties evaluated by humans, tests, static analysis, architecture tools, and change history.

# 11. ECMAScript / Specification Semantics

There is no ECMAScript algorithm named:

```text
CalculateCohesion()
CalculateCoupling()
```

These are design concepts.

The language provides mechanisms that create dependency relationships:

```text
imports/exports
objects
function calls
property access
prototype inheritance
closures
```

The design quality of those relationships is an architectural concern.

# 12. Advanced Behavior

## 12.1 Coincidental Cohesion

A unit groups unrelated things simply because they were convenient to place together.

Example:

```text
Utils
├── formatDate
├── encrypt
├── parseCsv
├── sendEmail
└── calculateTax
```

A "utility" label does not create conceptual cohesion.

---

## 12.2 Logical Cohesion

A module groups operations of the same broad category but selected by condition.

Example:

```js
function handle(type, data) {
  if (type === "email") {}
  if (type === "sms") {}
  if (type === "push") {}
}
```

The logic is related broadly, but responsibility may be better split if the variants evolve independently.

---

## 12.3 Temporal Cohesion

Operations are grouped because they happen at the same time.

Example:

```text
application startup
→ load config
→ create database
→ initialize metrics
→ send startup email
```

They may be temporally related but semantically unrelated.

---

## 12.4 Procedural Cohesion

Operations belong because they follow a required sequence.

```text
validate
→ transform
→ save
```

This can be appropriate for an orchestration object, but consider whether the orchestration itself is a cohesive responsibility.

---

## 12.5 Communicational Cohesion

Operations work on the same data.

For example:

```text
parseOrder
validateOrder
normalizeOrder
```

if all operate on one coherent Order representation.

---

## 12.6 Sequential Cohesion

Output from one operation becomes input to the next.

Example:

```text
parse
→ normalize
→ validate
```

This can be a useful pipeline but may still be better represented as separate composable stages.

---

## 12.7 Functional Cohesion

Everything in the unit contributes to one clearly defined responsibility.

Example:

```text
TaxCalculator
```

with:

```text
calculateTax
```

and closely related tax rules.

This is the strongest classic form of cohesion.

# 13. Coupling Categories

Useful conceptual categories include:

```text
data coupling
stamp coupling
control coupling
common/global coupling
content coupling
temporal coupling.
```

These categories are design lenses, not rigid laws.

# 14. Data Coupling

A module depends on another through required data.

Example:

```js
calculateTax(orderTotal, taxRate);
```

This is often relatively understandable coupling.

# 15. Stamp Coupling

A module receives a larger structure than it actually needs.

Example:

```js
calculateTax(order);
```

when it only needs:

```text
order.total
order.location
```

If the entire Order object is exposed unnecessarily, the callee becomes coupled to the larger structure.

A narrower contract might be:

```js
calculateTax(total, location);
```

when that represents the true requirement.

But avoid blindly flattening everything; a meaningful domain object may itself be the correct abstraction.

# 16. Control Coupling

One module tells another how to behave using control flags.

```js
generateReport(data, "pdf");
```

The callee must understand:

```text
pdf
excel
csv
```

This can spread behavioral-variant knowledge.

Polymorphism or strategy composition may be better when variants evolve independently.

# 17. Common / Global Coupling

Modules depend on shared global state.

Example:

```js
globalThis.config
```

or:

```js
const globalCache = new Map();
```

Problems can include:

```text
hidden dependency
test interference
lifecycle ambiguity
concurrency issues.
```

Explicit dependency injection often improves the boundary.

# 18. Content Coupling

One module depends on another module's internal implementation or representation.

Example:

```js
service.repository.connection.pool._internalState
```

This is extremely brittle.

Good encapsulation should prevent this level of coupling.

# 19. Temporal Coupling

A caller must invoke operations in a particular sequence.

Example:

```js
client.connect();
client.authenticate();
client.send();
```

If `send()` only works after `authenticate()`, the object has a temporal contract.

Sometimes this is valid.

But if possible, prefer an abstraction where invalid ordering is harder to represent.

# 20. Fan-In and Fan-Out

## Fan-In

How many consumers depend on a module.

High fan-in can indicate:

```text
stable shared capability.
```

But changes to the module can affect many consumers.

## Fan-Out

How many other modules one module depends on.

High fan-out can increase:

```text
change coordination
construction complexity
failure surface.
```

A service with:

```text
15 collaborators
```

deserves a cohesion review.

# 21. Afferent and Efferent Coupling

Afferent coupling:

```text
dependencies coming into a module
```

Efferent coupling:

```text
dependencies going out from a module.
```

A module used by many others can be difficult to change.

A module depending on many others can be difficult to isolate and test.

The balance matters.

# 22. Change Amplification

Suppose changing:

```text
payment provider
```

requires modifying:

```text
Checkout
Controller
Order
Invoice
Notification
Tests
```

The provider detail has high change amplification.

A better abstraction might localize:

```text
provider-specific changes
```

inside:

```text
PaymentGatewayAdapter.
```

# 23. Dependency Direction

A useful direction:

```text
application policy
      ↓
stable capability
      ↑
technology implementation
```

Avoid:

```text
domain object
      ↓
framework internals
      ↓
database SDK
```

when the domain does not truly require those details.

# 24. Inheritance Coupling

Inheritance creates dependency through:

```text
parent behavior
parent state
override points
constructor order
prototype relationships.
```

A child can break when the parent changes.

This is why inheritance depth and protected/internal hooks should be treated as coupling measurements.

# 25. Coupling Through Public Data

Example:

```js
order.items.push(item);
```

Clients now depend on:

```text
items being an array
```

Changing it to:

```text
Set
database-backed collection
lazy collection
```

could break consumers.

Encapsulation therefore reduces representation coupling.

# 26. Coupling Through Concrete Errors

If a service exposes:

```text
PostgresError
StripeError
AxiosError
```

then clients can become coupled to infrastructure.

Prefer domain/application errors where appropriate:

```text
PaymentDeclined
PersistenceUnavailable
```

The mapping belongs at the boundary.

# 27. Coupling Through Timing

If callers know:

```text
call A
wait
call B
then call C
```

the lifecycle contract is distributed.

Encapsulate workflows when practical.

Example:

```js
checkout.complete();
```

can hide:

```text
reserve
charge
record
notify
```

when those operations form one cohesive application-level workflow.

# 28. Advanced Behavior — Shotgun Surgery

A change requires editing many unrelated modules.

Example:

```text
tax rule change
→ 9 files
```

This is often a coupling smell.

The goal is not necessarily:

```text
one file.
```

The goal is:

```text
one coherent ownership boundary.
```

# 29. Advanced Behavior — Divergent Change

One module has many unrelated reasons to change.

Example:

```text
InvoiceService changes for:
database
PDF
tax
email
authentication.
```

That suggests low cohesion.

# 30. Advanced Behavior — Feature Envy

A method uses another object's data more than its own.

Example:

```js
class OrderReporter {
  total(order) {
    return order.items
      .map(...)
      .reduce(...);
  }
}
```

If total calculation is truly an Order responsibility, the behavior may belong with:

```js
order.total()
```

Feature envy is a signal, not an automatic refactoring command.

# 31. Advanced Behavior — God Object

A god object has:

```text
too many responsibilities
too many dependencies
too much state
too much coordination.
```

Symptoms:

```text
large constructor
many methods
many imports
many conditional branches
frequent changes for unrelated features.
```

Do not solve this automatically by creating dozens of tiny classes.

First identify responsibilities and change axes.

# 32. Advanced Behavior — Dependency Cycles

Graph:

```text
A → B
B → C
C → A
```

Cycles can make:

```text
initialization
testing
deployment
reasoning
```

harder.

A cycle may be acceptable at one level but problematic at another.

Look for:

```text
missing abstraction
wrong dependency direction
shared responsibility
```

# 33. Advanced Behavior — Temporal Coupling

Bad:

```js
service.init();
service.load();
service.start();
```

with undocumented ordering.

Better designs can encode lifecycle through:

```text
factory
builder
state object
single startup operation
```

The goal is to make invalid sequences harder to express.

# 34. Advanced Behavior — Cohesion Across Boundaries

Cohesion is not only a class property.

Evaluate:

```text
method
class
module
package
service
bounded context.
```

A class can be cohesive while its module boundary is poor.

A module can be cohesive while the package dependency graph is tangled.

Use the same question at every level:

```text
Do these responsibilities naturally belong together?
```

# 35. Edge Cases

## More Cohesion Can Increase Coupling

Combining related behavior into one object may increase the number of clients touching that object.

## Less Coupling Can Increase Complexity

Splitting a workflow into many independent units can create too much orchestration.

## High Fan-In Can Be Good

A stable foundational utility can intentionally have many consumers.

## High Fan-Out Can Be Valid

An orchestrator may legitimately coordinate many capabilities.

## Shared Domain Object

Several services can legitimately depend on one stable domain object.

The goal is not numerical perfection.

# 36. Common Misconceptions

```text
"high cohesion means one method."
"low coupling means no dependencies."
"more classes means higher cohesion."
"fewer classes means lower coupling."
"fan-out is always bad."
"fan-in is always bad."
"every god object should be split immediately."
"all cyclic dependencies are forbidden."
"all feature envy should be moved."
"every global is automatically a bug."
```

# 37. Common Mistakes

```text
[ ] splitting by arbitrary method count
[ ] creating utility classes with unrelated functions
[ ] allowing deep implementation access
[ ] sharing mutable global state
[ ] allowing concrete technology types into domain APIs
[ ] using flags for many behavioral variants
[ ] creating giant orchestration classes
[ ] allowing lifecycle order to remain implicit
[ ] ignoring dependency direction
[ ] refactoring without examining change scenarios
```

# 38. Comparison With Related Concepts

| Concept | Core question |
|---|---|
| Cohesion | Do responsibilities belong together? |
| Coupling | How strongly do units depend on each other? |
| Fan-in | How many consumers depend on this unit? |
| Fan-out | How many dependencies does this unit have? |
| Afferent coupling | How much depends on this unit? |
| Efferent coupling | How much does this unit depend on? |
| Shotgun surgery | Does one change spread across many places? |
| Divergent change | Does one module change for many unrelated reasons? |
| Feature envy | Is behavior closer to another object's data? |
| God object | Too many responsibilities/collaborations in one object |
| Composition | Explicit collaborator relationships |
| Encapsulation | Hide representation and control authority |

# 39. Performance Considerations

Cohesion/coupling are primarily design concerns, but they can affect runtime costs indirectly.

Potential costs of over-decomposition:

```text
extra function calls
extra object allocations
extra mapping
extra orchestration
```

Potential costs of under-decomposition:

```text
large hot objects
unnecessary data retention
complex branching
poor cache locality in some workloads.
```

Do not sacrifice good boundaries for hypothetical micro-optimizations.

Measure critical paths.

# 40. Memory Considerations

High fan-out objects may retain many collaborators:

```text
service
├── cache
├── client
├── repository
├── policy
└── logger
```

Long-lived objects holding short-lived dependencies can create retention problems.

Shared global objects can also keep large graphs alive for application lifetime.

Always reason about:

```text
ownership
lifetime
reachability.
```

# 41. Security Considerations

Coupling can become a security problem when sensitive capabilities spread unnecessarily.

Bad:

```text
every service receives raw database connection
```

Better:

```text
service receives least-privileged capability.
```

High cohesion helps keep security-sensitive responsibility localized.

Avoid global mutable authorization/configuration state where explicit scoped dependencies are possible.

# 42. Production Usage

Before creating or merging a class/module, ask:

```text
What is its primary responsibility?
What reasons cause it to change?
What does it depend on?
What implementation details does it expose?
Who depends on it?
What happens when one dependency changes?
```

Good boundaries are usually visible in:

```text
constructor dependencies
imports
public methods
returned types
error types
tests
```

# 43. Implementation From Scratch

## Exercise 1 — Split a God Object

Start with:

```js
class StoreManager {
  calculateTax() {}
  sendEmail() {}
  saveOrder() {}
  generateInvoice() {}
  authenticate() {}
  reserveInventory() {}
}
```

Refactor based on responsibility and change axes.

Do not create a class for every method.

---

## Exercise 2 — Measure Fan-Out

For a project directory, manually document each major module's:

```text
imports
direct collaborators
fan-out
```

Identify high-fan-out modules.

---

## Exercise 3 — Dependency Graph

Build a graph:

```text
Checkout
Order
PaymentGateway
Inventory
Notifier
Repository
TaxPolicy
Clock
```

Mark:

```text
direction
ownership
high fan-in
high fan-out
cycles.
```

---

## Exercise 4 — Remove Global Coupling

Replace:

```js
globalConfig.taxRate
```

with an explicit dependency.

Compare:

```text
testability
clarity
lifetime
```

---

## Exercise 5 — Reduce Control Coupling

Replace:

```js
generate(report, "pdf");
```

with a strategy/capability design.

Document whether the resulting design is actually better.

# 44. Debugging Exercises

## Debug 1 — Utility Class

```js
class Utils {
  formatDate() {}
  encrypt() {}
  calculateTax() {}
  sendEmail() {}
}
```

Identify the cohesion problem.

---

## Debug 2 — Deep Coupling

```js
service.repository.client.connection.pool.query(...)
```

Identify the coupling layers.

---

## Debug 3 — Global State

```js
const config = {
  taxRate: 0.18,
};

class TaxService {
  calculate(total) {
    return total * config.taxRate;
  }
}
```

Identify hidden coupling and redesign the boundary.

---

## Debug 4 — Temporal Coupling

```js
service.init();
service.authenticate();
service.start();
```

What contract does the caller have to remember?

Design a safer lifecycle.

---

## Debug 5 — God Object

```js
class ApplicationManager {
  users = [];
  orders = [];
  payments = [];
  reports = [];

  createUser() {}
  createOrder() {}
  chargePayment() {}
  generateReport() {}
  sendNotification() {}
}
```

Identify responsibilities and propose boundaries based on reasons to change.

# 45. Code Review Exercise

Review:

```js
class OrderService {
  constructor(
    database,
    redis,
    stripe,
    logger,
    email,
    tax,
    inventory,
    metrics,
  ) {}

  createOrder() {}
  calculateTax() {}
  reserveInventory() {}
  chargePayment() {}
  sendConfirmationEmail() {}
  generateMetrics() {}
  saveToDatabase() {}
}
```

Evaluate:

```text
1. Which responsibilities belong together?
2. Which dependencies are domain capabilities?
3. Which are infrastructure?
4. Which responsibilities should move?
5. Is this an orchestrator or a god object?
6. What changes together?
7. What can be hidden behind stable contracts?
8. What is the smallest meaningful decomposition?
```

# 46. Interview Questions

```text
1. What is cohesion?
2. What is coupling?
3. Why do we want high cohesion and controlled coupling?
4. Does low coupling mean zero dependencies?
5. What is coincidental cohesion?
6. What is logical cohesion?
7. What is temporal cohesion?
8. What is procedural cohesion?
9. What is communicational cohesion?
10. What is sequential cohesion?
11. What is functional cohesion?
12. What is data coupling?
13. What is stamp coupling?
14. What is control coupling?
15. What is common coupling?
16. What is content coupling?
17. What is temporal coupling?
18. What is fan-in?
19. What is fan-out?
20. What are afferent and efferent coupling?
21. What is change amplification?
22. What is shotgun surgery?
23. What is divergent change?
24. What is feature envy?
25. What is a god object?
26. How does inheritance increase coupling?
27. How does composition affect coupling?
28. When can splitting classes make a design worse?
29. How do you evaluate cohesion in a service class?
30. How would you use change scenarios to refactor a design?
```

# 47. Predict-the-Output Exercises

## Exercise A

```js
class TaxService {
  constructor(rate) {
    this.rate = rate;
  }

  calculate(total) {
    return total * this.rate;
  }
}

const tax = new TaxService(0.18);

console.log(tax.calculate(100));
```

## Exercise B

```js
const config = {
  rate: 0.18,
};

function calculate(total) {
  return total * config.rate;
}

config.rate = 0.2;

console.log(calculate(100));
```

Explain the global/shared-state coupling.

## Exercise C

```js
class Service {
  constructor(repository) {
    this.repository = repository;
  }

  save(value) {
    return this.repository.save(value);
  }
}

const repository = {
  save(value) {
    return `saved:${value}`;
  },
};

console.log(new Service(repository).save("A"));
```

## Exercise D

```js
class Order {
  constructor(items) {
    this.items = items;
  }

  total() {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }
}

const order = new Order([{ price: 10 }, { price: 20 }]);

console.log(order.total());
```

Explain why this is more cohesive than placing total calculation in an unrelated reporting class.

## Exercise E

```js
class App {
  constructor(a, b, c, d) {
    this.a = a;
    this.b = b;
    this.c = c;
    this.d = d;
  }
}

const a = {};
const b = {};
const c = {};
const d = {};

console.log(new App(a, b, c, d).a === a);
```

Explain why dependency count alone does not prove bad coupling.

# 48. Mastery Exercises

## Level 1 — Understand

Explain:

```text
cohesion
coupling
fan-in
fan-out
change amplification.
```

## Level 2 — Explain

Classify examples as:

```text
coincidental
logical
temporal
procedural
communicational
sequential
functional cohesion.
```

## Level 3 — Predict

Given a dependency graph, identify:

```text
high fan-out
high fan-in
cycles
hidden global coupling
representation coupling.
```

## Level 4 — Implement

Build:

```text
cohesive Order
tax policy
payment capability
repository boundary
notification adapter.
```

## Level 5 — Debug

Find:

```text
god object
utility-class cohesion problem
temporal coupling
deep concrete coupling
global coupling.
```

## Level 6 — Apply

Refactor a:

```text
Jewellery ERP OrderManager
```

that currently handles:

```text
order creation
inventory
gold calculation
tax
payment
invoice
notification
audit.
```

Split by real responsibilities and change axes, not arbitrary method count.

## Level 7 — Compare

Compare:

```text
one large cohesive domain object
vs
many tiny classes
```

and explain when either can be better.

## Level 8 — Defend

Answer:

> How do you know whether a class should be split?

Your answer must use:

```text
responsibilities
reasons to change
dependency graph
data ownership
client needs
change scenarios
cohesion
coupling.
```

# 49. Key Takeaways

```text
1. Cohesion measures how naturally responsibilities belong together.
2. Coupling measures how strongly units depend on one another.
3. Good design seeks high cohesion and controlled coupling.
4. Zero coupling is neither possible nor desirable in useful systems.
5. Hidden coupling is often more dangerous than explicit coupling.
6. Coincidental cohesion groups unrelated responsibilities.
7. Functional cohesion is a strong form of responsibility alignment.
8. Data coupling can be explicit and understandable.
9. Stamp coupling exposes more structure than a client needs.
10. Control coupling spreads behavioral-variant knowledge.
11. Global/common coupling hides dependencies.
12. Content coupling violates abstraction boundaries.
13. Temporal coupling distributes lifecycle knowledge across callers.
14. Fan-in can indicate a valuable shared capability.
15. Fan-out can indicate coordination complexity.
16. Afferent and efferent coupling help reason about dependency direction.
17. Change amplification is a practical way to observe coupling.
18. Shotgun surgery indicates change spread across boundaries.
19. Divergent change often indicates low cohesion.
20. Feature envy can indicate misplaced behavior.
21. God objects combine too many responsibilities and dependencies.
22. Inheritance creates coupling through state, overrides and lifecycle.
23. Composition makes dependencies explicit but can still become over-connected.
24. Splitting everything into tiny objects can create collaboration overhead.
25. The best boundary is usually discovered through responsibilities and change scenarios.
26. Cohesion and coupling are decision tools, not numerical purity tests.
```

# 50. Concept Connections

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
Chapter 11 — Composition & Delegation
identity
ownership
abstraction
composition
delegation
polymorphism
```

## Builds Toward

```text
Chapter 13 — Responsibility-Driven Object Design
Chapter 14 — GRASP
Chapter 15 — SOLID
Chapter 16 — Dependency Direction
Chapter 17 — Dependency Injection
Chapter 18 — Object Boundaries
design patterns
DDD
module design
architecture
refactoring
LLD decomposition
```

## Related Concepts

```text
responsibility
change axes
fan-in
fan-out
dependency direction
information hiding
shotgun surgery
feature envy
god object
temporal coupling
architecture boundaries
```

## Why This Chapter Matters

You now have the core object-design mechanisms:

```text
object
prototype
constructor
class
encapsulation
abstraction
inheritance
polymorphism
composition.
```

Cohesion and coupling provide the first systematic way to judge whether those mechanisms are being used well.

# 51. Completion Criteria

Mark:

```text
[+] Completed
```

when you can:

```text
identify cohesion problems
identify coupling problems
classify dependency types
trace fan-in/fan-out
find change amplification
identify god objects
propose responsibility-based boundaries.
```

Mark:

```text
[*] Mastered
```

when you can take an unfamiliar design and answer:

```text
What belongs together?
What should separate?
Who owns each state?
Which dependency is unnecessary?
Where does change propagate?
Which collaboration is intentional?
Which coupling is hidden?
```

and defend the answer using concrete change scenarios.

Reading alone does not mark mastery.

# 52. Revision / Retrieval Record

```md
# Chapter 12 — Revision / Retrieval Record

## Attempt
- Date:
- Duration:
- Status before:
- Status after:

## Cohesion
- Current responsibility:
- What belongs together?
- What does not?

## Coupling
- Dependencies:
- Hidden dependencies:
- Concrete representation dependencies:

## Coupling Types
- Data:
- Stamp:
- Control:
- Common/global:
- Content:
- Temporal:

## Graph
- Fan-in:
- Fan-out:
- Cycles:
- Afferent:
- Efferent:

## Change Scenarios
- What changes?
- Which modules change?
- Which modules should not change?

## Smells
- God object:
- Feature envy:
- Shotgun surgery:
- Divergent change:
- Temporal coupling:

## Refactoring
- Boundary:
- Responsibility:
- Dependency:
- Result:

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

# 53. Canonical References and Source Discipline

Primary language source:

```text
ECMAScript Language Specification
https://tc39.es/ecma262/
```

Design concepts are architectural rather than ECMAScript features.

Useful references for JavaScript/TypeScript mechanics:

```text
TypeScript Handbook
https://www.typescriptlang.org/docs/handbook/intro.html
```

Source discipline:

```text
language behavior
→ ECMAScript

type-system behavior
→ TypeScript documentation

cohesion/coupling
→ architecture/design analysis

refactoring decisions
→ responsibility, change scenarios, client needs
```

Do not present cohesion or coupling as exact runtime properties produced by the JavaScript engine.

# 54. Completion Snapshot

```text
Chapter: 012
Title: Cohesion & Coupling

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

Evaluate every class/module with two lenses.

## Inside the boundary

```text
COHESION

┌────────────────────────────┐
│        ONE UNIT            │
│                            │
│ responsibility A           │
│ responsibility A           │
│ responsibility A           │
│                            │
└────────────────────────────┘
```

Ask:

```text
Do these responsibilities share a meaningful reason to exist together?
Do they change together?
Do they operate on the same conceptual state?
```

## Across the boundary

```text
COUPLING

Module A
   │
   ├── stable capability
   │
   └── implementation dependency?
          │
          ▼
      Module B
```

Ask:

```text
Does A need to know how B works?
Does A depend on B's representation?
Does A rely on B's lifecycle?
Does A receive more authority/data than necessary?
Will B's change force A to change?
```

Then apply the change test:

```text
Requirement changes
       ↓
Which object should change?
       ↓
Which objects should remain unchanged?
       ↓
If too many change:
       ↓
inspect cohesion/coupling.
```

---

# Principal Design Principle

> **Place responsibilities where they naturally belong, then make the dependencies between those responsibilities as explicit, narrow, stable, and change-resistant as practical.**

# Track Mapping

```text
Track A — Core Theory
    cohesion
    coupling
    coupling categories
    fan-in/fan-out
    dependency direction
    change amplification
    design smells

Track B — Implementation
    responsibility refactoring
    dependency graphs
    global-coupling removal
    strategy-based redesign
    god-object decomposition

Track C — Interview / Reasoning
    cohesion classification
    coupling diagnosis
    change-scenario reasoning
    god-object analysis
    class splitting judgment
    architecture trade-offs
```
