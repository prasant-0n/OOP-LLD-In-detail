# Chapter 2 — Object Identity, Equality & Mutability

> **JavaScript OOP + LLD Mastery**
>
> This chapter turns the object-model foundation into design reasoning. Identity, equality, mutability, aliasing, copying, and ownership determine whether an object is a stable domain concept or a source of hidden coupling. These ideas will later drive entities vs value objects, encapsulation, aggregates, immutability, concurrency safety, caching, persistence, and API design.

**Status:** `[ ] Not Started`

---

# 1. Learning Objectives

By the end of this chapter, you should be able to:

```text
[ ] distinguish identity from value equality
[ ] explain === for objects
[ ] explain Object.is
[ ] explain SameValueZero conceptually
[ ] explain why NaN and signed zero are special
[ ] distinguish reference aliasing from copying
[ ] explain shallow copy
[ ] explain deep copy
[ ] identify nested-reference sharing
[ ] identify accidental shared mutable state
[ ] define mutability precisely
[ ] distinguish shallow immutability from deep immutability
[ ] explain const vs object immutability
[ ] explain Object.freeze limitations
[ ] reason about defensive copying
[ ] explain when copying is unnecessary
[ ] explain ownership in application design
[ ] explain identity-based entities
[ ] explain value-based objects
[ ] design explicit equality methods
[ ] explain canonicalization/interning conceptually
[ ] reason about mutation and invariants
[ ] identify mutation leaks at API boundaries
[ ] reason about copy-on-write conceptually
[ ] compare mutation, replacement and persistent-state styles
[ ] debug shared-reference defects
[ ] predict equality and mutation outcomes
[ ] implement safe boundary utilities
[ ] defend trade-offs among identity, copying and immutability
```

---

# 2. Prerequisites

Required foundations:

```text
Chapter 1 — JavaScript Object Model — Foundation
primitive values
objects
references
properties
prototype basics
functions
assignment
arrays
```

---

# 3. What Is It?

Five concepts form the core:

```text
Identity
Equality
Mutability
Aliasing
Ownership
```

A compact model:

```text
                 OBJECT
                   │
        ┌──────────┴──────────┐
        │                     │
     IDENTITY                STATE
        │                     │
        │                mutable/immutable
        │                     │
        └──────────┬──────────┘
                   │
                REFERENCES
                   │
              aliasing/copying
```

The central design problem is:

> **How do we control who can observe and change an object's state while preserving the meaning of the object?**

---

# 4. Why Does It Exist?

Real software contains concepts such as:

```text
Customer
Order
Money
Address
Cart
Product
UserSession
DatabaseConnection
CacheEntry
```

Some are identity-driven:

```text
Customer
Order
Session
```

Some are value-driven:

```text
Money
Date range
Percentage
Email address
Coordinates
```

Some should be mutable:

```text
shopping cart during checkout
workflow state
transaction context
```

Some should strongly prefer immutability:

```text
configuration snapshots
value objects
events
request objects
```

Incorrectly choosing an identity or mutability model creates bugs that often look unrelated to the original design decision.

---

# 5. Mental Model

## Identity Model

```text
Entity A ──────┐
               │
               ▼
            Object #17
               ▲
               │
Entity Ref ────┘
```

Same object:

```text
same identity
```

Two separately-created but equal-looking objects:

```text
Object #17
Object #42

same values
different identity
```

---

## Value Model

For a value object, identity may be irrelevant:

```text
Money(100, "INR")
```

is equivalent to another:

```text
Money(100, "INR")
```

when the domain defines equality by value.

Therefore:

```text
language identity
≠
domain equality.
```

---

# 6. Core Rules

## Rule 1 — `===` Does Not Deep-Compare Objects

```js
const a = { x: 1 };
const b = { x: 1 };

console.log(a === b);
```

Result:

```text
false
```

---

## Rule 2 — Two References Can Identify One Object

```js
const a = {};
const b = a;

console.log(a === b);
```

Result:

```text
true
```

---

## Rule 3 — `const` Prevents Rebinding, Not Mutation

```js
const user = {
  name: "Milan",
};

user.name = "Prasant";
```

This is allowed because the variable still refers to the same object.

But:

```js
user = {};
```

is not allowed because the binding is being reassigned.

---

## Rule 4 — Mutation Is an Operation on Existing Identity

```text
Object #17
   │
   ├── before: balance = 100
   │
   └── after:  balance = 50
```

The object identity remains:

