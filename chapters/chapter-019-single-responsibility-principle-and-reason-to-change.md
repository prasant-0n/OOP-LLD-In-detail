# Chapter 19 — Single Responsibility Principle and Reason to Change

> **Status:** `[~] In Progress`  
> **Part:** D — Core Design Principles  
> **Primary theme:** Single Responsibility Principle (SRP), responsibility boundaries, and reasons to change  
> **Tracks:** Track A — Core Theory · Track B — Implementation · Track C — Interview / Reasoning

---

# Chapter 19 — Learning Objectives

By the end of this chapter, you should be able to:

1. Define the Single Responsibility Principle precisely.
2. Explain why “one job” is too vague to be a useful SRP definition.
3. Use **reason to change** as a stronger guide for responsibility boundaries.
4. Distinguish responsibility, behavior, role, and implementation detail.
5. Explain why SRP is about **cohesion and change coupling**, not class size.
6. Detect SRP violations in classes, modules, services, controllers, repositories, and functions.
7. Distinguish true responsibility clustering from accidental similarity.
8. Recognize when a class contains multiple **actors**, **stakeholders**, or **change authorities**.
9. Refactor responsibility-heavy objects without creating unnecessary fragmentation.
10. Apply SRP to JavaScript and TypeScript production systems.
11. Connect SRP with cohesion, coupling, information hiding, abstraction, composition, and protected variations.
12. Understand SRP trade-offs and avoid over-splitting.
13. Reason about SRP in distributed systems, events, databases, security, and observability.
14. Defend SRP decisions in an interview or design review.
15. Apply SRP to a jewellery ERP domain using real operational responsibilities.

---

# Chapter 19 — Prerequisites

You should already understand:

```text
objects
classes
encapsulation
abstraction
composition
delegation
responsibility-driven design
GRASP
cohesion
coupling
contracts
invariants
refactoring
```

Most importantly, you should understand the earlier distinction:

```text
responsibility ≠ method count
responsibility ≠ class count
responsibility = meaningful role/accountability
```

---

# 1. What Is SRP?

The **Single Responsibility Principle** says:

> A module should have one reason to change.

This is stronger than:

> A class should do only one thing.

The “one thing” phrasing is often too simplistic because a single meaningful responsibility may require many operations.

A tax policy may:

```text
calculate tax
validate taxable items
select rate
round tax
explain adjustments
```

Those behaviors can still belong to one cohesive responsibility.

---

# 2. Why the Principle Exists

Software changes because something in the surrounding world changes.

Examples:

```text
tax law changes
database schema changes
API contract changes
authorization policy changes
UI workflow changes
logging requirements change
pricing rules change
```

If one object contains behavior owned by multiple change sources, unrelated changes become coupled.

That creates:

```text
change A
   ↓
same class
   ↓
accidental impact on B
```

SRP attempts to reduce that change coupling.

---

# 3. The Core Mental Model

Think:

```text
Responsibility
    ↓
owner / stakeholder
    ↓
reason to change
    ↓
kind of variation
```

A strong object boundary often groups behavior that changes together.

A weak boundary groups behavior merely because it happens to be used together today.

---

# 4. SRP Is About Change Coupling

Suppose:

```ts
class OrderService {
  createOrder() {}
  calculateTax() {}
  renderInvoiceHtml() {}
  sendEmail() {}
}
```

These methods may be related operationally.

But their change sources differ:

```text
order workflow
tax policy
presentation format
notification policy
```

The class has high change coupling.

---

# 5. SRP Is Not “One Method Per Class”

This is bad:

```ts
class TaxCalculator {
  calculate() {}
}

class TaxValidator {
  validate() {}
}

class TaxRounder {
  round() {}
}
```

These may all belong to one stable responsibility.

Splitting every behavior into a class creates:

```text
more objects
more wiring
more indirection
more cognitive overhead
```

SRP is not a weapon against cohesive complexity.

---

# 6. Responsibility vs Task

A task:

```text
calculateTax()
```

A responsibility:

```text
determine and apply the tax policy for a sale
```

A responsibility can span many tasks.

Design around semantic accountability, not individual verbs.

---

# 7. Responsibility vs Role

A role is a meaningful position in collaboration.

Example:

```text
PricingPolicy
```

may have one role:

```text
determine valid selling price
```

It can internally perform many calculations.

---

# 8. Responsibility vs Implementation Detail

These are implementation details:

```text
uses Map
uses SQL
uses Redis
uses regex
uses fetch
```

They should not automatically define separate responsibilities.

A module may use many mechanisms while retaining one coherent responsibility.

---

# 9. Reason to Change

Ask:

> Who or what would request a change to this code?

Potential answer:

```text
Finance
Security
Product
Operations
Compliance
Database team
UI team
External provider
```

If several independent authorities can request unrelated changes to the same module, SRP risk increases.

---

# 10. The Actor Test

Identify the actors that can cause requirements to change.

Example:

```text
InvoiceGenerator
```

may respond to:

```text
Accounting rules
Design/branding requests
Email delivery requirements
```

These are different actors.

The class may be carrying multiple responsibilities.

---

# 11. Actor ≠ End User

An actor in SRP reasoning is not necessarily a person clicking a screen.

It can be:

```text
business function
team
department
policy owner
external system
regulator
technical concern
```

The question is:

> Which source of change owns this behavior?

---

# 12. Stakeholder Test

Ask:

```text
Which stakeholder cares about this rule?
```

If:

```text
Finance
```

owns tax calculation and:

```text
Marketing
```

owns invoice branding, these changes should not necessarily be coupled.

---

# 13. Change-Axis Test

Group behavior that varies along the same axis.

For example:

```text
tax rules
```

vary when:

```text
tax policy changes
```

while:

```text
HTML rendering
```

varies when:

```text
presentation changes
```

Separate axes are warning signs.

---

# 14. Volatility Test

If two behaviors change at different frequencies:

```text
tax logic: monthly
PDF layout: yearly
```

frequency alone is not proof of separate responsibility, but it can reveal different change drivers.

Use volatility as evidence, not as the only rule.

---

# 15. Requirement Ownership Test

Imagine receiving this list:

```text
Finance asks for new tax behavior.
Operations asks for new stock behavior.
Security asks for a new permission rule.
```

If all changes require editing:

```ts
GodService
```

you likely have responsibility accumulation.

---

# 16. Cohesion Connection

SRP is strongly connected to cohesion.

High cohesion:

```text
behaviors support one meaningful responsibility
```

Low cohesion:

```text
behaviors happen to coexist
```

SRP asks why they belong together.

Cohesion asks how strongly they belong together.

---

# 17. Coupling Connection

When several unrelated responsibilities share one module:

```text
tax change
→ module changed
→ unrelated callers revalidate
```

This creates coupling.

SRP therefore often improves change isolation.

---

# 18. SRP and Change Amplification

If one requirement change requires:

```text
5 unrelated edits
```

because they share one object, change amplification is high.

A useful goal:

```text
one business change
→ mostly one responsibility boundary
```

Not necessarily exactly one file.

---

# 19. SRP and Stability

A stable abstraction often groups things that change for the same reason.

If a module combines:

```text
volatile business rule
+
volatile external protocol
```

its contract and implementation can both churn.

Separating change sources can stabilize dependencies.

---

# 20. SRP and Information Hiding

A responsibility boundary should hide information that primarily matters to that responsibility.

For a pricing policy:

```text
tax tables
discount algorithms
rounding rules
```

can remain internal.

Other modules should not need to know them.

---

# 21. SRP and Encapsulation

Encapsulation prevents arbitrary callers from manipulating internal state.

SRP decides:

```text
which state and behavior belong together
```

Together:

```text
SRP
  defines responsibility boundary

encapsulation
  protects that boundary
```

---

# 22. SRP and Abstraction

An abstraction should represent a coherent responsibility.

Bad:

```ts
interface CompanyEverything {
  calculateTax(): number;
  saveUser(): Promise<void>;
  sendEmail(): Promise<void>;
  generatePdf(): Buffer;
}
```

Better:

```ts
interface TaxPolicy {}
interface UserRepository {}
interface Mailer {}
interface InvoiceRenderer {}
```

---

# 23. SRP and Composition

Composition lets one use case combine multiple responsibilities:

```text
OrderApplicationService
  uses
    PricingPolicy
    InventoryPolicy
    TaxPolicy
    OrderRepository
    EventPublisher
```

The application service coordinates.

It does not need to own all domain decisions.

---

# 24. SRP and Delegation

Delegation is useful when a class is becoming a coordinator for responsibilities it should not own.

```ts
class CheckoutService {
  constructor(
    private readonly pricing: PricingPolicy,
    private readonly inventory: InventoryService,
    private readonly payments: PaymentGateway,
  ) {}
}
```

The service still has a coherent application responsibility:

```text
orchestrate checkout
```

---

# 25. SRP and GRASP

GRASP asks:

```text
who should have this responsibility?
```

SRP asks:

```text
does this responsibility grouping create one coherent reason to change?
```

They reinforce each other.

---

# 26. SRP and Information Expert

Information Expert suggests assigning behavior to the object with the information needed.

SRP adds a second question:

```text
Does this new behavior introduce another independent change reason?
```

Information ownership and change cohesion should both be considered.

---

# 27. SRP and Pure Fabrication

Pure Fabrication can create technical objects such as:

```text
Repository
Mapper
Serializer
```

These are acceptable when the technical responsibility is coherent.

Do not create them merely to satisfy “one method per class.”

---

# 28. SRP and Indirection

Indirection can separate change sources:

```text
application
  ↓
stable interface
  ↓
volatile implementation
```

The point is not “more abstractions.”

The point is reducing change coupling where variation matters.

---

# 29. SRP and Protected Variations

If a requirement is likely to vary:

```text
tax policy
payment provider
shipping calculation
```

a boundary can protect the rest of the system from that variation.

SRP helps identify which variation belongs behind which responsibility.

---

# 30. Example — User Service

Bad:

```ts
class UserService {
  registerUser() {}
  hashPassword() {}
  sendWelcomeEmail() {}
  renderWelcomeEmailHtml() {}
  saveUser() {}
  generateJwt() {}
}
```

Potential responsibilities:

```text
registration workflow
password hashing
notification
email presentation
persistence
token issuance
```

The class likely has multiple change authorities.

---

# 31. Better Decomposition

```ts
class RegistrationService {
  constructor(
    private readonly users: UserRepository,
    private readonly passwords: PasswordHasher,
    private readonly mailer: Mailer,
  ) {}

  async register(command: RegisterUser) {
    // coordinate registration
  }
}
```

