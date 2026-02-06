# Functional Programming: Chapter 2 - Essential Techniques

## Table of Contents

- [Map, Filter, Reduce](#map-filter-reduce)
- [Currying & Partial Application](#currying--partial-application)
- [Closures](#closures)
- [Recursion](#recursion)

---

## Map, Filter, Reduce

The three fundamental operations for data transformation in FP.

### Map: Transform Each Element

**Purpose**: Apply a function to every element, creating a new collection.

**Real-World Scenario: Product Price Conversion**

<details>
<summary><strong>View Examples</strong></summary>

```ts
interface Product {
  id: string;
  name: string;
  priceUSD: number;
  category: string;
}

const products: Product[] = [
  { id: 'P1', name: 'Laptop', priceUSD: 999, category: 'Electronics' },
  { id: 'P2', name: 'Mouse', priceUSD: 25, category: 'Electronics' },
  { id: 'P3', name: 'Desk', priceUSD: 450, category: 'Furniture' },
];

// Convert USD to EUR (rate: 0.92)
const convertToEUR = (product: Product) => ({
  ...product,
  priceEUR: product.priceUSD * 0.92
});

const productsInEUR = products.map(convertToEUR);
console.log(productsInEUR);
// [
//   { id: 'P1', name: 'Laptop', priceUSD: 999, category: 'Electronics', priceEUR: 919.08 },
//   ...
// ]

// Chain multiple transformations
interface DisplayProduct {
  id: string;
  displayName: string;
  formattedPrice: string;
}

const toDisplayFormat = (product: Product): DisplayProduct => ({
  id: product.id,
  displayName: `${product.name} (${product.category})`,
  formattedPrice: `$${product.priceUSD.toFixed(2)}`
});

const displayProducts = products.map(toDisplayFormat);
console.log(displayProducts);
```

**Go Example:**

```go
package main

import "fmt"

type Product struct {
    ID       string
    Name     string
    PriceUSD float64
    Category string
}

type ProductEUR struct {
    Product
    PriceEUR float64
}

// Map function for products
func mapProducts(products []Product, fn func(Product) ProductEUR) []ProductEUR {
    result := make([]ProductEUR, len(products))
    for i, p := range products {
        result[i] = fn(p)
    }
    return result
}

func convertToEUR(p Product) ProductEUR {
    return ProductEUR{
        Product:  p,
        PriceEUR: p.PriceUSD * 0.92,
    }
}

func main() {
    products := []Product{
        {ID: "P1", Name: "Laptop", PriceUSD: 999, Category: "Electronics"},
        {ID: "P2", Name: "Mouse", PriceUSD: 25, Category: "Electronics"},
    }
    
    productsEUR := mapProducts(products, convertToEUR)
    fmt.Printf("%+v\n", productsEUR)
}
```

</details>

---

### Filter: Select Elements

**Purpose**: Keep only elements that match a condition.

**Real-World Scenario: Inventory Management**

<details>
<summary><strong>View Examples</strong></summary>

```ts
interface InventoryItem {
  sku: string;
  name: string;
  quantity: number;
  reorderPoint: number;
  supplier: string;
  lastRestocked: Date;
}

const inventory: InventoryItem[] = [
  { sku: 'SKU001', name: 'Widget A', quantity: 5, reorderPoint: 10, supplier: 'SupplierX', lastRestocked: new Date('2024-01-15') },
  { sku: 'SKU002', name: 'Widget B', quantity: 50, reorderPoint: 20, supplier: 'SupplierY', lastRestocked: new Date('2024-02-01') },
  { sku: 'SKU003', name: 'Widget C', quantity: 8, reorderPoint: 15, supplier: 'SupplierX', lastRestocked: new Date('2024-01-20') },
  { sku: 'SKU004', name: 'Widget D', quantity: 100, reorderPoint: 25, supplier: 'SupplierZ', lastRestocked: new Date('2024-02-05') },
];

// Find items that need reordering
const needsReorder = (item: InventoryItem): boolean => 
  item.quantity <= item.reorderPoint;

const itemsToReorder = inventory.filter(needsReorder);
console.log(itemsToReorder);
// [ SKU001 (5 ≤ 10), SKU003 (8 ≤ 15) ]

// Combine multiple filters
const criticalSupplierXItems = inventory
  .filter(item => item.supplier === 'SupplierX')
  .filter(needsReorder);

console.log(criticalSupplierXItems);

// Filter with complex logic
const isStaleStock = (item: InventoryItem): boolean => {
  const sixtyDaysAgo = new Date();
  sixtyDaysAgo.setDate(sixtyDaysAgo.getDate() - 60);
  return item.lastRestocked < sixtyDaysAgo && item.quantity > item.reorderPoint * 2;
};

const staleItems = inventory.filter(isStaleStock);
```

**Go Example:**

```go
package main

import (
    "fmt"
    "time"
)

type InventoryItem struct {
    SKU           string
    Name          string
    Quantity      int
    ReorderPoint  int
    Supplier      string
    LastRestocked time.Time
}

// Generic filter function
func filterItems(items []InventoryItem, predicate func(InventoryItem) bool) []InventoryItem {
    result := []InventoryItem{}
    for _, item := range items {
        if predicate(item) {
            result = append(result, item)
        }
    }
    return result
}

func needsReorder(item InventoryItem) bool {
    return item.Quantity <= item.ReorderPoint
}

func main() {
    inventory := []InventoryItem{
        {SKU: "SKU001", Name: "Widget A", Quantity: 5, ReorderPoint: 10, Supplier: "SupplierX"},
        {SKU: "SKU002", Name: "Widget B", Quantity: 50, ReorderPoint: 20, Supplier: "SupplierY"},
    }
    
    toReorder := filterItems(inventory, needsReorder)
    fmt.Printf("Items to reorder: %+v\n", toReorder)
}
```

</details>

---

### Reduce: Aggregate to Single Value

**Purpose**: Combine all elements into a single result.

**Real-World Scenario: Sales Analytics**

<details>
<summary><strong>View Examples</strong></summary>

```ts
interface Sale {
  id: string;
  product: string;
  amount: number;
  quantity: number;
  region: 'North' | 'South' | 'East' | 'West';
  date: Date;
}

const sales: Sale[] = [
  { id: 'S1', product: 'Laptop', amount: 999, quantity: 2, region: 'North', date: new Date('2024-01-15') },
  { id: 'S2', product: 'Mouse', amount: 25, quantity: 10, region: 'South', date: new Date('2024-01-16') },
  { id: 'S3', product: 'Laptop', amount: 999, quantity: 1, region: 'North', date: new Date('2024-01-17') },
  { id: 'S4', product: 'Keyboard', amount: 75, quantity: 5, region: 'East', date: new Date('2024-01-18') },
];

// Simple: Total revenue
const totalRevenue = sales.reduce(
  (total, sale) => total + (sale.amount * sale.quantity),
  0
);
console.log(`Total Revenue: $${totalRevenue}`); // $3621

// Complex: Revenue by region
interface RegionRevenue {
  [region: string]: number;
}

const revenueByRegion = sales.reduce<RegionRevenue>(
  (acc, sale) => ({
    ...acc,
    [sale.region]: (acc[sale.region] || 0) + (sale.amount * sale.quantity)
  }),
  {}
);
console.log(revenueByRegion);
// { North: 2997, South: 250, East: 375 }

// Advanced: Product sales summary
interface ProductSummary {
  product: string;
  totalRevenue: number;
  totalQuantity: number;
  averagePrice: number;
}

const productSummaries = Object.values(
  sales.reduce<Record<string, ProductSummary>>((acc, sale) => {
    const existing = acc[sale.product] || {
      product: sale.product,
      totalRevenue: 0,
      totalQuantity: 0,
      averagePrice: 0
    };
    
    const totalRevenue = existing.totalRevenue + (sale.amount * sale.quantity);
    const totalQuantity = existing.totalQuantity + sale.quantity;
    
    return {
      ...acc,
      [sale.product]: {
        product: sale.product,
        totalRevenue,
        totalQuantity,
        averagePrice: totalRevenue / totalQuantity
      }
    };
  }, {})
);

console.log(productSummaries);
// [
//   { product: 'Laptop', totalRevenue: 2997, totalQuantity: 3, averagePrice: 999 },
//   { product: 'Mouse', totalRevenue: 250, totalQuantity: 10, averagePrice: 25 },
//   { product: 'Keyboard', totalRevenue: 375, totalQuantity: 5, averagePrice: 75 }
// ]
```

**Go Example:**

```go
package main

import "fmt"

type Sale struct {
    ID       string
    Product  string
    Amount   float64
    Quantity int
    Region   string
}

// Generic reduce function
func reduce[T any, R any](items []T, initial R, fn func(R, T) R) R {
    result := initial
    for _, item := range items {
        result = fn(result, item)
    }
    return result
}

func main() {
    sales := []Sale{
        {ID: "S1", Product: "Laptop", Amount: 999, Quantity: 2, Region: "North"},
        {ID: "S2", Product: "Mouse", Amount: 25, Quantity: 10, Region: "South"},
        {ID: "S3", Product: "Laptop", Amount: 999, Quantity: 1, Region: "North"},
    }
    
    // Total revenue
    totalRevenue := reduce(sales, 0.0, func(acc float64, sale Sale) float64 {
        return acc + (sale.Amount * float64(sale.Quantity))
    })
    
    fmt.Printf("Total Revenue: $%.2f\n", totalRevenue)
    
    // Revenue by region
    revenueByRegion := reduce(sales, make(map[string]float64), 
        func(acc map[string]float64, sale Sale) map[string]float64 {
            acc[sale.Region] += sale.Amount * float64(sale.Quantity)
            return acc
        })
    
    fmt.Printf("Revenue by Region: %v\n", revenueByRegion)
}
```

</details>

---

## Practice Questions

<details>
<summary><strong>View Questions</strong></summary>

### Fill in the Blanks

1. The `map` function transforms each element and returns a collection of the __________ length.
2. The `filter` function returns elements that __________ a predicate function.
3. The `reduce` function takes an __________ value and an accumulator function.
4. Chaining map, filter, and reduce creates a __________ pipeline.

<details>
<summary><strong>View Answers</strong></summary>

1. **same** - Map performs 1-to-1 transformation, so output length equals input length
2. **satisfy** (or "match") - Filter keeps only elements where the predicate returns true
3. **initial** - Reduce needs a starting value (accumulator) to begin the aggregation process
4. **data processing** - These operations can be chained to transform, filter, and aggregate data in sequence

</details>

---

### True/False

1. ⬜ The `map` function can change the type of elements in the collection
2. ⬜ Using `filter` before `map` is more efficient than using `map` before `filter`
3. ⬜ The `reduce` function can only return numbers
4. ⬜ Map, filter, and reduce operations mutate the original array

<details>
<summary><strong>View Answers</strong></summary>

1. **True** - Map can transform `Product[]` to `string[]`, `number[]`, or any other type
2. **True** - Filtering first reduces the number of elements that map needs to process, improving performance
3. **False** - Reduce can return any type: objects, arrays, strings, numbers, etc. The type is determined by the initial value
4. **False** - These are pure functions that create new arrays/collections without modifying the original

</details>

---

### Multiple Choice

1. What will this code output?

```ts
const numbers = [1, 2, 3, 4, 5];
const result = numbers
  .filter(n => n % 2 === 0)
  .map(n => n * 2)
  .reduce((sum, n) => sum + n, 0);
```

- A) 30
- B) 12
- C) 6
- D) 20

