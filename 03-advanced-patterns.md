# Functional Programming: Chapter 3 - Advanced Patterns

## Table of Contents

- [Functors & Monads](#functors--monads)
- [Algebraic Data Types](#algebraic-data-types)
- [Pattern Matching](#pattern-matching)
- [Lazy Evaluation](#lazy-evaluation)

---

## Functors & Monads

These are advanced FP concepts that help manage complexity when dealing with values in contexts (like nullable values, async operations, error handling).

**Important Note**: TypeScript and Go don't have native support for these patterns like pure FP languages (Haskell, Scala), but we can implement the concepts.

---

### Functors

**Definition**: A container that you can `map` over while preserving structure.

**Simple Rule**: If something has a `map` function that applies a transformation without changing the container type, it's a functor.

**Real-Life Analogy**:  
Think of a gift box. You can change what's inside the box (map over it), but it's still a box. Arrays are functors - you can map over them and get an array back.

**Real-World Scenario: Optional Values (Maybe/Option Pattern)**

<details>
<summary><strong>View Examples</strong></summary>

```ts
// The problem: Dealing with null/undefined
interface User {
  id: string;
  name: string;
  email?: string;
  address?: {
    street?: string;
    city?: string;
  };
}

// ❌ Traditional approach - null checking everywhere
function getUserCity(user: User | null): string | null {
  if (user === null) return null;
  if (user.address === undefined) return null;
  if (user.address.city === undefined) return null;
  return user.address.city;
}

// ✅ Functor approach: Maybe/Option
class Maybe<T> {
  private constructor(private value: T | null) {}
  
  static of<T>(value: T | null): Maybe<T> {
    return new Maybe(value);
  }
  
  static none<T>(): Maybe<T> {
    return new Maybe<T>(null);
  }
  
  isNone(): boolean {
    return this.value === null || this.value === undefined;
  }
  
  // Functor: map preserves the Maybe structure
  map<U>(fn: (value: T) => U): Maybe<U> {
    if (this.isNone()) {
      return Maybe.none<U>();
    }
    return Maybe.of(fn(this.value!));
  }
  
  // Chain for nested Maybes (flatMap)
  flatMap<U>(fn: (value: T) => Maybe<U>): Maybe<U> {
    if (this.isNone()) {
      return Maybe.none<U>();
    }
    return fn(this.value!);
  }
  
  // Get value or default
  getOrElse(defaultValue: T): T {
    return this.isNone() ? defaultValue : this.value!;
  }
  
  // Get value or throw
  getOrThrow(errorMessage: string): T {
    if (this.isNone()) {
      throw new Error(errorMessage);
    }
    return this.value!;
  }
}

// Usage: User profile
const user: User = {
  id: '1',
  name: 'Alice',
  address: {
    city: 'New York'
  }
};

const userMaybe = Maybe.of(user);

// Clean chaining without null checks
const city = userMaybe
  .map(u => u.address)
  .map(addr => addr?.city)
  .getOrElse('Unknown');

console.log(city); // "New York"

// With null user
const nullUserCity = Maybe.of(null as User | null)
  .map(u => u.address)
  .map(addr => addr?.city)
  .getOrElse('Unknown');

console.log(nullUserCity); // "Unknown"

// Real-world: API response handling
interface ApiResponse<T> {
  data?: T;
  error?: string;
}

function fetchUser(id: string): ApiResponse<User> {
  // Simulate API call
  if (id === '1') {
    return { data: user };
  }
  return { error: 'User not found' };
}

const response = fetchUser('1');
const email = Maybe.of(response.data)
  .map(u => u.email)
  .map(e => e?.toUpperCase())
  .getOrElse('NO EMAIL');

console.log(email);

// Chaining operations safely
const userDisplayName = Maybe.of(user)
  .map(u => u.name)
  .map(name => name.toUpperCase())
  .map(name => `Hello, ${name}!`)
  .getOrElse('Hello, Guest!');

console.log(userDisplayName); // "Hello, ALICE!"
```

**Go Example:**

```go
package main

import (
    "errors"
    "fmt"
    "strings"
)

// Maybe type
type Maybe[T any] struct {
    value *T
}

// Constructor
func Just[T any](value T) Maybe[T] {
    return Maybe[T]{value: &value}
}

func Nothing[T any]() Maybe[T] {
    return Maybe[T]{value: nil}
}

// IsNone checks if value is absent
func (m Maybe[T]) IsNone() bool {
    return m.value == nil
}

// Map applies function if value exists (Functor!)
func (m Maybe[T]) Map(fn func(T) T) Maybe[T] {
    if m.IsNone() {
        return Nothing[T]()
    }
    result := fn(*m.value)
    return Just(result)
}

// GetOrElse returns value or default
func (m Maybe[T]) GetOrElse(defaultValue T) T {
    if m.IsNone() {
        return defaultValue
    }
    return *m.value
}

// GetOrError returns value or error
func (m Maybe[T]) GetOrError(msg string) (T, error) {
    if m.IsNone() {
        var zero T
        return zero, errors.New(msg)
    }
    return *m.value, nil
}

type User struct {
    ID    string
    Name  string
    Email string
}

func main() {
    user := User{ID: "1", Name: "Alice", Email: "alice@example.com"}
    
    // Using Maybe
    userMaybe := Just(user)
    
    upperName := userMaybe.Map(func(u User) User {
        u.Name = strings.ToUpper(u.Name)
        return u
    })
    
    finalUser := upperName.GetOrElse(User{Name: "Unknown"})
    fmt.Printf("User: %+v\n", finalUser) // Name: ALICE
    
    // With nothing
    noUser := Nothing[User]()
    defaultUser := noUser.GetOrElse(User{Name: "Guest"})
    fmt.Printf("Default: %+v\n", defaultUser) // Name: Guest
}
```

</details>

---

### Monads

**Definition**: A functor with `flatMap` (also called `bind` or `chain`) that handles nested contexts.

**Simple Rule**: If you have nested contexts like `Maybe<Maybe<T>>`, a monad can flatten them.

**Real-World Scenario: Result/Either Pattern (Error Handling)**

<details>
<summary><strong>View Examples</strong></summary>

```ts
// Instead of throwing exceptions, return success or failure
class Result<T, E> {
  private constructor(
    private value?: T,
    private error?: E
  ) {}
  
  static ok<T, E>(value: T): Result<T, E> {
    return new Result<T, E>(value, undefined);
  }
  
  static err<T, E>(error: E): Result<T, E> {
    return new Result<T, E>(undefined, error);
  }
  
  isOk(): boolean {
    return this.value !== undefined;
  }
  
  isErr(): boolean {
    return this.error !== undefined;
  }
  
  // Functor: map only if success
  map<U>(fn: (value: T) => U): Result<U, E> {
    if (this.isErr()) {
      return Result.err<U, E>(this.error!);
    }
    return Result.ok<U, E>(fn(this.value!));
  }
  
  // Monad: flatMap flattens Result<Result<T, E>, E> to Result<T, E>
  flatMap<U>(fn: (value: T) => Result<U, E>): Result<U, E> {
    if (this.isErr()) {
      return Result.err<U, E>(this.error!);
    }
    return fn(this.value!);
  }
  
  // Handle errors
  mapErr<F>(fn: (error: E) => F): Result<T, F> {
    if (this.isOk()) {
      return Result.ok<T, F>(this.value!);
    }
    return Result.err<T, F>(fn(this.error!));
  }
  
  // Unwrap
  unwrap(): T {
    if (this.isErr()) {
      throw new Error(`Called unwrap on error: ${this.error}`);
    }
    return this.value!;
  }
  
  unwrapOr(defaultValue: T): T {
    return this.isOk() ? this.value! : defaultValue;
  }
  
  // Match for pattern-like handling
  match<U>(patterns: { ok: (value: T) => U; err: (error: E) => U }): U {
    return this.isOk() ? patterns.ok(this.value!) : patterns.err(this.error!);
  }
}

// Real-world: User registration flow
interface ValidationError {
  field: string;
  message: string;
}

function validateEmail(email: string): Result<string, ValidationError> {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  
  if (!email) {
    return Result.err({ field: 'email', message: 'Email is required' });
  }
  
  if (!emailRegex.test(email)) {
    return Result.err({ field: 'email', message: 'Invalid email format' });
  }
  
  return Result.ok(email);
}

function validatePassword(password: string): Result<string, ValidationError> {
  if (!password) {
    return Result.err({ field: 'password', message: 'Password is required' });
  }
  
  if (password.length < 8) {
    return Result.err({ field: 'password', message: 'Password must be at least 8 characters' });
  }
  
  return Result.ok(password);
}

function checkEmailAvailable(email: string): Result<string, ValidationError> {
  // Simulate database check
  const takenEmails = ['admin@example.com', 'user@example.com'];
  
  if (takenEmails.includes(email)) {
    return Result.err({ field: 'email', message: 'Email already taken' });
  }
  
  return Result.ok(email);
}

interface UserRegistration {
  email: string;
  password: string;
}

function registerUser(
  email: string,
  password: string
): Result<UserRegistration, ValidationError> {
  // Chain validations using flatMap (Monad power!)
  return validateEmail(email)
    .flatMap(validEmail => checkEmailAvailable(validEmail))
    .flatMap(availableEmail =>
      validatePassword(password).map(validPassword => ({
        email: availableEmail,
        password: validPassword
      }))
    );
}

// Usage
const successResult = registerUser('new@example.com', 'securePass123');
console.log(successResult.match({
  ok: user => `Registration successful: ${user.email}`,
  err: error => `Registration failed: ${error.message}`
}));
// "Registration successful: new@example.com"

const failResult = registerUser('admin@example.com', 'securePass123');
console.log(failResult.match({
  ok: user => `Success: ${user.email}`,
  err: error => `Failed on ${error.field}: ${error.message}`
}));
// "Failed on email: Email already taken"

const shortPasswordResult = registerUser('test@example.com', 'short');
console.log(shortPasswordResult.match({
  ok: user => `Success: ${user.email}`,
  err: error => `Failed on ${error.field}: ${error.message}`
}));
// "Failed on password: Password must be at least 8 characters"

// Real-world: Database operations
interface DbError {
  code: string;
  message: string;
}

function findUserById(id: string): Result<User, DbError> {
  // Simulate database query
  if (id === '1') {
    return Result.ok({ id: '1', name: 'Alice', email: 'alice@example.com' });
  }
  return Result.err({ code: 'NOT_FOUND', message: 'User not found' });
}

function updateUser(user: User, newEmail: string): Result<User, DbError> {
  // Simulate update
  if (!newEmail.includes('@')) {
    return Result.err({ code: 'INVALID_EMAIL', message: 'Invalid email' });
  }
  return Result.ok({ ...user, email: newEmail });
}

// Chain database operations
const updateResult = findUserById('1')
  .flatMap(user => updateUser(user, 'newemail@example.com'))
  .map(user => `Updated user: ${user.email}`);

console.log(updateResult.unwrapOr('Update failed'));
// "Updated user: newemail@example.com"

// Failed chain - stops at first error
const failedUpdate = findUserById('999')
  .flatMap(user => updateUser(user, 'new@example.com'))
  .map(user => `Updated: ${user.email}`);

console.log(failedUpdate.match({
  ok: msg => msg,
  err: error => `Error ${error.code}: ${error.message}`
}));
// "Error NOT_FOUND: User not found"
```

**Go Example (using generics):**

```go
package main

import (
    "fmt"
    "strings"
)

// Result type - Either success or error
type Result[T any] struct {
    value T
    error error
    isOk  bool
}

func Ok[T any](value T) Result[T] {
    return Result[T]{value: value, isOk: true}
}

func Err[T any](err error) Result[T] {
    var zero T
    return Result[T]{value: zero, error: err, isOk: false}
}

func (r Result[T]) IsOk() bool {
    return r.isOk
}

// Map - Functor
func (r Result[T]) Map(fn func(T) T) Result[T] {
    if !r.isOk {
        return Err[T](r.error)
    }
    return Ok(fn(r.value))
}

// FlatMap - Monad (flattens nested Results)
func FlatMap[T, U any](r Result[T], fn func(T) Result[U]) Result[U] {
    if !r.isOk {
        return Err[U](r.error)
    }
    return fn(r.value)
}

// Unwrap
func (r Result[T]) Unwrap() T {
    if !r.isOk {
        panic(fmt.Sprintf("called unwrap on error: %v", r.error))
    }
    return r.value
}

func (r Result[T]) UnwrapOr(defaultValue T) T {
    if r.isOk {
        return r.value
    }
    return defaultValue
}

type User struct {
    ID    string
    Email string
}

func validateEmail(email string) Result[string] {
    if !strings.Contains(email, "@") {
        return Err[string](fmt.Errorf("invalid email"))
    }
    return Ok(email)
}

func createUser(email string) Result[User] {
    return Ok(User{ID: "1", Email: email})
}

func main() {
    // Chain operations with FlatMap
    result := FlatMap(
        validateEmail("alice@example.com"),
        createUser,
    )
    
    if result.IsOk() {
        user := result.Unwrap()
        fmt.Printf("Created user: %+v\n", user)
    } else {
        fmt.Printf("Error: %v\n", result.error)
    }
    
    // Failed validation
    failResult := FlatMap(
        validateEmail("invalid-email"),
        createUser,
    )
    
    defaultUser := failResult.UnwrapOr(User{Email: "guest@example.com"})
    fmt.Printf("Result: %+v\n", defaultUser)
}
```

</details>

---

## Why Functors and Monads Matter

**Without them**:
```ts
const user = getUser(id);
if (user === null) return null;

const address = user.address;
if (address === undefined) return null;

const city = address.city;
if (city === undefined) return null;

return city.toUpperCase();
```

**With them**:
```ts
return Maybe.of(getUser(id))
  .map(u => u.address)
  .map(a => a.city)
  .map(c => c.toUpperCase())
  .getOrElse('UNKNOWN');
```

---

## Practice Questions

<details>
<summary><strong>View Questions</strong></summary>

### Fill in the Blanks

1. A functor is a container that implements a __________ method that preserves structure.
2. A monad is a functor with __________ (also called bind or chain) to flatten nested contexts.
3. The Maybe/Option monad helps handle __________ values without null checks.
4. The Result/Either monad is used for __________ handling without exceptions.

<details>
<summary><strong>View Answers</strong></summary>

1. **map** - The map method transforms the contained value while keeping it in the same container type
2. **flatMap** - FlatMap applies a function that returns a monad and flattens the result from `Monad<Monad<T>>` to `Monad<T>`
3. **nullable** (or "optional") - Maybe encapsulates the concept of presence/absence, eliminating manual null/undefined checks
4. **error** - Result represents either success or failure, making error handling explicit and composable

</details>

---

### True/False

1. ⬜ All arrays are functors because they have a map method
2. ⬜ Monads eliminate the need for error handling
3. ⬜ TypeScript has built-in Maybe and Result types
4. ⬜ Using Result instead of exceptions makes error handling explicit in function signatures

<details>
<summary><strong>View Answers</strong></summary>

1. **True** - Arrays have map that transforms elements while keeping the array structure, satisfying the functor laws
2. **False** - Monads don't eliminate errors; they make error handling more composable and explicit through types
3. **False** - TypeScript doesn't have these built-in; you need to implement them or use libraries like fp-ts
4. **True** - `function doThing(): Result<Data, Error>` explicitly shows it can fail, unlike `function doThing(): Data` which hides potential exceptions

</details>

---

### Multiple Choice

1. What's the main benefit of using Maybe over null checks?

- A) Better performance
- B) Less memory usage
- C) Composable operations without branching
- D) Automatic error recovery