Specialized collaborators own focused policies.

---

# 32. Is RegistrationService SRP-Compliant?

It can be.

Its responsibility may be:

```text
coordinate user registration
```

It can call:

```text
repository
password hasher
mailer
```

without owning their internal responsibilities.

This is why method count is not enough.

---

# 33. Example — Controller Blob

Bad controller:

```ts
class OrderController {
  create() {
    // validate request
    // authorize user
    // calculate price
    // update inventory
    // save order
    // send event
    // render response
  }
}
```

The controller is overloaded.

Potential responsibilities:

```text
transport
authorization
domain decision
persistence
integration
presentation
```

---

# 34. Better Controller Boundary

```ts
class OrderController {
  constructor(
    private readonly createOrder: CreateOrderUseCase
  ) {}

  async post(req: Request) {
    const command = mapRequest(req);
    const result = await this.createOrder.execute(command);
    return mapResponse(result);
  }
}
```

The controller has a transport responsibility.

---

# 35. Example — Repository Blob

Bad:

```ts
class UserRepository {
  findUser() {}
  calculateCreditScore() {}
  sendWelcomeEmail() {}
  renderAvatar() {}
}
```

Persistence responsibility has been mixed with unrelated concerns.

---

# 36. Repository Responsibility

A repository may reasonably own:

```text
persist and retrieve aggregate state
```

It may also contain:

```text
mapping between persistence representation and domain representation
```

if that mapping is part of its coherent responsibility.

---

# 37. Mapping and SRP

This is often reasonable:

```ts
class UserRepository {
  private toDomain(row: UserRow): User {}
  private toRow(user: User): UserRow {}
}
```

Both serve:

```text
persistence translation
```

Do not split methods solely because they are different verbs.

---

# 38. Example — Invoice Class

Suppose:

```ts
class Invoice {
  calculateTotal() {}
  addLine() {}
  removeLine() {}
  formatForPdf() {}
  saveToDatabase() {}
}
```

Domain responsibility:

```text
invoice state and business behavior
```

Presentation and persistence are separate change axes.

---

# 39. Domain Model Can Be Rich

SRP does not mean anemia.

A domain object can contain substantial behavior:

```text
addLine
removeLine
calculateTotal
applyDiscount
validate
```

if those operations belong to the invoice responsibility.

---

# 40. SRP and Anemic Models

Over-applying SRP can create:

```text
InvoiceData
InvoiceCalculator
InvoiceValidator
InvoiceLineManager
InvoiceDiscountService
InvoiceTotalService
```

where the domain model becomes empty.

This may reduce cohesion rather than improve it.

---

# 41. The Fragmentation Trap

A class can have:

```text
10 highly cohesive methods
```

and be healthy.

Creating:

```text
10 classes
```

does not automatically make it better.

The question is:

```text
Did we separate independent change reasons?
```

---

# 42. Method Cohesion

Methods are cohesive when they operate on:

```text
same state
same invariants
same domain concept
same change drivers
```

That is a stronger reason to keep them together.

---

# 43. Field Cohesion

Fields reveal responsibility boundaries.

If a class has:

```ts
orderLines
taxRate
smtpClient
redisClient
pdfTemplate
jwtSecret
```

this is a warning.

The data dependencies already expose multiple concerns.

---

# 44. Dependency Cohesion

Look at injected dependencies.

```ts
constructor(
  private db: Database,
  private mailer: Mailer,
  private pdf: PdfRenderer,
  private payment: PaymentGateway,
  private logger: Logger,
)
```

A broad dependency set can indicate responsibility accumulation.

But a coordinator may legitimately use several collaborators.

The key is the semantic role of the class.

---

# 45. Constructor Smell

A constructor with 15 dependencies is not automatically an SRP violation.

A workflow orchestrator may legitimately depend on many specialized components.

Ask:

```text
Are these dependencies used to perform one coherent use-case responsibility?
```

If yes, the class may still be cohesive.

---

# 46. Branching as SRP Evidence

A class containing:

```ts
if (provider === "stripe") ...
if (channel === "email") ...
if (channel === "sms") ...
if (role === "admin") ...
```

may be mixing multiple policy axes.

Not every conditional is an SRP violation.

But unrelated axes are a strong signal.

---

# 47. Parallel Change Test

Imagine requirements arriving in parallel:

```text
Tax team changes tax rules.
Security changes permission policy.
UI team changes invoice rendering.
```

If these changes repeatedly touch the same class, inspect its responsibilities.

---

# 48. Merge Conflict Test

Frequent merge conflicts can reveal responsibility overlap:

```text
developer A edits tax method
developer B edits notification
developer C edits serialization
```

all inside:

```ts
OrderService
```

The conflict is not the root problem.

The mixed responsibility is.

---

# 49. Code Ownership Signal

If different teams repeatedly edit different regions of the same module, the module may contain multiple responsibility boundaries.

This is an organizational signal of design coupling.

---

# 50. Change History as Design Evidence

Version control can reveal SRP issues.

Ask:

```text
Which files change together?
Which lines change for unrelated reasons?
Which stakeholders submit the changes?
```

Historical change coupling is valuable evidence.

---

# 51. Co-Change Analysis

Conceptually:

```text
File A changes with B 90% of the time
File A changes with C 10% of the time
```

A, B may form a cohesive boundary.

If one file changes with unrelated groups frequently, it may be a responsibility hub.

---

# 52. SRP and Architecture

SRP applies at many scales:

```text
function
class
module
package
service
database component
event consumer
team
```

The unit changes.

The principle remains:

```text
one coherent reason to change
```

---

# 53. Function-Level SRP

A function should not simultaneously:

```text
parse CSV
write database
send email
format HTML
```

unless that orchestration itself is the responsibility.

A function called:

```ts
processImport()
```

may legitimately orchestrate several specialized steps.

---

# 54. Module-Level SRP

A module may contain multiple classes if they form one cohesive responsibility.

Example:

```text
pricing/
  Money.ts
  DiscountPolicy.ts
  PriceCalculator.ts
  pricingErrors.ts
```

The package can still have one business responsibility:

```text
pricing
```

---

# 55. Package-Level SRP

Package boundaries are important in larger codebases.

A package like:

```text
billing/
```

may contain:

```text
Invoice
TaxPolicy
PaymentTerms
BillingRepository
BillingEvents
```

These may change together due to billing requirements.

---

# 56. Service-Level SRP

A service should ideally represent a coherent business capability.

Bad:

```text
misc-service
```

containing:

```text
billing
users
inventory
notifications
reporting
```

This creates high change coupling and deployment coupling.

---

# 57. Microservice Myth

SRP does not mean:

```text
one class
→ one microservice
```

Over-splitting into services can create:

```text
network coupling
distributed transactions
deployment overhead
observability complexity
```

Service boundaries need stronger evidence.

---

# 58. SRP in Distributed Systems

A service can legitimately own many internal operations when they serve one capability.

Example:

```text
Inventory Service
  reserve
  release
  adjust
  transfer
```

These can share:

```text
inventory responsibility
```

---

# 59. SRP and Service Ownership

Ask:

```text
Which business capability owns this decision?
```

This can be more useful than:

```text
Which class owns this method?
```

Architecture is responsibility assignment at a larger scale.

---

# 60. SRP and Event Consumers

Suppose:

```ts
OrderConsumer
```

handles:

```text
email
analytics
inventory
audit
```

This may actually be several consumer responsibilities.

Separate handlers can reduce change coupling.

---

# 61. Event Handler Cohesion

Good:

```text
InventoryReservedHandler
```

owns:

```text
reacting to inventory reservation
```

Bad:

```text
GenericOrderEventHandler
```

owns many unrelated reactions.

---

# 62. SRP and Command Handlers

A command handler can have one responsibility:

```text
execute one application use case
```

That use case may involve many collaborators.

The handler should coordinate, not accumulate every policy.

---

# 63. SRP and Query Handlers

Query handlers may own:

```text
one read use case
```

They can use:

```text
repositories
read models
mappers
authorization checks
```

when those support the same query responsibility.

---

# 64. SRP and Security

Security logic is often tempting to put everywhere.

Instead distinguish:

```text
authorization policy
authentication mechanism
business operation
audit behavior
```

These can be separate responsibilities even though they all concern “security.”

---

# 65. Security Example

Bad:

```ts
class OrderService {
  createOrder() {}
  verifyPassword() {}
  encryptSecrets() {}
  checkTenantAccess() {}
}
```

Not every security-related operation belongs in one object.

The security domain itself contains multiple responsibilities.

---

# 66. SRP and Observability

Observability also has multiple concerns:

```text
logging
metrics
tracing
audit
```

A domain class should not necessarily own all of them.

However, emitting a domain event may be part of a coherent state transition contract.

Distinguish required business side effects from technical instrumentation.

---

# 67. SRP and Audit

Audit is often a cross-cutting concern.

Avoid putting:

```ts
this.audit.log(...)
```

through every line of domain logic without a clear ownership model.

Use structured boundaries:

```text
domain event
→ audit subscriber
```

when appropriate.

---

# 68. SRP and Persistence

Persistence can be separated from domain responsibility:

```text
Domain
  owns business invariants

Repository
  owns persistence interaction
```

But repository methods should remain cohesive around the persistence responsibility.

---

# 69. SRP and Serialization

Serialization may be its own responsibility when:

```text
external schema changes independently
```

This protects domain objects from transport changes.

---

# 70. SRP and Mapping

Mapping code is often cohesive:

```text
OrderEntityMapper
```

may own:

```text
DB row ↔ domain object
```

If mapping conventions are shared across the package, a mapper can be valuable.

---

# 71. SRP and Validation

Validation has layers:

```text
shape validation
domain validation
authorization validation
database integrity
```

A single `Validator` class should not necessarily own all of them.

Put each rule where it has semantic ownership.

---

# 72. SRP and Business Rules

Business rules often belong together when they represent one policy.

Example:

```text
DiscountPolicy
```

may have:

```text
isEligible
calculateDiscount
explainDiscount
```

These can share one change reason:

```text
discount policy changes
```

---

# 73. Policy Objects

Policy objects are often SRP-friendly:

```ts
interface DiscountPolicy {
  isEligible(order: Order, customer: Customer): boolean;
  calculate(order: Order, customer: Customer): Money;
}
```

They isolate a coherent decision policy.

---

# 74. Strategy Pattern Connection

Strategy objects can encode a variable responsibility:

```text
PricingStrategy
TaxStrategy
ShippingStrategy
```

This is useful when the variation is real.

Do not create strategies for stable code solely to appear extensible.

---

