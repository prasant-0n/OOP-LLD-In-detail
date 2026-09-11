# Chapter 1 — JavaScript Object Model — Foundation

> **JavaScript OOP + LLD Mastery**
>
> This chapter establishes the runtime mental model for objects before introducing classes, inheritance, SOLID, design patterns, or LLD. The goal is to understand what a JavaScript object actually is, how identity and state work, how properties are represented, how behavior is attached, and how object relationships become the foundation for object-oriented design.

**Status:** `[ ] Not Started`

---

# 1. Learning Objectives

By the end of this chapter, you should be able to:

```text
[ ] define a JavaScript object precisely
[ ] distinguish object identity from object value
[ ] distinguish primitive values from objects
[ ] explain reference semantics and aliasing
[ ] explain mutable state and ownership
[ ] explain own and inherited properties
[ ] explain string and symbol property keys
[ ] explain property descriptors
[ ] distinguish data and accessor properties
[ ] explain getters and setters conceptually
[ ] explain [[Prototype]] and prototype delegation
[ ] distinguish [[Prototype]] from a function's prototype property
[ ] predict object-operation results before execution
[ ] trace property lookup conceptually
[ ] identify object-model mistakes and security risks
[ ] connect object semantics to OOP and LLD design
```

---

# 2. Prerequisites

You should already understand:

```text
variables
primitive values
assignment
functions
scope
basic JavaScript syntax
arrays
control flow
```

---

# 3. What Is It?

A JavaScript object is an identity-bearing value with properties and object semantics. A useful model is:

```text
OBJECT
│
├── identity
├── own properties
│    ├── data properties
│    └── accessor properties
└── [[Prototype]] relationship
```

Objects can represent:

```text
state
behavior
entities
records
namespaces
collections
resources
configuration
domain concepts
```

An object is not automatically a classical-OOP instance. JavaScript's deeper object model is prototype-based, while `class` provides a higher-level language construct for defining constructors, methods, fields, and inheritance.

---

# 4. Why Does It Exist?

Objects let related state and behavior travel together.

```js
const account = {
  owner: "Milan",
  balance: 1000,

  deposit(amount) {
    this.balance += amount;
  },
};
```

Here:

```text
state    → owner, balance
behavior → deposit
```

This creates a unit of identity and behavior that other code can interact with. Once state is shared between references, however, ownership, mutation, invariants, and coupling become design concerns.

---

# 5. Mental Model

```text
                 OBJECT IDENTITY
                       │
                       ▼
                ┌─────────────┐
                │   Object    │
                ├─────────────┤
                │ own state   │
                │ own methods │
                │ accessors   │
                └──────┬──────┘
                       │
                 [[Prototype]]
                       │
                       ▼
                Prototype Object
                       │
                       ▼
                  next prototype
                       │
                       ▼
                     null
```

Keep this distinction explicit:

```text
prototype chain ≠ class hierarchy
```

---

# 6. Core Rules

## Rule 1 — Objects Have Identity

```js
const a = { name: "A" };
const b = { name: "A" };

console.log(a === b); // false
```

The structures look alike, but they are different object identities.

## Rule 2 — Object Assignment Shares the Object

```js
const a = { count: 1 };
const b = a;

b.count = 2;

console.log(a.count); // 2
```

Conceptually:

```text
a ──────┐
        ├──→ same object
b ──────┘
```

## Rule 3 — Shallow Copy Is Not Deep Copy

```js
const a = { profile: { name: "Milan" } };
const b = { ...a };

b.profile.name = "Changed";

console.log(a.profile.name); // Changed
```

The outer object differs; the nested object is shared.

## Rule 4 — Property Lookup Can Cross a Prototype Boundary

If an own property is absent, lookup can continue through `[[Prototype]]`.

## Rule 5 — `[[Prototype]]` and `prototype` Are Different

`[[Prototype]]` is an internal object relationship. `prototype` is an ordinary property commonly present on constructor-like functions and used by construction/inheritance mechanisms.

---

# 7. Syntax

## Object Literal

```js
const user = {
  id: 1,
  name: "Milan",
  active: true,
};
```

## Property Access

