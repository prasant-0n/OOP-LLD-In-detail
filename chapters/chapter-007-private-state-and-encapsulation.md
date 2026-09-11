# Chapter 7 — Private State & Encapsulation

> **JavaScript OOP + LLD Mastery**
>
> Encapsulation is not simply “make fields private.” It is the design discipline of controlling **what an object exposes, what state it owns, who can mutate that state, which invariants it protects, and which implementation details callers must not depend on**.
>
> This chapter studies JavaScript encapsulation through private fields, closures, modules, methods, accessors, defensive copying, capability boundaries, controlled mutation, and invariant-preserving APIs.

**Status:** `[ ] Not Started`

# 1. Learning Objectives

By the end of this chapter, you should be able to:

```text
[ ] define encapsulation precisely
[ ] distinguish encapsulation from information hiding
[ ] distinguish private state from public state
[ ] explain why access control exists
[ ] explain #private fields
[ ] explain private methods/accessors
[ ] explain closure-based encapsulation
[ ] explain module-based encapsulation
[ ] explain naming-convention privacy
[ ] compare #private fields with closures
[ ] compare #private fields with Symbols
[ ] compare private state with WeakMap-based state
[ ] define an object API boundary
[ ] identify accidental state exposure
[ ] identify mutation leaks
[ ] identify representation leaks
[ ] protect collection state
[ ] preserve invariants through methods
[ ] reason about getters and setters
[ ] explain why public setters can weaken invariants
[ ] design command-style mutation APIs
[ ] design query APIs
[ ] explain read-only vs immutable
[ ] explain defensive copying
[ ] reason about capability exposure
[ ] distinguish encapsulation from security
[ ] identify abstraction leaks
[ ] review encapsulation quality
[ ] debug private-state failures
[ ] implement encapsulated objects
[ ] defend encapsulation trade-offs
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
objects
references
prototype
classes
private fields
mutability
ownership
```

# 3. What Is It?

Encapsulation is the practice of designing an object so that:

```text
internal representation
+
state ownership
+
invariant management
```

are controlled through a deliberate public interface.

A useful model:

```text
                 OBJECT BOUNDARY
            ┌────────────────────────┐
CALLER ───→ │ public API             │
            │                        │
            │ invariant protection   │
            │                        │
            │ private/internal state │
            └────────────────────────┘
```

Encapsulation answers:

```text
What can callers ask?
What can callers change?
How can they change it?
What can callers observe?
What implementation details can they depend on?
```

# 4. Why Does It Exist?

Suppose:

```js
class BankAccount {
  balance = 1000;
}
```

Then external code can do:

```js
account.balance = -500000;
```

If the domain requires:

```text
balance >= 0
```

the representation has escaped the object's control.

A more encapsulated design:

```js
class BankAccount {
  #balance = 1000;

  withdraw(amount) {
    if (amount > this.#balance) {
      throw new Error("Insufficient funds");
    }

    this.#balance -= amount;
  }
}
```

Now state changes through a controlled operation.

The important benefit is not simply:

```text
field is hidden.
```

It is:

```text
the object owns the rule governing valid state transitions.
```

# 5. Mental Model

Think of an encapsulated object as a boundary:

```text
             EXTERNAL CODE
                   │
          ┌────────▼────────┐
          │ PUBLIC INTERFACE│
          ├─────────────────┤
          │ commands        │
          │ queries         │
          │ events          │
          ├─────────────────┤
          │ INTERNAL RULES  │
          ├─────────────────┤
          │ private state   │
          │ collaborators   │
          │ representation  │
          └─────────────────┘
```

The caller should interact with:

```text
meaningful capabilities
```

rather than:

```text
internal representation.
```

# 6. Core Rules

## Rule 1 — Encapsulation Is Bigger Than Private Fields

An object can have private fields and still be poorly encapsulated.

Example:

```js
getItems() {
  return this.#items;
}
```

The field is private, but the mutable array has escaped.

---

## Rule 2 — A Public Field Is Part of the API

If callers can write:

```js
object.status = "anything";
```

then that assignment capability is part of the object's public contract.

Changing it later can become a breaking change.

---

## Rule 3 — Protect Invariants, Not Just Variables

