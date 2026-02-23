# LINQ Quantifiers

Quantifiers are used to determine whether some or all elements in a sequence satisfy a specific condition. They return a `bool` value.


# 1. `Contains` vs. `Any`

While they can often achieve the same result, `Contains` looks for a specific **value**, whereas `Any` looks for a **condition**.

### Contains

Returns `true` if the given element is present in the collection.

```csharp
bool hasAThree = new int[] { 2, 3, 4 }.Contains(3); 
// Result: true

```

### Any

Returns `true` if the expression is true for at least one element. It is more flexible than `Contains`.

```csharp
// Equivalent to Contains
bool hasAThree = new int[] { 2, 3, 4 }.Any(n => n == 3); 

// Advanced condition
bool hasABigNumber = new int[] { 2, 3, 4 }.Any(n => n > 10); 
// Result: false

```

> **Note:** Calling `.Any()` without a predicate simply checks if the sequence contains at least one element. It is generally more efficient than checking `.Count() > 0`.

---

# 2. `All`

`All` returns `true` only if **every** element in the sequence satisfies the predicate. If the sequence is empty, `All` returns `true` (this is known as a "vacuous truth").

```csharp
// Returns customers where EVERY purchase is under $100
var budgetCustomers = dbContext.Customers
    .Where(c => c.Purchases.All(p => p.Price < 100));

```

---

# 3. `SequenceEqual`

Compares two sequences for equality. To return `true`:

1. Both sequences must have the same number of elements.
2. Elements must be identical.
3. Elements must be in the **same order**.

```csharp
var seq1 = new int[] { 1, 2, 3 };
var seq2 = new int[] { 1, 2, 3 };
var seq3 = new int[] { 3, 2, 1 };

seq1.SequenceEqual(seq2); // true
seq1.SequenceEqual(seq3); // false (different order)

```
it first Compares using `HasCode` if they are same then compares using  `Equals` method ,but you can also override that by passing a `IEqualityComparer<T>` object