```js
user.name;
user["name"];
```

## Computed Key

```js
const key = "name";
const user = { [key]: "Milan" };
```

## Method

```js
const user = {
  name: "Milan",
  greet() {
    return `Hello ${this.name}`;
  },
};
```

## Symbol Key

```js
const internalKey = Symbol("internal");
const user = { [internalKey]: 42 };
```

## Null-Prototype Object

```js
const dictionary = Object.create(null);
dictionary.name = "Milan";
```

---

# 8. Basic Examples

## Example — Identity

```js
const a = { id: 1 };
const b = { id: 1 };
const c = a;

console.log(a === b); // false
console.log(a === c); // true
```

## Example — Aliasing

```js
const account = { balance: 1000 };
const reference = account;

reference.balance -= 100;

console.log(account.balance); // 900
```

## Example — Own vs Inherited

```js
const parent = { role: "admin" };
const child = Object.create(parent);
child.name = "Milan";

console.log(Object.hasOwn(child, "name")); // true
console.log(Object.hasOwn(child, "role")); // false
console.log("role" in child);              // true
```

## Example — Accessor

```js
const user = {
  firstName: "Milan",

  get displayName() {
    return this.firstName;
  },

  set displayName(value) {
    this.firstName = value.trim();
  },
};
```

---

# 9. Execution Walkthrough

Consider:

```js
const user = { name: "Milan" };
const other = user;
other.name = "Prasant";
console.log(user.name);
```

Trace:

```text
1. Create Object#1
2. user → Object#1
3. other → Object#1
4. other.name changes Object#1.name
5. user.name reads Object#1.name
6. result → "Prasant"
```

The mutation is visible through both bindings because there is only one object identity.

---

# 10. Internal Mechanics

At the ECMAScript semantic level, object operations are described through internal methods and slots. Important concepts include:

```text
[[Get]]
[[Set]]
[[HasProperty]]
[[Delete]]
[[OwnPropertyKeys]]
[[DefineOwnProperty]]
[[GetPrototypeOf]]
[[SetPrototypeOf]]
[[Prototype]]
[[Extensible]]
```

Normal syntax triggers these semantics rather than exposing the internal methods directly.

For example:

```js
user.name
```

conceptually performs the language's property-get behavior, while:

```js
user.name = value;
```

performs property-set behavior.

---

# 11. ECMAScript / Specification Semantics

The ECMAScript object model is more precise than the informal idea that objects are merely hash maps.

Objects involve:

```text
identity
internal methods
internal slots
property descriptors
prototype relationships
extensibility state
```

Some objects have specialized/exotic semantics rather than behaving exactly like ordinary objects. Arrays, proxies, module namespace objects, typed-array-related objects, and other built-ins illustrate that the term "object" covers several semantic categories.

The important engineering discipline is to distinguish:

```text
language guarantee
vs
engine implementation
vs
host/runtime behavior
```

---

# 12. Advanced Behavior

Advanced object behavior includes:

```text
prototype delegation
property descriptor effects
accessor execution
shadowing
extensibility rules
object identity and aliasing
specialized object behavior
```

The deeper question is never just "what value is stored?". Ask:

```text
Who owns it?
Who can observe it?
Who can mutate it?
Where is behavior found?
Which invariants depend on it?
```

---

# 13. Edge Cases

Important edge cases include:

```text
null
symbol keys
non-enumerable properties
null-prototype objects
shallow freezing
prototype shadowing
callable objects
array-specific semantics
accessors with side effects
prototype pollution
```

### `typeof null`

```js
typeof null; // "object"
```

This is a historic JavaScript quirk, not a complete taxonomy of objects.

### Functions Are Objects

Functions are callable objects and can have properties:

```js
function greet() {}
greet.version = 1;
```

### Arrays Are Objects

Arrays participate in the object model but have specialized array semantics around indices and `length`.

---

# 14. Common Misconceptions

### "JavaScript passes objects by reference."

More precise: JavaScript passes argument values. For an object, the value provides access to the object identity.

### "Spread deep-clones objects."

False. Spread is shallow for nested object values.

### "prototype is the object's prototype."

