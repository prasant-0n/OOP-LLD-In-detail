# Chapter 5 — JavaScript Classes Internally

> **JavaScript OOP + LLD Mastery**
>
> This chapter treats JavaScript classes as a language feature to understand rather than a syntax feature to memorize. The goal is to connect class syntax to constructors, prototypes, fields, private state, static state, inheritance, `super`, evaluation order, and real OOP design decisions.

**Status:** `[ ] Not Started`

# 1. Learning Objectives

By the end of this chapter, you should be able to:

```text
[ ] explain what a JavaScript class is
[ ] distinguish class syntax from constructor functions
[ ] explain class declarations and class expressions
[ ] explain class constructors and class strictness
[ ] explain instance methods and prototype sharing
[ ] explain static methods and static state
[ ] explain public instance fields
[ ] explain static fields
[ ] explain private fields and private methods
[ ] explain class field initialization
[ ] explain static initialization blocks
[ ] explain extends and prototype relationships
[ ] explain super in methods and constructors
[ ] explain derived-constructor initialization rules
[ ] explain class lexical/TDZ behavior
[ ] explain computed class elements
[ ] inspect class/prototype relationships with reflection
[ ] compare prototype methods with arrow-function fields
[ ] reason about method extraction and this
[ ] identify constructor initialization hazards
[ ] implement class behavior using lower-level constructs
[ ] choose classes deliberately in object-oriented designs
```

# 2. Prerequisites

```text
Chapter 1 — JavaScript Object Model — Foundation
Chapter 2 — Object Identity, Equality & Mutability
Chapter 3 — Prototype Chain Deep Dive
Chapter 4 — Constructor Functions & Instance Construction
objects
functions
references
prototype chains
constructability
this
new
```

# 3. What Is It?

A JavaScript `class` is a language construct for defining related construction, behavior, state, encapsulation and inheritance semantics.

Example:

```js
class User {
  constructor(name) {
    this.name = name;
  }

  greet() {
    return `Hello ${this.name}`;
  }
}
```

An instance still participates in the object/prototype model:

```text
user
 ↓
User.prototype
 ↓
Object.prototype
 ↓
null
```

The class syntax does not remove prototypes.

> **Classes organize object-oriented semantics; the underlying objects still participate in JavaScript's prototype system.**

# 4. Why Does It Exist?

Class syntax provides a structured way to express:

```text
instance initialization
shared behavior
static/class-side behavior
inheritance
encapsulation
```

It also gives language-level semantics for:

```text
strict class bodies
private names
super
class fields
static initialization
```

Therefore:

```text
class
≠
constructor function with prettier formatting
```

# 5. Mental Model

Think about a class as multiple coordinated surfaces:

```text
                         CLASS
                           │
              ┌────────────┴────────────┐
              │                         │
        constructor/class value      prototype
              │                         │
       static members             instance methods
              │                         │
              │                    instance objects
              │                         │
              └───────────────┐         │
                              │         │
                              └──→ [[Prototype]]
```

For inheritance:

```text
Dog instance
    ↓
Dog.prototype
    ↓
Animal.prototype
    ↓
Object.prototype
    ↓
null
```

Class inheritance also establishes a relationship on the constructor/class side.

# 6. Core Rules

## Rule 1 — Class Constructors Require Construction

```js
class User {}

User(); // TypeError
new User(); // valid
```

## Rule 2 — Class Bodies Are Strict

Methods in class definitions use strict-mode semantics.

## Rule 3 — Instance Methods Normally Live on the Prototype

```js
class User {
  greet() {}
}
```

For ordinary instances:

```js
const a = new User();
const b = new User();

a.greet === b.greet; // true
```

## Rule 4 — Static Members Live on the Class Value

```js
class User {
  static createGuest() {
    return new User();
  }
}
```

Call:

```js
User.createGuest();
```

## Rule 5 — Public Instance Fields Become Own State

```js
class User {
  active = true;
}
```

Each instance gets its own `active` field.

## Rule 6 — Private Fields Are Not Ordinary Property Keys

```js
class Account {
  #balance = 0;
}
```

The private name is governed by class private-name semantics.

## Rule 7 — `extends` Establishes Prototype Relationships

```js
class Dog extends Animal {}
```

The resulting inheritance includes a relationship between the prototype objects.

# 7. Syntax

## Class Declaration

```js
class User {
  constructor(name) {
    this.name = name;
  }

  greet() {
    return `Hello ${this.name}`;
  }
}
```

## Class Expression

```js
const User = class {
  greet() {
    return "hello";
  }
};
```

## Static Method

```js
class User {
  static createGuest() {
    return new User("Guest");
  }
}
```

