# Chapter 3 — Prototype Chain Deep Dive

> **JavaScript OOP + LLD Mastery**
>
> This chapter explains the prototype system at runtime depth. The goal is to understand how JavaScript finds properties, how objects delegate behavior, why shadowing happens, how constructor functions connect to prototypes, how classes participate in the same model, and how prototype decisions affect inheritance, composition, performance, and security.

**Status:** `[ ] Not Started`

---

# 1. Learning Objectives

By the end of this chapter, you should be able to:

```text
[ ] define [[Prototype]]
[ ] explain prototype delegation
[ ] trace property lookup through a prototype chain
[ ] distinguish own properties from inherited properties
[ ] explain shadowing
[ ] explain method lookup
[ ] explain why this is determined by call-site mechanics
[ ] distinguish [[Prototype]] from prototype
[ ] use Object.getPrototypeOf
[ ] use Object.setPrototypeOf safely and appropriately
[ ] use Object.create
[ ] inspect a prototype chain
[ ] explain Object.prototype
[ ] explain null-prototype objects
[ ] explain constructor functions
[ ] explain the prototype property of constructors
[ ] explain new at a conceptual level
[ ] connect new to prototype assignment
[ ] explain class prototype behavior
[ ] explain instance methods vs static methods
[ ] reason about prototype mutation
[ ] identify prototype-chain security issues
[ ] identify prototype-chain performance considerations
[ ] compare inheritance and delegation
[ ] debug unexpected inherited behavior
[ ] implement a prototype-based object model
[ ] design prototype use deliberately
```

---

# 2. Prerequisites

Required:

```text
Chapter 1 — JavaScript Object Model — Foundation
Chapter 2 — Object Identity, Equality & Mutability
objects
references
properties
property descriptors
functions
identity
mutability
```

---

# 3. What Is It?

A JavaScript object can have an internal prototype relationship:

```text
object.[[Prototype]]
```

That relationship points to another object or:

```text
null
```

A property lookup can therefore proceed through:

```text
object
  ↓
prototype
  ↓
prototype
  ↓
null
```

This is the prototype chain.

The critical idea is:

> **JavaScript object inheritance is fundamentally delegation through a chain of objects.**

---

# 4. Why Does It Exist?

Prototype delegation enables:

```text
shared behavior
inheritance
method reuse
fallback property lookup
constructor-based instance behavior
class-based inheritance.
```

Instead of putting a separate copy of a method on every object:

```text
Object A → greet function #1
Object B → greet function #2
Object C → greet function #3
```

a prototype can hold the common method:

```text
Object A ──┐
Object B ──┼──→ Prototype
Object C ──┘        │
                    └── greet()
```

This creates shared behavior without requiring a classical class object underneath every instance.

---

# 5. Mental Model

Think of property lookup as a walk:

```text
                 READ obj.x
                     │
                     ▼
              Does obj own "x"?
                 /        \
               yes         no
                │           │
                ▼           ▼
             return     get [[Prototype]]
             value           │
                             ▼
                         prototype
                             │
                       repeat lookup
                             │
                             ▼
                           null
                             │
                             ▼
                        return undefined
```

This model is more important than memorizing syntax.

---

# 6. Core Rules

## Rule 1 — Every Ordinary Object Has a `[[Prototype]]` Relationship

Conceptually:

```text
Object #1
   │
   └── [[Prototype]] → Object #2
```

The prototype can eventually be:

```text
null
```

---

## Rule 2 — Property Lookup Starts at the Receiver

```js
const parent = {
  value: 10,
};

const child = Object.create(parent);

child.value = 20;

console.log(child.value);
```

Lookup finds:

```text
child.value
```

before:

```text
parent.value
```

---

## Rule 3 — Own Properties Shadow Prototype Properties

```js
const parent = {
  role: "admin",
};

const child = Object.create(parent);

child.role = "user";

console.log(child.role);
```

Result:

```text
"user"
```

The own property shadows the inherited property.

---

## Rule 4 — Missing Properties Can Be Inherited

```js
const parent = {
  greet() {
    return "hello";
  },
};

const child = Object.create(parent);

console.log(child.greet());
```

`greet` is found on the prototype.

---

## Rule 5 — Prototype Lookup Does Not Clone the Property

If:

```text
child.greet
```

finds a function stored on the prototype, the function remains the same function object.

It is not copied onto `child` merely because it was accessed.

---

## Rule 6 — The Receiver and the Property Owner Can Differ

```js
const parent = {
  name: "Parent",

  greet() {
    return this.name;
  },
};

const child = Object.create(parent);

child.name = "Child";

console.log(child.greet());
```

