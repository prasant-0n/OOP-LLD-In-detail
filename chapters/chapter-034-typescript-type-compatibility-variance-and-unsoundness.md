# Chapter 034 — TypeScript Type Compatibility, Variance, and Deliberate Unsoundness

> **Role:** Principal JavaScript/TypeScript Engineer · Type-System Specialist · Runtime & LLD Architect  
> **Focus:** structural compatibility, assignability, variance, function types, method bivariance, generic substitution, strictness, unsoundness, API design  
> **Status:** `[~] In Progress`

---

## Chapter Purpose

This chapter examines the rules underneath “these two TypeScript types are compatible.”

The goal is not to memorize compiler behavior. The goal is to derive it from **value flow and substitutability**.

The central question is:

> **When can one statically typed value safely stand in for another, and where does TypeScript intentionally stop proving that safety?**

This chapter connects the type system to OOP/LLD concerns such as Liskov substitution, interface segregation, dependency inversion, encapsulation, mutable state, callback design, and runtime trust boundaries.

---

## 1. Learning Objectives

By the end of this chapter, you should be able to reason precisely about TypeScript assignability, structural compatibility, excess property checking, function parameter variance, method versus function variance, generic compatibility, unions and intersections, narrowing assumptions, bivariance, array covariance, indexed access hazards, and the practical sources of TypeScript unsoundness.

You should also be able to distinguish:

```text
type compatibility
≠
behavioral substitutability
≠
runtime safety
≠
domain correctness
```

The mastery sequence remains:

`Understand → Explain → Predict → Implement → Debug → Apply → Compare → Defend`

---

## 2. Prerequisites

This chapter assumes the preceding chapters on TypeScript structural typing, classes, generics, function types, control-flow analysis, modules, and advanced type-level programming.

The important prerequisite mental model is that TypeScript checks relationships among static types. JavaScript executes runtime values.

The gap between those two systems is where many of the hardest design bugs appear.

---

## 3. What Is Type Compatibility?

Type compatibility asks whether one static type can be used where another is expected.

In TypeScript, compatibility is primarily structural.

```ts
type Point2D = {
  x: number;
  y: number;
};

type NamedPoint = {
  x: number;
  y: number;
  name: string;
};

const p: Point2D = { x: 1, y: 2 };

const n: NamedPoint = {
  x: 1,
  y: 2,
  name: "A",
};
```

A `NamedPoint` contains everything required by `Point2D`, so this is compatible:

```ts
const p2: Point2D = n;
```

The crucial question is not:

> Are the names identical?

It is:

> Does the source provide the members the target requires with compatible types?

---

## 4. Why Structural Typing Exists

Structural typing is useful for JavaScript because JavaScript code naturally communicates through object shape and capabilities.

It enables:

```ts
type Printable = {
  print(): void;
};

function render(value: Printable) {
  value.print();
}
```

Any object with a compatible `print` method can participate.

Benefits include:

```text
flexibility
library interoperability
gradual adoption
duck-typing alignment
small interfaces
```

The cost is that names and declared intent do not automatically create nominal identity.

---

## 5. Structural Compatibility Is Not Behavioral Compatibility

Consider:

```ts
type PositiveAmount = number & {
  readonly __brand: "PositiveAmount";
};
```

At runtime, it is still a number.

Likewise two objects may satisfy the same interface while implementing different business semantics.

A compiler can prove:

```text
required members exist
```

It cannot generally prove:

```text
method obeys business policy
method has intended side effects
method preserves all domain invariants
method performs authorization correctly
```

This is why static compatibility must be supplemented by contracts and tests.

---

## 6. Compatibility Is Directional

Assignability is not necessarily symmetric.

```ts
type Animal = {
  name: string;
};

type Dog = {
  name: string;
  breed: string;
};

const dog: Dog = {
  name: "Rex",
  breed: "Shepherd",
};

const animal: Animal = dog;
```

This is valid because every required `Animal` member is present on `Dog`.

But:

```ts
const dog2: Dog = animal;
```

is not safe because `animal` may not have `breed`.

Mental model:

```text
source must satisfy target's required contract
```

Compatibility is about whether the target can safely use the source.

---

## 7. Width and Depth

Object compatibility has two useful dimensions.

**Width** asks how many members exist.

```text
larger shape → smaller required shape
```

is often compatible.

**Depth** asks whether corresponding member types are compatible.

Example:

```ts
type A = {
  value: string;
};

type B = {
  value: "admin";
};
```

A `B` can be used where `A` is expected because `"admin"` is a subtype of `string`.

Depth rules become much more interesting with functions because input and output positions behave differently.

---

## 8. Variance — The Core Idea

Variance describes how type relationships behave when a generic type constructor or function type contains another type.

Three common forms:

```text
covariant
contravariant
invariant
```

Suppose:

```text
Dog <: Animal
```

Covariance preserves direction:

```text
Container<Dog> <: Container<Animal>
```

Contravariance reverses direction:

```text
Handler<Animal> <: Handler<Dog>
```

Invariance allows neither direction.

Variance is fundamentally about substitutability under use, not about syntax preference.

---

## 9. Why Function Inputs Are Contravariant

Suppose:

```ts
type Animal = { name: string };
type Dog = Animal & { breed: string };
```

A consumer expecting a callback for `Dog` may call it with a `Dog`.

A function that can process any `Animal` is safe:

```ts
const animalHandler = (value: Animal) => {
  console.log(value.name);
};
```

It can safely process a `Dog`.

But a function that requires a `Dog` is not safe where a handler for arbitrary `Animal` is expected:

```ts
const dogHandler = (value: Dog) => {
  console.log(value.breed);
};
```

The caller might supply a plain `Animal`.

Therefore input positions naturally want contravariance.

---

## 10. Why Function Outputs Are Covariant

Suppose a caller expects:

```ts
type AnimalFactory = () => Animal;
```

A factory returning a `Dog` is safe:

```ts
const dogFactory = (): Dog => ({
  name: "Rex",
  breed: "Shepherd",
});

const animalFactory: AnimalFactory = dogFactory;
```

Every returned `Dog` is usable as an `Animal`.

Therefore output positions naturally want covariance.

The classic rule is:

```text
producer → covariance
consumer → contravariance
```

---

## 11. Function Type Compatibility

Consider:

```ts
type Handler<T> = (value: T) => void;

declare const animalHandler: Handler<Animal>;
declare const dogHandler: Handler<Dog>;
```

Under strict function checking, the direction of assignment is governed by the callback's parameter position.

The question to ask is:

> Which values might the receiving code legally pass?

This use-based reasoning is more reliable than memorizing arrows.

---

## 12. `strictFunctionTypes`

TypeScript's `strictFunctionTypes` option enables stricter checking for function parameter positions. The option is part of the broader strictness family documented by TypeScript. citeturn746530search2

Conceptually, it prevents assigning a narrower callback where a broader callback is required.

Without this discipline, callback APIs can accept functions that later receive values the function cannot safely handle.

For production systems, strict mode should generally be treated as the baseline unless there is a documented reason not to use it.