## Public Instance Field

```js
class User {
  active = true;
}
```

## Private Field

```js
class Account {
  #balance = 0;
}
```

## Static Field

```js
class Config {
  static version = 1;
}
```

## Private Method

```js
class User {
  #normalize(value) {
    return value.trim();
  }
}
```

# 8. Basic Examples

## Example 1 — Class and Prototype

```js
class User {
  greet() {
    return "hello";
  }
}

const a = new User();
const b = new User();

console.log(a.greet());
console.log(a.greet === b.greet);
console.log(Object.getPrototypeOf(a) === User.prototype);
```

**Prediction**

```text
hello
true
true
```

**Trace**

```text
a → User instance
a.[[Prototype]] → User.prototype
User.prototype.greet → shared function
```

## Example 2 — Static and Instance

```js
class User {
  constructor(name) {
    this.name = name;
  }

  greet() {
    return `Hello ${this.name}`;
  }

  static guest() {
    return new User("Guest");
  }
}

const user = User.guest();

console.log(user.greet());
```

**Prediction**

```text
Hello Guest
```

## Example 3 — Instance Field

```js
class Counter {
  count = 0;

  increment() {
    this.count += 1;
  }
}

const a = new Counter();
const b = new Counter();

a.increment();

console.log(a.count);
console.log(b.count);
```

**Prediction**

```text
1
0
```

# 9. Execution Walkthrough

Consider:

```js
class User {
  name = "Unknown";

  constructor(name) {
    this.name = name;
  }

  greet() {
    return `Hello ${this.name}`;
  }
}

const user = new User("Milan");
```

Conceptual flow:

```text
1. Evaluate the class definition.
2. Establish the class/constructor value.
3. Create the prototype object and class elements.
4. Define instance methods according to class semantics.
5. Construct an instance using new.
6. Establish the instance's prototype relationship.
7. Initialize instance fields according to base-class construction rules.
8. Execute constructor code.
9. Return the instance.
```

Final model:

```text
user
├── own name → "Milan"
│
└── [[Prototype]]
      ↓
User.prototype
      └── greet

User
└── static/class-side members
```

The detailed specification order becomes more involved with inheritance, private elements, computed elements and static initialization.

# 10. Internal Mechanics

A class definition coordinates several concepts:

```text
class value
constructor behavior
prototype object
method definitions
private-name environment
auto-created/explicit constructor behavior
fields
static fields
static blocks
superclass relationships.
```

This means:

```text
class = not merely one object
```

The class value is part of a group of related runtime structures.

# 11. ECMAScript / Specification Semantics

ECMAScript specifies class evaluation and class element definition through dedicated algorithms.

Important specification concepts include:

```text
ClassDefinitionEvaluation
constructor definitions
method definitions
field definitions
private names
superclass processing
class field initialization
static initialization.
```

The specification is the semantic authority. Engine memory layouts and optimization strategies are implementation details.

# 12. Advanced Behavior

## 12.1 Class Methods Are Prototype Methods

```js
class User {
  greet() {}
}
```

The method is normally installed on:

```text
User.prototype
```

A rough conceptual equivalent is:

```js
User.prototype.greet = function () {};
```

But this is not a complete translation because class methods have their own semantics for strictness, `super`, descriptors and method definition.

## 12.2 Static Methods

```js
class MathTools {
  static add(a, b) {
    return a + b;
  }
}
```

The behavior is accessed as:

```js
MathTools.add(1, 2);
```

## 12.3 Class Name and TDZ

```js
console.log(User);
class User {}
```

The class binding is lexical and remains uninitialized until evaluation reaches its declaration.

This produces a `ReferenceError` rather than function-declaration-style behavior.

## 12.4 Computed Elements

```js
const methodName = "greet";

class User {
  [methodName]() {
    return "hello";
  }
}
```

Class evaluation can execute expressions for computed names.

## 12.5 Named Class Expressions

```js
const User = class NamedUser {
  identity() {
    return NamedUser;
  }
};
```

The inner class name can be used for self-reference within the class expression.

# 13. Edge Cases

## Class Called Without `new`

```js
class User {}
User(); // TypeError
```

## Class Before Declaration

```js
new User();
class User {}
```

Fails because the lexical binding is in the temporal dead zone.

## Private Field on Wrong Receiver

```js
class User {
  #name = "Milan";

  getName() {
    return this.#name;
  }
}

const getName = User.prototype.getName;
getName.call({}); // TypeError
```

## Derived Constructor Without Proper Super Initialization

A derived constructor cannot complete normal construction while violating the superclass initialization rules.

## Private Syntax Outside Class Context

