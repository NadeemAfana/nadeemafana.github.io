---
layout: post
title: Memory Barriers in .NET
redirect_from:
- "/post/memory-barriers-in-dot-net.aspx.html"
tags: 
- calc
- calculator
- windows 10
---
## Memory Barriers in .NET
Nonblocking programming can provide performance benefits over locking, but getting it done right is significantly harder and requires careful testing. 
When it comes to memory barriers, there is all sorts of confusion and misleading information. Because memory barrier can be counterintuitive, using them wrongly is easy, and applying them correctly requires a lot of effort and headache. The benefits memory barriers provide may not be worth it in most high-level applications. However, memory barriers are handy in performance critical software. 

In the past, a system with a single processor executed concurrent threads using a trick called "timeslicing", where one thread is running and the others are sleeping. This means that all memory accesses done by one thread appeared in an exact order to all the other running threads. This is called a sequential consistency model. Nowadays, multiprocessor systems are very common and concurrent threads are truly running at the same time, so sequential consistency is not guaranteed because:

1. The compiler or JIT might re-order the memory instructions for optimization.
2. The processor can also re-order memory instructions for performance through different mechanisms including caching, load speculation, and delaying store operations. 

Consider the following program.

### Example 1 
```csharp
int x = 0, y = 0, r1 = 0, r2 = 0;
public void Thread1()
{    
    y = 1;  // Store y
    r1 = x; // Load x            
       
}
     
public void Thread2()
{
    x = 1;  // Store x
    r2 = y; // Load y     
}
```

If the methods run on different threads, there are a few possibilities for the values of `r1` and `r2`: 
1. Thread1 finishes before Thread 2 starts, so that r1 = 0 and r2 = 1.
2. Thread 2 finishes before Thread 1 starts, so that r2 = 0 and r1 = 1.
3. Both Thread1 and Thread2 interleave, so that either one of the following will happen:
    1. r1 = 1 and r2 = 1 or
    2. r1 = 0 and r2 = 1 or
    3. r1 = 1 and r2 = 0

In all of the cases above, either r1 = 0 or r2 = 0 but never both r1 and r2 are 0. Is it possible that both r1 = 0 and r2 = 0? 
The answer is **yes** though it is counterintuitive, but how ?
A processor can defer its store operations in a buffer, and this causes operations to be out of order. If both stores to x and y are delayed, then the load from x in Thread 1 will read 0, and the load from y in Thread2 will read 0 too! 

The real execution order might look like this: 
```csharp
public void Thread1()
{    
    r1 = x;  // Load x
    y  = 1;  // Store y          
       
}
     
public void Thread2()
{     
    r2 = y;  // Load y    
    x  = 1;  // Store x
}
```
If you want to try it yourself, compile and run the following program. I ran the following program on my Intel64 machine. 

**NOTE**: You must compile in release mode.
```csharp
using System;
using System.Threading.Tasks;
namespace ConsoleApplication
{
    class Program
    {
        static int x = 0;
        static int y = 0;
        static int r1 = 0;
        static int r2 = 0;
        static void Main(string[] args)
        {
            do {
                x = y = r1 = r2 = 0;
                var t1 = new Task(() => { Thread1(); });
                var t2 = new Task(() => { Thread2(); });
                t1.Start();
                t2.Start();
                Task.WaitAll(t1, t2);
            } while (!(r1 == 0 && r2 == 0));
            Console.WriteLine("r1={0}, r2={1}", r1, r2);
        }
        static public void Thread1()
        {
            y = 1;  // Store y
            r1 = x; // Load x            
        }
        static public void Thread2()
        {
            x = 1;  // Store x
            r2 = y; // Load y     
        }
    }
}
```
Even if you declare the variables as volatile, it won't help! I'll discuss volatile shortly. Anyways, to fix the problem, add a memory barrier `Thread.MemoryBarrier()` between the store and load operations in each thread. We'll come back to this example later after discussing memory barriers. 

The previous example was a hardware optimization. Let's see another example where software manifests its optimization. 

### Example 2 
The following loop will run forever. Why ? 
```csharp
bool terminate = false;
     
var t = new Thread(() => {       
    int x = 0; 
    while(!terminate){x = x * 1;}
});
     
t.Start();
Thread.Sleep(2000);
terminate = true;
t.Join();
Console.WriteLine("Done.");
```
Looking at the disassembly code, I see the following for the thread `t` body: 

```asm
00B400E1      mov         ebp,esp      
00B400E3      movzx       eax,byte ptr [ecx+4] ; Load terminate into EAX
00B400E7      test        eax,eax  
00B400E9      jne         00B400EF  
00B400EB      test        eax,eax  
00B400ED      je          00B400EB  
00B400EF      pop         ebp  
00B400F0      ret
```

This time it is JIT, not the hardware. It's clear that JIT has cached the value of the variable terminate in the EAX register. The program is now stuck in the loop highlighted above. If you don't know assembly well, the highlighted section above is equivalent to the following C# code. 

```csharp
while(true){}
```

Either using a `lock` or adding a `Thread.MemoryBarrier` inside the while loop will fix the problem. Or you can even use `Volatile.Read`.
The purpose of the memory barrier here is only to suppress JIT optimizations.
Now that we have seen how software and hardware can reorder memory operations, it's time to discuss memory barriers. 

## Memory Barriers

