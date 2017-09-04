---
layout: post
title:  "GCD (Grand Central Dispatch)- Multithreading in iOS"
date:   2017-09-01 01:20:00
author: mayank
categories: Technical iOS
---
# GCD (Grand Central Dispatch)

Managing threads in an application that uses threads extensively can sometimes be very cumbersome task making the developer to worry more about the threads rather than business logic.So, here comes the role of GCD!

## Basic Introduction :-
`GCD` (libdispatch) is a system level library (written in C) that takes all this responsibility of thread management and let the developer to focus on the business logic rather than managing threads.

`GCD` is built on top of threads. Under the hood it manages a shared thread pool.


## Dispatch Queue
The dominating concept in `GCD` is the `DispatchQueue`.
- Tasks (in the form of blocks of code) are submitted to dispatch queues that execute them in FIFO order guaranteeing that the queue that comes first for execution will also start first.
- These dispatch queues execute the tasks serially or concurrently depending on their type- Serial or Concurrent.

### Serial Dispatch Queue
- In a serial `DispatchQueue`, only one task can run at a time.
- GCD controls the execution timing. You won’t know the amount of time between one task ending and the next one beginning.

![Strongly Typed]({{https://koenig-media.raywenderlich.com/uploads/2014/09/Serial-Queue-Swift.png)
  Image Source - www.raywenderlich.com/

## Concurrent Dispatch Queue
- In a concurrent `DispatchQueue`, multiple tasks can run at a time .
- Tasks are guaranteed to start in the order they were added. Tasks can finish in any order and you have no knowledge of the time it will take for the next task to start, nor the number of tasks that are running at any given time.

![Strongly Typed]({{https://koenig-media.raywenderlich.com/uploads/2014/09/Concurrent-Queue-Swift.png)
  Image Source - www.raywenderlich.com/

  It is `GCD` that decides when to start a task. If the execution time of one task overlaps with another, it’s up to GCD to determine if it should run on a different core, if one is available, or instead to perform a context switch to run a different task.

  Each task submitted to a queue is processed on a pool of threads managed by the system.
  When a task is submitted to a dispatch queue, it is `GCD` that decides which thread to execute it on.So, you only needs to interact with GCD dispatch queues and not the threads.

  Tasks can be submitted to dispatch queues either synchronously or asynchronously.

  The key to use GCD is to choose the right kind of dispatch queue and the right dispatching function (i.e. _sync_, _async_) to submit your work to the queue.

  ## Dispatching functions (Synchronous and Asynchronous)
  With `GCD`, you can dispatch a task either synchronously or asynchronously.

  A _synchronous_ function returns control to the caller after the task is completed.

  An _asynchronous_ function returns immediately, ordering the task to be done but not waiting for it. Thus, an asynchronous function does not block the current thread of execution from proceeding on to the next function.

  ## Types of dispatch queues

**Main dispatch queue** - This is a globally available serial queue and it executes the tasks submitted to it on the application’s main thread.

**Global dispatch queue** - These are concurrent queues and are shared by the whole system. There are four such queues with different priorities : high, default, low, and background.

When setting up the global concurrent queues, you don’t specify the priority directly. Instead you specify a `Quality of Service` (QoS) class property. This will indicate the task’s importance and guide GCD into determining the priority to give to the task.

## Types of `QoS` classes
**User-interactive**: This represents tasks that need to be done immediately in order to provide a nice user experience. Use it for UI updates, event handling and small workloads that require low latency. The total amount of work done in this class during the execution of your app should be small. This should run on the main thread.

**User-initiated**: This represents tasks that are initiated from the UI and can be performed asynchronously. It should be used when the user is waiting for immediate results, and for tasks required to continue user interaction. This will get mapped into the high priority global queue.
Utility: This represents long-running tasks, typically with a user-visible progress indicator. Use it for computations, I/O, networking, continous data feeds and similar tasks. This class is designed to be energy efficient. This will get mapped into the low priority global queue.

**Background**: This represents tasks that the user is not directly aware of. Use it for prefetching, maintenance, and other tasks that don’t require user interaction and aren’t time-sensitive. This will get mapped into the background priority global queue.
The currently executing tasks run on distinct threads that are managed by the dispatch queue. The exact number of tasks executing at any given point is variable and depends on system conditions.

**Custom Queues** - These are the queues that you create by yourself and these can be serial or concurrent. These actually trickle down into being handled by one of the global queues.

## Examples

### Working with system queues

#### Main Queue
##### Submitting a task to the main queue asynchronously

```swift
  DispatchQueue.main.async {
        perform some task on main thread
  }
```

Since, the task is submitted to the main queue asynchronously, so the calling thread won’t wait for the completion for submitted task and will return immediately.

##### Submitting a task to the main queue asynchronously that will be executed after some delay

```swift
  DispatchQueue.main.asyncAfter(deadline: .now() + delay) {
       //task to be performed on main thread after 1 second of submission
  }
```

#### Global Concurrent Queue

```swift
DispatchQueue.global(qos: .utility).async {
    // Asynchronous code running on the low priority queue
}

DispatchQueue.global(qos: .userInitiated).sync {
    // Synchronous code running on the high prioriy queue
}
```

### Working with custom queue

##### Custom Serial Queue

```swift
let customSerialQueue = DispatchQueue(label: "com.appName.myCustomSerialQueue")
customSerialQueue.async {
    // Code
}
```

##### Custom Concurrent Queue
```swift
let customConcurrentQueue = DispatchQueue(label: "com.appName.myCustomConcurrentQueue", attributes: .concurrent)
customConcurrentQueue.async {
    // Code
}
```

#### Performing background tasks and then updating the UI

```swift
DispatchQueue.global(qos: .background).async {
    // Do some background work
    DispatchQueue.main.async {
        // Update the UI to indicate the work has been completed
    }
}
```

#### Be careful of deadlocks with serial queues

```swift
let customSerialQueue = DispatchQueue(label: "com.appName.myCustomSerialQueue")

customSerialQueue.sync {
    // Synchronous code
    customSerialQueue.sync {
        // This code will never be executed and the app is now in deadlock
    }
}
```

##### Beware of submitting task synchronously on the main thread from another thread

```swift
DispatchQueue.global(qos: .utility).sync {
    // Background Task
    DispatchQueue.main.sync {
        // App will crash
    }
}
```

### Inspiration

- https://www.raywenderlich.com/148513/grand-central-dispatch-tutorial-swift-3-part-1

- https://medium.com/modernnerd-code/grand-central-dispatch-crash-course-for-swift-3-8bf2652c1cb8

- https://developer.apple.com/library/content/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationQueues/OperationQueues.html
