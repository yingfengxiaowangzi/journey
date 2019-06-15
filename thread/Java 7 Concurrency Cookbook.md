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