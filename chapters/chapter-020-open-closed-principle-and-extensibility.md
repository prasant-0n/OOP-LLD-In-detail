# Chapter 20 — Open/Closed Principle and Extensibility

> **Status:** `[~] In Progress`  
> **Part:** D — Core Design Principles  
> **Primary theme:** Open/Closed Principle (OCP), extension seams, protected variation, and change without destabilizing existing behavior  
> **Tracks:** Track A — Core Theory · Track B — Implementation · Track C — Interview / Reasoning

---

# 1. Learning Objectives

By the end of this chapter, you should be able to:

1. Define the Open/Closed Principle precisely.
2. Explain the tension between “open for extension” and “closed for modification.”
3. Distinguish legitimate extension from uncontrolled configurability.
4. Identify stable variation points in JavaScript/TypeScript systems.
5. Explain how polymorphism, composition, dependency injection, factories, registries, strategies, and adapters create extension seams.
6. Recognize when OCP reduces change blast radius.
7. Recognize when OCP creates unnecessary abstraction and indirection.
8. Apply OCP to pricing, payments, notifications, storage, reporting, and workflow systems.
9. Connect OCP with SRP, LSP, ISP, DIP, protected variations, and composition.
10. Design TypeScript extension contracts without overfitting to current implementations.
11. Evolve APIs while preserving contracts and invariants.
12. Build plugin-style architecture deliberately.
13. Reason about extension in monoliths, packages, and distributed services.
14. Test extension contracts without coupling tests to implementation details.
15. Defend an OCP decision at principal-engineer level.

---

# 2. Prerequisites

You should understand:

```text
Chapter 17 → refactoring and design smells
Chapter 18 → stable contracts and invariants
Chapter 19 → Single Responsibility Principle
```

Also useful:

```text
composition
polymorphism
interfaces
dependency injection
GRASP Protected Variations
TypeScript structural typing
factories
adapters
strategy pattern
```

---

# 3. What Is the Open/Closed Principle?

The Open/Closed Principle says:

> **Software entities should be open for extension but closed for modification.**

The idea is not that source code must literally never change.

The useful interpretation is:

```text
new variation
    ↓
add or substitute extension
    ↓
without destabilizing already-correct behavior
```

A mature design creates **seams** where expected variation can enter.

---

# 4. The Apparent Paradox

The principle sounds contradictory:

```text
open for extension
+
closed for modification
```

It is not.

“Closed” means:

```text
existing stable behavior does not need invasive modification
```

“Open” means:

```text
new behavior can be introduced through an intentional variation point
```

---

# 5. Why OCP Exists

Every modification carries regression risk.

Suppose one class contains:

```ts
if (type === "A") ...
if (type === "B") ...
if (type === "C") ...
```

Adding:

```text
type D
```

requires editing existing decision logic.

If the decision point is stable but new behavior can be supplied through a strategy:

```text
new strategy
→ existing coordinator unchanged
```

the blast radius can shrink.

---

# 6. OCP Is About Controlled Change

OCP is not:

```text
never edit old code
```

It is:

```text
separate stable policy from expected variation
```

This creates a controlled change boundary.

---

# 7. OCP and SRP

SRP asks:

```text
Does this module have one coherent reason to change?
```

OCP asks:

```text
Can expected variation be added without repeatedly modifying the stable core?
```

The two principles interact:

```text
SRP
  identifies change responsibility

OCP
  creates extension boundaries around important variation
```

---

# 8. OCP and Change Axes

Suppose:

```text
payment provider
```

is an expected variation axis.

Instead of:

```ts
if (provider === "stripe") ...
else if (provider === "razorpay") ...
```

inside the use case:

```ts
interface PaymentGateway {
  charge(input: ChargeInput): Promise<ChargeResult>;
}
```

The use case depends on the stable responsibility.

Providers become extensions.

---

# 9. OCP and Protected Variations

GRASP Protected Variations asks:

> What point of predicted instability should we protect behind a stable interface?

This is closely aligned with OCP.

Examples:

```text
payment provider
tax policy
shipping algorithm
storage mechanism
notification channel
```

---

# 10. The Core Mental Model

```text
stable core
    ↓
extension point
    ↓
variation
```

Not:

```text
stable core
    ↓
100 configuration flags
    ↓
every possible behavior
```

The first creates explicit variation.

The second creates combinatorial complexity.

---

# 11. Closed for Modification Does Not Mean Frozen

A core abstraction may still change when:

```text
requirements change
contract changes
defect is fixed
new invariant is discovered
architecture evolves
```

OCP is about **avoiding repeated modification for expected variation**, not banning all maintenance.

---

# 12. Expected vs Unexpected Variation

OCP is valuable when variation is:

```text
known enough
valuable enough
likely enough
isolatable enough
```

If no real variation exists, abstraction may be premature.

---

# 13. YAGNI Constraint

Before creating an extension point ask:

```text
Do we actually expect multiple implementations?
Is change likely?
Is the cost of modification significant?
Would extension reduce future risk?
```

Do not build a plugin framework for one implementation.

---

# 14. OCP and Abstraction

An abstraction is useful when it captures a stable contract around variation.

Bad:

```ts
interface GenericEverything {
  execute(input: unknown): unknown;
}
```

Better:

```ts
interface DiscountPolicy {
  calculate(input: DiscountInput): Money;
}
```

The second expresses meaningful extension semantics.

---

# 15. OCP and Polymorphism

Polymorphism is one of the strongest OCP tools.

```ts
interface TaxPolicy {
  calculate(input: TaxInput): Money;
}

class IndiaTaxPolicy implements TaxPolicy {}
class ExportTaxPolicy implements TaxPolicy {}
```

The stable consumer calls:

```ts
policy.calculate(input);
```

without knowing concrete policy selection details.

---

# 16. OCP and Composition

Composition creates extension through object substitution.

```ts
class CheckoutService {
  constructor(
    private readonly payment: PaymentGateway,
  ) {}
}
```

Replace:

```text
StripeGateway
```

with:

```text
AlternativeGateway
```

without changing checkout orchestration.

---

# 17. OCP and Delegation

Delegation lets a stable object forward variable behavior.

```ts
class PriceCalculator {
  constructor(
    private readonly policy: PricingPolicy
  ) {}

  calculate(order: Order) {
    return this.policy.calculate(order);
  }
}
```

The calculator can remain stable while policy changes.

---

# 18. OCP and Dependency Injection

Dependency injection supplies extension points from outside.

```ts
const service = new CheckoutService(
  paymentGateway
);
```

The stable class does not need to know every future implementation.

---

# 19. OCP and Factories

A factory can isolate construction variation:

```ts
interface ReportExporter {
  export(report: Report): Buffer;
}
```

Selection logic:

```ts
function createExporter(format: ExportFormat): ReportExporter {
  // choose implementation
}
```

The consumer remains independent from concrete exporters.

---

# 20. Factory vs OCP

A factory by itself does not guarantee OCP.

Bad factory:

```ts
function createExporter(format) {
  if (format === "pdf") return new PdfExporter();
  if (format === "csv") return new CsvExporter();
  if (format === "xml") return new XmlExporter();
}
```

Adding a format still modifies the factory.

This may still be acceptable when:

```text
the finite set is centrally owned
```

OCP is about trade-offs, not dogma.

---

# 21. Registry-Based Extension

A registry can reduce central switch modification:

```ts
const exporters = new Map<string, ReportExporter>();

exporters.set("pdf", pdfExporter);
exporters.set("csv", csvExporter);
```

Adding implementations can become additive.

But the registry introduces its own contracts:

```text
registration
duplicate keys
initialization order
discovery
lifecycle
```

---

# 22. Plugin Architecture

A plugin architecture often has:

```text
stable host contract
+
independently supplied implementations
```

Example:

```ts
interface PaymentPlugin {
  id: string;
  authorize(input: AuthorizeInput): Promise<Result>;
}
```

Plugins extend capability.

---

# 23. OCP and Plugin Safety

Plugins create risks:

```text
untrusted code
resource exhaustion
dependency conflicts
version mismatch
contract drift
security authority
```

An extension point must have governance.

---

# 24. OCP and Contracts

Chapter 18 established:

```text
contract = observable promise
```

An extension mechanism is only useful when the contract is strong enough.

Otherwise:

```text
implementation A
```

and:

```text
implementation B
```

may behave incompatibly despite matching the interface.

---

# 25. OCP and LSP

OCP often relies on substitutable implementations.

Therefore:

```text
OCP
  extension without consumer modification

LSP
  extensions preserve behavioral expectations
```

If extensions violate the base contract, OCP creates unsafe extensibility.

---

# 26. OCP and ISP

Small interfaces create easier extension points.

```ts
interface MailSender {
  send(message: OutboundMessage): Promise<void>;
}
```

This is easier to implement correctly than:

```ts
interface MessagingEverything {
  sendEmail();
  sendSms();
  sendPush();
  createTemplate();
  renderPdf();
}
```

---

# 27. OCP and DIP

Dependency Inversion provides the dependency direction:

```text
stable high-level policy
        ↓
stable abstraction
        ↓
replaceable implementation
```

OCP benefits from this structure.

---

# 28. OCP and Encapsulation

Encapsulation protects the stable core from extension-specific state.

The extension can vary behind a boundary without exposing internal representation.

---

# 29. OCP and Information Hiding

Hide:

```text
provider details
algorithm details
storage details
protocol details
```

behind stable contracts.

Extension points should expose only what consumers need.

---

# 30. OCP and Cohesion

A good extension interface should represent one cohesive variation axis.

Bad:

```ts
interface Plugin {
  price();
  save();
  sendEmail();
  render();
}
```

Too broad.

Better:

```ts
interface PriceRule {
  calculate(input): Money;
}
```

---

# 31. OCP and Coupling

Without OCP-friendly boundaries:

```text
new implementation
→ modify central switch
→ touch old branches
→ regression risk
```

With a strong extension boundary:

```text
new implementation
→ add implementation
→ register/configure
→ old branches unchanged
```

---

# 32. OCP and Change Blast Radius

A useful design goal:

```text
new variation
→ small bounded change set
```

But not:

```text
new variation
→ unlimited framework complexity
```

The extension seam should be proportionate to expected change.

---

# 33. Example — Notifications

Bad:

```ts
class NotificationService {
  send(channel: string, message: string) {
    if (channel === "email") {}
    if (channel === "sms") {}
    if (channel === "whatsapp") {}
  }
}
```

