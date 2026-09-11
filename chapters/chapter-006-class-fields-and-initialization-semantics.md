# Chapter 6 — Class Fields & Initialization Semantics

> **JavaScript OOP + LLD Mastery**
>
> This chapter goes deeper than basic class syntax and focuses on **when and where class state is created**. Public fields, private fields, static fields, computed fields, field initializers, inheritance, and constructor execution order all affect whether an object is valid, predictable, and extensible.
>
> **Central question:** At every point during construction, which state is guaranteed to exist?

**Status:** `[ ] Not Started`

# 1. Learning Objectives

By the end of this chapter, you should be able to:

```text
[ ] explain public instance fields
[ ] explain public static fields
[ ] explain private instance fields
[ ] explain private static fields
[ ] explain instance field initializers
[ ] explain static field initializers
[ ] explain computed field names
[ ] explain class element evaluation at a high level
[ ] trace field initialization order
[ ] distinguish base-class and derived-class initialization
[ ] explain when this becomes available
[ ] explain why super() matters to derived constructors
[ ] reason about field initializers that reference other fields
[ ] reason about field initializers that call methods
[ ] explain private-field brand initialization conceptually
[ ] distinguish instance fields from prototype methods
[ ] distinguish static fields from instance fields
[ ] understand how inheritance interacts with fields
[ ] identify partial-initialization hazards
[ ] identify constructor/field-ordering hazards
[ ] reason about overriding and field initialization
[ ] compare field syntax with constructor assignment
[ ] compare public fields with private fields
[ ] compare field functions with prototype methods
[ ] design reliable initialization order
[ ] debug field initialization bugs
[ ] implement equivalent initialization using lower-level mechanisms
[ ] connect initialization semantics to invariants and lifecycle design
```

# 2. Prerequisites

Required:

```text
Chapter 1 — JavaScript Object Model — Foundation
Chapter 2 — Object Identity, Equality & Mutability
Chapter 3 — Prototype Chain Deep Dive
Chapter 4 — Constructor Functions & Instance Construction
Chapter 5 — JavaScript Classes Internally
objects
classes
constructors
prototype chains
constructability
this
new
extends
super
```

# 3. What Is It?

A class field declares state associated with an instance or the class/static side.

```js
class User {
  name = "Unknown";
  static category = "user";
}
```

Conceptually:

```text
User instance
├── name

User class
└── category
```

Private fields use a separate language-level private state mechanism:

```js
class Account {
  #balance = 0;
}
```

# 4. Why Does It Exist?

Before class fields, instance state was commonly initialized in constructors:

```js
class User {
  constructor(name) {
    this.name = name;
    this.active = true;
  }
}
```

Field syntax can express default state near the class definition:

```js
class User {
  name = "Unknown";
  active = true;

  constructor(name) {
    this.name = name;
  }
}
```

Fields also support:

```text
private state
static state
derived-class initialization
declarative defaults
```

But field syntax does not remove the need to understand initialization ordering.

# 5. Mental Model

Think of class evaluation and instance construction as two related phases.

```text
CLASS EVALUATION
      │
      ├── class methods
      ├── static members
      ├── private names
      ├── computed keys
      └── static initialization
                │
                ▼
          CLASS READY
                │
                ▼
        INSTANCE CONSTRUCTION
                │
                ├── base initialization
                ├── instance fields
                ├── constructor body
                │
                └── derived initialization
```

For inheritance:

```text
Base class
   ↓
base instance initialization
   ↓
base constructor
   ↓
derived fields
   ↓
derived constructor body
```

# 6. Core Rules

## Rule 1 — Instance Fields Belong to Each Instance

```js
class Counter {
  count = 0;
}
```

Each instance gets its own initialized `count`.

## Rule 2 — Static Fields Belong to the Class Side

```js
class Counter {
  static created = 0;
}
```

Access through `Counter.created`.

## Rule 3 — Private Fields Are Not Ordinary Public Properties

```js
class Account {
  #balance = 0;
}
```

The private name is not a string key accessible through `account["#balance"]`.

