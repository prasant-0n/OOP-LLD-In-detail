# Chapter 4 — Constructor Functions & Instance Construction

> **JavaScript OOP + LLD Mastery**
>
> This chapter explains how JavaScript creates object instances through constructor functions and the `new` operator. The objective is not to memorize `new`, but to understand constructability, prototype assignment, instance initialization, constructor return-value rules, and the bridge into ES classes.

**Status:** `[ ] Not Started`

# 1. Learning Objectives

By the end of this chapter, you should be able to:

```text
[ ] define a constructor function
[ ] distinguish callable from constructable functions
[ ] explain Call vs Construct
[ ] explain new conceptually
[ ] explain new.target
[ ] explain prototype selection during construction
[ ] explain constructor initialization
[ ] explain constructor return-value rules
[ ] distinguish object and primitive returns
[ ] explain why arrow functions are not constructors
[ ] explain why class constructors require construction
[ ] explain Constructor.prototype
[ ] explain the constructor property
[ ] explain shared prototype methods
[ ] compare prototype methods with per-instance methods
[ ] explain derived constructors and super
[ ] explain default constructors
[ ] explain instanceof at a high level
[ ] implement constructor-based objects
[ ] debug construction problems
[ ] reason about constructor responsibilities and invariants
```

# 2. Prerequisites

```text
Chapter 1 — JavaScript Object Model — Foundation
Chapter 2 — Object Identity, Equality & Mutability
Chapter 3 — Prototype Chain Deep Dive
functions
objects
references
prototype chains
this
```

# 3. What Is It?

A constructor is a constructable function or class used to create and initialize an instance.

```text
construction
    ↓
new instance
    ↓
prototype relationship
    ↓
constructor initialization
    ↓
constructed result
```

Example:

```js
function User(name) {
  this.name = name;
}

const user = new User("Milan");
```

The instance normally has:

```text
user
 └── [[Prototype]] → User.prototype
```

# 4. Why Does It Exist?

Object construction often needs a repeatable initialization protocol.

A constructor can establish:

```text
required inputs
default state
dependencies
identity
invariants
shared prototype behavior
```

Construction is therefore part of object design, not merely syntax.

# 5. Mental Model

For:

```js
function User(name) {
  this.name = name;
}

const user = new User("Milan");
```

think:

```text
new User(...)
      ↓
construct operation
      ↓
new/initial object
      ↓
prototype relationship
      ↓
constructor runs with instance as this
      ↓
instance state initialized
      ↓
result returned
```

This is a conceptual model; ECMAScript defines the exact construction algorithms.

# 6. Core Rules

## Rule 1 — Not Every Function Is Constructable

```js
function User() {}
const Factory = () => {};
```

`User` is constructable; the arrow function is not.

## Rule 2 — Call and Construct Are Different

```js
User();
new User();
```

are different language-level operations.

## Rule 3 — `new` Normally Initializes `this`

```js
function User(name) {
  this.name = name;
}
```

With `new User("Milan")`, `this` refers to the constructed instance.

## Rule 4 — Prototype Behavior and Instance State Are Separate

```js
function User(name) {
  this.name = name;
}

User.prototype.greet = function () {
  return this.name;
};
```

The instance owns `name`; the prototype owns the shared method.

## Rule 5 — Object Returns Can Replace the Constructed Result

```js
function Factory() {
  return { type: "replacement" };
}
```

A returned object can become the construction result.

## Rule 6 — Primitive Returns Do Not Normally Replace It

```js
function User() {
  this.value = 1;
  return 42;
}
```

The constructed object remains the result.

# 7. Syntax

## Constructor Function

```js
function User(name) {
  this.name = name;
}
```

## Construction

```js
const user = new User("Milan");
```

## Prototype Method

```js
User.prototype.greet = function () {
  return `Hello ${this.name}`;
};
```

## `new.target`

```js
function User() {
  console.log(new.target === User);
}
```

# 8. Basic Examples

## Example 1 — Basic Construction

```js
function User(name) {
  this.name = name;
}

const user = new User("Milan");

console.log(user.name);
console.log(Object.getPrototypeOf(user) === User.prototype);
```

