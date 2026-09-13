# Chapter 23 — Dependency Inversion Principle and Policy Boundaries

> **Part D — Core Design Principles**  
> **Status:** `[+] Completed`  
> **Tracks:** Track A — Core Theory · Track B — Implementation · Track C — Interview / Reasoning

## 1. Learning Objectives

By the end of this chapter you should be able to explain the Dependency Inversion Principle (DIP) from first principles; distinguish DIP from dependency injection; separate source-code dependency direction from runtime call direction; identify high-level policy, low-level detail, abstraction, ownership, and boundary; design consumer-oriented TypeScript ports; build a composition root; refactor direct infrastructure coupling; and evaluate security, tenancy, lifetime, retries, idempotency, consistency, observability, performance, and migration trade-offs.

## 2. Prerequisites

Review Chapters 11–22, especially composition, coupling, stable contracts, SRP, OCP, LSP, and ISP. You should know JavaScript modules, functions, classes, promises, TypeScript types/interfaces, and basic testing.

## 3. What Is DIP?

DIP says that high-level policy should not be forced to depend directly on low-level implementation details. Both should depend on stable abstractions, and details should depend on those abstractions.

The architectural question is not “Where can I put an interface?” It is:

> **Which policy owns the need, which detail is volatile, and how can source-code dependency be arranged so the stable policy is protected from that volatility?**

## 4. Why DIP Exists

Infrastructure changes for reasons unrelated to business policy:

- database or ORM migration
- provider replacement
- SDK upgrade
- queue replacement
- framework replacement
- HTTP client replacement
- environment changes
- testing requirements
- security controls
- cloud migration

A direct dependency such as:

```text
OrderPolicy → Prisma → PostgreSQL
```

causes infrastructure change to become policy change.

A controlled boundary is:

```text
OrderPolicy → OrderRepository
                   ↑
          PostgresAdapter
```

The runtime object can still be a PostgreSQL implementation; the source-code contract is owned at the policy boundary.

## 5. Mental Model

```text
VOLATILE DETAILS
┌──────────────────────────────────────────┐
│ DB │ SDK │ HTTP │ Queue │ Cache │ Clock │
└───────────────────┬──────────────────────┘
                    │ implements
                    ▼
              ┌──────────────┐
              │ Stable Ports │
              │ semantic need│
              └──────┬───────┘
                     │ used by
                     ▼
              ┌──────────────┐
              │ Stable Policy│
              │ decisions +  │
              │ invariants   │
              └──────────────┘

Composition Root connects concrete details to ports.
```

## 6. Dependency Is Multidimensional

“A depends on B” can mean several things:

1. source-code import
2. compile-time type dependency
3. runtime object dependency
4. configuration dependency
5. operational dependency
6. security/authority dependency
7. lifecycle dependency
8. temporal dependency

DIP primarily addresses architectural/source-code dependency direction, but production design must reason about all eight.

## 7. Source Dependency vs Runtime Call

These are different graphs.

Source-code graph:

```text
CheckoutService ─────► PaymentAuthorizer
StripeAdapter  ───────► PaymentAuthorizer
```

Runtime call graph:

```text
CheckoutService
      │ calls
      ▼
StripeAdapter
      │ calls
      ▼
Stripe provider
```

Therefore:

> **Call direction tells you who invokes whom. Dependency direction tells you who must know about whom in source code.**

## 8. High-Level Policy vs Low-Level Detail

High-level policy decides:

- business rules
- eligibility
- state transitions
- approval
- pricing
- workflow
- domain invariants

Low-level detail implements:

- SQL
- HTTP
- SDK calls
- serialization
- provider authentication
- message transport
- file access

“High-level” means policy, not file size.

## 9. Stable vs Volatile

Not every dependency deserves inversion.

Consider inversion when there is meaningful:

- volatility
- migration cost
- ownership separation
- testing value
- security authority
- lifecycle boundary
- deployment boundary
- external-system boundary

Direct use is often fine for stable local pure code.

## 10. DIP vs Dependency Injection

DIP is a design principle.

DI is a construction technique.

```text
DIP = where dependency should point
DI  = how a dependency can be supplied
```

This does not satisfy DIP automatically:

```ts
class CheckoutService {
  constructor(
    private readonly gateway: StripePaymentGateway
  ) {}
}
```

The dependency is injected, but the type is still a concrete provider detail.

## 11. Consumer-Oriented Ports

Prefer:

```ts
interface PaymentAuthorizer {
  authorize(
    request: AuthorizationRequest
  ): Promise<AuthorizationResult>;
}
```

over:

```ts
interface StripeClient {
  createPaymentIntent(...): Promise<unknown>;
  createCustomer(...): Promise<unknown>;
  refund(...): Promise<unknown>;
}
```

A port should describe the consumer's semantic need, not reproduce the provider's API.

## 12. Consumer-Side Ownership

A useful heuristic:

> **Put an abstraction with the component whose need should remain stable while implementations change.**

Example:

```text
application/
  ports/
    OrderRepository.ts

infrastructure/
  postgres/
    PostgresOrderRepository.ts
```