## Rule 4 — Instance Fields Are Initialized During Construction

A field declaration participates in class construction semantics rather than simply existing as source text.

## Rule 5 — Derived Classes Have Special Initialization Rules

A derived constructor must perform the superclass construction step before normal use of `this`.

## Rule 6 — Field Order Matters

```js
class User {
  first = "Milan";
  last = `${this.first} Prusty`;
}
```

The second initializer observes state established by the first.

## Rule 7 — Field Initializers Can Execute Code

```js
class Service {
  value = createValue();
}
```

The expression executes during relevant initialization and can have side effects, dependencies, cost, and failure.

# 7. Syntax

## Public Instance Field

```js
class User {
  name = "Milan";
}
```

## Public Static Field

```js
class User {
  static role = "user";
}
```

## Private Instance Field

```js
class Account {
  #balance = 0;
}
```

## Private Static Field

```js
class Registry {
  static #items = [];
}
```

## Private Method

```js
class User {
  #normalize(name) {
    return name.trim();
  }
}
```

## Computed Field Name

```js
const key = "name";

class User {
  [key] = "Milan";
}
```

# 8. Basic Examples

## Example 1 — Independent Instance State

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

**Trace**

```text
a → own count = 0
b → own count = 0

a.increment()
→ a.count becomes 1

b.count remains 0
```

## Example 2 — Static State

```js
class Counter {
  static created = 0;

  constructor() {
    Counter.created += 1;
  }
}

new Counter();
new Counter();

console.log(Counter.created);
```

**Prediction**

```text
2
```

## Example 3 — Initialization Order

```js
class User {
  first = "Milan";
  full = `${this.first} Prusty`;
}

const user = new User();

console.log(user.full);
```

**Prediction**

```text
Milan Prusty
```

# 9. Execution Walkthrough

Consider:

```js
class User {
  name = "Unknown";
  active = true;

  constructor(name) {
    this.name = name;
  }
}

const user = new User("Milan");
```

For a base class, reason conceptually:

```text
1. Class definition is evaluated.
2. Field definitions participate in class construction semantics.
3. new starts construction.
4. Instance is created.
5. Instance fields are initialized.
6. Constructor body executes.
7. Constructor overwrites name with "Milan".
8. Final instance is returned.
```

Final state:

```text
user
├── name → "Milan"
└── active → true
```

# 10. Internal Mechanics

A class field is an initialization step rather than a prototype method.

```js
class User {
  name = "Milan";

  greet() {}
}
```

Conceptually:

```text
instance
└── name

User.prototype
└── greet
```

The field is instance state. The method is prototype-shared behavior.

# 11. ECMAScript / Specification Semantics

ECMAScript specifies class evaluation and construction around concepts including:

```text
class definition evaluation
class elements
field definitions
private names
instance field initialization
static field initialization
constructor evaluation
derived construction
```

Source-code order must be interpreted through base/derived class semantics and field initialization rules rather than treated as a simple top-to-bottom script.

The ECMAScript specification is authoritative for exact observable behavior.

# 12. Advanced Behavior

## 12.1 Public Fields Become Own Properties

```js
class User {
  name = "Milan";
}

const user = new User();
console.log(Object.hasOwn(user, "name"));
```

Expected:

```text
true
```

## 12.2 Prototype Methods Remain Separate

```js
class User {
  name = "Milan";

  greet() {
    return this.name;
  }
}
```

Layout:

```text
user
├── name
└── [[Prototype]]
      ↓
User.prototype
└── greet
```

## 12.3 Field Initializers Can Read Earlier Fields

```js
class User {
  first = "Milan";
  last = "Prusty";
  full = `${this.first} ${this.last}`;
}
```

## 12.4 Field Initializers Can Call Methods

```js
class User {
  name = this.normalize("Milan");

  normalize(value) {
    return value.trim();
  }
}
```

This requires careful reasoning about lookup, receiver, and possible overriding.

# 13. Initialization Order

## Base Class

Conceptually:

```text
allocate instance
→ initialize base instance fields
→ execute base constructor body
```

## Derived Class

