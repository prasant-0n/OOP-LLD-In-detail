# Chapter 033 — TypeScript Runtime Validation, Schema Design, and Compile-Time/Runtime Contract Alignment

> **Role:** Principal JavaScript/TypeScript Engineer · ECMAScript Specialist · Runtime & LLD Architect  
> **Focus:** runtime trust boundaries, validation, schemas, codecs, domain construction, contract evolution, security, performance  
> **Status:** `[~] In Progress`

---

## Chapter Purpose

This chapter closes a critical gap between advanced TypeScript's static world and the runtime world where real systems receive data. It treats runtime validation as an architectural boundary rather than a collection of helper functions.

The central question is:

> **How do we turn an untrusted runtime value into a value that the rest of an object-oriented system can safely treat as a specific TypeScript/domain type?**

The answer requires coordination among the compiler, JavaScript runtime, transport contracts, domain invariants, authorization, and operational concerns.

---

## 1. Learning Objectives

By the end of this chapter, you should be able to explain the boundary between TypeScript's compile-time model and JavaScript's runtime values; design explicit runtime validation pipelines; distinguish type annotations, assertions, predicates, schemas, codecs, and generated contracts; model external data as `unknown`; preserve domain invariants after validation; compare schema-first, type-first, and contract-first approaches; and defend the trade-offs of runtime validation architecture in production systems.

Mastery follows:

`Understand → Explain → Predict → Implement → Debug → Apply → Compare → Defend`

A reader who can only repeat definitions has not completed this chapter. The mastery gate requires implementation, failure diagnosis, boundary design, and trade-off defense.

## 2. Prerequisites

You should already understand JavaScript values, objects, prototypes, classes, encapsulation, inheritance, composition, contracts, TypeScript structural typing, classes, generics, decorators, function types, control-flow narrowing, modules, and advanced type-level programming.

This chapter does not replace earlier chapters. It connects them into a single operational rule:

> Static types describe what your program assumes; runtime validation establishes what the program is actually allowed to assume.

This distinction matters because TypeScript's type information is primarily a compile-time facility and ordinary type annotations do not become runtime validators.

## 3. What Is It?

TypeScript can reject many invalid programs before execution, but a running process still receives values from HTTP requests, JSON, environment variables, databases, queues, files, browser storage, third-party SDKs, and untyped JavaScript.

An external value is not trustworthy because a TypeScript declaration says it has a type.

The boundary problem is therefore:

```text
unknown bytes / values
        ↓
parse / decode
        ↓
validate structure
        ↓
validate semantics
        ↓
construct trusted domain value
        ↓
business logic
```

The most important architectural idea is not “use a validation library.” It is to make the trust transition explicit.

## 4. Why Does It Exist?

TypeScript analyzes source code and emits JavaScript. A type annotation such as `const age: number` does not cause JavaScript to inspect an incoming object at runtime.

That means these are different questions:

1. What does the compiler believe?
2. What value actually exists?
3. What invariants has the application established?
4. What code is allowed to consume the resulting value?

Conflating them produces unsafe boundaries.

The practical consequence is simple:

```ts
const raw: unknown = await request.json();
// raw is still unknown here.
```

The correct next step is a runtime check or decoder. The unsafe shortcut is:

```ts
const raw = (await request.json()) as UserPayload;
```

The assertion changes the compiler's view, not the runtime object.

## 5. Mental Model

Use three layers when reasoning about TypeScript systems:

```text
World A — Source/type world
    interfaces, aliases, generics, unions, mapped types

World B — Runtime JavaScript world
    objects, strings, numbers, functions, bytes

World C — Contract/integration world
    HTTP, JSON, SQL, queues, env, files
```

A robust system defines explicit transitions:

```text
C → B     parse/decode
B → trusted application type   validate/refine
trusted type → domain object   construct/invariant-check
domain object → integration representation   serialize/encode
```

A type alias lives primarily in World A. A JSON object lives in World B. An OpenAPI document lives in World C. A good boundary architecture makes the mapping among those worlds deliberate.

## 6. Core Rules

Keep these terms separate.

**Annotation** tells the compiler how a value is intended to be used.

**Assertion** tells the compiler to trust the programmer's claim.

**Type guard** is executable code whose return type helps TypeScript narrow a value.

**Assertion function** throws or otherwise fails when an invariant is not satisfied and can communicate a narrowed type.

**Schema** is an executable description of accepted runtime data.

**Decoder** converts an input representation into a typed value, often returning structured failures rather than throwing.

**Codec** describes both directions: decode external representation and encode an internal representation.

**Parser** usually answers whether an input can be interpreted according to a grammar or representation.

**Validator** checks constraints against a value.

**Refinement** turns a broader static type into a narrower one after an established runtime condition.

**Contract** describes an agreement between producers and consumers.

A mature design treats these concepts as related but not interchangeable.

## 7. Syntax

A production boundary should usually have five distinct stages:

```text
Receive
  ↓
Parse
  ↓
Validate
  ↓
Normalize / construct
  ↓
Use
```

Example:

```ts
type CreateOrderInput = {
  customerId: string;
  quantity: number;
};

function isCreateOrderInput(value: unknown): value is CreateOrderInput {
  if (typeof value !== "object" || value === null) return false;

  const record = value as Record<string, unknown>;

  return (
    typeof record.customerId === "string" &&
    Number.isInteger(record.quantity) &&
    record.quantity > 0
  );
}
```

The important part is not the exact guard. It is that `unknown` remains outside the trusted core until executable evidence exists.

## 8. Basic Examples

`unknown` is often the correct static type for data whose shape has not yet been established.

Unlike `any`, `unknown` prevents arbitrary property access until the value is narrowed. TypeScript's documentation explicitly distinguishes the two: `unknown` requires checking before use, while `any` permits unchecked operations. citeturn746530search1

Preferred:

```ts
function processMessage(input: unknown) {
  if (typeof input === "string") {
    return input.trim();
  }

  throw new Error("Expected string");
}
```

Dangerous:

```ts
function processMessage(input: any) {
  return input.user.profile.permissions.admin;
}
```

The second example moves uncertainty into every consumer. The first localizes uncertainty at the boundary.

## 9. Execution Walkthrough

TypeScript understands many JavaScript runtime checks as type guards. `typeof`, `instanceof`, `in`, equality checks, control-flow reachability, and user-defined predicates can narrow values. citeturn746530search0

Example:

```ts
function readId(value: unknown): string {
  if (
    typeof value === "object" &&
    value !== null &&
    "id" in value &&
    typeof value.id === "string"
  ) {
    return value.id;
  }

  throw new Error("Invalid value");
}
```

The runtime check is evidence. The resulting narrowed type is the compiler's model of that evidence.

Remember: a user-defined predicate can lie.

```ts
function isUser(value: unknown): value is User {
  return true;
}
```

The compiler accepts the signature. Runtime correctness depends on the implementation.

## 10. Internal Mechanics

This code:

```ts
const user = value as User;
```

does not inspect `value`.

It is useful when the program has evidence outside the compiler's current knowledge, but it should be treated as an escape hatch.

A validation function should establish facts:

```ts
function assertUser(value: unknown): asserts value is User {
  if (!isUser(value)) {
    throw new Error("Invalid user");
  }
}
```

The assertion function still depends on the implementation of `isUser`. The type system is not independently verifying the runtime claim.

A useful review question is:

> What evidence makes this assertion true at this exact line?

## 11. ECMAScript / Specification Semantics

There are three broad strategies.

**Type-first**: write TypeScript types first and separately implement runtime validation.

Pros: excellent developer ergonomics and local static design.

Cons: the static type and runtime validator can drift.

**Schema-first**: define an executable schema first and infer or derive TypeScript types.

Pros: one executable source of truth for many runtime constraints.

Cons: framework/library coupling and sometimes less expressive type-level modeling.

**Contract-first**: define an external contract such as OpenAPI, JSON Schema, protobuf, or GraphQL, then generate runtime/client/server artifacts.

Pros: strong cross-language and organizational alignment.

Cons: generated-code lifecycle, versioning, and build complexity.

None is universally correct. The architectural question is where you want the source of truth to live.

## 12. Advanced Behavior

A TypeScript interface exists in the type namespace:

```ts
interface Customer {
  id: string;
}
```

There is no corresponding runtime constructor merely because the interface exists.

A class has both a type-side and value-side presence:

```ts
class Customer {
  constructor(public readonly id: string) {}
}
```

This can make `instanceof Customer` meaningful when the runtime value really is an instance of that class.

But JSON transport destroys class identity. A deserialized object is ordinarily just an object.

Therefore this is a common mistake:

```ts
const customer = JSON.parse(text) as Customer;
console.log(customer instanceof Customer); // false
```

The assertion did not construct a `Customer`.

## 13. Edge Cases

Transport data is often structurally shaped data:

```ts
type CustomerDto = {
  id: string;
  creditLimit: number;
};
```

A domain object may have stronger invariants and behavior:

```ts
class Customer {
  #creditLimit: number;

  constructor(
    public readonly id: string,
    creditLimit: number,
  ) {
    if (creditLimit < 0) {
      throw new Error("Credit limit cannot be negative");
    }

    this.#creditLimit = creditLimit;
  }

  get creditLimit() {
    return this.#creditLimit;
  }
}
```

The DTO is not automatically the domain object.

A robust design may therefore use:

```text
external payload
   ↓ validate
validated DTO
   ↓ map
domain factory/constructor
   ↓
trusted domain object
```

This separation prevents transport quirks from silently becoming business invariants.

## 14. Common Misconceptions

Do not force every boundary responsibility into one function.

**Parse** converts a representation into a value.

```ts
JSON.parse(text)
```

**Validate** determines whether the value satisfies a contract.

```ts
isCreateOrderInput(value)
```

**Normalize** converts equivalent accepted representations into one canonical representation.

```text
"  CUST-42 " → "CUST-42"
```

**Construct** establishes domain invariants and object identity.

A useful pipeline is:

```text
representation
 → parse
 → structural validation
 → normalization
 → semantic validation
 → domain construction
```

Mixing them can make failures ambiguous and tests brittle.

## 15. Common Mistakes

JSON defines a data representation, not your application's full domain semantics.

JSON can represent objects, arrays, strings, numbers, booleans, and null. It does not inherently express concepts such as:

- positive integer quantity
- branch identifier belonging to the authenticated tenant
- SKU that exists in inventory
- currency code supported by the business
- timestamp that falls within an allowed period

Those are application-level constraints.

Therefore:

```ts
type Quantity = number;
```

does not mean:

```text
quantity ∈ positive integers
```

The runtime boundary must establish the stronger rule if the business requires it.

## 16. Comparison

Structural validation asks:

```text
Is `quantity` a number?
Is `customerId` a string?
Is `items` an array?
```

Semantic validation asks:

```text
Is quantity positive?
Does customer exist?
Does branch own the customer?
Is currency allowed for this tenant?
Is the order state transition legal?
```

A schema can often handle structural and local semantic constraints. Cross-entity semantic rules usually belong to domain/application services because they require state, authorization, or policy.

Do not put database-dependent business rules into a transport schema simply because it is technically possible.

## 17. Performance

Model trust as increasing, not binary:

```text
T0 — Unknown
T1 — Parsed
T2 — Structurally validated
T3 — Normalized
T4 — Locally semantically valid
T5 — Authorized for actor/context
T6 — Domain-valid aggregate
T7 — Persisted system fact
```

The exact levels vary by application.

The value of the model is architectural clarity. A function should state what level of trust it expects.

For example:

```ts
function priceOrder(order: ValidatedOrder): Money
```

communicates a stronger precondition than:

```ts
function priceOrder(order: unknown): number
```

Types can represent some trust levels. Some trust levels require runtime evidence or contextual checks.

## 18. Memory

Branding can make certain validated values harder to misuse.

```ts
declare const CustomerIdBrand: unique symbol;

type CustomerId = string & {
  readonly [CustomerIdBrand]: "CustomerId";
};
```

A constructor can establish the brand:

```ts
function parseCustomerId(value: unknown): CustomerId {
  if (typeof value !== "string" || !/^CUST-[0-9]+$/.test(value)) {
    throw new Error("Invalid customer id");
  }

  return value as CustomerId;
}
```

The brand has no automatic runtime enforcement. It is useful only because application code controls creation of the branded value.

The rule becomes:

> Never accept an untrusted primitive where a validated domain-specific primitive is required.

## 19. Security

When a domain list is both a runtime validation set and a static union, prefer deriving one from the other.

```ts
const permissions = [
  "orders.read",
  "orders.write",
  "orders.cancel",
] as const;

type Permission = typeof permissions[number];
```

Runtime:

```ts
function isPermission(value: unknown): value is Permission {
  return typeof value === "string" &&
    (permissions as readonly string[]).includes(value);
}
```

Benefits:

- less duplication
- safer refactoring
- discoverable runtime source
- static union generated from the same values

Costs:

- the runtime collection becomes part of your domain API
- large sets may need more efficient lookup structures

## 20. Production Usage

For security-sensitive validation, distinguish own properties, inherited properties, and optional fields.

A careful baseline:

```ts
function isRecord(value: unknown): value is Record<string, unknown> {
  return typeof value === "object" && value !== null;
}
```

But `Record<string, unknown>` is a convenient static view, not a proof that the object has only string-keyed own properties.

Prototype-aware code should consider:

```ts
Object.hasOwn(value, "id")
```

rather than assuming `"id" in value` has the exact same security and semantic meaning.

This distinction matters when data may contain unusual prototypes or objects constructed by other code.

## 21. Implementation From Scratch

Security-sensitive validation should not blindly merge untrusted objects into application state.

Dangerous patterns include uncontrolled deep merges, dynamic property assignment, and assuming ordinary prototypes.

Prefer explicit field selection:

```ts
const input = raw as Record<string, unknown>;

const command = {
  customerId: input.customerId,
  quantity: input.quantity,
};
```

rather than:

```ts
const command = {
  ...input,
};
```

The second form may accidentally carry fields the domain never intended to accept.

Boundary validation should answer both:

```text
What is allowed?
What is forbidden?
```

Allow-listing fields is often easier to reason about than attempting to sanitize arbitrary input.

## 22. Debugging Exercises

Unknown keys can be handled in three legitimate ways.

**Reject unknown keys**: strict contracts, useful for configuration and security-sensitive commands.

**Strip unknown keys**: tolerant ingress, useful when producers may evolve independently.

**Preserve unknown keys**: forwarding/proxy scenarios where information must survive.

Do not choose based on taste. Choose based on compatibility and risk.

For commands such as:

```ts
UpdateUserCommand
```

rejecting unknown fields can catch client bugs and mass-assignment attempts.

For an API gateway forwarding vendor extensions, preserving unknown fields may be required.

## 23. Code Review

These states are not automatically interchangeable.

For a property:

```ts
type Input = {
  nickname?: string;
};
```

the key may be missing.

For:

```ts
type Input = {
  nickname: string | null;
};
```

the key must exist but may carry `null`.

For:

```ts
type Input = {
  nickname?: string | null;
};
```

the key may be missing or present with `string` or `null`.

Runtime schemas should make this distinction intentional because PATCH semantics often depend on it:

```text
missing     → do not change
null        → clear the value
string      → replace the value
```

Flattening all three into “optional” can create data-loss bugs.

## 24. Interview Questions

A string that looks like a timestamp is not automatically a valid domain time.

Validation may need to check:

- syntax
- timezone assumptions
- calendar validity
- allowed range
- business timezone
- whether the value represents an instant or a local civil time

Do not turn every date string into `Date` at the transport boundary without deciding which semantic the field represents.

For example:

```text
"2026-09-13"  → local business date
"2026-09-13T10:00:00Z" → instant
"10:00" → local wall-clock time
```

These are different domain concepts even if all can be stored as strings.

## 25. Predict-the-Output

JavaScript's `number` is broader than many domain concepts.

A valid finite number may still be:

- fractional when an integer is required
- negative when only positive values are allowed
- `NaN`
- `Infinity`
- outside a safe integer range

Therefore:

```ts
function isSafeNonNegativeInteger(value: unknown): value is number {
  return typeof value === "number" &&
    Number.isSafeInteger(value) &&
    value >= 0;
}
```

For money, consider whether binary floating-point numbers are the right representation at all. A domain may need integer minor units or a decimal library.

## 26. Mastery Exercises

`string` can represent thousands of different concepts.

A robust boundary may need:

```text
non-empty
bounded length
Unicode-aware normalization
allowed character set
canonical form
semantic format
```

For identifiers, case sensitivity and normalization should be explicit.

For user-controlled text, validation is not a replacement for context-specific output encoding. HTML, SQL, shell, logs, and headers have different injection models.

Runtime type validation answers “is this structurally acceptable?” Security encoding answers “is this safe for this output context?”

## 27. Key Takeaways

An array-level check is not an element-level proof.

```ts
Array.isArray(value)
```

only establishes that the value is an array.

You still need to validate each member:

```ts
function isStringArray(value: unknown): value is string[] {
  return Array.isArray(value) &&
    value.every(item => typeof item === "string");
}
```

Consider collection semantics too:

```text
order matters?
duplicates allowed?
maximum size?
empty allowed?
sorted?
unique by identity or value?
```

Those rules belong to the contract, not merely to the container type.

## 28. Concept Connections

Nested API data often forms recursive structures:

```ts
type Category = {
  id: string;
  children: Category[];
};
```

Static recursive types are straightforward. Runtime validation requires recursive execution.

A recursive validator should be designed for:

- stack depth
- cycle detection
- maximum input size
- maximum nesting depth
- meaningful error paths

For hostile input, a validator that recursively descends without limits can itself become a denial-of-service vector.

## 29. Completion Criteria

A validation failure is an API design problem, not merely a boolean.