```text
Object #17
```

only its state changes.

---

## Rule 5 — Replacement Creates a Different Identity

```js
let state = {
  count: 1,
};

state = {
  count: 2,
};
```

The variable has moved from:

```text
Object #1
```

to:

```text
Object #2
```

This differs fundamentally from mutating:

```js
state.count = 2;
```

---

## Rule 6 — Aliasing Makes Mutation Observable Across Boundaries

```text
A ─────┐
       ├──> Mutable Object
B ─────┘
```

Mutation through `A` is observable through `B`.

---

## Rule 7 — Copy Depth Matters

```text
shallow copy
→ new outer object
→ nested object references may remain shared
```

Deep copy attempts to create independent nested object state, subject to the copying mechanism's semantics and limitations.

---

# 7. Syntax

## Strict Equality

```js
a === b;
```

---

## Same-Value Comparison

```js
Object.is(a, b);
```

---

## Object Freezing

```js
Object.freeze(object);
```

---

## Copying

```js
const copy = { ...source };
const copy2 = Object.assign({}, source);
```

---

# 8. Basic Examples

## Example 1 — Identity

```js
const userA = { id: 1 };
const userB = { id: 1 };
const userC = userA;

console.log(userA === userB);
console.log(userA === userC);
```

Prediction:

```text
false
true
```

Trace:

```text
userA → Object #1
userB → Object #2
userC → Object #1
```

---

## Example 2 — `Object.is`

```js
console.log(Object.is(NaN, NaN));
console.log(Object.is(0, -0));
```

Prediction:

```text
true
false
```

---

## Example 3 — `includes`

```js
console.log([NaN].includes(NaN));
```

Prediction:

```text
true
```

This is a useful example of why equality semantics matter.

---

# 9. Execution Walkthrough — Aliasing

```js
const order = {
  total: 100,
};

const external = order;

external.total = 250;

console.log(order.total);
```

Execution:

```text
1. Create Object #1.
2. Bind order → Object #1.
3. Bind external → Object #1.
4. Write total = 250 through external.
5. Read total through order.
```

Result:

```text
250
```

The surprising behavior is not caused by assignment being "weird."

It is caused by two references naming one mutable identity.

---

# 10. ECMAScript / Specification Semantics

The equality relations discussed here are defined by ECMAScript algorithms and abstract comparison semantics. The language specification is the authoritative source for their exact rules.

At a design level, these relations answer different questions:

```text
Are these two values strictly equal?
Are these two values the same under SameValue?
Are these two values equivalent under SameValueZero?
```

The distinction matters most at edge cases such as:

```text
NaN
+0
-0
objects
```

---

# 11. Internal Mechanics

At runtime, equality and mutation are evaluated according to language semantics over values, references, and object state. For design reasoning, model an object reference as identifying an existing object rather than carrying an independent copy of that object.

```text
reference
   ↓
object identity
   ↓
current observable state
```

Mutation changes the reachable object state while preserving that object identity. Replacement changes which object a binding refers to.

This distinction is the foundation for reasoning about:

```text
shared state
cache entries
entity objects
value objects
repository identity
workflow state
```

---

# 12. Mutability

An object is mutable when its observable state can be changed after creation.

Examples:

```js
const user = {
  active: true,
};

user.active = false;
```

State changed.

This does not necessarily mean:

```text
all properties are mutable
```

or:

```text
the object can be reconfigured arbitrarily.
```

Mutability can be controlled.

---

# 13. `const` vs Mutability

This:

```js
const config = {
  retries: 3,
};

config.retries = 5;
```

is valid.

This:

```js
config = {};
```

is invalid.

Mental model:

```text
const
→ binding cannot be rebound

Object.freeze
→ object properties become restricted

custom immutable design
→ object graph/domain contract defines how state changes are controlled
```

These are different mechanisms.

---

# 14. Shallow Copy

Consider:

```js
const source = {
  name: "Milan",
  profile: {
    role: "developer",
  },
};

const copy = { ...source };
```

Model:

```text
source ───────→ Object #1
                  │
                  └── profile ──→ Object #2

copy ─────────→ Object #3
                  │
                  └── profile ──→ Object #2
```

Only the outer object was copied.

---

# 15. Deep Copy Is Not One Universal Operation

There is no single general rule saying:

```text
deep copy = recursively clone everything.
```

Real objects may involve:

```text
Date
Map
Set
RegExp
ArrayBuffer
TypedArrays
functions
class instances
accessors
symbols
non-enumerable properties
prototypes
cycles
shared subgraphs
WeakMap / WeakSet
Proxy
external resources.
```

Therefore:

> Copy semantics must be selected according to the data model.

---

# 16. `structuredClone`

Modern JavaScript environments provide:

```js
structuredClone(value);
```

for supported structured-cloneable values.

Example:

```js
const source = {
  nested: {
    value: 1,
  },
};

const copy = structuredClone(source);

copy.nested.value = 2;

console.log(source.nested.value);
console.log(copy.nested.value);
```

Typical result:

```text
1
2
```

But `structuredClone` is not equivalent to:

```text
clone any arbitrary object while preserving every runtime property
```

It has defined supported types and semantics.

---

# 17. JSON Copying Is Not General Deep Cloning

This pattern:

```js
JSON.parse(JSON.stringify(value));
```

has major semantic limitations.

Potential losses/problems include:

```text
undefined
functions
symbols
special numeric values
Date representation
Map/Set
prototype information
cycles
non-JSON data.
```

Therefore:

> JSON serialization is a serialization technique, not a universal cloning algorithm.

---

# 18. Defensive Copying

Suppose:

```js
class Profile {
  constructor(data) {
    this.data = data;
  }
}
```

External code can retain:

```js
const data = {
  name: "Milan",
};

const profile = new Profile(data);

data.name = "Changed";
```

Now the object's state has changed indirectly.

Defensive copying changes the boundary:

```js
this.data = { ...data };
```

But copying must match the data depth.

---

# 19. Copy on Input vs Copy on Output

Two API boundaries exist.

## Input

```text
caller
  ↓
copy
  ↓
internal state
```

## Output

```text
internal state
  ↓
copy
  ↓
caller
```

A robust encapsulation strategy may need both.

Example:

```js
class Cart {
  #items;

  constructor(items = []) {
    this.#items = [...items];
  }

  getItems() {
    return [...this.#items];
  }
}
```

This prevents the caller from directly retaining the internal array.

However, if items themselves are mutable objects:

```text
array copy
≠
deep isolation.
```

---

# 20. When Defensive Copying Is Appropriate

Consider it when:

```text
the object owns mutable state
the caller should not mutate that state
the state crosses a trust boundary
the object must preserve invariants
the data is small enough that copying is acceptable
```

Do not copy automatically.

Avoid unnecessary copies when:

```text
ownership is intentionally shared
objects are immutable
large graphs make copying expensive
the API contract explicitly shares state
```

---

# 21. Immutability

Immutability means that, according to the relevant contract, state cannot be changed after construction.

There are degrees:

```text
property-level immutability
object-level shallow immutability
deep immutability
domain-level immutability
persistent data structure semantics.
```

Do not use the word "immutable" without clarifying what scope you mean.

---

# 22. Shallow Immutability

```js
const state = Object.freeze({
  profile: {
    name: "Milan",
  },
});
```

The outer object's properties are restricted.

But:

```js
state.profile.name = "Changed";
```

may still mutate the nested object.

Model:

```text
frozen
Object #1
   │
   └── profile → mutable Object #2
```

---

# 23. Deep Immutability

Deep immutability requires the reachable state to obey the immutability contract as well.

That introduces the object-graph problem:

```text
root
 ↓
child
 ↓
child
 ↓
...
```

and cycles:

```text
A → B → C → A
```

Therefore deep immutability is a graph-wide design property, not a single method call.

---

# 24. Mutation vs Replacement

Two common state-update styles:

## Mutation

```js
cart.total = 100;
```

## Replacement

```js
cart = {
  ...cart,
  total: 100,
};
```

Mutation preserves identity.

Replacement creates a new identity.

Neither is universally superior.

---

# 25. Why Identity Matters in Design

Suppose:

```text
Order #1001
```

represents a business entity.

Replacing the object with a new equivalent object may preserve business identity only if the design explicitly carries that identity.

A domain model might use:

```js
class Order {
  #id;

  constructor(id) {
    this.#id = id;
  }

  get id() {
    return this.#id;
  }
}
```

Now:

```text
Object identity
```

and:

```text
business identity
```

are separate concepts.

---

# 26. Entity vs Value Object

A useful DDD distinction:

## Entity

Defined primarily by identity.

```text
Customer ID = C123
```

Two objects can represent the same customer if their identity is the same and the domain allows it.

## Value Object

Defined primarily by value.

```text
Money(500, "INR")
```

