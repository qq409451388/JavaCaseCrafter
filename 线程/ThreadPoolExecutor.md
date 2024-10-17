# 线程池用法与解释

## 1.概述
> 线程池是一种为了提高性能和资源利用率而设计的多线程管理机制。
> 通过线程池，应用程序可以更高效地管理线程的创建、调度和销毁，
> 避免频繁创建和销毁线程带来的开销。
> <br/>
> Java 提供了 ThreadPoolExecutor 类来实现线程池。它通过重用固定数量的线程来执行大量的任务。

## 2.参数说明
1. corePoolSize（核心线程数）：
> 描述：线程池中始终保持活动的线程数，即使它们处于空闲状态。
> <br/>
> 作用：确保线程池在空闲时也能快速响应新任务。
> <br/>
> 建议：根据系统资源和任务特性进行设置。

2. maximumPoolSize（最大线程数）：
>描述：线程池中允许的最大线程数。
> <br/>
>作用：限制线程池可以创建的最大线程数量。
> <br/>
> 建议：通常设置为 corePoolSize 的数倍，具体值取决于任务的性质（CPU 密集型或 I/O 密集型）。

3. keepAliveTime（线程存活时间）：
> 描述：当线程数超过 corePoolSize 时，多余的空闲线程在终止前等待新任务的最长时间。
> <br/>
> 作用：减少资源占用，当线程空闲时间超过此值时，线程会被终止。
> <br/>
> 建议：根据任务的频繁程度和响应时间要求进行调整。

4. unit（时间单位）：
> 描述：指定 keepAliveTime 参数的时间单位。
> <br/>
> 常用值：TimeUnit.SECONDS、TimeUnit.MILLISECONDS 等。

5.workQueue（任务队列）：

> 描述：用于存储等待执行任务的队列。
> <br/>
> 作用：缓冲提交的任务以便线程有空闲时执行。
> <br/>
> 常用实现：
> <br/>
> LinkedBlockingQueue（无界队列）：适用于任务提交速率低于处理速率的场景。
> <br/>
> ArrayBlockingQueue（有界队列）：适用于需要限制内存使用的场景。

6. threadFactory（线程工厂）：

> 描述：用于创建新线程的工厂。
> <br/>
> 作用：为线程池提供自定义的线程创建逻辑，如设置线程名称、优先级等。
> <br/>
> 建议：使用 Executors.defaultThreadFactory() 或实现自定义 ThreadFactory。

7. handler（拒绝策略）：

> 描述：当线程池和队列都满时，新任务的处理策略。
> <br/>
> 常用策略：
> <br/>
> AbortPolicy：抛出 RejectedExecutionException。
> <br/>
CallerRunsPolicy：由调用线程执行任务。
> <br/>
> DiscardPolicy：直接丢弃任务，不抛出异常。
> <br/>
> DiscardOldestPolicy：丢弃队列中最旧的任务，然后重试提交。

## 3.线程池执行execute和submit区别
* execute 方法
> 异常处理：
通过 execute 提交的任务如果抛出运行时异常，异常不会被捕获，直接传播到 ThreadPoolExecutor 的 afterExecute 方法或 Thread 的 UncaughtExceptionHandler。
线程池中的线程不会因为任务异常而被中断或停止。

> 适用场景： 适用于不需要返回值的任务，或不关心任务结果的场景。

* submit 方法
> 异常处理：
通过 submit 提交的任务如果抛出异常，异常会被捕获并封装在返回的 Future 对象中。
调用者可以通过 Future.get() 方法获取任务结果，并在此过程中捕获异常。

> 适用场景： 适用于需要获取任务结果或处理任务异常的场景。

## 4.线程池使用注意事项
1. **避免资源泄漏**：确保任务中使用的资源（如数据库连接、文件流）在任务完成后被正确关闭。
2. **处理异常**：使用 submit 方法提交任务，并通过 Future.get() 处理异常，避免线程池中的线程因未捕获异常而终止。
3. **监控线程池**：定期监控线程池的状态（如线程数、任务队列长度），以便及时调整配置。
4. **优雅关闭线程池**：使用 shutdown() 或 shutdownNow() 方法关闭线程池，确保所有任务执行完毕或被中断。

## 5.运行过程解析
假设我们配置一个 ThreadPoolExecutor，其参数如下：

* 核心线程数 (corePoolSize): 2
* 最大线程数 (maximumPoolSize): 5
* 任务队列: ArrayBlockingQueue，容量为 3
* 线程空闲时间 (keepAliveTime): 60 秒

**_现在，我们依次提交 8 个任务到这个线程池，看看线程池是如何处理这些任务的：_**

1. **提交第 1 个任务：**

   * 当前线程数为 0，小于 corePoolSize。
   * 创建一个新线程来处理任务 1。
   * 线程数变为 1。

2. **提交第 2 个任务：**

    * 当前线程数为 1，小于 corePoolSize。
    * 创建一个新线程来处理任务 2。
    * 线程数变为 2。

3. **提交第 3 个任务：**

    * 当前线程数为 2，等于 corePoolSize。
    * 任务 3 被放入任务队列。
    * 队列中任务数为 1。

4. **提交第 4 个任务：**

    * 当前线程数为 2，等于 corePoolSize。
    * 任务 4 被放入任务队列。
    * 队列中任务数为 2。

5. **提交第 5 个任务：**

    * 当前线程数为 2，等于 corePoolSize。
    * 任务 5 被放入任务队列。
    * 队列中任务数为 3（队列已满）。

6. **提交第 6 个任务：**

    * 当前线程数为 2，等于 corePoolSize。
    * 任务队列已满。
    * 当前线程数小于 maximumPoolSize。
    * 创建一个新线程来处理任务 6。
    * 线程数变为 3。

