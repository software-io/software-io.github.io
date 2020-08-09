---
layout: post
title: "Introduction to Kotlin Coroutines"
date: 2020-08-09
---

This articles outlines the main concepts related to coroutines which are lightweight threads and how they compare to actual Threads.
You will see how coroutines simplifies non-blocking asynchronous code with callbacks and synchronization.

##Scope

To create a coroutine you can use one of its builders, in this case launch which creates a CoroutineScope.
Using GlobalScope means the coroutine last for the lifecycle of the application. launch is comparable to thread { } and delay is special non-blocking suspending function which acts like Thread.sleep in a coroutine.

```kotlin
import kotlinx.coroutines.*;

fun main() { 
    GlobalScope.launch { // launch a new coroutine in background and continue
        delay(1000L)
        println("World!")
    }
    println("Hello,") // main thread continues here immediately
    runBlocking {     // but this expression blocks the main thread
        delay(2000L)  // ... while we delay for 2 seconds to keep JVM alive
    } 
}
```
Output
```kotlin
Hello,
World!
```
coroutines execute in a non-blocking manner, as you see in the Output below.

```kotlin
fun main() {
    println("1. runBlocking execute and block until complete")
    runBlocking {
        
        launch {
            delay(500L)
            println("2. launch complete")
        }
        
        GlobalScope.launch {
            delay(100L)
            println("3. GlobalScope complete")
        }

        coroutineScope {
            launch {
                delay(1000L)
                println("4. coroutineScope complete")
            }
        }
    }
    println("5. runBlocking complete")
}
```
Output
```kotlin
1. runBlocking execute and block until complete
3. GlobalScope complete
2. launch complete
4. coroutineScope complete
5. runBlocking complete
```

##Context

A context is a set of data that relates to the coroutine. All coroutines have an associated context
 - Dispatcher 	- which thread the coroutine is run on
 - Job 			- handle on the coroutine lifecycle

```kotlin
fun main() {
    runBlocking {
        launch(CoroutineName("ACoroutine")) { // context of the parent, main runBlocking coroutine
            println("main runBlocking      : in coroutine ${coroutineContext.get(CoroutineName.Key)}    : I'm working in thread ${Thread.currentThread().name}")
        }
        launch(Dispatchers.Unconfined) { // not confined -- will work with main thread
            println("Unconfined            : I'm working in thread ${Thread.currentThread().name}")
        }
        launch(Dispatchers.Default) { // will get dispatched to DefaultDispatcher 
            println("Default               : I'm working in thread ${Thread.currentThread().name}")
        }
        launch(newSingleThreadContext("MyOwnThread")) { // will get its own new thread
            println("newSingleThreadContext: I'm working in thread ${Thread.currentThread().name}")
        }
    }
}
```
Ouput
```kotlin
Unconfined            : I'm working in thread main @coroutine#3
Default               : I'm working in thread DefaultDispatcher-worker-1 @coroutine#4
main runBlocking      : in coroutine ACoroutine     : I'm working in thread main @coroutine#2
newSingleThreadContext: I'm working in thread MyOwnThread @coroutine#5
```

References
 kotlin coroutines basics - https://kotlinlang.org/docs/reference/coroutines/basics.html