# 75. SRP and YAGNI

A principle can be over-applied.

Before extracting:

```text
TaxPolicyFactory
TaxPolicyRegistry
TaxPolicyResolver
TaxPolicyProvider
```

ask:

```text
Is there actually one change axis?
Is there real variability?
Do consumers benefit?
```

YAGNI protects SRP from speculative fragmentation.

---

# 76. SRP and KISS

A simple cohesive class is often better than:

```text
15 abstractions for 15 lines
```

SRP should simplify change, not maximize type count.

---

# 77. SRP and DRY

DRY says avoid unjustified duplication.

SRP says keep independent reasons for change separate.

These can conflict.

Copying a small rule may be better than creating a shared abstraction between two independently changing policies.

---

# 78. False Duplication

Two code blocks may look similar:

```ts
calculateRetailTax()
calculateWholesaleTax()
```

but if regulations evolve independently, forcing them into one abstraction can violate SRP.

Structural similarity is not semantic sameness.

---

# 79. Abstraction Smell

Bad abstraction:

```ts
class GenericCalculator {
  calculate(input: unknown) {}
}
```

It hides unrelated policies behind one generic concept.

An abstraction should capture stable common responsibility, not accidental code similarity.

---

# 80. SRP and Semantic Coupling

Two methods may depend on the same data yet have different meanings.

Example:

```text
calculateInvoiceTax
calculateTaxReport
```

They may both use:

```text
tax rules
```

but serve different stakeholders.

Shared data is not sufficient reason for shared responsibility.

---

# 81. Shared Policy vs Shared Utility

A utility:

```ts
roundMoney(value)
```

may be widely reusable.

A policy:

```ts
calculateTax(invoice)
```

contains domain meaning.

Do not extract domain behavior into generic utilities merely to reduce class size.

---

# 82. Utility Class Smell

A giant:

```text
Utils
```

often contains multiple unrelated responsibilities.

Prefer semantic modules:

```text
money/rounding
dates/time
identifiers
validation
```

when the boundaries are meaningful.

---

# 83. Helper Function Extraction

Extracting a helper is not automatically SRP refactoring.

This:

```ts
function createOrder() {
  validate();
  calculate();
  save();
}
```

may become:

```ts
validateOrder()
calculateOrder()
saveOrder()
```

while the same class still owns all responsibilities.

Function extraction improves readability but may not change responsibility ownership.

---

# 84. True SRP Refactoring

A real responsibility refactor changes ownership:

```text
before:
OrderService
  tax
  persistence
  rendering

after:
Order
TaxPolicy
OrderRepository
InvoiceRenderer
```

The important change is responsibility placement.

---

# 85. Extract Class

A classic SRP refactoring:

```text
Identify independent change reason
→ define cohesive abstraction
→ move state + behavior together
→ introduce collaboration
→ preserve contract
```

Do not extract methods randomly.

---

# 86. Move Method

If a method uses mostly another object's state and serves another responsibility:

```text
move it to the information owner.
```

This can improve both cohesion and SRP.

---

# 87. Move Field

If fields exist mostly for one responsibility:

```text
move related state with its behavior.
```

State and behavior should travel together where possible.

---

# 88. Extract Policy

If conditionals encode one variation:

```ts
if (customer.type === "VIP") ...
if (customer.type === "WHOLESALE") ...
```

consider:

```ts
DiscountPolicy
```

when policy variation is genuine.

---

# 89. Extract Adapter

If external provider details dominate a class:

```text
provider SDK types
provider error codes
provider request formats
```

extract:

```text
ProviderAdapter
```

and protect the domain contract.

---

# 90. Extract Repository

If business logic is mixed with persistence:

```ts
db.query(...)
```

inside domain operations, move persistence access toward a repository or dedicated gateway when that boundary provides value.

---

# 91. Extract Renderer

If domain objects contain:

```text
HTML
PDF
CSV
JSON formatting
```

presentation may be a different reason to change.

Extract presentation responsibility.

---

# 92. Extract Notifier

Notification formatting and delivery often change independently:

```text
email templates
SMS provider
push notification
```

Separate notification responsibility from domain state transitions.

---

# 93. Avoid the “Service” Dump

A name like:

```text
OrderService
```

can hide multiple responsibilities.

Ask what the service actually does:

```text
CreateOrderService
ReserveStockService
CalculateOrderTotal
OrderRepository
OrderNotifier
```

Names can reveal responsibility.

---

# 94. Naming Test

A class name should let you complete:

> “This object is responsible for ______.”

If the answer needs:

```text
and
and
and
```

inspect the boundary.

But broad names can still be legitimate for orchestration.

---

# 95. “And” Heuristic

Weak:

```text
OrderManager
  validates orders and
  saves orders and
  emails customers and
  generates PDFs
```

Stronger:

```text
CreateOrderHandler
  coordinates order creation
```

The second has one application-level responsibility even though it coordinates several collaborators.

---

# 96. “Because” Heuristic

Complete:

> “We would change this code because ______.”

If the blank has multiple independent answers:

```text
tax law changes
PDF design changes
database changes
email provider changes
```

you likely have multiple reasons to change.

---

# 97. “Who Calls?” Test

Caller grouping can provide evidence.

If:

```text
Tax subsystem
```

uses one subset of methods and:

```text
Presentation subsystem
```

uses another, the object may span boundaries.

Caller patterns alone are not proof, but they are useful evidence.

---

# 98. “Who Owns the Rule?” Test

For every method ask:

```text
Who owns the rule this method implements?
```

If different methods have different owners:

```text
SRP risk increases.
```

---

# 99. Rule Ownership

Example:

```text
minimum margin
```

owned by:

```text
Pricing
```

while:

```text
password complexity
```

owned by:

```text
Security
```

They should not become one “business rules” class merely because both are rules.

---

# 100. SRP and Domain Boundaries

Boundaries should often reflect:

```text
domain concepts
policy ownership
change authority
invariants
```

The same principle scales from one class to a bounded context.

---

# 101. Example — Jewellery ERP Product

Consider a:

```ts
JewelleryItem
```

Possible domain responsibilities:

```text
identity
weight/purity state
stock status transitions
pricing metadata
```

But this does not mean it should own:

```text
SQL persistence
HTTP serialization
PDF invoice rendering
email delivery
```

Those have different change reasons.

---

# 102. Jewellery ERP Pricing

Potential responsibility:

```text
PricingPolicy
```

may own:

```text
metal rate
making charge
wastage
stone charge
discount rules
rounding
```

if these evolve under the same pricing policy authority.

---

# 103. Jewellery ERP Inventory

Potential responsibility:

```text
InventoryAggregate
```

may own:

```text
reserve
release
adjust
transfer
availability
```

when they share inventory invariants and state ownership.

---

# 104. Jewellery ERP Audit

Audit can be separate:

```text
AuditRecorder
```

or event-driven subscriber.

Do not put all audit formatting and storage details into inventory domain objects.

---

# 105. Jewellery ERP Multi-Tenant Context

Tenant resolution and tenant-scoped data access may be infrastructure/application responsibilities.

The core domain should still enforce relevant business rules.

SRP does not mean security disappears from the domain; it means ownership is explicit.

---

# 106. Jewellery ERP Branch Responsibility

A branch-level policy may own:

```text
branch-specific price overrides
operating hours
local approvals
```

Do not mix those rules into global product identity if they have independent policy ownership.

---

# 107. SRP and Multi-Tenancy

Tenant context can be:

```text
application/security context
```

while tenant-specific business policy belongs to the appropriate domain service.

The exact boundary depends on where tenant semantics are authoritative.

---

# 108. SRP and Configuration

Configuration parsing is different from business policy.

Example:

```text
MAX_RETRY
```

belongs to operational configuration.

```text
customer discount eligibility
```

belongs to business policy.

Both are “rules,” but they change for different reasons.

---

# 109. SRP and Feature Flags

Feature-flag evaluation may be technical/platform responsibility.

The business policy still needs to be coherent.

Avoid sprinkling:

```ts
if (flags.newPricing) ...
```

through every class.

Centralize feature-variation decisions at deliberate boundaries.

---

# 110. SRP and Framework Code

Framework conventions can tempt you into large files:

```text
controller
component
resolver
handler
```

Framework role does not justify unlimited responsibilities.

Use framework boundaries as shells around cohesive application behavior.

---

# 111. SRP in React

A component can reasonably handle:

```text
one UI responsibility
```

and still contain:

```text
state
event handling
rendering
accessibility
```

These are often facets of one presentation responsibility.

Extract when independent change reasons emerge.

---

# 112. SRP in Node.js

A Node service may contain:

```text
request handling
orchestration
domain logic
data access
```

If all live in one class because “Node services are usually services,” SRP is being ignored by naming convention.

---

# 113. SRP in NestJS

A NestJS provider named:

```text
OrderService
```

could be:

```text
use-case coordinator
```

or:

```text
domain service
```

or:

```text
repository wrapper
```

The framework does not decide responsibility.

The semantics do.

---

# 114. TypeScript and SRP

TypeScript interfaces can make boundaries visible.

```ts
interface TaxPolicy {
  calculate(input: TaxInput): Money;
}
```

A narrow interface is easier to change safely than:

```ts
interface EverythingService {
  // dozens of unrelated methods
}
```

---

# 115. Structural Typing and SRP

Structural typing means a type may satisfy an interface accidentally.

This can be useful.

But semantic responsibility still matters.

A class implementing:

```ts
TaxPolicy
UserRepository
Mailer
```

may satisfy all structural contracts while having poor responsibility cohesion.

---

# 116. Type-Level Segregation

Small interfaces help apply SRP at the contract layer.

```ts
interface UserReader {}
interface UserWriter {}
interface PasswordVerifier {}
```

This reduces unnecessary coupling between clients and implementations.

---

# 117. SRP and Interface Segregation

SRP and ISP often reinforce one another:

```text
SRP
  cohesive implementation responsibility

ISP
  cohesive client-facing contract
```

But do not mechanically create one interface per method.

---

# 118. SRP and Dependency Inversion

A stable abstraction should usually represent a meaningful responsibility.

Bad:

```ts
interface IService {
  doEverything(): Promise<void>;
}
```

Better:

```ts
interface PaymentGateway {}
interface OrderRepository {}
interface NotificationPort {}
```

Each abstraction protects a change boundary.

---

# 119. SRP and Stable Dependencies

A dependency should not force consumers to change for unrelated reasons.

If:

```ts
OrderService
```

depends on a giant:

```ts
CompanyService
```

then unrelated company changes can ripple through order code.

Split along meaningful responsibilities.

---