A private name such as `#value` cannot be used as an ordinary external property access expression.

# 14. Common Misconceptions

```text
"class means classical runtime objects with no prototypes."
"class methods are copied to every instance."
"static methods exist on instances."
"private fields are hidden string properties."
"class automatically means immutable."
"extends copies every parent method into the child."
"super means parent-code copying."
"fields and prototype methods are the same thing."
"arrow-function fields are prototype methods."
"class declarations are hoisted exactly like functions."
```

# 15. Common Mistakes

```text
[ ] using classes where a factory is clearer
[ ] creating too many arrow-function fields
[ ] doing large workflows in constructors
[ ] calling overridable methods during construction
[ ] confusing static and instance state
[ ] treating private fields as security boundaries
[ ] ignoring derived initialization order
[ ] overusing inheritance
[ ] forgetting that class behavior still depends on prototypes
```

# 16. Comparison With Related Concepts

| Concept | Core role |
|---|---|
| Class | Organizes construction, behavior, state and inheritance semantics |
| Constructor function | Constructable function |
| Prototype | Shared delegation object |
| Instance field | Own instance state |
| Prototype method | Shared instance behavior |
| Static member | Class-side behavior |
| Private field | Class-scoped internal state |
| Factory | Object-producing abstraction |
| Composition | Collaborating object design |
| Inheritance | Delegation/subtyping relationship |

# 17. Performance Considerations

Consider:

```text
prototype-shared methods
per-instance function fields
field allocation
object-shape stability
inheritance depth
closure retention.
```

Compare:

```js
class User {
  greet() {}
}
```

with:

```js
class User {
  greet = () => {};
}
```

The second can allocate one function per instance.

Do not assume one strategy is globally faster. Measure the workload.

# 18. Memory Considerations

Potential memory contributors include:

```text
instance fields
per-instance closures
large private object graphs
static caches
registries
prototype structures.
```

Private state can retain data just as public state can. Privacy changes access semantics, not garbage-collection reachability.

# 19. Security Considerations

Private elements improve encapsulation but are not substitutes for:

```text
authentication
authorization
input validation
cryptographic protection
process isolation.
```

Prototype pollution remains relevant in class-based systems because class instances still use prototypes.

Do not use:

```text
instanceof
constructor
private fields
```

as accidental substitutes for an explicit security model.

# 20. Production Usage

A class is often justified when the design benefits from:

```text
stable instance identity
shared behavior
explicit lifecycle
encapsulation
polymorphic contracts
class-level behavior.
```

A factory/module/composed object can be clearer when:

```text
returned shapes vary
closure state is simpler
inheritance would add coupling
creation is the main abstraction.
```

# 21. Implementation From Scratch

## Exercise 1 — Class to Constructor Function

Translate a class with:

```text
constructor
prototype method
static method
```

into constructor-function form.

Compare observable behavior.

## Exercise 2 — Factory Equivalent

Implement a factory equivalent for a small class and compare:

```text
privacy
prototype sharing
identity
memory
ergonomics.
```

## Exercise 3 — Inheritance Equivalent

Implement `Animal` and `Dog` using:

```text
classes
```

and then using explicit prototype relationships.

Draw both graphs.

## Exercise 4 — Class Inspector

For a class, inspect:

```text
class value
prototype
instance own properties
prototype methods
static properties
descriptors.
```

## Exercise 5 — Private-State Alternative

Recreate a class with `#state` using a closure-based factory.

Compare the abstraction boundaries.

# 22. Debugging Exercises

## Debug 1 — Detached Method

```js
class User {
  name = "Milan";

  greet() {
    return this.name;
  }
}

const user = new User();
const greet = user.greet;

console.log(greet());
```

Explain the `this` behavior. Compare with an arrow-function field.

## Debug 2 — Virtual Method During Construction

```js
class Base {
  constructor() {
    this.init();
  }

  init() {}
}

class Child extends Base {
  value = 100;

  init() {
    console.log(this.value);
  }
}

new Child();
```

Explain the initialization-order hazard.

## Debug 3 — Static vs Instance

```js
class User {
  static role = "admin";
  role = "user";
}

const user = new User();

console.log(user.role);
console.log(User.role);
```

## Debug 4 — Private Receiver

```js
class Account {
  #balance = 100;

  getBalance() {
    return this.#balance;
  }
}

const account = new Account();
const getBalance = account.getBalance;

console.log(getBalance.call({}));
```

Explain the error.

# 23. Code Review Exercise

Review:

```js
class OrderService {
  orders = [];

  constructor(database) {
    this.database = database;
    this.load();
  }

  async load() {
    this.orders = await this.database.findAll();
  }

  create(order) {
    this.orders.push(order);
  }
}
```

