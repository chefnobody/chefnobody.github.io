---
title: iOS - Doing Work with DispatchQueue
date: "2019-5-26"
---

iOS has a fantastic API for doing work on the system in a distributed fashion. That API is called [Grand Central Dispatch](https://developer.apple.com/documentation/dispatch) (GCD) or [`libdispatch`](https://apple.github.io/swift-corelibs-libdispatch/), and you interface with it through a handful of Swift classes. There is a ton you can do with GCD, to be sure.

Let's do a really simple dive into [`DispatchQueue`](https://developer.apple.com/documentation/dispatch/dispatchqueue) and [`DispatchGroup`](https://developer.apple.com/documentation/dispatch/dispatchgroup). We will dispatch some work synchronously (serially), asynchronously and then as a group so we can be notified when work dispatched to a queue has completed.

We will _not_ cover the many of the differences  of the system's global queue, or which quality of service classes or even thread safety. We're just going to watch our tasks happen in order and out of order.

### Define A Unit Of Work

Let's say you have some work to do like a network call, or to read/write to a data store, or manipulate an image. For the sake of this example let's say you just want to make some other thread do that work and avoid blocking the user's experience with your applicaiton. Here is a very naive function that does some work. It takes a name `String` and a time interval `CFTimeInterval` and causes it's thread to sleep for the specified time.

```swift
var work:(String, CFTimeInterval) -> () = { name, interval in
    Thread.sleep(forTimeInterval: interval)
    print("\(name) completed at: \(String(describing: Date.init(timeIntervalSinceNow: 0)))")
}
```

This is naive but now we have some unit of work that will "finish" in the future and that we can dispatch somewhere.

### Dispatch Synchronously

Let's dispatch 4 units of work, in order on our global queue and see what happens.

```swift
// Dispatch a bunch of work sequentially
print("---START-SYNC---")
DispatchQueue.global().sync { work("Long task", 10) }
DispatchQueue.global().sync { work("Medium task", 5) }
DispatchQueue.global().sync { work("Short task", 2) }
DispatchQueue.global().sync { work("Medium Task", 4) }
print("---FINISH-SYNC---")
```

Each of these blocks of work will complete in order and so you should see something like this in your log:

```terminal
---START-SYNC---
Long task completed at: 2019-05-26 21:27:55 +0000
Medium task completed at: 2019-05-26 21:28:00 +0000
Short task completed at: 2019-05-26 21:28:02 +0000
Medium Task completed at: 2019-05-26 21:28:06 +0000
---FINISH-SYNC---
```

This seems right. Each unit of work is dispatched synchronously to a queue and so I think we expected this output.

### Dispatch Asynchronously

Now let's dispatch those same units of work in asynchronous fashion:

```swift
// Dispatch a bunch of work in parallel where shorter tasks finish first
print("---START-ASYNC---")
DispatchQueue.global().async { work("Long Task", 10) }
DispatchQueue.global().async { work("Medium Task", 5) }
DispatchQueue.global().async { work("Short Task", 2) }
DispatchQueue.global().async { work("Medium Task", 4) }
print("---FINISH-ASYNC--")
```

You should see the following in your log:

```terminal
---START-ASYNC---
---FINISH-ASYNC--
Short Task completed at: 2019-05-26 21:47:53 +0000
Medium Task completed at: 2019-05-26 21:47:55 +0000
Medium Task completed at: 2019-05-26 21:47:56 +0000
Long Task completed at: 2019-05-26 21:48:01 +0000
```

This also seems right. Our two `print` statements exectued immediately after the 4 units of work were dispatched asynchronously. The tasks then finished in order, by name and length of sleep time. 

### Dispatch Group

What if we had some group of work that was comprised of several units of work and we need to know when _all_ of those units of work complete? For such a scenario we could use a [DispatchGroup](https://developer.apple.com/documentation/dispatch/dispatchgroup):

```swift
// Dispatch a bunch of work as a group on a queue and receive a notification when all tasks in the group complete
let group = DispatchGroup.init()
group.notify(queue: .main) { print("All tasks on the main queue are complete!") }

// Assuming this is on the main queue:
for i in 0..<5 {
    group.enter()
    work("Task \(String(describing: i))", Double(i))
    group.leave()
}
```

Here we create a `DispatchGroup`, then trivially, loop from 0 to 4 calling `enter()` on the group, firing our unit of work and calling `leave()` on the `DispatchGroup`. Note that we're working from the assumption that our for loop is on the same thread as the `DispatchGroup` (the `.main` thread) and that our unit of work executes immediately.

Our log should look something like this:

```terminal
Task 0 completed at: 2019-05-26 21:52:50 +0000
Task 1 completed at: 2019-05-26 21:52:51 +0000
Task 2 completed at: 2019-05-26 21:52:53 +0000
Task 3 completed at: 2019-05-26 21:52:56 +0000
Task 4 completed at: 2019-05-26 21:53:00 +0000
All tasks on the main queue are complete!
```

But, what's up with the `enter` and `leave` functions, right? To understand them better it helps to understand what this line is doing:

```swift
group.notify(queue: .main) { print("All tasks on the main queue are complete!") }
```

The docs for this method say: 

> Schedules the submission of a block to a queue when all tasks in the current group have finished executing.

This begs the question, "What are all the tasks on the current group?" The answer is to look at the docs for the `enter()` and `leave()` methods. 

`enter()`:
> Explicitly indicates that a block has entered the group.

`leave()`:
> Explicitly indicates that a block in the group finished executing.

So, what's happening here, I believe is that by calling `enter` we're telling the `DispatchGroup` instance that some work is about to occur on the thread we care about. 

Then by calling `leave()` we're telling it that the some work is done on this thread.

Think of the `DispatchGroup` instance as having an internal counter of all the "work" it is tracking. Once its counter reaches 0 it will trigger this notification. You can increment and decrement the counter by calling `enter()` and `leave()` respectively.

### Summary

Life is short. Go eat some tacos. 

### Resources

* [DispatchQueue Documentation](https://developer.apple.com/documentation/dispatch/dispatchqueue)
* [DispatchGroup Documentation](https://developer.apple.com/documentation/dispatch/dispatchgroup)
* [Grand Central Dispatch Documentation](https://developer.apple.com/documentation/dispatch)
* [Grand Central Dispatch Wikipedia](https://en.wikipedia.org/wiki/Grand_Central_Dispatch)
* [Grand Central Dispatch Github Page](https://apple.github.io/swift-corelibs-libdispatch/)
* [swift-corelibs-dispatch](https://github.com/apple/swift-corelibs-libdispatch/)