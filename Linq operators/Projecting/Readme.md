# LINQ Projection

# Select (1-to-1 Mapping)

* **Purpose**: Transforms each input element using a lambda expression; the output always has the same number of elements as the input.
**Indexed Projection**: Supports an optional integer argument `(item, index)` to track the element's position (supported only in local queries).

**Query Syntax:**

```csharp
var query = from item in source
            select item.Property; // Or select new { ... }

```

**Fluent Syntax:**

```csharp
// Source: IEnumerable<TSource> -> Result: IEnumerable<TResult>
var query = source.Select(item => item.Property); 

```


# SelectMany  (1-to-Many / Flattening)
* **Purpose**: Concatenates subsequences as For every input element, it can yield 0 to *n* output elements.


**Query Syntax:**
In query syntax, `SelectMany` is invoked by adding an extra `from` clause.

```csharp
var query = from outer in seq1
            from inner in seq2 
            select ....;

```

**Fluent Syntax:**

```csharp
// Flatting
var query = source.SelectMany(outer => outer.ChildCollection);

//OR 

//Flattening with Result Selector
var query = dbContext.Customers.SelectMany(
    c => c.Purchases,                      // 1. Collection Selector: The child sequence
    (c, p) => new { c.Name, p.Description } // 2. Result Selector
);
```



# Object Hierarchies vs. Flattening

## **Hierarchical Projection**

you can think of **Hierarchical Projection** as creating a parent object that contains its own child collection, similar to a JSON tree structure. Because this uses a `Select` subquery

Here is how to achieve this using both syntaxes.

### 📥 Sample Data Context

* **Parents**: `Departments` (Engineering, HR, Marketing)
* **Children**: `Employees` (Only Engineering and HR have employees)

---

### 1. Query Syntax

In query syntax, you nest a second `from` clause directly inside your `select` statement. This is often called a **correlated subquery** because the inner query references the variable `d` from the outer query.

```csharp
var query = from d in dbContext.Departments
            select new 
            {
                DepartmentName = d.Name,
                // The subquery creates the hierarchy
                Staff = from e in d.Employees
                        where e.IsActive
                        select new { e.FullName, e.Position }
            };

```

---

### 2. Fluent Syntax

In fluent syntax, you use the `.Select()` method and define the sub-collection mapping inside the lambda expression. This causes **double-deferred execution** in local queries; the inner collection isn't filtered or projected until you actually enumerate the inner `Staff` list.

```csharp
var query = dbContext.Departments.Select(d => new 
{
    DepartmentName = d.Name,
    // Using a navigation property or a manual filter
    Staff = d.Employees
        .Where(e => e.IsActive)
        .Select(e => new { e.FullName, e.Position })
        .ToList() 
});

```

* **EF Core Optimization**: In EF Core, hierarchical subqueries are processed as a single unit, preventing "N+1" round-tripping to the database.


## **Flattening** 


### How to Achieve Flattening

You can achieve flattening using either **Query Syntax** or **Fluent Syntax** using :

*  Inner Join 
*  Left Outer Join 
>### We explain both below 



# 4. Emulating SQL Joins

## **Inner Join**

Achieved by using `SelectMany` or `from .. from`  clause 

For an **Inner Join** the logic ensures that only matching pairs are returned. If a customer has no orders, they are excluded entirely from the output.

### 📥 Input Data

**Customers**
| ID | Name |
| :--- | :--- |
| 1 | Tom |
| 2 | Jay |
| 3 | Mary |

**Orders** (Linked to Customers)
| ID | CustomerID | OrderDate |
| :--- | :--- | :--- |
| 101 | 1 (Tom) | 2023-01-01 |
| 102 | 1 (Tom) | 2023-05-10 |
| 103 | 2 (Jay) | 2023-06-15 |
| *Note* | *3 (Mary)* | *No Orders* |

---

### 1-The Query syntax

```csharp
var innerJoin = from c in Customers
                from o in c.Orders // Expanding the collection property
                select new { c.Name, o.OrderDate };

```