Each new channel modifies existing logic.

---

# 34. Strategy-Based Notification

```ts
interface NotificationChannel {
  send(message: NotificationMessage): Promise<void>;
}

class NotificationService {
  constructor(
    private readonly channel: NotificationChannel
  ) {}

  send(message: NotificationMessage) {
    return this.channel.send(message);
  }
}
```

A new channel can be introduced as another implementation.

---

# 35. Strategy vs Branching

Branching may be better when:

```text
few stable variants
variants rarely change
selection is centralized
behavior is simple
```

Strategies become more attractive when:

```text
variants grow
logic is complex
variants evolve independently
testing benefits from isolation
```

---

# 36. Example — Tax

Bad:

```ts
function calculateTax(order: Order, region: string) {
  if (region === "IN") return ...
  if (region === "US") return ...
  if (region === "EU") return ...
}
```

Better:

```ts
interface TaxPolicy {
  calculate(order: Order): Money;
}
```

Selection remains separate.

---

# 37. Example — Shipping

```ts
interface ShippingPolicy {
  calculate(order: Order): Money;
}
```

Implementations:

```text
FlatRateShipping
WeightBasedShipping
DistanceBasedShipping
CarrierQuoteShipping
```

The application does not need to know calculation internals.

---

# 38. Example — Pricing

Pricing may vary by:

```text
customer type
branch
promotion
channel
currency
time period
```

Do not automatically create one strategy for every dimension.

First determine which variation actually needs independent evolution.

---

# 39. Multi-Dimensional Variation

A dangerous design creates:

```text
CustomerPolicy
BranchPolicy
ChannelPolicy
PromotionPolicy
CurrencyPolicy
```

and combines all of them.

This can cause combinatorial configuration.

Sometimes one cohesive pricing policy is simpler.

---

# 40. Composition Over Combinatorial Subclasses

Avoid:

```text
VipBranchWholesalePricing
VipOnlineWholesalePricing
RetailBranchPricing
...
```

Composition can model independent dimensions.

```text
basePrice
+
discountPolicy
+
channelAdjustment
+
taxPolicy
```

But only when those dimensions are truly independent.

---

# 41. OCP and Configuration

Configuration can be an extension mechanism:

```json
{
  "maxRetries": 3
}
```

But configuration is not a substitute for abstraction.

A configuration file cannot safely encode arbitrary business behavior without creating a new language and governance problem.

---

# 42. Data-Driven Extension

Sometimes behavior can be safely represented as data.

Example:

```text
tax rates by jurisdiction
```

A new jurisdiction may require:

```text
new data
```

rather than:

```text
new code
```

This is a valid OCP style when the behavior is genuinely data-driven.

---

# 43. Algorithm vs Data Variation

Ask:

```text
Does the variation change data?
or
does it change algorithm?
```

Data variation:

```text
new rate table
```

Algorithm variation:

```text
new tax calculation formula
```

The first may need configuration/data.

The second may need strategy/policy extension.

---

# 44. Rule Engines

A rule engine can make behavior configurable.

But rule engines introduce:

```text
DSL complexity
validation
debugging
versioning
security
performance
observability
```

Do not build one just to avoid a few conditionals.

---

# 45. OCP and Domain Rules

A domain rule should be extensible only when the business actually expects multiple rule variants.

Do not abstract away a stable business truth.

---

# 46. OCP and Stable Core

A useful architecture:

```text
stable domain contract
        ↑
extension policies
        ↑
volatile implementations
```

The core should contain:

```text
invariants
stable semantics
domain meaning
```

Extensions should contain:

```text
expected variation
provider specifics
algorithm alternatives
```

---

# 47. OCP and Invariants

An extension must preserve core invariants.

Example:

```text
PaymentGateway
```

may vary implementation.

But every gateway must preserve:

```text
currency semantics
amount semantics
result contract
error taxonomy
idempotency contract
```

---

# 48. Extension Contract Design

A good extension interface specifies:

```text
inputs
outputs
errors
side effects
ordering
timeouts
idempotency
security
resource limits
```

Do not expose an interface with only method names.

---

# 49. Capability-Oriented Extension

Instead of:

```ts
interface Provider {
  everything(): void;
}
```

define the exact capability:

```ts
interface RefundGateway {
  refund(input: RefundInput): Promise<RefundResult>;
}
```

This limits extension responsibility.

---

# 50. Optional Capabilities

Not every provider supports every feature.

Avoid pretending:

```ts
interface PaymentProvider {
  capture();
  refund();
  tokenize();
  recurring();
  disputes();
}
```

all providers support everything.

Use capability-specific interfaces:

```text
Authorizer
Capturer
Refunder
Tokenizer
```

or capability negotiation where appropriate.

---

# 51. OCP and Capability Negotiation

An extension can expose:

```ts
capabilities(): readonly Capability[];
```

This is useful when provider feature sets legitimately differ.

But the core must still define behavior when a capability is absent.

---

# 52. Optional Interface Methods

This:

```ts
interface PaymentProvider {
  refund?(): Promise<void>;
}
```

can make consumers branch everywhere.

Capability-specific interfaces often provide cleaner contracts.

---

# 53. OCP and Null Object

A Null Object can provide a valid extension implementation:

```ts
class NoopNotifier implements Notifier {
  async send() {}
}
```

Useful when:

```text
absence itself is a supported policy.
```

Do not use it to hide mandatory failures.

---

# 54. OCP and Defaults

Defaults can preserve simple use cases:

```ts
class CheckoutService {
  constructor(
    private readonly taxPolicy: TaxPolicy = defaultTaxPolicy
  ) {}
}
```

The abstraction exists for meaningful variation.

The default keeps DX simple.

---

# 55. OCP and Builders

Builders can support extension of construction.

But builders can become:

```text
giant mutable configuration objects
```

Use them when construction complexity justifies the abstraction.

---

# 56. OCP and Abstract Factory

Abstract Factory is useful when a **family of related products** must vary together.

Example:

```text
IndiaTaxFactory
IndiaInvoiceFactory
IndiaPaymentFactory
```

But if only one product varies, Strategy or Factory may be simpler.

---

# 57. OCP and Template Method

Template Method uses inheritance:

```text
stable algorithm skeleton
+
subclass hooks
```

It can satisfy OCP.

But inheritance can create:

```text
fragile base class
tight coupling
subclass lifecycle constraints
```

Composition is often safer when variation is independent.

---

# 58. OCP and Strategy

Strategy usually offers:

```text
composition
runtime substitution
isolated tests
lower inheritance coupling
```

It is a common OCP mechanism.

Still, a strategy interface should exist because the variation is meaningful.

---

# 59. OCP and Adapter

Adapters let new external implementations conform to an existing contract.

```text
external provider
      ↓
adapter
      ↓
stable port
```

The new provider extends the system without changing core logic.

---

# 60. OCP and Decorator

Decorator can extend behavior without modifying the wrapped implementation:

```ts
new RetryGateway(
  new LoggingGateway(
    actualGateway
  )
);
```

This creates composable extension.

---

# 61. Decorator Risks

Too many decorators create:

```text
call-chain complexity
ordering ambiguity
debugging difficulty
```

Use them where cross-cutting extension is real.

---

# 62. OCP and Middleware

Middleware is an extension mechanism:

```text
request
→ middleware A
→ middleware B
→ handler
```

New cross-cutting behaviors can be added without modifying every handler.

---

# 63. Middleware Ordering Contract

Middleware extension creates ordering semantics:

```text
auth
→ authorization
→ validation
→ handler
```

Changing order can change behavior.

The extension model must define lifecycle and ordering.

---

# 64. OCP and Event-Driven Design

Event-driven systems can be open for extension through new subscribers:

```text
OrderCreated
  ↓
InventoryProjection
  ↓
AnalyticsProjection
  ↓
NotificationHandler
```

A new consumer can often be added without modifying the producer.

---

# 65. Event Extension Risks

Adding event consumers creates:

```text
new side effects
new operational load
new privacy concerns
new failure modes
```

“Add a subscriber” is not free.

---

# 66. OCP and Observer

Observer is an extension mechanism.

But too many observers can create:

```text
hidden side effects
ordering uncertainty
debugging complexity
```

Use explicit event contracts.

---

# 67. OCP and Persistence

Repositories can support multiple implementations:

```text
InMemoryRepository
SqlRepository
CachedRepository
```

A stable domain/application contract can remain unchanged.

---

# 68. OCP and Repository Abstractions

Do not create:

```ts
interface Repository<T>
```

just because generic repositories look reusable.

A repository abstraction should protect a meaningful persistence contract.

---

# 69. OCP and Storage Migration

A good repository seam can allow:

```text
Mongo
→ PostgreSQL
```

without changing domain logic.

But the repository contract must not expose provider-specific semantics.

---

# 70. OCP and Serialization

Stable domain models can be extended with new serializers:

```text
JsonSerializer
CsvSerializer
XmlSerializer
```

when those are meaningful representations.

---

# 71. OCP and Reporting

A report generator may support:

```text
CSV
PDF
Excel
JSON
```

Through exporters.

But if reporting formats rarely change, a simple dispatcher may be cheaper.

---

# 72. OCP and Query Models

Different consumers may need different read models.

Extension can happen through:

```text
projection
query adapter
read model
```

rather than modifying the core domain aggregate for every reporting need.

---

# 73. OCP and APIs

An API can support extension by adding:

```text
new endpoints
new optional fields
new capabilities
```

without changing existing semantics.

Backward compatibility remains important.

---

# 74. OCP and API Versioning

Sometimes extension means:

```text
new version
```

rather than modifying old endpoint behavior.

Versioning preserves the old contract while exposing a new contract.

---

# 75. Additive vs Breaking Change

Additive:

```text
new optional field
new endpoint
new optional capability
```

Breaking:

```text
remove field
change semantics
change error meaning
tighten valid inputs
```

OCP encourages additive evolution where sensible.

---

# 76. OCP and TypeScript Structural Typing

Structural typing makes extension easy:

```ts
const strategy = {
  calculate(input: TaxInput) {
    return ...
  }
};
```

No explicit inheritance is required.

This can make composition lightweight.

---

# 77. OCP Without Classes

Functional design can follow OCP:

```ts
type TaxCalculator = (input: TaxInput) => Money;

function checkout(
  taxCalculator: TaxCalculator
) {
  // ...
}
```

The variation is supplied as a function.

---

# 78. Function Types as Extension Contracts

