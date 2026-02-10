---
name: clean-code
model: standard
category: testing
description: Pragmatic coding standards for writing clean, maintainable code — naming, functions, structure, anti-patterns, and pre-edit safety checks. Use when writing new code, refactoring existing code, reviewing code quality, or establishing coding standards.
version: 2.0
---

# Clean Code

> Be **concise, direct, and solution-focused**. Clean code reads like well-written prose — every name reveals intent, every function does one thing, and every abstraction earns its place.


## Installation

### OpenClaw / Moltbot / Clawbot

```bash
npx clawhub@latest install clean-code
```


---

## Core Principles

| Principle | Rule | Practical Test |
|-----------|------|----------------|
| **SRP** | Single Responsibility — each function/class does ONE thing | "Can I describe what this does without using 'and'?" |
| **DRY** | Don't Repeat Yourself — extract duplicates, reuse | "Have I written this logic before?" |
| **KISS** | Keep It Simple — simplest solution that works | "Is there a simpler way to achieve this?" |
| **YAGNI** | You Aren't Gonna Need It — don't build unused features | "Does anyone need this right now?" |
| **Boy Scout** | Leave code cleaner than you found it | "Is this file better after my change?" |

---

## Naming Rules

Names are the most important documentation. A good name eliminates the need for a comment.

| Element | Convention | Bad | Good |
|---------|------------|-----|------|
| **Variables** | Reveal intent | `n`, `d`, `tmp` | `userCount`, `elapsed`, `activeUsers` |
| **Functions** | Verb + noun | `user()`, `calc()` | `getUserById()`, `calculateTotal()` |
| **Booleans** | Question form | `active`, `flag` | `isActive`, `hasPermission`, `canEdit` |
| **Constants** | SCREAMING_SNAKE | `max`, `timeout` | `MAX_RETRY_COUNT`, `REQUEST_TIMEOUT_MS` |
| **Classes** | Noun, singular | `Manager`, `Data` | `UserRepository`, `OrderService` |
| **Enums** | PascalCase values | `'pending'` string | `Status.Pending` |

> **Rule:** If you need a comment to explain a name, rename it.

### Naming Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| Cryptic abbreviations (`usrMgr`, `cfg`) | Unreadable in 6 months | Spell it out — IDE autocomplete makes long names free |
| Generic names (`data`, `info`, `item`, `handler`) | Says nothing about purpose | Use domain-specific names that reveal intent |
| Misleading names (`getUserList` returns one user) | Actively deceives readers | Match name to behavior, or change the behavior |
| Hungarian notation (`strName`, `nCount`, `IUser`) | Redundant with type system | Let TypeScript/IDE show types; names describe purpose |

---

## Function Rules

| Rule | Guideline | Why |
|------|-----------|-----|
| **Small** | Max 20 lines, ideally 5-10 | Fits in your head |
| **One Thing** | Does one thing, does it well | Testable and nameable |
| **One Level** | One level of abstraction per function | Readable top to bottom |
| **Few Args** | Max 3 arguments, prefer 0-2 | Easy to call correctly |
| **No Side Effects** | Don't mutate inputs unexpectedly | Predictable behavior |

### Guard Clauses

Flatten nested conditionals with early returns. Never nest deeper than 2 levels.

```typescript
// BAD — 5 levels deep
function processOrder(order: Order) {
  if (order) {
    if (order.items.length > 0) {
      if (order.customer) {
        if (order.customer.isVerified) {
          return submitOrder(order);
        }
      }
    }
  }
  throw new Error('Invalid order');
}

// GOOD — guard clauses flatten the structure
function processOrder(order: Order) {
  if (!order) throw new Error('No order');
  if (!order.items.length) throw new Error('No items');
  if (!order.customer) throw new Error('No customer');
  if (!order.customer.isVerified) throw new Error('Customer not verified');

  return submitOrder(order);
}
```

### Parameter Objects

When a function needs more than 3 arguments, use an options object.

```typescript
// BAD — too many parameters, order matters
createUser('John', 'Doe', 'john@example.com', 'secret', 'admin', 'Engineering');

// GOOD — self-documenting options object
createUser({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@example.com',
  password: 'secret',
  role: 'admin',
  department: 'Engineering',
});
```

---

## Code Structure Patterns

| Pattern | When to Apply | Benefit |
|---------|--------------|---------|
| **Guard Clauses** | Edge cases at function start | Flat, readable flow |
| **Flat > Nested** | Any nesting beyond 2 levels | Reduced cognitive load |
| **Composition** | Complex operations | Small, testable pieces |
| **Colocation** | Related code across files | Easier to find and change |
| **Extract Function** | Comments separating "sections" | Self-documenting code |

### Composition Over God Functions

```typescript
// BAD — god function doing everything
async function processOrder(order: Order) {
  // Validate... (15 lines)
  // Calculate totals... (15 lines)
  // Process payment... (10 lines)
  // Send notifications... (10 lines)
  // Update inventory... (10 lines)
  return { success: true };
}

// GOOD — composed of small, focused functions
async function processOrder(order: Order) {
  validateOrder(order);
  const totals = calculateOrderTotals(order);
  const payment = await processPayment(order.customer, totals);
  await sendOrderConfirmation(order, payment);
  await updateInventory(order.items);
  return { success: true, orderId: payment.orderId };
}
```

---

## Return Type Consistency

Functions should return consistent types. Use discriminated unions for multiple outcomes.

```typescript
// BAD — returns different types
function getUser(id: string) {
  const user = database.find(id);
  if (!user) return false;     // boolean
  if (user.isDeleted) return null; // null
  return user;                 // User
}

// GOOD — discriminated union
type GetUserResult =
  | { status: 'found'; user: User }
  | { status: 'not_found' }
  | { status: 'deleted' };

function getUser(id: string): GetUserResult {
  const user = database.find(id);
  if (!user) return { status: 'not_found' };
  if (user.isDeleted) return { status: 'deleted' };
  return { status: 'found', user };
}
```

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| Comment every line | Noise obscures signal | Delete obvious comments; comment *why*, not *what* |
| Helper for one-liner | Unnecessary indirection | Inline the code |
| Factory for 2 objects | Over-engineering | Direct instantiation |
| `utils.ts` with 1 function | Junk drawer file | Put code where it's used |
| Deep nesting | Unreadable flow | Guard clauses and early returns |
| Magic numbers | Unclear intent | Named constants |
| God functions | Untestable, unreadable | Split by responsibility |
| Commented-out code | Dead code confusion | Delete it; git remembers |
| TODO sprawl | Never gets done | Track in issue tracker, not code |
| Premature abstraction | Wrong abstraction is worse than none | Wait for 3+ duplicates before abstracting |
| Copy-paste programming | Duplicated bugs | Extract shared logic |
| Exception-driven control flow | Slow and confusing | Use explicit conditionals |
| Stringly-typed code | Typos and missed cases | Use enums or union types |
| Callback hell | Pyramid of doom | Use async/await |

---

## Pre-Edit Safety Check

Before changing any file, answer these questions to avoid cascading breakage:

| Question | Why |
|----------|-----|
| **What imports this file?** | Dependents might break on interface changes |
| **What does this file import?** | You might need to update the contract |
| **What tests cover this?** | Tests might fail — update them alongside code |
| **Is this a shared component?** | Multiple consumers means wider blast radius |

```
File to edit: UserService.ts
├── Who imports this? → UserController.ts, AuthController.ts
├── Do they need changes too? → Check function signatures
└── What tests cover this? → UserService.test.ts
```

> **Rule:** Edit the file + all dependent files in the SAME task. Never leave broken imports or missing updates.

---

## Self-Check Before Completing

Before marking any task complete, verify:

| Check | Question |
|-------|----------|
| **Goal met?** | Did I do exactly what was asked? |
| **Files edited?** | Did I modify all necessary files, including dependents? |
| **Code works?** | Did I verify the change compiles and runs? |
| **No errors?** | Do lint and type checks pass? |
| **Nothing forgotten?** | Any edge cases or dependent files missed? |

---

## NEVER Do

1. **NEVER add comments that restate the code** — if the code needs a comment to explain *what* it does, rename things until it doesn't
2. **NEVER create abstractions for fewer than 3 use cases** — premature abstraction is worse than duplication
3. **NEVER leave commented-out code in the codebase** — delete it; version control exists for history
4. **NEVER write functions longer than 20 lines** — extract sub-functions until each does one thing
5. **NEVER nest deeper than 2 levels** — use guard clauses, early returns, or extract functions
6. **NEVER use magic numbers or strings** — define named constants with clear semantics
7. **NEVER edit a file without checking what depends on it** — broken imports and missing updates are the most common source of bugs in multi-file changes
8. **NEVER leave a task with failing lint or type checks** — fix all errors before marking complete

---

## References

Detailed guides for specific clean code topics:

| Reference | Description |
|-----------|-------------|
| [Anti-Patterns](references/anti-patterns.md) | 21 common mistakes with bad/good code examples across naming, functions, structure, and comments |
| [Code Smells](references/code-smells.md) | Classic code smells catalog with detection patterns — Bloaters, OO Abusers, Change Preventers, Dispensables, Couplers |
| [Refactoring Catalog](references/refactoring-catalog.md) | Essential refactoring patterns with before/after examples and step-by-step mechanics |
# Anti-Patterns Gallery

Common coding mistakes with explanations and fixes. Each pattern includes bad/good examples to make code review and refactoring actionable.

---

## Naming Anti-Patterns

### 1. Cryptic Abbreviations

**Problem**: Abbreviations save keystrokes but cost readability. Future readers (including yourself) won't remember what `usrMgr` means.

```typescript
// ❌ Bad
const usrMgr = new UsrMgr();
const cfg = loadCfg();
const btn = document.getElementById('sbmt');
const val = calc(x, y, z);
```

```typescript
// ✅ Good
const userManager = new UserManager();
const config = loadConfig();
const submitButton = document.getElementById('submit');
const totalPrice = calculateTotalPrice(quantity, unitPrice, taxRate);
```

