[✔]: https://raw.githubusercontent.com/foyez/webdev-roadmap/master/assets/images/checkbox-small-blue.png

# Functional Programming

**Functional Programming (FP)** is a programming paradigm that treats computation as the evaluation of mathematical functions and avoids changing state and mutable data.

### Think of it like this:
**OOP** = Objects + Messages (like a company with departments)  
**FP** = Functions + Data Flow (like a production assembly line)

### Key Philosophy:
```
Input → Pure Function → Output
(No surprises, no side effects, predictable results)
```

### Why Functional Programming?

**Traditional OOP Problems:**
- `this` keyword confusion in JavaScript
- Complex prototypal inheritance
- Shared mutable state causing bugs
- Hard to test and debug
- Race conditions in concurrent code

**FP Solutions:**
- ✅ **Predictable** - Same input always gives same output
- ✅ **Testable** - Pure functions are easy to test
- ✅ **Debuggable** - No hidden state to track
- ✅ **Concurrent** - No shared state = no race conditions
- ✅ **Composable** - Build complex functions from simple ones
- ✅ **Maintainable** - Less code, clearer intent

---

## Core Concepts of FP

Think of these as the **PICHI** principles:
1. **P**ure Functions
2. **I**mmutability
3. **C**omposition
4. **H**igher-Order Functions
5. **I**mperative → Declarative

---

### 1. Pure Functions 🔵
**"Same input → Same output, every time. No surprises!"**

#### Real-Life Analogy
A **vending machine**: You put $1 + press button B3 → Always get the same candy bar. The machine doesn't randomly give you different items or run out without telling you.