# 120. SRP and Circular Dependencies

Responsibility mixing can contribute to cycles:

```text
OrderService
 ↔
CustomerService
 ↔
BillingService
```

Clear ownership can reduce cycles.

SRP does not automatically solve cycles, but it often exposes the missing boundary.

---

# 121. SRP and Dependency Direction

A cohesive responsibility should have clear dependencies.

For example:

```text
Application
   ↓
Domain policy
   ↓
Infrastructure adapter
```

When one module contains all layers, direction becomes harder to maintain.

---

# 122. SRP and Layering

Layering answers:

```text
which kind of responsibility belongs where?
```

SRP asks:

```text
does each layer/module have coherent responsibility?
```

Use both.

---

# 123. SRP and Boundary Erosion

A common smell:

```ts
Order
  imports database client
  imports HTTP client
  imports PDF library
  imports SMTP client
```

The domain object has become a boundary-crossing object.

Even if methods are individually understandable, responsibility is fragmented.

---

# 124. The “Everything Near the Use Case” Trap

Developers sometimes keep all logic in one use-case file because it is convenient.

Short-term benefit:

```text
easy navigation
```

Long-term risk:

```text
mixed policies
harder reuse
harder testing
frequent conflicts
```

Balance locality against responsibility cohesion.

---

# 125. SRP and Locality

Not every extracted class improves maintainability.

Too many abstractions can make the reader jump:

```text
use case
→ helper
→ policy
→ helper
→ adapter
→ utility
```

Good SRP preserves conceptual locality where reasonable.

---

# 126. Package Cohesion

A package should make semantic sense as a unit.

Bad package:

```text
common/
  tax.ts
  jwt.ts
  invoicePdf.ts
  inventory.ts
  email.ts
```

“Common” is not a responsibility.

---

# 127. Responsibility-Oriented Folders

Prefer:

```text
billing/
inventory/
identity/
notifications/
pricing/
```

when the domain supports these boundaries.

---

# 128. SRP and Refactoring Safety

Before extraction:

```text
capture current behavior
identify public contract
identify invariants
map callers
extract responsibility
run tests
```

This connects Chapter 17 refactoring with Chapter 18 contracts.

---

# 129. Contract Preservation During SRP Refactoring

A refactor should preserve:

```text
input semantics
output semantics
errors
ordering
side effects
security
transactionality
```

unless a deliberate contract change is intended.

---

# 130. Characterization Testing

Legacy object:

```ts
class LegacyOrderService {
  // huge behavior
}
```

Before splitting:

```text
characterize outputs
errors
side effects
ordering
```

Then refactor.

This protects behavior while responsibility ownership changes.

---

# 131. SRP Refactoring Sequence

```text
1. identify reasons to change
2. identify invariants
3. identify dependencies
4. identify contract surface
5. identify cohesive state
6. extract responsibility
7. preserve public contract
8. migrate callers
9. delete old path
```

---

# 132. Responsibility Matrix

Create a table:

| Behavior | State Used | Rule Owner | Change Source | Candidate Owner |
|---|---|---|---|---|
| reserve stock | inventory quantity | Operations | stock policy | Inventory |
| calculate tax | tax inputs | Finance | tax law | TaxPolicy |
| render invoice | invoice data | Design | template | Renderer |
| persist order | order state | Platform | DB schema | Repository |

This exposes mixed responsibilities.

---

# 133. Change-Cause Matrix

Another useful representation:

| Requirement Change | Affects | Should Affect |
|---|---|---|
| tax law | tax rules | TaxPolicy |
| invoice branding | HTML/PDF | Renderer |
| DB migration | persistence | Repository |
| payment provider | integration | PaymentAdapter |
| stock policy | inventory | Inventory |

A class affected by every row is suspicious.

---

# 134. SRP and Blast Radius

A responsibility boundary reduces blast radius when:

```text
one policy changes
→ fewer unrelated modules recompile/retest/redeploy
```

But too many boundaries can also increase operational overhead.

---

# 135. SRP and Build Performance

In a large TypeScript monorepo, cohesive packages can improve:

```text
incremental builds
test targeting
ownership
dependency analysis
```

But excessive package fragmentation can increase graph complexity.

Measure.

---

# 136. SRP and Runtime Performance

Refactoring a class into collaborators may introduce:

```text
more allocations
more calls
more object graphs
```

Usually this is negligible at application boundaries.

In hot computational paths, measure before optimizing.

---

# 137. SRP and Memory

More objects can increase memory overhead.

But a monolithic object can also retain unrelated state longer.

The trade-off is:

```text
object count
vs
state locality
vs
lifetime
```

---

# 138. SRP and Security

Smaller responsibility boundaries can reduce authority.

For example:

```text
InvoiceRenderer
```

does not need:

```text
database write permission
```

A clear boundary can support least privilege.

---

# 139. SRP and Reliability

A module with multiple unrelated responsibilities has more reasons to fail.

Separating responsibilities can make failure containment easier:

```text
email provider outage
→ notification subsystem degraded
→ order creation remains functional
```

when architecture supports this.

---

# 140. SRP and Observability

Observability should help identify which responsibility failed.

Prefer signals such as:

```text
pricing_failure
inventory_conflict
payment_provider_timeout
notification_failure
```

rather than one:

```text
OrderServiceError
```

---

# 141. SRP and Testing

A cohesive responsibility is easier to test.

For:

```ts
TaxPolicy
```

tests can focus on:

```text
tax scenarios
rounding
jurisdiction rules
```

For:

```text
InvoiceRenderer
```

tests focus on:

```text
presentation output
```

---

# 142. SRP and Test Setup

A giant class often requires:

```text
database mock
mailer mock
clock mock
payment mock
renderer mock
logger mock
```

just to test one method.

This can indicate mixed responsibilities.

Not every multi-dependency test proves SRP violation, but it is useful evidence.

---

# 143. Testability as Diagnostic

If isolating one behavior requires constructing half the application, inspect responsibility boundaries.

Test difficulty can reveal coupling hidden by production execution.

---

# 144. SRP and Property-Based Testing

Properties can be tested around a focused policy:

```text
tax calculation preserves non-negative invariant
```

A cohesive policy makes property boundaries clearer.

---

# 145. SRP and Contract Tests

A stable responsibility can have a stable contract:

```ts
interface PaymentGateway
```

and multiple adapters can satisfy it.

If one class combines gateway behavior and unrelated policy, contract tests become harder to isolate.

---

# 146. SRP and State Ownership

A strong rule:

> State should usually live with the responsibility that owns the rules governing its transitions.

This connects SRP to invariants from Chapter 18.

---

# 147. State Ownership Example

If:

```text
InventoryAggregate
```

owns:

```text
availableQuantity
reservedQuantity
```

then methods that mutate those values should remain close to the inventory responsibility.

Do not move the fields away merely to create smaller classes.

---

# 148. SRP Does Not Mean Statelessness

A cohesive responsibility may own substantial state.

Example:

```ts
class ShoppingCart {
  #lines: Map<ProductId, CartLine>;
  #customer: CustomerId;
}
```

This can be strongly cohesive.

State is not the enemy.

Unrelated state is.

---

# 149. SRP and Immutable Values

Value objects make responsibility boundaries clearer.

```text
Money
Address
Quantity
Sku
```

Each can own one coherent value concept and its invariants.

---

# 150. SRP and Domain Services

Domain services should represent domain responsibilities that do not naturally belong to one entity.

Example:

```ts
PricingService
```

may coordinate:

```text
product data
customer status
pricing policy
```

But it should not also own:

```text
HTTP serialization
SQL persistence
email delivery
```

---

# 151. SRP and Application Services

Application services orchestrate use cases.

Their responsibility may legitimately involve:

```text
load
authorize
invoke domain
persist
publish
```

The key is that these steps are all part of one use-case responsibility.

---

# 152. Application Service vs God Service

Difference:

```text
Application service:
  one use case / capability

God service:
  many unrelated capabilities
```

A service becomes suspicious when its name is the only thing tying its methods together.

---

# 153. SRP and Use-Case Boundaries

Good:

```text
CreateOrder
CancelOrder
ReserveStock
CapturePayment
```

Each represents a meaningful business operation.

This often provides better responsibility boundaries than:

```text
OrderManager
```

with 40 methods.

---

# 154. SRP and Commands

A command is a useful responsibility boundary:

```ts
type CancelOrderCommand = {
  orderId: OrderId;
  actorId: UserId;
  reason: string;
};
```

The command expresses a coherent intent.

---

# 155. SRP and Queries

Queries can separate read responsibilities from state-changing responsibilities.

```text
GetOrderDetails
ListBranchInventory
FindCustomer
```

These may have distinct change reasons because their read models evolve independently.

---

# 156. SRP and CQS

Command-Query Separation complements SRP:

```text
commands
  change state

queries
  answer questions
```

Separating them can reduce responsibility overlap.

---

# 157. SRP and Law of Demeter

A class that knows too many internal collaborators may be doing too much.

Long chains:

```ts
order.customer.account.profile...
```

can reveal responsibility leakage.

Law of Demeter and SRP often expose different sides of the same boundary problem.

---

# 158. SRP and Tell, Don't Ask

Instead of:

```ts
if (order.status === "PAID") ...
```

outside the order:

```ts
order.canShip()
```

The object owns the relevant rule.

This can improve responsibility cohesion.

---

# 159. SRP and Polymorphism

Polymorphism can isolate a responsibility's variation:

```ts
interface ShippingPolicy {
  calculate(order: Order): Money;
}
```

Different shipping policies can evolve without changing unrelated callers.

---

# 160. SRP and Conditional Explosion

Many branches often reveal multiple policy responsibilities.

Example:

```text
if currency
if region
if customerType
if channel
if branch
```

The class may encode several independent change axes.

Do not automatically replace every branch with polymorphism.

First identify whether the axes truly vary independently.

---

# 161. Multi-Axis Conditional

A dangerous method:

```ts
calculatePrice(
  currency,
  region,
  customerType,
  channel,
  branch,
  promotion
)
```

This may be a combinatorial responsibility.

Extracting each axis can help only when the domain actually models them separately.

---

# 162. Decision Table and SRP

Create a decision table:

```text
Dimension | Owner | Variation
currency  | Money  | currency rules
region    | Tax    | tax jurisdiction
customer  | Pricing| discount eligibility
channel   | Sales  | channel pricing
```

This reveals responsibility axes.

---

# 163. SRP and Feature Growth

A small class can become a god object gradually:

```text
v1 one responsibility
v2 + persistence
v3 + notifications
v4 + reporting
v5 + authorization
```