2. Which operation would you use to find the highest-priced product?

- A) map
- B) filter
- C) reduce
- D) forEach

<details>
<summary><strong>View Answers</strong></summary>

1. **B** - 12
   - Filter even numbers: [2, 4]
   - Map (multiply by 2): [4, 8]
   - Reduce (sum): 4 + 8 = 12

2. **C** - reduce
   - Reduce can compare all elements and accumulate the maximum value
   - You could also use `Math.max(...products.map(p => p.price))` but that uses map + spread, not a single operation

</details>

---

### Code Challenge

Given this data structure, complete the tasks:

```ts
interface Transaction {
  id: string;
  userId: string;
  amount: number;
  type: 'deposit' | 'withdrawal';
  timestamp: Date;
}

const transactions: Transaction[] = [
  { id: 'T1', userId: 'U1', amount: 1000, type: 'deposit', timestamp: new Date('2024-01-15') },
  { id: 'T2', userId: 'U2', amount: 500, type: 'withdrawal', timestamp: new Date('2024-01-16') },
  { id: 'T3', userId: 'U1', amount: 200, type: 'withdrawal', timestamp: new Date('2024-01-17') },
  { id: 'T4', userId: 'U1', amount: 300, type: 'deposit', timestamp: new Date('2024-01-18') },
];

// TODO:
// 1. Calculate total deposits
// 2. Find all withdrawal transaction IDs
// 3. Calculate balance per user (deposits - withdrawals)
```

<details>
<summary><strong>View Answers</strong></summary>