**Rule**: Spell it out. IDE autocomplete makes long names free.

---

### 2. Generic Names

**Problem**: Names like `data`, `info`, `item`, `thing`, `manager`, `handler` tell you nothing about what the code does.

```typescript
// ❌ Bad
function processData(data: any) {
  const info = getData();
  const result = handle(info);
  return result;
}

class Manager {
  items: any[] = [];
  process() { /* ... */ }
}
```

```typescript
// ✅ Good
function validateUserRegistration(registration: UserRegistration) {
  const existingUser = findUserByEmail(registration.email);
  const validationResult = checkEmailAvailability(existingUser);
  return validationResult;
}

class ShoppingCart {
  lineItems: CartLineItem[] = [];
  calculateTotal() { /* ... */ }
}
```

**Rule**: Names should reveal intent. Ask "what does this actually do?"

---

### 3. Misleading Names

**Problem**: Names that lie are worse than bad names. They actively deceive readers.

```typescript
// ❌ Bad - name lies about what it does
function getUserList() {
  // Actually returns a single user, not a list
  return this.currentUser;
}

const isValid = checkDate(date); // Returns the date, not a boolean

class AccountList extends Map { } // It's a Map, not a List
```

```typescript
// ✅ Good - names match behavior
function getCurrentUser() {
  return this.currentUser;
}

const normalizedDate = normalizeDate(date);

class AccountRegistry extends Map { }
```

**Rule**: If the name doesn't match the behavior, change the name (or the behavior).

---

### 4. Hungarian Notation

**Problem**: Encoding types in names was useful in weakly-typed languages. TypeScript makes it redundant and noisy.

```typescript
// ❌ Bad
const strName: string = 'Alice';
const nCount: number = 42;
const arrUsers: User[] = [];
const bIsActive: boolean = true;
interface IUser { } // "I" prefix for interfaces
type TUserRole = 'admin' | 'user'; // "T" prefix for types
```

```typescript
// ✅ Good
const name: string = 'Alice';
const count: number = 42;
const users: User[] = [];
const isActive: boolean = true;
interface User { }
type UserRole = 'admin' | 'user';
```

**Rule**: Let the type system handle types. Names should describe purpose.

---

## Function Anti-Patterns

### 5. God Functions

**Problem**: Functions over 20 lines are hard to understand, test, and modify. They usually do too many things.

```typescript
// ❌ Bad - 50+ line function doing everything
async function processOrder(order: Order) {
  // Validate order
  if (!order.items.length) throw new Error('Empty order');
  if (!order.customer) throw new Error('No customer');
  // ... 10 more validation lines
  
  // Calculate totals
  let subtotal = 0;
  for (const item of order.items) {
    subtotal += item.price * item.quantity;
    // ... discount logic
  }
  // ... 15 more calculation lines
  
  // Process payment
  const paymentResult = await stripe.charge(/* ... */);
  // ... 10 more payment lines
  
  // Send notifications
  await sendEmail(/* ... */);
  await sendSMS(/* ... */);
  // ... more notification logic
  
  // Update inventory
  // ... 10 more lines
  
  return { success: true };
}
```

```typescript
// ✅ Good - composed of small, focused functions
async function processOrder(order: Order) {
  validateOrder(order);
  const totals = calculateOrderTotals(order);
  const payment = await processPayment(order.customer, totals);
  await sendOrderConfirmation(order, payment);
  await updateInventory(order.items);
  return { success: true, orderId: payment.orderId };
}

function validateOrder(order: Order): void {
  if (!order.items.length) throw new Error('Empty order');
  if (!order.customer) throw new Error('No customer');
}

function calculateOrderTotals(order: Order): OrderTotals {
  const subtotal = order.items.reduce(
    (sum, item) => sum + item.price * item.quantity, 0
  );
  return { subtotal, tax: subtotal * 0.1, total: subtotal * 1.1 };
}
```

**Rule**: Extract until each function does exactly one thing.

---

### 6. Too Many Parameters

**Problem**: Functions with 4+ parameters are hard to call correctly and often indicate the function does too much.

```typescript
// ❌ Bad - too many parameters, order matters
function createUser(
  firstName: string,
  lastName: string,
  email: string,
  password: string,
  role: string,
  department: string,
  startDate: Date,
  managerId: string | null,
  isActive: boolean
) {
  // ...
}

// Callers must remember order
createUser('John', 'Doe', 'john@example.com', 'secret', 'admin', 'Engineering', new Date(), null, true);
```

```typescript
// ✅ Good - use an options object
interface CreateUserOptions {
  firstName: string;
  lastName: string;
  email: string;
  password: string;
  role: UserRole;
  department: string;
  startDate?: Date;
  managerId?: string;
  isActive?: boolean;
}

function createUser(options: CreateUserOptions) {
  const { firstName, lastName, email, role, isActive = true } = options;
  // ...
}

// Callers have self-documenting code
createUser({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@example.com',
  password: 'secret',
  role: 'admin',
  department: 'Engineering',
});
```

**Rule**: More than 3 parameters? Use an options object.

---

### 7. Boolean Flag Parameters

**Problem**: Boolean parameters hide branching logic and make function calls unreadable.

```typescript
// ❌ Bad - what does `true` mean here?
renderButton('Submit', true, false, true);

function renderButton(
  label: string,
  isPrimary: boolean,
  isDisabled: boolean,
  isLoading: boolean
) {
  // Complex branching based on booleans
}
```

```typescript
// ✅ Good - use options object or separate functions
renderButton({
  label: 'Submit',
  variant: 'primary',
  state: 'loading',
});

// Or separate functions for distinct behaviors
renderPrimaryButton('Submit');
renderLoadingButton('Submit');

// Or use enums
type ButtonVariant = 'primary' | 'secondary' | 'ghost';
type ButtonState = 'default' | 'loading' | 'disabled';
```

**Rule**: Boolean parameters should be in an options object with named properties.

---

### 8. Side Effects in Getters

**Problem**: Getters that modify state violate the principle of least surprise. Readers expect getters to be pure.

```typescript
// ❌ Bad - getter with hidden side effect
class ShoppingCart {
  private _items: CartItem[] = [];
  private _lastAccessed: Date;

  get items() {
    this._lastAccessed = new Date(); // Side effect!
    this.logAccess(); // Another side effect!
    return this._items;
  }

  get totalPrice() {
    this.recalculateDiscounts(); // Mutation!
    return this._items.reduce((sum, i) => sum + i.price, 0);
  }
}
```

```typescript
// ✅ Good - getters are pure, side effects are explicit
class ShoppingCart {
  private _items: CartItem[] = [];
  private _lastAccessed: Date;

  get items() {
    return this._items;
  }

  get totalPrice() {
    return this._items.reduce((sum, i) => sum + i.price, 0);
  }

  recordAccess() {
    this._lastAccessed = new Date();
    this.logAccess();
  }

  applyDiscounts() {
    this.recalculateDiscounts();
  }
}
```

**Rule**: Getters should be pure. Make side effects explicit with verbs.

---

### 9. Returning Different Types

**Problem**: Functions that return different types based on conditions make code unpredictable and hard to type.

```typescript
// ❌ Bad - return type depends on runtime condition
function getUser(id: string) {
  const user = database.find(id);
  if (!user) {
    return false; // boolean
  }
  if (user.isDeleted) {
    return null; // null
  }
  return user; // User object
}

// Caller must handle all cases
const result = getUser('123');
if (result === false) { /* not found */ }
else if (result === null) { /* deleted */ }
else { /* use result.name */ }
```

```typescript
// ✅ Good - consistent return type with discriminated union
type GetUserResult =
  | { status: 'found'; user: User }
  | { status: 'not_found' }
  | { status: 'deleted' };

function getUser(id: string): GetUserResult {
  const user = database.find(id);
  if (!user) {
    return { status: 'not_found' };
  }
  if (user.isDeleted) {
    return { status: 'deleted' };
  }
  return { status: 'found', user };
}

// Caller has type-safe handling
const result = getUser('123');
if (result.status === 'found') {
  console.log(result.user.name); // TypeScript knows user exists
}
```

**Rule**: Return consistent types. Use discriminated unions for multiple outcomes.

---

## Structure Anti-Patterns

### 10. Deep Nesting (Pyramid of Doom)

**Problem**: Deeply nested code is hard to follow and usually indicates missing abstractions.

```typescript
// ❌ Bad - 5 levels deep
function processOrder(order: Order) {
  if (order) {
    if (order.items.length > 0) {
      if (order.customer) {
        if (order.customer.isVerified) {
          if (order.paymentMethod) {
            // Finally, the actual logic buried 5 levels deep
            return submitOrder(order);
          } else {
            throw new Error('No payment method');
          }
        } else {
          throw new Error('Customer not verified');
        }
      } else {
        throw new Error('No customer');
      }
    } else {
      throw new Error('No items');
    }
  } else {
    throw new Error('No order');
  }
}
```

```typescript
// ✅ Good - guard clauses flatten the structure
function processOrder(order: Order) {
  if (!order) throw new Error('No order');
  if (!order.items.length) throw new Error('No items');
  if (!order.customer) throw new Error('No customer');
  if (!order.customer.isVerified) throw new Error('Customer not verified');
  if (!order.paymentMethod) throw new Error('No payment method');

  return submitOrder(order);
}
```

**Rule**: Use guard clauses for early returns. Max 2 levels of nesting.

---

### 11. Premature Abstraction

**Problem**: Creating abstractions before you understand the problem leads to wrong abstractions that are hard to change.

```typescript
// ❌ Bad - abstraction created for one use case
interface DataFetcher<T> {
  fetch(): Promise<T>;
  cache(): void;
  invalidate(): void;
}

class GenericRepository<T> implements DataFetcher<T> {
  constructor(private adapter: StorageAdapter<T>) {}
  // ... complex implementation
}

// Used exactly once:
const userRepo = new GenericRepository(new UserAdapter());
```

