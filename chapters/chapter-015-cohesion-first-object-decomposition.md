# Chapter 15 — Cohesion-First Object Decomposition

> **Part:** C — Responsibility-Driven Design
> **Prerequisite:** Chapters 12–14
> **Next:** Chapter 16 — Coupling-First Dependency Design
> **Status:** `[+] Completed`

---

# Chapter Position

Chapter 12 introduced cohesion and coupling as design forces.

Chapter 13 established responsibility-driven object design.

Chapter 14 formalized those decisions through GRASP.

This chapter turns **cohesion into a decomposition method**.

The central question is:

> **Given a large object, service, module, or component, how do you decide what should stay together and what should become a separate responsibility?**

The answer is not:

```text
"Split large classes."
```

The better question is:

```text
"Which responsibilities have a strong reason to belong together?"
```

---

# 1. Learning Objectives

By the end of this chapter, you should be able to:

- define cohesion at object, class, module, package, service, and subsystem levels
- distinguish strong functional cohesion from weak coincidental cohesion
- identify logical, temporal, procedural, communicational, and sequential cohesion
- decompose a large class by meaningful responsibility clusters
- use state, invariants, shared knowledge, and change reasons as cohesion evidence
- distinguish under-decomposition from over-decomposition
- detect god objects, service blobs, controller blobs, feature envy, shotgun surgery, divergent change, data clumps, and primitive obsession
- use TypeScript modules and capability interfaces to reinforce cohesive boundaries
- decide when a class should remain together
- decide when a function, value object, policy, adapter, repository, or use case is the better boundary
- evaluate decomposition using coupling, performance, memory, security, reliability, and observability
- apply cohesion-first decomposition to enterprise and jewellery ERP workflows
- defend decomposition choices in LLD interviews

---

# 2. Prerequisites

This chapter assumes:

- JavaScript object identity and prototypes
- constructors and classes
- class fields and private state
- abstraction and contracts
- inheritance and subtyping
- polymorphism
- composition and delegation
- cohesion and coupling
- responsibility-driven design
- GRASP

Chapter 12, Chapter 13, and Chapter 14 are the immediate conceptual prerequisites.

---

# 3. What Is Cohesion?

**Cohesion** describes how strongly the responsibilities inside a unit belong together.

The unit can be:

```text
function
class
module
package
component
service
subsystem
```

The practical question is:

> **Why are these responsibilities together?**

Strong answer:

```text
They serve the same meaningful purpose.
```

Weak answer:

```text
They happened to be created in the same file.
```

---

# 4. Cohesion Is About Relationships

Consider:

```ts
class Order {
  addLine() {}
  removeLine() {}
  total() {}
  confirm() {}
  cancel() {}
}
```

These operations share a strong conceptual center:

```text
order lifecycle and state
```

Compare:

```ts
class Order {
  addLine() {}
  sendEmail() {}
  exportPdf() {}
  hashPassword() {}
  resizeImage() {}
}
```

A shared class name does not create cohesion.

The relationships between responsibilities are weak.

---

# 5. Cohesion Is Not a Class-Size Rule

A class can be:

```text
large and cohesive
```

or:

```text
small and incoherent
```

Example:

```text
Order
    20 related methods
```

can be healthier than:

```text
20 one-method classes
```

where each extraction exists only to reduce line count.

The objective is:

```text
semantic cohesion
```

not:

```text
minimum class size
```

---

# 6. Common Cohesion Categories

A useful classic spectrum is:

```text
Coincidental
    ↓
Logical
    ↓
Temporal
    ↓
Procedural
    ↓
Communicational
    ↓
Sequential
    ↓
Functional
```

These categories are diagnostic vocabulary.

They are not a universal numeric score.

---

# 7. Coincidental Cohesion

Coincidental cohesion exists when unrelated responsibilities are grouped without meaningful conceptual reason.

Example:

```ts
class UtilityService {
  calculateTax() {}
  resizeImage() {}
  sendEmail() {}
  parseCsv() {}
}
```

The members are together because they are convenient to place together.

This is among the weakest forms of cohesion.

---

# 8. Logical Cohesion

Logical cohesion groups related kinds of operations selected by a mode or type.

Example:

```ts
class InputHandler {
  handle(type: "mouse" | "keyboard" | "touch") {}
}
```

There is a conceptual relation:

```text
input handling
```

but implementation may become a large conditional.

If variants evolve independently, polymorphism can provide stronger boundaries.

---

# 9. Temporal Cohesion

Temporal cohesion groups responsibilities because they happen at the same time.

Example:

```text
application startup
    load configuration
    connect database
    warm cache
    register routes
```

They share lifecycle timing.

That can be perfectly valid.

But temporal coincidence alone does not prove strong functional cohesion.

---

# 10. Procedural Cohesion

Procedural cohesion exists when operations belong together because they follow a sequence.

Example:

```text
validate
    ↓
load
    ↓
transform
    ↓
save
```

The order creates a relationship.

But sequence alone does not prove that one class should own every step.

---

# 11. Communicational Cohesion

Communicational cohesion exists when operations use the same meaningful data.

Example:

```text
CustomerProfile
    read profile
    validate profile
    update profile
```

The methods share a meaningful information center.

Still ask whether they also share:

```text
business purpose
invariants
change reasons
```

---

# 12. Sequential Cohesion

Sequential cohesion exists when one operation's output becomes another's input.

Example:

```text
parse
  ↓
validate
  ↓
transform
```

This can describe a coherent pipeline.

It does not imply the entire pipeline must be one class.

A pipeline can remain highly cohesive while using several functions.

---

# 13. Functional Cohesion

Functional cohesion is strong when everything in a unit contributes to one well-defined purpose.

Example:

```text
TaxCalculator
    determine taxable amount
    apply jurisdiction rules
    return tax
```

All behavior supports the same function.

This is usually a desirable target.

---

# 14. Responsibility and Cohesion

From responsibility-driven design:

```text
Responsibility
    ↓
Who should know/do this?
```

Cohesion adds:

```text
What else belongs with that responsibility?
```

Therefore:

```text
responsibilities
    ↓
responsibility clusters
    ↓
cohesive objects/modules
```

---

# 15. Cohesion-First Decomposition

A repeatable method:

```text
Start with behavior
    ↓
list responsibilities
    ↓
identify state
    ↓
identify invariants
    ↓
group related responsibilities
    ↓
identify common change reasons
    ↓
create meaningful boundaries
    ↓
review dependencies
    ↓
validate scenarios
```

Do not start with:

```text
"This class has 800 lines."
```

Line count is a signal, not a reason.

---

# 16. Decompose by Responsibility, Not Line Count

Bad:

```text
first 100 lines → A
next 100 lines → B
```

Better:

```text
pricing → PricingPolicy
payment → PaymentGateway
order lifecycle → Order
persistence → OrderRepository
```

Every boundary should have a semantic explanation.

---

# 17. Decompose by Change Reason

Ask:

> What reason would make this code change?

Suppose:

```text
tax logic
payment provider integration
order state transitions
```

change independently.

That is evidence for separate responsibility clusters.

A practical decomposition question is:

```text
Would the same business or technical change affect all members?
```

---

# 18. Change Cohesion

Strong cohesion often correlates with:

```text
members changing together
```

Weak cohesion often correlates with:

```text
members changing for unrelated reasons
```

Example:

```text
Invoice
    calculateTotal
    approve
    renderPdf
    sendEmail
```

These may respond to four different sources of change.

---

# 19. Scenario Cohesion

Use a real workflow:

```text
FinalizeSale
```

List what happens:

```text
validate sale
calculate amount
reserve inventory
charge payment
persist invoice
audit action
```

Then group them by conceptual ownership.

Do not keep all steps in one service just because they are one scenario.

---

# 20. Data Cohesion vs Behavioral Cohesion

Two operations may use the same data and still belong to different capabilities.

Example:

```text
Order
    toPdf()
    toCsv()
```

Both use order data.

But representation formats may evolve independently.

