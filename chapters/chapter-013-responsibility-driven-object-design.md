# Chapter 13 — Responsibility-Driven Object Design

> **Part:** C — Responsibility-Driven Design  
> **Prerequisite:** Chapters 1–12  
> **Next:** Chapter 14 — GRASP  
> **Status:** `[+] Completed`

---

## Chapter Position

Responsibility-driven object design answers the most important practical question in low-level design:

> **Which object should know or do what?**

The previous chapters established the mechanics and forces of JavaScript objects:

- object identity and mutability
- prototypes and construction
- classes and initialization
- encapsulation
- abstraction
- inheritance and subtyping
- polymorphism
- composition and delegation
- cohesion and coupling

This chapter converts those mechanics into **design judgment**.

A class diagram is not a good design merely because every noun became a class.  
A design becomes useful when responsibilities are placed where they can be understood, protected, tested, changed, and collaborated on with minimal accidental coupling.

This chapter is the bridge from:

```text
"I know how objects work."
```

to:

```text
"I can decide how responsibilities should be distributed among objects."
```

---

# 1. Learning Objectives

By the end of this chapter, you should be able to:

1. Define responsibility precisely in OOP and LLD.
2. Distinguish state, behavior, and coordination responsibilities.
3. Decide **who should know** a fact and **who should perform** an operation.
4. Apply responsibility-assignment ideas from GRASP before formally studying GRASP in Chapter 14.
5. Use Information Expert, Creator, Controller, Pure Fabrication, Indirection, Polymorphism, and Protected Variations as design heuristics.
6. Apply Low Coupling and High Cohesion while assigning responsibilities.
7. Recognize misplaced responsibilities before implementation becomes expensive.
8. Use CRC cards and responsibility matrices to model designs before coding.
9. Distinguish application coordination from domain behavior.
10. Decide when behavior belongs on an entity, value object, domain service, factory, repository, or application service.
11. Detect anemic domain models, god objects, feature envy, data clumps, and manager/service dumping grounds.
12. Apply Tell, Don't Ask without turning objects into rigid black boxes.
13. Use collaboration boundaries to enforce invariants.
14. Model use cases as responsibility flows rather than controller scripts.
15. Validate responsibility placement through interaction diagrams and tests.
16. Implement responsibility-driven designs in JavaScript and TypeScript.
17. Refactor poorly allocated responsibilities into more coherent designs.
18. Explain responsibility allocation during an LLD interview.
19. Evaluate responsibility placement using principal-level trade-offs.
20. Defend why a responsibility belongs in one object instead of another.

---

# 2. Prerequisites

You should already understand:

- JavaScript objects and property lookup
- object identity and aliasing
- prototype chains
- constructor and class semantics
- private state
- abstraction and stable interfaces
- inheritance and subtyping
- polymorphism and dynamic dispatch
- composition and delegation
- cohesion and coupling

If these concepts are weak, revisit Chapters 1–12 before treating this chapter as a design-judgment chapter.

---

# 3. What Is a Responsibility?

A **responsibility** is an obligation assigned to an object or component.

It usually takes one of two broad forms:

```text
Knowing responsibility
    ↓
What information should this object know?

Doing responsibility
    ↓
What behavior should this object perform?
```

A useful expanded model is:

```text
Responsibility
├── Know
│   ├── know its own state
│   ├── know related domain facts
│   └── know collaborators needed for decisions
│
├── Do
│   ├── perform business behavior
│   ├── enforce invariants
│   └── change owned state safely
│
└── Coordinate
    ├── sequence a use case
    ├── invoke collaborators
    ├── manage boundaries
    └── translate technical concerns
```

Responsibility is therefore broader than "method."

A responsibility can correspond to:

- a method
- a group of methods
- ownership of an invariant
- ownership of state
- a collaboration boundary
- a lifecycle decision
- creation of another object
- orchestration of a use case

### Example

Suppose an order has items.

Weak thinking:

```text
OrderService.calculateTotal(order)
```

Better responsibility thinking:

```text
Order knows how to calculate its own total.
```

Why?

Because the order owns the relevant collection and its pricing invariant.

The important question is not:

> "Where is there already a service class?"

The important question is:

> "Which object has the information and authority required to perform this behavior correctly?"

That question drives this entire chapter.

---

# 4. Why Responsibility Assignment Exists

Without explicit responsibility reasoning, systems tend to drift toward one of three shapes.

## Shape A — Data Bags

```ts
class Customer {
  id!: string;
  name!: string;
  creditLimit!: number;
}
```

And all behavior goes elsewhere:

```ts
class CustomerService {
  validateCredit(customer: Customer) {}
  changeName(customer: Customer) {}
  canPlaceOrder(customer: Customer) {}
}
```

This can create an anemic domain model.

---

## Shape B — God Objects

One class receives every responsibility:

```text
ERPService
├── authenticateUser
├── calculateInvoice
├── reserveStock
├── processPayment
├── sendEmail
├── createCustomer
├── exportReport
├── updateLedger
├── generateNumber
└── auditAction
```

The class becomes a change-amplification hotspot.

---

## Shape C — Random Service Sprawl

Every new behavior becomes:

```text
SomethingService
SomethingManager
SomethingHelper
SomethingProcessor
SomethingUtil
SomethingHandler
```

The design may look modular while responsibilities remain unclear.

The goal is not:

> "More classes."

The goal is:

> **A deliberate distribution of responsibilities.**

---

# 5. The Three Major Responsibility Categories

A practical design starts by separating responsibilities into three categories.

## 5.1 State Responsibility

An object may be responsible for maintaining a piece of state.

Example:

```ts
class Account {
  #balance = 0;

  get balance() {
    return this.#balance;
  }
}
```

The account owns the balance.

The important implication is:

```text
Owner of state
    ↓
owner of the rules that protect that state
```

If `balance` cannot be negative, the account should usually protect that invariant.

---

## 5.2 Behavior Responsibility

An object may be responsible for performing a domain operation.

```ts
class Account {
  #balance = 0;

  withdraw(amount: number) {
    if (amount <= 0) {
      throw new Error("Amount must be positive");
    }

    if (amount > this.#balance) {
      throw new Error("Insufficient funds");
    }

    this.#balance -= amount;
  }
}
```

The behavior belongs with the state it protects.

---

## 5.3 Coordination Responsibility

Some responsibilities naturally coordinate several collaborators.

```ts
class CheckoutApplicationService {
  constructor(
    private readonly cartRepository: CartRepository,
    private readonly inventory: Inventory,
    private readonly paymentGateway: PaymentGateway,
    private readonly orderRepository: OrderRepository
  ) {}

  async checkout(cartId: string) {
    const cart = await this.cartRepository.getById(cartId);
    const order = cart.checkout();

    await this.inventory.reserve(order.items);
    await this.paymentGateway.charge(order.total());

    await this.orderRepository.save(order);

    return order;
  }
}
```

This service does not need to become the owner of every domain rule.

Its responsibility is coordination.

That distinction is critical.

---

# 6. The Fundamental Questions

Whenever designing an object, repeatedly ask:

### Question 1 — Who owns this state?

```text
Who can legitimately change it?
```

### Question 2 — Who has the information needed?

```text
Who already knows enough?
```

### Question 3 — Who should enforce the invariant?

```text
Who can prevent invalid state most reliably?
```

### Question 4 — Who should perform this behavior?

```text
Who has the authority and knowledge?
```

### Question 5 — Is this coordination or domain behavior?

```text
Is an object making a domain decision,
or merely sequencing collaborators?
```

### Question 6 — What changes together?

```text
Which methods and data have a strong reason to evolve together?
```

### Question 7 — What would become coupled if I placed it here?

```text
Does this create unnecessary dependencies?
```

These questions are more valuable than memorizing class diagrams.

---

# 7. Information Expert

The **Information Expert** principle says:

> Assign responsibility to the object that has the information necessary to fulfill it.

This is one of the most practical responsibility heuristics.

## Example

Suppose:

```ts
class Order {
  constructor(
    public readonly items: OrderLine[]
  ) {}
}
```

Each line knows:

```ts
class OrderLine {
  constructor(
    public readonly unitPrice: number,
    public readonly quantity: number
  ) {}

  subtotal() {
    return this.unitPrice * this.quantity;
  }
}
```

Then the order knows how to calculate the total:

```ts
class Order {
  constructor(
    public readonly items: OrderLine[]
  ) {}

  total() {
    return this.items.reduce(
      (sum, item) => sum + item.subtotal(),
      0
    );
  }
}
```

Why not:

```ts
class PricingService {
  total(order: Order) {
    return order.items.reduce(
      (sum, item) => sum + item.unitPrice * item.quantity,
      0
    );
  }
}
```

The pricing service may be appropriate for complex pricing policies, but plain summation is already close to the information expert.

---

## 7.1 Information Expert Is Not "Put Everything Where the Data Is"

This principle is a heuristic, not a universal law.

Suppose pricing depends on:

- customer tier
- promotion rules
- tax jurisdiction
- currency
- time windows
- external pricing tables

No single entity has all information.

Forcing all behavior into `Order` can produce a new god object.

Therefore:

```text
Information Expert
+
Cohesion
+
Coupling
+
Protected Variations
+
Domain boundaries
```

must be considered together.

---

# 8. "Who Should Know This?"

A very effective design question is:

> Which object should be the authoritative source for this fact?

Example:

```text
Where should order status live?

Order
```

not:

```text
OrderController.status
OrderService.status
OrderRepository.status
```

The repository persists status; it does not become the domain authority.

Another example:

```text
Who should know whether an account can withdraw?
```

The account.

Not:

```ts
BankService.canWithdraw(account)
```

unless the decision genuinely depends on information external to the account.

---

# 9. "Who Should Do This?"

The second fundamental question is:

> Which object should perform this behavior?

Consider:

```ts
cart.addItem(product, quantity);
```

versus:

```ts
cartService.addItem(cart, product, quantity);
```

If adding an item requires changing cart state and enforcing cart-specific invariants, the cart is a strong responsibility candidate.

---

# 10. Responsibility and Invariants

Responsibility is tightly connected to invariants.

An invariant is a condition that must remain true.

Example:

```text
Order quantity > 0
Account balance >= 0
Invoice total >= 0
Cart item quantity >= 1
```

The strongest design often puts the invariant and the state-changing behavior together.

```ts
class CartLine {
  #quantity: number;

  constructor(
    readonly productId: string,
    quantity: number
  ) {
    this.#setQuantity(quantity);
  }

  increaseBy(amount: number) {
    this.#setQuantity(this.#quantity + amount);
  }

  private #setQuantity(quantity: number) {
    if (!Number.isInteger(quantity) || quantity <= 0) {
      throw new Error("Quantity must be a positive integer");
    }

    this.#quantity = quantity;
  }
}
```

A caller cannot silently violate the rule through direct mutation.

---

# 11. Responsibility as Authority

A deeper model is:

```text
State ownership
        ↓
Decision authority
        ↓
Behavior responsibility
        ↓
Invariant protection
```

If a component owns state but another component has unrestricted authority to change it, the model is fractured.

Example of fractured ownership:

```ts
order.status = "PAID";
```

from anywhere in the system.

Better:

```ts
order.markPaid(paymentId);
```

The order can validate:

```text
PENDING → PAID
```

while rejecting:

```text
CANCELLED → PAID
```

The method becomes a controlled state transition.

---

# 12. Creator Responsibility

The **Creator** heuristic asks:

> Which object should be responsible for creating another object?

Strong creator candidates often include an object that:

- contains the created object
- aggregates the created object
- closely uses the created object
- has initialization information
- records the created object's lifecycle

Example:

```ts
class Cart {
  #lines: CartLine[] = [];

  addProduct(productId: string, quantity: number) {
    const line = new CartLine(productId, quantity);
    this.#lines.push(line);
  }
}
```

The cart creates the line because the line belongs to the cart's collection and represents cart state.

---

# 13. Creator Does Not Mean "Always Use `new` Inside the Aggregate"

Dependency boundaries matter.

For more complex construction:

```ts
class OrderFactory {
  create(command: CreateOrderCommand): Order {
    // complex construction policy
  }
}
```

Now the factory may own the creation responsibility.

Examples where a factory becomes useful:

- multiple construction variants
- expensive initialization
- dependency graph assembly
- configuration-driven creation
- polymorphic creation
- validation requiring several inputs
- hiding concrete implementations

The principle is not:

> "Factories are always good."

It is:

> **Put creation where the lifecycle and construction knowledge naturally belongs.**

---

# 14. Controller Responsibility

A **Controller** in responsibility-driven design is commonly responsible for receiving a system-level operation and delegating to domain/application collaborators.

Example:

```ts
class CreateOrderController {
  constructor(
    private readonly checkout: CheckoutApplicationService
  ) {}

  async handle(input: CreateOrderHttpRequest) {
    const result = await this.checkout.createOrder(input);

    return {
      statusCode: 201,
      body: result
    };
  }
}
```

The controller should usually not contain:

```text
pricing rules
inventory policy
payment rules
order invariants
database query details
```

It is a boundary responsibility.

---

# 15. Controller vs Application Service vs Domain Object

These roles are commonly confused.

A useful distinction:

```text
Transport Controller
    ↓
interprets external request
    ↓
Application Service / Use Case
    ↓
coordinates workflow
    ↓
Domain Objects
    ↓
enforce domain behavior/invariants
```

Example:

```text
HTTP request
   ↓
OrderController
   ↓
CreateOrderUseCase
   ├── CustomerRepository
   ├── OrderFactory
   ├── PricingPolicy
   └── Order
```

A controller should not become the application's business-rule dumping ground.

---

# 16. Application Service Responsibility

An application service usually coordinates a **use case**.

Typical responsibilities:

- start the operation
- load required objects
- invoke domain behavior
- coordinate external collaborators
- manage transaction boundaries
- map results to application output
- enforce application-level authorization checks where appropriate

Typical non-responsibilities:

- becoming a replacement for every domain object
- containing every calculation
- directly manipulating entity internals
- becoming a collection of static utility methods

Example:

```ts
class TransferMoney {
  constructor(
    private readonly accounts: AccountRepository,
    private readonly auditLog: AuditLog
  ) {}

  async execute(command: TransferCommand) {
    const source = await this.accounts.getById(command.sourceId);
    const target = await this.accounts.getById(command.targetId);

    source.withdraw(command.amount);
    target.deposit(command.amount);

    await this.accounts.save(source);
    await this.accounts.save(target);

    await this.auditLog.record({
      type: "MONEY_TRANSFERRED",
      sourceId: source.id,
      targetId: target.id,
      amount: command.amount
    });
  }
}
```

The application service coordinates.

The account owns withdrawal/deposit rules.

---

# 17. Pure Fabrication

Sometimes no domain object is a natural information expert.

A **Pure Fabrication** is an invented object introduced to achieve good design quality.

Examples:

```text
EmailSender
AuditLogger
PasswordHasher
ReportRenderer
PaymentGatewayAdapter
```

These are often not natural business entities.

They exist because creating them improves:

- cohesion
- coupling
- testability
- reuse
- replaceability

Example:

Bad:

```ts
class Customer {
  async sendWelcomeEmail() {
    // SMTP details
  }
}
```

Better:

```ts
class WelcomeEmailSender {
  constructor(private readonly mailer: Mailer) {}

  async send(customer: Customer) {
    await this.mailer.send({
      to: customer.email,
      template: "welcome"
    });
  }
}
```

The customer does not need to know SMTP.

---

# 18. Indirection

**Indirection** assigns responsibility to an intermediary to avoid direct coupling.

Example:

```text
Order
  ↓
PaymentGateway
  ↓
StripeAdapter
  ↓
Stripe
```

instead of:

```text
Order
  ↓
Stripe SDK
```

The abstraction protects the domain from infrastructure details.

---

# 19. Polymorphism as Responsibility Allocation

When behavior varies by type, polymorphism can place the responsibility on the varying object.

Bad:

```ts
function calculateShipping(order: Order, type: string) {
  if (type === "STANDARD") {
    return 50;
  }

  if (type === "EXPRESS") {
    return 150;
  }

  if (type === "INTERNATIONAL") {
    return 500;
  }

  throw new Error("Unsupported type");
}
```

Alternative:

```ts
interface ShippingMethod {
  calculate(order: Order): Money;
}

class StandardShipping implements ShippingMethod {
  calculate(order: Order) {
    return Money.of(50);
  }
}

class ExpressShipping implements ShippingMethod {
  calculate(order: Order) {
    return Money.of(150);
  }
}
```

Now the varying behavior owns itself.

The responsibility is distributed according to behavior variation.

---

# 20. Protected Variations

**Protected Variations** means identifying a point of likely change and creating a stable boundary around it.

Potential variation points:

- payment provider
- tax engine
- shipping algorithm
- notification mechanism
- database implementation
- clock
- ID generation
- external API
- discount policy

Example:

```ts
interface TaxPolicy {
  calculate(input: TaxInput): Money;
}
```

The order depends on the stable concept:

```text
TaxPolicy
```

rather than:

```text
SpecificTaxVendorSDK
```

This is especially important in enterprise systems where change is normal.

---

# 21. Low Coupling and High Cohesion

Responsibility assignment cannot be separated from coupling and cohesion.

A responsibility should generally move toward an object when that move:

- increases cohesion
- reduces unnecessary coupling
- preserves invariants
- clarifies ownership
- minimizes change amplification

Example:

Bad:

```ts
class OrderService {
  calculateTotal(order: Order) {}
  renameCustomer(customer: Customer) {}
  generateInvoice(invoice: Invoice) {}
  sendEmail(customer: Customer) {}
}
```

This has several unrelated responsibility clusters.

Better:

```text
Order
Customer
Invoice
WelcomeEmailSender
```

with an application service coordinating the use case where necessary.

---

# 22. Tell, Don't Ask

The **Tell, Don't Ask** heuristic says:

> Tell an object what to do rather than asking for its internal data and making the decision elsewhere.

Bad:

```ts
if (order.status === "PENDING") {
  order.status = "CONFIRMED";
}
```

Better:

```ts
order.confirm();
```

Why?

Because:

```text
decision rule
+
state transition
```

stay together.

The object controls its own invariant.

---

# 23. Tell, Don't Ask Is Not "Never Use Getters"

Getters are sometimes appropriate.

There is no universal rule that:

```ts
get status()
```

is bad.

The problem is uncontrolled domain decision-making outside the owner.

Useful:

```ts
if (order.isOverdue(now)) {
  ...
}
```

Potentially weaker:

```ts
if (order.dueDate < now && order.status !== "PAID") {
  ...
}
```

The latter leaks domain knowledge.

Use getters when reading information is genuinely part of the object's public contract.

Use behavior methods when callers should not duplicate the object's rules.

---

# 24. Law of Demeter — Nuanced Use

A common simplification is:

> "Never chain method calls."

That is too crude.

The more useful idea is:

> An object should minimize dependence on distant object structure.

Risky:

```ts
order.getCustomer()
    .getAccount()
    .getAddress()
    .getCountry()
    .getTaxProfile();
```

Better:

```ts
order.taxRegion();
```

The order can delegate internally.

The goal is not to ban every chain such as:

```ts
users.filter(...).map(...)
```

The goal is to avoid clients depending on unstable object topology.

---

# 25. Anemic Domain Model

An anemic model stores domain state while pushing most domain behavior into services.

Example:

```ts
class Invoice {
  total!: number;
  status!: string;
}
```

and:

```ts
class InvoiceService {
  approve(invoice: Invoice) {}
  cancel(invoice: Invoice) {}
  calculateLateFee(invoice: Invoice) {}
}
```

This may be reasonable in simple CRUD systems.

It becomes dangerous when:

- invariants are complex
- state transitions matter
- business rules are numerous
- the same entity is manipulated from many services
- rules become duplicated

The key question is not:

> "Are services bad?"

It is:

> "Where should the domain decision and invariant live?"

---

# 26. Rich Domain Model

A richer model keeps meaningful behavior close to domain state.

```ts
class Invoice {
  #status: InvoiceStatus = "DRAFT";

  approve() {
    if (this.#status !== "DRAFT") {
      throw new Error("Only draft invoices can be approved");
    }

    this.#status = "APPROVED";
  }

  cancel() {
    if (this.#status === "PAID") {
      throw new Error("Paid invoices cannot be cancelled");
    }

    this.#status = "CANCELLED";
  }
}
```

Now invalid transitions are harder to express.

---

# 27. Domain Service

Not every behavior naturally belongs to an entity.

A **domain service** may be justified when:

- the behavior is domain-significant
- it spans multiple domain objects
- no single object is the natural expert
- putting it on one entity would distort cohesion

Example:

```ts
class CurrencyExchangeService {
  exchange(
    money: Money,
    targetCurrency: Currency,
    rate: ExchangeRate
  ): Money {
    return ...
  }
}
```

Do not create domain services merely because a method "sounds complicated."

Complexity alone does not justify externalizing behavior.

---

# 28. Repository Responsibility

A repository usually owns persistence-oriented responsibilities such as:

```text
retrieve aggregate
save aggregate
query by domain-relevant criteria
```

Example:

```ts
interface OrderRepository {
  findById(id: OrderId): Promise<Order | null>;
  save(order: Order): Promise<void>;
}
```

The repository should not become the place for:

```text
pricing rules
workflow decisions
authorization policy
business state transitions
```

Those belong elsewhere.

---

# 29. Factory Responsibility

Factories own controlled construction when object creation has meaningful policy.

```ts
class OrderFactory {
  constructor(
    private readonly idGenerator: IdGenerator,
    private readonly clock: Clock
  ) {}

  create(customerId: CustomerId): Order {
    return new Order(
      this.idGenerator.generate(),
      customerId,
      this.clock.now()
    );
  }
}
```

This is useful when construction needs external dependencies.

The entity should not necessarily create its own infrastructure dependencies.

---

# 30. Responsibility Matrix

Before writing code, you can create a simple matrix.

| Concern | Candidate | Reason |
|---|---|---|
| Validate quantity | OrderLine | Owns quantity |
| Calculate line subtotal | OrderLine | Owns price + quantity |
| Calculate order total | Order | Owns lines |
| Approve invoice | Invoice | Owns status transition |
| Load order | OrderRepository | Persistence responsibility |
| Coordinate checkout | CheckoutUseCase | Use-case coordination |
| Send email | EmailSender | External communication |
| Create complex order | OrderFactory | Construction policy |
| Determine tax | TaxPolicy | Variation point |
| Convert HTTP request | Controller | Transport boundary |

This makes design choices explicit.

---

# 31. CRC Cards

**CRC** means:

```text
Class
Responsibilities
Collaborators
```

It is a lightweight technique for modeling responsibilities before implementation.

A CRC card might look like:

```text
Class: Order

Responsibilities:
- maintain order lines
- calculate total
- confirm order
- cancel order

Collaborators:
- Customer
- Payment
- PricingPolicy
```

Another:

```text
Class: CheckoutUseCase

Responsibilities:
- load cart
- create order
- reserve stock
- charge payment
- save order

Collaborators:
- CartRepository
- Inventory
- PaymentGateway
- OrderRepository
```

CRC forces you to think in terms of behavior and collaboration instead of fields alone.

---

# 32. CRC Modeling Example

Suppose a jewellery ERP needs:

```text
SalesOrder
SalesOrderLine
Customer
Inventory
Payment
```

The first instinct may be:

```text
SalesOrderService
```

Instead, write CRC cards.

### SalesOrder

Responsibilities:

- own line items
- calculate gross amount
- apply order-level state transitions
- enforce order invariants

Collaborators:

- SalesOrderLine
- DiscountPolicy

### SalesOrderLine

Responsibilities:

- own quantity
- know item price
- calculate line amount

Collaborators:

- Product
- Money

### CheckoutUseCase

Responsibilities:

- load order
- reserve stock
- process payment
- persist successful state

Collaborators:

- OrderRepository
- Inventory
- PaymentGateway

The resulting design is easier to reason about.

---

# 33. Responsibility Matrix for a Use Case

Use-case modeling can be expressed as:

```text
Actor
  ↓
Controller
  ↓
Application Service
  ↓
Domain Objects
  ↓
Infrastructure Ports
```

Example:

```text
Customer
  ↓
CreateSaleController
  ↓
CreateSaleUseCase
  ├── CustomerRepository
  ├── SaleFactory
  ├── Sale
  ├── Inventory
  └── PaymentGateway
```

The key is to separate:

```text
who receives
who coordinates
who decides
who persists
who integrates
```

---

# 34. Message-Oriented Design

Objects should be understood as collaborators exchanging messages.

Instead of designing:

```text
classes + properties
```

design:

```text
objects + messages + responsibilities
```

Example:

```text
CheckoutUseCase
    └── checkout(cartId)

Cart
    └── checkout()

Inventory
    └── reserve(lines)

PaymentGateway
    └── charge(payment)

OrderRepository
    └── save(order)
```

This reveals the dynamic behavior of the system.

---

# 35. Scenario-Driven Responsibility Assignment

Responsibility should be validated against actual scenarios.

Take:

```text
Customer attempts checkout.
```

Trace:

```text
1. Receive checkout command.
2. Load cart.
3. Ask cart to create/check out an order.
4. Reserve inventory.
5. Charge payment.
6. Persist order.
7. Return result.
```

Now ask:

```text
Who owns each decision?
Who owns each state transition?
Who merely coordinates?
```

A good responsibility design emerges from scenarios.

---

# 36. Interaction Diagram as a Validation Tool

Represent the scenario as messages:

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
  | create Order
  v
OrderFactory
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

If one object receives almost every message:

```text
God Object risk
```

If every object exposes internal state:

```text
Encapsulation risk
```

If the controller makes every decision:

```text
Responsibility inversion
```

---

# 37. Delegation vs Orchestration

A common confusion:

```text
delegation
```

versus:

```text
orchestration
```

Delegation:

```ts
order.confirm();
```

The order performs the relevant behavior.

Orchestration:

```ts
await checkout.execute(command);
```

The application service coordinates multiple participants.

A useful distinction:

```text
One object owns the decision
    → delegate

Several collaborators must be sequenced
    → orchestrate
```

---

# 38. Avoiding Manager / Service Dumping Grounds

Warning signs:

```text
OrderManager
OrderService
OrderHelper
OrderProcessor
OrderCoordinator
OrderUtil
```

all contain pieces of order behavior.

This often means responsibility assignment was never resolved.

Instead, group behavior by coherent responsibility:

```text
Order
OrderLine
OrderPolicy
OrderFactory
OrderRepository
CheckoutUseCase
```

Do not force every behavior into `Order`.