---

## 13. Method Bivariance

TypeScript has a special compatibility behavior for method and constructor parameter positions in certain structural comparisons.

This is often described as **bivariance**.

The practical consequence is important:

```ts
type Consumer<T> = {
  consume(value: T): void;
};
```

can have looser assignability behavior than:

```ts
type Consumer<T> = {
  consume: (value: T) => void;
};
```

The two forms look similar, but their type-checking behavior can differ.

Do not rely on visual similarity. Know which member form your API exposes.

---

## 14. Why Bivariance Exists

The historical reason is largely compatibility with existing JavaScript patterns and common callback-style APIs.

The trade-off is that bivariance can admit assignments that are less strictly safe.

This creates a principal-level API question:

> Should a public interface expose a method or a function-valued property when callback variance matters?

For callback-heavy libraries, explicit function properties can provide stricter variance behavior under `strictFunctionTypes`.

---

## 15. `void` in Function Compatibility

A function returning a value can often be used where a `void`-returning callback is expected because the caller can ignore the result.

Example:

```ts
const callback: () => void = () => 42;
```

This does not mean the function has no return value.

It means the consuming callback contract does not use the result.

Do not generalize this special compatibility behavior to mean that all return-type differences are ignored.

---

## 16. Optional Parameters and Compatibility

Optional parameters affect which calls are legal and therefore affect callback compatibility.

```ts
type Listener = (value: string, index?: number) => void;
```

A function with an optional parameter can be more flexible than one requiring the parameter.

When reviewing callback compatibility, reason from the set of calls the receiver may make.

Do not determine safety solely by comparing declarations visually.

---

## 17. Rest Parameters

Rest parameters model variable arity:

```ts
type Log = (...values: string[]) => void;
```

The type system must account for the fact that callers may provide different numbers of arguments.

Generic tuple types can preserve exact argument relationships:

```ts
function call<T extends readonly unknown[], R>(
  fn: (...args: T) => R,
  ...args: T
): R {
  return fn(...args);
}
```

The same principle from Chapter 29 applies here: tuple relationships preserve the call contract more precisely than broad arrays.

---

## 18. Parameter Count and Function Assignability

Function compatibility considers more than just parameter types.

For example, a function that needs fewer arguments can often be used where a function accepting more arguments is expected because extra arguments can be ignored.

```ts
const acceptsOne = (x: string) => {};
const callerMayPassTwo: (x: string, y: number) => void = acceptsOne;
```

This is safe because `acceptsOne` does not need `y`.

Reverse the dependency and the safety reasoning changes.

---

## 19. Optional vs Required Properties

These are different contracts:

```ts
type A = {
  id?: string;
};

type B = {
  id: string;
};
```

A `B` satisfies `A`, but an `A` does not necessarily satisfy `B`.

With `exactOptionalPropertyTypes`, TypeScript can also distinguish absence from explicitly assigning `undefined` more precisely in optional property declarations. citeturn746530search2

This matters for patch APIs, configuration objects, and serialization semantics.

---

## 20. Excess Property Checking

Consider:

```ts
type User = {
  id: string;
};

const a: User = {
  id: "1",
  name: "A",
};
```

TypeScript reports an excess-property error for this fresh object literal.

But:

```ts
const source = {
  id: "1",
  name: "A",
};

const b: User = source;
```

may be accepted because structural assignability focuses on required members.

This can surprise developers who think TypeScript always enforces exact object shapes.

---

## 21. Excess Property Checking Is Not Exact Object Typing

Excess property checking is best understood as a contextual safeguard for object literals, not a universal exactness system.

This means:

```text
fresh literal
→ extra-property diagnostics

existing variable
→ ordinary structural assignability
```

The distinction explains many “why did TypeScript allow this?” questions.

When exactness is a true domain requirement, encode and enforce it intentionally at runtime as well.

---

## 22. Open Object Shapes

Structural object types should often be thought of as open contracts:

```ts
type UserSummary = {
  id: string;
  name: string;
};
```

This does not automatically mean:

```text
the runtime object contains exactly two keys
```

It means:

```text
these members are required for this static view
```

This is valuable for extensibility but dangerous when developers assume unknown fields are impossible.

---

## 23. `Record<string, unknown>` Is Not a Universal Dictionary

A common pattern is:

```ts
type JsonObject = Record<string, unknown>;
```

This can be useful, but it should not be mistaken for a proof that an arbitrary JavaScript object is a simple dictionary.

Objects can have:

```text
symbol keys
prototype properties
accessors
non-enumerable properties
special internal behavior
```

The semantic contract should determine whether `Record<string, unknown>` is an appropriate abstraction.

---

## 24. Arrays Are a Major Variance Trap

Arrays are mutable, which makes naive covariance dangerous.

Conceptually:

```text
Dog[] <: Animal[]
```

can look convenient because every dog is an animal.

But if code receives an `Animal[]` and writes a non-Dog animal into it, a Dog-only consumer can observe an invalid value.

TypeScript historically permits useful array relationships while relying on other checks and runtime semantics. This is one of the classic examples showing that the type system is intentionally not perfectly sound.

The safe design principle is:

```text
readonly collections are easier to reason about than mutable covariant collections
```

---

## 25. `ReadonlyArray` and Covariance

When a collection is only consumed:

```ts
function printAnimals(values: readonly Animal[]) {}
```

using `readonly` expresses that the function cannot mutate the collection through that reference.

This removes one major source of variance problems.

Prefer:

```ts
readonly T[]
ReadonlyArray<T>
```

for API inputs that do not need mutation.

The API communicates both intent and a stronger safety property.

---

## 26. Tuple Variance

Tuples encode position:

```ts
type Pair = [string, number];
```

A tuple is not merely an array.

With variadic tuples, TypeScript can express precise relationships among positions:

```ts
type Command = ["create", string];
```

Variance still matters when tuples are mutable because writes can violate assumptions.

Use readonly tuples for immutable protocol-like values:

```ts
type Command = readonly ["create", string];
```

---

## 27. Function Properties vs Methods

Compare:

```ts
interface A<T> {
  handle(value: T): void;
}
```

and:

```ts
interface B<T> {
  handle: (value: T) => void;
}
```

The syntax is close, but method parameters receive special compatibility treatment.

For public callback contracts where strict variance is important, function-valued properties can make the intent clearer.

This is a design-level decision, not a style-only decision.

---

## 28. Generic Compatibility

Generic types are compared based on how their type parameters are actually used.

Consider:

```ts
interface Box<T> {
  get(): T;
}
```

`Box<Dog>` can safely behave like `Box<Animal>` because `T` is used only as an output.

Now:

```ts
interface Sink<T> {
  put(value: T): void;
}
```

The direction changes because `T` is an input.

Mixed positions create more complex behavior.

---

## 29. Covariant Generic Wrapper

Example:

```ts
interface Producer<T> {
  produce(): T;
}

declare const dogProducer: Producer<Dog>;

const animalProducer: Producer<Animal> = dogProducer;
```