2. What does flatMap do that map doesn't?

- A) Executes faster
- B) Flattens nested monads
- C) Handles errors
- D) Validates input

<details>
<summary><strong>View Answers</strong></summary>

1. **C** - Composable operations without branching. Maybe lets you chain operations cleanly without if statements at each step

2. **B** - Flattens nested monads. If `map` would create `Maybe<Maybe<T>>`, flatMap flattens it to `Maybe<T>`

</details>

---

### Code Challenge

Implement a Task monad for async operations:

```ts
// TODO: Complete the Task monad
class Task<T> {
  constructor(private computation: () => Promise<T>) {}
  
  // TODO: Implement map
  // TODO: Implement flatMap
  // TODO: Implement run()
}

// Usage should work like:
// const task = new Task(() => fetchUser(1))
//   .map(user => user.email)
//   .flatMap(email => new Task(() => sendEmail(email)));
// await task.run();
```

<details>
<summary><strong>View Answers</strong></summary>

```ts
class Task<T> {
  constructor(private computation: () => Promise<T>) {}
  
  // Map transforms the result
  map<U>(fn: (value: T) => U): Task<U> {
    return new Task(async () => {
      const value = await this.computation();
      return fn(value);
    });
  }
  
  // FlatMap for chaining tasks
  flatMap<U>(fn: (value: T) => Task<U>): Task<U> {
    return new Task(async () => {
      const value = await this.computation();
      const nextTask = fn(value);
      return await nextTask.run();
    });
  }
  
  // Execute the computation
  run(): Promise<T> {
    return this.computation();
  }
  
  // Combine with error handling
  catch<E>(handler: (error: any) => E): Task<T | E> {
    return new Task(async () => {
      try {
        return await this.computation();
      } catch (error) {
        return handler(error);
      }
    });
  }
}

// Example: User notification system
interface User {
  id: string;
  email: string;
  preferences: { emailNotifications: boolean };
}

// Simulate API calls
const fetchUser = (id: string): Promise<User> => {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve({
        id,
        email: `user${id}@example.com`,
        preferences: { emailNotifications: true }
      });
    }, 100);
  });
};

const sendEmail = (email: string, message: string): Promise<string> => {
  return new Promise(resolve => {
    setTimeout(() => {
      console.log(`Email sent to ${email}: ${message}`);
      resolve(`Sent to ${email}`);
    }, 100);
  });
};

// Build task pipeline
const notifyUser = (userId: string, message: string) =>
  new Task(() => fetchUser(userId))
    .map(user => {
      console.log(`Fetched user: ${user.email}`);
      return user;
    })
    .flatMap(user => {
      if (!user.preferences.emailNotifications) {
        return new Task(() => Promise.resolve('Notifications disabled'));
      }
      return new Task(() => sendEmail(user.email, message));
    })
    .catch(error => `Failed: ${error.message}`);

// Execute
notifyUser('123', 'Hello from Task monad!')
  .run()
  .then(result => console.log(`Result: ${result}`));

// Real-world: Multi-step data pipeline
const processOrder = (orderId: string) =>
  new Task(() => fetch(`/api/orders/${orderId}`).then(r => r.json()))
    .map(order => {
      console.log('Processing order:', order);
      return order;
    })
    .flatMap(order =>
      new Task(() => fetch('/api/inventory/check', {
        method: 'POST',
        body: JSON.stringify({ items: order.items })
      }).then(r => r.json()))
      .map(inventory => ({ order, inventory }))
    )
    .flatMap(({ order, inventory }) => {
      if (!inventory.available) {
        return new Task(() => Promise.reject(new Error('Out of stock')));
      }
      return new Task(() => fetch('/api/payments/process', {
        method: 'POST',
        body: JSON.stringify({ orderId: order.id })
      }).then(r => r.json()));
    })
    .catch(error => ({ status: 'failed', message: error.message }));

// Usage
processOrder('ORD-123')
  .run()
  .then(result => console.log('Order result:', result));
```

**Explanation**: The Task monad wraps async operations, making them composable. `map` transforms results, `flatMap` chains dependent async operations, and `catch` handles errors - all without deeply nested promises!

</details>

</details>

---

