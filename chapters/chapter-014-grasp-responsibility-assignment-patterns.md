# Chapter 14 — GRASP — Responsibility Assignment Patterns

> **Part:** C — Responsibility-Driven Design
> **Prerequisite:** Chapter 13 — Responsibility-Driven Object Design
> **Next:** Chapter 15 — Cohesion-First Object Decomposition
> **Status:** `[+] Completed`

---

## Chapter Position

Chapter 13 established the question:

> **Who should know? Who should do? Who should coordinate?**

GRASP formalizes recurring answers to that question. It is a responsibility-assignment vocabulary, not a JavaScript library, framework, or rigid class-generation recipe.

The nine commonly taught GRASP patterns are:

1. Information Expert
2. Creator
3. Controller
4. Low Coupling
5. High Cohesion
6. Polymorphism
7. Pure Fabrication
8. Indirection
9. Protected Variations

The patterns work together. A good design does not apply them mechanically.

---

# 1. Learning Objectives

By the end of this chapter you should be able to:

- explain all nine GRASP patterns
- choose candidate responsibility owners from knowledge, state, authority and invariants
- distinguish system-operation responsibility from domain behavior
- use GRASP with JavaScript and TypeScript
- combine GRASP with cohesion, coupling, encapsulation, SOLID and DDD
- model responsibilities with CRC cards and interaction diagrams
- refactor god objects, controller blobs, service blobs and anemic models
- reason about variation, external dependencies, testing, security and reliability
- defend design choices during LLD interviews
- apply GRASP to a multi-tenant jewellery ERP workflow

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
- responsibility-driven object design

Chapter 13 is the immediate prerequisite.

---

# 3. What Is GRASP?

GRASP stands for **General Responsibility Assignment Software Patterns**.

The central purpose is to help answer:

```text
Who should be responsible for this?
```

Typical design questions become:

```text
Who has the information?
Who owns the state?
Who should create this object?
Who receives the system operation?
Who should coordinate this workflow?
Where does varying behavior live?
Where should external dependencies be isolated?
Where should technical responsibilities live?
```

GRASP turns these questions into reusable heuristics.

---

# 4. GRASP Is Not a Framework

There is no:

```text
npm install grasp
```

GRASP does not provide decorators, base classes, runtime behavior, or a package structure.

It changes how you **reason about design**.

---

# 5. GRASP Is Not a Rigid Rule Set

A pattern is a strong heuristic, not a mathematical law.

Example:

```ts
order.total()
```

may be excellent because `Order` owns its lines.

But if total pricing depends on several independently changing policies, a pricing abstraction may produce better cohesion.

Always evaluate:

```text
knowledge
authority
invariants
cohesion
coupling
variation
testability
change
```

---

# 6. The Nine Patterns

| Pattern | Core Question |
|---|---|
| Information Expert | Who has the needed information? |
| Creator | Who should create this object? |
| Controller | Who handles the system operation? |
| Low Coupling | How can unnecessary dependency be reduced? |
| High Cohesion | How can responsibilities stay focused? |
| Polymorphism | Where should type-varying behavior live? |
| Pure Fabrication | What focused object should hold a responsibility with no natural domain owner? |
| Indirection | What intermediary can reduce direct coupling? |
| Protected Variations | What stable boundary can protect against likely change? |

---

# 7. A Practical GRASP Sequence

Use this workflow:

```text
system operation
    ↓
responsibilities
    ↓
state + invariants
    ↓
Information Expert
    ↓
Creator
    ↓
Controller
    ↓
High Cohesion
    ↓
Low Coupling
    ↓
Polymorphism
    ↓
Pure Fabrication
    ↓
Indirection
    ↓
Protected Variations
    ↓
scenario validation
```

This is a thinking aid, not a mandatory ritual.

---

# 8. Pattern 1 — Information Expert

Information Expert says:

> Assign a responsibility to the class that has the information necessary to fulfill it.

Example:

```ts
class OrderLine {
  constructor(
    readonly unitPrice: number,
    readonly quantity: number
  ) {}

  subtotal() {
    return this.unitPrice * this.quantity;
  }
}
```

`OrderLine` has the information needed for its subtotal.

---

# 9. Information Expert — Why It Helps

Placing behavior near relevant information often improves:

```text
encapsulation
cohesion
readability
testability
change localization
```

It also reduces external inspection of internal representation.

---

# 10. Information Expert — State Ownership

The heuristic becomes stronger when the object also owns the affected state.

```ts
class InventoryItem {
  #available = 10;

  reserve(quantity: number) {
    if (quantity <= 0) {
      throw new Error("Quantity must be positive");
    }

    if (quantity > this.#available) {
      throw new Error("Insufficient stock");
    }

    this.#available -= quantity;
  }
}
```

The object knows the state, changes the state, and protects the invariant.

---

# 11. Information Expert — Do Not Overapply It

Do not interpret Information Expert as:

> Put every responsibility involving this object's data on the object.

If `Order` contains a customer reference, it does not become the expert for:

```text
password hashing
email delivery
PDF rendering
database SQL
payment provider APIs
```

The behavior must fit the concept.

---

# 12. Information Expert — Distributed Knowledge

Some decisions require several sources of information.

Example:

```text
Order
Customer
TaxPolicy
Jurisdiction
```

Tax may therefore belong to a policy or domain service rather than being forced into `Order`.

Information Expert is evaluated against cohesion and domain meaning.

---

# 13. Information Expert — Example

```ts
class Order {
  constructor(
    private readonly lines: OrderLine[]
  ) {}

  total(): number {
    return this.lines.reduce(
      (sum, line) => sum + line.subtotal(),
      0
    );
  }
}
```

Two responsibility levels exist:

```text
OrderLine → line subtotal
Order     → aggregate total
```

This is often more expressive than exposing `price * quantity` to every caller.

---

# 14. Information Expert — Anti-Pattern

Weak:

```ts
class OrderCalculator {
  total(order: Order) {
    return order.lines.reduce(
      (sum, line) => sum + line.price * line.quantity,
      0
    );
  }
}
```

This can be fine for a deliberate pricing boundary.

It becomes weak when every basic domain behavior is moved into generic calculators and services.

---

# 15. Information Expert — Interview Answer

A strong answer:

> "I start with the object that owns the information needed for the responsibility. If it also owns the affected state and invariant, that is stronger evidence. I still verify cohesion, coupling, and variation so I do not create a god object."

---

# 16. Pattern 2 — Creator

Creator asks:

> Which object should create this object?

Strong candidates often:

- contain it
- aggregate it
- closely use it
- have initialization data
- control its lifecycle

Example:

```ts
class Cart {
  #lines: CartLine[] = [];

  add(productId: string, quantity: number) {
    const line = new CartLine(productId, quantity);
    this.#lines.push(line);
  }
}
```

---

# 17. Creator — Lifecycle Reasoning

If:

```text
A owns B
```

then A is often a good creator of B.

But construction complexity can change the answer.

Consider:

```text
simple construction
vs
multiple dependencies
vs
polymorphic creation
vs
infrastructure-dependent creation
```

---

# 18. Creator — When a Factory Wins

```ts
class SaleFactory {
  constructor(
    private readonly clock: Clock,
    private readonly ids: IdGenerator
  ) {}

  create(customerId: string) {
    return new Sale(
      this.ids.generate(),
      customerId,
      this.clock.now()
    );
  }
}
```

The factory becomes useful because creation depends on external capabilities.

---

# 19. Creator — Do Not Create Factories Everywhere

This:

```ts
const point = new Point(x, y);
```

does not need:

```text
PointFactory
PointFactoryService
PointFactoryProvider
```

Factories earn their existence through meaningful creation policy.

---

# 20. Creator — Polymorphic Creation

Factories can select implementations.

```ts
class PaymentMethodFactory {
  create(type: PaymentType): PaymentMethod {
    switch (type) {
      case "CARD":
        return new CardPayment();
      case "UPI":
        return new UpiPayment();
      default:
        throw new Error("Unsupported type");
    }
  }
}
```

If the variants grow frequently, a registry or polymorphic construction mechanism may reduce conditional growth.

---

# 21. Pattern 3 — Controller

Controller asks:

> Who should receive a system-level operation?

A controller is usually a boundary object.