Another `Money(500, "INR")` is equivalent under domain equality.

This is not enforced automatically by JavaScript.

The developer must model it.

---

# 27. Explicit Domain Equality

A value object can define:

```js
class Money {
  constructor(amount, currency) {
    this.amount = amount;
    this.currency = currency;
  }

  equals(other) {
    return (
      other instanceof Money &&
      this.amount === other.amount &&
      this.currency === other.currency
    );
  }
}
```

Now:

```js
const a = new Money(500, "INR");
const b = new Money(500, "INR");

console.log(a === b);
console.log(a.equals(b));
```

Result:

```text
false
true
```

This is a core OOP insight:

> JavaScript identity equality and domain equality can coexist.

---

# 28. Equality Must Respect Domain Rules

For a value object, equality should answer:

```text
Which fields define the value?
Which fields are derived?
Are units relevant?
Are currencies relevant?
Are case differences meaningful?
Are normalization rules required?
```

Example:

```text
Email("User@Example.com")
```

might or might not be equal to:

```text
Email("user@example.com")
```

depending on the domain contract.

Do not let accidental string comparison define domain semantics.

---

# 29. Equality Invariants

A well-designed equality operation should generally aim for:

```text
reflexivity
symmetry
transitivity
consistency
```

For example:

```text
a.equals(a) → true
```

and:

```text
a.equals(b) = true
→
b.equals(a) = true
```

and if:

```text
a = b
b = c
```

then:

```text
a = c
```

assuming the relevant values remain stable.

Mutability can make equality reasoning significantly harder.

---

# 30. Equality and Mutability Interaction

Suppose:

```js
class Money {
  constructor(amount, currency) {
    this.amount = amount;
    this.currency = currency;
  }

  equals(other) {
    return (
      this.amount === other.amount &&
      this.currency === other.currency
    );
  }
}
```

If `amount` changes after inserting the object into a collection or using it as a cache key, its logical value changed.

This creates consistency questions:

```text
Was the object supposed to be mutable?
Can its equality change?
Does a cache still work?
Can a map lookup become inconsistent with expectations?
```

This is one reason value objects often benefit from immutability.

---

# 31. Identity Maps and Object Identity

Some systems maintain a rule such as:

```text
one in-memory object per database identity within a scope.
```

Conceptually:

```text
DB id C123
   ↓
Identity Map
   ↓
Object #17
```

A later lookup for C123 returns:

```text
Object #17
```

instead of a second object.

Benefits can include:

```text
consistent in-memory identity
reduced duplicate state
change tracking
```

Costs include:

```text
cache lifecycle complexity
memory retention
stale data considerations
```

Persistence and identity maps will be studied later.

---

# 32. Canonicalization

Canonicalization means representing equivalent values using a preferred canonical representation.

Examples:

```text
normalized email
normalized currency code
normalized path
canonical configuration.
```

This can reduce equality complexity.

But canonicalization is a domain operation, not merely an optimization.

---

# 33. Interning

Interning keeps one shared representative for equivalent values.

Conceptually:

```text
"same logical value"
        ↓
canonical instance
```

Interning can reduce allocations and enable identity-based comparison in controlled cases.

But it introduces:

```text
shared ownership
lifetime questions
memory retention
cache policy.
```

Therefore it should be intentional.

---

# 34. API Boundary Design

Whenever an object crosses a boundary, ask:

```text
Is ownership transferred?
Is state shared?
Is state copied?
Is the object immutable?
Can the receiver mutate it?
Can the sender still mutate it?
```

Examples:

```text
service → repository
authorization layer → domain
domain → event publisher
module → caller
worker → main process
```

These are LLD questions even when no class diagram exists.

---

# 35. Advanced Behavior — Mutation Through Nested Paths

Consider:

```js
const state = {
  user: {
    preferences: {
      theme: "dark",
    },
  },
};
```

The mutation:

```js
state.user.preferences.theme = "light";
```

can be thought of as:

```text
root
 ↓
user
 ↓
preferences
 ↓
theme
```

Only the final property changes.

But every parent object remains the same identity.

This matters to:

```text
memoization
change detection
copy-on-write
state management
cache invalidation.
```

---

# 36. Advanced Behavior — Shared Graphs

Consider:

```js
const address = {
  city: "Hyderabad",
};

const customer = {
  address,
};

const order = {
  shippingAddress: address,
};
```

Graph:

```text
customer ──────┐
               ├──→ Address #7
order ─────────┘
```

