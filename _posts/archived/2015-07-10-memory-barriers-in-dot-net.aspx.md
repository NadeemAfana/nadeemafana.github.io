---
layout: post
title: Memory Barriers in .NET
redirect_from:
- "/post/memory-barriers-in-dot-net.aspx.html"
tags: [memory barrier fence intel intel64 amd64 amd reordering processor]
---
## Memory Barriers in .NET
Nonblocking programming can provide performance benefits over locking, but getting it done right is significantly harder and requires careful testing. 
When it comes to memory barriers, there is all sorts of confusion and misleading information. Because memory barrier can be counterintuitive, using them wrongly is easy, and applying them correctly requires a lot of effort and headache. The benefits memory barriers provide may not be worth it in most high-level applications. However, memory barriers are handy in performance critical software. 

In the past, a system with a single processor executed concurrent threads using a trick called "timeslicing", where one thread is running and the others are sleeping. This means that all memory accesses done by one thread appeared in an exact order to all the other running threads. This is called a sequential consistency model. Nowadays, multiprocessor systems are very common and concurrent threads are truly running at the same time, so sequential consistency is not guaranteed because:

1. The compiler or JIT might re-order the memory instructions for optimization.
2. The processor can also re-order memory instructions for performance through different mechanisms including caching, load speculation, and delaying store operations. 

Consider the following program.
<a name="example-1"></a>
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

