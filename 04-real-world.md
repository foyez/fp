# Functional Programming: Chapter 4 - Real-World Applications

## Table of Contents

- [Error Handling in FP](#error-handling-in-fp)
- [State Management](#state-management)
- [Async Operations](#async-operations)
- [Testing Pure Functions](#testing-pure-functions)

---

## Error Handling

**The Problem with Exceptions**:
- Break referential transparency
- Hidden control flow (not visible in function signatures)
- Hard to compose
- Can crash your program

**FP Solution**: Make errors explicit using types (Result/Either pattern).

---

### Result/Either Pattern

**Real-World Scenario: User Registration System**

<details>
<summary><strong>View Examples</strong></summary>

```ts
// Result type (already seen, but expanded)
type Result<T, E = Error> = 
  | { ok: true; value: T }
  | { ok: false; error: E };

// Helper constructors
const Ok = <T, E = Error>(value: T): Result<T, E> => ({ ok: true, value });
const Err = <T, E = Error>(error: E): Result<T, E> => ({ ok: false, error });

// Error types for different failures
type ValidationError = {
  type: 'validation';
  field: string;
  message: string;
};

type DatabaseError = {
  type: 'database';
  code: string;
  message: string;
};

type NetworkError = {
  type: 'network';
  statusCode: number;
  message: string;
};

type EmailError = {
  type: 'email';
  message: string;
};

type AppError = ValidationError | DatabaseError | NetworkError | EmailError;

// Domain types
interface UserRegistration {
  email: string;
  password: string;
  username: string;
  age: number;
}

interface User {
  id: string;
  email: string;
  username: string;
  createdAt: Date;
}

// Validation functions
function validateEmail(email: string): Result<string, ValidationError> {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  
  if (!email) {
    return Err({ type: 'validation', field: 'email', message: 'Email is required' });
  }
  
  if (!emailRegex.test(email)) {
    return Err({ type: 'validation', field: 'email', message: 'Invalid email format' });
  }
  
  return Ok(email.toLowerCase());
}

function validatePassword(password: string): Result<string, ValidationError> {
  if (!password) {
    return Err({ type: 'validation', field: 'password', message: 'Password is required' });
  }
  
  if (password.length < 8) {
    return Err({ type: 'validation', field: 'password', message: 'Password must be at least 8 characters' });
  }
  
  if (!/[A-Z]/.test(password)) {
    return Err({ type: 'validation', field: 'password', message: 'Password must contain uppercase letter' });
  }
  
  if (!/[0-9]/.test(password)) {
    return Err({ type: 'validation', field: 'password', message: 'Password must contain a number' });
  }
  
  return Ok(password);
}

function validateUsername(username: string): Result<string, ValidationError> {
  if (!username) {
    return Err({ type: 'validation', field: 'username', message: 'Username is required' });
  }
  
  if (username.length < 3) {
    return Err({ type: 'validation', field: 'username', message: 'Username must be at least 3 characters' });
  }
  
  if (!/^[a-zA-Z0-9_]+$/.test(username)) {
    return Err({ type: 'validation', field: 'username', message: 'Username can only contain letters, numbers, and underscores' });
  }
  
  return Ok(username);
}

function validateAge(age: number): Result<number, ValidationError> {
  if (age < 13) {
    return Err({ type: 'validation', field: 'age', message: 'Must be at least 13 years old' });
  }
  
  if (age > 120) {
    return Err({ type: 'validation', field: 'age', message: 'Invalid age' });
  }
  
  return Ok(age);
}

// Database operations (simulated)
function checkEmailExists(email: string): Result<boolean, DatabaseError> {
  // Simulate database check
  const existingEmails = ['admin@example.com', 'test@example.com'];
  
  if (Math.random() < 0.05) {
    return Err({ 
      type: 'database', 
      code: 'CONNECTION_ERROR', 
      message: 'Failed to connect to database' 
    });
  }
  
  return Ok(existingEmails.includes(email));
}

function saveUser(registration: UserRegistration): Result<User, DatabaseError> {
  // Simulate database save
  if (Math.random() < 0.05) {
    return Err({ 
      type: 'database', 
      code: 'WRITE_ERROR', 
      message: 'Failed to save user to database' 
    });
  }
  
  const user: User = {
    id: `U${Math.random().toString(36).substr(2, 9)}`,
    email: registration.email,
    username: registration.username,
    createdAt: new Date()
  };
  
  return Ok(user);
}

// Email service (simulated)
function sendWelcomeEmail(user: User): Result<void, EmailError> {
  // Simulate email sending
  if (Math.random() < 0.1) {
    return Err({ 
      type: 'email', 
      message: 'Failed to send welcome email' 
    });
  }
  
  console.log(`✉️  Welcome email sent to ${user.email}`);
  return Ok(undefined);
}

// Helper functions for Result
function map<T, U, E>(
  result: Result<T, E>,
  fn: (value: T) => U
): Result<U, E> {
  if (result.ok) {
    return Ok(fn(result.value));
  }
  return result;
}

function flatMap<T, U, E>(
  result: Result<T, E>,
  fn: (value: T) => Result<U, E>
): Result<U, E> {
  if (result.ok) {
    return fn(result.value);
  }
  return result;
}

function mapError<T, E1, E2>(
  result: Result<T, E1>,
  fn: (error: E1) => E2
): Result<T, E2> {
  if (!result.ok) {
    return Err(fn(result.error));
  }
  return Ok(result.value);
}

// Combine multiple Results
function combine<T, E>(results: Result<T, E>[]): Result<T[], E> {
  const values: T[] = [];
  
  for (const result of results) {
    if (!result.ok) {
      return result;
    }
    values.push(result.value);
  }
  
  return Ok(values);
}

// Validate all fields
function validateRegistration(
  registration: UserRegistration
): Result<UserRegistration, ValidationError> {
  const emailResult = validateEmail(registration.email);
  if (!emailResult.ok) return emailResult as Result<UserRegistration, ValidationError>;
  
  const passwordResult = validatePassword(registration.password);
  if (!passwordResult.ok) return passwordResult as Result<UserRegistration, ValidationError>;
  
  const usernameResult = validateUsername(registration.username);
  if (!usernameResult.ok) return usernameResult as Result<UserRegistration, ValidationError>;
  
  const ageResult = validateAge(registration.age);
  if (!ageResult.ok) return ageResult as Result<UserRegistration, ValidationError>;
  
  return Ok({
    email: emailResult.value,
    password: passwordResult.value,
    username: usernameResult.value,
    age: ageResult.value
  });
}

// Main registration flow
function registerUser(registration: UserRegistration): Result<User, AppError> {
  // Step 1: Validate
  const validationResult = validateRegistration(registration);
  if (!validationResult.ok) {
    return validationResult;
  }
  
  // Step 2: Check if email exists
  const emailCheckResult = checkEmailExists(registration.email);
  if (!emailCheckResult.ok) {
    return emailCheckResult;
  }
  
  if (emailCheckResult.value) {
    return Err({ 
      type: 'validation', 
      field: 'email', 
      message: 'Email already registered' 
    });
  }
  
  // Step 3: Save user
  const saveResult = saveUser(validationResult.value);
  if (!saveResult.ok) {
    return saveResult;
  }
  
  // Step 4: Send welcome email (non-blocking, log error but don't fail)
  const emailResult = sendWelcomeEmail(saveResult.value);
  if (!emailResult.ok) {
    console.warn(`⚠️  Failed to send welcome email: ${emailResult.error.message}`);
    // Continue anyway - email failure shouldn't block registration
  }
  
  return saveResult;
}

// Usage
console.log('=== User Registration with Result Pattern ===\n');

// Successful registration
const validRegistration: UserRegistration = {
  email: 'alice@example.com',
  password: 'SecurePass123',
  username: 'alice_wonderland',
  age: 25
};

const result1 = registerUser(validRegistration);
if (result1.ok) {
  console.log('✓ Registration successful!');
  console.log(`  User ID: ${result1.value.id}`);
  console.log(`  Username: ${result1.value.username}`);
  console.log(`  Email: ${result1.value.email}`);
} else {
  console.log('✗ Registration failed!');
  console.log(`  Error type: ${result1.error.type}`);
  console.log(`  Message: ${result1.error.message}`);
}

// Invalid email
const invalidEmail: UserRegistration = {
  email: 'not-an-email',
  password: 'SecurePass123',
  username: 'bob',
  age: 30
};

const result2 = registerUser(invalidEmail);
if (!result2.ok && result2.error.type === 'validation') {
  console.log('\n✗ Validation error:');
  console.log(`  Field: ${result2.error.field}`);
  console.log(`  Message: ${result2.error.message}`);
}

// Weak password
const weakPassword: UserRegistration = {
  email: 'carol@example.com',
  password: 'weak',
  username: 'carol',
  age: 28
};

const result3 = registerUser(weakPassword);
if (!result3.ok && result3.error.type === 'validation') {
  console.log('\n✗ Validation error:');
  console.log(`  Field: ${result3.error.field}`);
  console.log(`  Message: ${result3.error.message}`);
}

// Using functional composition
const registerUserFunctional = (registration: UserRegistration): Result<User, AppError> =>
  flatMap(
    validateRegistration(registration),
    validated =>
      flatMap(
        checkEmailExists(validated.email),
        exists =>
          exists
            ? Err({ type: 'validation', field: 'email', message: 'Email already registered' })
            : flatMap(
                saveUser(validated),
                user => {
                  sendWelcomeEmail(user); // Fire and forget
                  return Ok(user);
                }
              )
      )
  );

// Collect all validation errors (instead of failing on first)
function validateAllFields(
  registration: UserRegistration
): Result<UserRegistration, ValidationError[]> {
  const errors: ValidationError[] = [];
  
  const emailResult = validateEmail(registration.email);
  if (!emailResult.ok) errors.push(emailResult.error);
  
  const passwordResult = validatePassword(registration.password);
  if (!passwordResult.ok) errors.push(passwordResult.error);
  
  const usernameResult = validateUsername(registration.username);
  if (!usernameResult.ok) errors.push(usernameResult.error);
  
  const ageResult = validateAge(registration.age);
  if (!ageResult.ok) errors.push(ageResult.error);
  
  if (errors.length > 0) {
    return Err(errors);
  }
  
  return Ok({
    email: emailResult.value,
    password: passwordResult.value,
    username: usernameResult.value,
    age: ageResult.value
  });
}

// Test collecting all errors
const multipleErrors: UserRegistration = {
  email: 'invalid',
  password: 'weak',
  username: 'ab',
  age: 10
};

const result4 = validateAllFields(multipleErrors);
if (!result4.ok) {
  console.log('\n✗ Multiple validation errors:');
  result4.error.forEach(err => {
    console.log(`  - ${err.field}: ${err.message}`);
  });
}

// Async version with Promises
async function registerUserAsync(
  registration: UserRegistration
): Promise<Result<User, AppError>> {
  // Simulate async operations
  await new Promise(resolve => setTimeout(resolve, 100));
  return registerUser(registration);
}

// Usage
(async () => {
  console.log('\n=== Async Registration ===');
  const asyncResult = await registerUserAsync(validRegistration);
  
  if (asyncResult.ok) {
    console.log('✓ Async registration successful!');
  } else {
    console.log('✗ Async registration failed!');
  }
})();
```

**Go Example:**

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

// Result type using generics
type Result[T any] struct {
    value T
    err   error
    ok    bool
}

func Ok[T any](value T) Result[T] {
    return Result[T]{value: value, ok: true}
}

func Err[T any](err error) Result[T] {
    var zero T
    return Result[T]{value: zero, err: err, ok: false}
}

// Error types
type ValidationError struct {
    Field   string
    Message string
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("%s: %s", e.Field, e.Message)
}

// Domain types
type UserRegistration struct {
    Email    string
    Password string
    Username string
    Age      int
}

type User struct {
    ID        string
    Email     string
    Username  string
}

// Validation functions
func validateEmail(email string) Result[string] {
    if email == "" {
        return Err[string](ValidationError{Field: "email", Message: "Email is required"})
    }
    
    emailRegex := regexp.MustCompile(`^[^\s@]+@[^\s@]+\.[^\s@]+$`)
    if !emailRegex.MatchString(email) {
        return Err[string](ValidationError{Field: "email", Message: "Invalid email format"})
    }
    
    return Ok(strings.ToLower(email))
}

func validatePassword(password string) Result[string] {
    if password == "" {
        return Err[string](ValidationError{Field: "password", Message: "Password is required"})
    }
    
    if len(password) < 8 {
        return Err[string](ValidationError{Field: "password", Message: "Password must be at least 8 characters"})
    }
    
    return Ok(password)
}

func validateUsername(username string) Result[string] {
    if username == "" {
        return Err[string](ValidationError{Field: "username", Message: "Username is required"})
    }
    
    if len(username) < 3 {
        return Err[string](ValidationError{Field: "username", Message: "Username must be at least 3 characters"})
    }
    
    return Ok(username)
}

// FlatMap helper
func FlatMap[T, U any](r Result[T], fn func(T) Result[U]) Result[U] {
    if !r.ok {
        return Err[U](r.err)
    }
    return fn(r.value)
}

// Validate registration
func validateRegistration(reg UserRegistration) Result[UserRegistration] {
    emailResult := validateEmail(reg.Email)
    if !emailResult.ok {
        return Err[UserRegistration](emailResult.err)
    }
    
    passwordResult := validatePassword(reg.Password)
    if !passwordResult.ok {
        return Err[UserRegistration](passwordResult.err)
    }
    
    usernameResult := validateUsername(reg.Username)
    if !usernameResult.ok {
        return Err[UserRegistration](usernameResult.err)
    }
    
    return Ok(UserRegistration{
        Email:    emailResult.value,
        Password: passwordResult.value,
        Username: usernameResult.value,
        Age:      reg.Age,
    })
}

// Save user (simulated)
func saveUser(reg UserRegistration) Result[User] {
    user := User{
        ID:       "U123",
        Email:    reg.Email,
        Username: reg.Username,
    }
    return Ok(user)
}

// Register user
func registerUser(reg UserRegistration) Result[User] {
    return FlatMap(
        validateRegistration(reg),
        func(validated UserRegistration) Result[User] {
            return saveUser(validated)
        },
    )
}

func main() {
    fmt.Println("=== User Registration ===\n")
    
    validReg := UserRegistration{
        Email:    "alice@example.com",
        Password: "SecurePass123",
        Username: "alice",
        Age:      25,
    }
    
    result := registerUser(validReg)
    
    if result.ok {
        fmt.Println("✓ Registration successful!")
        fmt.Printf("  User ID: %s\n", result.value.ID)
        fmt.Printf("  Username: %s\n", result.value.Username)
    } else {
        fmt.Println("✗ Registration failed!")
        fmt.Printf("  Error: %v\n", result.err)
    }
    
    // Invalid registration
    invalidReg := UserRegistration{
        Email:    "not-an-email",
        Password: "weak",
        Username: "ab",
        Age:      10,
    }
    
    invalidResult := registerUser(invalidReg)
    if !invalidResult.ok {
        fmt.Printf("\n✗ Error: %v\n", invalidResult.err)
    }
}
```

</details>

---

### Railway Oriented Programming

**Concept**: Think of your program as railway tracks - success track and error track. Once you're on the error track, you stay there until handled.

<details>
<summary><strong>View Examples</strong></summary>

```ts
// Railway-oriented programming visualization:
//
// Input ──→ [Validate] ──→ [Check DB] ──→ [Save] ──→ [Email] ──→ Success
//              ↓              ↓            ↓          ↓
//           [Error] ──────→ [Error] ───→ [Error] ──→ [Error] ──→ Failure

// Pipeline builder
class Pipeline<T, E> {
  constructor(private result: Result<T, E>) {}
  
  static of<T, E>(value: T): Pipeline<T, E> {
    return new Pipeline(Ok<T, E>(value));
  }
  
  // Continue on success track, switch to error track on failure
  then<U>(fn: (value: T) => Result<U, E>): Pipeline<U, E> {
    if (!this.result.ok) {
      return new Pipeline(this.result as Result<U, E>);
    }
    return new Pipeline(fn(this.result.value));
  }
  
  // Transform value (stays on same track)
  map<U>(fn: (value: T) => U): Pipeline<U, E> {
    if (!this.result.ok) {
      return new Pipeline(this.result as Result<U, E>);
    }
    return new Pipeline(Ok<U, E>(fn(this.result.value)));
  }
  
  // Transform error
  mapError<F>(fn: (error: E) => F): Pipeline<T, F> {
    if (this.result.ok) {
      return new Pipeline(Ok<T, F>(this.result.value));
    }
    return new Pipeline(Err<T, F>(fn(this.result.error)));
  }
  
  // Execute side effect (doesn't change track)
  tap(fn: (value: T) => void): Pipeline<T, E> {
    if (this.result.ok) {
      fn(this.result.value);
    }
    return this;
  }
  
  // Execute side effect on error
  tapError(fn: (error: E) => void): Pipeline<T, E> {
    if (!this.result.ok) {
      fn(this.result.error);
    }
    return this;
  }
  
  // Recover from error (switch back to success track)
  recover(fn: (error: E) => T): Pipeline<T, E> {
    if (!this.result.ok) {
      return new Pipeline(Ok<T, E>(fn(this.result.error)));
    }
    return this;
  }
  
  // Get final result
  run(): Result<T, E> {
    return this.result;
  }
}

// Real-world: Payment processing pipeline
interface PaymentRequest {
  userId: string;
  amount: number;
  currency: string;
  paymentMethod: string;
}

interface ValidatedPayment {
  userId: string;
  amount: number;
  currency: string;
  paymentMethodId: string;
}

interface ProcessedPayment {
  transactionId: string;
  userId: string;
  amount: number;
  status: 'success';
  timestamp: Date;
}

type PaymentError =
  | { type: 'validation'; message: string }
  | { type: 'insufficient_funds'; balance: number; required: number }
  | { type: 'payment_failed'; reason: string }
  | { type: 'notification_failed'; message: string };

// Pipeline steps
function validatePayment(request: PaymentRequest): Result<ValidatedPayment, PaymentError> {
  if (request.amount <= 0) {
    return Err({ type: 'validation', message: 'Amount must be positive' });
  }
  
  if (!['USD', 'EUR', 'GBP'].includes(request.currency)) {
    return Err({ type: 'validation', message: 'Unsupported currency' });
  }
  
  return Ok({
    userId: request.userId,
    amount: request.amount,
    currency: request.currency,
    paymentMethodId: request.paymentMethod
  });
}

function checkBalance(payment: ValidatedPayment): Result<ValidatedPayment, PaymentError> {
  // Simulate balance check
  const userBalance = 1000; // Mock
  
  if (payment.amount > userBalance) {
    return Err({ 
      type: 'insufficient_funds', 
      balance: userBalance, 
      required: payment.amount 
    });
  }
  
  return Ok(payment);
}

function processPayment(payment: ValidatedPayment): Result<ProcessedPayment, PaymentError> {
  // Simulate payment processing
  if (Math.random() < 0.1) {
    return Err({ 
      type: 'payment_failed', 
      reason: 'Payment gateway timeout' 
    });
  }
  
  return Ok({
    transactionId: `TXN${Math.random().toString(36).substr(2, 9)}`,
    userId: payment.userId,
    amount: payment.amount,
    status: 'success',
    timestamp: new Date()
  });
}

function sendReceipt(payment: ProcessedPayment): Result<ProcessedPayment, PaymentError> {
  // Simulate email sending
  if (Math.random() < 0.05) {
    return Err({ 
      type: 'notification_failed', 
      message: 'Failed to send receipt email' 
    });
  }
  
  console.log(`📧 Receipt sent for transaction ${payment.transactionId}`);
  return Ok(payment);
}

// Build payment pipeline
function processPaymentPipeline(request: PaymentRequest): Result<ProcessedPayment, PaymentError> {
  return Pipeline.of<PaymentRequest, PaymentError>(request)
    .tap(req => console.log(`Processing payment for user ${req.userId}`))
    .then(validatePayment)
    .tap(() => console.log('✓ Validation passed'))
    .then(checkBalance)
    .tap(() => console.log('✓ Balance check passed'))
    .then(processPayment)
    .tap(payment => console.log(`✓ Payment processed: ${payment.transactionId}`))
    .then(sendReceipt)
    .tapError(error => console.error(`✗ Payment failed: ${error.type}`))
    .recover(error => {
      // Only recover from notification failures
      if (error.type === 'notification_failed') {
        console.warn('⚠️  Receipt failed but payment succeeded');
        // Return a mock processed payment since we can't extract it from error
        return {
          transactionId: 'UNKNOWN',
          userId: request.userId,
          amount: request.amount,
          status: 'success',
          timestamp: new Date()
        };
      }
      throw error; // Re-throw other errors
    })
    .run();
}

// Test the pipeline
console.log('=== Payment Processing Pipeline ===\n');

const validRequest: PaymentRequest = {
  userId: 'U123',
  amount: 50,
  currency: 'USD',
  paymentMethod: 'PM_card_123'
};

const result = processPaymentPipeline(validRequest);

if (result.ok) {
  console.log('\n✓ Payment successful!');
  console.log(`  Transaction ID: ${result.value.transactionId}`);
  console.log(`  Amount: $${result.value.amount}`);
} else {
  console.log('\n✗ Payment failed!');
  console.log(`  Error type: ${result.error.type}`);
  
  if (result.error.type === 'insufficient_funds') {
    console.log(`  Balance: $${result.error.balance}`);
    console.log(`  Required: $${result.error.required}`);
  }
}

// Test with invalid amount
console.log('\n=== Invalid Payment ===\n');

const invalidRequest: PaymentRequest = {
  userId: 'U123',
  amount: -10,
  currency: 'USD',
  paymentMethod: 'PM_card_123'
};

const invalidResult = processPaymentPipeline(invalidRequest);

if (!invalidResult.ok) {
  console.log('\n✗ Payment failed!');
  console.log(`  Error: ${invalidResult.error.message}`);
}

// Test with insufficient funds
console.log('\n=== Insufficient Funds ===\n');

const largeRequest: PaymentRequest = {
  userId: 'U123',
  amount: 5000,
  currency: 'USD',
  paymentMethod: 'PM_card_123'
};

const insufficientResult = processPaymentPipeline(largeRequest);

if (!insufficientResult.ok && insufficientResult.error.type === 'insufficient_funds') {
  console.log('\n✗ Insufficient funds!');
  console.log(`  Balance: $${insufficientResult.error.balance}`);
  console.log(`  Required: $${insufficientResult.error.required}`);
  console.log(`  Shortfall: $${insufficientResult.error.required - insufficientResult.error.balance}`);
}
```

</details>

---

## Practice Questions

<details>
<summary><strong>View Questions</strong></summary>

### Fill in the Blanks

1. The Result/Either pattern makes errors __________ in function signatures.
2. Railway-oriented programming visualizes programs as success and __________ tracks.
3. Unlike exceptions, Result types are __________ (can be passed around like values).
4. The `flatMap` operation is used to chain operations that return __________.

<details>
<summary><strong>View Answers</strong></summary>

1. **explicit** - Function signatures like `Result<User, Error>` clearly show the function can fail
2. **error** (or "failure") - Once on the error track, you stay there until explicitly recovered
3. **composable** - Results are values that can be stored, passed to functions, and combined
4. **Results** (or "wrapped values") - flatMap unwraps Result\<T> to T, applies a function returning Result\<U>, avoiding Result\<Result\<U>>

</details>

---

### True/False

1. ⬜ Throwing exceptions breaks referential transparency
2. ⬜ The Result pattern eliminates all runtime errors
3. ⬜ Railway-oriented programming makes error paths explicit
4. ⬜ You must handle errors immediately with the Result pattern

<details>
<summary><strong>View Answers</strong></summary>

1. **True** - Exceptions create hidden control flow; the same inputs can produce different outputs (value or exception)
2. **False** - Result handles expected errors (validation, network, etc.) but can't prevent all runtime errors (null pointer, out of memory, etc.)
3. **True** - Every operation explicitly shows whether it can fail and how, making error paths visible in code
4. **False** - Results can be passed around, stored, and composed; you handle errors when appropriate in your pipeline

</details>

---

### Multiple Choice

1. What's the main benefit of Result over exceptions?

- A) Faster performance
- B) Less code
- C) Explicit error handling in type signatures
- D) Automatic error recovery

2. When should you use Result pattern?

- A) For all errors including programming bugs
- B) For expected, recoverable errors
- C) Never, exceptions are always better
- D) Only for async operations

<details>
<summary><strong>View Answers</strong></summary>

1. **C** - Explicit error handling in type signatures. `function login(): Result<User, AuthError>` clearly shows it can fail with AuthError

2. **B** - For expected, recoverable errors. Use Result for validation, network issues, business logic failures. Use exceptions/panics for programming bugs

</details>

---

### Code Challenge

Build an order processing system with error handling:

```ts
// TODO: Implement using Result pattern and Railway-oriented programming
// Steps:
// 1. Validate order (items exist, quantities positive)
// 2. Check inventory
// 3. Calculate total with discounts
// 4. Process payment
// 5. Update inventory
// 6. Send confirmation email

// Should handle errors at each step gracefully
```

<details>
<summary><strong>View Answers</strong></summary>

```ts
// Types
interface OrderItem {
  productId: string;
  quantity: number;
  price: number;
}

interface Order {
  orderId: string;
  userId: string;
  items: OrderItem[];
}

interface ValidatedOrder extends Order {
  validated: true;
}

interface InventoryCheckedOrder extends ValidatedOrder {
  inventoryReserved: true;
}

interface PricedOrder extends InventoryCheckedOrder {
  totalAmount: number;
  discountApplied: number;
}

interface ProcessedOrder extends PricedOrder {
  paymentId: string;
  status: 'completed';
  completedAt: Date;
}

type OrderError =
  | { type: 'validation'; field: string; message: string }
  | { type: 'inventory'; productId: string; available: number; requested: number }
  | { type: 'payment'; reason: string }
  | { type: 'database'; operation: string; message: string }
  | { type: 'notification'; message: string };

// Mock data
const inventory: Record<string, number> = {
  'P1': 100,
  'P2': 50,
  'P3': 25,
  'P4': 0
};

const productPrices: Record<string, number> = {
  'P1': 29.99,
  'P2': 49.99,
  'P3': 99.99,
  'P4': 19.99
};

// Step 1: Validate order
function validateOrder(order: Order): Result<ValidatedOrder, OrderError> {
  if (!order.orderId) {
    return Err({ type: 'validation', field: 'orderId', message: 'Order ID is required' });
  }
  
  if (!order.userId) {
    return Err({ type: 'validation', field: 'userId', message: 'User ID is required' });
  }
  
  if (!order.items || order.items.length === 0) {
    return Err({ type: 'validation', field: 'items', message: 'Order must have at least one item' });
  }
  
  // Validate each item
  for (const item of order.items) {
    if (!productPrices[item.productId]) {
      return Err({ 
        type: 'validation', 
        field: 'productId', 
        message: `Product ${item.productId} does not exist` 
      });
    }
    
    if (item.quantity <= 0) {
      return Err({ 
        type: 'validation', 
        field: 'quantity', 
        message: 'Quantity must be positive' 
      });
    }
  }
  
  return Ok({ ...order, validated: true });
}

// Step 2: Check inventory
function checkInventory(order: ValidatedOrder): Result<InventoryCheckedOrder, OrderError> {
  for (const item of order.items) {
    const available = inventory[item.productId] || 0;
    
    if (available < item.quantity) {
      return Err({
        type: 'inventory',
        productId: item.productId,
        available,
        requested: item.quantity
      });
    }
  }
  
  return Ok({ ...order, inventoryReserved: true });
}

// Step 3: Calculate total
function calculateTotal(order: InventoryCheckedOrder): Result<PricedOrder, OrderError> {
  let subtotal = 0;
  
  for (const item of order.items) {
    const price = productPrices[item.productId];
    subtotal += price * item.quantity;
  }
  
  // Apply discount (10% if over $100)
  const discount = subtotal > 100 ? subtotal * 0.1 : 0;
  const totalAmount = subtotal - discount;
  
  return Ok({
    ...order,
    totalAmount,
    discountApplied: discount
  });
}

// Step 4: Process payment
function processPayment(order: PricedOrder): Result<PricedOrder, OrderError> {
  // Simulate payment processing
  if (Math.random() < 0.1) {
    return Err({
      type: 'payment',
      reason: 'Payment gateway timeout'
    });
  }
  
  console.log(`💳 Processing payment: $${order.totalAmount.toFixed(2)}`);
  return Ok(order);
}

// Step 5: Update inventory
function updateInventory(order: PricedOrder): Result<PricedOrder, OrderError> {
  for (const item of order.items) {
    inventory[item.productId] -= item.quantity;
  }
  
  console.log('📦 Inventory updated');
  return Ok(order);
}

// Step 6: Send confirmation
function sendConfirmation(order: PricedOrder): Result<ProcessedOrder, OrderError> {
  // Simulate email sending
  if (Math.random() < 0.05) {
    return Err({
      type: 'notification',
      message: 'Failed to send confirmation email'
    });
  }
  
  console.log(`📧 Confirmation email sent to user ${order.userId}`);
  
  return Ok({
    ...order,
    paymentId: `PAY${Math.random().toString(36).substr(2, 9)}`,
    status: 'completed',
    completedAt: new Date()
  });
}

// Complete pipeline
function processOrder(order: Order): Result<ProcessedOrder, OrderError> {
  return Pipeline.of<Order, OrderError>(order)
    .tap(o => console.log(`\n🛒 Processing order ${o.orderId}`))
    .then(validateOrder)
    .tap(() => console.log('✓ Order validated'))
    .then(checkInventory)
    .tap(() => console.log('✓ Inventory checked'))
    .then(calculateTotal)
    .tap(o => console.log(`✓ Total calculated: $${o.totalAmount.toFixed(2)} (discount: $${o.discountApplied.toFixed(2)})`))
    .then(processPayment)
    .tap(() => console.log('✓ Payment processed'))
    .then(updateInventory)
    .tap(() => console.log('✓ Inventory updated'))
    .then(sendConfirmation)
    .tapError(error => {
      console.error(`\n✗ Order failed: ${error.type}`);
      // Rollback inventory if payment or later steps failed
      if (error.type === 'payment' || error.type === 'database' || error.type === 'notification') {
        console.log('🔄 Rolling back inventory...');
        // Restore inventory (simplified)
      }
    })
    .recover(error => {
      // Only recover from notification failures
      if (error.type === 'notification') {
        console.warn('⚠️  Order completed but notification failed');
        return {
          ...order as any,
          paymentId: 'UNKNOWN',
          status: 'completed',
          completedAt: new Date()
        };
      }
      throw error;
    })
    .run();
}

// Test cases
console.log('=== Order Processing System ===');

// Successful order
const validOrder: Order = {
  orderId: 'ORD001',
  userId: 'U123',
  items: [
    { productId: 'P1', quantity: 2, price: 29.99 },
    { productId: 'P2', quantity: 1, price: 49.99 }
  ]
};

const result1 = processOrder(validOrder);
if (result1.ok) {
  console.log('\n✅ Order completed successfully!');
  console.log(`   Payment ID: ${result1.value.paymentId}`);
  console.log(`   Total: $${result1.value.totalAmount.toFixed(2)}`);
  console.log(`   Status: ${result1.value.status}`);
}

// Out of stock order
const outOfStockOrder: Order = {
  orderId: 'ORD002',
  userId: 'U124',
  items: [
    { productId: 'P4', quantity: 5, price: 19.99 } // P4 has 0 stock
  ]
};

const result2 = processOrder(outOfStockOrder);
if (!result2.ok && result2.error.type === 'inventory') {
  console.log('\n❌ Order failed: Out of stock');
  console.log(`   Product: ${result2.error.productId}`);
  console.log(`   Available: ${result2.error.available}`);
  console.log(`   Requested: ${result2.error.requested}`);
}

// Invalid order
const invalidOrder: Order = {
  orderId: 'ORD003',
  userId: 'U125',
  items: [
    { productId: 'P99', quantity: 1, price: 99.99 } // Non-existent product
  ]
};

const result3 = processOrder(invalidOrder);
if (!result3.ok && result3.error.type === 'validation') {
  console.log('\n❌ Order validation failed');
  console.log(`   Field: ${result3.error.field}`);
  console.log(`   Message: ${result3.error.message}`);
}

console.log('\n=== Final Inventory ===');
console.log(inventory);
```

**Explanation**:
- Each step returns Result, making errors explicit
- Pipeline chains operations on "success track"
- Errors automatically propagate down "error track"
- tapError allows side effects (logging, rollback) without changing result
- recover enables conditional error handling (notification failure non-fatal)
- Type system ensures all intermediate states are properly validated

</details>

</details>

---

## State Management

**The Challenge**: FP emphasizes immutability, but applications need to manage state (user sessions, shopping carts, game state, etc.).

**Solution**: Instead of mutating state, create new versions with changes applied.

---

### Immutable State Updates

**Real-World Scenario: Shopping Cart Management**

<details>
<summary><strong>View Examples</strong></summary>

```ts
// State types
interface CartItem {
  productId: string;
  name: string;
  price: number;
  quantity: number;
}

interface ShoppingCart {
  userId: string;
  items: CartItem[];
  discountCode?: string;
  discountPercent: number;
  subtotal: number;
  total: number;
}

// Helper to recalculate totals
function recalculateTotals(cart: Omit<ShoppingCart, 'subtotal' | 'total'>): ShoppingCart {
  const subtotal = cart.items.reduce(
    (sum, item) => sum + (item.price * item.quantity),
    0
  );
  
  const total = subtotal * (1 - cart.discountPercent / 100);
  
  return {
    ...cart,
    subtotal,
    total
  };
}

// ❌ MUTABLE approach (avoid this!)
function addItemMutable(cart: ShoppingCart, item: CartItem): void {
  cart.items.push(item); // Mutation!
  cart.subtotal += item.price * item.quantity; // Mutation!
  cart.total = cart.subtotal * (1 - cart.discountPercent / 100); // Mutation!
}

// ✅ IMMUTABLE approach
function addItem(cart: ShoppingCart, newItem: CartItem): ShoppingCart {
  const existingItemIndex = cart.items.findIndex(
    item => item.productId === newItem.productId
  );
  
  let updatedItems: CartItem[];
  
  if (existingItemIndex >= 0) {
    // Update quantity of existing item
    updatedItems = cart.items.map((item, index) =>
      index === existingItemIndex
        ? { ...item, quantity: item.quantity + newItem.quantity }
        : item
    );
  } else {
    // Add new item
    updatedItems = [...cart.items, newItem];
  }
  
  return recalculateTotals({
    ...cart,
    items: updatedItems
  });
}

function removeItem(cart: ShoppingCart, productId: string): ShoppingCart {
  return recalculateTotals({
    ...cart,
    items: cart.items.filter(item => item.productId !== productId)
  });
}

function updateQuantity(
  cart: ShoppingCart,
  productId: string,
  quantity: number
): ShoppingCart {
  if (quantity <= 0) {
    return removeItem(cart, productId);
  }
  
  return recalculateTotals({
    ...cart,
    items: cart.items.map(item =>
      item.productId === productId
        ? { ...item, quantity }
        : item
    )
  });
}

function applyDiscount(cart: ShoppingCart, code: string): ShoppingCart {
  // Simulate discount lookup
  const discounts: Record<string, number> = {
    'SAVE10': 10,
    'SAVE20': 20,
    'WELCOME': 15
  };
  
  const discountPercent = discounts[code] || 0;
  
  return recalculateTotals({
    ...cart,
    discountCode: code,
    discountPercent
  });
}

function clearCart(cart: ShoppingCart): ShoppingCart {
  return recalculateTotals({
    ...cart,
    items: [],
    discountCode: undefined,
    discountPercent: 0
  });
}

// Usage
console.log('=== Immutable Shopping Cart ===\n');

const emptyCart: ShoppingCart = {
  userId: 'U123',
  items: [],
  discountPercent: 0,
  subtotal: 0,
  total: 0
};

// Add items
const cart1 = addItem(emptyCart, {
  productId: 'P1',
  name: 'Laptop',
  price: 999,
  quantity: 1
});

console.log('Cart after adding laptop:');
console.log(`  Items: ${cart1.items.length}`);
console.log(`  Total: $${cart1.total.toFixed(2)}`);

const cart2 = addItem(cart1, {
  productId: 'P2',
  name: 'Mouse',
  price: 25,
  quantity: 2
});

console.log('\nCart after adding mouse:');
console.log(`  Items: ${cart2.items.length}`);
console.log(`  Total: $${cart2.total.toFixed(2)}`);

// Apply discount
const cart3 = applyDiscount(cart2, 'SAVE10');

console.log('\nCart after 10% discount:');
console.log(`  Subtotal: $${cart3.subtotal.toFixed(2)}`);
console.log(`  Discount: ${cart3.discountPercent}%`);
console.log(`  Total: $${cart3.total.toFixed(2)}`);

// Update quantity
const cart4 = updateQuantity(cart3, 'P1', 2);

console.log('\nCart after increasing laptop quantity to 2:');
console.log(`  Total: $${cart4.total.toFixed(2)}`);

// Original cart unchanged!
console.log('\nOriginal empty cart still empty:');
console.log(`  Items: ${emptyCart.items.length}`);
console.log(`  Total: $${emptyCart.total.toFixed(2)}`);

// Time travel - can go back to any previous state
console.log('\nTime travel to cart after first item:');
console.log(`  Total: $${cart1.total.toFixed(2)}`);

// History tracking
const cartHistory: ShoppingCart[] = [emptyCart, cart1, cart2, cart3, cart4];

console.log('\n=== Cart History ===');
cartHistory.forEach((cart, index) => {
  console.log(`Step ${index}: ${cart.items.length} items, $${cart.total.toFixed(2)}`);
});

// Undo/Redo functionality
class CartWithHistory {
  private history: ShoppingCart[] = [];
  private currentIndex = -1;
  
  constructor(initialCart: ShoppingCart) {
    this.history.push(initialCart);
    this.currentIndex = 0;
  }
  
  get current(): ShoppingCart {
    return this.history[this.currentIndex];
  }
  
  update(updateFn: (cart: ShoppingCart) => ShoppingCart): void {
    const newCart = updateFn(this.current);
    
    // Remove any future history when making a new change
    this.history = this.history.slice(0, this.currentIndex + 1);
    this.history.push(newCart);
    this.currentIndex++;
  }
  
  undo(): boolean {
    if (this.currentIndex > 0) {
      this.currentIndex--;
      return true;
    }
    return false;
  }
  
  redo(): boolean {
    if (this.currentIndex < this.history.length - 1) {
      this.currentIndex++;
      return true;
    }
    return false;
  }
  
  canUndo(): boolean {
    return this.currentIndex > 0;
  }
  
  canRedo(): boolean {
    return this.currentIndex < this.history.length - 1;
  }
}

// Usage with history
console.log('\n=== Undo/Redo ===');

const cartWithHistory = new CartWithHistory(emptyCart);

cartWithHistory.update(cart => addItem(cart, {
  productId: 'P1',
  name: 'Laptop',
  price: 999,
  quantity: 1
}));
console.log(`After add: $${cartWithHistory.current.total.toFixed(2)}`);

cartWithHistory.update(cart => addItem(cart, {
  productId: 'P2',
  name: 'Mouse',
  price: 25,
  quantity: 2
}));
console.log(`After add: $${cartWithHistory.current.total.toFixed(2)}`);

cartWithHistory.update(cart => applyDiscount(cart, 'SAVE20'));
console.log(`After discount: $${cartWithHistory.current.total.toFixed(2)}`);

console.log('\nUndo:');
cartWithHistory.undo();
console.log(`After undo: $${cartWithHistory.current.total.toFixed(2)}`);

cartWithHistory.undo();
console.log(`After undo: $${cartWithHistory.current.total.toFixed(2)}`);

console.log('\nRedo:');
cartWithHistory.redo();
console.log(`After redo: $${cartWithHistory.current.total.toFixed(2)}`);
```

**Go Example:**

```go
package main

import "fmt"

type CartItem struct {
    ProductID string
    Name      string
    Price     float64
    Quantity  int
}

type ShoppingCart struct {
    UserID          string
    Items           []CartItem
    DiscountCode    string
    DiscountPercent float64
    Subtotal        float64
    Total           float64
}

func recalculateTotals(cart ShoppingCart) ShoppingCart {
    subtotal := 0.0
    for _, item := range cart.Items {
        subtotal += item.Price * float64(item.Quantity)
    }
    
    total := subtotal * (1 - cart.DiscountPercent/100)
    
    cart.Subtotal = subtotal
    cart.Total = total
    return cart
}

// Immutable add item
func addItem(cart ShoppingCart, newItem CartItem) ShoppingCart {
    // Find existing item
    existingIndex := -1
    for i, item := range cart.Items {
        if item.ProductID == newItem.ProductID {
            existingIndex = i
            break
        }
    }
    
    var updatedItems []CartItem
    
    if existingIndex >= 0 {
        // Update existing item
        updatedItems = make([]CartItem, len(cart.Items))
        copy(updatedItems, cart.Items)
        updatedItems[existingIndex].Quantity += newItem.Quantity
    } else {
        // Add new item
        updatedItems = append([]CartItem{}, cart.Items...)
        updatedItems = append(updatedItems, newItem)
    }
    
    cart.Items = updatedItems
    return recalculateTotals(cart)
}

func removeItem(cart ShoppingCart, productID string) ShoppingCart {
    updatedItems := []CartItem{}
    for _, item := range cart.Items {
        if item.ProductID != productID {
            updatedItems = append(updatedItems, item)
        }
    }
    
    cart.Items = updatedItems
    return recalculateTotals(cart)
}

func applyDiscount(cart ShoppingCart, code string) ShoppingCart {
    discounts := map[string]float64{
        "SAVE10":  10,
        "SAVE20":  20,
        "WELCOME": 15,
    }
    
    discountPercent := discounts[code]
    cart.DiscountCode = code
    cart.DiscountPercent = discountPercent
    
    return recalculateTotals(cart)
}

func main() {
    fmt.Println("=== Immutable Shopping Cart ===\n")
    
    emptyCart := ShoppingCart{
        UserID: "U123",
        Items:  []CartItem{},
    }
    
    // Add items
    cart1 := addItem(emptyCart, CartItem{
        ProductID: "P1",
        Name:      "Laptop",
        Price:     999,
        Quantity:  1,
    })
    
    fmt.Printf("Cart after adding laptop:\n")
    fmt.Printf("  Items: %d\n", len(cart1.Items))
    fmt.Printf("  Total: $%.2f\n", cart1.Total)
    
    cart2 := addItem(cart1, CartItem{
        ProductID: "P2",
        Name:      "Mouse",
        Price:     25,
        Quantity:  2,
    })
    
    fmt.Printf("\nCart after adding mouse:\n")
    fmt.Printf("  Items: %d\n", len(cart2.Items))
    fmt.Printf("  Total: $%.2f\n", cart2.Total)
    
    // Apply discount
    cart3 := applyDiscount(cart2, "SAVE10")
    
    fmt.Printf("\nCart after 10%% discount:\n")
    fmt.Printf("  Subtotal: $%.2f\n", cart3.Subtotal)
    fmt.Printf("  Discount: %.0f%%\n", cart3.DiscountPercent)
    fmt.Printf("  Total: $%.2f\n", cart3.Total)
    
    // Original cart unchanged
    fmt.Printf("\nOriginal empty cart still empty:\n")
    fmt.Printf("  Items: %d\n", len(emptyCart.Items))
    fmt.Printf("  Total: $%.2f\n", emptyCart.Total)
}
```

</details>

---

### State Machines

**Concept**: Model state transitions explicitly - what states exist and which transitions are valid.

**Real-World Scenario: Order State Machine**

<details>
<summary><strong>View Examples</strong></summary>

```ts
// Order states
type PendingState = {
  status: 'pending';
  orderId: string;
  items: CartItem[];
  createdAt: Date;
};

type ProcessingState = {
  status: 'processing';
  orderId: string;
  items: CartItem[];
  createdAt: Date;
  paymentId: string;
  startedProcessingAt: Date;
};

type ShippedState = {
  status: 'shipped';
  orderId: string;
  items: CartItem[];
  createdAt: Date;
  paymentId: string;
  startedProcessingAt: Date;
  trackingNumber: string;
  shippedAt: Date;
};

type DeliveredState = {
  status: 'delivered';
  orderId: string;
  items: CartItem[];
  createdAt: Date;
  paymentId: string;
  startedProcessingAt: Date;
  trackingNumber: string;
  shippedAt: Date;
  deliveredAt: Date;
  signature: string;
};

type CancelledState = {
  status: 'cancelled';
  orderId: string;
  items: CartItem[];
  createdAt: Date;
  cancelledAt: Date;
  reason: string;
};

type OrderState =
  | PendingState
  | ProcessingState
  | ShippedState
  | DeliveredState
  | CancelledState;

// State transition functions
function processOrder(state: PendingState, paymentId: string): ProcessingState {
  return {
    status: 'processing',
    orderId: state.orderId,
    items: state.items,
    createdAt: state.createdAt,
    paymentId,
    startedProcessingAt: new Date()
  };
}

function shipOrder(state: ProcessingState, trackingNumber: string): ShippedState {
  return {
    status: 'shipped',
    orderId: state.orderId,
    items: state.items,
    createdAt: state.createdAt,
    paymentId: state.paymentId,
    startedProcessingAt: state.startedProcessingAt,
    trackingNumber,
    shippedAt: new Date()
  };
}

function deliverOrder(state: ShippedState, signature: string): DeliveredState {
  return {
    status: 'delivered',
    orderId: state.orderId,
    items: state.items,
    createdAt: state.createdAt,
    paymentId: state.paymentId,
    startedProcessingAt: state.startedProcessingAt,
    trackingNumber: state.trackingNumber,
    shippedAt: state.shippedAt,
    deliveredAt: new Date(),
    signature
  };
}

function cancelOrder(
  state: PendingState | ProcessingState,
  reason: string
): CancelledState {
  return {
    status: 'cancelled',
    orderId: state.orderId,
    items: state.items,
    createdAt: state.createdAt,
    cancelledAt: new Date(),
    reason
  };
}

// Type-safe transition manager
type TransitionResult<T extends OrderState> =
  | { ok: true; state: T }
  | { ok: false; error: string };

function transition<T extends OrderState>(
  currentState: OrderState,
  action: (state: any) => T
): TransitionResult<T> {
  try {
    const newState = action(currentState);
    return { ok: true, state: newState };
  } catch (error) {
    return { ok: false, error: (error as Error).message };
  }
}

// Order manager with state machine
class OrderManager {
  private state: OrderState;
  
  constructor(state: OrderState) {
    this.state = state;
  }
  
  getState(): OrderState {
    return this.state;
  }
  
  process(paymentId: string): Result<ProcessingState, string> {
    if (this.state.status !== 'pending') {
      return Err(`Cannot process order in ${this.state.status} state`);
    }
    
    const newState = processOrder(this.state, paymentId);
    this.state = newState;
    return Ok(newState);
  }
  
  ship(trackingNumber: string): Result<ShippedState, string> {
    if (this.state.status !== 'processing') {
      return Err(`Cannot ship order in ${this.state.status} state`);
    }
    
    const newState = shipOrder(this.state, trackingNumber);
    this.state = newState;
    return Ok(newState);
  }
  
  deliver(signature: string): Result<DeliveredState, string> {
    if (this.state.status !== 'shipped') {
      return Err(`Cannot deliver order in ${this.state.status} state`);
    }
    
    const newState = deliverOrder(this.state, signature);
    this.state = newState;
    return Ok(newState);
  }
  
  cancel(reason: string): Result<CancelledState, string> {
    if (this.state.status !== 'pending' && this.state.status !== 'processing') {
      return Err(`Cannot cancel order in ${this.state.status} state`);
    }
    
    const newState = cancelOrder(this.state as PendingState | ProcessingState, reason);
    this.state = newState;
    return Ok(newState);
  }
  
  getStatusMessage(): string {
    switch (this.state.status) {
      case 'pending':
        return `Order ${this.state.orderId} is pending (created ${this.state.createdAt.toLocaleDateString()})`;
      
      case 'processing':
        return `Order ${this.state.orderId} is being processed (payment: ${this.state.paymentId})`;
      
      case 'shipped':
        return `Order ${this.state.orderId} has been shipped (tracking: ${this.state.trackingNumber})`;
      
      case 'delivered':
        return `Order ${this.state.orderId} was delivered on ${this.state.deliveredAt.toLocaleDateString()}`;
      
      case 'cancelled':
        return `Order ${this.state.orderId} was cancelled: ${this.state.reason}`;
    }
  }
}

// Test the state machine
console.log('\n=== Order State Machine ===\n');

const initialState: PendingState = {
  status: 'pending',
  orderId: 'ORD-123',
  items: [
    { productId: 'P1', name: 'Laptop', price: 999, quantity: 1 }
  ],
  createdAt: new Date()
};

const order = new OrderManager(initialState);

console.log('Initial state:');
console.log(order.getStatusMessage());

// Valid transition: pending -> processing
const processResult = order.process('PAY-456');
if (processResult.ok) {
  console.log('\n✓ Order processed');
  console.log(order.getStatusMessage());
}

// Valid transition: processing -> shipped
const shipResult = order.ship('TRACK-789');
if (shipResult.ok) {
  console.log('\n✓ Order shipped');
  console.log(order.getStatusMessage());
}

// Invalid transition: shipped -> processing
const invalidProcess = order.process('PAY-999');
if (!invalidProcess.ok) {
  console.log('\n✗ Invalid transition:');
  console.log(`  ${invalidProcess.error}`);
}

// Valid transition: shipped -> delivered
const deliverResult = order.deliver('John Doe');
if (deliverResult.ok) {
  console.log('\n✓ Order delivered');
  console.log(order.getStatusMessage());
}

// Invalid transition: delivered -> cancelled
const invalidCancel = order.cancel('Changed mind');
if (!invalidCancel.ok) {
  console.log('\n✗ Invalid transition:');
  console.log(`  ${invalidCancel.error}`);
}

// Test cancellation path
console.log('\n=== Testing Cancellation Path ===\n');

const cancelOrder1 = new OrderManager({
  status: 'pending',
  orderId: 'ORD-456',
  items: [{ productId: 'P2', name: 'Mouse', price: 25, quantity: 2 }],
  createdAt: new Date()
});

const cancelResult = cancelOrder1.cancel('Customer requested cancellation');
if (cancelResult.ok) {
  console.log('✓ Order cancelled');
  console.log(cancelOrder1.getStatusMessage());
}

// Visualize state transitions
console.log('\n=== Valid State Transitions ===');
console.log(`
pending ──────→ processing ──────→ shipped ──────→ delivered
   │                 │
   │                 │
   └────→ cancelled ←┘

Rules:
- pending → processing (when payment processed)
- pending → cancelled (before processing)
- processing → shipped (when shipped)
- processing → cancelled (before shipping)
- shipped → delivered (when delivered)
- No transitions allowed from delivered or cancelled states
`);
```

</details>

---

### Reducers Pattern (Redux-style)

**Concept**: Centralized state updates through pure reducer functions.

<details>
<summary><strong>View Examples</strong></summary>

```ts
// Application state
interface TodoItem {
  id: string;
  text: string;
  completed: boolean;
  createdAt: Date;
}

interface TodoState {
  todos: TodoItem[];
  filter: 'all' | 'active' | 'completed';
}

// Actions (what happened)
type TodoAction =
  | { type: 'ADD_TODO'; text: string }
  | { type: 'TOGGLE_TODO'; id: string }
  | { type: 'DELETE_TODO'; id: string }
  | { type: 'SET_FILTER'; filter: 'all' | 'active' | 'completed' }
  | { type: 'CLEAR_COMPLETED' }
  | { type: 'TOGGLE_ALL' };

// Reducer (how state changes)
function todoReducer(state: TodoState, action: TodoAction): TodoState {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        ...state,
        todos: [
          ...state.todos,
          {
            id: `todo-${Date.now()}`,
            text: action.text,
            completed: false,
            createdAt: new Date()
          }
        ]
      };
    
    case 'TOGGLE_TODO':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.id
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
    
    case 'DELETE_TODO':
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.id)
      };
    
    case 'SET_FILTER':
      return {
        ...state,
        filter: action.filter
      };
    
    case 'CLEAR_COMPLETED':
      return {
        ...state,
        todos: state.todos.filter(todo => !todo.completed)
      };
    
    case 'TOGGLE_ALL':
      const allCompleted = state.todos.every(todo => todo.completed);
      return {
        ...state,
        todos: state.todos.map(todo => ({
          ...todo,
          completed: !allCompleted
        }))
      };
    
    default:
      const _exhaustive: never = action;
      return state;
  }
}

