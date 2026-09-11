# Chapter 10 — Polymorphism & Dynamic Dispatch

> **JavaScript OOP + LLD Mastery**
>
> Polymorphism is the ability to work with different concrete objects through a common contract while preserving the behavior required by the client.
>
> In JavaScript, polymorphism is deeply connected to **property lookup, prototypes, method calls, `this`, duck typing, composition, and late binding**. This chapter turns the inheritance and subtyping ideas from Chapter 9 into runtime dispatch reasoning.

**Status:** `[ ] Not Started`

# 1. Learning Objectives

```text
[ ] define polymorphism
[ ] distinguish subtype polymorphism from duck typing
[ ] distinguish polymorphism from inheritance
[ ] explain dynamic dispatch
[ ] explain late binding
[ ] trace method lookup at runtime
[ ] explain receiver-based dispatch
[ ] explain the role of this in method calls
[ ] explain overriding in dynamic dispatch
[ ] distinguish super dispatch from ordinary dynamic dispatch
[ ] explain polymorphism through composition
[ ] explain polymorphism through functions
[ ] explain structural polymorphism in TypeScript
[ ] distinguish structural compatibility from behavioral substitutability
[ ] identify accidental polymorphism
[ ] design stable polymorphic contracts
[ ] reason about dispatch tables conceptually
[ ] compare inheritance polymorphism with strategy composition
[ ] identify polymorphism failure modes
[ ] debug wrong-method dispatch
[ ] debug detached methods
[ ] implement polymorphic systems
[ ] compare alternative polymorphism mechanisms
[ ] reason about polymorphism in LLD
```

# 2. Prerequisites

Required:

```text
Chapter 1 — JavaScript Object Model — Foundation
Chapter 2 — Object Identity, Equality & Mutability
Chapter 3 — Prototype Chain Deep Dive
Chapter 4 — Constructor Functions & Instance Construction
Chapter 5 — JavaScript Classes Internally
Chapter 6 — Class Fields & Initialization Semantics
Chapter 7 — Private State & Encapsulation
Chapter 8 — Abstraction & Stable Object Interfaces
Chapter 9 — Inheritance & Subtyping
objects
classes
prototypes
contracts
subtyping
inheritance
composition
```

# 3. What Is It?

Polymorphism means:

```text
one client-facing contract
+
multiple valid implementations.
```

Example:

```js
function checkout(paymentMethod, amount) {
  return paymentMethod.charge(amount);
}
```

Different objects can satisfy the expected capability:

```text
CardPayment
CashPayment
WalletPayment
MockPayment
```

The caller uses:

```text
paymentMethod.charge(...)
```

without needing to know the concrete implementation.

A useful model:

```text
                  CLIENT
                    │
                    ▼
              common contract
                    │
          ┌─────────┼─────────┐
          │         │         │
          ▼         ▼         ▼
        Card      Cash      Wallet
          │         │         │
          └─────────┼─────────┘
                    ▼
             runtime behavior
```

# 4. Why Does It Exist?

Without polymorphism, client code often becomes:

```js
if (type === "card") {
  // card logic
} else if (type === "cash") {
  // cash logic
} else if (type === "wallet") {
  // wallet logic
}
```

As variants grow:

```text
conditional complexity grows
client knowledge grows
change propagation grows
```

With polymorphism:

```js
paymentMethod.charge(amount);
```

The variant owns its behavior.

This can improve:

```text
extensibility
locality of change
cohesion
testability.
```

But polymorphism is not automatically better than conditionals.

The domain must actually contain meaningful variation.

# 5. Mental Model

Dynamic dispatch can be visualized as:

```text
                 obj.method()
                      │
                      ▼
                receiver = obj
                      │
                      ▼
             resolve property "method"
                      │
              prototype lookup
                      │
                      ▼
                selected function
                      │
                      ▼
                 call with
                this = obj
```

When multiple concrete objects provide the same capability:

```text
same call site
      │
      ├── object A → implementation A
      ├── object B → implementation B
      └── object C → implementation C
```

That runtime selection is dynamic dispatch.

# 6. Core Rules