```ts
class CreateOrderController {
  constructor(
    private readonly useCase: CreateOrderUseCase
  ) {}

  async handle(request: CreateOrderRequest) {
    return this.useCase.execute(request);
  }
}
```

The controller translates external interaction into an application operation.

---

# 22. Controller — What It Should Not Become

A controller should not normally accumulate:

```text
business pricing
state transitions
stock rules
SQL
payment SDK logic
PDF generation
notification delivery
```

A controller blob is a responsibility problem.

---

# 23. Controller — System Operation

Distinguish:

```text
System operation:
    createOrder(command)

Domain operation:
    order.confirm()
```

The controller can receive the system operation.

The order can own the domain transition.

---

# 24. Controller — Application Service

A common structure:

```text
Controller
    ↓
Application Service / Use Case
    ↓
Domain Objects
    ↓
Infrastructure Ports
```

This is not mandatory, but it gives clear responsibility layering.

---

# 25. Controller — Message and CLI Variants

GRASP Controller applies beyond HTTP.

Entry boundaries can include:

```text
HTTP controller
message consumer
CLI command
scheduled job handler
GraphQL resolver
RPC endpoint
```

The exact framework name is less important than the responsibility.

---

# 26. Pattern 4 — Low Coupling

Low Coupling asks:

> How can unnecessary dependency be minimized?

It targets accidental dependency on:

```text
object structure
concrete providers
frameworks
databases
globals
volatile APIs
implementation details
```

Low coupling does not mean no coupling.

Collaboration is necessary.

---

# 27. Low Coupling — Direct Dependency

Weak:

```ts
class Order {
  async pay() {
    await stripe.charges.create(...);
  }
}
```

The domain object now knows a provider.

Stronger:

```ts
interface PaymentGateway {
  charge(input: ChargeInput): Promise<ChargeResult>;
}
```

The provider can sit behind an adapter.

---

# 28. Low Coupling — Necessary Coupling

This is meaningful:

```text
Order → OrderLine
```

because an order naturally owns its lines.

The target is not "remove dependency."

The target is:

> Remove dependency that exists only because of implementation choices.

---

# 29. Low Coupling — Fan-Out

High fan-out is a review signal.

```text
CheckoutService
├── Stripe
├── Redis
├── PostgreSQL
├── SMTP
├── S3
└── Analytics
```

This may be legitimate orchestration, but it can also indicate responsibility overload.

---

# 30. Low Coupling — Fan-In

High fan-in means many components depend on one component.

That may be healthy for stable capabilities:

```text
Money
Clock
IdGenerator
Policy
```

But a volatile high-fan-in dependency can create large change amplification.

---

# 31. Low Coupling — TypeScript

```ts
interface PaymentGateway {
  charge(input: ChargeInput): Promise<ChargeResult>;
}

class CheckoutUseCase {
  constructor(
    private readonly payment: PaymentGateway
  ) {}
}
```

The use case depends on a capability rather than a provider.

This also supports Protected Variations.

---

# 32. Low Coupling — JavaScript

Useful mechanisms include:

```text
modules
closures
functions
composition
dependency injection
narrow capability objects
factory functions
```

The language does not create the architecture for you.

---

# 33. Pattern 5 — High Cohesion

High Cohesion asks:

> Do the responsibilities of this class belong together?

Example:

```text
Order
    addLine
    removeLine
    total
    confirm
    cancel
```

These responsibilities share an understandable domain center.

---

# 34. High Cohesion — Unrelated Responsibilities

Weak:

```text
Order
    confirm
    exportPdf
    sendEmail
    authenticateUser
    resizeImage
```

The shared noun does not create cohesion.

---

# 35. High Cohesion — Avoid Fragmentation

The opposite mistake is:

```text
OrderLineAdder
OrderLineRemover
OrderTotalCalculator
OrderConfirmHandler
```

for trivial behavior.

High Cohesion does not mean:

```text
one method = one class
```

It means related responsibilities remain together.

---

# 36. High Cohesion — Service Blob

A service with:

```text
createOrder
calculateTax
sendEmail
resetPassword
generateReport
```

has unrelated reasons to change.

Split responsibility clusters first.

---

# 37. High Cohesion — Aggregate Roots

An aggregate root can have several operations and still be cohesive.

```text
Order
    addLine
    removeLine
    confirm
    cancel
    markPaid
```

The common center is order lifecycle and consistency.

---

# 38. Pattern 6 — Polymorphism

Polymorphism asks:

> Where should behavior that varies by type live?

Instead of:

```ts
if (type === "CARD") ...
if (type === "UPI") ...
if (type === "CASH") ...
```

consider:

```ts
interface PaymentMethod {
  pay(amount: Money): Promise<PaymentResult>;
}
```

Each variant owns its behavior.

---

# 39. Polymorphism — Stable Contract

The stable concept is:

```text
perform payment
```

The implementation can vary:

```text
CardPayment
UpiPayment
CashPayment
```

The caller depends on behavior, not concrete representation.

---

# 40. Polymorphism — Composition

Polymorphism does not require inheritance.

```ts
interface DiscountPolicy {
  calculate(order: Order): Money;
}

const weekendDiscount: DiscountPolicy = {
  calculate(order) {
    return Money.of(100, "INR");
  }
};
```

This fits structural typing and composition.

---

# 41. Polymorphism — Conditional Logic

A switch can signal variation:

```ts
switch (shippingType) {
  case "STANDARD":
  case "EXPRESS":
  case "INTERNATIONAL":
}
```

But not every switch should become a class hierarchy.

Ask:

```text
Will variants grow?
Do they change independently?
Is a stable capability valuable?
```

---

# 42. Polymorphism — Behavioral Contract

Polymorphic implementations must preserve the responsibility contract.

That includes compatible:

```text
preconditions
postconditions
failure semantics
output meaning
```

Polymorphism without a stable contract is unsafe.

---

# 43. Pattern 7 — Pure Fabrication

Pure Fabrication asks:

> When no domain object is a natural owner, should we create a focused non-domain object?

Examples:

```text
PasswordHasher
EmailSender
AuditLogger
ReportExporter
Repository
ProviderAdapter
```

These objects can improve cohesion and decoupling.

---

# 44. Pure Fabrication — Technical Responsibility

Not every useful responsibility is a domain noun.

Technical capabilities still need homes.

Instead of:

```ts
class Customer {
  async sendWelcomeEmail() { ... }
}
```

use:

```ts
class WelcomeEmailSender {
  constructor(private readonly mailer: Mailer) {}

  send(customer: Customer) {
    return this.mailer.send({
      to: customer.email,
      template: "welcome"
    });
  }
}
```

---

# 45. Pure Fabrication — Utility Trap

Do not create:

```text
EverythingUtils
CommonService
SystemManager
UniversalHelper
```

A fabricated object must have a focused responsibility.

---

# 46. Pattern 8 — Indirection

Indirection asks:

> What intermediary can reduce direct coupling?

Example:

```text
Checkout
    ↓
PaymentGateway
    ↓
StripeAdapter
    ↓
Stripe SDK
```

The intermediary absorbs volatile details.

---

# 47. Indirection — Cost

Indirection adds:

```text
types
files
navigation
configuration
possible runtime calls
```

It is worth the cost when it buys:

```text
replaceability
translation
testability
failure isolation
dependency control
```

---

# 48. Indirection — Adapter

```ts
interface PaymentGateway {
  charge(input: ChargeInput): Promise<ChargeResult>;
}

class StripePaymentGateway implements PaymentGateway {
  constructor(private readonly stripe: StripeClient) {}

  async charge(input: ChargeInput) {
    const result = await this.stripe.charge({
      amount: input.amount,
      currency: input.currency
    });

    return mapStripeResult(result);
  }
}
```

The adapter is an explicit boundary.

---

# 49. Pattern 9 — Protected Variations

Protected Variations asks:

> What is likely to change, and what stable boundary can protect clients?

Typical variation points:

```text
payment provider
tax policy
pricing algorithm
shipping algorithm
clock
ID generator
database technology
notification provider
external API
```

---

# 50. Protected Variations — Stable Concept

Example:

```text
PaymentGateway
    stable capability

StripePaymentGateway
    volatile implementation
```

The stable boundary protects the client.

---

# 51. Protected Variations — Do Not Abstract Imaginary Change

This:

```ts
function formatOrderNumber(id: string) {
  return `ORD-${id}`;
}
```