// Selectors (derived state)
function getFilteredTodos(state: TodoState): TodoItem[] {
  switch (state.filter) {
    case 'all':
      return state.todos;
    case 'active':
      return state.todos.filter(todo => !todo.completed);
    case 'completed':
      return state.todos.filter(todo => todo.completed);
  }
}

function getStats(state: TodoState) {
  const total = state.todos.length;
  const completed = state.todos.filter(t => t.completed).length;
  const active = total - completed;
  
  return { total, completed, active };
}

// Store
class Store<S, A> {
  private state: S;
  private listeners: Array<(state: S) => void> = [];
  
  constructor(
    private reducer: (state: S, action: A) => S,
    initialState: S
  ) {
    this.state = initialState;
  }
  
  getState(): S {
    return this.state;
  }
  
  dispatch(action: A): void {
    this.state = this.reducer(this.state, action);
    this.listeners.forEach(listener => listener(this.state));
  }
  
  subscribe(listener: (state: S) => void): () => void {
    this.listeners.push(listener);
    
    // Return unsubscribe function
    return () => {
      this.listeners = this.listeners.filter(l => l !== listener);
    };
  }
}

// Usage
console.log('\n=== Todo App with Reducer Pattern ===\n');

const initialState: TodoState = {
  todos: [],
  filter: 'all'
};

