# Multi-threading in Java
(In Progress)

This repo is a collection of multi-threading concept demo in Java.

## Contents
- Thread basics
  - Thread Creation: <i>Thread</i> class, <i>Runnable</i> interface, and <i>Callable</i> interface.
  - Thread Pool by java.util.concurrent.<i>Executors</i>;
  - Thread Properties: ID, name, priority, state, and daemon thread;
  - Thread Pool Properties: blocking queue, hook methods, thread pool shutdown properly.

- Thread communications
  - stop(), join(), interrupt(), yield(), and InterruptedException;
  - <i>ThreadLocal</i>;
  - <i>CountDownLatch</i>.
    
- Critical section, data racing, atomicity, and locking.
- Producer-consumer, semaphore, and more.
  
- Demo of using multi-threading.

## 1. Thread Basics

### 1.1 Single Thread Creation

The first step of multi-threading.

#### 1.1.1 Inheriting <i>Thread</i> Class :link:[link](src/johnston/thread/basic/creation/single_thread/InheritedThread.java)

- Create a class that extends <i>Thread</i> class;
- Override run() method by putting the multi-threading logic in it;
- Create an object of the multi-threaded class;
- Call start() method.

#### 1.1.2 Implementing <i>Runnable</i> Class :link:[link](src/johnston/thread/basic/creation/single_thread/RunnableImpl.java)
- Create a class that implements <i>Runnable</i> interface;
- This interface has run() method only, and no other fields or methods;
- So override run() method by putting the multi-threading logic in it;
- Create a <i>Runnable</i> object, and we cannot directly call start()!
- Create a <i>Thread</i> object, and pass the Runnable object into the constructor;
- Call the Thread object start() method.

#### 1.1.3 Anonymous Class  :link:[link](src/johnston/thread/basic/creation/single_thread/AnonymousThread.java)
- Create a Thread object, then new a Runnable implementation inside the constructor;
- Call start() method!

#### 1.1.4 Anonymous Class with Lambda Expression :link:[link](src/johnston/thread/basic/creation/single_thread/AnonymousLambdaThread.java)
- Similar to 1.1.3, but the code is less again.

#### 1.1.5 :warning: Comparing Thread and Runnable :link:[link](src/johnston/thread/basic/creation/single_thread/ThreadRunnableComparison.java)
In OOD, it favors interface over inheritance, so implementing Runnable interface for thread creation is preferable.

Also, using Runnable interface can greatly decouple data and logic. Class which implements Runnable can start multiple threads
to work on the same data which is inside the class. However, the data should be free from data racing!

In this demo, I created two groups (Alice and Bob group) to eat 5 apples.
- Alice group extends Thread, while Bob group extends Runnable;
- Each Alice group thread cannot collaborate to eat a same set of apples, unless the apple is outside the class;
- A Bob group can start multiple threads to eat a same set of apples, because interface Runnable holds no data field!

#### 1.1.6 Async Thread with Return Value: Callable & FutureTask :link:[link](src/johnston/thread/basic/creation/single_thread/CallableThread.java)
Async thread: thread that runs async task, and supports return result. Basic Thread class and Runnable interface do not support return value on run() method.
- <i>Callable</i> interface: has only one method named call() which is like run() method in Runnable but can return value and throw exception. Callable object cannot be the target
  of Thread instance.
- <i>FutureTask</i> class: the connection between Callable interface and Thread instance.

<br />

### 1.2 Thread Pool Creating Threads
Using Thread class start() method to run a thread is a one-time process. When a thread is terminated, it cannot run again.
For a large scale multi-threaded system, it's very resource-consuming.

There is an alternate way to run multi-threading: using thread pool. Thread pool is provided in Java concurrency package. Its thread is reusable, and also provide better thread management
than threads run by start() method.

Thread pool is created by <i>Executors</i>, a factory method provided by Java concurrency package.

#### 1.2.1 Single Thread Pool :link:[link](src/johnston/thread/basic/creation/executors/ExecutorSingleThread.java)
Executor can create a thread pool with single thread. The execution order is guaranteed FIFO.
 
When using Thread class to run a thread, it's one time usage, but a thread pool can reuse
its thread slot.
 
In this Demo, a single thread pool is loaded multiple threads, and these threads will be
executed in FIFO order.

Single thread pool is a blocking queue with unlimited size.

#### 1.2.2 Fixed-size Thread Pool :link:[link](src/johnston/thread/basic/creation/executors/ExecutorFixedThreadPool.java)
Thread pool with fixed size n can allow at most n threads run at the same time. The order is
still FIFO.