Functions are useful when:

```text
variation has small surface area
state is unnecessary
lifecycle is simple
```

A class is not required for OCP.

---

# 79. Object Extension vs Data Extension

Sometimes extension can be:

```text
add object
```

Sometimes:

```text
add configuration/data
```

Choose based on whether the variation is:

```text
behavioral
or
declarative
```

---

# 80. Declarative Extension

Example:

```json
{
  "discountRules": [
    {
      "customerType": "VIP",
      "percentage": 10
    }
  ]
}
```

This can extend data-driven behavior without code changes.

But you need:

```text
schema
validation
versioning
safe defaults
```

---

# 81. OCP and Schema Versioning

Data-driven extensions can break if schema changes.

Treat configuration as a contract:

```text
schema version
validation
migration
compatibility
```

---

# 82. OCP and Security

Every extension point expands the trusted computing surface.

Questions:

```text
Who can register an extension?
What permissions does it receive?
What data can it access?
Can it execute arbitrary code?
Can it exhaust resources?
```

Open architecture does not mean unrestricted architecture.

---

# 83. Plugin Sandboxing

For untrusted extensions, consider:

```text
process boundary
worker boundary
restricted capabilities
resource quotas
network restrictions
timeouts
```

Exact mechanisms depend on the environment.

---

# 84. Extension Authority

Pass only required capabilities.

Bad:

```ts
plugin.initialize({
  database,
  secrets,
  filesystem,
  httpClient,
});
```

Better:

```ts
plugin.initialize({
  paymentApi,
  logger,
});
```

Least authority improves security.

---

# 85. OCP and Multi-Tenancy

Different tenants may need different policies:

```text
tax
pricing
approval
notification
```

A policy extension can support tenant variation.

But tenant-specific extensions increase:

```text
configuration complexity
test matrix
observability
security risk
```

---

# 86. Tenant Policy Registry

Conceptually:

```text
tenantId
  ↓
policy selection
  ↓
stable policy contract
```

Avoid leaking tenant-specific branching into every domain method.

---

# 87. OCP and Branching

Not every `if` violates OCP.

Example:

```ts
if (amount < 0) throw ...
```

This is a core invariant check, not a variation point.

OCP concerns **expected behavioral variation**, not all conditional logic.

---

# 88. Stable Validation vs Variant Policy

Keep stable validation:

```ts
quantity > 0
```

inside the stable domain contract.

Move variable policy:

```text
quantity limit differs by tenant
```

behind a policy abstraction if necessary.

---

# 89. OCP and Invariant Enforcement

Do not move invariant enforcement entirely into extensions.

The stable core should preserve invariants that are universally required.

---

# 90. OCP and Core Invariants

Example:

```text
Order.total >= 0
```

should remain a core invariant.

A pricing strategy may calculate the value.

The order still protects its valid state.

---

# 91. OCP and Extension Validation

The system should validate extension implementations.

At minimum:

```text
contract tests
startup validation
capability checks
failure isolation
```

---

# 92. Contract Test for Strategy

```ts
function taxPolicyContract(
  createPolicy: () => TaxPolicy
) {
  const policy = createPolicy();

  // assert stable semantic rules
}
```

Every implementation runs the same behavior suite.

---

# 93. Property-Based Extension Testing

For a policy:

```text
result must be non-negative
```

or:

```text
same normalized input produces deterministic result
```

Properties can validate all implementations.

---

# 94. OCP and Fuzzing

Extension boundaries are good fuzzing targets because implementations may receive broad inputs.

Test:

```text
large values
empty data
malformed data
boundary values
unexpected Unicode
timeouts
duplicate calls
```

---

# 95. OCP and Failure Isolation

An extension should fail within its responsibility boundary.

Example:

```text
recommendation plugin fails
→ checkout remains operational
```

when recommendations are non-critical.

---

# 96. OCP and Bulkheads

Separate extension resources:

```text
payment provider pool
notification provider pool
analytics provider pool
```

so one extension cannot exhaust shared capacity.

---

# 97. OCP and Timeouts

Each extension contract should define:

```text
timeout
cancellation
retry
fallback
```

Otherwise extensions can destabilize the core.

---

# 98. OCP and Observability

Extensions should expose enough metadata to identify:

```text
which implementation ran
duration
success/failure
retry count
fallback use
contract violations
```

---

# 99. OCP and Logging

Do not log every extension detail blindly.

Prefer structured data:

```text
extensionId
operation
result
duration
tenant-safe context
```

Avoid secrets.

---

# 100. OCP and Metrics

Measure extension behavior separately:

```text
payment_provider_latency
tax_policy_failures
notification_channel_errors
```

This lets operators compare implementations.

---

# 101. OCP and Distributed Services

A service API can be treated as an extension boundary.

Example:

```text
Payment service
```

may allow different:

```text
providers
routing policies
fraud policies
```

internally.

Do not move every variation into separate services.

---

# 102. Service-Level Extension

A service can remain stable while implementation changes behind the API:

```text
old database
→ new database
```

This is OCP at architectural scale.

---

# 103. OCP and Deployment Independence

Extension can permit:

```text
new provider implementation
```

without modifying the stable domain service.

But separate deployment adds:

```text
network contracts
versioning
operational ownership
```

---

# 104. OCP and Event Versioning

New event versions can extend capability without mutating historical event meaning.

Example:

```text
OrderCreated v1
OrderCreated v2
```

Consumers migrate independently when supported.

---

# 105. OCP and Schema Evolution

Additive schemas often support OCP:

```text
existing readers
+
new fields
```

But consumers must tolerate unknown fields.

---

# 106. OCP and Backward Compatibility

An extension mechanism is only useful if old consumers remain stable.

Therefore:

```text
extension
→ new implementation
→ compatibility contract
```

must be reviewed together.

---

# 107. OCP and Legacy Code

Legacy systems often have central switches.

Example:

```ts
switch (report.type) {
  case "sales":
  case "inventory":
  case "tax":
}
```

Do not immediately introduce abstract factories everywhere.

First measure:

```text
change frequency
branch complexity
consumer count
regression risk
```

---

# 108. Strangler-Style OCP Refactoring

Introduce:

```text
stable interface
        ↓
existing implementation adapter
```

Then add:

```text
new implementation
```

The old implementation remains valid during migration.

---

# 109. Branch by Abstraction

A practical sequence:

```text
legacy behavior
       ↓
abstraction
    ↙     ↘
legacy   new
```

After migration:

```text
new implementation
```

This uses OCP as a migration strategy.

---

# 110. Anti-Corruption Layer

External systems can be adapted into the stable core contract.

This protects domain logic from provider-specific variation.

---

# 111. OCP and Provider Lock-In

Bad:

```ts
OrderService
  directly uses ProviderSdkType
```

Better:

```text
OrderService
  → PaymentGateway
  → ProviderAdapter
  → Provider SDK
```

Provider changes become extension changes.

---

# 112. OCP and Data Access

Repository abstraction can hide storage variation.

But do not abstract provider details that the domain genuinely requires.

A stable abstraction must preserve important semantics.

---

# 113. Leaky Extension Boundary

A boundary is leaky when callers must know:

```text
provider error codes
provider pagination
provider retry rules
provider-specific IDs
```

The extension mechanism has failed to protect variation.

---

# 114. OCP and Semantic Translation

Adapters should translate:

```text
provider semantics
→ application semantics
```

not merely rename fields.

---

# 115. OCP and Error Contracts

Every extension must produce errors compatible with the stable contract.

Example:

```text
provider timeout
→ TransientPaymentFailure
```

not:

```text
AxiosError
```

leaking into domain callers.

---

# 116. OCP and Idempotency

Extensions performing side effects must preserve idempotency expectations.

If the stable contract says:

```text
retry-safe
```

every implementation must honor it.

---

# 117. OCP and Concurrency

A strategy implementation cannot silently weaken concurrency guarantees.

Example:

```text
InventoryPolicy
```

must not allow:

```text
negative stock
```

just because a new implementation was introduced.

---

# 118. OCP and Transaction Boundaries

An extension can change transaction requirements.

Before substituting an implementation, verify:

```text
atomicity
isolation
commit timing
rollback behavior
```

---

# 119. OCP and Performance Contracts

A new extension may be functionally correct but operationally unacceptable.

Example:

```text
new tax provider
  500 ms latency per calculation
```

If checkout requires:

```text
p95 < 100 ms
```

the implementation violates the broader operational contract.

---

# 120. OCP and Resource Contracts

Extension contracts may require:

```text
maximum response size
maximum memory
maximum call count
timeout
rate limits
```

---

# 121. OCP and Caching

Different extension implementations may have different cacheability.

The stable contract should specify whether callers may rely on:

```text
freshness
immutability
determinism
```

---

# 122. OCP and Determinism

A strategy may be expected to be deterministic:

```text
same input
→ same result
```

If randomness is allowed, make that explicit in the contract.

---

# 123. OCP and Time

Time-sensitive strategies should receive an explicit clock or context where deterministic behavior matters.

This avoids hidden extension differences caused by system time.

---

# 124. OCP and Randomness

Random algorithms should use explicit random sources when testing and reproducibility matter.

---

# 125. OCP and Feature Flags

Feature flags can select extensions:

```text
tenant A → Policy A
tenant B → Policy B
```

But too many flags become a distributed decision tree.

Centralize variation selection where possible.

---

# 126. OCP and Configuration Explosion

Bad:

```text
enableNewTax
enableNewTaxV2
enableLegacyTaxFallback
enableNewTaxByRegion
enableTaxExperiment
```

This is not clean OCP.

It is configuration complexity.

---

# 127. OCP and Strategy Selection

Selection itself is often a responsibility.

```ts
interface PricingPolicyResolver {
  resolve(context: PricingContext): PricingPolicy;
}
```

The resolver owns:

```text
which policy applies
```

The policy owns:

```text
how pricing is calculated
```

---

# 128. OCP and Factory Selection

A factory/resolver can isolate extension selection.

This is useful when selection rules are themselves cohesive.

---

# 129. OCP and Policy Composition

Some policies can compose:

```text
BasePricePolicy
→ DiscountPolicy
→ TaxPolicy
→ RoundingPolicy
```

But composition order becomes a contract.

---

# 130. Policy Pipeline

Example:

```text
base
→ discount
→ markup
→ tax
→ round
```

If order changes:

```text
discount before tax
vs
tax before discount
```

results may change.

Pipeline order must be explicit.