## Rule 1 — Polymorphism Is About the Client's Point of View

The key question is:

```text
Can the client interact with different implementations through the same contract?
```

---

## Rule 2 — Inheritance Is One Way to Achieve Polymorphism

You can achieve polymorphism through:

```text
inheritance
composition
duck typing
function values
closures
higher-order functions
interfaces/types
```

---

## Rule 3 — Dynamic Dispatch Depends on the Runtime Receiver

```js
obj.run()
```

looks up `run` through the receiver's property/prototype behavior.

---

## Rule 4 — Method Location and Receiver Are Different

A method may live on:

```text
Parent.prototype
```

while:

```text
this
```

is the child instance.

---

## Rule 5 — `super` Is Not Ordinary Dynamic Dispatch

```js
super.run()
```

uses superclass-oriented semantics.

```js
this.run()
```

uses ordinary receiver-based property access and may dispatch to an override.

---

## Rule 6 — Matching Shape Is Not Enough

TypeScript can verify:

```text
method exists
argument types match
return type matches
```

but cannot by itself verify:

```text
behavior
invariants
side effects
semantics.
```

# 7. Syntax

## Inheritance Polymorphism

```js
class PaymentMethod {
  charge(amount) {}
}

class CardPayment extends PaymentMethod {
  charge(amount) {
    return `card:${amount}`;
  }
}
```

---

## Duck-Typed Polymorphism

```js
function process(paymentMethod) {
  return paymentMethod.charge(100);
}
```

Any suitable object can be passed.

---

## Composition Polymorphism

```js
class Checkout {
  constructor(paymentGateway) {
    this.paymentGateway = paymentGateway;
  }

  pay(amount) {
    return this.paymentGateway.charge(amount);
  }
}
```

---

## TypeScript Interface

```ts
interface PaymentMethod {
  charge(amount: number): Promise<void>;
}
```

# 8. Basic Examples

## Example 1 — Runtime Dispatch

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

class Cat extends Animal {
  speak() {
    return "meow";
  }
}

function makeSound(animal) {
  return animal.speak();
}

console.log(makeSound(new Dog()));
console.log(makeSound(new Cat()));
```

**Prediction**

```text
bark
meow
```

**Trace**

```text
makeSound(dog)
→ dog.speak
→ Dog.prototype.speak

makeSound(cat)
→ cat.speak
→ Cat.prototype.speak
```

---

## Example 2 — Duck Typing

```js
const printer = {
  speak() {
    return "print";
  },
};

const robot = {
  speak() {
    return "beep";
  },
};

function execute(actor) {
  return actor.speak();
}

console.log(execute(printer));
console.log(execute(robot));
```

No shared class is required.

---

## Example 3 — Strategy Composition

```js
class Checkout {
  constructor(pricingStrategy) {
    this.pricingStrategy = pricingStrategy;
  }

  total(order) {
    return this.pricingStrategy.calculate(order);
  }
}
```

Different strategies can implement the same capability.

# 9. Execution Walkthrough

Consider:

```js
class Notification {
  send(message) {
    return `generic:${message}`;
  }
}

class EmailNotification extends Notification {
  send(message) {
    return `email:${message}`;
  }
}

class SmsNotification extends Notification {
  send(message) {
    return `sms:${message}`;
  }
}

function notify(notification, message) {
  return notification.send(message);
}
```

Call:

```js
notify(new EmailNotification(), "Hello");
```

Runtime reasoning:

```text
1. Construct EmailNotification.
2. notify receives the object.
3. Evaluate notification.send.
4. Start lookup at EmailNotification instance.
5. No own send property.
6. Check EmailNotification.prototype.
7. Find send.
8. Call it with this = notification.
9. Return email implementation result.
```

Change:

```js
notify(new SmsNotification(), "Hello");
```

and the client stays the same while dispatch changes.

# 10. Internal Mechanics

A method call such as:

```js
obj.run()
```

combines:

```text
property resolution
+
call evaluation
+
receiver binding.
```

A simplified mental model:

```text
Get(obj, "run")
       ↓
function
       ↓