The function is owned by:

```text
parent
```

but the call receiver is:

```text
child
```

Therefore:

```text
this === child
```

for this call.

---

# 7. Syntax

## `Object.create`

```js
const child = Object.create(parent);
```

---

## Read Prototype

```js
Object.getPrototypeOf(object);
```

---

## Set Prototype

```js
Object.setPrototypeOf(object, prototype);
```

Use runtime prototype mutation carefully.

---

## Prototype Property

```js
function User() {}

console.log(User.prototype);
```

---

## Constructor Access

For many ordinary constructor-created objects:

```js
instance.constructor
```

may resolve through the prototype chain.

Do not assume it is always an own property or always a reliable security boundary.

---

# 8. Basic Examples

## Example 1 — Direct Delegation

```js
const animal = {
  speak() {
    return "sound";
  },
};

const dog = Object.create(animal);

console.log(dog.speak());
```

Prediction:

```text
"sound"
```

Trace:

```text
dog.speak → not own
          ↓
animal.speak → found
          ↓
call function with dog as receiver
```

---

## Example 2 — Shadowing

```js
const parent = {
  value: 10,
};

const child = Object.create(parent);

console.log(child.value);

child.value = 20;

console.log(child.value);
console.log(parent.value);
```

Prediction:

```text
10
20
10
```

---

## Example 3 — Missing Property

```js
const parent = {};

const child = Object.create(parent);

console.log(child.missing);
```

Result:

```text
undefined
```

The lookup reaches:

```text
null
```

without finding the property.

---

# 9. Execution Walkthrough

Consider:

```js
const grandParent = {
  level: "grandparent",
};

const parent = Object.create(grandParent);

parent.level = "parent";

const child = Object.create(parent);

console.log(child.level);
```

Chain:

```text
child
 ↓
parent
 ↓
grandParent
 ↓
Object.prototype? / null depending on construction
```

Lookup:

```text
child.level
```

1. Check `child` → not found.
2. Go to `parent`.
3. Check `parent` → found.
4. Return `"parent"`.

The lookup does not continue after a match.

---

# 10. Internal Mechanics

At the specification level, property access is expressed through internal methods and abstract operations.

Important concepts include:

```text
[[Get]]
[[Set]]
[[HasProperty]]
[[GetOwnProperty]]
[[GetPrototypeOf]]
[[SetPrototypeOf]]
```

A read such as:

```js
obj.key
```

uses object property access semantics.

For ordinary objects, the relevant lookup behavior follows the ordinary internal method algorithms.

You should think:

```text
read
→ inspect receiver
→ if missing, follow [[Prototype]]
→ repeat
```

rather than:

```text
search one giant dictionary.
```

---

# 11. ECMAScript / Specification Semantics

The ECMAScript specification defines:

```text
[[Prototype]]
```

as an internal slot of objects that participate in the ordinary object model.

Property lookup is specified using abstract operations and internal methods, not by requiring a particular engine implementation.

Therefore:

```text
prototype chain behavior
```

is standardized,

while:

```text
hidden class names
inline cache implementation
memory layout
specific optimization heuristics
```

are engine details.

---

# 12. Advanced Behavior

## 12.1 Shadowing

```text
prototype property
      ↓
own property with same key
      ↓
own property wins
```

---

## 12.2 Dynamic Prototype Chain

Conceptually, changing an object's prototype changes where future property lookup may continue.

```js
const a = { x: 1 };
const b = { x: 2 };

const obj = Object.create(a);

console.log(obj.x);

Object.setPrototypeOf(obj, b);

console.log(obj.x);
```

Potential result:

```text
1
2
```

This is powerful but can complicate reasoning and optimization.

---

## 12.3 Property Descriptors on Prototypes

A prototype property can have descriptor behavior.

Example:

```js
const parent = {};

Object.defineProperty(parent, "x", {
  value: 10,
  writable: false,
});

const child = Object.create(parent);

child.x = 20;
```

In strict code or under relevant assignment semantics, assignment may fail rather than creating a new property because the inherited property is non-writable.

Therefore:

> Property assignment depends on the descriptor behavior encountered along the prototype chain, not only on whether the receiver currently owns the property.

---

# 13. Getter Inheritance

A getter may be defined on a prototype:

```js
const parent = {
  get label() {
    return this.name.toUpperCase();
  },
};

const child = Object.create(parent);

child.name = "Milan";

console.log(child.label);
```

The getter is inherited.

But:

```text
this
```

is the receiver:

```text
child
```

not necessarily the prototype object.

---