Regular responsibility review prevents accretion.

---

# 164. Responsibility Drift

Responsibility drift is:

```text
small unrelated feature
→ added to convenient existing class
→ boundary slowly expands
```

It is often caused by short-term convenience.

---

# 165. Guarding Against Drift

Before adding a method, ask:

```text
Does this behavior share the same reason to change?
Does it protect the same invariant?
Does it serve the same stakeholder?
Does it use the same conceptual state?
```

If not, consider another owner.

---

# 166. SRP Review in Pull Requests

Review question:

> Which existing responsibility does this code belong to, and why?

If the answer is:

```text
because this was the easiest file to find
```

that is an SRP warning.

---

# 167. “Convenient File” Smell

The wrong class often accumulates behavior because:

```text
it already has access to the needed dependencies.
```

Dependency availability is not responsibility ownership.

---

# 168. SRP and Access to Data

A class having access to some data does not mean it should own all behavior over that data.

Ask:

```text
Who owns the rule?
```

not merely:

```text
Who can reach the object?
```

---

# 169. SRP and Shared Models

A shared:

```ts
User
```

model can be used by:

```text
billing
identity
support
analytics
```

That does not mean all their behaviors belong on `User`.

Shared data does not imply shared responsibility.

---

# 170. Rich Domain Model Boundary

Put behavior on an entity when:

```text
it protects the entity's invariant
uses its state
represents its domain meaning
```

Keep out behavior whose owner lies elsewhere.

---

# 171. Transaction Boundary and SRP

A transaction often spans several technical operations.

That does not mean one class should own every underlying responsibility.

An application service can coordinate one business transaction using specialized components.

---

# 172. SRP and Transaction Scripts

A transaction script may be acceptable when:

```text
workflow is simple
domain complexity is low
business rules are limited
```

The goal is not to force everything into rich objects.

SRP remains about change responsibility.

---

# 173. SRP and Functional Style

Functional modules can also follow SRP.

Example:

```ts
pricing.ts
inventory.ts
tax.ts
```

The principle is not class-specific.

It applies to modules and responsibilities.

---

# 174. SRP and Closures

A closure-based module may own:

```text
one coherent state machine
```

without classes.

Same design reasoning.

---

# 175. SRP and JavaScript Prototypes

A prototype method belongs to its object abstraction.

The language mechanism does not determine whether the abstraction has one responsibility.

SRP remains a design-level principle.

---

# 176. SRP and Private Fields

Private fields can protect state inside a responsibility.

```ts
class Inventory {
  #available: number;
}
```

If unrelated concerns begin requiring access to `#available`, ask whether the responsibilities are being mixed.

---

# 177. SRP and Static Methods

Static utility-like methods can hide unrelated responsibilities:

```ts
UserService.parse()
UserService.hash()
UserService.validate()
UserService.save()
```

Staticness does not create cohesion.

---

# 178. SRP and Module Functions

A module can be cohesive:

```ts
export function calculateTotal() {}
export function validateLine() {}
export function normalizeLine() {}
```

if all serve one pricing/order-line responsibility.

Again:

```text
responsibility > class count
```

---

# 179. SRP and Namespaces

A module namespace should not become a dumping ground.

Prefer domain names:

```text
money
pricing
inventory
identity
```

rather than:

```text
misc
helpers
utils
services
```

when semantics permit.

---

# 180. SRP and Circular Import Symptoms

A module with many responsibilities often becomes a dependency hub:

```text
A → GodModule ← B
             ↑
             C
```

Splitting by responsibility can reduce hub pressure.

---

# 181. SRP and Public API Surface

A broad module export list can be an SRP smell:

```ts
export {
  calculateTax,
  sendEmail,
  hashPassword,
  createPdf,
  queryUsers,
  reserveStock,
};
```

The public API itself reveals mixed concepts.

---

# 182. SRP and Contract Surface

Chapter 18 showed:

```text
contract surface area
```

SRP helps keep that surface aligned around one semantic responsibility.

A mixed module often exposes unrelated contracts.

---

# 183. SRP and Versioning

If one package exposes:

```text
tax API
PDF API
payment API
```

then any breaking change forces a versioning discussion for unrelated consumers.

Separating contracts reduces release coupling.

---

# 184. SRP and Deployment

In modular or service architectures:

```text
billing change
```

should not automatically require:

```text
inventory deployment
```

when those capabilities are independent.

Responsibility boundaries can support deployment independence.

---

# 185. SRP and Team Boundaries

A useful organizational signal:

```text
Team A owns pricing
Team B owns notifications
```

Putting both in one class can create code ownership conflict.

Teams are not the only basis for architecture, but repeated ownership boundaries matter.

---

# 186. Conway's Law Consideration

System structures tend to reflect communication structures.

If independent teams repeatedly need the same module, SRP issues may appear.

Use organizational boundaries as evidence, not as a rigid rule.

---

# 187. SRP and Repository Ownership

A package owned by one team can still contain multiple responsibilities.

Conversely, one responsibility can cross team boundaries in legacy systems.

SRP is semantic first.

---

# 188. SRP and Monorepos

Monorepo organization can expose boundaries:

```text
packages/pricing
packages/inventory
packages/billing
```

But package extraction should follow responsibility evidence.

Do not create 100 packages because “micro-packages are cleaner.”

---

# 189. SRP and Shared Infrastructure

Shared logging, configuration, and HTTP clients can remain shared infrastructure because their responsibility is infrastructure-wide.

Sharing does not automatically violate SRP.

---

# 190. SRP and Cross-Cutting Concerns

Some concerns legitimately cross many modules:

```text
logging
tracing
authorization
validation
```

Do not force every cross-cutting concern into domain objects.

Use appropriate architectural mechanisms:

```text
middleware
interceptors
decorators
policies
events
adapters
```

while preserving domain responsibility.

---

# 191. SRP and Middleware

HTTP middleware can own:

```text
request authentication
```

rather than making every controller parse tokens.

That creates a coherent transport/security boundary.

---

# 192. SRP and Interceptors

Interceptors can own:

```text
logging
metrics
tracing
```

when framework semantics support them.

This keeps instrumentation from overwhelming business classes.

---

# 193. SRP and Decorators

Decorators can attach cross-cutting behavior.

But beware:

```text
hidden behavior
```

A decorator-heavy system can become difficult to reason about.

SRP is not improved if responsibility becomes merely invisible.

---

# 194. SRP and Dependency Injection

Dependency injection supports responsibility separation by making collaborators explicit.

It does not guarantee SRP.

A class with 12 dependencies can still be a god object.

---

# 195. SRP and Service Locator

Service locators hide dependencies:

```ts
container.resolve(...)
```

This can obscure responsibility boundaries.

Explicit dependencies usually make SRP review easier.

---

# 196. SRP and Testing Mocks

Mock-heavy classes may indicate:

```text
too many responsibilities
```

especially when each test mocks a different subsystem.

But some coordinators legitimately depend on many collaborators.

Use the pattern as evidence, not proof.

---

# 197. SRP and Complexity Metrics

Traditional metrics such as:

```text
LOC
cyclomatic complexity
method count
dependency count
```

can be signals.

None directly measures responsibility.

Semantic analysis remains primary.

---

# 198. SRP and Change Coupling Metrics

Historical co-change data can be more informative.

Useful analysis:

```text
commit co-occurrence
pull request ownership
 test failure clusters
release coupling
```

This is especially powerful in mature repositories.

---

# 199. SRP and Failure Correlation

If unrelated incidents repeatedly originate from one module:

```text
tax incident
email incident
inventory incident
```

that module may be absorbing unrelated responsibilities.

---

# 200. SRP and Incident Blast Radius

A failure in one responsibility should ideally not corrupt unrelated responsibilities.

Clear boundaries can support:

```text
containment
degraded mode
independent retry
independent monitoring
```

---

# 201. SRP and Retry Domains

Different responsibilities may have different retry policies.

```text
database persistence
  retryable in some cases

email delivery
  asynchronous retry

domain calculation
  no retry needed
```

Combining them into one method complicates failure semantics.

---

# 202. SRP and Idempotency

Different responsibilities may have different idempotency semantics.

```text
calculateTax()
  naturally deterministic

sendEmail()
  requires duplicate protection

reserveStock()
  requires domain concurrency semantics
```

One giant workflow can hide these differences.

---

# 203. SRP and Timeouts

External responsibilities can have distinct timeout policies:

```text
payment
notification
search
```

Separating them lets infrastructure policies align with responsibility boundaries.

---

# 204. SRP and Consistency

Different responsibilities may use different consistency requirements:

```text
inventory
  strong/local authoritative

analytics
  eventual

notifications
  asynchronous
```

A monolithically designed service may accidentally impose one policy on all.

---

# 205. SRP and Security Authority

Different collaborators may require different privileges:

```text
OrderDomain
  no email credentials

Mailer
  notification provider credentials

Repository
  database access
```

Separating responsibilities supports least privilege.

---

# 206. SRP and Tenant Data

Tenant-scoped repositories can isolate tenant responsibility from transport concerns.

A controller should not need to know database filtering details.

---

# 207. SRP and Auditability

A focused responsibility is easier to audit.

For example:

```text
PricingPolicy
```

can be reviewed for pricing rules without reading:

```text
email
SQL
HTTP
PDF
```

---

# 208. SRP and Compliance

Compliance rules may change due to:

```text
law
regulation
audit requirements
```

Separating compliance policy from technical presentation can reduce unrelated churn.

---

# 209. SRP and Data Retention

Retention policy:

```text
delete old records after N years
```

is not necessarily the same responsibility as:

```text
write order data
```

Though a lifecycle component may coordinate both if retention is part of the owned capability.

---

# 210. SRP and Schema Migration

Schema migration concerns belong to persistence evolution.

Avoid making domain objects aware of:

```text
migration version numbers
SQL ALTER statements
database-specific syntax
```

unless persistence is explicitly part of the abstraction.

---

# 211. SRP and ORM Entities

ORM entities can mix:

```text
database mapping
domain behavior
serialization
```

This can be acceptable in some designs.

The question is whether those concerns genuinely change together.

---

# 212. SRP and Active Record

Active Record combines:

```text
domain-ish data
persistence operations
```

This can be reasonable for simple applications.

As persistence and domain rules evolve independently, the mixed responsibility becomes more costly.

---

# 213. SRP and Data Mapper

Data Mapper separates persistence from domain objects.

This can improve change isolation when:

```text
domain rules
and
database representation
```

change independently.

---

# 214. SRP and Event Sourcing

Event-sourced aggregates own:

```text
state transition rules
```

