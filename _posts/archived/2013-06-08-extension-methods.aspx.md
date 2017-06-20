---
layout: post
title: Extension Methods
redirect_from:
- "/post/extension-methods.aspx/"
tags: [c#, extension, methods]
---

Extension methods were added in C# 3.0. They are syntactic sugar which allows you to define new methods based on existing types without the need for the source code. An extension method is essentially a static method in a static class. 

```csharp
public static class StringExtensions    
{
    public static bool IsUpperCase(this string s)
    {
        return !string.IsNullOrWhiteSpace(s) &&  s.ToUpper() == s;  
    }
}
```

The method above returns true if the string is in upper case and returns false otherwise. The `this` modifier is applied to the first parameter and indicates the type to be extended. You can use it in the following way: 

```csharp
string name = "Nadeem";
     
Console.WriteLine( name.IsUpperCase() );    // False
     
Console.WriteLine( "ABCD".IsUpperCase() ); // True
```

The C# compiler translates the previous calls into the ordinary static method calls: 

```csharp
Console.WriteLine( StringExtensions.IsUpperCase(name) ); // False
     
Console.WriteLine( StringExtensions.IsUpperCase("ABCD") ); // True 
```

The class defining the extension method must be in scope. If not, you have to import the namespace. Extension methods are used heavily in LINQ. In fact, LINQ is made of extension methods defined on the `IEnumerable<T>` interface: 

```csharp
public static TSource First<TSource>(this IEnumerable<TSource> source)
{
        if (source == null) throw new ArgumentNullException ("source");
         
        var list = source as IList<TSource>;
        if (list != null && list.Count > 0) {
                return list[0];              
        }
        else
        {
           foreach(var item in source)
                return item;
        }
         
        throw new Exception("No elements");
}
```

The previous method is a LINQ method that returns the first element of a collection.

```csharp
// Because a string is an array of characters
Console.WriteLine("ABCD".First()); // A
```

Extension methods can take parameters too:

```csharp
// Read n characters from the left of a string
public static string Left(this string source, int length)
{
    return source == null || length > source.Length ? source :  source.Substring(0, length);
}
 
string name =  "Adam";
 
Console.WriteLine(name.Left(3));    // Ada
```

C# provides a static `Format` method which allows you to format a string: 

```csharp
Console.WriteLine(string.Format("hello {0}!", "Adam")); // hello Adam!
```

Some people prefer to call methods on instance types like Python users. Here is a Python-like version of `string.Format`: 

```csharp
public static string format(this string s, params object[] args)
{
            if (args != null && args.Length > 0)
                return string.Format(s, args);
            else
                return s;
} 
 
Console.WriteLine( "hello {0}!".format("Adam")); // hello Adam!
```

I find this version more fluent than the regular `string.Format` version.

Here is an extension method to turn a Boolean value into Yes or No string. You will find this very useful especially in ASP.NET: 

```csharp
public static string ToYesNo (this bool value)  
{
    return value ? "Yes" : "No";
}
bool isUsed = false;
     
Console.WriteLine (isUsed.ToYesNo());   // No
```

Extensions methods can also provide a nice method chaining: 

```csharp
"hello    {0}!".format("Adam").Capitalize().RemoveSpaces();
```

It is important to know that Instance methods will always take precedence over extension methods: 

```csharp
public static string ToUpper(this string s)
{
       return  string.IsNullOrWhiteSpace(s) ? s :  s.ToUpper();
}
 
string s = "my string";
Console.WriteLine (s.ToUpper());        // MY STRING (instance method called).
```

In the previous code, the instance method version was called and not the extension method. That is because a string has an instance method called `ToUpper`. 