Changing:

```js
customer.address.city = "Bengaluru";
```

also changes what `order.shippingAddress` observes.

The object graph is shared.

---

# 37. Advanced Behavior — Cycles

Objects may reference themselves:

```js
const node = {};

node.self = node;
```

Graph:

```text
Node #1
  └────self────→ Node #1
```

This affects:

```text
serialization
deep cloning
graph traversal
equality algorithms
debugging
garbage collection reasoning.
```

A recursive algorithm must track visited identities to avoid infinite traversal.

---

# 38. Edge Cases

## `NaN`

```js
NaN === NaN         // false
Object.is(NaN, NaN) // true
```

## Signed Zero

```js
0 === -0             // true
Object.is(0, -0)     // false
```

## Null

```js
Object.is(null, null); // true
```

## Different Empty Objects

```js
Object.is({}, {}); // false
```

## Same Reference

```js
const x = {};
Object.is(x, x); // true
```

---

# 39. Edge Cases — Frozen Nested State

```js
const config = Object.freeze({
  database: {
    host: "localhost",
  },
});
```

The nested `database` object can still be mutable unless separately protected.

---

# 40. Edge Cases — Getter Side Effects

An API that appears to return state:

```js
const obj = {
  get value() {
    return computeSomething();
  },
};
```

may actually execute code.

Therefore "reading an object property" does not always mean "reading stored data."

This matters for:

```text
copying
logging
serialization
equality
performance
side effects.
```

---

# 41. Edge Cases — Proxies

A `Proxy` can intercept operations such as:

```text
get
set
has
deleteProperty
ownKeys
```

Therefore object operations may have custom behavior.

This is another reason not to assume:

```text
object = passive dictionary.
```

---

# 42. Common Misconceptions

## "Objects are copied when assigned."

False.

```js
const b = a;
```

creates another reference to the same object.

---

## "`const` makes objects immutable."

False.

It makes the binding non-reassignable.

---

## "`Object.freeze()` recursively freezes nested objects."

False.

---

## "Spread creates independent nested state."

False.

It is shallow.

---

## "Deep equality is built into JavaScript objects."

Not as a general `===` operation.

---

## "Domain equality should always use `===`."

False.

Some domain concepts require value-based equality.

---

## "Immutability always means no performance cost."

False.

Copies can cost CPU and memory, while uncontrolled mutation can create other costs.

---

## "All data should be deeply cloned."

False.

Deep copies can be expensive, semantically wrong, or unnecessary.

---

# 43. Common Mistakes

```text
[ ] returning internal arrays directly
[ ] storing caller-owned objects without a boundary contract
[ ] assuming spread isolates nested state
[ ] using JSON serialization as universal cloning
[ ] mutating value objects after using them as keys/cache inputs
[ ] implementing equals using the wrong fields
[ ] confusing business identity with JavaScript object identity
[ ] assuming Object.freeze means deep immutability
[ ] copying huge graphs without measurement
[ ] hiding aliasing from API documentation
[ ] allowing mutable objects to cross trust boundaries blindly
```

---

# 44. Comparison With Related Concepts

| Concept | Identity | Typical mutation model | Equality |
|---|---|---|---|
| Plain object | JavaScript identity | mutable by default | reference-based with `===` |
| Entity | business identity | often mutable | domain identity |
| Value Object | secondary/irrelevant | often immutable | value-based |
| DTO | transport representation | usually simple/controlled | usually structural/application-defined |
| Event | occurrence identity may exist | usually immutable after publication | domain/event semantics |
| Cache entry | resource/key context | controlled | key-based |
| Snapshot | point-in-time representation | immutable by contract | value/state-based |

---

# 45. Performance Considerations

Copying has a cost.

If:

```text
N = number of reachable copied values
```

then a full traversal is generally at least related to:

```text
O(N)
```

work in the size of the copied graph, ignoring implementation details.

Repeated copying can create:

```text
CPU overhead
allocation pressure
garbage collection pressure
latency.
```

But mutation also creates costs:

```text
debugging complexity
coordination
cache invalidation
concurrent-state hazards
```

Therefore:

> Optimize the ownership model, not merely the number of copies.

---

# 46. Memory Considerations

A shallow copy may be cheap:

```text
new outer object
+ reused nested references
```

A deep copy can duplicate a large graph.

Consider:

```text
10 MB configuration graph
```

copied 100 times.

That can create substantial memory pressure.