---

# 131. OCP and Decorator Ordering

For decorators:

```text
Retry(
  Logging(
    Gateway
  )
)
```

versus:

```text
Logging(
  Retry(
    Gateway
  )
)
```

can produce different observability.

Decorator ordering is part of the contract.

---

# 132. OCP and Middleware Ordering

Same principle:

```text
auth
before
cache
```

can differ from:

```text
cache
before
auth
```

Security-sensitive ordering must be explicit.

---

# 133. OCP and Security Review

Before exposing an extension point ask:

```text
Can the extension bypass authorization?
Can it read secrets?
Can it cross tenants?
Can it create side effects?
Can it exhaust resources?
```

---

# 134. OCP and Least Privilege

Pass minimal capabilities:

```ts
plugin = createPlugin({
  reportReader,
});
```

rather than:

```ts
plugin = createPlugin({
  fullApplicationContainer,
});
```

---

# 135. OCP and Tenant Isolation

Every extension that sees tenant data must preserve:

```text
tenantId(resource) === tenantId(context)
```

where required.

Extension points are common places for isolation bugs.

---

# 136. OCP and Audit

Extensions performing sensitive operations should generate appropriate audit evidence.

The core contract can require:

```text
operation identity
actor
tenant
timestamp
result
```

---

# 137. OCP and Compatibility

An extension API evolves too.

Potential strategies:

```text
add optional capability
version interface
add new adapter
introduce new port
deprecate old contract
```

---

# 138. Versioned Extension Contracts

Example:

```ts
interface PaymentGatewayV1 { ... }
interface PaymentGatewayV2 { ... }
```

Use versions when semantics truly change.

Do not version merely because implementations differ.

---

# 139. OCP and Default Implementations

Defaults can allow gradual adoption:

```ts
new CheckoutService({
  payment: defaultGateway,
});
```

New implementations can be injected when needed.

---

# 140. OCP and Type Compatibility

A structurally compatible implementation can still violate:

```text
error contract
idempotency
latency
security
```

TypeScript compatibility is not behavioral compatibility.

---

# 141. OCP and `satisfies`

Configuration objects can be checked against extension contracts:

```ts
const builtInPlugins = {
  payments: paymentPlugin,
  notifications: notificationPlugin,
} satisfies Record<string, Plugin>;
```

Compile-time conformance is useful.

Runtime behavior still requires testing.

---

# 142. OCP and `unknown`

Extensions that consume external data should use:

```ts
unknown
```

at the boundary and validate before using it.

Do not let extensions widen the trusted data surface indiscriminately.

---

# 143. OCP and Branded Types

Branded types can restrict extension contracts:

```ts
type PositiveQuantity =
  number & { readonly __brand: "PositiveQuantity" };
```

This helps communicate established domain concepts.

Runtime validation remains necessary.

---

# 144. OCP and Discriminated Unions

Extension states can use tagged unions:

```ts
type PaymentResult =
  | { kind: "approved"; transactionId: string }
  | { kind: "declined"; reason: string }
  | { kind: "retryable"; retryAfterMs: number };
```

This gives implementations a precise result vocabulary.

---

# 145. OCP and Exhaustiveness

A stable consumer can handle known variants exhaustively.

When a new variant is added, compiler errors can reveal consumers that need deliberate modification.

This is an important nuance:

```text
OCP is not “zero modification forever.”
```

Some semantic changes should intentionally modify consumers.

---

# 146. OCP and Algebraic Data Types

Sometimes the best model is a closed set of variants:

```ts
type PaymentStatus =
  | "pending"
  | "approved"
  | "declined";
```

A closed algebraic data type can be better than open polymorphism when the domain requires a finite, controlled set.

---

# 147. Open vs Closed World Modeling

This distinction is essential.

Open-world:

```text
new implementations are expected
```

Use:

```text
interfaces
strategies
plugins
registries
```

Closed-world:

```text
all variants should be known centrally
```

Use:

```text
unions
enums
exhaustive switches
```

---

# 148. OCP Does Not Mean Everything Must Be Open

Some domain concepts should remain intentionally closed.

Example:

```text
OrderState
```

may have a controlled finite state machine.

Allowing arbitrary external states could violate invariants.

---

# 149. Open Extension Boundary

A payment provider boundary is often open:

```text
new provider expected
```

An order lifecycle may be closed:

```text
DRAFT
CONFIRMED
PAID
SHIPPED
CANCELLED
```

This is architectural judgment.

---

# 150. OCP and Exhaustive Switches

This can be healthy:

```ts
switch (state.kind) {
  case "draft":
    return ...;
  case "confirmed":
    return ...;
}
```

when the domain intentionally requires every state to be known.

Do not replace every switch with polymorphism.

---

# 151. OCP and Strategy Explosion

Overusing strategies creates:

```text
StrategyFactory
StrategyResolver
StrategyRegistry
StrategyProvider
```

for trivial variation.

At that point abstraction cost exceeds change benefit.

---

# 152. OCP and Complexity Budget

An extension seam should earn its complexity.

Evaluate:

```text
expected future changes
+
consumer count
+
risk of modification
```

against:

```text
extra abstraction
+
indirection
+
testing
+
operational complexity
```

---

# 153. OCP and Premature Abstraction

Bad:

```ts
interface FuturePaymentProvider {
  ...
}
```

before there is any evidence of provider variation.

Design for known or strongly expected change.

---

# 154. OCP and Speculative Generality

A generic framework for 20 future variants can be worse than a simple implementation that is changed safely when the second variant actually arrives.

---

# 155. OCP and Refactoring

A healthy path is:

```text
simple implementation
→ observe variation
→ extract seam
→ stabilize contract
→ add implementation
```

not:

```text
imagine every future variation
→ build framework
```

---

# 156. Extract Interface Refactoring

Before:

```ts
class CheckoutService {
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

Then:

```ts
StripeGateway implements PaymentGateway
```

The interface is introduced at the real variation boundary.

---

# 157. Replace Conditional with Polymorphism

Before:

```ts
if (type === "A") return ruleA(input);
if (type === "B") return ruleB(input);
```

After:

```ts
interface Rule {
  apply(input: Input): Output;
}
```

Use when the variants have meaningful independent behavior.

---

# 158. Replace Conditional with Data

Sometimes:

```ts
const rates = {
  retail: 0.1,
  wholesale: 0.05,
};
```

is better than:

```ts
class RetailRule
class WholesaleRule
```

when the variation is purely data.

---

# 159. OCP and Table-Driven Design

Table-driven designs can be powerful:

```text
input category
→ configured value
```

They reduce code modification when new values are expected.

But validate input and configuration semantics.

---

# 160. OCP and Rule Ordering

Data-driven rules can still contain precedence:

```text
more specific rule
before
general rule
```

Ordering must be explicit to prevent accidental behavior.

---

# 161. OCP and Plugin Registration

Registration approaches:

```text
explicit code
configuration
dependency injection
module loading
dynamic discovery
```

Each has trade-offs.

---

# 162. Explicit Registration

```ts
registry.register("pdf", new PdfExporter());
```

Pros:

```text
easy to reason about
explicit dependencies
easy to test
```

Cons:

```text
requires central registration modification
```

Still may be the better design.

---

# 163. Dynamic Discovery

Plugins can register themselves.

Pros:

```text
less central modification
```

Cons:

```text
hidden dependencies
startup ordering
discovery failures
security concerns
```

Do not equate less modification with better design.

---

# 164. OCP and Dependency Visibility

A plugin mechanism that hides all implementations can make the system harder to understand.

Explicitness has value.

---

# 165. OCP and Build-Time Extensions

TypeScript libraries can expose extension interfaces:

```ts
export interface Serializer {
  serialize(value: DomainObject): string;
}
```

Consumers provide implementations.

The library core remains stable.

---

# 166. OCP and Runtime Plugins

Runtime plugins require stronger governance:

```text
version compatibility
sandboxing
timeouts
resource limits
failure isolation
```

---

# 167. OCP and Package Boundaries

Packages can be stable extension points:

```text
@company/payment-core
@company/payment-stripe
@company/payment-test
```

The core owns the contract.

Adapters own provider variation.

---

# 168. OCP and Monorepos

A monorepo can evolve extension packages without separate repositories.

But shared types can create accidental coupling.

Keep contracts focused.

---

# 169. OCP and Semantic Versioning

Changing an extension contract can break third-party implementations.

For a public library:

```text
interface evolution
```

is an API design concern.

Treat it like public API.

---

# 170. OCP and Deprecation

Deprecate extension contracts deliberately:

```text
old interface
→ adapter
→ new interface
```

This supports migration without forcing every consumer to upgrade immediately.

---

# 171. OCP and Documentation

An extension contract should document:

```text
required methods
behavior
errors
lifecycle
thread/concurrency assumptions
timeouts
resource limits
security
```

---

# 172. OCP and Examples

Provide at least one correct implementation.

A concrete example makes the extension semantics visible.

---

# 173. OCP and Contract Suites

Publish contract tests when consumers implement your interfaces.

Example:

```ts
export function paymentGatewayContract(
  factory: () => PaymentGateway
) {
  // shared behavior tests
}
```

This helps external implementations validate compatibility.

---

# 174. OCP and Consumer-Driven Contracts

Consumers can specify the subset of behavior they depend on.

Use carefully to avoid encoding accidental behavior.

---

# 175. OCP and Test Double Design

Test doubles should implement the same stable contract.

Avoid mocks that reproduce implementation details.

Prefer:

```text
fake gateway
contract-compliant stub
in-memory repository
```

when they improve semantic testing.

---

# 176. OCP and Integration Tests

A new extension implementation should pass:

```text
unit
contract
integration
performance
security
```

tests appropriate to its risk.

---

# 177. OCP and Mutation Testing

Mutation testing can reveal weak extension contracts.

If a provider implementation changes:

```text
retry behavior
error classification
amount handling
```

strong tests should detect the semantic break.

---

# 178. OCP and Property Testing

Properties should be written against the stable contract.

All extensions should satisfy them.

---

# 179. OCP and State Machines

If extensions implement state transitions, test that all valid transitions preserve core invariants.

---

# 180. OCP and Observability Contracts

Extension implementations should emit compatible telemetry fields where operational comparison matters.

---

# 181. OCP and Performance Testing

A new implementation may satisfy functional tests but fail latency or throughput contracts.

Performance validation belongs at important extension boundaries.

---

# 182. OCP and Memory Testing

An extension may leak memory even when functional behavior is correct.

Long-running plugins need resource testing.

---

# 183. OCP and Security Testing

Extension implementations should be checked for:

```text
tenant isolation
authorization
input validation
secret handling
output filtering
dependency vulnerabilities
```

---

# 184. OCP and Failure Injection

Test:

```text
timeout
exception
partial response
malformed response
duplicate call
cancellation
dependency outage
```

at the extension boundary.

---

# 185. OCP and Retry

The extension contract should state:

```text
which errors are retryable
whether retry is safe
who owns backoff
```

Avoid nested retry loops.

---

# 186. OCP and Circuit Breakers

A provider implementation can use a circuit breaker.

The outer contract should preserve stable failure semantics.

---

# 187. OCP and Fallbacks

Fallbacks are themselves extension policy.

Example:

```text
primary tax provider
→ cached policy
→ fail closed
```

The fallback order is contractual.

---

# 188. OCP and Optional Dependencies

Optional integrations should not destabilize mandatory workflows.

Example:

```text
recommendation extension fails
→ checkout succeeds
```

if recommendation is non-critical.

---

# 189. OCP and Graceful Degradation

Extension contracts should define degraded behavior:

```text
feature unavailable
→ explicit fallback
```

not:

```text
silent semantic corruption
```

---

# 190. OCP and Reliability

An open extension point should not become a single point of system instability.

Use:

```text
timeouts
bulkheads
circuit breakers
resource limits
```

where appropriate.

---

# 191. OCP and Operational Complexity

Every extension mechanism creates:

```text
registration
versioning
deployment
monitoring
testing
support
```

cost.

Count that cost before choosing architecture.

---

# 192. OCP and Developer Experience

Good extension APIs are:

```text
small
discoverable
typed
documented
easy to test
hard to misuse
```

An abstraction that is technically flexible but painful to implement is not a good extension boundary.

---

# 193. OCP and Error Messages

Extension implementers need stable error semantics.

Human-readable messages should not be the only contract.

Use typed errors or codes where appropriate.

---

# 194. OCP and Nullability

An extension should not randomly switch:

```text
null
undefined
exception
```

for absence.

Stable contract semantics must be preserved.

---

# 195. OCP and Serialization

Extension output must preserve serialization contracts.

Provider-specific objects should not escape stable APIs.

---

# 196. OCP and Identity

Extensions should preserve identity semantics.

Example:

```text
PaymentGateway
```

must not create a new payment identity on every retry unless the contract says so.

---

# 197. OCP and Equality

If extensions return value objects, equality semantics should remain stable across implementations.

---

# 198. OCP and Tenant Scope

Extension registries should consider tenant boundaries.

Avoid:

```text
tenant A accidentally selects tenant B's configuration
```

---

# 199. OCP and Caching

Extension identity can be part of cache keys when outputs differ:

```text
tenantId + extensionId + inputVersion
```

Avoid cross-implementation cache contamination.

---

# 200. OCP and Event Consumers

Adding an event consumer is additive structurally but not necessarily operationally.

Review:

```text
load
privacy
side effects
failure
ordering
```

---

# 201. OCP and Event Schema

Consumers should depend on stable facts rather than producer implementation details.

---

# 202. OCP and Versioned Plugins

A plugin runtime may require:

```text
plugin API version
capabilities
minimum host version
```

This is essential when extensions are independently released.

---

# 203. Compatibility Handshake

A plugin can declare:

```ts
type PluginManifest = {
  id: string;
  apiVersion: string;
  capabilities: readonly string[];
};
```

The host validates compatibility before activation.

---

# 204. OCP and Lifecycle Hooks

Plugins may need:

```text
initialize
start
stop
healthCheck
```

Do not treat lifecycle as incidental.

It is part of the contract.

---

# 205. Plugin Shutdown Contract

A plugin should release:

```text
connections
timers
workers
file handles
subscriptions
```

Failure to do so can create memory and reliability issues.

---

# 206. OCP and Resource Ownership

The host should define:

```text
who owns resources
who closes them
```

to avoid lifecycle leaks.

---

# 207. OCP and Hot Reloading

Hot-swapping extensions introduces:

```text
state migration
resource cleanup
concurrency
in-flight requests
```

Do not implement hot reload unless the operational value justifies complexity.

---

# 208. OCP and Dynamic Code Loading

Dynamic loading increases supply-chain and execution risks.

Use explicit trust boundaries.

---

# 209. OCP and Sandboxing Boundaries

For untrusted code:

```text
host
  ↓ capability boundary