This is safe because every result from `dogProducer` is an `Animal`.

Mental model:

```text
producer of narrower thing
→ producer of broader thing
```

---

## 30. Contravariant Generic Wrapper

Example:

```ts
interface Consumer<T> {
  consume(value: T): void;
}

declare const animalConsumer: Consumer<Animal>;

const dogConsumer: Consumer<Dog> = animalConsumer;
```

The animal consumer can handle a dog because every dog is an animal.

Mental model:

```text
consumer of broader thing
→ consumer of narrower thing
```

The direction feels reversed only until you reason from calls the consumer may receive.

---

## 31. Invariance by Design

Sometimes a type should not be substitutable in either direction.

A mutable abstraction often needs both read and write behavior:

```ts
interface Box<T> {
  get(): T;
  set(value: T): void;
}
```

If `Box<Dog>` were freely interchangeable with `Box<Animal>`, writes could violate the Dog invariant.

The conceptual answer is invariance.

In TypeScript, actual assignability can depend on the precise declaration shape, method syntax, and compiler options, so use examples and tests rather than assuming mathematical purity from the abstraction name.

---

## 32. Phantom Type Parameters

A type parameter can affect the static model without appearing in runtime values.

```ts
type Id<T> = string & {
  readonly __entity: T;
};
```

Here `T` has no runtime representation.

This is useful for preventing accidental mixing:

```ts
type UserId = Id<"User">;
type OrderId = Id<"Order">;
```

But it does not create runtime distinction.

The boundary must still validate the underlying string if it is externally supplied.

---

## 33. Variance and Branded Values

Brands often create nominal-like separation:

```ts
type UserId = string & { readonly __brand: "UserId" };
type OrderId = string & { readonly __brand: "OrderId" };
```

This prevents many accidental assignments in TypeScript.

However, runtime code sees:

```text
string
```

unless an actual wrapper object or class is used.

Therefore brands provide static separation, not runtime security.

---

## 34. Unions Are Not “Common Fields Only”

For:

```ts
type A = { kind: "a"; value: string };
type B = { kind: "b"; count: number };

type U = A | B;
```

a consumer can access only properties known to be safe for every member until narrowing establishes a specific variant.

This is one reason discriminated unions are powerful.

The discriminant converts an ambiguous union into a control-flow proof.

---

## 35. Union Assignability

A value assignable to one member can often be assigned to the union:

```ts
const a: A = {
  kind: "a",
  value: "x",
};

const u: A | B = a;
```

The union expresses multiple legal possibilities.

But when passing a union to a function requiring one concrete member, narrowing is needed.

This is the same principle as:

```text
broader possibility set
→ narrower required contract
```

which is why compatibility is directional.

---

## 36. Intersection Types

An intersection:

```ts
type AdminUser = User & AdminRole;
```

models a value satisfying both contracts.

Intersections can become surprising when properties conflict:

```ts
type A = { value: string };
type B = { value: number };

type Impossible = A & B;
```

The resulting property is effectively uninhabitable under ordinary values.

When intersections become deeply contradictory, readability and compiler behavior suffer.

---

## 37. `never` as the Empty Type

`never` represents an impossible value.

It is useful for exhaustive checking:

```ts
function assertNever(value: never): never {
  throw new Error(`Unexpected: ${String(value)}`);
}
```

A discriminated-union switch can then make missing cases visible.

`never` does not mean “null” or “undefined.” It means no value can exist at that type.

---

## 38. Exhaustiveness and Substitutability

Exhaustiveness protects assumptions about closed sets.

```ts
type State = "draft" | "confirmed";
```

When the union expands, exhaustive code should fail to compile until the new case is handled.

This is valuable because static compatibility otherwise tends toward openness.

Use discriminants plus exhaustive handling when the domain truly has a closed finite set.

---

## 39. `any` Breaks the Contract Graph

`any` can effectively bypass many assignability checks.

```ts
const value: any = getExternalValue();

const user: User = value;
```

The compiler cannot protect the edge.

An `any` introduced at the edge can spread through:

```text
controller
→ service
→ repository
→ domain
```

The resulting uncertainty becomes harder to localize.

Use `unknown` at untrusted boundaries and convert it to a precise type through evidence.

---

## 40. Type Assertions Break Proof Chains

Assertions can create types without proving their preconditions:

```ts
const user = value as User;
```

Once asserted, later code may appear perfectly type-safe.

The danger is not the syntax itself. It is the loss of proof provenance.

A principal-level review asks:

```text
What established this invariant?
Can the evidence be pointed to?
Can the assertion be eliminated?
```

---

## 41. Non-Null Assertions

The postfix `!` removes `null`/`undefined` from the static type:

```ts
const element = document.getElementById("app")!;
```

It does not change the runtime value.

If the element is absent, runtime failure remains possible.

Non-null assertions are acceptable when a strong invariant has already been established elsewhere. They are dangerous when used merely to silence the compiler.

---

## 42. Index Access Unsoundness

With:

```ts
const values: string[] = [];
const value = values[0];
```

JavaScript returns:

```text
undefined
```

even though a broad array element type may suggest `string`.

TypeScript can make this safer with `noUncheckedIndexedAccess`, which adds `undefined` to unchecked indexed reads. citeturn746530search2

This is a textbook example of a deliberate trade-off:

```text
ergonomic type
vs
more precise runtime possibility model
```

---

## 43. Dictionary Index Signatures

Consider:

```ts
type Messages = {
  [key: string]: string;
};
```

The type says a string key produces a string.

But ordinary JavaScript objects may not contain every requested key.

With stricter indexed-access settings, TypeScript can surface this uncertainty.

Design principle:

> An index signature should describe actual runtime availability, not merely intended key space.

---

## 44. Array Bounds Are Runtime Facts

The compiler generally does not know that:

```ts
values[10]
```

is within bounds.

For performance and simplicity, ordinary arrays are modeled broadly.

When correctness matters, use APIs that encode the possibility:

```ts
const value = values.at(10);
```

and handle:

```text
T | undefined
```

explicitly.

Do not treat array indexing as proof of element existence.

---

## 45. Enum and Numeric Compatibility Hazards

Numeric enums and plain numbers can interact in ways that surprise developers because numeric values are broadly representable at runtime.

For boundary data, string-literal unions plus explicit runtime validation can be easier to reason about:

```ts
type Role = "admin" | "user";
```

Especially across APIs, explicit wire values reduce accidental coupling between internal enum representations and external data.

---

## 46. Class Compatibility

Classes participate in structural compatibility too, but private and protected members affect compatibility.

Two classes with private members from unrelated declarations are not treated as freely interchangeable merely because their public shape is identical.

This gives class-private state some nominal-like behavior.

The deeper lesson is:

```text
TypeScript is mostly structural
+
certain declarations introduce identity constraints
```

---

## 47. Private State and Identity

Consider:

```ts
class Account {
  #balance = 0;
}
```

ECMAScript private fields are runtime-private and are not merely TypeScript type annotations.

By contrast:

```ts
class Account {
  private balance = 0;
}
```

is a TypeScript access-control construct whose main enforcement is compile-time.

Earlier chapters cover the runtime distinction in greater depth. Here the compatibility lesson is:

```text
static private ≠ runtime private
```

---

## 48. Generic Functions and Universal Claims

A generic function:

```ts
function identity<T>(value: T): T {
  return value;
}
```

makes a promise:

```text
for any T, the output is the same T
```

A dishonest implementation:

```ts
function broken<T>(value: T): T {
  return "not the input" as T;
}
```

can be made to compile with assertions.

The type signature is a contract. The implementation must preserve it for all legal instantiations.

This becomes critical for library authors because generic APIs make universal claims.

---

## 49. Generic Constraints and Variance

A constraint:

```ts
function use<T extends Animal>(value: T) {}
```

says:

```text
T is some subtype of Animal
```

It does not mean:

```text
T is exactly Animal
```

Nor does it automatically establish runtime guarantees.

Variance analysis must consider how `T` is consumed and produced inside the generic abstraction, not merely that it has a constraint.

---

## 50. Conditional Types and Compatibility

Conditional types can transform assignability relationships:

```ts
type Element<T> =
  T extends readonly (infer U)[] ? U : T;
```

The conditional itself is static.

The runtime value still behaves according to JavaScript.

When a type-level transformation appears “proof-like,” ask:

```text
What runtime fact corresponds to this transformation?
```

Sometimes there is one. Sometimes there is none.

---

## 51. Mapped Types Can Preserve or Destroy Intent

Mapped types are powerful:

```ts
type ReadonlyUser = {
  readonly [K in keyof User]: User[K];
};
```

But static modifiers do not automatically enforce external runtime immutability.

Likewise:

```ts
Partial<User>
```

can broaden the set of statically accepted objects without defining the runtime patch semantics.

Always separate static projection from runtime behavior.

---

## 52. `Readonly` Is a Static Contract

`readonly` prevents certain writes through a particular TypeScript view.

```ts
const user: Readonly<User> = source;
```

The underlying object may still be mutable through another reference.

For actual runtime immutability, JavaScript mechanisms such as `Object.freeze` may be needed, though freezing has its own semantics and performance costs.

The static and runtime contracts should be intentionally aligned.

---

## 53. `satisfies` and Compatibility Verification

The `satisfies` operator is useful when you want to verify a value against a type without widening away valuable literal inference.

```ts
const routes = {
  users: "/users",
  orders: "/orders",
} satisfies Record<string, `/${string}`>;
```

This is a compile-time compatibility check.

It does not:

```text
validate JSON
freeze the object
enforce runtime shape
authorize consumers
```

Treat it as a source-authoring tool.

---

## 54. Contextual Typing

TypeScript often infers a function or object literal from its context.

```ts
const handler: (value: Animal) => void = value => {
  console.log(value.name);
};
```

The contextual type shapes what the callback can assume.

This can be powerful but can also hide where a type came from.

For difficult inference problems, temporarily add explicit annotations and inspect the resulting type.

---

## 55. Type Inference Is Not Specification Proof

Inference may produce a type that is technically compatible while still being semantically wrong for the domain.

Example:

```ts
const currency = "USD";
```

The inferred literal type may be narrow.

But:

```ts
const amount = parseFloat(input);
```

produces a number even if the domain needs:

```text
finite
non-negative
currency-specific
minor-unit-aligned
```

Inference captures static facts, not every business invariant.

---

## 56. Unsoundness — What Does It Mean?

A type system is sound when well-typed programs cannot violate the model's guarantees under the defined semantics.

TypeScript intentionally does not aim for complete soundness. Its design prioritizes practical JavaScript compatibility, expressive structural typing, and developer ergonomics.

Examples of known unsoundness areas include:

```text
type assertions
any
unchecked indexed access
some variance behavior
mutable covariance patterns
ambient declarations that do not match runtime reality
```

The correct response is not to reject TypeScript. It is to know where the compiler stops proving facts.

---

## 57. Why TypeScript Chooses Practical Unsoundness

A fully sound type system would impose substantial friction on ordinary JavaScript patterns.

TypeScript instead tries to make common code productive while providing strong static guarantees where practical.

The trade-off is explicit at the language-design level:

```text
more soundness
↔
more friction / less JavaScript compatibility
```

Principal engineers should optimize for the team's actual risk profile rather than treating “sound” as an unconditional design objective.

---

## 58. Soundness Boundary Model

A useful system model is:

```text
TypeScript compiler
   ↓
static confidence
   ↓
runtime boundary
   ↓
actual evidence
   ↓
domain invariant
   ↓
operational behavior
```

Static checks are strongest for code under compiler control.

Confidence decreases when values come from:

```text
any
assertions
external data
ambient declarations
dynamic libraries
runtime mutation
reflection
```

Your architecture should compensate exactly where confidence decreases.

---

## 59. Ambient Declarations and False Confidence

A `.d.ts` file can claim:

```ts
declare function getUser(): User;
```

If the runtime implementation actually returns malformed data, the compiler cannot detect the mismatch.

Ambient declarations are therefore contracts supplied from outside the implementation.

Treat them as trustworthy only when the source system actually enforces the contract.

---

## 60. Declaration Drift

A common enterprise failure is:

```text
runtime changes
   ↓
declaration file remains old
   ↓
TypeScript continues to compile
```

This is particularly dangerous in:

```text
SDKs
generated clients
shared packages
plugin systems
microfrontends
```

Automated contract tests should verify runtime behavior against declarations or schemas where practical.

---

## 61. Library Design: Narrow Inputs, Broad Outputs?

Traditional API design often favors:

```text
accept broad valid inputs
return precise outputs
```

Variance helps explain why.

A consumer should not demand more than its public contract promises.

A producer can return a narrower subtype when callers only require a broader abstraction.

This is closely related to substitutability and Liskov reasoning.

---

## 62. Liskov Connection

If:

```text
Dog <: Animal
```

then substitutability says a Dog should be usable wherever the Animal contract promises are sufficient.

Variance operationalizes this for higher-order abstractions:

```text
producer → covariance
consumer → contravariance
```

But static variance is only one part of behavioral substitutability. Runtime semantics still matter.

---

## 63. Interface Segregation Connection

Variance becomes easier when interfaces are small.

Compare:

```ts
interface Repository<T> {
  get(): T;
  save(value: T): void;
}
```

with separated roles:

```ts
interface Reader<T> {
  get(): T;
}

interface Writer<T> {
  save(value: T): void;
}
```

The smaller interfaces have clearer variance behavior and lower coupling.

ISP and variance reinforce each other.

---

## 64. Composition and Variance

Composition allows you to place variance-sensitive pieces behind explicit boundaries.

```ts
class Pipeline<I, O> {
  constructor(
    readonly run: (input: I) => O,
  ) {}
}
```

The input and output positions are visible.

When a composed type becomes difficult to reason about, decompose the responsibilities into smaller producers and consumers.

---

## 65. Event Handler Variance