```typescript
// ✅ Good - start simple, abstract when patterns emerge
// First implementation: just fetch users
async function fetchUsers(): Promise<User[]> {
  return await db.query('SELECT * FROM users');
}

// Later, if you need caching, add it:
async function fetchUsersWithCache(): Promise<User[]> {
  const cached = cache.get('users');
  if (cached) return cached;
  
  const users = await db.query('SELECT * FROM users');
  cache.set('users', users);
  return users;
}

// Abstract only when you see the SAME pattern 3+ times
```

**Rule**: "Duplication is far cheaper than the wrong abstraction." - Sandi Metz

---

### 12. Over-Engineering

**Problem**: Building for hypothetical future requirements adds complexity without value.

```typescript
// ❌ Bad - enterprise FizzBuzz
interface FizzBuzzStrategy {
  applies(n: number): boolean;
  execute(n: number): string;
}

class FizzStrategy implements FizzBuzzStrategy {
  applies(n: number) { return n % 3 === 0; }
  execute() { return 'Fizz'; }
}

class BuzzStrategy implements FizzBuzzStrategy {
  applies(n: number) { return n % 5 === 0; }
  execute() { return 'Buzz'; }
}

class FizzBuzzProcessor {
  constructor(private strategies: FizzBuzzStrategy[]) {}
  process(n: number): string {
    return this.strategies
      .filter(s => s.applies(n))
      .map(s => s.execute(n))
      .join('') || String(n);
  }
}

const processor = new FizzBuzzProcessor([
  new FizzStrategy(),
  new BuzzStrategy(),
]);
```

```typescript
// ✅ Good - solve the actual problem
function fizzBuzz(n: number): string {
  if (n % 15 === 0) return 'FizzBuzz';
  if (n % 3 === 0) return 'Fizz';
  if (n % 5 === 0) return 'Buzz';
  return String(n);
}
```

**Rule**: Solve today's problem. Refactor when requirements actually change.

---

### 13. Copy-Paste Programming

**Problem**: Duplicated code means duplicated bugs and maintenance burden.

```typescript
// ❌ Bad - same validation logic copied
function createUser(data: UserInput) {
  if (!data.email) throw new Error('Email required');
  if (!data.email.includes('@')) throw new Error('Invalid email');
  if (data.email.length > 255) throw new Error('Email too long');
  // ... create user
}

function updateUser(id: string, data: UserInput) {
  if (!data.email) throw new Error('Email required');
  if (!data.email.includes('@')) throw new Error('Invalid email');
  if (data.email.length > 255) throw new Error('Email too long');
  // ... update user
}

function inviteUser(data: UserInput) {
  if (!data.email) throw new Error('Email required');
  if (!data.email.includes('@')) throw new Error('Invalid email');
  if (data.email.length > 255) throw new Error('Email too long');
  // ... invite user
}
```

```typescript
// ✅ Good - extract shared logic
function validateEmail(email: string): void {
  if (!email) throw new Error('Email required');
  if (!email.includes('@')) throw new Error('Invalid email');
  if (email.length > 255) throw new Error('Email too long');
}

function createUser(data: UserInput) {
  validateEmail(data.email);
  // ... create user
}

function updateUser(id: string, data: UserInput) {
  validateEmail(data.email);
  // ... update user
}

function inviteUser(data: UserInput) {
  validateEmail(data.email);
  // ... invite user
}
```

**Rule**: If you copy-paste, you're probably missing an abstraction.

---

### 14. God Objects

**Problem**: Classes that know too much and do too much become unmaintainable.

```typescript
// ❌ Bad - class does everything
class ApplicationManager {
  users: User[] = [];
  orders: Order[] = [];
  products: Product[] = [];
  
  // User operations
  createUser() { /* ... */ }
  deleteUser() { /* ... */ }
  authenticateUser() { /* ... */ }
  
  // Order operations
  createOrder() { /* ... */ }
  cancelOrder() { /* ... */ }
  refundOrder() { /* ... */ }
  
  // Product operations
  addProduct() { /* ... */ }
  updateInventory() { /* ... */ }
  
  // Reporting
  generateSalesReport() { /* ... */ }
  generateUserReport() { /* ... */ }
  
  // Notifications
  sendEmail() { /* ... */ }
  sendSMS() { /* ... */ }
}
```

```typescript
// ✅ Good - separate concerns
class UserService {
  createUser() { /* ... */ }
  deleteUser() { /* ... */ }
}

class AuthService {
  authenticate() { /* ... */ }
}

class OrderService {
  createOrder() { /* ... */ }
  cancelOrder() { /* ... */ }
}

class NotificationService {
  sendEmail() { /* ... */ }
  sendSMS() { /* ... */ }
}
```

**Rule**: Each class should have one reason to change.

---

## Comment Anti-Patterns

### 15. Commented-Out Code

**Problem**: Commented code is dead code. It confuses readers and never gets cleaned up.

```typescript
// ❌ Bad
function calculateTotal(items: Item[]) {
  let total = 0;
  for (const item of items) {
    total += item.price;
    // total += item.price * item.quantity; // Old calculation
    // if (item.discount) {
    //   total -= item.discount;
    // }
  }
  // return total * 1.1; // With tax
  // return total * 1.08; // Old tax rate
  return total;
}
```

```typescript
// ✅ Good - delete it, git remembers
function calculateTotal(items: Item[]) {
  return items.reduce((total, item) => total + item.price, 0);
}
```

**Rule**: Delete commented code. Use version control for history.

---

### 16. Obvious Comments

**Problem**: Comments that restate the code add noise without value.

```typescript
// ❌ Bad - comments that add nothing
// Increment counter
counter++;

// Check if user is null
if (user === null) {
  // Return early
  return;
}

// Loop through all items
for (const item of items) {
  // Add item price to total
  total += item.price;
}
```

```typescript
// ✅ Good - code is self-documenting, comments explain WHY
counter++; // No comment needed

if (!user) return;

const total = items.reduce((sum, item) => sum + item.price, 0);

// Business rule: Premium members get early access 48 hours before launch
if (user.isPremium && hoursUntilLaunch < 48) {
  showEarlyAccess();
}
```

**Rule**: Don't comment WHAT, comment WHY (if not obvious).

---

### 17. TODO Sprawl

**Problem**: TODOs accumulate and never get done. They become invisible noise.

```typescript
// ❌ Bad - TODO graveyard
function processPayment(amount: number) {
  // TODO: Add retry logic
  // TODO: Handle currency conversion
  // TODO: Add logging
  // TODO: Optimize this (added 2019)
  // FIXME: This is broken sometimes
  // HACK: Temporary fix, remove later
  // XXX: Why does this work?
  return charge(amount);
}
```

```typescript
// ✅ Good - TODOs are tracked in issues, not code
function processPayment(amount: number) {
  // See JIRA-1234 for planned retry logic
  return charge(amount);
}

// Or just fix it now:
async function processPayment(amount: number) {
  return await retry(() => charge(amount), { attempts: 3 });
}
```

**Rule**: TODOs belong in your issue tracker, not your code.

---

### 18. Outdated Comments

**Problem**: Comments that contradict the code are actively harmful.

```typescript
// ❌ Bad - comment lies about the code
// Returns the user's full name (first + last)
function getUserName(user: User): string {
  return user.email; // Actually returns email!
}

// Validates and saves the user
function processUser(user: User) {
  // No validation, just saves
  database.save(user);
}

// This function is deprecated, use newFunction() instead
function oldFunction() {
  // Still actively used throughout codebase
}
```

```typescript
// ✅ Good - update or remove outdated comments
function getUserEmail(user: User): string {
  return user.email;
}

function saveUser(user: User) {
  database.save(user);
}

/** @deprecated Use {@link newFunction} instead */
function oldFunction() {
  console.warn('oldFunction is deprecated');
  return newFunction();
}
```

**Rule**: When you change code, update or delete related comments.

---

## Control Flow Anti-Patterns

### 19. Exception-Driven Control Flow

**Problem**: Using exceptions for normal control flow is slow and hard to follow.

```typescript
// ❌ Bad - exceptions for expected cases
function findUser(id: string): User {
  try {
    return database.getUser(id);
  } catch {
    try {
      return cache.getUser(id);
    } catch {
      try {
        return createDefaultUser(id);
      } catch {
        throw new Error('Cannot get user');
      }
    }
  }
}
```

```typescript
// ✅ Good - explicit control flow
function findUser(id: string): User | null {
  const dbUser = database.getUser(id);
  if (dbUser) return dbUser;

  const cachedUser = cache.getUser(id);
  if (cachedUser) return cachedUser;

  return createDefaultUser(id);
}
```

**Rule**: Exceptions are for exceptional situations, not control flow.

---

### 20. Stringly-Typed Code

**Problem**: Using strings where enums or types would be safer leads to typos and missing cases.

```typescript
// ❌ Bad - magic strings everywhere
function handleStatus(status: string) {
  if (status === 'pending') { /* ... */ }
  else if (status === 'Pending') { /* ... */ } // Typo variant
  else if (status === 'active') { /* ... */ }
  else if (status === 'actve') { /* ... */ } // Typo!
}

user.role = 'admni'; // Typo, no error!
```

```typescript
// ✅ Good - type-safe enums or unions
type Status = 'pending' | 'active' | 'completed' | 'cancelled';
type UserRole = 'admin' | 'user' | 'guest';

function handleStatus(status: Status) {
  switch (status) {
    case 'pending': /* ... */ break;
    case 'active': /* ... */ break;
    case 'completed': /* ... */ break;
    case 'cancelled': /* ... */ break;
    // TypeScript ensures exhaustive handling
  }
}

user.role = 'admni'; // TypeScript error!
```

**Rule**: Use types instead of strings for fixed sets of values.

---

### 21. Callback Hell

**Problem**: Nested callbacks create unreadable, hard-to-debug code.

