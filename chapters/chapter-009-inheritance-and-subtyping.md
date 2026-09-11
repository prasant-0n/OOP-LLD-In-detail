# Chapter 9 — Inheritance & Subtyping

> **JavaScript OOP + LLD Mastery**
>
> Inheritance is one of the easiest OOP concepts to use and one of the easiest to misuse. This chapter separates the **mechanics of inheritance** from the **design meaning of subtyping**.
>
> The central question is not:
>
> > “Can this class extend that class?”
>
> It is:
>
> > **“Can every valid instance of the subtype safely be used wherever the parent abstraction is expected?”**
>
> That question leads directly to substitutability, Liskov Substitution Principle, behavioral contracts, polymorphism, composition, and maintainable LLD.

**Status:** `[ ] Not Started`

# 1. Learning Objectives

```text
[ ] define inheritance
[ ] distinguish inheritance from subtyping
[ ] explain prototype inheritance in JavaScript
[ ] explain class inheritance with extends
[ ] explain super
[ ] explain method overriding
[ ] explain shadowing
[ ] explain constructor inheritance
[ ] explain inherited state and behavior
[ ] explain subtype contracts
[ ] explain behavioral substitutability
[ ] explain the Liskov Substitution Principle
[ ] identify precondition strengthening problems
[ ] identify postcondition weakening problems
[ ] identify invariant violations
[ ] identify exception-contract violations
[ ] identify semantic subtype failures
[ ] explain fragile base class problems
[ ] identify inheritance hierarchy smells
[ ] compare inheritance with composition
[ ] compare abstract base classes with interfaces
[ ] explain JavaScript's structural/dynamic aspects
[ ] design shallow and deep hierarchies
[ ] reason about inheritance depth
[ ] debug override failures
[ ] design substitutable subtypes
[ ] implement inheritance examples
[ ] review inheritance-based designs
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
Chapter 8 — Abstraction & Stable Object Interfaces
objects
prototypes
constructors
classes
private state
abstraction
contracts
```

# 3. What Is It?

Inheritance is a mechanism for reusing and specializing behavior across related object types.

In JavaScript classes:

```js
class Animal {
  speak() {
    return "sound";
  }
}

class Dog extends Animal {
  bark() {
    return "woof";
  }
}
```

the resulting prototype chain is conceptually:

```text
dog
 ↓
Dog.prototype
 ↓
Animal.prototype
 ↓
Object.prototype
 ↓
null
```

But inheritance has two distinct meanings:

```text
implementation inheritance
→ reuse behavior/state

subtyping
→ promise compatibility with a more general abstraction
```

These are related but not identical.

# 4. Why Does It Exist?

Inheritance can express:

```text
shared behavior
shared contract
specialization
polymorphism
common lifecycle
```

For example:

```text
PaymentMethod
   ↓
CardPayment
CashPayment
```

may be useful when every subtype genuinely satisfies the parent contract.

But inheritance also introduces:

```text
coupling
override dependencies
initialization dependencies
fragile base-class risks
```

Therefore the presence of shared code alone is not enough to justify inheritance.

# 5. Mental Model

A subtype relationship can be visualized as:

```text
GENERAL CONTRACT
      │
      ▼
 Parent abstraction
      │
      │ subtype promise
      ▼
 Child abstraction
      │
      ▼
 concrete behavior
```

A valid subtype should preserve the expectations established by the parent.

This gives:

```text
parent contract
      ↓
subtype must remain compatible
      ↓
client remains safe
```

# 6. Core Rules

## Rule 1 — Inheritance Is More Than Code Reuse

```js
class Dog extends Animal {}
```

should ideally express a meaningful semantic relationship.

Do not choose inheritance simply because:

```text
the child needs some parent methods.
```

## Rule 2 — Subtypes Must Preserve Parent Expectations

If a client expects:

```js
animal.speak()
```

the subtype should not unexpectedly:

```text
break the return contract
throw unrelated errors
require impossible inputs
invalidate expected invariants.
```

## Rule 3 — Overriding Changes Behavior for Existing Clients

When a child overrides a method, every client using the parent abstraction must remain valid.

This is the practical heart of substitutability.

## Rule 4 — `super` Does Not Make a Design Safe

Calling:

```js
super.method()
```

can reuse behavior, but does not prove that the subtype is semantically valid.

## Rule 5 — A Deeper Hierarchy Means More Coupling

Each additional inheritance level creates another dependency:

```text
Grandparent
    ↓
Parent
    ↓
Child
    ↓
ConcreteChild
```

Deep hierarchies increase reasoning and change-propagation cost.

## Rule 6 — Composition Is a First-Class Alternative

If the relationship is:

```text
has-a
uses-a
delegates-to
contains-a
```

composition is often clearer than inheritance.

# 7. Syntax

## Basic Inheritance

```js
class Dog extends Animal {
  speak() {
    return "bark";
  }
}
```

## Super Method

```js
class Dog extends Animal {
  speak() {
    return super.speak();
  }
}
```

## Super Constructor

```js
class Dog extends Animal {
  constructor(name) {
    super(name);
  }
}
```

# 8. Basic Examples

## Example 1 — Simple Inheritance

```js
class Animal {
  speak() {
    return "sound";
  }
}

class Dog extends Animal {
  bark() {
    return "woof";
  }
}

const dog = new Dog();

console.log(dog.speak());
console.log(dog.bark());
```

**Prediction**

```text
sound
woof
```

**Trace**

```text
dog.speak
 ↓
Dog.prototype → not found
 ↓
Animal.prototype → found
```

## Example 2 — Override

```js
class Animal {
  speak() {
    return "sound";
  }
}

class Dog extends Animal {
  speak() {
    return "bark";
  }
}

console.log(new Dog().speak());
```

Prediction:

```text
bark
```

## Example 3 — `super`

```js
class Animal {
  speak() {
    return "sound";
  }
}

class Dog extends Animal {
  speak() {
    return super.speak() + " bark";
  }
}

console.log(new Dog().speak());
```

Prediction:

```text
sound bark
```

# 9. Execution Walkthrough

Consider:

```js
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    return `${this.name} makes a sound`;
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);
    this.breed = breed;
  }

  speak() {
    return `${this.name} barks`;
  }
}
```

Construction:

```text
new Dog("Bruno", "Labrador")
        ↓
Dog constructor starts
        ↓
super("Bruno")
        ↓
Animal constructor
        ↓
name initialized
        ↓
Dog constructor continues
        ↓
breed initialized
        ↓
Dog instance ready
```

Method lookup:

```text
dog.speak
 ↓
Dog.prototype.speak
 ↓
found
```

# 10. Internal Mechanics

In class inheritance, two prototype relationships are important.

Instance side:

```text
Dog.prototype
     ↓
Animal.prototype
```

Constructor/class side also has an inheritance relationship.

This allows mechanisms such as:

```js
super()
super.method()
```

to work according to class semantics.

The object still participates in ordinary JavaScript prototype behavior.

# 11. ECMAScript / Specification Semantics

ECMAScript defines class inheritance through:

```text
extends
[[Prototype]]
prototype relationships
constructor semantics
super property references
derived constructor rules
```

The language defines observable behavior.

Engine-specific implementation strategies such as:

```text
hidden classes
inline caches
prototype validity tracking
```

are implementation details.

# 12. Advanced Behavior

## 12.1 Method Overriding

```js
class Parent {
  run() {
    return "parent";
  }
}

class Child extends Parent {
  run() {
    return "child";
  }
}
```

The child prototype owns the overriding method.

Lookup stops there.

## 12.2 Field Shadowing

```js
class Parent {
  value = 1;
}

class Child extends Parent {
  value = 2;
}
```

Both initialization and method behavior must be considered independently.

A field override is not the same concept as method override.

## 12.3 Method Lookup vs `this`

```js
class Parent {
  name = "parent";

  greet() {
    return this.name;
  }
}

class Child extends Parent {
  name = "child";
}

console.log(new Child().greet());
```

The inherited method uses the receiver:

```text
Child instance
```

for `this`.

# 13. Subtyping

Subtyping means:

```text
a value of subtype S
can be used where supertype T is expected
without violating T's behavioral contract.
```

This is stronger than:

```text
S extends T
```

or:

```text
S has the same methods.
```

The key is client behavior.

# 14. Liskov Substitution Principle

The Liskov Substitution Principle can be stated operationally:

> **Objects of a subtype should be usable wherever objects of the parent type are expected without breaking the client-visible correctness assumptions of the parent contract.**

This means a subtype should respect:

```text
preconditions
postconditions
invariants
observable behavior
error expectations
```

# 15. Preconditions

A subtype should not arbitrarily require more than the parent contract required.

Parent:

```text
withdraw(amount)
requires amount > 0
```

