---
layout: post
title: .NET ArrayPool and MemoryPool
tags:
- array
- memory
- pool
---
<span style="display: none;" class="excerpt">This post explains how to use .NET ArrayPool and MemoryPool and their differences.</span>

# Introduction
In this post, I am going to explain what `MemoryPool<T>` and `ArrayPool<T>` are, their differences and how to use them.
I will also introduce a slight variation of `ArrayPool<T>` that provides better maintenance with some real-world examples that you can use in your hot paths in the production environment.

# ArrayPool and MemoryPool
Both `MemoryPool<T>` and `ArrayPool<T>` provide reusable instances of memory buffers that you can use and later return when you are done with them. 
Reusing memory buffers reduces memory pressure on the garbage collector and thus can boost the performance of your app. 
Make sure to always benchmark and collect performance metrics after any changes to your app.

`ArrayPool<T>` is an abstract class on its own from which one can create their own implementation. The .NET framework provides a default implementation called `System.Buffers.ArrayPool<T>.Shared` which is ready for use. 
For example, 
```c#
    byte[] buffer = System.Buffers.ArrayPool<byte>.Shared.Rent(1_000);
    try
    {
        // Logic that uses buffer goes here 
        for (int i = 0; i < 1_000; i++)
        {
            buffer[i] = (byte)(i % 256);
        }
    }
    finally
    {
        System.Buffers.ArrayPool<byte>.Shared.Return(buffer);
    }
```

The above example borrows an array of `1000` bytes for some logic and then returns them. The data type is not restricted to `byte` as in here; it could be technically anything (eg `char`). 
The key is to return the buffer after use which can be safely done in a `finally` block should exception occur. 
Also, the buffer **should not** be used once it has been returned to the pool.
It seems more convenient to use the buffer for local scope though technically the buffer can be passed around (just be careful when doing so).

Similarly, for `MemoryPool<T>` 

```c#
    using IMemoryOwner<byte> buffer = System.Buffers.MemoryPool<byte>.Shared.Rent(1_000);
    var memory = buffer.Memory.Span;

    for (int i = 0; i < 1_000; i++)
    {
        memory[i] = (byte)(i % 256);
    }
```

`MemoryPool<T>` as of this writing is simply a wrapper around `ArrayPool<T>`.
Notice the difference in the returned data type. While `ArrayPool<T>` returns arrays, `MemoryPool<T>` returns `IMemoryOwner<T>` which 
defines the owner responsible for the lifetime management of its `Memory<T>` buffer. This makes the data returned by `MemoryPool<T>` easier to pass around your components where another component becomes the owner and returns the buffer. 
Additionally, lots of the .NET framework methods have been refactored to accept `Memory<T>` directly in their overloads which can be seen as an added benefit. Note it is possible to obtain a `Memory<T>` from an array by simply calling the allocation-free method `AsMemory()`.

The buffer from `MemoryPool<T>` is returned by simply disposing of it instead of maunally returning it as in the case of `ArrayPool<T>`. 
Under the hood, disposing the buffer returns its memory to the underlying `ArrayPool<T>`.
If you want to to transfer ownership of the buffer, make sure to not dispose of it before doing the transfer.
Ownership here means who is responsible for calling `Dispose` to return the buffer.

From a performance standpoint, since `MemoryPool<T>` is a wrapper, we expect some small penalty in performance. 
Hence I ran the following benchmarks

```c#
[Benchmark(Baseline = true)]
public void ArrayPool()
{
    var buffer = System.Buffers.ArrayPool<byte>.Shared.Rent(1_000);
    try
    {
        for (int i = 0; i < 1_000; i++)
        {
            buffer[i] = (byte)(i % 256);
        }
    }
    finally
    {
        System.Buffers.ArrayPool<byte>.Shared.Return(buffer);
    }
}
```

```c#
[Benchmark]
public void MemoryPool()
{
    using var buffer = MemoryPool<byte>.Shared.Rent(1_000);
    var span = buffer.Memory.Span;
    for (int i = 0; i < 1_000; i++)
    {
        span[i] = (byte)(i % 256);
    }	
}
```



|          Method |     Mean |    Error |   StdDev | Ratio | RatioSD |   Gen0 |   Gen1 | Allocated | Alloc Ratio |
|---------------- |---------:|---------:|---------:|------:|--------:|-------:|-------:|----------:|------------:|
|       ArrayPool | 662.0 ns |  2.80 ns |  5.97 ns |  1.00 |    0.00 |      - |      - |         - |          NA |
|      MemoryPool | 786.9 ns | 13.92 ns | 13.02 ns |  1.19 |    0.02 | 0.0019 | 0.0010 |      24 B |          NA |

<small style='font-size: 0.7rem;'>
BenchmarkDotNet=v0.13.2, OS=Windows 11 (10.0.22621.1848) .NET SDK=7.0.100  
</small>

As we can see, there is small overhead in using `MemoryPool`. It is 19% slower and has higher memory footprint, but the overall difference is not bad. 

