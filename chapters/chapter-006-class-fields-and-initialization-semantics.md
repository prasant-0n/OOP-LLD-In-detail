# Chapter 6 — Class Fields & Initialization Semantics

> **JavaScript OOP + LLD Mastery**
>
> This chapter explains **when, where, and how class state comes into existence**. Public fields, private fields, static fields, computed fields, initialization order, and inheritance all affect whether an object is valid and predictable.
>
> **Central question:** At every point during construction, which state is guaranteed to exist?

**Status:** `[ ] Not Started`

# 1. Learning Objectives

```text
[ ] explain public instance fields
[ ] explain public static fields
[ ] explain private instance fields
[ ] explain private static fields
[ ] explain field initializers
[ ] explain computed field names
[ ] explain class evaluation at a high level
[ ] trace base-class initialization
[ ] trace derived-class initialization
[ ] explain super() timing
[ ] explain field ordering
[ ] explain instance fields vs prototype methods
[ ] explain static fields vs instance fields
[ ] explain private-field brand initialization
[ ] identify partial-initialization hazards
[ ] reason about initialization invariants
[ ] compare field syntax with constructor assignment
[ ] compare prototype methods with field functions
[ ] debug initialization-order failures
[ ] design safe construction timelines
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

A class field declares state associated with either an instance or the class/static side.

```js
class User {
  name = "Unknown";
  active = true;
  static category = "user";
  #token = "private";
}
```

Conceptually:

```text
User instance
├── name
├── active
└── private state

User class
└── category
```

Fields are different from methods:

```text
field  → state
method → behavior
```

# 4. Why Does It Exist?

Constructor assignment can become repetitive:

```js
class User {
  constructor() {
    this.active = true;
    this.role = "user";
  }
}
```

Fields make defaults visible at the class definition:

```js
class User {
  active = true;
  role = "user";
}
```

They also provide syntax for:

```text
public instance state
private instance state
public static state
private static state
```

But fields do not remove initialization-order or lifecycle concerns.

# 5. Mental Model

Think in two phases.

```text
CLASS EVALUATION
       │
       ├── methods
       ├── private names
       ├── static elements
       ├── computed keys
       └── static initialization
                  │
                  ▼
             CLASS READY
                  │
                  ▼
        INSTANCE CONSTRUCTION
                  │
                  ├── base fields
                  ├── base constructor
                  ├── derived fields
                  └── derived constructor body
```

For a base class:

```text
instance creation
→ instance fields
→ constructor body
```

For a derived class:

```text
derived construction
→ super()
→ base construction
→ derived instance fields
→ derived constructor body
```

# 6. Core Rules

## Rule 1 — Instance Fields Are Per-Instance State

```js
class Counter {
  count = 0;
}

const a = new Counter();
const b = new Counter();

a.count = 1;

console.log(a.count);
console.log(b.count);
```

Prediction:

```text
1
0
```

## Rule 2 — Static Fields Are Class-Level State

```js
class Counter {
  static created = 0;
}
```

Access through:

```js
Counter.created;
```

not through:

```js
new Counter().created;
```

## Rule 3 — Public Instance Fields Are Own Properties

```js
class User {
  name = "Milan";
}

const user = new User();
console.log(Object.hasOwn(user, "name"));
```

Result:

```text
true
```

## Rule 4 — Prototype Methods Are Separate

```js
class User {
  name = "Milan";

  greet() {
    return this.name;
  }
}
```

Model:

```text
user
├── name
└── [[Prototype]]
      ↓
User.prototype
└── greet
```

## Rule 5 — Private Fields Are Not Ordinary Public Keys

```js
class Account {
  #balance = 0;
}
```

Private names are not string keys such as `"#balance"`.

## Rule 6 — Field Initializers Execute

```js
class Service {
  value = createValue();
}
```

The expression executes during relevant initialization and can:

```text
throw
allocate
call methods
read state
have side effects
```

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
  static #items = new Map();
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

## Computed Field Name

```js
const key = "displayName";

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
a.count starts at 0
b.count starts at 0

a.increment()
→ only a.count changes
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
  last = "Prusty";
  full = `${this.first} ${this.last}`;
}

console.log(new User().full);
```

**Prediction**

```text
Milan Prusty
```

# 9. Execution Walkthrough

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

Conceptually:

```text
1. Class definition is evaluated.
2. Class elements are established.
3. new starts construction.
4. The instance is created.
5. Base instance fields initialize.
6. Constructor body runs.
7. name changes from "Unknown" to "Milan".
8. Construction completes.
```

Final state:

```text
user.name   → "Milan"
user.active → true
```

# 10. Internal Mechanics

A field is an initialization step; a prototype method is normally shared behavior.

```js
class User {
  name = "Milan";
  greet() {}
}
```

Conceptual layout:

```text
instance
└── name