Bad subtype:

```text
withdraw(amount)
requires amount > 0 AND amount % 100 === 0
```

If clients were never required to provide multiples of 100, the subtype has strengthened the precondition.

That can violate substitutability.

# 16. Postconditions

A subtype should not weaken guarantees promised by the parent.

Parent contract:

```text
save()
→ item is persisted when successful
```

Bad subtype:

```text
save()
→ places item only in an in-memory buffer
```

if the parent contract promised durable persistence.

The subtype has weakened the postcondition.

# 17. Invariants

Suppose:

```text
Account.balance >= 0
```

is part of the parent invariant.

A subtype that allows:

```text
balance = -1000
```

without changing the contract is not substitutable.

The subtype has violated the state expectations.

# 18. Behavioral Compatibility

A subtype can satisfy every TypeScript signature and still violate the semantic contract.

Example:

```ts
interface Queue {
  enqueue(item: Item): void;
  dequeue(): Item | undefined;
}
```

A class can implement the interface while:

```text
dequeue()
```

returns random items.

The shape matches.

The behavior does not.

Therefore:

```text
structural compatibility
≠
behavioral substitutability.
```

# 19. Exception Compatibility

Consider a parent contract that allows:

```text
invalid input → domain validation error
```

A subtype that unexpectedly throws:

```text
network error
```

for the same valid operation may break clients.

Substitutability includes error behavior where the contract makes errors observable.

# 20. Output Compatibility

A subtype should preserve meaningful output expectations.

Parent:

```text
calculateTotal()
→ finite non-negative monetary value
```

A subtype that returns:

```text
string
NaN
currency-incompatible value
```

is not behaviorally compatible even if the method name matches.

# 21. Temporal Compatibility

Some interfaces contain timing expectations.

Example:

```text
cache.get()
→ returns immediately from local state
```

A subtype that turns every call into:

```text
remote network request
```

may preserve syntax while violating important performance or side-effect expectations.

When timing is part of the contract, it is part of substitutability.

# 22. Mutable State and Subtyping

Inheritance becomes harder to reason about when subclasses mutate shared parent state.

Questions:

```text
Who owns the state?
Who can override transitions?
Which invariants belong to the parent?
Can the child invalidate them?
```

This is why protected mutable state can create tight coupling.

JavaScript's private fields can limit direct subclass access, but that does not automatically solve semantic coupling.

# 23. Protected-State Problem

JavaScript does not have the same `protected` keyword semantics as languages such as Java.

Developers may simulate protected conventions with:

```text
_this
underscored fields
symbols
```

but these are not identical language-level mechanisms.

A subclass that depends on internal parent representation is tightly coupled to that representation.

# 24. Fragile Base Class Problem

A base class can change:

```text
method ordering
internal calls
invariants
state representation
constructor behavior
```

and accidentally break subclasses.

Example:

```js
class Base {
  process() {
    this.step1();
    this.step2();
  }
}
```

A subclass may override:

```js
step1()
```

assuming:

```text
step2()
```

is called under a certain invariant.

A future base-class change can break that assumption.

This is the fragile base class problem.

# 25. Template Method Pattern Risk

Template-method style:

```js
class BaseProcessor {
  process() {
    this.load();
    this.transform();
    this.save();
  }

  load() {}
  transform() {}
  save() {}
}
```

can be useful.

But each override point becomes a contract.

More extension points mean:

```text
more subclass knowledge
more coupling
more lifecycle assumptions.
```

Use extension points deliberately.

# 26. Inheritance for Code Reuse

Bad motivation:

```text
Child needs utility method from Parent.
```

This often creates a semantic lie:

```text
Child is-a Parent
```

when the actual relationship is:

```text
Child uses helper.
```

Prefer:

```text
composition
module
utility
delegation
```

when there is no true subtype relationship.

# 27. Composition Alternative

Instead of:

```js
class PdfReport extends ReportEngine {}
```

consider:

```js
class PdfReport {
  constructor(renderer) {
    this.renderer = renderer;
  }
}
```

The report has a renderer.

Now:

```text
Report
 └── uses Renderer
```

instead of:

```text
PdfReport is-a ReportEngine
```

This reduces inheritance coupling.

# 28. Inheritance vs Composition