Bad:

```js
#balance;
```

Good:

```text
#balance
+
withdraw
+
deposit
+
validity rules.
```

Encapsulation is valuable because rules live near the state they govern.

---

## Rule 4 — Returning Internal Mutable State Is a Leak

```js
getItems() {
  return this.#items;
}
```

allows:

```text
caller → internal array
```

and destroys the intended boundary.

---

## Rule 5 — Read-Only Is Not the Same as Immutable

Returning:

```text
readonly view
```

or exposing no mutator is not automatically deep immutability.

The underlying objects may still be mutable.

---

## Rule 6 — Encapsulation Controls Dependencies

A caller should depend on:

```text
public behavior
```

rather than:

```text
private representation.
```

This reduces change propagation.

# 7. Syntax

## Private Instance Field

```js
class Account {
  #balance = 0;
}
```

## Private Method

```js
class Account {
  #validateAmount(amount) {
    return Number.isFinite(amount) && amount >= 0;
  }
}
```

## Private Getter

```js
class Account {
  get #balanceValue() {
    return this.#balance;
  }
}
```

## Public Method

```js
class Account {
  deposit(amount) {}
}
```

# 8. Basic Examples

## Example 1 — Public Representation Leak

```js
class Cart {
  items = [];

  add(item) {
    this.items.push(item);
  }
}
```

The caller can bypass:

```js
cart.items.push(null);
```

The object cannot enforce item rules consistently.

---

## Example 2 — Private State

```js
class Cart {
  #items = [];

  add(item) {
    this.#items.push(item);
  }

  size() {
    return this.#items.length;
  }
}
```

The representation is internal.

---

## Example 3 — Controlled Transition

```js
class Account {
  #balance = 0;

  deposit(amount) {
    if (!Number.isFinite(amount) || amount <= 0) {
      throw new RangeError("Amount must be positive");
    }

    this.#balance += amount;
  }

  getBalance() {
    return this.#balance;
  }
}
```

The invariant is enforced at the mutation boundary.

# 9. Execution Walkthrough

Consider:

```js
class Cart {
  #items = [];

  add(item) {
    this.#items.push(item);
  }

  getItems() {
    return [...this.#items];
  }
}
```

Flow:

```text
1. Construct Cart.
2. Initialize private #items.
3. add(item) accesses private state through the class.
4. push changes internal state.
5. getItems creates a new outer array.
6. Caller receives the copy.
7. Caller cannot directly mutate #items through that array reference.
```

The boundary is:

```text
internal array
      ↓
copy
      ↓
external caller
```

# 10. Internal Mechanics

Private elements use language-level private-name semantics.

They are not equivalent to:

```js
this["_balance"]
```

and not equivalent to:

```js
this["#balance"]
```

The private name is associated with the class's private environment.

Access requires an appropriate private-brand-compatible receiver.

This gives the language a real distinction between:

```text
public property key
```

and:

```text
private class element.
```

# 11. ECMAScript / Specification Semantics

The ECMAScript specification defines private names/elements, including:

```text
private name creation
private fields
private methods
private accessors
private brand checks
initialization
```

Important design consequence:

```text
#balance
```

is not simply a convention the developer promises to respect.

It is language-enforced access syntax.

# 12. Advanced Behavior

## 12.1 Private Fields and Wrong Receivers

```js
class Account {
  #balance = 100;

  getBalance() {
    return this.#balance;
  }
}

const account = new Account();

console.log(account.getBalance());
```

works.

But:

```js
const getBalance = account.getBalance;

getBalance.call({});
```

throws because the receiver does not have the appropriate private state.

---

## 12.2 Private State Survives Public Key Inspection

```js
Object.keys(account);
Reflect.ownKeys(account);
Object.getOwnPropertyNames(account);
```

do not expose private names as ordinary property keys.

This is a language-access distinction, not a guarantee that runtime implementation memory is physically inaccessible.

---

## 12.3 Private Fields Do Not Use Prototype Lookup

Public properties can be inherited:

```text
receiver → prototype → next prototype
```

Private access is based on the class's private semantics rather than ordinary property-key lookup.

This distinction is important for inheritance reasoning.