**Prediction**

```text
"Milan"
true
```

**Trace**

```text
new
↓
instance
↓
[[Prototype]] = User.prototype
↓
User executes with this = instance
↓
name is assigned
```

## Example 2 — Shared Prototype Method

```js
function User(name) {
  this.name = name;
}

User.prototype.greet = function () {
  return `Hello ${this.name}`;
};

const a = new User("A");
const b = new User("B");

console.log(a.greet());
console.log(a.greet === b.greet);
```

**Prediction**

```text
Hello A
true
```

## Example 3 — Per-Instance Method

```js
function User(name) {
  this.name = name;
  this.greet = function () {
    return `Hello ${this.name}`;
  };
}

const a = new User("A");
const b = new User("B");

console.log(a.greet === b.greet);
```

**Prediction**

```text
false
```

# 9. Execution Walkthrough

```js
function Account(owner, balance) {
  this.owner = owner;
  this.balance = balance;
}

Account.prototype.deposit = function (amount) {
  this.balance += amount;
};

const account = new Account("Milan", 1000);
```

Conceptual sequence:

```text
1. Evaluate Account.
2. Start a Construct operation.
3. Create the new instance.
4. Select Account.prototype as its prototype in the normal case.
5. Invoke Account with the new instance as this.
6. Initialize owner.
7. Initialize balance.
8. Complete construction.
9. Return the construction result.
```

Final model:

```text
account
├── owner
├── balance
└── [[Prototype]]
      ↓
 Account.prototype
      └── deposit
```

# 10. Internal Mechanics

JavaScript distinguishes:

```text
[[Call]]
[[Construct]]
```

A function can be callable without being constructable.

Conceptually:

```text
fn(...)
→ Call

new fn(...)
→ Construct
```

Construction uses constructor-specific semantics instead of simply calling the function and returning its ordinary return value.

# 11. ECMAScript / Specification Semantics

ECMAScript specifies construction using internal methods and abstract operations including:

```text
[[Construct]]
newTarget
prototype selection
instance creation
constructor execution
return-value handling
```

The exact algorithms are defined by the ECMAScript specification.

Engine details such as:

```text
allocation strategy
hidden classes/shapes
inline caches
memory layout
optimization heuristics
```

are implementation details, not universal language guarantees.

# 12. Advanced Behavior

## 12.1 Constructability

```text
callable ≠ necessarily constructable
```

Arrow functions are callable but not constructable.

## 12.2 `new.target`

```js
function User() {
  console.log(new.target?.name);
}

new User();
User();
```

Conceptually:

```text
new User() → new.target is User
User()     → new.target is undefined
```

## 12.3 Constructor Invariants

```js
function Money(amount, currency) {
  if (!Number.isFinite(amount)) {
    throw new TypeError("amount must be finite");
  }

  if (typeof currency !== "string") {
    throw new TypeError("currency required");
  }

  this.amount = amount;
  this.currency = currency.toUpperCase();
}
```

Construction establishes a stronger state contract.

## 12.4 Prototype Mutation vs Replacement

```js
User.prototype.greet = function () {};
```

mutates the existing prototype.

```js
User.prototype = {};
```

replaces the prototype used by future construction.

Existing instances retain their previous prototype relationship.

## 12.5 Constructor Return

```js
function User() {
  this.name = "Milan";
  return { name: "Replacement" };
}
```

The explicit object can become the final result.

# 13. Edge Cases

## Arrow Functions

```js
const User = () => {};
new User(); // TypeError
```

## Class Constructor Without `new`

```js
class User {}
User(); // TypeError
```

## Primitive Return

```js
function User() {
  this.value = 1;
  return 42;
}

const user = new User();
console.log(user.value); // 1
```

## Object Return

```js
function User() {
  this.value = 1;
  return { value: 2 };
}

const user = new User();
console.log(user.value); // 2
```

## Prototype Replacement

```js
function User() {}

const first = new User();

User.prototype = {};

const second = new User();

Object.getPrototypeOf(first) !== Object.getPrototypeOf(second);
```

## Constructor Property

`prototype.constructor` is a property and can be shadowed, deleted, or changed. It is not an absolute provenance guarantee.