sandbox/plugin
```

The boundary should restrict:

```text
filesystem
network
secrets
CPU
memory
processes
```

---

# 210. OCP and Supply Chain

Third-party extensions introduce dependency risk.

Review:

```text
provenance
versions
signatures where appropriate
security scans
permissions
update process
```

---

# 211. OCP and Backward-Compatible Extension

An extension contract should ideally evolve additively:

```text
new optional capability
```

rather than changing required semantics for all implementations.

---

# 212. OCP and Capability Interfaces

Instead of one broad interface:

```ts
interface Gateway {
  charge();
  refund();
  dispute();
}
```

use:

```ts
interface Charger {}
interface Refunder {}
interface DisputeReader {}
```

Extensions can implement only supported capabilities.

---

# 213. OCP and Adapter Capability

An adapter can expose multiple small capability interfaces around one provider.

This keeps consumers focused.

---

# 214. OCP and Dependency Graphs

Extension architecture should avoid:

```text
plugin A
→ core
→ plugin B
→ plugin A
```

Cycles undermine isolation.

---

# 215. OCP and Stable Core Dependencies

A plugin should generally depend on:

```text
stable core contract
```

rather than:

```text
other plugin internals
```

unless cross-plugin dependency is part of the architecture.

---

# 216. OCP and Extension Ordering

If multiple extensions run in sequence, define:

```text
ordering
precedence
short-circuit behavior
failure behavior
```

---

# 217. OCP and Interceptor Chains

Interceptor systems can be OCP-friendly:

```text
authorization
→ metrics
→ tracing
→ caching
→ handler
```

But chain ordering becomes part of architecture.

---

# 218. OCP and AOP-Like Designs

Aspect-oriented mechanisms can reduce repetitive modification.

But hidden control flow can make behavior harder to reason about.

Prefer explicit extension mechanisms when clarity matters.

---

# 219. OCP and Code Generation

Generated code can be an extension mechanism:

```text
schema
→ generated implementation
```

Ensure generated output remains behind stable contracts.

---

# 220. OCP and Reflection

Reflection can discover implementations dynamically.

Trade-offs:

```text
flexibility
vs
discoverability
vs
type safety
vs
startup complexity
```

---

# 221. OCP and Metadata

Metadata can support registration:

```ts
{
  id: "stripe",
  capabilities: ["charge", "refund"]
}
```

But metadata itself becomes a contract.

---

# 222. OCP and Type Erasure

TypeScript interfaces disappear at runtime.

If runtime plugin discovery needs interfaces, add explicit runtime metadata or schema validation.

---

# 223. OCP and Runtime Contracts

A plugin may satisfy:

```ts
interface Plugin
```

at compile time but be invalid when loaded dynamically.

Runtime validation is necessary across dynamic boundaries.

---

# 224. OCP and Dependency Injection Containers

DI containers can automate extension selection.

But a giant container can hide dependencies.

Use explicit registration where it improves reasoning.

---

# 225. OCP and Service Locators

Service locators make extension resolution easy:

```ts
locator.get(PaymentGateway)
```

but hide dependency relationships.

This can weaken design clarity.

---

# 226. OCP and Global Registries

Global registries can cause:

```text
test contamination
initialization ordering
hidden state
```

Prefer scoped registries when possible.

---

# 227. OCP and Test Isolation

Extension registration should be isolated per test:

```text
new registry
→ register fake
→ run test
→ discard registry
```

Avoid global mutable plugin registries.

---

# 228. OCP and Memory Leaks

Registries retaining extension objects can extend their lifetime unintentionally.

Review:

```text
subscriptions
timers
closures
cached instances
```

---

# 229. OCP and Event Subscription Leaks

A plugin subscribed to events must unsubscribe at shutdown.

Otherwise the host retains the plugin.

---

# 230. OCP and Weak References

Weak references can sometimes help lifecycle-sensitive registries.

But they add complexity and should not replace clear ownership.

---

# 231. OCP and Hot Path Performance

Extension dispatch usually adds an indirect call.

For I/O-bound application services, this is often small.

For hot computational loops, benchmark.

---

# 232. OCP and Inlineability

Highly dynamic dispatch can affect engine optimization.

Do not assume abstractions are free in performance-critical loops.

---

# 233. OCP and Allocation

Creating strategy objects per request may allocate unnecessarily.

Prefer reuse when lifecycle and thread-safety semantics permit.

---

# 234. OCP and Concurrency Safety

A shared extension instance must be safe under concurrent requests.

Document whether implementations are:

```text
stateless
request-scoped
stateful
thread/concurrency safe
```

---

# 235. OCP and Async Safety

Async extension methods may interleave operations.

Do not store request-specific mutable state on singleton extensions.

---

# 236. OCP and State Leakage

Bad:

```ts
class Provider {
  currentTenantId?: string;
}
```

shared across requests.

Use explicit parameters or safe request-scoped context.

---

# 237. OCP and Request Context

A stable extension contract can accept:

```ts
type RequestContext = {
  tenantId: TenantId;
  actorId: UserId;
  correlationId: string;
};
```

Only include context that is genuinely part of the contract.

---

# 238. OCP and Context Explosion

Do not pass:

```text
context: everything
```

when only two fields are needed.

A giant context object creates hidden coupling.

---

# 239. OCP and Version Negotiation

Long-lived plugins may need:

```text
host API v2
plugin supports v1-v2
```

Negotiate explicitly.

---

# 240. OCP and Feature Capability

Expose:

```text
supports("refund")
```

only when capability variability is real.

Do not force consumers to probe arbitrary features.

---

# 241. OCP and Strategy Configuration

Strategies can be selected by:

```text
tenant
region
request type
feature rollout
configuration
```

Selection logic should remain separate from the strategy algorithm.

---

# 242. OCP and Factory Resolution

A resolver can encapsulate selection:

```ts
class TaxPolicyResolver {
  resolve(context: TaxContext): TaxPolicy {
    // selection
  }
}
```

Then:

```text
Checkout
→ resolver
→ policy
```

---

# 243. OCP and Responsibility Separation

Resolver responsibility:

```text
which implementation
```

Policy responsibility:

```text
how behavior works
```

This aligns with SRP.

---

# 244. OCP and Dynamic Configuration

When configuration changes at runtime, consider:

```text
thread safety
cache invalidation
consistency
rollback
audit
```

An extension system can turn configuration updates into production behavior changes.

---

# 245. OCP and Safe Rollouts

New extensions can be rolled out:

```text
internal
→ canary
→ small tenant cohort
→ wider rollout
```

Feature selection becomes operationally significant.

---

# 246. OCP and Rollback

A stable extension point should allow:

```text
new implementation
→ revert to previous implementation
```

without data corruption.

This requires compatibility and state migration planning.

---

# 247. OCP and Data Compatibility

If a new extension stores different data, rollback may not be trivial.

Plan:

```text
schema compatibility
migration
dual read/write if needed
rollback path
```

---

# 248. OCP and Event Compatibility

A new implementation may emit different events.

The stable event contract must remain compatible unless a deliberate version transition occurs.

---

# 249. OCP and Side Effects

An extension can create new side effects.

The core should define:

```text
what side effects are permitted
```

and:

```text
which side effects are optional.
```

---

# 250. OCP and Observed Behavior

An extension may be functionally correct but observably different in:

```text
latency
errors
logging
metrics
```

Operational contracts matter.

---

# 251. OCP and Testing Matrix

For a high-risk extension:

```text
contract tests
+ integration tests
+ performance tests
+ security tests
+ failure tests
+ rollout tests
```

---

# 252. OCP and Consumer Stability

The strongest value of OCP appears when:

```text
many consumers
+
frequent variation
```

exist behind one stable boundary.

---

# 253. OCP and Single Consumer

With one consumer and one expected implementation, OCP may have little immediate value.

Avoid abstraction without evidence.

---

# 254. OCP and Many Consumers

When many consumers depend on one contract:

```text
new implementation
→ consumer code unchanged
```

becomes increasingly valuable.

---

# 255. OCP and Many Implementations

When multiple implementations already exist:

```text
A
B
C
```

the extension contract has evidence of reality.

This is often the right time to stabilize the abstraction.

---

# 256. OCP and Change Frequency

A stable but rarely changing policy may not justify an extension seam.

An unstable and high-risk policy may.

Use change history.

---

# 257. OCP and Volatility

A useful signal:

```text
high volatility
+
high blast radius
=
strong candidate for protected extension
```

---

# 258. OCP Decision Matrix

| Question | Low | High |
|---|---|---|
| Expected variants | simple code | extension candidate |
| Change frequency | modify directly | stabilize seam |
| Consumer count | few | many |
| Regression risk | low | high |
| Abstraction cost | high | reconsider |
| Contract clarity | weak | delay |

This is a heuristic, not a formula.

---

# 259. OCP Failure Mode — Over-Abstraction

Symptoms:

```text
interfaces everywhere
factories everywhere
registration everywhere
little actual variation
```

Result:

```text
more code
less clarity
```

---

# 260. OCP Failure Mode — Under-Abstraction

Symptoms:

```text
central switch grows
branch modifications frequent
regressions increase
provider details leak
```

Result:

```text
high change coupling
```

---

# 261. OCP Failure Mode — Fake Extension

Example:

```ts
interface Strategy {
  execute(input: any): any;
}
```

but every implementation requires different assumptions.

This is not a real stable contract.

---

# 262. OCP Failure Mode — Configuration Maze

Hundreds of flags can create a pseudo-extension framework that is harder to reason about than explicit implementations.

---

# 263. OCP Failure Mode — Leaky Abstraction

Consumers inspect:

```text
provider type
provider error
provider ID
```

to recover missing semantics.

---

# 264. OCP Failure Mode — Invariant Leakage

Every extension must manually preserve:

```text
stock >= 0
```

without core enforcement.

The stable core is too weak.

---

# 265. OCP Failure Mode — Version Lock

An extension point changes so frequently that implementations cannot keep up.

The contract itself is unstable.

---

# 266. OCP Failure Mode — Giant Base Interface

A huge interface makes extensions fragile.

Use focused capabilities.

---

# 267. OCP Failure Mode — Extension Framework Before Need

Building a plugin architecture before actual plugin requirements is speculative generality.

---

# 268. OCP Failure Mode — Too Much Runtime Magic

Dynamic discovery can reduce compile-time visibility and make failure modes harder to detect.

---

# 269. OCP Failure Mode — Hidden Registration

If a plugin registers itself through side effects, tests and startup behavior may become non-deterministic.

---

# 270. OCP Failure Mode — Global Mutable Registry

A global registry can leak state between tests and requests.

Scope it deliberately.

---

# 271. OCP Failure Mode — Unsafe Third Party

Untrusted extensions can become arbitrary-code execution or resource-exhaustion risks.

---

# 272. OCP Failure Mode — Ignoring Performance

An extension boundary may introduce overhead in a hot loop.

Measure before and after.

---

# 273. OCP Failure Mode — Ignoring Lifecycle

Plugins that retain timers, sockets, or event listeners can create leaks.

---

# 274. OCP Failure Mode — Ignoring Concurrency

Singleton extensions may accidentally share request-specific state.

---

# 275. OCP Failure Mode — Ignoring Observability

When multiple implementations exist, inability to distinguish them makes incidents harder to diagnose.

---

# 276. OCP Failure Mode — Ignoring Rollback

New extensions may write incompatible state that prevents safe rollback.

---

# 277. OCP Failure Mode — Breaking Consumer Assumptions

An implementation can change:

```text
null behavior
ordering
idempotency
latency
error category
```

while satisfying the same TypeScript interface.

---

# 278. Code Review Exercise — Payment Provider

```ts
class CheckoutService {
  async pay(order: Order, provider: string) {
    if (provider === "stripe") {
      // Stripe implementation
    }

    if (provider === "razorpay") {
      // Razorpay implementation
    }

    if (provider === "paypal") {
      // PayPal implementation
    }
  }
}
```

Identify:

```text
stable responsibility
variation axis
extension seam
selection responsibility
provider adapter responsibility
```

---

# 279. Code Review Exercise — Tax

```ts
function tax(order: Order, region: string) {
  if (region === "IN") return order.total * 0.18;
  if (region === "US") return order.total * 0.05;
  return 0;
}
```

Question:

```text
Should this become polymorphism?
```

Answer only after assessing:

```text
expected variation
number of rules
change frequency
consumer count
semantic complexity
```

---

# 280. Code Review Exercise — Stable Closed Set

```ts
type PaymentState =
  | "pending"
  | "approved"
  | "declined";