By contrast:

```text
Order
    confirm()
    cancel()
    markPaid()
```

share:

```text
state
invariants
domain lifecycle
```

That is stronger cohesion evidence.

---

# 21. Cohesion and State Ownership

A strong cluster often contains:

```text
state
+
rules
+
transitions
```

Example:

```ts
class Invoice {
  #status = "DRAFT";

  approve() {}
  cancel() {}
  markPaid() {}
}
```

These methods are cohesive because they operate on one lifecycle.

---

# 22. Cohesion and Invariants

Shared invariants are powerful evidence.

Example:

```text
quantity > 0
line total = quantity × unit price
```

Both relate strongly to:

```text
OrderLine
```

The line can own the state and the behavior that protects it.

---

# 23. Cohesion and Aggregate Boundaries

An aggregate root may be cohesive around one consistency boundary.

```text
Order
├── OrderLine
├── ShippingAddress
└── Totals
```

But do not move every leaf behavior into the root.

Use:

```text
aggregate-level cohesion
+
local object cohesion
```

together.

---

# 24. Cohesion and Modules

A module can be cohesive even with multiple files.

```text
pricing/
  money.ts
  pricing-policy.ts
  tax-policy.ts
```

The module-level question is:

> Do these files together form one useful capability?

---

# 25. Cohesion Across Levels

Review cohesion at:

```text
function
class
module
package
component
service
bounded context
```

A cohesive class inside a badly structured module is still part of a broader cohesion problem.

---

# 26. Cohesion and Interfaces

A cohesive capability often produces a focused interface.

Good:

```ts
interface PaymentGateway {
  charge(input: ChargeInput): Promise<ChargeResult>;
}
```

Suspicious:

```ts
interface PaymentManager {
  charge();
  refund();
  exportReport();
  sendEmail();
  getCustomer();
}
```

The second may bundle unrelated responsibilities.

---

# 27. Interface Cohesion

Ask:

```text
Why should a consumer need these methods together?
```

Do not split merely because one consumer uses only one method.

Split when the methods represent genuinely distinct capabilities or change reasons.

---

# 28. Cohesive Capability Interfaces

Example:

```ts
interface PaymentCharger {
  charge(input: ChargeInput): Promise<ChargeResult>;
}

interface PaymentReader {
  getPayment(id: string): Promise<Payment>;
}
```

This can help when consumers require different capabilities.

---

# 29. Cohesion-First Refactoring Workflow

Use:

```text
1. Name the existing unit.
2. Inventory every responsibility.
3. Group them semantically.
4. Mark state each group uses.
5. Mark invariants.
6. Mark collaborators.
7. Mark change reasons.
8. Identify cohesive clusters.
9. Extract only meaningful clusters.
10. Delegate from the old location.
11. Run behavior tests.
12. Re-check cohesion and coupling.
```

---

# 30. Responsibility Inventory

| Responsibility | State | Invariant | Change Reason | Candidate |
|---|---|---|---|---|
| confirm order | order status | valid transition | domain rule | Order |
| total order | lines | total correctness | pricing | Order/Pricing |
| charge payment | payment data | provider protocol | provider change | Gateway |
| persist order | order data | storage consistency | DB change | Repository |
| render PDF | order data | document format | renderer change | Renderer |

The table makes candidate clusters visible.

---

# 31. Cohesion Cluster Map

```text
Order responsibilities
├── lifecycle
│   ├── confirm
│   ├── cancel
│   └── markPaid
│
├── lines
│   ├── add
│   ├── remove
│   └── total
│
├── integration
│   └── charge
│
└── representation
    ├── toPdf
    └── toCsv
```

Now decide which clusters belong to:

```text
Order
PaymentGateway
OrderPdfRenderer
OrderCsvExporter
```

---

# 32. Extract the Strongest Cluster First

When refactoring, extract the clearest independent cluster first.

For example:

```text
PDF rendering
```

is usually easy to explain as:

```text
InvoicePdfRenderer
```

Then re-evaluate what remains.

Incremental refactoring is safer than giant redesign.

---

# 33. Cohesion Before Coupling

A useful sequence:

```text
1. What belongs together?
2. What should depend on what?
```

If you begin with dependency minimization before understanding responsibilities, you can end up with many isolated but poorly meaningful components.

---

# 34. Cohesion Does Not Ignore Coupling

You can create:

```text
20 highly cohesive classes
```

with:

```text
50 unnecessary dependencies
```

That is not automatically better.

Therefore:

```text
cohesion-first
    ≠
cohesion-only
```

---

# 35. Cohesion–Coupling Balance

The practical target is:

```text
strong internal relationship
+
limited unnecessary external dependency
```

This becomes the foundation for the next chapter on coupling.

---

# 36. Decomposition by Knowledge

Group behavior around stable knowledge boundaries.

Example:

```text
OrderLine
    subtotal
    changeQuantity
    validateQuantity
```

These responsibilities rely on line state.

---

# 37. Decomposition by Invariant

Suppose:

```text
Account balance >= 0
```

Responsibilities such as:

```text
withdraw
deposit
reserve
release
```

may share strong account-state cohesion.

If an operation also introduces an unrelated external policy, extract that policy while preserving account state ownership.

---

# 38. Decomposition by Policy

Policies often form cohesive decision boundaries:

```text
TaxPolicy
DiscountPolicy
ShippingPolicy
ApprovalPolicy
PricingPolicy
```

A policy is cohesive when its behavior serves one decision concept.

---

# 39. Decomposition by Workflow

A use case can be cohesive around one business outcome:

```text
CheckoutUseCase
FinalizeSaleUseCase
RefundPaymentUseCase
```

Its responsibility is coordination.

It does not automatically own all business rules.

---

# 40. Decomposition by Integration

Integration responsibilities are often cohesive:

```text
StripePaymentGateway
PostgresOrderRepository
EmailSender
S3DocumentStore
```

Their common center is interaction with a technical boundary.

---

# 41. Decomposition by Representation

Representation concerns can be grouped:

```text
OrderDtoMapper
OrderPdfRenderer
OrderCsvExporter
```

Do not make the domain object responsible for every external representation.

---

# 42. Cohesion and Serialization

Serialization may be a focused capability.

```ts
class OrderDtoMapper {
  toResponse(order: Order) {}
}
```

PDF:

```ts
class OrderPdfRenderer {
  render(order: Order) {}
}
```

CSV:

```ts
class OrderCsvExporter {
  export(order: Order) {}
}
```

---

# 43. Cohesion and Validation

Validation itself can be decomposed:

```text
input validation
domain invariants
storage constraints
```

They change for different reasons.

Keep them separate when that improves clarity.

---

# 44. Cohesion and Authorization

Authorization policies can be cohesive:

```ts
class SaleApprovalPolicy {
  canApprove(actor: Actor, sale: Sale) {
    ...
  }
}
```

Do not scatter the same permission rule across unrelated classes.

---

# 45. Cohesion and Audit

Audit recording can be a focused technical capability:

```ts
interface AuditLogger {
  record(entry: AuditEntry): Promise<void>;
}
```

Domain state changes remain separate.

---

# 46. Cohesion and Observability

Telemetry can be cohesive around operational concerns:

```text
Metrics
Tracing
StructuredLogging
Audit
```

Avoid a single catch-all manager unless it has a meaningful center.

---

# 47. Cohesion and Error Mapping

Provider-specific error mapping can be isolated:

```ts
class StripeErrorMapper {
  toPaymentFailure(error: unknown) {}
}
```

This prevents external error semantics from spreading through the domain.

---

# 48. Cohesion and Retry Policies

Retry logic can form a cohesive technical boundary:

```ts
class RetryPolicy {
  execute<T>(operation: () => Promise<T>): Promise<T> {
    ...
  }
}
```

Do not duplicate retry loops in every adapter.

---

# 49. Cohesion and Circuit Breakers

Circuit state:

```text
closed
open
half-open
```

forms a coherent resilience capability.

A `CircuitBreaker` can encapsulate:

```text
failure counting
state transitions
recovery behavior
```

---

# 50. Cohesion and Caching

Caching is cohesive when the component owns:

```text
lookup
store
eviction policy
cache-specific concerns
```

Example:

```ts
interface ProductCache {
  get(id: string): Promise<Product | null>;
  set(product: Product): Promise<void>;
}
```

---

# 51. Cohesion and Persistence

A repository should remain focused:

```ts
interface OrderRepository {
  findById(id: string): Promise<Order | null>;
  save(order: Order): Promise<void>;
}
```

If it also sends email and renders PDFs, cohesion has collapsed.

---

# 52. Cohesion and Factories

A factory should be cohesive around one creation family.

Good:

```text
OrderFactory
```

Suspicious:

```text
ApplicationFactory
    creates orders
    users
    invoices
    payments
    reports
```

---

# 53. Cohesion and Dependency Injection

Dependency injection does not create cohesion.

This can still be a blob:

```ts
class GodService {
  constructor(a, b, c, d, e, f, g, h) {}
}
```

The constructor merely makes the coupling visible.

---

# 54. Cohesion and Constructor Dependencies

Ask:

> Do these dependencies support one coherent responsibility?

For:

```text
CheckoutUseCase
    OrderRepository
    Inventory
    PaymentGateway
```

the answer may be yes because all support checkout coordination.

For:

```text
CustomerService
    Stripe
    S3
    Mailer
    ReportEngine
    Redis
```

the answer deserves review.

---

# 55. Cohesion and Public API Size

A cohesive API is semantically understandable.

Weak:

```text
manager.run()
manager.process()
manager.handle()
```

Better:

```text
order.confirm()
order.cancel()
order.total()
```

Method names reveal responsibility.

---

# 56. Cohesion and Private State

Private fields can keep state inside a cohesive boundary.

```ts
class Cart {
  #items: CartItem[] = [];

  addItem() {}
  removeItem() {}
  total() {}
}
```

The state and behavior have one conceptual center.

---

# 57. Cohesion and JavaScript Modules

Modules provide useful boundaries.

```ts
// money.ts
export class Money {}

// pricing.ts
export interface PricingPolicy {}
```

Avoid giant modules exporting unrelated helpers.

---

# 58. Cohesion and Closures

A closure can represent a cohesive stateful capability:

```ts
function createSequence() {
  let next = 0;

  return {
    allocate() {
      return next++;
    }
  };
}
```

State and behavior share one small concept.

---

# 59. Cohesion and Functions

Pure functions can be highly cohesive:

```ts
function calculateSubtotal(
  price: number,
  quantity: number
) {
  return price * quantity;
}
```

Do not create a class merely to satisfy an object-oriented aesthetic.

---

# 60. Cohesion and Value Objects

A value object can encapsulate one cohesive value concept:

```text
Money
Quantity
Currency
Percentage
EmailAddress
DateRange
```

Each concept can own:

```text
validation
comparison
operations
invariants
```

---

# 61. Cohesion and Entities

An entity often has a cohesive center around:

```text
identity
lifecycle
state
invariants
domain behavior
```

Not every function involving one of its fields belongs there.

---

# 62. Cohesion and Domain Services

A domain service should represent one meaningful domain operation.

Weak:

```text
DomainService
    tax
    inventory
    reporting
    notifications
```

Better:

```text
CurrencyExchangeService
```

when exchange is genuinely a domain operation spanning concepts.

---

# 63. Cohesion and Application Services

An application service can be cohesive around one use case or closely related use cases.

Good:

```text
CheckoutUseCase
```

Suspicious:

```text
BusinessService
    40 unrelated operations
```

---

# 64. Cohesion and Controllers

Controllers should remain cohesive around transport boundaries.

```text
OrdersController
    order endpoints
```

Avoid a universal controller containing:

```text
orders
users
reports
payments
inventory
```

---

# 65. Feature-Based Cohesion

A feature module can group:

```text
controller
use case
domain
ports
adapters
```

around one business capability:

```text
orders/
payments/
inventory/
pricing/
```

This can be more discoverable than a repository organized only by technical layer.

---

# 66. Cohesion vs Folder Structure

Folder placement does not prove cohesion.

This can still be coherent:

```text
services/
    order.ts
    payment.ts
    tax.ts
```

And this can still be incoherent:

```text
orders/
    everything.ts
```

Use structure to reinforce semantic boundaries.

---

# 67. Cohesion and Bounded Contexts

A bounded context is a larger semantic boundary.

```text
Ordering
Payments
Inventory
Reporting
```

Each should contain a coherent model and language.

---

# 68. Cohesion and Anti-Corruption Layers

An anti-corruption layer can be cohesive around translation:

```text
ExternalPaymentModel
    ↓
PaymentTranslator
    ↓
InternalPaymentModel
```

Its purpose is representation and model protection.

---

# 69. Cohesion and Legacy Adapters

A legacy adapter is cohesive when it primarily:

```text
calls
maps
translates
handles legacy protocol
```

It should not become a home for unrelated domain rules.

---

# 70. Cohesion and Event Handlers

An event handler can own one event responsibility:

```ts
class SaleFinalizedHandler {
  handle(event: SaleFinalized) {}
}
```

Avoid one giant handler consuming every domain event and containing unrelated business logic.

---

# 71. Cohesion and Jobs

A background job should have a focused purpose.

Good:

```text
ExpireReservationsJob
```

Potentially weak:

```text
MaintenanceJob
    expire reservations
    send emails
    rebuild indexes
    clean caches
```

The latter is grouped mainly by operational convenience.

---

# 72. Cohesion and Scheduling

A scheduler may be temporally cohesive around:

```text
when to run
```

while the task itself should remain behaviorally cohesive around:

```text
what to do
```

---

# 73. Cohesion and Long Methods

A long method may contain several responsibility clusters.

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

If it is a use-case orchestrator, this may still be cohesive.

Ask which operations are:

```text
coordination
domain decision
infrastructure
```

before extracting them.

---

# 74. Cohesion and Long Parameter Lists

Repeated parameters can reveal missing concepts:

```ts
createSale(
  customerId,
  branchId,
  metal,
  weight,
  purity,
  price,
  taxRate,
  discount
)
```

Potential concepts:

```text
SaleContext
JewellerySpecification
PricingInput
TaxContext
```

Extract only concepts with real meaning.

---

# 75. Cohesion and Data Clumps

Repeated parameter groups can indicate a concept that deserves its own cohesive boundary.

```text
country
currency
taxRate
```

may form:

```text
TaxContext
```

when rules and semantics justify it.

---

# 76. Cohesion and Primitive Obsession

Primitive values can hide cohesive rules.

```text
Money
Quantity
OrderStatus
EmailAddress
```

Wrapping them can centralize:

```text
validation
operations
comparisons
semantics
```

---

# 77. Cohesion and Feature Envy

Feature envy can indicate that behavior has stronger cohesion with another object's data.

Example:

```ts
class ReportService {
  total(order: Order) {
    return order.items.reduce(...);
  }
}
```

If the behavior is a natural order responsibility:

```ts
order.total()
```

may improve cohesion.

Do not move code blindly.

---

# 78. Cohesion and Shotgun Surgery

If one conceptual rule changes many places:

```text
controller
service
invoice
report
```

the responsibility may be too fragmented.

Find the cohesive owner.

---

# 79. Cohesion and Divergent Change

One class changing for many unrelated reasons is a warning:

```text
OrderService
    tax
    payment
    reporting
    email
```

The class has several responsibility centers.

---

# 80. Cohesion and God Objects

God objects often combine:

```text
many responsibilities
many collaborators
many state areas
many reasons to change
```

The goal is not merely "smaller."

The goal is:

```text
clear responsibility clusters
```

---

# 81. Cohesion and Service Blobs

A service blob can sound business-focused while still being incoherent.

Example:

```text
BusinessService
    createOrder
    approveInvoice
    reserveStock
    calculateTax
```