Conceptually:

```text
derived constructor begins
→ super(...)
→ base construction/initialization
→ derived instance fields initialize
→ derived constructor body continues
```

Use the specification for exact corner cases.

# 14. Derived Field Initialization

```js
class Animal {
  name = "animal";
}

class Dog extends Animal {
  breed = "unknown";
}

const dog = new Dog();

console.log(dog.name);
console.log(dog.breed);
```

Result:

```text
animal
unknown
```

# 15. Overriding Field Names

```js
class Base {
  value = "base";
}

class Child extends Base {
  value = "child";
}

console.log(new Child().value);
```

Result:

```text
child
```

This is state initialization/shadowing, not prototype method overriding.

# 16. Fields Are Not Virtual Methods

A field:

```js
value = 10;
```

stores state.

A method:

```js
getValue() {
  return 10;
}
```

expresses behavior.

They should not be treated as interchangeable design mechanisms.

# 17. Constructor and Field Interactions

```js
class Base {
  value = 10;

  constructor() {
    console.log(this.value);
  }
}

new Base();
```

A base constructor can observe the initialized base field.

# 18. Derived Constructor Hazard

```js
class Base {
  constructor() {
    console.log(this.value);
  }
}

class Child extends Base {
  value = 20;
}

new Child();
```

When the base constructor executes, derived fields have not completed their initialization.

Core rule:

> A base constructor must not assume derived instance fields already exist.

# 19. Constructor Virtual Dispatch Hazard

```js
class Base {
  constructor() {
    this.initialize();
  }

  initialize() {
    return "base";
  }
}

class Child extends Base {
  value = 100;

  initialize() {
    console.log(this.value);
  }
}

new Child();
```

The base constructor invokes behavior overridden by the child before derived fields are initialized.

Design rule:

> Avoid calling overridable behavior from constructors unless partial initialization is explicitly safe.

# 20. Field Initializers and `this`

```js
class User {
  name = "Milan";
  greeting = `Hello ${this.name}`;
}
```

The later initializer uses the instance state established by the earlier initializer.

# 21. Field Initializers and Errors

If:

```js
class Service {
  client = createClient();
}
```

and `createClient()` throws, construction fails.

Therefore decide carefully whether work belongs in:

```text
field initialization
constructor
factory
async initialization
application startup
```

# 22. Private Field Initialization

```js
class Account {
  #balance = 100;

  getBalance() {
    return this.#balance;
  }
}
```

The relevant private state is initialized during construction.

# 23. Private Brand Reasoning

Useful mental model:

```text
class private definition
        ↓
private name/brand
        ↓
instance initialization
        ↓
instance gains appropriate private state
```

Calling a method that accesses private state with an unrelated receiver can throw.

# 24. Private Fields and Inheritance

Private fields are not ordinary inherited public properties.

```js
class Base {
  #secret = 1;

  getSecret() {
    return this.#secret;
  }
}

class Child extends Base {}
```

Inherited behavior can access the Base private state of a compatible instance, but subclasses cannot simply access the Base private name as an ordinary property.

# 25. Same Private Name Syntax, Different Class

Private names declared in separate class definitions are distinct even if their spelling is identical.

Conceptually:

```text
Base.#value ≠ Child.#value
```

unless referring to the exact same private declaration.

# 26. Static Fields and Initialization

```js
class Config {
  static version = 1;
}
```

The field belongs to the class/static side.

# 27. Static Field Dependencies

```js
class Config {
  static version = 1;
  static label = `v${Config.version}`;
}
```

Static initialization order matters.

# 28. Static Private State

```js
class Registry {
  static #items = new Map();

  static add(key, value) {
    this.#items.set(key, value);
  }

  static get(key) {
    return this.#items.get(key);
  }
}
```

Because this state is class-level, it can create long-lived shared state and testing concerns.

# 29. Computed Fields

```js
const key = "displayName";

class User {
  [key] = "Milan";
}
```

The expression producing the key participates in class evaluation and can have dependencies, side effects, or failures.

# 30. Field Initializers vs Constructor Assignment

Compare:

```js
class User {
  active = true;
}
```

with:

```js
class User {
  constructor() {
    this.active = true;
  }
}
```

They can express similar simple state, but field initialization has class-specific ordering semantics, especially with inheritance.

# 31. Default State vs Derived State

Prefer:

```js
class User {
  firstName = "";
  lastName = "";

  get displayName() {
    return `${this.firstName} ${this.lastName}`.trim();
  }
}
```

when duplicated mutable state would otherwise drift.

# 32. Fields as Configuration

```js
class RetryPolicy {
  maxRetries = 3;
  baseDelay = 100;
}
```

Ask whether these should instead be constructor inputs or external configuration.

# 33. Fields and Invariants

```js
class Money {
  amount;
  currency;

  constructor(amount, currency) {
    if (!Number.isFinite(amount)) {
      throw new TypeError("Invalid amount");
    }

    this.amount = amount;
    this.currency = currency.toUpperCase();
  }
}
```

Fields can declare state while the constructor enforces domain validity.

# 34. Advanced Behavior — Field Initializer Side Effects

Avoid hidden expensive lifecycle work such as:

```js
class Service {
  database = connectToDatabase();
}
```

unless the lifecycle explicitly requires it.

Prefer explicit dependencies when appropriate:

```js
class Service {
  constructor(database) {
    this.database = database;
  }
}
```

# 35. Advanced Behavior — Field Function Allocation

Compare:

```js
class User {
  greet = () => this.name;
}
```

with:

```js
class User {
  greet() {
    return this.name;
  }
}
```

The first is a per-instance function field; the second is normally a shared prototype method.

# 36. Advanced Behavior — Field Function and Inheritance

```js
class Base {
  greet = () => "base";
}

class Child extends Base {
  greet = () => "child";
}
```

This is per-instance function state, not prototype method overriding.

# 37. Advanced Behavior — Field Ordering Within One Class

```js
class Example {
  a = 1;
  b = this.a + 1;
  c = this.b + 1;
}
```

Reordering the fields can change behavior because initializers can depend on earlier fields.

# 38. Edge Cases

Important edge cases include:

```text
field reads before initialization
derived fields before super
private access before brand initialization
static initialization failure
computed-key failure
large field allocations
field functions created per instance
base constructor reading derived state
constructor virtual dispatch
```

# 39. Edge Case — Later Field Reference

```js
class Example {
  b = this.a + 1;
  a = 1;
}
```

The initializer for `b` cannot assume that the later `a` field has already been initialized.

# 40. Edge Case — Derived Initialization Boundary

A derived constructor cannot initialize derived instance fields before the superclass construction step makes the receiver available.

# 41. Edge Case — Private Receiver

```js
class Account {
  #balance = 100;

  getBalance() {
    return this.#balance;
  }
}

const getBalance = new Account().getBalance;
getBalance.call({});
```

The unrelated receiver lacks the required private state and the operation throws.

# 42. Common Misconceptions

```text
"All class fields live on the prototype."
"Static fields exist on instances."
"Private fields are ordinary hidden properties."
"Field initializers have no ordering semantics."
"Derived fields exist before super()."
"Field functions are prototype methods."
"Fields are free allocations."
"Fields automatically enforce domain invariants."
"Private fields are security/encryption mechanisms."
```

# 43. Common Mistakes

```text
[ ] depending on field ordering accidentally
[ ] using expensive expressions in field initializers
[ ] creating per-instance arrow functions without considering allocation
[ ] calling overridable methods during initialization
[ ] assuming base constructors can read derived fields
[ ] using static mutable fields as hidden global state
[ ] duplicating derived state unnecessarily
[ ] using field defaults where validation is required
[ ] hiding lifecycle work inside field initializers
```

# 44. Comparison With Related Concepts

| Concept | Lifecycle / location |
|---|---|
| Instance field | Initialized per instance |
| Static field | Initialized on class/static side |
| Private field | Per-instance private state |
| Static private field | Class-level private state |
| Prototype method | Shared behavior |
| Arrow-function field | Per-instance function value |
| Constructor assignment | Explicit initialization logic |
| Getter | Computed access behavior |
| Factory | Externalized object creation |
| Static registry | Shared class-level state |