# 14. Setter Inheritance

A setter can also be inherited.

This creates subtle assignment semantics.

Example:

```js
const parent = {
  set value(v) {
    this._value = v * 2;
  },
};

const child = Object.create(parent);

child.value = 10;
```

The inherited setter executes with:

```text
this === child
```

and may create or modify state on `child`.

This pattern is useful but can surprise developers who expect assignment to always define an own data property.

---

# 15. `hasOwn` vs `in`

```js
const parent = {
  role: "admin",
};

const child = Object.create(parent);

child.name = "Milan";

console.log(Object.hasOwn(child, "role"));
console.log("role" in child);
```

Result:

```text
false
true
```

Use:

```js
Object.hasOwn(obj, key)
```

when you mean:

```text
direct ownership.
```

Use:

```js
key in obj
```

when inherited properties count.

---

# 16. `Object.getOwnPropertyNames` and Symbols

Prototype-aware design requires understanding property-key inspection.

```js
Object.keys(obj);
Object.getOwnPropertyNames(obj);
Object.getOwnPropertySymbols(obj);
Reflect.ownKeys(obj);
```

These APIs differ in:

```text
enumerability
symbol inclusion
scope
```

None of them automatically mean:

```text
all properties from the full prototype chain.
```

---

# 17. Traversing the Entire Prototype Chain

Conceptually:

```js
function getPrototypeChain(object) {
  const chain = [];

  let current = object;

  while (current !== null) {
    chain.push(current);
    current = Object.getPrototypeOf(current);
  }

  return chain;
}
```

This follows:

```text
object
→ prototype
→ prototype
→ null
```

It does not copy any objects.

---

# 18. `Object.prototype`

Many standard object literals have a chain that reaches:

```js
Object.prototype
```

Example:

```js
const obj = {};

console.log(Object.getPrototypeOf(obj) === Object.prototype);
```

Typically:

```text
true
```

Then:

```js
Object.getPrototypeOf(Object.prototype) === null
```

is:

```text
true
```

---

# 19. Null-Prototype Objects

```js
const dict = Object.create(null);
```

Chain:

```text
dict
 ↓
null
```

No:

```text
Object.prototype
```

is inherited.

This can be useful for pure dictionary-style structures.

But it changes expectations around inherited methods such as:

```text
toString
hasOwnProperty
constructor
```

The recommended explicit ownership check remains:

```js
Object.hasOwn(dict, key);
```

---

# 20. Constructor Functions

Before `class` syntax, constructor functions were a common OOP mechanism:

```js
function User(name) {
  this.name = name;
}
```

A function object can have:

```text
prototype
```

property.

For a conventional constructor:

```js
User.prototype
```

is an object used as the prototype of instances created through:

```js
new User(...)
```

---

# 21. The `prototype` Property of a Constructor

Consider:

```js
function User() {}
```

Conceptually:

```text
User
 │
 └── prototype → Prototype Object
                    │
                    └── constructor → User
```

Then:

```js
const user = new User();
```

gives the new object a prototype relationship to:

```js
User.prototype
```

subject to the constructor's construction semantics.

This is why:

```js
Object.getPrototypeOf(user) === User.prototype
```

is normally:

```text
true
```

for this simple case.

---

# 22. `new` — Conceptual Walkthrough

For a constructable ordinary function, a useful conceptual model is:

```text
new User("Milan")
        │
        ▼
create new object
        │
        ▼
set object's [[Prototype]]
to User.prototype
        │
        ▼
call User with this = new object
        │
        ▼
return constructed result
```

This is a mental model, not a complete specification transcription.

Constructors can explicitly return objects, and JavaScript has additional construction rules and edge cases.

---

# 23. Prototype Methods

Instead of:

```js
function User(name) {
  this.name = name;

  this.greet = function () {
    return this.name;
  };
}
```

you can share behavior:

```js
function User(name) {
  this.name = name;
}

User.prototype.greet = function () {
  return this.name;
};
```

Now instances delegate to:

```text
User.prototype.greet
```

instead of storing a separate method function on every instance.

---

# 24. Instance Behavior vs Static Behavior

Prototype methods:

```js
User.prototype.greet
```

are normally available to instances.

Static methods:

```js
User.findById
```

belong to the constructor/function itself.

Conceptually:

```text
User
├── static methods
│
└── prototype ──→ instance methods
                     ↑
                     │
                  user1
                  user2
```

This distinction is essential for class design.

---

# 25. Classes Use Prototypes

Example:

```js
class User {
  greet() {
    return "hello";
  }
}
```

Instances typically share:

```text
User.prototype.greet
```

rather than owning an independent `greet` function each.

Therefore class syntax does not eliminate the prototype model.

It provides a higher-level language construct over it.

---

# 26. Static Class Members

```js
class User {
  static createGuest() {
    return new User();
  }

  greet() {
    return "hello";
  }
}
```

Then:

```js
User.createGuest(); // static side
new User().greet(); // instance side
```

Mental model:

```text
User
 ├── static side
 │
 └── prototype
       └── instance methods
```

---

# 27. Class Inheritance

Consider:

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

A useful model:

```text
dog instance
   ↓
Dog.prototype
   ↓
Animal.prototype
   ↓
Object.prototype
   ↓
null
```

Thus class inheritance still results in a prototype chain.

---

# 28. Shadowing in Class Inheritance

```js
class Animal {
  speak() {
    return "animal";
  }
}

class Dog extends Animal {
  speak() {
    return "dog";
  }
}
```

Lookup:

```text
dog.speak
 ↓
Dog.prototype.speak ✓
```

The parent implementation is shadowed by the child prototype's own method.

---

# 29. `super`

Within class methods, `super` provides superclass-oriented property access semantics.

```js
class Animal {
  speak() {
    return "animal";
  }
}

class Dog extends Animal {
  speak() {
    return super.speak() + " dog";
  }
}
```

This does not mean:

```text
copy parent methods into child.
```

It means the language provides specific superclass reference semantics based on the method's home object and prototype relationships.

---

# 30. `this` and Prototypes

Important principle:

```text
method location
≠
this value
```

Example:

```js
const parent = {
  name: "Parent",

  greet() {
    return this.name;
  },
};

const child = Object.create(parent);

child.name = "Child";

console.log(child.greet());
```

The first call has:

```text
this = child
```

If you detach:

```js
const fn = child.greet;
```

and later call:

```js
fn();
```

the `this` behavior changes according to the call form and execution mode.

Prototype lookup finds the function; the call expression determines its receiver semantics.

---

# 31. Prototype vs `[[Prototype]]`

This is one of the highest-priority distinctions in JavaScript OOP.

```text
[[Prototype]]
```

is an internal relationship of an object.

```text
prototype
```

is an ordinary property, commonly exposed on constructor functions.

Example:

```js
function User() {}

const user = new User();

Object.getPrototypeOf(user) === User.prototype;
```

Here:

```text
user.[[Prototype]] === User.prototype
```

Conceptually.

But:

```text
User.[[Prototype]]
```

is another relationship entirely.

Do not collapse these into one concept.

---

# 32. `Object.setPrototypeOf`

Example:

```js
const a = {
  x: 1,
};

const b = {
  x: 2,
};

const obj = Object.create(a);

console.log(obj.x);

Object.setPrototypeOf(obj, b);

console.log(obj.x);
```

This demonstrates dynamic delegation.

However, changing prototypes after creation can make object behavior harder to reason about and can have engine-specific performance consequences.

Prefer stable construction patterns unless dynamic prototype changes are genuinely required.

---

# 33. Why Dynamic Prototype Mutation Can Be Expensive

JavaScript engines may optimize property access using internal structures and caches.

Changing prototypes can invalidate assumptions.

The exact mechanism is engine-specific.

Therefore the safe rule is:

```text
stable prototype relationships
→ generally easier for engines and humans to reason about

dynamic prototype mutation
→ powerful but use intentionally
```

---

# 34. Prototype Pollution

Prototype pollution occurs when attacker-controlled or otherwise unintended data changes prototype state in a way that affects unrelated objects.

Conceptual risk:

```text
untrusted input
     ↓
prototype mutation
     ↓
shared inherited property
     ↓
unexpected behavior elsewhere
```

This can become a security vulnerability in applications using unsafe recursive merges, path setters, or object update utilities.

Defenses include:

```text
validate keys
avoid unsafe prototype mutation
treat untrusted object data as untrusted
use appropriate null-prototype/Map structures where useful
```

Security will be covered in much greater depth later.

---

# 35. Prototype Chain and Composition

Prototype inheritance can model:

```text
"is-a"
```

relationships.

But LLD often benefits more from:

```text
"has-a"
```

relationships represented through composition.

Example:

```js
class Car {
  constructor(engine) {
    this.engine = engine;
  }
}
```

Here:

```text
Car
 └── has an Engine
```

This does not require prototype inheritance between:

```text
Car
Engine
```

Use the prototype system when delegation/inheritance represents the design.

Do not use inheritance simply because it is available.

---

# 36. Prototype Chain as Delegation