Not necessarily. The object's internal relationship is `[[Prototype]]`.

### "Everything is an object."

JavaScript also has primitive values.

### "Object.freeze makes the whole graph immutable."

False. It is shallow.

### "Same properties means equal objects."

Ordinary object `===` compares identity, not structural equality.

---

# 15. Common Mistakes

```text
[ ] confusing identity with structural equality
[ ] accidentally sharing mutable state
[ ] assuming spread deep-clones
[ ] confusing prototype with [[Prototype]]
[ ] using `in` when own-property checks are needed
[ ] forgetting null in object checks
[ ] treating freeze as deep immutability
[ ] modifying Object.prototype carelessly
[ ] treating every object as a dictionary
[ ] exposing mutable internal collections
[ ] creating classes before establishing ownership
```

---

# 16. Comparison With Related Concepts

| Concept | Core idea |
|---|---|
| Primitive | Value with primitive semantics |
| Object | Identity-bearing structured value |
| Reference | Means of reaching an object value |
| Own property | Property directly belonging to an object |
| Inherited property | Property discovered through prototype lookup |
| `[[Prototype]]` | Internal prototype relationship |
| `prototype` property | Property used by constructor/class mechanisms |
| Class | Higher-level construction/inheritance mechanism |
| Module | Encapsulation and dependency boundary |
| Entity | Domain concept defined by identity |
| Value Object | Domain concept defined by value/equality |

---

# 17. Performance Considerations

Object design can influence:

```text
allocation
property access
object shapes
prototype lookup
polymorphic call sites
copying
serialization
GC pressure
```

Modern engines use implementation techniques such as hidden classes/shapes and inline caches, but those are engine-specific details rather than ECMAScript guarantees.

Engineering rule:

```text
Correctness
→ Measure
→ Profile
→ Optimize
```

Do not optimize object structure from assumptions alone.

---

# 18. Memory Considerations

Memory is affected by:

```text
object count
object graphs
retained references
closures
arrays/maps/sets
caches
event listeners
nested data
```

A reachable object cannot be garbage-collected merely because no local variable currently appears to use it. Ownership and lifetime therefore matter in production design.

---

# 19. Security Considerations

Object design intersects security through:

```text
prototype pollution
untrusted keys
unsafe object merging
deserialization
mutable shared state
property enumeration
```

For example, blindly merging untrusted input into security-sensitive objects can create unexpected behavior. Later chapters will treat prototype pollution and trust boundaries in depth.

---

# 20. Production Usage

Before exposing an object from a module or API, ask:

```text
Who owns its state?
Who may mutate it?
Can callers retain aliases?
What is public?
What is internal?
What invariants must always hold?
What lifecycle does it have?
```

Good object boundaries reduce accidental coupling and make later testing, refactoring, and LLD easier.

---

# 21. Implementation From Scratch

## Exercise 1 — Identity Inspector

Implement:

```js
function sameObject(a, b) {
  return a === b;
}
```

Extend the exercise to explain why structural equality is a separate domain operation.

## Exercise 2 — Own Property Inspector

Implement:

```js
function hasOwnKey(object, key) {
  return Object.hasOwn(object, key);
}
```

Test own, inherited, missing, and symbol keys.

## Exercise 3 — Prototype Chain Inspector

Return the sequence of prototypes from an object until `null`, without mutating the input.

## Exercise 4 — Shallow Clone

Implement a shallow clone and prove with a nested object that the nested reference remains shared.

## Exercise 5 — Encapsulated Collection

Design a `Cart` with `addItem`, `removeItem`, `hasItem`, and `getItems`. Decide deliberately whether `getItems` returns an alias, a copy, or another controlled representation.

## Exercise 6 — Descriptor Inspector

Print each own property's descriptor and classify it as a data or accessor property.

---

# 22. Debugging Exercises

## Debug 1 — Shared Defaults

```js
const defaults = { filters: [] };

function createConfig() {
  return defaults;
}

const a = createConfig();
const b = createConfig();
a.filters.push("gold");

console.log(b.filters);
```

Predict, trace the alias, and fix the ownership problem.

## Debug 2 — Prototype Lookup