#### 1.2.3 Cached Thread Pool :link:[link](src/johnston/thread/basic/creation/executors/ExecutorCachedThreadPool.java)
Cached thread pool has mutable size. Each new coming task will be allocated a new thread. The
size of cached thread pool is unbounded until reaches the limit that JVM allows.

A thread whose idle time is over 60s will be terminated.

#### 1.2.4 Thread Pool Returning Result :link:[link](src/johnston/thread/basic/creation/executors/ExecutorReturnablePool.java)
Using <i>Executors</i> factory can create a thread pool. The thread pool can run Runnable thread or Callable thread.

The thread pool can better manage multi running threads, including limiting maximum concurrent threads, and utilize system resources.

#### 1.2.5 Scheduled Thread Pool :link:[link](src/johnston/thread/basic/creation/executors/ExecutorScheduledThreadPool.java)
Scheduled thread pool supports repeatedly running a thread task with fixed time of period.
Thread pool will run the scheduled task repeatedly until calling shutdown() method.

<i>scheduledThreadPool</i> has two types of schedule methods: fixed period or fixed delay.

For fixed period, the period is minimal runtime of one task. If task A1 runs longer than
the period, than it won't start task A2 until A1 finishes. If task A1 runs faster than the
period, A2 won't start but wait until the minimal interval time is satisfied.

For fixed delay, the delay is the fixed interval between two tasks. If task A1 finishes, it
will wait the delay time than start task A2.

Beside scheduled thread pool, Executors also provides single thread scheduled executor.

In this demo, a set of threads with different scheduled periods are loaded into to thread pool,
and let the thread pool run for a while. The result shows that threads with smaller scheduled
period run more times than others. Fixed delay demo skips because it's very similar to this one.

#### 1.2.6 :warning: Standard ThreadPoolExecutor :link:[link](src/johnston/thread/basic/creation/executors/StandardThreadPoolExecutor.java)
In general, using Executors factory to create thread pool is not allowed in large-scale
development. The standard method is to use standard ThreadPoolExecutor, although Executors
factory invokes ThreadPoolExecutor.

Important ThreadPoolExecutor constructor parameter
corePoolSize: the minimum amount of core threads;
maximumPoolSize: the maximum amount of threads in the pool.
keepAliveTime: the maximum duration of a non-core threads being idle.
threadFactory: creations of new thread.
BlockingQueue<Runnable>: queue to hold tasks when no idle core threads available.
RejectedExecutionHandler: ways to handle new tasks when the pool is full, like throwing
  exception, discard, or replacing the oldest blocking task.

Thread pool task scheduling policy when getting a new task:
 - If the number current core threads is < corePoolSize, new a new thread for this task, even
   though some core threads are idle;
 - If the number current core threads is >= corePoolSize...
   - Find if there are any idle core threads, then replace it;
   - else if no idle core threads, and the blocking queue is not full, enqueue;
   - else if the blocking queue is full, new a new thread until the total number of threads > maximumPoolSize
   - else execute rejected execution policy.

This demo shows how threadPoolExecutor scheduling tasks with different thread pool parameters
and different reject task policies.

#### 1.2.7 ThreadFactory Interface :link:[link](src/johnston/thread/basic/creation/executors/ThreadFactoryDemo.java)
<i>ThreadFactory</i> is an interface from concurrent package. It has only one
method: newThread(Runnable r). User can implement this interface to customize the method of
creating new threads, and pass this interface instance to a thread pool. The thread pools then
will use this thread factory to create reusable new threads.

In this demo, an implementation of ThreadFactory is created and pass it to the thread pool.
When the thread pool receives new Runnable tasks, it will use the threads created by
ThreadFactory to run it.

### 1.3 Thread Properties

#### 1.3.1 Thread ID :link:[link](src/johnston/thread/basic/properties/single_thread/ThreadID.java)
Each thread has a unique ID assigned by the JVM.

To get the id of a thread, call:
- threadA.getId();
- Thread.currentThread().getId();

#### 1.3.2 Thread Naming  :link:[link](src/johnston/thread/basic/properties/single_thread/ThreadNaming.java)
Each thread has its name, either assigned by the JVM or by user.
To set the name of a thread:
- Pass in the name when calling constructor;
- Call threadA.setName("NewName");

To get the name of a thread, call threadA.getName().