| Question | Inheritance | Composition |
|---|---|---|
| Reuse mechanism | delegation through prototype hierarchy | object collaboration |
| Coupling | often higher | often lower |
| Runtime variation | less flexible | flexible |
| State ownership | shared hierarchy rules | explicit collaborators |
| Override risk | high | lower |
| Construction coupling | potentially high | explicit dependencies |
| Best for | genuine subtype relationship | has-a / uses-a relationship |

Neither is universally better.

# 29. Deep Hierarchies

A hierarchy such as:

```text
Entity
 ↓
BusinessEntity
 ↓
AuditedEntity
 ↓
Document
 ↓
Order
 ↓
OnlineOrder
 ↓
MarketplaceOrder
```

may look organized but can become difficult to reason about.

Each level adds:

```text
state
methods
constructor rules
invariants
override interactions
```

Prefer flatter hierarchies where possible.

# 30. Advanced Behavior — Multiple Inheritance

JavaScript classes do not provide traditional multiple class inheritance.

This:

```js
class C extends A, B {}
```

is not valid.

Composition and mixin techniques can combine behavior where needed.

Multiple inheritance problems remain relevant conceptually:

```text
diamond inheritance
conflicting state
ambiguous behavior
```

# 31. Mixins

Mixins can add behavior without a traditional single-parent hierarchy.

Example:

```js
const Timestamped = Base => class extends Base {
  createdAt = new Date();
};

const Identified = Base => class extends Base {
  id = crypto.randomUUID();
};
```

Mixins are powerful but can still create:

```text
initialization coupling
method collisions
implicit dependencies
deep generated inheritance chains.
```

They are not a free escape from inheritance complexity.

# 32. Interface Inheritance

TypeScript can express interface extension:

```ts
interface Animal {
  speak(): string;
}

interface Dog extends Animal {
  bark(): string;
}
```

This is a type-level relationship.

It does not automatically create runtime prototype inheritance.

Therefore distinguish:

```text
type/interface inheritance
vs
class/prototype inheritance.
```

# 33. Abstract Classes

TypeScript supports:

```ts
abstract class PaymentMethod {
  abstract charge(amount: number): Promise<void>;
}
```

An abstract class can provide:

```text
shared implementation
shared state
abstract behavior
```

It still participates in JavaScript class/prototype runtime semantics after compilation.

Use it when both:

```text
contract
+
shared base implementation/state
```

are useful.

# 34. Abstract Class vs Interface

| Feature | Interface | Abstract class |
|---|---|---|
| Runtime object | no | yes |
| Shared implementation | no | yes |
| State | no runtime state | can hold state |
| Multiple type inheritance | flexible | single class ancestry |
| Runtime identity | no | yes |
| Main purpose | contract | contract + implementation |

# 35. Edge Cases

## Base Constructor Calls Overridden Method

Can observe partially initialized derived state.

## Parent Private Field

Subclass cannot directly access the parent's private field name.

## Constructor Return

Derived construction has additional rules around the returned receiver.

## Prototype Mutation

Changing prototype relationships can affect behavior and `instanceof`.

## Static Inheritance

Static properties/methods and instance prototype relationships are separate inheritance surfaces.

## Field Shadowing

A child field can establish own state that differs from parent initialization.

# 36. Common Misconceptions

```text
"extends means the child is automatically a good subtype."
"inheritance is the preferred reuse mechanism."
"super makes overrides safe."
"matching method names means substitutability."
"TypeScript interface compatibility proves behavioral compatibility."
"deep hierarchies are more reusable."
"child classes should access parent internals directly."
"private parent fields should be duplicated in the child."
"composition is only for objects without inheritance."
"abstract classes are just interfaces with syntax."
```

# 37. Common Mistakes

```text
[ ] inheriting for convenience
[ ] violating parent preconditions
[ ] weakening parent postconditions
[ ] breaking parent invariants
[ ] exposing fragile protected state
[ ] calling overridable methods from constructors
[ ] creating deep hierarchies
[ ] relying on super as a correctness mechanism
[ ] mixing unrelated responsibilities in a base class
[ ] using inheritance to model has-a relationships
[ ] creating mixin chains without explicit dependency contracts
```

# 38. Comparison With Related Concepts

| Concept | Core relationship |
|---|---|
| Inheritance | subtype/specialization and delegation |
| Composition | has-a/uses-a |
| Delegation | forward responsibility to another object |
| Interface | behavioral/type contract |
| Abstract class | contract + shared base implementation |
| Mixin | behavior composition through generated inheritance |
| Adapter | translate one interface to another |
| Polymorphism | use common contract with varying implementations |
| Dependency injection | supply collaborators externally |