```js
const parent = { status: "parent" };
const child = Object.create(parent);

console.log(Object.hasOwn(child, "status"));
console.log("status" in child);
```

Explain why the answers differ.

## Debug 3 — Shallow Copy

```js
const source = { settings: { theme: "dark" } };
const copy = { ...source };
copy.settings.theme = "light";

console.log(source.settings.theme);
```

Identify the shared nested identity.

## Debug 4 — Freeze Boundary

```js
const state = { user: { name: "Milan" } };
Object.freeze(state);
state.user.name = "Changed";
console.log(state.user.name);
```

Explain the shallow boundary.

---

# 23. Code Review Exercise

Review:

```js
class User {
  constructor(profile) {
    this.profile = profile;
  }

  getProfile() {
    return this.profile;
  }
}
```

Discuss:

```text
Who owns profile?
Can callers mutate internal state?
Should profile be copied?
Should profile be immutable?
What contract does getProfile expose?
What are the performance costs of the alternatives?
```

Do not assume copying is always the correct answer; defend the boundary based on requirements.

---

# 24. Interview Questions

```text
1. What is a JavaScript object?
2. What is object identity?
3. Why are identical object literals not equal?
4. What is aliasing?
5. What is the difference between own and inherited properties?
6. What is [[Prototype]]?
7. What is the prototype chain?
8. How is prototype different from [[Prototype]]?
9. What is a property descriptor?
10. What is the difference between data and accessor properties?
11. Why is Object.freeze shallow?
12. Why are functions objects?
13. Why are arrays objects?
14. Why does typeof null return object?
15. Why might Object.create(null) be useful?
16. How can shared mutable references create bugs?
17. How would you protect an object's internal collection?
18. When would you choose composition instead of inheritance?
```

---

# 25. Predict-the-Output Exercises

## A

```js
const a = {};
const b = {};
const c = a;

console.log(a === b);
console.log(a === c);
```

Predict first.

## B

```js
const parent = { value: 10 };
const child = Object.create(parent);

console.log(child.value);
child.value = 20;
console.log(child.value);
console.log(parent.value);
```

## C

```js
const a = { nested: { value: 1 } };
const b = { ...a };
b.nested.value = 2;

console.log(a.nested.value);
console.log(a.nested === b.nested);
```

## D

```js
const obj = {};
Object.defineProperty(obj, "x", {
  value: 10,
  enumerable: false,
});

console.log(obj.x);
console.log(Object.keys(obj));
console.log(Object.hasOwn(obj, "x"));
```

## E

```js
const parent = {
  greet() {
    return this.name;
  },
};

const child = Object.create(parent);
child.name = "Milan";

console.log(child.greet());
```

For every exercise, record:

```text
Prediction
Actual Result
Trace
Why
Rule
```

---

# 26. Mastery Exercises

## Understand

Explain object, identity, property, prototype, and aliasing without notes.

## Explain

Explain `prototype` vs `[[Prototype]]` with a diagram.

## Predict

Predict identity, aliasing, shadowing, descriptor, and shallow-copy behavior before execution.

## Implement

Build an own-property checker, prototype inspector, descriptor inspector, shallow clone, and encapsulated collection.

## Debug

Fix a shared-state bug while explicitly stating the ownership model.

## Apply

Design a `ShoppingCart` whose state cannot be arbitrarily replaced by callers.

## Compare

Defend:

```text
public array
vs
copied array
vs
controlled read-only representation
vs
method-based mutation
```

## Defend

Answer:

> Why must JavaScript's object model be understood before learning classes and design patterns?

Your answer must connect:

```text
identity
state
ownership
prototype
encapsulation
composition
LLD.
```

---

# 27. Key Takeaways

```text
1. Objects have identity.
2. Multiple bindings can refer to one object.
3. Aliasing creates shared mutable state.
4. Property access includes more semantics than dictionary lookup.
5. Own and inherited properties are distinct.
6. Prototype delegation is fundamental to the object model.
7. [[Prototype]] is different from a function's prototype property.
8. Data and accessor properties have different semantics.
9. Property descriptors matter for API design.
10. freeze/seal/preventExtensions are object-boundary tools, not universal deep immutability.
11. Arrays and functions are specialized object forms.
12. OOP is a design paradigm; the object model is a language/runtime mechanism.
13. Good object design begins with ownership, invariants, and responsibility—not class syntax.
```