Call(function, obj)
```

The specification details are more precise, but this separation is useful for debugging.

Polymorphism emerges when different receivers resolve the same capability to different implementations.

# 11. ECMAScript / Specification Semantics

ECMAScript defines object property access and function calls through:

```text
[[Get]]
property lookup
Call
this binding
prototype delegation
super property references
```

There is no single ECMAScript operation named:

```text
"polymorphism"
```

because polymorphism is a design/runtime behavior emerging from these mechanisms.

TypeScript adds compile-time type constructs but they do not change JavaScript's fundamental runtime dispatch model.

# 12. Advanced Behavior

## 12.1 Dynamic Dispatch Through Prototypes

```js
const parent = {
  run() {
    return "parent";
  },
};

const child = Object.create(parent);

child.run = function () {
  return "child";
};

console.log(child.run());
```

The receiver's own property wins.

---

## 12.2 Dynamic Dispatch Through Inheritance

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

function execute(value) {
  return value.run();
}
```

The same client can execute both implementations.

---

## 12.3 Dynamic Dispatch and `this`

```js
class Parent {
  value = "parent";

  getValue() {
    return this.value;
  }
}

class Child extends Parent {
  value = "child";
}

console.log(new Child().getValue());
```

The inherited method reads:

```text
this.value
```

from the child receiver.

# 13. Early Binding vs Late Binding

A simplified contrast:

```text
early/static selection
→ implementation known from the declared/reference type

dynamic/late selection
→ implementation selected from the runtime receiver/behavior.
```

JavaScript commonly performs dynamic property/method lookup at runtime.

TypeScript may narrow or statically analyze types before execution, but the emitted runtime behavior remains JavaScript.

# 14. Polymorphism Through Composition

You do not need inheritance.

```js
class OrderCalculator {
  constructor(discountPolicy) {
    this.discountPolicy = discountPolicy;
  }

  calculate(order) {
    return this.discountPolicy.apply(order);
  }
}
```

Implementations:

```js
const NoDiscount = {
  apply(order) {
    return order.total;
  },
};

const TenPercentDiscount = {
  apply(order) {
    return order.total * 0.9;
  },
};
```

Now:

```text
OrderCalculator
      ↓
DiscountPolicy capability
      ↓
different implementation
```

# 15. Function Polymorphism

Functions themselves can be capabilities.

```js
function calculateTotal(order, discountCalculator) {
  return discountCalculator(order);
}
```

Now variation is represented by:

```text
function value
```

instead of:

```text
class hierarchy.
```

This is particularly natural in JavaScript.

# 16. Closure-Based Polymorphism

```js
function createPricingPolicy(rate) {
  return {
    calculate(order) {
      return order.total * rate;
    },
  };
}
```

Each generated object implements the required behavior.

This can provide polymorphism without inheritance.

# 17. Interface Polymorphism in TypeScript

```ts
interface Logger {
  info(message: string): void;
}

class ConsoleLogger implements Logger {
  info(message: string) {
    console.log(message);
  }
}

class TestLogger implements Logger {
  messages: string[] = [];

  info(message: string) {
    this.messages.push(message);
  }
}
```

A client can depend on:

```ts
Logger
```

while receiving different concrete implementations.

Again:

```text
TypeScript interface
→ compile-time contract

runtime object
→ actual dispatch.
```

# 18. Structural Polymorphism

Because TypeScript is structurally typed:

```ts
const logger = {
  info(message: string) {
    console.log(message);
  },
};
```

can satisfy:

```ts
Logger
```

without:

```ts
implements Logger
```

This provides flexible polymorphism.

But it does not prove behavioral equivalence.

# 19. Behavioral Polymorphism

Suppose:

```ts
interface Cache {
  get(key: string): string | null;
}
```

Two implementations might be:

```text
MemoryCache
RemoteCache
```

Both satisfy the type.

But if `get()` in one:

```text
throws on missing key
```

while the contract expects:

```text
null
```

the implementation is not behaviorally substitutable.

Polymorphism therefore requires:

```text
contract compatibility
+
behavioral compatibility.
```

# 20. `super` vs `this`

Consider:

```js
class Base {
  run() {
    return "base";
  }
}

class Child extends Base {
  run() {
    return this.extra() + ":" + super.run();
  }

  extra() {
    return "child";
  }
}
```

Here:

```text
this.extra()
→ ordinary dynamic dispatch on receiver

super.run()
→ superclass-oriented lookup
```

These are different dispatch mechanisms.

# 21. Method Extraction Breaks Receiver-Based Polymorphism

```js
class Dog {
  speak() {
    return this.name;
  }
}

const dog = new Dog();

dog.name = "Bruno";

const speak = dog.speak;

console.log(speak());
```

The function was retrieved from the object but is no longer being called with `dog` as its receiver.

Use explicit binding when required:

```js
const bound = dog.speak.bind(dog);
```

# 22. Polymorphism and Optional Capabilities

A design may have several capabilities:

```text
Payable
Refundable
Cancelable
Auditable
```

Do not force all implementations into one huge interface if clients need only subsets.

Prefer focused capabilities.

# 23. Accidental Polymorphism

JavaScript can accept any object with the expected property:

```js
process({
  run() {
    return "ok";
  },
});
```

This flexibility is useful, but accidental contracts can emerge when code relies on undocumented behavior.

Document the meaningful protocol:

```text
run()
→ returns Result
→ may throw DomainError
→ must be idempotent
```

not merely:

```text
has run property.
```

# 24. Advanced Behavior — Optional Dispatch

```js
handler?.run?.();
```

supports optional capabilities.

This can be useful for hooks and plugins, but excessive optional dispatch can make contracts vague.

# 25. Advanced Behavior — Dispatch Tables

Sometimes a dispatch table is clearer:

```js
const handlers = {
  card: handleCard,
  cash: handleCash,
  wallet: handleWallet,
};

function process(type, data) {
  return handlers[type](data);
}
```

This is also runtime dispatch.

Choice depends on:

```text
variation
state ownership
extensibility
data locality
domain semantics.
```

# 26. Advanced Behavior — Double Dispatch

Some problems vary along two dimensions:

```text
operation type
+
object type
```

A single subtype hierarchy may not scale cleanly. Double dispatch, visitors, or explicit maps can be appropriate depending on the problem.

# 27. Edge Cases

```text
Detached method → receiver is lost.
Prototype mutation → dispatch can change at runtime.
Monkey patching → shared prototype behavior can change for many objects.
Optional method → missing capability can fail at runtime.
Structural match → TypeScript shape does not prove semantics.
super → superclass lookup differs from this-based lookup.
Field function → per-instance function can change ordinary method overriding assumptions.
```

# 28. Common Misconceptions

```text
"polymorphism means inheritance."
"polymorphism requires classes."
"polymorphism means method overloading."
"matching an interface guarantees behavior."
"super and this use the same dispatch."
"dynamic dispatch always means inheritance."
"duck typing is automatically safe."
"more subclasses means more polymorphism."
"same method name means same contract."
"polymorphism is always better than conditionals."
```

# 29. Common Mistakes

```text
[ ] using a huge common interface
[ ] treating shape compatibility as behavioral compatibility
[ ] detaching methods without binding
[ ] relying on mutable shared prototypes
[ ] using inheritance for every variation
[ ] hiding important contract semantics
[ ] creating optional-method interfaces with unclear guarantees
[ ] replacing simple dispatch tables with unnecessary class hierarchies
[ ] dispatching on type strings throughout business logic
[ ] ignoring error/timing semantics in polymorphic contracts.
```

# 30. Comparison With Related Concepts

| Concept | How variation is selected |
|---|---|
| Dynamic dispatch | runtime receiver/lookup |
| Inheritance | prototype/class hierarchy |
| Duck typing | required behavior exists at runtime |
| Structural typing | compatible shape at compile time |
| Composition | delegated capability |
| Function strategy | function value |
| Dispatch table | key selects implementation |
| Overloading | multiple signatures/selection rules |
| Adapter | translates one contract to another |
| Visitor | dispatch across varying operation/object dimensions |

# 31. Performance Considerations

Modern engines optimize common method dispatch patterns.

Potential factors include:

```text
polymorphic call sites
object shapes
prototype depth
function identity
dynamic mutation
dispatch tables
strategy-object allocation.
```

Do not assume:

```text
polymorphism = slow
conditionals = fast
```

without measurement.

# 32. Memory Considerations

Potential costs include:

```text
strategy objects
per-instance function fields
closure environments
large prototype graphs
dispatch-table entries
cached implementations.
```

Composition can create more objects while improving ownership and testability.

# 33. Security Considerations

Polymorphic dispatch can become dangerous if attackers control:

```text
implementation selection
method names
dispatch keys
plugin registration
prototype mutation.
```

Prefer:

```text
allowlists
validated capabilities
trusted registries
explicit authorization.
```

# 34. Production Usage

A production polymorphic boundary should define:

```text
required methods/capabilities
input contract
output contract
errors
side effects
timing
idempotency
state guarantees.
```

If code repeatedly branches on concrete implementation identity, investigate whether the abstraction is missing.

# 35. Implementation From Scratch

## Exercise 1 — Payment Polymorphism

Implement:

```text
PaymentMethod
CardPayment
CashPayment
WalletPayment
```

with:

```text
charge
refund
```

Define the contract.

## Exercise 2 — Strategy Composition

Implement:

```text
PricingEngine
NoDiscount
FlatDiscount
PercentageDiscount
```

using composition rather than inheritance.

## Exercise 3 — Duck-Typed Logger

Write:

```js
function runTask(logger) {}
```

so any object with the required logging capability works.

Document the behavioral contract.

## Exercise 4 — TypeScript Structural Contract

Define:

```ts
interface Notifier {
  send(message: string): Promise<void>;
}
```

Implement Email, SMS and Test notifiers.

## Exercise 5 — Dispatch Table

Implement:

```text
payment type → handler
```

and compare it with polymorphic objects.

# 36. Debugging Exercises

## Debug 1 — Detached Dispatch

```js
class User {
  name = "Milan";

  greet() {
    return this.name;
  }
}

const user = new User();
const fn = user.greet;

console.log(fn());
```

Explain why receiver-based dispatch cannot preserve `this` automatically.

## Debug 2 — Broken Behavioral Contract

```ts
interface Cache {
  get(key: string): string | null;
}

class BadCache implements Cache {
  get(key: string): string {
    throw new Error("missing");
  }
}
```

Identify the behavioral contract failure.

## Debug 3 — Concrete Branching

```js
function notify(value) {
  if (value instanceof EmailNotifier) {
    return value.send();
  }

  if (value instanceof SmsNotifier) {
    return value.send();
  }

  throw new Error("unsupported");
}
```

Redesign around a polymorphic capability.

## Debug 4 — Optional Contract

```js
function pluginHook(plugin) {
  plugin.run?.();
}
```

Determine whether `run` should be required or optional and what that means for the contract.

# 37. Code Review Exercise

Review:

```js
class ReportService {
  generate(report) {
    if (report.type === "pdf") {
      return this.generatePdf(report);
    }

    if (report.type === "excel") {
      return this.generateExcel(report);
    }

    if (report.type === "csv") {
      return this.generateCsv(report);
    }

    throw new Error("Unknown report");
  }
}
```

Evaluate:

```text
1. Is the variation stable?
2. Should this be polymorphic behavior?
3. Would a dispatch table be clearer?
4. Would composition be clearer?
5. Who owns report-specific rules?
6. What changes when a new report type is added?
```

# 38. Interview Questions

```text
1. What is polymorphism?
2. What is dynamic dispatch?
3. What is late binding?
4. Does polymorphism require inheritance?
5. What is duck typing?
6. What is structural typing?
7. How does JavaScript achieve polymorphism?
8. How does prototype lookup participate in dispatch?
9. What role does this play in method dispatch?
10. What is the difference between this.method() and super.method()?
11. Why does method extraction break ordinary receiver-based behavior?
12. What is behavioral substitutability?
13. Why does TypeScript structural compatibility not guarantee behavioral polymorphism?
14. When is composition better than inheritance?
15. When is a dispatch table better than polymorphic objects?
16. What is double dispatch?
17. What is accidental polymorphism?
18. How would you design a stable polymorphic API?
19. How can polymorphism reduce conditional complexity?
20. When can polymorphism itself become over-engineering?
```