# 39. Performance Considerations

Potential considerations:

```text
prototype lookup depth
overridden method call patterns
dynamic prototype changes
per-instance fields across hierarchy
generated mixin chains
```

Modern engines can optimize common inheritance patterns heavily.

Do not assume:

```text
inheritance = slow
composition = fast
```

as a universal rule.

The important engineering concern is usually:

```text
reasoning complexity
change coupling
and actual measured workload.
```

# 40. Memory Considerations

Hierarchy-level state can include:

```text
base fields
derived fields
private fields
per-instance function fields
static caches
```

Deep inheritance does not automatically duplicate prototype methods, but it can increase the number of stateful initialization layers an instance passes through.

Mixin-generated classes can also increase prototype complexity.

# 41. Security Considerations

Do not use:

```text
inheritance
instanceof
prototype membership
```

as the sole authorization mechanism.

A malicious or accidental object may satisfy an interface without being trusted.

Security checks should rely on:

```text
trusted context
explicit capabilities
validated identity
authorization rules.
```

Prototype pollution can also affect inheritance-based assumptions.

# 42. Production Usage

Use inheritance when all of these are reasonably true:

```text
the subtype relationship is semantically real
the parent contract is stable
subtypes preserve parent expectations
shared implementation is genuinely valuable
the hierarchy remains shallow enough to reason about.
```

Prefer composition when:

```text
behavior varies independently
dependencies should be swappable
state ownership should remain explicit
the relationship is has-a/uses-a
multiple capabilities are needed.
```

# 43. Implementation From Scratch

## Exercise 1 — Animal Hierarchy

Implement:

```text
Animal
Dog
Cat
```

Requirements:

```text
shared behavior
safe overriding
subtype-compatible contracts
```

## Exercise 2 — Payment Hierarchy

Create:

```text
PaymentMethod
CardPayment
CashPayment
```

Define:

```text
authorize
capture
refund
```

Then specify preconditions, postconditions and errors for each.

## Exercise 3 — LSP Failure

Create a deliberately invalid subtype.

Example:

```text
Rectangle
Square
```

Then identify which parent assumptions fail.

Do not stop at “LSP says square is problematic.”

Write the actual client contract that breaks.

## Exercise 4 — Composition Rewrite

Take the hierarchy:

```text
Report
PdfReport
ExcelReport
EmailReport
```

and redesign it using:

```text
Report
Renderer
DeliveryChannel
```

Explain why composition changes coupling.

## Exercise 5 — TypeScript Contract

Define:

```ts
interface PaymentMethod {}
abstract class BasePayment {}
```

Implement one concrete subtype for each.

Compare:

```text
compile-time contract
runtime behavior
shared implementation.
```

# 44. Debugging Exercises

## Debug 1 — Broken Preconditions

```js
class Account {
  withdraw(amount) {
    if (amount <= 0) {
      throw new RangeError("positive amount required");
    }
  }
}

class ATMAccount extends Account {
  withdraw(amount) {
    if (amount % 100 !== 0) {
      throw new RangeError("multiples of 100 only");
    }
  }
}
```

Explain whether every `Account` client can substitute `ATMAccount`.

## Debug 2 — Broken Postcondition

```js
class Repository {
  async save(item) {
    // contract: successful return means persisted
  }
}

class MemoryRepository extends Repository {
  async save(item) {
    // stores only temporarily
  }
}
```

Write the missing behavioral distinction.

## Debug 3 — Fragile Base Class

```js
class Processor {
  process() {
    this.prepare();
    this.run();
    this.finish();
  }

  prepare() {}
  run() {}
  finish() {}
}

class ChildProcessor extends Processor {
  prepare() {
    this.run();
  }
}
```

Find the hidden dependency.

## Debug 4 — Constructor Override

```js
class Base {
  constructor() {
    this.initialize();
  }

  initialize() {}
}

class Child extends Base {
  value = 10;

  initialize() {
    console.log(this.value);
  }
}

new Child();
```

Explain the partial-state problem.

# 45. Code Review Exercise

Review:

```js
class Employee {
  calculatePay() {
    return 0;
  }

  save() {}
  sendEmail() {}
  generateReport() {}
}

class Manager extends Employee {
  calculatePay() {
    return "salary";
  }
}
```

Evaluate:

```text
1. Is Manager substitutable?
2. Is calculatePay returning a compatible value?
3. What responsibilities are incorrectly combined in Employee?
4. Should persistence and email be inherited?
5. Would composition improve the design?
6. What should the stable contract be?
```

# 46. Interview Questions

```text
1. What is inheritance?
2. What is subtyping?
3. Are inheritance and subtyping the same?
4. How does extends affect the prototype chain?
5. What does super do?
6. What is method overriding?
7. What is shadowing?
8. What is the Liskov Substitution Principle?
9. What is a precondition violation?
10. What is a postcondition violation?
11. What is an invariant violation?
12. What is a fragile base class?
13. Why is inheriting for code reuse risky?
14. When is composition better?
15. What is the difference between interface inheritance and class inheritance?
16. What is an abstract class?
17. What are mixins?
18. What is structural compatibility?
19. Why does matching TypeScript signatures not prove behavioral substitutability?
20. Why can constructor-time virtual dispatch break subtype safety?
21. How would you decide whether to use inheritance in LLD?
```

# 47. Predict-the-Output Exercises

## Exercise A

```js
class Animal {
  speak() {
    return "sound";
  }
}

class Dog extends Animal {
  speak() {
    return "bark";
  }
}

const dog = new Dog();

console.log(dog.speak());
console.log(dog instanceof Animal);
console.log(dog instanceof Dog);
```

## Exercise B

```js
class Parent {
  value = "parent";
}

class Child extends Parent {
  value = "child";
}

console.log(new Child().value);
```

## Exercise C

```js
class Animal {
  speak() {
    return "sound";
  }
}

class Dog extends Animal {
  speak() {
    return super.speak() + " bark";
  }
}

console.log(new Dog().speak());
```

## Exercise D

```js
class Base {
  constructor() {
    this.init();
  }

  init() {
    console.log("base");
  }
}

class Child extends Base {
  init() {
    console.log("child");
  }
}

new Child();
```

## Exercise E

```js
class Base {
  static role = "base";
}

class Child extends Base {}

console.log(Child.role);
console.log(Object.hasOwn(Child, "role"));
```

## Exercise F

```js
class Base {
  #secret = 10;

  getSecret() {
    return this.#secret;
  }
}

class Child extends Base {}

console.log(new Child().getSecret());
```

# 48. Mastery Exercises

## Level 1 — Understand

Explain:

```text
inheritance
subtyping
overriding
substitutability
composition.
```

## Level 2 — Explain

Explain why:

```text
extends
```

does not automatically imply:

```text
valid subtype.
```

## Level 3 — Predict

Analyze unfamiliar hierarchies for:

```text
method lookup
super
field shadowing
constructor behavior
static inheritance.
```

## Level 4 — Implement

Build:

```text
Animal/Dog/Cat
Payment hierarchy
TypeScript interfaces
abstract base class
```

## Level 5 — Debug

Find:

```text
precondition strengthening
postcondition weakening
invariant violation
fragile base dependency
constructor dispatch hazard.
```

## Level 6 — Apply

Design:

```text
Notification
EmailNotification
SmsNotification
PushNotification
```

using inheritance only if the behavioral contract truly supports substitution.

## Level 7 — Compare

Compare:

```text
inheritance
composition
delegation
mixin
adapter
```

for a plugin-like system.

## Level 8 — Defend

Answer:

> What evidence would convince you that an inheritance hierarchy is justified in an LLD problem?

Connect:

```text
semantic relationship
contract
substitutability
invariants
coupling
change frequency
composition alternative.
```

# 49. Key Takeaways

```text
1. Inheritance and subtyping are related but not identical.
2. extends creates inheritance relationships; it does not prove good design.
3. JavaScript class inheritance remains prototype-based.
4. Overriding changes behavior for parent-typed clients.
5. super provides superclass-oriented behavior/constructor access.
6. Subtypes must preserve parent behavioral contracts.
7. Stronger subtype preconditions can break substitutability.
8. Weaker subtype postconditions can break substitutability.
9. Violated invariants can make subtypes invalid.
10. Behavioral compatibility is stronger than TypeScript structural compatibility.
11. Error and timing behavior can be part of a practical contract.
12. Base classes can be fragile when subclasses depend on internal sequencing.
13. Constructor-time virtual dispatch is a common inheritance hazard.
14. Protected/internal representation sharing increases coupling.
15. Deep hierarchies increase state and lifecycle complexity.
16. Mixins solve some reuse needs but can still create inheritance-like coupling.
17. Interfaces and abstract classes serve different roles.
18. Composition is often the better choice for has-a/uses-a relationships.
19. Inheritance should express semantic specialization, not convenience.
20. A good LLD hierarchy remains shallow, contract-driven, and substitutable.
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
prototype chains
constructors
private state
contracts
```

