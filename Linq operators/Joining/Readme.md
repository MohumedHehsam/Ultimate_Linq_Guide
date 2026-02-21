# 🏗️ Core Joining Concepts
# 1. `Join` (inner joining)
### makes `Inner Joins` Only. If there is no match for an item in the outer collection, that item is discarded. It produces a `Flat result set`.

#### **📥 Input Data**

| Managers (Outer) | ID | OfficeId |  | Offices (Inner) | OfficeId | Location |
| --- | --- | --- | --- | --- | --- | --- |
| Alice | 1 | **10** |  | Office A | **10** | New York |
| Bob | 2 | **20** |  | Office B | **20** | London |
| Charlie | 3 | **99** |  | Office C | **30** | Tokyo |

#### **💻 Implementation**

**Query Syntax:**

```csharp
var query = from m in managers
            join o in offices on m.OfficeId equals o.OfficeId
            select new { m.Name, o.Location };

```

**Fluent Syntax:**

```csharp
var query = managers.Join(
    offices,                    // Inner collection
    m => m.OfficeId,            // Outer key selector
    o => o.OfficeId,            // Inner key selector
    (m, o) => new { m.Name, o.Location } // Result selector
);

```

#### **📤 Output Result**

| Name | Location |
| --- | --- |
| Alice | New York |
| Bob | London |

---

# 2. `GroupJoin` (Hierarchical Join)

### Makes `Left Outer Join` because it returns every item from the outer sequence. If an item has no matches, it gets an empty collection. It produces a `Hierarchical result set`.

#### **📥 Input Data**

| Authors (Outer) | ID | Name |  | Books (Inner) | AuthorId |
| --- | --- | --- | --- | --- | --- |
| 1 | Stephen King |  |  | 1 | IT |  
| 2 | J.K. Rowling |  |  | 1 | The Shining |  
| 3 | New Author |  |  | 2 | Harry Potter |  

#### **💻 Implementation**

**Query Syntax:**

```csharp
var query = from a in authors
            join b in books on a.ID equals b.AuthorId into authorBooks
            select new { a.Name, Books = authorBooks };

```

**Fluent Syntax:**

```csharp
var query = authors.GroupJoin(
    books,                      // Inner collection
    a => a.ID,                  // Outer key selector
    b => b.AuthorId,            // Inner key selector
    (a, authorBooks) => new {   // Result selector (authorBooks is IEnumerable<Book>)
        a.Name, 
        Books = authorBooks 
    }
);

```

#### **📤 Output Result**

| Name | Books (Nested Collection) |
| --- | --- |
| Stephen King | `["IT", "The Shining"]` |
| J.K. Rowling | `["Harry Potter"]` |
| New Author | `[]` (Empty List) |

---

### 📝 Key Takeaways for Syntax

* **Equals Keyword**: In Query Syntax, you **must** use the `equals` keyword for joins; the `==` operator will cause a compiler error.
* **Result Selector**: In Fluent Syntax, the last argument is a lambda that defines how to combine the two matched objects into your final result.
* **Into Clause**: In Query Syntax, the `into` keyword is what transforms a standard `join` into a `GroupJoin`.
# 3. `Zip` Operator

###  `Zip` groups two input sequences and applies a specified function to each pair to produce a single result.

* **Handling Mismatched Lengths**: If one sequence is longer than the other, any extra elements in the 
* **Database Support**: `Zip` is not supported by **EF Core**

```csharp
int[] numbers = { 3, 5, 7 };
string[] words = { "three", "five", "seven", "eight" };

// Zipping the two arrays together
IEnumerable<string> zip = numbers.Zip(words, (n, w) => n + "=" + w);

```

**Output:**

* `3=three`
* `5=five`
* `7=seven`
* (Note: "eight" is discarded because there is no matching number in the first array.)*

---


# 🔍 Key Implementation Details

### Joining on Multiple Keys

join using multiple keys by projecting them into **anonymous types**. For this to work, both anonymous types must be structured **identically** .

```csharp
//Query Syntax
join y in sequenceY on new { K1 = x.Prop1, K2 = x.Prop2 } 
                equals new { K1 = y.Prop3, K2 = y.Prop4 }
                ....

//Fluent Syntax
var query = sales.Join(
    targets,
    s => new { Y = s.Year, R = s.RegionID },
    t => new { Y = t.Year, R = t.RegionID }
    ...

```

# What is a Lookup?

A **Lookup** (`ILookup<TKey, TElement>`) is a data structure that maps a single key to a **sequence** of values.

* The key can have multiple values.
* Unlike a standard `Dictionary`, it is **read-only** 
* if a Key doesn't exist it simply returns an empty sequence.

---
Why we care about Lookup anyway ?


1. **Build Lookup**: The entire inner sequence is loaded into a read-only `ILookup<TKey, TElement>` (a "multidictionary").
2. **Query**: The outer sequence is then queried against this lookup.

The `Join` and `GroupJoin` methods in `Enumerable` (local queries) depend on lookups internally wich makes their Time Complexity **O(N + M)** instead of **O(N^2)** : where N and M are inner and outer sequence length

---

> ## Example :-
### 📥 The Input Data

Imagine you have a flat list of tasks, and each task belongs to a project.

| Task Name | ProjectID |
| --- | --- |
| Database Setup | 1 |
| API Design | 1 |
| UI Mockups | 2 |
| Unit Testing | 1 |

---

### 1. Creating the Lookup (`ToLookup`)
Creation with `.ToLookup()`

You create a lookup using the `.ToLookup()` extension method. It takes two primary lambdas:

1. **Key Selector**: What the key of lookup? (e.g., `ProjectId`)
2. **Element Selector**: What data do we want to store? (e.g., the whole `Name` of the Task)




```csharp

var tasks = new[] {
    new { Name = "Database Setup", ProjectId = 1 },
    new { Name = "API Design", ProjectId = 1 },
    new { Name = "UI Mockups", ProjectId = 2 },
    new { Name = "Unit Testing", ProjectId = 1 }
};


ILookup<int,string>  lookup =tasks.ToLookup(x=>x.ProjectId,t=>t.Name);

foreach(IGrouping<int,string> item in lookup)
{
    Console.WriteLine("key:"+item.Key);
    
    foreach(var val in item)
    Console.Write(val+",");

    Console.WriteLine("\n-------------");
}

//Output
key:1
Database Setup,API Design,Unit Testing,
-------------
key:2
UI Mockups,
-------------

```

---

### 2. Accessing Values Manually

Because it's a lookup, accessing a key gives you a **collection**, even if there is only one item or no items.

```csharp
var p1 = projectTasks[1]; 
// Output: ["Database Setup", "API Design", "Unit Testing"]

var p2 = projectTasks[2]; 
// Output: ["UI Mockups"]

var p99 = projectTasks[99]; 
// Output: [] (An empty sequence, NO Exception thrown!)

```


