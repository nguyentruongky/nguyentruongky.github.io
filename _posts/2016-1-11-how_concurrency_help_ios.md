---
layout: post
title:  "How concurrency helps in iOS"
subtitle: "Concurrency, a technique can't be ignored in any languages"
date:   2016-1-11
background: '/img/posts/queue.jpg'
---

Without concurrency, all tasks run in the main thread, and the app will be freeze when a heavy task is (eg: downloading, uploading, image processing...) running. Concurrency can give the users a better experience. The heavy tasks are running in background and they can do anything they want on the UI as usual.

**This is not a tutorial, it's only a note. What do I have to remember and understand about concurrency programming. The demo project can be found [here](https://github.com/nguyentruongky/ConcurrentProgrammingDemo).**

There are 2 concurrency concepts: dispatch queues and NSOperationQueues

## GCD - Grand Central Dispatch

GCD conform with the FIFO order. GCD will run tasks in separate queues, not the main queue so that your app won't be freezed. There are 2 dispatch queues: serial queues and concurrency queues.

#### Serial Queues

The serial queue can only run a task at a time. The second task will run right after the first task has been done. Fortunately, we can create many serial queues and run them concurrently.

Serial queues are useful for managing shared resources and avoiding race condition. In addition, the advantage of serial queues I like the most is tasks are executed in a predictable order. They executed in the same order as they are inserted.

In below demo, I downloaded 4 images by using serial queue.

> ```
> func runSerialQueue() {
>     let serialQueue = dispatch_queue_create("imagesQueue", DISPATCH_QUEUE_SERIAL)
>     dispatch_async(serialQueue) { () -> Void in
>         self.runDownloadTask(0)
>     }
>     dispatch_async(serialQueue) { () -> Void in
>         self.runDownloadTask(1)
>     }
>     dispatch_async(serialQueue) { () -> Void in
>         self.runDownloadTask(2)
>     }
>     dispatch_async(serialQueue) { () -> Void in
>         self.runDownloadTask(3)
>     }
> }
> 
> ```

You can create more queues and add task to new queues. Those queues will run concurrently with `imagesQueue`.

And here is download function:

> ```
> func runDownloadTask(index: Int) {
>     let img = self.downloadImage(self.imageUrl[index])
>     dispatch_async(dispatch_get_main_queue(), {
>         self.images[index].image = img
>         self.downloadCompleted()
>     })
> }
> 
> ```

Take a look at this cheat sheet. It can help you remember about GCD.

![gcd-cheatsheet](https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fgcd%2Fgcd-cheatsheet.png?alt=media&token=01784ece-24b1-48b7-be8a-073a3c0116bf)

#### Concurrent Queues

Concurrent queues execute multiple tasks at a time. You will not know the order of execution, execution time.

Besides the main queue, the system provides four concurrent queues. We call them Global Dispatch queues.

-   DISPATCH\_QUEUE\_PRIORITY_HIGH
-   DISPATCH\_QUEUE\_PRIORITY_DEFAULT
-   DISPATCH\_QUEUE\_PRIORITY_LOW
-   DISPATCH\_QUEUE\_PRIORITY_BACKGROUND

Here is how to download images with concurrent queues

> ```
> func runConcurrentQueue() {
>     let queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)
>     dispatch_async(queue) { () -> Void in
>         self.runDownloadTask(0)
>     }
>     dispatch_async(queue) { () -> Void in
>        self.runDownloadTask(1)
>     }
>     dispatch_async(queue) { () -> Void in
>         self.runDownloadTask(2)
>     }
>     dispatch_async(queue) { () -> Void in
>         self.runDownloadTask(3)
>     }
> }
> ```

## Operation Queue

Here are how operation queues are different from dispatch queues:

-   Don't follow FIFO: in operation queues, you can set an execution priority for operations and you can add dependencies between operations which means you can define that some operations will only be executed after the completion of other operations.
-   By default, they operate concurrently: while you can't change its type to serial queues, there is still a workaround to execute tasks in operation queues in sequence by using dependencies between operations.
-   Operation queues are instances of class NSOperationQueue and its tasks are encapsulated in instances of NSOperation.

The advantages of NSOperation

-   First, they support dependencies through the method addDependency(op: NSOperation) in the NSOperation class. When you need to start an operation that depends on the execution of the other, you will want to use NSOperation.

![NSOperation-Fig2](https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fgcd%2Fnsoperation-fig2.png?alt=media&token=102f9633-4ad6-454a-ae30-00fdcda667a3)

-   You can decide the operation priority which higher priority will be executed first.
-   You can cancel an operation after it's added to the queue.

Let's see how it works.

> ```
>  func runNSOperationQueue() {
> 
>     queue = NSOperationQueue();
> 
>     func createOperation(index: Int) -> NSBlockOperation {
> 
>         let operation = NSBlockOperation(block: {
> 
>             self.runDownloadTask(index)
>         })
> 
>         operation.completionBlock = {
>             print("Operation \(index + 1) completed, cancelled:\(operation.cancelled)")
>         }
> 
>         return operation
>     }
> 
>     let operation1 = createOperation(0)
>     queue.addOperation(operation1)
> 
>     let operation2 = createOperation(1)
>     operation2.addDependency(operation1)
>     queue.addOperation(operation2)
> 
>     let operation3 = createOperation(2)
>     operation3.addDependency(operation2)
>     queue.addOperation(operation3)
> 
>     let operation4 = createOperation(3)
>     queue.addOperation(operation4)
> }
> 
> ```

We can see operation 2 (op2) depends on op1 and op3 depends on op2. It means, op1 and op4 run at a time and op1 completed, op2 runs and same to op3.

## Conclusion

Concurrency is extremely strong in building app contacted with server, downloading or processing heavy tasks. It helps our apps run incredible smoothly.

This note is based on [http://www.appcoda.com/ios-concurrency/](http://www.appcoda.com/ios-concurrency/). 
You can download the complete project [at my github](https://github.com/nguyentruongky/ConcurrentProgrammingDemo).