## Builds Toward

```text
Chapter 10 — Polymorphism
Chapter 11 — Composition
Chapter 12 — Cohesion & Coupling
GRASP
SOLID / Liskov Substitution Principle
TypeScript abstract classes/interfaces
design patterns
domain modeling
dependency injection
architecture-level substitution
```

## Related Concepts

```text
delegation
polymorphism
composition
mixins
abstract classes
interfaces
adapters
behavioral contracts
Liskov Substitution Principle
```

## Why This Chapter Matters

Inheritance is frequently taught as:

```text
reuse parent code
```

but LLD needs a stronger question:

```text
Can clients trust the subtype to preserve the parent contract?
```

That question is what separates:

```text
working inheritance
```

from:

```text
maintainable inheritance.
```

# 51. Completion Criteria

Mark:

```text
[+] Completed
```

when you can:

```text
explain inheritance mechanics
explain subtyping
apply LSP
identify broken contracts
compare inheritance and composition
inspect inheritance hierarchy risks.
```

Mark:

```text
[*] Mastered
```

when you can take an unfamiliar hierarchy and determine:

```text
whether the "is-a" relationship is real
which contract clients rely on
whether every subtype is substitutable
where inheritance coupling exists
whether composition is safer.
```

Reading alone does not mark mastery.

# 52. Revision / Retrieval Record

```md
# Chapter 9 — Revision / Retrieval Record

## Attempt
- Date:
- Duration:
- Status before:
- Status after:

## Inheritance
- What does extends establish?
- What is inherited?
- What is overridden?

## Subtyping
- What does substitutability mean?
- Which client assumptions matter?

## LSP
- Preconditions:
- Postconditions:
- Invariants:
- Errors:
- Timing:

## Hierarchy Review
- Parent:
- Child:
- Semantic relationship:
- Coupling risks:

## Composition
- What could be composed instead?
- Why?

## Debugging
- Issue:
- Root cause:
- Fix:

## Weak Areas
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

Primary source:

```text
ECMAScript Language Specification
https://tc39.es/ecma262/
```

Useful references:

```text
MDN — Inheritance and the prototype chain
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain

MDN — extends
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/extends

TypeScript Handbook — Classes
https://www.typescriptlang.org/docs/handbook/2/classes.html

TypeScript Handbook — Type Compatibility
https://www.typescriptlang.org/docs/handbook/type-compatibility.html
```

Source discipline:

```text
prototype/class semantics
→ ECMAScript

TypeScript type behavior
→ TypeScript documentation

behavioral design
→ contracts and domain requirements

performance
→ measurement + engine-specific evidence
```

# 54. Completion Snapshot

```text
Chapter: 009
Title: Inheritance & Subtyping

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

Separate three questions:

```text
QUESTION 1
Can JavaScript implement the hierarchy?
        ↓
prototype/class mechanics

QUESTION 2
Does the subtype preserve the parent contract?
        ↓
subtyping / LSP

QUESTION 3
Should the hierarchy exist at all?
        ↓
design judgment
```

For every inheritance relationship, ask:

```text
1. What does "is-a" mean here?
2. What contract does the parent promise?
3. Can every subtype preserve that contract?
4. Are preconditions strengthened?
5. Are postconditions weakened?
6. Are invariants preserved?
7. Are errors/timing still compatible?
8. Does the child depend on parent internals?
9. How deep is the hierarchy?
10. Would composition create a cleaner boundary?
```

# Principal Design Principle

> **Use inheritance when specialization preserves a stable behavioral contract; use composition when you mainly need to reuse or combine behavior.**

# Track Mapping

```text
Track A — Core Theory
    inheritance
    subtyping
    overriding
    super
    contracts
    Liskov Substitution Principle
    fragile base class
    hierarchy design

Track B — Implementation
    class hierarchies
    prototype inspection
    abstract classes
    interfaces
    composition rewrites
    deliberate LSP failures

Track C — Interview / Reasoning
    subtype validity
    precondition/postcondition analysis
    hierarchy smells
    inheritance vs composition
    constructor dispatch hazards
    design judgment
```