User.prototype
└── greet
```

For instance fields:

```text
each instance receives its own state
```

For prototype methods:

```text
instances delegate to shared behavior
```

# 11. ECMAScript / Specification Semantics

ECMAScript defines class semantics around:

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

Exact behavior must be reasoned from the language specification rather than from an imagined source-to-source transform.

Important distinction:

```text
language guarantee
≠
engine implementation detail
```

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

## 12.2 Field Initializers Can Read Earlier Fields

```js
class User {
  first = "Milan";
  last = "Prusty";
  full = `${this.first} ${this.last}`;
}
```

## 12.3 Field Initializers Can Call Methods

```js
class User {
  name = this.normalize("Milan");

  normalize(value) {
    return value.trim();
  }
}
```

This requires careful reasoning about lookup, receiver, and overriding.

## 12.4 Field Initializers Can Throw

```js
class Service {
  client = createClient();
}
```

If `createClient()` throws, construction fails.

# 13. Initialization Order

## Base Class

```text
create instance
→ initialize base instance fields
→ run base constructor body
```

## Derived Class

```text
derived constructor begins
→ super(...)
→ base construction
→ derived instance fields
→ derived constructor body
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

This is instance-state initialization, not prototype method overriding.

# 16. Fields Are Not Virtual Methods

```js
value = 10;
```

stores state.

```js
getValue() {
  return 10;
}
```

expresses behavior.

This distinction matters for polymorphism and invariants.

# 17. Constructor and Field Interaction

```js
class Base {
  value = 10;

  constructor() {
    console.log(this.value);
  }
}

new Base();
```

A base constructor can observe its initialized base field.

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

> **A base constructor must not assume derived instance fields already exist.**

# 19. Constructor Virtual Dispatch Hazard

```js
class Base {
  constructor() {
    this.initialize();
  }

  initialize() {}
}

class Child extends Base {
  value = 100;

  initialize() {
    console.log(this.value);
  }
}

new Child();
```

The base constructor invokes child behavior before derived fields have initialized.

Design rule:

> Avoid calling overridable behavior from constructors unless partial initialization is explicitly safe.

# 20. Field Initializers and `this`

```js
class User {
  name = "Milan";
  greeting = `Hello ${this.name}`;
}
```

Later initializers can use earlier instance state.

# 21. Private Field Initialization

```js
class Account {
  #balance = 100;

  getBalance() {
    return this.#balance;
  }
}
```

The private state is initialized as part of the construction lifecycle.

# 22. Private Brand Reasoning

Useful mental model:

```text
private declaration
      ↓
private name/brand
      ↓
instance initialization
      ↓
instance has compatible private state
```

Wrong receivers can cause private-field access to throw.

# 23. Private Fields and Inheritance

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

Inherited behavior can use Base private state on a compatible instance, but the private name is not a normal public property available to Child.

# 24. Static Fields and Initialization

```js
class Config {
  static version = 1;
  static label = `v${Config.version}`;
}
```

Static initialization order matters because later static initializers can depend on earlier state.

# 25. Static Private State

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

Treat static mutable state as shared state with lifecycle and testing implications.

# 26. Computed Fields

```js
const key = "displayName";

class User {
  [key] = "Milan";
}
```

Computed key expressions participate in class evaluation and can have dependencies or fail.

# 27. Field Initializers vs Constructor Assignment

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

# 28. Default State vs Derived State

Prefer computed state when duplication would drift:

```js
class User {
  firstName = "";
  lastName = "";

  get displayName() {
    return `${this.firstName} ${this.lastName}`.trim();
  }
}
```

# 29. Fields and Invariants

Fields declare state; constructors can establish invariants.

```js
class Money {
  amount;
  currency;

  constructor(amount, currency) {
    if (!Number.isFinite(amount)) {
      throw new TypeError("Invalid amount");
    }

    if (typeof currency !== "string" || currency.length !== 3) {
      throw new TypeError("Invalid currency");
    }

    this.amount = amount;
    this.currency = currency.toUpperCase();
  }
}
```

The object should satisfy the domain contract when construction completes.

# 30. Advanced Behavior — Per-Instance Functions

Compare:

```js
class User {
  greet() {
    return this.name;
  }
}
```

with:

```js
class User {
  greet = () => this.name;
}
```

The first is normally a shared prototype method. The second stores a function per instance.

Trade-off:

```text
prototype method:
+ shared function
+ traditional override model

arrow field:
+ lexical this
- per-instance function allocation
```

# 31. Edge Cases

Important cases:

```text
later field referenced by earlier initializer
derived construction before derived fields exist
base constructor reading derived state
private access with unrelated receiver
static initializer throwing
computed-key failure
large field allocations
field functions created per instance
constructor calling overridden behavior
```