# Alternative
As you start using `ArrayPool` you will notice that the manual invocation of the `Return` method can clutter the source code quickly especially if one is using multiple buffers in the same code block. It would be nice if one could use `Dispose` to return the buffer just like `MemoryPool` but without the performance overhead. The following wrapper achieves that:

```c#
public static class ArrayPoolHelper
{
	public static SharedObject<T> Rent<T>(int minimumLength)
	{
		return new SharedObject<T>(minimumLength);
	}
		
	public struct SharedObject<T> : IDisposable
	{		
		private readonly T[] _value;

		public SharedObject(int minimumLength)
		{
			_value = ArrayPool<T>.Shared.Rent(minimumLength);
		}

		public T[] Value
		{
			get {				
				return _value;
			}
		}
		
		public void Dispose()
		{			
			ArrayPool<T>.Shared.Return(Value);			
		}
	}
}
```

which can be used like the following
```c#
[Benchmark]
public void ArrayPoolHelper()
{
    using var buffer = ArrayPoolHelper.Rent<byte>(1_000);
    for(int i=0; i<1_000;i++) {
        buffer.Value[i]=(byte) (i % 256);
    }
}
```

The following benchmark shows how all three rank together:


|          Method |     Mean |    Error |   StdDev | Ratio | RatioSD |   Gen0 |   Gen1 | Allocated | Alloc Ratio |
|---------------- |---------:|---------:|---------:|------:|--------:|-------:|-------:|----------:|------------:|
| ArrayPoolHelper | 647.8 ns |  6.48 ns |  5.41 ns |  0.98 |    0.01 |      - |      - |         - |          NA |
|       ArrayPool | 658.7 ns |  6.45 ns |  6.03 ns |  1.00 |    0.00 |      - |      - |         - |          NA |
|      MemoryPool | 779.8 ns | 13.08 ns | 12.23 ns |  1.18 |    0.03 | 0.0019 | 0.0010 |      24 B |          NA |

As we can see above, `ArrayPoolHelper` is as fast as `ArrayPool` and without any allocation penalties. 

# Examples
It is common in apps nowadays to use Base64 encoding to transfer binary data as text. 
Since Base64 is heavy on using byte arrays, we could put the code above to use and see how it performs.

The following method decodes text from Base64 to `byte[]`, which is the equivalent of `Convert.FromBase64String`

```c#
public static SharedObject<byte> FromBase64String(string value, out int bytesWritten)
{
    using var buffer = Rent<byte>(Encoding.UTF8.GetMaxByteCount(value.Length));
    int bufferSize = Encoding.UTF8.GetBytes(value, buffer.Memory);
    var decodedBuffer = Rent<byte>(Base64.GetMaxDecodedFromUtf8Length(value.Length));
    try {
        Base64.DecodeFromUtf8(buffer.Memory.AsSpan(0, bufferSize), decodedBuffer.Memory, out int _, out bytesWritten);
        if (bytesWritten == 0) {
            throw new InvalidOperationException("Error writing to buffer.");
        }
    }
    catch {
        decodedBuffer.Dispose();
        throw;
    }
    return decodedBuffer;
}
```

and the benchmarks

```c#
// 2048 bytes encoded in base64.
const string base64Text = "9uGC8l4jWOD+...";

[Benchmark]
public void ArrayPoolHelperBase64()
{    
    using var bytes = UserQuery.ArrayPoolHelper.FromBase64String(base64Text, out int size);       
}

[Benchmark(Baseline = true)]
public void ConvertFromBase64()
{
    byte[] bytes = Convert.FromBase64String(base64Text);
}
```

|                Method |       Mean |    Error |   StdDev | Ratio |   Gen0 | Allocated | Alloc Ratio |
|---------------------- |-----------:|---------:|---------:|------:|-------:|----------:|------------:|
| ArrayPoolHelperBase64 |   224.5 ns |  2.54 ns |  2.38 ns |  0.11 |      - |         - |        0.00 |
|     ConvertFromBase64 | 2,003.1 ns | 29.91 ns | 27.98 ns |  1.00 | 0.1640 |    2072 B |        1.00 |

`ArrayPoolHelperBase64` is clearly the winner being 90% faster than `Convert.FromBase64String` and no allocations.

If you are using `Convert.FromBase64String` in one of your hot paths, your app will most likely see benefits from using the above method.

For completion, below is the `Convert.ToBase64String` implementation

```c#
public static string ToBase64String(ReadOnlySpan<byte> value)
{
    using var encodedBuffer = Rent<byte>(Base64.GetMaxEncodedToUtf8Length(value.Length));
    Base64.EncodeToUtf8(value, encodedBuffer.Value, out int _, out int bytesWritten);
    if (bytesWritten == 0)
    {
        throw new InvalidOperationException("Error writing to buffer.");
    }

    // Convert bytes to string            
    return Encoding.UTF8.GetString(encodedBuffer.Value.AsSpan(0, bytesWritten));
}
```