#### 1.3.3 Thread Priority :link:[link](src/johnston/thread/basic/properties/single_thread/ThreadPriority.java)
Each thread can set its priority, from 1 as the lowest, to 10 as the highest. Thread
has higher priority can schedule earlier than other threads with lower priority.

To set the priority of threadA, call threadA.setPriority(1).
To get the priority of threadA, call threadA.getPriority().
<i>Thread</i> class provides priority constants:
- Thread.MAX_PRIORITY: 10;
- Thread.MIN_PRIORITY: 1.

The default priority is 5.

This demo class creates 10 threads, and each of them runs a heavy task. The last thread is set
to the highest priority, while others have the lowest priority. Then the highest one always 
finishes first.

#### 1.3.4 Thread State :link:[link](src/johnston/thread/basic/properties/single_thread/ThreadState.java)
A thread can have 6 different states, which represent 6 certain parts of its life cycle.

- NEW: a multi-threaded object is created, and before it runs;
- RUNNABLE: when a thread is running or scheduled to run (JVM called start(), and wait for CPU time slice);
- BLOCKED: when a thread is waiting to enter critical section;
- WAITING/TIMED_WAITING: a thread is called wait(), join(), or sleep();
- TERMINATED: a thread is finished running.

This class simulates all these states of a thread.

#### 1.3.5 :warning: Difference between run() and start() :link:[link](src/johnston/thread/basic/properties/single_thread/RunStartDiff.java)
run() method contains the main logic/task for a thread to run.
start() method make a thread start running run().
So simply calling run() directly would not start multi-threading, it just calls run() method in 
the current thread!

#### 1.3.6 Daemon Thread  :link:[link](src/johnston/thread/basic/properties/single_thread/DaemonThread.java)
In general, a process would not exit execution as long as it has at least one running thread, but some threads are doing 
background task, like file auto saving thread, or producer thread using external resources. These threads should not block 
the process from exiting.

We can set these threads as daemon thread, so they won't bother the main thread when main is about to exit.

Call threadA.setDaemon(true) to mark it as a daemon thread. The default is false.

Only can set a thread daemon before it starts. Trying to set its daemon status when running will
throw InterruptedException.

The threads created by Daemon thread are Daemon threads by default. It's allowed to manually set them as user thread.

<br />

### 1.4 Thread Pool Properties


#### 1.4.1 Thread Pool Shutdown :link:[link](src/johnston/thread/basic/properties/executors/ThreadPoolShutdown.java)
A thread pool can run forever if not calling shutdown() method, which will prevent the main
thread from terminating.

When shutdown() is called, the thread pool will no longer receive new thread task, and waits
for all threads in queue are executed, then exits.

Adding new threads on a shutdown thread pool would throw RejectedExecutionException.

In this demo, four tasks are added to the thread pool then shut it down. Four tasks will be
finished after shutting down the thread pool.

#### 1.4.2 Thread Pool Shutdown Now :link:[link](src/johnston/thread/basic/properties/executors/ThreadPoolShutdownNow.java)
If shutdownNow() is called, the thread pool will terminate all threads that are currently
running, and return the tasks that are not yet started.

#### 1.4.3 :warning: BlockingQueue :link:[link](src/johnston/thread/basic/properties/executors/ThreadBlockingQueue.java)
Blocking queue is a queue that if the size is empty and one thread requires dequeue, then it
blocks that thread (wait) until one element is available to dequeue. Each thread pool needs
a blocking queue.

Blocking queue has several implementations:
- <i>ArrayBlockingQueue</i>: queue implemented by array. The order is FIFO. Size must be defined.

- <i>LinkedBlockingQueue</i>: queue implemented by linked-list. The order is FIFO. Size can be
  defined. If not, then the size is unlimited.

- <i>PriorityBlockingQueue</i>: a priority queue that can take in comparator. If no comparator is
  provided, then use the natural order of the elements. Size is unlimited.

- <i>DelayQueue</i>: like a blocking queue. Elements can dequeue only if their time is expired. It
  can be used in producer-consumer scenario.
  The elements stored in DelayQueue must implement Delayed interface. The element that will
  expire first will be dequeued first. It cannot dequeue unexpired elements.

- <i>SynchronousQueue</i>: a queue with size 1. This queue allows two threads exchange data
  thread-safely.
    - take(): if the queue has no data, then the caller will be waiting until data available.
    - poll(): like take(), but if no data available, then it returns null instead of waiting.
    - put(E e): the caller has to wait until another thread takes away the data.
    - offer(E e): enqueue data only if another thread is waiting for data.
      -isEmpty(): ALWAYS return true!