All are business operations, but they represent different capability centers.

---

# 82. Cohesion and Generic Names

Suspicious names:

```text
Manager
Processor
Helper
Util
Common
Service
```

These are not automatically bad.

Inspect the responsibility behind the name.

---

# 83. Cohesion Cluster Test

For a candidate cluster, complete:

```text
These responsibilities belong together because
______________________________________________

They share state:
______________________________________________

They protect invariants:
______________________________________________

They change together when:
______________________________________________

They serve this capability:
______________________________________________
```

A weak explanation means the boundary may be arbitrary.

---

# 84. Cohesion Boundary Test

Ask:

> Can I name this unit's central responsibility in one phrase?

Good:

```text
Manage order lifecycle.
Calculate tax.
Charge payments.
Persist orders.
Coordinate checkout.
```

Weak:

```text
Handle business logic.
Do order stuff.
Manage data.
```

---

# 85. "Because" Test

Complete:

> These methods belong together **because** ________.

Example:

```text
confirm
cancel
markPaid
```

Because:

```text
they manage the order state machine
```

That is strong evidence.

---

# 86. "And" Smell

Weak:

> This service validates orders **and** sends email **and** saves data.

Potentially several responsibilities.

A better decomposition may be:

```text
Use Case
    coordinates

Order
    validates

EmailSender
    sends

Repository
    saves
```

---

# 87. Scenario Slice

Trace:

```text
validate quantity
    → OrderLine

calculate line amount
    → OrderLine

reserve stock
    → Inventory

charge payment
    → PaymentGateway

persist sale
    → SaleRepository
```

The scenario exposes responsibility clusters.

---

# 88. Interaction Frequency

Frequent interaction may suggest cohesion, but frequency alone is insufficient.

Two components may communicate often because the architecture is badly decomposed.

Use:

```text
interaction
+
semantic relationship
+
change reason
```

---

# 89. Shared Data Is Not Enough

This is weak reasoning:

> Both methods use `customerId`, therefore they belong together.

Instead ask:

```text
Do they represent one coherent capability?
Do they share invariants?
Do they change together?
```

---

# 90. Shared Invariants Are Strong Evidence

If:

```text
quantity > 0
```

is protected by:

```text
setQuantity
increase
decrease
subtotal
```

they form a strong line-level cluster.

---

# 91. Shared Lifecycle Is Strong Evidence

If:

```text
OrderLine
    created inside order
    removed from order
    contributes to order total
```

those relationships support cohesion.

---

# 92. Shared Change Is Strong Evidence

If:

```text
tax formula changes
```

and:

```text
tax calculation
tax validation
tax policy selection
```

all change together, a tax responsibility cluster may make sense.

---

# 93. Volatility and Cohesion

Two responsibilities that vary independently are often candidates for separation.

Example:

```text
payment provider
tax rules
```

They both appear in checkout but may have different volatility.

---

# 94. Stability and Cohesion

Two responsibilities may be stable enough to remain together.

Do not split theoretically distinct concerns when the practical boundary is stable, clear, and cheap.

---

# 95. Cohesion and Complexity Budget

Each new boundary adds:

```text
names
files
interfaces
navigation
configuration
tests
documentation
```

Decomposition must earn that cost.

---

# 96. The Abstraction Budget

For every extraction, ask:

```text
What benefit do I receive?
```

Potential benefits:

```text
change isolation
testability
reuse
security
replaceability
clarity
```

Potential costs:

```text
indirection
configuration
cognitive load
runtime overhead
```

---

# 97. Cohesion and Performance

Decomposition can affect:

```text
allocation
calls
serialization
network boundaries
query patterns
caching
```

Do not assume a cleaner class graph is automatically faster.

Measure meaningful bottlenecks.

---

# 98. Cohesion and Memory

More objects can increase allocations.

For high-volume objects, compare:

```text
class instances
plain objects
arrays/maps
functions
```

A responsibility can be cohesive without requiring an object allocation per record.

---

# 99. Cohesion and Security

Cohesive capabilities often support narrow authority.

Example:

```text
PaymentGateway
```

instead of injecting:

```text
Database
```

This supports least authority.

---

# 100. Cohesion and Reliability

Separate technical responsibilities can isolate failure:

```text
PaymentGateway
    provider failure

Repository
    persistence failure

NotificationSender
    delivery failure
```

This can improve recovery strategy.

---

# 101. Cohesion and Transactions

A use case may be cohesive around one transaction workflow:

```text
TransferMoneyUseCase
```

while individual domain objects still own their local invariants.

---

# 102. Cohesion and Concurrency

Concurrency responsibilities can be separate:

```text
OptimisticLock
ReservationCoordinator
IdempotencyStore
```

Do not scatter locking logic throughout domain objects.

---

# 103. Cohesion and Distributed Systems

Distributed services should represent coherent capabilities.

```text
Ordering
Payments
Inventory
Reporting
```

But class-level cohesion alone does not justify creating a network service.

Network boundaries introduce:

```text
latency
failure
versioning
consistency
operations
```

---

# 104. Cohesion and Microservices

A cohesive module is not automatically a microservice.

Before creating a service boundary ask:

```text
Can it change independently?
Can it deploy independently?
Can it tolerate network failure?
Can data be consistent independently?
Is operational cost justified?
```

---

# 105. Cohesion and Bounded Contexts

A bounded context is a broader semantic boundary than a class.

Use it to keep:

```text
language
rules
ownership
models
```

cohesive.

---

# 106. Cohesion and API Boundaries

APIs should represent coherent capabilities.

Prefer:

```text
Orders API
Payments API
Inventory API
```

over a universal API that mixes unrelated operations.

---

# 107. Cohesion and Team Boundaries

Cohesive modules can reduce cross-team coordination.

```text
Ordering team
Payments team
Inventory team
```

Low coupling then reduces shared change surfaces.

---

# 108. Cohesion and Deployment Boundaries

Do not turn every cohesive module into a deployable service.

Evaluate:

```text
deployment cost
network cost
operational complexity
failure modes
```

---

# 109. Cohesion and CQRS

Read and write models can intentionally have different cohesion.

```text
Write model
    business rules

Read model
    query optimization
```

Do not force one model to own both responsibilities.

---

# 110. Cohesion and Event-Driven Design

Events can reveal capability boundaries:

```text
SaleFinalized
PaymentCaptured
InvoiceIssued
```

Group event producers and consumers by responsibility.

---

# 111. Cohesion and Event Payloads

An event payload should contain data relevant to its event meaning.

Avoid turning:

```text
SaleFinalized
```

into a dump of every internal field.

---

# 112. Cohesion and Idempotency

Idempotency can be a cohesive application responsibility:

```text
IdempotencyStore
```

paired with:

```text
Use Case
```

rather than embedded in every entity.

---

# 113. Cohesion and Retry

Retry can be a reusable technical capability:

```text
RetryPolicy
```

rather than duplicated in every network call.

---

# 114. Cohesion and Configuration

Configuration can become a blob.

Prefer meaningful configuration boundaries:

```text
PaymentConfig
PricingConfig
DatabaseConfig
```

when those configurations evolve independently.

---

# 115. Cohesion and Feature Flags

Feature flag evaluation can be cohesive:

```ts
interface FeatureFlagService {
  isEnabled(key: string, context: FlagContext): boolean;
}
```

Do not scatter provider-specific flag logic across domain objects.

---

# 116. Cohesion and Environment Access

Centralize:

```text
process.env
```

behind configuration boundaries when environment behavior needs validation or isolation.

---

# 117. Cohesion and Dependency Injection Containers

A DI container is a wiring mechanism.

It does not define business cohesion.

A domain class should not resolve itself from the container.

---

# 118. Cohesion and Reflection

Decorators and reflection can hide dependencies.

Review the effective responsibility graph rather than trusting framework metadata.

---

# 119. Cohesion and ORM Models

An ORM model may combine persistence and domain responsibilities.

That can work for simple systems.