Do not force every behavior out of `Order`.

Make the responsibility explicit.

---

# 39. God Object

A god object often has:

- too much state
- too many methods
- too many collaborators
- many unrelated reasons to change
- high fan-out
- low conceptual cohesion

Example:

```text
ERPManager
```

knows:

```text
customer
product
inventory
pricing
payment
invoice
reporting
notifications
audit
authentication
```

This is not just a "large class."

The deeper issue is:

> **The object owns decisions that belong to different responsibility clusters.**

Refactor by identifying responsibilities, not by arbitrarily extracting methods.

---

# 40. Feature Envy as a Responsibility Signal

Feature envy occurs when one object repeatedly uses another object's data.

Example:

```ts
class InvoiceRenderer {
  render(invoice: Invoice) {
    return `
      ${invoice.customer.name}
      ${invoice.customer.address.city}
      ${invoice.lines.map(
        line => line.quantity * line.price
      )}
    `;
  }
}
```

Rendering itself may belong in a renderer.

But repeated business calculations over foreign data can indicate misplaced behavior.

Ask:

```text
Why does this object know so much about another object's representation?
```

---

# 41. Data Clumps as Responsibility Signals

Repeated parameter groups are a design smell.

Bad:

```ts
createOrder(
  customerId,
  currency,
  country,
  taxRate
);
```

Repeated elsewhere:

```ts
calculateTax(
  customerId,
  currency,
  country,
  taxRate
);
```

Potentially:

```ts
interface TaxContext {
  currency: Currency;
  country: Country;
  taxRate: TaxRate;
}
```

The point is not "always make a class."

The point is to discover a stable concept carrying related responsibility.

---

# 42. Primitive Obsession as a Responsibility Signal

This:

```ts
type OrderStatus = string;
```

allows arbitrary states.

A stronger domain boundary may be:

```ts
type OrderStatus =
  | "DRAFT"
  | "CONFIRMED"
  | "CANCELLED"
  | "PAID";
```

Or a behavior-oriented object.

```ts
class OrderStatus {
  constructor(private readonly value: string) {
    // validate supported state
  }

  canTransitionTo(next: OrderStatus) {
    // transition policy
  }
}
```

As complexity increases, state representation itself becomes a responsibility.

---

# 43. State Ownership Example

Suppose:

```ts
class ShoppingCart {
  #items: CartItem[] = [];

  clear() {
    this.#items = [];
  }
}
```

Who should clear it?

The cart.

Bad:

```ts
cart.items = [];
```

The caller takes control of internal representation.

Better:

```ts
cart.clear();
```

The responsibility is:

```text
Cart owns collection lifecycle.
```

---

# 44. Collection Ownership

A subtle but important question is:

> Does the object own the collection, or merely expose a collection?

Compare:

```ts
class Cart {
  items: CartItem[] = [];
}
```

with:

```ts
class Cart {
  #items: CartItem[] = [];

  get items(): readonly CartItem[] {
    return this.#items;
  }
}
```

The second design communicates ownership more clearly.

For mutable collections, a useful rule is:

```text
If an object owns the invariant,
it should control mutation of the collection.
```

---

# 45. Responsibility and `readonly`

TypeScript:

```ts
readonly items: readonly CartItem[];
```

can express some design intent at compile time.

But remember:

```text
TypeScript readonly
    ≠
Runtime immutability
```

Responsibility still has to be enforced through runtime behavior when untrusted or dynamic JavaScript crosses the boundary.

---

# 46. JavaScript-Specific Responsibility Design

JavaScript gives many mechanisms for responsibility boundaries.

## Private fields

```ts
class Account {
  #balance = 0;
}
```

Good for strong runtime encapsulation.

## Closures

```ts
function createCounter() {
  let count = 0;

  return {
    increment() {
      count++;
    },
    value() {
      return count;
    }
  };
}
```

Good for module/factory-level private state.

## Modules

```ts
const SECRET = Symbol("secret");

export function createThing() {}
```

Good for module-scoped implementation details.

## WeakMap

Useful when external identity should map to private metadata without exposing storage.

The underlying principle remains:

> Choose the mechanism that best preserves the responsibility boundary.

---

# 47. TypeScript Interfaces as Responsibility Contracts

Interfaces can clarify which responsibility an object promises.

```ts
interface TaxCalculator {
  calculate(input: TaxInput): Money;
}
```

Now the caller depends on a responsibility:

```text
calculate tax
```

rather than implementation:

```text
StripeTaxSdkAdapter
```

This supports dependency inversion and protected variations.

---

# 48. Structural Typing and Responsibility

TypeScript is structurally typed.

This means:

```ts
interface Clock {
  now(): Date;
}
```

does not require a particular class.

Any structurally compatible object can satisfy it.

```ts
const fakeClock = {
  now: () => new Date("2026-01-01T00:00:00Z")
};
```

This can improve testability.

The interface documents responsibility:

```text
Clock = responsibility to provide current time
```

rather than:

```text
SystemClock = specific implementation
```

---

# 49. Dependency Direction

Responsibility design should also control dependency direction.

Weak:

```text
Domain → Express
Domain → Prisma
Domain → Stripe
```

Stronger:

```text
Domain
  ↓
stable ports

Infrastructure
  ↓
implements ports
```

Example:

```ts
interface PaymentGateway {
  charge(request: ChargeRequest): Promise<ChargeResult>;
}
```

Infrastructure:

```ts
class StripePaymentGateway implements PaymentGateway {
  ...
}
```

The responsibility boundary is explicit.

---

# 50. Responsibility Placement and Change

A responsibility should be placed where the expected change is local.

Suppose tax rules change frequently.

Bad:

```text
Order
  + country-specific tax logic
  + vendor-specific tax API
  + database query
```

Better:

```text
Order
TaxPolicy
TaxGateway
TaxRepository
```

Now different change scenarios affect smaller units.

This is a practical combination of:

```text
Responsibility assignment
+
Cohesion
+
Protected Variations
```

---

# 51. Responsibility Assignment Heuristic Stack

A useful ordering is:

```text
1. Who owns the state?
2. Who has the required information?
3. Who should enforce the invariant?
4. Would placing it there keep cohesion high?
5. Would it create unnecessary coupling?
6. Is the behavior stable or variable?
7. Is this domain behavior or coordination?
8. Does another abstraction provide a cleaner boundary?
9. Will the resulting design be easy to test?
10. What change scenario will stress this decision?
```

No single heuristic is always dominant.

---

# 52. Example — Bad Responsibility Allocation

Consider:

```ts
class OrderService {
  calculateTotal(order: Order) {
    return order.items.reduce(
      (total, item) =>
        total + item.price * item.quantity,
      0
    );
  }

  confirm(order: Order) {
    if (order.status !== "PENDING") {
      throw new Error("Invalid state");
    }

    order.status = "CONFIRMED";
  }

  addItem(order: Order, item: OrderItem) {
    order.items.push(item);
  }
}
```

Problems:

```text
Order has state
OrderService has business decisions
OrderService mutates Order internals
Rules are outside their state owner
```

---

# 53. Refactored Responsibility Allocation

```ts
class Order {
  #status: OrderStatus = "PENDING";
  #items: OrderItem[] = [];

  addItem(item: OrderItem) {
    this.#items.push(item);
  }

  total() {
    return this.#items.reduce(
      (total, item) =>
        total + item.subtotal(),
      0
    );
  }

  confirm() {
    if (this.#status !== "PENDING") {
      throw new Error("Invalid state");
    }

    this.#status = "CONFIRMED";
  }
}
```

Then:

```ts
class ConfirmOrder {
  constructor(
    private readonly repository: OrderRepository
  ) {}

  async execute(orderId: string) {
    const order = await this.repository.findById(orderId);

    if (!order) {
      throw new Error("Order not found");
    }

    order.confirm();

    await this.repository.save(order);
  }
}
```

Now:

```text
Order
    owns business behavior

Use case
    coordinates application flow

Repository
    owns persistence
```

---

# 54. A More Difficult Example

Suppose order pricing requires:

```text
base item price
+
customer discount
+
promotion
+
tax
+
shipping
```

A naive design could put everything in `Order`.

But responsibility analysis may produce:

```text
Order
  → owns lines

OrderLine
  → knows line subtotal

DiscountPolicy
  → calculates discount

PromotionPolicy
  → applies promotion rules

TaxPolicy
  → calculates tax

ShippingPolicy
  → calculates shipping

OrderPricingService
  → combines pricing policies
```

The final design depends on domain boundaries and change patterns.

The right answer is not:

> "Always use Information Expert."

The right answer is:

> **Use Information Expert while preserving cohesion and variation boundaries.**

---

# 55. Responsibility Splitting by Change Scenario

Suppose:

```text
Tax rules change weekly.
Order state rules change rarely.
Shipping providers change quarterly.
```

Then:

```text
Tax responsibility
    → isolated

Order state transitions
    → Order

Shipping integration
    → ShippingGateway
```

A good design aligns responsibilities with change boundaries.

---

# 56. Responsibility Splitting by Invariant

Suppose:

```text
quantity > 0
```

and:

```text
line total = unit price × quantity
```

Both depend directly on line state.

Therefore:

```text
OrderLine
```

is a strong owner.

If:

```text
tax depends on country + jurisdiction + exemption status
```

the expert may be:

```text
TaxPolicy
```

not necessarily the order.

---

# 57. Responsibility Splitting by Collaboration

Suppose:

```text
Checkout
```

needs:

```text
Cart
Inventory
Payment
OrderRepository
```

A single application service can coordinate these.

But domain decisions stay local:

```text
Cart.checkout()
Inventory.reserve()
PaymentGateway.charge()
Order.markPaid()
```

This avoids:

```text
ApplicationService manipulating every field.
```

---

# 58. Avoiding Over-Objectification

Responsibility-driven design does not mean:

```text
One concept = one class
```

A function may be the correct abstraction.

Example:

```ts
function clamp(value: number, min: number, max: number) {
  return Math.min(max, Math.max(min, value));
}
```

Creating:

```text
ClampStrategyFactory
ClampProcessor
ClampService
```

would be absurd.

Responsibility must be proportional to domain complexity.

---

# 59. When a Function Is the Better Responsibility Holder

Prefer a function when:

- behavior is pure
- state is not independently meaningful
- no lifecycle is needed
- no polymorphic collaboration is needed
- the abstraction has one obvious operation
- introducing an object would only add ceremony

Example:

```ts
function calculateSubtotal(
  price: number,
  quantity: number
) {
  return price * quantity;
}
```

Object design is a tool, not a goal.

---

# 60. When a Value Object Is Better

If a concept has behavior and invariants but no identity:

```text
Money
EmailAddress
PhoneNumber
Quantity
Percentage
DateRange
```

a value object can own the responsibility.

```ts
class Money {
  constructor(
    readonly amount: number,
    readonly currency: string
  ) {
    if (!Number.isFinite(amount)) {
      throw new Error("Invalid amount");
    }
  }

  add(other: Money) {
    if (other.currency !== this.currency) {
      throw new Error("Currency mismatch");
    }

    return new Money(
      this.amount + other.amount,
      this.currency
    );
  }
}
```

The responsibility is attached to the concept.

---

# 61. When an Entity Is Better

Use an entity when:

- identity matters
- lifecycle matters
- state transitions matter
- behavior is centered on that identity

Example:

```text
Order
Customer
Invoice
Warehouse
Product
```

---

# 62. When a Domain Service Is Better

Use a domain service when:

```text
behavior is domain-significant
AND
multiple objects participate
AND
no single object is the natural owner
```

Do not use it merely because:

```text
"service classes are common."
```

---

# 63. When an Application Service Is Better

Use an application service when the responsibility is:

```text
use-case coordination
```

not:

```text
entity invariant enforcement
```

Example:

```text
PlaceOrderUseCase
CancelOrderUseCase
GenerateInvoiceUseCase
TransferMoneyUseCase
```

---

# 64. Repository as an Architectural Responsibility

Repositories form a persistence boundary.

```ts
interface CustomerRepository {
  findById(id: CustomerId): Promise<Customer | null>;
  save(customer: Customer): Promise<void>;
}
```

The repository hides persistence mechanics.

The customer should not contain:

```ts
await prisma.customer.update(...)
```

That would collapse domain and persistence responsibilities.

---

# 65. Responsibility and Transactions

Responsibility also affects transactional boundaries.

Example:

```text
TransferMoney
```

may coordinate:

```text
withdraw source
deposit target
save both
```

The use-case layer may define the transaction boundary.

But the account remains responsible for:

```text
withdrawal validity
deposit validity
```

This distinction will become important in later transaction and concurrency chapters.

---

# 66. Responsibility and Concurrency

Suppose:

```text
inventory.reserve()
```

owns the business rule:

```text
available quantity must not become negative
```

But concurrency control may additionally require:

- database transaction
- optimistic locking
- atomic update
- distributed lock

Therefore:

```text
Domain responsibility
    ≠
all technical enforcement mechanisms
```

A responsibility can cross abstraction layers while ownership of the decision remains clear.

---

# 67. Responsibility and Security

Security-sensitive responsibilities should be explicit.

Examples:

```text
PasswordHasher
AuthorizationPolicy
TokenVerifier
AuditLogger
TenantResolver
```

Do not scatter authorization checks:

```ts
if (user.role === "admin") ...
```

through unrelated domain objects.

Instead define clear policy responsibilities.

