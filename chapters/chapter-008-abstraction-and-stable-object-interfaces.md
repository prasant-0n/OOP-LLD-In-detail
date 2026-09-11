# Chapter 8 — Abstraction & Stable Object Interfaces

> **JavaScript OOP + LLD Mastery**
>
> Encapsulation protects internal state. **Abstraction decides what the outside world should know about the object at all.**
>
> This chapter develops the next layer of object design: stable interfaces, contracts, capabilities, commands, queries, dependency boundaries, abstraction leakage, and substitutability. The goal is to stop designing APIs around implementation details and start designing them around responsibilities.

**Status:** `[ ] Not Started`

# 1. Learning Objectives

By the end of this chapter, you should be able to:

```text
[ ] define abstraction precisely
[ ] distinguish abstraction from encapsulation
[ ] distinguish interface from implementation
[ ] define a stable object interface
[ ] identify implementation details
[ ] identify abstraction leaks
[ ] design narrow public APIs
[ ] explain why fewer public concepts can reduce coupling
[ ] distinguish commands from queries
[ ] reason about query side effects
[ ] reason about command return values
[ ] define object contracts
[ ] distinguish syntax-level contracts from behavioral contracts
[ ] explain preconditions
[ ] explain postconditions
[ ] explain invariants
[ ] explain substitutability
[ ] identify interface assumptions in client code
[ ] recognize accidental interfaces
[ ] design dependency-facing abstractions
[ ] compare concrete dependencies with abstraction boundaries
[ ] explain dependency inversion at an object-design level
[ ] compare inheritance-based abstraction with composition
[ ] use TypeScript interfaces conceptually for contracts
[ ] explain structural typing limits
[ ] distinguish compile-time and runtime contracts
[ ] design adapters around unstable implementations
[ ] identify leaky abstractions
[ ] reason about future change
[ ] debug contract violations
[ ] implement abstraction-oriented APIs
[ ] defend interface design decisions
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
objects
classes
private state
mutability
ownership
invariants
```

# 3. What Is It?

Abstraction means exposing the concepts and operations that clients need while hiding irrelevant implementation details.

```text
                         CLIENT
                           │
                           ▼
                    STABLE INTERFACE
                           │
                ┌──────────┴──────────┐
                │                     │
             contract            capabilities
                │                     │
                └──────────┬──────────┘
                           ▼
                    IMPLEMENTATION
                           │
              ┌────────────┼────────────┐
              │            │            │
           storage      algorithms    dependencies
```

The client should depend on:

```text
what the object guarantees
```

rather than:

```text
how the object happens to implement those guarantees.
```

# 4. Why Does It Exist?

Implementation details change.

For example, a repository may move from:

```text
in-memory array
```

to:

```text
PostgreSQL
```

while the domain still needs:

```text
find customer
save customer
```

A stable abstraction protects clients from unnecessary change.

Without abstraction:

```text
controller
 ↓
SQL details
 ↓
schema details
 ↓
driver details
```

With a boundary:

```text
controller
 ↓
CustomerRepository
 ↓
database implementation
```

The client depends on a responsibility, not a storage mechanism.

# 5. Mental Model

Think:

```text
Implementation
       │
       ▼
   abstraction
       │
       ▼
 stable contract
       │
       ▼
    clients
```

A strong abstraction answers:

```text
What can I ask?
What can I expect?
What can I rely on?
What should I not need to know?
```

A weak abstraction leaks:

```text
SQL
database states
caching rules
transport details
internal object graphs
framework quirks
```

into unrelated callers.

# 6. Core Rules

## Rule 1 — Encapsulation and Abstraction Are Related but Different

```text
encapsulation
→ controls access/authority

abstraction
→ controls what concepts the client must understand
```

You can have private state and still expose a poor abstraction.

## Rule 2 — An Interface Is a Client Dependency

An interface is not merely a list of methods.