# 45. Performance Considerations

Consider:

```text
per-instance allocations
field initializer complexity
function-valued fields
large arrays/maps
repeated construction
constructor + field duplicated work
```

Per-instance arrow-function fields can allocate a function for every instance. Prototype methods normally share one function through the prototype.

Profile before optimizing.

# 46. Memory Considerations

Potential memory growth comes from:

```text
large instance fields
per-instance closures
private state
static collections
cached computed values
duplicated derived state
```

A static collection may remain reachable for the lifetime of the class/module.

# 47. Security Considerations

Public fields are public API surface.

Private fields improve encapsulation but do not replace:

```text
authorization
input validation
secret management
process isolation
cryptographic protection
```

# 48. Production Usage

Prefer predictable initialization:

```text
cheap defaults
explicit dependencies
validated state
clear lifecycle
limited initialization side effects
```

Be cautious with:

```text
database connections in fields
network requests in fields
large allocations in defaults
virtual method calls during construction
static mutable global-like registries
```

# 49. Implementation From Scratch

## Exercise 1 — Field-to-Constructor Translation

Translate:

```js
class User {
  name = "Unknown";
  active = true;
}
```

into constructor assignments and document where the translation is only approximate.

## Exercise 2 — Field Ordering Detector

Identify class fields whose initializers reference later fields.

## Exercise 3 — Private State Alternative

Implement a closure-based equivalent of:

```js
class Account {
  #balance = 0;

  deposit(amount) {
    this.#balance += amount;
  }

  getBalance() {
    return this.#balance;
  }
}
```

Compare encapsulation, memory, prototype sharing, debugging, and inheritance.

## Exercise 4 — Static Registry

Implement a class with a private static `Map` and `add`, `get`, `remove`, `clear` operations. Analyze lifetime and test coupling.

## Exercise 5 — Safe Initialization

Design an `Order` with required constructor inputs, default fields, validated fields, derived state, and private state. Document the initialization order.

# 50. Debugging Exercises

## Debug 1 — Field Order

```js
class Example {
  full = `${this.first} ${this.last}`;
  first = "Milan";
  last = "Prusty";
}

console.log(new Example().full);
```

Explain and fix it.

## Debug 2 — Derived State

```js
class Base {
  constructor() {
    console.log(this.value);
  }
}

class Child extends Base {
  value = 100;
}

new Child();
```

Explain the initialization boundary.

## Debug 3 — Virtual Dispatch During Construction

```js
class Base {
  constructor() {
    this.init();
  }

  init() {}
}

class Child extends Base {
  value = 10;

  init() {
    console.log(this.value);
  }
}

new Child();
```

Find the lifecycle/design bug.

## Debug 4 — Per-Instance Function

```js
class User {
  handler = () => {};
}

const a = new User();
const b = new User();

console.log(a.handler === b.handler);
```

Explain the result and memory implication.

## Debug 5 — Static Shared State

```js
class Cache {
  static items = new Map();
}

const a = Cache.items;
a.set("x", 1);

console.log(Cache.items.get("x"));
```

Explain ownership and lifetime.

# 51. Code Review Exercise

Review:

```js
class OrderService {
  database = connect();
  logger = createLogger();
  cache = new Map();

  constructor() {
    this.load();
  }

  load() {
    this.database.query("SELECT ...");
  }
}
```

Evaluate:

```text
1. Which fields should be injected?
2. Which initialization is hidden?
3. Which operations can throw?
4. Is construction still cheap?
5. What is the lifecycle of database?
6. Who owns cache?
7. Should load be asynchronous?
8. What invariant exists when construction returns?
9. Would a factory/application startup workflow be clearer?
```

# 52. Interview Questions