# 14. Common Misconceptions

```text
"new simply calls the function."
"every function can be a constructor."
"prototype means the instance itself."
"prototype methods are copied to each instance."
"any constructor return value replaces the instance."
"instanceof proves exactly which constructor created an object."
"classes remove prototypes."
"constructors should perform the whole application workflow."
```

# 15. Common Mistakes

```text
[ ] putting every method on each instance unnecessarily
[ ] hiding required dependencies in globals
[ ] allowing invalid state after construction
[ ] confusing factories with constructors
[ ] replacing prototypes unintentionally
[ ] relying blindly on constructor
[ ] overusing deep inheritance
[ ] doing heavy I/O during simple construction
[ ] mixing initialization with long business workflows
```

# 16. Comparison With Related Concepts

| Concept | Primary purpose |
|---|---|
| Constructor function | Construct and initialize instances |
| Class constructor | Class-specific construction |
| Factory function | Produce objects through a creation API |
| Object literal | Direct object creation |
| Object.create | Direct prototype-oriented creation |
| Prototype method | Shared instance behavior |
| Per-instance method | Instance-specific function/closure |
| Static method | Constructor/class-side behavior |
| Dependency injection | Explicit collaborator supply |
| Builder | Stepwise construction of complex objects |

# 17. Performance Considerations

Prototype-shared methods often avoid one function allocation per instance:

```js
User.prototype.greet = function () {};
```

compared with:

```js
this.greet = function () {};
```

The second can create a new function for every construction.

But actual performance depends on:

```text
engine
allocation behavior
object shape
closure captures
call patterns
workload.
```

Measure before optimizing.

# 18. Memory Considerations

For many instances:

```text
prototype method
→ shared function object

per-instance method
→ potentially one function object per instance
```

Per-instance closures may be justified when they intentionally capture instance-specific state.

# 19. Security Considerations

Constructors can establish trusted state such as:

```text
identity
permissions
tenant information
resource access
ownership.
```

Do not use incidental prototype relationships as authorization.

Do not let untrusted input bypass constructor invariants through later mutation.

# 20. Production Usage

Good construction boundaries commonly make:

```text
required dependencies explicit
initial state valid
invariants established
object immediately usable
```

Avoid turning constructors into hidden:

```text
database clients
HTTP workflows
application orchestrators
long asynchronous initialization pipelines
```

unless the lifecycle contract explicitly requires it.

# 21. Implementation From Scratch

## Exercise 1 — Constructor

Implement:

```js
function User(name, role) {}
```

Requirements:

```text
validate input
initialize state
add shared prototype behavior
```

## Exercise 2 — Prototype Inspector

For an instance, display:

```text
own properties
prototype
prototype-owned methods
constructor relationship
```

## Exercise 3 — Educational `construct`

Build:

```js
function construct(Constructor, args) {}
```

Model:

```text
prototype connection
constructor call
object return handling
```

Explicitly document that it is educational and does not fully reimplement ECMAScript.

## Exercise 4 — Money Constructor

Implement:

```text
Money
amount
currency
validation
normalization
equals
immutability policy
```

## Exercise 5 — Dependency Constructor

Design:

```js
class UserRepository {
  constructor(database, clock, logger) {}
}
```

State which dependencies are required and why.

# 22. Debugging Exercises

## Debug 1 — Missing `new`

```js
function User(name) {
  this.name = name;
}

const user = User("Milan");
console.log(user);
```

Explain the result under relevant invocation semantics and why relying on accidental call mode is unsafe.

## Debug 2 — Per-Instance Method

```js
function User(name) {
  this.name = name;

  this.greet = function () {
    return this.name;
  };
}
```

Move shared behavior to the prototype and explain the trade-off.

## Debug 3 — Prototype Replacement

```js
function User() {}

const a = new User();

User.prototype = {
  greet() {
    return "hello";
  },
};

const b = new User();

console.log(Object.getPrototypeOf(a) === Object.getPrototypeOf(b));
```

Trace both prototype identities.

## Debug 4 — Constructor Return