# 13. Encapsulation vs Information Hiding

These terms overlap but are not identical.

## Encapsulation

Groups:

```text
state
behavior
rules
```

behind a boundary.

## Information Hiding

Reduces dependence on implementation details.

You can have:

```text
encapsulation without strong hiding
```

and:

```text
hidden implementation details through modules/functions
```

without a large class hierarchy.

The strongest designs often use both.

# 14. Encapsulation vs Abstraction

Encapsulation asks:

```text
What is inside the boundary?
What can access it?
```

Abstraction asks:

```text
What essential concept should the caller interact with?
```

Example:

```text
BankAccount
```

encapsulates:

```text
balance storage
transaction rules.
```

and abstracts:

```text
deposit
withdraw
available balance.
```

The two concepts reinforce one another but are not identical.

# 15. Encapsulation Techniques in JavaScript

Common techniques:

```text
public/private class fields
closures
modules
factory functions
WeakMap/WeakSet
Symbols
convention-based naming
restricted exports
defensive copies
immutable values
capability-limited APIs.
```

No technique is universally best.

# 16. Technique 1 — Private Fields

```js
class Account {
  #balance = 0;
}
```

Pros:

```text
language-enforced
readable
direct class support
good tool support
```

Cons:

```text
class-oriented
private names cannot be dynamically generated like ordinary keys
cannot be accessed by subclasses as ordinary public properties
```

# 17. Technique 2 — Closures

```js
function createCounter() {
  let count = 0;

  return {
    increment() {
      count += 1;
    },

    getCount() {
      return count;
    },
  };
}
```

`count` is inaccessible through ordinary object properties.

Pros:

```text
simple privacy
excellent factory composition
natural capability boundary.
```

Cons:

```text
closure-per-instance state/functions
different optimization/memory profile
no class private syntax
```

# 18. Technique 3 — Modules

```js
const secret = new Map();

export function add() {}

export function get() {}
```

The module can hide implementation details.

This is often the right answer when the hidden state belongs to:

```text
a subsystem
registry
service
module
```

rather than one object instance.

# 19. Technique 4 — WeakMap State

Conceptually:

```js
const privateState = new WeakMap();

class User {
  constructor(name) {
    privateState.set(this, {
      name,
    });
  }

  getName() {
    return privateState.get(this).name;
  }
}
```

This can associate hidden state with instances.

Trade-offs:

```text
more complex
indirect lookup
legacy/private-state patterns
```

Private fields are usually clearer for new class code unless WeakMap semantics are specifically useful.

# 20. Technique 5 — Symbols

```js
const secret = Symbol("secret");

const object = {
  [secret]: 123,
};
```

Symbols reduce accidental string-key collisions, but they are still object properties.

They can be discovered when the symbol is available through reflection.

Therefore:

```text
Symbol privacy
≠
private fields.
```

# 21. Technique 6 — Naming Conventions

```js
class Account {
  _balance = 0;
}
```

The underscore communicates:

```text
internal convention.
```

It does not enforce access restrictions.

This can be acceptable in simple internal codebases but should not be mistaken for language-level privacy.

# 22. Public API Design

A class should expose meaningful operations.

Bad:

```js
setBalance(value) {}
setStatus(value) {}
setItems(value) {}
```

This can turn the object into:

```text
mutable record with methods.
```

Better:

```js
deposit(amount) {}
withdraw(amount) {}
cancel() {}
addItem(item) {}
removeItem(id) {}
```

These methods express domain transitions.

# 23. Commands and Queries

A useful design distinction:

```text
Command
→ changes state

Query
→ observes state
```

Examples:

```text
deposit(amount) → command
withdraw(amount) → command
getBalance() → query
isActive() → query
```

A clear API can make mutation boundaries obvious.

# 24. Setters Can Weaken Encapsulation

```js
class Account {
  #balance = 0;

  set balance(value) {
    this.#balance = value;
  }
}
```

This preserves the field's privacy but exposes arbitrary replacement of the state.

If the domain says:

```text
balance changes only through transactions
```

then a public setter violates the conceptual boundary.

Private storage alone does not create a good design.

# 25. Getters Can Leak Representation