does not automatically need:

```text
OrderNumberFormatter interface
```

Abstract evidence-driven variation, not hypothetical change.

---

# 52. Protected Variations — Clock

```ts
interface Clock {
  now(): Date;
}
```

Production:

```ts
class SystemClock implements Clock {
  now() {
    return new Date();
  }
}
```

Test:

```ts
const fixedClock: Clock = {
  now: () => new Date("2026-01-01T00:00:00Z")
};
```

Time is a real variation point when deterministic testing matters.

---

# 53. Pattern Relationships — Expert + Cohesion

Information Expert proposes a candidate.

High Cohesion asks whether the candidate fits.

```text
Who knows?
    ↓
Would this behavior belong here?
```

Both must be answered.

---

# 54. Pattern Relationships — Expert + Coupling

A behavior located near its information can reduce data exposure.

Instead of:

```ts
customer.address.country.code
```

a domain abstraction might expose:

```ts
customer.taxRegion()
```

The client depends less on representation structure.

---

# 55. Pattern Relationships — Creator + Cohesion

Creation should align with lifecycle responsibility.

Good:

```text
Cart → CartLine
```

Potentially weak:

```text
GlobalFactory → every object
```

Centralized creation can become a low-cohesion dependency hub.

---

# 56. Pattern Relationships — Controller + Low Coupling

A controller absorbs transport-specific details.

```text
HTTP
  ↓
Controller
  ↓
Use Case
```

The use case does not need to understand HTTP request objects.

---

# 57. Pattern Relationships — Polymorphism + Protected Variations

These are natural partners.

```text
Protected Variations
    isolates the point of change.

Polymorphism
    lets each variant implement the changing behavior.
```

Example:

```text
TaxPolicy
├── DomesticTaxPolicy
├── ExportTaxPolicy
└── ExemptTaxPolicy
```

---

# 58. Pattern Relationships — Indirection + Low Coupling

Indirection is often the mechanism that lowers coupling.

```text
Application
    ↓
PaymentGateway
    ↓
ProviderAdapter
```

The stable intermediary absorbs provider changes.

---

# 59. Pattern Relationships — Fabrication + Cohesion

A Pure Fabrication should solve a focused responsibility.

Good:

```text
AuditLogger
```

Bad:

```text
EverythingLogger
    tax
    pricing
    persistence
    email
```

---

# 60. GRASP and Invariants

Invariants are powerful clues.

Example:

```text
PAID cannot become CANCELLED
```

The order/invoice that owns status is a strong candidate for:

```ts
cancel()
```

because it can enforce the transition.

---

# 61. GRASP and Tell, Don't Ask

Weak:

```ts
if (order.status === "PENDING") {
  order.status = "CONFIRMED";
}
```

Better:

```ts
order.confirm();
```

The state owner makes the decision.

---

# 62. GRASP and Law of Demeter

GRASP's low-coupling thinking complements Law of Demeter.

Risky:

```ts
sale.customer.account.branch.taxProfile.rate
```

Potentially better:

```ts
sale.taxRate();
```

Do not turn Law of Demeter into a ban on every fluent chain.

The goal is reducing dependence on object structure.

---

# 63. GRASP and SOLID

Useful connections:

```text
High Cohesion ↔ SRP
Protected Variations ↔ OCP
Polymorphism ↔ OCP / LSP
Indirection ↔ DIP
Low Coupling ↔ DIP
```

GRASP gives responsibility-oriented reasoning.

SOLID gives broader design principles.

---

# 64. GRASP and DDD

GRASP helps assign behavior among:

```text
entities
value objects
domain services
factories
repositories
application services
```

DDD adds boundaries such as:

```text
aggregates
bounded contexts
domain events
```

GRASP helps decide what responsibilities live inside those concepts.

---

# 65. GRASP and Aggregates

An aggregate root is often responsible for:

```text
aggregate consistency
invariant coordination
state transitions spanning child objects
```

But child objects should retain local behavior where appropriate.

Do not turn the root into a god object.

---

# 66. GRASP and Value Objects

Value objects are natural Information Experts for their value semantics.

```ts
class Money {
  add(other: Money) {
    if (other.currency !== this.currency) {
      throw new Error("Currency mismatch");
    }

    return new Money(
      this.amount + other.amount,
      this.currency
    );
  }

  constructor(
    readonly amount: number,
    readonly currency: string
  ) {}
}
```

---

# 67. GRASP and Entities

Entities commonly own:

```text
identity
state
invariants
lifecycle
behavior
```

They do not automatically own:

```text
database access
network calls
logging infrastructure
```

---

# 68. GRASP and Domain Services

A domain service is useful when:

```text
behavior is domain-significant
+
multiple domain concepts participate
+
no natural single owner exists
```

Do not use "service" merely because a method looks complicated.

---

# 69. GRASP and Application Services

Application services commonly coordinate:

```text
load
authorize
invoke domain behavior
coordinate infrastructure
persist
publish
```

They should not automatically become the home of every business rule.

---

# 70. GRASP and Repositories

Repositories are often Pure Fabrication plus Indirection around persistence.

```ts
interface OrderRepository {
  findById(id: OrderId): Promise<Order | null>;
  save(order: Order): Promise<void>;
}
```

The domain should not need SQL or ORM details.

---

# 71. GRASP and Factories

Factories combine Creator with useful boundary mechanisms when construction is:

```text
complex
polymorphic
dependency-heavy
configuration-driven
```

Simple object construction should remain simple.

---

# 72. GRASP and Adapters

Adapters commonly combine:

```text
Pure Fabrication
Indirection
Protected Variations
```

They translate external representations to internal capabilities.

---

# 73. GRASP and Policies

Policies are strong candidates when decisions vary:

```text
TaxPolicy
DiscountPolicy
PricingPolicy
ShippingPolicy
ApprovalPolicy
```

They often use Polymorphism and Protected Variations.

---

# 74. JavaScript — GRASP Without Classes

GRASP does not require classes.

A function may own a responsibility:

```ts
function calculateSubtotal(
  price: number,
  quantity: number
) {
  return price * quantity;
}
```

Use a class only when identity, state, lifecycle, encapsulation or collaboration justify it.

---

# 75. JavaScript — Closures as Responsibility Owners

```ts
function createRateLimiter(limit: number) {
  let count = 0;

  return {
    allow() {
      if (count >= limit) return false;
      count++;
      return true;
    }
  };
}
```

The closure owns its state and behavior without a class.

---

# 76. JavaScript — Private Fields

```ts
class InventoryItem {
  #available = 10;

  reserve(quantity: number) {
    // enforce invariant
  }
}
```

GRASP decides ownership.

The private field helps enforce it at runtime.

---

# 77. TypeScript — Capability Interfaces

```ts
interface Clock {
  now(): Date;
}

interface IdGenerator {
  generate(): string;
}

interface PaymentGateway {
  charge(input: ChargeInput): Promise<ChargeResult>;
}
```

The interfaces communicate responsibilities rather than implementations.

---

# 78. TypeScript — Structural Typing

A capability can be satisfied by:

```text
class
object literal
fake
adapter
wrapper
```

because TypeScript is structurally typed.

This works especially well with narrow responsibilities.

---

# 79. Runtime vs Compile-Time

TypeScript interfaces disappear at runtime.

Therefore external boundaries still need runtime validation for:

```text
HTTP
queues
webhooks
external APIs
database data
untrusted input
```

GRASP does not replace runtime contracts.

---

# 80. GRASP and HTTP

A reasonable flow:

```text
HTTP request
    ↓
Controller
    ↓
Use Case
    ↓
Domain
    ↓
Ports
    ↓
Infrastructure
```

Each layer has a different responsibility.

---

# 81. GRASP and Message Consumers

For a queue consumer:

```text
Consumer
    decode message

Use Case
    coordinate

Domain
    decide

Repository
    persist

Retry/Ack policy
    manage delivery outcome
```

Do not make the consumer the domain model.

---

# 82. GRASP and Scheduled Jobs

```text
Scheduler
    determines when

Job Handler
    invokes task

Use Case
    coordinates business workflow

Domain
    enforces rules
```

This distinction prevents scheduling concerns from leaking into domain objects.

---

# 83. GRASP and NestJS

A possible NestJS structure:

```text
orders/
  controllers/
  application/
  domain/
  infrastructure/
```