```js
function User() {
  this.name = "Milan";
  return { name: "Replacement" };
}

const user = new User();

console.log(user.name);
console.log(user instanceof User);
```

Explain both outputs.

# 23. Code Review Exercise

Review:

```js
class OrderService {
  constructor(database) {
    this.database = database;
    this.loadAllOrders();
  }

  async loadAllOrders() {
    return this.database.query("SELECT * FROM orders");
  }
}
```

Evaluate:

```text
1. Should construction perform I/O?
2. Does the instance become usable immediately?
3. Where should asynchronous initialization live?
4. What invariant should exist after construction?
5. Is database an explicit dependency?
6. Would a factory or startup workflow be clearer?
```

# 24. Interview Questions

```text
1. What is a constructor function?
2. What is the difference between Call and Construct?
3. What does new do conceptually?
4. What is constructability?
5. Why are arrow functions not constructors?
6. How is Constructor.prototype related to an instance?
7. Where should shared instance methods live?
8. What happens if Constructor.prototype is replaced?
9. What happens when a constructor returns an object?
10. What happens when it returns a primitive?
11. What is new.target?
12. Why does a derived constructor need super?
13. What is instanceof testing at a high level?
14. Why is constructor not a perfect provenance check?
15. When should a factory replace a constructor?
16. What should a constructor be responsible for?
17. Should a constructor do database I/O?
18. How do constructor invariants improve object design?
```

# 25. Predict-the-Output Exercises

## A

```js
function User(name) {
  this.name = name;
}

const user = new User("Milan");

console.log(user.name);
console.log(Object.getPrototypeOf(user) === User.prototype);
```

## B

```js
function User() {
  this.value = 1;
  return { value: 2 };
}

const user = new User();

console.log(user.value);
console.log(user instanceof User);
```

## C

```js
function User() {
  this.value = 1;
  return 100;
}

const user = new User();

console.log(user.value);
```

## D

```js
function User() {}

User.prototype.greet = function () {
  return "hello";
};

const a = new User();
const b = new User();

console.log(a.greet === b.greet);
```

## E

```js
function User() {}

const a = new User();

User.prototype = {};

const b = new User();

console.log(Object.getPrototypeOf(a) === Object.getPrototypeOf(b));
```

## F

```js
function User() {
  console.log(new.target === User);
}

new User();
User();
```

# 26. Mastery Exercises

## Level 1 — Understand

Explain:

```text
Call
Construct
constructor
new
constructability
prototype
instance
```

## Level 2 — Explain

Explain why:

```text
function can be callable without being constructable.
```

## Level 3 — Predict

Trace:

```text
prototype selection
constructor execution
this
object return
primitive return
```

## Level 4 — Implement

Build:

```text
constructor
prototype methods
validator
prototype inspector
educational construct helper
```

## Level 5 — Debug

Fix:

```text
missing new
per-instance method duplication
prototype replacement surprise
constructor-return confusion
```

## Level 6 — Apply

Design an `Order` whose construction establishes:

```text
id
items
status
createdAt
invariants
```

## Level 7 — Compare

Compare:

```text
constructor
factory
object literal
Object.create
class
```

## Level 8 — Defend

Answer:

> Why should object construction be treated as a design boundary rather than merely a syntax operation?

Connect:

```text
identity
initial state
invariants
dependencies
prototype behavior
lifecycle
testability.
```

# 27. Key Takeaways

```text
1. Call and Construct are different language operations.
2. Not every callable function is constructable.
3. Arrow functions are not constructors.
4. Classes require construction semantics.
5. new performs constructor-specific instance creation and initialization.
6. Constructor.prototype normally becomes the new instance's [[Prototype]].
7. Shared methods can live on the prototype.
8. Per-instance methods create separate function values.
9. Replacing the prototype affects future construction more directly than existing instances.
10. constructor is a property, not a magical provenance guarantee.
11. Object-returning constructors can replace the normal result.
12. Primitive returns do not normally replace it.
13. new.target exposes construction context.
14. Derived constructors have special super/this rules.
15. Constructor invariants can make objects safe immediately after creation.
16. Factories are often clearer when production is the primary intent.
17. Constructor lifecycle decisions affect dependency injection and testability.
```

