## 线程管理（二）获取和设置线程信息

- Thread类的对象中保存了一些属性信息

- - Status: 线程的状态。在Java中，线程只能有这6种中的一种状态： 

    - new, runnable, blocked, waiting, time waiting, 或 terminated.

    #### JAVA 线程状态 阻塞和等待 bloked 和 waiting 区别

    **NEW**

    > A thread that has not yet started is in this state.

    一个被创建的线程，但是还没有调用start方法

    **RUNNABLE**

    > A thread executing in the Java virtual machine is in this state.

    一个正在被执行的线程的状态

    **BLOCKED**

    > A thread that is blocked waiting for a monitor lock is in this state.

    一个线程因为等待临界区的锁被阻塞产生的状态 
    Lock 或者synchronize 关键字产生的状态

    **WAITING**

    > A thread that is waiting indefinitely for another thread to perform a 
    > particular action is in this state.

    一个线程进入了锁，但是需要等待其他线程执行某些操作。时间不确定 
    当wait，join，park方法调用时，进入waiting状态。前提是这个线程已经拥有锁了。

    **TIMED_WAITING**

    > A thread that is waiting for another thread to perform an action for 
    > up to a specified waiting time is in this state.

    一个线程进入了锁，但是需要等待其他线程执行某些操作。时间确定 
    通过sleep或wait timeout方法进入的限期等待的状态)

    **TERMINATED**

    > A thread that has exited is in this state.

    退出

  -  new, runnable, blocked,terminated：例子中存在这四种状态。

## 线程管理（三）线程的中断

- Java提供中断机制来通知线程表明我们想要结束它。中断机制的特性是线程需要检查是否被中断，而且还可以决定是否响应结束的请求。所以，线程可以忽略中断请求并且继续运行。
- join方法，main线程调用一个线程的join方法，随后主线程会等待这个线程执行结束？
- interrupt()方法，正在执行的线程，会执行结束？？

## 基本线程同步（七）修改Lock的公平性

- 选择等待时间最长的线程。

# 第三章

## 线程同步工具（一）控制并发访问资源

如果semaphore的计数器的值等于0，那么semaphore让线程进入休眠状态一直到计数器大于0。计数器的值等于0表示全部的共享资源都正被线程们使用，所以此线程想要访问就必须等到某个资源成为自由的。

称为binary semaphores。这个semaphores种类保护访问共享资源的独特性，所以semaphore的内部计数器的值只能是1或者0。

```java
public Semaphore(int permits) {
  sync = new NonfairSync(permits);
}

// Semaphores的公平性
public Semaphore(int permits, boolean fair) {
  sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}

public void acquire() throws InterruptedException {
  sync.acquireSharedInterruptibly(1);
}

public void release() {
  sync.releaseShared(1);
}
```

The acquire(), acquireUninterruptibly(), tryAcquire(),和release()方法有一个外加的包含一个int参数的版本。这个参数表示 线程想要获取或者释放semaphore的许可数。

* CountDownLatch 

​		机制不是用来保护共享资源或者临界区。它是用来同步一个或者多个执行多个任务的线程。它只能使用一次。像之前解说的，一旦CountDownLatch的计数器到达0，任何对它的方法的调用都是无效的。如果你想再次同步，你必须创建新的对象。

## 线程同步工具（二）控制并发访问多个资源

Semaphore对象创建的构造方法是使用3作为参数的。这个例子中，前3个调用acquire() 方法的线程会获得临界区的访问权，其余的都会被阻塞 。当一个线程结束临界区的访问并解放semaphore时，另外的线程才可能获得访问权。

## 线程同步工具（四）在同一个点同步任务

Java 并发 API 提供了可以允许2个或多个线程在在一个确定点的同步应用。它是 CyclicBarrier 类。

使用 CyclicBarrier 类来让一组线程在一个确定点同步

```java
// Runnable对象作为CyclicBarrier的构造参数。等待屏障上的线程达到一定数量后开始执行。
// 在给定的线程数量执行完成后，barrierAction开始执行；由进入屏障的最后一个线程执行。
public CyclicBarrier(int parties, Runnable barrierAction) {
        if (parties <= 0) throw new IllegalArgumentException();
        this.parties = parties;
        this.count = parties;
        this.barrierCommand = barrierAction;
    }

// 等到所有{@linkplain #getParties party}都被调用
java.util.concurrent.CyclicBarrier#await()
```

