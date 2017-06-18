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

```masm
00B400E1      mov         ebp,esp      
00B400E3      movzx       eax,byte ptr [ecx+4] ; Load terminate into EAX
00B400E7      test        eax,eax  
00B400E9      jne         00B400EF  
00B400EB      test        eax,eax  
00B400ED      je          00B400EB  
00B400EF      pop         ebp  
00B400F0      ret
```
{: class="highlight:[5,6]"}