```

A developer proposes:

```ts
interface PaymentStateHandler {}
```

Ask:

```text
Does the domain need open extension?
```

If the state set is intentionally closed, the union may be better.

---

# 281. Code Review Exercise — Plugin Registry

```ts
const plugins = new Map();

export function register(name, plugin) {
  plugins.set(name, plugin);
}
```

Review:

```text
who may register?
when?
duplicate keys?
version compatibility?
lifecycle?
security?
testing?
```

---

# 282. Implementation — Strategy

```ts
interface DiscountPolicy {
  calculate(input: DiscountInput): Money;
}

class NoDiscountPolicy implements DiscountPolicy {
  calculate(input: DiscountInput): Money {
    return Money.zero(input.currency);
  }
}

class VipDiscountPolicy implements DiscountPolicy {
  calculate(input: DiscountInput): Money {
    return input.subtotal.multiply(0.10);
  }
}
```

The consumer can remain stable.

---

# 283. Implementation — Strategy Consumer

```ts
class CheckoutPricing {
  constructor(
    private readonly discount: DiscountPolicy
  ) {}

  calculate(input: DiscountInput) {
    return this.discount.calculate(input);
  }
}
```

The class is open to a new policy implementation.

---

# 284. Implementation — Function Extension

```ts
type DiscountCalculator =
  (input: DiscountInput) => Money;

function calculateCheckout(
  input: DiscountInput,
  discount: DiscountCalculator
) {
  return discount(input);
}
```

No class hierarchy is required.

---

# 285. Implementation — Registry

```ts
class ExporterRegistry {
  #exporters = new Map<string, ReportExporter>();

  register(
    id: string,
    exporter: ReportExporter
  ) {
    if (this.#exporters.has(id)) {
      throw new Error(`Exporter already registered: ${id}`);
    }

    this.#exporters.set(id, exporter);
  }

  get(id: string): ReportExporter {
    const exporter = this.#exporters.get(id);

    if (!exporter) {
      throw new Error(`Exporter not found: ${id}`);
    }

    return exporter;
  }
}
```

Notice that registry invariants are themselves part of the design.

---

# 286. Implementation — Adapter

```ts
interface PaymentGateway {
  charge(input: ChargeInput): Promise<ChargeResult>;
}

class ProviderAdapter implements PaymentGateway {
  constructor(
    private readonly provider: ProviderClient
  ) {}

  async charge(input: ChargeInput) {
    // translate provider semantics
    // return stable application result
  }
}
```

---

# 287. Implementation — Decorator

```ts
class RetryingGateway implements PaymentGateway {
  constructor(
    private readonly inner: PaymentGateway,
    private readonly retryPolicy: RetryPolicy
  ) {}

  charge(input: ChargeInput) {
    return this.retryPolicy.execute(
      () => this.inner.charge(input)
    );
  }
}
```

The decorator extends behavior without modifying the wrapped gateway.

---

# 288. Implementation — Capability Interfaces

```ts
interface Charger {
  charge(input: ChargeInput): Promise<ChargeResult>;
}

interface Refunder {
  refund(input: RefundInput): Promise<RefundResult>;
}
```

A provider can implement:

```ts
class Provider implements Charger, Refunder {}
```

A consumer depending only on `Charger` does not care whether refunds exist.

---

# 289. Implementation — Plugin Contract

```ts
type PluginManifest = {
  id: string;
  apiVersion: string;
  capabilities: readonly string[];
};