Possible roles:

```text
controller       → transport
use case         → orchestration
entity           → domain behavior
repository port  → capability
adapter          → persistence
```

Folder structure is secondary to responsibility.

---

# 84. GRASP and Browser Applications

The same ideas can guide front-end design:

```text
UI component
    → presentation

application state/use case
    → workflow

domain object
    → domain behavior

API client
    → integration
```

Avoid making UI components the universal business-rule owner.

---

# 85. GRASP and Security

Responsibility assignment can improve security.

```text
narrow capability
    ↓
less authority
    ↓
smaller blast radius
```

Example:

```ts
constructor(
  private readonly orders: OrderRepository
) {}
```

is narrower than:

```ts
constructor(
  private readonly db: Database
) {}
```

---

# 86. GRASP and Least Authority

Treat object references as capabilities.

If you inject:

```ts
Database
```

you grant broad power.

If you inject:

```ts
OrderRepository
```

you grant a narrower power.

Low Coupling and least authority can reinforce each other.

---

# 87. GRASP and Authorization

Example:

```text
Only managers can approve >20% discount.
```

Possible responsibilities:

```text
AuthorizationPolicy
    actor permission

DiscountPolicy
    discount rule

Sale
    state transition
```

One requirement can cross several responsibility boundaries.

---

# 88. GRASP and Audit

Audit is frequently Pure Fabrication.

```ts
interface AuditLogger {
  record(entry: AuditEntry): Promise<void>;
}
```

The domain can produce meaningful events without knowing the audit storage mechanism.

---

# 89. GRASP and Observability

Telemetry responsibilities can be isolated as:

```text
Metrics
Tracer
AuditLogger
```

Avoid making domain entities directly dependent on a specific monitoring provider.

---

# 90. GRASP and Reliability

External failures belong to the appropriate boundary.

```text
PaymentGateway
    external failure

Use Case
    workflow response

Order
    domain state transition
```

Do not make the order object understand every network failure mechanism.

---

# 91. GRASP and Retries

Retries are usually application/infrastructure responsibilities.

A retry wrapper may sit around:

```text
PaymentGateway
```

without forcing the domain to know about:

```text
backoff
timeouts
circuit state
```

---

# 92. GRASP and Circuit Breakers

A circuit breaker is Pure Fabrication around an unstable external dependency.

```text
PaymentGateway
    ↓
CircuitBreaker
    ↓
ProviderAdapter
```

The domain remains unaware of provider health mechanics.

---

# 93. GRASP and Rate Limiting

Rate limiting is usually technical/application responsibility.

Keep:

```text
RateLimiter
```

separate from entities unless the quota itself is a business concept.

---

# 94. GRASP and Transactions

Transaction scope often belongs at the application boundary.

Example:

```text
TransferMoneyUseCase
    transaction
    ├── load source
    ├── load target
    ├── withdraw
    ├── deposit
    └── save
```

The accounts still own withdrawal and deposit rules.

---

# 95. GRASP and Concurrency

Domain ownership and technical atomicity are related but different.

```text
Inventory
    owns reservation semantics

database/transaction
    enforces atomicity
```

A database lock is not the domain responsibility.

---

# 96. GRASP and Idempotency

Idempotency can be a use-case boundary concern.

```text
IdempotencyStore
    remembers command keys

Use Case
    performs workflow

Domain
    enforces business state
```

Do not leak transport-level request keys into unrelated domain concepts without a domain reason.

---

# 97. GRASP and Caching

A cache is generally an optimization responsibility.

```text
Cache
    speed

Domain
    correctness
```

If a cache becomes authoritative, that is an explicit consistency decision rather than a GRASP default.

---

# 98. GRASP and Read Models

A read model can intentionally duplicate information.

```text
Order
    authoritative domain state

OrderReadModel
    optimized query representation
```

The read model does not automatically own the business state.

---

# 99. GRASP and Events

A domain object can own a transition:

```ts
order.confirm();
```

A separate publisher can own event delivery.

This keeps:

```text
state change
```

distinct from:

```text
transport
```

---

# 100. GRASP and Distributed Systems

In distributed systems:

```text
Order Service
    owns order state

Payment Service
    owns payment state
```

Messages coordinate them.

Do not create two authoritative owners for one state without an intentional consistency model.

---

# 101. GRASP and Bounded Contexts

A bounded context can protect its own model from an external model.

```text
Ordering
    ↓
PaymentGateway
    ↓
Payment context
```

An anti-corruption layer may implement Indirection and Protected Variations.

---

# 102. GRASP and Legacy Systems

For legacy integration:

```text
Domain
    ↓
LegacyGateway
    ↓
Legacy API
```

Do not spread legacy schemas throughout the domain.

The gateway translates and contains the external model.

---

# 103. GRASP and Change Amplification

If one pricing rule requires edits in:

```text
OrderController
OrderService
InvoiceService
ReportService
```

responsibility is probably fragmented.

A coherent pricing boundary can reduce change amplification.

---

# 104. GRASP and Shotgun Surgery

Shotgun surgery means one conceptual change requires many edits.

GRASP response:

```text
find the responsibility
find the owner
consolidate the rule
protect variation
```

This is often a responsibility-placement problem.

---

# 105. GRASP and Divergent Change

If one class changes for:

```text
payment
tax
email
reporting
state transitions
```

High Cohesion is likely being violated.

---

# 106. GRASP and Feature Envy

If a method repeatedly reaches into another object's internals:

```ts
function calculate(order: Order) {
  return order.customer.address.country.code;
}
```

ask:

```text
Who should know this?
Should the object expose a semantic operation?
Should a policy own the rule?
```

---

# 107. GRASP and Data Clumps

Repeated parameters may reveal a missing concept:

```text
country
currency
taxRate
```

Potentially:

```text
TaxContext
```

The object should exist because it represents meaningful responsibility, not merely because parameters repeat.

---

# 108. GRASP and Primitive Obsession

Repeated primitive rules can indicate a value object.

Example:

```text
Money
Quantity
Percentage
EmailAddress
```

A value object can become the Information Expert for its own invariants.

---

# 109. GRASP and God Objects

A god object often has:

```text
too much state
too many collaborators
too many reasons to change
too many decisions
```

First identify responsibility clusters.

Do not split it mechanically by method count.

---

# 110. GRASP and Service Blobs

This:

```text
OrderService
PaymentService
TaxService
CustomerService
```

is not automatically good or bad.

Inspect what each service actually owns.

A service can be cohesive.

A service can also be a dumping ground.

Naming does not determine responsibility.

---

# 111. GRASP and Controller Blobs

A controller with:

```text
validation
pricing
inventory
payment
persistence
email
```

usually combines transport, coordination, domain, and infrastructure responsibilities.

Refactor by responsibility, not by framework preference.

---

# 112. GRASP and Anemic Models

An anemic model stores data while services contain most domain decisions.

This can be acceptable for simple CRUD.

It becomes dangerous when:

```text
state transitions are complex
invariants matter
rules are duplicated
many workflows manipulate the same state
```

Then Information Expert and encapsulation become more valuable.

---

# 113. GRASP and Simple CRUD

Do not over-engineer:

```text
single form
simple record
simple persistence
few invariants
```

A simple:

```text
Controller → Service → Repository
```

may be perfectly appropriate.

GRASP includes recognizing when extra abstraction is not useful.

---

# 114. GRASP and Enterprise Complexity

GRASP pays off more as the system has:

```text
multiple teams
long-lived code
complex invariants
many integrations
regulatory requirements
multi-tenancy
provider changes
high audit needs
```

The cost of unclear responsibility becomes larger.

---

# 115. Responsibility Matrix

Use:

| Responsibility | Candidate | GRASP Reason |
|---|---|---|
| line subtotal | OrderLine | Information Expert |
| create line | Order | Creator |
| receive HTTP request | Controller | Controller |
| coordinate checkout | Use Case | Controller/Application coordination |
| tax variation | TaxPolicy | Polymorphism |
| persistence | Repository | Pure Fabrication |
| provider integration | Adapter | Indirection |
| provider replacement | Gateway | Protected Variations |

---

# 116. CRC Cards

CRC means:

```text
Class
Responsibilities
Collaborators
```

Example:

```text
Class:
Order

Responsibilities:
- own lines
- calculate total
- confirm
- cancel

Collaborators:
- OrderLine
- PricingPolicy
```