**Resources for Deep Dive**:
- **fp-ts** (TypeScript FP library): [gcanti.github.io/fp-ts](https://gcanti.github.io/fp-ts/)
- **Monad Tutorial**: [adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)
- **Go Generics & FP**: [The Go Blog - Generics](https://go.dev/blog/intro-generics)

## Algebraic Data Types (ADTs)

**Definition**: Types formed by combining other types. Two main categories:

1. **Product Types**: Combine types with AND (like structs/objects)
2. **Sum Types**: Combine types with OR (like unions/enums)

---

### Product Types

Types where you have ALL fields together.

**Examples**: Objects, Structs, Tuples

<details>
<summary><strong>View Examples</strong></summary>

```ts
// Product type: User has name AND email AND age
interface User {
  name: string;
  email: string;
  age: number;
}

// Tuple - fixed size array with different types
type Coordinate = [number, number]; // x AND y
type RGB = [number, number, number]; // red AND green AND blue

// Real-world: E-commerce order
interface Product {
  id: string;
  name: string;
  price: number;
}

interface ShippingAddress {
  street: string;
  city: string;
  zipCode: string;
  country: string;
}

// Product type: Order has ALL these fields
interface Order {
  orderId: string;
  customer: User;
  products: Product[];
  shippingAddress: ShippingAddress;
  totalAmount: number;
  createdAt: Date;
}

const order: Order = {
  orderId: 'ORD-001',
  customer: { name: 'Alice', email: 'alice@example.com', age: 30 },
  products: [
    { id: 'P1', name: 'Laptop', price: 999 }
  ],
  shippingAddress: {
    street: '123 Main St',
    city: 'New York',
    zipCode: '10001',
    country: 'USA'
  },
  totalAmount: 999,
  createdAt: new Date()
};
```

**Go Example:**

```go
package main

import "time"

// Product types - structs
type User struct {
    Name  string
    Email string
    Age   int
}

type Product struct {
    ID    string
    Name  string
    Price float64
}

type ShippingAddress struct {
    Street  string
    City    string
    ZipCode string
    Country string
}

type Order struct {
    OrderID         string
    Customer        User
    Products        []Product
    ShippingAddress ShippingAddress
    TotalAmount     float64
    CreatedAt       time.Time
}

func main() {
    order := Order{
        OrderID: "ORD-001",
        Customer: User{
            Name:  "Alice",
            Email: "alice@example.com",
            Age:   30,
        },
        Products: []Product{
            {ID: "P1", Name: "Laptop", Price: 999},
        },
        ShippingAddress: ShippingAddress{
            Street:  "123 Main St",
            City:    "New York",
            ZipCode: "10001",
            Country: "USA",
        },
        TotalAmount: 999,
        CreatedAt:   time.Now(),
    }
    
    _ = order
}
```

</details>

---

### Sum Types (Tagged Unions)

Types where you have ONE of several possibilities (OR).

**Real-World Scenario: Payment Methods**

<details>
<summary><strong>View Examples</strong></summary>

```ts
// Sum type: Payment is CreditCard OR PayPal OR BankTransfer
type CreditCard = {
  type: 'credit_card';
  cardNumber: string;
  expiryDate: string;
  cvv: string;
};

type PayPal = {
  type: 'paypal';
  email: string;
};

type BankTransfer = {
  type: 'bank_transfer';
  accountNumber: string;
  routingNumber: string;
};

// Sum type: One of these three
type PaymentMethod = CreditCard | PayPal | BankTransfer;

// Pattern matching with type discrimination
function processPayment(payment: PaymentMethod, amount: number): string {
  switch (payment.type) {
    case 'credit_card':
      return `Charging $${amount} to card ending in ${payment.cardNumber.slice(-4)}`;
    
    case 'paypal':
      return `Sending $${amount} via PayPal to ${payment.email}`;
    
    case 'bank_transfer':
      return `Transferring $${amount} to account ${payment.accountNumber}`;
    
    default:
      // TypeScript ensures exhaustiveness
      const _exhaustive: never = payment;
      return _exhaustive;
  }
}

// Usage
const creditCardPayment: PaymentMethod = {
  type: 'credit_card',
  cardNumber: '4532-1234-5678-9010',
  expiryDate: '12/25',
  cvv: '123'
};

const paypalPayment: PaymentMethod = {
  type: 'paypal',
  email: 'alice@example.com'
};

console.log(processPayment(creditCardPayment, 99.99));
// "Charging $99.99 to card ending in 9010"

console.log(processPayment(paypalPayment, 49.99));
// "Sending $49.99 via PayPal to alice@example.com"

// Real-world: API Response
type LoadingState = { status: 'loading' };
type SuccessState<T> = { status: 'success'; data: T };
type ErrorState = { status: 'error'; error: string };

type AsyncState<T> = LoadingState | SuccessState<T> | ErrorState;

// Usage in UI
interface UserProfile {
  name: string;
  bio: string;
}

function renderProfile(state: AsyncState<UserProfile>): string {
  switch (state.status) {
    case 'loading':
      return 'Loading...';
    
    case 'success':
      return `
        <div>
          <h1>${state.data.name}</h1>
          <p>${state.data.bio}</p>
        </div>
      `;
    
    case 'error':
      return `<div class="error">Error: ${state.error}</div>`;
  }
}

// Type-safe state transitions
const loadingState: AsyncState<UserProfile> = { status: 'loading' };

const successState: AsyncState<UserProfile> = {
  status: 'success',
  data: { name: 'Alice', bio: 'Software Engineer' }
};

const errorState: AsyncState<UserProfile> = {
  status: 'error',
  error: 'Failed to fetch user'
};

console.log(renderProfile(loadingState)); // "Loading..."
console.log(renderProfile(successState)); // Renders profile
console.log(renderProfile(errorState));   // Error message

// Real-world: Order status
type PendingOrder = {
  status: 'pending';
  orderId: string;
  createdAt: Date;
};

type ProcessingOrder = {
  status: 'processing';
  orderId: string;
  paymentId: string;
  startedAt: Date;
};

type ShippedOrder = {
  status: 'shipped';
  orderId: string;
  trackingNumber: string;
  shippedAt: Date;
};

type DeliveredOrder = {
  status: 'delivered';
  orderId: string;
  deliveredAt: Date;
  signature: string;
};

type CancelledOrder = {
  status: 'cancelled';
  orderId: string;
  reason: string;
  cancelledAt: Date;
};

type OrderStatus =
  | PendingOrder
  | ProcessingOrder
  | ShippedOrder
  | DeliveredOrder
  | CancelledOrder;

function getOrderMessage(order: OrderStatus): string {
  switch (order.status) {
    case 'pending':
      return `Order ${order.orderId} is pending (created ${order.createdAt.toLocaleDateString()})`;
    
    case 'processing':
      return `Processing order ${order.orderId} (payment: ${order.paymentId})`;
    
    case 'shipped':
      return `Order ${order.orderId} shipped! Track: ${order.trackingNumber}`;
    
    case 'delivered':
      return `Order ${order.orderId} delivered at ${order.deliveredAt.toLocaleString()}`;
    
    case 'cancelled':
      return `Order ${order.orderId} cancelled: ${order.reason}`;
  }
}

const shippedOrder: OrderStatus = {
  status: 'shipped',
  orderId: 'ORD-123',
  trackingNumber: 'TRK-456',
  shippedAt: new Date()
};

console.log(getOrderMessage(shippedOrder));
```

**Go Example (using interfaces):**

```go
package main

import (
    "fmt"
    "time"
)

// Sum type using interfaces and type assertions
type PaymentMethod interface {
    ProcessPayment(amount float64) string
}

type CreditCard struct {
    CardNumber string
    ExpiryDate string
    CVV        string
}

func (c CreditCard) ProcessPayment(amount float64) string {
    lastFour := c.CardNumber[len(c.CardNumber)-4:]
    return fmt.Sprintf("Charging $%.2f to card ending in %s", amount, lastFour)
}

type PayPal struct {
    Email string
}

func (p PayPal) ProcessPayment(amount float64) string {
    return fmt.Sprintf("Sending $%.2f via PayPal to %s", amount, p.Email)
}

type BankTransfer struct {
    AccountNumber string
    RoutingNumber string
}

func (b BankTransfer) ProcessPayment(amount float64) string {
    return fmt.Sprintf("Transferring $%.2f to account %s", amount, b.AccountNumber)
}

// Order status using interfaces
type OrderStatus interface {
    GetMessage() string
}

type PendingOrder struct {
    OrderID   string
    CreatedAt time.Time
}

func (o PendingOrder) GetMessage() string {
    return fmt.Sprintf("Order %s is pending", o.OrderID)
}

type ShippedOrder struct {
    OrderID        string
    TrackingNumber string
    ShippedAt      time.Time
}

func (o ShippedOrder) GetMessage() string {
    return fmt.Sprintf("Order %s shipped! Track: %s", o.OrderID, o.TrackingNumber)
}

func main() {
    // Using payment methods
    var payment PaymentMethod
    
    payment = CreditCard{
        CardNumber: "4532-1234-5678-9010",
        ExpiryDate: "12/25",
        CVV:        "123",
    }
    fmt.Println(payment.ProcessPayment(99.99))
    
    payment = PayPal{Email: "alice@example.com"}
    fmt.Println(payment.ProcessPayment(49.99))
    
    // Using order status
    var status OrderStatus
    
    status = ShippedOrder{
        OrderID:        "ORD-123",
        TrackingNumber: "TRK-456",
        ShippedAt:      time.Now(),
    }
    fmt.Println(status.GetMessage())
}
```

</details>

---

### Combining Product and Sum Types

**Real-World Scenario: Form Validation**

<details>
<summary><strong>View Examples</strong></summary>

```ts
// Product type: ValidationError has field AND message
type ValidationError = {
  field: string;
  message: string;
};

// Sum type: ValidationResult is Success OR Failure
type Success<T> = {
  type: 'success';
  value: T;
};

type Failure = {
  type: 'failure';
  errors: ValidationError[];
};

type ValidationResult<T> = Success<T> | Failure;

// Product type: Form data
interface RegistrationForm {
  username: string;
  email: string;
  password: string;
  confirmPassword: string;
  age: number;
}

// Validators
function validateUsername(username: string): ValidationResult<string> {
  if (username.length < 3) {
    return {
      type: 'failure',
      errors: [{ field: 'username', message: 'Username must be at least 3 characters' }]
    };
  }
  
  if (!/^[a-zA-Z0-9_]+$/.test(username)) {
    return {
      type: 'failure',
      errors: [{ field: 'username', message: 'Username can only contain letters, numbers, and underscores' }]
    };
  }
  
  return { type: 'success', value: username };
}

function validateEmail(email: string): ValidationResult<string> {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  
  if (!emailRegex.test(email)) {
    return {
      type: 'failure',
      errors: [{ field: 'email', message: 'Invalid email format' }]
    };
  }
  
  return { type: 'success', value: email };
}

function validatePassword(password: string, confirmPassword: string): ValidationResult<string> {
  const errors: ValidationError[] = [];
  
  if (password.length < 8) {
    errors.push({ field: 'password', message: 'Password must be at least 8 characters' });
  }
  
  if (!/[A-Z]/.test(password)) {
    errors.push({ field: 'password', message: 'Password must contain uppercase letter' });
  }
  
  if (!/[0-9]/.test(password)) {
    errors.push({ field: 'password', message: 'Password must contain a number' });
  }
  
  if (password !== confirmPassword) {
    errors.push({ field: 'confirmPassword', message: 'Passwords do not match' });
  }
  
  if (errors.length > 0) {
    return { type: 'failure', errors };
  }
  
  return { type: 'success', value: password };
}

function validateAge(age: number): ValidationResult<number> {
  if (age < 13) {
    return {
      type: 'failure',
      errors: [{ field: 'age', message: 'Must be at least 13 years old' }]
    };
  }
  
  if (age > 120) {
    return {
      type: 'failure',
      errors: [{ field: 'age', message: 'Invalid age' }]
    };
  }
  
  return { type: 'success', value: age };
}

// Combine all validations
function validateRegistrationForm(form: RegistrationForm): ValidationResult<RegistrationForm> {
  const allErrors: ValidationError[] = [];
  
  const usernameResult = validateUsername(form.username);
  if (usernameResult.type === 'failure') {
    allErrors.push(...usernameResult.errors);
  }
  
  const emailResult = validateEmail(form.email);
  if (emailResult.type === 'failure') {
    allErrors.push(...emailResult.errors);
  }
  
  const passwordResult = validatePassword(form.password, form.confirmPassword);
  if (passwordResult.type === 'failure') {
    allErrors.push(...passwordResult.errors);
  }
  
  const ageResult = validateAge(form.age);
  if (ageResult.type === 'failure') {
    allErrors.push(...ageResult.errors);
  }
  
  if (allErrors.length > 0) {
    return { type: 'failure', errors: allErrors };
  }
  
  return { type: 'success', value: form };
}

// Usage
const validForm: RegistrationForm = {
  username: 'alice_123',
  email: 'alice@example.com',
  password: 'SecurePass1',
  confirmPassword: 'SecurePass1',
  age: 25
};

const invalidForm: RegistrationForm = {
  username: 'al',
  email: 'invalid-email',
  password: 'weak',
  confirmPassword: 'different',
  age: 10
};

const validResult = validateRegistrationForm(validForm);
if (validResult.type === 'success') {
  console.log('✓ Registration valid!', validResult.value);
} else {
  console.log('✗ Validation failed:');
  validResult.errors.forEach(err => {
    console.log(`  - ${err.field}: ${err.message}`);
  });
}

const invalidResult = validateRegistrationForm(invalidForm);
if (invalidResult.type === 'success') {
  console.log('✓ Registration valid!');
} else {
  console.log('✗ Validation failed:');
  invalidResult.errors.forEach(err => {
    console.log(`  - ${err.field}: ${err.message}`);
  });
}
// ✗ Validation failed:
//   - username: Username must be at least 3 characters
//   - email: Invalid email format
//   - password: Password must be at least 8 characters
//   - password: Password must contain uppercase letter
//   - password: Password must contain a number
//   - confirmPassword: Passwords do not match
//   - age: Must be at least 13 years old
```

**Go Example:**

```go
package main

import (
    "fmt"
    "regexp"
)

// Product type
type ValidationError struct {
    Field   string
    Message string
}

// Sum type using interface
type ValidationResult interface {
    IsSuccess() bool
}

type Success struct {
    Value interface{}
}

func (s Success) IsSuccess() bool {
    return true
}

type Failure struct {
    Errors []ValidationError
}

func (f Failure) IsSuccess() bool {
    return false
}

// Product type
type RegistrationForm struct {
    Username        string
    Email           string
    Password        string
    ConfirmPassword string
    Age             int
}

func validateUsername(username string) ValidationResult {
    if len(username) < 3 {
        return Failure{
            Errors: []ValidationError{
                {Field: "username", Message: "Username must be at least 3 characters"},
            },
        }
    }
    
    matched, _ := regexp.MatchString(`^[a-zA-Z0-9_]+$`, username)
    if !matched {
        return Failure{
            Errors: []ValidationError{
                {Field: "username", Message: "Invalid username format"},
            },
        }
    }
    
    return Success{Value: username}
}

func validateEmail(email string) ValidationResult {
    emailRegex := regexp.MustCompile(`^[^\s@]+@[^\s@]+\.[^\s@]+$`)
    
    if !emailRegex.MatchString(email) {
        return Failure{
            Errors: []ValidationError{
                {Field: "email", Message: "Invalid email format"},
            },
        }
    }
    
    return Success{Value: email}
}

func validateRegistrationForm(form RegistrationForm) ValidationResult {
    var allErrors []ValidationError
    
    usernameResult := validateUsername(form.Username)
    if !usernameResult.IsSuccess() {
        failure := usernameResult.(Failure)
        allErrors = append(allErrors, failure.Errors...)
    }
    
    emailResult := validateEmail(form.Email)
    if !emailResult.IsSuccess() {
        failure := emailResult.(Failure)
        allErrors = append(allErrors, failure.Errors...)
    }
    
    if len(allErrors) > 0 {
        return Failure{Errors: allErrors}
    }
    
    return Success{Value: form}
}

func main() {
    invalidForm := RegistrationForm{
        Username: "al",
        Email:    "invalid-email",
    }
    
    result := validateRegistrationForm(invalidForm)
    
    if result.IsSuccess() {
        fmt.Println("✓ Validation passed")
    } else {
        failure := result.(Failure)
        fmt.Println("✗ Validation failed:")
        for _, err := range failure.Errors {
            fmt.Printf("  - %s: %s\n", err.Field, err.Message)
        }
    }
}
```

</details>

---

## Practice Questions

<details>
<summary><strong>View Questions</strong></summary>

### Fill in the Blanks

1. Product types combine fields with __________ (all fields present).
2. Sum types represent __________ of several possibilities.
3. TypeScript discriminated unions use a __________ property for type discrimination.
4. Exhaustive pattern matching ensures all cases of a sum type are __________.

<details>
<summary><strong>View Answers</strong></summary>

1. **AND** - Product types have all fields together (User has name AND email AND age)
2. **one** (or "choice") - Sum types represent alternatives (Payment is CreditCard OR PayPal OR BankTransfer)
3. **literal** (or "tag"/"discriminant") - The `type` or `status` field with literal string values enables type narrowing
4. **handled** - TypeScript's `never` type ensures you've covered all possible cases in switch/if statements

</details>

---

### True/False

1. ⬜ Structs/objects in TypeScript and Go are product types
2. ⬜ Sum types make illegal states unrepresentable
3. ⬜ Go has native support for discriminated unions like TypeScript
4. ⬜ Pattern matching on sum types is safer than using if/else chains

<details>
<summary><strong>View Answers</strong></summary>

1. **True** - Structs combine multiple fields together with AND, making them product types
2. **True** - By modeling states as distinct types (Pending, Shipped, Delivered), you prevent impossible combinations like "shipped but not paid"
3. **False** - Go uses interfaces and type assertions/switches for sum type patterns; TypeScript has more direct support via union types
4. **True** - Pattern matching with exhaustiveness checking catches missing cases at compile time, while if/else chains can miss cases

</details>

---

### Multiple Choice

1. Which is a sum type?

```ts
// A
type User = { name: string; age: number };

// B
type Result<T> = Success<T> | Failure;

// C
type Coordinate = [number, number];

// D
interface Product { id: string; price: number; }
```

2. What's the main benefit of sum types?

- A) Better performance
- B) Less memory usage
- C) Type-safe state representation
- D) Faster compilation