```typescript
// ❌ Bad - callback pyramid
getUser(userId, (err, user) => {
  if (err) return handleError(err);
  getOrders(user.id, (err, orders) => {
    if (err) return handleError(err);
    getOrderDetails(orders[0].id, (err, details) => {
      if (err) return handleError(err);
      processDetails(details, (err, result) => {
        if (err) return handleError(err);
        sendNotification(result, (err) => {
          if (err) return handleError(err);
          console.log('Done!');
        });
      });
    });
  });
});
```

```typescript
// ✅ Good - async/await
async function processUserOrder(userId: string) {
  try {
    const user = await getUser(userId);
    const orders = await getOrders(user.id);
    const details = await getOrderDetails(orders[0].id);
    const result = await processDetails(details);
    await sendNotification(result);
    console.log('Done!');
  } catch (error) {
    handleError(error);
  }
}
```

**Rule**: Use async/await for asynchronous code.

---

## Quick Reference

| Anti-Pattern | Fix |
|--------------|-----|
| Cryptic abbreviations | Spell it out |
| Generic names | Reveal intent |
| Misleading names | Match name to behavior |
| Hungarian notation | Let types handle types |
| God functions | Extract smaller functions |
| Too many parameters | Use options object |
| Boolean flags | Named options or separate functions |
| Side effects in getters | Make mutations explicit |
| Different return types | Use discriminated unions |
| Deep nesting | Guard clauses |
| Premature abstraction | Wait for patterns to emerge |
| Over-engineering | Solve today's problem |
| Copy-paste | Extract shared logic |
| God objects | Single responsibility |
| Commented-out code | Delete it |
| Obvious comments | Let code self-document |
| TODO sprawl | Use issue tracker |
| Outdated comments | Update or delete |
| Exception control flow | Explicit conditionals |
| Stringly-typed | Use enums/unions |
| Callback hell | async/await |
# Code Smells Catalog

Code smells are symptoms that indicate deeper problems. They're not bugs—the code works—but they signal design issues that make code harder to understand, change, and maintain.

Based on Martin Fowler's refactoring catalog with TypeScript examples.

---

## Bloaters

Code that has grown too large to work with effectively.

### Long Method

**Symptoms**:
- Function exceeds 20 lines
- Multiple levels of abstraction in one function
- Comments separating "sections" within a function
- Difficult to name—does too many things

```typescript
// ❌ Smell: Long method with multiple responsibilities
async function processCheckout(cart: Cart, user: User) {
  // Validate cart
  if (!cart.items.length) throw new Error('Empty cart');
  for (const item of cart.items) {
    const product = await getProduct(item.productId);
    if (product.stock < item.quantity) {
      throw new Error(`Insufficient stock for ${product.name}`);
    }
  }
  
  // Calculate totals
  let subtotal = 0;
  for (const item of cart.items) {
    const product = await getProduct(item.productId);
    subtotal += product.price * item.quantity;
  }
  const tax = subtotal * 0.1;
  const shipping = subtotal > 100 ? 0 : 10;
  const total = subtotal + tax + shipping;
  
  // Process payment
  const paymentIntent = await stripe.paymentIntents.create({
    amount: Math.round(total * 100),
    currency: 'usd',
  });
  
  // Create order
  const order = await db.orders.create({
    userId: user.id,
    items: cart.items,
    total,
    paymentIntentId: paymentIntent.id,
  });
  
  // Send confirmation
  await sendEmail(user.email, 'Order Confirmed', `Order #${order.id}`);
  
  // Update inventory
  for (const item of cart.items) {
    await db.products.update(item.productId, {
      stock: { decrement: item.quantity },
    });
  }
  
  return order;
}
```

**Refactoring**: Extract Method

```typescript
// ✅ Refactored: Each function does one thing
async function processCheckout(cart: Cart, user: User) {
  await validateCart(cart);
  const totals = await calculateTotals(cart);
  const payment = await processPayment(totals.total);
  const order = await createOrder(user, cart, totals, payment);
  await sendOrderConfirmation(user, order);
  await updateInventory(cart.items);
  return order;
}

async function validateCart(cart: Cart): Promise<void> {
  if (!cart.items.length) throw new Error('Empty cart');
  for (const item of cart.items) {
    const product = await getProduct(item.productId);
    if (product.stock < item.quantity) {
      throw new Error(`Insufficient stock for ${product.name}`);
    }
  }
}

async function calculateTotals(cart: Cart): Promise<OrderTotals> {
  const subtotal = await calculateSubtotal(cart.items);
  const tax = subtotal * 0.1;
  const shipping = subtotal > 100 ? 0 : 10;
  return { subtotal, tax, shipping, total: subtotal + tax + shipping };
}
```

**Prevention**: If you're adding a comment to separate sections, extract a function instead.

---

### Large Class

**Symptoms**:
- Class has 10+ methods
- Class has 10+ fields
- Class name includes "Manager", "Processor", "Handler" doing everything
- You can't summarize what the class does in one sentence

```typescript
// ❌ Smell: God class that does everything
class UserManager {
  // User CRUD
  createUser(data: UserInput) { /* ... */ }
  updateUser(id: string, data: UserInput) { /* ... */ }
  deleteUser(id: string) { /* ... */ }
  getUser(id: string) { /* ... */ }
  listUsers(filters: UserFilters) { /* ... */ }
  
  // Authentication
  login(email: string, password: string) { /* ... */ }
  logout(userId: string) { /* ... */ }
  resetPassword(email: string) { /* ... */ }
  verifyEmail(token: string) { /* ... */ }
  
  // Authorization
  checkPermission(userId: string, resource: string) { /* ... */ }
  assignRole(userId: string, role: string) { /* ... */ }
  
  // Profile
  updateProfile(userId: string, profile: Profile) { /* ... */ }
  uploadAvatar(userId: string, file: File) { /* ... */ }
  
  // Notifications
  sendWelcomeEmail(userId: string) { /* ... */ }
  sendPasswordResetEmail(email: string) { /* ... */ }
  
  // Analytics
  trackLogin(userId: string) { /* ... */ }
  getLoginHistory(userId: string) { /* ... */ }
}
```

**Refactoring**: Extract Class

```typescript
// ✅ Refactored: Single Responsibility
class UserRepository {
  create(data: UserInput) { /* ... */ }
  update(id: string, data: UserInput) { /* ... */ }
  delete(id: string) { /* ... */ }
  findById(id: string) { /* ... */ }
  findMany(filters: UserFilters) { /* ... */ }
}

class AuthService {
  login(email: string, password: string) { /* ... */ }
  logout(userId: string) { /* ... */ }
  resetPassword(email: string) { /* ... */ }
  verifyEmail(token: string) { /* ... */ }
}

class AuthorizationService {
  checkPermission(userId: string, resource: string) { /* ... */ }
  assignRole(userId: string, role: string) { /* ... */ }
}

class ProfileService {
  updateProfile(userId: string, profile: Profile) { /* ... */ }
  uploadAvatar(userId: string, file: File) { /* ... */ }
}

class UserNotificationService {
  sendWelcomeEmail(userId: string) { /* ... */ }
  sendPasswordResetEmail(email: string) { /* ... */ }
}
```

**Prevention**: Each class should have one reason to change.

---

### Long Parameter List

**Symptoms**:
- Function has 4+ parameters
- Parameters are often passed together
- Hard to remember parameter order
- Boolean parameters with unclear meaning

```typescript
// ❌ Smell: Too many parameters
function createReport(
  title: string,
  startDate: Date,
  endDate: Date,
  format: string,
  includeCharts: boolean,
  includeTables: boolean,
  groupBy: string,
  sortBy: string,
  sortOrder: string,
  limit: number,
  userId: string
) {
  // ...
}
```

**Refactoring**: Introduce Parameter Object

```typescript
// ✅ Refactored: Options object
interface ReportOptions {
  title: string;
  dateRange: {
    start: Date;
    end: Date;
  };
  format: 'pdf' | 'excel' | 'csv';
  includes?: {
    charts?: boolean;
    tables?: boolean;
  };
  sorting?: {
    field: string;
    order: 'asc' | 'desc';
  };
  groupBy?: string;
  limit?: number;
  userId: string;
}

function createReport(options: ReportOptions) {
  const { title, dateRange, format, includes, sorting, limit = 100 } = options;
  // ...
}
```

**Prevention**: If parameters travel together, they belong together.

---

### Data Clumps

**Symptoms**:
- Same 3+ fields appear together in multiple places
- Functions pass the same group of parameters
- Classes have fields that are always used together

```typescript
// ❌ Smell: Same fields everywhere
function calculateDistance(
  startLat: number, startLng: number,
  endLat: number, endLng: number
) { /* ... */ }

function formatAddress(
  street: string, city: string, state: string, zip: string
) { /* ... */ }

class Delivery {
  pickupStreet: string;
  pickupCity: string;
  pickupState: string;
  pickupZip: string;
  
  dropoffStreet: string;
  dropoffCity: string;
  dropoffState: string;
  dropoffZip: string;
}
```

**Refactoring**: Extract Class

```typescript
// ✅ Refactored: Create value objects
interface Coordinates {
  latitude: number;
  longitude: number;
}

interface Address {
  street: string;
  city: string;
  state: string;
  zip: string;
}

function calculateDistance(start: Coordinates, end: Coordinates) { /* ... */ }
function formatAddress(address: Address) { /* ... */ }

class Delivery {
  pickup: Address;
  dropoff: Address;
}
```

**Prevention**: When you see fields traveling together, introduce a class.

---

### Primitive Obsession

**Symptoms**:
- Using strings/numbers where a class would be clearer
- Validation logic repeated for the same concept
- Magic strings or numbers scattered in code

```typescript
// ❌ Smell: Primitives everywhere
function processOrder(
  userId: string,           // Could be UserId
  email: string,            // Could be Email
  phone: string,            // Could be PhoneNumber
  amount: number,           // Could be Money
  currency: string,         // Could be Currency
  status: string            // Could be OrderStatus
) {
  // Email validation repeated everywhere
  if (!email.includes('@')) throw new Error('Invalid email');
  
  // Currency logic scattered
  if (currency === 'USD') { /* ... */ }
  else if (currency === 'EUR') { /* ... */ }
  
  // Status checks using magic strings
  if (status === 'pending') { /* ... */ }
}
```

**Refactoring**: Replace Primitive with Object

```typescript
// ✅ Refactored: Domain types encapsulate rules
class Email {
  private constructor(private readonly value: string) {}
  