```js
get items() {
  return this.#items;
}
```

Even though the field is private, the getter gives the caller the same mutable reference.

A safer API may be:

```js
get items() {
  return [...this.#items];
}
```

But copying may not be enough when elements themselves are mutable.

# 26. Read-Only Views

Sometimes the goal is:

```text
caller may inspect
caller may not replace
```

Possible approaches:

```text
copy
immutable collection
iterator
projection object
query methods.
```

Choose based on:

```text
size
mutation behavior
performance
API semantics.
```

# 27. Iterator-Based Exposure

Instead of:

```js
getItems() {
  return [...this.#items];
}
```

a class may expose:

```js
*items() {
  yield* this.#items;
}
```

This can avoid copying the outer collection.

However, yielded elements may still be mutable.

Again:

```text
iteration control
≠
deep immutability.
```

# 28. Defensive Copying Revisited

Encapsulation and Chapter 2 connect directly.

When returning:

```text
mutable internal state
```

ask whether to:

```text
copy
freeze
wrap
project
iterate
avoid exposing.
```

There is no universal rule:

```text
always clone
```

or:

```text
never clone.
```

# 29. Representation Independence

A strong encapsulation boundary lets the internal representation change while the external contract remains stable.

For example:

```text
Version 1:
items = Array

Version 2:
items = Map

Version 3:
items = specialized indexed structure
```

If callers depend only on:

```text
addItem
removeItem
findItem
count
```

the representation can evolve.

If callers depend on:

```js
order.items[4]
```

the representation is exposed.

This is representation independence.

# 30. Information Leakage Through Types

Even when runtime state is private, a public type/API can leak design assumptions.

For example:

```ts
class Cart {
  get items(): Item[] { ... }
}
```

communicates:

```text
array representation
mutable element type
```

A more intentional contract might use:

```ts
readonly Item[]
```

or:

```text
Iterable<Item>
```

depending on the required capability.

Type-level design becomes important later in TypeScript.

# 31. Capability-Oriented Thinking

Instead of asking:

```text
Can caller access object?
```

ask:

```text
What capabilities does caller receive?
```

Example:

```text
full object
→ many capabilities

read-only projection
→ fewer capabilities

deposit-only interface
→ limited mutation

query interface
→ observation only.
```

This makes API design more precise.

# 32. Encapsulation and Dependency Injection

Suppose:

```js
class OrderService {
  constructor(database) {
    this.database = database;
  }
}
```

The service can encapsulate:

```text
how database operations are coordinated
```

without hiding the dependency itself.

This is a useful distinction:

```text
dependency visibility
≠
implementation leakage.
```

A dependency often should be explicit.

Its internal behavior does not need to be exposed.

# 33. Encapsulation and Composition

Composition naturally creates boundaries.

```js
class Checkout {
  constructor(paymentGateway, inventory) {
    this.paymentGateway = paymentGateway;
    this.inventory = inventory;
  }
}
```

The Checkout object controls:

```text
workflow coordination
```

while collaborators control:

```text
their own state and behavior.
```

Good encapsulation often emerges from assigning each object clear ownership.

# 34. Advanced Behavior — Encapsulating Collaborators

Consider:

```js
class Order {
  #pricing;

  constructor(pricing) {
    this.#pricing = pricing;
  }
}
```

The collaborator is private.

But privacy does not mean the collaborator itself is immutable.

External code may still hold the same reference:

```text
external ──→ pricing
             ↑
Order.#pricing
```

Therefore:

> Encapsulating a reference is not the same as taking ownership of the referenced object.

# 35. Ownership vs Encapsulation

A private field can answer:

```text
Who can access this field through this class syntax?
```

Ownership asks:

```text
Who controls the referenced object?
Who can mutate it?
Who retains aliases?
```

These are different.

Strong designs address both.

# 36. Advanced Behavior — Invariant-Preserving Methods

Suppose an Order has:

```text
status = "paid"
```

A generic setter:

```js
setStatus("cancelled")
```

may bypass transition rules.

A domain operation:

```js
cancel()
```

can enforce:

```text
paid orders cannot be cancelled
```

This is a more meaningful encapsulation boundary.

