# LINQ Grouping & Chunking Operations

# 1.`GroupBy`

**`GroupBy`** transforms a **`IEnumerable<T>`** into a **`IEnumerable<Igrouping<TKey,TElement>>`**  where each Element is a Group with a unique **Key** and contains a collection of **Elements**.

#### Fluent Syntax
```csharp

var fruits = new List<string> { "apple", "apricot", "banana" };

var groups = fruits.GroupBy(
    f => f[0],           // 1. Key Selector (First char)
    f => f.ToUpper()     // 2. Element Selector (Transform to Upper)
);


```
#### Query Syntax
```Csharp
var query = from f in fruits
            group f.ToUpper() by f[0];
            //    ^ Element      ^ Key
            //    Selector       Selector

```
```Csharp
 OUTPUT:
Key: 'a' 
  Elements: "APPLE", "APRICOT"
Key: 'b' 
  Elements: "BANANA"

```

### ⚠️ Remember : There is a difference between `Key Selector` and `Element Selector` , The Key is not Transformed by Element Selector in the Output

---

### The `IGrouping` Interface
### Note : `GroupBy` returns `IEnumerable<IGrouping<TKey, TElement>>`.


```csharp
public interface IGrouping <out TKey, out TElement> : IEnumerable<TElement>, IEnumerable
{
    TKey Key { get; } 
}

```
###  Each `IGoruping` has `Key` of each Group , also `IGrouping` Implements `IEnumerable<TElement>` so you can use foreach to loop the Group
```csharp
// Each 'group' here is an IGrouping<int, string>
foreach (var group in query) 
{
    // 1. The Key: The single value we grouped by (e.g., DeptId)
    int departmentId = group.Key; 

    // 2. The Collection: The group ITSELF is the list of elements
    foreach (string name in group) 
    {
        Console.WriteLine(name);
    }
}
```
---

## Advanced Grouping Techniques

### Filter After Grouping (Like `Having` in SQL)

In SQL, you use `HAVING` to filter groups. In LINQ, you use a `where` clause after an `into` continuation.

```csharp
var query = from file in files
            group f.ToUpper() by f[0] into g
            where g.Count() >= 5
            select g;
```
> ### Remember : you can end the Query at `group f.ToUpper() by f[0] into g` , But  to continue the Query like this example you have to use `into` 

### Grouping by Multiple Keys

Use **anonymous types** to create composite keys.

```csharp
var query = from n in names
            group n by new { FirstLetter = n[0], Length = n.Length };

```
---
## Another Limitaion for `GroupBy`
This limitation exists because of a "language barrier" between **C# (LINQ)** and **SQL**.

In C#, a `GroupBy` creates a **hierarchical structure** (a parent "Key" with a child "List"). SQL, however, is designed to return **flat rows and columns**.

### The Scenario

Imagine you have a `Purchases` table. You want to group them by **Year**, and for each year, you want the **List of Product Names** sold.

### ⛔ The "Illegal" Query (doesn't work in EF Core)

If you try to do this entirely on the database side:

```csharp
var query = dbContext.Purchases
    .GroupBy(p => p.Date.Year)
    .Select(group => new 
    {
        Year = group.Key,
        Products = group.Select(p => p.ProductName).ToList() // <--- THE PROBLEM
    })
    .ToList();

```

**Why it fails:**
EF Core tries to translate this into a single SQL statement. But SQL's `GROUP BY` cannot return a "List" inside a column. It only knows how to return a single value per group (like `SUM` or `COUNT`). it throws an exception:

---

### ✅ use only "Aggregated" Query (works in EF Core)

If you only want **numbers**, EF Core translates it perfectly because SQL understands math on groups.

```csharp
var query = dbContext.Purchases
    .GroupBy(p => p.Date.Year)
    .Select(group => new 
    {
        Year = group.Key,
        TotalSales = group.Sum(p => p.Price), // SQL loves Sum()
        ItemsCount = group.Count()           // SQL loves Count()
    })
    .ToList();

```

---

### ✅ The "Workaround" (Client-Side Evaluation)

```csharp
var result = dbContext.Purchases
    .Where(p => p.Date.Year > 2020) // (1) SQL filters the rows
    .AsEnumerable()                // (2) Now work in C# memory
    .GroupBy(p => p.Date.Year)     
    .Select(group => new 
    {
        Year = group.Key,
        Products = group.Select(p => p.ProductName).ToList() 
    })
    .ToList();

```

> ### Make Sure to perform filtering before grouping so that you only fetch the data you need from the server.
---

# 2.`Chunk`

`Chunk` divides source sequence and into arrays of the specified `size`


###  Example :
This is the simplest use case: taking a flat list and breaking it into smaller arrays.

```csharp
int[] numbers = { 1, 2, 3, 4, 5, 6, 7, 8 };

int[] chunk = numbers.Chunk(3)

    // Output: IEnumerable<int[]>
    // [1, 2, 3]  [4, 5, 6]  [7, 8]

```