  static create(value: string): Email {
    if (!value.includes('@')) throw new Error('Invalid email');
    return new Email(value.toLowerCase());
  }
  
  toString() { return this.value; }
}

class Money {
  constructor(
    private readonly amount: number,
    private readonly currency: Currency
  ) {}
  
  add(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new Error('Currency mismatch');
    }
    return new Money(this.amount + other.amount, this.currency);
  }
}

type OrderStatus = 'pending' | 'confirmed' | 'shipped' | 'delivered';

function processOrder(
  userId: UserId,
  email: Email,
  amount: Money,
  status: OrderStatus
) {
  // No validation needed—types enforce rules
}
```

**Prevention**: If you validate a primitive the same way multiple times, wrap it.

---

## Object-Orientation Abusers

Incorrect or incomplete application of OO principles.

### Switch Statements

**Symptoms**:
- Same switch/if-else chain in multiple places
- Adding a new case requires changes in multiple files
- Switches based on type codes

```typescript
// ❌ Smell: Type-based switching repeated everywhere
function calculatePay(employee: Employee): number {
  switch (employee.type) {
    case 'hourly':
      return employee.hoursWorked * employee.hourlyRate;
    case 'salaried':
      return employee.annualSalary / 12;
    case 'commissioned':
      return employee.baseSalary + employee.sales * employee.commissionRate;
    default:
      throw new Error('Unknown employee type');
  }
}

function getTimeOffDays(employee: Employee): number {
  switch (employee.type) {
    case 'hourly': return 10;
    case 'salaried': return 20;
    case 'commissioned': return 15;
    default: return 0;
  }
}
```

**Refactoring**: Replace Conditional with Polymorphism

```typescript
// ✅ Refactored: Polymorphism
interface Employee {
  calculatePay(): number;
  getTimeOffDays(): number;
}

class HourlyEmployee implements Employee {
  constructor(
    private hoursWorked: number,
    private hourlyRate: number
  ) {}
  
  calculatePay() { return this.hoursWorked * this.hourlyRate; }
  getTimeOffDays() { return 10; }
}

class SalariedEmployee implements Employee {
  constructor(private annualSalary: number) {}
  
  calculatePay() { return this.annualSalary / 12; }
  getTimeOffDays() { return 20; }
}

class CommissionedEmployee implements Employee {
  constructor(
    private baseSalary: number,
    private sales: number,
    private commissionRate: number
  ) {}
  
  calculatePay() {
    return this.baseSalary + this.sales * this.commissionRate;
  }
  getTimeOffDays() { return 15; }
}
```

**Prevention**: If switches on type appear in multiple places, use polymorphism.

---

### Temporary Field

**Symptoms**:
- Fields that are only set in certain situations
- Null checks scattered throughout the class
- Fields used by only some methods

```typescript
// ❌ Smell: Fields only valid sometimes
class Order {
  items: OrderItem[];
  customer: Customer;
  
  // Only set during discount calculation
  discountPercentage?: number;
  discountReason?: string;
  discountApprover?: string;
  
  // Only set after shipping
  trackingNumber?: string;
  carrier?: string;
  estimatedDelivery?: Date;
  
  calculateTotal() {
    let total = this.items.reduce((sum, i) => sum + i.price, 0);
    // Null check because field might not exist
    if (this.discountPercentage) {
      total *= (1 - this.discountPercentage / 100);
    }
    return total;
  }
}
```

**Refactoring**: Extract Class or Introduce Null Object

```typescript
// ✅ Refactored: Separate concerns
class Order {
  items: OrderItem[];
  customer: Customer;
  discount: Discount = Discount.none();
  shipping?: ShippingInfo;
  
  calculateTotal() {
    const subtotal = this.items.reduce((sum, i) => sum + i.price, 0);
    return this.discount.applyTo(subtotal);
  }
}

class Discount {
  private constructor(
    private percentage: number,
    private reason: string,
    private approver: string | null
  ) {}
  
  static none() { return new Discount(0, '', null); }
  
  static create(percentage: number, reason: string, approver: string) {
    return new Discount(percentage, reason, approver);
  }
  
  applyTo(amount: number) {
    return amount * (1 - this.percentage / 100);
  }
}

interface ShippingInfo {
  trackingNumber: string;
  carrier: string;
  estimatedDelivery: Date;
}
```

**Prevention**: If fields are only used together, extract them to a class.

---

## Change Preventers

Code structures that make changes difficult.

### Divergent Change

**Symptoms**:
- One class is modified for multiple unrelated reasons
- Changes in different domains affect the same file
- "I need to change X, Y, and Z in this file"

```typescript
// ❌ Smell: Class changes for different reasons
class ReportGenerator {
  // Changes when report format changes
  generatePDF(data: ReportData) { /* ... */ }
  generateExcel(data: ReportData) { /* ... */ }
  generateCSV(data: ReportData) { /* ... */ }
  
  // Changes when data source changes
  fetchFromDatabase(query: string) { /* ... */ }
  fetchFromAPI(endpoint: string) { /* ... */ }
  
  // Changes when business logic changes
  calculateTotals(data: ReportData) { /* ... */ }
  applyFilters(data: ReportData) { /* ... */ }
}
```

**Refactoring**: Extract Class

```typescript
// ✅ Refactored: Separate by reason for change
class ReportFormatter {
  toPDF(data: ReportData) { /* ... */ }
  toExcel(data: ReportData) { /* ... */ }
  toCSV(data: ReportData) { /* ... */ }
}

class DataFetcher {
  fromDatabase(query: string) { /* ... */ }
  fromAPI(endpoint: string) { /* ... */ }
}

class ReportCalculator {
  calculateTotals(data: ReportData) { /* ... */ }
  applyFilters(data: ReportData) { /* ... */ }
}

class ReportGenerator {
  constructor(
    private fetcher: DataFetcher,
    private calculator: ReportCalculator,
    private formatter: ReportFormatter
  ) {}
  
  generate(source: DataSource, format: Format): Report {
    const data = this.fetcher.fetch(source);
    const processed = this.calculator.process(data);
    return this.formatter.format(processed, format);
  }
}
```

**Prevention**: Each class should have one reason to change.

---

### Shotgun Surgery

**Symptoms**:
- A single change requires editing many files
- Related code is scattered across the codebase
- Easy to miss one of the required changes

```typescript
// ❌ Smell: Adding a field requires changes everywhere
// user.ts
interface User { name: string; email: string; }

// api/users.ts
app.post('/users', (req) => {
  const { name, email } = req.body;
  // ...
});

// validation.ts
function validateUser(data: unknown) {
  if (!data.name) throw new Error('Name required');
  if (!data.email) throw new Error('Email required');
}

// database/user-repo.ts
function createUser(name: string, email: string) {
  db.query('INSERT INTO users (name, email) VALUES (?, ?)', [name, email]);
}

// Adding 'phone' requires changes in 4+ files!
```

**Refactoring**: Move Method, Inline Class

```typescript
// ✅ Refactored: Centralize related logic
// user.ts - Single source of truth
interface User {
  name: string;
  email: string;
  phone?: string;
}

const UserSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
  phone: z.string().optional(),
});

class UserRepository {
  create(data: User) {
    const validated = UserSchema.parse(data);
    return db.users.create(validated);
  }
}

// api/users.ts - Just orchestration
app.post('/users', async (req) => {
  const user = await userRepository.create(req.body);
  return user;
});
```

**Prevention**: Keep related behavior together.

---

### Feature Envy

**Symptoms**:
- A method uses more features of another class than its own
- Lots of getter calls on other objects
- Logic that should belong to the data it operates on

```typescript
// ❌ Smell: OrderPrinter is too interested in Order
class OrderPrinter {
  print(order: Order) {
    console.log(`Order: ${order.getId()}`);
    console.log(`Customer: ${order.getCustomer().getName()}`);
    console.log(`Address: ${order.getCustomer().getAddress().getFullAddress()}`);
    
    let total = 0;
    for (const item of order.getItems()) {
      const price = item.getProduct().getPrice();
      const qty = item.getQuantity();
      const discount = item.getProduct().getDiscount();
      const lineTotal = price * qty * (1 - discount);
      total += lineTotal;
      console.log(`${item.getProduct().getName()}: $${lineTotal}`);
    }
    
    console.log(`Total: $${total}`);
  }
}
```

**Refactoring**: Move Method

```typescript
// ✅ Refactored: Move logic to where data lives
class Order {
  getFormattedSummary(): string {
    return [
      `Order: ${this.id}`,
      `Customer: ${this.customer.name}`,
      `Address: ${this.customer.address.format()}`,
      ...this.items.map(item => item.format()),
      `Total: $${this.calculateTotal()}`,
    ].join('\n');
  }
  
  calculateTotal(): number {
    return this.items.reduce((sum, item) => sum + item.calculateLineTotal(), 0);
  }
}

class OrderItem {
  calculateLineTotal(): number {
    return this.product.price * this.quantity * (1 - this.product.discount);
  }
  
  format(): string {
    return `${this.product.name}: $${this.calculateLineTotal()}`;
  }
}

class OrderPrinter {
  print(order: Order) {
    console.log(order.getFormattedSummary());
  }
}
```

**Prevention**: Put behavior with the data it needs.

---

## Dispensables

Code that serves no purpose and should be removed.

### Dead Code

**Symptoms**:
- Unreachable code after return/throw
- Unused variables, parameters, or functions
- Commented-out code
- Obsolete feature flags

```typescript
// ❌ Smell: Code that's never executed
function processPayment(amount: number) {
  if (amount <= 0) {
    throw new Error('Invalid amount');
    console.log('This never runs'); // Dead code
  }
  
  const result = charge(amount);
  return result;
  
  // Everything below is dead code
  sendReceipt(result);
  updateAnalytics(result);
}