```ts
interface Transaction {
  id: string;
  userId: string;
  amount: number;
  type: 'deposit' | 'withdrawal';
  timestamp: Date;
}

const transactions: Transaction[] = [
  { id: 'T1', userId: 'U1', amount: 1000, type: 'deposit', timestamp: new Date('2024-01-15') },
  { id: 'T2', userId: 'U2', amount: 500, type: 'withdrawal', timestamp: new Date('2024-01-16') },
  { id: 'T3', userId: 'U1', amount: 200, type: 'withdrawal', timestamp: new Date('2024-01-17') },
  { id: 'T4', userId: 'U1', amount: 300, type: 'deposit', timestamp: new Date('2024-01-18') },
];

// 1. Total deposits
const totalDeposits = transactions
  .filter(t => t.type === 'deposit')
  .reduce((sum, t) => sum + t.amount, 0);

console.log(`Total Deposits: $${totalDeposits}`); // $1300

// 2. Withdrawal transaction IDs
const withdrawalIds = transactions
  .filter(t => t.type === 'withdrawal')
  .map(t => t.id);

console.log(`Withdrawal IDs: ${withdrawalIds}`); // ['T2', 'T3']

// 3. Balance per user
interface UserBalance {
  [userId: string]: number;
}

const userBalances = transactions.reduce<UserBalance>((acc, t) => {
  const currentBalance = acc[t.userId] || 0;
  const change = t.type === 'deposit' ? t.amount : -t.amount;
  
  return {
    ...acc,
    [t.userId]: currentBalance + change
  };
}, {});

console.log(userBalances); // { U1: 1100, U2: -500 }

// Bonus: Most active user
const mostActiveUser = Object.entries(
  transactions.reduce<Record<string, number>>((acc, t) => ({
    ...acc,
    [t.userId]: (acc[t.userId] || 0) + 1
  }), {})
).reduce((max, [userId, count]) => 
  count > max.count ? { userId, count } : max,
  { userId: '', count: 0 }
);

console.log(mostActiveUser); // { userId: 'U1', count: 3 }
```

</details>

</details>

---

## Currying & Partial Application

### Currying

**Definition**: Transform a function with multiple arguments into a sequence of functions, each taking a single argument.

`f(a, b, c)` → `f(a)(b)(c)`

**Real-World Scenario: Configurable Logging System**

<details>
<summary><strong>View Examples</strong></summary>

```ts
// Non-curried version
function log(level: string, module: string, message: string): void {
  console.log(`[${level}] [${module}] ${message}`);
}

log('ERROR', 'Auth', 'Invalid credentials');
log('ERROR', 'Auth', 'Token expired');
log('INFO', 'Database', 'Connection established');

// ✅ Curried version
const curriedLog = (level: string) => 
  (module: string) => 
    (message: string): void => {
      console.log(`[${level}] [${module}] ${message}`);
    };

// Create specialized loggers
const errorLog = curriedLog('ERROR');
const infoLog = curriedLog('INFO');

const authErrorLog = errorLog('Auth');
const dbInfoLog = infoLog('Database');

// Usage is now cleaner
authErrorLog('Invalid credentials');
authErrorLog('Token expired');
dbInfoLog('Connection established');

// Real-world: Discount calculator
const calculateDiscount = (discountPercent: number) =>
  (taxRate: number) =>
    (price: number): number => {
      const discounted = price * (1 - discountPercent / 100);
      return discounted * (1 + taxRate / 100);
    };

// Create specialized calculators
const memberDiscount = calculateDiscount(10); // 10% off
const memberDiscountWithTax = memberDiscount(8.5); // 8.5% tax

console.log(memberDiscountWithTax(100)); // $97.65
console.log(memberDiscountWithTax(50));  // $48.825

// Different discount tier
const vipDiscount = calculateDiscount(25);
const vipDiscountWithTax = vipDiscount(8.5);

console.log(vipDiscountWithTax(100)); // $81.375
```

**Go Example:**

```go
package main

import "fmt"

// Curried log function
func curriedLog(level string) func(string) func(string) {
    return func(module string) func(string) {
        return func(message string) {
            fmt.Printf("[%s] [%s] %s\n", level, module, message)
        }
    }
}

// Curried discount calculator
func calculateDiscount(discountPercent float64) func(float64) func(float64) float64 {
    return func(taxRate float64) func(float64) float64 {
        return func(price float64) float64 {
            discounted := price * (1 - discountPercent/100)
            return discounted * (1 + taxRate/100)
        }
    }
}

func main() {
    errorLog := curriedLog("ERROR")
    authErrorLog := errorLog("Auth")
    
    authErrorLog("Invalid credentials")
    authErrorLog("Token expired")
    
    memberDiscount := calculateDiscount(10)
    memberDiscountWithTax := memberDiscount(8.5)
    
    fmt.Printf("Price: $%.2f\n", memberDiscountWithTax(100))
}
```

</details>

---

### Partial Application

**Definition**: Fix some arguments of a function, creating a new function with fewer parameters.

**Real-World Scenario: API Request Builder**

<details>
<summary><strong>View Examples</strong></summary>

```ts
// Generic API request function
interface RequestConfig {
  method: string;
  baseURL: string;
  endpoint: string;
  headers: Record<string, string>;
  body?: any;
}

function makeRequest(config: RequestConfig): Promise<any> {
  const url = `${config.baseURL}${config.endpoint}`;
  return fetch(url, {
    method: config.method,
    headers: config.headers,
    body: config.body ? JSON.stringify(config.body) : undefined
  }).then(res => res.json());
}

// Partial application: Fix base configuration
const createAPIClient = (baseURL: string, authToken: string) => {
  const baseHeaders = {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${authToken}`
  };
  
  return {
    get: (endpoint: string) => 
      makeRequest({ method: 'GET', baseURL, endpoint, headers: baseHeaders }),
    
    post: (endpoint: string, body: any) =>
      makeRequest({ method: 'POST', baseURL, endpoint, headers: baseHeaders, body }),
    
    put: (endpoint: string, body: any) =>
      makeRequest({ method: 'PUT', baseURL, endpoint, headers: baseHeaders, body }),
    
    delete: (endpoint: string) =>
      makeRequest({ method: 'DELETE', baseURL, endpoint, headers: baseHeaders })
  };
};

// Usage
const apiClient = createAPIClient('https://api.example.com', 'secret-token-123');

// Now we only need to specify the endpoint
apiClient.get('/users');
apiClient.post('/users', { name: 'Alice', email: 'alice@example.com' });
apiClient.put('/users/123', { name: 'Alice Smith' });
apiClient.delete('/users/456');