<details>
<summary><strong>View Answers</strong></summary>

1. **B** - `Result<T> = Success<T> | Failure` is a sum type (one of two options). A, C, D are product types (all fields together)

2. **C** - Type-safe state representation. Sum types prevent invalid states and enable exhaustive checking, catching errors at compile time

</details>

---

### Code Challenge

Model a notification system using ADTs:

```ts
// TODO: Create sum types for:
// 1. NotificationType: Email, SMS, Push
// 2. NotificationStatus: Pending, Sent, Failed
// 3. Notification (product type combining above)

// Then implement:
// - sendNotification(notification): changes status based on type
// - getStatusMessage(notification): returns user-friendly message
```

<details>
<summary><strong>View Answers</strong></summary>

```ts
// Sum type: Notification types
type EmailNotification = {
  type: 'email';
  to: string;
  subject: string;
  body: string;
};

type SMSNotification = {
  type: 'sms';
  phoneNumber: string;
  message: string;
};

type PushNotification = {
  type: 'push';
  deviceId: string;
  title: string;
  body: string;
};

type NotificationType = EmailNotification | SMSNotification | PushNotification;

// Sum type: Status
type PendingStatus = {
  status: 'pending';
  queuedAt: Date;
};

type SentStatus = {
  status: 'sent';
  sentAt: Date;
  deliveryId: string;
};

type FailedStatus = {
  status: 'failed';
  failedAt: Date;
  error: string;
  retryCount: number;
};

type NotificationStatus = PendingStatus | SentStatus | FailedStatus;

// Product type: Notification has id AND type AND status
interface Notification {
  id: string;
  notificationType: NotificationType;
  notificationStatus: NotificationStatus;
}

// Send notification (simulated)
function sendNotification(notification: Notification): Notification {
  const { notificationType } = notification;
  
  // Simulate sending based on type
  switch (notificationType.type) {
    case 'email':
      // Simulate email sending
      console.log(`Sending email to ${notificationType.to}`);
      
      // Simulate success
      if (Math.random() > 0.2) {
        return {
          ...notification,
          notificationStatus: {
            status: 'sent',
            sentAt: new Date(),
            deliveryId: `EMAIL-${Math.random().toString(36).substr(2, 9)}`
          }
        };
      } else {
        return {
          ...notification,
          notificationStatus: {
            status: 'failed',
            failedAt: new Date(),
            error: 'SMTP server unavailable',
            retryCount: 1
          }
        };
      }
    
    case 'sms':
      console.log(`Sending SMS to ${notificationType.phoneNumber}`);
      
      if (Math.random() > 0.1) {
        return {
          ...notification,
          notificationStatus: {
            status: 'sent',
            sentAt: new Date(),
            deliveryId: `SMS-${Math.random().toString(36).substr(2, 9)}`
          }
        };
      } else {
        return {
          ...notification,
          notificationStatus: {
            status: 'failed',
            failedAt: new Date(),
            error: 'Invalid phone number',
            retryCount: 1
          }
        };
      }
    
    case 'push':
      console.log(`Sending push to device ${notificationType.deviceId}`);
      
      return {
        ...notification,
        notificationStatus: {
          status: 'sent',
          sentAt: new Date(),
          deliveryId: `PUSH-${Math.random().toString(36).substr(2, 9)}`
        }
      };
  }
}

// Get status message
function getStatusMessage(notification: Notification): string {
  const { notificationType, notificationStatus } = notification;
  
  // Get notification type description
  let typeDesc: string;
  switch (notificationType.type) {
    case 'email':
      typeDesc = `Email to ${notificationType.to}`;
      break;
    case 'sms':
      typeDesc = `SMS to ${notificationType.phoneNumber}`;
      break;
    case 'push':
      typeDesc = `Push to device ${notificationType.deviceId}`;
      break;
  }
  
  // Get status description
  switch (notificationStatus.status) {
    case 'pending':
      return `${typeDesc} is pending (queued at ${notificationStatus.queuedAt.toLocaleTimeString()})`;
    
    case 'sent':
      return `${typeDesc} was sent successfully at ${notificationStatus.sentAt.toLocaleTimeString()} (ID: ${notificationStatus.deliveryId})`;
    
    case 'failed':
      return `${typeDesc} failed: ${notificationStatus.error} (attempt ${notificationStatus.retryCount})`;
  }
}

// Usage
const emailNotification: Notification = {
  id: 'N-001',
  notificationType: {
    type: 'email',
    to: 'alice@example.com',
    subject: 'Welcome!',
    body: 'Thanks for signing up'
  },
  notificationStatus: {
    status: 'pending',
    queuedAt: new Date()
  }
};

console.log(getStatusMessage(emailNotification));
// "Email to alice@example.com is pending (queued at 10:30:45 AM)"

const sentNotification = sendNotification(emailNotification);
console.log(getStatusMessage(sentNotification));
// "Email to alice@example.com was sent successfully at 10:30:46 AM (ID: EMAIL-abc123)"

// SMS example
const smsNotification: Notification = {
  id: 'N-002',
  notificationType: {
    type: 'sms',
    phoneNumber: '+1234567890',
    message: 'Your verification code is 123456'
  },
  notificationStatus: {
    status: 'pending',
    queuedAt: new Date()
  }
};

const sentSMS = sendNotification(smsNotification);
console.log(getStatusMessage(sentSMS));

// Batch processing
const notifications: Notification[] = [emailNotification, smsNotification];

const results = notifications.map(n => {
  const sent = sendNotification(n);
  return {
    id: sent.id,
    message: getStatusMessage(sent),
    success: sent.notificationStatus.status === 'sent'
  };
});

console.log('\nBatch Results:');
results.forEach(r => {
  const icon = r.success ? '✓' : '✗';
  console.log(`${icon} ${r.message}`);
});
```

**Explanation**: 
- Sum types (`NotificationType`, `NotificationStatus`) make states explicit and type-safe
- Product type (`Notification`) combines all required fields
- Pattern matching ensures all notification types and statuses are handled
- Impossible states (like "sent but no delivery ID") are prevented by the type system

</details>

</details>

---

## Pattern Matching

**Definition**: A way to check values against patterns and extract data, more powerful than simple if/else or switch statements.

**Real-Life Analogy**:  
Think of sorting mail. Instead of checking "Is this a letter? Is it a package? Is it urgent?", you match the pattern: "If it's a small envelope with a stamp → letter bin. If it's a box → package area. If it has red 'URGENT' → priority queue."

---

### Pattern Matching in TypeScript

TypeScript doesn't have native pattern matching (like Rust or Haskell), but we can achieve it using:
1. Discriminated unions
2. Type guards
3. Switch statements with exhaustiveness checking

**Real-World Scenario: HTTP Response Handling**

<details>
<summary><strong>View Examples</strong></summary>

```ts
// HTTP Response types
type SuccessResponse<T> = {
  status: 'success';
  statusCode: 200 | 201;
  data: T;
  timestamp: Date;
};

type NotFoundResponse = {
  status: 'not_found';
  statusCode: 404;
  message: string;
};

type UnauthorizedResponse = {
  status: 'unauthorized';
  statusCode: 401;
  message: string;
  loginUrl: string;
};

type ServerErrorResponse = {
  status: 'server_error';
  statusCode: 500 | 502 | 503;
  error: string;
  retryAfter?: number;
};

type ValidationErrorResponse = {
  status: 'validation_error';
  statusCode: 400;
  errors: Array<{ field: string; message: string }>;
};

type HttpResponse<T> =
  | SuccessResponse<T>
  | NotFoundResponse
  | UnauthorizedResponse
  | ServerErrorResponse
  | ValidationErrorResponse;

// Pattern matching using switch
function handleResponse<T>(response: HttpResponse<T>): string {
  switch (response.status) {
    case 'success':
      console.log('Data received:', response.data);
      return `Success: Received data at ${response.timestamp.toISOString()}`;
    
    case 'not_found':
      console.error('Resource not found');
      return `Error: ${response.message}`;
    
    case 'unauthorized':
      console.error('Authentication required');
      return `Please login at: ${response.loginUrl}`;
    
    case 'server_error':
      console.error('Server error:', response.error);
      if (response.retryAfter) {
        return `Server error. Retry after ${response.retryAfter} seconds`;
      }
      return `Server error: ${response.error}`;
    
    case 'validation_error':
      console.error('Validation failed');
      const errorMessages = response.errors
        .map(e => `${e.field}: ${e.message}`)
        .join(', ');
      return `Validation errors: ${errorMessages}`;
    
    default:
      // Exhaustiveness check
      const _exhaustive: never = response;
      return _exhaustive;
  }
}

// Usage
interface User {
  id: string;
  name: string;
  email: string;
}

const successResponse: HttpResponse<User> = {
  status: 'success',
  statusCode: 200,
  data: { id: '1', name: 'Alice', email: 'alice@example.com' },
  timestamp: new Date()
};

const notFoundResponse: HttpResponse<User> = {
  status: 'not_found',
  statusCode: 404,
  message: 'User not found'
};

const validationErrorResponse: HttpResponse<User> = {
  status: 'validation_error',
  statusCode: 400,
  errors: [
    { field: 'email', message: 'Invalid email format' },
    { field: 'age', message: 'Must be at least 18' }
  ]
};

console.log(handleResponse(successResponse));
// "Success: Received data at 2024-02-06T..."

console.log(handleResponse(notFoundResponse));
// "Error: User not found"

console.log(handleResponse(validationErrorResponse));
// "Validation errors: email: Invalid email format, age: Must be at least 18"

// Advanced: Pattern matching with predicates
type Shape =
  | { type: 'circle'; radius: number }
  | { type: 'rectangle'; width: number; height: number }
  | { type: 'triangle'; base: number; height: number };

function calculateArea(shape: Shape): number {
  switch (shape.type) {
    case 'circle':
      return Math.PI * shape.radius ** 2;
    
    case 'rectangle':
      return shape.width * shape.height;
    
    case 'triangle':
      return (shape.base * shape.height) / 2;
    
    default:
      const _exhaustive: never = shape;
      return _exhaustive;
  }
}

function describeShape(shape: Shape): string {
  switch (shape.type) {
    case 'circle':
      if (shape.radius > 10) {
        return `Large circle with radius ${shape.radius}`;
      }
      return `Small circle with radius ${shape.radius}`;
    
    case 'rectangle':
      if (shape.width === shape.height) {
        return `Square with side ${shape.width}`;
      }
      return `Rectangle ${shape.width}x${shape.height}`;
    
    case 'triangle':
      return `Triangle with base ${shape.base} and height ${shape.height}`;
  }
}

const circle: Shape = { type: 'circle', radius: 5 };
const square: Shape = { type: 'rectangle', width: 10, height: 10 };
const rectangle: Shape = { type: 'rectangle', width: 10, height: 20 };

console.log(calculateArea(circle)); // 78.54
console.log(describeShape(square)); // "Square with side 10"
console.log(describeShape(rectangle)); // "Rectangle 10x20"

// Real-world: Event handling
type UserEvent =
  | { type: 'user_registered'; userId: string; email: string; timestamp: Date }
  | { type: 'user_login'; userId: string; ipAddress: string; timestamp: Date }
  | { type: 'user_logout'; userId: string; sessionDuration: number; timestamp: Date }
  | { type: 'password_changed'; userId: string; timestamp: Date }
  | { type: 'profile_updated'; userId: string; changes: string[]; timestamp: Date };

function handleUserEvent(event: UserEvent): void {
  switch (event.type) {
    case 'user_registered':
      console.log(`🎉 New user registered: ${event.email}`);
      // Send welcome email
      break;
    
    case 'user_login':
      console.log(`👤 User ${event.userId} logged in from ${event.ipAddress}`);
      // Track login analytics
      break;
    
    case 'user_logout':
      const minutes = Math.floor(event.sessionDuration / 60);
      console.log(`👋 User ${event.userId} logged out after ${minutes} minutes`);
      break;
    
    case 'password_changed':
      console.log(`🔒 User ${event.userId} changed password`);
      // Send security notification
      break;
    
    case 'profile_updated':
      console.log(`✏️ User ${event.userId} updated: ${event.changes.join(', ')}`);
      break;
    
    default:
      const _exhaustive: never = event;
      return _exhaustive;
  }
}

// Event stream
const events: UserEvent[] = [
  {
    type: 'user_registered',
    userId: 'U1',
    email: 'alice@example.com',
    timestamp: new Date()
  },
  {
    type: 'user_login',
    userId: 'U1',
    ipAddress: '192.168.1.1',
    timestamp: new Date()
  },
  {
    type: 'profile_updated',
    userId: 'U1',
    changes: ['avatar', 'bio'],
    timestamp: new Date()
  }
];

events.forEach(handleUserEvent);
// 🎉 New user registered: alice@example.com
// 👤 User U1 logged in from 192.168.1.1
// ✏️ User U1 updated: avatar, bio

// Pattern matching helper (custom implementation)
type Pattern<T, R> = {
  [K in T extends { type: infer U } ? U : never]: (value: Extract<T, { type: K }>) => R;
};

function match<T extends { type: string }, R>(
  value: T,
  patterns: Pattern<T, R> & { _?: (value: T) => R }
): R {
  const handler = (patterns as any)[value.type] || patterns._;
  if (!handler) {
    throw new Error(`No pattern matched for type: ${value.type}`);
  }
  return handler(value);
}

// Usage with match helper
const responseMessage = match(successResponse, {
  success: (r) => `Got data: ${JSON.stringify(r.data)}`,
  not_found: (r) => `Not found: ${r.message}`,
  unauthorized: (r) => `Login at: ${r.loginUrl}`,
  server_error: (r) => `Server error: ${r.error}`,
  validation_error: (r) => `Validation failed: ${r.errors.length} errors`,
});

console.log(responseMessage);
```