A useful object-oriented mental model is:

```text
receiver
  ↓
"Can you handle this?"
  ↓ no
prototype
  ↓
"Can you handle this?"
  ↓ no
next prototype
  ↓
...
```

This is delegation.

A prototype is therefore not merely:

```text
"parent data."
```

It is a fallback object participating in property behavior.

---

# 37. Advanced Behavior — Writable Inheritance

Consider:

```js
const parent = {
  x: 1,
};

const child = Object.create(parent);

child.x = 2;
```

Because the inherited property is a writable data property, ordinary assignment can create an own property on the receiver.

Afterward:

```text
parent.x = 1
child.x  = 2
```

This is classic shadowing.

---

# 38. Advanced Behavior — Non-Writable Inheritance

Now:

```js
const parent = {};

Object.defineProperty(parent, "x", {
  value: 1,
  writable: false,
});

const child = Object.create(parent);

child.x = 2;
```

Under strict mode, this assignment throws a `TypeError`.

The prototype's descriptor matters.

This shows:

> The prototype chain participates in the semantics of assignment, not merely reads.

---

# 39. Advanced Behavior — Accessor Inheritance

```js
const parent = {
  set x(value) {
    this._x = value * 2;
  },
};

const child = Object.create(parent);

child.x = 10;

console.log(child._x);
```

Result:

```text
20
```

The setter was inherited but wrote through:

```text
this
```

to the receiver.

---

# 40. Advanced Behavior — Missing Setter

A data-property/setter distinction matters.

When an inherited property is an accessor with no setter, assignment does not behave like ordinary creation of an own data property under all circumstances.

This is another reason property descriptors are part of the object model.

---

# 41. Edge Cases

## Null Prototype

```js
Object.getPrototypeOf(Object.create(null)) === null
```

---

## Prototype Cycles

Ordinary prototype chains cannot form arbitrary cycles under standard constraints. Attempts to create cyclic prototype relationships are rejected.

---

## Own `__proto__`

Modern code should distinguish:

```text
__proto__ accessor behavior
```

from:

```text
[[Prototype]]
```

and prefer:

```js
Object.getPrototypeOf
Object.setPrototypeOf
Object.create
```

for explicit prototype operations.

---

## Constructor Property

`obj.constructor` may be inherited and can be shadowed.

Therefore:

```js
obj.constructor === ExpectedConstructor
```

is not a universally trustworthy test of object provenance.

---

# 42. Edge Cases — Property Key Coercion

Property access:

```js
obj[key]
```

performs property-key conversion according to language semantics.

Objects used as property keys can be coerced rather than acting as object-identity keys.

For keying by arbitrary object identity, use:

```text
Map
WeakMap
```

rather than plain-object properties.

---

# 43. Common Misconceptions

## "The prototype is copied into the object."

False.

Lookup delegates to another object.

---

## "Every object has a `.prototype` property."

False.

Objects have an internal:

```text
[[Prototype]]
```

relationship.

---

## "Only classes use prototypes."

False.

Object literals, constructor functions, built-in objects and manually-created objects all participate in prototype semantics.

---

## "Changing a prototype only affects one method."

Potentially false.

It can affect multiple inherited property lookups.

---

## "Methods belong to instances."

Often they are found on prototypes instead.

---

## "If a method is on a prototype, this points to the prototype."

False.

`this` depends on the call.

---

## "Inheritance means copying parent state."

Not fundamentally.

Prototype inheritance is delegation.

---

# 44. Common Mistakes

```text
[ ] confusing prototype with [[Prototype]]
[ ] using setPrototypeOf casually
[ ] assuming inherited properties are own properties
[ ] assuming methods are copied to every instance
[ ] misunderstanding shadowing
[ ] forgetting inherited descriptors affect assignment
[ ] using constructor as a security/provenance guarantee
[ ] using object property keys for object-identity maps
[ ] ignoring prototype pollution risks
[ ] forcing inheritance where composition is clearer
```

---

# 45. Comparison With Related Concepts

| Concept | Meaning |
|---|---|
| `[[Prototype]]` | Internal prototype relationship |
| `prototype` | Property commonly used by constructors |
| Prototype chain | Sequence of delegated prototype objects |
| Inheritance | Reuse/delegation relationship |
| Composition | Building objects from collaborating objects |
| Own property | Directly stored on receiver |
| Inherited property | Resolved through prototype chain |
| Class | Higher-level construction/inheritance syntax |
| Static method | Behavior on constructor/class side |
| Instance method | Behavior typically reachable through prototype |
| Delegation | Request falls back to another object |

---