Infrastructure depends inward on the application's contract.

## 13. DIP + ISP

ISP asks:

> Does the client depend on more contract surface than it needs?

DIP asks:

> Does the client depend on the right abstraction and dependency direction?

Together they encourage focused capabilities:

```ts
interface PaymentReader {
  getPayment(id: PaymentId): Promise<Payment>;
}

interface PaymentRefunder {
  refund(input: RefundRequest): Promise<RefundResult>;
}
```

A reporting client should not receive refund authority.

## 14. DIP + OCP

A stable port creates an extension seam:

```text
Policy → Port
          ↑
     ┌────┴────┐
 Adapter A  Adapter B
```

This supports extension without repeatedly modifying the protected policy.

## 15. DIP + LSP

Every implementation must preserve the behavioral contract.

If production can return:

```text
approved
declined
timeout_unknown
provider_unavailable
```

but a fake always returns `approved`, tests can give false confidence.

Type compatibility is not enough.

## 16. DIP + SRP

Infrastructure coupling often creates responsibility drift.

Bad:

```ts
class OrderService {
  calculatePrice();
  executeSql();
  callStripe();
  parseEnv();
  publishKafka();
}
```

The issue is not simply “large class.” The issue is multiple change authorities and coupled reasons to change.

## 17. Composition Root

The composition root:

- validates configuration
- creates concrete infrastructure
- controls lifetime
- connects implementations to ports
- defines the runtime object graph

Example:

```ts
const db = createDatabase(config.database);
const orders = new PostgresOrderRepository(db);
const payments = new StripePaymentAuthorizer(config.stripe);

const checkout = new CheckoutService(orders, payments);
```

Policy does not discover providers.

## 18. Service Locator

Avoid:

```ts
class CheckoutService {
  checkout() {
    const payments =
      container.resolve<PaymentAuthorizer>("payments");
    // ...
  }
}
```

The constructor hides requirements.

Problems include:

- hidden dependencies
- weak local reasoning
- runtime-only failures
- harder tests
- unclear lifetime
- poor architecture visibility

An explicit constructor is usually easier to understand.

## 19. Injection Styles

### Constructor injection

Use for required object dependencies.

```ts
constructor(
  private readonly repository: OrderRepository
) {}
```

### Parameter injection

Use for one operation.

```ts
function formatReport(
  report: Report,
  formatter: ReportFormatter
) {}
```

### Factory injection

Use when creation itself is part of the capability.

```ts
interface PaymentGatewayFactory {
  forTenant(tenantId: TenantId): PaymentAuthorizer;
}
```

### Function injection

Use when one function is the whole dependency.

```ts
type HashPassword =
  (password: string) => Promise<string>;
```

### Context injection

Use only when context is a genuine semantic object. A giant `AppContext` is a disguised service locator.

## 20. TypeScript Contracts

Useful tools include:

- `interface`
- `type`
- function types
- `satisfies`
- `readonly`
- discriminated unions
- branded identifiers
- `unknown`

Important limitation:

> TypeScript interfaces are compile-time constructs. They do not make external runtime data trustworthy.

External responses should be validated and mapped.

## 21. Interface vs Abstract Class

Use an interface for an implementation-independent behavior contract.

Use an abstract class only when shared state or behavior genuinely belongs in the abstraction.

Do not introduce inheritance merely to satisfy DIP.

## 22. Function Types

A function is often the best abstraction for one operation:

```ts
type PublishEvent =
  (event: DomainEvent) => Promise<void>;
```

DIP does not require classes.

## 23. `satisfies`

Adapter verification can use:

```ts
const notifier = {
  async send(message: CustomerMessage) {
    // ...
  },
} satisfies CustomerNotifier;
```

This checks compatibility without changing the inferred object type.

## 24. Ports and Adapters

```text
Inbound adapter
 HTTP / CLI / Job
       ↓
Application policy
       ↓
Outbound port
       ↑
Outbound adapter
 DB / Queue / SDK / API
```

Hexagonal/Clean Architecture use related dependency-direction ideas. DIP is the underlying design reasoning, not a framework-specific feature.

## 25. Adapter as Firebreak

An adapter may handle:

- request mapping
- response mapping
- provider errors
- authentication
- retries
- timeout
- provider pagination
- version differences
- telemetry

An adapter should not become the dumping ground for business policy.

## 26. Error Translation

Bad:

```ts
if (error.code === "stripe_card_declined") {
  // business policy now knows Stripe
}
```

Better:

```ts
interface PaymentAuthorizer {
  authorize(
    request: AuthorizationRequest
  ): Promise<AuthorizationResult>;
}
```

The adapter maps provider errors into application semantics.

## 27. Provider-Shaped Abstraction Smell

Suspicious:

```ts
interface PaymentGateway {
  createPaymentIntent();
  attachPaymentMethod();
  setStripeMetadata();
}
```

Better:

```ts
interface PaymentAuthorizer {
  authorize(input: AuthorizationRequest):
    Promise<AuthorizationResult>;
}
```