// More partial application: Database query builder
type QueryExecutor = (sql: string, params: any[]) => Promise<any>;

const createQueryBuilder = (tableName: string, executor: QueryExecutor) => ({
  findById: (id: number) => 
    executor(`SELECT * FROM ${tableName} WHERE id = ?`, [id]),
  
  findAll: () => 
    executor(`SELECT * FROM ${tableName}`, []),
  
  create: (data: Record<string, any>) => {
    const keys = Object.keys(data);
    const values = Object.values(data);
    const placeholders = keys.map(() => '?').join(', ');
    
    return executor(
      `INSERT INTO ${tableName} (${keys.join(', ')}) VALUES (${placeholders})`,
      values
    );
  },
  
  update: (id: number, data: Record<string, any>) => {
    const sets = Object.keys(data).map(k => `${k} = ?`).join(', ');
    
    return executor(
      `UPDATE ${tableName} SET ${sets} WHERE id = ?`,
      [...Object.values(data), id]
    );
  }
});

// Create specialized query builders
const mockExecutor: QueryExecutor = async (sql, params) => {
  console.log('Executing:', sql, 'with params:', params);
  return { success: true };
};

const userQueries = createQueryBuilder('users', mockExecutor);
const productQueries = createQueryBuilder('products', mockExecutor);

// Clean, specialized usage
await userQueries.findById(1);
await userQueries.create({ name: 'Bob', email: 'bob@example.com' });
await productQueries.findAll();
```

**Go Example:**

```go
package main

import "fmt"

type QueryExecutor func(sql string, params []interface{}) error

type QueryBuilder struct {
    tableName string
    executor  QueryExecutor
}

func createQueryBuilder(tableName string, executor QueryExecutor) QueryBuilder {
    return QueryBuilder{tableName: tableName, executor: executor}
}

func (qb QueryBuilder) FindByID(id int) error {
    sql := fmt.Sprintf("SELECT * FROM %s WHERE id = ?", qb.tableName)
    return qb.executor(sql, []interface{}{id})
}

func (qb QueryBuilder) FindAll() error {
    sql := fmt.Sprintf("SELECT * FROM %s", qb.tableName)
    return qb.executor(sql, []interface{}{})
}

func mockExecutor(sql string, params []interface{}) error {
    fmt.Printf("Executing: %s with params: %v\n", sql, params)
    return nil
}

func main() {
    userQueries := createQueryBuilder("users", mockExecutor)
    productQueries := createQueryBuilder("products", mockExecutor)
    
    userQueries.FindByID(1)
    productQueries.FindAll()
}
```

</details>

### Currying vs Partial Application

| Aspect | Currying | Partial Application |
|--------|----------|---------------------|
| **Definition** | Transform multi-arg function to single-arg chain | Fix some arguments, return function with remaining |
| **Structure** | `f(a)(b)(c)` | `f(a, b, ...)` where some args are preset |
| **Flexibility** | All arguments one at a time | Can fix any number of arguments |
| **Use Case** | Configurable pipelines | Specialized versions of functions |

---

## Closures

**Definition**: A function that "remembers" variables from its outer scope, even after that scope has finished executing.

**Real-World Scenario: Rate Limiter**

<details>
<summary><strong>View Examples</strong></summary>

```ts
// Create a rate limiter that allows N calls per time window
function createRateLimiter(maxCalls: number, windowMs: number) {
  let calls: number[] = []; // Closure: this persists between function calls
  
  return function(action: () => void): boolean {
    const now = Date.now();
    
    // Remove calls outside the time window
    calls = calls.filter(callTime => now - callTime < windowMs);
    
    if (calls.length < maxCalls) {
      calls.push(now);
      action();
      return true;
    }
    
    console.log('Rate limit exceeded');
    return false;
  };
}

// Usage: API client with rate limiting
const apiCall = createRateLimiter(5, 10000); // 5 calls per 10 seconds

for (let i = 0; i < 10; i++) {
  apiCall(() => console.log(`API call ${i + 1}`));
}
// First 5 succeed, next 5 are rate-limited

// Real-world: Counter factory
function createCounter(initialValue: number = 0) {
  let count = initialValue; // Private state
  
  return {
    increment: () => ++count,
    decrement: () => --count,
    getValue: () => count,
    reset: () => { count = initialValue; }
  };
}

const cartCounter = createCounter(0);
console.log(cartCounter.increment()); // 1
console.log(cartCounter.increment()); // 2
console.log(cartCounter.getValue());  // 2
cartCounter.reset();
console.log(cartCounter.getValue());  // 0

// Advanced: Memoization with closure
function memoize<T extends (...args: any[]) => any>(fn: T): T {
  const cache = new Map<string, ReturnType<T>>();
  
  return ((...args: Parameters<T>) => {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      console.log('Cache hit!');
      return cache.get(key)!;
    }
    
    console.log('Computing...');
    const result = fn(...args);
    cache.set(key, result);
    return result;
  }) as T;
}

// Expensive computation
const fibonacci = memoize((n: number): number => {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
});

console.log(fibonacci(10)); // Computing... 55
console.log(fibonacci(10)); // Cache hit! 55
console.log(fibonacci(11)); // Partially cached
```

**Go Example:**

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Rate limiter using closure
func createRateLimiter(maxCalls int, windowMs int64) func(func()) bool {
    var calls []int64
    var mu sync.Mutex
    
    return func(action func()) bool {
        mu.Lock()
        defer mu.Unlock()
        
        now := time.Now().UnixMilli()
        
        // Filter old calls
        validCalls := []int64{}
        for _, callTime := range calls {
            if now-callTime < windowMs {
                validCalls = append(validCalls, callTime)
            }
        }
        calls = validCalls
        
        if len(calls) < maxCalls {
            calls = append(calls, now)
            action()
            return true
        }
        
        fmt.Println("Rate limit exceeded")
        return false
    }
}

// Counter factory
func createCounter(initialValue int) map[string]func() int {
    count := initialValue
    
    return map[string]func() int{
        "increment": func() int {
            count++
            return count
        },
        "decrement": func() int {
            count--
            return count
        },
        "getValue": func() int {
            return count
        },
        "reset": func() int {
            count = initialValue
            return count
        },
    }
}

func main() {
    apiCall := createRateLimiter(5, 10000)
    
    for i := 0; i < 10; i++ {
        apiCall(func() {
            fmt.Printf("API call %d\n", i+1)
        })
    }
    
    counter := createCounter(0)
    fmt.Println(counter["increment"]()) // 1
    fmt.Println(counter["increment"]()) // 2
    fmt.Println(counter["getValue"]())  // 2
}
```