# 28. Concept Connections

## Depends On

```text
Chapter 1 — JavaScript Object Model — Foundation
Chapter 2 — Object Identity, Equality & Mutability
Chapter 3 — Prototype Chain Deep Dive
this
prototype
object identity
```

## Builds Toward

```text
Chapter 5 — JavaScript Classes Internally
Chapter 6 — Class Fields and Initialization
Chapter 7 — Private State & Encapsulation
Chapter 8 — Abstraction
Chapter 9 — Inheritance
Chapter 10 — Polymorphism
Chapter 11 — Composition
GRASP
TypeScript constructors
dependency injection
factories
builders
domain invariants
```

## Related Concepts

```text
factory methods
builders
dependency injection
object lifecycle
resource management
value objects
entities
aggregate construction
```

## Why This Chapter Matters

Construction determines the starting state of every object.

Weak construction creates:

```text
invalid state
hidden dependencies
ambiguous lifecycle
```

Strong construction creates:

```text
explicit dependencies
valid state
clear ownership
predictable lifecycle
```

# 29. Completion Criteria

Mark:

```text
[+] Completed
```

when you can:

```text
explain Call vs Construct
trace new
explain prototype selection
explain constructor return rules
explain new.target
explain prototype replacement
explain derived construction.
```

Mark:

```text
[*] Mastered
```

when you can choose and defend:

```text
constructor
vs
factory
vs
builder
vs
direct object creation
```

based on lifecycle, invariants, ownership, dependencies, and complexity.

Reading alone does not mark mastery.

# 30. Revision / Retrieval Record

```md
# Chapter 4 — Revision / Retrieval Record

## Attempt
- Date:
- Duration:
- Status before:
- Status after:

## Construction
- What is Call?
- What is Construct?
- What does new conceptually do?

## Constructor
- Which state is required?
- Which invariants are established?
- Which dependencies are explicit?

## Prototype
- What is Constructor.prototype?
- What is the instance's [[Prototype]]?
- What happens when the prototype is replaced?

## Return Semantics
- Object return:
- Primitive return:

## new.target
- What is it?
- Why is it useful?

## Classes
- Why is super required?
- What changes for derived construction?

## Design Review
- Is the constructor doing too much?
- Can invalid state escape?
- Would a factory be clearer?

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

# 31. Canonical References and Source Discipline

Primary source:

```text
ECMAScript Language Specification
https://tc39.es/ecma262/
```

Useful references:

```text
MDN — new operator
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new

MDN — Classes
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes

MDN — instanceof
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof
```

Source discipline:

```text
construction semantics
→ ECMAScript

standard API behavior
→ standard documentation

engine optimization
→ engine-specific sources

constructor quality
→ application/domain requirements
```

# 32. Completion Snapshot

```text
Chapter: 004
Title: Constructor Functions & Instance Construction

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
new Constructor(args)
        │
        ▼
     Construct
        │
        ▼
   instance created
        │
        ▼
[[Prototype]] → Constructor.prototype
        │
        ▼
constructor executes
with this = instance
        │
        ▼
instance state initialized
        │
        ├─────────────┐
        │             │
   object return   no object return
        │             │
        ▼             ▼
returned object   constructed instance
```

Then ask:

```text
1. Can this thing be constructed?
2. What instance is being initialized?
3. What prototype should it use?
4. What invariants must be true afterward?
5. Can a constructor return a replacement object?
6. Which dependencies are required?
7. Is constructor the right abstraction?
```

# Principal Design Principle

> **Construction is the first lifecycle boundary of an object: make the resulting object valid, explicit about its dependencies, and safe to use immediately.**

# Track Mapping

```text
Track A — Core Theory
    Call vs Construct
    [[Construct]]
    constructability
    new
    new.target
    prototype selection
    constructor return semantics
    class construction

Track B — Implementation
    constructor functions
    prototype methods
    invariant validation
    prototype inspectors
    educational construct helper

Track C — Interview / Reasoning
    construction tracing
    constructor edge cases
    prototype replacement
    return semantics
    constructor vs factory
    lifecycle/design judgment
```