It is the set of assumptions a client makes about:

```text
inputs
outputs
errors
timing
side effects
state transitions
performance expectations
```

## Rule 3 — Stable Interfaces Should Express Responsibilities

Prefer:

```js
order.confirm();
```

over exposing:

```js
order.status = "confirmed";
```

when confirmation has domain rules.

## Rule 4 — Hide Volatile Details

If a detail is likely to change independently:

```text
database
network protocol
cache strategy
serialization format
library
algorithm
```

consider hiding it behind a stable responsibility boundary.

## Rule 5 — Small Does Not Automatically Mean Good

An interface with one method can still be badly designed.

For example:

```js
save(data)
```

may be vague if callers need to know:

```text
what gets saved
which state is valid
what errors mean
whether save is idempotent
```

Interface quality is about semantic clarity, not merely method count.

## Rule 6 — Every Public Member Creates Coupling

Exposing:

```js
order.items
order.status
order.discountPolicy
order.internalCache
```

creates multiple client dependencies.

A narrow interface reduces the number of implementation facts that clients can accidentally depend on.

# 7. Syntax

## JavaScript Object Interface

```js
const paymentGateway = {
  async charge(request) {
    // implementation
  },
};
```

The interface is implicit.

## TypeScript Interface

```ts
interface PaymentGateway {
  charge(request: PaymentRequest): Promise<PaymentResult>;
}
```

This expresses a compile-time shape contract.

## Type Alias

```ts
type PaymentGateway = {
  charge(request: PaymentRequest): Promise<PaymentResult>;
};
```

## Abstract Class

```ts
abstract class Repository<T> {
  abstract findById(id: string): Promise<T | null>;
}
```

Use abstract classes only when shared implementation/state or nominal class semantics genuinely help.

# 8. Basic Examples

## Example 1 — Behavior-Oriented API

```js
class BankAccount {
  #balance = 0;

  deposit(amount) {
    if (amount <= 0) {
      throw new RangeError("Invalid amount");
    }

    this.#balance += amount;
  }

  withdraw(amount) {
    if (amount > this.#balance) {
      throw new Error("Insufficient funds");
    }

    this.#balance -= amount;
  }

  getBalance() {
    return this.#balance;
  }
}
```

The interface communicates:

```text
deposit
withdraw
observe balance
```

The representation:

```text
#balance
```

remains internal.

## Example 2 — Repository Abstraction

```ts
interface UserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<void>;
}
```

A service depends on:

```text
UserRepository responsibility
```

rather than:

```text
PostgresUserRepository implementation details.
```

## Example 3 — Stable API, Replaceable Algorithm

```js
class PriceCalculator {
  constructor(pricingPolicy) {
    this.pricingPolicy = pricingPolicy;
  }

  calculate(order) {
    return this.pricingPolicy.calculate(order);
  }
}
```

The calculator depends on:

```text
pricing behavior
```

not on a particular pricing algorithm.

# 9. Execution Walkthrough

Consider:

```js
class CheckoutService {
  constructor(paymentGateway) {
    this.paymentGateway = paymentGateway;
  }

  async checkout(order) {
    const request = {
      amount: order.total,
      currency: order.currency,
    };

    return this.paymentGateway.charge(request);
  }
}
```

Flow:

```text
CheckoutService
      ↓
PaymentGateway abstraction
      ↓
charge()
      ↓
concrete gateway implementation
```

The service needs to know:

```text
how to request a charge
```

It does not need to know:

```text
HTTP
headers
JSON wire format
SDK-specific retry rules
provider endpoint
```

A good abstraction removes irrelevant knowledge from the client.

# 10. Internal Mechanics

JavaScript itself does not require explicit interface declarations for object use.

A client can depend on:

```text
duck-typed behavior
```

such as:

```js
function process(gateway) {
  return gateway.charge(...);
}
```

The runtime checks behavior when the operation occurs.

TypeScript can add compile-time contracts:

```ts
interface Gateway {
  charge(request: Request): Promise<Result>;
}
```

But the interface generally disappears from runtime JavaScript.

Therefore:

```text
TypeScript interface
→ compile-time design contract

JavaScript object
→ runtime behavior.
```

# 11. ECMAScript / Specification Semantics

ECMAScript does not have a runtime `interface` keyword equivalent to TypeScript interfaces.

JavaScript objects expose behavior through:

```text
properties
methods
getters/setters
callability
protocols such as symbols.
```

Abstraction is therefore primarily an application/design concept in JavaScript.

TypeScript adds static typing constructs, but those are checked before runtime and do not automatically enforce arbitrary external inputs.

# 12. Advanced Behavior

## 12.1 Behavioral Interface

An interface is better described behaviorally:

```text
charge(request)
→ accepted request
→ Promise
→ success/failure contract.
```

rather than only:

```text
has a method named charge.
```

## 12.2 Accidental Interface

Consider:

```js
order.items.map(...)
```

A caller may accidentally depend on:

```text
items being an Array
```

even if the actual requirement was:

```text
iterate over line items.
```

That creates an accidental interface.

## 12.3 Leaky Abstraction

Suppose:

```js
repository.findById(id)
```

returns a database driver document containing:

```text
ObjectId
raw field names
database metadata
lazy-loading behavior.
```

Then the repository has leaked persistence details into its clients.

# 13. Commands and Queries

A useful API distinction:

```text
Command
→ asks the object to change or perform an action

Query
→ asks for information.
```

Examples:

```text
order.confirm()       → command
order.total()         → query
account.withdraw(100) → command
account.balance()     → query
```

This helps reason about:

```text
side effects
state transitions
testability
client expectations.
```

# 14. Command Semantics

Weak:

```js
order.setStatus("shipped");
```

Stronger:

```js
order.ship();
```

The stronger command gives the object responsibility for deciding whether shipping is valid.

# 15. Query Semantics

A query should ideally avoid surprising externally meaningful side effects.

Bad surprise:

```text
getTotal()
```

that silently:

```text
writes to database
mutates domain state
triggers external network calls.
```

Internal caching can exist, but externally meaningful side effects should be deliberate and documented.

# 16. Contracts

A contract describes what the client can rely on.

```text
Preconditions
    ↓
operation
    ↓
Postconditions
    ↓
Invariants remain true
```

Example `withdraw(amount)`:

```text
Preconditions:
amount > 0
amount <= available balance

Postcondition:
balance decreased by amount

Invariant:
balance >= 0
```

A signature alone is not the complete contract.

# 17. Behavioral Contract

A behavioral contract includes:

```text
valid inputs
return meaning
error behavior
state changes
side effects
timing expectations
idempotency where relevant
```

A good client should not need to inspect implementation code to understand these guarantees.

# 18. Preconditions

A precondition is expected to be true before an operation.

Example:

```text
withdraw(amount)
requires amount > 0
```

The object may validate and reject invalid requests.

# 19. Postconditions

A postcondition describes what must be true after successful completion.

Example:

```text
withdraw(100)
→ newBalance = oldBalance - 100
```

Strong postconditions make tests and client reasoning easier.

# 20. Invariants

An invariant is a rule that remains true for every valid externally observable state.

Examples:

```text
balance >= 0
order total equals line-item total
tenantId cannot change
completed order cannot return to draft
```

Encapsulation protects transitions that preserve invariants.

Abstraction tells clients how to request those transitions without learning representation details.

# 21. Stable Interface Design

A stable interface should change less frequently than its implementation.

Prefer:

```text
stable domain concept
→ public API

volatile technology
→ internal implementation
```

Example:

```text
PaymentGateway
```

is usually more stable than:

```text
StripeHttpClient
```

when the business concept is:

```text
charge a payment.
```

# 22. Dependency Direction