Conversely, sharing a 10 MB object can retain it for longer than intended if many long-lived references exist.

The trade-off is:

```text
copying
vs
sharing
vs
lifetime control.
```

---

# 47. Security Considerations

Shared mutable state can cross security boundaries.

Dangerous patterns include:

```text
untrusted caller receives internal state
untrusted input object becomes trusted internal state
mutable configuration is shared across tenants
cached objects are returned without ownership control
```

Security-sensitive design should prefer clear boundaries:

```text
validate
normalize
own
restrict
expose only what is required.
```

---

# 48. Production Usage

In production, document ownership when it is non-obvious.

Good API questions:

```text
Does this method retain the object I pass?
Can I continue mutating it?
Does this method return internal state?
Do I own the returned object?
Is this object immutable?
What equality semantics apply?
```

For large systems, these rules should appear in:

```text
type contracts
domain rules
API documentation
tests
code review standards.
```

---

# 49. Implementation From Scratch

## Exercise 1 — Equality Matrix

Implement a helper that compares two values using:

```text
===
Object.is
```

and report where they differ.

Test:

```text
NaN
0
-0
objects
same references
```

---

## Exercise 2 — Shallow Copy Detector

Implement:

```js
function sharesNestedReference(original, copy, key) {
  // return whether the specified nested reference is shared
}
```

---

## Exercise 3 — Defensive Collection

Build:

```js
class TaskList {
  #tasks = [];

  add(task) {}
  remove(id) {}
  getAll() {}
}
```

Requirements:

```text
caller cannot directly replace #tasks
caller cannot mutate the internal array through getAll()
```

Then discuss what happens when `task` itself is mutable.

---

## Exercise 4 — Immutable Value Object

Implement:

```js
class Money {
  constructor(amount, currency) {}
  equals(other) {}
  add(other) {}
}
```

Requirements:

```text
instances cannot be mutated after construction
add returns a new Money
currency mismatch is rejected
equals is value-based
```

---

## Exercise 5 — Deep Freeze

Implement a deep-freeze utility.

Requirements:

```text
nested objects
arrays
symbol keys
cycles
already-frozen objects
```

Then explain the cases your implementation intentionally does not support.

---

## Exercise 6 — Reference Graph Printer

Given a graph of objects, generate a visualization-friendly representation while avoiding infinite recursion from cycles.

---

# 50. Debugging Exercises

## Debug 1 — Constructor Aliasing

```js
class Basket {
  constructor(items) {
    this.items = items;
  }
}

const source = [];
const basket = new Basket(source);

source.push("gold");

console.log(basket.items);
```

Identify:

```text
ownership
alias
defect or intentional behavior
```

---

## Debug 2 — Getter Leak

```js
class Basket {
  #items = [];

  add(item) {
    this.#items.push(item);
  }

  getItems() {
    return this.#items;
  }
}

const basket = new Basket();

basket.add("gold");

const items = basket.getItems();
items.length = 0;

console.log(basket.getItems());
```

Identify the encapsulation failure.

---

## Debug 3 — Value Object Mutation

```js
const money = {
  amount: 100,
  currency: "INR",
};

const cacheKey = `${money.amount}:${money.currency}`;

money.amount = 200;

console.log(cacheKey);
console.log(`${money.amount}:${money.currency}`);
```

Explain why cached identity/value assumptions can diverge.

---

## Debug 4 — Accidental Shared Defaults

```js
const DEFAULT_SETTINGS = {
  permissions: [],
};

function createUserSettings() {
  return {
    ...DEFAULT_SETTINGS,
  };
}

const a = createUserSettings();
const b = createUserSettings();

a.permissions.push("admin");

console.log(b.permissions);
```

Find the shallow-copy bug.

---

# 51. Code Review Exercise

Review:

```js
class Order {
  constructor(items, customer) {
    this.items = items;
    this.customer = customer;
  }

  getItems() {
    return this.items;
  }

  getCustomer() {
    return this.customer;
  }
}
```

Answer:

```text
1. Who owns items?
2. Who owns customer?
3. Are callers allowed to mutate them?
4. Does the Order protect invariants?
5. Should it copy?
6. What should it return?
7. What happens if customer is shared by many orders?
8. Which objects are entities?
9. Which objects could be value objects?
10. What is the performance trade-off?
```

---

# 52. Interview Questions

## Fundamentals

```text
1. Why does {} === {} return false?
2. What does const actually guarantee?
3. What is aliasing?
4. What is the difference between mutation and replacement?
5. What is shallow copying?
6. What is deep copying?
7. Why is Object.freeze shallow?
```