Provider concepts should terminate at the adapter unless they are genuinely part of the business domain.

## 28. Persistence

Prefer:

```ts
interface OrderReader {
  findById(id: OrderId): Promise<Order | null>;
}
```

over:

```ts
interface SqlExecutor {
  query(sql: string, params: unknown[]): Promise<unknown[]>;
}
```

The first preserves semantic ownership.

The second makes the application a database programmer.

## 29. Repository Warning

A repository can become another ORM wrapper:

```text
findAll
findByAnything
rawQuery
execute
join
```

If the interface reproduces the ORM, the boundary has not achieved much.

## 30. HTTP Provider Boundary

Bad:

```ts
constructor(private readonly http: AxiosInstance) {}
```

Better:

```ts
interface TaxRateProvider {
  getRate(input: TaxRateRequest): Promise<TaxRate>;
}
```

The adapter owns:

- URL
- headers
- serialization
- timeout
- retry
- provider authentication

## 31. Queue Boundary

Use:

```ts
interface JobDispatcher {
  dispatch(job: Job): Promise<void>;
}
```

But do not hide semantics the policy needs.

If ordering matters, represent ordering.

If idempotency matters, represent idempotency.

If capacity/backpressure matters, preserve it.

## 32. Event Boundary

Policy:

```ts
interface EventPublisher {
  publish(event: DomainEvent): Promise<void>;
}
```

Infrastructure:

```text
EventPublisher
      ↑
KafkaAdapter / SNSAdapter / local adapter
```

The policy states the event meaning, not the transport topic.

## 33. Clock Injection

Time is an environment dependency.

```ts
interface Clock {
  now(): Date;
}

class CouponPolicy {
  constructor(private readonly clock: Clock) {}

  isExpired(coupon: { expiresAt: Date }) {
    return this.clock.now() >= coupon.expiresAt;
  }
}
```

Benefits:

- deterministic tests
- reproducible simulations
- explicit temporal dependency

## 34. Randomness

Use an injected source when determinism or security requires control:

```ts
interface RandomSource {
  nextInt(maxExclusive: number): number;
}
```

Do not wrap every standard library call simply because you can.

## 35. Configuration

Prefer:

```text
environment
   ↓
validated config
   ↓
composition root
   ↓
typed dependency
   ↓
policy
```

Avoid parsing:

```ts
process.env.X
```

throughout business policy.

## 36. Security and Least Privilege

A dependency grants authority.

Compare:

```ts
Database
```

with:

```ts
OrderReader
```

The second is narrower.

Use capability-focused ports to reduce accidental privilege, but remember:

> DIP is not authorization.

Authorization, tenant checks, credential security, and auditing remain separate requirements.

## 37. Tenant-Aware Dependency Design

For multi-tenant systems ask:

- Who selects the tenant?
- Can an implementation cross tenant boundaries?
- Is tenant scope explicit?
- How are credentials mapped?
- What is the object lifetime?

Possible contract:

```ts
interface InventoryAllocator {
  allocate(input: {
    tenantId: TenantId;
    branchId: BranchId;
    itemIds: readonly InventoryItemId[];
  }): Promise<AllocationResult>;
}
```

A scoped implementation can also bind tenant identity during construction.

## 38. Lifetime

Common scopes:

```text
application
request
job
transaction
operation
```

A correct port with an incorrect lifetime can still cause:

- data races
- stale state
- memory retention
- connection exhaustion
- tenant leakage

## 39. Transaction Boundaries

A port does not make operations atomic.

If:

```text
save order
allocate inventory
capture payment
publish event
```

span different systems, DIP does not create one distributed transaction.

Consistency strategy must remain explicit.

## 40. Retry Semantics

Ask:

```text
What failed?
Can retry change state?
Is retry idempotent?
What if the provider succeeded but the client timed out?
```

A useful contract can distinguish:

```text
declined
unavailable
timeout_unknown
```

rather than mapping every error to “failed.”

## 41. Idempotency

When retrying external writes:

```text
request A
↓
provider accepts
↓
client times out
↓
retry request A
```

the port may need an idempotency key.

Example:

```ts
type AuthorizationRequest = {
  payment: Payment;
  idempotencyKey: string;
};
```

The semantics should be documented and tested.

## 42. Unknown Outcomes

Important distributed pattern:

```text
request sent
   ↓
   timeout
   ↓
provider outcome unknown
```

Do not automatically classify this as business failure if retrying can duplicate an operation.

## 43. Timeout Ownership

Transport timeout normally belongs near the adapter.

Business deadline may belong to application policy.

These are different concepts.

```text
business deadline
      ↓
transport timeout budget
      ↓
provider call
```

## 44. Cancellation

Use cancellation where the policy genuinely needs it.

```ts
interface DocumentFetcher {
  fetch(
    id: DocumentId,
    options?: { signal?: AbortSignal }
  ): Promise<Document>;
}
```

Do not expose transport-specific cancellation details without a semantic reason.

## 45. Cache Dependencies

A cache is not necessarily transparent storage.

Better:

```ts
interface ProductSnapshotCache {
  get(id: ProductId): Promise<Product | null>;
  put(product: Product): Promise<void>;
}
```

Document:

- freshness
- invalidation
- failure
- consistency

## 46. Batching

An abstraction must preserve scale requirements.

Bad:

```ts
get(id)
```

called 10,000 times.

Better when required:

```ts
interface ProductReader {
  getMany(
    ids: readonly ProductId[]
  ): Promise<readonly Product[]>;
}
```

The abstraction should not accidentally create N+1 calls.

## 47. Streaming

For large data:

```ts
interface AuditReader {
  stream(
    filter: AuditFilter
  ): AsyncIterable<AuditEvent>;
}
```

Do not force a giant `Promise<Event[]>` if the policy genuinely needs streaming and backpressure.

## 48. Functional Dependency Inversion

DIP works without objects:

```ts
type CheckoutDependencies = {
  orders: OrderReader;
  payments: PaymentAuthorizer;
};

function createCheckout(
  deps: CheckoutDependencies
) {
  return async function checkout(
    command: CheckoutCommand
  ) {
    const order =
      await deps.orders.findById(command.orderId);

    if (!order) return { status: "missing" as const };

    return deps.payments.authorize({
      amount: order.total,
      currency: order.currency,
    });
  };
}
```

Functions, closures, and dependency records are valid DI techniques.

## 49. Avoid Giant Dependency Records

Bad:

```ts
type AppDependencies = {
  db: Database;
  redis: Redis;
  kafka: Kafka;
  stripe: Stripe;
  logger: Logger;
  config: Config;
  clock: Clock;
  ...
};
```

Prefer focused dependencies:

```ts
type PricingDependencies = {
  taxRates: TaxRateProvider;
  clock: Clock;
};
```

## 50. Testing

Use the appropriate level:

```text
unit
  policy + substitute ports

contract
  common behavior for implementations

integration
  policy + real adapter + test infrastructure

composition
  verify the production object graph
```

A unit-test seam is not enough if the production wiring is broken.

## 51. Contract Tests

If several implementations exist:

```text
OrderRepository contract
       ↑
   ┌───┼────┐
 fake Postgres Mongo
```

Run shared tests for:

- missing entity behavior
- save/read behavior
- errors
- consistency
- idempotency
- relevant ordering/lifecycle semantics

## 52. State-Machine Contracts

For workflow dependencies model legal states.

```text
Created → Authorized → Captured → Settled
```

An adapter that permits invalid transitions violates the behavioral contract even when TypeScript compiles.

## 53. Property-Based Reasoning

Examples:

```text
save(x); find(x.id) → equivalent x
```

and:

```text
same idempotency key + same input
→ same logical outcome
```

Properties expose hidden assumptions in dependencies.

## 54. Runtime Validation

External data enters through adapters.

Preferred flow:

```text
external data
   ↓
runtime validation
   ↓
provider mapping
   ↓
application/domain type
   ↓
policy
```

TypeScript types alone do not validate untrusted network/database data.

## 55. Framework Isolation

Keep these near delivery/infrastructure boundaries unless semantically required deeper inside:

- Express Request/Response
- framework decorators
- ORM client objects
- cloud SDK objects
- queue consumer metadata

Map them to application commands.

```text
HTTP Request
    ↓
HTTP Adapter
    ↓
CreateOrderCommand
    ↓
Use Case
```

## 56. ORM Isolation

Bad:

```ts
class OrderPolicy {
  constructor(
    private readonly prisma: PrismaClient
  ) {}
}
```

Better:

```ts
class OrderPolicy {
  constructor(
    private readonly orders: OrderRepository
  ) {}
}
```

Provider-specific DTOs should terminate at the persistence adapter.

## 57. Import Dependency Graph

Healthy package direction:

```text
web ───────────────► application
infrastructure ────► application
application ───────► domain
```

Avoid:

```text
application ───────► infrastructure
domain ─────────────► web
```

Use exports/lint/CI rules when possible to defend boundaries.

## 58. Module Cycles

Constructor injection does not repair circular module dependencies.

Watch for:

```text
application → infrastructure
infrastructure → application
```

Create ports in the intended inner boundary and keep concrete construction outward.

## 59. Two-Graph Audit

For each dependency document:

```text
Source import:
Runtime object:
Caller:
Callee:
Contract owner:
Constructor owner:
Lifetime:
Authority:
Tenant scope:
Failure:
Retry:
Consistency:
Observability:
```

This reveals problems that one dependency graph misses.

## 60. Provider Migration

Goal:

```text
Policy → Port ← Old Adapter
             ↑
             New Adapter
```

Migration sequence:

1. characterize current behavior
2. define semantic port
3. wrap old detail
4. inject port
5. add contract tests
6. introduce new adapter
7. run comparison tests
8. switch composition
9. preserve rollback

## 61. Branch by Abstraction

Large migrations can use:

```text
             Stable Port
              ↑      ↑
          Old Impl  New Impl
```

This lets callers migrate incrementally.

Useful for:

- databases
- payment providers
- queues
- search systems
- logging
- storage

## 62. Consumer Port vs Provider API

Consumer:

```ts
interface CustomerLookup {
  findByExternalId(
    id: ExternalCustomerId
  ): Promise<Customer | null>;
}
```

Provider API:

```ts
class VendorClient {
  retrieveCustomer(...);
  createCustomer(...);
  updateCustomer(...);
}
```

The adapter maps vendor semantics to consumer semantics.

## 63. Security Boundary in Construction

Secrets should enter through trusted infrastructure construction.

Bad:

```ts
process.env.STRIPE_SECRET_KEY
```

inside policy.

Better:

```text
config → composition root → provider adapter
```

Policy never handles provider secret material.

## 64. Capability Security

Think of each dependency as a capability.

```text
OrderReader
OrderWriter
PaymentReader
PaymentRefunder
AuditRecorder
```

Smaller capabilities reduce accidental authority.

In an ERP, this supports role-aware assembly:

```text
Cashier   → sales + payment authorization
Auditor   → audit + reports
Warehouse → inventory operations
Finance   → financial operations
```

## 65. Observability

Policy telemetry should be semantic:

```text
order placed
payment declined
inventory allocated
```

Adapter telemetry can be technical:

```text
provider latency
HTTP status
retry count
queue publish latency
```

Preserve:

- correlation ID
- tenant context
- actor context
- outcome
- dependency latency
- failure reason

## 66. Performance

Evaluate:

- remote call count
- batching
- streaming
- serialization
- object allocation
- cache behavior
- connection pooling
- promise layers

Do not reject a useful seam because of theoretical interface dispatch cost without measuring the real path.

## 67. Memory

Watch:

- long-lived closures
- accidental singleton request state
- unbounded caches
- per-request clients
- duplicated SDK instances
- forgotten resources

Lifetime is part of dependency correctness.

## 68. Plugin Boundaries

A plugin contract may be:

```ts
interface PricingRule {
  calculate(
    input: PricingInput
  ): PricingAdjustment;
}
```

Production plugin design also needs:

- versioning
- lifecycle
- resource limits
- compatibility
- failure isolation
- security

DIP alone does not make a plugin safe.

## 69. Feature Flags

A semantic contract can isolate a flag provider:

```ts
interface FeatureFlags {
  isEnabled(
    flag: FeatureFlag,
    context: FlagContext
  ): boolean;
}
```

Document default behavior, targeting, caching, and failure semantics when they affect policy.

## 70. Remote vs Local Dependencies

A local port can be implemented remotely, but do not pretend a network call is equivalent to an in-memory call when latency, availability, consistency, or retry behavior changes policy correctness.

> **Abstract the capability, not away the reality.**

## 71. Transaction Example

For:

```text
save sale
allocate inventory
capture payment
publish event
```

DIP can isolate implementations, but you still need a consistency strategy:

- local transaction
- transactional outbox
- compensation
- workflow
- reconciliation

The port does not create atomicity.

## 72. Hidden Dependencies

Smells include:

```ts
globalThis.db
Repository.instance
AppContext.resolve(...)
process.env.X
Date.now()
```

inside stable policy.

A dependency is still hidden when it is reached indirectly.

## 73. Over-Abstraction

Bad:

```ts
interface AddNumber {
  add(a: number, b: number): number;
}
```

for a private arithmetic operation that has no meaningful volatility.

DIP is not:

> “Every class must implement an interface.”

It is:

> “Invert dependency direction where the boundary protects meaningful policy from meaningful change.”

## 74. Interface Explosion

Warning signs:

- interfaces for every class
- wrappers with no semantic value
- dozens of one-off abstractions
- only one implementation with no volatility reason
- tests that mostly verify wiring

Prefer coherent capability boundaries.

## 75. Generic Gateway Smell

Suspicious:

```ts
interface Gateway {
  execute(type: string, payload: unknown):
    Promise<unknown>;
}
```

This weakens type meaning and often becomes a hidden god interface.

Prefer semantic contracts.

## 76. Adapter Overreach

Bad adapter:

```text
HTTP mapping
+ business pricing
+ eligibility
+ inventory policy
+ discount policy
```

An adapter should protect/translate the boundary, not become the new business-policy blob.

## 77. Policy Overreach

Bad policy:

```text
SQL
+ ORM DTOs
+ Kafka topics
+ SDK errors
+ provider auth
+ raw environment parsing
```

The boundary is too weak.

## 78. Composition Root Anti-Pattern

A composition root can still become unhealthy when it contains large business decisions.

Wiring should assemble known policies and details.

Business provider selection belongs in policy when selection is a business rule rather than simple infrastructure configuration.

## 79. Provider Selection

Ask:

```text
Is provider choice configuration?
or
Is provider choice business policy?
```

Configuration-driven choice can live near composition.

Business-driven choice may need an explicit policy component.

## 80. Dependency Lifetime Matrix