# 46. Performance Considerations

Prototype lookup is often highly optimized by modern engines.

Engine-specific optimization concepts can include:

```text
hidden classes / shapes
inline caches
prototype validity assumptions
polymorphic access patterns
```

Names and exact mechanisms vary by engine and version.

From an application perspective:

```text
stable object structure
stable prototype relationships
predictable access patterns
```

are usually easier for both humans and runtimes to optimize than highly dynamic object mutation.

Measure actual workloads before changing design for performance.

---

# 47. Memory Considerations

Prototype sharing can reduce repeated method storage.

Conceptually:

```text
10,000 instances
        │
        └──→ one shared prototype method
```

instead of:

```text
10,000 instances
        │
        └──→ 10,000 separate function values
```

But prototypes themselves remain objects.

Long prototype chains can also increase conceptual complexity and potentially affect lookup costs, although engines can optimize common cases.

---

# 48. Security Considerations

Prototype relationships are a security-sensitive surface.

Key risks include:

```text
prototype pollution
unsafe merges
dynamic key paths
untrusted assignment
accidental inherited configuration
```

For security-sensitive dictionaries, consider whether a `Map` or null-prototype object gives a clearer boundary.

Never treat:

```text
constructor
prototype
__proto__
```

as harmless user-controlled keys without evaluating the context.

---

# 49. Production Usage

Use prototypes naturally when:

```text
instances share behavior
inheritance is semantically justified
a framework/API expects prototype-based extension
```

Avoid prototype manipulation when:

```text
composition expresses the domain more clearly
runtime mutation is unnecessary
a stable class/factory/module boundary is sufficient
```

Production object design should favor:

```text
stable relationships
explicit responsibilities
clear ownership
limited inheritance depth.
```

---

# 50. Implementation From Scratch

## Exercise 1 — Prototype Chain Printer

Implement:

```js
function describePrototypeChain(object) {}
```

Return:

```text
depth
object identity marker
constructor name when available
```

Do not rely on constructor name as a correctness guarantee.

---

## Exercise 2 — Property Lookup Simulator

Implement a simplified:

```js
function findPropertyOwner(object, key) {}
```

Return the first object in the chain that owns the property.

Requirements:

```text
own property
inherited property
missing property
symbol key
null prototype.
```

---

## Exercise 3 — Constructor-Based OOP

Implement:

```js
function User(name) {
  this.name = name;
}

User.prototype.greet = function () {
  return `Hello ${this.name}`;
};
```

Create multiple users.

Prove:

```text
user1 !== user2
user1.greet === user2.greet
```

---

## Exercise 4 — Class Equivalent

Rewrite the constructor-function solution using:

```js
class User {}
```

Then inspect:

```text
User.prototype
user.__proto__ / Object.getPrototypeOf(user)
User static side
```

Use standard APIs in production code rather than depending on non-standard legacy shortcuts.

---

## Exercise 5 — Safe Dictionary

Compare:

```text
{}
Object.create(null)
Map
```

for a use case involving arbitrary external keys.

Defend the choice.

---

# 51. Debugging Exercises

## Debug 1 — Unexpected Inheritance

```js
const defaults = {
  timeout: 1000,
};

const config = Object.create(defaults);

config.retries = 3;

console.log(config.timeout);
```

Determine whether `timeout` is owned by `config`.

---

## Debug 2 — Shadowing

```js
const base = {
  status: "base",
};

const child = Object.create(base);

child.status = "child";

console.log(base.status);
console.log(child.status);
```

Explain why two reads differ.

---

## Debug 3 — Detached Method

```js
const user = {
  name: "Milan",

  greet() {
    return this.name;
  },
};

const fn = user.greet;

console.log(fn());
```

Explain why extracting a method changes the `this` situation.

---

## Debug 4 — Inherited Setter

```js
const parent = {
  set value(v) {
    this._value = v * 2;
  },
};

const child = Object.create(parent);

child.value = 10;

console.log(child._value);
```

Trace the receiver and the property owner.

---

# 52. Code Review Exercise

Review:

```js
class Admin extends User {
  constructor(name) {
    super(name);
    this.permissions = [];
  }
}
```

Discuss:

```text
1. Does inheritance represent a real "is-a" relationship?
2. Should permissions belong to Admin or a separate policy object?
3. What behavior actually needs to be inherited?
4. Would composition reduce coupling?
5. What is the prototype chain?
6. What state is owned by each instance?
7. What is static vs instance behavior?
```

---

# 53. Interview Questions

## Fundamentals