const store = new Store(todoReducer, initialState);

// Subscribe to state changes
store.subscribe(state => {
  console.log('\n--- State Updated ---');
  const stats = getStats(state);
  console.log(`Total: ${stats.total}, Active: ${stats.active}, Completed: ${stats.completed}`);
  console.log(`Filter: ${state.filter}`);
  
  const filtered = getFilteredTodos(state);
  console.log(`Showing ${filtered.length} todos:`);
  filtered.forEach(todo => {
    const status = todo.completed ? '✓' : ' ';
    console.log(`  [${status}] ${todo.text}`);
  });
});

// Dispatch actions
store.dispatch({ type: 'ADD_TODO', text: 'Learn FP' });
store.dispatch({ type: 'ADD_TODO', text: 'Build a project' });
store.dispatch({ type: 'ADD_TODO', text: 'Write tests' });

const firstTodoId = store.getState().todos[0].id;
store.dispatch({ type: 'TOGGLE_TODO', id: firstTodoId });

store.dispatch({ type: 'SET_FILTER', filter: 'active' });
store.dispatch({ type: 'SET_FILTER', filter: 'all' });

store.dispatch({ type: 'ADD_TODO', text: 'Deploy to production' });
store.dispatch({ type: 'TOGGLE_ALL' });
store.dispatch({ type: 'CLEAR_COMPLETED' });
```

</details>

---

## Practice Questions

<details>
<summary><strong>View Questions</strong></summary>

### Fill in the Blanks

1. Immutable state updates create __________ versions instead of modifying existing ones.
2. State machines make valid state __________ explicit in the type system.
3. Reducers are __________ functions that take current state and action, returning new state.
4. The spread operator `{...obj}` creates a __________ copy of an object.

<details>
<summary><strong>View Answers</strong></summary>

1. **new** - Instead of `cart.items.push(item)`, we do `items: [...cart.items, item]`
2. **transitions** - Type system prevents invalid transitions like "delivered → pending"
3. **pure** - Reducers have no side effects and always return the same output for same inputs
4. **shallow** - Spread only copies top level; nested objects are still referenced

</details>

---

### True/False

1. ⬜ Immutable updates are always slower than mutable updates
2. ⬜ State machines prevent impossible states at compile time
3. ⬜ Reducers can perform async operations
4. ⬜ Immutability enables features like undo/redo

<details>
<summary><strong>View Answers</strong></summary>

1. **False** - While immutable updates have some overhead, modern JS engines optimize them well, and structural sharing minimizes memory costs. The benefits (predictability, time travel debugging) often outweigh performance costs
2. **True** - With TypeScript's discriminated unions, invalid transitions like `shipOrder(pendingState)` are compile errors
3. **False** - Reducers must be pure and synchronous. Async logic belongs in middleware or action creators
4. **True** - Because each state is preserved, you can keep a history and navigate backwards/forwards through it

</details>

---

### Code Challenge

Build a game state manager with immutable updates:

```ts
// TODO: Create a turn-based game state manager
// - Player health, inventory, position
// - Enemy spawning and combat
// - Item collection
// - Level progression
// - Implement undo/redo
// - Use reducer pattern for state updates
```

<details>
<summary><strong>View Answers</strong></summary>

```ts
// Game types
interface Position {
  x: number;
  y: number;
}