| Dependency | Typical scope |
|---|---|
| Pure policy | application |
| DB pool | application |
| Provider client | application or controlled scope |
| Tenant context | request/job |
| Transaction | transaction |
| Request logger | request |
| Cache | application |
| Random source | application/operation |

These are heuristics; the correct scope depends on mutability and ownership.

## 81. Jewellery ERP Scenario

Consider a multi-tenant, multi-branch jewellery ERP.

A sale completion policy may need:

```ts
interface SaleReader {
  findDraft(id: SaleId): Promise<Sale | null>;
}

interface InventoryAllocator {
  allocate(
    input: AllocationRequest
  ): Promise<AllocationResult>;
}

interface PaymentAuthorizer {
  authorize(
    input: AuthorizationRequest
  ): Promise<AuthorizationResult>;
}

interface AuditSink {
  record(event: AuditEvent): Promise<void>;
}

interface SalesEventPublisher {
  publish(event: SalesEvent): Promise<void>;
}

interface Clock {
  now(): Date;
}
```

Concrete implementations may use:

```text
Prisma
Payment provider SDK
Inventory service
Audit store
Kafka
System clock
```

The policy stays semantic.

## 82. ERP Tenant Boundary

Make tenant/branch scope explicit or safely bound.

Ask:

```text
Can tenant A access tenant B?
Can branch A reserve branch B inventory?
Can a cashier receive finance-only capabilities?
```

DIP helps create narrow seams but does not replace enforcement.

## 83. ERP Capability Assembly

Possible assemblies:

```text
Cashier:
  SaleWriter
  PaymentAuthorizer
  ReceiptPrinter

Warehouse:
  InventoryReader
  InventoryAllocator

Auditor:
  AuditReader
  ReportReader

Finance:
  PaymentReader
  RefundProcessor
  FinancialReportReader
```

This combines DIP, ISP, SRP, and least-privilege reasoning.

## 84. ERP Provider Migration

If payment provider A changes to provider B:

```text
CheckoutPolicy → PaymentAuthorizer
                       ↑
              ┌────────┴────────┐
              A                 B
```

Policy changes only if payment semantics themselves change.

## 85. ERP Distributed Inventory

If inventory moves from the monolith to a remote service, the port can remain semantic, but the contract must address:

- timeout
- retries
- idempotency
- partial allocation
- consistency
- correlation
- reconciliation

A local interface does not erase distributed failure.

## 86. Refactoring Workflow

```text
1. Characterize behavior.
2. Identify stable policy.
3. Inventory direct dependencies.
4. Classify volatility and authority.
5. Define semantic consumer ports.
6. Wrap existing details with adapters.
7. Inject ports.
8. Move construction to the composition root.
9. Add contract tests.
10. Enforce package direction.
11. Remove obsolete imports.
12. Measure whether change coupling improved.
```

## 87. Characterization Tests

Before a legacy refactor, record:

- outputs
- exceptions
- side effects
- call ordering
- retries
- idempotency
- consistency assumptions

Do not accidentally “clean up” behavior that clients currently rely on.

## 88. Golden Master

For difficult migrations:

```text
old implementation(input)
          ↓
recorded outcome
          ↓
new implementation(input)
          ↓
compare normalized outcome
```

This is useful for provider replacement and infrastructure migrations.

## 89. Differential Testing

For a provider migration:

```text
adapterA(input)
adapterB(input)
```

compare:

- business outcome
- error category
- side effects
- normalized response
- telemetry expectations

## 90. Architectural Enforcement

Use CI/lint/package rules to prohibit:

```text
domain → infrastructure
application → provider SDK
domain → web framework
```

Architecture becomes safer when important rules are executable.

## 91. Code Review Exercise

Review:

```ts
class OrderService {
  constructor(private readonly app: AppContext) {}

  async submit(order: Order) {
    const db = this.app.container.resolve("db");
    const payment = this.app.container.resolve("stripe");
    const logger = this.app.container.resolve("logger");

    await db.orders.insert(order);
    await payment.charge(order.total);
    logger.info("success");
  }
}
```

Identify:

```text
hidden dependencies
service locator
database leakage
provider leakage
generic context
unclear retry semantics
unclear transaction semantics
unclear ordering
broad authority
weak test seam
```

## 92. Refactored Shape

```ts
class SubmitOrder {
  constructor(
    private readonly orders: OrderWriter,
    private readonly payments: PaymentAuthorizer,
    private readonly audit: AuditSink,
  ) {}

  async execute(command: SubmitOrderCommand) {
    // semantic policy
  }
}
```

Infrastructure supplies implementations.

## 93. Debugging Exercise 1

A test injects a fake payment gateway, but a module-level singleton was constructed using the real gateway.

Diagnosis:

```text
Construction happened before the test substitution.
```

Lesson:

> A constructor seam is useful only when object creation is controllable.

## 94. Debugging Exercise 2

A singleton repository stores the current tenant in mutable state.

Potential result:

```text
request A tenant
      ↕
request B tenant
```

Lesson:

> Correct abstraction + wrong lifetime = incorrect system.

## 95. Debugging Exercise 3

A fake inventory service always succeeds, but production can partially allocate.