GRASP adds a vocabulary for discussing why each responsibility sits there.

---

# 117. CRC + GRASP Worksheet

```text
Class:
________________

Responsibilities:
________________

Collaborators:
________________

Information Expert?
________________

Creator?
________________

Controller?
________________

Polymorphic variation?
________________

Pure Fabrication?
________________

Indirection?
________________

Protected Variation?
________________
```

---

# 118. Interaction Diagrams

Scenario:

```text
Client
  |
  | checkout()
  v
CheckoutUseCase
  |
  | loadCart()
  v
CartRepository
  |
  | return Cart
  v
CheckoutUseCase
  |
  | checkout()
  v
Cart
  |
  | reserve()
  v
Inventory
  |
  | charge()
  v
PaymentGateway
  |
  | save()
  v
OrderRepository
```

Now explain each message through GRASP.

---

# 119. Scenario-Driven GRASP

Start with a real scenario:

```text
Customer checks out.
```

Extract verbs:

```text
load
checkout
reserve
charge
save
notify
```

Then assign each responsibility.

This is usually more effective than inventing classes from nouns first.

---

# 120. Responsibility Decomposition Algorithm

```text
1. Identify system operation.
2. List business responsibilities.
3. Identify state.
4. Identify invariants.
5. Assign state ownership.
6. Apply Information Expert.
7. Apply Creator.
8. identify Controller.
9. Check High Cohesion.
10. Check Low Coupling.
11. Identify behavior variation.
12. Consider Polymorphism.
13. Add Pure Fabrication when needed.
14. Add Indirection where useful.
15. Add Protected Variations around credible change.
16. Trace a scenario.
17. Review trade-offs.
```

---

# 121. GRASP Review Questions

Ask:

```text
Who knows?
Who owns?
Who decides?
Who creates?
Who coordinates?
What varies?
What changes?
What can fail?
What is externally controlled?
What should be isolated?
```

---

# 122. Responsibility Scorecard

Score a candidate owner against:

| Dimension | Question |
|---|---|
| Knowledge | Does it know enough? |
| Authority | Does it own the state? |
| Invariant | Can it protect the rule? |
| Cohesion | Does the behavior fit? |
| Coupling | What dependency is created? |
| Variation | Is change isolated? |
| Testability | Is the boundary testable? |
| Security | Is unnecessary authority avoided? |
| Evolution | Will future changes stay local? |

---

# 123. GRASP Anti-Pattern — Cargo Cult

Weak:

```text
"Use a factory because GRASP."
"Use an interface because GRASP."
"Use a service because controllers are thin."
```

Better:

```text
identify problem
    ↓
identify responsibility
    ↓
compare candidates
    ↓
choose least harmful design
```

---

# 124. GRASP Anti-Pattern — Over-Abstraction

Signs:

```text
Interface
Adapter
Factory
Provider
Strategy
Facade
Proxy
```

for one tiny stable behavior.

The abstraction budget matters.

---

# 125. GRASP Anti-Pattern — Under-Abstraction

Signs:

```text
Controller → Stripe
Controller → SQL
Controller → domain rules
```

Direct coupling may be cheap initially and expensive later.

Protect actual change.

---

# 126. GRASP Anti-Pattern — Wrong Expert

A class having a field does not make it the expert for every rule touching that field.

```text
Order has customerId
```

does not make Order an authentication expert.

Knowledge must be relevant.

---

# 127. GRASP Anti-Pattern — Fake Cohesion

Everything having the word "Customer" does not make a class cohesive.

```text
CustomerService
    createOrder
    exportReport
    sendMarketingEmail
    resetPassword
```

These can change for unrelated reasons.

---

# 128. GRASP Anti-Pattern — Distributed Invariants

If:

```text
status == PENDING
```

is checked in many places, there may be no invariant owner.

A state-owning object should usually expose semantic operations.

---

# 129. GRASP — Semantic APIs

Prefer:

```ts
invoice.approve();
invoice.pay();
invoice.cancel();
```

over:

```ts
invoice.setStatus("APPROVED");
```

Semantic commands communicate responsibility and protect transitions.

---

# 130. GRASP — Public Surface

A cohesive object often has a small semantic API:

```ts
class Order {
  addLine(line: OrderLine) {}
  removeLine(id: string) {}
  total(): Money {}
  confirm() {}
  cancel() {}
}
```

The API expresses responsibility.

---

# 131. GRASP — Representation Independence

Suppose clients use:

```ts
order.addLine(...)
order.total()
```

The internal collection can change from:

```ts
OrderLine[]
```

to:

```ts
Map<string, OrderLine>
```

without breaking the semantic contract.

Responsibility-driven APIs support evolution.

---

# 132. GRASP — Method Naming

Strong:

```text
reserveStock
approve
finalize
reconcile
calculateTax
```

Weak:

```text
process
handle
manage
doWork
```

unless the generic name is meaningful at the abstraction level, such as a standard `execute()` use-case method.

---

# 133. GRASP — Error Ownership

Ask:

> Which responsibility decides that this error exists?

Example:

```text
Order
    invalid state transition

PaymentGateway
    provider rejected payment

Repository
    persistence failure

Use Case
    workflow response
```

Error interpretation should match responsibility.

---

# 134. GRASP — Validation

Layer validation by responsibility.

```text
transport shape
    boundary

domain invariant
    domain object

storage constraint
    database/repository
```

Do not assume one layer can validate every concern.

---

# 135. GRASP — Logging

Avoid:

```ts
class Order {
  confirm() {
    console.log("confirmed");
  }
}
```

unless output is the actual domain requirement.

Keep operational telemetry outside the domain when appropriate.

---

# 136. GRASP — Time

Instead of:

```ts
class Subscription {
  isExpired() {
    return new Date() > this.expiresAt;
  }
}
```

prefer:

```ts
class Subscription {
  isExpired(now: Date) {
    return now >= this.expiresAt;
  }
}
```

or use a `Clock` capability when current time is a replaceable dependency.

---

# 137. GRASP — Randomness

Identity generation can be isolated:

```ts
interface IdGenerator {
  generate(): string;
}
```

A factory can use it without coupling domain construction to `Math.random()` or another implementation.

---

# 138. GRASP — Serialization

Keep external representations separate where complexity warrants it.

```text
Order
    domain behavior

OrderDtoMapper
    API representation

InvoiceRenderer
    PDF representation

CsvExporter
    CSV representation
```

One object should not automatically own every representation.

---

# 139. GRASP — Persistence

Typical separation:

```text
Domain
    behavior

Repository
    load/store

ORM Adapter
    database mapping

Database
    physical persistence
```

This is a common combination of Pure Fabrication, Indirection, and Low Coupling.

---

# 140. GRASP — Active Record Trade-Off

Active Record intentionally combines domain and persistence.

This can be good for:

```text
small CRUD
rapid development
simple invariants
```

A richer boundary may be justified as domain complexity grows.

---

# 141. Jewellery ERP — Responsibility Map

Consider:

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

Potential assignment:

```text
Tenant
    tenant identity and invariants

Branch
    branch state and rules

Sale
    lifecycle and sale-level invariants

SaleLine
    line quantity/value

PricingPolicy
    pricing variation

DiscountPolicy
    discount decision

TaxPolicy
    tax decision

Inventory
    stock reservation

PaymentGateway
    payment integration

SaleRepository
    persistence

FinalizeSaleUseCase
    workflow coordination

AuditLogger
    audit responsibility
```

---

# 142. Jewellery ERP — Scenario

```text
Finalize sale in Branch A
for Tenant T1
with jewellery item J1
using current pricing
apply discount
calculate tax
reserve stock
accept payment
finalize invoice
record audit
```

The important design question is not:

```text
"Which class contains finalizeSale?"
```

It is:

```text
Who owns each responsibility?
```

---

# 143. Jewellery ERP — Information Expert

If `SaleLine` owns:

```text
quantity
weight
unit price
making charge
```

it is a strong expert for line-level value.

`Sale` is then a candidate for aggregation across lines.

---

# 144. Jewellery ERP — Polymorphism

```ts
interface PricingPolicy {
  price(input: PricingInput): Money;
}
```

Potential variants:

```text
WeightBasedPricing
PieceBasedPricing
MarketLinkedPricing
FixedPricePricing
```