## Equality

```text
8. What is the difference between === and Object.is?
9. Why is NaN special?
10. Why does Object.is(0, -0) return false?
11. What is SameValueZero?
12. Where is SameValueZero used?
```

## Design

```text
13. What is defensive copying?
14. When should you not defensively copy?
15. How would you protect an internal array?
16. Why are value objects often immutable?
17. What is the difference between JavaScript identity and domain identity?
18. What is the difference between an entity and a value object?
19. How can mutability break equality reasoning?
20. How would you design equality for a Money class?
```

## Principal-level

```text
21. Would you choose immutable or mutable state for a workflow object? Defend it.
22. When does copying become harmful?
23. How can aliasing create hidden coupling?
24. How would ownership rules change across service boundaries?
25. When can object interning improve a design, and when can it make it worse?
```

---

# 53. Predict-the-Output Exercises

## Exercise A

```js
const a = {};
const b = a;

console.log(a === b);
console.log(Object.is(a, b));
```

---

## Exercise B

```js
console.log(NaN === NaN);
console.log(Object.is(NaN, NaN));
console.log([NaN].includes(NaN));
```

---

## Exercise C

```js
console.log(0 === -0);
console.log(Object.is(0, -0));
```

---

## Exercise D

```js
const source = {
  nested: {
    value: 1,
  },
};

const copy = { ...source };

copy.nested.value = 99;

console.log(source.nested.value);
console.log(source.nested === copy.nested);
```

---

## Exercise E

```js
const original = {
  nested: {
    value: 1,
  },
};

const copy = structuredClone(original);

copy.nested.value = 99;

console.log(original.nested.value);
console.log(original.nested === copy.nested);
```

---

## Exercise F

```js
const a = {
  value: 1,
};

const b = a;

a = {
  value: 2,
};
```

Before running, explain why this program is invalid.

---

# 54. Mastery Exercises

## Level 1 — Understand

Explain:

```text
identity
equality
mutability
aliasing
ownership
```

without using memorized definitions.

---

## Level 2 — Explain

Explain:

```text
=== vs Object.is vs SameValueZero
```

and give an example where the distinction matters.

---

## Level 3 — Predict

Predict the output of unfamiliar code involving:

```text
aliasing
shallow copies
nested mutation
freeze
NaN
signed zero
```

---

## Level 4 — Implement

Build:

```text
defensive collection
immutable Money value object
cycle-safe deep freeze
reference graph inspector.
```

---

## Level 5 — Debug

Find and fix:

```text
a mutation leak
a shared-default bug
a shallow-copy bug
a mutable value-object bug.
```

---

## Level 6 — Apply

Design an:

```text
Order
```

with explicit decisions for:

```text
items
customer
shipping address
money values
ownership
mutation
equality.
```

---

## Level 7 — Compare

Defend:

```text
defensive copy
vs
immutable object
vs
read-only API
vs
shared mutable object.
```

---

## Level 8 — Defend

Answer:

> Why is ownership a more useful design question than simply asking whether an object is mutable?

Your answer must connect:

```text
identity
references
aliasing
encapsulation
invariants
performance
LLD.
```

---

# 55. Key Takeaways

```text
1. JavaScript object identity is not deep value equality.
2. === compares object identity for objects.
3. Object.is implements a different equality relation.
4. SameValueZero is another equality relation used by several APIs.
5. const does not make objects immutable.
6. Mutation preserves identity; replacement creates another identity.
7. Aliasing makes shared mutation observable.
8. Spread and Object.assign are shallow copies.
9. Deep cloning has data-model and semantic limitations.
10. JSON serialization is not a universal cloning strategy.
11. Defensive copying is an ownership decision.
12. Immutability can exist at multiple depths and contractual levels.
13. Entities usually care about identity.
14. Value objects usually care about value equality.
15. Domain equality must be designed explicitly.
16. Mutable equality-defining state can make reasoning difficult.
17. Shared object graphs create hidden coupling.
18. Cycles make naive recursive copying/traversal unsafe.
19. Copying and sharing both have costs.
20. Strong LLD starts by deciding ownership and identity semantics.
```

---

# 56. Concept Connections

## Depends On

```text
Chapter 1 — JavaScript Object Model — Foundation
references
objects
property access
assignment
```

## Builds Toward