interface Player {
  health: number;
  maxHealth: number;
  position: Position;
  inventory: string[];
  gold: number;
}

interface Enemy {
  id: string;
  health: number;
  position: Position;
  damage: number;
}

interface Item {
  id: string;
  type: 'health_potion' | 'gold' | 'weapon';
  position: Position;
}

interface GameState {
  player: Player;
  enemies: Enemy[];
  items: Item[];
  level: number;
  turnNumber: number;
  gameOver: boolean;
  message: string;
}

// Actions
type GameAction =
  | { type: 'MOVE_PLAYER'; direction: 'north' | 'south' | 'east' | 'west' }
  | { type: 'ATTACK_ENEMY'; enemyId: string }
  | { type: 'COLLECT_ITEM'; itemId: string }
  | { type: 'USE_ITEM'; itemType: string }
  | { type: 'SPAWN_ENEMY'; enemy: Enemy }
  | { type: 'NEXT_LEVEL' }
  | { type: 'END_TURN' };

// Helpers
function distance(pos1: Position, pos2: Position): number {
  return Math.abs(pos1.x - pos2.x) + Math.abs(pos1.y - pos2.y);
}

function movePosition(pos: Position, direction: string): Position {
  switch (direction) {
    case 'north': return { ...pos, y: pos.y - 1 };
    case 'south': return { ...pos, y: pos.y + 1 };
    case 'east': return { ...pos, x: pos.x + 1 };
    case 'west': return { ...pos, x: pos.x - 1 };
    default: return pos;
  }
}

// Reducer
function gameReducer(state: GameState, action: GameAction): GameState {
  if (state.gameOver) {
    return { ...state, message: 'Game Over! Cannot perform actions.' };
  }
  
  switch (action.type) {
    case 'MOVE_PLAYER': {
      const newPosition = movePosition(state.player.position, action.direction);
      
      // Check for items at new position
      const itemAtPosition = state.items.find(item =>
        item.position.x === newPosition.x && item.position.y === newPosition.y
      );
      
      if (itemAtPosition) {
        return gameReducer(state, { type: 'COLLECT_ITEM', itemId: itemAtPosition.id });
      }
      
      return {
        ...state,
        player: {
          ...state.player,
          position: newPosition
        },
        message: `Moved ${action.direction}`
      };
    }
    
    case 'ATTACK_ENEMY': {
      const enemy = state.enemies.find(e => e.id === action.enemyId);
      if (!enemy) {
        return { ...state, message: 'Enemy not found' };
      }
      
      // Check if enemy is in range
      if (distance(state.player.position, enemy.position) > 1) {
        return { ...state, message: 'Enemy too far to attack' };
      }
      
      // Attack enemy
      const damage = 10;
      const updatedEnemy = { ...enemy, health: enemy.health - damage };
      
      let newState: GameState = {
        ...state,
        enemies: updatedEnemy.health > 0
          ? state.enemies.map(e => e.id === enemy.id ? updatedEnemy : e)
          : state.enemies.filter(e => e.id !== enemy.id),
        message: `Attacked enemy for ${damage} damage`
      };
      
      // Enemy counterattack if alive
      if (updatedEnemy.health > 0) {
        newState = {
          ...newState,
          player: {
            ...newState.player,
            health: newState.player.health - enemy.damage
          },
          message: `${newState.message}. Enemy hit you for ${enemy.damage} damage!`
        };
        
        // Check if player died
        if (newState.player.health <= 0) {
          newState = {
            ...newState,
            gameOver: true,
            message: 'You have been defeated!'
          };
        }
      } else {
        newState = {
          ...newState,
          message: `${newState.message}. Enemy defeated!`,
          player: {
            ...newState.player,
            gold: newState.player.gold + 10
          }
        };
      }
      
      return newState;
    }
    
    case 'COLLECT_ITEM': {
      const item = state.items.find(i => i.id === action.itemId);
      if (!item) {
        return { ...state, message: 'Item not found' };
      }
      
      let newPlayer = state.player;
      let message = '';
      
      switch (item.type) {
        case 'health_potion':
          newPlayer = {
            ...state.player,
            inventory: [...state.player.inventory, item.type]
          };
          message = 'Collected health potion';
          break;
        
        case 'gold':
          newPlayer = {
            ...state.player,
            gold: state.player.gold + 5
          };
          message = 'Collected 5 gold';
          break;
        
        case 'weapon':
          newPlayer = {
            ...state.player,
            inventory: [...state.player.inventory, item.type]
          };
          message = 'Collected weapon';
          break;
      }
      
      return {
        ...state,
        player: newPlayer,
        items: state.items.filter(i => i.id !== item.id),
        message
      };
    }
    
    case 'USE_ITEM': {
      if (!state.player.inventory.includes(action.itemType)) {
        return { ...state, message: 'Item not in inventory' };
      }
      
      let newPlayer = state.player;
      let message = '';
      
      if (action.itemType === 'health_potion') {
        const healAmount = 30;
        newPlayer = {
          ...state.player,
          health: Math.min(state.player.health + healAmount, state.player.maxHealth),
          inventory: state.player.inventory.filter((item, index) =>
            index !== state.player.inventory.indexOf('health_potion')
          )
        };
        message = `Used health potion, healed ${healAmount} HP`;
      }
      
      return {
        ...state,
        player: newPlayer,
        message
      };
    }
    
    case 'SPAWN_ENEMY': {
      return {
        ...state,
        enemies: [...state.enemies, action.enemy],
        message: 'An enemy appeared!'
      };
    }
    
    case 'NEXT_LEVEL': {
      if (state.enemies.length > 0) {
        return { ...state, message: 'Defeat all enemies first!' };
      }
      
      return {
        ...state,
        level: state.level + 1,
        enemies: [],
        items: [],
        player: {
          ...state.player,
          position: { x: 0, y: 0 },
          health: state.player.maxHealth
        },
        message: `Advanced to level ${state.level + 1}!`
      };
    }
    
    case 'END_TURN': {
      return {
        ...state,
        turnNumber: state.turnNumber + 1,
        message: `Turn ${state.turnNumber + 1}`
      };
    }
    
    default:
      const _exhaustive: never = action;
      return state;
  }
}