Event systems are a classic variance case.

Suppose:

```ts
type Events = {
  userCreated: UserCreated;
  orderCreated: OrderCreated;
};
```

A handler for one event type should not accidentally be substituted for a handler of another event type.

A generic event dispatcher should preserve the key-to-payload relationship:

```ts
type HandlerMap<E> = {
  [K in keyof E]?: (event: E[K]) => void;
};
```

This is a strong use of mapped types plus function variance.

---

## 66. Middleware Variance

Middleware often transforms one request/response type into another.

Conceptually:

```ts
type Middleware<I, O> = (input: I, next: (value: I) => O) => O;
```

As middleware becomes more sophisticated, variance can become difficult.

The practical design is to keep transformations explicit and avoid over-generalizing middleware until the input/output contracts are understood.

---

## 67. Dependency Injection and Variance

A dependency injected as a consumer often benefits from contravariance.

For:

```ts
interface Logger {
  log(message: string): void;
}
```

a dependency that accepts a broader message abstraction is often safer than one that only accepts a narrow subtype.

Likewise a factory that produces a more specific dependency can often satisfy a broader provider contract.

Analyze each port by how values flow through it.

---

## 68. Repository Interfaces

Repository interfaces often combine read and write operations:

```ts
interface Repository<T> {
  find(id: string): T | undefined;
  save(value: T): void;
}
```

The combined interface has mixed variance pressure.

Separating:

```text
Reader<T>
Writer<T>
```

can make substitution rules clearer and improve testing and dependency inversion.

---

## 69. Command Bus Design

A command bus frequently needs to preserve:

```text
command type
→ handler input
→ result type
```

A naive:

```ts
handle(command: any): any
```

destroys compile-time guarantees.

A stronger generic design encodes the relationship, while runtime dispatch still validates external command envelopes.

Static relationships and runtime dispatch must cooperate.

---

## 70. Domain Collections

For domain aggregates, prefer exposing behavior rather than writable mutable arrays:

```ts
class Order {
  #items: readonly OrderLine[] = [];

  get items(): readonly OrderLine[] {
    return this.#items;
  }
}
```

This prevents consumers from using covariance plus mutation to corrupt aggregate state.

Encapsulation and readonly views work together to reduce unsoundness exposure.

---

## 71. Generic Mutable State

Mutable generic state is where variance hazards become especially severe.

Ask:

```text
Can callers read T?
Can callers write T?
Can callers replace the entire value?
Can callers alias the same mutable object?
```

The more directions values flow, the harder substitution becomes.

Immutability simplifies the proof.

---

## 72. Alias Analysis at the Design Level

Two references may point to the same runtime object:

```ts
const a = source;
const b = source;
```

A `readonly` static view on `a` does not prevent mutation through `b`.

Therefore:

```text
readonly view
≠
deep immutable object
```

This matters for shared state, caches, domain entities, and configuration.

Architectural ownership should be clearer than a type modifier alone.

---

## 73. Deep Readonly Is Still Not a Runtime Freeze

A recursive mapped type can model deep readonly:

```ts
type DeepReadonly<T> =
  T extends (...args: any[]) => unknown ? T :
  T extends object ? {
    readonly [K in keyof T]: DeepReadonly<T[K]>;
  } : T;
```

This is static.

It does not recursively freeze runtime objects.

Again:

```text
static immutability model
vs
runtime object mutability
```

must not be conflated.

---

## 74. Function Variance and Security

Security bugs can arise when a callback's accepted input domain is narrower than the caller assumes.

For example, a policy hook declared as:

```ts
(value: Resource) => boolean
```

should not silently become:

```ts
(value: SensitiveResource) => boolean
```

if the dispatcher may pass ordinary resources.

A variance error can become an authorization bug when the callback performs security-sensitive decisions.

---

## 75. Variance and Multi-Tenant Systems

In a multi-tenant application, distinguish:

```text
TenantScopedResource
GlobalResource
BranchScopedResource
```

A handler that can process only branch-scoped resources should not automatically satisfy a handler contract for all resources.

Static brands and discriminants can help, but runtime authorization must still establish tenant/branch scope.

---

## 76. Variance and ERP Policy Handlers

Suppose:

```ts
type Order = {
  tenantId: string;
};

type JewelleryOrder = Order & {
  purity: number;
};
```

A handler capable of processing every `Order` can generally process a `JewelleryOrder`.

A handler requiring `purity` cannot safely process arbitrary `Order`.

This is a direct practical example of contravariant input reasoning.

---

## 77. Runtime Semantics Still Win

Even when the compiler accepts an assignment, runtime values can violate assumptions through:

```text
JavaScript consumers
JSON
reflection
prototype mutation
casts
any
foreign packages
declaration drift
```

Therefore critical invariants should be enforced through runtime boundaries and domain construction.

Static compatibility should reduce bugs, not become a substitute for runtime verification.

---

## 78. Debugging Method: Reduce to Variance

When TypeScript reports a complex generic incompatibility:

1. Identify the outer type constructor.
2. Identify where each type parameter appears.
3. Mark each position as producer or consumer.
4. Determine the safe direction.
5. Expand aliases until the actual function/property shape is visible.
6. Check whether methods receive special treatment.
7. Reduce the example to two concrete types.

This is usually faster than staring at the full compiler diagnostic.

---

## 79. Debugging Method: Expand the Generic

Turn:

```ts
Repository<A>
```

into:

```ts
{
  get(): A;
  save(value: A): void;
}
```

Now the variance problem often becomes obvious.

If `A` appears in both read and write positions, mixed variance is present.

This technique works for:

```text
callbacks
repositories
event handlers
state containers
middleware
dependency ports
```

---

## 80. Debugging Method: Ask “Who Calls Whom?”

For callback errors, ignore the syntax temporarily.

Ask:

```text
Who owns the callback?
Who invokes it?
What argument values can the caller supply?
What values can the callback safely handle?
```

This caller/callee analysis usually reveals contravariance faster than memorizing subtype arrows.

---

## 81. Debugging Method: Check Freshness

When an object unexpectedly fails assignment:

```text
Is it a fresh object literal?
Is excess property checking active?
Is it first assigned to a variable?
Are properties optional?
Are there index signatures?
```

This explains many differences between:

```ts
const x: Target = { ... };
```

and:

```ts
const source = { ... };
const x: Target = source;
```

---

## 82. Debugging Method: Remove Assertions

When a type error disappears after:

```ts
as SomeType
```

that is often evidence that the type system found a real uncertainty.

Temporarily remove assertions and inspect the true source type.

Then ask:

```text
Can the API be redesigned?
Can the boundary validate?
Can control-flow narrowing establish the fact?
Can a generic constraint encode the relationship?
```

Use assertions after understanding the proof gap, not before.

---

## 83. Debugging Method: Enable Strict Flags

For compatibility-heavy code, consider a strong baseline including:

```text
strict
strictFunctionTypes
noUncheckedIndexedAccess
exactOptionalPropertyTypes
useUnknownInCatchVariables
noImplicitOverride
```