For complex domains, separate mapping and domain behavior when the split provides meaningful cohesion.

---

# 120. Cohesion and Active Record

Active Record can be cohesive for simple CRUD:

```text
User
    save
    update
    delete
```

As domain rules become more complex, separating persistence may create clearer responsibility boundaries.

---

# 121. Cohesion and Repository Interfaces

Repository capabilities should stay focused.

Weak:

```ts
interface SystemRepository {
  saveOrder();
  saveCustomer();
  saveProduct();
  sendEmail();
}
```

Better:

```text
OrderRepository
CustomerRepository
ProductRepository
```

when concepts evolve independently.

---

# 122. Cohesion and Factory Interfaces

Factories should represent a meaningful creation family.

```text
PaymentMethodFactory
OrderFactory
```

A universal factory can become a dependency hub.

---

# 123. Cohesion and Adapter Interfaces

An adapter should translate one external capability.

```text
StripePaymentGateway
```

is cohesive around payment integration.

---

# 124. Cohesion and Mappers

Mappers should group related conversion responsibilities.

```text
OrderDtoMapper
PaymentDtoMapper
```

Avoid a universal mapper with every domain type.

---

# 125. Cohesion and Error Types

Error classes can represent cohesive semantic categories:

```text
OrderStateError
PaymentDeclinedError
InventoryUnavailableError
```

Do not build a giant error hierarchy only for decoration.

---

# 126. Cohesion and Audit Events

Business events can form cohesive audit categories:

```text
SALE_FINALIZED
PAYMENT_CAPTURED
DISCOUNT_APPROVED
```

This is more useful than one generic event when business semantics matter.

---

# 127. Cohesion and Logging

Structured logs should focus on meaningful fields for the capability.

Example:

```text
orderId
tenantId
branchId
paymentId
```

Avoid making every component log every possible field.

---

# 128. Cohesion and Metrics

Metrics should describe coherent operational concerns:

```text
checkout.duration
payment.failure.count
inventory.reserve.latency
```

---

# 129. Cohesion and Tracing

Trace spans can align with capability boundaries:

```text
checkout
inventory.reserve
payment.charge
order.save
```

---

# 130. Cohesion and Documentation

Documentation should mirror capabilities:

```text
orders/
payments/
inventory/
pricing/
```

Each should explain a coherent responsibility set.

---

# 131. Cohesion and Code Navigation

Good cohesion improves findability.

When debugging payment:

```text
payments/
```

should contain the core payment responsibility rather than forcing a search across unrelated utilities.

---

# 132. Cohesion and Naming

The name should describe the common responsibility.

Good:

```text
TaxPolicy
OrderRepository
CheckoutUseCase
```

Suspicious:

```text
Misc
Common
GeneralHelper
UniversalManager
```

---

# 133. Cohesion and Public Exports

Export stable capabilities.

Keep internal helpers private when possible.

This reduces the externally visible responsibility surface.

---

# 134. Cohesion and Versioning

A cohesive API often localizes breaking changes.

A giant general-purpose interface creates unrelated version pressure.

---

# 135. Cohesion and Migration

Migrate one responsibility cluster at a time:

```text
identify
    ↓
extract
    ↓
delegate
    ↓
test
    ↓
remove duplication
```

This is safer than rewriting a whole subsystem.

---

# 136. Cohesion and Branch-by-Abstraction

Introduce a cohesive boundary:

```text
PricingPolicy
├── OldPricing
└── NewPricing
```

Then switch implementations behind that boundary.

---

# 137. Cohesion and Backward Compatibility

A legacy facade can preserve old APIs:

```ts
class LegacyOrderService {
  confirm(order: Order) {
    return order.confirm();
  }
}
```

The responsibility can move without immediate external breakage.

---

# 138. Cohesion and Observability During Migration

Preserve:

```text
correlation IDs
key metrics
important logs
traces
```

while changing internals.

This helps verify safe migration.

---

# 139. Cohesion and Production Rollout

A cohesive boundary can support:

```text
feature flags
canary rollout
shadow execution
dual writes
```

where appropriate.

---

# 140. Over-Decomposition

Warning signs:

```text
tiny classes
one-method wrappers
long chains
excessive interfaces
navigation-heavy code
```

The design may be too fragmented.

---

# 141. Under-Decomposition

Warning signs:

```text
god object
service blob
controller blob
large public API
many unrelated imports
many change reasons
```

The unit may be too broad.

---

# 142. Goldilocks Boundary

The desired boundary is:

```text
meaningful enough to justify existence
small enough to avoid unrelated change
large enough to avoid fragmentation
```

There is no universal method count.

---

# 143. "Reason to Exist" Test

For every extracted object:

> Why does this object deserve to exist?

Good answers:

```text
It owns an invariant.
It isolates a provider variation.
It coordinates one use case.
It translates one external capability.
It represents a value concept.
```

Weak answer:

```text
I moved three methods out because the class was large.
```

---

# 144. Could This Be a Function?

Before creating a class:

```text
Does it need identity?
Does it own mutable state?
Does it have lifecycle?
Does it need polymorphism?
Does it manage long-lived collaborators?
```

If no:

```text
function
```

may be the better boundary.

---

# 145. Could This Be a Module?

Some cohesive capabilities can be module functions:

```ts
export function calculateTax(...) {}
export function validateTaxInput(...) {}
```

if they have no meaningful object identity or state.

---

# 146. Could This Be a Value Object?

If behavior centers on one value:

```text
Money
Quantity
Percentage
```

a value object may provide the strongest cohesion.

---

# 147. Could This Be a Policy?

If the responsibility is a changing decision:

```text
TaxPolicy
DiscountPolicy
ShippingPolicy
```

may be the best cohesion boundary.

---

# 148. Could This Be an Adapter?

If code primarily exists to translate external representations:

```text
Adapter
```

is a natural cohesive boundary.

---

# 149. Could This Be a Use Case?

If behavior coordinates one business outcome:

```text
CheckoutUseCase
FinalizeSaleUseCase
RefundPaymentUseCase
```

can form cohesive application boundaries.

---

# 150. Should This Stay Together?

Sometimes yes.

Example:

```text
Order.confirm
Order.cancel
Order.markPaid
```

can remain together because they form one state-machine responsibility.

Do not extract them simply because they are individually callable.

---

# 151. Method-by-Method Extraction Anti-Pattern

Weak:

```text
OrderConfirmService
OrderCancelService
OrderTotalService
```

when they all belong to one coherent order model.

This often creates fragmented responsibility.

---

# 152. Data-Based Extraction Anti-Pattern

Weak:

```text
CustomerFieldsService
OrderFieldsService
```

merely because each accesses certain fields.

Data access alone does not create semantic cohesion.

---

# 153. Framework-Shaped Extraction Anti-Pattern

Avoid:

```text
Service1
Service2
Service3
```

solely because the framework makes provider registration easy.

Framework structure should support responsibility, not replace it.

---

# 154. Utility Explosion Anti-Pattern

If every extraction becomes:

```text
Utils.ts
```

you may have transformed one blob into several smaller blobs.

Name the actual capability.

---

# 155. Interface Explosion Anti-Pattern

One interface per class can create artificial abstraction.

Prefer interfaces around:

```text
capabilities
variation points
architectural boundaries
```

---

# 156. Circular Dependency Risk

Over-decomposition can produce:

```text
A → B
B → C
C → A
```

Circular dependencies often indicate unclear responsibility ownership or bidirectional coupling.

Review ownership before adding more interfaces.

---

# 157. Shared Mutable State Risk

Splitting a class can accidentally produce multiple objects mutating the same shared state.

This may reduce cohesion and make invariants harder to protect.

Prefer clear state ownership.

---

# 158. Callback and Promise Fragmentation

Async over-decomposition can produce difficult collaboration graphs.

Avoid unnecessary:

```text
A → B → C → D
```

for trivial operations.

---

# 159. Distributed State Machine Anti-Pattern

If one lifecycle is spread across:

```text
OrderStateService
PaymentStateService
InvoiceStateService
```