**Go Example:**

```go
package main

import (
    "fmt"
    "time"
)

// Using type switches for pattern matching
type HttpResponse interface {
    GetStatus() string
}

type SuccessResponse struct {
    StatusCode int
    Data       interface{}
    Timestamp  time.Time
}

func (s SuccessResponse) GetStatus() string { return "success" }

type NotFoundResponse struct {
    StatusCode int
    Message    string
}

func (n NotFoundResponse) GetStatus() string { return "not_found" }

type ServerErrorResponse struct {
    StatusCode int
    Error      string
    RetryAfter *int
}

func (s ServerErrorResponse) GetStatus() string { return "server_error" }

type ValidationErrorResponse struct {
    StatusCode int
    Errors     []ValidationError
}

func (v ValidationErrorResponse) GetStatus() string { return "validation_error" }

type ValidationError struct {
    Field   string
    Message string
}

// Pattern matching using type switch
func handleResponse(response HttpResponse) string {
    switch r := response.(type) {
    case SuccessResponse:
        return fmt.Sprintf("Success: Received data at %s", r.Timestamp.Format(time.RFC3339))
    
    case NotFoundResponse:
        return fmt.Sprintf("Error: %s", r.Message)
    
    case ServerErrorResponse:
        if r.RetryAfter != nil {
            return fmt.Sprintf("Server error. Retry after %d seconds", *r.RetryAfter)
        }
        return fmt.Sprintf("Server error: %s", r.Error)
    
    case ValidationErrorResponse:
        errorMsg := ""
        for i, err := range r.Errors {
            if i > 0 {
                errorMsg += ", "
            }
            errorMsg += fmt.Sprintf("%s: %s", err.Field, err.Message)
        }
        return fmt.Sprintf("Validation errors: %s", errorMsg)
    
    default:
        return "Unknown response type"
    }
}

// Shape example
type Shape interface {
    Area() float64
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return 3.14159 * c.Radius * c.Radius
}

type Rectangle struct {
    Width  float64
    Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func describeShape(shape Shape) string {
    switch s := shape.(type) {
    case Circle:
        if s.Radius > 10 {
            return fmt.Sprintf("Large circle with radius %.2f", s.Radius)
        }
        return fmt.Sprintf("Small circle with radius %.2f", s.Radius)
    
    case Rectangle:
        if s.Width == s.Height {
            return fmt.Sprintf("Square with side %.2f", s.Width)
        }
        return fmt.Sprintf("Rectangle %.2fx%.2f", s.Width, s.Height)
    
    default:
        return "Unknown shape"
    }
}

func main() {
    // HTTP Response handling
    success := SuccessResponse{
        StatusCode: 200,
        Data:       map[string]string{"name": "Alice"},
        Timestamp:  time.Now(),
    }
    
    notFound := NotFoundResponse{
        StatusCode: 404,
        Message:    "User not found",
    }
    
    fmt.Println(handleResponse(success))
    fmt.Println(handleResponse(notFound))
    
    // Shape handling
    circle := Circle{Radius: 5}
    square := Rectangle{Width: 10, Height: 10}
    
    fmt.Println(describeShape(circle))
    fmt.Println(describeShape(square))
    fmt.Printf("Circle area: %.2f\n", circle.Area())
}
```

</details>

---

### Advanced Pattern Matching: Nested Patterns

**Real-World Scenario: Shopping Cart Discount Logic**

<details>
<summary><strong>View Examples</strong></summary>

```ts
// Customer types
type GuestCustomer = {
  type: 'guest';
  email: string;
};

type RegisteredCustomer = {
  type: 'registered';
  id: string;
  email: string;
  membershipLevel: 'bronze' | 'silver' | 'gold' | 'platinum';
  joinDate: Date;
};

type Customer = GuestCustomer | RegisteredCustomer;

// Cart types
type EmptyCart = {
  status: 'empty';
  items: [];
};

type ActiveCart = {
  status: 'active';
  items: CartItem[];
  subtotal: number;
};

type CheckedOutCart = {
  status: 'checked_out';
  items: CartItem[];
  subtotal: number;
  discountApplied: number;
  total: number;
  orderId: string;
};

type Cart = EmptyCart | ActiveCart | CheckedOutCart;

interface CartItem {
  productId: string;
  name: string;
  price: number;
  quantity: number;
}

// Calculate discount based on customer type and cart
function calculateDiscount(customer: Customer, cart: Cart): number {
  // Pattern matching on nested conditions
  switch (cart.status) {
    case 'empty':
      return 0;
    
    case 'active':
      switch (customer.type) {
        case 'guest':
          // Guests: no discount, but 5% if cart over $100
          return cart.subtotal > 100 ? cart.subtotal * 0.05 : 0;
        
        case 'registered':
          // Registered: discount based on membership level
          const { membershipLevel, joinDate } = customer;
          const daysSinceJoined = Math.floor(
            (Date.now() - joinDate.getTime()) / (1000 * 60 * 60 * 24)
          );
          
          // Base discount by tier
          let discount = 0;
          switch (membershipLevel) {
            case 'bronze':
              discount = cart.subtotal * 0.05; // 5%
              break;
            case 'silver':
              discount = cart.subtotal * 0.10; // 10%
              break;
            case 'gold':
              discount = cart.subtotal * 0.15; // 15%
              break;
            case 'platinum':
              discount = cart.subtotal * 0.20; // 20%
              break;
          }
          
          // Loyalty bonus: extra 5% if member for over 1 year
          if (daysSinceJoined > 365) {
            discount += cart.subtotal * 0.05;
          }
          
          // Big spender bonus: extra 5% if cart over $500
          if (cart.subtotal > 500) {
            discount += cart.subtotal * 0.05;
          }
          
          return discount;
      }
    
    case 'checked_out':
      return cart.discountApplied; // Already calculated
    
    default:
      const _exhaustive: never = cart;
      return _exhaustive;
  }
}

// Format message based on customer and discount
function getDiscountMessage(customer: Customer, cart: Cart, discount: number): string {
  if (discount === 0) {
    switch (customer.type) {
      case 'guest':
        return 'Sign up for instant discounts!';
      case 'registered':
        return 'Add more items to unlock discounts!';
    }
  }
  
  switch (customer.type) {
    case 'guest':
      return `You saved $${discount.toFixed(2)}! Sign up to save even more.`;
    
    case 'registered':
      const percentSaved = (discount / (cart as ActiveCart).subtotal * 100).toFixed(0);
      return `${customer.membershipLevel.toUpperCase()} member discount: ${percentSaved}% off ($${discount.toFixed(2)})`;
  }
}

// Test scenarios
const guestCustomer: Customer = {
  type: 'guest',
  email: 'guest@example.com'
};

const bronzeCustomer: Customer = {
  type: 'registered',
  id: 'U1',
  email: 'bronze@example.com',
  membershipLevel: 'bronze',
  joinDate: new Date('2023-01-15')
};

const platinumCustomer: Customer = {
  type: 'registered',
  id: 'U2',
  email: 'platinum@example.com',
  membershipLevel: 'platinum',
  joinDate: new Date('2020-01-01') // Long-time member
};

const activeCart: ActiveCart = {
  status: 'active',
  items: [
    { productId: 'P1', name: 'Laptop', price: 999, quantity: 1 },
    { productId: 'P2', name: 'Mouse', price: 25, quantity: 2 }
  ],
  subtotal: 1049
};

const smallCart: ActiveCart = {
  status: 'active',
  items: [
    { productId: 'P3', name: 'Cable', price: 15, quantity: 1 }
  ],
  subtotal: 15
};

// Scenario 1: Guest with large cart
console.log('\n=== Guest with large cart ===');
const guestDiscount = calculateDiscount(guestCustomer, activeCart);
console.log(`Discount: $${guestDiscount.toFixed(2)}`);
console.log(getDiscountMessage(guestCustomer, activeCart, guestDiscount));
// Discount: $52.45
// You saved $52.45! Sign up to save even more.

// Scenario 2: Bronze member with large cart
console.log('\n=== Bronze member with large cart ===');
const bronzeDiscount = calculateDiscount(bronzeCustomer, activeCart);
console.log(`Discount: $${bronzeDiscount.toFixed(2)}`);
console.log(getDiscountMessage(bronzeCustomer, activeCart, bronzeDiscount));
// Discount: $157.35 (5% base + 5% loyalty + 5% big spender)
// BRONZE member discount: 15% off ($157.35)

// Scenario 3: Platinum member with large cart
console.log('\n=== Platinum member with large cart ===');
const platinumDiscount = calculateDiscount(platinumCustomer, activeCart);
console.log(`Discount: $${platinumDiscount.toFixed(2)}`);
console.log(getDiscountMessage(platinumCustomer, activeCart, platinumDiscount));
// Discount: $314.70 (20% base + 5% loyalty + 5% big spender)
// PLATINUM member discount: 30% off ($314.70)

// Scenario 4: Bronze member with small cart
console.log('\n=== Bronze member with small cart ===');
const smallDiscount = calculateDiscount(bronzeCustomer, smallCart);
console.log(`Discount: $${smallDiscount.toFixed(2)}`);
console.log(getDiscountMessage(bronzeCustomer, smallCart, smallDiscount));
// Discount: $1.50 (5% base + 5% loyalty)
// BRONZE member discount: 10% off ($1.50)
```