The fake violates useful behavioral substitutability.

The contract needs to model partial outcomes if policy depends on them.

## 96. Debugging Exercise 4

The application imports an ORM entity type directly into a domain package.

Diagnosis:

```text
Provider/data-model leakage through a type dependency.
```

Map it to a stable domain/application type at the adapter boundary.

## 97. Predict-the-Design Exercise

Given:

```ts
class InvoiceService {
  constructor(
    private readonly prisma: PrismaClient
  ) {}
}
```

Predict:

- ORM replacement requires policy edits
- unit tests know persistence details
- remote migration becomes harder
- transaction semantics may be hidden
- tenant constraints may be scattered in query code

## 98. Predict-the-Output Exercise

```ts
interface Writer {
  write(value: string): void;
}

class MemoryWriter implements Writer {
  values: string[] = [];

  write(value: string) {
    this.values.push(value);
  }
}

class Report {
  constructor(private readonly writer: Writer) {}

  run() {
    this.writer.write("A");
  }
}

const writer = new MemoryWriter();
new Report(writer).run();

console.log(writer.values);
```

**Prediction:** `["A"]`

**Trace:** the concrete object is `MemoryWriter`; the policy receives it through the `Writer` abstraction.

**Rule:** the port controls source dependency; construction controls the concrete runtime object.

## 99. Implementation Progression

### Guided

Implement:

```ts
interface Clock {
  now(): Date;
}
```

Create `SystemClock` and `FakeClock`.

### Partially Guided

Implement:

```text
OrderReader
OrderWriter
PaymentAuthorizer
```

and matching fakes.

### No Reference

Build a provider adapter around a payment SDK without leaking its types.

### Edge-Case Hardened

Add:

```text
tenant scope
timeout
retry
idempotency
partial failure
contract tests
```

### Production Grade

Add:

```text
composition root
observability
security review
lifecycle management
architectural CI checks
migration/rollback plan
```

## 100. Track A — Core Theory

Study:

```text
policy vs detail
source dependency direction
consumer ownership
ports
adapters
composition root
volatility
stable abstractions
lifetime
security authority
failure semantics
```

## 101. Track B — Implementation

You should be able to build:

```text
policy
ports
adapters
fakes
contract tests
composition root
dependency graph checks
```

without relying on a DI framework.

## 102. Track C — Interview / Reasoning

Practice explaining:

```text
DIP ≠ DI
source graph ≠ runtime graph
port ownership
provider leakage
composition roots
service locator risk
lifetime
tenant safety
retry/idempotency
when not to invert
```

## 103. Interview Answer

A strong explanation:

> DIP protects stable high-level policy from volatile implementation details by placing a stable, client-oriented abstraction at the policy boundary and making implementation details conform to that abstraction. Dependency injection is one technique for supplying the implementation, but DIP is fundamentally about dependency ownership and direction.

## 104. Interview Scenario

Question:

> How would you decouple an order service from PostgreSQL?

Answer:

```text
1. Identify policy.
2. Identify actual persistence capability.
3. Define consumer-oriented OrderRepository.
4. Implement PostgresOrderRepository.
5. Inject the port.
6. Assemble in composition root.
7. Add contract tests.
8. Keep transaction/consistency semantics explicit.
9. Enforce package dependency direction.
```

## 105. When Not to Use DIP

Do not add inversion merely because a principle says so.

Direct use is reasonable when:

- dependency is stable
- scope is local
- abstraction adds no semantic value
- substitution is implausible
- cost exceeds expected benefit
- code is a pure local operation

Example:

```ts
function normalizeSku(value: string) {
  return value.trim().toUpperCase();
}
```

No interface is needed just to satisfy a slogan.

## 106. Principal-Level Trade-Offs

Evaluate:

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

A senior design review should explain the trade-off instead of declaring “DIP is always better.”

## 107. Principal Questions

For a candidate dependency ask:

```text
What policy needs it?
What detail changes?
Who owns the need?
Who owns the abstraction?
What authority does it grant?
What is the lifetime?
What can fail?
Can the operation be retried?
Can the outcome be unknown?
What consistency matters?
What tenant scope applies?
What telemetry must survive?
Does the port preserve batching/streaming?
What does the abstraction cost?
```

## 108. Abstraction Stability Test

An abstraction is stronger when:

1. clients need only semantic operations
2. implementations can vary
3. provider details can change independently
4. failure semantics are explicit
5. authority is appropriately narrow
6. contract churn is low
7. consumers do not need provider-specific workarounds

## 109. Contract Smell

If clients repeatedly ask:

```text
“Can I get the raw provider object?”
“Can I use the provider's special operation?”
“Why does this port return a provider error?”
“Why can't this operation batch?”
```

the abstraction may be wrong or too weak.

Do not solve this by leaking the entire provider.

Revisit the semantic boundary.

## 110. Architecture Smell

If the dependency graph looks like:

```text
Domain
 ↓
Application
 ↓
Infrastructure
 ↑
Application
```

you may have a cycle.