A service should depend on what it needs:

```text
responsibility/capability
```

rather than:

```text
concrete mechanism.
```

Example:

```js
class InvoiceService {
  constructor(invoiceRepository) {}
}
```

means:

```text
I need invoice persistence capability.
```

Whereas:

```js
class InvoiceService {
  constructor(postgresClient) {}
}
```

means:

```text
I need PostgreSQL.
```

The second can unnecessarily couple policy to technology.

# 23. Dependency Inversion

At the object-design level:

```text
high-level policy
        ↓
abstraction/capability
        ↑
low-level implementation
```

The abstraction should express what the policy actually needs.

# 24. Composition and Abstraction

Composition can provide abstraction without inheritance:

```js
class Checkout {
  constructor(paymentGateway, notifier) {
    this.paymentGateway = paymentGateway;
    this.notifier = notifier;
  }
}
```

The object collaborates with capabilities rather than requiring a class hierarchy.

# 25. Inheritance as Abstraction

Inheritance can define a common behavioral surface:

```js
class PaymentMethod {
  authorize() {}
}
```

with:

```js
class CardPayment extends PaymentMethod {}
class CashPayment extends PaymentMethod {}
```

But inheritance adds:

```text
prototype coupling
subclass obligations
initialization coupling
override risks
```

Abstraction therefore does not imply inheritance.

# 26. TypeScript Interfaces

```ts
interface PaymentGateway {
  charge(request: PaymentRequest): Promise<PaymentResult>;
}
```

Then a service can depend on the contract:

```ts
class CheckoutService {
  constructor(
    private readonly gateway: PaymentGateway,
  ) {}

  checkout(request: PaymentRequest) {
    return this.gateway.charge(request);
  }
}
```

The compile-time type contract helps clients and implementations agree on shape.

# 27. Structural Typing

TypeScript interfaces are structurally typed.

An object with the required shape can satisfy an interface without explicitly declaring `implements`.

That is powerful, but:

```text
shape compatibility
≠
behavioral compatibility.
```

Two objects can have the same method signature and different semantics.

# 28. Compile-Time vs Runtime Contracts

TypeScript can catch:

```text
wrong argument type
missing member
incorrect return type
```

It cannot automatically guarantee:

```text
database connection is alive
payment is authorized
tenant is correct
business invariant is preserved
external JSON is trusted.
```

Production systems often need:

```text
static contracts
+
runtime validation
+
behavioral tests.
```

# 29. Abstraction Leakage

A service that returns ORM-specific objects leaks persistence representation.

A stronger boundary can use:

```text
ORM model
   ↓
mapper
   ↓
domain/application representation
   ↓
client
```

Use the extra boundary when representation volatility and ownership justify it.

# 30. Stable DTO vs Domain Object

Do not automatically treat:

```text
DTO
```

as:

```text
domain object.
```

A DTO may stabilize a transport boundary while a domain object carries behavior and invariants.

# 31. Abstraction and Serialization

```js
JSON.stringify(order)
```

can expose representation details if consumers treat the serialized shape as a public contract.

Decide explicitly:

```text
public transport representation
vs
internal state.
```

# 32. Advanced Behavior — Adapters

An adapter translates one interface into another:

```text
Client
  ↓
Expected Interface
  ↓
Adapter
  ↓
Existing Implementation
```

Example:

```js
class StripeGatewayAdapter {
  constructor(stripeClient) {
    this.stripeClient = stripeClient;
  }

  async charge(request) {
    return this.stripeClient.paymentIntents.create({
      amount: request.amount,
      currency: request.currency,
    });
  }
}
```

The adapter keeps provider-specific concepts from spreading through the application.

# 33. Advanced Behavior — External Vocabulary

When integrating an external system, map:

```text
external vocabulary
```

to:

```text
internal vocabulary.
```

Example:

```text
provider.status = "succeeded"
```

can map to an internal payment status representation.