interface Plugin {
  manifest: PluginManifest;
  start(): Promise<void>;
  stop(): Promise<void>;
}
```

A real production plugin contract would require stronger lifecycle, error, security, and resource semantics.

---

# 290. Implementation — Runtime Guard

```ts
function isPlugin(value: unknown): value is Plugin {
  if (typeof value !== "object" || value === null) {
    return false;
  }

  const plugin = value as Record<string, unknown>;

  return (
    typeof plugin.start === "function" &&
    typeof plugin.stop === "function"
  );
}
```

Runtime validation protects dynamic extension boundaries.

---

# 291. Implementation — `satisfies`

```ts
const builtInPlugins = {
  payments: paymentPlugin,
  notifications: notificationPlugin,
} satisfies Record<string, Plugin>;
```

This checks compile-time conformance while preserving useful inference.

---

# 292. Implementation — Stable Error Contract

```ts
type ChargeResult =
  | {
      kind: "approved";
      transactionId: string;
    }
  | {
      kind: "declined";
      reason: string;
    }
  | {
      kind: "retryable";
      retryAfterMs: number;
    };
```

Every implementation must map provider behavior into this vocabulary.

---

# 293. Implementation — Contract Test

```ts
async function paymentGatewayContract(
  createGateway: () => PaymentGateway
) {
  const gateway = createGateway();

  const result = await gateway.charge({
    amount: 1000,
    currency: "INR",
    idempotencyKey: "test-key",
  });

  // assert stable result semantics
}
```

---

# 294. Debugging Exercise 1 — Provider Leakage

Symptom:

```text
Checkout catches StripeError.
```

Problem:

```text
provider-specific exception leaked through the stable contract.
```

Fix:

```text
ProviderAdapter
  → stable PaymentError
```

---

# 295. Debugging Exercise 2 — OCP Regression

A new provider is added by modifying:

```ts
CheckoutService
```

with another branch.

Ask:

```text
Is provider variation now a protected change axis?
```

If yes, extract the gateway contract.

---

# 296. Debugging Exercise 3 — Unsafe Strategy

A new:

```ts
TaxPolicy
```

returns negative tax.

The interface compiles.

The domain invariant fails.

Fix the core contract/invariant boundary; do not rely only on the extension author.

---

# 297. Debugging Exercise 4 — Plugin Leak

A plugin is removed, but memory usage grows.

Possible causes:

```text
event listeners
timers
open sockets
registry retention
closures
```

Review lifecycle contract.

---

# 298. Debugging Exercise 5 — Capability Confusion

Consumers repeatedly check:

```ts
if (gateway.refund) ...
```

The interface is too broad or capabilities are under-modeled.

Introduce separate capability contracts.

---

# 299. Debugging Exercise 6 — Configuration Explosion

A team creates:

```text
20 feature flags
```

to simulate extensions.

Ask whether a stable policy boundary would be simpler.

---

# 300. Debugging Exercise 7 — Closed Set Mis-modeled as Open

A developer creates plugins for:

```text
OrderState
```

which should be a centrally controlled finite state machine.

The extension model weakens business guarantees.

---

# 301. Debugging Exercise 8 — Hidden Selection Logic

Each caller independently decides:

```text
which provider to use
```

Selection responsibility is duplicated.

Extract a resolver/policy selector.

---

# 302. Predict-the-Output Exercise 1

```ts
interface Formatter {
  format(value: number): string;
}

const formatter: Formatter = {
  format(value) {
    return `[${value}]`;
  }
};

console.log(formatter.format(10));
```

### Prediction

```text
[10]
```

The extension can be supplied as a plain object because TypeScript uses structural typing.

---

# 303. Predict-the-Output Exercise 2

```ts
const strategies = {
  a: (x: number) => x + 1,
  b: (x: number) => x * 2,
};

console.log(strategies.a(3));
```

### Prediction

```text
4
```

A data structure can serve as an extension registry without classes.

---

# 304. Predict-the-Output Exercise 3

```ts
class Base {
  run(fn: () => number) {
    return fn();
  }
}

console.log(new Base().run(() => 5));
```

### Prediction

```text
5
```

Function injection is an OCP mechanism.

---

# 305. Predict-the-Output Exercise 4

```ts
const pipeline = [
  (x: number) => x + 1,
  (x: number) => x * 2,
];

console.log(pipeline.reduce((x, fn) => fn(x), 3));
```

### Prediction

```text
8
```

because:

```text
3 + 1 = 4
4 × 2 = 8
```

Extension order is observable behavior.

---

# 306. Predict-the-Output Exercise 5

```ts
const pipeline = [
  (x: number) => x * 2,
  (x: number) => x + 1,
];

console.log(pipeline.reduce((x, fn) => fn(x), 3));
```

### Prediction

```text
7
```

The extension pipeline is not commutative.

Ordering is part of the contract.

---

# 307. Mastery Exercise 1 — Payment Gateway

Design:

```text
PaymentGateway
StripeAdapter
AlternativeProviderAdapter
CheckoutUseCase
```

Requirements:

```text
stable errors
idempotency
timeouts
tenant safety
observability
contract tests
```

---

# 308. Mastery Exercise 2 — Tax Policy

Implement:

```text
DefaultTaxPolicy
RegionalTaxPolicy
```

Then decide whether region selection belongs in:

```text
resolver
policy
application service
configuration
```

Defend the decision.

---

# 309. Mastery Exercise 3 — Notification

Build:

```text
EmailChannel
SmsChannel
WhatsAppChannel
NotificationRouter
```

Then define:

```text
selection
retryability
ordering
fallback
rate limit
```

---

# 310. Mastery Exercise 4 — Report Exporters

Build:

```text
CsvExporter
PdfExporter
JsonExporter
```

Use a stable contract and registry.

Then test:

```text
duplicate registration
unknown format
large report
export failure
```

---

# 311. Mastery Exercise 5 — Jewellery ERP Pricing

Design extension points for:

```text
metal rate policy
making charge policy
discount policy
tax policy
rounding policy
```

Then identify which should actually be extensible.

Do not assume every policy dimension needs a separate abstraction.

---

# 312. Mastery Exercise 6 — Multi-Tenant Policies

Design:

```text
TenantPricingResolver
PricingPolicy
TenantContext
```

Requirements:

```text
tenant isolation
safe defaults
cache key isolation
auditability
rollout control
```

---

# 313. Mastery Exercise 7 — Plugin Lifecycle

Implement:

```text
initialize
start
healthCheck
stop
```

Then test:

```text
startup failure
shutdown failure
duplicate load
version mismatch
resource cleanup
```

---

# 314. Mastery Exercise 8 — Open vs Closed

For each domain concept, classify:

```text
open
or
closed
```

Examples:

```text
payment providers
notification channels
order states
currency set
export formats
authorization capabilities
```

Defend each classification.

---

# 315. Mastery Exercise 9 — Refactoring

Take a switch with five algorithms.

Determine:

```text
which branches are genuine variation
which are core invariant checks
which should become data
which should become strategy
which should remain a closed switch
```

---

# 316. Mastery Exercise 10 — Principal Defense

Explain:

```text
Why not use a strategy?
Why not use a switch?
Why not build a plugin registry?
Why not use configuration?
Why not use inheritance?
Why not split into microservices?
```

The correct answer depends on evidence and trade-offs.

---

# 317. Interview Question 1

**What does the Open/Closed Principle mean?**

> Design stable modules so expected new behavior can be added through intentional extension points without repeatedly modifying already-correct core behavior.

---

# 318. Interview Question 2

**Does OCP mean existing code can never change?**

No.

It means expected variation should be isolated so normal additions do not require invasive modification to stable code.

---

# 319. Interview Question 3

**How does OCP relate to SRP?**

SRP identifies coherent reasons to change.

OCP helps isolate expected variation within those responsibilities.

---

# 320. Interview Question 4

**Does every conditional violate OCP?**

No.

Invariant checks and intentionally closed finite state machines often require conditionals.

---

# 321. Interview Question 5

**When should you use polymorphism instead of a switch?**

When variants have meaningful independent behavior and are expected to evolve independently.

A closed, small set may be clearer as a switch or discriminated union.

---

# 322. Interview Question 6

**Is a factory automatically OCP-compliant?**

No.

A factory with a central switch still requires modification for every new variant.

---

# 323. Interview Question 7

**What is the relationship between OCP and LSP?**

OCP relies on extension implementations behaving according to the stable contract; LSP protects substitutability.

---

# 324. Interview Question 8

**Can functional programming follow OCP?**

Yes.

Functions, higher-order functions, tables, and composition can all create extension points.

---

# 325. Interview Question 9

**When is a closed-world design better?**

When the set of variants should be centrally controlled and exhaustive.

---

# 326. Interview Question 10

**Why can too much OCP be harmful?**

Because extension mechanisms add:

```text
abstraction
indirection
testing
documentation
lifecycle
operational
```

complexity.

---

# 327. Interview Question 11

**What is protected variation?**

A stable interface or boundary that shields the rest of the system from a volatile or uncertain change source.

---

# 328. Interview Question 12

**How does dependency injection support OCP?**

It allows the stable consumer to receive different implementations without hard-coding concrete dependencies.

---

# 329. Interview Question 13

**Why is a giant plugin interface bad?**

It forces implementations to understand unrelated capabilities and increases coupling.

---

# 330. Interview Question 14

**Can configuration satisfy OCP?**

Sometimes, when the variation is genuinely data-driven.

Configuration is not a substitute for arbitrary behavioral abstraction.

---

# 331. Interview Question 15

**What is a leaky extension boundary?**

One where consumers must know implementation-specific details to use the abstraction correctly.

---

# 332. Interview Question 16

**Why are contract tests important for OCP?**

Because compile-time conformance does not guarantee behavioral compatibility.

---

# 333. Interview Question 17

**What is the open/closed tension in TypeScript unions?**

Unions are excellent for closed sets but require deliberate consumer updates when new variants are added. Polymorphic interfaces support a more open set of implementations.

---

# 334. Interview Question 18

**Does OCP always reduce code modification?**

Not necessarily.

It reduces modification of stable core code for a particular class of expected extension.

---

# 335. Interview Question 19

**What is the biggest OCP mistake?**

Turning every possible change into an abstraction before evidence shows that the variation is real.

---

# 336. Interview Question 20

**What is the principal-level interpretation of OCP?**

> Protect high-risk, high-volatility change axes behind stable contracts, while keeping the extension mechanism no more complex than the expected value of future change justifies.

---

# 337. Principal Scenario — Jewellery ERP Payment

A jewellery ERP supports:

```text
UPI
card
bank transfer
cash
external gateway
```

Decide:

```text
Which parts should be open?
Which state transitions should remain closed?
Where does provider-specific logic live?
What is the stable contract?
```

A strong design may use:

```text
PaymentMethod
PaymentGateway
PaymentPolicy
```

without turning every payment state into a plugin.

---

# 338. Principal Scenario — Pricing

Suppose branches can configure:

```text
making charge
wastage
discount
tax
```

Ask:

```text
Which values are data?
Which are policy?
Which are stable invariants?
Which are tenant-specific?
Which require independent versioning?
```

This prevents a configuration maze.

---

# 339. Principal Scenario — Reporting

A system supports:

```text
PDF
CSV
Excel
```

Ask:

```text
Does format really vary often?
Do exports share one stable contract?
Should the registry be explicit?
Do large exports require async jobs?
```

OCP includes operational design.

---

# 340. Principal Scenario — Notifications

A business wants:

```text
email
SMS
WhatsApp
push
```

Ask:

```text
Are channels interchangeable?
Do all support the same delivery guarantees?
What capabilities differ?
Should retry semantics be shared?
Should templates be shared?
```

Do not force false uniformity.

---

# 341. Principal Scenario — Order State

A developer proposes external plugins for:

```text
DRAFT
CONFIRMED
PAID
SHIPPED
CANCELLED
```

Reject the design if business safety requires a closed state machine.

OCP is not a command to make safety-critical state open.

---

# 342. Principal Scenario — Payment Provider Migration

Current:

```text
Provider A
```

Target:

```text
Provider B
```

Use:

```text
PaymentGateway
  ↓