# 32. Common Misconceptions

```text
"All class fields live on the prototype."
"Static fields exist on every instance."
"Private fields are hidden string properties."
"Field initialization has no ordering semantics."
"Derived fields exist before super()."
"Arrow-function fields are prototype methods."
"Fields are free allocations."
"Fields automatically enforce domain invariants."
"Private means encrypted."
```

# 33. Common Mistakes

```text
[ ] accidental field-order dependencies
[ ] expensive work in field initializers
[ ] unnecessary per-instance arrow functions
[ ] base constructors assuming derived state
[ ] constructor-time virtual dispatch
[ ] static mutable state acting as hidden global state
[ ] duplicated derived state
[ ] defaults used where validation is required
[ ] I/O hidden inside construction
```

# 34. Comparison With Related Concepts

| Concept | Meaning |
|---|---|
| Instance field | Per-instance state |
| Static field | Class-level state |
| Private field | Per-instance private state |
| Static private field | Class-level private state |
| Prototype method | Shared behavior |
| Arrow-function field | Per-instance function value |
| Constructor assignment | Explicit initialization |
| Getter | Computed access behavior |
| Factory | Externalized creation/lifecycle |
| Static registry | Shared class-level state |

# 35. Performance Considerations

Watch:

```text
per-instance allocations
large default arrays/maps
field functions
expensive initializer expressions
duplicate derived state
static caches
```

A function-valued field can allocate once per instance. A prototype method is normally shared.

Profile before optimizing.

# 36. Memory Considerations

Potential memory costs:

```text
large instance graphs
per-instance closures
private state
static registries
per-instance caches
duplicate derived state
```

A static collection may remain reachable for a long time.

# 37. Security Considerations

Public fields are public API surface.

Private fields improve encapsulation but do not replace:

```text
authorization
input validation
secret management
process isolation
cryptography
```

Static shared state can also accidentally cross request/tenant boundaries.

# 38. Production Usage

Prefer:

```text
explicit dependencies
small/predictable initialization
validated state
clear ownership
limited side effects
```

Be cautious with:

```text
database connections in fields
network requests in fields
large allocations in defaults
virtual calls during construction
static global-like state
```

Complex asynchronous initialization may be clearer in a factory or application lifecycle.

# 39. Implementation From Scratch

## Exercise 1 — Field-to-Constructor Translation

Translate:

```js
class User {
  name = "Unknown";
  active = true;
}
```

into constructor assignments and note what the translation does not capture exactly.

## Exercise 2 — Field Dependency Detector

Identify fields whose initializers reference fields declared later.

## Exercise 3 — Private-State Alternative

Create a closure-based alternative to:

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

Compare privacy, memory, prototype sharing, and inheritance.

## Exercise 4 — Static Registry

Implement a private static `Map` with:

```text
add
get
remove
clear
```

Then analyze lifetime and test coupling.

## Exercise 5 — Initialization-Safe Order

Design an `Order` with:

```text
id
items
status
createdAt
private totals
derived display state
```

and document the initialization order.

# 40. Debugging Exercises

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

## Debug 2 — Base Reads Derived State

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

Explain the result.

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

Find the lifecycle bug.

## Debug 4 — Per-Instance Function

```js
class User {
  handler = () => {};
}

const a = new User();
const b = new User();

console.log(a.handler === b.handler);
```

Explain the result.

## Debug 5 — Shared Static State

```js
class Cache {
  static items = new Map();
}

Cache.items.set("x", 1);

console.log(Cache.items.get("x"));
```

Explain ownership and lifetime.

# 41. Code Review Exercise

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
1. Which state should be injected?
2. Which initialization is hidden?
3. What can throw?
4. Is construction still cheap?
5. Who owns database?
6. Who owns cache?
7. Is load asynchronous?
8. What invariant exists when construction returns?
9. Would a factory/startup workflow be clearer?
```

# 42. Interview Questions

```text
1. What is a class field?
2. Are public instance fields own properties?
3. What is a static field?
4. What is a private field?
5. When are instance fields initialized?
6. Why does field order matter?
7. What is different about base and derived initialization?
8. Why does super() matter?
9. Can a base constructor safely read derived fields?
10. Why is calling overridable methods from constructors risky?
11. Where do prototype methods live?
12. How are arrow-function fields different from prototype methods?
13. What is private-field branding?
14. How are static fields different from instance fields?
15. Why can static mutable state behave like global state?
16. Should field initializers perform I/O?
17. When should state be a constructor parameter instead?
18. How would you design a reliable initialization timeline?
```

# 43. Predict-the-Output Exercises

## A

```js
class User {
  first = "Milan";
  last = "Prusty";
  full = `${this.first} ${this.last}`;
}