This demo contains usage of four types of BlockingQueue implementations.

#### 1.4.4 Thread Pool Hook Methods :link:[link](src/johnston/thread/basic/properties/executors/ThreadPoolHookMethods.java)
ThreadPoolExecutor has three hood methods:
 - void beforeExecute(Thread t, Runnable target): this method runs before each task begins.
 - afterExecute(Runnable target, Throwable t): this method runs after each task begins.
 - terminated(): this method runs when the pool is shutdown.

 These three methods can be overridden to do some tasks, like customize environment, clean up
 data, etc.

 This demo overrides these three methods to record the total runtime of all Runnable tasks. The
 task will sleep random milli sec.

#### 1.4.5 Thread Pool Rejection Policy
 - <i>DiscardOldestPolicy()</i>: dequeue & discard one task to make room
 - <i>AbortPolicy()</i>: throw RejectedExecutionException
 - <i>CallerRunsPolicy()</i>: thread who submit task run task itself.
 - <i>DiscardPolicy()</i>: silent AbortPolicy().
 - self-defined policy: implements the method <i>rejectedExecution</i> in the interface RejectedExecutionHandler.

Demo see 1.2.6.

#### 1.4.6 Thread Pool Properly Shutdown :link:[link](src/johnston/thread/basic/properties/executors/ThreadPoolProperShutDown.java)
General way to shut down a thread pool:
 - Call shutdown() to stop accepting new task;
 - awaitTermination(long timeout, TimeUnit unit) to wait for the existing task to finish.
 - If timeout, call shutdownNow() to force all task finished.
 - Iteratively call shutdownNow() -- awaitTermination()

## 2. Thread Communications

#### 2.1 (Danger) Stop a thread :link:[link](src/johnston/thread/communications/ThreadStop.java)
Calling stop() method of a thread can immediately terminate that thread.

This operation is not safety. If a stopped thread holds a lock, then it
would not release it. If the thread which is about to stop is using database, then it would
cause data inconsistency. So this method is @Deprecated.

#### 2.2 Thread Join :link:[link](src/johnston/thread/communications/ThreadJoin.java)
 When calling threadB.join() in threadA, threadA would wait until threadB finishes the
 work.

 Suppose threadA needs the result from threadB, then threadA can call threadB.join() to
 wait for the threadB instead of using a while loop to run a spinlock.

 ThreadA that called join() may throw InterruptedException if other threads interrupt ThreadA.

 This demo creates two threads (A & B) to run a random int generator until get the target number.
 Main thread will wait until A exits, but will not wait B. So main thread always get the target
 number from A and needs some luck to get the same from B.
 
#### 2.3 Thread Interruption :link:[link](src/johnston/thread/communications/ThreadInterruption.java)
When threadA calls threadB.interrupt(), threadB can receive an interruption signal, and
decide what to do next. So calling interrupt() to threadB does not stop or pause threadB if
it's running (not waiting or sleeping).

This demo is to interrupt a thread.

#### 2.4 InterruptedException :link:[link](src/johnston/thread/communications/ThreadInterruptedException.java)
When interrupting a sleeping or joined thread, it will throw InterruptedException.

#### 2.5 Thread Yielding :link:[link](src/johnston/thread/communications/YieldThread.java)
Method yield() is provided by Thread class. When a thread calls Thread.yield(), it gives up
the usage of CPU and put itself into the thread scheduling, which depends on the thread
priority. The state is still Runnable, but it's "ready to run" instead of "running".

In this demo, an array of low priority thread, and an array of high priority thread yield to
each other, and the final result shows that high priority threads have more execution times than the lower
priority ones.

#### 2.6 ThreadLocal :link:[link](src/johnston/thread/communications/ThreadLocalInnerCommunication.java)
<i>ThreadLocal</i> is a convenient way to ensure data safety in multi-threading. It's like a hash map
where the key is the thread task id, and the value is the variable belonging to that thread task
only. It's a more efficient way to ensure data-racing free than using "synchronized" keyword.
ThreadLocal also helps decouple among multi-threading methods and class when variable sharing is
required.

ThreadLocal is usually static and final. Static ensures it can be used across all threads, and
final ensures it won't be replaced causing threads losing their data.

Always use remove() method to remove instance when the current thread is about to terminate, because
ThreadLocal is static and final, so it will hold reference of a thread instance forever even though
it's terminated.

