---
layout: post
title: Tasks in C#
redirect_from:
- "/post/tasks-csharp.aspx/"
tags: [aspnet, C#, task]
---
A `Task` represents an asynchronous operation. Managing threads is a tedious job. Tasks provide more features than threads such as continuation. They also provide easier exception handling, and return values which makes Tasks easier to use. Tasks use threads behind the scenes to accomplish the job. Tasks were introduced in .NET Framework 4 and were enhanced in C# 5.0.

By default, Tasks use pooled threads. Unlike foreground threads, a pooled thread is a background thread which ends as soon as the main thread in the program ends.

Creating a `Task` is quite straightforward. Just call the static `Run` method with an `Action` delegate or an equivalent lambda expression:
```csharp
Task.Run (() => Console.WriteLine ("Hello Task!"));  // Hello Task!
```
The previous code is equivalent to the following:
```csharp
new Thread(() => Console.WriteLine ("Hello Task!")).Start();
```

If you are using .NET Framework 4.0, you cannot use the `Run` method. You can use `Task.Factory.New` method:

```csharp
Task.Factory.StartNew (() => Console.WriteLine ("Hello Task!")); // Hello Task!
```

`Task.Run` returns a Task object which can be used to track the progress of the task:
```csharp
void Main()
{
     var task = Task.Run ( () => Console.WriteLine ("Hello Task!"));
      
     Console.WriteLine (task.IsCompleted);  // False
}
```

The previous returns false because the task has not completed yet. The `Wait` method blocks the current thread until the task completes:

```csharp
void Main()
{
     var task = Task.Run ( () => Console.WriteLine ("Hello Task!"));
      
     task.Wait (); // Wait until task completes
      
     Console.WriteLine (task.IsCompleted);  // True
}
```

A Task can also return a value. The generic class `Task<TResult>` `Run` method can accept a `Func<TResult>` delegate or an equivalent lambda expression:

```csharp
void Main()
{
     var task = Task<int>.Run ( () => 1000);        
      
     Console.WriteLine (task.Result);  // 1000
         
}
```

The property `Result` holds the result and accessing this property implicitly waits for the task to complete.

Tasks propagate exceptions. This means if the code in a task throws an unhandled exception, the exception propagates to the caller of the task.

```csharp
void Main()
{
     int i = 0;
     var task = Task<int>.Run ( () => 1 / i);       
      
     try
     {
        Console.WriteLine (task.Result);  // Exception
     }
      
     catch(AggregateException ex)
     {
       if (ex.InnerException is DivideByZeroException)
            Console.WriteLine ("Division by 0");
       else
            Console.WriteLine (ex);
     }
         
}
```

The exception is wrapped in `AggregateException` in order to fit well in Parallel programming.

A continuation tells a task to do something else whenever it completes. It is normally implemented using callbacks. There are two ways to implement continuation. The first one is by using an awaiter object (this is only available since Framework 4.5)

```csharp
var task = Task.Run (() => Enumerable
       .Range(1, 1000000)
       .Count (n => (n & 1) == 0)); // Count the even integers between 0 and million
      
     var awaiter = task.GetAwaiter();
      
     awaiter.OnCompleted (() => {
      
        int result = awaiter.GetResult();
        Console.WriteLine (result);
      
     });
```

The method `GetAwaiter` returns an awaiter object. The `awaiter.OnCompleted` method tells the antecedent task to execute a delegate whenever it completes or faults.

Note that the result was accessed using the `GetResult` method. We could have accessed it using task.Result. The difference is that if `GetResult` is called and the antecedent task throws an exception, the exception will not be wrapped in `AggregateException` and this makes the exception handler code cleaner.

The other way to attach a continuation method is by calling `ContinueWith`. The method ContinueWith returns a Task object, so the result can be accessed via the property `Result`:

```csharp
var task = Task.Run (() => Enumerable
        .Range(1, 1000000)
        .Count (n => (n & 1) == 0)); // Count the even integers between 0 and million
      
    task.ContinueWith ( t => {
      
        int result = t.Result;
        Console.WriteLine (result);
      
     });
```