```text
1. What is a class field?
2. What is the difference between an instance field and a prototype method?
3. What is a static field?
4. What is a private field?
5. What is a static private field?
6. Are class fields own properties?
7. When are instance fields initialized?
8. What is the difference between base and derived initialization?
9. Why does super() matter?
10. Why can field order matter?
11. Can a field initializer call a method?
12. What happens if a field initializer throws?
13. Are derived fields available inside the base constructor?
14. What is dangerous about constructor-time virtual dispatch?
15. How are public fields overridden?
16. Are private fields inherited like public properties?
17. When should a field be private?
18. When should a value be a constructor parameter instead of a field default?
19. When should a field initializer not perform I/O?
20. What is the trade-off of arrow-function fields?
21. Why can static mutable fields behave like global state?
22. How would you design initialization for a complex domain object?
```

# 53. Predict-the-Output Exercises

## Exercise A

```js
class User {
  first = "Milan";
  last = "Prusty";
  full = `${this.first} ${this.last}`;
}

console.log(new User().full);
```

## Exercise B

```js
class User {
  full = `${this.first}`;
  first = "Milan";
}

console.log(new User().full);
```

## Exercise C

```js
class Base {
  value = "base";

  constructor() {
    console.log(this.value);
  }
}

class Child extends Base {
  value = "child";
}

new Child();
```

## Exercise D

```js
class User {
  handler = () => {};
  method() {}
}

const a = new User();
const b = new User();

console.log(a.handler === b.handler);
console.log(a.method === b.method);
```

## Exercise E

```js
class Config {
  static version = 1;
  static label = `v${Config.version}`;
}

console.log(Config.label);
```

## Exercise F

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

# 54. Mastery Exercises

## Level 1 — Understand

Explain:

```text
public field
private field
static field
static private field
field initializer
```

## Level 2 — Explain

Draw initialization for `Base` and `Child extends Base`, including base fields, base constructor, derived fields, and derived constructor.

## Level 3 — Predict

Predict code involving:

```text
field ordering
super
base constructors
derived fields
private state
static initialization
```

## Level 4 — Implement

Build:

```text
validated domain object
private-state class
static registry
closure equivalent
initialization-order test cases
```

## Level 5 — Debug

Fix:

```text
late field dependency
derived-field access from base constructor
constructor virtual dispatch
unnecessary per-instance functions
hidden static state
```

## Level 6 — Apply

Design an `Order` with:

```text
id
items
status
createdAt
private totals
derived display data
```

and document the initialization order.

## Level 7 — Compare

Compare:

```text
field initializer
constructor assignment
getter
prototype property
static field
```

## Level 8 — Defend

Answer:

> Why is initialization order a design concern rather than merely a language-detail concern?

Connect:

```text
partial state
invariants
inheritance
overridable behavior
side effects
testability
LLD
```

# 55. Key Takeaways

```text
1. Instance fields create per-instance state.
2. Static fields create class-level state.
3. Private fields use private-name semantics rather than ordinary string keys.
4. Instance fields are separate from prototype methods.
5. Field initializers execute during construction/evaluation according to class semantics.
6. Field order can affect observable behavior.
7. Base and derived classes have different initialization sequencing.
8. Derived constructors require superclass initialization before normal this use.
9. Base constructors should not assume derived fields are initialized.
10. Calling overridable methods during construction can expose partial state.
11. Field initializers can execute arbitrary expressions and can throw.
12. Large/default allocations in fields can create memory costs.
13. Arrow-function fields are per-instance function values.
14. Static mutable state can behave like a hidden global.
15. Private state improves encapsulation but is not a security system.
16. Constructor assignment and field syntax can express similar state but have different initialization semantics.
17. Initialization should be designed around invariants and lifecycle.
18. Complex initialization may belong in factories or application workflows.
19. Public fields are part of the external object surface.
20. Understanding initialization order is essential for safe inheritance design.
```

# 56. Concept Connections

## Depends On

```text
Chapter 1 — JavaScript Object Model — Foundation
Chapter 2 — Object Identity, Equality & Mutability
Chapter 3 — Prototype Chain Deep Dive
Chapter 4 — Constructor Functions & Instance Construction
Chapter 5 — JavaScript Classes Internally
new
this
prototype
extends
super
```

## Builds Toward