Evaluate:

```text
1. Should construction trigger asynchronous work?
2. What state is valid before load resolves?
3. Is orders domain state or a cache?
4. Should callers mutate it directly?
5. Is database an explicit dependency?
6. Would a repository/service split be clearer?
7. Should the object be usable immediately after construction?
8. Is a class improving this design?
```

# 24. Interview Questions

```text
1. What is a JavaScript class?
2. Are JavaScript classes still prototype-based?
3. Where do instance methods live?
4. Where do static methods live?
5. What is the difference between an instance field and a prototype method?
6. Why can't a class constructor be called directly?
7. What happens conceptually when a class is evaluated?
8. What is the relationship between instance and Class.prototype?
9. How does extends affect prototypes?
10. What does super do?
11. Why are class bodies strict?
12. What are private fields?
13. How are private fields different from Symbol properties?
14. What happens if a private method uses the wrong receiver?
15. Why are arrow-function fields different from prototype methods?
16. What is class TDZ behavior?
17. What is static initialization?
18. Why is calling overridable methods from constructors risky?
19. When should you use a factory instead of a class?
20. Why should classes not be chosen merely because they look object-oriented?
```

# 25. Predict-the-Output Exercises

## Exercise A

```js
class User {
  greet() {
    return "hello";
  }
}

const a = new User();
const b = new User();

console.log(a.greet === b.greet);
console.log(Object.getPrototypeOf(a) === User.prototype);
```

## Exercise B

```js
class User {
  static role = "admin";
  role = "user";
}

const user = new User();

console.log(user.role);
console.log(User.role);
```

## Exercise C

```js
class User {
  #name = "Milan";

  getName() {
    return this.#name;
  }
}

const user = new User();

console.log(user.getName());
console.log(Object.hasOwn(user, "#name"));
```

## Exercise D

```js
class Base {
  value = 10;

  constructor() {
    console.log(this.value);
  }
}

class Child extends Base {
  value = 20;
}

new Child();
```

Predict initialization order.

## Exercise E

```js
class User {
  constructor(name) {
    this.name = name;
  }

  greet() {
    return this.name;
  }
}

const user = new User("Milan");
const fn = user.greet;

console.log(fn());
```

# 26. Mastery Exercises

## Level 1 — Understand

Explain:

```text
class
constructor
prototype
instance field
static field
private field
extends
super
```

## Level 2 — Explain

Draw:

```text
User
User.prototype
instance
instance.[[Prototype]]
```

and then do the same for:

```text
Dog extends Animal
```

## Level 3 — Predict

Trace:

```text
field initialization
constructor execution
prototype lookup
static access
private access
derived construction
```

## Level 4 — Implement

Build:

```text
class
constructor-function equivalent
factory equivalent
prototype inheritance equivalent
private-state closure equivalent
```

## Level 5 — Debug

Fix:

```text
detached method
constructor virtual-dispatch bug
static/instance confusion
private receiver error
```

## Level 6 — Apply

Design:

```text
Payment
CardPayment
CashPayment
```

using either inheritance or composition and defend the choice.

## Level 7 — Compare

Compare:

```text
class
constructor function
factory
object literal
module
```

for creating a domain service.

## Level 8 — Defend

Answer:

> Why should a JavaScript engineer understand prototypes before using classes heavily?

Connect:

```text
class syntax
prototype behavior
identity
inheritance
composition
performance
encapsulation.
```

# 27. Key Takeaways

```text
1. JavaScript classes are language constructs built on the object/prototype model.
2. Class constructors require construction semantics.
3. Class bodies use strict semantics.
4. Instance methods normally live on Class.prototype.
5. Static methods live on the class value.
6. Public instance fields become own instance properties.
7. Private elements have class-level private-name semantics rather than ordinary property-key semantics.
8. Static fields belong to the class side.
9. Static initialization can execute during class evaluation.
10. extends establishes prototype inheritance relationships.
11. super provides superclass-oriented access/construction semantics.
12. Derived constructors have special initialization rules.
13. Prototype methods are normally shared.
14. Arrow-function fields are per-instance functions.
15. Class declarations use lexical/TDZ behavior.
16. Computed class elements can execute during class evaluation.
17. Calling overridable methods from constructors can expose partially initialized state.
18. Private fields improve encapsulation but are not security mechanisms.
19. Class syntax does not remove the need to understand prototypes.
20. A class is a design abstraction, not a syntax requirement.
```

# 28. Concept Connections

## Depends On

