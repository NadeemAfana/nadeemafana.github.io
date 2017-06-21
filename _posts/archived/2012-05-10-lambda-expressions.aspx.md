---
layout: post
title: Lambda Expressions
redirect_from:
- "/post/lambda-expressions.aspx.html"
- "/post/lambda-expressions.aspx/"
tags: [c#, lambda]
---
A lambda expression is an anonymous function (ie a function without a name). It is an inline expression or statement that can be used wherever a delegate type is accepted. There are two types of lambdas: Expression lambdas and statement lambdas.

An expression lambda has the following form:
![](/images/posts/archived/lambda-expressions-1.png)

The `=>` is called the lambda operator and is read as "goes to". The left side specifies the input parameters (if any), and the right side holds the expression or statements (in case of a statement lambda).

For example
```csharp
(a, b) => a + b
```

The previous lambda accepts two input parameters (a and b) and sums the two of them `(a + b)`.

The previous lambda accepts two input parameters (a and b) and sums the two of them `(a + b)`.

The following lambda squares an integer

```csharp
a => a * a
```

When there is only one input parameter, you can omit the parenthesis. Also, a lambda expression can invoke another method:

```csharp
() => callMethod()
```

The previous lambda takes no input and invokes the method `callMethod`.

he input parameters data type can be inferred from the context. However, sometimes it is hard to guess the right data type so in that case you must specify it

```csharp
(char c, string s) => s[0] == c;
```

The lambda above tests if a string starts with a specified character. The return value from the right side part (expression) is obviously Boolean.

The second type of lamda is called a statement lambda. A statement lambda can have statements in its body.

```csharp
x => { return 2 * x; }    
```

In fact, this lambda returns double of the input x, and it is equivalent to the following lambda

```csharp
x => 2 * x          
```

### All the previous examples can be done using regular C# methods, so what are Lambda expressions for?

A lambda expression is a convenient way to initialize a delegate. Recall that a delegate is a data type that stores an address to a method. It is like a function pointer in C++, but a delegate is type safe. You may have heard of events in Windows Forms or ASP.NET WebForms. In fact, an Event is a special type of delegate. For example, the Button class offers a Click event, you would define a handler method that is invoked whenever this event is fired.

A delegate can also be used to pass methods as arguments to other methods. For example, LINQ methods are a place where you would find lambda expressions very convenient.

```csharp
var numbers = Enumerable.Range(1, 20); // Generate integers from 1 to 20
 
int[] evens = numbers.Where(n => n % 2 == 0).ToArray(); // Only get even integers only         
```

The lambda expression `n => n % 2 == 0` takes an integer as input and returns true if the number is even and false if the number is odd.

The following lambda expression doubles a set of numbers

```csharp
int[] doubles = numbers.Select(n => n * 2).ToArray();
```

### Func and Action delegates

If you hover the mouse over the Select method above, you will see the dollowing delegate as the input type

```csharp
Func<int, int> selector
```

Instead of creating your own custom delegates with different input parameters and return data type, the .NET Framework provides for you a flexible set of generic delegates, and these are `Func` and `Action`. The `Action` delegate references a method that has no return value (void), whereas `Func` references a method that has a return value.

These generic delegates can take up to 16 parameters of different data types. In the case of Func, the last parameter is always the return data type and all the previous ones are considered input parameters.

The following table helps you understand how they work

|*Delegate*|*Description*|
|----------|--------------|
|`Func<int>`  |Takes no input. Returns an integer.|
|`Func <bool, string>`| Takes a bool value. Returns a string.|
|`Func <int, int, int>`|Takes two integers. Returns an integer.|
|`Func <string, string, bool>`|Takes two strings. Returns a bool.|
|`Action`|Takes no input. No return value.|
|`Action<int>`|Takes an inetger. No return value.|
|`Action<int, bool>`|Takes an integer and bool. No return value.|
|`Action<int, int, string>`|Takes two integers and a string. No return value.|



The following code takes advantage of one of these generic delegates

```csharp
static void Main(string[] args)
{
    int[] arr = Enumerable.Range(1, 10).ToArray();
 
    var doubles = Process(arr, n => 2 * n); // 2, 4, 6, 8 ...
    var triples = Process(arr, n => 3 * n); // 3, 6, 9, 12, ...
    var squared = Process(arr, n => n * n); // 1, 4, 9, 16, ...
 
 
}
 
static int[] Process(int[] arr, Func<int, int> processor)
{
    int[] result = new int[arr.Length];
    for (int i = 0; i < arr.Length; i++)
        result[i] = processor(arr[i]); 
 
    return result;
}
```