```ts
interface PermissionChecker {
  can(
    actor: Actor,
    action: Action,
    resource: Resource
  ): boolean;
}
```

The exact placement depends on whether the rule is application, domain, or infrastructure-specific.

---

# 68. Multi-Tenant Responsibility

In a multi-tenant system, ask:

```text
Who is responsible for tenant context?
Who validates tenant ownership?
Who filters tenant-scoped persistence?
```

Do not assume:

```text
controller
```

should be the only protection.

A robust design may use several boundaries:

```text
Request context
    ↓
Application authorization
    ↓
Domain tenant rules
    ↓
Repository tenant scoping
    ↓
Database isolation constraints
```

Responsibility can be layered for defense in depth.

---

# 69. Responsibility and Observability

Observability should not accidentally become business logic.

Instead:

```text
Order
    emits domain-relevant event
        ↓
Application/Infrastructure
    records metrics/logs/traces
```

This helps keep:

```text
business responsibility
```

separate from:

```text
operational telemetry responsibility
```

---

# 70. Responsibility and Testing

Tests reveal responsibility placement.

A good unit test often reads naturally:

```ts
order.confirm();
```

rather than:

```ts
orderService.confirm(order);
```

when confirmation is an order invariant.

Application tests should validate coordination:

```ts
await useCase.execute(command);
```

Infrastructure tests should validate adapters/repositories.

A useful test taxonomy:

```text
Entity/value-object tests
    → domain behavior

Use-case tests
    → collaboration/orchestration

Repository tests
    → persistence behavior

Adapter tests
    → external integration behavior
```

---

# 71. Tests as Responsibility Evidence

Ask:

> Can I test this responsibility without constructing unrelated infrastructure?

If the answer is:

```text
No — I need database + HTTP + payment + queue
```

the responsibility may be misplaced.

Not always, but it is a strong smell.

---

# 72. Responsibility-Driven Refactoring Workflow

When reviewing an existing class:

### Step 1

List all public methods.

### Step 2

For each method, ask:

```text
What does this method know?
What state does it change?
What business rule does it enforce?
What external system does it call?
```

### Step 3

Group methods by responsibility.

### Step 4

Identify groups with unrelated change reasons.

### Step 5

Find the natural owner.

### Step 6

Move behavior toward the owner.

### Step 7

Introduce collaborators for true cross-object concerns.

### Step 8

Protect variation points.

### Step 9

Run scenario tests.

### Step 10

Re-evaluate coupling and cohesion.

---

# 73. Refactoring Exercise — Service Dumping Ground

Given:

```ts
class UserService {
  createUser() {}
  changeEmail() {}
  resetPassword() {}
  sendWelcomeEmail() {}
  saveUser() {}
  generateReport() {}
}
```

Classify:

```text
createUser
    → factory/application responsibility

changeEmail
    → user/domain responsibility

resetPassword
    → user/security/domain/application boundary

sendWelcomeEmail
    → notification responsibility

saveUser
    → repository responsibility

generateReport
    → reporting responsibility
```

The important result is not the exact final class list.

The result is:

> Each responsibility has a reason for existing.

---

# 74. Refactoring Exercise — Feature Envy

Bad:

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

Potential decomposition:

```text
Order
    → owns line collection

OrderLine
    → owns line amount

Product
    → owns category

DiscountPolicy
    → owns discount rule
```

The exact placement depends on which facts are stable and which rules vary.

---

# 75. Refactoring Exercise — God Object

Bad:

```ts
class EcommerceManager {
  users = [];
  orders = [];
  products = [];
  inventory = [];
  payments = [];

  createUser() {}
  placeOrder() {}
  reserveStock() {}
  chargePayment() {}
  refundPayment() {}
  sendEmail() {}
  exportCsv() {}
  authenticate() {}
}
```

Do not immediately extract eight classes.

First create a responsibility inventory:

```text
Identity
Ordering
Inventory
Payment
Notification
Reporting
Authentication
```

Then identify actual boundaries.

---

# 76. Responsibility Inventory Worksheet

For any feature, write:

```text
Feature:
____________________________________

Business goal:
____________________________________

State involved:
____________________________________

Invariants:
____________________________________

Who owns the state?
____________________________________

Who has the necessary information?
____________________________________

Who performs the behavior?
____________________________________

Who coordinates the use case?
____________________________________

Who persists?
____________________________________

What changes frequently?
____________________________________

What external system varies?
____________________________________
```

This becomes a repeatable design tool.

---

# 77. Responsibility Review Questions

Before approving a design, ask:

```text
1. Is every important state owned?
2. Is every invariant protected?
3. Are business decisions near the right knowledge?
4. Are workflows separated from domain rules?
5. Are external dependencies behind stable boundaries?
6. Are responsibilities cohesive?
7. Are collaborators necessary?
8. Is there hidden duplication of rules?
9. Is one object becoming the system brain?
10. Can the design survive the next major change?
```

---

# 78. Production Example — Jewellery Sale

Consider a sale workflow:

```text
Create sale
    ↓
Select customer
    ↓
Add jewellery items
    ↓
Validate quantities
    ↓
Calculate metal/value pricing
    ↓
Apply discount
    ↓
Calculate tax
    ↓
Reserve inventory
    ↓
Receive payment
    ↓
Finalize sale
    ↓
Persist
    ↓
Audit
```

Responsibility candidates:

```text
Sale
    owns sale lifecycle

SaleLine
    owns line quantity and value

PricingPolicy
    owns pricing variation

DiscountPolicy
    owns discounts

TaxPolicy
    owns tax rules

Inventory
    owns stock reservation

PaymentGateway
    owns payment integration

FinalizeSaleUseCase
    coordinates workflow

SaleRepository
    owns persistence

AuditLog
    owns audit recording
```

Notice what is absent:

```text
JewellerySaleManager
```

There may still be a class with a similar name in a real codebase, but the design begins with responsibilities rather than naming conventions.

---

# 79. Message Trace — Jewellery Sale

A simplified collaboration:

```text
SalesController
    |
    | execute(command)
    v
FinalizeSaleUseCase
    |
    | load()
    v
SaleRepository
    |
    | return Sale
    v
FinalizeSaleUseCase
    |
    | finalize()
    v
Sale
    |
    | amount()
    v
PricingPolicy
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
SaleRepository
    |
    | record()
    v
AuditLog
```

Each message should have an understandable reason to exist.

---

# 80. Responsibility and Domain Events

Sometimes an object performs a state transition:

```ts
sale.finalize();
```

and records an event:

```text
SaleFinalized
```

The domain object is responsible for the transition.

A separate application/infrastructure component may be responsible for publishing or transporting the event.

This keeps:

```text
business state transition
```

separate from:

```text
delivery mechanism
```

---

# 81. Responsibility and Dependency Injection

Dependency injection is not itself responsibility assignment.

It is a mechanism for providing collaborators.

Example:

```ts
class PlaceOrderUseCase {
  constructor(
    private readonly repository: OrderRepository,
    private readonly inventory: Inventory
  ) {}
}
```

The class still needs a coherent responsibility.

Bad:

```text
GodService
with
50 injected dependencies
```

is still a god object.

Injection does not repair poor design.

---

# 82. Responsibility and Constructor Size

Large constructors can be a symptom.

```ts
class OrderManager {
  constructor(
    a,
    b,
    c,
    d,
    e,
    f,
    g,
    h
  ) {}
}
```

Do not use a numeric rule such as:

> "More than 5 dependencies is always bad."

Instead ask:

```text
Do these dependencies support one coherent responsibility?
```

A workflow orchestrator may legitimately coordinate several ports.

The important signal is cohesion.

---

# 83. Responsibility and Composition

Composition works naturally with responsibility-driven design.

```text
Order
├── PricingPolicy
├── DiscountPolicy
└── TaxPolicy
```

Each collaborator owns a focused concern.

This makes the design easier to extend than a hierarchy that attempts to encode every variation through inheritance.

---

# 84. Responsibility and Inheritance

Inheritance can blur responsibility.

Example:

```ts
class BaseOrder {
  calculateTotal() {}
}

class InternationalOrder extends BaseOrder {
  calculateTax() {}
}
```

Ask:

```text
Is tax actually an identity difference?
Or is it a policy variation?
```

Often:

```text
TaxPolicy
```

is a cleaner responsibility boundary.

Use inheritance when the subtype genuinely is substitutable and shares a stable responsibility structure.

---

# 85. Responsibility and Polymorphism

Polymorphism is especially powerful when a responsibility has stable input/output but varying implementation.

```ts
interface PricingPolicy {
  price(order: Order): Money;
}
```

Now:

```text
RetailPricingPolicy
WholesalePricingPolicy
MemberPricingPolicy
SeasonalPricingPolicy
```

can vary independently.

This prepares directly for the formal GRASP Polymorphism discussion.

---

# 86. Responsibility and Protected Variations

For every major dependency, ask:

```text
What am I afraid will change?
```

Examples:

```text
Stripe → payment provider may change
PostgreSQL → persistence technology may change
system time → clock behavior may be nondeterministic in tests
Math.random → ID generation may change
SMTP → messaging provider may change
```

Place stable boundaries around meaningful variation points.

---

# 87. Responsibility and Time

Time is a dependency.

Bad:

```ts
class Subscription {
  isExpired() {
    return new Date() > this.expiresAt;
  }
}
```

More testable:

```ts
class Subscription {
  isExpired(now: Date) {
    return now >= this.expiresAt;
  }
}
```

Or:

```ts
interface Clock {
  now(): Date;
}
```

The responsibility becomes explicit:

```text
Subscription
    → determine expired state

Clock
    → provide current time
```

---

# 88. Responsibility and Randomness

Randomness should also be isolated.

Bad:

```ts
class OrderFactory {
  create() {
    const id = Math.random().toString(36);
    ...
  }
}
```

More explicit:

```ts
interface IdGenerator {
  generate(): string;
}
```

Now:

```text
OrderFactory
    → construct order

IdGenerator
    → generate identity
```

This improves testing and replacement.

---

# 89. Responsibility and Logging

Avoid:

```ts
class Order {
  confirm() {
    console.log("Order confirmed");
    ...
  }
}
```

unless direct console output is genuinely the domain requirement.

Better:

```text
Order
    → confirm
Application/infrastructure
    → logging/telemetry
```

This avoids coupling domain behavior to environment-specific mechanisms.

---

# 90. Responsibility and Error Ownership

Ask:

> Which layer should decide that the error exists?

Example:

```text
Order.confirm()
```

can decide:

```text
invalid state transition
```

But:

```text
database unavailable
```

belongs to infrastructure.

Application code can translate technical failures into application-level outcomes without pretending the domain owns the failure.

---

# 91. Responsibility and Validation

Validation should be placed according to meaning.

### Domain invariant

```ts
quantity > 0
```

belongs close to the domain concept.

### Request shape

```ts
quantity is a number
```

may be validated at the boundary.

### Database constraint

```text
unique order number
```

may be enforced by both application/domain policy and the database.

Validation is layered.

Responsibility is not necessarily singular.

---

# 92. Responsibility and Duplication

Duplicated rules are a responsibility warning.

Example:

```ts
OrderService.confirm()
InvoiceService.confirmOrder()
Controller.confirmOrder()
```

all contain:

```text
if status !== "PENDING"
```

The rule has no clear owner.

Move it toward the state owner.

---

# 93. Responsibility and API Design

A public method should represent a meaningful responsibility.

Prefer:

```ts
order.cancel()
```

over:

```ts
order.setStatus("CANCELLED")
```

when cancellation has domain rules.

The first is a semantic command.

The second exposes representation.

---

# 94. Responsibility and Method Naming

Good names expose responsibility.

Prefer:

```text
reserveStock()
approve()
cancel()
addLine()
calculateTotal()
authorize()
charge()
publish()
```

over generic names:

```text
process()
handle()
manage()
executeEverything()
doStuff()
```

Generic naming often hides unresolved responsibility.

Note:

```text
execute()
```

can still be valid for an application-use-case interface where the abstraction is explicitly "execute this use case."

---

# 95. Responsibility and Public Surface Area

A coherent responsibility often implies a smaller public API.

Bad:

```ts
class Order {
  items: OrderLine[];
  status: string;
  total: number;
  discount: number;
}
```

Better:

```ts
class Order {
  addLine(line: OrderLine) {}
  removeLine(lineId: string) {}
  total(): Money {}
  confirm() {}
  cancel() {}
}
```

The API communicates what the object is responsible for.

---

# 96. Responsibility and Representation Independence

A strong object can change internal representation without breaking clients.

Today:

```ts
#items: OrderLine[]
```

Tomorrow:

```ts
#items: Map<string, OrderLine>
```

Clients still use:

```ts
order.addLine(...)
order.removeLine(...)
order.total()
```

This is a direct payoff of responsibility-driven encapsulation.

---

# 97. Responsibility and Performance

Responsibility placement can affect performance.

For example:

```ts
order.total()
```

may recalculate every line each time.

Options include:

```text
recalculate
cache
incremental total
materialized value
```

Do not move total calculation into a service merely for performance.

Instead ask:

```text
Who owns the semantic responsibility?
What implementation best satisfies the performance constraint?
```

Responsibility and optimization are separate questions.

---

# 98. Responsibility and Memory

Per-instance methods created through class fields:

```ts
class Cart {
  total = () => {};
}
```

may allocate a function per instance.

Prototype method:

```ts
class Cart {
  total() {}
}
```