</details>

---

## Practice Questions

<details>
<summary><strong>View Questions</strong></summary>

### Fill in the Blanks

1. Currying transforms a function with N arguments into __________ functions each taking __________ argument.
2. Partial application fixes some __________ of a function, returning a new function.
3. A closure is a function that remembers variables from its __________ scope.
4. Memoization uses closures to __________ previously computed results.

<details>
<summary><strong>View Answers</strong></summary>

1. **N**, **one** - Currying creates a chain where each function accepts exactly one parameter
2. **arguments** (or "parameters") - Partial application pre-fills certain parameters, creating a specialized version
3. **outer** (or "enclosing") - Closures capture and retain access to variables from the scope where they were created
4. **cache** - Memoization stores results in a closure variable to avoid recomputing for the same inputs

</details>

---

### True/False

1. ⬜ Currying and partial application are the same thing
2. ⬜ Closures can access variables from their outer scope even after that scope has returned
3. ⬜ Every curried function is automatically memoized
4. ⬜ Closures enable data privacy by keeping variables inaccessible from outside

<details>
<summary><strong>View Answers</strong></summary>

1. **False** - Currying creates single-argument function chains; partial application fixes some arguments. They're related but different techniques
2. **True** - This is the defining characteristic of closures - they "close over" variables from their lexical scope
3. **False** - Memoization must be explicitly implemented; currying alone doesn't cache results
4. **True** - Variables captured in closures are private and can only be accessed through the closure's interface

</details>

---

### Code Challenge

Create a configurable validator using currying and closures:

```ts
// TODO: Implement these functions

// 1. Create a curried validation function
// validate(minLength)(maxLength)(input) => boolean

// 2. Create a validator factory with closures
// const emailValidator = createValidator([rules...])
// emailValidator('test@example.com') => { valid: boolean, errors: string[] }

// Rules should include:
// - minLength
// - maxLength
// - contains (substring check)
// - matches (regex check)
```

<details>
<summary><strong>View Answers</strong></summary>

```ts
// 1. Curried length validator
const validateLength = (minLength: number) =>
  (maxLength: number) =>
    (input: string): boolean => 
      input.length >= minLength && input.length <= maxLength;

const passwordLengthValidator = validateLength(8)(20);
console.log(passwordLengthValidator('secret')); // false (too short)
console.log(passwordLengthValidator('mySecurePassword')); // true

// 2. Validator factory with closures
type ValidationRule = {
  name: string;
  validate: (input: string) => boolean;
  message: string;
};

type ValidationResult = {
  valid: boolean;
  errors: string[];
};

function createValidator(rules: ValidationRule[]) {
  // Closure: rules array is captured
  return function(input: string): ValidationResult {
    const errors: string[] = [];
    
    for (const rule of rules) {
      if (!rule.validate(input)) {
        errors.push(rule.message);
      }
    }
    
    return {
      valid: errors.length === 0,
      errors
    };
  };
}

// Create reusable validation rules
const minLength = (length: number): ValidationRule => ({
  name: 'minLength',
  validate: (input) => input.length >= length,
  message: `Must be at least ${length} characters`
});

const maxLength = (length: number): ValidationRule => ({
  name: 'maxLength',
  validate: (input) => input.length <= length,
  message: `Must be no more than ${length} characters`
});

const contains = (substring: string): ValidationRule => ({
  name: 'contains',
  validate: (input) => input.includes(substring),
  message: `Must contain "${substring}"`
});

const matches = (pattern: RegExp, description: string): ValidationRule => ({
  name: 'matches',
  validate: (input) => pattern.test(input),
  message: description
});

// Usage: Email validator
const emailValidator = createValidator([
  minLength(5),
  maxLength(100),
  contains('@'),
  matches(/^[^\s@]+@[^\s@]+\.[^\s@]+$/, 'Must be a valid email format')
]);

console.log(emailValidator('test@example.com'));
// { valid: true, errors: [] }

console.log(emailValidator('bad'));
// { valid: false, errors: ['Must be at least 5 characters', 'Must contain "@"', ...] }

// Password validator
const passwordValidator = createValidator([
  minLength(8),
  maxLength(50),
  matches(/[A-Z]/, 'Must contain at least one uppercase letter'),
  matches(/[a-z]/, 'Must contain at least one lowercase letter'),
  matches(/[0-9]/, 'Must contain at least one number'),
  matches(/[!@#$%^&*]/, 'Must contain at least one special character')
]);

console.log(passwordValidator('weak'));
// { valid: false, errors: [...] }

console.log(passwordValidator('SecurePass123!'));
// { valid: true, errors: [] }
```

**Explanation**: The validator factory uses closures to remember the rules array. Each validation rule is a curried function that returns a ValidationRule object. This makes validators highly reusable and composable!

</details>

</details>

---

## Recursion

**Definition**: A function that calls itself to solve a problem by breaking it into smaller, similar subproblems.

**Key Components**:
1. **Base case**: Condition that stops recursion
2. **Recursive case**: Function calls itself with a simpler input

**Real-Life Analogy**:  
Think of Russian nesting dolls (Matryoshka). To find the smallest doll, you open the current doll (recursive case) until you reach a doll that doesn't open (base case).

---

### When to Use Recursion

✅ **Good for**:
- Tree/graph traversal
- Divide and conquer algorithms
- Mathematical sequences
- Hierarchical data structures

❌ **Avoid when**:
- Simple iteration works better
- Deep recursion risks stack overflow
- Performance is critical (iteration is often faster)

---

### Basic Recursion Examples

**Real-World Scenario: File System Navigation**

<details>
<summary><strong>View Examples</strong></summary>

