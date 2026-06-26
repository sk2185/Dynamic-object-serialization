# Dynamic Object Serialization in C# with Reflection

Serialize any object to delimited text (CSV, `;`-separated, etc.) **without hardcoding property names**. Add or remove a property on your model and the output adapts automatically.

## The Model

```csharp
public class Employee
{
    public int EmpId { get; set; }
    public string EmpName { get; set; }
    public string Dob { get; set; }
    public string Mob { get; set; }
}
```

## 1. Serialize all properties (declaration order)

`GetProperties()` makes no ordering guarantee across runtimes. Sorting by
`MetadataToken` gives reliable source-declaration order for a single class.

```csharp
using System;
using System.Linq;
using System.Reflection;

var props = typeof(Employee).GetProperties()
    .OrderBy(p => p.MetadataToken)
    .ToArray();

var lines = employees.Select(e =>
    string.Join(";", props.Select(p => Convert.ToString(p.GetValue(e)))));

string output = string.Join(Environment.NewLine, lines);
```

**Output**

```
1001;name1;01/01/2005;015444545
1002;name2;01/05/2004;054545454
```

## 2. Exclude specific properties

```csharp
string[] exclude = { "Dob", "Mob" };

var props = typeof(Employee).GetProperties()
    .Where(p => !exclude.Contains(p.Name))
    .OrderBy(p => p.MetadataToken)
    .ToArray();
```

## 3. Custom order + renamed headers

When the column order or header names should differ from the property names,
map them explicitly instead of relying on reflection order.

```csharp
var order = new[]
{
    (Header: "ID",    PropName: "EmpId"),
    (Header: "Name",  PropName: "EmpName"),
    (Header: "Dob",   PropName: "Dob"),
    (Header: "Phone", PropName: "Mob")
};

var props  = order.Select(o => typeof(Employee).GetProperty(o.PropName)).ToArray();
string header = string.Join(";", order.Select(o => o.Header));

var lines = employees.Select(e =>
    string.Join(";", props.Select(p => Convert.ToString(p.GetValue(e)))));

string output = header + Environment.NewLine + string.Join(Environment.NewLine, lines);
```

**Output**

```
ID;Name;Dob;Phone
1001;name1;01/01/2005;015444545
1002;name2;01/05/2004;054545454
```

## Notes & Tradeoffs

- **Caching:** Resolve the `PropertyInfo[]` once (as above) rather than per row — reflection lookups are comparatively expensive.
- **Hot paths:** For high-throughput serialization, prefer a source generator or a library (e.g. `CsvHelper`) over runtime reflection.
- **Null handling:** `Convert.ToString(null)` returns an empty string, which is usually what you want for delimited output.
- **Case-insensitive exclude:** use `exclude.Contains(p.Name, StringComparer.OrdinalIgnoreCase)`.
- **Inheritance:** `MetadataToken` ordering is reliable within one class; mixing base + derived members has no guaranteed order.

## Online Compiler
https://onecompiler.com/csharp/44tecj3zd


## License

MIT
