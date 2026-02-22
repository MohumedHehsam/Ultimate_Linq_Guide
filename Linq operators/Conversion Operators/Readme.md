# LINQ Conversion Methods

## Conversion Methods

| Method | Execution |
| --- | --- | 
| **`OfType<T>`** |  Deferred |
| **`Cast<T>`** |  Deferred |
| **`ToArray`** |Immediate |
| **`ToList`** | Immediate |
| **`ToDictionary`** | Immediate |
| **`ToLookup`** | Immediate |
| **`AsEnumerable`** |  |
| **`AsQueryable`** |  |


## 1. `OfType<T>` vs. `Cast<T>`

Both methods are used to convert a non-generic `IEnumerable` (like an `ArrayList`) into a generic `IEnumerable<T>` that can be queried with LINQ.

### Key Differences

The primary difference lies in how they handle incompatible types:

* **`Cast<T>`**: Expects every element to be compatible with `T`. If not, it throws an `InvalidCastException`.
* **`OfType<T>`**: Filters the collection. It only returns elements that are compatible with `T`, skipping the rest.

```csharp
ArrayList list = new ArrayList { 3, 4, "invalid", 5 };

// Throws Exception when it hits "invalid"
IEnumerable<int> casted = list.Cast<int>(); 

// Returns { 3, 4, 5 } - ignores "invalid"
IEnumerable<int> filtered = list.OfType<int>(); 

```
### Implementation Logic

The compatibility check follows C#'s `is` operator, supporting only **reference conversions** and **unboxing conversions**.

> Note : You cannot use `Cast<T>` for numeric conversions (e.g., `long` doesn't inherit `int`). Use `.Select(x => (long)x)` instead.
```CSharp
int[] numbers = { 1, 2, 3 };

// WRONG: This throws an exception!
// It tries to do: (long)(object)1
IEnumerable<long> bad = numbers.Cast<long>();

// RIGHT: Use Select
// This performs a standard numeric conversion (int -> long)
IEnumerable<long> good = numbers.Select(n => (long)n);
```

## 2. Creating Collections (`ToArray`, `ToList`, `ToDictionary`, `ToLookup`)

These methods trigger **Immediate Execution**, meaning the query is evaluated and the results are stored in memory immediately.

### `ToDictionary` vs. `ToLookup`



**Arguments for both:**
1. **Key Selector**: `TSource => TKey`
2. **Element Selector** (Optional): `TSource => TElement`
3. **Comparer** (Optional): `IEqualityComparer<TKey>`

### **`ToDictionary`**: Requires each key to be **unique**. If a duplicate key is encountered, it throws an exception.

```Csharp
var users = new[] 
{ 
    new { Id = 1, Name = "Alice" }, 
    new { Id = 2, Name = "Bob" } 
};

// Creates Dictionary<int, string>
var userMap = users.ToDictionary(u => u.Id, u => u.Name);

Console.WriteLine(userMap[1]); // Output: Alice
```

### **`ToLookup`**: Allows **multiple elements** to be grouped under the same key , **Immutable** , return **Empty sequence** if key isn't found.

```Csharp
var products = new[] 
{ 
    new { Category = "Electronics", Name = "Phone" }, 
    new { Category = "Electronics", Name = "Laptop" }, 
    new { Category = "Food", Name = "Apple" } 
};

// Creates ILookup<string, string>
var categoryLookup = products.ToLookup(p => p.Category, p => p.Name);

// Returns a sequence containing "Phone" and "Laptop"
var electronics = categoryLookup["Electronics"]; 

// Returns an EMPTY sequence (count = 0)
var toys = categoryLookup["Toys"];
```
---

## 3. `AsEnumerable` and `AsQueryable`

* **`AsEnumerable`**: Effectively hides the specialized interface of a collection (like `IQueryable` in Entity Framework) and treats it as a local `IEnumerable`. This is useful for forcing the remaining part of a query to run in-memory (LINQ to Objects) rather than on a database server.
* **`AsQueryable`**: Downcasts an `IEnumerable` to `IQueryable` if the underlying provider supports it.