The object owns the valid transitions.

# 37. State Machines as Encapsulation

An object can encapsulate a state machine:

```text
draft
  ↓
submitted
  ↓
approved
  ↓
paid
```

Each command checks:

```text
current state
+
allowed transition
+
required conditions.
```

This pattern will later connect to:

```text
state modeling
workflow design
domain invariants.
```

# 38. Abstraction Leak

An abstraction leak occurs when callers must understand internal details to use the API correctly.

Examples:

```text
caller must know internal array indexes
caller must manually update two fields together
caller must call methods in fragile order
caller must know cache invalidation rules
caller must synchronize duplicated state.
```

Encapsulation reduces these leaks.

# 39. Law of Demeter Connection

A common smell:

```js
order.customer.address.city
```

This can create long knowledge chains.

Deep navigation is not automatically wrong, but it may indicate:

```text
leaky boundaries
high coupling
anemic abstractions.
```

Later GRASP/LLD chapters will explore this in depth.

# 40. Encapsulation and Anemic Domain Models

An object with only:

```text
public fields
getters
setters
```

may lack meaningful behavior.

Example:

```js
class Order {
  items = [];
  status = "draft";
}
```

with external code deciding every valid transition.

This can become an:

```text
anemic domain model.
```

Encapsulation encourages placing relevant behavior near the state it protects.

# 41. Advanced Behavior — Encapsulate Rules, Not Every Detail

Do not make every field private just for appearance.

Good encapsulation asks:

```text
Which state needs protection?
Which invariant depends on it?
Which collaborator should own the rule?
Which representation may change?
Which capability should be exposed?
```

Over-encapsulation can also create:

```text
unnecessary wrappers
boilerplate
awkward APIs
debugging friction.
```

# 42. Edge Cases

## Private Field Access

```js
obj["#value"]
```

does not access:

```text
#value
```

---

## Object Spread

Private state is not copied as ordinary enumerable properties:

```js
const copy = { ...object };
```

This does not transfer the object's private class state.

---

## Object.assign

Similarly:

```js
Object.assign({}, instance);
```

does not copy private elements as ordinary properties.

---

## JSON Serialization

Private fields do not become ordinary JSON properties simply because they exist on the instance.

---

## Public Method Leakage

A public method can still intentionally expose private state:

```js
getState() {
  return this.#state;
}
```

Private declaration alone does not prevent representation leaks.

# 43. Common Misconceptions

```text
"Private fields automatically create good encapsulation."
"Private means secure."
"Private means immutable."
"Returning a getter is always safe."
"readonly means deep immutable."
"Closure state is always slower."
"Symbols are equivalent to private fields."
"Encapsulation means hiding everything."
"Every field should be private."
"Setters are always harmless."
"Private collaborators are owned collaborators."
```

# 44. Common Mistakes

```text
[ ] exposing internal arrays
[ ] exposing mutable nested objects
[ ] adding unrestricted setters
[ ] duplicating state outside the owner
[ ] putting domain rules in callers
[ ] using private fields as a security boundary
[ ] hiding required dependencies
[ ] overusing getters as data dumps
[ ] creating anemic models
[ ] over-encapsulating trivial immutable data
[ ] confusing access control with ownership
```

# 45. Comparison With Related Concepts

| Technique / Concept | What it controls | Main trade-off |
|---|---|---|
| `#private` | language-level access | class-specific |
| Closure | lexical access | per-instance closure structure |
| Module | subsystem-level visibility | broader scope |
| WeakMap | hidden association | indirection/complexity |
| Symbol | key collision/access convention | not true private syntax |
| `_name` convention | developer discipline | no enforcement |
| Defensive copy | aliasing | copy cost |
| Immutable value | mutation | creation/replacement cost |
| Read-only API | exposed capability | underlying graph may remain mutable |
| Encapsulated command | valid state transition | more domain methods |

# 46. Performance Considerations

Encapsulation choices can affect:

```text
allocation
copying
method calls
closure creation
object shape
serialization
collection exposure.
```

Example:

```js
getItems() {
  return [...this.#items];
}
```

costs proportional to the copied collection size.

An iterator can avoid copying the outer array but may expose element references.