```ts
// File system structure
interface FileNode {
  name: string;
  type: 'file' | 'folder';
  size?: number;
  children?: FileNode[];
}

const fileSystem: FileNode = {
  name: 'root',
  type: 'folder',
  children: [
    {
      name: 'documents',
      type: 'folder',
      children: [
        { name: 'resume.pdf', type: 'file', size: 1024 },
        { name: 'cover-letter.docx', type: 'file', size: 512 },
        {
          name: 'projects',
          type: 'folder',
          children: [
            { name: 'project1.txt', type: 'file', size: 256 },
            { name: 'project2.txt', type: 'file', size: 128 }
          ]
        }
      ]
    },
    {
      name: 'images',
      type: 'folder',
      children: [
        { name: 'photo1.jpg', type: 'file', size: 2048 },
        { name: 'photo2.png', type: 'file', size: 1536 }
      ]
    },
    { name: 'readme.txt', type: 'file', size: 64 }
  ]
};

// Calculate total size recursively
function calculateTotalSize(node: FileNode): number {
  // Base case: it's a file
  if (node.type === 'file') {
    return node.size || 0;
  }
  
  // Recursive case: it's a folder, sum children
  if (node.children) {
    return node.children.reduce(
      (total, child) => total + calculateTotalSize(child),
      0
    );
  }
  
  return 0;
}

console.log(`Total size: ${calculateTotalSize(fileSystem)} bytes`); // 5568

// Find all files matching a pattern
function findFiles(node: FileNode, pattern: RegExp): string[] {
  // Base case: it's a file
  if (node.type === 'file') {
    return pattern.test(node.name) ? [node.name] : [];
  }
  
  // Recursive case: it's a folder
  if (node.children) {
    return node.children.flatMap(child => findFiles(child, pattern));
  }
  
  return [];
}

console.log(findFiles(fileSystem, /\.txt$/)); 
// ['project1.txt', 'project2.txt', 'readme.txt']

// Get folder structure as string
function printStructure(node: FileNode, indent: string = ''): string {
  let result = `${indent}${node.name}\n`;
  
  if (node.children) {
    for (const child of node.children) {
      result += printStructure(child, indent + '  ');
    }
  }
  
  return result;
}

console.log(printStructure(fileSystem));
// root
//   documents
//     resume.pdf
//     cover-letter.docx
//     projects
//       project1.txt
//       project2.txt
//   images
//     photo1.jpg
//     photo2.png
//   readme.txt
```

**Go Example:**

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

type FileNode struct {
    Name     string
    Type     string // "file" or "folder"
    Size     int
    Children []FileNode
}

func calculateTotalSize(node FileNode) int {
    // Base case: file
    if node.Type == "file" {
        return node.Size
    }
    
    // Recursive case: folder
    total := 0
    for _, child := range node.Children {
        total += calculateTotalSize(child)
    }
    return total
}

func findFiles(node FileNode, pattern *regexp.Regexp) []string {
    // Base case: file
    if node.Type == "file" {
        if pattern.MatchString(node.Name) {
            return []string{node.Name}
        }
        return []string{}
    }
    
    // Recursive case: folder
    result := []string{}
    for _, child := range node.Children {
        result = append(result, findFiles(child, pattern)...)
    }
    return result
}

func printStructure(node FileNode, indent string) string {
    result := indent + node.Name + "\n"
    
    for _, child := range node.Children {
        result += printStructure(child, indent+"  ")
    }
    
    return result
}

func main() {
    fs := FileNode{
        Name: "root",
        Type: "folder",
        Children: []FileNode{
            {
                Name: "documents",
                Type: "folder",
                Children: []FileNode{
                    {Name: "resume.pdf", Type: "file", Size: 1024},
                    {Name: "project.txt", Type: "file", Size: 256},
                },
            },
            {Name: "readme.txt", Type: "file", Size: 64},
        },
    }
    
    fmt.Printf("Total size: %d bytes\n", calculateTotalSize(fs))
    
    pattern := regexp.MustCompile(`\.txt$`)
    fmt.Printf("Text files: %v\n", findFiles(fs, pattern))
    
    fmt.Println(printStructure(fs, ""))
}
```

</details>

---

### Tail Recursion

**Problem**: Regular recursion can cause stack overflow with deep calls.  
**Solution**: Tail recursion - recursive call is the last operation (can be optimized by compiler).

**Real-World Scenario: Processing Large Dataset**

<details>
<summary><strong>View Examples</strong></summary>

```ts
// ❌ Regular recursion (not tail-recursive)
function sumArray(arr: number[]): number {
  if (arr.length === 0) return 0;
  return arr[0] + sumArray(arr.slice(1)); // Addition happens AFTER recursive call
}

// ✅ Tail recursive version
function sumArrayTail(arr: number[], accumulator: number = 0): number {
  if (arr.length === 0) return accumulator;
  return sumArrayTail(arr.slice(1), accumulator + arr[0]); // Recursive call is LAST
}

console.log(sumArrayTail([1, 2, 3, 4, 5])); // 15

// Real-world: Factorial
function factorial(n: number): number {
  if (n <= 1) return 1;
  return n * factorial(n - 1); // Not tail recursive
}

function factorialTail(n: number, accumulator: number = 1): number {
  if (n <= 1) return accumulator;
  return factorialTail(n - 1, n * accumulator); // Tail recursive
}

console.log(factorialTail(5)); // 120

// Processing transactions in batches
interface Transaction {
  id: string;
  amount: number;
}

async function processTransactions(
  transactions: Transaction[],
  processed: Transaction[] = []
): Promise<Transaction[]> {
  // Base case
  if (transactions.length === 0) {
    return processed;
  }
  
  // Process first transaction
  const [current, ...rest] = transactions;
  console.log(`Processing transaction ${current.id}`);
  
  // Simulate async processing
  await new Promise(resolve => setTimeout(resolve, 100));
  
  // Tail recursive call
  return processTransactions(rest, [...processed, current]);
}

// Usage
const transactions: Transaction[] = [
  { id: 'T1', amount: 100 },
  { id: 'T2', amount: 200 },
  { id: 'T3', amount: 150 }
];

processTransactions(transactions).then(result => {
  console.log('All processed:', result);
});
```

**Go Example:**

```go
package main

import "fmt"

// Regular recursion
func sumArray(arr []int) int {
    if len(arr) == 0 {
        return 0
    }
    return arr[0] + sumArray(arr[1:])
}

// Tail recursive version
func sumArrayTail(arr []int, accumulator int) int {
    if len(arr) == 0 {
        return accumulator
    }
    return sumArrayTail(arr[1:], accumulator+arr[0])
}

// Factorial - tail recursive
func factorialTail(n, accumulator int) int {
    if n <= 1 {
        return accumulator
    }
    return factorialTail(n-1, n*accumulator)
}

// Helper function with default accumulator
func factorial(n int) int {
    return factorialTail(n, 1)
}

func main() {
    fmt.Println(sumArrayTail([]int{1, 2, 3, 4, 5}, 0)) // 15
    fmt.Println(factorial(5))                          // 120
}
```

</details>

---

### Recursion vs Iteration

**Real-World Scenario: Binary Search**

<details>
<summary><strong>View Examples</strong></summary>

```ts
// Product catalog sorted by price
interface Product {
  id: string;
  name: string;
  price: number;
}

