# LINQ Aggregation Methods

Aggregation operators perform a mathematical operation over a sequence of values and return a single scalar value.

## Summary Table

| Method | Description | SQL Equivalent |
| --- | --- | --- |
| `Count`, `LongCount` | Returns the number of elements (optionally matching a predicate). | `COUNT(...)` |
| `Min`, `Max` | Returns the smallest or largest element. | `MIN(...)`, `MAX(...)` |
| `Sum`, `Average` | Calculates a numeric sum or average. | `SUM(...)`, `AVG(...)` |
| `Aggregate` | Performs a custom accumulation algorithm. | *N/A* |

---

# 1. `Count` & `LongCount`

`Count` enumerates a sequence to return the number of items.

* **Optimization:** If the source implements `ICollection<T>`, LINQ uses the `.Count` property instead of iterating.
* **Predicate:** You can pass a filter directly into the method.

```csharp
int fullCount = new int[] { 1 , 2 , 3 , "Not Digit" }.Count(); // 4

// Using a predicate
int digitCount = "pa55w0rd".Count(c => char.IsDigit(c)); // 3

```

---

# 2. `Min` & `Max`

These return the extreme values in a sequence.

### Basic Usage

```csharp
int[] numbers = { 28, 32, 14 };
int smallest = numbers.Min(); // 14

```

### With Selectors

> elements or selector key must implement `IComparable<T>`

```csharp
// This would throw a runtime error if Purchases isn't comparable
var error = dbContext.Purchases.Min(); 

// Correct usage: project to a numeric type
decimal? lowestPrice = dbContext.Purchases.Min(p => p.Price);

```

---

# 3. `Sum` & `Average`

works only with numeric input types (`int`, `long`, `float`, `double`, `decimal`) and their nullables.

```csharp
// Average implicitly upscales to prevent precision loss
double avg = new int[] { 3, 4 }.Average(); // 3.5

```

### With Selectors
```csharp
var orders = new List<Order>
{
    new Order { OrderId = 1, TotalAmount = 50.00m },
    new Order { OrderId = 2, TotalAmount = 150.00m },
    new Order { OrderId = 3, TotalAmount = 10.00m }
};

// 1. Using a Selector to get the average price
decimal averagePrice = orders.Average(o => o.TotalAmount); // 70.00```
```
---

# 4. `Aggregate` (Custom Aggregation)

`Aggregate` is used for specialized logic not covered by standard methods.

> **Warning** : `Aggregate` is **not supported** in EF Core and is generally intended for local collections (LINQ to Objects).

### Seeded vs. Unseeded

* **Seeded:** Starts with a defined initial value.
* **Unseeded:** without initial value , considers first element as a seed

```csharp
int[] numbers = { 1, 2, 3 };

// Seeded: -2 + 1 + 2 + 3
int seeded = numbers.Aggregate(-2, (total, n) => total + n); // 4

// Unseeded: 1 * 2 * 3
int unseeded = numbers.Aggregate((prod, n) => prod * n); // 6

```

### The Math Logic

* **Seeded:** If you sum `{1, 2, 3}` with seed `-2`, LINQ calculates: `-2 + 1 + 2 + 3 = 4`.
* **Unseeded:** LINQ calculates: `1 + 2 + 3 = 6`.


## Why it Fails in Complex Logic

The danger arises when your function is not **associative**  or **commutative** 

### Example: Sum of Squares

Imagine you want to calculate the sum of squares for the array `{2, 3, 4}`. The goal is .

If you use an unseeded aggregate:

```csharp
int[] numbers = { 2, 3, 4 };
int sum = numbers.Aggregate((total, n) => total + n * n); // Output : 27 not 29

```

**Here is what happens step-by-step:**

1. **Seed Selection:** LINQ grabs the first element: `total = 2`.
2. **Step 2:** It start processesing from 2nd element: `2 + (3 * 3) = 11`.
3. **Step 3:** It processes the last element: `11 + (4 * 4) = 27`.

## The PLINQ Nightmare (Parallelism)

When you use `.AsParallel()`, the sequence is split into "partitions" processed by different CPU cores. In an unseeded aggregation, **each partition picks its own seed**.

## How to Fix It

As a best practice in backend development:

1. **Always use a Seed:** This ensures the first element is also processed by your logic.
```csharp
numbers.Aggregate(0, (total, n) => total + n * n); // Result: 29

```

2. **Use Built-in Methods:** Methods like `Sum` and `Average` are safe for both LINQ and PLINQ.

```csharp
numbers.Sum(n => n * n); // Cleanest and safest

```