Use this only when pricing variation genuinely deserves an abstraction.

---

# 145. Jewellery ERP — Protected Variations

Likely variation points:

```text
metal pricing source
tax rules
discount rules
payment provider
inventory backend
number generator
notification provider
```

Protect meaningful volatility.

Do not abstract every dependency.

---

# 146. Jewellery ERP — Controller

```text
POST /sales/finalize
        ↓
FinalizeSaleController
        ↓
FinalizeSaleUseCase
```

The controller maps the external request.

The use case coordinates.

The sale enforces domain rules.

---

# 147. Jewellery ERP — Use Case

A use case may coordinate:

```text
load sale
verify access
finalize sale
reserve stock
charge payment
persist
audit
```

The exact transaction and failure model depends on the system's consistency requirements.

---

# 148. Jewellery ERP — Security

Tenant and branch boundaries may require defense in depth:

```text
request context
    ↓
authorization
    ↓
use-case scoping
    ↓
repository tenant filtering
    ↓
database constraints
```

No single GRASP pattern solves multi-tenancy.

---

# 149. Jewellery ERP — Audit

Audit is a separate responsibility:

```text
Sale
    state transition

AuditLogger
    record audit

Infrastructure
    persist audit
```

This prevents audit storage details from polluting the domain object.

---

# 150. Jewellery ERP — Payment

Payment integration:

```text
Sale/Use Case
    ↓
PaymentGateway
    ↓
ProviderAdapter
```

The provider should not dictate the domain model.

---

# 151. Jewellery ERP — Inventory

Inventory should own stock reservation semantics:

```ts
inventory.reserve(lines);
```

The database may implement atomicity.

The domain responsibility is still:

```text
stock cannot be over-reserved
```

---

# 152. Jewellery ERP — Transaction

A use-case boundary may coordinate:

```text
sale
inventory
payment
invoice
```

but external payment systems may not share the same local transaction.

That requires an explicit consistency model.

GRASP identifies responsibilities; it does not solve distributed transactions by itself.

---

# 153. Jewellery ERP — Reconciliation

If payment succeeds but local persistence fails:

```text
Payment service
    success

Order/Sale persistence
    failure
```

a reconciliation responsibility may be needed:

```text
PaymentReconciliationJob
```

This belongs outside the sale entity.

---

# 154. Jewellery ERP — Read Model

For dashboards:

```text
Sale
    authoritative state

SalesDashboardReadModel
    query optimization
```

Do not make dashboard projections authoritative unless explicitly designed that way.

---

# 155. Implementation Exercise — Payment

Implement:

```ts
interface PaymentGateway {
  charge(input: ChargeInput): Promise<ChargeResult>;
}

class CheckoutUseCase {
  constructor(
    private readonly gateway: PaymentGateway
  ) {}

  async execute(command: CheckoutCommand) {
    // coordinate
  }
}
```

Prove:

```text
provider independence
testability
narrow capability
provider-result mapping
```

---

# 156. Implementation Exercise — Pricing

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

Then explain why the abstraction is justified or not.

---

# 157. Implementation Exercise — Repository

Implement:

```ts
interface OrderRepository {
  findById(id: string): Promise<Order | null>;
  save(order: Order): Promise<void>;
}
```

Then provide:

```text
InMemoryOrderRepository
PostgresOrderRepository
```

Demonstrate that domain code does not know the persistence implementation.

---

# 158. Implementation Exercise — Creator

Implement:

```text
OrderFactory
```

with:

```text
Clock
IdGenerator
```

Then test deterministic construction.

Identify:

```text
Factory → creation
Clock → time
IdGenerator → identity
Order → domain state
```

---

# 159. Implementation Exercise — Pure Fabrication

Create:

```ts
interface PasswordHasher {
  hash(password: string): Promise<string>;
  verify(password: string, hash: string): Promise<boolean>;
}
```

Explain why hashing should usually not be embedded in the entity.

---

# 160. Design Exercise — CRC

Create CRC cards for:

```text
Customer
Order
OrderLine
Inventory
Payment
```

Then label the GRASP patterns supporting each responsibility.

---

# 161. Design Exercise — Full Scenario

Scenario:

```text
Customer places order.
Inventory is reserved.
Payment succeeds.
Order is confirmed.
Audit is recorded.
```

Produce:

```text
responsibility matrix
CRC cards
interaction diagram
invariants
variation points
GRASP explanation
```

---

# 162. Refactoring Exercise — Service Blob

Given:

```ts
class OrderService {
  confirm(order) {}
  calculateTotal(order) {}
  sendEmail(order) {}
  save(order) {}
  charge(order) {}
}
```

Possible mapping:

```text
confirm
    → Order

calculateTotal
    → Order / pricing boundary

sendEmail
    → EmailSender

save
    → OrderRepository

charge
    → PaymentGateway

workflow
    → Use Case
```

The point is reasoned assignment, not mechanical extraction.

---

# 163. Refactoring Exercise — Controller Blob

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

After:

```text
Controller
    mapping

FinalizeOrderUseCase
    coordination

Order
    domain behavior

Inventory
    reservation

PaymentGateway
    payment integration

Notification
    delivery
```

---

# 164. Refactoring Exercise — God Object

Before:

```text
ERPManager
    users
    orders
    inventory
    payment
    reports
    audit
    email
```

First create responsibility clusters.

Then extract only meaningful boundaries.

---

# 165. Refactoring Exercise — Anemic Model

Before:

```ts
class Order {
  status!: string;
  items!: OrderLine[];
}
```

and:

```ts
OrderService.confirm(order)
OrderService.cancel(order)
OrderService.total(order)
```

Move invariant-heavy behavior toward the natural owner.

Keep real cross-object policy responsibilities external when appropriate.

---

# 166. Refactoring Exercise — Feature Envy

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

Review:

```text
Who owns category?
Who owns line subtotal?
Is the discount rule stable?
Should a policy own variation?
```

Do not move code blindly.

---

# 167. Refactoring Exercise — Long Conditional

Given:

```ts
switch (shippingType) {
  case "STANDARD":
  case "EXPRESS":
  case "INTERNATIONAL":
}
```

Evaluate:

```text
frequency of change
number of variants
independent behavior
testing needs
configuration needs
```

Then decide whether Polymorphism earns its cost.

---

# 168. Refactoring Exercise — Missing Indirection

Five modules depend on a provider SDK.

Introduce:

```text
ProviderGateway
    ↓
ProviderAdapter
```

if provider replacement and isolation are credible requirements.

---

# 169. Testing and GRASP

Tests reveal responsibility boundaries.

```text
Entity tests
    domain behavior

Value-object tests
    value semantics

Use-case tests
    collaboration

Repository tests
    persistence

Adapter tests
    integration mapping
```

Natural test boundaries often signal good responsibility boundaries.

---

# 170. GRASP and Mocks

Do not create an interface only because a test needs a mock.

Prefer:

```text
meaningful responsibility
    ↓
stable boundary
    ↓
natural test seam
```

The abstraction should make sense outside the test.

---

# 171. GRASP and Constructor Size

A large constructor can be a signal, not a rule violation.

Ask:

```text
Do these collaborators support one coherent responsibility?
```

A checkout use case may legitimately have several capabilities.

A generic manager with twenty unrelated dependencies is suspicious.

---

# 172. GRASP and Abstraction Budget

Every abstraction costs:

```text
names
files
configuration
navigation
documentation
cognitive load
runtime indirection
```

Benefits can include:

```text
change isolation
replaceability
testability
security
clarity
```

Add an abstraction when the benefit is real.

---

# 173. GRASP and Stable vs Volatile

Ask:

```text
What is stable?
What is volatile?
```

Protect volatile details behind stable concepts.

Do not wrap stable concepts in unnecessary abstraction.

---

# 174. GRASP and Dependency Direction

Preferred:

```text
Domain
   ↓
stable capability/port

Infrastructure
   ↓
implementation
```

Avoid forcing domain code to depend on volatile frameworks when the boundary does not require it.

---

# 175. GRASP and API Versioning

A gateway or adapter can isolate provider versions.

```text
Internal capability
    ↓
v1 adapter
v2 adapter
```

The domain remains stable while external schemas evolve.

---

# 176. GRASP and Schema Evolution

Repositories and mappers can protect the domain from database representation changes.