The TypeScript compiler options reference documents the exact behavior and interactions of these options. citeturn746530search2

Do not enable flags merely for aesthetics. Understand which runtime risk each flag reduces.

---

## 84. Performance Cost of Stronger Types

Most static checks have little or no production runtime cost because they disappear after compilation.

However, stronger type models can increase:

```text
developer cognitive load
compiler work
type-instantiation complexity
declaration size
IDE latency
```

Advanced conditional/mapped types can become expensive for the compiler.

The principal trade-off is:

```text
proof value
vs
tooling complexity
```

---

## 85. Compiler Performance and Type Complexity

Watch for:

```text
deep recursive conditional types
large unions
cross-product template literals
highly generic libraries
recursive mapped types
```

Symptoms include:

```text
slow builds
IDE lag
large generated declaration files
```

The solution is often to simplify type computations, introduce named intermediate types, or move expensive validation into runtime code.

Do not solve a runtime problem by creating a compiler performance problem.

---

## 86. Security: Type-Level Brands Are Not Secrets

A brand such as:

```ts
type AdminId = string & { readonly __brand: "AdminId" };
```

is not security.

An attacker does not need to satisfy the TypeScript compiler.

Security boundaries need:

```text
authentication
authorization
runtime validation
server-side policy
```

Static brands are useful for preventing accidental internal misuse, not adversarial compromise.

---

## 87. Security: Ambient Types and Third-Party Code

Treat external declarations as claims.

If a package says:

```ts
function getPermission(): Permission;
```

but its runtime can return arbitrary strings, the type is not a security control.

Critical security decisions should be based on runtime-verified values and trusted policy sources.

---

## 88. Security: Callback Confusion

Generic callback systems can create confused-deputy risks if the wrong resource reaches the wrong handler.

Use discriminants, capability-focused interfaces, and explicit dispatch maps.

A security-sensitive dispatcher should validate:

```text
event type
payload
tenant context
actor context
handler capability
```

before executing privileged code.

---

## 89. Production Pattern: Capability-Specific Functions

Instead of:

```ts
function process(value: SuperUnion) {}
```

prefer capability-specific contracts:

```ts
function processCreate(command: CreateCommand) {}
function processCancel(command: CancelCommand) {}
```

This reduces variance complexity and improves the compiler's ability to reject accidental substitution.

---

## 90. Production Pattern: Read/Write Separation

A powerful pattern is:

```text
Reader<T>
Writer<T>
```

rather than:

```text
Repository<T>
```

when both directions have different substitution needs.

This aligns with:

```text
variance
interface segregation
dependency inversion
testability
```

---

## 91. Production Pattern: Immutable Inputs

Prefer:

```ts
type Handler<T> = (input: Readonly<T>) => Result;
```

or:

```ts
type Handler<T> = (input: T) => Result;
```

where mutation is unnecessary.

Immutable inputs reduce aliasing and variance hazards.

Do not make everything deeply readonly by default without considering ergonomics and performance; model ownership intentionally.

---

## 92. Production Pattern: Explicit Adapters

When two generic interfaces do not align safely, an adapter is often better than a cast.

```ts
function adaptDogHandler(
  handler: (dog: Dog) => void,
): (animal: Animal) => void {
  return (animal) => {
    if (!isDog(animal)) {
      throw new Error("Expected dog");
    }
    handler(animal);
  };
}
```

The adapter makes the runtime precondition explicit.

This is safer than:

```ts
handler as (animal: Animal) => void
```

---

## 93. Production Pattern: Type-Safe Event Registry

A registry can preserve event-key relationships:

```ts
type Events = {
  created: { id: string };
  deleted: { id: string; reason: string };
};

type Handlers<E> = {
  [K in keyof E]?: (event: E[K]) => void;
};
```

Then the compiler prevents wiring:

```text
deleted handler → created event
```

At runtime, the dispatcher still needs to validate incoming messages before indexing into the registry.

---

## 94. Production Pattern: Boundary-to-Domain Proof

Use a deliberate sequence:

```text
unknown
→ runtime decoder
→ branded/validated primitive
→ command
→ domain constructor
→ aggregate
```

Each transition should have a reason.

A type should become narrower only when evidence justifies the narrowing.

This makes the system's proof chain inspectable during code review.

---

## 95. Implementation From Scratch: Variance Playground

Create:

```ts
type Animal = { name: string };
type Dog = Animal & { breed: string };

type Producer<T> = () => T;
type Consumer<T> = (value: T) => void;
type MutableBox<T> = {
  get(): T;
  set(value: T): void;
};
```

Write every plausible assignment and predict:

```text
compiles
or
fails
```

Then explain each result using actual value flow.

Do not memorize a variance table until you have reasoned through the examples.

---

## 96. Implementation From Scratch: Strict Event Bus

Build:

```ts
interface Events {
  userCreated: { id: string };
  orderCreated: { id: string };
}

class EventBus<E extends Record<string, unknown>> {
  // register handlers
  // publish events
}
```

Requirements:

```text
key-specific payloads
no any
read-only event data
handler type safety
runtime unknown payload validation
```

Then test cross-wiring failures.

---

## 97. Implementation From Scratch: Read/Write Repository Split

Build:

```ts
interface Reader<T> {
  find(id: string): T | undefined;
}

interface Writer<T> {
  save(value: T): void;
}
```

Then create:

```text
Animal reader
Dog reader
Animal writer
Dog writer
```

Experiment with assignments and explain which are safe.

Repeat with method syntax and function-property syntax.

---

## 98. Implementation From Scratch: Safe Adapter

Create an explicit adapter for a deliberately unsafe callback assignment.

Requirements:

```text
static contract
runtime guard
structured error
unit tests
```

The purpose is to show that adaptation can recover safety without weakening the whole type system.

---

## 99. Implementation From Scratch: Mutable Box Failure

Implement:

```ts
class Box<T> {
  constructor(private value: T) {}

  get(): T {
    return this.value;
  }

  set(value: T): void {
    this.value = value;
  }
}
```

Try to make:

```text
Box<Dog>
```

behave as:

```text
Box<Animal>
```

Then demonstrate why mutation creates a soundness problem.

Afterward implement:

```ts
ReadonlyBox<T>
```

and compare.

---

## 100. Debugging Exercise 1

Given:

```ts
type Animal = { name: string };
type Dog = Animal & { breed: string };

type Handler<T> = (value: T) => void;

declare const dogHandler: Handler<Dog>;
declare const animalHandler: Handler<Animal>;
```

Determine which assignments should be accepted under strict function checking.

Then explain the answer only in terms of:

```text
possible caller inputs
callback requirements
```

---

## 101. Debugging Exercise 2

Explain why these declarations can behave differently:

```ts
interface MethodHandler<T> {
  handle(value: T): void;
}

interface PropertyHandler<T> {
  handle: (value: T) => void;
}
```

Write a minimal reproducible example under strict compiler settings.

Then state which form you would choose for a security-sensitive callback API and why.

---

## 102. Debugging Exercise 3