// Game manager with undo/redo
class GameManager {
  private history: GameState[] = [];
  private currentIndex = -1;
  
  constructor(initialState: GameState) {
    this.history.push(initialState);
    this.currentIndex = 0;
  }
  
  get state(): GameState {
    return this.history[this.currentIndex];
  }
  
  dispatch(action: GameAction): void {
    const newState = gameReducer(this.state, action);
    
    // Remove future history
    this.history = this.history.slice(0, this.currentIndex + 1);
    this.history.push(newState);
    this.currentIndex++;
  }
  
  undo(): boolean {
    if (this.currentIndex > 0) {
      this.currentIndex--;
      return true;
    }
    return false;
  }
  
  redo(): boolean {
    if (this.currentIndex < this.history.length - 1) {
      this.currentIndex++;
      return true;
    }
    return false;
  }
  
  printState(): void {
    const s = this.state;
    console.log(`\n=== Turn ${s.turnNumber}, Level ${s.level} ===`);
    console.log(`Player: HP ${s.health}/${s.maxHealth}, Gold: ${s.gold}, Pos: (${s.player.position.x},${s.player.position.y})`);
    console.log(`Inventory: ${s.player.inventory.join(', ') || 'empty'}`);
    console.log(`Enemies: ${s.enemies.length}`);
    s.enemies.forEach(e => {
      console.log(`  - Enemy ${e.id} at (${e.position.x},${e.position.y}), HP: ${e.health}`);
    });
    console.log(`Items: ${s.items.length}`);
    console.log(`Message: ${s.message}`);
  }
}

// Initialize game
console.log('\n=== RPG Game with Immutable State ===');

const initialGameState: GameState = {
  player: {
    health: 100,
    maxHealth: 100,
    position: { x: 0, y: 0 },
    inventory: [],
    gold: 0
  },
  enemies: [],
  items: [
    { id: 'item1', type: 'health_potion', position: { x: 1, y: 0 } },
    { id: 'item2', type: 'gold', position: { x: 2, y: 0 } }
  ],
  level: 1,
  turnNumber: 0,
  gameOver: false,
  message: 'Game started!'
};

const game = new GameManager(initialGameState);
game.printState();

// Play the game
game.dispatch({ type: 'MOVE_PLAYER', direction: 'east' });
game.printState();

game.dispatch({ 
  type: 'SPAWN_ENEMY', 
  enemy: { id: 'enemy1', health: 30, position: { x: 1, y: 1 }, damage: 5 }
});
game.printState();

game.dispatch({ type: 'MOVE_PLAYER', direction: 'east' });
game.printState();

game.dispatch({ type: 'MOVE_PLAYER', direction: 'south' });
game.printState();

game.dispatch({ type: 'ATTACK_ENEMY', enemyId: 'enemy1' });
game.printState();

game.dispatch({ type: 'ATTACK_ENEMY', enemyId: 'enemy1' });
game.printState();

game.dispatch({ type: 'ATTACK_ENEMY', enemyId: 'enemy1' });
game.printState();

// Undo last action
console.log('\n--- UNDO ---');
game.undo();
game.printState();

// Redo
console.log('\n--- REDO ---');
game.redo();
game.printState();
```

**Explanation**:
- All state updates are immutable (spread operators everywhere)
- Reducer is pure - no side effects, same input = same output
- State history enables undo/redo naturally
- Type system prevents invalid actions (can't attack with wrong ID)
- Complex game logic broken into simple, testable reducer cases

</details>

</details>

---

## Async Operations

**The Challenge**: Asynchronous operations (API calls, file I/O, timers) introduce side effects and complexity.

**FP Solutions**:
1. Promises with functional composition
2. Task/Future monads
3. Async/await with Result types
4. Observable streams (reactive programming)

---

### Promises with Functional Patterns

**Real-World Scenario: Multi-Step API Workflow**

<details>
<summary><strong>View Codes/Examples/Script</strong></summary>

```ts
// Types
interface User {
  id: string;
  name: string;
  email: string;
  preferences: {
    notifications: boolean;
    theme: 'light' | 'dark';
  };
}

interface Post {
  id: string;
  userId: string;
  title: string;
  content: string;
  likes: number;
}

interface Comment {
  id: string;
  postId: string;
  userId: string;
  text: string;
}

type ApiError = {
  code: string;
  message: string;
};

// Simulated API calls
function fetchUser(userId: string): Promise<Result<User, ApiError>> {
  return new Promise((resolve) => {
    setTimeout(() => {
      if (userId === 'invalid') {
        resolve(Err({ code: 'NOT_FOUND', message: 'User not found' }));
      } else {
        resolve(Ok({
          id: userId,
          name: `User ${userId}`,
          email: `user${userId}@example.com`,
          preferences: { notifications: true, theme: 'light' }
        }));
      }
    }, 100);
  });
}

function fetchUserPosts(userId: string): Promise<Result<Post[], ApiError>> {
  return new Promise((resolve) => {
    setTimeout(() => {
      if (Math.random() < 0.1) {
        resolve(Err({ code: 'SERVER_ERROR', message: 'Failed to fetch posts' }));
      } else {
        const posts: Post[] = [
          { id: 'P1', userId, title: 'First Post', content: 'Hello world', likes: 10 },
          { id: 'P2', userId, title: 'Second Post', content: 'FP is great', likes: 25 }
        ];
        resolve(Ok(posts));
      }
    }, 150);
  });
}

function fetchPostComments(postId: string): Promise<Result<Comment[], ApiError>> {
  return new Promise((resolve) => {
    setTimeout(() => {
      const comments: Comment[] = [
        { id: 'C1', postId, userId: 'U2', text: 'Great post!' },
        { id: 'C2', postId, userId: 'U3', text: 'Thanks for sharing' }
      ];
      resolve(Ok(comments));
    }, 100);
  });
}

// ❌ Callback hell (avoid this)
function getUserDataCallbackHell(userId: string, callback: (data: any) => void) {
  fetchUser(userId).then(userResult => {
    if (!userResult.ok) return callback({ error: userResult.error });
    
    fetchUserPosts(userId).then(postsResult => {
      if (!postsResult.ok) return callback({ error: postsResult.error });
      
      const firstPost = postsResult.value[0];
      fetchPostComments(firstPost.id).then(commentsResult => {
        if (!commentsResult.ok) return callback({ error: commentsResult.error });
        
        callback({
          user: userResult.value,
          posts: postsResult.value,
          comments: commentsResult.value
        });
      });
    });
  });
}

// ✅ Async/await with Result pattern
async function getUserData(userId: string): Promise<Result<{
  user: User;
  posts: Post[];
  firstPostComments: Comment[];
}, ApiError>> {
  // Fetch user
  const userResult = await fetchUser(userId);
  if (!userResult.ok) return userResult;
  
  // Fetch posts
  const postsResult = await fetchUserPosts(userId);
  if (!postsResult.ok) return postsResult;
  
  // Fetch comments for first post
  if (postsResult.value.length === 0) {
    return Ok({
      user: userResult.value,
      posts: [],
      firstPostComments: []
    });
  }
  
  const commentsResult = await fetchPostComments(postsResult.value[0].id);
  if (!commentsResult.ok) return commentsResult;
  
  return Ok({
    user: userResult.value,
    posts: postsResult.value,
    firstPostComments: commentsResult.value
  });
}

// Usage
console.log('=== Async Operations with Result Pattern ===\n');

(async () => {
  const result = await getUserData('U1');
  
  if (result.ok) {
    console.log('✓ Successfully fetched user data');
    console.log(`  User: ${result.value.user.name}`);
    console.log(`  Posts: ${result.value.posts.length}`);
    console.log(`  Comments on first post: ${result.value.firstPostComments.length}`);
  } else {
    console.log('✗ Failed to fetch user data');
    console.log(`  Error: ${result.error.message}`);
  }
  
  // Test error case
  const errorResult = await getUserData('invalid');
  if (!errorResult.ok) {
    console.log(`\n✗ Expected error: ${errorResult.error.message}`);
  }
})();

// Parallel async operations
async function fetchMultipleUsers(
  userIds: string[]
): Promise<Result<User[], ApiError>> {
  const results = await Promise.all(userIds.map(id => fetchUser(id)));
  
  // Check if any failed
  const failures = results.filter(r => !r.ok);
  if (failures.length > 0) {
    return failures[0] as Result<User[], ApiError>;
  }
  
  // All succeeded
  const users = results.map(r => (r as { ok: true; value: User }).value);
  return Ok(users);
}

// Sequential vs Parallel comparison
async function sequentialFetch(): Promise<number> {
  console.log('\n=== Sequential Fetch ===');
  const start = Date.now();
  
  await fetchUser('U1');
  await fetchUser('U2');
  await fetchUser('U3');
  
  const elapsed = Date.now() - start;
  console.log(`Completed in ${elapsed}ms`);
  return elapsed;
}

async function parallelFetch(): Promise<number> {
  console.log('\n=== Parallel Fetch ===');
  const start = Date.now();
  
  await Promise.all([
    fetchUser('U1'),
    fetchUser('U2'),
    fetchUser('U3')
  ]);
  
  const elapsed = Date.now() - start;
  console.log(`Completed in ${elapsed}ms`);
  return elapsed;
}

(async () => {
  const seqTime = await sequentialFetch();
  const parTime = await parallelFetch();
  console.log(`\n⚡ Parallel was ${Math.round(seqTime / parTime)}x faster`);
})();