可以作为一个Runnable实现类的属性，在run()方法内调用await()，由进入屏障的最后一个线程执行。

**重置 CyclicBarrier 对象**
CyclicBarrier 类与CountDownLatch有一些共同点，但是也有一些不同。最主要的不同是，CyclicBarrier对象可以重置到它的初始状态，重新分配新的值给内部计数器，即使它已经被初始过了。

可以使用 CyclicBarrier的reset() 方法来进行重置操作。当这个方法被调用后，全部的正在await() 方法里等待的线程接收到一个 BrokenBarrierException 异常。

CyclicBarrier 对象可能处于一个特殊的状态，称为 broken。当多个线程正在 await() 方法中等待时，其中一个被中断了，此线程会收到 InterruptedException 异常，但是其他正在等待的线程将收到 BrokenBarrierException 异常，并且 CyclicBarrier 会被置于broken 状态中。

## 线程同步工具（五）运行阶段性并发任务

Phaser类运行阶段性的并发任务

Phaser类提供的机制是在每个步骤的结尾同步线程，所以除非全部线程完成第一个步骤，否则线程不能开始进行第二步。

必须初始化Phaser类与这次同步操作有关的任务数，我们可以通过增加或者减少来不断的改变这个数。

Phaser对象的 arriveAndAwaitAdvance()：

​		Phaser减少终结actual phase的线程数，并让这个线程进入休眠 直到全部其余线程结束phase

arriveAndDeregister()：

​		它通知phaser线程结束了actual phase， 但是它将不会继续参见后面的phases,所以请phaser不要再等待它了。

isTerminated() ：

​		判断是否进入termination状态。

​		当phaser 有0个参与者时，它进入termination状态

Phaser 对象可能是在这2中状态

​		Active

​		Termination: 默认状态，当Phaser里全部的参与者都取消注册，它进入这个状态，所以这时 Phaser 有0个参与者。



## 线程同步工具（六）控制并发阶段性任务的改变

onAdvance() 方法：在phaser前进的时候；on，当 。。。。的时候

​		on方法，一般都是回调的时候使用。

​		onAdvance() 方法返回 Boolean 值表明 phaser 终结与否。

register() 方法：创建了phaser的参与者的注册



## 线程同步工具（七）在并发任务间交换数据

2个并发任务间相互交换数据的同步应用

Exchanger 类允许在2个线程间定义同步点，当2个线程到达这个点，他们相互交换数据类型，使用第一个线程的数据类型变成第二个的，然后第二个线程的数据类型变成第一个的。

生产者和消费者问题: 只是Exchange类只能同步2个线程，所以你只能在你的生产者和消费者问题中只有一个生产者和一个消费者时使用这个类。

exchange()方法：

​		`public V exchange(V x) throws InterruptedException {}`



# 第四章

## 线程执行者（二）创建一个线程执行者

使用ThreadPoolExecutor类的shutdown()方法来表明你想要结束执行者。在你调用shutdown()方法之后，如果你试图提交其他任务给执行者，它将会拒绝，并且抛出RejectedExecutionException异常。

- getPoolSize()：此方法返回线程池实际的线程数。
- getActiveCount()：此方法返回在执行者中正在执行任务的线程数。
- `getCompletedTaskCount()`：此方法返回执行者完成的任务数。

- shutdownNow()：此方法立即关闭执行者。它不会执行待处理的任务，但是它会返回待处理任务的列表。当你调用这个方法时，正在运行的任务继续它们的执行，但这个方法并不会等待它们的结束。
- isTerminated()：如果你已经调用shutdown()或shutdownNow()方法，并且执行者完成关闭它的处理时，此方法返回true。
- isShutdown()：如果你在执行者中调用shutdown()方法，此方法返回true。
- awaitTermination(long timeout, TimeUnit unit)：此方法阻塞调用线程，直到执行者的任务结束或超时。TimeUnit类是个枚举类，有如下常 量：DAYS，HOURS，MICROSECONDS， MILLISECONDS， MINUTES,，NANOSECONDS 和SECONDS。