</details>

---

## Practice Questions

<details>
<summary><strong>View Questions</strong></summary>

### Fill in the Blanks

1. Pattern matching is more powerful than switch statements because it can __________ data from complex structures.
2. The `never` type in TypeScript ensures __________ checking in switch statements.
3. Go uses __________ switches to implement pattern matching on interface types.
4. Discriminated unions use a __________ field to enable pattern matching.

<details>
<summary><strong>View Answers</strong></summary>

1. **extract** (or "destructure") - Pattern matching not only checks types but also pulls out nested values
2. **exhaustiveness** - The `never` type causes a compile error if you haven't handled all cases
3. **type** - Go's type switch allows matching on the concrete type of an interface value
4. **literal** (or "tag"/"discriminant") - A field with literal string values like `type: 'success'` enables type narrowing

</details>

---

### True/False

1. ⬜ Pattern matching eliminates the need for if/else statements
2. ⬜ TypeScript's discriminated unions provide compile-time safety for pattern matching
3. ⬜ Go has built-in pattern matching like Rust or Haskell
4. ⬜ Exhaustiveness checking helps catch bugs at compile time

<details>
<summary><strong>View Answers</strong></summary>

1. **False** - Pattern matching is a more powerful alternative for certain cases, but if/else is still useful for simple conditions and predicates
2. **True** - TypeScript narrows types within each case, catching missing cases at compile time
3. **False** - Go uses type switches and interfaces for pattern-matching-like behavior, but doesn't have Rust/Haskell-style pattern matching
4. **True** - When all cases must be handled (via `never` in TS), you'll get compile errors if you miss a case, preventing runtime bugs

</details>

---

### Multiple Choice

1. What happens if you don't handle all cases in a TypeScript discriminated union switch?

- A) Runtime error
- B) Compile-time error (with exhaustiveness check)
- C) Nothing, it's optional
- D) Default case is used

2. Which is the best use case for pattern matching?

- A) Simple boolean checks
- B) Complex nested data structures with multiple types
- C) String comparison
- D) Numeric calculations

<details>
<summary><strong>View Answers</strong></summary>

1. **B** - Compile-time error (with exhaustiveness check). If you assign the default case to `never`, TypeScript will error if you haven't handled all union members

2. **B** - Complex nested data structures with multiple types. Pattern matching shines when dealing with sum types and extracting data from variants

</details>

---

### Code Challenge

Build a command parser using pattern matching:

```ts
// TODO: Create types for:
// - Commands: Create, Update, Delete, List, Help
// - Each command should have different required fields
// - Implement executeCommand() using pattern matching
// - Implement getCommandHelp() that shows usage

// Example:
// executeCommand({ type: 'create', name: 'user1', email: 'user@example.com' })
// executeCommand({ type: 'delete', id: '123' })
// executeCommand({ type: 'list', filter: 'active' })
```

<details>
<summary><strong>View Answers</strong></summary>

```ts
// Command types
type CreateCommand = {
  type: 'create';
  resourceType: 'user' | 'product' | 'order';
  name: string;
  data: Record<string, any>;
};

type UpdateCommand = {
  type: 'update';
  resourceType: 'user' | 'product' | 'order';
  id: string;
  changes: Record<string, any>;
};

type DeleteCommand = {
  type: 'delete';
  resourceType: 'user' | 'product' | 'order';
  id: string;
  force?: boolean;
};

type ListCommand = {
  type: 'list';
  resourceType: 'user' | 'product' | 'order';
  filter?: string;
  limit?: number;
};

type HelpCommand = {
  type: 'help';
  commandName?: 'create' | 'update' | 'delete' | 'list';
};

type Command = CreateCommand | UpdateCommand | DeleteCommand | ListCommand | HelpCommand;

// Command execution result
type CommandResult = {
  success: boolean;
  message: string;
  data?: any;
};

// Execute command using pattern matching
function executeCommand(command: Command): CommandResult {
  switch (command.type) {
    case 'create':
      console.log(`Creating ${command.resourceType}: ${command.name}`);
      console.log('Data:', command.data);
      
      // Simulate creation
      const newId = Math.random().toString(36).substr(2, 9);
      return {
        success: true,
        message: `${command.resourceType} '${command.name}' created successfully`,
        data: { id: newId, name: command.name, ...command.data }
      };
    
    case 'update':
      console.log(`Updating ${command.resourceType} ${command.id}`);
      console.log('Changes:', command.changes);
      
      // Validate ID exists (simulated)
      if (command.id.length < 3) {
        return {
          success: false,
          message: `${command.resourceType} not found: ${command.id}`
        };
      }
      
      return {
        success: true,
        message: `${command.resourceType} ${command.id} updated`,
        data: { id: command.id, ...command.changes }
      };
    
    case 'delete':
      console.log(`Deleting ${command.resourceType} ${command.id}`);
      
      if (!command.force) {
        return {
          success: false,
          message: `Add --force flag to confirm deletion of ${command.id}`
        };
      }
      
      return {
        success: true,
        message: `${command.resourceType} ${command.id} deleted permanently`
      };
    
    case 'list':
      console.log(`Listing ${command.resourceType}s`);
      if (command.filter) {
        console.log(`Filter: ${command.filter}`);
      }
      
      // Simulate listing
      const items = Array.from({ length: command.limit || 5 }, (_, i) => ({
        id: `${command.resourceType}-${i + 1}`,
        name: `${command.resourceType} ${i + 1}`
      }));
      
      return {
        success: true,
        message: `Found ${items.length} ${command.resourceType}s`,
        data: items
      };
    
    case 'help':
      return {
        success: true,
        message: getCommandHelp(command.commandName)
      };
    
    default:
      const _exhaustive: never = command;
      return _exhaustive;
  }
}

// Get help text using pattern matching
function getCommandHelp(commandName?: string): string {
  if (!commandName) {
    return `
Available Commands:
  create  - Create a new resource
  update  - Update an existing resource
  delete  - Delete a resource
  list    - List resources
  help    - Show this help message

Use 'help <command>' for detailed information.
    `.trim();
  }
  
  switch (commandName) {
    case 'create':
      return `
CREATE Command:
  Usage: create <type> <name> [options]
  
  Examples:
    create user "Alice" --email alice@example.com --age 30
    create product "Laptop" --price 999 --stock 50
      `.trim();
    
    case 'update':
      return `
UPDATE Command:
  Usage: update <type> <id> [changes]
  
  Examples:
    update user U123 --email newemail@example.com
    update product P456 --price 899 --stock 45
      `.trim();
    
    case 'delete':
      return `
DELETE Command:
  Usage: delete <type> <id> [--force]
  
  Examples:
    delete user U123 --force
    delete product P456 --force
      `.trim();
    
    case 'list':
      return `
LIST Command:
  Usage: list <type> [--filter <value>] [--limit <n>]
  
  Examples:
    list users --filter active --limit 10
    list products --filter "price>100"
      `.trim();
    
    default:
      const _exhaustive: never = commandName;
      return _exhaustive;
  }
}

// Test commands
const commands: Command[] = [
  {
    type: 'create',
    resourceType: 'user',
    name: 'Alice',
    data: { email: 'alice@example.com', age: 30 }
  },
  {
    type: 'update',
    resourceType: 'user',
    id: 'U123',
    changes: { email: 'alice.new@example.com' }
  },
  {
    type: 'delete',
    resourceType: 'user',
    id: 'U456',
    force: false
  },
  {
    type: 'delete',
    resourceType: 'user',
    id: 'U456',
    force: true
  },
  {
    type: 'list',
    resourceType: 'product',
    filter: 'active',
    limit: 3
  },
  {
    type: 'help',
    commandName: 'create'
  }
];

console.log('=== Command Execution Demo ===\n');

commands.forEach((cmd, index) => {
  console.log(`\n--- Command ${index + 1} ---`);
  const result = executeCommand(cmd);
  
  if (result.success) {
    console.log(`✓ ${result.message}`);
  } else {
    console.log(`✗ ${result.message}`);
  }
  
  if (result.data) {
    console.log('Data:', result.data);
  }
});

// Usage with pattern matching on results
function handleCommandResult(result: CommandResult): void {
  if (result.success) {
    console.log(`✓ Success: ${result.message}`);
    if (result.data) {
      if (Array.isArray(result.data)) {
        console.log(`  Items: ${result.data.length}`);
      } else {
        console.log(`  ID: ${result.data.id || 'N/A'}`);
      }
    }
  } else {
    console.error(`✗ Error: ${result.message}`);
  }
}

console.log('\n\n=== Using Result Handler ===\n');

const createResult = executeCommand({
  type: 'create',
  resourceType: 'product',
  name: 'New Laptop',
  data: { price: 1299, stock: 10 }
});

handleCommandResult(createResult);
```

**Explanation**:
- Each command type has specific required fields enforced by TypeScript
- Pattern matching on `command.type` ensures all cases are handled
- Exhaustiveness checking with `never` catches missing command types
- Nested pattern matching on command results provides type-safe handling
- This pattern is common in CLI tools, REST APIs, and event systems

</details>

</details>

---