A few comments based on the table above and after examining detailed documentation:
* .NET Framework 4.5 does not support Itanium processors. However, I added it in the table for legacy purpose only. 
* Intel x86, Intel64 and AMD 64 are the strongest hardware memory models and IA64 is the weakest.
* CLR 2.0+ prevents reordering two store operations, even without using volatile.
* For x86, Intel64 and AMD 64, a LOAD,LOAD operation usually is not reordered; however, Intel mentions that a processor can reorder a LOAD,LOAD operation, but this reordering should not be visible to software. See Section 8.2.3 in [Intel® 64 and IA-32 Architectures Software Developer's Manual: Volume 3A](http://download.intel.com/design/processor/manuals/253668.pdf)

    AMD64 documentation mentions that a LOAD,LOAD operation can be reordered only if the memory type is different for each LOAD (eg LOAD (WB), LOAD (WC+) . See table 7-3 in [AMD64 Architecture Programmer's Manual Volume 2: System Programming](http://support.amd.com/TechDocs/24593.pdf)

* If you have Intel x86, Intel64 or AMD64, then volatile does not provide any extra hardware ordering restrictions. This means stores by default are release fences and reads are acquire fences. 

    **So why use volatile?**
   
    Although using volatile is not recommended because of its subtleties, it can be used to suppress JIT compiler optimizations. Additionally, some older programs still run on IA64. 
## Atomic Operations
Although a C# `lock` eliminates the need for a memory barrier. A C# lock is sometimes considered an expensive operation because it can lead to contention and thread descheduling. Smart programmers avoid locking if possible. Fortunately, processors provide atomic instructions such as xchg. Also, a simple write or read is considered an atomic operation on modern processors. Atomic instructions happen instantaneously and are much more efficient than locking. For example, the Interlocked class exposes methods that provide atomic behavior that take advantage of the processor atomic instructions directly.

**Why do we need atomic instructions in high-level programs?**
In the following example, if you run the two methods concurrently on two different threads, you expect `result` to be `200,000`.

```csharp
public int  result;
       
public void ThreadA()
{
    for (int i = 1; i <= 100000; i++) {
        result++;
    }
}
public void ThreadB()
{
    for (int i = 1; i <= 100000; i++) {
        result++; 
    }
}
```
But after running the example on my machine, I see the following: 

![Atomic operations example 1](/images/posts/archived/memory-barriers-in-dot-net-6.png "Atomic operations example 1")  

In fact, every time I run it, I get different a result! 

Note that even when declaring result as `volatile`, I always get a corrupt value.

Looking at the disassembly code, I see that JIT translates it into:

```asm
000007FE6F1501B5 mov         eax,dword ptr [rcx+8]  ; Load result into EAX
000007FE6F1501B8 inc         eax                    ; increment EAX 
000007FE6F1501BA mov         dword ptr [rcx+8],eax  ; Load EAX into result
```

Obviously, the above instructions are not atomic. Imagine that both threads execute at the same time, and the 2nd thread reads the same value for result as the first thread, then both will set result to the same value. This happens because result was out of date. That is also why result is always less than or equal to 200,000. 

Although locking around the increment would solve the problem, there is a more efficient way to fix the problem. `Interlocked` comes to rescue: 

```csharp
public void ThreadA()
{
    for (int i = 1; i <= 100000; i++) {
        Interlocked.Increment(ref result);
    }
}
public void ThreadB()
{
    for (int i = 1; i <= 100000; i++) {
        Interlocked.Increment(ref result);
    }
}
```

Now the result is always guaranteed to be `200,000`. 
![Atomic operations example 2](/images/posts/archived/memory-barriers-in-dot-net-7.png "Atomic operations example 2") 

Looking at the disassembly after we introduced the `Interlocked` class

```asm
000007FE6F1601F0        mov         eax,1  
000007FE6F1601F5        lock xadd   dword ptr [rcx+8],eax   ; add 1 to result
000007FE6F1601FA        cmp         byte ptr [rcx],0      
000007FE6F1601FD        mov         eax,1  
000007FE6F160202        lock xadd   dword ptr [rcx+8],eax   ; add 1 to result
000007FE6F160207        add         edx,2                   ; i = i + 2
000007FE6F16020A        cmp         edx,186A0h              ; if i <= 10,000 
000007FE6F160210        jle         000007FE6F1601F0        ; loop again
```

JIT used a locked instruction on my Intel64 machine. A locked instruction is guaranteed to be atomic according to Intel documentation. `XADD` is an exchange-and-add instruction, which in this case exchanges `EAX` with `result` and stores the sum of both in `result`. 

**Why did JIT not use lock add instead of lock xadd?**

It's because the implementation of `Interlocked.Increment` in the .NET Framework looks like this

```csharp
ExchangeAdd(ref location1, value);
```

`Interlocked` has other useful methods such as `Decrement`, `Add`, and `CompareExchange`.

Let's revisit the [Example 1](#example-1) to see how JIT translates a memory barrier:

```csharp
public void Thread1()
{    
    y  = 1;  // Store y 
    Thread.MemoryBarrier();                  
    r1 = x; // Load x
}
     
public void Thread2()
{     
    x  = 1;  // Store x 
    Thread.MemoryBarrier();
    r2 = y; // Load y     
}
```

Looking at the disassembly of Thread1 method after introducing the memory barriers, we see: 

```asm
01270270       mov         dword ptr ds:[11C32A0h],1    ; Store 1 in y
0127027A       lock or     dword ptr [esp],0            ; Thread.MemoryBarrier()    
0127027F       mov         eax,dword ptr ds:[011C329Ch] ; Load x into EAX
```

On my Intel64 machine, JIT replaced the memory barrier with a dummy instruction (ie `lock or`). In this instance, this instruction does change the value of anything (since any value bitwise OR 0 yields the same value). However, locked instructions have the same effect as a memory barrier in my case, which means they also flush the store buffer.

## Facts about memory barriers

Although the effects of memory barriers depend on the hardware, software must be designed in a way to allow for the common denominator. Additionally, there are common facts about memory barriers: 

1. Each processor sees its own access orders without needing a memory barrier.
2. Inserting a memory barrier in one processor does not have a direct effect on another processor unless they both have matching memory barriers.
3. If a variable is being updated by many processors, then the sequence of values observed by one processor will be the same sequence of the other processors.

4. Inserting a memory barrier does not guarantee the order of operations before the memory barrier. A barrier can be thought of as a line of separation. For example
    
    <table class="table-memory-reordering" style="width: 20%;">
                    <thead>
                        <tr>
                            <th>Processor 1</th>
                        </tr>
                    </thead>

                    <tbody>
                        <tr>
                            <td>X = 1</td>
                        </tr>
                        <tr>
                            <td>B = 1</td>
                        </tr>
                        <tr>
                            <td>Y = 1</td>
                        </tr>
                        <tr>
                            <td><span style="background-color: #ffff33;">memory barrier</span> </td>
                        </tr>
                        <tr>
                            <td>Z = 2</td>
                        </tr>
                    </tbody>
    </table>

    The stores from processor 1 will be committed in 1 of 6 different ways:
    
    1. {X = 1, B = 1, Y = 1, Z = 2}
    2. {X = 1, Y = 1, B = 1, Z = 2}
    3. {B = 1, X = 1, Y = 1, Z = 2}
    4. {B = 1, Y = 1, X = 1, Z = 2}
    5. {Y = 1, X = 1, B = 1, Z = 2}
    6. {Y = 1, B = 1, X = 1, Z = 2}

5. In the example below, assuming `X = Y = 0`, if `LOAD Y` comes up with 1, then `LOAD X` must come up with 1. 

  <table class="table-memory-reordering">
                    <thead>
                        <tr>
                            <th>Processor 1</th>
                            <th>Processor 2</th>
                        </tr>
                    </thead>

                    <tbody>
                        <tr>
                            <td>X = 1 </td>
                            <td>LOAD Y</td>
                        </tr>
                        <tr>
                            <td><span style="background-color: #ffff33;">memory barrier</span> </td>
                            <td><span style="background-color: #ffff33;">memory barrier</span> </td>
                        </tr>
                        <tr>
                            <td>Y = 1 </td>
                            <td>LOAD X</td>
                        </tr>
                    </tbody>
    </table>

6. In the example below , assuming `X = Y = 0`, if `LOAD Y` comes up with 1, then `LOAD X` cannot come up with 1. 

    <table class="table-memory-reordering">
                    <thead>
                        <tr>
                            <th>Processor 1</th>
                            <th>Processor 2</th>
                        </tr>
                    </thead>

                    <tbody>
                        <tr>
                            <td>Y = 1 </td>
                            <td>X = 1</td>
                        </tr>
                        <tr>
                            <td><span style="background-color: #ffff33;">memory barrier</span> </td>
                            <td><span style="background-color: #ffff33;">memory barrier</span> </td>
                        </tr>
                        <tr>
                            <td>LOAD X</td>
                            <td>LOAD Y</td>
                        </tr>
                    </tbody>
    </table>

7. In the example below, assuming `X = Y = 0`, if `LOAD X` comes up with 1, then processor 1 store to Y must happen after processor 2 store to Y. Therefore, Y must be 2. 

    <table class="table-memory-reordering">
                    <thead>
                        <tr>
                            <th>Processor 1</th>
                            <th>Processor 2</th>
                        </tr>
                    </thead>

                    <tbody>
                        <tr>
                            <td>Y = 2</td>
                            <td>Y = 1</td>
                        </tr>
                        <tr>
                            <td><span style="background-color: #ffff33;">memory barrier</span> </td>
                            <td><span style="background-color: #ffff33;">memory barrier</span> </td>
                        </tr>
                        <tr>
                            <td>LOAD X</td>
                            <td>X = 1</td>
                        </tr>
                    </tbody>
        </table>

## Load Speculation

Modern processors prefetch data from main memory before the program needs it. The goal is to predict which data the program needs to consume. This makes the data immediately available when the program execution instruction gets to the point of the LOAD. 

It is possible that the processor did not need the value it had prefecthed. This is probably because a branch has circumvented the execution flow. In this case, the prefecthed value can be discarded or cached for later use. 

Although load speculation makes a program run faster, it can make the program see stale values.

For example, initially assume `X = Y = 0`.

### Example 3

<table class="table-memory-reordering">
            <thead>
                <tr>
                    <th>Processor 1</th>
                    <th>Processor 2</th>
                </tr>
            </thead>

            <tbody>
                <tr>
                    <td></td>
                    <td>LOAD X</td>
                </tr>

                <tr>
                    <td>Y = 2</td>
                    <td>Long latency instructions such as division and square-root</td>
                </tr>

                <tr>
                    <td></td>
                    <td>LOAD Y</td>
                </tr>
            </tbody>
</table>

![Load speculation example 3](/images/posts/archived/memory-barriers-in-dot-net-8.png "Load speculation example 3") 
#### Figure 7: Load Speculation. The green color indicates a current value, whereas the red color indicates a stale value. The dashed line indicates the processor is busy.

The order of memory operations as in the above figure:
1. Initially, processor 2 loads X where its value is 0.
2. Processor 2 starts executing long latency instructions. The processor realizes that there is an upcoming load of Y, so it speculates so it pretches Y before even the program got to the point where Y is needed.
3. In the meantime, processor 1 updates the value of Y to 2. However, this new value is not visible to processor 2 yet.

4. When processor 2 has finished the long latency instructions, the old value of Y is immediately served.

5. The value "0" of Y that processor 2 has prefetched is now out of date!

A memory barrier in this case would fix the issue:


<table class="table-memory-reordering">
            <thead>
                <tr>
                    <th>Processor 1</th>
                    <th>Processor 2</th>
                </tr>
            </thead>

            <tbody>
                <tr>
                    <td></td>
                    <td>LOAD X</td>
                </tr>

                <tr>
                    <td>Y = 2 <br> <span style="background-color: #ffff33;">memory barrier</span></td>
                    <td>Long latency instructions such as division and square-root</td>
                </tr>

                <tr>
                    <td></td>
                    <td><span style="background-color: #ffff33;">memory barrier</span> <br>LOAD Y</td>
                </tr>
            </tbody>
</table>

The memory barrier above LOAD Y tells processor 2 to check the value of Y since it could have changed. If Y has changed, then it will be reloaded from memory. 

## Memory Barrier Pairing
If the purpose of a memory barrier is only multiprocessor synchronization, and by that, I mean hardware synchronization and not suppress JIT or compiler optimizations, then memory barriers should be paired.

Looking again at solved [Example 1](#example-1)
![Load speculation of example 1](/images/posts/archived/memory-barriers-in-dot-net-8.png "Load speculation of example 1") 

Note that the store to y before the barrier in Thread1 matches the load of y after the barrier in Thread2, and so is the same for x. 

A single barrier alone will not solve the problem that we discussed earlier. You can try that yourself by removing one of the barriers and running the program again! 

A lack pf pairing can cause problems, for example, consider X = Y = 0 initially.

### Example 4

<table class="table-memory-reordering">
            <thead>
                <tr>
                    <th>Processor 1</th>
                    <th>Processor 2</th>
                </tr>
            </thead>

            <tbody>
                <tr>
                    <td>X = 1 </td>
                    <td></td>
                </tr>
                <tr>
                    <td><span style="background-color: #ffff33;">memory barrier</span> </td>
                    <td></td>
                </tr>
                <tr>
                    <td>Y = 1</td>
                    <td></td>
                </tr>

                <tr>
                    <td></td>
                    <td>LOAD Y</td>
                </tr>
                <tr>
                    <td></td>
                    <td>LOAD X</td>
                </tr>
            </tbody>
</table>

Processor 2 might see the order of memory operations as follows
![Load speculation of example 4](/images/posts/archived/memory-barriers-in-dot-net-9.png "Load speculation of example 4") 
#### Figure 9: Barrier Missing. The green color indicates a current value, whereas the red color indicates a stale value.

Despite the barrier in processor 1, processor 2 might perceive the wrong value of X. To solve the problem, a memory barrier is needed above the LOAD of X in processor 2. 

<table class="table-memory-reordering">
            <thead>
                <tr>
                    <th>Processor 1</th>
                    <th>Processor 2</th>
                </tr>
            </thead>

            <tbody>
                <tr>
                    <td>X = 1 </td>
                    <td></td>
                </tr>
                <tr>
                    <td><span style="background-color: #ffff33;">memory barrier</span> </td>
                    <td></td>
                </tr>
                <tr>
                    <td>Y = 1</td>
                    <td></td>
                </tr>

                <tr>
                    <td></td>
                    <td>LOAD Y</td>
                </tr>
                <tr>
                    <td></td>
                    <td><span style="background-color: #ffff33;">memory barrier</span> </td>
                </tr>
                <tr>
                    <td></td>
                    <td>LOAD X</td>
                </tr>
            </tbody>
</table>

![Load speculation of example 4 fixed](/images/posts/archived/memory-barriers-in-dot-net-10.png "Load speculation of example 4 fixed") 
#### Figure 10: Added missing barrier. The green color indicates a current value, whereas the red color indicates a stale value.

Now, the effects before the memory barrier in processor 1 (STORE X = 1) are available after the barrier in processor 2 (LOAD X). 

## Cache Coherence
Processors rely heavily on caches. It is in fact common for each core to have its own cache, and it is possible for some cores to share the same cache. Modern Intel processors support a mechanism called "Cache locking" where the processor instead of locking on the system bus, it updates the value internally in its cache and relies on cache coherency. 

Maintaining the coherence of all the caches is not the programmer's job. However, the programmer should be aware of the consequences of cache coherence. Although all caches should be coherent, the order in which the memory operations happen is not guaranteed. Think of cache coherence as a notification system, if a processor makes changes, all the other processors will be notified immediately of the changes, but the actual changes may not be applied immediately because a cache has a queue of all the memory operations that need to be applied to it. Memory barriers have effects on this cache and therefore can restrict ordering. 

A cache line is a fixed-length block of data. Data flows among processors' caches and main memory in terms of cache lines, not individual bytes. 

Looking at example 4 again,

<table class="table-memory-reordering">
            <thead>
                <tr>
                    <th>Processor 1</th>
                    <th>Processor 2</th>
                </tr>
            </thead>

            <tbody>
                <tr>
                    <td>X = 1 </td>
                    <td></td>
                </tr>
                <tr>
                    <td><span style="background-color: #ffff33;">memory barrier</span> </td>
                    <td></td>
                </tr>
                <tr>
                    <td>Y = 1</td>
                    <td></td>
                </tr>

                <tr>
                    <td></td>
                    <td>LOAD Y</td>
                </tr>
                <tr>
                    <td></td>
                    <td><span style="background-color: #ffff33;">memory barrier</span> </td>
                </tr>
                <tr>
                    <td></td>
                    <td>LOAD X</td>
                </tr>
            </tbody>
</table>


**Is it possible that LOAD Y comes up with 0 even though it ran after processor 1 `STORE Y = 1`?**

The answer is "**yes**". 

The barrier between the loads in processor 2 does not guarantee that Y will come up with 1, but it does guarantee that X will always be 1 provided that it ran after processor 1's memory barrier.

Here is how Y might come up with 0 instead of 1 even though it ran after the store Y = 1:

Assuming X = Y = 0 initially and both X and Y are in 1 and 2 caches. 

1. Processor 1 executes X = 1. It sends an "invalidate" message to processor 2 in order to evict the matching cache line.
2. Processor 2 receives the "invalidate" message from processor 1, queues the message and acknowledges it by responding to it immediately.
3. Processor 1 receives processor 2 response and continues by flushing its store buffer due to the memory barrier.
4. Processor 1 executes Y = 1. It sends an "invalidate" message in order to evict the matching cache line in processor 2.
5. Processor 2 receives the "invalidate" message from processor 1, queues the message, acknowledges it, and immediately responds to it.
6. Processor 2 executes LOAD Y. Since Y is already in its cache, the value 0 is used.
7. Processor 2 now sees the memory barrier and must wait until all the queued "invalidate" messages are processed, and therefore it invalidates the cache lines containing X and Y.
8. Processor 2 executes LOAD X. Since the cache line of X is no longer in processor 2 cache, it sends a "read" message.
9. Processor 1 receives the "read" message of processor 2 and transmits the cache line containing the new value of X.
10. Processor 2 receives the cache line with the new value "1" of X.

To fix this issue, a new pair of memory barriers is needed:

<table class="table-memory-reordering">
            <thead>
                <tr>
                    <th>Processor 1</th>
                    <th>Processor 2</th>
                </tr>
            </thead>

            <tbody>
                <tr>
                    <td>X = 1 </td>
                    <td></td>
                </tr>
                <tr>
                    <td><span style="background-color: #ffff33;">memory barrier</span> </td>
                    <td></td>
                </tr>
                <tr>
                    <td>Y = 1</td>
                    <td></td>
                </tr>
                <tr>
                    <td><span style="background-color: #ffff33;">memory barrier</span> </td>
                    <td></td>
                </tr>

                <tr>
                    <td></td>
                    <td><span style="background-color: #ffff33;">memory barrier</span> </td>
                </tr>
                <tr>
                    <td></td>
                    <td>LOAD Y</td>
                </tr>
                <tr>
                    <td></td>
                    <td><span style="background-color: #ffff33;">memory barrier</span> </td>
                </tr>
                <tr>
                    <td></td>
                    <td>LOAD X</td>
                </tr>
            </tbody>
</table>

Note that one barrier alone is not enough.

The new barrier in processor 1 is needed to flush the store buffer, and the new barrier in processor 2 is needed to make sure Y is up to date by flushing the invalidate queues.

Notice how all the barriers are paired.

![Memory barriers pairing](/images/posts/archived/memory-barriers-in-dot-net-11.png "Memory barriers pairing") 