// Unused function
function legacyPaymentProcessor() {
  // Old code nobody uses
}

// Unused variable
const FEATURE_FLAG_OLD = false;
```

**Refactoring**: Remove Dead Code

```typescript
// ✅ Refactored: Delete it
function processPayment(amount: number) {
  if (amount <= 0) {
    throw new Error('Invalid amount');
  }
  
  const result = charge(amount);
  sendReceipt(result);
  updateAnalytics(result);
  return result;
}
```

**Prevention**: Use your IDE's "find unused" feature regularly.

---

### Speculative Generality

**Symptoms**:
- Abstract classes with only one implementation
- Parameters or methods that are never used
- "We might need this someday" code
- Complex framework for simple problem

```typescript
// ❌ Smell: Over-engineered for imaginary requirements
interface PaymentProcessor {
  process(payment: Payment): Promise<Result>;
  refund(paymentId: string): Promise<Result>;
  void(paymentId: string): Promise<Result>;
  partialRefund(paymentId: string, amount: number): Promise<Result>;
  recurring(subscription: Subscription): Promise<Result>;
}

// Only implementation
class StripeProcessor implements PaymentProcessor {
  // We only use process() and refund()
  // Other methods throw "not implemented"
  process(payment: Payment) { /* ... */ }
  refund(paymentId: string) { /* ... */ }
  void() { throw new Error('Not implemented'); }
  partialRefund() { throw new Error('Not implemented'); }
  recurring() { throw new Error('Not implemented'); }
}
```

**Refactoring**: Collapse Hierarchy, Remove Parameter

```typescript
// ✅ Refactored: Only what's actually needed
class PaymentService {
  constructor(private stripe: Stripe) {}
  
  async charge(amount: number, customer: string) {
    return this.stripe.charges.create({ amount, customer });
  }
  
  async refund(chargeId: string) {
    return this.stripe.refunds.create({ charge: chargeId });
  }
}
```

**Prevention**: YAGNI—You Aren't Gonna Need It.

---

### Duplicate Code

**Symptoms**:
- Same code structure in multiple places
- Copy-pasted code with minor variations
- Parallel inheritance hierarchies

```typescript
// ❌ Smell: Same logic in multiple places
class AdminController {
  async getUsers(req: Request) {
    const page = parseInt(req.query.page as string) || 1;
    const limit = parseInt(req.query.limit as string) || 10;
    const offset = (page - 1) * limit;
    
    const users = await db.users.findMany({ skip: offset, take: limit });
    const total = await db.users.count();
    
    return {
      data: users,
      meta: { page, limit, total, pages: Math.ceil(total / limit) }
    };
  }
}

class ProductController {
  async getProducts(req: Request) {
    const page = parseInt(req.query.page as string) || 1;
    const limit = parseInt(req.query.limit as string) || 10;
    const offset = (page - 1) * limit;
    
    const products = await db.products.findMany({ skip: offset, take: limit });
    const total = await db.products.count();
    
    return {
      data: products,
      meta: { page, limit, total, pages: Math.ceil(total / limit) }
    };
  }
}
```

**Refactoring**: Extract Function, Extract Class

```typescript
// ✅ Refactored: Shared pagination logic
interface PaginationParams {
  page: number;
  limit: number;
}

function parsePagination(query: Record<string, string>): PaginationParams {
  return {
    page: parseInt(query.page) || 1,
    limit: parseInt(query.limit) || 10,
  };
}

async function paginate<T>(
  query: { findMany: Function; count: Function },
  params: PaginationParams
) {
  const { page, limit } = params;
  const offset = (page - 1) * limit;
  
  const [data, total] = await Promise.all([
    query.findMany({ skip: offset, take: limit }),
    query.count(),
  ]);
  
  return {
    data,
    meta: { page, limit, total, pages: Math.ceil(total / limit) }
  };
}

class AdminController {
  async getUsers(req: Request) {
    return paginate(db.users, parsePagination(req.query));
  }
}

class ProductController {
  async getProducts(req: Request) {
    return paginate(db.products, parsePagination(req.query));
  }
}
```

**Prevention**: If you copy-paste, extract immediately.

---

## Couplers

Code with excessive coupling between classes.

### Message Chains

**Symptoms**:
- Long chains of method calls: `a.getB().getC().getD().doSomething()`
- Client depends on navigation structure
- Changes to intermediate objects break clients

```typescript
// ❌ Smell: Reaching through objects
function getManagerEmail(employee: Employee): string {
  return employee
    .getDepartment()
    .getManager()
    .getContactInfo()
    .getEmail()
    .toLowerCase();
}

function sendReport(order: Order) {
  const warehouse = order
    .getShipment()
    .getRoute()
    .getDestination()
    .getWarehouse();
    
  warehouse.receive(order);
}
```

**Refactoring**: Hide Delegate

```typescript
// ✅ Refactored: Ask, don't navigate
class Employee {
  getManagerEmail(): string {
    return this.department.getManagerEmail();
  }
}

class Department {
  getManagerEmail(): string {
    return this.manager.email.toLowerCase();
  }
}

// Client is simple
function getManagerEmail(employee: Employee): string {
  return employee.getManagerEmail();
}

// Or use delegation
class Order {
  getDestinationWarehouse(): Warehouse {
    return this.shipment.getDestinationWarehouse();
  }
}
```

**Prevention**: Follow the Law of Demeter—only talk to immediate friends.

---

### Middle Man

**Symptoms**:
- Class mostly delegates to another class
- No added value, just passing through
- Interface mirrors another class

```typescript
// ❌ Smell: Pure delegation
class PersonWrapper {
  private person: Person;
  
  getName() { return this.person.getName(); }
  setName(name: string) { this.person.setName(name); }
  getAge() { return this.person.getAge(); }
  setAge(age: number) { this.person.setAge(age); }
  getAddress() { return this.person.getAddress(); }
  setAddress(addr: Address) { this.person.setAddress(addr); }
  // Every method just delegates
}
```

**Refactoring**: Remove Middle Man

```typescript
// ✅ Refactored: Use the class directly
// Delete PersonWrapper, use Person directly

// If some delegation is valuable, keep only that:
class PersonView {
  constructor(private person: Person) {}
  
  // Only expose what's needed, add value
  getDisplayName(): string {
    return `${this.person.getName()} (${this.person.getAge()})`;
  }
}
```

**Prevention**: Only add indirection when it provides value.

---

## Quick Reference

| Category | Smell | Fix |
|----------|-------|-----|
| **Bloaters** | Long Method | Extract Method |
| | Large Class | Extract Class |
| | Long Parameter List | Parameter Object |
| | Data Clumps | Extract Class |
| | Primitive Obsession | Replace with Object |
| **OO Abusers** | Switch Statements | Polymorphism |
| | Temporary Field | Extract Class |
| **Change Preventers** | Divergent Change | Extract Class |
| | Shotgun Surgery | Move Method |
| | Feature Envy | Move Method |
| **Dispensables** | Dead Code | Delete it |
| | Speculative Generality | Delete it |
| | Duplicate Code | Extract |
| **Couplers** | Message Chains | Hide Delegate |
| | Middle Man | Remove it |
# Refactoring Catalog

Essential refactoring patterns with before/after examples. Each refactoring is a small, reversible transformation that improves code structure without changing behavior.

Based on Martin Fowler's refactoring catalog with TypeScript examples.

---

## Composing Methods

Break down large methods into smaller, focused pieces.

### Extract Function

**Motivation**: A code fragment that can be grouped together and named. The most common refactoring.

**When to use**:
- Code block has a comment explaining what it does
- Same code appears in multiple places
- Function is too long (20+ lines)
- Logic is at a different level of abstraction

```typescript
// Before
function printInvoice(invoice: Invoice) {
  console.log('=================');
  console.log('=== INVOICE =====');
  console.log('=================');
  
  // Print details
  console.log(`Customer: ${invoice.customer}`);
  console.log(`Date: ${invoice.date}`);
  
  // Calculate total
  let total = 0;
  for (const item of invoice.items) {
    total += item.price * item.quantity;
  }
  console.log(`Total: $${total}`);
  
  console.log('=================');
}
```

```typescript
// After
function printInvoice(invoice: Invoice) {
  printHeader();
  printDetails(invoice);
  printTotal(invoice);
  printFooter();
}

function printHeader() {
  console.log('=================');
  console.log('=== INVOICE =====');
  console.log('=================');
}

function printDetails(invoice: Invoice) {
  console.log(`Customer: ${invoice.customer}`);
  console.log(`Date: ${invoice.date}`);
}

function printTotal(invoice: Invoice) {
  const total = calculateTotal(invoice.items);
  console.log(`Total: $${total}`);
}

function calculateTotal(items: InvoiceItem[]): number {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}

function printFooter() {
  console.log('=================');
}
```

**Mechanics**:
1. Create new function named after what it does (not how)
2. Copy the code fragment to the new function
3. Pass any needed variables as parameters
4. Replace the original code with a call to the new function

---

### Inline Function

**Motivation**: The opposite of Extract Function. The function body is as clear as the name, or the function is only delegating.

**When to use**:
- Function body is as obvious as its name
- You have a group of badly factored functions and want to re-extract differently
- Too much indirection

```typescript
// Before
function moreThanFiveOrders(customer: Customer): boolean {
  return customer.orders.length > 5;
}