**Resources for Further Learning**:
- **TypeScript Discriminated Unions**: [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions)
- **ts-pattern** (Pattern matching library): [github.com/gvergnaud/ts-pattern](https://github.com/gvergnaud/ts-pattern)
- **Go Type Switches**: [Go by Example - Type Switches](https://gobyexample.com/type-switches)

## Lazy Evaluation

**Definition**: Delaying computation until the result is actually needed. Instead of computing values immediately, create "recipes" that compute on demand.

**Real-Life Analogy**:  
Think of a restaurant menu. The menu doesn't have pre-cooked food - it's a promise of food. The cooking only happens when you order (when the value is needed). This saves resources - no wasted food for dishes nobody orders.

---

### Why Lazy Evaluation?

**Benefits**:
1. **Infinite data structures**: Work with infinite sequences
2. **Performance**: Only compute what's needed
3. **Composability**: Chain operations without intermediate results
4. **Memory efficiency**: Process large datasets without loading everything

**Trade-offs**:
- Harder to predict when code executes
- Debugging can be trickier
- May cache computed values (memoization)

---

### Lazy Evaluation in JavaScript/TypeScript

JavaScript is **eager** by default, but we can implement lazy evaluation with:
1. Generators
2. Closures
3. Thunks (functions that delay computation)

**Real-World Scenario: Processing Large Datasets**

<details>
<summary><strong>View Examples</strong></summary>

```ts
// ❌ EAGER: Processes entire array immediately
function eagerProcessing(numbers: number[]): number[] {
  console.log('Mapping (eager)...');
  const doubled = numbers.map(n => {
    console.log(`Doubling ${n}`);
    return n * 2;
  });
  
  console.log('Filtering (eager)...');
  const filtered = doubled.filter(n => {
    console.log(`Filtering ${n}`);
    return n > 10;
  });
  
  console.log('Taking first 3 (eager)...');
  return filtered.slice(0, 3);
}

// Even though we only want 3 items, it processes ALL items
const eagerResult = eagerProcessing([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);
console.log('Result:', eagerResult);
// Logs show it processes all 10 items before returning 3

// ✅ LAZY: Using generators
function* lazyMap<T, U>(
  iterable: Iterable<T>,
  fn: (value: T) => U
): Generator<U> {
  for (const item of iterable) {
    console.log(`Lazy mapping ${item}`);
    yield fn(item);
  }
}

function* lazyFilter<T>(
  iterable: Iterable<T>,
  predicate: (value: T) => boolean
): Generator<T> {
  for (const item of iterable) {
    console.log(`Lazy filtering ${item}`);
    if (predicate(item)) {
      yield item;
    }
  }
}

function* lazyTake<T>(iterable: Iterable<T>, count: number): Generator<T> {
  let taken = 0;
  for (const item of iterable) {
    if (taken >= count) break;
    yield item;
    taken++;
  }
}

// Create lazy pipeline
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const lazyPipeline = lazyTake(
  lazyFilter(
    lazyMap(numbers, n => n * 2),
    n => n > 10
  ),
  3
);

console.log('\n=== Lazy Evaluation ===');
const lazyResult = Array.from(lazyPipeline);
console.log('Result:', lazyResult);
// Only processes items until it gets 3 results!

// Real-world: Reading large log files
interface LogEntry {
  timestamp: Date;
  level: 'INFO' | 'WARN' | 'ERROR';
  message: string;
  userId?: string;
}

function* readLogFile(filename: string): Generator<LogEntry> {
  // Simulate reading large file line by line
  const lines = [
    '2024-02-06T10:00:00Z INFO User logged in userId=U1',
    '2024-02-06T10:01:00Z WARN High memory usage',
    '2024-02-06T10:02:00Z ERROR Database connection failed',
    '2024-02-06T10:03:00Z INFO User logged out userId=U1',
    '2024-02-06T10:04:00Z ERROR Payment processing failed userId=U2',
    // ... imagine millions more lines
  ];
  
  for (const line of lines) {
    console.log(`Reading line: ${line.substring(0, 30)}...`);
    
    const [timestamp, level, ...messageParts] = line.split(' ');
    const message = messageParts.join(' ');
    const userIdMatch = message.match(/userId=(\w+)/);
    
    yield {
      timestamp: new Date(timestamp),
      level: level as LogEntry['level'],
      message,
      userId: userIdMatch ? userIdMatch[1] : undefined
    };
  }
}

function* filterByLevel<T extends LogEntry>(
  logs: Iterable<T>,
  level: LogEntry['level']
): Generator<T> {
  for (const log of logs) {
    if (log.level === level) {
      yield log;
    }
  }
}

function* filterByUser<T extends LogEntry>(
  logs: Iterable<T>,
  userId: string
): Generator<T> {
  for (const log of logs) {
    if (log.userId === userId) {
      yield log;
    }
  }
}

// Lazy log processing - only reads what's needed
console.log('\n=== Lazy Log Processing ===');
const errorLogs = lazyTake(
  filterByLevel(readLogFile('app.log'), 'ERROR'),
  2 // Only get first 2 errors
);

for (const log of errorLogs) {
  console.log(`Found error: ${log.message}`);
}
// Only reads until it finds 2 errors, doesn't process entire file!

// Infinite sequences with lazy evaluation
function* infiniteNumbers(start: number = 0): Generator<number> {
  let current = start;
  while (true) {
    yield current++;
  }
}

function* fibonacci(): Generator<number> {
  let [a, b] = [0, 1];
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

// Take first 10 Fibonacci numbers
console.log('\n=== Infinite Sequences ===');
const first10Fib = Array.from(lazyTake(fibonacci(), 10));
console.log('First 10 Fibonacci:', first10Fib);
// [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

// Real-world: Paginated API data
interface ApiPage<T> {
  data: T[];
  nextPage?: number;
}

function* fetchAllPages<T>(
  fetchPage: (page: number) => ApiPage<T>
): Generator<T> {
  let page = 1;
  while (true) {
    console.log(`Fetching page ${page}...`);
    const result = fetchPage(page);
    
    for (const item of result.data) {
      yield item;
    }
    
    if (!result.nextPage) break;
    page = result.nextPage;
  }
}

// Simulate API
function fetchUsersPage(page: number): ApiPage<{ id: string; name: string }> {
  if (page > 3) {
    return { data: [] }; // No more pages
  }
  
  return {
    data: [
      { id: `U${page}-1`, name: `User ${page}-1` },
      { id: `U${page}-2`, name: `User ${page}-2` },
    ],
    nextPage: page + 1
  };
}

// Lazy pagination - only fetches pages as needed
console.log('\n=== Lazy Pagination ===');
const first5Users = lazyTake(fetchAllPages(fetchUsersPage), 5);

for (const user of first5Users) {
  console.log(`Processing user: ${user.name}`);
}
// Only fetches pages 1-3 to get 5 users, doesn't fetch all pages!

// Lazy evaluation with caching (memoization)
class LazyValue<T> {
  private computed = false;
  private cachedValue?: T;
  
  constructor(private computation: () => T) {}
  
  get value(): T {
    if (!this.computed) {
      console.log('Computing value...');
      this.cachedValue = this.computation();
      this.computed = true;
    } else {
      console.log('Using cached value...');
    }
    return this.cachedValue!;
  }
}

// Example: Expensive computation
const expensiveResult = new LazyValue(() => {
  console.log('Performing expensive calculation...');
  let sum = 0;
  for (let i = 0; i < 1000000; i++) {
    sum += i;
  }
  return sum;
});

console.log('\n=== Lazy Value ===');
console.log('Created lazy value (not computed yet)');
// ... do other work ...
console.log('Accessing value first time:');
console.log(expensiveResult.value); // Computes here
console.log('Accessing value second time:');
console.log(expensiveResult.value); // Uses cache

// Lazy property initialization
class UserProfile {
  private _friends?: string[];
  
  constructor(private userId: string) {}
  
  // Lazy-loaded friends list
  get friends(): string[] {
    if (!this._friends) {
      console.log(`Loading friends for ${this.userId}...`);
      // Simulate expensive database query
      this._friends = [`friend1`, `friend2`, `friend3`];
    }
    return this._friends;
  }
}

console.log('\n=== Lazy Properties ===');
const profile = new UserProfile('U123');
console.log('Profile created (friends not loaded)');
// ... other operations ...
console.log('Accessing friends:');
console.log(profile.friends); // Loads here
console.log('Accessing friends again:');
console.log(profile.friends); // Uses cached value
```

**Go Example:**

```go
package main

import (
    "fmt"
    "time"
)

// Lazy evaluation using channels and goroutines

// Lazy sequence using channels
func lazyMap(input <-chan int, fn func(int) int) <-chan int {
    output := make(chan int)
    go func() {
        defer close(output)
        for value := range input {
            fmt.Printf("Lazy mapping %d\n", value)
            output <- fn(value)
        }
    }()
    return output
}

func lazyFilter(input <-chan int, predicate func(int) bool) <-chan int {
    output := make(chan int)
    go func() {
        defer close(output)
        for value := range input {
            fmt.Printf("Lazy filtering %d\n", value)
            if predicate(value) {
                output <- value
            }
        }
    }()
    return output
}

func lazyTake(input <-chan int, count int) <-chan int {
    output := make(chan int)
    go func() {
        defer close(output)
        taken := 0
        for value := range input {
            if taken >= count {
                break
            }
            output <- value
            taken++
        }
    }()
    return output
}

// Generate numbers lazily
func numberGenerator(start, end int) <-chan int {
    output := make(chan int)
    go func() {
        defer close(output)
        for i := start; i <= end; i++ {
            output <- i
        }
    }()
    return output
}

// Infinite sequence
func infiniteNumbers(start int) <-chan int {
    output := make(chan int)
    go func() {
        current := start
        for {
            output <- current
            current++
        }
    }()
    return output
}

// Fibonacci sequence
func fibonacci() <-chan int {
    output := make(chan int)
    go func() {
        a, b := 0, 1
        for {
            output <- a
            a, b = b, a+b
        }
    }()
    return output
}

// Lazy value with caching
type LazyValue struct {
    computation func() int
    computed    bool
    cachedValue int
}

func NewLazyValue(computation func() int) *LazyValue {
    return &LazyValue{computation: computation}
}

func (lv *LazyValue) Value() int {
    if !lv.computed {
        fmt.Println("Computing value...")
        lv.cachedValue = lv.computation()
        lv.computed = true
    } else {
        fmt.Println("Using cached value...")
    }
    return lv.cachedValue
}

func main() {
    // Lazy pipeline
    fmt.Println("=== Lazy Pipeline ===")
    numbers := numberGenerator(1, 10)
    doubled := lazyMap(numbers, func(n int) int { return n * 2 })
    filtered := lazyFilter(doubled, func(n int) bool { return n > 10 })
    result := lazyTake(filtered, 3)
    
    for value := range result {
        fmt.Printf("Result: %d\n", value)
    }
    
    // Infinite sequence
    fmt.Println("\n=== Infinite Fibonacci ===")
    fib := fibonacci()
    first10 := lazyTake(fib, 10)
    
    for value := range first10 {
        fmt.Printf("%d ", value)
    }
    fmt.Println()
    
    // Lazy value
    fmt.Println("\n=== Lazy Value ===")
    expensiveResult := NewLazyValue(func() int {
        fmt.Println("Performing expensive calculation...")
        time.Sleep(100 * time.Millisecond)
        sum := 0
        for i := 0; i < 1000; i++ {
            sum += i
        }
        return sum
    })
    
    fmt.Println("Created lazy value")
    time.Sleep(50 * time.Millisecond)
    fmt.Println("Accessing first time:")
    fmt.Println(expensiveResult.Value())
    fmt.Println("Accessing second time:")
    fmt.Println(expensiveResult.Value())
}
```

</details>

---

### Lazy vs Eager: Performance Comparison

<details>
<summary><strong>View Examples</strong></summary>

```ts
// Performance comparison: Eager vs Lazy

// Dataset: 1 million numbers
const largeDataset = Array.from({ length: 1_000_000 }, (_, i) => i);

console.log('=== Performance Comparison ===\n');

// EAGER approach
console.time('Eager processing');
const eagerResults = largeDataset
  .map(n => n * 2)
  .filter(n => n % 3 === 0)
  .filter(n => n > 1000)
  .slice(0, 10);
console.timeEnd('Eager processing');
console.log('Eager results:', eagerResults.length);
// Creates 3 intermediate arrays of 1 million items each!

// LAZY approach
function* lazyPipeline() {
  for (let i = 0; i < 1_000_000; i++) {
    const doubled = i * 2;
    if (doubled % 3 !== 0) continue;
    if (doubled <= 1000) continue;
    yield doubled;
  }
}

console.time('Lazy processing');
const lazyResults = Array.from(lazyTake(lazyPipeline(), 10));
console.timeEnd('Lazy processing');
console.log('Lazy results:', lazyResults.length);
// Processes only what's needed to get 10 results!

// Real-world: Search in large dataset
interface Product {
  id: string;
  name: string;
  price: number;
  category: string;
  inStock: boolean;
}

// Generate large product catalog
const products: Product[] = Array.from({ length: 100_000 }, (_, i) => ({
  id: `P${i}`,
  name: `Product ${i}`,
  price: Math.random() * 1000,
  category: ['electronics', 'clothing', 'books', 'food'][i % 4],
  inStock: Math.random() > 0.3
}));

// EAGER: Find first 5 expensive electronics in stock
console.log('\n=== Product Search (Eager) ===');
console.time('Eager search');
const eagerSearch = products
  .filter(p => p.category === 'electronics')
  .filter(p => p.inStock)
  .filter(p => p.price > 500)
  .sort((a, b) => b.price - a.price)
  .slice(0, 5);
console.timeEnd('Eager search');
console.log(`Found ${eagerSearch.length} products`);

// LAZY: Same search
console.log('\n=== Product Search (Lazy) ===');
console.time('Lazy search');

function* searchProducts(): Generator<Product> {
  for (const product of products) {
    if (
      product.category === 'electronics' &&
      product.inStock &&
      product.price > 500
    ) {
      yield product;
    }
  }
}

const lazySearch = Array.from(lazyTake(searchProducts(), 5))
  .sort((a, b) => b.price - a.price);

console.timeEnd('Lazy search');
console.log(`Found ${lazySearch.length} products`);

// Memory usage comparison
console.log('\n=== Memory Comparison ===');

// Eager: Loads everything into memory
function eagerProcessLargeFile(): number[] {
  const allLines: number[] = [];
  // Simulate reading 10 million lines
  for (let i = 0; i < 10_000_000; i++) {
    allLines.push(i);
  }
  
  // Then process
  return allLines
    .filter(n => n % 2 === 0)
    .map(n => n * 2)
    .slice(0, 100);
}

// Lazy: Processes on-the-fly
function* lazyProcessLargeFile(): Generator<number> {
  // Simulate reading line-by-line
  for (let i = 0; i < 10_000_000; i++) {
    if (i % 2 === 0) {
      yield i * 2;
    }
  }
}

console.log('Eager: Loads 10M items into memory');
console.log('Lazy: Processes one item at a time');

console.time('Lazy file processing');
const lazyFileResult = Array.from(lazyTake(lazyProcessLargeFile(), 100));
console.timeEnd('Lazy file processing');
console.log(`Processed ${lazyFileResult.length} items`);

// When to use each?
console.log('\n=== Guidelines ===');
console.log(`
Use EAGER when:
✓ Dataset is small
✓ Need the entire result
✓ Will use the data multiple times
✓ Want predictable execution timing

Use LAZY when:
✓ Large or infinite datasets
✓ Only need partial results
✓ Expensive computations
✓ Memory constrained
✓ Processing pipelines with early termination
`);
```

</details>

---

## Practice Questions

<details>
<summary><strong>View Questions</strong></summary>

### Fill in the Blanks

1. Lazy evaluation delays __________ until the value is actually needed.
2. JavaScript generators use the __________ keyword to produce values on demand.
3. Lazy evaluation enables working with __________ sequences like Fibonacci.
4. The main performance benefit of lazy evaluation is avoiding __________ arrays.

<details>
<summary><strong>View Answers</strong></summary>

1. **computation** (or "execution") - Lazy evaluation creates a "recipe" that executes only when values are requested
2. **yield** - The yield keyword pauses the generator and returns a value, resuming when the next value is requested
3. **infinite** - Since values are computed on-demand, you can represent infinite sequences without actually storing infinite values
4. **intermediate** - Eager evaluation creates temporary arrays at each step; lazy evaluation processes items one at a time

</details>

---

### True/False

1. ⬜ JavaScript is lazy by default like Haskell
2. ⬜ Lazy evaluation always uses less memory than eager evaluation
3. ⬜ Generators in JavaScript provide lazy evaluation
4. ⬜ Lazy evaluation can improve performance when you only need the first few results from a large dataset

<details>
<summary><strong>View Answers</strong></summary>

1. **False** - JavaScript is eager by default. Lazy evaluation must be explicitly implemented using generators, closures, or libraries
2. **True (mostly)** - Lazy evaluation processes items one at a time instead of creating intermediate collections, using constant memory for pipelines
3. **True** - Generators yield values on-demand, computing only what's needed when `next()` is called
4. **True** - Lazy evaluation stops processing once you have enough results, avoiding unnecessary computation on the rest of the dataset

</details>

---

### Multiple Choice

1. What's the main advantage of lazy evaluation?

- A) Faster code execution
- B) Easier debugging
- C) Only compute what's needed
- D) Better type safety