#### Definition
A pure function:
1. **Always returns the same output for the same input** (deterministic)
2. **Has no side effects** (doesn't modify external state)
3. **Doesn't depend on external state** (only uses its parameters)

<details>
<summary><strong>View contents</strong></summary>

#### Examples

**❌ Impure Functions:**

```typescript
// TypeScript - Impure examples
let multiplier = 2;

// Impure: depends on external state
function multiplyImpure(x: number): number {
  return x * multiplier;  // Depends on 'multiplier' variable
}

console.log(multiplyImpure(5));  // 10
multiplier = 3;
console.log(multiplyImpure(5));  // 15 - Different output for same input!

// Impure: modifies external state
let total = 0;
function addToTotalImpure(x: number): number {
  total += x;  // Side effect: modifies 'total'
  return total;
}

// Impure: has side effects
function greetImpure(name: string): string {
  console.log(`Hello ${name}`);  // Side effect: I/O operation
  return `Hello ${name}`;
}

// Impure: depends on external data
function getCurrentTimeImpure(): Date {
  return new Date();  // Different output each call!
}
```

**✅ Pure Functions:**

```typescript
// TypeScript - Pure examples
function multiplyPure(x: number, multiplier: number): number {
  return x * multiplier;  // Only uses parameters
}

console.log(multiplyPure(5, 2));  // 10
console.log(multiplyPure(5, 2));  // 10 - Always the same!

// Pure: returns value without side effects
function greetPure(name: string): string {
  return `Hello ${name}`;  // Just returns, doesn't print
}

// Pure: all inputs provided
function addPure(x: number, y: number): number {
  return x + y;
}

// Pure: complex calculation, but deterministic
function calculateDiscount(price: number, discountPercent: number): number {
  return price - (price * discountPercent / 100);
}
```

**Go Examples:**

```go
package main

import "fmt"

// ❌ Impure: modifies external state
var counter = 0

func incrementImpure() int {
    counter++  // Side effect
    return counter
}

// ✅ Pure: no side effects
func incrementPure(n int) int {
    return n + 1
}

// ❌ Impure: depends on external state
var taxRate = 0.08

func calculateTotalImpure(price float64) float64 {
    return price + (price * taxRate)
}

// ✅ Pure: all dependencies provided as parameters
func calculateTotalPure(price, taxRate float64) float64 {
    return price + (price * taxRate)
}

func main() {
    // Impure function gives different results
    fmt.Println(incrementImpure())  // 1
    fmt.Println(incrementImpure())  // 2
    
    // Pure function gives same results
    fmt.Println(incrementPure(5))   // 6
    fmt.Println(incrementPure(5))   // 6
}
```

</details>

#### Benefits of Pure Functions:
1. **Memoization** - Cache results since output is deterministic
2. **Parallel execution** - No shared state = safe to run in parallel
3. **Testability** - No setup/teardown needed
4. **Debugging** - Easy to reproduce bugs
5. **Referential transparency** - Can replace function call with its result

#### Interview Tip:
"Pure functions are like mathematical functions: f(x) = 2x always gives the same result. They're the foundation of FP because they're predictable, testable, and composable."

---

### 2. Immutability 🔒
**"Don't change data. Create new data instead!"**

#### Real-Life Analogy
**Photocopy machines**: When you need to edit a document, you don't write on the original (mutation). You make a copy, edit that, and keep the original safe (immutability).

#### Definition
Immutability means once data is created, it cannot be changed. Instead, you create new data with the desired changes.

#### Why Immutability?
- ✅ **No unexpected changes** - Data can't be modified elsewhere
- ✅ **Time travel debugging** - Keep history of all states
- ✅ **Thread safety** - Multiple threads can't corrupt data
- ✅ **Simpler reasoning** - Data doesn't change under your feet
- ✅ **Undo/Redo** - Easy to implement with immutable history

<details>
<summary><strong>View contents</strong></summary>

#### Examples

**❌ Mutable (Bad):**

```typescript
// TypeScript - Mutation
interface User {
  name: string;
  age: number;
  email: string;
}

const user: User = { name: "Alice", age: 25, email: "alice@email.com" };

// Mutation: modifying original object
function updateEmail(user: User, newEmail: string): void {
  user.email = newEmail;  // Modifies the original!
}

updateEmail(user, "alice@newemail.com");
console.log(user.email);  // "alice@newemail.com" - Original changed!

// Array mutation
const numbers = [1, 2, 3, 4, 5];
numbers.push(6);           // Mutates original array
numbers[0] = 99;           // Mutates original array
console.log(numbers);      // [99, 2, 3, 4, 5, 6] - Original changed!
```

**✅ Immutable (Good):**

```typescript
// TypeScript - Immutability
const user: User = { name: "Alice", age: 25, email: "alice@email.com" };

// Immutable: returns new object
function updateEmailImmutable(user: User, newEmail: string): User {
  return { ...user, email: newEmail };  // Creates new object
}

const updatedUser = updateEmailImmutable(user, "alice@newemail.com");
console.log(user.email);         // "alice@email.com" - Original unchanged!
console.log(updatedUser.email);  // "alice@newemail.com" - New object

// Array immutability
const numbers = [1, 2, 3, 4, 5];

// Add element (immutable way)
const withSix = [...numbers, 6];              // [1, 2, 3, 4, 5, 6]

// Remove element (immutable way)
const withoutThree = numbers.filter(n => n !== 3);  // [1, 2, 4, 5]

// Update element (immutable way)
const withUpdated = numbers.map(n => n === 1 ? 99 : n);  // [99, 2, 3, 4, 5]

console.log(numbers);  // [1, 2, 3, 4, 5] - Original unchanged!
```

**Go Examples:**

```go
package main

import "fmt"

// Go structs are copied by value, but slices/maps are not

// ❌ Mutable approach with slice
func addMutable(numbers []int, value int) []int {
    numbers = append(numbers, value)  // Might affect original
    return numbers
}

// ✅ Immutable approach
func addImmutable(numbers []int, value int) []int {
    // Create new slice
    result := make([]int, len(numbers)+1)
    copy(result, numbers)
    result[len(numbers)] = value
    return result
}

// ❌ Mutable struct approach
type User struct {
    Name  string
    Email string
}

func updateEmailMutable(user *User, email string) {
    user.Email = email  // Modifies original
}

// ✅ Immutable struct approach
func updateEmailImmutable(user User, email string) User {
    return User{
        Name:  user.Name,
        Email: email,  // Returns new struct
    }
}

func main() {
    // Slice immutability
    numbers := []int{1, 2, 3}
    newNumbers := addImmutable(numbers, 4)
    
    fmt.Println(numbers)    // [1, 2, 3] - original
    fmt.Println(newNumbers) // [1, 2, 3, 4] - new slice
    
    // Struct immutability
    user := User{Name: "Alice", Email: "alice@email.com"}
    updatedUser := updateEmailImmutable(user, "alice@newemail.com")
    
    fmt.Println(user.Email)        // alice@email.com - original
    fmt.Println(updatedUser.Email) // alice@newemail.com - new
}
```

</details>

#### Immutable Operations Cheat Sheet:

| Operation | Mutable ❌ | Immutable ✅ |
|-----------|-----------|-------------|
| Add to array | `arr.push(x)` | `[...arr, x]` |
| Remove from array | `arr.splice(i, 1)` | `arr.filter((_, idx) => idx !== i)` |
| Update array element | `arr[i] = x` | `arr.map((v, idx) => idx === i ? x : v)` |
| Update object property | `obj.prop = x` | `{...obj, prop: x}` |
| Merge objects | `Object.assign(obj, x)` | `{...obj, ...x}` |
| Add to set | `set.add(x)` | `new Set([...set, x])` |

#### Interview Tip:
"Immutability prevents bugs from unexpected data changes. Instead of modifying data, we create new copies. It's like preferring 'Save As' over 'Save' - you keep the original safe!"

---

### 3. Higher-Order Functions 🎯
**"Functions that take functions as input or return functions as output"**

#### Real-Life Analogy
A **coffee shop customizer**: The shop (higher-order function) takes your preferences (function parameter) and returns a customized coffee maker (function return). You give it "make it strong" and "add milk," and it returns a coffee maker configured your way.

#### Definition
Higher-order functions (HOF) are functions that:
1. Take one or more functions as arguments, OR
2. Return a function as result

#### Why Higher-Order Functions?
- ✅ **Code reuse** - Abstract common patterns
- ✅ **Composition** - Build complex behavior from simple pieces
- ✅ **Flexibility** - Behavior can be customized
- ✅ **Declarative** - Express what, not how

<details>
<summary><strong>View contents</strong></summary>

#### Examples

**JavaScript/TypeScript:**

```typescript
// 1. Function that takes a function as argument
function applyOperation(x: number, y: number, operation: (a: number, b: number) => number): number {
  return operation(x, y);
}

const add = (a: number, b: number) => a + b;
const multiply = (a: number, b: number) => a * b;

console.log(applyOperation(5, 3, add));       // 8
console.log(applyOperation(5, 3, multiply));  // 15

// 2. Function that returns a function
function makeMultiplier(factor: number): (x: number) => number {
  return (x: number) => x * factor;
}

const double = makeMultiplier(2);
const triple = makeMultiplier(3);

console.log(double(5));  // 10
console.log(triple(5));  // 15

// 3. Function that does both
function makeLogger(prefix: string): (message: string) => void {
  return (message: string) => {
    console.log(`[${prefix}] ${message}`);
  };
}

const errorLogger = makeLogger("ERROR");
const infoLogger = makeLogger("INFO");

errorLogger("Something went wrong");  // [ERROR] Something went wrong
infoLogger("Process completed");      // [INFO] Process completed

// Real-world example: Event handlers
type EventHandler = (event: Event) => void;

function debounce(fn: EventHandler, delay: number): EventHandler {
  let timeoutId: NodeJS.Timeout;
  
  return (event: Event) => {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn(event), delay);
  };
}

const handleSearch = (event: Event) => {
  console.log("Searching...");
};

// Debounced version waits 300ms after last call
const debouncedSearch = debounce(handleSearch, 300);

// Common HOFs: map, filter, reduce
const numbers = [1, 2, 3, 4, 5];

// map: transform each element
const doubled = numbers.map(n => n * 2);  // [2, 4, 6, 8, 10]

// filter: select elements
const evens = numbers.filter(n => n % 2 === 0);  // [2, 4]

// reduce: combine elements
const sum = numbers.reduce((acc, n) => acc + n, 0);  // 15
```

**Go Examples:**

```go
package main

import "fmt"

// 1. Function that takes a function as argument
func applyOperation(x, y int, operation func(int, int) int) int {
    return operation(x, y)
}

func main() {
    // Anonymous functions as arguments
    sum := applyOperation(5, 3, func(a, b int) int {
        return a + b
    })
    
    product := applyOperation(5, 3, func(a, b int) int {
        return a * b
    })
    
    fmt.Println(sum)     // 8
    fmt.Println(product) // 15
    
    // 2. Function that returns a function
    double := makeMultiplier(2)
    triple := makeMultiplier(3)
    
    fmt.Println(double(5)) // 10
    fmt.Println(triple(5)) // 15
    
    // 3. Real-world example: Middleware pattern
    handler := authenticate(authorize(handleRequest))
    handler("sensitive data")
}

func makeMultiplier(factor int) func(int) int {
    return func(x int) int {
        return x * factor
    }
}

// Middleware pattern (common in web servers)
type Handler func(string)

func authenticate(next Handler) Handler {
    return func(data string) {
        fmt.Println("Authenticating...")
        next(data)
    }
}

func authorize(next Handler) Handler {
    return func(data string) {
        fmt.Println("Authorizing...")
        next(data)
    }
}

func handleRequest(data string) {
    fmt.Println("Handling:", data)
}

// Custom map function for slices
func mapInt(slice []int, fn func(int) int) []int {
    result := make([]int, len(slice))
    for i, v := range slice {
        result[i] = fn(v)
    }
    return result
}

// Custom filter function
func filterInt(slice []int, fn func(int) bool) []int {
    result := []int{}
    for _, v := range slice {
        if fn(v) {
            result = append(result, v)
        }
    }
    return result
}

// Custom reduce function
func reduceInt(slice []int, fn func(int, int) int, initial int) int {
    result := initial
    for _, v := range slice {
        result = fn(result, v)
    }
    return result
}
```

#### Built-in Higher-Order Functions:

**JavaScript/TypeScript:**
```typescript
const users = [
  { name: "Alice", age: 25, active: true },
  { name: "Bob", age: 30, active: false },
  { name: "Charlie", age: 35, active: true }
];

// map: transform
const names = users.map(u => u.name);
// ["Alice", "Bob", "Charlie"]

// filter: select
const activeUsers = users.filter(u => u.active);
// [{name: "Alice", age: 25, active: true}, {name: "Charlie", age: 35, active: true}]

// reduce: aggregate
const totalAge = users.reduce((sum, u) => sum + u.age, 0);
// 90

// find: get first match
const bob = users.find(u => u.name === "Bob");
// {name: "Bob", age: 30, active: false}

// some: check if any match
const hasInactive = users.some(u => !u.active);
// true

// every: check if all match
const allActive = users.every(u => u.active);
// false

// sort: order (with custom comparator)
const byAge = [...users].sort((a, b) => a.age - b.age);
// Sorted by age
```

</details>

#### Interview Tip:
"Higher-order functions are the building blocks of functional programming. They let us abstract patterns like 'do this to each item' (map) or 'keep only items matching this' (filter). In OOP we use classes and objects; in FP we use higher-order functions."

---

### 4. Function Composition 🔗
**"Combining simple functions to build complex ones"**

#### Real-Life Analogy
**Assembly line**: Each station (function) does one simple task. The final product is the result of data flowing through all stations in sequence. f(g(h(x))) is like: Raw material → Cut → Polish → Package → Final product.

#### Definition
Function composition is the process of combining two or more functions to produce a new function. The output of one function becomes the input of the next.

#### Mathematical Notation:
```
(f ∘ g)(x) = f(g(x))
```

<details>
<summary><strong>View contents</strong></summary>

#### Examples

**TypeScript:**

```typescript
// Simple functions
const addOne = (x: number): number => x + 1;
const double = (x: number): number => x * 2;
const square = (x: number): number => x * x;

// Manual composition (right to left)
const result1 = square(double(addOne(5)));  // ((5 + 1) * 2)² = 144

// Compose utility function (right to left execution)
function compose<T>(...fns: Array<(arg: T) => T>): (arg: T) => T {
  return (arg: T) => fns.reduceRight((acc, fn) => fn(acc), arg);
}

// Pipe utility function (left to right execution - more intuitive)
function pipe<T>(...fns: Array<(arg: T) => T>): (arg: T) => T {
  return (arg: T) => fns.reduce((acc, fn) => fn(acc), arg);
}

// Using compose (reads right to left)
const composedFn = compose(square, double, addOne);
console.log(composedFn(5));  // 144

// Using pipe (reads left to right)
const pipedFn = pipe(addOne, double, square);
console.log(pipedFn(5));  // 144

// Real-world example: Data transformation pipeline
interface User {
  firstName: string;
  lastName: string;
  age: number;
}

const getFullName = (user: User): string => 
  `${user.firstName} ${user.lastName}`;

const toUpperCase = (str: string): string => 
  str.toUpperCase();

const addGreeting = (name: string): string => 
  `Hello, ${name}!`;

// Compose multiple transformations
const greetUser = pipe(
  getFullName,
  toUpperCase,
  addGreeting
);

const user: User = { firstName: "Alice", lastName: "Smith", age: 25 };
console.log(greetUser(user));  // "Hello, ALICE SMITH!"

// Another example: String processing
const trim = (str: string): string => str.trim();
const toLowerCase = (str: string): string => str.toLowerCase();
const removeSpaces = (str: string): string => str.replace(/\s+/g, '-');

const slugify = pipe(
  trim,
  toLowerCase,
  removeSpaces
);

console.log(slugify("  Hello World Example  "));  // "hello-world-example"

// Example: Number validation pipeline
const isPositive = (n: number): boolean => n > 0;
const isInteger = (n: number): boolean => Number.isInteger(n);
const isLessThan100 = (n: number): boolean => n < 100;

// Compose predicates
function allPredicates<T>(...predicates: Array<(arg: T) => boolean>): (arg: T) => boolean {
  return (arg: T) => predicates.every(pred => pred(arg));
}

const isValidScore = allPredicates(isPositive, isInteger, isLessThan100);

console.log(isValidScore(50));    // true
console.log(isValidScore(-5));    // false
console.log(isValidScore(150));   // false
console.log(isValidScore(50.5));  // false
```

**Go Examples:**

```go
package main

import (
    "fmt"
    "strings"
)

// Function composition in Go
type IntFunc func(int) int
type StringFunc func(string) string

// Compose for integer functions (right to left)
func composeInt(fns ...IntFunc) IntFunc {
    return func(x int) int {
        result := x
        // Apply functions in reverse order
        for i := len(fns) - 1; i >= 0; i-- {
            result = fns[i](result)
        }
        return result
    }
}

// Pipe for integer functions (left to right)
func pipeInt(fns ...IntFunc) IntFunc {
    return func(x int) int {
        result := x
        for _, fn := range fns {
            result = fn(result)
        }
        return result
    }
}

// Pipe for string functions
func pipeString(fns ...StringFunc) StringFunc {
    return func(s string) string {
        result := s
        for _, fn := range fns {
            result = fn(result)
        }
        return result
    }
}

func main() {
    // Simple functions
    addOne := func(x int) int { return x + 1 }
    double := func(x int) int { return x * 2 }
    square := func(x int) int { return x * x }
    
    // Using pipe
    pipedFn := pipeInt(addOne, double, square)
    fmt.Println(pipedFn(5))  // 144
    
    // String processing pipeline
    trim := func(s string) string { return strings.TrimSpace(s) }
    lower := func(s string) string { return strings.ToLower(s) }
    removeSpaces := func(s string) string { return strings.ReplaceAll(s, " ", "-") }
    
    slugify := pipeString(trim, lower, removeSpaces)
    fmt.Println(slugify("  Hello World Example  "))  // "hello-world-example"
    
    // Real-world example: Request processing
    processRequest := pipeString(
        validateInput,
        sanitizeInput,
        transformInput,
    )
    
    result := processRequest("  <script>alert('xss')</script>  ")
    fmt.Println(result)
}

func validateInput(s string) string {
    if len(s) == 0 {
        return "ERROR: Empty input"
    }
    return s
}

func sanitizeInput(s string) string {
    // Remove HTML tags
    s = strings.ReplaceAll(s, "<", "")
    s = strings.ReplaceAll(s, ">", "")
    return s
}

func transformInput(s string) string {
    return strings.TrimSpace(strings.ToLower(s))
}
```

#### Point-Free Style (Tacit Programming):

```typescript
// With explicit parameter
const doubleNumbers = (numbers: number[]) => numbers.map(n => n * 2);

// Point-free style (no explicit parameter)
const doubleNumbersPointFree = (numbers: number[]) => numbers.map(double);

// Another example
const getUserNames = (users: User[]) => users.map(u => u.firstName);

// Point-free with reusable function
const getName = (user: User) => user.firstName;
const getUserNamesPointFree = (users: User[]) => users.map(getName);
```

</details>

#### Benefits of Composition:
1. **Modularity** - Each function does one thing
2. **Reusability** - Small functions can be reused everywhere
3. **Testability** - Test small functions independently
4. **Readability** - Clear data flow (especially with pipe)
5. **Maintainability** - Easy to add/remove steps

#### Interview Tip:
"Function composition is like LEGO blocks - build complex behavior from simple, reusable pieces. Pipe reads naturally left-to-right like data flowing through a pipeline. It's the essence of 'Don't Repeat Yourself' in FP."

---

### 5. Declarative vs Imperative 📝
**"Say WHAT you want, not HOW to do it"**

#### Real-Life Analogy
**Imperative** (How): "Go to the kitchen, open the fridge, take out eggs, crack them in a bowl, beat them, pour into pan, stir until cooked."

**Declarative** (What): "Make me scrambled eggs."

#### Definition
- **Imperative**: Explicitly describe HOW to do something (step-by-step instructions)
- **Declarative**: Describe WHAT you want (desired outcome)

<details>
<summary><strong>View contents</strong></summary>

#### Examples

**❌ Imperative Style:**

```typescript
// Imperative: Manual loops, mutations, step-by-step
const numbers = [1, 2, 3, 4, 5];

// Get sum of even numbers doubled
let result = 0;
for (let i = 0; i < numbers.length; i++) {
  if (numbers[i] % 2 === 0) {
    result += numbers[i] * 2;
  }
}
console.log(result);  // 12

// Get names of active users
interface User {
  name: string;
  active: boolean;
}

const users: User[] = [
  { name: "Alice", active: true },
  { name: "Bob", active: false },
  { name: "Charlie", active: true }
];

const activeNames: string[] = [];
for (let i = 0; i < users.length; i++) {
  if (users[i].active) {
    activeNames.push(users[i].name);
  }
}
console.log(activeNames);  // ["Alice", "Charlie"]
```

**✅ Declarative Style:**

```typescript
// Declarative: Express what you want
const numbers = [1, 2, 3, 4, 5];

// Get sum of even numbers doubled
const result = numbers
  .filter(n => n % 2 === 0)    // Keep evens
  .map(n => n * 2)             // Double them
  .reduce((sum, n) => sum + n, 0);  // Sum them
  
console.log(result);  // 12

// Get names of active users
const activeNames = users
  .filter(u => u.active)
  .map(u => u.name);
  
console.log(activeNames);  // ["Alice", "Charlie"]
```

**Go Examples:**

```go
package main

import "fmt"

func main() {
    numbers := []int{1, 2, 3, 4, 5}
    
    // ❌ Imperative style
    result := 0
    for i := 0; i < len(numbers); i++ {
        if numbers[i]%2 == 0 {
            result += numbers[i] * 2
        }
    }
    fmt.Println(result)  // 12
    
    // ✅ Declarative style (with helper functions)
    evens := filter(numbers, isEven)
    doubled := mapInt(evens, double)
    sum := reduce(doubled, add, 0)
    
    fmt.Println(sum)  // 12
    
    // Even more declarative with composition
    result2 := pipe(
        filterEvens,
        doubleAll,
        sumAll,
    )(numbers)
    
    fmt.Println(result2)  // 12
}

// Helper functions
func isEven(n int) bool {
    return n%2 == 0
}

func double(n int) int {
    return n * 2
}

func add(a, b int) int {
    return a + b
}

// Declarative higher-order functions
func filter(slice []int, predicate func(int) bool) []int {
    result := []int{}
    for _, v := range slice {
        if predicate(v) {
            result = append(result, v)
        }
    }
    return result
}

func mapInt(slice []int, fn func(int) int) []int {
    result := make([]int, len(slice))
    for i, v := range slice {
        result[i] = fn(v)
    }
    return result
}

func reduce(slice []int, fn func(int, int) int, initial int) int {
    result := initial
    for _, v := range slice {
        result = fn(result, v)
    }
    return result
}

// Composed operations
func filterEvens(numbers []int) []int {
    return filter(numbers, isEven)
}

func doubleAll(numbers []int) []int {
    return mapInt(numbers, double)
}

func sumAll(numbers []int) int {
    return reduce(numbers, add, 0)
}

// Pipe for slice transformations
func pipe(fns ...func([]int) any) func([]int) any {
    return func(input []int) any {
        var result any = input
        for _, fn := range fns {
            // Type assertion based on function signature
            switch v := result.(type) {
            case []int:
                result = fn(v)
            }
        }
        return result
    }
}
```

#### Real-World Comparisons:

```typescript
// Example 1: DOM Manipulation

// ❌ Imperative
const div = document.createElement('div');
div.className = 'container';
const p = document.createElement('p');
p.textContent = 'Hello World';
div.appendChild(p);
document.body.appendChild(div);

// ✅ Declarative (React JSX)
<div className="container">
  <p>Hello World</p>
</div>

// Example 2: Data Fetching

// ❌ Imperative
let data;
let error;
let loading = true;

fetch('/api/users')
  .then(response => response.json())
  .then(json => {
    data = json;
    loading = false;
  })
  .catch(err => {
    error = err;
    loading = false;
  });

// ✅ Declarative (React Query / SWR)
const { data, error, loading } = useQuery('/api/users');

// Example 3: State Management

// ❌ Imperative
function updateCart(cart, action) {
  if (action.type === 'ADD_ITEM') {
    cart.items.push(action.item);
    cart.total += action.item.price;
  } else if (action.type === 'REMOVE_ITEM') {
    const index = cart.items.findIndex(i => i.id === action.itemId);
    cart.total -= cart.items[index].price;
    cart.items.splice(index, 1);
  }
  return cart;
}

// ✅ Declarative (Redux-style reducer)
function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      return {
        ...state,
        items: [...state.items, action.item],
        total: state.total + action.item.price
      };
    case 'REMOVE_ITEM':
      return {
        ...state,
        items: state.items.filter(i => i.id !== action.itemId),
        total: state.total - action.item.price
      };
    default:
      return state;
  }
}
```

</details>

#### Interview Tip:
"Declarative code focuses on WHAT, not HOW. It's like SQL: you say 'SELECT users WHERE active = true' not 'loop through users, check each one...' Declarative code is more readable, maintainable, and less bug-prone."

---

## Advanced FP Concepts

### 1. Currying 🍛
**"Transform a function with multiple arguments into a sequence of functions with single arguments"**

#### Real-Life Analogy
**Coffee shop order form**: Instead of one big form (name, size, type, sugar, milk), you have a series of questions: "Name?" → "Size?" → "Type?" → etc. Each answer leads to the next question.

#### Definition
Currying converts a function that takes multiple arguments into a sequence of functions that each take a single argument.

```
f(a, b, c) → f(a)(b)(c)
```

<details>
<summary><strong>View contents</strong></summary>

#### Examples

**TypeScript:**

```typescript
// Regular function
function add(a: number, b: number, c: number): number {
  return a + b + c;
}
console.log(add(1, 2, 3));  // 6

// Curried version
function addCurried(a: number) {
  return function(b: number) {
    return function(c: number) {
      return a + b + c;
    };
  };
}

console.log(addCurried(1)(2)(3));  // 6

// Arrow function syntax (cleaner)
const addCurriedArrow = (a: number) => (b: number) => (c: number) => a + b + c;
console.log(addCurriedArrow(1)(2)(3));  // 6

// Generic curry utility
function curry<A, B, C, R>(fn: (a: A, b: B, c: C) => R) {
  return (a: A) => (b: B) => (c: C) => fn(a, b, c);
}

const curriedAdd = curry(add);
console.log(curriedAdd(1)(2)(3));  // 6

// Practical example: Partial application
const multiply = (a: number) => (b: number) => a * b;

const double = multiply(2);     // Partially applied
const triple = multiply(3);     // Partially applied

console.log(double(5));  // 10
console.log(triple(5));  // 15

// Real-world example: Logging with context
const log = (level: string) => (module: string) => (message: string) => {
  console.log(`[${level}] [${module}] ${message}`);
};

const errorLog = log('ERROR');
const errorLogAuth = errorLog('AUTH');

errorLogAuth('Invalid credentials');  // [ERROR] [AUTH] Invalid credentials
errorLogAuth('Session expired');      // [ERROR] [AUTH] Session expired

const infoLog = log('INFO');
const infoLogDB = infoLog('DATABASE');

infoLogDB('Connection established');  // [INFO] [DATABASE] Connection established

// Example: Event handlers with configuration
const addEventListener = (event: string) => (element: HTMLElement) => (handler: EventListener) => {
  element.addEventListener(event, handler);
};

const onClick = addEventListener('click');
const onClickButton = onClick(document.querySelector('button')!);

onClickButton((e) => console.log('Button clicked!'));

// Example: API calls with configuration
const makeApiCall = (baseUrl: string) => (endpoint: string) => (params: object) => {
  return fetch(`${baseUrl}${endpoint}?${new URLSearchParams(params as any)}`);
};

const apiV1 = makeApiCall('https://api.example.com/v1');
const getUsers = apiV1('/users');

getUsers({ active: true, limit: 10 });  // Fetch active users
```

**Go Examples:**

```go
package main

import "fmt"

// Currying in Go
func add(a int) func(int) func(int) int {
    return func(b int) func(int) int {
        return func(c int) int {
            return a + b + c
        }
    }
}

func multiply(a int) func(int) int {
    return func(b int) int {
        return a * b
    }
}

func main() {
    // Using curried function
    result := add(1)(2)(3)
    fmt.Println(result)  // 6
    
    // Partial application
    double := multiply(2)
    triple := multiply(3)
    
    fmt.Println(double(5))  // 10
    fmt.Println(triple(5))  // 15
    
    // Practical example: Logger
    errorLogAuth := log("ERROR")("AUTH")
    errorLogAuth("Invalid credentials")  // [ERROR] [AUTH] Invalid credentials
}

// Curried logger
func log(level string) func(string) func(string) {
    return func(module string) func(string) {
        return func(message string) {
            fmt.Printf("[%s] [%s] %s\n", level, module, message)
        }
    }
}

// Example: HTTP request builder
type RequestBuilder func(string) func(map[string]string) string

func makeRequest(method string) func(string) func(map[string]string) string {
    return func(url string) func(map[string]string) string {
        return func(headers map[string]string) string {
            return fmt.Sprintf("%s %s with headers: %v", method, url, headers)
        }
    }
}

func exampleUsage() {
    getRequest := makeRequest("GET")
    getUsersUrl := getRequest("/api/users")
    
    result := getUsersUrl(map[string]string{
        "Authorization": "Bearer token123",
        "Content-Type":  "application/json",
    })
    
    fmt.Println(result)
}
```

</details>

#### Benefits of Currying:
1. **Partial Application** - Create specialized functions
2. **Configuration** - Set up function behavior in stages
3. **Reusability** - Share common configurations
4. **Function Composition** - Easier to compose single-argument functions
5. **Dependency Injection** - Inject dependencies step by step

#### Currying vs Partial Application:

<details>
<summary><strong>View contents</strong></summary>

```typescript
// Currying: Transform to single-argument functions
const curriedAdd = (a: number) => (b: number) => (c: number) => a + b + c;

// Partial Application: Fix some arguments
function partial<A, B, C, R>(
  fn: (a: A, b: B, c: C) => R,
  a: A
): (b: B, c: C) => R {
  return (b: B, c: C) => fn(a, b, c);
}

const add = (a: number, b: number, c: number) => a + b + c;
const addFive = partial(add, 5);

console.log(addFive(2, 3));  // 10
```

</details>

#### Interview Tip:
"Currying transforms f(a,b,c) into f(a)(b)(c). It's useful for creating specialized versions of functions. Think of it as a vending machine where you select category → subcategory → specific item, each step narrowing down your choice."

---

### 2. Partial Application 🧩

**"Fix some arguments of a function, creating a new function with fewer arguments"**

<details>
<summary><strong>View contents</strong></summary>

```typescript
// Partial application utility
function partial<T, U, R>(fn: (a: T, b: U) => R, a: T): (b: U) => R {
  return (b: U) => fn(a, b);
}

const multiply = (a: number, b: number) => a * b;
const multiplyBy5 = partial(multiply, 5);

console.log(multiplyBy5(10));  // 50
console.log(multiplyBy5(20));  // 100

// Real-world: Discount calculator
const applyDiscount = (discountPercent: number, price: number) => 
  price - (price * discountPercent / 100);

const apply10PercentOff = partial(applyDiscount, 10);
const apply25PercentOff = partial(applyDiscount, 25);

console.log(apply10PercentOff(100));  // 90
console.log(apply25PercentOff(100));  // 75
```

</details>

---

### 3. Memoization 💾
**"Cache function results to avoid recomputation"**

#### Real-Life Analogy
**Calculator with memory**: Instead of recalculating 25 × 25 every time, the calculator remembers the answer (625) and returns it instantly next time.

<details>
<summary><strong>View contents</strong></summary>

```typescript
// Memoization utility
function memoize<T extends any[], R>(fn: (...args: T) => R): (...args: T) => R {
  const cache = new Map<string, R>();
  
  return (...args: T): R => {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      console.log('Cache hit!');
      return cache.get(key)!;
    }
    
    console.log('Computing...');
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}

// Example: Expensive Fibonacci calculation
const fibonacci = (n: number): number => {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
};

const memoizedFib = memoize(fibonacci);

console.log(memoizedFib(40));  // Computing... (takes time)
console.log(memoizedFib(40));  // Cache hit! (instant)

// Real-world: API calls
interface User {
  id: number;
  name: string;
}

const fetchUser = async (id: number): Promise<User> => {
  console.log(`Fetching user ${id}...`);
  const response = await fetch(`/api/users/${id}`);
  return response.json();
};

const memoizedFetchUser = memoize(fetchUser);

// First call makes API request
await memoizedFetchUser(1);  // Fetching user 1...

// Second call uses cache
await memoizedFetchUser(1);  // Cache hit!
```

**Go Example:**

```go
package main

import (
    "fmt"
    "sync"
)

// Memoization in Go with generics (Go 1.18+)
func Memoize[K comparable, V any](fn func(K) V) func(K) V {
    cache := make(map[K]V)
    mu := sync.RWMutex{}
    
    return func(key K) V {
        // Check cache first (read lock)
        mu.RLock()
        if val, ok := cache[key]; ok {
            fmt.Println("Cache hit!")
            mu.RUnlock()
            return val
        }
        mu.RUnlock()
        
        // Compute and store (write lock)
        mu.Lock()
        defer mu.Unlock()
        
        // Double-check (another goroutine might have computed it)
        if val, ok := cache[key]; ok {
            return val
        }
        
        fmt.Println("Computing...")
        result := fn(key)
        cache[key] = result
        return result
    }
}

func fibonacci(n int) int {
    if n <= 1 {
        return n
    }
    return fibonacci(n-1) + fibonacci(n-2)
}

func main() {
    memoFib := Memoize(fibonacci)
    
    fmt.Println(memoFib(40))  // Computing... (takes time)
    fmt.Println(memoFib(40))  // Cache hit! (instant)
}
```

</details>

---

### 4. Functors and Monads 🎁
**Advanced topic - Common in FP languages like Haskell, Scala**

#### Functor
A container that can be mapped over.

<details>
<summary><strong>View contents</strong></summary>

```typescript
// Array is a functor - it has a map method
const numbers = [1, 2, 3];
const doubled = numbers.map(n => n * 2);  // [2, 4, 6]

// Maybe/Option functor (handles null/undefined safely)
class Maybe<T> {
  constructor(private value: T | null) {}
  
  map<U>(fn: (value: T) => U): Maybe<U> {
    if (this.value === null) {
      return new Maybe<U>(null);
    }
    return new Maybe(fn(this.value));
  }
  
  getOrElse(defaultValue: T): T {
    return this.value === null ? defaultValue : this.value;
  }
}

const maybeNumber = new Maybe(5);
const doubled2 = maybeNumber.map(n => n * 2);  // Maybe(10)

const maybeNull = new Maybe<number>(null);
const doubled3 = maybeNull.map(n => n * 2);    // Maybe(null) - doesn't crash!

console.log(doubled3.getOrElse(0));  // 0
```

</details>

---

## FP vs OOP

### Comparison Table

| Aspect | OOP | FP |
|--------|-----|-----|
| **Building Block** | Objects (data + methods) | Functions (pure transformations) |
| **State** | Encapsulated, mutable | Immutable, passed explicitly |
| **Data & Behavior** | Coupled together in classes | Separated (data structures + functions) |
| **Code Reuse** | Inheritance, polymorphism | Composition, higher-order functions |
| **Side Effects** | Acceptable, managed in methods | Avoided, isolated at boundaries |
| **Testing** | Need to mock objects/state | Easy - pure functions |
| **Concurrency** | Difficult (shared mutable state) | Easy (no shared state) |
| **Real-world modeling** | Natural (objects mirror reality) | Abstract (functions model transformations) |

### When to Use What?

**Use OOP when:**
- Modeling real-world entities (User, Order, Product)
- Building UIs with components (React classes, Angular)
- Complex state machines
- Need encapsulation and inheritance
- Working with frameworks that expect OOP (Java/Spring, C#/.NET)

**Use FP when:**
- Data transformations and pipelines
- Concurrent/parallel processing
- Mathematical computations
- Need predictability and testability
- Building reactive systems
- Working with immutable data (Redux, React hooks)

**Hybrid Approach (Most Common):**

```typescript
// OOP for structure, FP for logic
class UserService {
  // OOP: Encapsulate dependencies
  constructor(private database: Database, private logger: Logger) {}
  
  // FP: Pure function for business logic
  private calculateDiscount(user: User, amount: number): number {
    return pipe(
      getBasicDiscount,
      applyLoyaltyBonus(user.loyaltyPoints),
      applySeniorDiscount(user.age),
      capDiscount(amount * 0.5)
    )(amount);
  }
  
  // OOP: Method with side effects (controlled)
  async processOrder(user: User, order: Order): Promise<void> {
    const discount = this.calculateDiscount(user, order.total);  // FP
    const finalAmount = order.total - discount;
    
    await this.database.save({ ...order, finalAmount });  // Side effect
    this.logger.log(`Order processed for ${user.name}`);  // Side effect
  }
}
```

---

## Real-World Applications

### 1. Redux (State Management)

<details>
<summary><strong>View contents</strong></summary>

```typescript
// Pure reducer function
interface State {
  count: number;
  todos: string[];
}

type Action = 
  | { type: 'INCREMENT' }
  | { type: 'ADD_TODO'; payload: string };

// Pure function: (state, action) => newState
const reducer = (state: State, action: Action): State => {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };  // Immutable
    
    case 'ADD_TODO':
      return { ...state, todos: [...state.todos, action.payload] };  // Immutable
    
    default:
      return state;
  }
};

// Time-travel debugging possible because of pure functions!
```

</details>

### 2. React Hooks (Functional React)

<details>
<summary><strong>View contents</strong></summary>

```typescript
// Before (OOP): Class components
class Counter extends React.Component {
  state = { count: 0 };
  
  increment = () => {
    this.setState({ count: this.state.count + 1 });
  };
  
  render() {
    return <button onClick={this.increment}>{this.state.count}</button>;
  }
}

// After (FP): Function components with hooks
const Counter = () => {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
};
```

</details>

### 3. Data Processing Pipeline

<details>
<summary><strong>View contents</strong></summary>

```typescript
interface Product {
  id: number;
  name: string;
  price: number;
  category: string;
  inStock: boolean;
}

const products: Product[] = [
  { id: 1, name: "Laptop", price: 1000, category: "Electronics", inStock: true },
  { id: 2, name: "Phone", price: 500, category: "Electronics", inStock: false },
  { id: 3, name: "Desk", price: 300, category: "Furniture", inStock: true },
];

// Functional pipeline
const getElectronicsInStockUnder800 = pipe(
  (products: Product[]) => products.filter(p => p.category === "Electronics"),
  (products: Product[]) => products.filter(p => p.inStock),
  (products: Product[]) => products.filter(p => p.price < 800),
  (products: Product[]) => products.map(p => ({ name: p.name, price: p.price }))
);

const result = getElectronicsInStockUnder800(products);
// [{ name: "Phone", price: 500 }]
```

</details>

---

## Memory Tips & Tricks

<details>
<summary><strong>View contents</strong></summary>

### 1. Remember Core Concepts (PICHI)

**"Pure Is Completely Healthy Idiomatically"**
- **P**ure Functions - Same input → Same output
- **I**mmutability - Don't mutate, create new
- **C**omposition - Build complex from simple
- **H**igher-Order Functions - Functions as values
- **I**mperative → Declarative - What not how

### 2. Pure Function Test (SNES)

**"Same No External Side"**
- **S**ame output for same input
- **N**o randomness or time dependency
- **E**xternal state not used
- **S**ide effects avoided

### 3. Visual Memory Aids

```
Pure Function 🔵 = Math function (f(x) = 2x)
Immutability 🔒 = Photocopy (don't edit original)
Composition 🔗 = Assembly line (step by step)
Higher-Order 🎯 = Function factory
Currying 🍛 = Multi-step form
```

</details>

## Quick Tests

**Is it pure?**
```typescript
// Can I replace it with its return value?
const add = (a, b) => a + b;
const result = add(2, 3);
// Yes! Can replace add(2, 3) with 5 everywhere
```

**Is it immutable?**
```typescript
// Does original data change?
const arr = [1, 2, 3];
const newArr = [...arr, 4];  // arr unchanged? Yes! ✅
```

**Is it declarative?**
```typescript
// Am I saying WHAT, not HOW?
users.filter(u => u.active);  // WHAT: get active users ✅
// vs for loop (HOW: iterate, check, add to array) ❌
```

---

## Interview Questions & Answers

### Q1: What is a pure function and why is it important?

<details>
<summary><strong>View contents</strong></summary>

**Answer:**
"A pure function is a function that:
1. Always returns the same output for the same inputs (deterministic)
2. Has no side effects (doesn't modify external state)
3. Doesn't depend on external state

```typescript
// ❌ Impure
let tax = 0.1;
function calculateTotal(price: number): number {
  return price + (price * tax);  // Depends on external 'tax'
}

// ✅ Pure
function calculateTotal(price: number, taxRate: number): number {
  return price + (price * taxRate);  // All inputs provided
}
```

**Why it's important:**
- **Testable**: No setup/teardown needed
- **Predictable**: Same input = same output
- **Cacheable**: Can memoize results
- **Parallelizable**: Safe to run concurrently
- **Debuggable**: Easy to reproduce bugs
- **Composable**: Can combine reliably

Think of it like a vending machine - you always get the same candy for the same button press, and it doesn't affect anything else in the world."

</details>

---

### Q2: Explain the difference between map, filter, and reduce

<details>
<summary><strong>View contents</strong></summary>

**Answer:**

```typescript
const numbers = [1, 2, 3, 4, 5];

// map: Transform each element (1-to-1 mapping)
const doubled = numbers.map(n => n * 2);
// [2, 4, 6, 8, 10] - same length, transformed values

// filter: Select elements (keep or discard)
const evens = numbers.filter(n => n % 2 === 0);
// [2, 4] - fewer elements (or same), values unchanged

// reduce: Combine into single value (aggregation)
const sum = numbers.reduce((acc, n) => acc + n, 0);
// 15 - single value
```

**When to use:**
- **map**: "Do this to every item" (transform)
- **filter**: "Keep only items that match" (select)
- **reduce**: "Combine all items into one result" (aggregate)

**Real-world:**
- map: Convert prices from USD to EUR
- filter: Get users over 18 years old
- reduce: Calculate shopping cart total"

</details>

---

### Q3: What is function composition and why use it?

<details>
<summary><strong>View contents</strong></summary>

**Answer:**
"Function composition is combining simple functions to create complex ones. The output of one function becomes the input of the next.

```typescript
// Math notation: (f ∘ g)(x) = f(g(x))

const addOne = (x: number) => x + 1;
const double = (x: number) => x * 2;
const square = (x: number) => x * x;

// Manual composition
const result = square(double(addOne(5)));  // ((5+1)*2)² = 144

// Using pipe (more readable, left-to-right)
const transform = pipe(addOne, double, square);
console.log(transform(5));  // 144
```

**Why use it:**
1. **Modularity**: Each function does one thing
2. **Reusability**: Small functions used everywhere
3. **Testability**: Test each piece independently
4. **Readability**: Clear data flow
5. **Maintainability**: Easy to add/remove steps

Real-world example: Processing user input
```typescript
const validateUser = pipe(
  trim,           // Remove whitespace
  toLowerCase,    // Normalize case
  removeSpecialChars,  // Sanitize
  validateEmail   // Check format
);
```

It's like an assembly line - each station does one simple task, and you get a complex product at the end!"

</details>

---

### Q4: What's the difference between imperative and declarative programming?

<details>
<summary><strong>View contents</strong></summary>

**Answer:**
"**Imperative**: You tell the computer HOW to do something (step-by-step instructions)
**Declarative**: You tell the computer WHAT you want (desired outcome)

```typescript
const numbers = [1, 2, 3, 4, 5];

// ❌ Imperative: HOW to get even numbers
const evens = [];
for (let i = 0; i < numbers.length; i++) {
  if (numbers[i] % 2 === 0) {
    evens.push(numbers[i]);
  }
}

// ✅ Declarative: WHAT we want
const evens = numbers.filter(n => n % 2 === 0);
```

**Real-life analogy:**
- Imperative: "Turn left, go 3 blocks, turn right, go 2 blocks..."
- Declarative: "Take me to the library"

**Benefits of declarative:**
- More readable (intent is clear)
- Less code
- Fewer bugs (no loop indices to get wrong)
- Easier to maintain
- Easier to optimize (compiler/engine handles HOW)

SQL is declarative - you say `SELECT * FROM users WHERE active = true`, not how to loop through users!"

</details>

---

### Q5: Explain currying and when to use it

<details>
<summary><strong>View contents</strong></summary>

**Answer:**
"Currying transforms a function that takes multiple arguments into a sequence of functions that each take one argument.

```typescript
// Regular function
const add = (a: number, b: number, c: number) => a + b + c;
add(1, 2, 3);  // 6

// Curried function
const addCurried = (a: number) => (b: number) => (c: number) => a + b + c;
addCurried(1)(2)(3);  // 6

// Practical benefit: Partial application
const multiply = (a: number) => (b: number) => a * b;

const double = multiply(2);  // Specialized function
const triple = multiply(3);  // Specialized function

console.log(double(5));  // 10
console.log(triple(5));  // 15
```

**When to use:**
1. **Configuration**: Set up function behavior in stages
```typescript
const log = (level: string) => (module: string) => (message: string) => {
  console.log(`[${level}] [${module}] ${message}`);
};

const errorLog = log('ERROR');
const errorLogAuth = errorLog('AUTH');
errorLogAuth('Login failed');  // [ERROR] [AUTH] Login failed
```

2. **Function composition**: Easier to compose single-argument functions
3. **Dependency injection**: Inject dependencies step by step
4. **Code reuse**: Create specialized versions

Think of it like a vending machine where you select category → type → specific item. Each choice narrows down the options!"

</details>

---

### Q6: Why is immutability important in functional programming?

<details>
<summary><strong>View contents</strong></summary>

**Answer:**
"Immutability means once data is created, it cannot be changed. Instead, you create new data with desired changes.

```typescript
// ❌ Mutable (bad)
const user = { name: 'Alice', age: 25 };
user.age = 26;  // Modifies original!

// ✅ Immutable (good)
const user = { name: 'Alice', age: 25 };
const updatedUser = { ...user, age: 26 };  // New object
// user is still { name: 'Alice', age: 25 }
```

**Why it matters:**
1. **Predictability**: Data can't change unexpectedly
2. **Debugging**: Easier to track down where changes occur
3. **Time-travel**: Can keep history of all states (great for undo/redo)
4. **Thread safety**: Multiple threads can't corrupt data
5. **Caching**: Can safely cache data since it never changes
6. **Optimization**: React/Redux can detect changes by reference equality

**Real-world example: Redux**
```typescript
// Redux enforces immutability
const reducer = (state, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };  // New state
    default:
      return state;
  }
};
```

Think of it like photocopying - when you need changes, make a copy and edit that, keeping the original safe!"

</details>

---

### Q7: What's the difference between currying and partial application?

<details>
<summary><strong>View contents</strong></summary>

**Answer:**
"Both create specialized functions, but differently:

**Currying**: Transforms multi-argument function into sequence of single-argument functions
```typescript
// Curried: Always one argument at a time
const add = (a: number) => (b: number) => (c: number) => a + b + c;
add(1)(2)(3);  // Must call with one arg each time
```

**Partial Application**: Fixes some arguments, returns function expecting the rest
```typescript
// Partial: Can fix any number of arguments
function partial(fn, ...fixedArgs) {
  return (...remainingArgs) => fn(...fixedArgs, ...remainingArgs);
}

const add = (a, b, c) => a + b + c;
const addFive = partial(add, 5);  // Fix first argument
addFive(2, 3);  // 10 (provides remaining 2 arguments)
```

**Key difference:**
- Currying: f(a,b,c) → f(a)(b)(c) - always one arg
- Partial: f(a,b,c) with a=5 → f(b,c) - any number of args

**When to use:**
- Currying: When you want maximum flexibility and composition
- Partial: When you know upfront which arguments to fix

Both achieve similar goals - creating reusable, specialized functions!"

</details>

---

### Q8: How does functional programming help with concurrency?

<details>
<summary><strong>View contents</strong></summary>

**Answer:**
"FP makes concurrent programming safer through immutability and pure functions.

**The Problem in OOP:**
```typescript
// Shared mutable state = race conditions
class Counter {
  private count = 0;
  
  increment() {
    this.count++;  // Multiple threads = data corruption!
  }
}
```

**FP Solution:**
```typescript
// Immutable state + pure functions = thread-safe
const increment = (count: number): number => count + 1;

// Each thread works with its own copy
// No shared mutable state = no race conditions
```

**Real-world benefits:**
1. **No race conditions**: Immutable data can't be corrupted
2. **Easy parallelization**: Pure functions safe to run anywhere
3. **Simpler reasoning**: No locks, mutexes, or synchronization
4. **Better performance**: Can parallelize without fear

**Example: Processing large dataset**
```go
// Go example: Parallel map
func parallelMap(data []int, fn func(int) int) []int {
    result := make([]int, len(data))
    var wg sync.WaitGroup
    
    for i, v := range data {
        wg.Add(1)
        go func(index, value int) {
            result[index] = fn(value)  // Safe - no shared mutable state
            wg.Done()
        }(i, v)
    }
    
    wg.Wait()
    return result
}
```

Think of it like factory workers each with their own tools (immutable data) vs. fighting over shared tools (mutable state)!"

</details>

---

### Q9: What are side effects and why avoid them?

<details>
<summary><strong>View contents</strong></summary>

**Answer:**
"A side effect is when a function does more than just return a value - it changes something outside its scope.

**Common side effects:**
```typescript
// 1. Modifying external variables
let count = 0;
function increment() {
  count++;  // Side effect!
}

// 2. I/O operations
function save(data) {
  console.log(data);        // Side effect: logging
  localStorage.set(data);   // Side effect: storage
  fetch('/api', data);      // Side effect: network
}

// 3. Mutating input parameters
function addItem(array, item) {
  array.push(item);  // Side effect: modifies input!
}

// 4. Random or time-dependent
function getRandom() {
  return Math.random();  // Side effect: non-deterministic
}
```

**Why avoid them:**
1. **Unpredictable**: Same input, different outcomes
2. **Hard to test**: Need to set up external state
3. **Hard to debug**: Bug could be anywhere in call chain
4. **Not composable**: Can't combine reliably
5. **Not parallelizable**: Race conditions

**How to handle:**
```typescript
// ✅ Push side effects to boundaries
// Pure core, impure shell

// Pure logic
const calculateOrder = (items, tax) => {
  const subtotal = items.reduce((sum, item) => sum + item.price, 0);
  const total = subtotal + (subtotal * tax);
  return { subtotal, total, items };
};

// Side effects at the boundary
async function processOrder(items) {
  const order = calculateOrder(items, 0.08);  // Pure
  
  // Side effects isolated here
  await db.save(order);
  await sendEmail(order);
  console.log('Order processed');
  
  return order;
}
```

The goal isn't zero side effects (impossible!) but to isolate them and keep core logic pure!"

</details>

---

### Q10: When would you choose FP over OOP?

<details>
<summary><strong>View contents</strong></summary>

**Answer:**
"Neither is universally better - choose based on the problem:

**Choose FP when:**
1. **Data transformations**: ETL pipelines, data processing
```typescript
const processData = pipe(
  parseCSV,
  filterInvalid,
  transformRecords,
  aggregateResults
);
```

2. **Concurrency**: Multiple threads processing data
3. **Mathematical computations**: Finance, statistics, ML
4. **Testing critical**: Need high test coverage
5. **State management**: Redux, event sourcing
6. **Reactive systems**: Streams, observables

**Choose OOP when:**
1. **Domain modeling**: Users, Orders, Products
```typescript
class Order {
  constructor(private items: Item[]) {}
  calculateTotal() { ... }
  applyDiscount() { ... }
}
```

2. **UI components**: React class components, Angular
3. **Complex state machines**: Game engines, workflows
4. **Need encapsulation**: Hide implementation details
5. **Framework expectations**: Spring, .NET, Django

**Hybrid Approach (Best Practice):**
```typescript
// OOP for structure
class OrderService {
  constructor(private db: Database) {}  // Encapsulation
  
  // FP for logic
  async processOrder(order: Order) {
    const validated = pipe(
      validateItems,
      calculateTotals,
      applyDiscounts
    )(order);  // Pure functions
    
    await this.db.save(validated);  // Side effect
  }
}
```

**My recommendation**: Use OOP for architecture and structure, FP for business logic and data transformations. Modern codebases use both - React uses FP (hooks) but has OOP (components), Redux is pure FP, etc."

</details>

---

## Common FP Mistakes to Avoid

### ❌ Mistake 1: Thinking FP means no loops

<details>
<summary><strong>View contents</strong></summary>

```typescript
// Wrong thinking: "FP = no for loops ever"
// Reality: Prefer map/filter/reduce, but loops are fine for performance-critical code

// ✅ This is fine in FP
for (let i = 0; i < 1000000; i++) {
  // Performance-critical operation
}
```

</details>

### ❌ Mistake 2: Over-engineering with FP concepts

<details>
<summary><strong>View contents</strong></summary>

```typescript
// ❌ Too much abstraction
const add = curry((a, b) => a + b);
const increment = compose(add(1));
const result = pipe(increment)(5);

// ✅ Simple and clear
const result = 5 + 1;
```

</details>

### ❌ Mistake 3: Ignoring performance of immutability

<details>
<summary><strong>View contents</strong></summary>

```typescript
// ❌ Inefficient: Creates new array each iteration
let result = [];
for (let i = 0; i < 10000; i++) {
  result = [...result, i];  // O(n²) time!
}

// ✅ Better: Use mutable structure during build, return immutable
const result = [];
for (let i = 0; i < 10000; i++) {
  result.push(i);
}
return Object.freeze(result);  // Make immutable when done
```

</details>

### ❌ Mistake 4: Not isolating side effects

<details>
<summary><strong>View contents</strong></summary>

```typescript
// ❌ Side effects mixed with logic
function processUser(user) {
  console.log('Processing:', user.name);  // Side effect
  const validated = validateUser(user);
  db.save(validated);  // Side effect
  return validated;
}

// ✅ Pure logic, side effects at boundary
function processUser(user) {
  return pipe(
    validateUser,
    normalizeData,
    enrichUser
  )(user);  // Pure
}

// Call site handles side effects
const processed = processUser(user);
console.log('Processed:', processed.name);
db.save(processed);
```

</details>

---

## Language-Specific Notes

### TypeScript/JavaScript
- ✅ First-class functions
- ✅ Closures
- ✅ Array methods (map, filter, reduce)
- ⚠️ Not purely functional (allows mutation)
- ⚠️ `this` binding can be confusing
- Libraries: Ramda, Lodash/FP, Immutable.js

### Go
- ✅ First-class functions
- ✅ Closures
- ❌ No built-in map/filter/reduce
- ❌ No generics for FP utilities (pre-1.18)
- ✅ Excellent for concurrent FP (goroutines)
- Style: Use FP concepts but don't force it

### Python
- ✅ First-class functions
- ✅ Built-in map/filter/reduce
- ✅ List comprehensions
- ⚠️ Not enforced (allows mutation)
- Libraries: toolz, fn.py

### Haskell/Scala/Clojure
- ✅ Pure functional languages
- ✅ Enforced immutability
- ✅ Advanced FP concepts built-in
- Use for learning pure FP

---

## Quick Reference Card

```
┌────────────────────────────────────────────────────────────────┐
│                    FP COMPLETE CHEAT SHEET                     │
├────────────────────────────────────────────────────────────────┤
│ CORE CONCEPTS - PICHI                                          │
├────────────────────────────────────────────────────────────────┤
│ PURE FUNCTIONS 🔵                                              │
│   Rule: Same input → Same output, no side effects             │
│   Test: Can I replace call with return value?                 │
│   Real: f(5) = 10, f(5) = 10, f(5) = 10... (like math)       │
│                                                                │
│ IMMUTABILITY 🔒                                                │
│   Rule: Don't change data, create new                         │
│   Test: Does original data stay the same?                     │
│   Real: Photocopy (edit copy, keep original)                  │
│                                                                │
│ COMPOSITION 🔗                                                 │
│   Rule: f(g(h(x))) - combine simple → complex                │
│   Test: Can I break into small, reusable pieces?              │
│   Real: Assembly line (each station = one function)           │
│                                                                │
│ HIGHER-ORDER 🎯                                                │
│   Rule: Functions take/return functions                       │
│   Test: Does it take function arg or return function?         │
│   Real: Coffee shop (takes preferences, returns maker)        │
│                                                                │
│ DECLARATIVE 📝                                                 │
│   Rule: Say WHAT you want, not HOW                            │
│   Test: Am I describing result or steps?                      │
│   Real: "Make scrambled eggs" not "crack eggs..."             │
├────────────────────────────────────────────────────────────────┤
│ KEY OPERATIONS                                                 │
├────────────────────────────────────────────────────────────────┤
│ map:    [1,2,3].map(x => x*2) → [2,4,6] (transform)          │
│ filter: [1,2,3].filter(x => x>1) → [2,3] (select)            │
│ reduce: [1,2,3].reduce((a,b) => a+b) → 6 (aggregate)         │
├────────────────────────────────────────────────────────────────┤
│ ADVANCED CONCEPTS                                              │
├────────────────────────────────────────────────────────────────┤
│ Currying: f(a,b,c) → f(a)(b)(c)                               │
│ Partial:  f(a,b,c) with a=5 → f(b,c)                          │
│ Memoize:  Cache results of expensive functions                │
│ Pipe:     pipe(f, g, h) → left to right flow                 │
├────────────────────────────────────────────────────────────────┤
│ IMMUTABLE OPERATIONS                                           │
├────────────────────────────────────────────────────────────────┤
│ Add:    [...arr, x] or arr.concat(x)                          │
│ Remove: arr.filter(v => v !== x)                              │
│ Update: arr.map(v => v === old ? new : v)                     │
│ Clone:  {...obj} or {...obj, key: newVal}                     │
├────────────────────────────────────────────────────────────────┤
│ FP vs OOP                                                      │
├────────────────────────────────────────────────────────────────┤
│ OOP: Objects + Methods + Mutable State                        │
│ FP:  Functions + Immutable Data + Transformations             │
│                                                                │
│ Use OOP for: Structure, modeling, encapsulation               │
│ Use FP for:  Logic, transformations, concurrency              │
└────────────────────────────────────────────────────────────────┘
```

---

## Conclusion

Master FP by:
1. **Starting simple** - Pure functions and immutability first
2. **Using tools** - map/filter/reduce become second nature
3. **Composing** - Build complex from simple pieces
4. **Practicing** - Refactor imperative code to declarative
5. **Balancing** - Use FP where it helps, not everywhere

Remember: FP is a tool, not a religion. Use it where it makes code clearer, more testable, and more maintainable!


## ![✔] FP libraries for JS/TS

- Mori (http://swannodette.github.io/mori)
- Immutable.js (https://facebook.github.io/immutable-js/)
- Underscore (http://underscorejs.org)
- Lodash (https://lodash.com)
- Ramda (http://ramdajs.com)

## ![✔] Useful links

- [Curry and Function Composition](https://medium.com/javascript-scene/curry-and-function-composition-2c208d774983)
- [Composing Software: The Book](https://medium.com/javascript-scene/composing-software-the-book-f31c77fc3ddc)
- [An introduction to functional programming](https://codewords.recurse.com/issues/one/an-introduction-to-functional-programming)
- [Functional Programming in Javascript: How and Why](https://blog.bitsrc.io/functional-programming-in-javascript-how-and-why-94e7a97343b)
- [Functional JavaScript: Function Composition For Every Day Use](https://hackernoon.com/javascript-functional-composition-for-every-day-use-22421ef65a10)
- [An introduction to functional programming in JavaScript](https://opensource.com/article/17/6/functional-javascript)

<br>[⬆ Back to top](#Functional-Programming)