Private fields can have highly optimized engine implementations, but exact costs are engine-specific.

Do not sacrifice a correct boundary based on unmeasured micro-optimizations.

# 47. Memory Considerations

Compare:

```text
private array
vs
copied array
vs
closure state
vs
WeakMap state.
```

Memory effects depend on:

```text
number of instances
state size
lifetime
aliasing
copy frequency
```

A static registry can create long-lived retention even though it is private.

# 48. Security Considerations

Encapsulation reduces accidental access but is not equivalent to security.

Security requires:

```text
authentication
authorization
validation
trusted boundaries
secret management
least privilege
```

Use encapsulation to reduce the attack surface of ordinary application code.

Do not assume:

```text
private field = protected secret.
```

# 49. Production Usage

A production object should expose:

```text
meaningful capabilities
queries
valid state transitions
```

rather than its entire storage structure.

Prefer:

```text
deposit()
withdraw()
approve()
cancel()
addItem()
removeItem()
```

over:

```text
setBalance()
setStatus()
setItems()
```

when domain rules exist.

# 50. Implementation From Scratch

## Exercise 1 — Encapsulated Counter

Implement:

```js
class Counter {
  #value = 0;

  increment();
  decrement();
  value();
}
```

Requirements:

```text
no public writable value
valid transitions
predictable API.
```

## Exercise 2 — Encapsulated Cart

Implement:

```text
addItem
removeItem
contains
size
getItems
```

Requirements:

```text
no internal-array leak
```

Then discuss nested mutable items.

## Exercise 3 — Bank Account

Implement:

```text
#balance
deposit
withdraw
getBalance
```

Invariants:

```text
positive deposits
no overdraft
finite amounts.
```

## Exercise 4 — Closure Equivalent

Reimplement BankAccount using:

```js
function createBankAccount() {}
```

Compare:

```text
privacy
identity
memory
testing
extensibility.
```

## Exercise 5 — Read-Only Projection

Build:

```text
Order
OrderView
```

where callers can inspect:

```text
id
status
total
```

but cannot mutate Order state.

## Exercise 6 — Capability-Limited Interface

Design:

```text
BillingReader
BillingOperator
```

where the first can query and the second can perform controlled mutations.

# 51. Debugging Exercises

## Debug 1 — Array Leak

```js
class Cart {
  #items = [];

  add(item) {
    this.#items.push(item);
  }

  getItems() {
    return this.#items;
  }
}

const cart = new Cart();
cart.add("gold");

cart.getItems().length = 0;

console.log(cart.getItems());
```

Identify the representation leak.

## Debug 2 — Setter Leak

```js
class Account {
  #balance = 100;

  set balance(value) {
    this.#balance = value;
  }
}
```

Explain why privacy did not preserve the domain invariant.

## Debug 3 — Nested Mutable Leak

```js
class User {
  #profile;

  constructor(profile) {
    this.#profile = { ...profile };
  }

  getProfile() {
    return this.#profile;
  }
}

const profile = { preferences: { theme: "dark" } };
const user = new User(profile);

user.getProfile().preferences.theme = "light";

console.log(user.getProfile().preferences.theme);
```

Identify why the outer defensive copy was insufficient.

## Debug 4 — Hidden Collaborator Alias

```js
class Service {
  #config;

  constructor(config) {
    this.#config = config;
  }
}
```

Explain why `#config` does not necessarily mean the Service owns the configuration.

# 52. Code Review Exercise

Review:

```js
class Order {
  #items = [];
  #status = "draft";

  get items() {
    return this.#items;
  }

  set status(value) {
    this.#status = value;
  }

  addItem(item) {
    this.#items.push(item);
  }
}
```

Evaluate:

```text
1. Is items encapsulated?
2. Is status transition-controlled?
3. What capabilities are exposed?
4. Could callers violate invariants?
5. Should there be add/remove commands?
6. Should status be a state machine?
7. Should items be copied or projected?
8. What does Order actually own?
```

# 53. Interview Questions