2. Which is a good use case for lazy evaluation?

- A) Sorting a small array
- B) Reading and filtering a 10GB log file
- C) Simple arithmetic
- D) Rendering a small list

<details>
<summary><strong>View Answers</strong></summary>

1. **C** - Only compute what's needed. Lazy evaluation defers computation and stops early when possible, avoiding wasted work

2. **B** - Reading and filtering a 10GB log file. Lazy evaluation can process huge files line-by-line without loading everything into memory

</details>

---

### Code Challenge

Build a lazy data processing pipeline for analytics:

```ts
// TODO: Implement lazy operations for:
// 1. Stream of events from multiple sources
// 2. Filter by event type
// 3. Transform events to metrics
// 4. Aggregate metrics (running totals)
// 5. Take first N results

// Should handle infinite event streams efficiently
```

<details>
<summary><strong>View Answers</strong></summary>

```ts
// Event types
type UserEvent = {
  type: 'page_view' | 'click' | 'purchase' | 'signup';
  userId: string;
  timestamp: Date;
  data: Record<string, any>;
};

type Metric = {
  eventType: string;
  count: number;
  totalValue: number;
  averageValue: number;
  timestamp: Date;
};

// Lazy event stream generator
function* eventStream(): Generator<UserEvent> {
  const eventTypes: UserEvent['type'][] = ['page_view', 'click', 'purchase', 'signup'];
  let eventId = 0;
  
  while (true) {
    const type = eventTypes[eventId % eventTypes.length];
    const userId = `U${Math.floor(eventId / 10)}`;
    
    yield {
      type,
      userId,
      timestamp: new Date(),
      data: {
        value: type === 'purchase' ? Math.random() * 100 : 1,
        page: `/page${eventId % 5}`
      }
    };
    
    eventId++;
    
    // Simulate delay between events
    // In real system, this would be actual event arrival
  }
}

// Lazy filter by event type
function* filterByType<T extends UserEvent>(
  events: Iterable<T>,
  ...types: UserEvent['type'][]
): Generator<T> {
  for (const event of events) {
    if (types.includes(event.type)) {
      console.log(`Filtering: ${event.type} event from ${event.userId}`);
      yield event;
    }
  }
}

// Lazy transform events to metrics
function* transformToMetrics(
  events: Iterable<UserEvent>
): Generator<Metric> {
  const aggregates = new Map<string, { count: number; total: number }>();
  
  for (const event of events) {
    const key = event.type;
    const existing = aggregates.get(key) || { count: 0, total: 0 };
    
    const value = event.data.value || 1;
    const updated = {
      count: existing.count + 1,
      total: existing.total + value
    };
    
    aggregates.set(key, updated);
    
    console.log(`Transforming: ${event.type} (value: ${value.toFixed(2)})`);
    
    yield {
      eventType: key,
      count: updated.count,
      totalValue: updated.total,
      averageValue: updated.total / updated.count,
      timestamp: event.timestamp
    };
  }
}

// Lazy running aggregate
function* runningAggregate(
  metrics: Iterable<Metric>
): Generator<Metric> {
  let totalCount = 0;
  let totalValue = 0;
  
  for (const metric of metrics) {
    totalCount += metric.count;
    totalValue += metric.totalValue;
    
    yield {
      ...metric,
      count: totalCount,
      totalValue,
      averageValue: totalValue / totalCount
    };
  }
}

// Lazy deduplicate consecutive duplicates
function* deduplicate<T>(
  items: Iterable<T>,
  keyFn: (item: T) => string
): Generator<T> {
  let lastKey: string | null = null;
  
  for (const item of items) {
    const key = keyFn(item);
    if (key !== lastKey) {
      yield item;
      lastKey = key;
    }
  }
}

// Lazy take with condition
function* takeWhile<T>(
  items: Iterable<T>,
  predicate: (item: T) => boolean
): Generator<T> {
  for (const item of items) {
    if (!predicate(item)) {
      break;
    }
    yield item;
  }
}

// Lazy batch processing
function* batch<T>(items: Iterable<T>, size: number): Generator<T[]> {
  let currentBatch: T[] = [];
  
  for (const item of items) {
    currentBatch.push(item);
    if (currentBatch.length >= size) {
      yield currentBatch;
      currentBatch = [];
    }
  }
  
  if (currentBatch.length > 0) {
    yield currentBatch;
  }
}

// Main analytics pipeline
console.log('=== Lazy Analytics Pipeline ===\n');

// Create pipeline
const events = eventStream();
const purchaseEvents = filterByType(events, 'purchase', 'signup');
const metrics = transformToMetrics(purchaseEvents);
const aggregated = runningAggregate(metrics);
const first20 = lazyTake(aggregated, 20);

console.log('Processing first 20 metrics...\n');

for (const metric of first20) {
  console.log(`
  Event Type: ${metric.eventType}
  Total Count: ${metric.count}
  Total Value: $${metric.totalValue.toFixed(2)}
  Average: $${metric.averageValue.toFixed(2)}
  ---
  `.trim());
}

// Real-world: Time-windowed analytics
function* timeWindow<T extends { timestamp: Date }>(
  items: Iterable<T>,
  windowMs: number
): Generator<T[]> {
  let windowStart: Date | null = null;
  let currentWindow: T[] = [];
  
  for (const item of items) {
    if (!windowStart) {
      windowStart = item.timestamp;
    }
    
    const elapsed = item.timestamp.getTime() - windowStart.getTime();
    
    if (elapsed >= windowMs) {
      if (currentWindow.length > 0) {
        yield currentWindow;
      }
      currentWindow = [item];
      windowStart = item.timestamp;
    } else {
      currentWindow.push(item);
    }
  }
  
  if (currentWindow.length > 0) {
    yield currentWindow;
  }
}

// Multi-source event stream merger
function* mergeStreams<T>(
  ...streams: Array<Iterable<T>>
): Generator<T> {
  const iterators = streams.map(s => s[Symbol.iterator]());
  
  while (true) {
    let allDone = true;
    
    for (const iterator of iterators) {
      const result = iterator.next();
      if (!result.done) {
        allDone = false;
        yield result.value;
      }
    }
    
    if (allDone) break;
  }
}

// Advanced: Lazy caching with expiration
class LazyCache<K, V> {
  private cache = new Map<K, { value: V; expiresAt: number }>();
  
  constructor(private ttlMs: number) {}
  
  get(key: K, compute: () => V): V {
    const cached = this.cache.get(key);
    
    if (cached && Date.now() < cached.expiresAt) {
      console.log(`Cache hit for ${key}`);
      return cached.value;
    }
    
    console.log(`Cache miss for ${key}, computing...`);
    const value = compute();
    this.cache.set(key, {
      value,
      expiresAt: Date.now() + this.ttlMs
    });
    
    return value;
  }
  
  clear(): void {
    this.cache.clear();
  }
}

// Usage
console.log('\n=== Lazy Cache ===');
const cache = new LazyCache<string, number>(5000); // 5 second TTL

const expensive = (id: string) => {
  console.log(`Expensive computation for ${id}...`);
  return Math.random() * 100;
};

console.log(cache.get('user1', () => expensive('user1')));
console.log(cache.get('user1', () => expensive('user1'))); // Cached
console.log(cache.get('user2', () => expensive('user2')));

// Real-world: Lazy pagination with prefetch
function* lazyPaginatedFetch<T>(
  fetchPage: (page: number) => Promise<T[]>,
  prefetchCount: number = 2
): Generator<Promise<T>, void, unknown> {
  let currentPage = 1;
  const prefetchCache = new Map<number, Promise<T[]>>();
  
  // Helper to prefetch pages
  const prefetch = (page: number) => {
    if (!prefetchCache.has(page)) {
      prefetchCache.set(page, fetchPage(page));
    }
  };
  
  while (true) {
    // Prefetch upcoming pages
    for (let i = 1; i <= prefetchCount; i++) {
      prefetch(currentPage + i);
    }
    
    // Get current page
    prefetch(currentPage);
    const pageData = prefetchCache.get(currentPage)!;
    
    const items = await pageData;
    if (items.length === 0) break;
    
    for (const item of items) {
      yield Promise.resolve(item);
    }
    
    // Clean up old cached pages
    prefetchCache.delete(currentPage - 1);
    currentPage++;
  }
}

console.log('\n=== Summary ===');
console.log(`
Lazy evaluation enables:
✓ Infinite data structures
✓ Early termination (process only what's needed)
✓ Memory efficiency (one item at a time)
✓ Composable pipelines
✓ On-demand computation
`);
```

**Explanation**:
- Event stream is infinite but only generates values as needed
- Each transformation is lazy - only processes when downstream asks
- Pipeline stops at `take(20)`, avoiding unnecessary computation
- Memory usage stays constant regardless of event stream size
- Real-world patterns: time windows, caching, prefetching

</details>

</details>

---

**Resources for Further Learning**:
- **RxJS** (Reactive Extensions for JS): [rxjs.dev](https://rxjs.dev/) - Advanced lazy evaluation with observables
- **IxJS** (Interactive Extensions): [github.com/ReactiveX/IxJS](https://github.com/ReactiveX/IxJS) - Lazy iterable operations
- **lazy.js**: [danieltao.com/lazy.js](http://danieltao.com/lazy.js/) - Lazy evaluation library for JavaScript
- **Go Channels Guide**: [Go by Example - Channels](https://gobyexample.com/channels)