typically uses a shared prototype method.

Responsibility is still:

```text
Cart → calculate total
```

while the implementation choice affects memory.

---

# 99. Responsibility and Security Boundaries

A responsibility should not implicitly grant unnecessary authority.

Example:

```ts
class OrderService {
  constructor(private db: Database) {}
}
```

If the service only needs a repository, giving it the whole database grants more authority than necessary.

Prefer:

```ts
constructor(
  private readonly orders: OrderRepository
) {}
```

This is least authority applied to object design.

---

# 100. Responsibility and Object Capability

A useful mental model:

```text
Reference
    = authority
```

If an object receives:

```ts
Database
```

it may gain broad power.

If it receives:

```ts
OrderRepository
```

its authority is narrower.

Responsibility boundaries and security boundaries often reinforce each other.

---

# 101. A Principal-Level Responsibility Decision

Suppose both `Order` and `PricingService` could calculate a total.

Do not answer:

> "Information Expert says Order."

Ask:

```text
What information is required?
What rules vary?
Who owns the invariant?
Who changes more often?
How reusable is this behavior?
Would Order become a policy god object?
Will callers need the same policy?
What testing boundary is preferable?
What dependency direction is desired?
```

Then decide.

This is principal-level responsibility assignment.

---

# 102. Responsibility Is a Design Decision, Not a Discovery

There may be several valid designs.

For example:

```text
TaxPolicy.calculate(order)
```

and:

```text
order.calculateTax(taxPolicy)
```

can both be defensible.

The correct question is:

> Which design produces the best ownership, coupling, cohesion, variation protection, and evolution characteristics for this domain?

LLD is not a puzzle where every responsibility has exactly one mathematically correct owner.

---

# 103. A Responsibility Scorecard

For a candidate responsibility assignment, score:

| Dimension | Question |
|---|---|
| Knowledge | Does it have the necessary information? |
| Authority | Does it own the affected state? |
| Invariant | Can it protect the rule? |
| Cohesion | Does the responsibility fit naturally here? |
| Coupling | What new dependencies are introduced? |
| Variation | Is likely change isolated? |
| Reuse | Will reuse be improved or harmed? |
| Testability | Can it be tested independently? |
| Security | Does it receive unnecessary authority? |
| Performance | Does placement create expensive interactions? |
| Evolution | Will future changes remain localized? |
| Communication | Does the API express the concept clearly? |

This scorecard is more reliable than using slogans mechanically.

---

# 104. Responsibility-Driven Design Algorithm

Use this algorithm during LLD.

```text
INPUT:
    business scenario

1. Identify business concepts.
2. Identify state.
3. Identify invariants.
4. Identify events/state transitions.
5. Identify system operations.
6. List important responsibilities.
7. Identify information experts.
8. Assign state ownership.
9. Assign invariant protection.
10. Assign domain behavior.
11. Identify coordination responsibilities.
12. Identify creation responsibilities.
13. Identify persistence responsibilities.
14. Identify external integration responsibilities.
15. Identify variation points.
16. Introduce abstractions only where justified.
17. Draw collaboration flow.
18. Check cohesion.
19. Check coupling.
20. Check change scenarios.
21. Check testability.
22. Check security boundaries.
23. Implement.
24. Refactor using real scenarios.
25. Re-evaluate.
```

---

# 105. Implementation Progression

## Stage 1 — Guided

Given a responsibility table, implement classes.

## Stage 2 — Partially Guided

Given only a use case and invariants, identify owners.

## Stage 3 — No Reference

Given a business scenario, design the responsibility model from scratch.

## Stage 4 — Edge-Case Hardened

Add:

- invalid transitions
- missing collaborators
- duplicate data
- concurrency risks
- authorization
- external failures

## Stage 5 — Production Grade

Add:

- logging
- metrics
- tracing
- persistence boundaries
- transaction handling
- retry behavior
- security controls
- migration strategy
- observability

---

# 106. Track A — Core Theory Retrieval

Without notes, explain:

```text
1. What is a responsibility?
2. What is the difference between knowing and doing responsibilities?
3. Explain Information Expert.
4. Explain when not to use Information Expert mechanically.
5. Explain Creator and Controller.
6. Explain Pure Fabrication and Indirection.
7. Explain Polymorphism and Protected Variations.
8. Why do Low Coupling and High Cohesion matter?
9. What is the difference between domain behavior and coordination?
10. How do invariants influence responsibility?
```

---

# 107. Track B — Implementation Exercise

Implement:

```ts
class ShoppingCart {
  addItem(productId: string, quantity: number): void;
  removeItem(productId: string): void;
  total(): Money;
  checkout(): Order;
}
```

Constraints:

- positive quantities
- no duplicate representation of the same product unless explicitly designed
- cart owns item lifecycle
- order creation should have clear responsibility
- money calculations should be testable
- no direct external API calls from `ShoppingCart`

Then implement:

```text
CheckoutUseCase
Inventory
PaymentGateway
OrderRepository
```

with explicit responsibility boundaries.

---

# 108. Track B — No-Reference Challenge

Design the responsibility model for:

```text
A customer buys jewellery from a branch.
The item may be sold by weight or by piece.
A discount policy may apply.
Tax varies by jurisdiction.
Stock must be reserved.
Payment may be cash/card/UPI.
The sale must be auditable.
```

Do not begin by writing classes.

First produce:

```text
responsibility matrix
CRC cards
message flow
invariants
variation points
```

Only then code.

---

# 109. Track C — Interview Reasoning

An interviewer asks:

> "Where would you calculate the order total?"

Weak:

> "Inside OrderService."

Better:

> "I would first ask what information the calculation requires. If the total is a straightforward aggregation of order lines, Order is the information expert because it owns the lines. If pricing requires variable policies such as promotions, tax, or external pricing rules, I would keep the order's core responsibility small and collaborate with explicit pricing policies."

This answer demonstrates judgment.

---

# 110. Interview Question Bank

### Q1
What is responsibility-driven design?

### Q2
What is the difference between knowing and doing responsibilities?

### Q3
Explain Information Expert.

### Q4
Is Information Expert always the correct answer?

### Q5
What is Creator?

### Q6
What is Controller?

### Q7
How is an application service different from a domain object?

### Q8
What is Pure Fabrication?

### Q9
What is Indirection?

### Q10
How does polymorphism help responsibility assignment?

### Q11
What is Protected Variations?

### Q12
How do cohesion and coupling influence responsibility placement?

### Q13
What is an anemic domain model?

### Q14
When is an anemic model acceptable?

### Q15
When should behavior be moved from a service into an entity?

### Q16
When should behavior remain in a service?

### Q17
What is Tell, Don't Ask?

### Q18
Why is Law of Demeter useful?

### Q19
Can application services contain business logic?

### Q20
What is a god object?

### Q21
How do you refactor a god object?

### Q22
How do tests reveal misplaced responsibility?

### Q23
How would you assign responsibility in a checkout flow?

### Q24
How would you protect a payment provider integration from change?

### Q25
How does responsibility relate to invariants?

---

# 111. Predict-the-Design Exercises

For each scenario, predict the best primary responsibility owner.

## Exercise 1

```text
Validate that an OrderLine quantity is positive.
```

Answer target:

```text
OrderLine
```

---

## Exercise 2

```text
Load an order from PostgreSQL.
```

Answer target:

```text
OrderRepository
```

---

## Exercise 3

```text
Coordinate inventory reservation + payment + persistence.
```

Answer target:

```text
Application service / use case
```

---

## Exercise 4

```text
Select among multiple shipping algorithms.
```

Answer target:

```text
Shipping policy / polymorphic strategy
```

---

## Exercise 5

```text
Protect "PAID cannot become CANCELLED".
```

Answer target:

```text
Order / Invoice state owner
```

---

# 112. Design Review Exercise

Review:

```ts
class OrderController {
  async confirm(req) {
    const order = await db.orders.find(req.params.id);

    if (order.status === "PENDING") {
      order.status = "CONFIRMED";
    }

    const total = order.lines.reduce(
      (sum, line) =>
        sum + line.price * line.quantity,
      0
    );

    await stripe.charge(total);

    await db.orders.save(order);

    return order;
  }
}
```

Identify at least:

```text
transport responsibility
persistence responsibility
domain responsibility
payment responsibility
calculation responsibility
workflow responsibility
```

Then redesign the collaboration model.

---

# 113. Mastery Exercise — Full Scenario

Design a library borrowing system.

Requirements:

```text
Member can borrow books.
A book copy can be unavailable.
Borrow duration depends on membership type.
Late fees are calculated by policy.
A member cannot exceed borrowing limit.
The system must record borrowing history.
Notifications may be sent.
```

Produce:

```text
1. entities
2. value objects
3. responsibilities
4. collaborators
5. invariants
6. CRC cards
7. responsibility matrix
8. message flow
9. application service
10. repository boundaries
11. variation points
12. TypeScript implementation
13. unit tests
14. refactoring discussion
```

---

# 114. Mastery Exercise — Enterprise Jewellery ERP

Model:

```text
Branch
Tenant
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
Create a sale in Branch A
for Tenant T1
with jewellery item J1
priced by current pricing policy,
apply eligible discount,
calculate tax,
reserve inventory,
accept payment,
finalize invoice,
record audit.
```

Your responsibility analysis must explicitly answer:

```text
Who owns tenant context?
Who validates branch validity?
Who owns sale state?
Who owns line quantity?
Who owns pricing decision?
Who owns discount decision?
Who owns tax decision?
Who owns stock reservation?
Who owns payment integration?
Who owns invoice persistence?
Who owns audit?
Who coordinates the workflow?
```

Do not allow "SaleService" to become a default answer.

---

# 115. Debugging Responsibility Problems

When behavior is wrong, inspect ownership first.

### Symptom

Two services calculate tax differently.

Likely issue:

```text
No clear tax responsibility.
```

### Symptom

Any caller can assign status.

Likely issue:

```text
State ownership is broken.
```

### Symptom

Changing payment provider requires changes in domain classes.

Likely issue:

```text
Missing protected variation / indirection.
```

### Symptom

A controller contains 300 lines of business rules.

Likely issue:

```text
Application/domain responsibility collapsed into boundary code.
```

### Symptom

Every domain method requires a database connection.

Likely issue:

```text
Domain and persistence responsibilities are coupled.
```

---

# 116. Responsibility Debugging Procedure

```text
1. Reproduce the scenario.
2. Identify the incorrect decision.
3. Identify the state involved.
4. Find the actual decision-maker.
5. Ask whether it owns the relevant knowledge.
6. Ask whether it owns the invariant.
7. Identify duplicate decisions.
8. Find the missing boundary.
9. Refactor ownership.
10. Add a regression test.
```

---

# 117. Common Misconceptions

## Misconception 1

> Every behavior must live inside a class.

False.

Functions are valid responsibility holders.

---

## Misconception 2

> Every business operation belongs to a service.

False.

Entities and value objects often own meaningful behavior.

---

## Misconception 3

> Information Expert means the largest data structure gets every behavior.

False.

Cohesion and variation matter.

---

## Misconception 4

> Tell, Don't Ask means getters are forbidden.

False.

It targets leaked decisions and invariants, not all reads.

---

## Misconception 5

> Controller means business logic belongs in controllers.

False.

Controller usually represents a system-operation boundary.

---

## Misconception 6

> More classes mean better responsibility design.

False.

Over-objectification creates ceremony and indirection without value.

---

## Misconception 7

> Dependency injection solves responsibility problems.

False.

Injection only supplies dependencies; it does not establish coherent ownership.

---

## Misconception 8

> A domain service is a place for any logic that feels complicated.

False.

It should represent a meaningful domain responsibility with no better natural owner.

---

# 118. Common Mistakes

### Mistake 1 — Starting with database tables

This produces persistence-driven objects.

Better:

```text
scenario → responsibilities → collaboration → persistence
```

---

### Mistake 2 — Starting with nouns only

A noun list gives entities but not behavior.

Always identify:

```text
state
behavior
invariants
messages
```

---

### Mistake 3 — Using `Service` as a universal fallback

This hides design uncertainty.

Ask:

```text
Who owns this?
```

---

### Mistake 4 — Moving everything into entities

This creates god entities.

Entities should remain cohesive.

---

### Mistake 5 — Making controllers "thin" by moving logic into one giant service

That only relocates the god object.

---

### Mistake 6 — Ignoring change boundaries

A responsibility may be technically correct but operationally painful if volatile concerns are coupled.

---

### Mistake 7 — Ignoring authorization

Responsibility allocation should also consider who is allowed to invoke behavior.

---

# 119. Comparison — Responsibility Roles

| Role | Primary Responsibility |
|---|---|
| Entity | identity + domain state + behavior |
| Value Object | value invariants + value behavior |
| Aggregate | consistency boundary |
| Domain Service | domain operation spanning multiple concepts |
| Application Service | use-case coordination |
| Controller | external/system operation boundary |
| Factory | controlled complex creation |
| Repository | persistence boundary |
| Adapter | external-system translation |
| Policy/Strategy | variable decision algorithm |
| Infrastructure Service | technical capability |

These labels are guides, not mandatory class names.

---

# 120. Responsibility vs Architecture

Responsibility-driven design operates at several levels.

```text
Method responsibility
    ↓
Object responsibility
    ↓
Component responsibility
    ↓
Application responsibility
    ↓
System responsibility
```

A poor responsibility assignment at a low level can force complexity upward.

Example:

```text
Order
```

does too much.

Then:

```text
OrderService
```

does too much.

Then:

```text
Application
```

becomes tightly coupled.

Good local boundaries compound into better architecture.

---

# 121. Responsibility Graph

Think of the system as a graph.

```text
      ┌───────────────┐
      │ Controller    │
      └───────┬───────┘
              │
              v
      ┌───────────────┐
      │ Use Case      │
      └──┬─────┬───┬──┘
         │     │   │
         v     v   v
      Order  Stock Payment
         │
         v
       Lines
```

A responsibility graph should reveal:

```text
who asks
who decides
who changes
who coordinates
who persists
```

---

# 122. Responsibility and Dependency Graphs

A strong design tends to avoid unnecessary dependency fan-out.

Instead of:

```text
Order → DB
Order → Stripe
Order → Email
Order → Tax API
Order → Redis
```

prefer:

```text
Use Case
 ├── Order
 ├── Inventory
 ├── PaymentGateway
 ├── TaxPolicy
 └── NotificationPort
```

with infrastructure implementing ports.

The object graph becomes more meaningful.

---

# 123. Responsibility and Failure Boundaries

Different responsibilities may fail differently.

```text
Order.confirm()
    → domain validation failure

PaymentGateway.charge()
    → external failure

OrderRepository.save()
    → persistence failure

EmailSender.send()
    → notification failure
```

Keeping responsibilities distinct helps determine:

- retryability
- transactionality
- user-visible errors
- compensating actions
- observability

---

# 124. Responsibility and Reliability

When assigning responsibility, ask:

```text
What happens if this collaborator fails?
```

A payment gateway failure should not make the order object responsible for:

```text
HTTP retries
queue backoff
circuit breakers
```

Those are technical reliability responsibilities.

The order may still own:

```text
payment state transition rules
```

---

# 125. Responsibility and Distributed Systems

In distributed systems, one responsibility can cross process boundaries.

Example:

```text
Order
    → requests payment

Payment Service
    → owns payment state
```

Do not duplicate payment authority in both services.

Define:

```text
bounded responsibility
+
message contract
+
ownership
```

This will become central in later distributed-object chapters.

---

# 126. Responsibility and Eventual Consistency

Suppose:

```text
Order
```

is immediately confirmed but:

```text
SearchIndex
```

updates asynchronously.

Then:

```text
Order
    → owns authoritative order state

SearchIndexer
    → owns search representation
```

The index should not become the source of truth for the domain.

---

# 127. Responsibility and Caches

A cache may be responsible for:

```text
temporary representation / lookup optimization
```

not for core business truth.

Bad:

```text
Order.status = cache.get(...)
```

as authoritative state without a defined consistency model.

Responsibility analysis helps distinguish:

```text
source of truth
```

from:

```text
derived optimization
```

---

# 128. Responsibility and Lifecycle

Ask:

```text
Who creates it?
Who owns it?
Who uses it?
Who disposes it?
```

Lifecycle responsibility matters for:

- connections
- subscriptions
- transactions
- workers
- locks
- caches
- resources

This becomes increasingly important as object design moves toward infrastructure and concurrency.

---

# 129. Responsibility and Resource Ownership

Example:

```ts
class DatabaseUnitOfWork {
  async run<T>(work: () => Promise<T>): Promise<T> {
    // transaction lifecycle
  }
}
```

The unit of work owns transaction lifecycle.

The domain object should not own database connection lifecycle.

---

# 130. Responsibility and Naming Boundaries

A useful technique:

> If a class name needs "Manager" to sound meaningful, inspect the responsibilities again.

Prefer names expressing concepts:

```text
Order
PricingPolicy
CheckoutUseCase
PaymentGateway
Inventory
AuditLog
```

Names should emerge from responsibility modeling.

---

# 131. Responsibility and Module Boundaries

Classes are not the only unit of responsibility.

Modules can own responsibilities too.

```text
pricing/
  policy.ts
  money.ts
  tax.ts

ordering/
  order.ts
  order-line.ts
  checkout.ts

payments/
  gateway.ts
  stripe-adapter.ts
```

A module boundary can prevent accidental coupling between responsibility groups.

---

# 132. Responsibility and Public Exports

Do not export internal implementation unnecessarily.

Prefer:

```ts
export { Order } from "./order";
export type { OrderRepository } from "./ports";
```

instead of exporting every helper.

Public exports define responsibility boundaries at the module level.

---

# 133. Responsibility and API Contracts

An interface should express a capability.

Good:

```ts
interface Inventory {
  reserve(items: ReservationRequest[]): Promise<void>;
}
```

Less useful:

```ts
interface InventoryManager {
  getDatabase(): Database;
  getProducts(): Product[];
  updateStock(...): void;
  ...
}
```

Capability-based interfaces keep responsibility focused.

---

# 134. Responsibility and Interface Segregation

If a consumer needs only:

```ts
charge()
```

do not inject:

```ts
FullPaymentSystem
```

Prefer:

```ts
interface PaymentGateway {
  charge(input: ChargeInput): Promise<ChargeResult>;
}
```

This aligns interface scope with responsibility scope.

---

# 135. Responsibility and Stable Abstractions

A responsibility boundary is more valuable when the concept is stable.

Do not create interfaces solely for:

```text
every class
```

Create them around:

```text
meaningful capabilities
variation points
architectural boundaries
test seams
```

---

# 136. Responsibility-Driven Design and SOLID

Responsibility assignment connects directly to SOLID.

### SRP

One coherent responsibility cluster.

### OCP

Protect variation behind stable boundaries.

### LSP

Subtypes must preserve responsibility contracts.

### ISP

Interfaces represent focused responsibilities.

### DIP

High-level responsibilities depend on stable abstractions rather than volatile details.

SOLID becomes easier to understand when treated as consequences of good responsibility allocation rather than isolated rules.

---

# 137. Responsibility-Driven Design and GRASP

This chapter introduces the conceptual foundation.

Chapter 14 formalizes the GRASP family:

```text
Information Expert
Creator
Controller
Low Coupling
High Cohesion
Polymorphism
Pure Fabrication
Indirection
Protected Variations
```

Think of this chapter as:

```text
Why responsibility assignment matters
```

and Chapter 14 as:

```text
A more systematic vocabulary and set of heuristics
```

---

# 138. Principal Design Rule

A powerful general rule is:

> **Place a responsibility at the narrowest boundary that has enough knowledge and authority to perform it correctly without creating unhealthy coupling.**

This balances:

```text
knowledge
authority
cohesion
coupling
variation
security
testability
evolution
```

---

# 139. Decision Tree

Use this during design.

```text
Does the behavior protect state?
        |
       yes
        ↓
Does the object own that state?
        |
       yes
        ↓
Consider placing behavior there.

       no
        ↓
Does one object clearly have the required information?
        |
       yes
        ↓
Information Expert candidate.

       no
        ↓
Does behavior represent a stable use-case coordination?
        |
       yes
        ↓
Application Service / Controller candidate.

       no
        ↓
Does it span domain concepts without a natural owner?
        |
       yes
        ↓
Domain Service candidate.

       no
        ↓
Is the main issue external variation?
        |
       yes
        ↓
Port / Adapter / Policy candidate.

       no
        ↓
Re-examine the model.
```

---

# 140. Production Checklist

Before shipping a design, verify:

```text
[ ] State ownership is explicit.
[ ] Domain invariants have owners.
[ ] Business rules are not duplicated.
[ ] Controllers are boundary-focused.
[ ] Application services coordinate rather than monopolize business logic.
[ ] Entities own meaningful state behavior where appropriate.
[ ] Value objects own their value invariants.
[ ] Domain services have a justified reason to exist.
[ ] Repositories own persistence concerns.
[ ] Factories own genuinely complex construction.
[ ] External integrations are isolated.
[ ] Variation points are protected.
[ ] Interfaces express capabilities.
[ ] Collaborators have least-authority access.
[ ] Tests map naturally to responsibilities.
[ ] High fan-out objects have been reviewed.
[ ] Change scenarios have been considered.
```

---

# 141. Predict-the-Output / Runtime Exercise

Responsibility design is not separate from JavaScript runtime knowledge.

Predict:

```ts
class Counter {
  #value = 0;

  increment() {
    this.#value++;
    return this;
  }

  get value() {
    return this.#value;
  }
}

const counter = new Counter();

const increment = counter.increment;

increment();
```

Question:

```text
Does this work?

What does it reveal about:
- method ownership?
- receiver binding?
- responsibility vs invocation?
```

Expected concept:

```text
The method responsibility belongs to Counter,
but extracting the method loses the intended receiver.
```

The runtime model from earlier chapters still matters.

---

# 142. JavaScript `this` and Responsibility

A method can be conceptually owned by an object while its function value is callable elsewhere.

```ts
const fn = object.doSomething;
fn();
```

The language does not guarantee:

```text
"function extracted from object automatically retains object"
```

Therefore responsibility design and invocation design are related.

Possible solutions:

```ts
const fn = object.doSomething.bind(object);
```

or a closure/arrow function where appropriate.

Do not confuse:

```text
ownership of a responsibility
```

with:

```text
automatic binding of the method's receiver
```

---

# 143. Private Fields and Responsibility

Private fields reinforce ownership.

```ts
class Invoice {
  #status = "DRAFT";

  approve() {
    this.#status = "APPROVED";
  }
}
```

External callers cannot directly replace:

```text
#status
```

This gives runtime support to responsibility boundaries.

The language mechanism does not automatically create good responsibility allocation, but it can enforce a good allocation.

---

# 144. Proxies and Responsibility Boundaries

A `Proxy` can intercept operations.

```ts
const protectedObject = new Proxy(target, {
  set() {
    throw new Error("Mutation blocked");
  }
});
```

This can enforce cross-cutting boundaries.

But do not replace clear domain APIs with generic proxy magic when explicit responsibility methods are easier to understand.

Good:

```ts
order.confirm()
```

is usually more communicative than:

```text
proxy.set("status", "CONFIRMED")
```

---

# 145. Responsibility and Metaprogramming

Decorators, proxies, reflection, and dependency containers can automate wiring.

They should not hide core business ownership.

A useful rule:

```text
Infrastructure may automate collaboration.
Domain behavior should remain understandable.
```

---

# 146. Debugging with Responsibility Traces

When investigating a bug, write:

```text
Input
  ↓
Message
  ↓
Responsible object
  ↓
Decision
  ↓
State change
  ↓
Collaborator
```

Example:

```text
confirm request
  ↓
ConfirmOrderUseCase.execute
  ↓
Order.confirm
  ↓
status transition
  ↓
OrderRepository.save
```

If the trace instead looks like:

```text
Controller
 → DB object
 → raw row
 → helper
 → service
 → another service
 → mutate object
```

responsibility boundaries may be unclear.

---

# 147. Responsibility and Code Review

A strong code review question is:

> Why does this object own this behavior?

Not:

> "Can we put this in a utility?"

Ask the author to explain:

```text
knowledge
authority
invariant
cohesion
variation
coupling
```

A short explanation can reveal a weak design immediately.

---

# 148. Code Review Exercise

Review:

```ts
class ProductService {
  calculateSellingPrice(product: Product) {
    return (
      product.basePrice -
      product.discount +
      product.tax
    );
  }

  markOutOfStock(product: Product) {
    product.quantity = 0;
  }
}
```

Questions:

```text
1. Who owns quantity?
2. Who owns stock state transitions?
3. Are discount and tax stable product properties or policy decisions?
4. Is selling price a product responsibility, pricing responsibility, or both?
5. What variation points exist?
6. What invariant is being bypassed?
```

Do not refactor mechanically. Defend the responsibility placement.

---

# 149. Design Exercise — Three Alternatives

Given:

```ts
total(order)
```

Compare:

### Option A

```ts
order.total()
```

### Option B

```ts
pricingService.total(order)
```

### Option C

```ts
pricingService.calculate(order, pricingPolicy)
```

Evaluate each using:

```text
knowledge
cohesion
variation
reuse
complexity
testability
future change
```

The answer depends on domain complexity.

---

# 150. Design Exercise — Controller or Domain?

Scenario:

```text
User submits "cancel order".
```

Possible implementation:

```ts
controller.cancel()
```

or:

```ts
order.cancel()
```

or:

```ts
cancelOrderUseCase.execute()
```

Strong architecture may use all three, because they own different responsibilities:

```text
Controller
    receives

Use Case
    coordinates

Order
    decides whether cancellation is valid
```

This is a key lesson:

> One user action can cross multiple responsibility boundaries.

---

# 151. Responsibility Layering

A single operation often has layered responsibilities:

```text
HTTP layer
    parse/authorize/map

Application layer
    coordinate

Domain layer
    decide/enforce

Persistence layer
    store/retrieve

Infrastructure layer
    integrate
```

Do not force one class to own the entire operation.

---

# 152. Responsibility vs Business Rule Location

A business rule may involve several layers.

Example:

```text
"Only managers can approve discounts above 20%."
```

Possible responsibilities:

```text
Authorization policy
    → can actor approve?

Discount policy
    → is 20% allowed?

Order/Sale
    → maintain approved state
```

One sentence in requirements may map to multiple responsibilities.

---

# 153. Responsibility Decomposition Technique

Take a requirement:

> "Finalize the sale after stock and payment succeed."

Break it into verbs:

```text
finalize
verify stock
reserve stock
charge payment
persist result
record audit
```

Then ask:

```text
Who should do each verb?
```

This transforms requirements into responsibility candidates.

---

# 154. Noun–Verb Trap

Do not automatically produce:

```text
SaleManager
StockManager
PaymentManager
AuditManager
```

because the requirement contains these nouns.

Instead identify:

```text
Sale
Inventory
PaymentGateway
AuditLog
```

and their interactions.

---

# 155. Responsibility Discovery from Invariants

Requirements often reveal responsibilities through "must" statements.

Examples:

```text
must not exceed limit
must have positive quantity
must not cancel paid invoice
must belong to tenant
must use valid branch
```

Turn each into:

```text
invariant
→ owner
→ operation
```

Example:

```text
"Paid invoice cannot be cancelled."

Invoice
  owns status

Invoice.cancel()
  enforces transition
```

---

# 156. Responsibility Discovery from Events

Events also reveal responsibility.

```text
OrderConfirmed
PaymentCaptured
StockReserved
InvoiceIssued
```

Ask:

```text
Who is responsible for causing this event?
Who owns the state transition that makes it true?
```

Usually the event should follow from an authoritative state change rather than being emitted by an unrelated observer.

---

# 157. Responsibility Discovery from Data Ownership

Inspect data:

```text
customerId
orderId
quantity
price
status
```

Ask:

```text
Who should have authority over it?
```

Data ownership is not merely database ownership.

A PostgreSQL table may contain columns for many concepts while domain ownership remains distinct.

---

# 158. Responsibility Discovery from Failure

Ask:

> Who should decide what happens when this operation fails?

Example:

```text
Payment rejected
```

Potential responsibilities:

```text
PaymentGateway
    reports failure

Order
    remains un-paid

CheckoutUseCase
    determines workflow result
```

Again, multiple layers participate.

---

# 159. Responsibility and State Machines

State machines make responsibility clearer.

```text
DRAFT
  ↓ approve
APPROVED
  ↓ pay
PAID
  ↓ cancel? no
```

The state owner should usually enforce valid transitions.

```ts
class Invoice {
  approve() {}
  pay() {}
  cancel() {}
}
```

This is preferable to exposing arbitrary transition:

```ts
invoice.status = "PAID";
```

---

# 160. Responsibility and Temporal Rules

Temporal rules often belong near the relevant concept.

```text
Subscription
    isActiveAt(date)

Coupon
    isValidAt(date)
```

Instead of:

```ts
DateUtils.isSubscriptionActive(subscription, date)
```

unless the temporal logic is actually a shared policy independent of the concept.

---

# 161. Responsibility and Aggregate Boundaries

If several objects must remain consistent together:

```text
Order
 ├── OrderLine
 ├── OrderAddress
 └── OrderTotals
```

the aggregate root may coordinate changes across them.

But avoid making the aggregate root implement every leaf behavior.

Instead:

```text
Order
   → controls aggregate invariants

OrderLine
   → owns line behavior

Address
   → owns address invariants
```

This creates layered responsibility.

---

# 162. Responsibility and Encapsulation

Encapsulation is a mechanism.

Responsibility assignment is a design decision.

Example:

```ts
class Order {
  #status = "PENDING";
}
```

Good encapsulation does not answer:

```text
Who should decide when it changes?
```

Responsibility design does.

Therefore:

```text
Encapsulation
    protects responsibility boundaries.

Responsibility design
    chooses those boundaries.
```

---

# 163. Responsibility and Abstraction

Abstraction defines what an object promises.

Responsibility defines what obligation that promise represents.

Example:

```ts
interface PaymentGateway {
  charge(input: ChargeRequest): Promise<ChargeResult>;
}
```

Abstraction:

```text
PaymentGateway capability
```

Responsibility:

```text
perform payment charge operation
```

The two concepts reinforce one another.

---

# 164. Responsibility and Subtyping

If two implementations claim the same responsibility:

```ts
interface DiscountPolicy {
  calculate(order: Order): Money;
}
```

they must satisfy the same behavioral contract.

Otherwise polymorphism becomes unsafe.

Responsibility assignment therefore connects directly to LSP.

---

# 165. Responsibility and Contracts

For each important responsibility, specify:

```text
Preconditions
Postconditions
Invariants
Failure modes
```

Example:

```text
Order.confirm()

Pre:
    order is PENDING

Post:
    order becomes CONFIRMED

Invariant:
    order has at least one line

Failure:
    invalid transition
```

Contracts make responsibilities concrete.

---

# 166. Responsibility and Semantic APIs

A semantic API describes the domain intention.

Prefer:

```ts
invoice.approve()
invoice.pay()
invoice.cancel()
```

over:

```ts
invoice.setStatus(...)
```

The semantic method makes responsibility observable in code.

---

# 167. Responsibility and Documentation

Class documentation should state responsibility, not implementation trivia.

Good:

```ts
/**
 * Maintains sale lifecycle invariants and computes
 * sale totals from owned sale lines.
 */
class Sale {}
```

Less useful:

```ts
/**
 * Contains an array and several methods.
 */
```

Documentation should reinforce ownership.

---

# 168. Responsibility and Team Design

Responsibility boundaries also help teams.

A coherent module can be owned by a team.

```text
Ordering team
    → ordering responsibilities

Payments team
    → payment responsibilities
```

Poorly assigned responsibilities create cross-team changes.

This is architectural cohesion at an organizational level.

---

# 169. Responsibility and Operational Ownership

Production systems need clear operational ownership too.

Ask:

```text
Who owns retries?
Who owns audit?
Who owns reconciliation?
Who owns cache invalidation?
Who owns schema migration?
```

These are responsibilities even when they are not domain classes.

---

# 170. Responsibility and Reconciliation

Payment systems often require reconciliation.

The payment gateway may report:

```text
SUCCESS
```

while the order update fails.

Now an explicit reconciliation process may be needed.

```text
PaymentReconciliationJob
```

is a legitimate responsibility holder.

Do not push this into:

```text
Order
```

just because it is related to orders.

---

# 171. Responsibility and Jobs

Background jobs are system responsibilities.

Example:

```text
SendInvoiceEmailJob
ExpirePendingOrdersJob
ReconcilePaymentsJob
ReleaseExpiredReservationsJob
```

Each job should have a coherent operational purpose.

The job may invoke domain/application responsibilities rather than replacing them.

---

# 172. Responsibility and Queues

Queue responsibility:

```text
delivery/buffering/asynchronous transport
```

Business responsibility:

```text
the decision or state transition
```

Keep those concepts distinct.

---

# 173. Responsibility and Schedulers

A scheduler is responsible for:

```text
when to run
```

The task is responsible for:

```text
what to do
```

This distinction prevents scheduler classes from becoming business-rule containers.

---

# 174. Responsibility and Caching

Cache responsibility:

```text
speed up access
```

Domain object responsibility:

```text
correct business state
```

The cache should not silently become the authority unless the architecture explicitly defines it as such.

---

# 175. Responsibility and Read Models

A query/read model may intentionally duplicate data for performance.

This does not mean it owns the domain responsibility.

For example:

```text
OrderReadModel
```

may render:

```text
customer name
order total
payment status
```

while:

```text
Order
```

remains the authoritative behavior owner.

This distinction becomes important in CQRS-style designs.

---

# 176. Responsibility and Reporting

Reports often need cross-domain information.

Do not force:

```text
Order
```

to know:

```text
all reporting queries
```

A reporting component can own read-side composition.

This is another example of avoiding god entities.

---

# 177. Responsibility and Serialization

Serialization is usually boundary behavior.

Bad:

```ts
class Order {
  toJsonForEveryApiVersion() {}
  toCsv() {}
  toPdf() {}
}
```

Better:

```text
Order
    domain representation

OrderDtoMapper
    API mapping

InvoiceRenderer
    PDF

CsvExporter
    CSV
```

One domain concept can have multiple external representations.

---

# 178. Responsibility and Mapping

Mapping responsibilities include:

```text
HTTP DTO → command
database row → domain object
domain object → DTO
external API response → domain concept
```

Adapters/mappers prevent representation details from leaking.

---

# 179. Responsibility and ORM Active Record

Active Record can combine:

```text
domain
+
persistence
```

This can be appropriate for simple applications.

In complex domains, separate responsibilities may provide:

```text
better invariants
clearer boundaries
less infrastructure coupling
```

The design choice depends on complexity and goals.

---

# 180. Responsibility and CRUD Systems

For simple CRUD:

```text
Controller → Service → Repository
```

may be sufficient.

Do not over-engineer every system with rich entities and dozens of policies.

Responsibility-driven design includes knowing when simplicity is the correct responsibility allocation.

---

# 181. When Anemic Models Are Fine

An anemic model may be acceptable when:

- the system is mostly CRUD
- invariants are weak
- workflows are simple
- domain behavior is limited
- rapid delivery matters more than elaborate domain abstraction

The mistake is turning this into:

> "All applications should use anemic models."

Context matters.

---

# 182. When Rich Domain Models Earn Their Cost

Rich models become more valuable when:

- state transitions matter
- invariants are non-trivial
- business language is important
- multiple workflows manipulate the same concepts
- duplicate rules are appearing
- domain behavior needs strong encapsulation

---

# 183. Responsibility Refactoring Smell Table

| Smell | Responsibility Signal |
|---|---|
| God Object | Too many responsibility clusters |
| Feature Envy | Behavior likely near another object's data |
| Data Clumps | Missing concept/value object |
| Primitive Obsession | Missing domain boundary |
| Shotgun Surgery | Responsibility fragmented too broadly |
| Divergent Change | Unrelated responsibilities combined |
| Long Method | Hidden sub-responsibilities |
| Service Blob | Domain responsibility pushed outward |
| Controller Blob | Coordination and domain logic collapsed |
| Anemic Model | State and behavior disconnected |

---

# 184. Relationship with Cohesion

Chapter 12 established types of cohesion.

Responsibility-driven design seeks to increase **functional/communicational cohesion** where possible.

Example:

```text
Order
    line management
    total calculation
    state transitions
```

These relate strongly to order lifecycle.

Compare:

```text
Order
    sendEmail
    exportPdf
    calculateTax
    authenticateUser
```

Much weaker cohesion.

---

# 185. Relationship with Coupling

Responsibility should avoid unnecessary coupling.

A useful intuition:

```text
Good responsibility allocation
    → high internal cohesion
    → smaller unnecessary dependency surface
    → lower change amplification
```

But low coupling alone is not enough.

A highly fragmented system may have low pairwise coupling but terrible navigability and cohesion.

---

# 186. Responsibility and Fan-In/Fan-Out

A responsibility used by many components:

```text
high fan-in
```

may be a stable shared capability.

A responsibility depending on many volatile components:

```text
high fan-out
```

may require review.

Neither is automatically bad.

Use them as signals.

---

# 187. Responsibility and Dependency Distance

A caller should not need to understand several layers of object structure.

Good:

```ts
order.taxAmount()
```

Potentially fragile:

```ts
order.customer.address.country.taxProfile.rate
```

Responsibility can absorb knowledge about object relationships.

---

# 188. Responsibility and API Evolution

Suppose:

```ts
order.status
```

is public.

Later, status becomes:

```text
state machine
+
reason
+
timestamps
+
actor
```

If callers use semantic operations:

```ts
order.confirm()
order.cancel()
```

the internal representation can evolve.

This is a powerful form of future-proof responsibility assignment.

---

# 189. Responsibility and Migration

When migrating from:

```text
legacy service
```

to:

```text
domain behavior
```

 do it incrementally.

Example:

```text
LegacyOrderService
    ↓ delegates
Order.confirm()
```

Then migrate callers.

Responsibility-driven refactoring can be evolutionary rather than a rewrite.

---

# 190. Strangler-Style Responsibility Migration

A practical sequence:

```text
1. Identify one business rule.
2. Establish its natural owner.
3. Add a semantic method.
4. Make the legacy service delegate.
5. Move tests.
6. Remove duplicated logic.
7. Repeat.
```

This is safer than a big-bang redesign.

---

# 191. Responsibility and Backward Compatibility

When changing responsibility ownership, preserve external contracts when necessary.

Example:

```ts
legacyService.confirm(order)
```

can temporarily become:

```ts
legacyService.confirm(order) {
  return order.confirm();
}
```

Now ownership moved without immediate caller disruption.

---

# 192. Responsibility and Test Seams

Explicit responsibilities produce natural test seams.

```text
TaxPolicy
PaymentGateway
Inventory
Clock
IdGenerator
```

can be replaced with fakes.

This is one reason Pure Fabrication and Indirection are useful.

---

# 193. Responsibility and Mocking

Do not create interfaces solely to make mocking possible.

A better order of reasoning is:

```text
meaningful responsibility
    ↓
stable boundary
    ↓
test seam
```

not:

```text
Need mock
    ↓
create interface
```

---

# 194. Responsibility and Contract Testing

For an infrastructure port:

```ts
interface PaymentGateway {
  charge(input): Promise<ChargeResult>;
}
```

contract tests can verify that:

```text
StripePaymentGateway
FakePaymentGateway
OtherProvider
```

preserve the responsibility contract.

This connects responsibility design to subtype safety.

---

# 195. Responsibility and Idempotency

Idempotency can itself be a responsibility.

Example:

```text
PaymentCommandProcessor
    owns duplicate-command detection
```

while:

```text
Payment
    owns payment state transition
```

Do not automatically place all idempotency logic inside the domain entity.