```text
1. What is encapsulation?
2. Is encapsulation the same as private fields?
3. What is information hiding?
4. How do #private fields work conceptually?
5. How are private fields different from Symbols?
6. How are private fields different from closures?
7. Why can returning a private array still leak state?
8. What is a representation leak?
9. What is a mutation leak?
10. What is the difference between read-only and immutable?
11. Why can setters weaken encapsulation?
12. How does encapsulation protect invariants?
13. What is an abstraction leak?
14. What is an anemic domain model?
15. What is capability-oriented API design?
16. Does private state imply ownership?
17. How would you encapsulate a collection?
18. When should a field remain public?
19. When is a factory better than a class?
20. Why is encapsulation not the same as security?
```

# 54. Predict-the-Output Exercises

## Exercise A

```js
class Account {
  #balance = 100;

  deposit(amount) {
    this.#balance += amount;
  }

  getBalance() {
    return this.#balance;
  }
}

const account = new Account();

account.deposit(50);

console.log(account.getBalance());
```

## Exercise B

```js
class User {
  #name = "Milan";

  getName() {
    return this.#name;
  }
}

const user = new User();

console.log(user["#name"]);
console.log(user.getName());
```

## Exercise C

```js
class Cart {
  #items = [];

  add(item) {
    this.#items.push(item);
  }

  getItems() {
    return [...this.#items];
  }
}

const cart = new Cart();

cart.add("gold");

const items = cart.getItems();
items.push("silver");

console.log(cart.getItems().length);
```

## Exercise D

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

## Exercise E

```js
class Base {
  #value = 10;

  getValue() {
    return this.#value;
  }
}

class Child extends Base {}

const child = new Child();

console.log(child.getValue());
```

Explain why inherited behavior can access Base private state without exposing the private name as an ordinary public property.

# 55. Mastery Exercises

## Level 1 — Understand

Explain:

```text
encapsulation
information hiding
private state
ownership
invariant
representation leak
```

## Level 2 — Explain

Explain why:

```text
private field
```

does not automatically mean:

```text
well-encapsulated object.
```

## Level 3 — Predict

Predict:

```text
private access
array leaks
nested mutation
setter behavior
wrong receivers
```

## Level 4 — Implement

Build:

```text
encapsulated counter
bank account
cart
read-only projection
capability-limited interface.
```

## Level 5 — Debug

Fix:

```text
internal collection leak
unrestricted setter
nested mutable leak
shared collaborator ownership ambiguity.
```

## Level 6 — Apply

Design an:

```text
Order
```

where:

```text
status
items
total
customer reference
```

have explicit ownership and mutation rules.

## Level 7 — Compare

Compare:

```text
#private
closure
module
WeakMap
Symbol
naming convention
```

and choose one for three different use cases.

## Level 8 — Defend

Answer:

> What exactly are you trying to protect when you encapsulate a field?

A strong answer must cover:

```text
state
invariants
representation
capabilities
ownership
changeability
coupling.
```

# 56. Key Takeaways

```text
1. Encapsulation is a boundary around state, behavior and invariants.
2. Private fields are one technique, not the definition of encapsulation.
3. Public fields become part of the API contract.
4. Returning internal mutable state can leak representation.
5. Defensive copies control aliases but cost resources.
6. Read-only is not necessarily immutable.
7. Setters can bypass meaningful domain transitions.
8. Commands often provide stronger boundaries than generic setters.
9. Queries should expose observations rather than mutable representation.
10. Private fields enforce language-level access rules.
11. Symbols are properties, not private fields.
12. Closures provide a different privacy and ownership model.
13. Modules can encapsulate subsystem-level state.
14. WeakMap can associate hidden state with object identities.
15. Private collaborators are not automatically owned collaborators.
16. Encapsulation and ownership are related but distinct.
17. Good encapsulation supports representation independence.
18. Abstraction leaks increase coupling.
19. Anemic models often expose too much state and too little behavior.
20. Encapsulation is not the same as security.
21. Strong objects own their rules and valid state transitions.
```

# 57. Concept Connections

## Depends On

```text
Chapter 1 — JavaScript Object Model — Foundation
Chapter 2 — Object Identity, Equality & Mutability
Chapter 3 — Prototype Chain Deep Dive
Chapter 4 — Constructor Functions & Instance Construction
Chapter 5 — JavaScript Classes Internally
Chapter 6 — Class Fields & Initialization Semantics
private fields
mutability
ownership
object identity
```