// Retry logic with exponential backoff
async function withRetry<T, E>(
  operation: () => Promise<Result<T, E>>,
  maxAttempts: number = 3,
  delayMs: number = 1000
): Promise<Result<T, E>> {
  let lastError: E | undefined;
  
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    console.log(`Attempt ${attempt}/${maxAttempts}...`);
    
    const result = await operation();
    
    if (result.ok) {
      return result;
    }
    
    lastError = result.error;
    
    if (attempt < maxAttempts) {
      const delay = delayMs * Math.pow(2, attempt - 1); // Exponential backoff
      console.log(`  Failed, retrying in ${delay}ms...`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  
  return Err(lastError!);
}

// Usage
(async () => {
  console.log('\n=== Retry with Backoff ===');
  
  const result = await withRetry(
    () => fetchUserPosts('U1'),
    3,
    500
  );
  
  if (result.ok) {
    console.log(`✓ Fetched ${result.value.length} posts`);
  } else {
    console.log(`✗ All attempts failed: ${result.error.message}`);
  }
})();

// Timeout wrapper
async function withTimeout<T, E>(
  operation: Promise<Result<T, E>>,
  timeoutMs: number,
  timeoutError: E
): Promise<Result<T, E>> {
  return Promise.race([
    operation,
    new Promise<Result<T, E>>((resolve) => {
      setTimeout(() => resolve(Err(timeoutError)), timeoutMs);
    })
  ]);
}

// Usage
(async () => {
  console.log('\n=== Timeout Wrapper ===');
  
  const result = await withTimeout(
    fetchUser('U1'),
    50, // Very short timeout
    { code: 'TIMEOUT', message: 'Request timed out' }
  );
  
  if (result.ok) {
    console.log(`✓ Fetched user: ${result.value.name}`);
  } else {
    console.log(`✗ ${result.error.message}`);
  }
})();

// Batch processing with concurrency limit
async function processBatch<T, R, E>(
  items: T[],
  processOne: (item: T) => Promise<Result<R, E>>,
  concurrency: number = 3
): Promise<Result<R[], E>> {
  const results: R[] = [];
  
  for (let i = 0; i < items.length; i += concurrency) {
    const batch = items.slice(i, i + concurrency);
    console.log(`Processing batch ${Math.floor(i / concurrency) + 1}...`);
    
    const batchResults = await Promise.all(batch.map(processOne));
    
    // Check for errors
    const error = batchResults.find(r => !r.ok);
    if (error && !error.ok) {
      return error as Result<R[], E>;
    }
    
    results.push(...batchResults.map(r => (r as { ok: true; value: R }).value));
  }
  
  return Ok(results);
}

// Usage
(async () => {
  console.log('\n=== Batch Processing ===');
  
  const userIds = ['U1', 'U2', 'U3', 'U4', 'U5', 'U6', 'U7'];
  
  const result = await processBatch(
    userIds,
    (id) => fetchUser(id),
    3 // Process 3 at a time
  );
  
  if (result.ok) {
    console.log(`✓ Processed ${result.value.length} users`);
  }
})();

// Cache wrapper for async operations
class AsyncCache<K, V> {
  private cache = new Map<K, { value: V; expiresAt: number }>();
  
  constructor(private ttlMs: number = 60000) {}
  
  async get(
    key: K,
    fetch: () => Promise<V>
  ): Promise<V> {
    const cached = this.cache.get(key);
    
    if (cached && Date.now() < cached.expiresAt) {
      console.log(`Cache hit: ${key}`);
      return cached.value;
    }
    
    console.log(`Cache miss: ${key}, fetching...`);
    const value = await fetch();
    
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
(async () => {
  console.log('\n=== Async Caching ===');
  
  const userCache = new AsyncCache<string, Result<User, ApiError>>(5000);
  
  const user1 = await userCache.get('U1', () => fetchUser('U1'));
  const user1Again = await userCache.get('U1', () => fetchUser('U1'));
  const user2 = await userCache.get('U2', () => fetchUser('U2'));
})();
```

**Go Example:**

```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

type User struct {
    ID    string
    Name  string
    Email string
}

type ApiError struct {
    Code    string
    Message string
}

type Result[T any] struct {
    value T
    err   *ApiError
    ok    bool
}

func Ok[T any](value T) Result[T] {
    return Result[T]{value: value, ok: true}
}

func Err[T any](err ApiError) Result[T] {
    var zero T
    return Result[T]{value: zero, err: &err, ok: false}
}

// Simulated API call
func fetchUser(ctx context.Context, userID string) Result[User] {
    select {
    case <-time.After(100 * time.Millisecond):
        if userID == "invalid" {
            return Err[User](ApiError{Code: "NOT_FOUND", Message: "User not found"})
        }
        return Ok(User{
            ID:    userID,
            Name:  fmt.Sprintf("User %s", userID),
            Email: fmt.Sprintf("user%s@example.com", userID),
        })
    case <-ctx.Done():
        return Err[User](ApiError{Code: "TIMEOUT", Message: "Request timeout"})
    }
}

// Parallel fetch with goroutines
func fetchMultipleUsers(ctx context.Context, userIDs []string) Result[[]User] {
    type result struct {
        user User
        err  *ApiError
    }
    
    resultChan := make(chan result, len(userIDs))
    
    for _, id := range userIDs {
        go func(userID string) {
            r := fetchUser(ctx, userID)
            if r.ok {
                resultChan <- result{user: r.value}
            } else {
                resultChan <- result{err: r.err}
            }
        }(id)
    }
    
    users := make([]User, 0, len(userIDs))
    
    for i := 0; i < len(userIDs); i++ {
        r := <-resultChan
        if r.err != nil {
            return Err[[]User](*r.err)
        }
        users = append(users, r.user)
    }
    
    return Ok(users)
}

// Retry with backoff
func withRetry[T any](
    operation func() Result[T],
    maxAttempts int,
    initialDelay time.Duration,
) Result[T] {
    var lastErr *ApiError
    
    for attempt := 1; attempt <= maxAttempts; attempt++ {
        fmt.Printf("Attempt %d/%d...\n", attempt, maxAttempts)
        
        result := operation()
        if result.ok {
            return result
        }
        
        lastErr = result.err
        
        if attempt < maxAttempts {
            delay := initialDelay * time.Duration(1<<uint(attempt-1))
            fmt.Printf("  Failed, retrying in %v...\n", delay)
            time.Sleep(delay)
        }
    }
    
    return Err[T](*lastErr)
}

// Worker pool for batch processing
func processBatch[T, R any](
    items []T,
    processOne func(T) Result[R],
    concurrency int,
) Result[[]R] {
    jobs := make(chan T, len(items))
    results := make(chan Result[R], len(items))
    
    // Start workers
    var wg sync.WaitGroup
    for i := 0; i < concurrency; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for item := range jobs {
                results <- processOne(item)
            }
        }()
    }
    
    // Send jobs
    for _, item := range items {
        jobs <- item
    }
    close(jobs)
    
    // Wait and close results
    go func() {
        wg.Wait()
        close(results)
    }()
    
    // Collect results
    output := make([]R, 0, len(items))
    for result := range results {
        if !result.ok {
            return Err[[]R](*result.err)
        }
        output = append(output, result.value)
    }
    
    return Ok(output)
}

func main() {
    fmt.Println("=== Async Operations in Go ===\n")
    
    ctx := context.Background()
    
    // Single fetch
    user := fetchUser(ctx, "U1")
    if user.ok {
        fmt.Printf("✓ Fetched user: %s\n", user.value.Name)
    }
    
    // Parallel fetch
    fmt.Println("\n=== Parallel Fetch ===")
    start := time.Now()
    users := fetchMultipleUsers(ctx, []string{"U1", "U2", "U3"})
    elapsed := time.Since(start)
    
    if users.ok {
        fmt.Printf("✓ Fetched %d users in %v\n", len(users.value), elapsed)
    }
    
    // Retry example
    fmt.Println("\n=== Retry with Backoff ===")
    result := withRetry(
        func() Result[User] {
            return fetchUser(ctx, "U1")
        },
        3,
        500*time.Millisecond,
    )
    
    if result.ok {
        fmt.Printf("✓ Fetched user: %s\n", result.value.Name)
    }
    
    // Batch processing
    fmt.Println("\n=== Batch Processing ===")
    userIDs := []string{"U1", "U2", "U3", "U4", "U5"}
    
    batchResult := processBatch(
        userIDs,
        func(id string) Result[User] {
            return fetchUser(ctx, id)
        },
        3,
    )
    
    if batchResult.ok {
        fmt.Printf("✓ Processed %d users\n", len(batchResult.value))
    }
}
```

</details>

---

### Task Monad for Async

**Concept**: Wrap async operations in a lazy Task monad that doesn't execute until explicitly run.

<details>
<summary><strong>View Codes/Examples/Script</strong></summary>

```ts
// Task monad - lazy async computation
class Task<T, E = Error> {
  constructor(
    private computation: () => Promise<Result<T, E>>
  ) {}
  
  // Create from value
  static of<T, E = Error>(value: T): Task<T, E> {
    return new Task(() => Promise.resolve(Ok<T, E>(value)));
  }
  
  // Create from error
  static error<T, E = Error>(error: E): Task<T, E> {
    return new Task(() => Promise.resolve(Err<T, E>(error)));
  }
  
  // Map over success value
  map<U>(fn: (value: T) => U): Task<U, E> {
    return new Task(async () => {
      const result = await this.computation();
      if (!result.ok) return result;
      return Ok<U, E>(fn(result.value));
    });
  }
  
  // Chain tasks (flatMap)
  flatMap<U>(fn: (value: T) => Task<U, E>): Task<U, E> {
    return new Task(async () => {
      const result = await this.computation();
      if (!result.ok) return result;
      return await fn(result.value).run();
    });
  }
  
  // Map over error
  mapError<F>(fn: (error: E) => F): Task<T, F> {
    return new Task(async () => {
      const result = await this.computation();
      if (result.ok) return Ok<T, F>(result.value);
      return Err<T, F>(fn(result.error));
    });
  }
  
  // Recover from error
  recover(fn: (error: E) => T): Task<T, E> {
    return new Task(async () => {
      const result = await this.computation();
      if (result.ok) return result;
      return Ok<T, E>(fn(result.error));
    });
  }
  
  // Execute the task
  run(): Promise<Result<T, E>> {
    return this.computation();
  }
  
  // Execute and unwrap (throws on error)
  async runOrThrow(): Promise<T> {
    const result = await this.computation();
    if (!result.ok) {
      throw result.error;
    }
    return result.value;
  }
}

// Helper to create task from async function
function task<T, E = Error>(
  fn: () => Promise<Result<T, E>>
): Task<T, E> {
  return new Task(fn);
}

// Real-world example: User onboarding flow
interface OnboardingData {
  user: User;
  posts: Post[];
  recommendations: User[];
}

const fetchUserTask = (userId: string): Task<User, ApiError> =>
  task(() => fetchUser(userId));

const fetchUserPostsTask = (userId: string): Task<Post[], ApiError> =>
  task(() => fetchUserPosts(userId));

const fetchRecommendationsTask = (userId: string): Task<User[], ApiError> =>
  task(async () => {
    // Simulate recommendation API
    await new Promise(resolve => setTimeout(resolve, 100));
    return Ok([
      { id: 'U2', name: 'Recommended User 1', email: 'rec1@example.com', preferences: { notifications: true, theme: 'dark' } },
      { id: 'U3', name: 'Recommended User 2', email: 'rec2@example.com', preferences: { notifications: false, theme: 'light' } }
    ]);
  });

// Compose tasks
const onboardNewUser = (userId: string): Task<OnboardingData, ApiError> =>
  fetchUserTask(userId)
    .flatMap(user =>
      fetchUserPostsTask(userId)
        .flatMap(posts =>
          fetchRecommendationsTask(userId)
            .map(recommendations => ({
              user,
              posts,
              recommendations
            }))
        )
    );

// Usage
console.log('\n=== Task Monad Example ===');

(async () => {
  const onboardingTask = onboardNewUser('U1');
  console.log('Task created (not executed yet)');
  
  // ... do other work ...
  
  console.log('Now executing task...');
  const result = await onboardingTask.run();
  
  if (result.ok) {
    console.log('✓ Onboarding complete');
    console.log(`  User: ${result.value.user.name}`);
    console.log(`  Posts: ${result.value.posts.length}`);
    console.log(`  Recommendations: ${result.value.recommendations.length}`);
  } else {
    console.log(`✗ Onboarding failed: ${result.error.message}`);
  }
})();

// Parallel task execution
class ParallelTasks {
  static all<T, E>(tasks: Task<T, E>[]): Task<T[], E> {
    return new Task(async () => {
      const results = await Promise.all(tasks.map(t => t.run()));
      
      // Check for errors
      const error = results.find(r => !r.ok);
      if (error && !error.ok) {
        return error as Result<T[], E>;
      }
      
      const values = results.map(r => (r as { ok: true; value: T }).value);
      return Ok(values);
    });
  }
  
  static race<T, E>(tasks: Task<T, E>[]): Task<T, E> {
    return new Task(() => Promise.race(tasks.map(t => t.run())));
  }
}

// Usage
(async () => {
  console.log('\n=== Parallel Tasks ===');
  
  const tasks = [
    fetchUserTask('U1'),
    fetchUserTask('U2'),
    fetchUserTask('U3')
  ];
  
  const parallelTask = ParallelTasks.all(tasks);
  const result = await parallelTask.run();
  
  if (result.ok) {
    console.log(`✓ Fetched ${result.value.length} users in parallel`);
  }
})();

// Task with timeout
function withTaskTimeout<T, E>(
  task: Task<T, E>,
  timeoutMs: number,
  timeoutError: E
): Task<T, E> {
  const timeoutTask = new Task<T, E>(() =>
    new Promise((resolve) => {
      setTimeout(() => resolve(Err(timeoutError)), timeoutMs);
    })
  );
  
  return ParallelTasks.race([task, timeoutTask]);
}

// Usage
(async () => {
  console.log('\n=== Task with Timeout ===');
  
  const slowTask = fetchUserTask('U1');
  const timedTask = withTaskTimeout(
    slowTask,
    50,
    { code: 'TIMEOUT', message: 'Task timed out' }
  );
  
  const result = await timedTask.run();
  if (!result.ok) {
    console.log(`✗ ${result.error.message}`);
  }
})();
```

</details>

---

## Practice Questions

<details>
<summary><strong>View Questions</strong></summary>

### Fill in the Blanks

1. Combining async/await with Result types makes errors __________ in async code.
2. Promise.all executes promises in __________, while sequential await is __________.
3. The Task monad is __________ - it doesn't execute until run() is called.
4. Retry with exponential backoff increases delay by a factor of __________ on each attempt.

<details>
<summary><strong>View Answers</strong></summary>

1. **explicit** - Function returns `Promise<Result<T, E>>` showing it's async AND can fail
2. **parallel**, **serial** - Promise.all runs concurrently; sequential await waits for each to finish
3. **lazy** - Tasks are descriptions of async work that execute only when explicitly run
4. **2** (or "power of 2") - Common pattern: 1s, 2s, 4s, 8s, etc.

</details>

---

### True/False

1. ⬜ Async/await makes asynchronous code look synchronous
2. ⬜ Promise.all fails fast if any promise rejects
3. ⬜ Tasks execute immediately when created
4. ⬜ Parallel execution is always faster than sequential

<details>
<summary><strong>View Answers</strong></summary>

1. **True** - Async/await allows writing async code in a linear, synchronous-looking style
2. **True** - Promise.all rejects as soon as the first promise rejects (doesn't wait for others)
3. **False** - Tasks are lazy; they only execute when run() is called
4. **False** - Parallel adds overhead; for very fast operations or CPU-bound work, sequential might be faster

</details>

---

### Multiple Choice

1. What's the advantage of Task over Promise?

- A) Tasks are faster
- B) Tasks are lazy (don't execute until run)
- C) Tasks use less memory
- D) Tasks are easier to debug

2. When should you use sequential vs parallel async operations?

- A) Always use parallel
- B) Sequential when operations depend on each other, parallel when independent
- C) Always use sequential
- D) It doesn't matter

<details>
<summary><strong>View Answers</strong></summary>

1. **B** - Tasks are lazy. Promises execute immediately when created; Tasks are descriptions that execute only when run()

2. **B** - Sequential when operations depend on each other, parallel when independent. Example: fetch user THEN fetch their posts (sequential) vs fetch multiple independent users (parallel)

</details>

</details>

---

# Functional Programming: Chapter 4 (Continued) - Testing Pure Functions

## Testing Pure Functions

**Why FP Makes Testing Easier**:
- Pure functions: same input → same output (deterministic)
- No side effects to mock or stub
- No hidden dependencies
- Easy to test in isolation
- Predictable and reproducible

---

### Testing Pure Functions - The Basics

**Real-World Scenario: E-commerce Calculations**

<details>
<summary><strong>View Codes/Examples/Script</strong></summary>

```ts
// Functions to test
function calculateDiscount(price: number, discountPercent: number): number {
  if (price < 0 || discountPercent < 0 || discountPercent > 100) {
    throw new Error('Invalid input');
  }
  return price * (discountPercent / 100);
}

function applyDiscount(price: number, discountPercent: number): number {
  const discount = calculateDiscount(price, discountPercent);
  return price - discount;
}

function calculateTax(price: number, taxRate: number): number {
  return price * (taxRate / 100);
}

function calculateTotal(
  price: number,
  quantity: number,
  discountPercent: number,
  taxRate: number
): number {
  const subtotal = price * quantity;
  const afterDiscount = applyDiscount(subtotal, discountPercent);
  const tax = calculateTax(afterDiscount, taxRate);
  return afterDiscount + tax;
}

// Simple test framework (for demonstration)
type TestResult = { passed: boolean; message: string };

function assertEquals<T>(actual: T, expected: T, message: string): TestResult {
  const passed = JSON.stringify(actual) === JSON.stringify(expected);
  return {
    passed,
    message: passed
      ? `✓ ${message}`
      : `✗ ${message}\n  Expected: ${expected}\n  Actual: ${actual}`
  };
}

function assertThrows(fn: () => void, message: string): TestResult {
  try {
    fn();
    return {
      passed: false,
      message: `✗ ${message}\n  Expected error but none was thrown`
    };
  } catch (error) {
    return {
      passed: true,
      message: `✓ ${message}`
    };
  }
}

function describe(suiteName: string, tests: () => void): void {
  console.log(`\n${suiteName}`);
  console.log('='.repeat(suiteName.length));
  tests();
}

function it(testName: string, testFn: () => TestResult): void {
  const result = testFn();
  console.log(result.message);
}

// Test suite
console.log('=== Testing Pure Functions ===');

describe('calculateDiscount', () => {
  it('should calculate 10% discount correctly', () =>
    assertEquals(calculateDiscount(100, 10), 10, 'Discount of 10% on $100')
  );
  
  it('should calculate 25% discount correctly', () =>
    assertEquals(calculateDiscount(200, 25), 50, 'Discount of 25% on $200')
  );
  
  it('should handle 0% discount', () =>
    assertEquals(calculateDiscount(100, 0), 0, 'No discount')
  );
  
  it('should handle 100% discount', () =>
    assertEquals(calculateDiscount(100, 100), 100, 'Full discount')
  );
  
  it('should throw on negative price', () =>
    assertThrows(
      () => calculateDiscount(-10, 10),
      'Negative price throws error'
    )
  );
  
  it('should throw on invalid discount percentage', () =>
    assertThrows(
      () => calculateDiscount(100, 150),
      'Discount over 100% throws error'
    )
  );
});

describe('applyDiscount', () => {
  it('should apply 20% discount to $100', () =>
    assertEquals(applyDiscount(100, 20), 80, 'Price after 20% discount')
  );
  
  it('should return original price with 0% discount', () =>
    assertEquals(applyDiscount(100, 0), 100, 'No discount applied')
  );
});

describe('calculateTotal', () => {
  it('should calculate total with discount and tax', () => {
    const total = calculateTotal(
      100,  // price
      2,    // quantity
      10,   // 10% discount
      8     // 8% tax
    );
    // Subtotal: 200
    // After 10% discount: 180
    // Tax 8%: 14.4
    // Total: 194.4
    return assertEquals(total, 194.4, 'Total with discount and tax');
  });
  
  it('should handle no discount', () => {
    const total = calculateTotal(50, 1, 0, 10);
    // Subtotal: 50
    // After 0% discount: 50
    // Tax 10%: 5
    // Total: 55
    return assertEquals(total, 55, 'Total with no discount');
  });
});
```

</details>

---

### Property-Based Testing

**Concept**: Instead of testing specific inputs, test properties that should always hold true.

<details>
<summary><strong>View Codes/Examples/Script</strong></summary>

```ts
// Property-based test helpers
function randomInt(min: number, max: number): number {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

function randomPrice(): number {
  return randomInt(1, 1000);
}

function randomPercent(): number {
  return randomInt(0, 100);
}

// Property: Applying 0% discount should return original price
function testDiscountZeroProperty(): TestResult {
  const samples = 100;
  
  for (let i = 0; i < samples; i++) {
    const price = randomPrice();
    const result = applyDiscount(price, 0);
    
    if (result !== price) {
      return {
        passed: false,
        message: `✗ Property violated: 0% discount should not change price (price: ${price}, result: ${result})`
      };
    }
  }
  
  return {
    passed: true,
    message: `✓ Property holds: 0% discount returns original price (tested ${samples} cases)`
  };
}

// Property: Discount should never make price negative
function testDiscountNonNegativeProperty(): TestResult {
  const samples = 100;
  
  for (let i = 0; i < samples; i++) {
    const price = randomPrice();
    const discount = randomPercent();
    const result = applyDiscount(price, discount);
    
    if (result < 0) {
      return {
        passed: false,
        message: `✗ Property violated: Result should not be negative (price: ${price}, discount: ${discount}%, result: ${result})`
      };
    }
  }
  
  return {
    passed: true,
    message: `✓ Property holds: Discount never produces negative price (tested ${samples} cases)`
  };
}

// Property: 100% discount should result in 0
function testDiscount100Property(): TestResult {
  const samples = 100;
  
  for (let i = 0; i < samples; i++) {
    const price = randomPrice();
    const result = applyDiscount(price, 100);
    
    if (result !== 0) {
      return {
        passed: false,
        message: `✗ Property violated: 100% discount should result in 0 (price: ${price}, result: ${result})`
      };
    }
  }
  
  return {
    passed: true,
    message: `✓ Property holds: 100% discount always results in 0 (tested ${samples} cases)`
  };
}

// Property: Applying discount twice with half percentage should equal applying once
function testDiscountCommutativityProperty(): TestResult {
  const samples = 50;
  
  for (let i = 0; i < samples; i++) {
    const price = randomPrice();
    const discount = randomPercent();
    
    // Apply discount once
    const onceResult = applyDiscount(price, discount);
    
    // Apply half discount twice
    const halfDiscount = discount / 2;
    const twiceResult = applyDiscount(applyDiscount(price, halfDiscount), halfDiscount);
    
    // Allow small floating point difference
    const difference = Math.abs(onceResult - twiceResult);
    
    if (difference > 0.01) {
      return {
        passed: false,
        message: `✗ Property violated: Two half discounts ≠ one full discount (price: ${price}, discount: ${discount}%, once: ${onceResult}, twice: ${twiceResult})`
      };
    }
  }
  
  return {
    passed: true,
    message: `✓ Property holds: Two half discounts ≈ one full discount (tested ${samples} cases)`
  };
}

describe('Property-Based Tests', () => {
  it('0% discount returns original price', testDiscountZeroProperty);
  it('Discount never produces negative price', testDiscountNonNegativeProperty);
  it('100% discount always results in 0', testDiscount100Property);
  it('Two half discounts equal one full discount', testDiscountCommutativityProperty);
});

// Property: Idempotence - applying filter twice = applying once
function filterActive<T extends { active: boolean }>(items: T[]): T[] {
  return items.filter(item => item.active);
}

function testFilterIdempotenceProperty(): TestResult {
  const samples = 50;
  
  for (let i = 0; i < samples; i++) {
    // Generate random items
    const items = Array.from({ length: randomInt(5, 20) }, (_, i) => ({
      id: i,
      active: Math.random() > 0.5
    }));
    
    const onceFiltered = filterActive(items);
    const twiceFiltered = filterActive(filterActive(items));
    
    if (JSON.stringify(onceFiltered) !== JSON.stringify(twiceFiltered)) {
      return {
        passed: false,
        message: '✗ Property violated: Filter should be idempotent'
      };
    }
  }
  
  return {
    passed: true,
    message: `✓ Property holds: Filter is idempotent (tested ${samples} cases)`
  };
}

describe('Filter Properties', () => {
  it('Filter is idempotent', testFilterIdempotenceProperty);
});

// Property: Commutativity - order of operations shouldn't matter
function add(a: number, b: number): number {
  return a + b;
}

function testAddCommutativityProperty(): TestResult {
  const samples = 100;
  
  for (let i = 0; i < samples; i++) {
    const a = randomInt(-1000, 1000);
    const b = randomInt(-1000, 1000);
    
    if (add(a, b) !== add(b, a)) {
      return {
        passed: false,
        message: `✗ Property violated: Addition should be commutative (a: ${a}, b: ${b})`
      };
    }
  }
  
  return {
    passed: true,
    message: `✓ Property holds: Addition is commutative (tested ${samples} cases)`
  };
}

describe('Arithmetic Properties', () => {
  it('Addition is commutative', testAddCommutativityProperty);
});
```

</details>

---

### Testing Higher-Order Functions

<details>
<summary><strong>View Codes/Examples/Script</strong></summary>

```ts
// Higher-order functions to test
function map<T, U>(arr: T[], fn: (item: T) => U): U[] {
  const result: U[] = [];
  for (const item of arr) {
    result.push(fn(item));
  }
  return result;
}

function filter<T>(arr: T[], predicate: (item: T) => boolean): T[] {
  const result: T[] = [];
  for (const item of arr) {
    if (predicate(item)) {
      result.push(item);
    }
  }
  return result;
}

function reduce<T, R>(arr: T[], fn: (acc: R, item: T) => R, initial: R): R {
  let result = initial;
  for (const item of arr) {
    result = fn(result, item);
  }
  return result;
}

function compose<A, B, C>(f: (b: B) => C, g: (a: A) => B): (a: A) => C {
  return (a: A) => f(g(a));
}

describe('map function', () => {
  it('should transform array elements', () => {
    const input = [1, 2, 3];
    const double = (n: number) => n * 2;
    const result = map(input, double);
    return assertEquals(result, [2, 4, 6], 'Map doubles each element');
  });
  
  it('should handle empty array', () => {
    const result = map([], (n: number) => n * 2);
    return assertEquals(result, [], 'Map on empty array returns empty');
  });
  
  it('should preserve array length', () => {
    const input = [1, 2, 3, 4, 5];
    const result = map(input, n => n * 2);
    return assertEquals(result.length, input.length, 'Map preserves length');
  });
});

describe('filter function', () => {
  it('should filter elements by predicate', () => {
    const input = [1, 2, 3, 4, 5];
    const isEven = (n: number) => n % 2 === 0;
    const result = filter(input, isEven);
    return assertEquals(result, [2, 4], 'Filter keeps even numbers');
  });
  
  it('should return empty array when nothing matches', () => {
    const input = [1, 3, 5];
    const isEven = (n: number) => n % 2 === 0;
    const result = filter(input, isEven);
    return assertEquals(result, [], 'Filter returns empty when no matches');
  });
  
  it('should return all elements when all match', () => {
    const input = [2, 4, 6];
    const isEven = (n: number) => n % 2 === 0;
    const result = filter(input, isEven);
    return assertEquals(result, [2, 4, 6], 'Filter returns all when all match');
  });
});

describe('reduce function', () => {
  it('should sum array elements', () => {
    const input = [1, 2, 3, 4];
    const sum = reduce(input, (acc, n) => acc + n, 0);
    return assertEquals(sum, 10, 'Reduce sums elements');
  });
  
  it('should handle empty array', () => {
    const sum = reduce([], (acc: number, n: number) => acc + n, 0);
    return assertEquals(sum, 0, 'Reduce on empty array returns initial');
  });
  
  it('should build object from array', () => {
    const input = ['a', 'b', 'c'];
    const obj = reduce(
      input,
      (acc, key) => ({ ...acc, [key]: key.toUpperCase() }),
      {} as Record<string, string>
    );
    return assertEquals(
      obj,
      { a: 'A', b: 'B', c: 'C' },
      'Reduce builds object'
    );
  });
});

describe('compose function', () => {
  it('should compose two functions', () => {
    const add1 = (n: number) => n + 1;
    const double = (n: number) => n * 2;
    const add1ThenDouble = compose(double, add1);
    
    return assertEquals(add1ThenDouble(5), 12, 'Compose: (5+1)*2 = 12');
  });
  
  it('should execute right to left', () => {
    const toString = (n: number) => n.toString();
    const length = (s: string) => s.length;
    const getLength = compose(length, toString);
    
    return assertEquals(getLength(12345), 5, 'Compose: 12345 → "12345" → 5');
  });
});

// Test composition laws
describe('Composition Laws', () => {
  it('should satisfy identity law', () => {
    const identity = <T>(x: T) => x;
    const double = (n: number) => n * 2;
    
    const composed = compose(identity, double);
    
    // f ∘ id = f
    return assertEquals(composed(5), double(5), 'Compose with identity equals original');
  });
  
  it('should satisfy associativity law', () => {
    const add1 = (n: number) => n + 1;
    const double = (n: number) => n * 2;
    const square = (n: number) => n * n;
    
    // (f ∘ g) ∘ h = f ∘ (g ∘ h)
    const left = compose(compose(square, double), add1);
    const right = compose(square, compose(double, add1));
    
    const input = 3;
    return assertEquals(
      left(input),
      right(input),
      'Composition is associative'
    );
  });
});
```

</details>

---

### Testing with Mock Data Generators

<details>
<summary><strong>View Codes/Examples/Script</strong></summary>

```ts
// Domain types
interface Product {
  id: string;
  name: string;
  price: number;
  category: string;
  inStock: boolean;
}

interface Order {
  orderId: string;
  userId: string;
  items: Array<{ productId: string; quantity: number }>;
  total: number;
}

// Pure functions to test
function calculateOrderTotal(
  items: Array<{ productId: string; quantity: number }>,
  products: Product[]
): number {
  return items.reduce((total, item) => {
    const product = products.find(p => p.id === item.productId);
    if (!product) return total;
    return total + (product.price * item.quantity);
  }, 0);
}

function getProductsByCategory(products: Product[], category: string): Product[] {
  return products.filter(p => p.category === category);
}

function getInStockProducts(products: Product[]): Product[] {
  return products.filter(p => p.inStock);
}

function findMostExpensiveProduct(products: Product[]): Product | null {
  if (products.length === 0) return null;
  
  return products.reduce((max, product) =>
    product.price > max.price ? product : max
  );
}

// Test data generators
class TestDataGenerator {
  private idCounter = 0;
  
  generateId(): string {
    return `TEST-${++this.idCounter}`;
  }
  
  generateProduct(overrides?: Partial<Product>): Product {
    const categories = ['Electronics', 'Clothing', 'Books', 'Food'];
    
    return {
      id: this.generateId(),
      name: `Product ${this.idCounter}`,
      price: randomInt(10, 1000),
      category: categories[randomInt(0, categories.length - 1)],
      inStock: Math.random() > 0.3,
      ...overrides
    };
  }
  
  generateProducts(count: number, overrides?: Partial<Product>): Product[] {
    return Array.from({ length: count }, () => this.generateProduct(overrides));
  }
  
  generateOrder(productIds: string[], overrides?: Partial<Order>): Order {
    return {
      orderId: this.generateId(),
      userId: this.generateId(),
      items: productIds.map(id => ({
        productId: id,
        quantity: randomInt(1, 5)
      })),
      total: 0,
      ...overrides
    };
  }
}

// Tests using generated data
describe('calculateOrderTotal with generated data', () => {
  it('should calculate total for single item', () => {
    const gen = new TestDataGenerator();
    const product = gen.generateProduct({ price: 100 });
    const items = [{ productId: product.id, quantity: 2 }];
    
    const total = calculateOrderTotal(items, [product]);
    return assertEquals(total, 200, 'Total for 2 items at $100 each');
  });
  
  it('should handle missing products', () => {
    const gen = new TestDataGenerator();
    const product = gen.generateProduct({ price: 50 });
    const items = [
      { productId: product.id, quantity: 1 },
      { productId: 'NON-EXISTENT', quantity: 5 }
    ];
    
    const total = calculateOrderTotal(items, [product]);
    return assertEquals(total, 50, 'Ignores non-existent products');
  });
  
  it('should work with many products', () => {
    const gen = new TestDataGenerator();
    const products = gen.generateProducts(100);
    const items = products.slice(0, 5).map(p => ({
      productId: p.id,
      quantity: 1
    }));
    
    const total = calculateOrderTotal(items, products);
    const expectedTotal = products
      .slice(0, 5)
      .reduce((sum, p) => sum + p.price, 0);
    
    return assertEquals(total, expectedTotal, 'Calculates total for many products');
  });
});

describe('getProductsByCategory with generated data', () => {
  it('should filter by category', () => {
    const gen = new TestDataGenerator();
    const electronics = gen.generateProducts(3, { category: 'Electronics' });
    const books = gen.generateProducts(2, { category: 'Books' });
    const allProducts = [...electronics, ...books];
    
    const result = getProductsByCategory(allProducts, 'Electronics');
    
    return assertEquals(
      result.length,
      3,
      'Returns only Electronics products'
    );
  });
  
  it('should return empty array for non-existent category', () => {
    const gen = new TestDataGenerator();
    const products = gen.generateProducts(5);
    
    const result = getProductsByCategory(products, 'NonExistent');
    return assertEquals(result, [], 'Returns empty for non-existent category');
  });
});

describe('findMostExpensiveProduct with generated data', () => {
  it('should find most expensive product', () => {
    const gen = new TestDataGenerator();
    const products = [
      gen.generateProduct({ price: 100 }),
      gen.generateProduct({ price: 500 }),
      gen.generateProduct({ price: 250 })
    ];
    
    const result = findMostExpensiveProduct(products);
    return assertEquals(result?.price, 500, 'Finds most expensive');
  });
  
  it('should return null for empty array', () => {
    const result = findMostExpensiveProduct([]);
    return assertEquals(result, null, 'Returns null for empty array');
  });
});

// Snapshot testing (simplified)
function createSnapshot<T>(value: T): string {
  return JSON.stringify(value, null, 2);
}

function matchesSnapshot<T>(actual: T, expected: string): TestResult {
  const actualSnapshot = createSnapshot(actual);
  const passed = actualSnapshot === expected;
  
  return {
    passed,
    message: passed
      ? '✓ Matches snapshot'
      : `✗ Snapshot mismatch\n  Expected:\n${expected}\n  Actual:\n${actualSnapshot}`
  };
}

describe('Snapshot Tests', () => {
  it('should match product structure', () => {
    const product: Product = {
      id: 'P1',
      name: 'Laptop',
      price: 999,
      category: 'Electronics',
      inStock: true
    };
    
    const expectedSnapshot = `{
  "id": "P1",
  "name": "Laptop",
  "price": 999,
  "category": "Electronics",
  "inStock": true
}`;
    
    return matchesSnapshot(product, expectedSnapshot);
  });
});
```

</details>

---

### Testing Async Pure Functions

<details>
<summary><strong>View Codes/Examples/Script</strong></summary>

```ts
// Async pure functions (deterministic with mocked dependencies)
async function fetchAndTransform(
  fetcher: (id: string) => Promise<Result<User, ApiError>>,
  userId: string
): Promise<Result<string, ApiError>> {
  const result = await fetcher(userId);
  
  if (!result.ok) return result as Result<string, ApiError>;
  
  return Ok(`Hello, ${result.value.name}!`);
}

async function processUsers(
  fetcher: (id: string) => Promise<Result<User, ApiError>>,
  userIds: string[]
): Promise<Result<User[], ApiError>> {
  const results = await Promise.all(userIds.map(id => fetcher(id)));
  
  const failed = results.find(r => !r.ok);
  if (failed && !failed.ok) {
    return failed as Result<User[], ApiError>;
  }
  
  const users = results.map(r => (r as { ok: true; value: User }).value);
  return Ok(users);
}

// Test helpers for async
async function asyncAssertEquals<T>(
  actual: T,
  expected: T,
  message: string
): Promise<TestResult> {
  return assertEquals(actual, expected, message);
}

// Mock fetcher
function createMockFetcher(
  responses: Record<string, Result<User, ApiError>>
): (id: string) => Promise<Result<User, ApiError>> {
  return async (id: string) => {
    await new Promise(resolve => setTimeout(resolve, 10)); // Simulate delay
    return responses[id] || Err({ code: 'NOT_FOUND', message: 'User not found' });
  };
}

// Async tests
async function runAsyncTests() {
  console.log('\n=== Async Function Tests ===');
  
  // Test successful fetch
  const mockSuccess = createMockFetcher({
    'U1': Ok({
      id: 'U1',
      name: 'Alice',
      email: 'alice@example.com',
      preferences: { notifications: true, theme: 'light' }
    })
  });
  
  const result1 = await fetchAndTransform(mockSuccess, 'U1');
  const test1 = await asyncAssertEquals(
    result1.ok ? result1.value : null,
    'Hello, Alice!',
    'Transforms user to greeting'
  );
  console.log(test1.message);
  
  // Test error propagation
  const mockError = createMockFetcher({});
  const result2 = await fetchAndTransform(mockError, 'U1');
  const test2 = assertEquals(
    result2.ok,
    false,
    'Propagates error from fetcher'
  );
  console.log(test2.message);
  
  // Test parallel processing
  const mockMultiple = createMockFetcher({
    'U1': Ok({ id: 'U1', name: 'Alice', email: 'alice@example.com', preferences: { notifications: true, theme: 'light' } }),
    'U2': Ok({ id: 'U2', name: 'Bob', email: 'bob@example.com', preferences: { notifications: false, theme: 'dark' } }),
    'U3': Ok({ id: 'U3', name: 'Carol', email: 'carol@example.com', preferences: { notifications: true, theme: 'light' } })
  });
  
  const result3 = await processUsers(mockMultiple, ['U1', 'U2', 'U3']);
  const test3 = assertEquals(
    result3.ok ? result3.value.length : 0,
    3,
    'Processes multiple users in parallel'
  );
  console.log(test3.message);
}

runAsyncTests();
```

</details>

---

## Practice Questions

<details>
<summary><strong>View Questions</strong></summary>

### Fill in the Blanks

1. Pure functions are __________ - same input always produces same output.
2. Property-based testing checks __________ that should always hold rather than specific cases.
3. Higher-order functions are tested by passing different __________ as arguments.
4. Mock data generators help create __________ test data for comprehensive testing.

<details>
<summary><strong>View Answers</strong></summary>

1. **deterministic** - Pure functions have no randomness or hidden dependencies affecting output
2. **properties** (or "invariants") - Like "filtering twice equals filtering once" or "discount never produces negative price"
3. **functions** - Test map/filter/reduce by passing different transformation/predicate functions
4. **realistic** (or "random") - Generators create varied, realistic test data without manual setup

</details>

---

### True/False

1. ⬜ Pure functions require mocking dependencies for testing
2. ⬜ Property-based testing is more thorough than example-based testing
3. ⬜ Testing pure functions requires complex test frameworks
4. ⬜ Composition laws can be verified through testing

<details>
<summary><strong>View Answers</strong></summary>

1. **False** - Pure functions have no dependencies; they only depend on their inputs
2. **True** - Property tests run hundreds of random cases vs a few hand-picked examples, finding edge cases
3. **False** - Pure functions are simple to test with basic assertions (though frameworks help with organization)
4. **True** - You can test that `compose(f, compose(g, h)) === compose(compose(f, g), h)` for various functions

</details>

---

### Multiple Choice

1. What makes pure functions easy to test?

- A) They're always fast
- B) Same input → same output, no side effects
- C) They use less memory
- D) They're shorter

2. What's the main advantage of property-based testing?

- A) Faster execution
- B) Less code to write
- C) Finds edge cases you didn't think of
- D) Easier to read

<details>
<summary><strong>View Answers</strong></summary>

1. **B** - Same input → same output, no side effects. Makes tests deterministic with no mocks needed

2. **C** - Finds edge cases you didn't think of. Random testing with hundreds of cases catches bugs that slip through hand-picked examples

</details>

---

### Code Challenge

Write comprehensive tests for a shopping cart system:

```ts
// TODO: Test these pure functions thoroughly
interface CartItem {
  productId: string;
  price: number;
  quantity: number;
}

function addToCart(cart: CartItem[], item: CartItem): CartItem[];
function removeFromCart(cart: CartItem[], productId: string): CartItem[];
function updateQuantity(cart: CartItem[], productId: string, quantity: number): CartItem[];
function calculateSubtotal(cart: CartItem[]): number;
function applyBulkDiscount(cart: CartItem[], minQuantity: number, discountPercent: number): number;

// Write:
// 1. Unit tests for each function
// 2. Property-based tests
// 3. Test data generators
// 4. Integration tests combining functions
```

<details>
<summary><strong>View Answers</strong></summary>

```ts
// Implementation
interface CartItem {
  productId: string;
  price: number;
  quantity: number;
}

function addToCart(cart: CartItem[], item: CartItem): CartItem[] {
  const existingIndex = cart.findIndex(i => i.productId === item.productId);
  
  if (existingIndex >= 0) {
    return cart.map((i, idx) =>
      idx === existingIndex
        ? { ...i, quantity: i.quantity + item.quantity }
        : i
    );
  }
  
  return [...cart, item];
}

function removeFromCart(cart: CartItem[], productId: string): CartItem[] {
  return cart.filter(item => item.productId !== productId);
}

function updateQuantity(
  cart: CartItem[],
  productId: string,
  quantity: number
): CartItem[] {
  if (quantity <= 0) {
    return removeFromCart(cart, productId);
  }
  
  return cart.map(item =>
    item.productId === productId
      ? { ...item, quantity }
      : item
  );
}

function calculateSubtotal(cart: CartItem[]): number {
  return cart.reduce((sum, item) => sum + (item.price * item.quantity), 0);
}

function applyBulkDiscount(
  cart: CartItem[],
  minQuantity: number,
  discountPercent: number
): number {
  const subtotal = calculateSubtotal(cart);
  const totalItems = cart.reduce((sum, item) => sum + item.quantity, 0);
  
  if (totalItems >= minQuantity) {
    return subtotal * (1 - discountPercent / 100);
  }
  
  return subtotal;
}

// Test data generator
class CartTestGenerator {
  generateItem(overrides?: Partial<CartItem>): CartItem {
    return {
      productId: `P${randomInt(1, 100)}`,
      price: randomInt(10, 500),
      quantity: randomInt(1, 10),
      ...overrides
    };
  }
  
  generateCart(itemCount: number): CartItem[] {
    return Array.from({ length: itemCount }, () => this.generateItem());
  }
}

// Unit Tests
describe('Cart Functions - Unit Tests', () => {
  describe('addToCart', () => {
    it('should add new item to empty cart', () => {
      const item: CartItem = { productId: 'P1', price: 100, quantity: 1 };
      const result = addToCart([], item);
      return assertEquals(result, [item], 'Adds item to empty cart');
    });
    
    it('should add new item to existing cart', () => {
      const cart: CartItem[] = [
        { productId: 'P1', price: 100, quantity: 1 }
      ];
      const newItem: CartItem = { productId: 'P2', price: 50, quantity: 2 };
      const result = addToCart(cart, newItem);
      
      return assertEquals(result.length, 2, 'Cart has 2 items');
    });
    
    it('should increment quantity for existing item', () => {
      const cart: CartItem[] = [
        { productId: 'P1', price: 100, quantity: 2 }
      ];
      const sameItem: CartItem = { productId: 'P1', price: 100, quantity: 1 };
      const result = addToCart(cart, sameItem);
      
      const updatedItem = result.find(i => i.productId === 'P1');
      return assertEquals(updatedItem?.quantity, 3, 'Quantity increased to 3');
    });
  });
  
  describe('removeFromCart', () => {
    it('should remove item from cart', () => {
      const cart: CartItem[] = [
        { productId: 'P1', price: 100, quantity: 1 },
        { productId: 'P2', price: 50, quantity: 2 }
      ];
      const result = removeFromCart(cart, 'P1');
      
      return assertEquals(result.length, 1, 'Cart has 1 item after removal');
    });
    
    it('should handle removing non-existent item', () => {
      const cart: CartItem[] = [
        { productId: 'P1', price: 100, quantity: 1 }
      ];
      const result = removeFromCart(cart, 'P99');
      
      return assertEquals(result.length, 1, 'Cart unchanged');
    });
  });
  
  describe('updateQuantity', () => {
    it('should update item quantity', () => {
      const cart: CartItem[] = [
        { productId: 'P1', price: 100, quantity: 1 }
      ];
      const result = updateQuantity(cart, 'P1', 5);
      
      const item = result.find(i => i.productId === 'P1');
      return assertEquals(item?.quantity, 5, 'Quantity updated to 5');
    });
    
    it('should remove item when quantity is 0', () => {
      const cart: CartItem[] = [
        { productId: 'P1', price: 100, quantity: 1 }
      ];
      const result = updateQuantity(cart, 'P1', 0);
      
      return assertEquals(result.length, 0, 'Item removed when quantity 0');
    });
  });
  
  describe('calculateSubtotal', () => {
    it('should calculate subtotal correctly', () => {
      const cart: CartItem[] = [
        { productId: 'P1', price: 100, quantity: 2 },
        { productId: 'P2', price: 50, quantity: 3 }
      ];
      const result = calculateSubtotal(cart);
      
      // (100 * 2) + (50 * 3) = 350
      return assertEquals(result, 350, 'Subtotal is 350');
    });
    
    it('should return 0 for empty cart', () => {
      const result = calculateSubtotal([]);
      return assertEquals(result, 0, 'Empty cart subtotal is 0');
    });
  });
  
  describe('applyBulkDiscount', () => {
    it('should apply discount when quantity threshold met', () => {
      const cart: CartItem[] = [
        { productId: 'P1', price: 100, quantity: 10 }
      ];
      const result = applyBulkDiscount(cart, 10, 10);
      
      // 1000 * 0.9 = 900
      return assertEquals(result, 900, 'Applies 10% discount');
    });
    
    it('should not apply discount when threshold not met', () => {
      const cart: CartItem[] = [
        { productId: 'P1', price: 100, quantity: 5 }
      ];
      const result = applyBulkDiscount(cart, 10, 10);
      
      return assertEquals(result, 500, 'No discount applied');
    });
  });
});

// Property-Based Tests
describe('Cart Functions - Property Tests', () => {
  it('Adding then removing item returns original cart', () => {
    const samples = 50;
    const gen = new CartTestGenerator();
    
    for (let i = 0; i < samples; i++) {
      const cart = gen.generateCart(5);
      const item = gen.generateItem({ productId: 'NEW' });
      
      const withItem = addToCart(cart, item);
      const withoutItem = removeFromCart(withItem, 'NEW');
      
      if (JSON.stringify(cart) !== JSON.stringify(withoutItem)) {
        return {
          passed: false,
          message: '✗ Add then remove should return original cart'
        };
      }
    }
    
    return {
      passed: true,
      message: `✓ Add then remove returns original (tested ${samples} cases)`
    };
  });
  
  it('Subtotal is never negative', () => {
    const samples = 100;
    const gen = new CartTestGenerator();
    
    for (let i = 0; i < samples; i++) {
      const cart = gen.generateCart(randomInt(0, 20));
      const subtotal = calculateSubtotal(cart);
      
      if (subtotal < 0) {
        return {
          passed: false,
          message: '✗ Subtotal should never be negative'
        };
      }
    }
    
    return {
      passed: true,
      message: `✓ Subtotal is never negative (tested ${samples} cases)`
    };
  });
  
  it('Removing all items results in empty cart', () => {
    const samples = 50;
    const gen = new CartTestGenerator();
    
    for (let i = 0; i < samples; i++) {
      let cart = gen.generateCart(randomInt(1, 10));
      
      // Remove all items
      const productIds = cart.map(item => item.productId);
      for (const id of productIds) {
        cart = removeFromCart(cart, id);
      }
      
      if (cart.length !== 0) {
        return {
          passed: false,
          message: '✗ Removing all items should result in empty cart'
        };
      }
    }
    
    return {
      passed: true,
      message: `✓ Removing all items results in empty cart (tested ${samples} cases)`
    };
  });
});

// Integration Tests
describe('Cart Functions - Integration Tests', () => {
  it('Complete shopping flow', () => {
    let cart: CartItem[] = [];
    
    // Add items
    cart = addToCart(cart, { productId: 'P1', price: 100, quantity: 2 });
    cart = addToCart(cart, { productId: 'P2', price: 50, quantity: 3 });
    cart = addToCart(cart, { productId: 'P3', price: 25, quantity: 10 });
    
    // Update quantity
    cart = updateQuantity(cart, 'P2', 5);
    
    // Remove one item
    cart = removeFromCart(cart, 'P3');
    
    // Calculate final total
    const subtotal = calculateSubtotal(cart);
    // (100 * 2) + (50 * 5) = 450
    
    return assertEquals(subtotal, 450, 'Complete flow calculates correctly');
  });
  
  it('Bulk discount integration', () => {
    let cart: CartItem[] = [];
    
    // Add items to meet bulk threshold
    cart = addToCart(cart, { productId: 'P1', price: 100, quantity: 8 });
    cart = addToCart(cart, { productId: 'P2', price: 50, quantity: 3 });
    
    // Total quantity: 11, should get discount
    const total = applyBulkDiscount(cart, 10, 15);
    // Subtotal: 950
    // With 15% discount: 807.5
    
    return assertEquals(total, 807.5, 'Bulk discount applied correctly');
  });
});

console.log('\n=== Running Complete Test Suite ===');
```

**Explanation**:
- **Unit tests**: Test each function in isolation with specific inputs
- **Property tests**: Verify invariants hold across random inputs
- **Integration tests**: Test functions working together in realistic workflows
- **Test generators**: Create realistic random test data
- **Coverage**: Tests happy paths, edge cases, and error conditions

</details>

</details>

---