```text
1. What is the prototype chain?
2. What is [[Prototype]]?
3. What is the difference between prototype and [[Prototype]]?
4. How does property lookup work?
5. What is shadowing?
6. What does Object.create do?
7. What does Object.getPrototypeOf do?
```

## Constructors

```text
8. What is Function.prototype in relation to constructor functions?
9. Why does new User() connect the instance to User.prototype?
10. Where do prototype methods live?
11. Why are prototype methods shared?
```

## Classes

```text
12. Do classes remove prototypes?
13. What is the difference between instance and static methods?
14. How does extends relate to the prototype chain?
15. What does super do conceptually?
```

## Design

```text
16. When should inheritance be used?
17. When is composition better?
18. Why can dynamic prototype mutation be problematic?
19. What is prototype pollution?
20. How would you prevent inherited properties from affecting a dictionary design?
```

---

# 54. Predict-the-Output Exercises

## Exercise A

```js
const parent = {
  x: 10,
};

const child = Object.create(parent);

console.log(child.x);
console.log(Object.hasOwn(child, "x"));
```

---

## Exercise B

```js
const parent = {
  x: 10,
};

const child = Object.create(parent);

child.x = 20;

console.log(child.x);
console.log(parent.x);
console.log(Object.hasOwn(child, "x"));
```

---

## Exercise C

```js
function User() {}

User.prototype.greet = function () {
  return "hello";
};

const a = new User();
const b = new User();

console.log(a.greet());
console.log(a.greet === b.greet);
console.log(Object.getPrototypeOf(a) === User.prototype);
```

---

## Exercise D

```js
const parent = {
  name: "Parent",

  greet() {
    return this.name;
  },
};

const child = Object.create(parent);

child.name = "Child";

console.log(child.greet());
```

---

## Exercise E

```js
const parent = {
  value: 1,
};

const child = Object.create(parent);

console.log(child.value);

Object.setPrototypeOf(child, {
  value: 2,
});

console.log(child.value);
```

---

## Exercise F

```js
const dictionary = Object.create(null);

dictionary.name = "Milan";

console.log(Object.getPrototypeOf(dictionary));
console.log(Object.hasOwn(dictionary, "name"));
```

---

# 55. Mastery Exercises

## Level 1 — Understand

Explain:

```text
prototype
[[Prototype]]
prototype chain
delegation
shadowing.
```

---

## Level 2 — Explain

Draw:

```text
instance
→ subclass prototype
→ superclass prototype
→ Object.prototype
→ null
```

for a three-level class hierarchy.

---

## Level 3 — Predict

Trace property reads and writes involving:

```text
own data property
inherited data property
inherited getter
inherited setter
non-writable inherited property.
```

---

## Level 4 — Implement

Build:

```text
prototype inspector
property-owner finder
constructor-function model
prototype-chain visualizer.
```

---

## Level 5 — Debug

Fix:

```text
wrong shadowing
unexpected inherited configuration
detached method failure
unsafe prototype mutation.
```

---

## Level 6 — Apply

Design a plugin system where plugin implementations share behavior through either:

```text
prototype inheritance
or
composition.
```

Defend the choice.

---

## Level 7 — Compare

Compare:

```text
prototype delegation
class inheritance
composition
duck typing
```

for an extensible business rule system.

---

## Level 8 — Defend

Answer:

> Why is understanding the prototype chain necessary before judging whether a JavaScript class hierarchy is well designed?

Your answer must connect:

```text
lookup
delegation
inheritance
shadowing
this
coupling
composition.
```

---

# 56. Key Takeaways

```text
1. [[Prototype]] is the internal prototype relationship.
2. The prototype chain is a delegation chain.
3. Property lookup starts with the receiver.
4. Own properties shadow inherited properties.
5. Prototype properties are not copied into instances during lookup.
6. this depends on the call, not where a function was found.
7. Function constructors commonly use their prototype property for instances.
8. new connects constructed instances to the constructor's prototype by default.
9. Class instance methods participate in prototype-based sharing.
10. Static behavior belongs to the constructor/class side.
11. extends creates prototype relationships between class prototypes.
12. Property descriptors on prototypes can affect assignment semantics.
13. Object.create is a direct prototype-oriented construction tool.
14. Dynamic prototype mutation should be deliberate.
15. Null-prototype objects eliminate Object.prototype inheritance.
16. Prototype pollution is a security concern.
17. Prototype inheritance is delegation, not object copying.
18. Composition is often a better alternative than inheritance.
19. Stable prototype relationships help reasoning and may help engine optimization.
20. Understanding the prototype chain is mandatory for serious JavaScript OOP.
```

---

