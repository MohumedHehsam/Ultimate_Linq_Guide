# LINQ Generation Methods

# 1. `Enumerable.Empty<T>`

returns an empty sequence. This is more memory-efficient than returning a `new List<T>()` because it returns a cached internal singleton instance.

### The "Null-Safety" Pattern

The most powerful use case for `Empty` is in conjunction with the null-coalescing operator (`??`) to prevent `NullReferenceException` during flattening operations.

**The Problem:**

```csharp
int[][] numbers = { new int[] { 1, 2 }, null };

// ❌ This line CRASHES because it hits the null
var flat = numbers.SelectMany(x => x);
```

**The Solution:**

```csharp
var flat = numbers.SelectMany(x => x ?? Enumerable.Empty<int>());

foreach (int i in flat)
{
    Console.Write(i + " "); 
}
// ✅ Output: 1 2

```

---

# 2. `Enumerable.Range` vs. `Enumerable.Repeat`

These methods are useful for initializing test data, creating loops, or generating sequences for mathematical operations.

### Enumerable.Range

**Example: Generating a list of years for a dropdown**

```csharp
// Result: 2020, 2021, 2022, 2023, 2024
IEnumerable<int> years = Enumerable.Range(2020, 5);

```

---

### Enumerable.Repeat

**Example: Creating a set of "Pending" status icons**

```csharp
// Result: "Pending", "Pending", "Pending"
var statuses = Enumerable.Repeat("Pending", 3);

```

---
> `Enumerable.Range` and `Enumerable.Repeat` works also in Deferred Exection 

> **Pro Tip:** Use `Enumerable.Range` to replace traditional `for` loops when you want to use LINQ syntax immediately on the index, such as `Enumerable.Range(0, 10).Select(i => new User(i))`.