使用Executor framework的第一步就是创建一个ThreadPoolExecutor类的对象。你可以使用这个类提供的4个构造器或Executors工厂类来 创建ThreadPoolExecutor。一旦有执行者，你就可以提交Runnable或Callable对象给执行者来执行。

缓存线程池的缺点是，为新任务不断创建线程， 所以如果你提交过多的任务给执行者，会使系统超载。

这个执行者为每个接收到的任务创建一个线程（如果池中没有空闲的线程），所以，如果你提交大量的任务，并且它们有很长的（执行）时间，你会使系统过载和引发应用程序性能不佳的问题。



## 线程执行者（三）创建一个大小固定的线程执行者

固定的线程执行者。这个执行者有最大线程数。 如果你提交超过这个最大线程数的任务，这个执行者将不会创建额外的线程，并且剩下的任务将会阻塞，直到执行者有空闲线程。这种行为，保证执行者不会引发应用程序性能不佳的问题。

`executor=(ThreadPoolExecutor)Executors.newFixedThreadPool(5);`

- getPoolSize()：此方法返回线程池实际的线程数。
- getActiveCount()：此方法返回在执行者中正在执行任务的线程数。

submit()方法提交一个Callable对象给执行者执行。这个方法接收Callable对象参数，并且返回一个Future对象

- 你可以控制任务的状态：你可以取消任务，检查任务是否已经完成。基于这个目的，你已经使用isDone()方法来检查任务是否已经完成。
- 你 可以获取call()方法返回的结果。基于这个目的，你已经使用了get()方法。这个方法会等待，直到Callable对象完成call()方法的执 行，并且返回它的结果。如果线程在get()方法上等待结果时被中断，它将抛出InterruptedException异常。如果call()方法抛出 异常，这个方法会抛出ExecutionException异常。

## 线程执行者（五）运行多个任务并处理第一个结果

ThreadPoolExecutor类中的invokeAny()方法接收任务数列，并启动它们，返回完成时没有抛出异常的第一个 任务的结果。该方法返回的数据类型与启动任务的call()方法返回的类型一样。

我们有两个任务，可以返回true值或抛出异常。有以下4种情况：

- 两个任务都返回ture。invokeAny()方法的结果是第一个完成任务的名称。
- 第一个任务返回true，第二个任务抛出异常。invokeAny()方法的结果是第一个任务的名称。
- 第一个任务抛出异常，第二个任务返回true。invokeAny()方法的结果是第二个任务的名称。
- 两个任务都抛出异常。在本例中，invokeAny()方法抛出一个ExecutionException异常。

ThreadPoolExecutor类提供其他版本的invokeAny()方法：

- invokeAny(Collection<? extends Callable<T>> tasks, long timeout,TimeUnit unit)：此方法执行所有任务，并返回第一个完成（未超时）且没有抛出异常的任务的结果。

```java
<T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;
```



## 线程执行者（六）运行多个任务并处理所有结果

```java
// 返回值是一个list列表
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;
```

如果你想要等待一个任务完成，你可以使用以下两种方法：

- 如果任务执行完成，Future接口的isDone()方法将返回true。
- ThreadPoolExecutor类的awaitTermination()方法使线程进入睡眠，直到每一个任务调用shutdown()方法之后完成执行。

这两种方法都有一些缺点。第一个方法，你只能控制一个任务的完成。第二个方法，你必须等待一个线程来关闭执行者，否则这个方法的调用立即返回。

invokeAll()方法，你会使用Future对象获取任务的结果。当所有的任务完成时，这个方法结束（阻塞，等待所有线程都返回结果），如果你调用返回的Future对象的isDone()方法，所有调用都将返回true值。



## 线程执行者（七）执行者延迟运行一个任务

```java
ScheduledThreadPoolExecutor executor = (ScheduledThreadPoolExecutor)
                Executors.newScheduledThreadPool(1);

executor.schedule(task, i + 1, TimeUnit.SECONDS);
```

```java
// shutdown方法，下边的代码继续执行
executor.shutdown();
try {
  	// 阻塞等待。和shutdown配合使用。
  	executor.awaitTermination(1, TimeUnit.DAYS);
} catch (InterruptedException e) {
  	e.printStackTrace();
}
```

