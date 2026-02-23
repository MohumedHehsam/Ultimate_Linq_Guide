

# 📂 LINQ Operator Categories

LINQ operators are categorized by their input and output shapes. Understanding these signatures is key to mastering it

## 1. Sequence In → Sequence Out (Transformation)

The most common category. These operators accept one or more sequences and return a new sequence.

### 🧩 Shape-Changing & Filtering

| Category | Signature | Description | Operators |
| --- | --- | --- | --- |
| **Filtering** | `Seq<T> → Seq<T>` | Returns a subset of elements. | `Where`, `Take`, `Skip`, `Distinct`, `DistinctBy` |
| **Projecting** | `Seq<T> → Seq<R>` | Transforms elements or flattens hierarchies. | `Select`, `SelectMany` |
| **Joining** | `Seq<T>, Seq<U> → Seq<R>` | Meshes two sequences together. | `Join`, `GroupJoin`, `Zip` |
| **Grouping** | `Seq<T> → Seq<Group>` | Partitions a sequence into buckets. | `GroupBy`, `Chunk` |

### 📂 Set & Ordering

* **Ordering:** Reorders the sequence without changing the data (`OrderBy`, `ThenBy`, `Reverse`).
* **Set Operators:** Performs mathematical set operations on two sequences of the same type (`Union`, `Intersect`, `Except`, `Concat`).
* **Conversion (Import):** Converts non-generic collections to generic sequences (`OfType`, `Cast`).

---

## 2. Sequence In → Element or Scalar Out

These operators "consume" a sequence to produce a single result.

### 🎯 Element Operators

*Signature: `IEnumerable<TSource> → TSource*` Used to pick a specific item from a collection.

> `First`, `FirstOrDefault`, `Last`, `Single`, `SingleOrDefault`, `ElementAt`, `MinBy`, `MaxBy`

### 🧮 Aggregation & Quantifiers

| Type | Description | Operators |
| --- | --- | --- |
| **Aggregation** | Returns a numeric or computed scalar. | `Count`, `Sum`, `Average`, `Min`, `Max`, `Aggregate` |
| **Quantifiers** | Returns a `bool` based on a condition. | `Any`, `All`, `Contains`, `SequenceEqual` |

---

## 3. Void → Sequence (Generation)

These methods are static members of the `Enumerable` class and manufacture a sequence from scratch.

*Signature: `void → IEnumerable<TResult>*`

* **`Enumerable.Empty<T>()`**: Returns an empty sequence.
* **`Enumerable.Range(start, count)`**: Generates a sequence of integral numbers.
* **`Enumerable.Repeat(element, count)`**: Generates a sequence that contains one repeated value.


## 🛠️ Export Methods

These operators are used to "materialize" a LINQ query into a concrete data structure or change its execution behavior.

* **To Collections:** `ToList`, `ToArray`, `ToDictionary`, `ToLookup`.
* **To Wrappers:** `AsEnumerable` (hides provider-specific methods), `AsQueryable` (converts to `IQueryable` for EF Core).

---

*Click a category to jump to the detailed explanation and code examples.*

### 1. [Filtering](Filtering/)

* **Operators:** `Where`, `Take`, `Skip`, `Distinct`, `Chunk`.
* **Goal:** Slicing datasets and implementing efficient pagination.

### 2. [Projecting](Projecting/)

* **Operators:** `Select`, `SelectMany`.
* **Goal:** Transforming objects and shaping the output data.

### 3. [Joining](Joining/)

* **Operators:** `Join`, `GroupJoin`.
* **Goal:** Flattening nested collections and mastering many-to-many relationships.

### 4. [Ordering](Ordering/)

* **Operators:** `OrderBy`, `ThenBy`, `Reverse`.
* **Goal:** Managing data sequences and complex object sorting.

### 5. [Grouping](Grouping/)

* **Operators:** `GroupBy`, `ToLookup`.
* **Goal:** Organizing data into keys and creating "HAVING" logic equivalents in C#.

### 6. [Set Operators](SetOperators/)

* **Operators:** `Union`, `Intersect`, `Except`.
* **Goal:** Comparing two lists or finding unique commonalities.

### 7. [Conversion Methods](Conversion%20Operators/)

* **Operators:** `ToList`, `ToArray`, `ToDictionary`, `AsEnumerable`, `AsQueryable`.
* **Goal:** Changing the underlying storage type or changing query execution behavior.

### 8. [Element Operators](Element%20Operators/)

* **Operators:** `First`, `FirstOrDefault`, `Last`, `Single`, `ElementAt`.
* **Goal:** Extracting a specific, single record safely from a sequence.

### 9. [Aggregation Methods](Aggregate%20Methods/)

* **Operators:** `Count`, `Sum`, `Min`, `Max`, `Average`, `Aggregate`.
* **Goal:** Summarizing a collection into a single numerical or calculated value.

### 10. [Quantifiers](Quantifier/)

* **Operators:** `Any`, `All`, `Contains`.
* **Goal:** Validating the presence of data or checking if a collection meets specific criteria.

### 11. [Generation Methods](Generator%20Methods/)

* **Operators:** `Range`, `Repeat`, `Empty`.
* **Goal:** Generating sequences of data programmatically for testing or logic flow.