# 57. Concept Connections

## Depends On

```text
Chapter 1 — JavaScript Object Model — Foundation
Chapter 2 — Object Identity, Equality & Mutability
properties
descriptors
functions
references
```

## Builds Toward

```text
Chapter 4 — Constructor Functions & Instance Construction
Chapter 5 — JavaScript Classes Internally
Chapter 6 — Encapsulation Boundaries
Chapter 7 — Abstraction
Chapter 8 — Inheritance
Chapter 9 — Polymorphism
Chapter 11 — Composition
Chapter 17 — Cohesion & Coupling
Chapter 19+ — GRASP
Chapter 45+ — TypeScript Class/Object Design
```

## Related Concepts

```text
delegation
inheritance
composition
method lookup
this binding
descriptors
prototype pollution
```

## Why This Chapter Matters

A JavaScript class is easier to reason about when you can mentally reduce it to:

```text
object identity
+
prototype relationship
+
property lookup
+
call semantics.
```

That is the runtime foundation of JavaScript OOP.

---

# 58. Completion Criteria

Mark:

```text
[+] Completed
```

when you can:

```text
trace a prototype chain
explain shadowing
distinguish prototype from [[Prototype]]
explain constructor prototypes
explain class prototypes
inspect prototypes with standard APIs
debug inherited-property problems.
```

Mark:

```text
[*] Mastered
```

when you can design and defend:

```text
inheritance
delegation
composition
prototype mutation decisions
```

without confusing language mechanics with design quality.

Reading alone does not mark mastery.

---

# 59. Revision / Retrieval Record

```md
# Chapter 3 — Revision / Retrieval Record

## Attempt
- Date:
- Duration:
- Status before:
- Status after:

## Prototype Model
- What is [[Prototype]]?
- What is prototype?
- How does lookup proceed?

## Shadowing
- What gets shadowed?
- What happens to the parent property?

## Constructors
- What does new conceptually do?
- What role does Constructor.prototype play?

## Classes
- Where do instance methods live?
- Where do static methods live?
- How does extends affect the prototype chain?

## Debugging
- Prototype issue found:
- Root cause:
- Fix:

## Performance
- What dynamic behavior should be avoided?
- What did I learn about engine-specific optimization?

## Security
- What is prototype pollution?
- How would I defend against it?

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

---

# 60. Canonical References and Source Discipline

Primary semantic source:

```text
ECMAScript Language Specification
https://tc39.es/ecma262/
```

Useful reference material:

```text
MDN — Inheritance and the prototype chain
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain

MDN — Object.create
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create

MDN — Object.getPrototypeOf
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getPrototypeOf

MDN — Object.setPrototypeOf
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf

MDN — Classes
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes
```

Source discipline:

```text
prototype semantics
→ ECMAScript

API behavior
→ standard library/runtime documentation

optimization
→ engine-specific documentation

design quality
→ application requirements and engineering trade-offs
```

---

# 61. Completion Snapshot

```text
Chapter: 003
Title: Prototype Chain Deep Dive

Theory         [ ]
Implementation  [ ]
Debugging       [ ]
Code Review     [ ]
Interview       [ ]
Prediction      [ ]
Mastery         [ ]

Overall: [ ] Not Started
```

---

# Final Mental Model

Always visualize:

```text
                 PROPERTY READ
                      │
                      ▼
                 receiver
                      │
             own property?
                /         \
              yes          no
               │             │
               ▼             ▼
             value      [[Prototype]]
                              │
                              ▼
                          next object
                              │
                           repeat
                              │
                              ▼
                            null
```

And for constructors/classes:

```text
             Constructor / Class
                    │
                    ├── static behavior
                    │
                    └── prototype
                          │
                     instance methods
                          ▲
                          │
                      instance
                          │
                    [[Prototype]]
```

Then ask:

```text
1. Where is the property actually owned?
2. Which object will lookup inspect next?
3. Is the property shadowed?
4. What descriptor controls the operation?
5. What object becomes this?
6. Is this inheritance actually the right design?
```

---

# Principal Design Principle

> **Prototype mechanics explain how behavior is found; they do not by themselves prove that inheritance is a good design.**

---

# Track Mapping

```text
Track A — Core Theory
    prototype chains
    [[Prototype]]
    property lookup
    shadowing
    descriptors
    constructor/class relationships

Track B — Implementation
    prototype inspectors
    constructor OOP
    class equivalents
    dictionary design

Track C — Interview / Reasoning
    lookup tracing
    shadowing prediction
    new mental model
    inheritance vs composition
    prototype security/performance trade-offs
```