const products: Product[] = [
  { id: 'P1', name: 'Mouse', price: 25 },
  { id: 'P2', name: 'Keyboard', price: 75 },
  { id: 'P3', name: 'Monitor', price: 300 },
  { id: 'P4', name: 'Laptop', price: 999 },
  { id: 'P5', name: 'Desk', price: 1500 }
];

// ❌ Iterative binary search
function binarySearchIterative(
  products: Product[],
  targetPrice: number
): Product | null {
  let left = 0;
  let right = products.length - 1;
  
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    const midPrice = products[mid].price;
    
    if (midPrice === targetPrice) {
      return products[mid];
    } else if (midPrice < targetPrice) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  
  return null;
}

// ✅ Recursive binary search
function binarySearchRecursive(
  products: Product[],
  targetPrice: number,
  left: number = 0,
  right: number = products.length - 1
): Product | null {
  // Base case
  if (left > right) {
    return null;
  }
  
  const mid = Math.floor((left + right) / 2);
  const midPrice = products[mid].price;
  
  // Base case: found
  if (midPrice === targetPrice) {
    return products[mid];
  }
  
  // Recursive cases
  if (midPrice < targetPrice) {
    return binarySearchRecursive(products, targetPrice, mid + 1, right);
  } else {
    return binarySearchRecursive(products, targetPrice, left, mid - 1);
  }
}

console.log(binarySearchRecursive(products, 300)); 
// { id: 'P3', name: 'Monitor', price: 300 }

console.log(binarySearchIterative(products, 999));
// { id: 'P4', name: 'Laptop', price: 999 }

// When to use which?
// Iteration: Better for simple loops, better performance
// Recursion: Better for tree/graph traversal, clearer for divide-and-conquer
```

**Go Example:**

```go
package main

import "fmt"

type Product struct {
    ID    string
    Name  string
    Price int
}

// Iterative binary search
func binarySearchIterative(products []Product, targetPrice int) *Product {
    left := 0
    right := len(products) - 1
    
    for left <= right {
        mid := (left + right) / 2
        midPrice := products[mid].Price
        
        if midPrice == targetPrice {
            return &products[mid]
        } else if midPrice < targetPrice {
            left = mid + 1
        } else {
            right = mid - 1
        }
    }
    
    return nil
}

// Recursive binary search
func binarySearchRecursive(products []Product, targetPrice, left, right int) *Product {
    if left > right {
        return nil
    }
    
    mid := (left + right) / 2
    midPrice := products[mid].Price
    
    if midPrice == targetPrice {
        return &products[mid]
    }
    
    if midPrice < targetPrice {
        return binarySearchRecursive(products, targetPrice, mid+1, right)
    }
    return binarySearchRecursive(products, targetPrice, left, mid-1)
}

func main() {
    products := []Product{
        {ID: "P1", Name: "Mouse", Price: 25},
        {ID: "P2", Name: "Keyboard", Price: 75},
        {ID: "P3", Name: "Monitor", Price: 300},
    }
    
    result := binarySearchRecursive(products, 300, 0, len(products)-1)
    if result != nil {
        fmt.Printf("Found: %+v\n", *result)
    }
}
```

</details>

---

### Advanced Recursion: Memoization

**Problem**: Recursive functions often recalculate the same values.  
**Solution**: Cache results to avoid redundant calculations.

**Real-World Scenario: Dynamic Pricing Calculator**

<details>
<summary><strong>View Examples</strong></summary>

```ts
// Calculate optimal pricing strategy
// Without memoization - SLOW!
function calculateOptimalPrice(
  basePrice: number,
  competitors: number,
  marketDemand: number
): number {
  if (competitors === 0) return basePrice;
  if (marketDemand === 0) return basePrice * 0.5;
  
  // Expensive recursive calculations
  const competitorAdjustment = calculateOptimalPrice(
    basePrice,
    competitors - 1,
    marketDemand
  );
  const demandAdjustment = calculateOptimalPrice(
    basePrice,
    competitors,
    marketDemand - 1
  );
  
  return (competitorAdjustment + demandAdjustment) / 2;
}

// With memoization - FAST!
function createMemoizedPricing() {
  const cache = new Map<string, number>();
  
  function calculatePrice(
    basePrice: number,
    competitors: number,
    marketDemand: number
  ): number {
    const key = `${basePrice}-${competitors}-${marketDemand}`;
    
    if (cache.has(key)) {
      return cache.get(key)!;
    }
    
    // Base cases
    if (competitors === 0) return basePrice;
    if (marketDemand === 0) return basePrice * 0.5;
    
    // Recursive cases
    const competitorAdjustment = calculatePrice(
      basePrice,
      competitors - 1,
      marketDemand
    );
    const demandAdjustment = calculatePrice(
      basePrice,
      competitors,
      marketDemand - 1
    );
    
    const result = (competitorAdjustment + demandAdjustment) / 2;
    cache.set(key, result);
    return result;
  }
  
  return calculatePrice;
}

const memoizedPricing = createMemoizedPricing();

console.time('Without memoization');
console.log(calculateOptimalPrice(100, 10, 10));
console.timeEnd('Without memoization'); // ~2000ms

console.time('With memoization');
console.log(memoizedPricing(100, 10, 10));
console.timeEnd('With memoization'); // ~2ms

// Classic example: Fibonacci
function fibonacciSlow(n: number): number {
  if (n <= 1) return n;
  return fibonacciSlow(n - 1) + fibonacciSlow(n - 2);
}

function createFibonacci() {
  const cache = new Map<number, number>();
  
  function fib(n: number): number {
    if (n <= 1) return n;
    
    if (cache.has(n)) {
      return cache.get(n)!;
    }
    
    const result = fib(n - 1) + fib(n - 2);
    cache.set(n, result);
    return result;
  }
  
  return fib;
}

const fibonacci = createFibonacci();

console.time('Slow fib(40)');
console.log(fibonacciSlow(40)); // Takes several seconds!
console.timeEnd('Slow fib(40)');

console.time('Fast fib(40)');
console.log(fibonacci(40)); // Instant!
console.timeEnd('Fast fib(40)');

// O(2^n) → O(n) improvement!
```

**Go Example:**

```go
package main

import (
    "fmt"
    "time"
)

// Fibonacci without memoization
func fibonacciSlow(n int) int {
    if n <= 1 {
        return n
    }
    return fibonacciSlow(n-1) + fibonacciSlow(n-2)
}

// Fibonacci with memoization
func createFibonacci() func(int) int {
    cache := make(map[int]int)
    
    var fib func(int) int
    fib = func(n int) int {
        if n <= 1 {
            return n
        }
        
        if val, exists := cache[n]; exists {
            return val
        }
        
        result := fib(n-1) + fib(n-2)
        cache[n] = result
        return result
    }
    
    return fib
}