# 39. Predict-the-Output Exercises

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

function makeSound(animal) {
  return animal.speak();
}

console.log(makeSound(new Animal()));
console.log(makeSound(new Dog()));
```

## Exercise B

```js
class Base {
  run() {
    return this.value;
  }
}

class Child extends Base {
  value = "child";
}

console.log(new Child().run());
```

## Exercise C

```js
class Base {
  run() {
    return "base";
  }
}

class Child extends Base {
  run() {
    return this.extra() + ":" + super.run();
  }

  extra() {
    return "child";
  }
}

console.log(new Child().run());
```

## Exercise D

```js
class User {
  greet() {
    return this.name;
  }
}

const user = new User();
user.name = "Milan";

const greet = user.greet;

console.log(greet.call(user));
```

## Exercise E

```js
const a = {
  run() {
    return "A";
  },
};

const b = {
  run() {
    return "B";
  },
};

function execute(value) {
  return value.run();
}

console.log(execute(a));
console.log(execute(b));
```

## Exercise F

```js
class Base {
  static role() {
    return "base";
  }
}

class Child extends Base {
  static role() {
    return "child";
  }
}

console.log(Child.role());
```

# 40. Mastery Exercises

## Level 1 — Understand

Explain:

```text
polymorphism
dynamic dispatch
late binding
duck typing
structural typing
```

## Level 2 — Explain

Trace:

```text
obj.method()

receiver
→ lookup
→ selected function
→ this
→ result
```

## Level 3 — Predict

Analyze:

```text
inheritance dispatch
prototype dispatch
composition dispatch
function strategies
super
detached methods.
```

## Level 4 — Implement

Build:

```text
payment polymorphism
pricing strategies
logging capability
dispatch table
TypeScript contract.
```

## Level 5 — Debug

Fix:

```text
detached method
broken behavioral contract
concrete branching
unclear optional capability.
```

## Level 6 — Apply

Design:

```text
NotificationService
EmailNotifier
SmsNotifier
PushNotifier
```

using a stable capability rather than concrete-type branching.

## Level 7 — Compare

Compare:

```text
inheritance polymorphism
composition polymorphism
duck typing
function strategies
dispatch tables.
```

## Level 8 — Defend

Answer:

> When should a design use polymorphism instead of a conditional or dispatch table?

Connect:

```text
variation
change frequency
state ownership
contract
extensibility
complexity
observability.
```

# 41. Key Takeaways

```text
1. Polymorphism lets one client contract work with multiple implementations.
2. Inheritance is only one way to achieve polymorphism.
3. Dynamic dispatch is based on runtime object behavior.
4. JavaScript's prototype lookup participates directly in method dispatch.
5. this represents the receiver in ordinary method calls.
6. super uses superclass-oriented semantics and is not identical to this-based dispatch.
7. Duck typing can provide polymorphism without classes.
8. Composition can provide flexible polymorphism without inheritance.
9. Function values are natural strategy implementations in JavaScript.
10. TypeScript structural typing enables compile-time polymorphism by shape.
11. Structural compatibility does not guarantee behavioral substitutability.
12. Method extraction can lose receiver-based dispatch.
13. Polymorphic contracts should define semantic behavior, not only method names.
14. Optional methods can weaken an interface if their necessity is unclear.
15. Dispatch tables are a legitimate alternative to polymorphic objects.
16. Double dispatch may be useful when behavior varies along two dimensions.
17. Polymorphism should reduce concrete-type knowledge in clients.
18. Excessive abstraction can make simple variation harder to understand.
19. Performance should be measured rather than assumed.
20. Good polymorphism is contract-driven, behaviorally compatible, and aligned with real variation.
```

# 42. Concept Connections

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
Chapter 9 — Inheritance & Subtyping
method lookup
this
super
contracts
composition
```

## Builds Toward