# 34. Advanced Behavior — Semantic Compatibility

A public object API can break clients even without changing method names.

Breaking changes include:

```text
different error semantics
different timing
new side effects
changed nullability
changed mutation behavior
changed equality assumptions
```

Syntax compatibility is not the complete compatibility contract.

# 35. Advanced Behavior — Interface Segregation

A client should depend on the smallest meaningful capability it needs.

Instead of:

```ts
interface Storage {
  save();
  delete();
  backup();
  restore();
  encrypt();
  migrate();
}
```

a reporting service may only need:

```ts
interface ReportReader {
  findReport(id: string): Promise<Report>;
}
```

This prepares the ground for Interface Segregation Principle.

# 36. Edge Cases

```text
False stability
→ an interface looks stable while exposing ORM records or mutable internals

Over-abstraction
→ wrappers exist without meaningful variation or responsibility

Generic interfaces
→ too generic to communicate domain behavior

Optional-member interfaces
→ clients cannot tell which capabilities are actually guaranteed

Return-type leakage
→ implementation classes become client dependencies
```

# 37. Common Misconceptions

```text
"abstraction means interface keyword."
"encapsulation and abstraction are identical."
"small interface always means good interface."
"every class needs an interface."
"every dependency should have an abstraction."
"TypeScript interface guarantees runtime behavior."
"inheritance is required for polymorphic abstraction."
"getters are always good abstractions."
"DTOs and domain objects are interchangeable."
"wrapping a class automatically creates a good abstraction."
```

# 38. Common Mistakes

```text
[ ] designing APIs around database schemas
[ ] exposing framework objects
[ ] leaking mutable internal collections
[ ] exposing implementation-specific errors
[ ] using concrete dependencies everywhere
[ ] introducing abstractions before understanding actual variation
[ ] creating generic interfaces with weak semantics
[ ] relying only on TypeScript types at runtime boundaries
[ ] confusing method shape with behavioral contract
[ ] making clients know too much about implementation
```

# 39. Comparison With Related Concepts

| Concept | Main question |
|---|---|
| Encapsulation | Who can access/change state? |
| Abstraction | What concepts does the client need to know? |
| Interface | What capability/contract does the client depend on? |
| Implementation | How is the capability delivered? |
| DTO | What data crosses a boundary? |
| Adapter | How are incompatible interfaces translated? |
| Factory | How is construction hidden? |
| Composition | How are capabilities assembled? |
| Inheritance | What behavior/subtyping relationship is shared? |
| Dependency inversion | Which dependency direction should exist? |

# 40. Performance Considerations

Abstraction can introduce:

```text
extra objects
adapter calls
mapping
validation
allocation of DTOs
indirection
```

But it can reduce larger system costs:

```text
change propagation
testing difficulty
technology coupling
coordination cost
```

Do not remove valuable boundaries merely because they add one function call.

# 41. Memory Considerations

Mapping layers may allocate:

```text
DTOs
domain objects
adapter request objects
snapshots
```

At high throughput, unnecessary conversion can matter.

Conversely, retaining provider objects can retain:

```text
internal graphs
metadata
buffers
```

and increase coupling and memory lifetime.

# 42. Security Considerations

A stable abstraction can also be a security boundary.

Examples:

```text
read-only repository
tenant-scoped repository
payment authorization capability
sanitized DTO
```

Avoid giving a collaborator a capability broader than its responsibility requires.

This aligns abstraction design with least-authority principles.

# 43. Production Usage

Prefer APIs that express:

```text
business intent
stable capabilities
explicit contracts.
```

Avoid exposing technology-specific objects unless consumers genuinely need them.

A stable interface should provide enough information to use the capability correctly and no more.

# 44. Implementation From Scratch

## Exercise 1 — Storage Abstraction

Define:

```ts
interface UserReader {
  findById(id: string): Promise<User | null>;
}
```

Implement:

```text
InMemoryUserReader
DatabaseUserReader
```

Make a service depend only on `UserReader`.

## Exercise 2 — Payment Adapter

Design:

```text
PaymentGateway
StripeAdapter
MockPaymentGateway
```

The application should know nothing about Stripe request objects.

## Exercise 3 — Command-Oriented Order

Replace:

```js
order.status = "cancelled";
```

with:

```js
order.cancel();
```

Protect relevant transition rules.

## Exercise 4 — Capability Segregation

Split a large storage contract into focused capabilities for:

```text
reader
writer
deleter
reporting service
```

## Exercise 5 — Runtime Boundary

Accept external JSON and convert it into a validated internal object.

Document:

```text
external contract
normalization
internal contract
```

# 45. Debugging Exercises

## Debug 1 — Database Leak

```js
class UserService {
  constructor(db) {
    this.db = db;
  }

  async getUser(id) {
    return this.db.users.findUnique({ where: { id } });
  }
}
```

Explain the persistence leak and redesign the boundary.

## Debug 2 — Getter Abstraction Leak

```js
class Order {
  #items = [];

  get items() {
    return this.#items;
  }
}
```

Identify the mutation authority being exposed.

## Debug 3 — TypeScript False Security

```ts
interface UserRepository {
  save(user: User): Promise<void>;
}
```

Explain what a TypeScript interface guarantees at compile time and what it does not guarantee about an arbitrary runtime object.

## Debug 4 — Provider Error Leak

```js
class PaymentService {
  constructor(stripe) {
    this.stripe = stripe;
  }

  async charge(request) {
    return this.stripe.paymentIntents.create(request);
  }
}
```

Identify provider-specific concepts leaked into the service.

## Debug 5 — Over-Abstraction

Review:

```text
IUserRepository
IUserRepositoryFactory
IUserRepositoryProvider
IUserRepositoryManager
```

Identify which abstractions have semantic responsibility and which may be accidental indirection.

# 46. Code Review Exercise

Review:

```js
class OrderController {
  constructor(prisma) {
    this.prisma = prisma;
  }

  async cancelOrder(id) {
    const order = await this.prisma.order.findUnique({
      where: { id },
    });

    if (order.status === "DRAFT") {
      await this.prisma.order.update({
        where: { id },
        data: { status: "CANCELLED" },
      });
    }

    return order;
  }
}
```

Evaluate:

```text
1. Who owns the cancellation invariant?
2. Is persistence leaking into the controller?
3. Is status treated as raw data instead of behavior?
4. Where should cancellation rules live?
5. What abstraction should the controller depend on?
6. Does the returned object represent internal persistence state?
7. What happens if another entry point cancels an order?
```

# 47. Interview Questions

```text
1. What is abstraction?
2. How is abstraction different from encapsulation?
3. What makes an interface stable?
4. What is an abstraction leak?
5. What is a behavioral contract?
6. What are preconditions and postconditions?
7. What is an invariant?
8. What is the difference between a command and a query?
9. Why can raw setters weaken abstraction?
10. What is dependency inversion?
11. When should you introduce an interface?
12. Why is "every class needs an interface" a poor rule?
13. What is an adapter?
14. What is structural typing?
15. What can TypeScript interfaces guarantee?
16. What can they not guarantee at runtime?
17. Why can returning ORM objects be an abstraction leak?
18. How can abstraction improve security?
19. When does abstraction become over-engineering?
```

# 48. Predict-the-Output Exercises

## Exercise A

```js
class Order {
  #status = "draft";

  confirm() {
    this.#status = "confirmed";
  }

  get status() {
    return this.#status;
  }
}

const order = new Order();

console.log(order.status);
order.confirm();
console.log(order.status);
```

## Exercise B

```js
class Cart {
  #items = [];

  add(item) {
    this.#items.push(item);
  }

  get items() {
    return this.#items;
  }
}

const cart = new Cart();
cart.add("gold");

const items = cart.items;
items.push("silver");

console.log(cart.items);
```