Find the hole:

```ts
type User = { id: string };

function loadUser(): User {
  return JSON.parse(text) as User;
}
```

The bug is not merely “using `as`.”

Identify the missing proof chain:

```text
text
→ parse
→ validate
→ construct/use
```

Then add a decoder that returns structured validation errors.

---

## 103. Debugging Exercise 4

Find the indexing hazard:

```ts
function first(values: string[]): string {
  return values[0];
}
```

Explain why the signature can be stronger than the runtime behavior.

Refactor under `noUncheckedIndexedAccess` and decide whether the new return type should be:

```text
string | undefined
```

or whether an explicit domain precondition should reject empty arrays.

---

## 104. Debugging Exercise 5

Given:

```ts
type Patch = {
  name?: string;
};

function update(input: Patch) {
  if (input.name === undefined) {
    // ...
  }
}
```

Determine whether this distinguishes:

```text
missing
explicit undefined
```

under the project's TypeScript configuration.

Then compare the semantics under `exactOptionalPropertyTypes`.

---

## 105. Code Review Exercise: Broad Interface

Review:

```ts
interface Repository<T> {
  get(id: string): T;
  save(value: T): void;
  delete(value: T): void;
  watch(listener: (value: T) => void): void;
}
```

Questions:

```text
Is T both consumed and produced?
Should the interface be split?
Does watch require strict callback variance?
Can readonly views reduce mutation risk?
Does every consumer need all capabilities?
```

Produce a redesigned set of interfaces and defend the variance consequences.

---

## 106. Code Review Exercise: Cast-Based Event Bus

Review:

```ts
function publish(event: { type: string; payload: unknown }) {
  const handler = handlers[event.type] as (payload: any) => void;
  handler(event.payload);
}
```

Identify every broken guarantee.

A better design should establish:

```text
event type
→ schema for that event
→ decoded payload
→ correctly typed handler
```

The cast should disappear from the critical dispatch path.

---

## 107. Code Review Exercise: Mutable Domain Exposure

Review:

```ts
class Order {
  items: OrderLine[] = [];
}
```

Ask:

```text
Who owns items?
Can callers mutate it?
Can callers insert invalid line values?
Can aliases retain the array?
Can runtime data bypass construction?
```

A stronger design uses encapsulation, readonly views, and domain methods.

---

## 108. Predict-the-Output Lab 1

Code:

```ts
type Animal = { name: string };
type Dog = Animal & { breed: string };

const dog: Dog = {
  name: "Rex",
  breed: "Shepherd",
};

const animal: Animal = dog;

console.log(animal.name);
console.log("breed" in animal);
```

Prediction:

```text
Rex
true
```

Static narrowing concerns what the compiler exposes. Runtime property existence is independent and can reveal additional fields.

---

## 109. Predict-the-Output Lab 2

Code:

```ts
class User {
  constructor(readonly id: string) {}
}

const value: unknown = new User("1");

console.log(value instanceof User);
console.log((value as User).id);
```

Prediction:

```text
true
1
```

The value is actually a `User` instance. The assertion does not create that identity; the constructor call did.

---

## 110. Predict-the-Output Lab 3

Code:

```ts
const values: string[] = [];

console.log(values[0]);
console.log(values.at(0));
```

Prediction:

```text
undefined
undefined
```

The example demonstrates why static element types should not be interpreted as proof of index existence.

---

## 111. Predict-the-Output Lab 4

Code:

```ts
const handler = (animal: Animal) => animal.name;

console.log(handler({
  name: "Rex",
  breed: "Shepherd",
}));
```

Prediction:

```text
Rex
```

A broader consumer can handle a narrower produced value.

---

## 112. Predict-the-Output Lab 5

Code:

```ts
const value = { id: "1", extra: true };

function useUser(user: { id: string }) {
  console.log(user.id);
}

useUser(value);
```

Prediction:

```text
1
```

The existing variable is structurally assignable despite carrying an extra property.

This is different from directly supplying a fresh object literal under excess property checking.

---

## 113. Interview Questions — Foundation

1. What is structural typing?
2. Why is compatibility directional?
3. What is covariance?
4. What is contravariance?
5. Why are function outputs covariant?
6. Why are function inputs contravariant?
7. What is bivariance?
8. What does `strictFunctionTypes` change?
9. Why can method syntax differ from function-property syntax?
10. What is excess property checking?

---

## 114. Interview Questions — Intermediate

1. Why are mutable arrays a variance concern?
2. Why does `ReadonlyArray<T>` improve safety?
3. What does `noUncheckedIndexedAccess` solve?
4. Why can `as` create false confidence?
5. What is the difference between structural compatibility and behavioral subtyping?
6. Why can a repository interface have mixed variance?
7. What is a phantom type parameter?
8. Why are branded types not runtime security?
9. What does `satisfies` verify?
10. Why are ambient declarations dangerous when stale?

---

## 115. Interview Questions — Principal Level

1. Explain TypeScript's deliberate unsoundness philosophy.
2. How would you design a callback API with predictable variance?
3. When should you split a generic read/write abstraction?
4. How would you review an event bus for variance and runtime safety?
5. How does variance connect to Liskov substitution?
6. How would you prevent `any` from contaminating a large codebase?
7. Which strict compiler flags materially reduce runtime contract risk?
8. How do you balance compiler complexity against stronger types?
9. Where would runtime validation compensate for static unsoundness?
10. How would you explain TypeScript unsoundness to a staff engineer without claiming the language is “broken”?

---

## 116. Comparison: Structural vs Nominal Thinking

| Concern | Structural typing | Nominal typing |
|---|---|---|
| primary criterion | shape/capabilities | declared identity |
| JS interoperability | excellent | requires adaptation |
| accidental compatibility | higher | lower |
| flexible library composition | strong | more constrained |
| domain identity | needs explicit modeling | naturally represented |
| runtime guarantee | neither by itself | neither by itself |

TypeScript is primarily structural, with selected nominal-like behavior through private/protected members, unique symbols, and explicit branding patterns.

---

## 117. Comparison: Method vs Function Property

| Surface | Typical callback variance behavior | Design implication |
|---|---|---|
| method | special compatibility rules can apply | convenient but potentially looser |
| function-valued property | strict function variance applies | clearer for callback contracts |
| standalone function type | strict parameter comparison | explicit and composable |

Choose based on semantic needs, not formatting preference.

---

## 118. Comparison: `any` vs `unknown` vs Assertion

| Tool | Compiler restriction | Runtime validation | Best use |
|---|---|---|---|
| `any` | very low | none | narrowly isolated legacy/interoperability cases |
| `unknown` | high until narrowed | indirect through checks | untrusted input |
| `as T` | bypasses checks | none | justified compiler escape hatch |

The safer default for external input is `unknown`, followed by runtime evidence.

---

## 119. Comparison: Mutable vs Readonly Generic APIs