```text
Chapter 11 — Composition
Chapter 12 — Cohesion & Coupling
Chapter 13 — Object Responsibilities
GRASP
SOLID
Strategy Pattern
Template Method
Adapter
Factory
Visitor
State Pattern
dependency injection
TypeScript generic/type-level polymorphism
DDD policy objects
```

## Related Concepts

```text
dynamic dispatch
late binding
duck typing
structural typing
delegation
strategy
dispatch tables
double dispatch
interfaces
substitutability
```

## Why This Chapter Matters

Polymorphism is where object design becomes flexible.

Instead of:

```text
client knows every variant
```

we want:

```text
client knows the stable contract
```

and the runtime selects the appropriate behavior.

# 43. Completion Criteria

Mark:

```text
[+] Completed
```

when you can:

```text
define polymorphism
trace dynamic dispatch
explain this vs super
compare inheritance and composition
design behavioral contracts
identify concrete-type branching
```

Mark:

```text
[*] Mastered
```

when you can choose:

```text
polymorphism
vs
conditional
vs
dispatch table
vs
strategy function
vs
composition
```

based on domain variation and change cost.

Reading alone does not mark mastery.

# 44. Revision / Retrieval Record

```md
# Chapter 10 — Revision / Retrieval Record

## Attempt
- Date:
- Duration:
- Status before:
- Status after:

## Polymorphism
- What is polymorphism?
- What is dynamic dispatch?
- What is late binding?

## Dispatch
- Receiver:
- Lookup:
- Selected implementation:
- this:

## Inheritance
- What does override change?
- How is super different?

## Composition
- What capability is delegated?
- Why does composition help?

## Structural Typing
- Compile-time contract:
- Behavioral limitation:

## Alternatives
- Conditional:
- Dispatch table:
- Function strategy:
- Polymorphic object:

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

# 45. Canonical References and Source Discipline

Primary source:

```text
ECMAScript Language Specification
https://tc39.es/ecma262/
```

Useful references:

```text
MDN — Inheritance and the prototype chain
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain

MDN — this
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this

MDN — super
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/super

TypeScript Handbook — Type Compatibility
https://www.typescriptlang.org/docs/handbook/type-compatibility.html
```

Source discipline:

```text
dispatch semantics
→ ECMAScript

TypeScript structural typing
→ TypeScript documentation

performance
→ measurement + engine-specific evidence

polymorphic contract quality
→ behavioral requirements and domain semantics
```

# 46. Completion Snapshot

```text
Chapter: 010
Title: Polymorphism & Dynamic Dispatch

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

For:

```js
object.operation(input)
```

reason:

```text
object
  │
  ▼
property lookup
  │
  ├── own operation?
  │
  └── prototype chain
          │
          ▼
     selected function
          │
          ▼
      this = object
          │
          ▼
      implementation
          │
          ▼
        result
```

For polymorphic clients:

```text
                 COMMON CONTRACT
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Object A     Object B     Object C
          │            │            │
          ▼            ▼            ▼
       behavior A   behavior B   behavior C
```

Then ask:

```text
1. What capability does the client actually require?
2. Which implementations satisfy that capability?
3. Is the contract behavioral or only structural?
4. How is the implementation selected?
5. Does this require inheritance?
6. Would composition or a function strategy be clearer?
7. What happens when another implementation is added?
8. Does the client still avoid concrete-type branching?
9. Are errors, timing and side effects compatible?
10. Is polymorphism actually reducing complexity?
```

# Principal Design Principle

> **Use polymorphism when a stable client contract must accommodate genuine behavioral variation; keep the variation behind the boundary instead of spreading concrete-type knowledge through the system.**

# Track Mapping

```text
Track A — Core Theory
    polymorphism
    dynamic dispatch
    late binding
    prototype lookup
    this
    super
    behavioral contracts
    structural typing

Track B — Implementation
    inheritance dispatch
    duck-typed capabilities
    strategy composition
    function polymorphism
    dispatch tables
    TypeScript interfaces

Track C — Interview / Reasoning
    dispatch tracing
    this vs super
    polymorphism vs conditionals
    inheritance vs composition
    structural vs behavioral compatibility
    design trade-offs
```