Useful error information can include:

```text
path
code
expected
received category
message
source
```

Example:

```ts
type ValidationIssue = {
  path: readonly (string | number)[];
  code: string;
  message: string;
};
```

Avoid leaking raw internal object contents into production error messages.

Separate:

```text
developer diagnostics
consumer-safe error response
security-sensitive internal context
```

The domain may need error codes such as:

```text
INVALID_CUSTOMER_ID
INVALID_QUANTITY
UNKNOWN_FIELD
UNSUPPORTED_CURRENCY
```

Stable codes are often more useful for clients than fragile human-readable strings.

## 30. Error Paths as Data

Nested paths should be data, not only strings.

```ts
type PathSegment = string | number;

type ValidationIssue = {
  path: readonly PathSegment[];
  code: string;
};
```

For:

```json
{
  "items": [
    { "sku": "A", "quantity": -1 }
  ]
}
```

a useful path is:

```ts
["items", 0, "quantity"]
```

This can later be rendered as:

```text
items[0].quantity
```

Keeping paths structured makes localization, UI mapping, machine processing, and logging easier.

## 31. Fail-Fast vs Error Accumulation

Two common strategies:

**Fail fast**

```text
first invalid field → stop
```

Pros: lower work, simple control flow, suitable for internal commands.

**Accumulate issues**

```text
inspect all fields → return all known failures
```

Pros: better user experience for forms and batch imports.

The trade-off is not just performance. Error accumulation may require traversing more attacker-controlled data and therefore needs input-size limits.

## 32. Synchronous vs Asynchronous Validation

A validator is synchronous when all rules are local:

```ts
isPositiveInteger
isEmailShape
isKnownEnumValue
```

An asynchronous validator is appropriate when it needs I/O:

```text
does customer exist?
does SKU belong to branch?
is invoice number already used?
```

Do not hide database access inside a predicate named `isX`.

A useful split is:

```text
schema validation
     ↓
application checks
     ↓
domain invariants
```

This keeps transport validation predictable and prevents accidental N+1 queries during object decoding.

## 33. Validation Is Not Authorization

A payload can be perfectly valid and still forbidden.

```text
POST /orders
{
  "customerId": "CUST-10",
  "branchId": "BR-99"
}
```

Structural validation can prove both IDs are strings.

It does not prove the authenticated actor may operate on branch `BR-99`.

Authorization requires context:

```text
actor
tenant
branch
resource
action
policy
```

Keep the distinction:

```text
valid data ≠ permitted action
```

## 34. Multi-Tenant and Branch-Aware Boundaries

In multi-tenant systems, one of the most dangerous mistakes is trusting tenant or branch identifiers supplied by the caller.

A request may contain:

```ts
type CreateOrderInput = {
  branchId: string;
};
```

But the authoritative branch context may come from:

```text
authenticated session
token claims
request context
server-side routing
```

A safer design resolves or verifies the effective tenant/branch in application policy rather than treating client-provided tenancy fields as authoritative.

The runtime validator establishes shape. The authorization layer establishes ownership and scope.

## 35. DTO → Command → Domain Object

A clean application architecture often separates:

```text
DTO
  ↓
Command
  ↓
Application service
  ↓
Domain aggregate
```

The DTO is transport-shaped.

The command is application-intent-shaped.

The aggregate is domain-invariant-shaped.

Example:

```ts
type CreateOrderRequest = {
  customerId: string;
  items: Array<{
    sku: string;
    quantity: number;
  }>;
};

type CreateOrderCommand = {
  customerId: CustomerId;
  items: ReadonlyArray<{
    sku: Sku;
    quantity: PositiveInt;
  }>;
};
```

The command can only be built after validation/refinement.

## 36. Schema-to-Type Inference

Many validation systems allow a runtime schema to drive static types.

Conceptually:

```ts
const UserSchema = object({
  id: string(),
  age: integer(),
});

type User = Infer<typeof UserSchema>;
```

This is attractive because the runtime and compile-time representations have a shared source.

But inferred types should not be treated as proof that the schema encodes every business rule. A schema might validate shape while domain policy still lives elsewhere.

The best architecture is often:

```text
schema = transport/runtime contract
domain types = business semantic contract
```

with an explicit mapping between them.

## 37. Generated Types and Drift

Generated types are valuable when an external contract is authoritative.

Examples include:

```text
OpenAPI → client/server types
JSON Schema → validators/types
protobuf → generated classes/types
GraphQL → generated operation types
SQL schema → data access types
```

Generated artifacts introduce another system boundary:

```text
source contract
   ↓ generation
generated code
   ↓ compile
application
```

The risk is stale generation.

CI should make “generated files are up to date” a verifiable property, not a tribal convention.

## 38. OpenAPI and Runtime Contract Alignment

OpenAPI can describe request and response shapes for HTTP APIs. The architecture challenge is preventing documentation, implementation, and runtime validation from diverging.

Possible models:

```text
OpenAPI → generate validators/types
```

or:

```text
runtime schema → generate OpenAPI
```

The important property is one authoritative source plus an automated generation path.

Manual duplication is acceptable only when the cost of synchronization is understood and controlled.

## 39. JSON Schema and Executable Validation

JSON Schema describes JSON-compatible structures and constraints and can be used by validators to enforce them at runtime.

A TypeScript type can be richer than what JSON Schema naturally expresses, and JSON Schema can be consumed by languages that do not know TypeScript.

This makes JSON Schema useful at organizational boundaries.

Treat it as a contract language, not as a replacement for domain modeling. Cross-field rules, authorization, database state, and process invariants may remain outside the schema.

## 40. Code Generation as a Contract Strategy

Generated artifacts can reduce duplicated handwritten code, but generation changes the operational model.

You need:

```text
versioned source contract
deterministic generation
reviewable output
CI drift check
compatibility policy
rollback story
```

A generated client that silently changes when a remote contract changes can be more dangerous than handwritten code because developers may not review every semantic change.

The principal-level question is not “can we generate it?” but:

> Does generation reduce total system complexity over the lifetime of the contract?

## 41. Versioned Contracts

External contracts evolve.

Common compatibility strategies include:

```text
additive fields
optional fields
versioned endpoints
versioned event schemas
consumer-driven compatibility tests
dual-read / dual-write migrations
```

A runtime validator should match the intended compatibility window.

For example, a consumer that must tolerate older producers may intentionally allow unknown fields while a command endpoint for an administrative operation may reject them.

## 42. Input vs Output Contracts

Do not assume request and response policies are identical.

Input:

```text
tolerant where safe
strict where ambiguity is dangerous
```

Output:

```text
stable
deliberate
versioned
```

An API may accept legacy fields for compatibility but never emit them.

Similarly, database records may contain fields that must never cross a public response boundary.

## 43. Serialization Boundaries

Serialization can erase runtime identity and semantics.

Examples:

```text
class instance → JSON → plain object
Date → string
Map → ordinary JSON object or custom representation
BigInt → custom encoding required
private fields → not equivalent to serialized state
```

Therefore deserialization is not the inverse of object construction unless the format explicitly encodes enough information.

A robust serializer has an explicit output contract and a robust decoder reconstructs allowed runtime state.

## 44. Codec Thinking

A codec can be modeled as:

```ts
interface Codec<TExternal, TInternal> {
  decode(value: unknown): TInternal;
  encode(value: TInternal): TExternal;
}
```

This is especially useful when external and internal representations differ.

Example:

```text
external: "2026-09-13T10:00:00Z"
internal: Instant-like domain value
```

The codec centralizes translation and gives tests a natural round-trip shape:

```text
decode(encode(internal)) ≈ internal
```

Exact equality may not hold for normalized representations, so define the intended equivalence relation.

## 45. Validation as Construction

A strong pattern is:

```text
invalid state cannot be constructed through the intended API
```

Example:

```ts
class PositiveQuantity {
  private constructor(public readonly value: number) {}

  static from(value: unknown): PositiveQuantity {
    if (
      typeof value !== "number" ||
      !Number.isSafeInteger(value) ||
      value <= 0
    ) {
      throw new Error("Invalid quantity");
    }

    return new PositiveQuantity(value);
  }
}
```

Now downstream code can depend on the constructor's invariant instead of repeatedly rechecking it.

This is one of the clearest connections between runtime validation and object-oriented encapsulation.

## 46. Factories vs Plain Object Refinement

Use a factory when construction has meaningful semantics, hidden state, identity, or invariants.

Use a plain refinement when the value is naturally structural and behaviorless.

A good heuristic:

```text
DTO / config / event payload → structural
domain entity / value object → constructor/factory
aggregate root → controlled construction
```

Do not wrap every `{ id: string }` in a class merely to satisfy an OOP aesthetic.

## 47. Dependency Injection and Validators

Validators can be pure dependencies:

```ts
interface Validator<T> {
  validate(value: unknown): Result<T, ValidationError[]>;
}
```

Injecting them can help when schemas differ by tenant, API version, or feature state.

But not every validator needs an interface. Pure functions are often simpler:

```ts
const parseCreateOrder = (value: unknown) => ...
```

The rule is consistent with dependency inversion:

> Introduce an abstraction when a meaningful variation or substitution boundary exists.

## 48. Result-Based Decoding

Exceptions are convenient for programmer errors and fail-fast boundaries. Result types are useful when invalid input is expected and should be returned as data.

Conceptual model:

```ts
type Result<T, E> =
  | { ok: true; value: T }
  | { ok: false; error: E };
```

Then:

```ts
function decodeUser(input: unknown): Result<User, ValidationIssue[]> {
  // ...
}
```

This makes failure part of the type-level API.

Avoid turning every application function into nested `Result<Result<...>>`. Choose where the error boundary belongs.

## 49. Validation Pipelines and Middleware

For HTTP systems, validation can live in middleware, controller adapters, route handlers, or generated handlers.

The architectural goal is:

```text
request
  ↓
transport adapter
  ↓
validated command
  ↓
application
```

Avoid scattering checks throughout business code:

```ts
if (typeof input.customerId !== "string") ...
if (!input.customerId.startsWith("CUST-")) ...
```

inside ten different services.

Centralize boundary validation, then keep domain invariants close to the domain model.

## 50. Event and Message Validation

Queues are different from synchronous HTTP because messages may outlive producer deployments.

A consumer should not assume:

```text
producer code version == consumer code version
```

Validate event envelopes at the boundary and handle schema versions deliberately.

Useful envelope fields can include:

```text
eventType
eventVersion
eventId
occurredAt
tenantId
payload
```

Do not trust `tenantId` from an event merely because the schema says it is a string. Event provenance and authorization semantics still matter.

## 51. Idempotency and Validation

Validation should happen before expensive side effects, but idempotency keys may need to be extracted very early.

A typical HTTP flow can be:

```text
parse headers/body
→ validate idempotency key format
→ authenticate
→ validate payload
→ authorize
→ execute idempotent command
```

The exact order depends on threat model and infrastructure.

The principal principle is:

> Validation order should be designed as a cost and security pipeline, not as arbitrary middleware ordering.

## 52. Environment Configuration Validation

Environment variables are strings or absent values at process startup.

Treat them as `unknown`/untrusted configuration rather than pretending:

```ts
const PORT: number = process.env.PORT;
```

A startup configuration pipeline should:

```text
read raw environment
→ parse
→ validate
→ normalize
→ freeze / expose trusted config
```

Failing fast at startup is often better than discovering an invalid configuration during a live request.

## 53. Database Data Is Not Always Trusted

A common misconception is that database values do not need validation because the database is internal.

Reality:

```text
old application versions
manual SQL
partial migrations
corrupted records
legacy rows
external imports
multiple writers
```

can violate current assumptions.

Whether to validate every read depends on the trust and consistency model. High-value boundaries often validate during migration, ingestion, or repository hydration rather than blindly trusting all historical rows forever.

## 54. Defensive Decoding from Third-Party SDKs

SDK TypeScript declarations are not always equivalent to runtime guarantees.

The remote service may:

```text
change behavior
return undocumented fields
return null
return error-shaped payloads
serve older versions
```

For critical integrations, treat third-party responses as an external boundary.

A small adapter should convert vendor data into an internal stable type:

```text
VendorResponse
   ↓ adapter/decoder
InternalCustomer
```

This isolates vendor churn.

## 55. GraphQL, RPC, and Protobuf Boundaries

Different transport technologies have different runtime guarantees.

GraphQL provides a schema and typed operation model, but resolvers, custom scalars, authorization, and backing data still have runtime concerns.

Protobuf provides strongly specified wire representations, but application-level invariants still need enforcement.

RPC frameworks may generate types, but generated client types do not automatically prove that a server follows business semantics.

Do not confuse protocol schema correctness with domain correctness.

## 56. Compile-Time Safety vs Runtime Safety Matrix

Use this mental matrix:

| Concern | TypeScript | Runtime validation | Domain logic | Authorization |
|---|---:|---:|---:|---:|
| property exists | yes | yes | sometimes | no |
| primitive kind | yes | yes | sometimes | no |
| numeric range | yes, only in limited type encodings | yes | yes | no |
| object shape | yes | yes | yes | no |
| cross-record rule | no | possible but usually wrong layer | yes | sometimes |
| actor may act | no | no | sometimes | yes |
| output escaping | no | no | no | context-specific |
| wire compatibility | partially | yes | no | no |

A mature design assigns each invariant to the layer that has the right information and responsibility.

## 57. Type Erasure: The Central Constraint

Many sophisticated TypeScript designs disappear at runtime.

These usually do not survive as runtime values:

```text
interface
type alias
generic parameter
conditional type
mapped type
template literal type
```

These do survive because they are JavaScript values:

```text
class
function
object
array
symbol
string
number
```

This distinction explains why advanced type-level programming cannot replace runtime validation.

A type like:

```ts
type User<TPermissions> = ...
```

cannot inspect a JSON payload merely because the compiler understands it.

## 58. Generic Runtime Validation

Generics are erased too, so this cannot work by itself:

```ts
function parse<T>(input: unknown): T {
  // Where would runtime code obtain T?
  return input as T;
}
```

The missing information must be supplied at runtime:

```ts
function parseWith<T>(
  input: unknown,
  validator: (value: unknown) => value is T,
): T {
  if (!validator(input)) throw new Error("Invalid");
  return input;
}
```

The generic describes the relationship between the validator and result. The validator provides runtime evidence.

## 59. Generic Constraints Do Not Validate Values

A constraint:

```ts
function save<T extends { id: string }>(value: T) {}
```

does not validate a runtime object.

It constrains callers known to the compiler.

This is a fundamental rule:

```text
generic constraint ≠ runtime contract
```

When a generic API crosses an untrusted boundary, pass the runtime evidence explicitly.

## 60. Conditional Types Do Not Execute

Conditional types can compute sophisticated static relationships:

```ts
type ResultOf<T> = T extends Promise<infer U> ? U : T;
```

That logic runs in the TypeScript type system, not in JavaScript.

A runtime validator needs executable branching:

```ts
function isPromiseLike(value: unknown): value is PromiseLike<unknown> {
  return (
    typeof value === "object" &&
    value !== null &&
    "then" in value &&
    typeof value.then === "function"
  );
}
```

Static computation and runtime computation can correspond, but one does not execute the other.

## 61. Decorators and Metadata Do Not Automatically Create Validation

Decorators can attach behavior or metadata, depending on the decorator model and runtime implementation. They do not magically turn TypeScript annotations into complete runtime schemas.

A metadata-driven framework still needs:

```text
runtime metadata
+
validation rules
+
execution policy
```

The architecture should document where those rules come from and how they are versioned.

Avoid believing that:

```ts
foo: CustomerId
```

automatically gives the runtime all information needed to validate `CustomerId`.

## 62. Reflection Boundaries

JavaScript reflection can inspect runtime structure:

```ts
Reflect.getOwnPropertyDescriptor(...)
Object.getOwnPropertyNames(...)
Object.getPrototypeOf(...)
```

But reflection sees runtime values, not the complete erased TypeScript type model.

This creates a useful design question:

> Do we need reflection, explicit schema data, code generation, or a combination?

Explicit metadata is often easier to reason about than trying to infer domain semantics from runtime JavaScript alone.

## 63. Single Source of Truth Is Not Always One File

A system can have multiple representations of one contract:

```text
business specification
transport schema
TypeScript types
runtime validator
database constraints
generated clients
tests
```

Trying to force one artifact to express every invariant can create complexity.

Instead, seek a **single authoritative source per concern** and a traceable transformation between concerns.

For example:

```text
OpenAPI = HTTP shape authority
domain model = business invariant authority
database schema = persistence authority
authorization policy = access authority
```

The architecture wins when the boundaries between authorities are explicit.

## 64. Contract Tests

Contract tests verify that a producer and consumer agree on a contract.

Useful forms include:

```text
request schema compatibility
response schema compatibility
event version compatibility
generated-client integration
consumer-driven contracts
```

Contract testing complements unit tests.

Unit test:

```text
validator accepts/rejects cases
```

Contract test:

```text
producer and consumer remain compatible
```

End-to-end test:

```text
whole deployed system behaves correctly
```

Use each where it has the highest signal.

## 65. Property-Based Validation Tests

Validators have many edge cases:

```text
empty strings
NaN
Infinity
very large numbers
unicode
duplicate keys
deep nesting
unexpected prototypes
unknown fields
boundary values
```

Property-based testing can explore these spaces.

Useful properties:

```text
valid generated values are accepted
obviously invalid values are rejected
normalization is idempotent
encoding/decoding preserves intended meaning
```

Do not rely only on a handful of hand-written happy-path tests.

## 66. Fuzzing the Boundary

A parser or validator that handles attacker-controlled input should be a good fuzzing target.

Look for:

```text
stack overflow
excessive CPU
memory growth
path explosion
catastrophic regular expressions
exception flooding
huge arrays
deep objects
```