```text
Chapter 1 — JavaScript Object Model — Foundation
Chapter 2 — Object Identity, Equality & Mutability
Chapter 3 — Prototype Chain Deep Dive
Chapter 4 — Constructor Functions & Instance Construction
new
this
prototype
constructability
```

## Builds Toward

```text
Chapter 6 — Class Fields & Initialization Semantics
Chapter 7 — Private State & Encapsulation
Chapter 8 — Abstraction
Chapter 9 — Inheritance
Chapter 10 — Polymorphism
Chapter 11 — Composition
Chapter 17 — Cohesion & Coupling
GRASP
SOLID
TypeScript OOP
Dependency Injection
Design Patterns
Domain Modeling
```

## Related Concepts

```text
constructor functions
factories
closures
delegation
composition
private state
polymorphism
class-side behavior
```

## Why This Chapter Matters

Knowing class syntax lets you write classes.

Knowing the internals lets you answer:

```text
Where does this behavior live?
Why is this method shared?
Why is this field per-instance?
Why does this private access fail?
Why does this derived constructor behave this way?
Why did this method lose its this?
Should this abstraction even be a class?
```

# 29. Completion Criteria

Mark:

```text
[+] Completed
```

when you can:

```text
explain class evaluation conceptually
trace instance construction
explain prototype method sharing
explain static behavior
explain public/private fields
explain extends/super
explain class TDZ
explain initialization order.
```

Mark:

```text
[*] Mastered
```

when you can take an unfamiliar class hierarchy and:

```text
draw its runtime relationships
predict initialization
identify state ownership
identify encapsulation leaks
spot inheritance hazards
justify class vs factory vs composition.
```

Reading alone does not mark mastery.

# 30. Revision / Retrieval Record

```md
# Chapter 5 — Revision / Retrieval Record

## Attempt
- Date:
- Duration:
- Status before:
- Status after:

## Class Internals
- What does a class create?
- What is Class.prototype?
- What is the class/static side?

## Methods
- Where do instance methods live?
- Where do static methods live?
- How are arrow-function fields different?

## Fields
- Public instance field:
- Static field:
- Private field:
- Static private field:

## Inheritance
- What does extends establish?
- What does super do?
- Why is derived construction special?

## Initialization
- What runs first?
- What can go wrong when constructors call overridable methods?

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

# 31. Canonical References and Source Discipline

Primary semantic source:

```text
ECMAScript Language Specification
https://tc39.es/ecma262/
```

Useful references:

```text
MDN — Classes
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes

MDN — Private elements
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_elements

MDN — Public class fields
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Public_class_fields

MDN — static
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/static

MDN — extends
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/extends
```

Source discipline:

```text
class evaluation semantics
→ ECMAScript

standard API behavior
→ standard documentation

engine optimization
→ engine-specific sources

design quality
→ domain requirements and architecture
```

Do not reduce class semantics to a simplistic transpilation model.

# 32. Completion Snapshot

```text
Chapter: 005
Title: JavaScript Classes Internally

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
class User {
  name = "Unknown";

  constructor(name) {
    this.name = name;
  }

  greet() {
    return `Hello ${this.name}`;
  }

  static guest() {
    return new User("Guest");
  }

  #secret = 123;
}
```

Reason in layers:

```text
                         User
                          │
             ┌────────────┴────────────┐
             │                         │
        static/class side         User.prototype
             │                         │
          guest()                   greet()
             │                         │
             │                         ▼
             │                     instance
             │                         │
             │                    own state
             │                    ├── name
             │                    └── #secret
             │
             └────────────────────────────
```

For inheritance:

```text
Dog instance
    ↓
Dog.prototype
    ↓
Animal.prototype
    ↓
Object.prototype
    ↓
null
```

Then ask:

```text
1. Where does each behavior live?
2. Which state is per-instance?
3. Which state is class-wide?
4. Which state is private?
5. What happens during construction?
6. What is the prototype chain?
7. Is inheritance necessary?
8. Could composition or a factory be clearer?
```

# Principal Design Principle

> **A class is valuable when its construction, state, behavior, lifecycle and polymorphic relationships form a meaningful abstraction—not simply because JavaScript has a `class` keyword.**

# Track Mapping

```text
Track A — Core Theory
    class evaluation
    prototypes
    constructors
    fields
    private names
    static side
    extends
    super
    initialization order

Track B — Implementation
    class/constructor equivalence
    prototype equivalence
    factory alternatives
    closure-based private state
    class inspection

Track C — Interview / Reasoning
    class internals
    TDZ
    static vs instance
    field ordering
    virtual-dispatch constructor hazards
    private-brand reasoning
    class vs factory/composition decisions
```
