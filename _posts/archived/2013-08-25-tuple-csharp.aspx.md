---
layout: post
title: Tuple in C#
redirect_from:
- "post/tuple-csharp.aspx/"
tags: [aspnet, C#, tuple]
---
.NET Framework 4.0 has introduced some new types. Some of them are not popular like `Tuple`. 

A tuple is a generic type that holds different elements, each of which can be of a different data type. A tuple can be useful when returning more than one value from a method. It is not meant to replace your domain model, but rather save you from creating a class for every set of data you use. The .NET Framework has a lot of methods that take advantage of `Tuple`. 

```csharp
public class Tuple <T1>
public class Tuple <T1, T2>
public class Tuple <T1, T2, T3>
public class Tuple <T1, T2, T3, T4>
public class Tuple <T1, T2, T3, T4, T5>
public class Tuple <T1, T2, T3, T4, T5, T6>
public class Tuple <T1, T2, T3, T4, T5, T6, T7>
public class Tuple <T1, T2, T3, T4, T5, T6, T7, TRest>
```

A `Tuple` can be instantiated in two ways:
1.  Constructor 
    ```csharp
    var t = new Tuple<int, string>(30, "Adam");
    ```
2.  Static Method
    ```csharp
    var t = Tuple.Create(30, "Adam");
    ```
The elements can be accessed using the read-only properties Item1, Item2, etc.

```csharp
Console.WriteLine (t.Item1);   // 30
Console.WriteLine (t.Item2.ToUpper());   // ADAM
```

Although you can use an array of objects to accomplish the same task, object arrays are not safely typed. They also have a performance penalty because of boxing/unboxing of value types.

The `==` operator always returns false for two different tuples even if their elements have the same values. However, the `Equals` method is overridden to compare each individual element.

```csharp
var t = Tuple.Create(30, "Adam");
var t2 = Tuple.Create(30, "Adam");
 
Console.WriteLine (t == t2);        // False
Console.WriteLine (t.Equals(t2));   // True
```

You can also implement `IStructureEquatable` for a custom comparison. 