```text
Chapter 3 — Prototype Chain Deep Dive
Chapter 4 — Constructor Functions & Instance Construction
Chapter 5 — JavaScript Classes Internally
Chapter 8 — Encapsulation
Chapter 9 — Abstraction
Chapter 11 — Composition
Chapter 17 — Cohesion & Coupling
Chapter 19+ — GRASP
Chapter 45+ — TypeScript Object Design
Chapter 120+ — DDD Value Objects & Entities
Chapter 150+ — Persistence Identity
Chapter 200+ — Transactions & Consistency
Chapter 400+ — Enterprise LLD
```

## Related Concepts

```text
immutability
persistent data structures
copy-on-write
entity identity
value equality
caching
memoization
identity maps
serialization
```

## Why This Chapter Matters

Many LLD bugs are not actually pattern mistakes.

They are ownership mistakes:

```text
wrong object owns state
wrong reference escapes
wrong equality assumption
wrong mutation boundary
wrong copy strategy.
```

When those are correct, later patterns become much easier to evaluate.

---

# 57. Completion Criteria

Mark:

```text
[~] In Progress
```

when you can explain the examples but still need reference material.

Mark:

```text
[+] Completed
```

when you can:

```text
explain identity
compare equality relations
trace aliasing
distinguish mutation from replacement
explain shallow/deep copying
design defensive boundaries
explain entity vs value semantics.
```

Mark:

```text
[*] Mastered
```

only when you can look at unfamiliar object code and determine:

```text
who owns the state
who aliases it
whether equality is identity or value based
whether copying is required
whether mutation is safe
what invariant could be violated.
```

Reading alone does not mark mastery.

---

# 58. Revision / Retrieval Record

```md
# Chapter 2 — Revision / Retrieval Record

## Attempt
- Date:
- Duration:
- Status before:
- Status after:

## Identity
- What does identity mean?
- How is identity different from business identity?

## Equality
- ===:
- Object.is:
- SameValueZero:

## Mutability
- What is mutable?
- What does const guarantee?
- What does Object.freeze guarantee?

## Aliasing
- Where did two references share state?
- What bug resulted?

## Copying
- Shallow:
- Deep:
- Why was a particular strategy chosen?

## Ownership
- Who owns the data?
- Who may mutate it?
- Who may retain references?

## DDD Connection
- Entity:
- Value Object:

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

---

# 59. Canonical References and Source Discipline

Use the specification as the semantic source of truth for equality operations and object behavior.

Recommended canonical references:

```text
ECMAScript Language Specification
https://tc39.es/ecma262/

MDN — Equality comparisons and sameness
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Equality_comparisons_and_sameness

MDN — Object.is
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is

MDN — structuredClone
https://developer.mozilla.org/en-US/docs/Web/API/Window/structuredClone

MDN — Object.freeze
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze

MDN — Object.assign
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign
```

Source discipline:

```text
language equality semantics
→ ECMAScript

environment support/details
→ host/runtime documentation

engine optimization
→ engine-specific documentation/source

domain equality
→ application requirements and domain rules
```

Do not confuse a domain equality rule with a JavaScript language rule.

---

# 60. Completion Snapshot

```text
Chapter: 002
Title: Object Identity, Equality & Mutability

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

For every object, reason through this chain:

```text
OBJECT
  │
  ├── IDENTITY
  │
  ├── STATE
  │      └── mutable / immutable
  │
  ├── REFERENCES
  │      └── aliases / boundaries
  │
  ├── EQUALITY
  │      ├── language identity
  │      └── domain value equality
  │
  └── OWNERSHIP
         ├── who creates?
         ├── who mutates?
         ├── who retains references?
         └── who may observe state?
```

Then ask:

```text
1. Is this concept identity-based or value-based?
2. Can its state change?
3. Who owns that state?
4. Who else can reference it?
5. Should state be shared, copied, or made immutable?
6. What equality semantics does the domain require?
7. What happens if two parts of the system observe different versions?
```

---

# Principal Design Principle

> **Do not choose mutation, copying, or immutability as a style preference. Choose them as an ownership and correctness strategy.**

---

# Track Mapping

```text
Track A — Core Theory
    identity
    equality relations
    mutability
    aliasing
    copying
    ownership

Track B — Implementation
    defensive APIs
    immutable value objects
    deep-freeze utility
    graph inspection
    boundary design

Track C — Interview / Reasoning
    prediction
    equality edge cases
    mutation debugging
    entity vs value-object reasoning
    trade-off defense
```