Layer by responsibility.

---

# 196. Responsibility and Authorization

Authorization may be:

```text
Application responsibility
Domain policy
Infrastructure enforcement
```

depending on the rule.

Example:

```text
"Only a branch manager can approve discount > 20%."
```

The system may use:

```text
AuthorizationPolicy
```

while `Sale` protects:

```text
approved discount state is valid
```

---

# 197. Responsibility and Auditability

Audit is often a separate cross-cutting concern.

Instead of:

```ts
sale.confirm() {
  ...
  auditDb.insert(...);
}
```

consider:

```text
Sale
    → state transition

Application layer
    → publish/record audit event
```

This preserves domain focus while maintaining auditability.

---

# 198. Responsibility and Compliance

Compliance may require:

```text
immutable audit trail
approval workflow
segregation of duties
tenant isolation
retention
```

These requirements create responsibilities that should be modeled explicitly rather than hidden in generic helpers.

---

# 199. Responsibility and Segregation of Duties

Suppose:

```text
Creator cannot approve own high-value sale.
```

Now there are at least:

```text
Sale
    → sale state

Authorization policy
    → approval permission

Application workflow
    → ensure actor differs from creator
```

This is a useful example where a single requirement crosses responsibility boundaries.

---

# 200. Responsibility and Domain Language

Good responsibility names use domain language.

Prefer:

```text
reserveInventory()
approveDiscount()
finalizeSale()
reconcilePayment()
```

over:

```text
processData()
executeLogic()
handleRecord()
```

Ubiquitous language helps both design and communication.

---

# 201. Responsibility and Interview Communication

During an interview, describe responsibility in this form:

```text
"I would put X in Y because Y owns/knows Z.
I would keep A in B because A varies independently.
The use case would coordinate C because it spans multiple collaborators."
```

Example:

> "I would keep inventory availability enforcement inside Inventory because it owns stock state. The checkout use case would coordinate inventory reservation with payment because that is a multi-collaborator workflow. The payment provider would sit behind a port because the provider is a protected variation."

This is much stronger than naming patterns alone.

---

# 202. Principal-Level Trade-Offs

Sometimes moving behavior into an entity improves cohesion but increases coupling to a policy.

Sometimes extracting a service reduces coupling but creates an anemic domain model.

Sometimes a pure function is simpler than an object.

Sometimes a domain service is justified.

Therefore always ask:

```text
What problem does this placement solve?
What new problem does it create?
```

That is design judgment.

---

# 203. Responsibility "Goldilocks" Principle

A responsibility should be:

```text
not too broad
not too fragmented
not too infrastructure-heavy
not too abstract
not too volatile
not too weakly owned
```

The goal is a useful boundary.

---

# 204. Responsibility and Refactoring Safety

When moving a responsibility:

```text
Preserve behavior
Preserve invariants
Preserve contracts
Preserve observable semantics
```

Then improve ownership.

This means responsibility-driven refactoring is behavior-preserving unless intentionally changing the domain model.

---

# 205. Completion Criteria

Do not mark this chapter mastered merely because you read it.

Mark `[+] Completed` when you can:

```text
[ ] Explain responsibility in your own words.
[ ] Identify state ownership.
[ ] Identify invariant ownership.
[ ] Use Information Expert.
[ ] Explain when not to use Information Expert mechanically.
[ ] Distinguish domain behavior from coordination.
[ ] Explain Creator and Controller.
[ ] Use Pure Fabrication and Indirection deliberately.
[ ] Identify variation points.
[ ] Use polymorphism for behavioral variation.
[ ] Apply Protected Variations.
[ ] Build CRC cards.
[ ] Build responsibility matrices.
[ ] Trace a use case as messages.
[ ] Detect god objects and service blobs.
[ ] Refactor feature envy.
[ ] Decide entity vs value object vs domain service.
[ ] Decide application service vs domain object.
[ ] Model repository/factory responsibilities.
[ ] Reason about responsibility in JavaScript/TypeScript.
[ ] Defend your design in an interview.
[ ] Complete the jewellery ERP responsibility exercise.
```

---

# 206. Mastery Gate

Use the global mastery sequence:

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

You have mastered responsibility-driven object design only when you can take an unfamiliar business scenario and independently decide:

```text
what each object should know
what each object should do
what each object should protect
what each object should delegate
what each service should coordinate
what should remain outside the domain
```

---

# 207. Key Takeaways

1. Responsibility is an obligation assigned to an object or component.
2. Responsibilities can involve knowing, doing, and coordinating.
3. State ownership strongly influences behavior ownership.
4. Invariants should be protected near the state they constrain.
5. Information Expert is a powerful heuristic, not an absolute law.
6. Creator identifies natural construction ownership.
7. Controller handles system-operation boundaries rather than becoming a business-rule container.
8. Pure Fabrication creates a focused object when no natural domain owner exists.
9. Indirection reduces direct coupling.
10. Polymorphism distributes varying responsibilities by behavioral type.
11. Protected Variations isolate expected change.
12. Low Coupling and High Cohesion influence every responsibility decision.
13. Tell, Don't Ask helps keep decisions near the data and invariants they depend on.
14. Law of Demeter is about dependency on object structure, not banning every method chain.
15. Application services coordinate use cases; they should not automatically own all business logic.
16. Repositories own persistence concerns.
17. Factories own complex construction when justified.
18. Domain services are for domain behavior with no better natural owner.
19. CRC cards and responsibility matrices make design reasoning explicit.
20. Scenarios and interaction diagrams validate responsibility placement.
21. Services are not automatically good; service dumping grounds are a real design smell.
22. Rich domain models are useful when business invariants and behavior matter.
23. Anemic models can be appropriate for simple CRUD systems.
24. Good responsibility boundaries improve testing, security, maintainability, and evolution.
25. A principal engineer chooses responsibility placement by trade-off, not by slogans.

---

# 208. Concept Connections

```text
Chapter 1
Object Model
    ↓
Objects have state and behavior

Chapter 2
Identity / Mutability
    ↓
Ownership and controlled mutation

Chapter 3
Prototype Model
    ↓
Method and property lookup

Chapter 4
Construction
    ↓
Creation responsibility

Chapter 5–6
Classes / Initialization
    ↓
Object lifecycle responsibilities

Chapter 7
Encapsulation
    ↓
Protect responsibility-owned state

Chapter 8
Abstraction
    ↓
Stable responsibility contracts

Chapter 9
Subtyping
    ↓
Behavioral responsibility compatibility

Chapter 10
Polymorphism
    ↓
Variation-specific responsibility

Chapter 11
Composition
    ↓
Collaborative responsibility distribution

Chapter 12
Cohesion / Coupling
    ↓
Quality criteria for assignment

Chapter 13
Responsibility-Driven Object Design
    ↓
Who knows?
Who does?
Who coordinates?
Who owns?
Who protects?

Chapter 14
GRASP
    ↓
Formal responsibility-assignment heuristics
```

---

# 209. Retrieval / Dependency Graph

```text
Object Model
   ├── Identity
   ├── State
   ├── Behavior
   └── Prototype / Class mechanics
            ↓
Encapsulation
            ↓
Abstraction
            ↓
Composition / Polymorphism
            ↓
Cohesion + Coupling
            ↓
Responsibility Assignment
            ↓
GRASP
            ↓
SOLID / Design Patterns
            ↓
DDD / Persistence / Transactions
            ↓
Concurrency / Resilience / Security
            ↓
Enterprise LLD
            ↓
Interview Design Judgment
```

---

# 210. Revision / Retrieval Record

Use this section after study sessions.

## First Pass

```text
Date:
Status:

Can define responsibility:
[ ]

Can distinguish state/behavior/coordination:
[ ]

Can apply Information Expert:
[ ]

Can explain Creator/Controller:
[ ]

Can explain Pure Fabrication/Indirection:
[ ]

Can explain Polymorphism/Protected Variations:
[ ]

Can build CRC:
[ ]

Can create a responsibility matrix:
[ ]
```

## Retrieval Session

```text
Date:
Weakest concept:
________________________

Most confusing trade-off:
________________________

Scenario that exposed weakness:
________________________

Correction:
________________________
```

## Interview Retrieval

```text
Question:
________________________

My answer:
________________________

Missing reasoning:
________________________
```

---

# 211. Canonical References and Source Discipline

When extending this chapter, prioritize sources in this order:

## 1. ECMAScript Specification

Use the ECMAScript specification for language semantics relevant to:

- classes
- objects
- property access
- functions
- private elements
- modules
- language-level behavior

Canonical source:

**ECMA-262 — ECMAScript Language Specification**  
https://tc39.es/ecma262/

---

## 2. JavaScript Engine Documentation

For implementation-specific behavior, use engine documentation such as V8.

Important distinction:

```text
ECMAScript guarantee
    ≠
V8 implementation detail
```

Do not present implementation behavior as a universal JavaScript rule.

---

## 3. TypeScript Documentation

For type-system responsibility boundaries, use the official TypeScript handbook and language documentation.

Canonical source:

https://www.typescriptlang.org/docs/

Remember:

```text
TypeScript interfaces
    → compile-time structural contracts

JavaScript runtime
    → actual runtime behavior
```

---

## 4. GRASP / Object-Oriented Design Literature

For terminology and original responsibility-assignment framing, use authoritative object-oriented design literature, especially Craig Larman's treatment of GRASP and responsibility-driven analysis.

Use secondary explanations only as explanatory aids; preserve the distinction between formal terminology and practical adaptation.

---

## 5. Repository-Specific Project Context

For this curriculum, future chapters should connect responsibility decisions to:

- JavaScript/TypeScript runtime behavior
- Node.js services
- enterprise backend systems
- multi-tenant applications
- jewellery ERP workflows
- production constraints
- interview reasoning

Do not allow framework conventions to override core responsibility reasoning without an explicit architectural reason.

---

# 212. Canonical Reference Notes

### Language-level facts

When discussing JavaScript behavior, prefer:

```text
ECMAScript specification
```

over blog posts or tutorials.

### Runtime facts

When discussing:

```text
V8 hidden classes
inline caches
deoptimization
memory layout
```

label them as engine-specific unless they are standardized behavior.

### Architectural claims

When discussing:

```text
domain services
application services
repositories
aggregate boundaries
```

present them as architectural/design guidance rather than JavaScript language semantics.

This distinction prevents category errors.

---

# 213. Principal Decision Framework

For every responsibility assignment, evaluate:

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

Recommended reasoning sequence:

```text
1. Correct ownership
2. Invariant protection
3. Cohesion
4. Coupling
5. Variation isolation
6. Security / least authority
7. Testability
8. Performance
9. Operational implications
10. Future evolution
```

Do not optimize before the responsibility boundary is understood.

---

# 214. Final Mental Model

When you see a requirement, do not immediately ask:

```text
"What classes should I create?"
```

Ask:

```text
What must remain true?
        ↓
What state exists?
        ↓
Who owns that state?
        ↓
Who has the necessary information?
        ↓
Who should make each decision?
        ↓
Who should execute each behavior?
        ↓
Who should coordinate the use case?
        ↓
What should vary independently?
        ↓
What should be abstracted?
        ↓
What should stay simple?
        ↓
How will the design change?
```

That is the mental model of responsibility-driven object design.

---

# 215. Chapter Completion Snapshot

```text
Chapter: 13
Title: Responsibility-Driven Object Design

Core Theory:
[+] Responsibility
[+] State ownership
[+] Behavior ownership
[+] Coordination
[+] Information Expert
[+] Creator
[+] Controller
[+] Low Coupling
[+] High Cohesion
[+] Pure Fabrication
[+] Indirection
[+] Polymorphism
[+] Protected Variations
[+] Tell, Don't Ask
[+] Law of Demeter
[+] Anemic vs Rich Domain Models

Design Practice:
[+] Responsibility matrix
[+] CRC cards
[+] Scenario decomposition
[+] Interaction/message modeling
[+] Entity/service/domain-service decisions
[+] Factory/repository responsibilities
[+] Application-service boundaries
[+] Refactoring responsibility smells

JavaScript/TypeScript:
[+] Private fields
[+] Modules
[+] TypeScript interfaces
[+] Structural typing
[+] `this` and extracted methods
[+] Dependency injection
[+] Runtime vs compile-time responsibility boundaries

Production:
[+] Security
[+] Multi-tenancy
[+] Observability
[+] Transactions
[+] Concurrency boundaries
[+] Reliability
[+] External integrations
[+] Distributed responsibility
[+] Caching/read models

Interview:
[+] Question bank
[+] Predict-the-design
[+] Design reviews
[+] Principal trade-offs
[+] Full enterprise scenario

Next:
Chapter 14 — GRASP
```

---

# 216. One-Sentence Mastery Test

Complete this sentence without notes:

> **I assign a responsibility to an object when __________, while moving it elsewhere is justified when __________.**

A strong answer should include:

```text
knowledge
authority
invariants
cohesion
coupling
variation
coordination
future change
```

---

# Chapter 13 — Completion Statement

This chapter establishes responsibility as the central bridge between JavaScript object mechanics and serious low-level design.

The target skill is not memorizing:

```text
"put this method in that class."
```

The target skill is being able to defend:

```text
"This object owns this responsibility because it
has the relevant knowledge and authority,
protects the associated invariant,
keeps the concept cohesive,
limits coupling,
and leaves likely variations behind stable boundaries."
```

That reasoning is the foundation for the formal GRASP chapter that follows.
