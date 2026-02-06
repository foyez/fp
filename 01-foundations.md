# Functional Programming: Chapter 1 - Core Concepts & Foundations

## Table of Contents

- [What is Functional Programming?](#what-is-functional-programming)
- [Pure Functions](#pure-functions)
- [Immutability](#immutability)
- [First-Class & Higher-Order Functions](#first-class--higher-order-functions)
- [Function Composition](#function-composition)
- [Declarative vs Imperative](#declarative-vs-imperative)

---

## What is Functional Programming?

Functional Programming (FP) is a programming paradigm that treats computation as the evaluation of mathematical functions and avoids changing state and mutable data.

**Core Principles:**
- **Pure Functions**: Same input → Same output, no side effects
- **Immutability**: Data doesn't change once created
- **Function Composition**: Build complex operations from simple functions
- **Declarative Style**: Focus on *what* to do, not *how*

**Real-Life Analogy:**  
Think of a recipe. A pure function is like a recipe that always produces the same cake given the same ingredients. The ingredients don't change (immutability), and combining recipes (composition) creates more complex dishes.

---

## Pure Functions

A pure function:
1. Returns the same output for the same input (deterministic)
2. Has no side effects (doesn't modify external state, I/O, etc.)

### Why Pure Functions Matter

**Real-World Scenario: E-commerce Price Calculator**

<details>
<summary><strong>View Examples</strong></summary>

```ts
// ❌ IMPURE: Depends on external state and modifies it
let taxRate = 0.1;
let discountApplied = false;

function calculateTotalImpure(price: number): number {
  discountApplied = true; // Side effect!
  return price + (price * taxRate); // Depends on external state
}

// ✅ PURE: All inputs explicit, no side effects
function calculateTotal(price: number, taxRate: number, discount: number): number {
  const discountedPrice = price - (price * discount);
  return discountedPrice + (discountedPrice * taxRate);
}

// Usage
const cartTotal = calculateTotal(100, 0.1, 0.15);
console.log(cartTotal); // Always 93.5 for these inputs

// Easy to test
console.assert(calculateTotal(100, 0.1, 0.15) === 93.5);
console.assert(calculateTotal(100, 0.1, 0.15) === 93.5); // Predictable!
```

**Go Example:**

```go
package main

import "fmt"

// ❌ IMPURE: Uses global state
var globalDiscount = 0.1

func calculateTotalImpure(price float64) float64 {
    return price - (price * globalDiscount)
}

// ✅ PURE: Explicit parameters
func calculateTotal(price, discount, taxRate float64) float64 {
    discountedPrice := price - (price * discount)
    return discountedPrice + (discountedPrice * taxRate)
}

func main() {
    total := calculateTotal(100, 0.15, 0.1)
    fmt.Printf("Total: %.2f\n", total) // 93.50
}
```

</details>

### Benefits of Pure Functions

1. **Testability**: No setup/teardown needed
2. **Cacheable**: Same inputs = same outputs (memoization)
3. **Parallel Execution**: No race conditions
4. **Easier Debugging**: No hidden dependencies

---

## Immutability

Data cannot be modified after creation. Instead, create new versions.

**Real-World Scenario: User Profile Updates**

<details>
<summary><strong>View Examples</strong></summary>

```ts
// ❌ MUTABLE: Modifying original data
interface User {
  id: number;
  name: string;
  email: string;
  preferences: { theme: string; notifications: boolean };
}

function updateEmailMutable(user: User, newEmail: string): void {
  user.email = newEmail; // Mutates original!
}

// ✅ IMMUTABLE: Creating new objects
function updateEmail(user: User, newEmail: string): User {
  return {
    ...user,
    email: newEmail
  };
}

function updatePreferences(user: User, theme: string): User {
  return {
    ...user,
    preferences: {
      ...user.preferences,
      theme
    }
  };
}

// Usage
const originalUser: User = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
  preferences: { theme: "dark", notifications: true }
};

const updatedUser = updateEmail(originalUser, "alice.smith@example.com");

console.log(originalUser.email);  // "alice@example.com" (unchanged)
console.log(updatedUser.email);   // "alice.smith@example.com"
```

**Go Example (using value semantics):**

```go
package main

import "fmt"

type User struct {
    ID    int
    Name  string
    Email string
}

// ✅ Returns new User, doesn't modify original
func updateEmail(user User, newEmail string) User {
    // In Go, structs are copied by value
    updatedUser := user
    updatedUser.Email = newEmail
    return updatedUser
}

func main() {
    original := User{ID: 1, Name: "Bob", Email: "bob@example.com"}
    updated := updateEmail(original, "bob.smith@example.com")
    
    fmt.Println(original.Email) // bob@example.com (unchanged)
    fmt.Println(updated.Email)  // bob.smith@example.com
}
```

</details>

### Why Immutability?

- **Time Travel**: Keep history of states (undo/redo)
- **Concurrency**: Multiple goroutines can safely read
- **Predictability**: Data doesn't change unexpectedly
- **Debugging**: Easier to trace data flow

---

## Practice Questions

<details>
<summary><strong>View Questions</strong></summary>

### Fill in the Blanks

1. A pure function always returns the __________ output for the same input.
2. Immutability means data cannot be __________ after creation.
3. Pure functions have no __________ effects.
4. The benefit of pure functions for parallel execution is no __________ conditions.

<details>
<summary><strong>View Answers</strong></summary>

1. **same** - This property is called determinism and makes functions predictable and testable
2. **modified** - Instead of changing data, we create new versions with the desired changes
3. **side** - Side effects include modifying global state, I/O operations, or mutating external data
4. **race** - Since pure functions don't share mutable state, they can run concurrently without conflicts

</details>

---

### True/False

1. ⬜ Pure functions can modify their input parameters
2. ⬜ Immutability makes debugging easier because data flow is more predictable
3. ⬜ A function that reads from a database is a pure function
4. ⬜ Creating new objects instead of modifying existing ones always wastes memory

<details>
<summary><strong>View Answers</strong></summary>

1. **False** - Pure functions never modify their inputs; they return new values instead
2. **True** - With immutable data, you can trace exactly how values changed throughout the program
3. **False** - Database reads are I/O operations (side effects), making the function impure
4. **False** - Modern runtimes use structural sharing and garbage collection to manage memory efficiently

</details>

---

### Multiple Choice

1. Which of the following is a pure function?

```ts
// A
let counter = 0;
function increment() { return ++counter; }

// B
function add(a: number, b: number) { return a + b; }

// C
function getTime() { return Date.now(); }

// D
function logMessage(msg: string) { console.log(msg); }
```

2. What's the main benefit of immutability in concurrent programming?
   - A) Faster execution
   - B) Less memory usage
   - C) No race conditions
   - D) Easier syntax

<details>
<summary><strong>View Answers</strong></summary>

1. **B** - `add(a, b)` always returns the same result for the same inputs and has no side effects. A modifies external state, C depends on time (non-deterministic), D performs I/O (side effect)

2. **C** - No race conditions. When data is immutable, multiple threads/goroutines can safely read it simultaneously without synchronization mechanisms, eliminating race conditions

</details>

</details>

---

## First-Class & Higher-Order Functions

**First-Class Functions**: Functions are values that can be:
- Assigned to variables
- Passed as arguments
- Returned from other functions

**Higher-Order Functions**: Functions that:
- Take functions as parameters, OR
- Return functions

**Real-World Scenario: Payment Processing Pipeline**

<details>
<summary><strong>View Examples</strong></summary>

```ts
// First-class functions: Assigning to variables
type PaymentProcessor = (amount: number) => boolean;

const processStripe: PaymentProcessor = (amount) => {
  console.log(`Processing $${amount} via Stripe`);
  return amount < 10000; // Success if under limit
};

const processPayPal: PaymentProcessor = (amount) => {
  console.log(`Processing $${amount} via PayPal`);
  return amount < 5000;
};

// Higher-order function: Takes function as parameter
function withRetry(
  processor: PaymentProcessor,
  maxAttempts: number
): PaymentProcessor {
  return (amount: number) => {
    for (let attempt = 1; attempt <= maxAttempts; attempt++) {
      console.log(`Attempt ${attempt}/${maxAttempts}`);
      if (processor(amount)) {
        return true;
      }
    }
    return false;
  };
}

// Higher-order function: Returns a function
function withLogging(processor: PaymentProcessor): PaymentProcessor {
  return (amount: number) => {
    console.log(`[LOG] Processing payment of $${amount}`);
    const result = processor(amount);
    console.log(`[LOG] Result: ${result ? 'Success' : 'Failed'}`);
    return result;
  };
}

// Composing processors
const robustStripeProcessor = withLogging(withRetry(processStripe, 3));

// Usage
robustStripeProcessor(500);
```

**Go Example:**

```go
package main

import (
    "fmt"
    "log"
)

// PaymentProcessor is a function type
type PaymentProcessor func(amount float64) bool

// Higher-order function: takes function as parameter
func withLogging(processor PaymentProcessor) PaymentProcessor {
    return func(amount float64) bool {
        log.Printf("[LOG] Processing payment of $%.2f", amount)
        result := processor(amount)
        log.Printf("[LOG] Result: %v", result)
        return result
    }
}

// Higher-order function: returns a function
func withRetry(processor PaymentProcessor, maxAttempts int) PaymentProcessor {
    return func(amount float64) bool {
        for attempt := 1; attempt <= maxAttempts; attempt++ {
            fmt.Printf("Attempt %d/%d\n", attempt, maxAttempts)
            if processor(amount) {
                return true
            }
        }
        return false
    }
}

func processStripe(amount float64) bool {
    fmt.Printf("Processing $%.2f via Stripe\n", amount)
    return amount < 10000
}

func main() {
    // Compose processors
    robustProcessor := withLogging(withRetry(processStripe, 3))
    robustProcessor(500)
}
```

</details>

---

## Function Composition

Combining simple functions to build complex operations.

**Mathematical View**: `(f ∘ g)(x) = f(g(x))`

**Real-World Scenario: Data Processing Pipeline**

<details>
<summary><strong>View Examples</strong></summary>

```ts
// Individual transformations for user data
interface RawUser {
  firstName: string;
  lastName: string;
  email: string;
  age: number;
}

interface User {
  fullName: string;
  email: string;
  isAdult: boolean;
}

// Simple, focused functions
const normalizeEmail = (user: RawUser) => ({
  ...user,
  email: user.email.toLowerCase().trim()
});

const createFullName = (user: RawUser) => ({
  ...user,
  fullName: `${user.firstName} ${user.lastName}`
});

const determineAdultStatus = (user: RawUser & { fullName?: string }) => ({
  ...user,
  isAdult: user.age >= 18
});

// Compose function
function compose<T>(...fns: Array<(arg: T) => T>): (arg: T) => T {
  return (arg: T) => fns.reduceRight((acc, fn) => fn(acc), arg);
}

// Pipe function (left to right, more intuitive)
function pipe<T>(...fns: Array<(arg: T) => T>): (arg: T) => T {
  return (arg: T) => fns.reduce((acc, fn) => fn(acc), arg);
}

// Build processing pipeline
const processUser = pipe(
  normalizeEmail,
  createFullName,
  determineAdultStatus
);

// Usage
const rawUser: RawUser = {
  firstName: "John",
  lastName: "Doe",
  email: "  JOHN.DOE@EXAMPLE.COM  ",
  age: 25
};

const processedUser = processUser(rawUser);
console.log(processedUser);
// {
//   firstName: "John",
//   lastName: "Doe",
//   email: "john.doe@example.com",
//   fullName: "John Doe",
//   age: 25,
//   isAdult: true
// }

// Real-world: Process multiple users
const users: RawUser[] = [
  { firstName: "Alice", lastName: "Smith", email: "ALICE@TEST.COM", age: 17 },
  { firstName: "Bob", lastName: "Jones", email: "bob@TEST.COM  ", age: 30 }
];

const processedUsers = users.map(processUser);
console.log(processedUsers);
```

**Go Example:**

```go
package main

import (
    "fmt"
    "strings"
)

type RawUser struct {
    FirstName string
    LastName  string
    Email     string
    Age       int
}

type ProcessedUser struct {
    RawUser
    FullName string
    IsAdult  bool
}

// Transform functions
type UserTransform func(ProcessedUser) ProcessedUser

func normalizeEmail(user ProcessedUser) ProcessedUser {
    user.Email = strings.ToLower(strings.TrimSpace(user.Email))
    return user
}

func createFullName(user ProcessedUser) ProcessedUser {
    user.FullName = fmt.Sprintf("%s %s", user.FirstName, user.LastName)
    return user
}

func determineAdultStatus(user ProcessedUser) ProcessedUser {
    user.IsAdult = user.Age >= 18
    return user
}

// Pipe function
func pipe(transforms ...UserTransform) UserTransform {
    return func(user ProcessedUser) ProcessedUser {
        result := user
        for _, transform := range transforms {
            result = transform(result)
        }
        return result
    }
}

func main() {
    processUser := pipe(
        normalizeEmail,
        createFullName,
        determineAdultStatus,
    )
    
    user := ProcessedUser{
        RawUser: RawUser{
            FirstName: "John",
            LastName:  "Doe",
            Email:     "  JOHN.DOE@EXAMPLE.COM  ",
            Age:       25,
        },
    }
    
    processed := processUser(user)
    fmt.Printf("%+v\n", processed)
}
```

</details>

### Benefits of Composition

- **Reusability**: Small functions used in multiple pipelines
- **Testability**: Test each step independently
- **Readability**: Clear data transformation flow
- **Maintainability**: Easy to add/remove steps

---

## Declarative vs Imperative

**Imperative**: Tell the computer *HOW* to do something (step-by-step)  
**Declarative**: Tell the computer *WHAT* you want (the result)

**Real-World Scenario: Filtering Active Premium Users**

<details>
<summary><strong>View Examples</strong></summary>

```ts
interface Customer {
  id: number;
  name: string;
  isPremium: boolean;
  isActive: boolean;
  lastPurchaseDate: Date;
}

const customers: Customer[] = [
  { id: 1, name: "Alice", isPremium: true, isActive: true, lastPurchaseDate: new Date('2024-01-15') },
  { id: 2, name: "Bob", isPremium: false, isActive: true, lastPurchaseDate: new Date('2024-02-01') },
  { id: 3, name: "Carol", isPremium: true, isActive: false, lastPurchaseDate: new Date('2023-12-20') },
  { id: 4, name: "Dave", isPremium: true, isActive: true, lastPurchaseDate: new Date('2024-01-28') }
];

// ❌ IMPERATIVE: How to do it
function getActivePremiumCustomersImperative(customers: Customer[]): string[] {
  const result: string[] = [];
  for (let i = 0; i < customers.length; i++) {
    const customer = customers[i];
    if (customer.isPremium && customer.isActive) {
      result.push(customer.name);
    }
  }
  return result;
}

// ✅ DECLARATIVE: What we want
function getActivePremiumCustomers(customers: Customer[]): string[] {
  return customers
    .filter(c => c.isPremium && c.isActive)
    .map(c => c.name);
}

console.log(getActivePremiumCustomers(customers)); // ["Alice", "Dave"]

// More complex declarative example
const recentActivePremiumCustomers = customers
  .filter(c => c.isPremium)
  .filter(c => c.isActive)
  .filter(c => {
    const thirtyDaysAgo = new Date();
    thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);
    return c.lastPurchaseDate >= thirtyDaysAgo;
  })
  .map(c => ({ name: c.name, id: c.id }));

console.log(recentActivePremiumCustomers);
```

**Go Example:**

```go
package main

import (
    "fmt"
    "time"
)

type Customer struct {
    ID              int
    Name            string
    IsPremium       bool
    IsActive        bool
    LastPurchaseDate time.Time
}

// ❌ IMPERATIVE
func getActivePremiumCustomersImperative(customers []Customer) []string {
    result := []string{}
    for i := 0; i < len(customers); i++ {
        customer := customers[i]
        if customer.IsPremium && customer.IsActive {
            result = append(result, customer.Name)
        }
    }
    return result
}

// ✅ DECLARATIVE (using helper functions)
func filter(customers []Customer, predicate func(Customer) bool) []Customer {
    result := []Customer{}
    for _, c := range customers {
        if predicate(c) {
            result = append(result, c)
        }
    }
    return result
}

func mapToNames(customers []Customer) []string {
    names := make([]string, len(customers))
    for i, c := range customers {
        names[i] = c.Name
    }
    return names
}

func getActivePremiumCustomers(customers []Customer) []string {
    filtered := filter(customers, func(c Customer) bool {
        return c.IsPremium && c.IsActive
    })
    return mapToNames(filtered)
}

func main() {
    customers := []Customer{
        {ID: 1, Name: "Alice", IsPremium: true, IsActive: true},
        {ID: 2, Name: "Bob", IsPremium: false, IsActive: true},
        {ID: 3, Name: "Carol", IsPremium: true, IsActive: false},
        {ID: 4, Name: "Dave", IsPremium: true, IsActive: true},
    }
    
    fmt.Println(getActivePremiumCustomers(customers)) // [Alice Dave]
}
```

</details>

---

## Practice Questions

<details>
<summary><strong>View Questions</strong></summary>

### Fill in the Blanks

1. A higher-order function either takes a function as a __________ or returns a __________.
2. Function composition follows the pattern f(g(x)), reading from __________ to __________.
3. In declarative programming, we focus on __________ we want, not __________ to get it.
4. The `pipe` function executes functions from __________ to __________.

<details>
<summary><strong>View Answers</strong></summary>

1. **parameter**, **function** - Higher-order functions operate on other functions, either accepting them as inputs or producing them as outputs
2. **right**, **left** - In mathematical composition f(g(x)), g is applied first, then f. This is right-to-left execution
3. **what**, **how** - Declarative code describes the desired result rather than the step-by-step process
4. **left**, **right** - Unlike `compose`, `pipe` is more intuitive as it processes functions in reading order (left to right)

</details>

---

### True/False

1. ⬜ First-class functions can be stored in data structures like arrays
2. ⬜ Imperative code is always faster than declarative code
3. ⬜ Function composition creates tightly coupled code
4. ⬜ Higher-order functions enable code reuse and abstraction

<details>
<summary><strong>View Answers</strong></summary>

1. **True** - As first-class citizens, functions can be treated like any other value: stored in arrays, objects, passed around, etc.
2. **False** - Performance depends on implementation. Modern JavaScript engines optimize declarative methods like `map/filter`, and readability often matters more than micro-optimizations
3. **False** - Function composition creates loosely coupled code. Each function is independent and can be tested/reused separately
4. **True** - Higher-order functions let you abstract common patterns (like retry logic, logging) and reuse them across different operations

</details>

---

### Code Challenge

Create a data processing pipeline for an e-commerce order system:

```ts
interface Order {
  id: string;
  customerId: string;
  amount: number;
  status: 'pending' | 'confirmed' | 'shipped' | 'delivered';
  items: number;
}

// TODO: Create these pure functions:
// 1. filterByStatus(status: string)
// 2. addProcessingFee(feePercent: number)
// 3. sortByAmount(ascending: boolean)
// 4. extractOrderIds()

// Then compose them into: getTopPendingOrderIds(orders, limit)
// Should return IDs of the top N pending orders by amount
```

<details>
<summary><strong>View Answers</strong></summary>

```ts
interface Order {
  id: string;
  customerId: string;
  amount: number;
  status: 'pending' | 'confirmed' | 'shipped' | 'delivered';
  items: number;
}

// Individual pure functions
const filterByStatus = (status: Order['status']) => 
  (orders: Order[]): Order[] => 
    orders.filter(o => o.status === status);

const addProcessingFee = (feePercent: number) => 
  (orders: Order[]): Order[] =>
    orders.map(o => ({
      ...o,
      amount: o.amount * (1 + feePercent / 100)
    }));

const sortByAmount = (ascending: boolean) => 
  (orders: Order[]): Order[] =>
    [...orders].sort((a, b) => 
      ascending ? a.amount - b.amount : b.amount - a.amount
    );

const extractOrderIds = (orders: Order[]): string[] =>
  orders.map(o => o.id);

const takeLimit = (limit: number) => 
  <T>(items: T[]): T[] => 
    items.slice(0, limit);

// Compose into pipeline
function pipe<T>(...fns: Array<(arg: any) => any>) {
  return (arg: T) => fns.reduce((acc, fn) => fn(acc), arg);
}

const getTopPendingOrderIds = (orders: Order[], limit: number): string[] =>
  pipe(
    filterByStatus('pending'),
    addProcessingFee(5), // Add 5% processing fee
    sortByAmount(false), // Descending (highest first)
    takeLimit(limit),
    extractOrderIds
  )(orders);

// Test
const orders: Order[] = [
  { id: 'A1', customerId: 'C1', amount: 100, status: 'pending', items: 2 },
  { id: 'A2', customerId: 'C2', amount: 500, status: 'confirmed', items: 5 },
  { id: 'A3', customerId: 'C3', amount: 200, status: 'pending', items: 3 },
  { id: 'A4', customerId: 'C4', amount: 150, status: 'pending', items: 1 },
];

console.log(getTopPendingOrderIds(orders, 2)); // ['A3', 'A4'] (with fees: 210, 157.5)
```

**Explanation**: Each function is pure and focused on one task. The pipeline is readable left-to-right, and each step can be tested independently. This is real FP in action!

</details>

</details>

---