7. **提交第 7 个任务：**

    * 当前线程数为 3，小于 maximumPoolSize。
    * 任务队列已满。
    * 创建一个新线程来处理任务 7。
    * 线程数变为 4。

8. **提交第 8 个任务：**

    * 当前线程数为 4，小于 maximumPoolSize。
    * 任务队列已满。
    * 创建一个新线程来处理任务 8。
    * 线程数变为 5。

> 此时，线程池已经达到了 maximumPoolSize。如果再有新任务提交，
> 由于任务队列已满且线程数已达最大值，新的任务将会根据配置的拒绝策略被拒绝。

> 在任务完成之后，线程池会尝试复用现有的线程来处理队列中的任务。
> 如果某些线程空闲超过 keepAliveTime，并且它们超过了核心线程数，它们将被终止，以减少资源消耗。

### 线程 1 和 2：
* 线程 1 和 2 是最先创建的，它们立即开始处理任务 1 和任务 2。

### 任务 3、4 和 5：
* 任务 3、4 和 5 被放入任务队列中等待，因为此时核心线程（线程 1 和 2）已经在处理任务。
这些任务在队列中等待空闲线程来处理。

### 线程 6、7 和 8：
* 当任务 6、7 和 8 提交时，任务队列已满，线程池创建新的线程（线程 3、4 和 5）来处理这些任务。
* 这些线程（3、4 和 5）立即开始处理任务 6、7 和 8。
### 因此，在运行过程中：

* 线程 1 和 2 处理任务 1 和 2。
* 任务 3、4 和 5 在任务队列中等待。
* 线程 3、4 和 5 处理任务 6、7 和 8。
* 任务 3、4 和 5 将在线程 1 和 2 完成其当前任务后被处理，或者在线程 3、4 和 5 完成它们的任务后被处理，具体取决于哪个线程先空闲下来。

> 因此，任务 3、4 和 5 并不会一直等待，它们会在有线程空闲时被处理。
> 线程池会尽量复用空闲的线程来处理队列中的任务，以提高效率。

## 6.线程完成任务后处理
在 ThreadPoolExecutor 中，当线程完成任务后，它的流转过程如下：

1. 任务完成：
    > 当一个线程完成当前任务后，它会尝试从任务队列中获取下一个任务。

2. 处理队列中的任务：
    > 如果任务队列中有等待的任务，空闲线程将立即从队列中取出任务并开始处理。
这意味着任务 3、4 和 5 会在有线程空闲时被处理。

3. 线程复用：
    > 线程池会复用空闲线程来处理后续的任务，而不是立即销毁这些线程。这有助于减少线程创建和销毁的开销，提高性能。

4. 线程空闲和回收：
    > 如果一个线程完成任务后，任务队列中没有待处理的任务，该线程将进入空闲状态。
对于超过核心线程数的线程（非核心线程），如果它们在空闲状态下超过 keepAliveTime，这些线程将被终止。
核心线程默认情况下是不会被终止的，除非调用 allowCoreThreadTimeOut(true)。

5. 新任务到来：
    > 如果有新任务提交到线程池，空闲线程将被唤醒来处理这些任务。
如果没有空闲线程且线程数小于 corePoolSize，线程池将创建新的线程来处理任务。

6. 总结
    > 因此，当线程 1 和 2 完成任务 1 和 2 后，它们会从任务队列中取出任务 3 和 4 来处理。
类似地，线程 3、4 和 5 在完成任务 6、7 和 8 后，也会检查任务队列并处理剩余的任务。

这种机制确保了线程池能够高效地利用线程资源，及时处理任务队列中的任务，同时避免不必要的线程创建和销毁。

## 7.TestCaseCode
### 通过增加启动参数来设置堆内存大小
```bash
-Xms100m -Xmx100m 
```

```java
@Slf4j
public class TestThread {
    public ThreadPoolExecutor threadPool =
            // 配置一个 核心数1 最大2 队列1的线程池
            new ThreadPoolExecutor(
                    1, 2,
                    10000, TimeUnit.MILLISECONDS,
                    new ArrayBlockingQueue<Runnable>(1),
                    Executors.defaultThreadFactory(),
                    new ThreadPoolExecutor.CallerRunsPolicy()
            );

    public static void main(String[] args) throws InterruptedException {
        TestThread testThread = new TestThread();
        Future<?>[] futures = new Future[10];
        for (int i = 0; i < 10; i++) {
            log.info("execute:{}", i);
            ThreadPoolExecutor threadPool = testThread.threadPool;
            Future<?> future = threadPool.submit(new TestExecutor());
            futures[i] = future;
        }

        // 等待所有执行器执行完成
        Thread.sleep(10000);
        for (int i = 0; i < 10; i++) {
            log.info("get:{}", i);
            try {
                System.out.println(futures[i].get());
            } catch (ExecutionException e) {
                log.info("exception {}", e.getMessage());
            }
        }
        testThread.threadPool.shutdown();
    }
}

@Slf4j
public class TestExecutor implements Runnable  {
    private final ThreadLocal<byte[]> testData = new ThreadLocal<>();
    public void run() {
        try {
            byte[] data = testData.get();
            if (Objects.isNull(data)) {
                // 创建一个20M的数据
                testData.set(new byte[1024 * 1024 * 20]);
            }
            // 不马上执行完 假装有一个大任务在执行，占用线程
            try {
                Thread.sleep(10000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        } finally {
            //如果不主动remove，线程池中的线程被复用的情况会导致对象一直存在，无法被回收，这里会导致OOM
            testData.remove();
        }
    }
}
```