func main() {
    fibonacci := createFibonacci()
    
    start := time.Now()
    result := fibonacci(40)
    duration := time.Since(start)
    
    fmt.Printf("fib(40) = %d (took %v)\n", result, duration)
}
```

</details>

---

## Practice Questions

<details>
<summary><strong>View Questions</strong></summary>

### Fill in the Blanks

1. Every recursive function must have a __________ case to prevent infinite recursion.
2. In tail recursion, the recursive call must be the __________ operation in the function.
3. Memoization improves recursive performance by __________ previously computed results.
4. The time complexity of naive Fibonacci is O(2^n), but with memoization it becomes O(__________).

<details>
<summary><strong>View Answers</strong></summary>

1. **base** - The base case provides the termination condition that stops the recursion
2. **last** - Tail recursion means no operations occur after the recursive call returns, allowing compiler optimization
3. **caching** (or "storing") - Memoization trades space for time by remembering results
4. **n** - Memoization ensures each subproblem is solved only once, making it linear time

</details>

---

### True/False

1. ⬜ Recursion is always more efficient than iteration
2. ⬜ Tail-recursive functions can be optimized to avoid stack overflow
3. ⬜ Every recursive function can be converted to an iterative one
4. ⬜ Memoization is only useful for recursive functions

<details>
<summary><strong>View Answers</strong></summary>

1. **False** - Recursion has overhead from function calls. Iteration is often faster but recursion can be more readable for certain problems
2. **True** - Tail call optimization converts tail recursion to iteration internally, using constant stack space
3. **True** - Any recursive algorithm can be rewritten iteratively using explicit stacks, though it may be less elegant
4. **False** - Memoization works with any function (recursive or not) that has expensive, repeated computations with the same inputs

</details>

---

### Multiple Choice

1. Which problem is BEST suited for recursion?

- A) Summing numbers in an array
- B) Traversing a tree structure
- C) Iterating over a list
- D) Finding the maximum in an array

2. What's the main risk of deep recursion without tail optimization?

- A) Slow performance
- B) Stack overflow
- C) Memory leak
- D) Infinite loop

<details>
<summary><strong>View Answers</strong></summary>

1. **B** - Traversing a tree structure. Trees are inherently recursive (each node is the root of a subtree), making recursion natural and elegant. The other options are simpler with iteration.

2. **B** - Stack overflow. Each recursive call adds a frame to the call stack. Without tail optimization, deep recursion exhausts stack space, causing crashes.

</details>

---

### Code Challenge

Implement these recursive functions for a comment system (nested comments like Reddit):

```ts
interface Comment {
  id: string;
  author: string;
  text: string;
  replies: Comment[];
}

// TODO:
// 1. countTotalComments(comment): Count all comments and nested replies
// 2. findCommentById(comment, id): Find a comment by ID (deep search)
// 3. flattenComments(comment): Convert tree to flat array
// 4. BONUS: Implement memoized version of countTotalComments
```

<details>
<summary><strong>View Answers</strong></summary>

```ts
interface Comment {
  id: string;
  author: string;
  text: string;
  replies: Comment[];
}

// Sample data
const commentTree: Comment = {
  id: 'c1',
  author: 'Alice',
  text: 'Great article!',
  replies: [
    {
      id: 'c2',
      author: 'Bob',
      text: 'I agree!',
      replies: [
        {
          id: 'c3',
          author: 'Carol',
          text: 'Me too!',
          replies: []
        }
      ]
    },
    {
      id: 'c4',
      author: 'Dave',
      text: 'Interesting point',
      replies: []
    }
  ]
};

// 1. Count total comments
function countTotalComments(comment: Comment): number {
  // Base case: this comment + no replies
  if (comment.replies.length === 0) {
    return 1;
  }
  
  // Recursive case: this comment + all replies
  return 1 + comment.replies.reduce(
    (total, reply) => total + countTotalComments(reply),
    0
  );
}

console.log(countTotalComments(commentTree)); // 4

// 2. Find comment by ID
function findCommentById(comment: Comment, id: string): Comment | null {
  // Base case: found it
  if (comment.id === id) {
    return comment;
  }
  
  // Base case: no replies
  if (comment.replies.length === 0) {
    return null;
  }
  
  // Recursive case: search replies
  for (const reply of comment.replies) {
    const found = findCommentById(reply, id);
    if (found) {
      return found;
    }
  }
  
  return null;
}

console.log(findCommentById(commentTree, 'c3'));
// { id: 'c3', author: 'Carol', ... }

// 3. Flatten comments to array
function flattenComments(comment: Comment): Comment[] {
  // Base case: just this comment
  if (comment.replies.length === 0) {
    return [comment];
  }
  
  // Recursive case: this comment + flattened replies
  const flattened = [comment];
  for (const reply of comment.replies) {
    flattened.push(...flattenComments(reply));
  }
  
  return flattened;
}

console.log(flattenComments(commentTree).map(c => c.id));
// ['c1', 'c2', 'c3', 'c4']

// 4. BONUS: Memoized count
function createMemoizedCounter() {
  const cache = new Map<string, number>();
  
  function count(comment: Comment): number {
    if (cache.has(comment.id)) {
      console.log(`Cache hit for ${comment.id}`);
      return cache.get(comment.id)!;
    }
    
    const total = 1 + comment.replies.reduce(
      (sum, reply) => sum + count(reply),
      0
    );
    
    cache.set(comment.id, total);
    return total;
  }
  
  return count;
}

const memoizedCount = createMemoizedCounter();
console.log(memoizedCount(commentTree)); // 4
console.log(memoizedCount(commentTree)); // 4 (cached)

// Alternative: Using reduce (more functional)
function countTotalCommentsFunctional(comment: Comment): number {
  return comment.replies.reduce(
    (total, reply) => total + countTotalCommentsFunctional(reply),
    1 // Start with 1 (this comment)
  );
}

console.log(countTotalCommentsFunctional(commentTree)); // 4
```

**Explanation**: 
- `countTotalComments`: Accumulates 1 for each comment recursively
- `findCommentById`: Early returns when found, otherwise searches children
- `flattenComments`: Converts tree to array using depth-first traversal
- Memoized version: Caches results by comment ID to avoid recalculation

</details>

</details>

---

**Resources for Further Learning**:
- Recursion Visualizer: [visualgo.net](https://visualgo.net/en/recursion)
- Tail Call Optimization: [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode#making_eval_and_arguments_simpler)
- Go Recursion Patterns: [Go by Example - Recursion](https://gobyexample.com/recursion)