* `java.util.concurrent.ScheduledThreadPoolExecutor#shutdown`
  * 启动有序关闭，其中先前提交的任务被执行，但不会接受任何新任务。
  * 阻塞操作？？？，线程执行完成后，才会被关闭。

## 线程执行者（八）执行者周期性地运行一个任务

通过ScheduledThreadPoolExecutor类可以执行周期性任务

```java
ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);

ScheduledFuture<?> result=executor.scheduleAtFixedRate(task,1, 2, TimeUnit.SECONDS);
```

很重要的一点需要考虑的是两次执行之间的（间隔）期间，是这两个执行开始之间的一段时间。如果你有一个花5秒执行的周期性任务，而你给一段3秒时间，同一时刻，你将会有两个任务在执行。

scheduleAtFixedRate() 方法返回ScheduledFuture对象，它继承Future接口，这个方法和调度任务一起协同工作。ScheduledFuture是一个参数化接口（校对注：ScheduledFuture<V>）

getDelay()方法返回直到任务的下次执行时间。这个方法接收一个TimeUnit常量，这是你想要接收结果的时间单位。

在 scheduledWithFixedRate()方法中，参数决定任务执行结束与下次执行开始之间的一段时间。

 shutdown()方法时，你也可以通过参数配置一个SeduledThreadPoolExecutor的行为。shutdown()方法默认的行为是，当你调用这个方法时，计划任务就结束。

setContinueExistingPeriodicTasksAfterShutdownPolicy()方法设置true值改变这个行为。在调用 shutdown()方法时，周期性任务将不会结束。

### 线程执行者（九）执行者取消一个任务

取消你已提交给执行者的任务，使用Future接口的cancel()方法。

- 如果这个任务已经完成或之前的已被取消或由于其他原因不能被取消，那么这个方法将会返回false并且这个任务不会被取消。
- 如果这个任务正在等待执行者获取执行它的线程，那么这个任务将被取消而且不会开始它的执行。如果这个任务已经正在运行，则视方法的参数情况而定。 cancel()方法接收一个Boolean值参数。如果参数为true并且任务正在运行，那么这个任务将被取消。如果参数为false并且任务正在运行，那么这个任务将不会被取消。



### 线程执行者（十）执行者控制一个任务完成

FutureTask类提供一个done()方法，允许你在执行者执行任务完成后执行一些代码。

在建立返回值和改变任务的状态为isDone状态后，done()方法被FutureTask类内部调用。你不能改变任务的结果值和它的状态，但你可以关闭任务使用的资源，写日志消息，或发送通知。

### 线程执行者（十一）执行者分离任务的启动和结果的处理	

提交任务给执行者在一个对象中，而处理结果在另一个对象中。

CompletionService 类有一个方法来提交任务给执行者和另一个方法来获取已完成执行的下个任务的Future对象。在内部实现中，它使用Executor对象执行任务。这种行为的优点是共享一个CompletionService对象，并提交任务给执行者，这样其他（对象）可以处理结果。其局限性是，第二个对象只能获取那些已经完成它们的执行的任务的Future对象，所以，这些Future对象只能获取任务的结果。

```java
public interface CompletionService<V> {
	  // 获取CompletionService执行的下个已完成任务的Future对象。一直等到超时时间。
   Future<V> poll(long timeout, TimeUnit unit) throws InterruptedException;
}
```

当其中一个任务被执行，CompletionService完成这个任务的执行时，这个CompletionService在一个队列中存储Future对象来控制它的执行。poll()方法用来查看这个列队，如果有任何任务执行完成，那么返回列队的第一个元素，它是一个已完成任务的Future对象。当poll()方法返回一个Future对象时，它将这个Future对象从队列中删除。这种情况下，你可以传两个属性给那个方法，表明你想要等任务结果的时间，以防队列中的已完成任务的结果是空的。

- poll()：不带参数版本的poll()方法，检查是否有任何Future对象在队列中。如果列队是空的，它立即返回null。否则，它返回第一个元素，并从列队中删除它。
- take()：这个方法，不带参数。检查是否有任何Future对象在队列中。如果队列是空的，它阻塞线程直到队列有一个元素。当队列有元素，它返回第一元素，并从列队中删除它。

