# Effective Java（六）

### 并发

### 七十八、同步访问共享的可变数据

互斥，当一个对象被一个线程修改的时候，可以阻止另一个线程观察到对象内部不一致的状态。

同步不仅仅是互斥，它不仅可以阻止一个线程看到对象处于不一致的状态之中，它还可以保证进人同步方法或者同步代码块的每个线程，都能看到由同一个锁保护的之前所有的修改效果。

个人理解：同步 = 互斥 + 通信，volatile只有通信功能，volatile修饰的int做复合操作，比如i++，并不是线程安全的，考虑使用AtomicInteger。

Java 语言规范中的内存模型（ memo可model ），它规定了一个线程所做的变化何时以及如何变成对其他线程可见。

虽然volatile 修饰符不执行互斥访问，但它可以保证任何一个线程在读取该域的时候都将看到最近刚刚被写入的值：

```java
public class StopThread {
    private static volatile boolean stopRequest;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundTask = new Thread(() -> {
            int i = 0;
            while (!stopRequest) {
                i++;
            }
        });
        backgroundTask.start();
        TimeUnit.SECONDS.sleep(1);
        stopRequest = true;
    }
}
```

### 七十九、避免过度同步

### 八十、executor、task和stream优先于线程

### 八十一、并发工具优于wait和notify

java.util.concurre 口t 中更高级的工具分成三类： Executor Framework 、并发集合（Concurrent Collection ）以及同步器（ Synchronizer）。

如果工作线程捕捉到InterruptedException ，就会利用习惯用法Thread.currentThread() . interrupt （）重新断言中断，并从它的run 方法中返回。这样就允许executor 在必要的时候处理中断，事实上也理应如此。

始终应该使用wait 循环模式来调用wait 方法；永远不要在循环之外调用wait 方法。