```text
Domain object
    ↓
Repository
    ↓
Database mapping
```

The domain contract need not mirror the storage schema.

---

# 177. GRASP and Serialization Boundaries

External data may require mapping:

```text
HTTP DTO → command
DB row → entity
entity → response DTO
provider response → domain result
```

Mapping is a responsibility.

Keep it explicit when representation differences matter.

---

# 178. GRASP and Workflow Engines

A workflow engine can own:

```text
sequence
state machine execution
recovery
workflow persistence
```

It does not automatically own domain invariants.

Separate orchestration from business truth.

---

# 179. GRASP and Event Choreography

In event-driven systems:

```text
SaleFinalized
    ↓
InvoiceService
    ↓
NotificationService
```

Each component owns its own responsibilities.

The trade-off is more asynchronous coordination and consistency complexity.

---

# 180. GRASP and Idempotent Event Handlers

A consumer may use:

```text
processed-event store
```

to avoid duplicate processing.

That is a technical/application responsibility.

The domain still owns the semantic state transition.

---

# 181. GRASP and Compliance

Compliance may create explicit responsibilities:

```text
AuditLogger
ApprovalPolicy
RetentionPolicy
SegregationOfDutiesPolicy
TenantIsolation
```

Do not hide compliance logic in generic helpers.

---

# 182. GRASP and Segregation of Duties

Example:

```text
Creator cannot approve own high-value sale.
```

Possible split:

```text
AuthorizationPolicy
    permission

Sale
    valid state transition

Application Use Case
    actor/context coordination
```

This shows why one requirement can span multiple responsibilities.

---

# 183. GRASP and Performance

Patterns can have runtime effects.

```text
extra indirection
polymorphic dispatch
object allocation
collaboration calls
```

Measure before optimizing.

Responsibility correctness and implementation optimization are separate decisions.

---

# 184. GRASP and Memory

For high-volume objects:

```ts
class Item {
  method() {}
}
```

uses a prototype method.

An instance field:

```ts
method = () => {}
```

can allocate a function per instance.

Do not choose representation solely from GRASP; evaluate runtime and memory needs separately.

---

# 185. GRASP and Reliability

A gateway boundary can isolate provider failure.

A use case can interpret:

```text
timeout
rejection
retryable error
permanent failure
```

The domain model can remain focused on business state.

---

# 186. GRASP and Failure Compensation

When an operation crosses boundaries:

```text
stock reservation
payment
order persistence
```

a failure may require:

```text
retry
compensation
reconciliation
manual review
```

Those are additional responsibilities, not automatic responsibilities of the domain entity.

---

# 187. GRASP and Read/Write Separation

A rich write model can own business behavior:

```text
Order
```

while a read model optimizes queries:

```text
OrderDashboardView
```

The two can intentionally have different responsibility structures.

---

# 188. GRASP and Operational Ownership

Ask:

```text
Who owns retries?
Who owns reconciliation?
Who owns audit?
Who owns cache invalidation?
Who owns migration?
```

These are responsibilities even when they are not domain concepts.

---

# 189. GRASP and Team Boundaries

A cohesive responsibility boundary can reduce cross-team changes.

```text
Ordering
Payments
Inventory
Reporting
```

Low coupling reduces shared change surfaces.

This is architecture-level GRASP reasoning.

---

# 190. GRASP Interview Framework

For an LLD problem:

```text
1. State the system operation.
2. Identify concepts.
3. Identify state and invariants.
4. Apply Information Expert.
5. Apply Creator.
6. Identify Controller.
7. Check High Cohesion.
8. Check Low Coupling.
9. Identify variation.
10. Apply Polymorphism where justified.
11. Use Pure Fabrication when no natural owner exists.
12. Add Indirection where direct coupling is expensive.
13. Protect credible variation.
14. Trace a scenario.
15. Explain trade-offs.
```

---

# 191. Interview — What Is GRASP?

Strong answer:

> "GRASP is a family of general responsibility-assignment heuristics for object-oriented design. It helps decide which object should know or do something, how system operations should enter the model, and how cohesion, coupling, variation and collaboration should be managed."

---

# 192. Interview — Information Expert

Strong answer:

> "Start with the object that has the information needed for the responsibility. If it also owns the state and invariant, that is stronger evidence. I still check cohesion and variation so I do not overload that object."

---

# 193. Interview — Creator vs Factory

Strong answer:

> "Creator is the responsibility-assignment heuristic. A factory is a concrete mechanism I may introduce when creation is complex, polymorphic, dependency-heavy, or itself a meaningful boundary."

---

# 194. Interview — Controller vs Application Service

Strong answer:

> "The controller receives a system operation at a boundary. The application service often coordinates the use case. Neither automatically becomes the owner of all business rules."

---

# 195. Interview — Low Coupling

Strong answer:

> "Low Coupling reduces unnecessary dependency. It does not mean no collaboration. I isolate volatile or broad dependencies when the reduction in change impact justifies the added abstraction."

---

# 196. Interview — High Cohesion

Strong answer:

> "High Cohesion means an object's responsibilities have a strong conceptual relationship. I also avoid over-fragmenting trivial behavior just to increase class count."

---

# 197. Interview — Pure Fabrication

Strong answer:

> "When no domain object naturally owns a responsibility, I can create a focused technical or application object such as a repository, audit logger, email sender, or adapter. The fabricated class must still be cohesive."

---

# 198. Interview — Indirection

Strong answer:

> "Indirection inserts an intermediary to reduce direct coupling or translate models. A gateway or adapter is a common example. The layer has a cost, so it should solve a real boundary problem."

---

# 199. Interview — Protected Variations

Strong answer:

> "Identify a credible point of change and depend on a stable concept around it. Providers, clocks, tax policies and pricing strategies are examples. I avoid abstracting hypothetical variation without evidence."

---

# 200. Interview — Polymorphism

Strong answer:

> "When behavior varies by type and that variation is meaningful, a stable responsibility contract can let each variant own its behavior. A small stable switch can still be simpler."

---

# 201. Interview — Anemic Models

Strong answer:

> "An anemic model is not automatically wrong. It can fit simple CRUD. It becomes problematic when invariants and state transitions repeatedly live in services, causing duplication and weak ownership."

---

# 202. Predict-the-Design

### Scenario 1

```text
Calculate OrderLine subtotal.
```

Likely:

```text
OrderLine
```

Reason:

```text
Information Expert
```

### Scenario 2

```text
Create Order with clock and ID generator.
```

Likely:

```text
OrderFactory
```

Reason:

```text
Creator + variation isolation
```

### Scenario 3

```text
Receive HTTP finalize-order request.
```

Likely:

```text
Controller
```

### Scenario 4

```text
Switch payment provider.
```

Likely:

```text
PaymentGateway + adapter
```

Reason:

```text
Indirection + Protected Variations
```

### Scenario 5

```text
No natural domain owner for password hashing.
```

Likely:

```text
PasswordHasher
```

Reason:

```text
Pure Fabrication
```

---

# 203. Predict-the-Design — Advanced

### Scenario

```text
Calculate final sale price using:
base price
discount policy
tax policy
shipping policy
customer segment
```

Do not immediately place all logic in `Sale`.

Reason through:

```text
Information Expert
High Cohesion
Polymorphism
Protected Variations
Domain Service
```

A good answer explains which concepts own stable behavior and which policies vary.

---

# 204. Code Review Exercise

Review:

```ts
class OrderController {
  async confirm(req) {
    const order = await db.orders.find(req.params.id);

    if (order.status === "PENDING") {
      order.status = "CONFIRMED";
    }

    const total = order.lines.reduce(
      (sum, line) => sum + line.price * line.quantity,
      0
    );

    await stripe.charge(total);
    await db.orders.save(order);

    return order;
  }
}
```

Identify:

```text
Controller responsibility
Persistence responsibility
Domain responsibility
Calculation responsibility
Payment responsibility
Workflow responsibility
```

Then redesign the collaboration.

---

# 205. Debugging GRASP

Symptoms:

```text
duplicated business rule
    → missing responsibility owner

giant service
    → low cohesion

long type conditional
    → possible polymorphism

provider leaks into domain
    → missing indirection

controller contains business rules
    → controller overload

many direct state mutations
    → weak ownership
```

Use these as signals, not automatic verdicts.

---

# 206. Debugging Procedure