without explicit authority, the model becomes hard to reason about.

Keep each state machine cohesive or clearly separated by ownership.

---

# 160. Generic Domain Utils Anti-Pattern

A `DomainUtils` module containing:

```text
tax
pricing
inventory
status
customer
```

often hides missing responsibility boundaries.

---

# 161. Cohesion and Code Review

Ask:

```text
What is this unit's central responsibility?
What else changes for the same reason?
What should not be here?
What state does it own?
Why are these dependencies needed?
```

Do not use line count as the only review criterion.

---

# 162. Cohesion Review Card

```text
Class:
____________________

Central responsibility:
____________________

Responsibilities:
____________________

Shared state:
____________________

Shared invariants:
____________________

Common change reasons:
____________________

Dependencies:
____________________

Candidate clusters:
____________________

Extraction benefit:
____________________

Extraction cost:
____________________
```

---

# 163. Cohesion Decision Tree

```text
Do responsibilities share a strong purpose?
        |
       yes
        ↓
Do they share state/invariants?
        |
       yes
        ↓
Keep together unless variation/coupling argues otherwise.

       no
        ↓
Do they only share data?
        |
       yes
        ↓
Check whether representation or policy should be separate.

Do they change independently?
        |
       yes
        ↓
Candidate for separation.

Would extraction create excessive coupling?
        |
       yes
        ↓
Reconsider.

       no
        ↓
Extract.
```

---

# 164. Cohesion and Encapsulation

Encapsulation asks:

```text
Who can access state?
```

Cohesion asks:

```text
Which responsibilities belong together?
```

Strong encapsulation cannot rescue a poorly cohesive class.

---

# 165. Cohesion and Abstraction

Abstraction asks:

```text
What should clients see?
```

Cohesion asks:

```text
What should the abstraction contain?
```

A clean interface can hide a poorly cohesive implementation.

---

# 166. Cohesion and Composition

Composition enables focused responsibilities:

```text
Order
├── PricingPolicy
├── TaxPolicy
└── PaymentGateway
```

Each collaborator can stay cohesive.

---

# 167. Cohesion and Inheritance

Inheritance can accidentally combine orthogonal responsibilities.

Ask whether the hierarchy represents one coherent variation axis.

If concerns vary independently, composition may produce better cohesion.

---

# 168. Cohesion and Polymorphism

Polymorphic implementations should each be cohesive around the same contract.

```text
DiscountPolicy
├── SeasonalDiscount
├── MemberDiscount
└── ClearanceDiscount
```

---

# 169. Cohesion and Protected Variations

A variation boundary should itself have a focused responsibility.

```text
PaymentGateway
```

should not also become:

```text
reports
email
customer management
tax
```

---

# 170. Cohesion and Indirection

An intermediary should have a reason to exist.

```text
StripeAdapter
```

is cohesive around provider translation.

A universal adapter is usually suspicious.

---

# 171. Cohesion and Pure Fabrication

A fabricated component should give one technical responsibility a meaningful home.

Example:

```text
AuditLogger
```

not:

```text
UniversalUtility
```

---

# 172. Cohesion and GRASP

The central relationship:

```text
Information Expert
    → candidate owner

High Cohesion
    → evaluate fit

Low Coupling
    → evaluate dependency cost
```

This is why cohesion belongs directly beside GRASP.

---

# 173. Cohesion and SOLID

Cohesion supports:

```text
SRP
ISP
OCP
DIP
```

but applies beyond SOLID to:

```text
modules
packages
services
teams
bounded contexts
```

---

# 174. Cohesion and DDD

DDD concepts often represent cohesive boundaries:

```text
Entity
Value Object
Aggregate
Policy
Domain Service
Repository
```

Do not adopt the names without preserving the responsibility meaning.

---

# 175. Cohesion and Event Storming

Events can expose domain capabilities:

```text
SaleCreated
PaymentCaptured
SaleFinalized
InvoiceIssued
```

Cluster events and commands around meaningful capabilities.

---

# 176. Cohesion and Commands

Commands represent requested behavior:

```text
FinalizeSaleCommand
ApproveDiscountCommand
ReserveInventoryCommand
```

Keep handlers cohesive around related application capabilities.

---

# 177. Cohesion and Queries

Read responsibilities can form separate capabilities:

```text
SalesDashboardQuery
CustomerBalanceQuery
InventoryAvailabilityQuery
```

Their cohesion can legitimately differ from write-side domain objects.

---

# 178. Cohesion and Reuse

Cohesive components may be easier to reuse.

But do not design abstractions solely for hypothetical reuse.

---

# 179. Cohesion and DRY

DRY prevents duplicated knowledge.

It does not mean:

```text
one universal shared class
```

Two similar components may intentionally remain separate when their rules change independently.

---

# 180. Cohesion and Shared Libraries

A shared library should also be cohesive.

Avoid:

```text
company-utils
    payment
    database
    email
    tax
    image
```

Prefer meaningful stable libraries.

---

# 181. Cohesion and Monorepos

A monorepo can align packages with capabilities:

```text
packages/
  ordering/
  payments/
  inventory/
  pricing/
```

This makes ownership visible.

---

# 182. Cohesion and Distributed Data

A cohesive service may own its own data.

Avoid creating one universal schema model that every service shares merely for convenience.

---

# 183. Cohesion and Read Models

Read models can intentionally duplicate data.

```text
Order
    source of business truth

OrderDashboard
    optimized representation
```

Different responsibility, different cohesion.

---

# 184. Cohesion and Persistence Mapping

Mapping can be cohesive around:

```text
domain ↔ persistence representation
```

Example:

```text
OrderMapper
```

Do not spread ORM mapping logic throughout domain methods.

---

# 185. Cohesion and External Model Translation

Adapters can group:

```text
translation
error mapping
provider-specific representation
```

This keeps external model assumptions in one place.

---

# 186. Cohesion and Security Policies

Security policies often deserve focused boundaries:

```text
DiscountApprovalPolicy
TenantAccessPolicy
PasswordPolicy
```

The business rule should not be scattered.

---

# 187. Cohesion and Multi-Tenancy

Tenant behavior spans multiple layers:

```text
TenantContext
    request/application context

TenantPolicy
    business authorization

TenantScopedRepository
    persistence enforcement
```

These are related but not identical responsibilities.

---

# 188. Jewellery ERP — Decomposition

Suppose one class contains:

```text
calculateMetalPrice
calculateMakingCharge
calculateTax
reserveStock
takePayment
issueInvoice
sendSms
generateReport
```

Cluster:

```text
pricing
tax
inventory
payment
invoice
notification
reporting
```

Then decide which boundaries are actually worthwhile.

---

# 189. Jewellery ERP — Pricing Cohesion

A pricing capability may contain:

```text
metal price source
making charge
pricing mode
rounding rules
```

Potential boundary:

```ts
interface PricingPolicy {
  calculate(input: PricingInput): Money;
}
```

---

# 190. Jewellery ERP — Inventory Cohesion

Inventory may own:

```text
availability
reservation
release
stock identity
```

The central capability is stock management.

---

# 191. Jewellery ERP — Payment Cohesion

A payment capability may contain:

```text
charge
refund
capture
status mapping
```

Whether every operation belongs in one interface depends on consumer needs and provider semantics.

---

# 192. Jewellery ERP — Invoice Cohesion

Invoice may own:

```text
invoice lifecycle
totals
approval
```

PDF generation can remain separate because representation is another change axis.

---

# 193. Jewellery ERP — Audit Cohesion

```text
AuditLogger
    record
```

is cohesive around audit recording.

The storage mechanism can vary.

---

# 194. Jewellery ERP — Tenant Cohesion

Tenant-related responsibilities:

```text
tenant identity
tenant status
tenant configuration
tenant-level invariants
```

should not automatically swallow all tenant-aware operations.

---

# 195. Jewellery ERP — Branch Cohesion

Branch may own:

```text
identity
status
branch rules
```