while:

```text
event serialization
storage
projection
```

can remain separate responsibilities.

---

# 215. SRP and Projections

A projection should own:

```text
turn event stream into a specific read model
```

not:

```text
also send marketing emails
also reserve inventory
also calculate tax
```

---

# 216. SRP and Outbox

An outbox publisher can own:

```text
publish durable integration messages
```

while domain logic owns:

```text
business event creation
```

The boundaries prevent transport details from contaminating business rules.

---

# 217. SRP and Workflow Engines

A workflow component may legitimately coordinate:

```text
payment
inventory
shipment
notifications
```

if its one responsibility is:

```text
orchestrate order fulfillment workflow
```

Orchestration itself is a valid responsibility.

---

# 218. The Coordinator Exception

Important rule:

> A coordinator can have many collaborators and still have one responsibility if the coordination itself is cohesive.

Do not split every coordinator into meaningless fragments.

---

# 219. Orchestrator vs God Object

Orchestrator:

```text
knows sequence
delegates decisions
owns workflow
```

God object:

```text
knows sequence
implements every decision
owns every state
touches every concern
```

This distinction is critical.

---

# 220. SRP Decision Tree

Use:

```text
Does this behavior belong to the same domain concept?
  ↓ yes
Does it share the same invariant/state ownership?
  ↓ yes
Does it change for the same primary reason?
  ↓ yes
Keep together.

Otherwise:
  consider a boundary.
```

Then validate with:

```text
contracts
coupling
operational cost
```

---

# 221. Extraction Cost

Before splitting, estimate:

```text
new interfaces
new objects
new tests
new wiring
new files
new runtime calls
new mental jumps
```

SRP is beneficial when the reduction in change coupling justifies the new complexity.

---

# 222. Extraction Benefit

Measure potential benefit:

```text
independent release
smaller tests
clear ownership
lower change blast radius
reduced coupling
better security boundary
```

---

# 223. Responsibility Granularity

There is no universal class size.

Possible healthy unit sizes:

```text
5-line value object
200-line aggregate
500-line parser
large application coordinator
```

Size is a signal, not a definition.

---

# 224. Large Class Without SRP Violation

A complex parser may have hundreds of lines while owning:

```text
parse this language
```

If its internal mechanisms change together and the abstraction is cohesive, size alone does not prove an SRP violation.

---

# 225. Small Class With SRP Violation

A 20-line class can mix:

```text
authorization
payment
logging
```

and violate SRP.

Small does not mean cohesive.

---

# 226. SRP and Principal Judgment

The principal question is not:

> “Can I make this smaller?”

It is:

> “Will this boundary reduce meaningful change coupling without creating more complexity than it removes?”

---

# 227. Decision Heuristic

Prefer separation when:

```text
change reasons are independent
+
ownership is distinct
+
contracts differ
+
invariants differ
+
release cadence differs
```

Be cautious when:

```text
behavior shares state deeply
+
invariants are cross-method
+
changes usually happen together
+
separation creates indirection without flexibility
```

---

# 228. Refactoring Exercise — Notification Blob

Given:

```ts
class NotificationService {
  sendEmail(user: User, message: string) {}
  sendSms(user: User, message: string) {}
  renderEmailTemplate(data: object) {}
  renderSmsTemplate(data: object) {}
  retryFailedDelivery() {}
}
```

Identify responsibilities.

Potential axes:

```text
channel delivery
template rendering
retry policy
```

Decide which should remain together and which should split.

---

# 229. Refactoring Exercise — Billing Blob

```ts
class BillingService {
  calculateTax() {}
  calculateDiscount() {}
  chargeCard() {}
  saveInvoice() {}
  generatePdf() {}
}
```

Map:

```text
responsibility
stakeholder
reason to change
candidate owner
```

Then design a cohesive application service that coordinates them.

---

# 230. Refactoring Exercise — Jewellery ERP

```ts
class JewelleryService {
  createItem() {}
  calculatePrice() {}
  reserveStock() {}
  issueStock() {}
  generateInvoicePdf() {}
  sendWhatsAppNotification() {}
  saveItem() {}
}
```

Possible change authorities:

```text
product management
pricing
inventory
billing
notifications
persistence
```

Do not stop at “split the class.”

Define contracts and state ownership.

---

# 231. Implementation — Before

```ts
class OrderService {
  async complete(order: Order) {
    const tax = this.calculateTax(order);
    const total = order.subtotal + tax;

    await this.db.save({
      ...order,
      total,
    });

    await this.mailer.send(order.customerEmail, String(total));
  }

  calculateTax(order: Order) {
    return order.subtotal * 0.18;
  }
}
```

Responsibilities:

```text
tax policy
order persistence
notification
workflow orchestration
```

---

# 232. Implementation — After

```ts
class CompleteOrder {
  constructor(
    private readonly taxPolicy: TaxPolicy,
    private readonly orders: OrderRepository,
    private readonly notifier: OrderNotifier,
  ) {}

  async execute(order: Order): Promise<void> {
    const tax = this.taxPolicy.calculate(order);
    order.applyTax(tax);

    await this.orders.save(order);
    await this.notifier.notifyCompleted(order);
  }
}
```

The coordinator still performs one coherent responsibility:

```text
complete an order
```

---

# 233. Why the Refactor Helps

Now:

```text
tax changes
→ TaxPolicy

database changes
→ OrderRepository

notification changes
→ OrderNotifier

workflow changes
→ CompleteOrder
```

Change axes are separated.

---

# 234. Contract Preservation

Existing caller contract can remain:

```text
complete(order)
  success → completed order persisted + notification behavior
```

Implementation ownership changed.

Contract need not change.

---

# 235. Error Semantics After Refactoring

Be careful.

Suppose the old method exposed:

```text
database error
```

directly.

The new repository might translate it.

That can change the contract.

Therefore SRP refactoring must be checked against Chapter 18's contract model.

---

# 236. Transaction Semantics After Refactoring

If:

```text
save
```

and:

```text
notify
```

previously happened in one workflow, extracting collaborators must not accidentally change:

```text
transaction boundaries
ordering
side effects
retry semantics
```

Responsibility separation is not permission to alter semantics accidentally.

---

# 237. SRP and Side-Effect Ordering

A refactor can change:

```text
save
→ notify
```

into:

```text
notify
→ save
```

which may break contracts.

Preserve sequence deliberately.

---

# 238. SRP and Async Boundaries

Async collaborator extraction can create:

```text
parallel calls
```

where old code was:

```text
sequential
```

Performance may improve or correctness may break.

Review timing semantics.

---

# 239. SRP and Dependency Lifetime

Moving responsibilities can change object lifetime:

```text
singleton
vs
request-scoped
vs
operation-scoped
```

In dependency-injection systems, verify lifecycle after extraction.

---

# 240. SRP and Shared Mutable State

If two extracted classes share mutable state:

```text
A mutates
B assumes invariant
```

you may have separated one responsibility incorrectly.

A good boundary often keeps tightly coupled invariant state together.

---

# 241. SRP and Invariant Clusters

Remember:

```text
same invariant
+
same state ownership
```

is strong evidence for keeping behavior together.

---

# 242. SRP and Aggregate Boundaries

An aggregate can contain multiple operations because they preserve one consistency boundary.

That is not an SRP violation merely because the aggregate has many methods.

---

# 243. SRP and Business Workflows

A workflow can span multiple aggregates.

An application service may orchestrate them.

The workflow itself is one responsibility.

---

# 244. SRP and Eventual Consistency

When splitting responsibilities across services, you may introduce:

```text
eventual consistency
```

This is an architectural cost.

Do not split solely to satisfy a simplistic interpretation of SRP.

---

# 245. SRP and Distributed Transactions

Separating components can force:

```text
saga
outbox
compensation
reconciliation
```

These operational costs matter.

---

# 246. SRP and API Calls

One class may be cohesive but involve 5 external calls.

That can still be correct if its responsibility is:

```text
orchestrate checkout
```

But each external capability should have a focused adapter.

---

# 247. SRP and Network Failure

Focused adapters let you define:

```text
payment retry
email retry
inventory conflict
```

independently.

A giant integration class tends to conflate these failure modes.

---

# 248. SRP and Cache

Caching may be:

```text
repository responsibility
```

or:

```text
dedicated caching infrastructure
```

depending on whether cache semantics are part of the repository's contract.

Do not introduce a cache class solely because “one class should do one thing.”

---

# 249. SRP and Performance Tuning

Sometimes a cohesive hot path should remain together for locality.

Do not perform architectural decomposition without measuring runtime impact when performance is sensitive.

---

# 250. SRP and Generated Code

Generated serializers, ORM models, clients, or schemas may be large and mechanically broad.

Do not manually refactor generated artifacts to satisfy SRP.

Instead, place a clean boundary around them.

---

# 251. SRP and External SDKs

Third-party SDK objects often have broad capabilities.

Do not let their responsibility shape become your domain shape automatically.

Wrap them behind focused adapters where semantic isolation matters.

---

# 252. SRP and Vendor Change

If provider A and provider B expose different capabilities, your domain contract may still be:

```ts
PaymentGateway
```

The adapter owns provider-specific change.

---

# 253. SRP and Backward Compatibility

A compatibility layer may have a legitimate responsibility:

```text
translate old contract to new contract
```

Do not remove it merely because it feels like “extra code.”

It may protect consumers from change.

---

# 254. SRP and Deprecation

A deprecation adapter can own:

```text
old API translation
```

while the new implementation owns:

```text
new contract.
```

This is often cleaner than contaminating the core with dual semantics.

---

# 255. SRP and Migrations

A migration module may own:

```text
data transformation
```

while domain objects own:

```text
business invariants
```

Migration code should not become part of runtime domain responsibilities.

---

# 256. SRP and Observability Data

Telemetry mapping may be its own concern:

```text
DomainEvent
  → MetricsEvent
  → Trace attributes
```

This can evolve independently from business logic.

---

# 257. SRP and Audit Event Construction

If audit events are business-required facts:

```text
domain operation
→ emits fact
```

the domain may own fact creation.

Storage and transport belong elsewhere.

This is a useful distinction.

---

# 258. SRP and Authorization Policy

Authorization often belongs in a policy abstraction:

```ts
interface OrderAuthorization {
  canCancel(actor: Actor, order: Order): boolean;
}
```

The order may still own lifecycle invariants.

Do not make one concern responsible for all security behavior.

---

# 259. SRP and Identity

Authentication identity creation may be separate from:

```text
user profile management
```

even though both concern users.

---

# 260. SRP and Domain Language

Names should come from domain responsibility:

```text
TaxPolicy
InventoryReservation
OrderFulfillment
PaymentGateway
InvoiceRenderer
```

not implementation convenience:

```text
Manager
Helper
Processor
Util
Service
```

unless those names have clear semantics in context.

---

# 261. Common Misconception — “Every Class Should Have One Method”

False.

SRP is about reason to change, not method count.

---

# 262. Common Misconception — “Large Class = SRP Violation”

False.

Size is evidence, not proof.

---

# 263. Common Misconception — “Small Class = Good SRP”

False.

A tiny class can still mix responsibilities.

---

# 264. Common Misconception — “One Stakeholder = One Method”

False.

A single policy owner may require many methods.

---

# 265. Common Misconception — “One Dependency = One Responsibility”

False.

A coordinator may use many dependencies for one workflow.

---

# 266. Common Misconception — “No Duplicate Code = Good SRP”

False.

Duplication can be healthier than coupling unrelated change axes.

---

# 267. Common Misconception — “SRP Means No Orchestration”

False.

Orchestration is a legitimate responsibility.

---

# 268. Common Misconception — “SRP Requires Microservices”

False.

It applies to functions, classes, modules, packages, and services.

---

# 269. Common Misconception — “SRP Is Only OOP”

False.

Functional modules and pipelines can violate or follow the principle too.

---

# 270. Common Misconception — “SRP Means One Business Noun Per Class”

False.

A cohesive responsibility may naturally involve several domain nouns.

---

# 271. Common Mistake — Premature Extraction

Symptoms:

```text
many tiny classes
heavy constructor wiring
poor locality
difficulty tracing a feature
```

Fix:

```text
extract only when independent change responsibility is real.
```

---

# 272. Common Mistake — Utility Explosion

Everything becomes:

```text
Validator
Mapper
Formatter
Helper
Manager
Processor
```

without semantic boundaries.

This increases indirection without improving responsibility.

---

# 273. Common Mistake — Extracting State Away

A developer extracts methods but leaves the state behind.

Now:

```text
object owns data
helper owns behavior
```

and invariant protection becomes harder.

Move related state and behavior together when appropriate.

---

# 274. Common Mistake — Splitting by Technical Layer Inside Every Class

Example:

```text
OrderCalculator
OrderValidator
OrderMapper
OrderRepository
```

can be reasonable.

But:

```text
OrderFieldReader
OrderFieldWriter
OrderSubtotalReader
```

is fragmentation.

---

# 275. Common Mistake — Splitting by File Size

A 400-line file is not automatically a problem.

Use responsibility evidence.

---

# 276. Common Mistake — Splitting by Team Alone

Different teams may have temporary organizational boundaries.

Architecture should not blindly mirror every team boundary.

---

# 277. Common Mistake — Treating Shared Code as Sacred

A widely used function may still contain multiple responsibilities.

Usage count does not create cohesion.

---

# 278. Common Mistake — Refactoring Without Characterization Tests

A responsibility extraction can accidentally change:

```text
error semantics
ordering
retryability
side effects
```

Protect contracts first.

---

# 279. Common Mistake — Ignoring Transaction Boundaries

Moving persistence or payment code can change atomicity.

Review transactions after SRP refactors.

---

# 280. Common Mistake — Ignoring Lifecycle

Extracted collaborators can have different:

```text
singleton/request
scopes
```

which can introduce shared-state bugs.

---

# 281. Common Mistake — Hiding Responsibility in a Generic “Core” Module

A:

```text
core/
```

package can become a new god object.

Names do not solve SRP.

---

# 282. Common Mistake — Overusing Polymorphism

Not every responsibility split requires:

```text
interface
abstract class
strategy
factory
```

A simple function or module may be enough.

---

# 283. Common Mistake — Extracting Stable Code

If two behaviors always change together and share invariants, extraction may reduce cohesion.

---

# 284. Debugging Exercise 1 — God Service

Given:

```ts
class CommerceService {
  createOrder() {}
  calculateTax() {}
  reserveStock() {}
  chargePayment() {}
  sendReceipt() {}
  generateInvoicePdf() {}
}
```

Find all distinct change axes.

Then identify the one responsibility that can remain:

```text
orchestration
```

---

# 285. Debugging Exercise 2 — False Positive

Given:

```ts
class Money {
  add() {}
  subtract() {}
  multiply() {}
  allocate() {}
  equals() {}
  format() {}
}
```

Determine whether all methods belong together.

Do not split mechanically.

Ask which methods are:

```text
value semantics
money invariants
presentation
```

Then decide whether `format()` is a separate change reason in your architecture.

---

# 286. Debugging Exercise 3 — Hidden Persistence

```ts
class Order {
  async cancel() {
    this.status = "CANCELLED";
    await db.save(this);
  }
}
```

Questions:

```text
Does Order own persistence?
Does it need async lifecycle?
Should repository persistence remain outside?
What contract does cancel() promise?
```

---

# 287. Debugging Exercise 4 — Hidden Security

```ts
class Order {
  async cancel(actor: Actor) {
    if (!canCancel(actor, this)) throw new Error();
    this.status = "CANCELLED";
  }
}
```

This may still be cohesive if authorization is part of the lifecycle decision.

The key is whether the authorization rule is:

```text
order-specific business policy
```

or:

```text
central security policy.
```

---

# 288. Debugging Exercise 5 — Hidden Rendering

```ts
class Invoice {
  total() {}
  toHtml() {}
  toPdf() {}
}
```

Ask whether presentation is a separate reason to change.

If HTML/PDF templates evolve independently from invoice business rules, extraction is valuable.

---

# 289. Debugging Exercise 6 — Hidden Retry Policy

```ts
class PaymentService {
  charge() {}
  retry() {}
  backoff() {}
  renderErrorMessage() {}
}
```

Potential responsibilities:

```text
payment operation
retry policy
presentation
```

Refactor based on contract ownership.

---

# 290. Code Review Exercise

Review:

```ts
class ProductService {
  constructor(
    private db: Database,
    private redis: Redis,
    private mailer: Mailer,
    private pricing: PricingPolicy,
  ) {}

  async updateProduct(productId: string, price: number) {
    const product = await this.db.products.find(productId);

    if (!product) throw new Error("not found");

    if (price < product.cost) {
      throw new Error("invalid price");
    }

    product.price = price;
    await this.db.products.save(product);

    await this.redis.del(`product:${productId}`);
    await this.mailer.send("price changed");
  }
}
```

Identify:

```text
product mutation
pricing policy
persistence
cache invalidation
notification
workflow
```

Then decide which can be coordinated by one use-case responsibility and which should be delegated.

---

# 291. Code Review Answer Framework

Use:

```text
1. What is the object's current responsibility?
2. Which state/invariants does it own?
3. Which collaborators are policies?
4. Which behavior belongs to infrastructure?
5. Which side effects are part of the use case?
6. Which changes would require unrelated edits?
```

---

# 292. Interview Question 1

**What is the Single Responsibility Principle?**

A strong answer:

> A module should have one coherent reason to change. The principle is about grouping behavior around a meaningful responsibility and separating independent sources of change.

---

# 293. Interview Question 2

**Why is “one reason to change” better than “one responsibility”?**

Because “responsibility” can be interpreted too loosely.

Reason to change ties the design to:

```text
stakeholder
policy
requirement
variation
```

making the boundary more actionable.

---

# 294. Interview Question 3

**Is a 500-line class automatically an SRP violation?**

No.

Size may indicate complexity, but responsibility depends on semantic cohesion and change coupling.

---

# 295. Interview Question 4

**Can a class with 10 methods follow SRP?**

Yes.

If those methods support one coherent role, state, and change reason.

---

# 296. Interview Question 5

**Can a class with 3 methods violate SRP?**

Yes.

Three unrelated methods can encode three independent change reasons.

---

# 297. Interview Question 6

**How do you identify reasons to change?**

Ask:

```text
Which stakeholder owns the rule?
What requirement changes it?
Which policy varies?
Who would request the modification?
```

---

# 298. Interview Question 7

**Does SRP mean one class per database table?**

No.

Database structure and domain responsibility are different concerns.

---

# 299. Interview Question 8

**Does SRP imply one microservice per responsibility?**

No.

Service boundaries have network, deployment, consistency, reliability, and operational costs.

---

# 300. Interview Question 9

**What is the relationship between SRP and cohesion?**

SRP encourages grouping behavior that changes together.

Cohesion measures how strongly the parts of the module belong together.

---

# 301. Interview Question 10

**What is the relationship between SRP and coupling?**

Mixing independent responsibilities creates change coupling.

Separating them can reduce the blast radius of changes.

---

# 302. Interview Question 11

**Can an application service depend on many classes and still follow SRP?**

Yes.

If its responsibility is one coherent use-case orchestration.

---

# 303. Interview Question 12

**Why is a generic `Manager` class often suspicious?**

The name can conceal multiple unrelated responsibilities that accumulated over time.

---

# 304. Interview Question 13

**When should you not split a class?**

When behaviors share:

```text
state
invariants
domain meaning
change drivers
```

and extraction would add complexity without meaningful isolation.

---

# 305. Interview Question 14

**How can Git history help identify SRP problems?**

Co-change analysis reveals whether unrelated requirements repeatedly modify the same module.

---

# 306. Interview Question 15

**How can team ownership reveal SRP issues?**

If independent teams repeatedly change different regions of one module, it may indicate multiple responsibility boundaries.

---

# 307. Interview Question 16

**How is SRP related to LSP?**

SRP creates focused abstractions.

LSP asks whether substitutions preserve behavioral contracts.

They solve different dimensions of design quality.

---

# 308. Interview Question 17

**How is SRP related to ISP?**

SRP focuses on implementation responsibility.

ISP focuses on client-facing interface segregation.

Both reduce unnecessary change coupling.

---

# 309. Interview Question 18

**How is SRP related to DIP?**

DIP allows high-level policy to depend on stable abstractions.

SRP helps make those abstractions semantically focused.

---

# 310. Interview Question 19

**What is the biggest SRP mistake?**

Treating it as a class-size or method-count rule instead of a change-coupling principle.

---

# 311. Interview Question 20

**What is the principal-engineer version of SRP?**

> Separate responsibilities when doing so meaningfully reduces independent change coupling, while preserving cohesive state ownership and avoiding unnecessary architectural complexity.

---

# 312. Predict-the-Output Exercise 1

```ts
const shared = { count: 0 };

class A {
  increment() {
    shared.count++;
  }
}

class B {
  value() {
    return shared.count;
  }
}

const a = new A();
const b = new B();

a.increment();
console.log(b.value());
```