| Design | Read | Write | Variance reasoning | Typical safety |
|---|---:|---:|---|---|
| Producer | yes | no | covariance-friendly | high |
| Consumer | no | yes | contravariance-friendly | high |
| Readonly collection | yes | no | simpler | high |
| Mutable container | yes | yes | mixed/invariant pressure | lower |

---

## 120. Principal Decision Framework

For a compatibility-heavy API, evaluate:

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
Who produces values?
Who consumes values?
Who mutates values?
Who owns aliases?
What static assumptions exist?
What runtime evidence exists?
Which strictness flags are active?
Where can unsoundness enter?
Where should adapters live?
```

This turns variance from an academic topic into an architecture tool.

---

## 121. Canonical Architecture

A strong TypeScript contract pipeline looks like:

```text
external data
   ↓
unknown
   ↓
runtime decoder
   ↓
precise structural type
   ↓
brand / value-object constructor
   ↓
domain contract
   ↓
producer / consumer interfaces
   ↓
polymorphic behavior
   ↓
controlled side effects
```

At every arrow, ask what evidence justifies the type transition.

That question is the practical meaning of soundness discipline.

---

## 122. Specification and Source Discipline

For this chapter, distinguish:

```text
TypeScript language/type-system behavior
compiler options
JavaScript runtime behavior
library declarations
application contracts
```

Do not attribute TypeScript-only behavior to ECMAScript.

Primary sources should be the TypeScript Handbook and compiler-options reference for type compatibility and strictness semantics. citeturn746530search2

For runtime behavior, consult ECMAScript and the relevant host documentation. TypeScript documentation itself emphasizes that the Handbook is a guide rather than a complete formal language specification. citeturn746530search3

---

## 123. Retrieval Record

From memory, explain:

```text
structural compatibility
width/depth
variance
function input/output variance
strictFunctionTypes
method bivariance
excess property checking
array covariance
noUncheckedIndexedAccess
type assertions
any
ambient declaration drift
deliberate unsoundness
```

Mark:

- `[ ] Not Started`
- `[~] In Progress`
- `[?] Needs Revision`
- `[+] Completed`
- `[*] Mastered`

Only mark `[*]` after you can derive the assignment direction rather than reciting it.

---

## 124. Mastery Gate

You have mastered this chapter when you can:

```text
[ ] Reduce structural compatibility to required members
[ ] Explain why compatibility is directional
[ ] Derive variance from value flow
[ ] Explain callback contravariance
[ ] Explain producer covariance
[ ] Explain method bivariance
[ ] Diagnose excess property checking
[ ] Diagnose array/indexing unsoundness
[ ] Use strict compiler flags intentionally
[ ] Separate static compatibility from runtime safety
[ ] Design variance-friendly interfaces
[ ] Split read/write abstractions when useful
[ ] Remove unsafe casts through adapters or validation
[ ] Explain deliberate TypeScript unsoundness
[ ] Defend these decisions at principal level
```

---

## 125. Concept Connections

This chapter connects directly to:

```text
Structural typing
   → foundation of compatibility

Generics
   → variance of type constructors

Function types
   → callback input/output variance

Control-flow narrowing
   → establishing runtime evidence

Runtime validation
   → compensating for static/runtime gaps

Encapsulation
   → controlling mutation and aliases

Abstraction
   → designing stable producer/consumer contracts

Composition
   → making value flow explicit

LSP
   → behavioral substitutability

ISP
   → smaller variance-friendly contracts

DIP
   → ports defined around capabilities

OCP
   → adapters absorbing incompatible implementations
```

---

## 126. What This Chapter Is Really Teaching

The deepest lesson is:

> **Type compatibility is a statement about what one piece of code may safely assume from another static type. It is not a guarantee that the runtime world obeys those assumptions.**

Variance gives a precise language for value flow:

```text
produced values
→ covariance

consumed values
→ contravariance

read + write
→ mixed/invariant pressure
```

TypeScript deliberately accepts some unsoundness because it is designed for practical JavaScript development.

A principal engineer therefore does not ask:

> “Can I make the compiler accept this?”

The better questions are:

```text
What substitution am I allowing?
What values can actually flow here?
Who can mutate them?
What runtime evidence exists?
Where can the type be wrong?
What architecture contains the risk?
```

That is how structural typing becomes a design instrument rather than merely a compiler feature.

---

## 127. Three-Track Study System

### Track A — Core Theory

Study compatibility from first principles:

```text
required members
→ direction of assignability
→ value flow
→ variance
→ substitutability
→ soundness boundary
```

Do not memorize “covariant/contravariant” arrows before you can answer “who produces and who consumes the value?”

### Track B — Implementation

Progress through:

```text
guided assignment matrix
→ variance playground
→ generic event bus
→ read/write split
→ safe adapter
→ production contract
```

Every implementation must include compiler-error analysis and runtime tests where relevant.

### Track C — Interview / Reasoning

Practice:

```text
predict
→ reduce
→ derive
→ compare
→ refactor
→ defend
```

For difficult compiler diagnostics, reduce the problem until only two concrete types remain.

---

## 128. Golden Rules

1. Compatibility is directional.
2. Structural typing compares required capabilities, not names alone.
3. Producers tend toward covariance.
4. Consumers tend toward contravariance.
5. Mutable read/write abstractions create mixed variance pressure.
6. Method syntax can receive different variance treatment from function-valued properties.
7. Excess property checking is not universal exact-object typing.
8. `readonly` reduces mutation-based unsoundness.
9. `unknown` localizes uncertainty; `any` spreads it.
10. Assertions create static claims but not runtime evidence.
11. Strict compiler options are architectural guardrails.
12. Type compatibility is not behavioral correctness.

---

## 129. Completion Snapshot

| Area | Status | Evidence |
|---|---|---|
| structural assignability | `[ ]` | explained from first principles |
| function variance | `[ ]` | assignment matrix |
| method bivariance | `[ ]` | compiler experiment |
| excess property checking | `[ ]` | fresh-vs-variable test |
| readonly safety | `[ ]` | mutable/readonly comparison |
| indexed access | `[ ]` | strict-flag exercise |
| deliberate unsoundness | `[ ]` | written explanation |
| event bus | `[ ]` | no-reference implementation |
| adapters | `[ ]` | unsafe cast removed |
| principal defense | `[ ]` | architecture review |

Reading alone is not completion evidence.

---

## 130. Final Principal Review

Before approving a generic TypeScript abstraction, answer:

```text
1. What does the type parameter represent?
2. Where is it produced?
3. Where is it consumed?
4. Can it be mutated?
5. Can aliases observe mutation?
6. What substitutions are safe?
7. Does method syntax affect variance?
8. Which strict compiler options are active?
9. Where can any/assertions enter?
10. What runtime evidence exists?
11. Which invariants remain outside the compiler?
12. Would splitting the interface simplify variance?
13. Would readonly semantics reduce risk?
14. Would an explicit adapter be clearer than a cast?
15. Does the design remain understandable five years from now?
```

The strongest type design is not the one with the most advanced generic machinery. It is the one whose static promises, runtime behavior, mutation model, and domain semantics line up closely enough that the next engineer can reason about them without guessing.