### 线程执行者（十二）执行者控制被拒绝的任务

RejectedExecutionHandler接口。该接口有带有两个参数的方法rejectedExecution()：

- Runnable对象，存储被拒绝的任务
- Executor对象，存储拒绝任务的执行者

`java.util.concurrent.ThreadPoolExecutor#setRejectedExecutionHandler`

```java
public void setRejectedExecutionHandler(RejectedExecutionHandler handler) {}
```

当执行者接收任务时，会检查shutdown()是否已经被调用了。如果被调用了，它拒绝这个任务。首先，它查找 setRejectedExecutionHandler()设置的处理器。如果有一个（处理器），它调用那个类的 rejectedExecution()方法，否则，它将抛RejectedExecutionExeption异常。这是一个运行时异常，所以你不需要 用catch语句来控制它。

接口的方法，拒绝任务时，回调这个方法：

`java.util.concurrent.RejectedExecutionHandler#rejectedExecution`

# 第五章

## Fork/Join框架（一）引言

Java 5引入了Executor和ExecutorService接口及其实现类进行了改进（ThreadPoolExecutor）：执行者框架将任务的创建与执行分离。有了它，你只要实现Runnable对象和使用Executor对象。你提交Runnable任务给执行者，它创建、管理线程来执行这些任务。

Java 7更进一步，包括一个面向特定问题的ExecutorService接口的额外实现，它就是Fork/Join框架。

![1](http://ifeve.com/wp-content/uploads/2013/08/1.jpg)

这个框架被设计用来解决可以使用分而治之技术将任务分解成更小的问题。

这个框架基于以下两种操作：

- fork操作：当你把任务分成更小的任务和使用这个框架执行它们。
- join操作：当一个任务等待它创建的任务的结束。

Fork/Join 和Executor框架主要的区别是work-stealing算法。不像Executor框架，当一个任务正在等待它使用join操作创建的子任务的结束时，执行这个任务的线程（工作线程）查找其他未被执行的任务并开始它的执行。通过这种方式，线程充分利用它们的运行时间，从而提高了应用程序的性能。

为实现这个目标，Fork/Join框架执行的任务有以下局限性：

- 任务只能使用fork()和join()操作，作为同步机制。如果使用其他同步机制，工作线程不能执行其他任务，当它们在同步操作时。比如，在Fork/Join框架中，你使任务进入睡眠，正在执行这个任务的工作线程将不会执行其他任务，在这睡眠期间内。
- 任务不应该执行I/O操作，如读或写数据文件。
- 任务不能抛出检查异常，它必须包括必要的代码来处理它们。

Fork/Join框架的核心是由以下两个类：

- ForkJoinPool：它实现ExecutorService接口和work-stealing算法。它管理工作线程和提供关于任务的状态和它们执行的信息。
- ForkJoinTask： 它是将在ForkJoinPool中执行的任务的基类。它提供在任务中执行fork()和join()操作的机制，并且这两个方法控制任务的状态。通常， 为了实现你的Fork/Join任务，你将实现两个子类的子类的类：RecursiveAction对于没有返回结果的任务和RecursiveTask 对于返回结果的任务。

## Fork/Join框架（二）创建一个Fork/Join池

`java.util.concurrent.RecursiveAction#compute`

`java.util.concurrent.ForkJoinTask#getQueuedTaskCount`

`java.util.concurrent.ForkJoinTask#invokeAll(java.util.concurrent.ForkJoinTask<?>, java.util.concurrent.ForkJoinTask<?>)`

`java.util.concurrent.ForkJoinPool#getActiveThreadCount`

`java.util.concurrent.ForkJoinPool#getStealCount`

`java.util.concurrent.ForkJoinPool#getParallelism`

```java
public abstract class RecursiveAction extends ForkJoinTask<Void> {}

public abstract class ForkJoinTask<V> implements Future<V>, Serializable {}
```

invokeAll()方法，执行每个任务所创建的子任务。这是一个同步调用，这个任务在继续（可能完成）它的执行之前，必须等待子任务的结束。当任务正在等待它的子任务（结束）时，正在执行它的工作线程执行其他正在等待的任务。在这种行为下，Fork/Join框架比Runnable和Callable对象本身提供一种更高效的任务管理。