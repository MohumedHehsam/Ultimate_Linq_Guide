# LINQ Filtering Operators & SQL Equivalents

The following table maps common **LINQ** methods to their **SQL** counterparts. Note that some methods do not have a direct translation in standard SQL providers and may throw an exception if used in an `IQueryable` context.

| Method | Description | SQL Equivalents |
| --- | --- | --- |
| **`Where`** | Returns a subset of elements that satisfy a given condition. | `WHERE` |
| **`Take`** | Returns the first count elements and discards the rest. | `WHERE ROW_NUMBER()...` or `TOP n` subquery |
| **`Skip`** | Ignores the first count elements and returns the rest. | `WHERE ROW_NUMBER()...` or `NOT IN (SELECT TOP n...)` |
| **`TakeLast`** | Takes only the last count elements. | *Exception thrown* |
| **`SkipLast`** | Ignores the last count element. | *Exception thrown* |
| **`TakeWhile`** | Emits elements from the input sequence until the predicate is false. | *Exception thrown* |
| **`SkipWhile`** | Ignores elements from the input sequence until the predicate is false, and then emits the rest. | *Exception thrown* |
| `Distinct`, `DistinctBy` | Returns a sequence that excludes duplicates. | `SELECT DISTINCT...` |

---

### 💡 Pro-Tip

Methods labeled as "**Exception thrown**" are generally intended for **LINQ to Objects** (in-memory collections). If you need to use these with a database, you must usually call `.AsEnumerable()` or `.ToList()` first to bring the data into memory—though be careful with large datasets!

---
Below are practical examples of the common LINQ methods. For these examples, assume we are working with the following data:
`int[] numbers = { 1, 2, 3, 4, 5, 1, 2 };`

### 1. Where

Returns a subset of elements that satisfy a given condition.

```csharp
var result = numbers.Where(n => n > 3);

// Output: [4, 5]

```

### 2. Take

Returns the first count elements and discards the rest.

```csharp
var result = numbers.Take(3);

// Output: [1, 2, 3]

```

### 3. Skip

Ignores the first count elements and returns the rest.

```csharp
var result = numbers.Skip(4);

// Output: [5, 1, 2]

```

### 4. TakeLast

Takes only the last count elements.

> **Note:** Typically throws an exception in `IQueryable` (EF Core).

```csharp
var result = numbers.TakeLast(2);

// Output: [1, 2]

```

### 5. SkipLast

Ignores the last count elements.

> **Note:** Typically throws an exception in `IQueryable` (EF Core).

```csharp
var result = numbers.SkipLast(2);

// Output: [1, 2, 3, 4, 5]

```

### 6. TakeWhile

Emits elements from the input sequence *until* the predicate is false.

```csharp
// Stops as soon as it hits '4'
var result = numbers.TakeWhile(n => n < 4);

// Output: [1, 2, 3]

```

### 7. SkipWhile

Ignores elements from the input sequence *until* the predicate is false, and then emits the rest.

```csharp
// Skips until it hits '4', then returns everything else
var result = numbers.SkipWhile(n => n < 4);

// Output: [4, 5, 1, 2]

```

### 8. Distinct & DistinctBy

Returns a sequence that excludes duplicates.

```csharp
// Distinct: Removes duplicates from the whole collection
var distinctNumbers = numbers.Distinct();
// Output: [1, 2, 3, 4, 5]

// DistinctBy: Removes duplicates based on a specific property (Key)
var distinctItems = items.DistinctBy(i => i.Id);

```

---