ProviderAdapter
```

Migrate behind the stable contract.

Preserve:

```text
idempotency
errors
amounts
currency
audit
reconciliation
```

---

# 343. Principal Scenario — Legacy Switch

A 1,500-line switch contains:

```text
20 report types
```

Do not blindly rewrite.

Measure:

```text
branch volatility
ownership
test coverage
regression rate
```

Extract only genuine independent variation.

---

# 344. Principal Scenario — Multi-Tenant Extension

Tenant A uses:

```text
PricingPolicyA
```

Tenant B:

```text
PricingPolicyB
```

Design:

```text
tenant-scoped resolver
+
stable policy contract
+
isolated cache
+
observability
```

The extension mechanism must preserve tenant isolation.

---

# 345. Principal Decision Framework

Evaluate any OCP proposal using:

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

Then answer:

```text
What variation are we protecting?
How likely is it?
How expensive is direct modification?
What contract can remain stable?
What complexity does the extension mechanism add?
```

---

# 346. OCP Design Scorecard

Score the proposal:

```text
Variation evidence
Change frequency
Regression risk
Consumer count
Contract clarity
Extension ergonomics
Testability
Security
Operational isolation
Rollback safety
```

Then compare against:

```text
abstraction cost
indirection
lifecycle cost
deployment cost
```

---

# 347. OCP Retrieval Drill

Without notes, explain:

```text
OCP
protected variation
strategy
adapter
registry
closed-world model
open-world model
leaky extension
extension contract
contract testing
```

Then provide a JavaScript or TypeScript example for each.

---

# 348. OCP Debugging Drill

Given a new feature request:

```text
“Add another payment provider.”
```

Immediately ask:

```text
What changes?
Where is provider variation currently located?
What contract exists?
What invariants must remain true?
What selection mechanism exists?
How will we test compatibility?
```

---

# 349. OCP Implementation Drill

Build the same extension three ways:

```text
1. switch
2. strategy
3. registry
```

Compare:

```text
complexity
testability
performance
discoverability
evolution
```

---

# 350. OCP Architecture Drill

Design:

```text
stable core
+
3 interchangeable providers
+
versioned contract
+
contract test suite
+
observability
+
safe rollout
```

Then identify where OCP stops being useful.

---

# 351. OCP Final Mental Model

Remember:

```text
stable core
    ↓
stable contract
    ↓
intentional extension seam
    ↓
new variation
```

But also:

```text
not every variation deserves an extension point
not every conditional is an OCP violation
not every set should be open
not every abstraction improves changeability
```

---

# 352. Final Master Rule

> **Protect important, expected variation behind the smallest stable contract that allows new behavior to be added without repeatedly destabilizing already-correct code.**

---

# 353. Completion Criteria

Do not mark this chapter mastered until you can:

- Define OCP precisely.
- Explain why “closed for modification” does not mean “never edit.”
- Distinguish expected from speculative variation.
- Explain protected variations.
- Apply OCP using composition and polymorphism.
- Know when a switch is better than polymorphism.
- Know when a closed union is better than an open interface.
- Use factories, registries, strategies, adapters, decorators, and middleware intentionally.
- Design extension contracts with behavioral semantics.
- Preserve invariants and contracts across implementations.
- Design secure plugin boundaries.
- Handle lifecycle, concurrency, performance, and observability concerns.
- Refactor legacy branching safely.
- Apply OCP to a jewellery ERP.
- Defend an OCP decision at principal level.

Status:

```text
[ ] Not Started
[~] In Progress
[?] Needs Revision
[+] Completed
[*] Mastered
```

---

# Chapter 20 — Key Takeaways

```text
OCP is about controlled extensibility.

Closed does not mean immutable.

Open does not mean infinitely configurable.

Expected variation should have intentional seams.

SRP identifies change responsibilities.

OCP protects important variation within those responsibilities.

LSP protects behavioral substitutability.

ISP keeps extension contracts focused.

DIP helps stable policy depend on abstractions.

Composition is often safer than inheritance for extension.

Not every conditional violates OCP.

Closed-world modeling is valuable for safety-critical finite sets.

Configuration is useful for data variation, not arbitrary behavior.

Plugin systems create security, lifecycle, versioning, and operational costs.

Extension contracts must preserve invariants, errors, idempotency, concurrency, and performance expectations.

The best OCP design uses the smallest extension mechanism justified by real change evidence.
```

---

# Chapter 20 — Concept Connections

Backward connections:

```text
Chapter 12 → Cohesion and coupling
Chapter 13 → Responsibility-driven design
Chapter 14 → GRASP
Chapter 15 → Cohesion-first decomposition
Chapter 16 → Coupling-first dependency design
Chapter 17 → Design smells and refactoring
Chapter 18 → Stable contracts and invariants
Chapter 19 → Single Responsibility Principle
```

Forward connections:

```text
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

# Chapter 20 — Dependency Graph

```text
Reason to Change
    ↓
Expected Variation
    ↓
Protected Variation
    ↓
Stable Contract
    ↓
Extension Seam
    ↓
Substitutability
    ↓
Composability
    ↓
Safe Evolution
```

---

# Chapter 20 — Revision / Retrieval Record

## Session Record

```text
Status:
[~] In Progress

Can define:
- OCP
- protected variation
- extension point
- open-world vs closed-world

Can compare:
- switch vs strategy
- configuration vs behavior
- registry vs explicit registration
- composition vs inheritance

Needs implementation:
- strategy
- registry
- adapter
- decorator
- plugin lifecycle
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

# Chapter 20 — Completion Snapshot

## Core Theory

```text
[+] OCP
[+] extension
[+] modification
[+] protected variation
[+] open-world modeling
[+] closed-world modeling
[+] change blast radius
[+] abstraction trade-offs
```

## Extension Mechanisms

```text
[+] polymorphism
[+] composition
[+] strategy
[+] factory
[+] registry
[+] adapter
[+] decorator
[+] middleware
[+] event consumers
```

## Production

```text
[+] security
[+] lifecycle
[+] concurrency
[+] performance
[+] observability
[+] versioning
[+] rollout
[+] rollback
[+] multi-tenancy
```

## Principal Judgment

```text
[+] avoid speculative abstraction
[+] protect high-value variation
[+] distinguish open and closed domains
[+] preserve contracts
[+] preserve invariants
[+] evaluate operational complexity
[+] defend trade-offs
```

---

# Chapter 20 — Canonical Source Discipline

When verifying this chapter:

```text
Design principle meaning
  → prioritize canonical design-principle sources and the repository's established framing

JavaScript / TypeScript mechanisms
  → ECMAScript and TypeScript documentation

Framework-specific extension systems
  → framework documentation

Database/runtime semantics
  → engine/runtime documentation

Security and operational behavior
  → environment-specific authoritative documentation

Domain rules
  → actual application requirements
```

Do not turn a simplified OCP slogan into an absolute engineering rule.

---

# Chapter 20 — Compact Mental Checklist

```text
□ What variation are we protecting?
□ Is the variation real or speculative?
□ What is the stable contract?
□ Is the domain open-world or closed-world?
□ Would a switch be clearer?
□ Would composition be simpler?
□ Does the extension preserve invariants?
□ Does it preserve error semantics?
□ Is idempotency preserved?
□ Are concurrency semantics preserved?
□ What lifecycle exists?
□ What resource limits exist?
□ What security authority does the extension receive?
□ How is the extension observed?
□ How is it tested?
□ How is it rolled back?
□ What complexity did we add?
```

---

# Chapter 20 — Standard Template Crosswalk

| Standard Area | Covered By |
|---|---|
| Learning Objectives | Section 1 |
| Prerequisites | Section 2 |
| What Is It? | Sections 3–12 |
| Why Does It Exist? | Sections 4–6 |
| Mental Model | Sections 10, 351 |
| Core Rules | Sections 13–52 |
| Syntax / APIs | Sections 282–293 |
| Basic Examples | Sections 33–40 |
| Execution / Runtime Considerations | Sections 220–236 |
| Advanced Behavior | Sections 53–277 |
| Edge Cases | Sections 140–150, 216–240 |
| Common Misconceptions | Sections 320, 337–345 |
| Common Mistakes | Sections 258–277 |
| Comparisons | Sections 20, 35, 147–159 |
| Performance | Sections 118–124, 231–236 |
| Memory | Sections 228–230, 233–236 |
| Security | Sections 82–85, 133–136, 208–210 |
| Production Usage | Sections 95–120, 170–236 |
| Implementation From Scratch | Sections 282–293 |
| Debugging | Sections 294–301 |
| Code Review | Sections 278–281 |
| Interview Questions | Sections 317–336 |
| Predict Output | Sections 302–306 |
| Mastery | Sections 307–316 |
| Completion | Section 353 |
| Key Takeaways | Key Takeaways section |
| Concept Connections | Concept Connections section |
| Revision Record | Revision / Retrieval Record |

---

# Chapter 20 — Mastery Gate

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

Reading alone does not mean mastery.

You should be able to explain not only **how** OCP is implemented, but **why the boundary exists, what variation it protects, and when not to use it**.

---
