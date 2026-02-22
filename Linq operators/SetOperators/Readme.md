# LINQ Set Operators

## 🚀 Overview

Set operators follow the signature:


| Method | Description |
| --- | --- | 
| **`Concat`** | Concatenates elements of two sequences. | 
| **`Union` / `UnionBy`** | Concatenates sequences and removes duplicates. | 
| **`Intersect` / `IntersectBy`** | Returns elements present in both sequences. |
| **`Except` / `ExceptBy`** | Returns elements in the first sequence but not the second. | 

---
## How Set Operations Know if 2 Elements are Equal?

By default, LINQ operations like `Union`, `Intersect`, `Except`, and `Distinct` use a specific hierarchy to decide if two elements are "the same":

### 1. The Default: Reference Equality

.NET compares using `GetHashCode` which compares by reference by default

* **Result:** Two different object instances are considered **different**, even if all their properties (like `Id` or `Name`) are identical.

### 2. The Internal Way: `IEquatable<T>`
.NET Compares using `Equals` method which by default Compares by reference too

If your class implements `IEquatable<T>`, the set operation follows this "Gatekeeper" logic:

1. **The Gatekeeper (`GetHashCode`):** It asks, "What is your Hash Code?" If two objects have different Hash Codes, .NET assumes they are different immediately. It **never** even looks at your `Equals` method.
2. **The Final Judge (`Equals`):** If (and only if) the Hash Codes match, it calls your `Equals(T)` logic to double-check if they are truly the same.

> **That's Why you MUST override GetHashCode:** By default, two different objects in memory have different Hash Codes. If you don't override it, the "Gatekeeper" will see different codes and discard the objects as "unique" before your `IEquatable` logic even has a chance to run.

### 3. The External Way: `IEqualityComparer<T>`

You can pass an object that implements `IEqualityComparer<T>` as an extra argument to the LINQ method.

* **Syntax:** `sequence.Union(otherSequence, new MyCustomComparer())`
* **Best for:** When you can't change the class code (Third-party DLLs), or you need multiple ways to compare (e.g., by `Email` vs. by `Username`).



## 🛠 Usage & Examples

### 1. Concat vs. Union

`Concat` is faster as it simply appends the second collection to the first. `Union` performs a distinct operation on the result.

```csharp
int[] seq1 = { 1, 2, 3 };
int[] seq2 = { 3, 4, 5 };

var concat = seq1.Concat(seq2); // { 1, 2, 3, 3, 4, 5 }
var union  = seq1.Union(seq2);  // { 1, 2, 3, 4, 5 }

```

### 2. Working with Base Types (Variance)
Imagine you have a list of **Apples** and a list of **Oranges**.


### Code Example: Animals

Let's say we have a simple hierarchy where `Dog` and `Cat` both inherit from `Animal`.

```csharp
Dog[] dogs = { new Dog { Name = "Buddy" } };
Cat[] cats = { new Cat { Name = "Whiskers" } };

// This would normally fail because a Cat is not a Dog
// IEnumerable<Animal> family = dogs.Concat(cats); // Compiler Error!

// By stating <Animal> explicitly, we tell LINQ to look at the parent class
IEnumerable<Animal> family = dogs.Concat<Animal>(cats); 

```

### Why do we do this?

When you call `dogs.Concat(cats)`, the compiler looks at the first variable (`dogs`) and assumes the entire resulting list **must** be `Dog`. When it sees `cats` in the second part, it throws an error . 

### 3. "By" Operators

The `By` variants allow you to define a key selector for equality, which is much cleaner than writing a custom `IEqualityComparer`.

```csharp
string[] seq1 = { "A", "b", "C" };
string[] seq2 = { "a", "B", "c" };

// Case-insensitive union using a key selector
var unionBy = seq1.UnionBy(seq2, x => x.ToUpperInvariant()); 
// Result: { "A", "b", "C" }

```

### 4. Intersect & Except

These are essential for finding commonalities or differences between datasets.

```csharp
int[] seq1 = { 1, 2, 3 };
int[] seq2 = { 3, 4, 5 };

var intersect = seq1.Intersect(seq2); // { 3 }
var except    = seq1.Except(seq2);    // { 1, 2 }

```

## 📖 Key Takeaways

* **Performance:** `Concat` is zero memory Overhead , while `Union`, `Intersect`, and `Except` involve hashing Overhead to find same elements.
* **Deferred Execution:** All set operators use deferred execution; the query isn't evaluated until you iterate (e.g., via `foreach` or `.ToList()`).