A sale does not become a Branch responsibility simply because it occurs at a branch.

---

# 196. Jewellery ERP — Multi-Tenant Cohesion

Tenant isolation may be layered:

```text
request context
    ↓
authorization
    ↓
use-case scope
    ↓
repository scope
    ↓
database constraint
```

This is broader than class-level cohesion.

---

# 197. Jewellery ERP — Finalize Sale Use Case

A cohesive use case can coordinate:

```text
load sale
validate application preconditions
finalize sale
reserve stock
charge payment
persist
audit
```

Its center is:

```text
finalize sale workflow
```

---

# 198. Jewellery ERP — Transaction Reality

Even if checkout is cohesive as one workflow, external payment may not share the same database transaction.

That introduces:

```text
failure
retry
idempotency
reconciliation
```

The workflow boundary remains cohesive while consistency is designed separately.

---

# 199. Jewellery ERP — Reconciliation

If:

```text
payment succeeds
```

but:

```text
local persistence fails
```

a reconciliation process may be needed:

```text
PaymentReconciliationJob
```

This is a separate operational responsibility.

---

# 200. Jewellery ERP — Read Model

A dashboard might use:

```text
SalesDashboardReadModel
```

rather than forcing `Sale` to own every reporting query.

---

# 201. Implementation Exercise — Cohesive Order

Implement:

```ts
class Order {
  addLine(line: OrderLine): void;
  removeLine(id: string): void;
  total(): Money;
  confirm(): void;
  cancel(): void;
}
```

Then implement:

```ts
interface PaymentGateway {
  charge(input: ChargeInput): Promise<ChargeResult>;
}

interface OrderRepository {
  findById(id: string): Promise<Order | null>;
  save(order: Order): Promise<void>;
}
```

Explain which responsibilities are cohesive inside `Order` and which are external.

---

# 202. Implementation Exercise — Pricing

Implement:

```ts
interface PricingPolicy {
  calculate(input: PricingInput): Money;
}
```

Provide:

```text
PieceBasedPricing
WeightBasedPricing
```

Then evaluate whether the abstraction improves cohesion enough to justify its complexity.

---

# 203. Implementation Exercise — Repository

Implement:

```ts
interface OrderRepository {
  findById(id: string): Promise<Order | null>;
  save(order: Order): Promise<void>;
}
```

Then:

```text
InMemoryOrderRepository
PostgresOrderRepository
```

Check whether storage responsibilities remain cohesive.

---

# 204. Implementation Exercise — Factory

Create:

```text
OrderFactory
```

with:

```text
Clock
IdGenerator
```

Verify that construction responsibility remains separate from lifecycle behavior.

---

# 205. Implementation Exercise — Policy

Implement:

```ts
interface TaxPolicy {
  calculate(input: TaxInput): Money;
}
```

Provide two jurisdictions and compare:

```text
cohesion
variation
testing
configuration
```

---

# 206. Code Review Exercise

Review:

```ts
class BusinessService {
  async createSale() {}
  async approveDiscount() {}
  async sendInvoiceEmail() {}
  async reserveInventory() {}
  async exportReport() {}
  async processPayment() {}
}
```

Identify:

```text
cohesive clusters
state ownership
change reasons
technical boundaries
target owners
```

---

# 207. Code Review Exercise — Entity

Review:

```ts
class Order {
  confirm() {}
  cancel() {}
  total() {}
  sendEmail() {}
  toPdf() {}
  save() {}
  charge() {}
}
```

Identify:

```text
order-domain cohesion
persistence cohesion
payment cohesion
notification cohesion
representation cohesion
```

---

# 208. Refactoring Exercise — Giant Service

Given:

```text
EcommerceManager
    createUser
    resetPassword
    placeOrder
    calculateTax
    chargePayment
    reserveStock
    exportInvoice
    sendSms
```

Produce:

```text
responsibility inventory
cohesion clusters
target boundaries
dependency graph
migration plan
tests
```

---

# 209. Refactoring Exercise — Controller Blob

Before:

```ts
class OrderController {
  async finalize(req) {
    // validation
    // pricing
    // inventory
    // payment
    // save
    // email
  }
}
```

Redesign:

```text
Controller
    transport

FinalizeOrderUseCase
    coordination

Order
    domain behavior

Inventory
    stock capability

PaymentGateway
    payment capability

NotificationSender
    delivery
```

---

# 210. Refactoring Exercise — Anemic Model

Before:

```ts
class Order {
  status!: string;
  items!: OrderLine[];
}
```

and:

```text
OrderService.confirm(order)
OrderService.cancel(order)
OrderService.total(order)
```

Evaluate which rules should move closer to `Order`.

Do not move integration concerns into the entity.

---

# 211. Refactoring Exercise — Feature Envy

Given:

```ts
class DiscountService {
  calculate(order: Order) {
    return order.items.reduce((total, item) => {
      if (item.product.category === "GOLD") {
        return total + item.price * item.quantity * 0.10;
      }

      return total;
    }, 0);
  }
}
```

Ask:

```text
Who owns category?
Who owns line amount?
Who owns discount policy?
Which concern varies?
```

Then redesign by meaningful cohesion.

---

# 212. Track A — Core Theory Retrieval

Explain without notes:

```text
What is cohesion?
Why is functional cohesion strong?
Why is coincidental cohesion weak?
How does change reason affect cohesion?
How do invariants affect cohesion?
Why can a large class still be cohesive?
Why can tiny classes still be incoherent?
```

---

# 213. Track B — Guided Implementation

Given a responsibility table, implement:

```text
Order
OrderLine
PricingPolicy
PaymentGateway
OrderRepository
CheckoutUseCase
```

Then map each method to its cohesion rationale.

---

# 214. Track B — No-Reference Challenge

Given only:

```text
Customer buys jewellery from a branch.
```

Design:

```text
sale
sale lines
pricing
tax
inventory
payment
invoice
audit
```

First produce:

```text
responsibility clusters
state ownership
change reasons
```

Then code.

---

# 215. Track C — Interview Reasoning

Question:

> "How do you decide whether to split a class?"

Strong answer:

> "I do not use line count as the primary criterion. I inventory responsibilities, cluster those with strong semantic and invariant relationships, inspect change reasons, and then evaluate the coupling and complexity created by extraction. I split when separate responsibility centers genuinely exist."

---

# 216. Track C — Advanced Follow-Up

Question:

> "What if extraction improves cohesion but creates many dependencies?"

Answer:

> "Then I evaluate the trade-off. High cohesion is not sufficient if the extraction creates excessive coupling, indirection, or operational complexity. I may keep the responsibilities together, introduce a narrower capability, or revisit the boundary."

---

# 217. Track C — Principal Follow-Up

Question:

> "Would you always extract tax from Order?"

Answer:

> "No. If tax is a simple stable calculation using order-owned information, keeping it near the order may preserve cohesion. If tax varies by jurisdiction, provider, or regulation and changes independently, a tax policy or domain service can provide a better boundary. I would justify the decision with knowledge, invariants, volatility, coupling, and change scenarios."

---

# 218. Predict-the-Design

### Scenario 1

```text
confirm
cancel
markPaid
```

Likely:

```text
Order
```

because they share lifecycle state.

### Scenario 2

```text
sendEmail
renderPdf
save
```

Likely:

```text
separate cohesive capabilities
```

because delivery, representation, and persistence have different change reasons.

### Scenario 3

```text
addLine
removeLine
total
```

Likely:

```text
Order
```

if it owns the line collection.

### Scenario 4

```text
calculateTax
validateTaxJurisdiction
```

Potentially:

```text
TaxPolicy
```

when both support the same tax capability.

### Scenario 5

```text
charge
refund
```

May remain together when they form one payment capability contract.

---

# 219. Debugging Cohesion Problems

Symptoms:

```text
duplicated rules
    → responsibility fragmented

giant service
    → low cohesion

long switch
    → possible variation cluster

provider SDK everywhere
    → missing integration boundary

controller with business logic
    → mixed responsibilities

entity with infrastructure mocks
    → domain/technical cohesion failure
```