We saw how the hardware and JIT could reorder memory instructions. This can be a problem for communication between different processors as we saw in the examples above. Memory barriers (aka memory fences) are a way to tell the compiler, JIT and the processor to restrict the ordering of memory instructions. 

It's important to note, in general, memory barriers are needed for communication between different processors (There are exceptions of course, such as using a memory barrier to suppress the compiler or JIT optimizations). If the system has only one single processor, then there may not be a need for memory barriers. 

There are different kinds of memory barriers: 

1. **Store memory barrier** (or write memory barrier)
 
    A store memory barrier ensures that no STORE operation can move across the barrier.

    All the STORE operations that appear before the memory barrier will appear to happen before all the STORE operations that appear after the memory barrier. The equivalent CPU instruction is `SFENCE`. 
    
    This has no effect whatsoever on LOAD operations.
    ![Store memory barrier](/images/posts/archived/memory-barriers-in-dot-net-1.png "Store memory barrier")

2.  **Load memory barrier** (or read memory barrier)

    A load memory barrier ensures that no LOAD operation can move across the barrier. 
    All the LOAD operations that appear before the barrier will appear to happen before all the LOAD operations that appear after the memory barrier. The equivalent CPU instruction is `LFENCE`. 

    This has no effect whatsoever on STORE operations.

    ![Read memory barrier](/images/posts/archived/memory-barriers-in-dot-net-2.png "Read memory barrier")

3.  **Full memory barrier**

    A full memory barrier ensures that no STORE or LOAD operation can move across the barrier.

    All the STORE and LOAD operations that appear before the barrier will appear to happen before all the STORE and LOAD operations that appear after the barrier. The equivalent CPU instruction is `MFENCE`.

    ![Full memory barrier](/images/posts/archived/memory-barriers-in-dot-net-3.png "Full memory barrier")

    A full memory barrier is the strongest and interesting one. At least all of the following generate a full memory barrier implicitly:
    
    * `Thread.MemoryBarrier`
    * C# `Lock` statement
    * `Monitor.Enter` and `Monitor.Exit`
    * `Task.Start`
    * `Task.Wait`
    * `Task` continuations
    * `Interlocked` class mehods
    * Any signaling operation such as `ManualResetEvent` and `AutoResetEvent`
    * `Thread.Sleep`
    * `Thread.Join`
    * `Thread.SpinWait`
    * `Thread.VolatileWrite`
    * `Thread.VolatileRead`

## Implicit Barriers

The following are one-way fences based on the direction of movement. 

4.  **Acquire**

    Ensures that no STORE or LOAD operation that appears after the barrier will move before the barrier. Operations that appear before the barrier can move after the barrier. 

    Reading a volatile variable or calling `Volatile.Read` is an acquire fence.

    ![Acquire memory barrier](/images/posts/archived/memory-barriers-in-dot-net-4.png "Acquire memory barrier") 

5.  **Release**

    Ensures that no STORE or LOAD operation that appears before the barrier will move after the barrier. Operations that appear after the barrier can move before the barrier.

    Writing to a volatile variable or calling `Volatile.Write` is a release fence.

    ![Release memory barrier](/images/posts/archived/memory-barriers-in-dot-net-5.png "Release memory barrier")        

## Memory Ordering

Now after introducing memory barriers, let's take a look at the memory models for certain hardware and CLR. The following table describes whether reordering is allowed for certain memory operations. A comma is used to indicate two consecutive operations. For example, Load, Load indicates a load operation followed by another load operation.

 <style type="text/css">
            .table-memory-reordering {
                border-spacing: 0;                
            }


                .table-memory-reordering th {
                    font-weight: bold;
                    text-align: center;
                }

                .table-memory-reordering, .table-memory-reordering th, .table-memory-reordering td {
                    border: 1px solid black;
                    border-collapse: collapse;
                    padding: 3px;
                }

                    .table-memory-reordering td {
                        text-align: left;
                        text-align: center;
                    }
        </style>
<table class="table-memory-reordering">
        <thead>
            <tr>
                <th></th>
                <th colspan="3">Hardware</th>
                <th colspan="2">CLR 2.0+</th>
            </tr>

            <tr>
                <th>Operation</th>
                <th>Intel x86 & Intel64</th>
                <th>IA&nbsp;64</th>
                <th>AMD64</th>
                <th>Without Volatile</th>
                <th>With Volatile</th>
            </tr>
        </thead>

        <tbody>
            <tr>
                <td> <b>LOAD,&nbsp;LOAD</b> </td>
                <td>No</td>
                <td>Yes</td>
                <td>No</td>
                <td>Yes</td>
                <td>No</td>
            </tr>

            <tr>
                <td> <b>LOAD,&nbsp;STORE</b> </td>
                <td>No</td>
                <td>Yes</td>
                <td>No</td>
                <td>Yes</td>
                <td>No</td>
            </tr>

            <tr>
                <td> <b>STORE,&nbsp;STORE</b> </td>
                <td>No</td>
                <td>Yes</td>
                <td>No</td>
                <td>No</td>
                <td>No</td>
            </tr>


            <tr>
                <td> <b>STORE,&nbsp;LOAD</b> </td>
                <td>Yes, only if load and store are to different locations.</td>
                <td>Yes</td>
                <td>Yes, only if load and store are to different locations.</td>
                <td>Yes</td>
                <td>Yes</td>
            </tr>
        </tbody>
    </table>