### Prediction

```text
1
```

### Lesson

Shared state can couple seemingly separate responsibilities.

---

# 313. Predict-the-Output Exercise 2

```ts
class Invoice {
  total = 100;

  toString() {
    return String(this.total);
  }
}

const invoice = new Invoice();
console.log(invoice.toString());
```

### Prediction

```text
100
```

Now ask:

```text
Does string presentation belong to Invoice?
```

The answer depends on whether the representation is stable value semantics or an independently changing external presentation contract.

---

# 314. Predict-the-Output Exercise 3

```ts
class TaxPolicy {
  rate = 0.18;

  calculate(amount: number) {
    return amount * this.rate;
  }
}

const policy = new TaxPolicy();
console.log(policy.calculate(100));
```

### Prediction

```text
18
```

One policy object can contain state and behavior while remaining cohesive.

---

# 315. Predict-the-Output Exercise 4

```ts
class Service {
  constructor(private readonly fn: () => number) {}

  run() {
    return this.fn();
  }
}

const s = new Service(() => 42);
console.log(s.run());
```

### Prediction

```text
42
```

Delegation does not automatically create an SRP violation.

---

# 316. Mastery Exercise 1 — Responsibility Map

Take a real service you know.

Create:

```text
method
state
dependency
stakeholder
reason to change
candidate owner
```

for every public method.

Identify responsibility clusters.

---

# 317. Mastery Exercise 2 — God Service Refactor

Take a service with at least:

```text
10 methods
```

Identify change axes.

Refactor only the meaningful independent responsibilities.

Do not split by method count.

---

# 318. Mastery Exercise 3 — Jewellery ERP

Model:

```text
Product
Inventory
Pricing
Billing
Notification
Audit
```

For each, define:

```text
responsibility
state owner
invariants
change authority
public contract
```

---

# 319. Mastery Exercise 4 — Coordinator Defense

Design:

```ts
CreateSaleUseCase
```

that depends on:

```text
pricing
inventory
payment
repository
event publisher
```

Defend why this class does or does not follow SRP.

Your answer should explain:

```text
orchestration responsibility
vs
delegated policy responsibilities
```

---

# 320. Mastery Exercise 5 — False Positive

Find a large class that looks like an SRP violation.

Prove why it is actually cohesive.

This is important because mature design includes knowing when **not** to refactor.

---

# 321. Mastery Exercise 6 — False Negative

Find a small class that looks clean.

Prove that it has multiple change reasons.

This builds sensitivity to semantic rather than visual design.

---

# 322. Mastery Exercise 7 — Change History

Use Git history on a project.

Find:

```text
files changed together
authors
teams
feature branches
```

Identify modules with suspicious change coupling.

---

# 323. Mastery Exercise 8 — Contract Preservation

Take a large legacy class and extract one responsibility.

Write characterization tests first.

Prove that:

```text
outputs
errors
ordering
side effects
```

remain stable.

---

# 324. Mastery Exercise 9 — Security Boundary

Design:

```text
OrderCancellation
OrderAuthorization
AuditRecorder
```

Explain which responsibility each owns.

---

# 325. Mastery Exercise 10 — Performance Trade-off

Take a hot-path algorithm and consider two designs:

```text
monolithic cohesive implementation
vs
multiple tiny collaborators
```

Measure:

```text
CPU
allocation
readability
change isolation
```

Choose the better trade-off.

---

# 326. Principal Design Exercise — Inventory Reservation

Design:

```text
ReserveStockUseCase
InventoryAggregate
InventoryRepository
AuthorizationPolicy
AuditPublisher
```

Explain:

```text
SRP boundary
contract boundary
invariant ownership
transaction boundary
concurrency boundary
```

---

# 327. Principal Design Exercise — Payment

Design:

```text
CapturePaymentUseCase
PaymentGateway
PaymentPolicy
PaymentRepository
IdempotencyStore
```

Explain which responsibility belongs where.

Do not hide:

```text
retry
idempotency
authorization
provider translation
```

inside a generic service.

---

# 328. Principal Design Exercise — Multi-Tenant ERP

Design:

```text
TenantContext
AuthorizationPolicy
BranchInventory
InventoryRepository
AuditPublisher
```

Explain:

```text
which object owns tenant isolation
which object owns inventory invariants
which object owns audit transport
```

---

# 329. Principal Design Exercise — Invoice

Design:

```text
Invoice
TaxPolicy
InvoiceRenderer
InvoiceRepository
InvoiceNotifier
```

Then answer:

```text
Why does Invoice remain rich?
Why are renderer and repository separate?
Which invariants stay inside Invoice?
```

---

# 330. Principal Decision Framework

Before extracting a responsibility, evaluate:

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
What meaningful change coupling disappears?
What complexity appears?
Which trade-off is worth it?
```

---

# 331. SRP Scoring Heuristic

For a candidate split, rate 1–5:

```text
independent change source
stakeholder distinction
policy volatility distinction
invariant distinction
contract distinction
test isolation benefit
deployment benefit
security benefit
```

Then rate the costs:

```text
indirection
wiring
runtime overhead
cognitive load
transaction complexity
distributed complexity
```

Use the matrix for judgment, not as a mathematical law.

---

# 332. Contract + SRP Review

For every extraction ask:

```text
Did the contract change?
Did invariant ownership change?
Did error semantics change?
Did transaction boundaries change?
Did retry behavior change?
Did security authority change?
```

This combines Chapters 17–19.

---

# 333. Chapter 19 — Mastery Gate

Use the repository's full mastery progression:

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

Reading is not mastery. A concept is mastered only when you can explain the reasoning, predict behavior, implement the design, debug failures, apply it to a new domain, compare alternatives, and defend the trade-offs.

---

# 334. Completion Criteria

Do not mark Chapter 19 mastered until you can:

- Define SRP as a reason-to-change principle.
- Identify stakeholders and change authorities.
- Distinguish responsibility from task and implementation detail.
- Detect responsibility drift and god-object growth.
- Distinguish a cohesive large class from a genuinely mixed-responsibility class.
- Refactor responsibility boundaries without unnecessary fragmentation.
- Preserve contracts, invariants, transaction semantics, security, and side-effect ordering during refactoring.
- Apply SRP to functions, classes, modules, packages, application services, domain services, event consumers, and larger capabilities.
- Use Git history and co-change behavior as evidence.
- Defend a responsibility boundary using correctness, performance, memory, security, reliability, maintainability, scalability, observability, developer experience, operational complexity, and future change.

Status:

```text
[ ] Not Started
[~] In Progress
[?] Needs Revision
[+] Completed
[*] Mastered
```

---

# Chapter 19 — Key Takeaways

```text
SRP is about reason to change.

Responsibility is broader than a single method.

One responsibility can contain many cohesive behaviors.

Class size is not the definition of SRP.

Method count is not the definition of SRP.

Stakeholders and change authorities reveal useful boundaries.

State + invariants + behavior often belong together.

Coordinators can have many dependencies and still follow SRP.

Over-separation creates indirection and operational cost.

Under-separation creates change coupling and blast radius.

Use history, ownership, contracts, invariants, and volatility as evidence.

Refactor toward meaningful responsibility boundaries, not aesthetic smallness.
```

---

# Chapter 19 — Concept Connections

Backward connections:

```text
Chapter 12 → Cohesion and coupling
Chapter 13 → Responsibility-driven design
Chapter 14 → GRASP
Chapter 15 → Cohesion-first decomposition
Chapter 16 → Coupling-first dependency design
Chapter 17 → Design smells and refactoring
Chapter 18 → Stable contracts and invariants
```

Forward connections:

```text
Chapter 20 → Open/Closed Principle
Chapter 21 → Liskov Substitution Principle
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

# Chapter 19 — Dependency Graph

```text
Responsibility
    ↓
Reason to Change
    ↓
Cohesion
    ↓
Change Coupling
    ↓
Boundary
    ↓
Contract
    ↓
Invariant Ownership
    ↓
Composition
    ↓
Stable Dependency
    ↓
Safe Evolution
```

---

# Chapter 19 — Revision / Retrieval Record

## Session Record

```text
Status:
[~] In Progress

Can define:
- SRP
- reason to change
- responsibility
- stakeholder
- actor
- change axis

Can identify:
- god service
- controller blob
- responsibility drift
- fragmentation

Needs implementation:
- responsibility matrix
- legacy extraction
- characterization tests
- jewellery ERP responsibility model
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

# Chapter 19 — Completion Snapshot

## Core Theory

```text
[+] SRP
[+] reason to change
[+] responsibility vs task
[+] stakeholder/actor analysis
[+] change-axis analysis
[+] cohesion connection
[+] coupling connection
[+] information hiding connection
[+] composition
[+] orchestration
```

## Refactoring

```text
[+] extract class
[+] move method
[+] move field
[+] extract policy
[+] extract adapter
[+] extract repository
[+] extract renderer
[+] contract preservation
```

## Production

```text
[+] security boundaries
[+] observability
[+] persistence
[+] distributed systems
[+] retries
[+] idempotency
[+] transaction boundaries
[+] multi-tenancy
```

## Principal Judgment

```text
[+] avoid over-fragmentation
[+] evaluate extraction cost
[+] evaluate change blast radius
[+] use Git history as evidence
[+] distinguish coordinator from god object
[+] defend the boundary using trade-offs
```

---

# Chapter 19 — Canonical Source Discipline

When verifying this chapter:

```text
SOLID principle meaning
  → prioritize original / canonical design-principle sources when available

JavaScript / TypeScript mechanisms
  → ECMAScript and TypeScript documentation

Framework-specific behavior
  → framework documentation

Database/runtime behavior
  → engine/runtime documentation

Architecture/domain rules
  → actual application requirements
```

Do not treat a simplified blog definition as stronger than the design reasoning behind the principle.

---

# Chapter 19 — Compact Mental Checklist

```text
□ What is the responsibility?
□ What is the reason to change?
□ Who owns the rule?
□ Which stakeholder can request the change?
□ Which state and invariants belong here?
□ Which dependencies support the same responsibility?
□ Which behavior belongs elsewhere?
□ Is this a coordinator or a god object?
□ Am I extracting because of semantics or class size?
□ What contract must remain stable?
□ Did transaction semantics change?
□ Did error semantics change?
□ Did security authority change?
□ Did the extraction reduce meaningful coupling?
□ What complexity did the extraction add?
```

---

# Chapter 19 — Master Rule

> **Keep behavior together when it belongs to the same responsibility and changes for the same meaningful reason; separate it when independent change forces otherwise unrelated code to move together.**