console.log(new User().full);
```

## B

```js
class User {
  full = this.first;
  first = "Milan";
}

console.log(new User().full);
```

## C

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

## D

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

## E

```js
class Config {
  static version = 1;
  static label = `v${Config.version}`;
}

console.log(Config.label);
```

## F

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

# 44. Mastery Exercises

## Level 1 — Understand

Explain:

```text
instance field
static field
private field
static private field
field initializer
```

## Level 2 — Explain

Draw initialization for Base and Child, marking base fields, base constructor, derived fields, and derived constructor.

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
field-order tests
```

## Level 5 — Debug

Fix:

```text
late field dependency
base reads derived state
constructor virtual dispatch
per-instance function duplication
hidden static state
```

## Level 6 — Apply

Design an `Order` using required inputs, defaults, validated fields, derived state, and private totals.

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

> Why is initialization order a design concern rather than merely a language detail?

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

# 45. Key Takeaways

```text
1. Instance fields create per-instance state.
2. Static fields create class-level state.
3. Private fields use private-name semantics.
4. Public instance fields are separate from prototype methods.
5. Field initializers execute as part of initialization.
6. Field order can affect observable behavior.
7. Base and derived initialization are different.
8. Derived constructors have a super()/this boundary.
9. Base constructors cannot assume derived fields are initialized.
10. Constructor-time virtual dispatch can expose partial state.
11. Arrow-function fields are per-instance functions.
12. Static mutable state can behave like hidden global state.
13. Private state improves encapsulation but is not security isolation.
14. Constructors and fields complement each other.
15. Initialization should be designed around invariants and lifecycle.
16. Complex initialization may belong in factories/workflows.
```

# 46. Concept Connections

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
cohesion
coupling
GRASP
SOLID
TypeScript OOP
dependency injection
domain invariants
```

## Related Concepts

```text
constructors
factories
builders
dependency injection
resource lifecycle
immutability
encapsulation
polymorphism
```

## Why This Chapter Matters

The rest of OOP assumes objects can be trusted after construction. Initialization semantics determine whether state, dependencies, private data, and invariants exist when expected.

# 47. Completion Criteria

Mark:

```text
[+] Completed
```

when you can:

```text
explain every field kind
trace base initialization
trace derived initialization
explain field ordering
explain private state initialization
debug constructor/field interaction
```

Mark:

```text
[*] Mastered
```

when you can inspect unfamiliar classes and determine:

```text
what state exists at each construction step
which methods are safe to call
which invariants are temporarily absent
which side effects can occur
```

without executing the code.

Reading alone does not mark mastery.

# 48. Revision / Retrieval Record

```md
# Chapter 6 — Revision / Retrieval Record

## Attempt
- Date:
- Duration:
- Status before:
- Status after:

## Fields
- Instance:
- Static:
- Private:
- Static private:

## Initialization
- Base order:
- Derived order:
- Why is super important?

## Ordering
- Which initializers depend on earlier fields?
- Did declaration order matter?

## Inheritance
- Can Base read derived state?
- What can virtual dispatch expose?

## Performance
- Which state allocates per instance?
- Are any functions unnecessarily duplicated?

## Design
- What belongs in a field?
- What belongs in a constructor?
- What belongs in a factory/workflow?

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

# 49. Canonical References and Source Discipline

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
language semantics
→ ECMAScript

standard API behavior
→ standard documentation

engine optimization
→ engine-specific sources

initialization design
→ domain/application requirements
```

# 50. Completion Snapshot

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
class evaluation
      ↓
new
      ↓
instance created
      ↓
base fields initialize
      ↓
base constructor body
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
derived fields
      ↓
derived constructor body
      ↓
ready object
```

For class-level state:

```text
class evaluation
      ↓
static fields/private static state
      ↓
class ready
```

Always ask:

```text
1. Which fields exist right now?
2. Which initializers have run?
3. Is this base or derived construction?
4. Has the super boundary been crossed?
5. Can a method observe partial state?
6. Which work happens per instance?
7. Which state is shared?
8. Which state is private?
9. Are field initializers doing too much?
10. What invariant must hold when construction completes?
```

# Principal Design Principle

> **Design the initialization timeline deliberately: an object should become valid in a predictable sequence, and externally meaningful code should not have to guess which state exists.**

# Track Mapping

```text
Track A — Core Theory
    class fields
    private/static state
    class evaluation
    field ordering
    base/derived initialization
    private branding

Track B — Implementation
    field-to-constructor translation
    closure private state
    static registries
    initialization analysis
    lifecycle-safe objects

Track C — Interview / Reasoning
    field ordering
    super timing
    base/derived reasoning
    constructor virtual dispatch
    per-instance allocation
    initialization trade-offs
```