Use these as signals, not automatic verdicts.

---

# 220. Debugging Procedure

```text
1. Reproduce the behavior.
2. Identify the failing responsibility.
3. Identify current owner.
4. Inventory nearby responsibilities.
5. Group by semantic purpose.
6. Inspect state and invariants.
7. Inspect change reasons.
8. Evaluate candidate extraction.
9. Check coupling.
10. Refactor incrementally.
11. Add regression tests.
```

---

# 221. Mastery Exercise — Library

Design:

```text
Member
Book
BookCopy
Loan
LateFeePolicy
NotificationSender
LoanRepository
BorrowBookUseCase
```

Requirements:

```text
member has borrowing limit
copy may be unavailable
loan duration varies by membership type
late fee varies by policy
history is retained
notifications may be sent
```

Produce:

```text
responsibility clusters
cohesion explanations
CRC cards
interaction diagram
invariants
TypeScript implementation
tests
```

---

# 222. Mastery Exercise — Jewellery ERP

Design:

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
AuditEntry
```

Scenario:

```text
Finalize a sale in Branch A for Tenant T1.
Pricing may be weight-based or piece-based.
Tax varies by jurisdiction.
Stock must be reserved.
Payment may be cash, card or UPI.
Invoice must be finalized.
Audit is mandatory.
```

Produce:

```text
1. responsibility clusters
2. state ownership
3. invariants
4. change reasons
5. target objects/modules
6. interfaces
7. dependency graph
8. tests
9. migration plan
```

---

# 223. Mastery Gate

Use:

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

You have mastered cohesion-first decomposition when you can:

```text
avoid god objects
avoid one-class-per-method fragmentation
identify semantic clusters
justify boundaries
measure extraction cost
```

---

# 224. Principal Decision Framework

Evaluate every decomposition using:

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
What became more cohesive?
What became more coupled?
What variation became safer?
What complexity appeared?
What complexity disappeared?
```

---

# 225. Cohesion Trade-Off Matrix

| Decision | Benefit | Risk |
|---|---|---|
| Extract policy | isolates variation | extra indirection |
| Extract adapter | provider isolation | extra boundary |
| Extract renderer | representation cohesion | more files |
| Keep entity rich | invariant locality | possible overload |
| Keep workflow together | simple orchestration | service blob risk |
| Merge tiny classes | fewer moving parts | less explicit separation |
| Split module | clear capability | navigation overhead |

---

# 226. Completion Criteria

```text
[ ] I can define cohesion.
[ ] I can explain seven common cohesion categories.
[ ] I can identify functional cohesion.
[ ] I can detect coincidental cohesion.
[ ] I can cluster responsibilities.
[ ] I can use state and invariants as cohesion evidence.
[ ] I can use change reasons as cohesion evidence.
[ ] I can distinguish semantic cohesion from shared data.
[ ] I can detect god objects.
[ ] I can detect service blobs.
[ ] I can detect feature envy.
[ ] I can detect shotgun surgery.
[ ] I can detect divergent change.
[ ] I can avoid over-decomposition.
[ ] I can reason at class/module/service/context levels.
[ ] I can apply cohesion to JavaScript modules.
[ ] I can apply cohesion to TypeScript interfaces.
[ ] I can choose function/value object/policy/object boundaries.
[ ] I can evaluate performance and memory trade-offs.
[ ] I can evaluate security and authority boundaries.
[ ] I can apply cohesion to jewellery ERP.
[ ] I can defend the decomposition in an LLD interview.
```

---

# 227. Revision / Retrieval Record

```text
Date:
________________

Unit reviewed:
________________

Central responsibility:
________________

Cohesion clusters:
________________

Weakest boundary:
________________

Potential over-decomposition:
________________

Potential under-decomposition:
________________

Change scenario:
________________

Final design:
________________
```

---

# 228. Canonical References and Source Discipline

### Object-Oriented Design

Use established software engineering and object-oriented design literature for formal cohesion terminology and responsibility-oriented decomposition. Craig Larman's responsibility-driven treatment is especially relevant to this curriculum.

### ECMAScript

For language semantics:

https://tc39.es/ecma262/

Use the ECMAScript specification for:

```text
objects
functions
classes
modules
property semantics
language-level guarantees
```

### TypeScript

Official documentation:

https://www.typescriptlang.org/docs/

Use it for:

```text
interfaces
structural typing
type-level design
module behavior
compiler semantics
```

### Runtime-Specific Claims

For:

```text
V8 hidden classes
inline caches
deoptimization
allocation behavior
```

use engine documentation and clearly distinguish implementation details from language guarantees.

### Architectural Claims

For:

```text
entities
value objects
aggregates
application services
domain services
repositories
bounded contexts
```

treat the material as architecture/domain-design guidance, not ECMAScript semantics.

---

# 229. Source Discipline Rules

Use:

```text
specification
    → language claim

engine/runtime documentation
    → implementation claim

architecture literature
    → design guidance

project requirements
    → project-specific decision
```

Do not treat framework conventions as proof of cohesion.

---

# 230. Concept Connections

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
Design Patterns
    ↓
DDD
    ↓
Persistence
    ↓
Transactions
    ↓
Concurrency
    ↓
Resilience
    ↓
Enterprise LLD
```

Core progression:

```text
responsibilities
    ↓
clusters
    ↓
cohesive boundaries
    ↓
coupling review
    ↓
scenario validation
```

---

# 231. Final Mental Model

When you see a large component, do not ask first:

```text
"How can I make it smaller?"
```

Ask:

```text
What does it fundamentally own?
        ↓
What responsibilities does it contain?
        ↓
Which responsibilities share state?
        ↓
Which share invariants?
        ↓
Which serve one business purpose?
        ↓
Which change together?
        ↓
Which vary independently?
        ↓
Which belong to infrastructure?
        ↓
Which belong to workflows?
        ↓
Would extraction improve semantic cohesion?
        ↓
What coupling would extraction create?
        ↓
What complexity would remain?
        ↓
What is the simplest stable design?
```

---

# 232. Final Design Principle

> **Decompose when responsibilities form distinct, meaningful clusters with different ownership, invariants, or change reasons; keep them together when they share a strong semantic center and extraction would mainly add indirection or coupling.**

This is the cohesion-first rule.

---

# 233. Chapter Completion Snapshot

```text
Chapter: 15
Title: Cohesion-First Object Decomposition

Theory:
[+] Cohesion
[+] Coincidental cohesion
[+] Logical cohesion
[+] Temporal cohesion
[+] Procedural cohesion
[+] Communicational cohesion
[+] Sequential cohesion
[+] Functional cohesion
[+] Change cohesion
[+] State/invariant cohesion

Design:
[+] Responsibility clustering
[+] Change-scenario decomposition
[+] CRC support
[+] Module cohesion
[+] Capability interfaces
[+] Policy boundaries
[+] Workflow boundaries
[+] Integration boundaries
[+] Representation boundaries
[+] Refactoring

JavaScript / TypeScript:
[+] Functions
[+] Closures
[+] Classes
[+] Private state
[+] Modules
[+] Interfaces
[+] Structural typing
[+] Value objects
[+] Capability-oriented APIs

Production:
[+] Security
[+] Least authority
[+] Performance
[+] Memory
[+] Reliability
[+] Observability
[+] Transactions
[+] Concurrency
[+] Distributed systems
[+] Multi-tenancy
[+] Migration

Interview:
[+] Question bank
[+] Predict-the-design
[+] Code review
[+] Jewellery ERP exercise
[+] Principal-level trade-offs
```

# Chapter 15 — Completion Statement

Cohesion-first decomposition is the discipline of finding meaningful responsibility clusters before creating boundaries. The objective is not minimum class size. The objective is a strong semantic center, clear state and invariant ownership, understandable change reasons, appropriate technical boundaries, and an acceptable collaboration graph.

Next chapter: **Coupling-First Dependency Design**.