```text
1. Reproduce the bug.
2. Identify the wrong decision.
3. Identify current owner.
4. Identify natural owner.
5. Check knowledge.
6. Check authority.
7. Check invariant.
8. Check cohesion.
9. Check coupling.
10. Check variation.
11. Refactor minimally.
12. Add regression test.
```

---

# 207. Mastery Exercise — Library

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
responsibility matrix
CRC cards
interaction diagram
invariants
GRASP justification
TypeScript implementation
tests
```

---

# 208. Mastery Exercise — Jewellery ERP

Design the responsibility model for:

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

Explicitly justify all nine GRASP patterns.

---

# 209. Mastery Exercise — No Reference

Given only:

```text
A customer checks out a cart.
```

Create from scratch:

```text
system operation
responsibilities
state
invariants
CRC cards
GRASP assignments
interaction diagram
TypeScript interfaces
implementation
tests
```

Do not start from a pre-existing service hierarchy.

---

# 210. Mastery Gate

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

You have not mastered GRASP because you can recite nine names.

You have mastered it when you can justify responsibility placement in an unfamiliar system.

---

# 211. Principal-Level GRASP Review

For each responsibility ask:

```text
1. Does this object know enough?
2. Does it own the relevant state?
3. Can it protect the invariant?
4. Does the behavior fit its concept?
5. What coupling is introduced?
6. What variation is exposed?
7. Is polymorphism actually useful?
8. Would fabrication improve cohesion?
9. Would indirection reduce real coupling?
10. Is the abstraction worth its cost?
11. What is the runtime cost?
12. What authority is granted?
13. What failure modes cross the boundary?
14. How will the design change?
```

---

# 212. Principal Judgment — Simplicity

Sometimes the best GRASP-aligned design is simply:

```ts
function calculateSubtotal(price, quantity) {
  return price * quantity;
}
```

Do not create architecture for its own sake.

Patterns are tools for solving problems.

---

# 213. Principal Judgment — Abstraction Budget

An abstraction should earn its cost through:

```text
replaceability
change isolation
testability
security
clarity
translation
```

Otherwise direct code may be better.

---

# 214. Principal Judgment — Stable vs Volatile

A useful rule:

```text
stable concept
    ↓
protect
volatile detail
    ↓
isolate
```

Do not invert the dependency.

---

# 215. Principal Judgment — Domain vs Technical

Classify responsibility:

```text
Domain
    business meaning

Application
    workflow

Infrastructure
    technical integration

Transport
    external protocol
```

Then place responsibility at the appropriate level.

---

# 216. Principal Judgment — Ownership

The deepest GRASP model is:

```text
state
  ↓
knowledge
  ↓
decision
  ↓
invariant
  ↓
behavior
  ↓
collaboration
  ↓
boundary
```

Good design makes these relationships visible.

---

# 217. Completion Checklist

```text
[ ] I can define GRASP.
[ ] I can explain Information Expert.
[ ] I can explain Creator.
[ ] I can explain Controller.
[ ] I can explain Low Coupling.
[ ] I can explain High Cohesion.
[ ] I can explain Polymorphism.
[ ] I can explain Pure Fabrication.
[ ] I can explain Indirection.
[ ] I can explain Protected Variations.
[ ] I can combine patterns rather than apply them mechanically.
[ ] I can build CRC cards.
[ ] I can build responsibility matrices.
[ ] I can model interaction sequences.
[ ] I can refactor service blobs.
[ ] I can refactor god objects.
[ ] I can distinguish domain behavior from coordination.
[ ] I can identify variation points.
[ ] I can reason about security and least authority.
[ ] I can apply GRASP in TypeScript.
[ ] I can apply GRASP in JavaScript without classes.
[ ] I can justify an architecture in an interview.
[ ] I can apply all nine patterns to the jewellery ERP scenario.
```

---

# 218. Revision / Retrieval Record

Use this after real retrieval sessions.

```text
Date:
________________

Patterns recalled:
________________

Weakest pattern:
________________

Pattern I over-applied:
________________

Pattern I under-used:
________________

Scenario:
________________

Initial assignment:
________________

Revision:
________________

Why:
________________
```

---

# 219. Canonical References and Source Discipline

### GRASP / Object-Oriented Design

The primary conceptual reference for formal GRASP terminology in this curriculum is Craig Larman's:

*Applying UML and Patterns: An Introduction to Object-Oriented Analysis and Design and Iterative Development.*

Use an authoritative edition when studying the formal vocabulary and examples.

### ECMAScript

For JavaScript language semantics:

https://tc39.es/ecma262/

Use ECMAScript for:

```text
language semantics
objects
classes
functions
private elements
modules
property behavior
```

### TypeScript

Official documentation:

https://www.typescriptlang.org/docs/

Use it for:

```text
type system
interfaces
structural typing
compiler behavior
```

### Runtime Distinction

Treat:

```text
V8 hidden classes
inline caches
deoptimization
```

as implementation details unless a behavior is standardized.

### Architecture Distinction

Concepts such as:

```text
application service
domain service
repository
aggregate
bounded context
```

are architectural/modeling concepts, not ECMAScript language features.

---

# 220. Source Discipline Rules

When writing future chapters:

```text
Specification
    → language-level claims

Engine documentation
    → engine-specific claims

Runtime documentation
    → host-specific claims

Architecture literature
    → architectural guidance

Project context
    → project-specific decisions
```

Do not blur these layers.

---

# 221. Concept Connections

```text
Chapters 1–6
    JavaScript object/runtime mechanics
            ↓
Chapter 7
    Encapsulation
            ↓
Chapter 8
    Abstraction
            ↓
Chapters 9–10
    Subtyping + Polymorphism
            ↓
Chapter 11
    Composition
            ↓
Chapter 12
    Cohesion + Coupling
            ↓
Chapter 13
    Responsibility-Driven Design
            ↓
Chapter 14
    GRASP
            ↓
Chapter 15
    Cohesion-First Object Decomposition
            ↓
SOLID + Patterns + DDD
            ↓
Transactions + Concurrency + Resilience
            ↓
Enterprise LLD
```

---

# 222. Final Mental Model

When a requirement arrives, do not begin with:

```text
"What classes do I create?"
```

Begin with:

```text
What operation is happening?
        ↓
What must remain true?
        ↓
What state exists?
        ↓
Who owns it?
        ↓
Who has the information?
        ↓
Who should create objects?
        ↓
Who receives the system operation?
        ↓
What responsibilities belong together?
        ↓
What dependencies are unnecessary?
        ↓
What behavior varies?
        ↓
What needs technical fabrication?
        ↓
Where does indirection help?
        ↓
What change needs protection?
        ↓
What does the message flow look like?
```

That is GRASP in practice.

---

# 223. One-Sentence Mastery Test

Complete without notes:

> **I assign a responsibility where the relevant knowledge and authority are closest, unless cohesion, coupling, variation, security, or change pressure makes another boundary more appropriate.**

If you can explain and defend that sentence with real examples, you have the core GRASP mindset.

---

# Chapter 14 — Completion Snapshot

```text
Core GRASP:
[+] Information Expert
[+] Creator
[+] Controller
[+] Low Coupling
[+] High Cohesion
[+] Polymorphism
[+] Pure Fabrication
[+] Indirection
[+] Protected Variations

Design:
[+] Responsibility matrices
[+] CRC cards
[+] Interaction diagrams
[+] Scenario decomposition
[+] Invariant ownership
[+] Change-scenario reasoning
[+] Refactoring

JavaScript / TypeScript:
[+] Classes
[+] Functions
[+] Closures
[+] Private fields
[+] Interfaces
[+] Structural typing
[+] Dependency injection
[+] Runtime vs compile-time boundaries

Production:
[+] Security
[+] Least authority
[+] Transactions
[+] Concurrency
[+] Idempotency
[+] Reliability
[+] Observability
[+] Multi-tenancy
[+] External integrations
[+] Distributed systems

Interview:
[+] GRASP question bank
[+] Predict-the-design
[+] Code review
[+] Jewellery ERP
[+] Principal-level judgment
```

# Chapter 14 — Completion Statement

GRASP is mastered when it becomes a design reasoning language rather than a list of nine definitions. The objective is to look at a scenario and explain, with evidence, who should know, who should decide, who should create, who should coordinate, what should vary, what should be isolated, and what should remain deliberately simple.