In this demo, each Runnable task has its own unique random number n, and it creates a variable
in the ThreadLocal object, then increment that variable n times. The result shows that
ThreadLocal would not mix the variables that each of them belongs to one Runnable task only.

#### 2.7 CountDownLatch Waiting :link:[link](src/johnston/thread/communications/CountDownLatchWaitBlocking.java)
<i>CountDownLatch</i> is a decremental counter for multi-threading. It inits as an integer, and any
threads can call countDown() to make is decrement one time. Threads calling CountDownLatch::await
will be blocked until the counter is 0. It's like a join() method that can specify the location
of exit-joining point instead of waiting the joined thread terminated.

#### 2.8 CountDownLatch All Threads Starting Together :link:[link](src/johnston/thread/communications/CountDownLatchWaitingToBegin.java)
This is a different usage of CountDownLatch. Instead of letting the calling wait() thread to
wait until decrement to 0, let all threads starts at the same time by calling await()!

## 3. Locking and Thread Safety

###3.1 Basics

#### 3.1.1 Data Racing :link:[link](src/johnston/thread/thread_safety_and_locking/DataRacing.java)
If a set of operation is not <i>atomic</i>, then <i>race condition</i> may have. Race condition can cause <i>data racing</i>.

This demo shows that incremental operation <i>var++</i> is not thread-safety.

#### 3.1.2 Lock Strategy Comparison :link:[link](src/johnston/thread/thread_safety_and_locking/LockStrategyComparison.java)
Exclusively locking critical section can avoid data racing, and comes with performance overhead.
So minimizing the critical section execution code and using more flexible locking strategy can
help to improve performance.

This demo compares performance on two different strategies of locking: using one lock for all
synced variable, and each variable uses one specific lock.

#### 3.1.3 Synchronized Static Method  :link:[link](src/johnston/thread/thread_safety_and_locking/StaticMethodLock.java)
Using synchronized keyword to modify static method is different to modify non-static method.
Synchronized non-static method is object-level lock, while synchronized static method is
class-level lock.

This demo shows that threads from two different objects can access a synchronized non-static
method at the same time, while synchronized static method allows only one thread entered at
the same time.

### 3.2 Producer-Consumer Problem
Producer-Consumer Problem is same as Bounded-Buffer Problem. Producers and consumers share a same buffer. Producers
put data to the buffer while consumers take data from it. It requires:
- Producers and consumers should work concurrently.
- The buffer capacity is limited.
- Producers cannot put data to the buffer when it's full.
- Consumers cannot take any data when the buffer is empty, and won't take duplicated data.
- When producers and consumers are waiting, their threads should not be blocked.

#### 3.2.1 Producer-Consumer Prep. :link:[link](src/johnston/thread/thread_safety_and_locking/consumer_producer/components)
This package contains:
- Producer and produce action (Callable);
- Consumer and consume action (Callable);
- Buffer interface and its implementations.

#### 3.2.2 Bad Producer-Consumer  :link:[link](src/johnston/thread/thread_safety_and_locking/consumer_producer/BadConsumingProducing.java)
Demo of producer and consumer using not thread-safe buffer. If there are more than one producer
and more than one consumer, then race condition exists at:
- Between buffer.get() size check and get -- size != 0 but get null, or index out of bound;
- Between buffer.put() size check and put -- size > capacity.

This is because both check size and get element operation are atomic, but putting them together
is not! Same as check size and put element operation.

#### 3.2.3 Synced Read & Write Producer-Consumer  :link:[link](src/johnston/thread/thread_safety_and_locking/consumer_producer/SyncedConsumingProducing.java)
Demo of producer and consumer using not thread-safe buffer. If there are more than one producer
and more than one consumer, then race condition exists at:
- Between buffer.get() size check and get -- size != 0 but get null, or index out of bound;
- Between buffer.put() size check and put -- size > capacity.

This is because both check size and get element operation are atomic, but putting them together
is not! Same as check size and put element operation.

## 4. 

## 5. Demos of Using Multi-threading

### 5.1 Matrix Multiplication :link:[link](src/johnston/thread/demo/multi_threading/MatrixMultiplication.java)
Matrix multiplication is a computationally heavy task. Using multi-threading to calculate matrix multiplication
is 4X faster than single-threaded. This demo uses thread pool to create worker threads.

User can define the CPU core amount and thread amount. The thread amount is how many working threads will be
created, and the CPU core amount is how many threads run concurrently.