## Exercise C

```js
class User {
  static create() {
    return new User();
  }

  greet() {
    return "hello";
  }
}

const user = User.create();

console.log(user.greet());
console.log(typeof User.create);
```

## Exercise D

```js
const service = {
  run() {
    return "ok";
  },
};

function execute(worker) {
  return worker.run();
}

console.log(execute(service));
```

Explain the implicit runtime interface.

## Exercise E

```ts
interface Reader {
  read(): string;
}

const reader = {
  read() {
    return "hello";
  },
};

const typedReader: Reader = reader;

console.log(typedReader.read());
```

Explain TypeScript's role versus runtime JavaScript.

# 49. Mastery Exercises

## Level 1 — Understand

Explain:

```text
abstraction
interface
implementation
contract
abstraction leak
```

## Level 2 — Explain

Compare:

```text
encapsulation
vs
abstraction
```

using a domain object.

## Level 3 — Predict

Identify:

```text
what clients depend on
what is implementation detail
what behavior is guaranteed
```

from unfamiliar APIs.

## Level 4 — Implement

Build:

```text
repository abstraction
payment gateway adapter
command-based domain object
capability-specific interfaces
```

## Level 5 — Debug

Find:

```text
ORM leak
provider error leak
mutable-reference leak
over-abstraction
```

## Level 6 — Apply

Design:

```text
CheckoutService
```

that depends on:

```text
PaymentGateway
InventoryReader
OrderRepository
Notifier
```

without knowing their technologies.

## Level 7 — Compare

Compare:

```text
concrete dependency
interface
adapter
composition
inheritance
```

for integrating an external provider.

## Level 8 — Defend

Answer:

> What makes an abstraction genuinely useful rather than merely adding another interface or wrapper?

Connect:

```text
responsibility
volatility
client knowledge
coupling
behavioral contract
change cost
testability
```

# 50. Key Takeaways

```text
1. Abstraction controls what clients need to understand.
2. Encapsulation controls access and authority.
3. Interfaces represent client dependencies, not just method lists.
4. Behavioral contracts matter more than method names alone.
5. Public APIs should express stable responsibilities.
6. Every exposed member creates potential coupling.
7. Commands express state-changing intent.
8. Queries should avoid surprising externally meaningful side effects.
9. Preconditions, postconditions, and invariants strengthen contracts.
10. Stable abstractions hide volatile technology details.
11. Dependency inversion points higher-level policy toward capabilities rather than mechanisms.
12. Composition can provide strong abstraction without inheritance.
13. TypeScript interfaces provide compile-time structural contracts.
14. TypeScript interfaces do not automatically validate runtime behavior.
15. Adapters translate external interfaces into internal concepts.
16. DTOs are not automatically domain objects.
17. Abstraction leaks occur when implementation details become client requirements.
18. Over-abstraction creates unnecessary indirection and complexity.
19. Least-authority capabilities are useful for design and security.
20. A good abstraction reduces the knowledge required to use an object correctly.
```

# 51. Concept Connections

## Depends On

```text
Chapter 1 — JavaScript Object Model — Foundation
Chapter 2 — Object Identity, Equality & Mutability
Chapter 3 — Prototype Chain Deep Dive
Chapter 4 — Constructor Functions & Instance Construction
Chapter 5 — JavaScript Classes Internally
Chapter 6 — Class Fields & Initialization Semantics
Chapter 7 — Private State & Encapsulation
identity
ownership
invariants
mutability
```

## Builds Toward

```text
Chapter 9 — Inheritance
Chapter 10 — Polymorphism
Chapter 11 — Composition
Chapter 12 — Cohesion & Coupling
GRASP
SOLID
TypeScript interfaces
Dependency Inversion
Dependency Injection
adapters
ports and adapters
DDD boundaries
```