### 2-The Fluent syntax

```csharp
var query = Customers
    .SelectMany(
        c => c.Orders,          // Inner sequence: navigation property
        (c, o) => new { c.Name, o.Description } // Result selector
    );
```
---

### 📤 Output Result

The result is a **flat** sequence where each row represents a unique combination of a customer and an order. Notice that **Mary** is missing because she has no corresponding orders to "join" with.

| Name | OrderDate |
| --- | --- |
| Tom | 2023-01-01 |
| Tom | 2023-05-10 |
| Jay | 2023-06-15 |

## **Cross Join** (or Cartesian Product)

matches every element of the first collection with every element of the second collection. the second sequence is **completely unrelated** to the first.

### 📥 Input Data

* **Numbers:** `[1, 2]`
* **Letters:** `["A", "B"]`


### 1. Query Syntax

In query syntax, a Cross Join is performed by introducing a second `from` clause that points to a sequence unrelated to the first range variable.

```csharp
int[] numbers = { 1, 2 };
string[] letters = { "A", "B" };

var query = from n in numbers
            from l in letters // 'letters' is not a property of 'n'
            select $"{n}{l}";

```
### 2. Fluent Syntax (`SelectMany`)

In fluent syntax, this uses the `SelectMany` method. The first argument (the collection selector) returns the second sequence without referencing the element from the first sequence.

```csharp
var query = numbers.SelectMany(
    n => letters,                 // The inner sequence is unrelated to 'n'
    (n, l) => $"{n}{l}"           // Result selector: matches every 'n' with every 'l'
);

```

---

### 📤 Output Results

Because every item from the first set is matched with every item from the second set, the result count is always **(Count A × Count B)**.

| Number (n) | Letter (l) | Result (n + l) |
| --- | --- | --- |
| 1 | A | "1A" |
| 1 | B | "1B" |
| 2 | A | "2A" |
| 2 | B | "2B" |



## **Left Outer Join**

In LINQ, `DefaultIfEmpty()` is the essential tool for ensuring a sequence is never empty. It is primarily used to perform **Left Outer Joins** 

#### What  `DefaultIfEmpty()`  actually does

* **Non-empty Sequence**: If the input sequence contains elements, `DefaultIfEmpty()` simply passes them through unchanged.
* **Empty Sequence**: If the sequence is empty, it returns a new sequence containing exactly **one element** with a default value (usually `null` for reference types or `0` for numeric types).

---

In a standard `SelectMany` or `from...from` query, if a parent (e.g., a Customer) has no children (e.g., Purchases), the parent is completely discarded from the result. This is an **Inner Join**.

By applying `DefaultIfEmpty()` to the child sequence, you force LINQ to produce a "placeholder" null if no children exist, ensuring the parent is still included in the final list.

### 1-**Query Syntax**

```csharp
var query = from c in Customers
            // If Purchases is empty, p becomes null instead of the row being skipped
            from p in c.Purchases.DefaultIfEmpty() 
            select new { 
                c.Name, 
                Product = p == null ? "No Purchases" : p.Description 
            };

```

### 2-**Fluent Syntax**

```csharp
var query = Customers.SelectMany(
    c => c.Purchases.DefaultIfEmpty(),
    (c, p) => new { 
        c.Name, 
        Product = p?.Description ?? "No Purchases" 
    }
);

```

### Important Implementation Note

remember that `DefaultIfEmpty()` can be customized. You can provide a specific default value if you don't want the standard `null` or `0`:

```csharp
// If the list is empty, return a list with one item: -1
var result = numbers.DefaultIfEmpty(-1); 

```


---



# 🛠️ Implementation & Efficiency

### Best Practices 

**Local vs. Interpreted**:
* For **Databases**: `Select` and `SelectMany` are the most versatile joining constructs.
* For **Local Queries**: `Join` and `GroupJoin`, They are significantly more efficient for in-memory data processing.

* **Filtering Order**: In local queries, always filter (`Where`) before joining to improve efficiency; in EF Core, the translator often optimizes this regardless of order.