Validation is part of the attack surface.

A validator is security-sensitive code because it runs before trust has been established.

## 67. Performance: Validation Cost

Validation adds CPU and allocation cost.

The cost is usually justified at untrusted boundaries, but architecture should prevent duplicate work.

Bad pipeline:

```text
gateway validates
→ controller validates
→ service validates
→ domain validates same DTO again
```

Better:

```text
transport validation once
→ application transformation
→ domain invariants at construction
```

This does not mean “never validate twice.” Re-validation can be justified at trust boundaries. It means duplicate validation should have a reason.

## 68. Performance: Schema Compilation

Some validator architectures interpret schemas repeatedly; others compile schemas into specialized functions.

For high-throughput systems, consider:

```text
startup compilation
schema caching
allocation behavior
error-path construction cost
short-circuit behavior
batch validation
```

Do not optimize prematurely. First establish where boundary validation appears in the profile.

For most applications, correct failure behavior matters more than micro-optimizing simple checks.

## 69. Performance: Large Payloads

Large payloads create pressure on:

```text
network
JSON parsing
allocation
validation traversal
error collection
logging
serialization
```

Validation cannot make an already-too-large payload safe.

Use layered limits:

```text
request size
array length
string length
nesting depth
field count
processing time
```

Rejecting oversized input early protects later layers.

## 70. Security: Validation Is Necessary but Insufficient

A value can be valid and still malicious in context.

Examples:

```text
valid HTML inserted without encoding
valid SQL fragment concatenated into SQL
valid path used for traversal
valid tenant ID used without authorization
valid filename used unsafely on a filesystem
```

Validation should be treated as one security layer:

```text
authentication
→ authorization
→ validation
→ canonicalization
→ context-specific encoding
→ safe API usage
```

Never make the claim “validated input is safe everywhere.”

## 71. Canonicalization Before Policy

Equivalent representations can undermine security policy.

Examples:

```text
Unicode forms
case differences
URL encodings
path normalization
whitespace
numeric formats
```

The safe sequence is often:

```text
parse
→ canonicalize
→ validate
→ authorize
→ act
```

But the exact order depends on the data type. The key is to avoid authorizing one representation and executing another normalized representation.

## 72. Prototype and Class Identity

`instanceof` checks prototype relationships, not structural shape.

This matters because:

```text
same fields ≠ same class identity
```

Two copies of a package can even produce surprising `instanceof` behavior because values may come from different constructor identities.

For transport validation, structural validation is usually more robust than requiring class instances.

Use runtime class identity where object behavior and identity genuinely depend on it.

## 73. Cross-Realm Objects

Objects from another realm, such as an iframe, can challenge assumptions about constructors like `Array` and `instanceof`.

For example, a cross-realm array may not satisfy a same-realm constructor check in the way developers expect.

Prefer semantic checks that match the boundary contract.

For arrays:

```ts
Array.isArray(value)
```

is designed for the array-ness question.

This is another reminder to select runtime checks based on the property you actually need to prove.

## 74. Validation of Map, Set, and Specialized Values

Some JavaScript values carry semantics that JSON cannot represent directly:

```text
Map
Set
Date
RegExp
URL
ArrayBuffer
TypedArray
BigInt
```

A validator should state whether the input is:

```text
already a runtime instance
or
a serialized representation
```

For example, an HTTP payload might encode a set as an array, while an internal value is a `Set<string>`.

Decode first, then validate the domain representation.

## 75. Configuration Contracts and Feature Flags

Configuration is a runtime boundary even when authored by developers.

Feature flags can have constraints:

```text
enabled
rollout percentage 0..100
allowed tenant set
effective date
dependency requirements
```

A static type can model the intended shape; startup validation should reject invalid configuration before the application begins serving traffic.

Feature-flag state can also require dynamic semantic checks because the remote flag service may return runtime data.

## 76. Schema Composition

Complex validators should be composed from smaller contracts.

Conceptually:

```text
CustomerSchema
AddressSchema
LineItemSchema
       ↓
OrderSchema
```

Composition reduces duplication.

But composition can also create hidden coupling when shared schemas become “god contracts.”

A reusable schema should represent a stable concept, not merely a convenient field bundle.

## 77. Partial Updates and Patch Semantics

A patch schema is not simply:

```ts
Partial<CreateUser>
```

because static optionality does not necessarily capture patch semantics.

You may need:

```text
missing
null
empty string
explicit false
zero
```

to have distinct meanings.

For each field define:

```text
Can it be omitted?
Can it be null?
What does null mean?
What does an empty value mean?
Can a field be cleared?
```

This is a contract question before it is a TypeScript question.

## 78. Defaults: Where Should They Apply?

Defaults can be applied at different layers:

```text
transport decoder
application command
domain constructor
database
```

Choosing the wrong layer can obscure provenance.

A transport default may normalize an API convenience:

```text
pageSize omitted → 50
```

A domain default may represent business meaning:

```text
new order state → Draft
```

A database default may be a persistence guarantee.

Do not use database defaults as a substitute for domain understanding.

## 79. Coercion vs Strict Validation

Should `"42"` become `42`?

Sometimes yes for human-entered configuration.

Sometimes no for an API contract where types should be explicit.

Coercion can improve usability but can also create ambiguity:

```text
"" → 0?
"  " → 0?
"1e309" → Infinity?
```

A coercive schema must document exactly what transformations occur.

Strict validation is easier to reason about; controlled coercion is sometimes the correct compatibility strategy.

## 80. Normalization and Idempotence

A good normalizer often has the property:

```text
normalize(normalize(x)) = normalize(x)
```

Examples:

```text
trim whitespace
uppercase canonical identifiers
sort set-like collections
canonicalize URLs
```

Idempotence makes pipelines easier to reason about.

If normalization is not idempotent, repeated application may silently change data.

## 81. Preserve Original Input for Diagnostics?

Keeping raw input can help debugging, but it has security and privacy costs.

Raw inputs may contain:

```text
passwords
tokens
personal data
payment data
secrets
```

Therefore:

```text
diagnostic value vs data exposure
```

must be balanced.

Prefer structured, redacted diagnostics and avoid logging full untrusted payloads by default.

## 82. Observability of Validation Failures

Validation failures should be observable without becoming a data-leak channel.

Useful telemetry:

```text
endpoint
schema/contract version
error code
field path
count
tenant-safe aggregate identifier
```

Avoid logging sensitive field contents.

A spike in `INVALID_ORDER_QUANTITY` may indicate a client regression. That insight is useful without retaining the submitted value.

## 83. Boundary Ownership

Every external boundary should have an owner.

Examples:

```text
HTTP adapter owns HTTP shape
repository adapter owns persistence mapping
message consumer owns event decoding
vendor adapter owns vendor payload mapping
domain owns business invariants
authorization policy owns access rules
```

When no layer owns a rule, it gets duplicated or disappears.

When too many layers own the same rule, drift becomes likely.

## 84. Anti-Pattern: Type Assertion Everywhere

A codebase full of:

```ts
as User
as Customer
as Record<string, unknown>
as any
```

usually indicates missing boundary design.

The correct question is not:

> How can I convince TypeScript?

It is:

> Where can I establish the fact that makes this type true?

## 85. Anti-Pattern: One Mega-Schema

A giant universal schema may validate:

```text
request
database record
domain object
event
response
```

but these are usually different contracts.

A single mega-schema creates:

```text
unnecessary coupling
harder evolution
ambiguous optionality
information leakage
larger change surface
```

Prefer contract-specific schemas linked through explicit mapping.

## 86. Anti-Pattern: Domain Rules in Transport Schemas

A transport schema may know:

```text
quantity > 0
```

but should not normally decide:

```text
customer can buy this SKU from this branch under this credit policy
```

The latter needs domain/application context.

Putting too many business rules in the transport layer makes reuse and testing harder.

## 87. Anti-Pattern: Trusting Generated Types

Generated TypeScript types are useful, but they answer:

```text
What does the generated model say?
```

not necessarily:

```text
What did the remote system actually send?
```

For critical integrations, generated types should be paired with protocol guarantees, runtime decoding, or server-enforced schemas according to the threat model.

## 88. Anti-Pattern: Boolean Validators Without Error Detail

`false` is sometimes enough.

For public APIs and complex ingestion pipelines, it is often not.

Compare:

```ts
false
```

with:

```ts
[
  { path: ["items", 0, "quantity"], code: "POSITIVE_INTEGER_REQUIRED" }
]
```

Diagnostic richness belongs at the right boundary. Do not force every domain predicate to construct rich transport errors if it does not need to.

## 89. Anti-Pattern: Catch Everything and Continue

A parser that catches every exception and turns it into “best effort” data can hide programmer bugs.

Distinguish:

```text
expected invalid input
unexpected program failure
resource failure
security event
```

Only expected validation failures should normally become consumer-facing validation results.

## 90. Anti-Pattern: Validation After Side Effects

Do not:

```text
create DB record
→ discover invalid field
→ reject request
```

Structure the pipeline so local validation occurs before irreversible work where practical.

For distributed workflows, later validation may still be necessary because external state can invalidate assumptions.

## 91. Designing a Runtime Contract Interface

A small abstraction can make validation strategy explicit:

```ts
interface Decoder<T> {
  decode(input: unknown): T;
}
```

Or:

```ts
interface SafeDecoder<T, E> {
  decode(input: unknown): Result<T, E>;
}
```

Use an interface when multiple implementations genuinely matter.

A simple function is often enough:

```ts
type Decoder<T> = (input: unknown) => T;
```

Abstraction is a tool, not an achievement badge.

## 92. Implementing a Minimal Schema Runtime

A learning implementation can expose:

```ts
interface Schema<T> {
  parse(input: unknown): T;
}
```

Primitive schema:

```ts
const stringSchema: Schema<string> = {
  parse(input) {
    if (typeof input !== "string") {
      throw new Error("Expected string");
    }
    return input;
  },
};
```

Object schema can combine field schemas.

The point of implementing this from scratch is to understand:

```text
schema as runtime data
schema as executable behavior
schema → inferred static relationship
error propagation
composition
```

A production library will add far more features, but the conceptual core is small.

## 93. Minimal Object Schema From Scratch

A simplified object-schema architecture:

```ts
type Shape = Record<string, Schema<unknown>>;

type InferShape<S extends Shape> = {
  [K in keyof S]: S[K] extends Schema<infer T> ? T : never;
};

function object<S extends Shape>(shape: S): Schema<InferShape<S>> {
  return {
    parse(input) {
      if (typeof input !== "object" || input === null) {
        throw new Error("Expected object");
      }

      const source = input as Record<string, unknown>;
      const result = {} as InferShape<S>;

      for (const key of Object.keys(shape) as Array<keyof S>) {
        result[key] = shape[key].parse(source[String(key)]);
      }

      return result;
    },
  };
}
```

This demonstrates a central idea:

```text
runtime schema value
        ↕
static inference
```

The runtime object provides the evidence. The generic type describes the relationship.

## 94. Improving the Minimal Runtime

A production-oriented implementation would need explicit choices for:

```text
unknown keys
optional fields
defaults
nullable fields
arrays
unions
recursion
error accumulation
custom refinements
async checks
performance
security limits
```

Do not add these features randomly.

For each feature define:

```text
semantic rule
runtime representation
static representation
failure behavior
test strategy
performance cost
compatibility implications
```

This is how a principal engineer turns a toy abstraction into an engineering system.

## 95. Validation vs Type Guards: Choosing the Surface

Use a type guard when the check is small, local, and naturally boolean:

```ts
function isOrderStatus(value: unknown): value is OrderStatus {
  ...
}
```

Use a decoder when consumers need structured failure:

```ts
parseOrderStatus(value)
```

Use a schema when the contract is composite, reusable, inspectable, or potentially generated.

Use a domain constructor when the invariant creates a semantic value.

These surfaces solve overlapping but different problems.

## 96. Validation and OOP Encapsulation

Earlier object-design chapters emphasized encapsulation and stable invariants.

Runtime validation is the front door to encapsulation.

A good sequence is:

```text
untrusted value
   ↓
boundary validation
   ↓
validated constructor/factory
   ↓
private state
   ↓
domain operations
```

The object should not need to defend itself against every impossible state if its public construction path already prevents those states.

However, internal mutation and deserialization paths must also preserve invariants.

## 97. Validation and Composition

Composition works well when each component owns a coherent contract.

Example:

```text
Money
CustomerId
Sku
Quantity
OrderLine
Order
```

Each smaller value object can establish local invariants.

Then `Order` can enforce aggregate-level invariants.

This creates a layered proof structure:

```text
primitive evidence
→ value-object evidence
→ component evidence
→ aggregate evidence
```

The resulting system becomes easier to reason about than one gigantic validation function.

## 98. Validation and Liskov Substitution

Runtime validators can become part of substitutability.

Suppose an interface accepts:

```ts
interface PaymentMethod {
  authorize(amount: Money): Promise<AuthResult>;
}
```

A concrete implementation must not require a stronger input precondition than the interface contract promises.

Transport validation may establish:

```text
PaymentRequest
```

but the polymorphic method contract must remain compatible across implementations.

Validation policy belongs to the contract boundary, not to accidental subclass assumptions.

## 99. Validation and Interface Segregation

A massive schema can be a sign of a massive interface.

If a handler receives one object containing:

```text
customer
inventory
pricing
payment
shipping
```

just because a shared DTO exists, the system may have poor capability boundaries.

Validation contracts should often be capability-focused:

```text
CreateOrderInput
PriceOrderInput
AuthorizePaymentInput
AllocateInventoryInput
```

Smaller contracts reduce accidental coupling.

## 100. Validation and Dependency Inversion

High-level application policy should depend on stable validated concepts, not raw transport shapes.

Prefer:

```text
HTTP JSON → decoder → command → use case
```

rather than:

```text
use case(req.body)
```

This means the use case can be tested independently of HTTP and can be invoked from jobs, messages, or CLI commands using the same command contract.

## 101. Validation and Open-Closed Design

Stable business code should not need to change every time a transport format evolves.

An adapter can absorb variations:

```text
VendorRequestV1
VendorRequestV2
   ↓
normalize
   ↓
InternalCommand
```

The core stays stable while the edge expands.

This is open-closed thinking applied to contract evolution.

## 102. Validation and SRP

A validator should not also:

```text
authorize user
write database
send event
log secrets
construct every domain object
```

unless those responsibilities are intentionally part of a higher-level orchestration component.

Keep responsibilities aligned:

```text
decoder = input contract
mapper = representation translation
policy = authorization
factory = domain construction
repository = persistence
```

## 103. Runtime Contract Review Checklist

When reviewing a new boundary, ask:

```text
What is the input representation?
What is truly untrusted?
Where is parse performed?
Where is runtime validation performed?
Which unknown keys are allowed?
Which normalization occurs?
Which errors are returned?
Which semantic rules require domain/application state?
Where is authorization enforced?
What type represents the trusted result?
How is it constructed?
How are versions handled?
What size/depth limits exist?
What is logged?
How is the contract tested?
```

A review that cannot answer these questions is not yet complete.

## 104. Predict-the-Output Lab 1

Code:

```ts
interface User {
  name: string;
}

const value: unknown = { name: "A" };
const user = value as User;

console.log(user.name);
console.log(user instanceof Object);
```

Prediction:

```text
A
true
```

Trace:

1. The assertion changes only the static view.
2. The runtime value remains the original object.
3. Property access works because the property exists.
4. The value is an ordinary object.

Rule:

> A type assertion never constructs runtime class identity.

## 105. Predict-the-Output Lab 2

Code:

```ts
class User {
  constructor(public name: string) {}
}

const raw = JSON.parse('{"name":"A"}') as User;

console.log(raw.name);
console.log(raw instanceof User);
```

Prediction:

```text
A
false
```

Trace:

`JSON.parse` creates a plain object. The `as User` assertion does not call the constructor or alter the prototype.

Rule:

> Serialization/deserialization changes representation; it does not automatically reconstruct domain identity.

## 106. Predict-the-Output Lab 3

Code:

```ts
const states = ["draft", "confirmed"] as const;

type State = typeof states[number];

function isState(value: unknown): value is State {
  return typeof value === "string" &&
    (states as readonly string[]).includes(value);
}

console.log(isState("draft"));
console.log(isState("paid"));
console.log(isState(1));
```

Prediction:

```text
true
false
false
```

The same runtime collection supports validation and static union derivation.

## 107. Predict-the-Output Lab 4

Code:

```ts
function isUser(value: unknown): value is { id: string } {
  return typeof value === "object" &&
    value !== null &&
    "id" in value;
}

const value: unknown = { id: 42 };

if (isUser(value)) {
  console.log(typeof value.id);
}
```

Prediction:

```text
number
```

The type predicate claims more than the implementation proves.

The compiler trusts the predicate signature. Runtime correctness still depends on the guard's logic.

Rule:

> A user-defined type predicate is an executable assertion boundary; a bad predicate creates unsoundness.

## 108. Predict-the-Output Lab 5

Code:

```ts
const input: unknown = null;

if (typeof input === "object") {
  console.log("object");
}
```

Prediction:

```text
object
```

Because JavaScript specifies `typeof null === "object"`. TypeScript's narrowing behavior accounts for this quirk. citeturn746530search0

Rule:

> Runtime semantics come first; TypeScript models them.

## 109. Debugging Exercise 1

Bug:

```ts
function createUser(input: CreateUserInput) {
  return save(input);
}

app.post("/users", async (req, res) => {
  const input = req.body as CreateUserInput;
  await createUser(input);
});
```

Find the trust hole.

Answer path:

```text
req.body
  ↓
assertion
  ↓
application
```

No runtime evidence is established.

Refactor toward:

```text
req.body
  ↓
decoder
  ↓
validated CreateUserCommand
  ↓
createUser
```

The use case should not need to know that its caller was HTTP.

## 110. Debugging Exercise 2

Bug:

```ts
type UpdateUser = Partial<{
  nickname: string;
  active: boolean;
}>;
```

The API uses:

```text
missing nickname = leave unchanged
nickname = "" = clear nickname
nickname = null = invalid
```

The type alone does not encode the full runtime contract, and a generic `Partial` may be too weak or ambiguous.

Design a decoder whose runtime semantics match the patch protocol.

## 111. Debugging Exercise 3

Bug:

```ts
function validateOrder(input: unknown): boolean {
  // validates shape and then queries the database for stock
  // ...
}
```

Problems:

```text
semantic name hides I/O
boolean loses diagnostics
potential N+1 behavior
mixes structural and contextual rules
```

Split into:

```text
decode order shape
→ validate local invariants
→ application service checks stock
→ domain operation
```

## 112. Debugging Exercise 4

Bug:

```ts
const config = JSON.parse(process.env.CONFIG_JSON!) as Config;
start(config);
```

The non-null assertion and type assertion create false certainty.

A robust startup path should:

```text
ensure variable exists
→ parse JSON
→ validate schema
→ normalize defaults
→ freeze/expose trusted config
→ start server
```

Startup failure is often preferable to late request-time failure for invalid mandatory configuration.

## 113. Code Review Exercise

Review:

```ts
type User = {
  id: string;
  role: "admin" | "user";
};

function handle(payload: unknown) {
  const user = payload as User;

  if (user.role === "admin") {
    performAdminAction(user);
  }
}
```

Questions:

```text
Who established `payload` is a User?
Can `role` be attacker-controlled?
Does admin role mean authentication plus authorization?
Is `id` scoped to the current tenant?
What runtime errors occur if payload is null?
```

The key issue is not merely the assertion. It is that authorization has been conflated with a field value.

## 114. Code Review: Better Boundary

A stronger shape is:

```ts
const raw = await request.json();

const input = createUserRequestDecoder.decode(raw);

const actor = authenticate(request);
const command = authorizeAndBuildCommand(actor, input);

await createUserUseCase.execute(command);
```

Now the layers have distinct responsibilities:

```text
decoder → shape/format
authentication → identity
authorization → permission/scope
command factory → application contract
use case → business workflow
domain → invariant
```

## 115. From Scratch: Primitive Schema

Implement a schema abstraction with:

```text
Schema<T>
string
number
boolean
literal
array
object
union
optional
refine
```

Start with primitives and make failures deterministic.

Guided target:

```ts
interface Schema<T> {
  parse(input: unknown, path?: readonly (string | number)[]): T;
}
```

Then add composition one primitive at a time.

Mastery requires rebuilding it without reference.

## 116. From Scratch: Object Schema

Requirements:

```text
field schemas
missing-field errors
unknown-key policy
nested error paths
deterministic traversal
```

Do not use `any` internally.

Use:

```text
unknown
Record<string, unknown>
generics
conditional/mapped types
```

The implementation should make the static and runtime relationships visible.

## 117. From Scratch: Union Schema

Implement a union that tries member schemas.

Design decisions:

```text
fail fast vs aggregate member failures
discriminator optimization
error selection
ambiguity
```

For discriminated unions, validate the discriminator first.

Example:

```ts
type Command =
  | { kind: "create"; name: string }
  | { kind: "delete"; id: string };
```

Runtime structure can mirror the static discriminant pattern.

## 118. From Scratch: Refinement Schema

Add:

```text
refine(predicate, error)
```

Conceptually:

```ts
const positive = numberSchema.refine(
  value => Number.isInteger(value) && value > 0,
  "Expected positive integer",
);
```

The refinement should run only after primitive validation succeeds.

This is a direct implementation of:

```text
base type
→ runtime predicate
→ refined semantic value
```

## 119. From Scratch: Result Decoder

Rebuild the decoder with:

```ts
type Result<T, E> =
  | { ok: true; value: T }
  | { ok: false; error: E };
```

Now implement:

```text
map
flatMap
mapError
combine
```

The exercise connects runtime contracts with functional composition.

The LLD lesson is that error flow is part of the object/function contract.

## 120. From Scratch: Contract-Driven API Adapter

Build a mini API adapter:

```text
raw HTTP object
→ request decoder
→ command
→ use case
→ domain entity
→ response encoder
```

No framework-specific code should enter the domain layer.

Add:

```text
unknown-field policy
error codes
request size limit
authorization stub
response schema
```

Then test the entire boundary as a contract.

## 121. Production Exercise: Configuration Loader

Build a configuration loader with:

```text
environment parsing
required variables
optional defaults
numeric ranges
URL validation
enum validation
secret redaction
immutable result
```

The loader should execute once during startup and expose only a trusted configuration object.

Add tests for malformed environments and unexpected extra fields.

## 122. Production Exercise: Event Decoder

Build an event consumer with:

```text
versioned envelope
event type
payload decoder
structured errors
dead-letter classification
idempotency key extraction
tenant context
observability
```

Test producer/consumer compatibility for at least two versions.

The objective is not building a queue library. It is designing a resilient trust boundary.

## 123. Production Exercise: ERP Order Boundary

Model a jewellery ERP order ingress:

```text
tenantId
branchId
customerId
items
sku
quantity
purity
metalWeight
stoneWeight
price
currency
```

Separate:

```text
transport validation
authorization
inventory checks
pricing policy
domain invariants
persistence
```

Explicitly decide which rules are:

```text
schema-local
contextual
domain-level
database-level
```

Do not let a single validator become the entire ERP policy engine.

## 124. Principal-Level Design Exercise

Design a contract architecture for:

```text
mobile client
web client
public REST API
internal message bus
vendor integration
database
```

Your deliverable should include:

```text
contract source of truth
runtime validation strategy
generated artifacts
mapping boundaries
versioning
error model
authorization
observability
compatibility policy
rollback strategy
```

Defend why each contract lives where it does.

## 125. Interview Questions — Foundation

1. Why does a TypeScript interface not validate JSON at runtime?
2. What is the difference between `unknown` and `any`?
3. What is the difference between a type assertion and a type guard?
4. Why can `instanceof` fail after JSON deserialization?
5. What does `as const` do?
6. What does `satisfies` do, and what does it not do?
7. Why are generic constraints not runtime validation?
8. What is type erasure?
9. Why is `unknown` preferable at untrusted boundaries?
10. What is the difference between parsing, validation, normalization, and construction?

## 126. Interview Questions — Design

1. Where should validation happen in a layered architecture?
2. Should domain objects validate themselves?
3. When would you choose schema-first over type-first?
4. How do you avoid drift between runtime validators and TypeScript types?
5. How do OpenAPI and generated clients affect architecture?
6. When should unknown fields be rejected?
7. What belongs in a transport schema versus a domain model?
8. How would you validate a multi-tenant request safely?
9. How do you design versioned event consumers?
10. How would you measure validator performance?

## 127. Interview Questions — Principal Level

1. Design a runtime-contract architecture for 100+ services.
2. How would you migrate from handwritten DTOs to generated contracts?
3. How would you prevent schema drift in CI?
4. How would you handle two producer versions and three consumer versions?
5. How would you separate validation from authorization?
6. What validation belongs in the API gateway versus the service?
7. How would you handle legacy database rows that violate current invariants?
8. When is duplicate validation justified?
9. How would you protect a validator from denial-of-service inputs?
10. How do you prove that a trusted domain type was constructed from validated data?

## 128. Comparison: Assertion vs Guard vs Decoder vs Schema vs Constructor

| Mechanism | Runtime work | Failure model | Best use |
|---|---|---|---|
| assertion | none | later failure possible | compiler escape hatch |
| type guard | yes | boolean | local narrowing |
| assertion function | yes | throw/terminate | establish invariant at a boundary |
| decoder | yes | structured result/throw | external data |
| schema | yes | composable contract | reusable runtime contract |
| constructor/factory | yes | controlled construction | domain invariant |

A mature codebase uses several of these rather than choosing one universally.

## 129. Comparison: Type-First vs Schema-First

| Dimension | Type-first | Schema-first |
|---|---|---|
| static ergonomics | excellent | depends on inference |
| runtime truth | separate implementation | executable by design |
| drift risk | higher | lower if truly single-source |
| library coupling | lower | often higher |
| external contract generation | possible | often natural |
| complex type expressiveness | excellent | may vary |
| onboarding | familiar to TS teams | requires schema model |

Choose based on lifecycle, not fashion.

## 130. Decision Framework

For every runtime-contract decision score:

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

Then identify the dominant constraint.

For example:

```text
public internet API
→ security/correctness dominate

high-throughput internal event bus
→ compatibility/performance dominate

startup configuration
→ correctness/reliability dominate

domain value object
→ invariant/maintainability dominate
```

## 131. Canonical Architecture

A robust TypeScript service can be pictured as:

```text
          EXTERNAL WORLD
                │
        ┌───────▼────────┐
        │ Parse / Decode │
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │ Runtime Schema │
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │ Normalize      │
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │ AuthN / AuthZ  │
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │ Command / DTO  │
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │ Domain Factory │
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │ Domain Model   │
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │ Side Effects   │
        └────────────────┘
```

Not every system needs every box, but every box should have a reason.

## 132. ECMAScript / Specification Discipline

When discussing runtime behavior, separate claims about the language from claims about the host.

Useful hierarchy:

```text
ECMAScript specification
    ↓
JavaScript engine
    ↓
host runtime (Node.js / browser / worker)
    ↓
framework / library
    ↓
application
```

For this chapter, `typeof`, objects, prototypes, `instanceof`, arrays, functions, and JavaScript coercion are runtime-language concerns. HTTP request bodies, environment variables, file systems, databases, and framework middleware are host/application concerns.

Do not cite a Node-specific behavior as if it were an ECMAScript guarantee.

## 133. Retrieval Record

Before completing the chapter, write from memory:

```text
What exactly does TypeScript know at compile time?
What evidence does runtime validation provide?
Why does `unknown` localize uncertainty?
Why does `as` not validate?
Why do generic parameters disappear?
How do schema-first and type-first differ?
Where should semantic validation stop?
Where does authorization begin?
How do DTOs become domain values?
What creates contract drift?
How do you test runtime contracts?
```

Mark each:

- `[ ] Not Started`
- `[~] In Progress`
- `[?] Needs Revision`
- `[+] Completed`
- `[*] Mastered`

Only mark `[*]` after implementing and defending the concept.

## 134. Three-Track Study System

### Track A — Core Theory

Study the boundary semantics:

```text
TypeScript types
→ erased JavaScript
→ runtime values
→ executable validation
→ trusted domain values
```

Be able to explain the difference between static compatibility and runtime evidence.

### Track B — Implementation

Progress through:

```text
Guided validator
→ partially guided schema
→ no-reference decoder
→ edge-case hardened decoder
→ production boundary
```

Every implementation must include malformed input tests.

### Track C — Interview / Reasoning

Practice:

```text
predict
→ explain
→ compare
→ refactor
→ defend
```

For every design, explain where trust increases and why.

## 135. The Golden Rules

1. Treat external data as `unknown` until you have evidence.
2. Do not confuse assertions with validation.
3. Do not confuse schema validity with authorization.
4. Keep transport contracts separate from domain invariants when their responsibilities differ.
5. Prefer one authoritative representation per concern and explicit mappings between concerns.
6. Make contract evolution a design feature, not an emergency response.
7. Treat validation code as security-sensitive infrastructure.
8. Measure before optimizing, but always bound attacker-controlled work.
9. Preserve stable domain concepts even when transport formats change.
10. Let types describe trusted assumptions and let runtime code establish them.

## 136. Completion Snapshot

At completion, record:

| Area | Status | Evidence |
|---|---|---|
| `unknown` boundaries | `[ ]` | explained + implemented |
| assertions vs validation | `[ ]` | code review |
| runtime type guards | `[ ]` | no-reference implementation |
| schema composition | `[ ]` | object/union/refinement runtime |
| DTO/domain mapping | `[ ]` | production exercise |
| contract evolution | `[ ]` | versioning exercise |
| security | `[ ]` | adversarial test set |
| performance | `[ ]` | measured profile |
| principal design | `[ ]` | architecture defense |

Reading is not evidence. Implementation and diagnosis are evidence.

## 137. Mastery Gate

You have completed this chapter only when you can:

```text
[ ] Explain type erasure without notes
[ ] Explain why `unknown` is a useful trust boundary
[ ] Write safe runtime type guards
[ ] Identify dishonest type predicates
[ ] Build a small schema runtime from scratch
[ ] Explain parse vs validate vs normalize vs construct
[ ] Design DTO → command → domain boundaries
[ ] Separate structural validation from authorization
[ ] Handle tenant/branch context safely
[ ] Design versioned event validation
[ ] Explain schema-first vs type-first trade-offs
[ ] Prevent static/runtime contract drift
[ ] Test error paths and malformed values
[ ] Identify validation-driven DoS risks
[ ] Defend a contract architecture at principal level
```

## 138. Concept Connections

This chapter connects directly to earlier work:

```text
Object Model
   → runtime values and identity

Encapsulation
   → controlled construction of valid objects

Abstraction
   → stable decoder/contract interfaces

Inheritance / Polymorphism
   → substitution and validator contract design

Composition
   → composed schemas and composed trust boundaries

Cohesion / Coupling
   → focused contract modules

SRP
   → boundary validation versus domain policy

OCP
   → adapters for contract evolution

LSP
   → compatible preconditions and validators

ISP
   → capability-focused DTOs/contracts

DIP
   → use cases depending on trusted commands, not transport details

Generics
   → type relationships around runtime evidence

Control-flow narrowing
   → compiler recognition of runtime checks

Modules
   → contract ownership and boundary isolation

Advanced type-level programming
   → static projection of runtime-authored values

Decorators/metadata
   → optional runtime contract infrastructure
```

## 139. What This Chapter Is Really Teaching

The deepest lesson is not “how to validate JSON.”

It is this:

> A type is an assumption. A runtime boundary is where you earn the right to make that assumption.

TypeScript is exceptionally powerful because static modeling can make valid states easy to express and invalid programs difficult to compile. But software systems exist at runtime, where data arrives without TypeScript's permission.

Principal-level TypeScript design therefore combines:

```text
static precision
+
runtime evidence
+
domain invariants
+
authorization context
+
contract evolution
+
operational discipline
```

The resulting architecture does not pretend the compiler controls the whole world. It clearly marks where trust begins, where it changes, and who is responsible for maintaining it.

## 140. Canonical References and Source Discipline

Primary reference:

- TypeScript Handbook — Narrowing: runtime-oriented checks, control-flow analysis, type predicates, assertion functions, `never`, and exhaustiveness. citeturn746530search0
- TypeScript Handbook — Basic Types: `unknown`, `any`, `never`, and related type semantics. citeturn746530search1
- TypeScript Handbook introduction: scope and purpose of the Handbook as a guide rather than a complete language specification. citeturn746530search3

When implementing production validation libraries, consult the exact library documentation and versioned specification for the chosen runtime contract technology. Do not infer library behavior from TypeScript's type system alone.

## 141. Final Principal Review

Before adopting a runtime-contract approach, answer these questions in writing:

```text
1. What is the authoritative source of this contract?
2. Where does runtime evidence come from?
3. What remains untrusted after decoding?
4. Which facts are local and which need application state?
5. Where are domain invariants established?
6. Where is authorization established?
7. How can the contract evolve without breaking consumers?
8. What happens to unknown fields?
9. What are the size/depth/resource limits?
10. How are failures observed without leaking sensitive data?
11. Which artifacts are generated?
12. How is generated drift detected?
13. What is duplicated intentionally?
14. What is duplicated accidentally?
15. How will the design behave five years and fifty versions from now?
```

A principal engineer is not finished when the validator works. The job is to make the trust architecture understandable, testable, evolvable, and defensible.

## 142. Canonical Architecture Review Notes

Use this section as a reusable review worksheet:

```text
Boundary:
Producer:
Consumer:
Representation:
Parser:
Runtime validator:
Normalizer:
Trusted type:
Domain constructor/factory:
Authorization context:
Error contract:
Versioning policy:
Unknown-key policy:
Resource limits:
Observability:
Compatibility tests:
Generated artifacts:
Rollback plan:
```

An empty field identifies an architectural question that still requires a decision.

## 143. Source Hierarchy

When resolving runtime-contract questions, use this hierarchy:

```text
ECMAScript specification
→ TypeScript language/reference documentation
→ official protocol/contract specification
→ runtime implementation documentation
→ library documentation
→ framework documentation
→ application conventions
```

Do not elevate a framework behavior into a language guarantee.

Do not elevate a TypeScript type declaration into a runtime guarantee.

Do not elevate a generated artifact into an authoritative source without knowing what generated it.

## 144. Final Retrieval Drill

Without notes, explain the following in five minutes:

```text
A JSON request enters a TypeScript service.
The body is `unknown`.
A schema validates the structure.
A normalizer canonicalizes identifiers.
An authorization policy verifies branch scope.
A factory constructs value objects.
A domain aggregate enforces business invariants.
A serializer produces a versioned response.
```

Then answer:

```text
Where did trust increase?
Which claims were static?
Which claims were runtime-proven?
Which claims depended on context?
Which objects gained domain identity?
Which boundaries must remain version-aware?
```

If you can answer precisely, the core objective of the chapter has been met.

## 145. Closing Rule

Remember this sequence:

```text
Types describe.
Runtime checks establish.
Constructors preserve.
Policies authorize.
Contracts evolve.
Tests verify.
Observability reveals.
```

That sequence is the bridge from advanced TypeScript syntax to production-grade object and system design.