function getDiscount(customer: Customer): number {
  if (moreThanFiveOrders(customer)) {
    return 0.1;
  }
  return 0;
}
```

```typescript
// After
function getDiscount(customer: Customer): number {
  if (customer.orders.length > 5) {
    return 0.1;
  }
  return 0;
}
```

**Mechanics**:
1. Check function isn't polymorphic (overridden in subclasses)
2. Find all callers
3. Replace each call with the function body
4. Delete the function

---

### Replace Temp with Query

**Motivation**: Temporary variables can be a problem—they make functions longer and block Extract Function.

**When to use**:
- A temp is assigned once and used multiple times
- The calculation could be a method
- You want to extract part of a function

```typescript
// Before
function getPrice(order: Order): number {
  const basePrice = order.quantity * order.itemPrice;
  const discount = Math.max(0, order.quantity - 100) * order.itemPrice * 0.05;
  const shipping = Math.min(basePrice * 0.1, 50);
  return basePrice - discount + shipping;
}
```

```typescript
// After
function getPrice(order: Order): number {
  return basePrice(order) - discount(order) + shipping(order);
}

function basePrice(order: Order): number {
  return order.quantity * order.itemPrice;
}

function discount(order: Order): number {
  return Math.max(0, order.quantity - 100) * order.itemPrice * 0.05;
}

function shipping(order: Order): number {
  return Math.min(basePrice(order) * 0.1, 50);
}
```

**Mechanics**:
1. Check the temp is only assigned once
2. Extract the assignment into a function
3. Replace references to the temp with function calls
4. Remove the temp declaration

---

### Introduce Explaining Variable

**Motivation**: Complex expressions are hard to read. Break them into named pieces.

**When to use**:
- A complex expression that's hard to understand
- Multiple conditions in an if statement
- Mathematical formulas

```typescript
// Before
function calculatePrice(order: Order): number {
  return order.quantity * order.itemPrice -
    Math.max(0, order.quantity - 100) * order.itemPrice * 0.05 +
    Math.min(order.quantity * order.itemPrice * 0.1, 50);
}

function isEligibleForDiscount(user: User): boolean {
  return user.age >= 65 || 
    (user.memberSince < new Date('2020-01-01') && user.purchaseCount > 10) ||
    user.isEmployee;
}
```

```typescript
// After
function calculatePrice(order: Order): number {
  const basePrice = order.quantity * order.itemPrice;
  const quantityDiscount = Math.max(0, order.quantity - 100) * order.itemPrice * 0.05;
  const shipping = Math.min(basePrice * 0.1, 50);
  return basePrice - quantityDiscount + shipping;
}

function isEligibleForDiscount(user: User): boolean {
  const isSenior = user.age >= 65;
  const isLoyalCustomer = user.memberSince < new Date('2020-01-01') && 
                          user.purchaseCount > 10;
  const isEmployee = user.isEmployee;
  
  return isSenior || isLoyalCustomer || isEmployee;
}
```

**Mechanics**:
1. Identify a complex expression or sub-expression
2. Create a variable with a meaningful name
3. Assign the expression to the variable
4. Replace the expression with the variable

---

## Moving Features

Move code to where it belongs.

### Move Function

**Motivation**: Functions should live with the data they use most.

**When to use**:
- Function uses more features of another class
- Function is in the wrong module
- Coupling would be reduced by moving

```typescript
// Before - calculateInterest uses account data, not the calculator's
class InterestCalculator {
  calculateInterest(account: Account, days: number): number {
    const balance = account.getBalance();
    const rate = account.getInterestRate();
    const type = account.getType();
    
    if (type === 'savings') {
      return balance * rate * days / 365;
    } else if (type === 'checking') {
      return balance * (rate - 0.01) * days / 365;
    }
    return 0;
  }
}
```

```typescript
// After - method moved to Account where the data lives
class Account {
  private balance: number;
  private interestRate: number;
  private type: 'savings' | 'checking';
  
  calculateInterest(days: number): number {
    if (this.type === 'savings') {
      return this.balance * this.interestRate * days / 365;
    } else if (this.type === 'checking') {
      return this.balance * (this.interestRate - 0.01) * days / 365;
    }
    return 0;
  }
}
```

**Mechanics**:
1. Look at what the function references—does it use more from elsewhere?
2. Check if it should be a method on one of its arguments
3. Move the function
4. Update all callers

---

### Extract Class

**Motivation**: A class is doing too much. Split it based on responsibilities.

**When to use**:
- Class has too many fields
- Class has too many methods
- Subsets of data/methods are used together
- Class name includes "And" or "Manager"

```typescript
// Before - Person has phone-related responsibilities mixed in
class Person {
  name: string;
  
  // These belong together
  officeAreaCode: string;
  officeNumber: string;
  
  // And these
  homeAreaCode: string;
  homeNumber: string;
  
  getOfficePhone(): string {
    return `(${this.officeAreaCode}) ${this.officeNumber}`;
  }
  
  getHomePhone(): string {
    return `(${this.homeAreaCode}) ${this.homeNumber}`;
  }
}
```

```typescript
// After - Phone is its own class
class PhoneNumber {
  constructor(
    private areaCode: string,
    private number: string
  ) {}
  
  format(): string {
    return `(${this.areaCode}) ${this.number}`;
  }
}

class Person {
  name: string;
  officePhone: PhoneNumber;
  homePhone: PhoneNumber;
  
  getOfficePhone(): string {
    return this.officePhone.format();
  }
  
  getHomePhone(): string {
    return this.homePhone.format();
  }
}
```

**Mechanics**:
1. Identify a subset of data and methods that belong together
2. Create a new class
3. Move fields and methods to the new class
4. Create a link from old class to new class

---

### Hide Delegate

**Motivation**: Reduce coupling by hiding the object structure from clients.

**When to use**:
- Clients navigate through one object to get to another
- Changes to the delegate affect all clients
- You're exposing internal structure

```typescript
// Before - Client knows about internal structure
class Person {
  department: Department;
}

class Department {
  manager: Person;
}

// Client code - knows too much about structure
const manager = person.department.manager;
```

```typescript
// After - Person hides the delegation
class Person {
  private department: Department;
  
  getManager(): Person {
    return this.department.manager;
  }
}

// Client code - simpler, less coupled
const manager = person.getManager();
```

**Mechanics**:
1. Create a delegating method on the server
2. Replace client calls to the delegate with calls to the server
3. Consider making the delegate field private

---

## Organizing Data

Improve how data is structured and accessed.

### Replace Magic Number with Constant

**Motivation**: Magic numbers obscure meaning and are easy to mistype.

**When to use**:
- Numbers appear in code without explanation
- Same number appears in multiple places
- The number has domain meaning

```typescript
// Before
function potentialEnergy(mass: number, height: number): number {
  return mass * height * 9.81;
}

function calculateDiscount(total: number): number {
  if (total > 100) {
    return total * 0.1;
  }
  return 0;
}
```

```typescript
// After
const GRAVITATIONAL_CONSTANT = 9.81;
const DISCOUNT_THRESHOLD = 100;
const DISCOUNT_RATE = 0.1;

function potentialEnergy(mass: number, height: number): number {
  return mass * height * GRAVITATIONAL_CONSTANT;
}

function calculateDiscount(total: number): number {
  if (total > DISCOUNT_THRESHOLD) {
    return total * DISCOUNT_RATE;
  }
  return 0;
}
```

**Mechanics**:
1. Create a constant with a meaningful name
2. Replace the magic number with the constant
3. Search for other occurrences of the same number

---

### Replace Primitive with Object

**Motivation**: Primitives with behavior should be objects.

**When to use**:
- Same validation logic for a primitive appears multiple times
- Formatting/parsing logic is scattered
- The value has business rules

```typescript
// Before - phone validation scattered everywhere
function validateOrder(order: Order) {
  const phone = order.phone;
  if (!phone.match(/^\d{10}$/)) {
    throw new Error('Invalid phone');
  }
}

function formatPhone(phone: string): string {
  return `(${phone.slice(0,3)}) ${phone.slice(3,6)}-${phone.slice(6)}`;
}
```

```typescript
// After - PhoneNumber encapsulates all behavior
class PhoneNumber {
  private readonly value: string;
  
  constructor(phone: string) {
    if (!phone.match(/^\d{10}$/)) {
      throw new Error('Invalid phone number');
    }
    this.value = phone;
  }
  
  format(): string {
    return `(${this.value.slice(0,3)}) ${this.value.slice(3,6)}-${this.value.slice(6)}`;
  }
  
  getAreaCode(): string {
    return this.value.slice(0, 3);
  }
  
  toString(): string {
    return this.value;
  }
}

// Usage
const phone = new PhoneNumber('5551234567');
console.log(phone.format()); // (555) 123-4567
```

**Mechanics**:
1. Create a class for the value
2. Add validation in constructor
3. Add any formatting/parsing methods
4. Replace usages of the primitive

---

### Encapsulate Collection

**Motivation**: Exposing a collection allows clients to modify it without the owner knowing.

**When to use**:
- A getter returns a raw collection
- Clients are adding/removing items directly
- Collection modifications should trigger other behavior

```typescript
// Before - collection exposed directly
class Course {
  name: string;
}

class Person {
  courses: Course[] = [];
  
  getCourses(): Course[] {
    return this.courses; // Returns mutable reference!
  }
}

// Client can bypass the Person
person.getCourses().push(newCourse);
person.getCourses().length = 0; // Dangerous!
```

```typescript
// After - collection encapsulated
class Person {
  private courses: Course[] = [];
  
  getCourses(): readonly Course[] {
    return [...this.courses]; // Return copy
  }
  
  addCourse(course: Course): void {
    this.courses.push(course);
    // Can add validation, events, logging here
  }
  
  removeCourse(course: Course): void {
    const index = this.courses.indexOf(course);
    if (index > -1) {
      this.courses.splice(index, 1);
    }
  }
  
  get numberOfCourses(): number {
    return this.courses.length;
  }
}
```

**Mechanics**:
1. Add methods for modifying the collection
2. Return a copy or readonly view from the getter
3. Replace direct modifications with method calls

---

## Simplifying Conditionals

Make conditional logic easier to understand.

### Decompose Conditional

**Motivation**: Complex conditionals are hard to read. Extract into well-named functions.

**When to use**:
- Conditional has complex conditions
- Then/else branches have substantial code
- The logic isn't immediately clear

```typescript
// Before
function calculateCharge(date: Date, quantity: number): number {
  if (date.getMonth() >= 5 && date.getMonth() <= 8) {
    return quantity * 1.2 + (quantity > 100 ? quantity * 0.05 : 0);
  } else {
    return quantity * 1.0 + (quantity > 50 ? quantity * 0.03 : 0);
  }
}
```

```typescript
// After
function calculateCharge(date: Date, quantity: number): number {
  if (isSummer(date)) {
    return summerCharge(quantity);
  } else {
    return regularCharge(quantity);
  }
}