```text
Chapter 7 — Private State & Encapsulation
Chapter 8 — Abstraction
Chapter 9 — Inheritance
Chapter 10 — Polymorphism
Chapter 11 — Composition
Chapter 17 — Cohesion & Coupling
GRASP
SOLID
TypeScript class design
dependency injection
object lifecycle
domain invariants
```

## Related Concepts

```text
constructor initialization
factories
builders
dependency injection
resource lifecycle
immutability
encapsulation
polymorphism
```

## Why This Chapter Matters

Most object design assumes the object is already initialized. Real failures often happen before that point.

Initialization semantics determine whether:

```text
the object is valid
the dependencies exist
the private state exists
the invariants hold
the subclass is safe
```

# 57. Completion Criteria

Mark:

```text
[+] Completed
```

when you can explain each field type, trace base/derived initialization, explain field order, explain private-field initialization, and debug constructor/field interaction.

Mark:

```text
[*] Mastered
```

when you can look at an unfamiliar class hierarchy and answer:

```text
what state exists at each point in construction
which fields are initialized
which methods are safe to call
which invariants are temporarily violated
which side effects can occur
```

without executing the program.

Reading alone does not mark mastery.

# 58. Revision / Retrieval Record

```md
# Chapter 6 — Revision / Retrieval Record

## Attempt
- Date:
- Duration:
- Status before:
- Status after:

## Fields
- Public instance:
- Static:
- Private:
- Static private:

## Initialization
- What happens for a base class?
- What happens for a derived class?
- Why does super matter?

## Ordering
- Which field depends on another?
- Did declaration order matter?

## Private State
- When does private state become available?
- What happens with the wrong receiver?

## Inheritance
- Can the base constructor access derived fields?
- What happens with virtual dispatch?

## Performance
- Which fields allocate per instance?
- Are any field functions unnecessarily duplicated?

## Design
- Which initialization belongs in the constructor?
- Which belongs in a factory/workflow?

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

# 59. Canonical References and Source Discipline

Primary source:

```text
ECMAScript Language Specification
https://tc39.es/ecma262/
```

Useful references:

```text
MDN — Public class fields
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Public_class_fields

MDN — Private elements
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_elements

MDN — Static
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/static

MDN — extends
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/extends
```

Source discipline:

```text
field semantics
→ ECMAScript

standard API behavior
→ standard documentation

engine optimization
→ engine-specific sources

initialization design
→ domain/application requirements
```

# 60. Completion Snapshot

```text
Chapter: 006
Title: Class Fields & Initialization Semantics

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

For a base class:

```text
class definition
      ↓
class elements established
      ↓
new
      ↓
instance created
      ↓
instance fields initialize
      ↓
constructor body
      ↓
ready instance
```

For a derived class:

```text
derived construction
      ↓
super()
      ↓
base initialization
      ↓
base constructor
      ↓
derived instance fields
      ↓
derived constructor body
      ↓
ready instance
```

For class-level state:

```text
class evaluation
      ↓
static fields / private static state
      ↓
class ready
```

Always ask:

```text
1. Which fields exist right now?
2. Which field initializers have run?
3. Is this a base or derived construction?
4. Has super() completed where required?
5. Can this method observe partially initialized state?
6. Which work happens per instance?
7. Which state is shared across all instances?
8. Which state is actually private?
9. Are field initializers doing too much?
10. What invariant must hold when construction finishes?
```

# Principal Design Principle

> **A well-designed object should have a deliberate initialization timeline: know exactly which state exists at every lifecycle step, and ensure no externally meaningful object escapes in an invalid state.**

# Track Mapping

```text
Track A — Core Theory
    class fields
    public/private state
    static state
    class evaluation
    initialization ordering
    base vs derived construction
    private branding

Track B — Implementation
    field/constructor translations
    private-state alternatives
    static registries
    initialization-order tooling
    lifecycle-safe domain objects

Track C — Interview / Reasoning
    field ordering
    super timing
    base/derived state
    constructor virtual dispatch
    per-instance function allocation
    initialization design judgment
```
