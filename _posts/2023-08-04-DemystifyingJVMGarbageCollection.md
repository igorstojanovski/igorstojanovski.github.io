---
layout: post
title: "Demystifying JVM's Garbage Collection"
excerpt: "Discover the unique world of JVM's Garbage Collector. It works like an invisible entity, constantly freeing your application's memory from unneeded data. Dive into the intricacies, the varieties available, and why it's essential to understand the Java Memory Model."
date: 2023-08-04
---

The Java Virtual Machine (JVM) boasts a distinctive feature vital to the Java programming language: implicit garbage collection. Put simply, 'garbage collection' is the JVM's inbuilt mechanism for clearing any memory not presently used by your application. The word 'implicit' signifies this process operates autonomously in the background, entirely removing the need for manual intervention. This cleanup task is undertaken by a vital JVM component named the Garbage Collector, or GC for short.

## Understanding Garbage Collection: A Real-Life Analogy

While the concept might initially appear simple, it holds greater complexity under the hood. To better grasp it, let's compare it to real-life waste disposal. Our everyday trash disposal process is predominantly 'explicit'. We collect the waste, often sorting it, before placing it in bins for collection. Regular waste collection trucks then swing by to pick up the trash we've consciously marked for disposal.

  
Imagine if our waste disposal process mirrored the workings of the JVM GC. It would mean an invisible person constantly present in your home, carefully observing your daily activities and habits to decide what should be kept and what should be thrown away. This person would periodically make all identified 'waste' disappear, resulting in a perpetually clean and clutter-free home. You wouldn't care about a thing. You wouldn't even know garbage existed. Quite magical, isn't it? That's exactly how the JVM's Garbage Collector works in your Java application!

## JVM Garbage Collector: What Does It Collect?

So, what exactly does the JVM GC collect? It scrutinises all memory allocations made by your application and decides which ones should be removed to free up memory. The challenging part lies in determining what is no longer in use and what still serves a purpose. Imagine the mess if the Garbage Collector started deleting the lists and strings you just made; it has to be careful in what it does. On the other hand, if the Garbage Collector doesn't remove enough, it can cause memory leaks. This might use up all the memory our application has to work with, which would also be a problem. It is same as continuously buying new milk cartons without discarding the old ones from the fridge. Soon, you'd find it impossible to shut the refrigerator door.

## The Continuous Evolution of Garbage Collectors

The complexity of designing an effective GC is evident in the continuous efforts of Java's creators to refine it over the years. What they've recognized is the absence of a one-size-fits-all GC, leading to the availability of various GC versions and types, both free and commercial, catering to different types of applications. Some are designed for memory-intensive apps, others for apps necessitating ultra-fast response times, and so on.

### The Array of GCs Available

Presently, the following GCs are at your disposal:

-   Parallel
-   Garbage First (G1)
-   Z Garbage Collector (ZGC)
-   Shenandoah
-   Serial
-   Concurrent mark sweep collector (CMS)
-   C4 (Continuously Concurrent Compacting Collector)

## Next Up: The Java Memory Model

Before we delve into the specifics of each type and understand why they differ, it's crucial to examine the Java Memory Model. Understanding how the JVM organises memory into different regions is essential for a comprehensive grasp of the GCs.

  
We'll discuss this in our next [post](https://igorski.co/all-you-need-to-know-as-a-java-developer-about-the-jvm-and-gc/).
