---
layout: post
title: Why is VSCode Consuming 1 TB of Virtual Memory
tags:
- memory
- chrome
- edge
- vscode
---

<span style="display: none;" class="excerpt">This post explains why Chrome and VSCode consume Terabyte of virtual memory.</span>

## Why is VSCode consuming Terabyte of virtual memory?

It's not a memory leak as one might first suspect.
The short answer is **Security**.  
Most of that memory is not real so it doesn't physically take any space in RAM or even on disk.


## How it works
The large virtual memory is due to the [V8 Javascript engine](https://en.wikipedia.org/wiki/V8_JavaScript_engine) that powers Chromium-based projects such as Google Chrome, Microsoft Edge and Electron, which is the framework behind VSCode.

Below is a screenshot of virtual memory mappings for a VSCode instance that shows 1 TB of virtual memory
![](/images/posts/archived/v8-code-maps.png)


V8 creates a sandbox of 1 TB (size might vary per platform and other conditions) in the virtual address space of the process. 
This is also a reason why each browser tab has its own dedicated system process.

The V8 sandbox contains objects such as the V8 heaps references such as [ArrayBuffers](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer) and [WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly) cages.
An attacker might be able to access memory inside the sandbox by performing a malicious memory access operation; however, the goal is to prevent an attacker from corrupting memory *outside* the sandbox.

The sandbox size has to be large in order to sufficiently host all the V8 objects. 
For example, each WebAssembly module must execute within a dedicated sandboxed environment and there can be many modules running in the same process.
Memory is accessed in WebAssembly in a way similar to the following instruction:

```
(i32.load offset=8 (get_local $a))
```

where `offset` (`8`) is added to the base local addresss at `$a` to get the resultant memory address. 

Both `offset` and `$a` values are 32-bit integers where a 32-bit integer can map up to 4 GB of virtual space, 
so the total memory that can be accessed using both is `2 * 4GB =` 8 GB.
Then it makes sense to allocate 8 GB then where the first 4 GB contains the base (i.e., `$a`) and leave the 2nd 4GB as a guard region.
In addition, a second guard region of 2 GB is allocated before the memory base to guard against potential sign extension bugs in implementations.
The calculated memory address (base + offset) on a 64-bit system ends up a 64-bit value (stored in a CPU register) that is used to store the sum of the two 32-bit values. 
This is done by zero-extending a 32-bit value (i.e., pre-padding with 0s). 
If the 32-bit value stored has its MSB (most significant bit) set to 1 (i.e., value greater than `0x80000000`) and was [sign-extended](https://en.wikipedia.org/wiki/Sign_extension) (i.e., pre-padding 1s) instead of zero-extended, then one ends up with a negative value between -1 and -2 GB. 

Thus each WebAssembly module requires 10 GB of virtual space which is often called a WASM cage.

![](/images/posts/archived/v8-wasm-cage.svg)

## Sandbox guard regions
In addition to the 1 TB sandbox which includes the WASM cages, guard regions are created around the sandbox to prevent memory access outside the sandbox.

![](/images/posts/archived/v8-sandbox.svg)

An access from an index into an ArrayBuffer must remain within the sandbox boundary. 
Because the index is a 32-bit unsigned integer, and the element accessed is 8 bytes in the worst case scenario (eg BigUint64Array), 
a region of 2^32 * 8 = 32 GB of contiguous space is needed around the sandbox. 
The guard regions themselves are not accessible and thus do not store anything.
This has the downside of imposing a limit on the size of ArrayBuffers and their indices.

Indeed, one gets an error in Chrome or Edge browser if they try to index beyond 4 GB in an array buffer.

```js
const buffer = new ArrayBuffer (8, {
    maxByteLength: Math.pow(2, 32) + 1 // 4 GB + 1 byte  (Remove the +1 to get rid of the error)
    })
```

Looking back at the top screenshot of VSCode memory mappings

![](/images/posts/archived/v8-code-maps.png)

Clearly, the 32.0 GB is just the front guard, and the 1.0 TB contains most of the sandbox and the 32.0 GB back guard.

## How TB of memory can be allocated

Most of the 1 TB memory is just reserved space and is not in fact accessible.
V8 allocates objects inside the sandbox on demand when needed. 
The space reservation is achieved using `MAP_ANONYMOUS` on POSIX systems (eg Linux) and `MEM_RESERVE_PLACEHOLDER` on Windows.

Below is a demo C program (Windows and Linux) that reserves 1.X TB of virtual space before exiting which gives one a chance to examine the process memory.

**POSIX**

```c
#include <sys/mman.h>
#include <stdio.h>
#include <stdlib.h>
#include <inttypes.h>

#define handleError(msg) \
           do { perror(msg); exit(EXIT_FAILURE); } while (0)

int main(int argc, char *argv[])
{
    const size_t guardSize = 1UL<<35; // 32 Gib
    const size_t sanboxSize = 1UL<<40; // 1 Tib     
    const size_t base = 0x3b9004800000; // Optional base virtual address.

    void* sanboxAddr = (void*)base;    
    mmap(sanboxAddr, sanboxSize + 2*guardSize, PROT_NONE, MAP_PRIVATE | MAP_ANONYMOUS | MAP_NORESERVE, -1, 0);
    
    if (sanboxAddr == MAP_FAILED) {
        handleError("mmap");
    }

    printf("Sandbox address %#lx - %#lx \n", (uintptr_t)sanboxAddr, (uintptr_t)sanboxAddr+sanboxSize);

    printf("Press enter to exit...\n");
    getchar();

    munmap(sanboxAddr, sanboxSize);

    exit(EXIT_SUCCESS);
  }
```

**Windows**

```c
#include <windows.h>
#include <stdio.h>
#include <inttypes.h>
#include <memoryapi.h>

#define handleError(msg) \
           do { perror(msg); exit(EXIT_FAILURE); } while (0)

int main()
{
    const size_t guardSize = static_cast<size_t>(1UL) << 35; // 32 GB
    const size_t sanboxSize = static_cast<size_t>(1UL) << 40; // 1 TB     
    const size_t base = 0x3b9004800000; // Optional base virtual address
    void* sanboxAddr = (void*)base;
    
    sanboxAddr = VirtualAlloc2(nullptr, sanboxAddr, sanboxSize,  MEM_RESERVE | MEM_RESERVE_PLACEHOLDER, PAGE_NOACCESS, nullptr, 0);
    
    if (sanboxAddr == NULL) {
        handleError("VirtualAlloc2");
    }

    printf("Sandbox address %#jx - %#jx \n", (uintptr_t)sanboxAddr, (uintptr_t)sanboxAddr + sanboxSize);

    printf("Press enter to exit...\n");
    getchar();

    bool bSuccess = VirtualFree(
        sanboxAddr,   
        0,             
        MEM_RELEASE);


    exit(bSuccess);
}
```

The memory and the guards are allocated together in one call, and this is to ensure the allocated memory range is contiguous (i.e., has no gaps).

Looking at the memory for this program, one sees similar behavior for VSCode or Chrome. 

![](/images/posts/archived/v8-mem-monitor.png)

![](/images/posts/archived/v8-mem-mappings.png)