function isSummer(date: Date): boolean {
  return date.getMonth() >= 5 && date.getMonth() <= 8;
}

function summerCharge(quantity: number): number {
  const baseCharge = quantity * 1.2;
  const bulkDiscount = quantity > 100 ? quantity * 0.05 : 0;
  return baseCharge + bulkDiscount;
}

function regularCharge(quantity: number): number {
  const baseCharge = quantity * 1.0;
  const bulkDiscount = quantity > 50 ? quantity * 0.03 : 0;
  return baseCharge + bulkDiscount;
}
```

**Mechanics**:
1. Extract the condition into a function
2. Extract the then-branch into a function
3. Extract the else-branch into a function

---

### Consolidate Conditional Expression

**Motivation**: Multiple conditions with the same result should be combined.

**When to use**:
- Several conditions return the same value
- The conditions are really checking one thing
- Combining makes intent clearer

```typescript
// Before
function disabilityAmount(employee: Employee): number {
  if (employee.seniority < 2) return 0;
  if (employee.monthsDisabled > 12) return 0;
  if (employee.isPartTime) return 0;
  // Calculate disability
  return employee.salary * 0.6;
}
```

```typescript
// After
function disabilityAmount(employee: Employee): number {
  if (isNotEligibleForDisability(employee)) return 0;
  return employee.salary * 0.6;
}

function isNotEligibleForDisability(employee: Employee): boolean {
  return employee.seniority < 2 ||
         employee.monthsDisabled > 12 ||
         employee.isPartTime;
}
```

**Mechanics**:
1. Combine conditions using logical operators
2. Extract the combined condition into a function
3. Give it a meaningful name

---

### Replace Nested Conditional with Guard Clauses

**Motivation**: Deeply nested conditionals are hard to follow. Use early returns.

**When to use**:
- Deep nesting (more than 2 levels)
- Some branches are exceptional/edge cases
- Happy path is buried in else clauses

```typescript
// Before - nested pyramid
function getPayAmount(employee: Employee): number {
  let result: number;
  if (employee.isSeparated) {
    result = 0;
  } else {
    if (employee.isRetired) {
      result = employee.pension;
    } else {
      if (employee.isOnLeave) {
        result = employee.salary * 0.5;
      } else {
        result = employee.salary;
      }
    }
  }
  return result;
}
```

```typescript
// After - flat with guard clauses
function getPayAmount(employee: Employee): number {
  if (employee.isSeparated) return 0;
  if (employee.isRetired) return employee.pension;
  if (employee.isOnLeave) return employee.salary * 0.5;
  return employee.salary;
}
```

**Mechanics**:
1. Identify edge cases
2. Replace each with a guard clause (early return)
3. Remove unnecessary else clauses

---

### Replace Conditional with Polymorphism

**Motivation**: Type-based conditionals often indicate missing polymorphism.

**When to use**:
- Switching on type code
- Same switch appears in multiple places
- Each case has substantially different behavior

```typescript
// Before
type BirdType = 'european' | 'african' | 'norwegian_blue';

function getSpeed(bird: { type: BirdType; voltage?: number }): number {
  switch (bird.type) {
    case 'european':
      return 35;
    case 'african':
      return 40 - 2 * bird.numberOfCoconuts;
    case 'norwegian_blue':
      return bird.voltage > 100 ? 20 : 0;
    default:
      return 0;
  }
}

function getPlumage(bird: { type: BirdType }): string {
  switch (bird.type) {
    case 'european':
      return 'average';
    case 'african':
      return bird.numberOfCoconuts > 2 ? 'tired' : 'average';
    case 'norwegian_blue':
      return bird.voltage > 100 ? 'scorched' : 'beautiful';
    default:
      return 'unknown';
  }
}
```

```typescript
// After
interface Bird {
  getSpeed(): number;
  getPlumage(): string;
}

class EuropeanSwallow implements Bird {
  getSpeed() { return 35; }
  getPlumage() { return 'average'; }
}

class AfricanSwallow implements Bird {
  constructor(private numberOfCoconuts: number) {}
  
  getSpeed() { return 40 - 2 * this.numberOfCoconuts; }
  getPlumage() { return this.numberOfCoconuts > 2 ? 'tired' : 'average'; }
}

class NorwegianBlueParrot implements Bird {
  constructor(private voltage: number) {}
  
  getSpeed() { return this.voltage > 100 ? 20 : 0; }
  getPlumage() { return this.voltage > 100 ? 'scorched' : 'beautiful'; }
}

// Factory to create the right type
function createBird(data: BirdData): Bird {
  switch (data.type) {
    case 'european': return new EuropeanSwallow();
    case 'african': return new AfricanSwallow(data.numberOfCoconuts);
    case 'norwegian_blue': return new NorwegianBlueParrot(data.voltage);
  }
}
```

**Mechanics**:
1. Create interface/base class
2. Create a class for each type
3. Move switch logic into each class
4. Replace switch with polymorphic call

---

## Simplifying Function Calls

Make functions easier to call and understand.

### Rename Function

**Motivation**: Good names are the best documentation. If you can't name it, you don't understand it.

**When to use**:
- Name doesn't describe what the function does
- Name is misleading
- Name is too technical for the domain

```typescript
// Before - unclear names
function calc(a: number, b: number): number { /* ... */ }
function process(data: unknown): void { /* ... */ }
function handle(event: Event): void { /* ... */ }
function inv(customer: Customer): Invoice { /* ... */ }
```

```typescript
// After - descriptive names
function calculateCompoundInterest(principal: number, rate: number): number { /* ... */ }
function validateAndSaveUserProfile(profile: UserProfile): void { /* ... */ }
function trackButtonClick(event: MouseEvent): void { /* ... */ }
function generateInvoiceForCustomer(customer: Customer): Invoice { /* ... */ }
```

**Mechanics**:
1. Choose a better name (this is the hard part)
2. Create new function with new name
3. Copy body to new function
4. Update all callers
5. Remove old function

---

### Introduce Parameter Object

**Motivation**: Groups of parameters that travel together should be a single object.

**When to use**:
- Same group of parameters appears in multiple functions
- Parameters have a clear relationship
- You want to add behavior to the group

```typescript
// Before - date range parameters everywhere
function getTotalSales(startDate: Date, endDate: Date): number { /* ... */ }
function getAverageOrders(startDate: Date, endDate: Date): number { /* ... */ }
function generateReport(startDate: Date, endDate: Date, format: string): Report { /* ... */ }
```

```typescript
// After - DateRange object
class DateRange {
  constructor(
    readonly start: Date,
    readonly end: Date
  ) {
    if (start > end) throw new Error('Invalid date range');
  }
  
  contains(date: Date): boolean {
    return date >= this.start && date <= this.end;
  }
  
  get durationInDays(): number {
    const ms = this.end.getTime() - this.start.getTime();
    return Math.ceil(ms / (1000 * 60 * 60 * 24));
  }
}

function getTotalSales(dateRange: DateRange): number { /* ... */ }
function getAverageOrders(dateRange: DateRange): number { /* ... */ }
function generateReport(dateRange: DateRange, format: string): Report { /* ... */ }
```

**Mechanics**:
1. Create a class for the parameter group
2. Add validation in constructor
3. Add any relevant methods
4. Replace parameter lists with the new object

---

### Replace Parameter with Method

**Motivation**: A parameter that can be derived from other available information is redundant.

**When to use**:
- Parameter value is derived from something else
- Parameter requires complex calculation by caller
- You're passing internal state back in

```typescript
// Before - caller computes discountRate
function getFinalPrice(
  basePrice: number,
  discountLevel: number,
  discountRate: number
): number {
  return basePrice - (basePrice * discountRate);
}

// Caller has to know how to calculate rate
const rate = getDiscountRate(customer.discountLevel);
const price = getFinalPrice(basePrice, customer.discountLevel, rate);
```

```typescript
// After - function derives discountRate internally
function getFinalPrice(basePrice: number, discountLevel: number): number {
  const discountRate = getDiscountRate(discountLevel);
  return basePrice - (basePrice * discountRate);
}

// Caller is simpler
const price = getFinalPrice(basePrice, customer.discountLevel);
```

**Mechanics**:
1. Extract the parameter calculation into a method
2. Call that method inside the function
3. Remove the parameter

---

## Quick Reference

| Category | Refactoring | When to Use |
|----------|-------------|-------------|
| **Composing Methods** | Extract Function | Code block can be named |
| | Inline Function | Body is clear as name |
| | Replace Temp with Query | Temp blocks extraction |
| | Introduce Explaining Variable | Complex expression |
| **Moving Features** | Move Function | Uses other class's data |
| | Extract Class | Class does too much |
| | Hide Delegate | Clients navigate too deep |
| **Organizing Data** | Replace Magic Number | Unexplained numbers |
| | Replace Primitive with Object | Primitive has behavior |
| | Encapsulate Collection | Exposed mutable collection |
| **Simplifying Conditionals** | Decompose Conditional | Complex if-then-else |
| | Consolidate Conditional | Same result, multiple checks |
| | Guard Clauses | Deep nesting |
| | Replace with Polymorphism | Switch on type |
| **Function Calls** | Rename Function | Name doesn't fit |
| | Parameter Object | Params travel together |
| | Replace Parameter with Method | Param can be derived |

---

## Refactoring Workflow

1. **Ensure tests pass** before starting
2. **Make one small change** at a time
3. **Run tests** after each change
4. **Commit frequently** so you can revert
5. **Never refactor and add features** in the same commit