---

# 28. Concept Connections

## Depends On

```text
values
variables
assignment
functions
property access
```

## Builds Toward

```text
Chapter 2 — Object Identity, Equality & Mutability
Chapter 3 — Prototype Chain Deep Dive
Chapter 4 — Constructor Functions
Chapter 5 — JavaScript Classes Internally
Chapter 8 — Encapsulation
Chapter 10 — Polymorphism
Chapter 11 — Composition
Responsibility / GRASP
TypeScript Object Design
Domain Modeling
LLD Decomposition
```

## Related Concepts

```text
references
memory
prototype chain
encapsulation
identity
value objects
entities
composition
ownership
```

## Why This Chapter Matters

If the object model is misunderstood, later OOP concepts become memorized syntax. If the object model is understood, classes, composition, inheritance, encapsulation, and LLD can be reasoned about from first principles.

---

# 29. Completion Criteria

Mark `[~] In Progress` when you can follow examples but still need reference material.

Mark `[+] Completed` when you can:

```text
define the object model
trace identity and aliasing
explain own vs inherited properties
explain [[Prototype]]
explain property descriptors
implement the exercises
```

Mark `[*] Mastered` only when you can inspect unfamiliar JavaScript object code, predict its behavior, identify ownership/aliasing problems, and defend the object boundary without relying on memorized definitions.

Reading alone does not mark mastery.

---

# 30. Revision / Retrieval Record

```md
# Chapter 1 — Revision / Retrieval Record

## Attempt
- Date:
- Duration:
- Status before:
- Status after:

## Object Model
- What is an object?
- What gives it identity?
- What are own properties?
- What are inherited properties?

## Prototype
- What is [[Prototype]]?
- What is prototype?
- How does lookup work?

## References / Aliasing
- What surprised me?
- Where did I observe shared mutation?

## Property Descriptors
- Data property:
- Accessor property:
- Writable:
- Enumerable:
- Configurable:

## Weak Areas
-

## Debugging Mistakes
-

## Interview Questions Missed
-

## Prediction Mistakes
-

## Design Insights
-

## Next Review
-
```

---

# 31. Canonical References and Source Discipline

Use authoritative sources in this order:

```text
1. ECMAScript specification
2. Official JavaScript runtime documentation
3. Engine documentation/source for implementation details
4. Project/application source when studying a specific system
```

Always distinguish:

```text
ECMAScript semantic guarantees
vs
engine optimizations
vs
host/runtime behavior.
```

A V8-specific optimization must never be presented as a universal JavaScript guarantee.

---

# 32. Completion Snapshot

```text
Chapter: 001
Title: JavaScript Object Model — Foundation

Track A — Core Theory       [ ]
Track B — Implementation    [ ]
Track C — Interview/Reasoning [ ]

Overall: [ ] Not Started
```

---

# Track Mapping

```text
Track A — Core Theory
Object semantics, identity, properties, descriptors, prototypes.

Track B — Implementation
Identity inspector, property checker, prototype inspector, cloning, encapsulation.

Track C — Interview / Reasoning
Prediction, debugging, ownership, API boundary, design defense.
```

---

# Final Mental Model

```text
OBJECT
│
├── IDENTITY
│
├── OWN PROPERTIES
│      ├── DATA
│      └── ACCESSOR
│
├── [[PROTOTYPE]]
│      ↓
│   PROTOTYPE
│      ↓
│   PROTOTYPE
│      ↓
│     null
│
└── BEHAVIOR
       ↓
  methods / getters / setters
```

Before designing a class, ask:

```text
1. What state does this object own?
2. What behavior does it own?
3. Who else can reference it?
4. Where does property lookup go?
5. What invariants must remain true?
```

These questions are the starting point for serious JavaScript OOP and LLD.

---

# Principal Design Principle

> **Before deciding what classes to create, understand what objects are, what state they own, how behavior is found, who can mutate them, and how they collaborate.**