## Builds Toward

```text
Chapter 8 — Abstraction
Chapter 9 — Inheritance
Chapter 10 — Polymorphism
Chapter 11 — Composition
Chapter 17 — Cohesion & Coupling
GRASP
SOLID
TypeScript visibility
TypeScript structural contracts
domain entities
value objects
aggregates
repositories
service boundaries
```

## Related Concepts

```text
information hiding
representation independence
capabilities
immutability
defensive copying
state machines
Law of Demeter
anemic domain models
dependency injection
```

## Why This Chapter Matters

A well-designed object should not require every caller to understand:

```text
how state is stored
which fields must change together
which transitions are valid
which representations may evolve
```

Encapsulation turns those rules into object responsibilities.

# 58. Completion Criteria

Mark:

```text
[+] Completed
```

when you can:

```text
define encapsulation
protect a mutable collection
design invariant-preserving commands
distinguish private access from ownership
identify representation leaks
compare encapsulation techniques
```

Mark:

```text
[*] Mastered
```

when you can inspect an unfamiliar object and identify:

```text
public capabilities
private state
mutation paths
aliasing paths
invariant boundaries
representation leaks
abstraction leaks
ownership ambiguities
```

and redesign the API without adding unnecessary abstraction.

Reading alone does not mark mastery.

# 59. Revision / Retrieval Record

```md
# Chapter 7 — Revision / Retrieval Record

## Attempt
- Date:
- Duration:
- Status before:
- Status after:

## Encapsulation
- Definition:
- What state is protected?
- Which invariants are protected?

## Private Mechanisms
- #private:
- Closure:
- Module:
- WeakMap:
- Symbol:
- Convention:

## API Boundary
- Commands:
- Queries:
- Exposed capabilities:

## Leakage
- Representation leak:
- Mutation leak:
- Abstraction leak:

## Ownership
- Who owns each mutable object?
- Which aliases remain outside?

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

# 60. Canonical References and Source Discipline

Primary source:

```text
ECMAScript Language Specification
https://tc39.es/ecma262/
```

Useful references:

```text
MDN — Private elements
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_elements

MDN — Classes
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes

MDN — Closures
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures

MDN — Modules
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules
```

Source discipline:

```text
private-element semantics
→ ECMAScript

JavaScript API behavior
→ standard documentation

engine implementation
→ engine-specific sources

encapsulation quality
→ design/domain requirements
```

# 61. Completion Snapshot

```text
Chapter: 007
Title: Private State & Encapsulation

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
                    OBJECT BOUNDARY
             ┌────────────────────────┐
             │ PUBLIC CAPABILITIES    │
             │                        │
CALLER ────→ │ commands               │
             │ queries                │
             │ controlled views       │
             ├────────────────────────┤
             │ INVARIANTS             │
             │ valid transitions      │
             │ ownership rules        │
             ├────────────────────────┤
             │ INTERNAL REPRESENTATION│
             │ #private state         │
             │ collaborators          │
             │ collections            │
             └────────────────────────┘
```

When reviewing an object, ask:

```text
1. What state does it own?
2. What invariants does it protect?
3. Which mutations are allowed?
4. Which capabilities are exposed?
5. Can a mutable reference escape?
6. Which implementation details can change without breaking callers?
7. Does the object expose rules or just storage?
8. Does private access actually correspond to ownership?
9. Is the boundary simpler than the problem it solves?
10. Would a module, closure, composition boundary, or value object be better?
```

# Principal Design Principle

> **Encapsulation is successful when callers depend on what an object means and can do, rather than on how its state happens to be represented.**

# Track Mapping

```text
Track A — Core Theory
    encapsulation
    information hiding
    private elements
    invariant protection
    capability boundaries
    representation independence

Track B — Implementation
    private-field classes
    closure objects
    WeakMap state
    defensive collection APIs
    read-only projections
    state-transition objects

Track C — Interview / Reasoning
    privacy vs ownership
    setters vs commands
    representation leaks
    abstraction leaks
    entity boundary reasoning
    encapsulation trade-offs
```
