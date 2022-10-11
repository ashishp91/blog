---
title: "Memory Allocation in Java"
date: 2022-10-10T22:06:14+05:30
draft: false
tags: ["Java"]
---

A typical Java process deals with the RAM, Non RAM memory and Registers. It's useful to understand how a Java process accesses and manages these memories:

1. **Registers** - the fastest storage in your computer are Registers. Why so ? Because Registers are present inside the processor itself. Registers cannot be accessed directly by a process.
2. **RAM** - The memory allocated to a process resides in the RAM. There are two types of allocation that happens:
- **Heap memory space** - The general purpose pool of memory where all Java objects reside. If you've used new operator to create an object, it'll be stored in the Heap memory. The Garbage Collector works on the Heap memory to free up objects that have no references.
- **Stack memory space** - When a function is called the Compiler stores pointers to the objects created, local variables and params in the stack memory. After the function call is over Compiler deallocates the stack memory.
![memory-allocation-in-java](/java/memory-allocation-in-java/java-heap-stack.png)
3. **Non-RAM** - Storage that lives outside the program. For eg. database, files.