The goal is to make dependencies point consistently toward the intended stable boundary while runtime calls can travel outward through ports.

## 111. Canonical References and Source Discipline

For language semantics, consult the ECMAScript and TypeScript documentation. For host behavior, use current Node.js or browser documentation. For DI containers and frameworks, use the current framework documentation.

Always distinguish:

```text
ECMAScript guarantee
TypeScript behavior
Host/runtime behavior
Framework behavior
Architecture/design judgment
```

DIP itself is an architectural principle, not an ECMAScript feature.

## 112. Completion Criteria

You are complete when you can:

```text
[+] explain DIP from first principles
[+] distinguish DIP from DI
[+] draw source and runtime dependency graphs
[+] identify policy and detail
[+] define consumer-owned ports
[+] implement adapters
[+] build a composition root
[+] refactor legacy direct coupling
[+] model failure/retry/idempotency
[+] reason about lifetime
[+] reason about tenant scope
[+] apply least-privilege capability design
[+] defend against over-abstraction
[+] enforce package direction
[+] test multiple implementations
[+] defend decisions at principal level
```

## 113. Mastery Gate

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

Reading alone does not mark mastery.

You have mastery when you can defend:

> **which abstraction, owned by whom, protecting which policy from which change, with which security/reliability/performance semantics, and at what cost.**

## 114. Concept Connections

```text
Chapter 18 — Stable Contracts
        ↓
defines what a good boundary means

Chapter 19 — SRP
        ↓
identifies reasons to change

Chapter 20 — OCP
        ↓
uses seams for controlled extension

Chapter 21 — LSP
        ↓
requires substitutable implementations

Chapter 22 — ISP
        ↓
keeps capability contracts focused

Chapter 23 — DIP
        ↓
controls dependency ownership and direction
```

## 115. Golden Rules

1. DIP is about dependency direction and ownership.
2. Dependency injection is only a technique.
3. Prefer semantic, consumer-oriented ports.
4. Concrete infrastructure belongs near the composition root.
5. Adapters translate and isolate provider details.
6. Runtime call direction can differ from source dependency direction.
7. Lifetime is part of dependency correctness.
8. Retry, timeout, idempotency, and consistency are contract concerns when policy depends on them.
9. Narrow capabilities reduce accidental authority.
10. Tenant scope must be explicit or safely bound.
11. Do not hide remote/distributed semantics behind a fake transparency.
12. Do not create interfaces for stable local code merely to satisfy DIP.
13. Use contract tests when multiple implementations matter.
14. Enforce important package rules mechanically when possible.
15. A good abstraction pays for itself by protecting a real boundary.

## 116. Revision / Retrieval Record

### First Retrieval

```text
[ ] Define DIP.
[ ] Distinguish DIP and DI.
[ ] Explain consumer-owned ports.
[ ] Draw source and runtime graphs.
[ ] Explain composition roots.
```

### Second Retrieval

```text
[ ] Refactor a provider-shaped contract.
[ ] Diagnose a service locator.
[ ] Explain wrong lifetime risks.
[ ] Model tenant-safe dependencies.
[ ] Model retry and unknown outcome semantics.
```

### Third Retrieval

```text
[ ] Defend against over-abstraction.
[ ] Defend port granularity.
[ ] Explain provider migration.
[ ] Explain distributed semantics.
[ ] Defend a production dependency design.
```

## 117. Principal Decision Worksheet

```text
Policy:
Detail:
Volatility:
Consumer:
Contract owner:
Port:
Adapter:
Composition root:
Source dependency:
Runtime dependency:
Authority:
Tenant scope:
Lifetime:
Failure modes:
Timeout:
Retry:
Idempotency:
Consistency:
Observability:
Performance constraints:
Migration plan:
Abstraction cost:
Decision:
```

## 118. Final Mental Model

```text
                  VOLATILE DETAILS
       ┌────────────────────────────────┐
       │ DB │ SDK │ HTTP │ Queue │ Time │
       └────────────────┬───────────────┘
                        │ implements
                        ▼
                ┌─────────────────┐
                │  STABLE PORTS   │
                │ semantic needs  │
                └────────┬────────┘
                         │
                         ▼
                ┌─────────────────┐
                │ STABLE POLICY   │
                │ rules + state   │
                └─────────────────┘

Composition Root:
  validate config
  create concrete details
  control lifetime
  connect details to ports
  expose only intended capabilities
```

The essential question is:

> **Which policy owns the need, which detail is volatile, and how should source-code dependencies be arranged so change flows outward rather than forcing stable policy to follow infrastructure?**

# Chapter 23 — Completion Snapshot

**Chapter:** Dependency Inversion Principle and Policy Boundaries  
**Status:** `[+] Completed`  
**Primary skill:** dependency direction and policy/detail separation  
**Primary technique:** consumer-oriented ports + adapters + composition root  
**Primary anti-pattern:** confusing dependency injection with dependency inversion  
**Principal test:** justify the boundary by change, ownership, semantics, security, lifetime, operational behavior, and cost

**Mastery status:** `[*] Mastered only after implementation + debugging + defense.`