## Related Concepts

```text
information hiding
contracts
preconditions
postconditions
commands
queries
capabilities
adapters
DTOs
dependency inversion
structural typing
```

## Why This Chapter Matters

The moment an application grows, objects stop working alone.

They collaborate.

Therefore the design question becomes:

```text
What must object A know about object B?
```

A strong abstraction answers:

```text
as little as necessary.
```

# 52. Completion Criteria

Mark:

```text
[+] Completed
```

when you can:

```text
define abstraction
design stable object APIs
identify leaks
write behavioral contracts
separate commands and queries
design capability-specific interfaces
```

Mark:

```text
[*] Mastered
```

when you can inspect an unfamiliar API and identify:

```text
stable concept
volatile detail
client assumptions
contract
abstraction leak
unnecessary abstraction
```

and redesign the boundary while clearly defending the trade-offs.

Reading alone does not mark mastery.

# 53. Revision / Retrieval Record

```md
# Chapter 8 — Revision / Retrieval Record

## Attempt
- Date:
- Duration:
- Status before:
- Status after:

## Abstraction
- What must the client know?
- What should remain hidden?

## Interface
- What capability is exposed?
- What behavior is guaranteed?

## Contract
- Preconditions:
- Postconditions:
- Invariants:
- Errors:
- Side effects:

## Commands / Queries
- Command:
- Query:
- Unexpected side effect:

## Leaks
- Implementation detail exposed:
- Why it is unstable:
- Proposed boundary:

## TypeScript
- Compile-time guarantee:
- Runtime limitation:

## Design
- Stable concept:
- Volatile technology:

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

# 54. Canonical References and Source Discipline

Primary language source:

```text
ECMAScript Language Specification
https://tc39.es/ecma262/
```

TypeScript references:

```text
TypeScript Handbook — Object Types
https://www.typescriptlang.org/docs/handbook/2/objects.html

TypeScript Handbook — Type Compatibility
https://www.typescriptlang.org/docs/handbook/type-compatibility.html
```

Source discipline:

```text
JavaScript runtime semantics
→ ECMAScript

TypeScript compile-time behavior
→ TypeScript documentation

Framework/library behavior
→ official framework/library documentation

abstraction quality
→ client needs, volatility, domain requirements, architecture
```

Do not claim that a TypeScript interface validates arbitrary runtime input unless explicit runtime validation exists.

# 55. Completion Snapshot

```text
Chapter: 008
Title: Abstraction & Stable Object Interfaces

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

```text
                 CLIENT
                    │
                    ▼
             STABLE CAPABILITY
                    │
             behavioral contract
                    │
                    ▼
              IMPLEMENTATION
             ┌──────┼──────┐
             │      │      │
          database  API   algorithm
```

The client knows:

```text
what it can ask
what it receives
what can fail
what state change occurs
```

The client does not need to know:

```text
how the capability is implemented.
```

For every public API, ask:

```text
1. What stable concept does this represent?
2. What implementation detail is being hidden?
3. What assumptions does the client still need?
4. What behavioral contract exists?
5. Does the API leak mutable state?
6. Does it leak technology-specific types/errors?
7. Is the abstraction actually needed?
8. Would composition make the boundary clearer?
9. What changes without breaking clients?
```

# Principal Design Principle

> **A good abstraction minimizes the knowledge and authority a client needs while preserving the behavior required by the responsibility.**

# Track Mapping

```text
Track A — Core Theory
    abstraction
    interfaces
    contracts
    commands/queries
    preconditions
    postconditions
    invariants
    dependency inversion
    structural typing

Track B — Implementation
    repository interfaces
    adapters
    capability interfaces
    runtime validation boundaries
    command-oriented APIs

Track C — Interview / Reasoning
    abstraction leaks
    over-abstraction
    concrete vs abstract dependencies
    TypeScript compile-time vs runtime
    interface stability
    design trade-offs
```
