# juc
## 为什么使用多线程

1. 摩尔定律失效
2. 系统的需要,异步回调的需求

## 进程,线程,管程

1. 进程是一个程序,里面可以包含多个线程,管程为锁,也就是控制线程并发的一种监视器

## 并发与并行

1. 并行是两个任务同时进行
2. 并发是两个任务轮流进行

## wait | sleep

* wait放开锁 Object里面的方法
* sleep不放开锁 Thread里面的方法

## synchronized 和 ReentrantLock 

* 相同点
    * 都是可重入锁
* 不同点
    * synchronized属于jvm层面的,lock是api层面的
    *  synchronized可以加载方法或者划定代码块,加载静态方法上,锁的目标是类,非静态方法锁的目标是this,也可以加代码块,代码块可以指定锁对象;ReentrantLock需要new出来,使用lock和unlock获取锁和释放锁.
    * synchronized非公平,ReentrantLock都可以
    * synchronized随机唤醒或者都唤醒,ReentrantLock可以精确唤醒.

## 多线程实现
* 继承Thread
* 实现Runnable接口

## callable接口
* call接口定义了一个带有返回值的任务

## Future接口
* Future接口代表一个异步计算的结果

## FutureTask任务
* 继承RunnableFuture接口,构造可以传入call接口的实现,可以实现异步,异步任务,异步返回结果

```java
FutureTask<Integer> integerFutureTask =
        new FutureTask<>(() -> {
            //执行异步任务
            Thread.sleep(1000);
            return 115;
        });
//开新线程
new Thread(integerFutureTask).start();
System.out.println(integerFutureTask.get());
System.out.println("被阻塞了");
```
* get会阻塞后面的任务执行

## 方法列举
* get() 获取异步任务值
* get(long,TimUnit) 给期限,超出就报错TimeoutException
* cancel() 取消任务执行(get会报错)
* idCanceled() 是否被取消
* isDone() 是否完成或者被取消
* run() 运行任务

## 线程池
* 分类
```java
        //固定大小的线程池
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        //缓存线程池
        ExecutorService executorService1 = Executors.newCachedThreadPool();
        //单个线程池
        ExecutorService executorService2 = Executors.newSingleThreadExecutor();
        //执行异步的线程池
        ExecutorService executorService3 = Executors.newWorkStealingPool();
        //固定大小的周期任务线程池
        ScheduledExecutorService scheduledExecutorService1 = Executors.newScheduledThreadPool(10);
        //单个周期任务线程池
        ScheduledExecutorService scheduledExecutorService = Executors.newSingleThreadScheduledExecutor();
```
* 生产中不建议使用这种方法创建线程池
* 自定义线程池
```java
//核心线程数
//最大线程数
//非核心线程空闲留置时间
//单位
//阻塞队列(可以传入排队线程的长度)
//线程标志
//拒绝的策略(线程池子不够了会执行的策略(默认报错后中断程序运行))
        ThreadPoolExecutor threadPoolExecutor =
                new ThreadPoolExecutor(
                        1,
                        2,
                        2,
                        TimeUnit.SECONDS,
                        new ArrayBlockingQueue<>(3),
                        (r)->{
                            Thread thread = new Thread(r);
                            thread.setName("ledger");
                            return new Thread(r);
                        },
                    new ThreadPoolExecutor.AbortPolicy()
                );
```
拒绝策略  |	描述
--- | ---
AbortPolicy (默认) | 直接抛出 RejectedExecutionException，表示拒绝新任务的提交。这是默认的拒绝策略。
CallerRunsPolicy | 使用提交任务的线程来执行该任务。如果线程池已经关闭，则丢弃该任务。
DiscardPolicy | 直接丢弃新任务，不抛出异常。如果线程池已经关闭，则丢弃该任务。
DiscardOldestPolicy	| 丢弃最早加入队列的未处理任务，然后尝试重新提交当前任务。如果线程池已经关闭，则丢弃该任务。

## 线程类方法
方法  |  描述
--- | ---
start()	| 启动线程，使线程进入可运行状态，并自动调用线程的run()方法。
run()	| 定义线程的执行逻辑。当线程被启动时，run()方法会被执行。
join()	| 等待该线程终止。调用join()方法的线程将被阻塞，直到被调用的线程执行完成。
isAlive()	| 判断线程是否还处于活动状态。如果线程已经启动且尚未终止，则返回true。
interrupt()	| 中断线程，给线程设置中断标志。通常用于终止线程执行，线程在阻塞状态下会抛出InterruptedException。
isInterrupted()	| 判断线程是否被中断。如果线程被中断，则返回true，否则返回false。不会清除中断标志。
currentThread()	| 静态方法，返回当前正在执行的线程。
sleep(long millis)	| 静态方法，使当前线程暂停指定的毫秒数。
yield()	| 静态方法，提示当前线程愿意让出CPU资源，使得其他线程有机会执行。
setName(String name)	| 设置线程的名称。
getName() | 获取线程的名称。

## 线程的生命周期
阶段 | 描述
--- | ---
新建（New）	| 线程对象被创建，但尚未启动执行。
可运行（Runnable）	| 线程对象调用了start()方法，进入可运行状态。此时线程可能会得到CPU的时间片，但并不代表一定会执行，取决于线程调度器的调度。
运行（Running）	| 线程获得CPU时间片，正在执行线程的run()方法。
阻塞（Blocked）	| 线程在某些条件下被暂停执行，进入阻塞状态。常见的阻塞原因包括等待I/O操作、等待获取锁、调用sleep()方法等。
终止（Terminated）	| 线程执行完run()方法或出现异常而终止。线程终止后，它的生命周期结束，不会再回到可运行或运行状态。

## 生产者和消费者
* 生产者线程用于生产数据
* 消费者线程用于消费数据

* 使用Sychronized实现(隐式锁)
* viod wait()
* void notify()
* void notifyAll()

* 使用wait来阻塞程序,使用notifyall来唤醒其他的wait程序
```java
   public static void main(String[] args) {
        myThread myThread = new myThread();
        new Thread(()->{
            try {
                for (int i = 0; i < 10; i++) {
                    myThread.decrement();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
        new Thread(()->{
            try {
                for (int i = 0; i < 10; i++) {
                    myThread.increment();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }
    static class myThread{
        private  int num=0;

        public  synchronized void increment() throws InterruptedException {
            if(num==0){
                System.out.println("加法现在num是"+(++num));
                notifyAll();
            }else {
                wait();
                increment();
            }
        }
        public  synchronized void decrement() throws InterruptedException {
            if(num!=0){
                System.out.println("减法现在num是"+(--num));
                notifyAll();
            }else {
                wait();
                decrement();
            }
        }
```
* 使用reentrantLock
```java
 public static void main(String[] args) {
        myThread myThread = new myThread();
        new Thread(() -> {
            try {
                for (int i = 0; i < 5; i++) {
                    myThread.decrement();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
        new Thread(() -> {
            try {
                for (int i = 0; i < 5; i++) {
                    myThread.increment();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }

    static class myThread {
        private int num = 0;
        private final ReentrantLock reentrantLock = new ReentrantLock();
        private final Condition condition = reentrantLock.newCondition();

        public void increment() throws InterruptedException {
            reentrantLock.lock();
            try {
                if (num == 0) {
                    System.out.println("加法现在num是" + (++num));
                } else {
                    condition.await();
                    increment();
                }
            } finally {
                condition.signalAll();
                reentrantLock.unlock();
            }
        }

        public void decrement() throws InterruptedException {
            reentrantLock.lock();
            try {
                if (num != 0) {
                    System.out.println("减法现在num是" + (--num));
                } else {
                    condition.await();
                    decrement();
                }
            } finally {
                condition.signalAll();
                reentrantLock.unlock();
            }
        }
    }
```
* tip: 
    * synchronized
        * 和notifyAll()和wait()配合使用,notifyAll()和wait()成对出现
    * reentrantLock
        * 获取Condition对象:reentrantLock.newCondition();
        * 和condition.await()和condition.signalAll()配合使用,两者成对出现

## 阻塞队列
* ArrayBlockingQueue 数组 有界
* LinkedBlockingQueue 链表 无界
* SynchronousQueue 单元素

## BlockingQueue和基本使用
方法	| 描述
--- | ---
put(E e) | 将指定的元素插入队列中，如果队列已满，则阻塞等待空闲位置。
offer(E e, long timeout, TimeUnit unit)	| 尝试在指定的时间内将元素插入队列中，如果队列已满，则等待指定的时间。如果在超时时间内仍未插入成功，则返回 false。
take()	| 阻塞式地获取队列头部的元素，如果队列为空，则等待新的元素到来。
poll(long timeout, TimeUnit unit)	| 尝试在指定的时间内获取并移除队列头部的元素。如果在超时时间内队列为空，则返回 null。
offer(E e)	| 尝试将元素插入队列中，如果队列已满，则返回 false。
poll()	| 获取并移除队列头部的元素，如果队列为空，则返回 null。
peek()	| 获取队列头部的元素，但不移除，如果队列为空，则返回 null。
remove(Object o)	| 从队列中移除指定元素。
remainingCapacity()| 	获取队列的剩余容量，即队列的最大容量减去当前元素的数量。
contains(Object o)	| 判断队列是否包含指定元素。
size()	| 获取队列当前元素的数量。
isEmpty()	| 判断队列是否为空，如果队列为空则返回 true。
isFull()	| 判断队列是否已满，如果队列已满则返回 true。
drainTo(Collection<? super E> c)	| 将队列中的所有元素转移至给定的集合，并从队列中移除这些元素。
drainTo(Collection<? super E> c, int maxElements)	| 将队列中的最多 maxElements 个元素转移至给定的集合，并从队列中移除这些元素。如果队列中的元素数量少于 maxElements，则只转移和移除队列中的所有元素。

## ThreadPoolExecutor的执行原理
* threadPoolExecutor的submit方法有三个重构
```JAVA
ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(2, 2, 2, TimeUnit.SECONDS, null);
//callable
threadPoolExecutor.submit(() -> {
    return "1";
});
//runable
threadPoolExecutor.submit(()->{
});
//runable,(object)result
threadPoolExecutor.submit(()->{
},10);

```
* 内部实现(定义在抽象类AbstractExecutorService中)
```java
     public Future<?> submit(Runnable task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<Void> ftask = newTaskFor(task, null);
        execute(ftask);
        return ftask;
    }
    public <T> Future<T> submit(Runnable task, T result) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<T> ftask = newTaskFor(task, result);
        execute(ftask);
        return ftask;
    }
    public <T> Future<T> submit(Callable<T> task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<T> ftask = newTaskFor(task);
        execute(ftask);
        return ftask;
    }
```
```java
    protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
        return new FutureTask<T>(runnable, value);
    }
    protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
        return new FutureTask<T>(callable);
    }
```
* submit()传入的不管是callable还是runnable都最后转换成为FutureTask
* 将FutureTask传入execute方法只
* 子类ThreadPoolExecutor对这个方法进行了实现
* 大概就是根据初始化线程池的时候,使用的参数来判断新任务的执行情况
```java
  public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        int c = ctl.get();
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
    }
```
* addWorker中是开启并执行新线程的方法,并return出来一个布尔值
## 锁的类型
* 乐观锁(版本号,CAS) 不加锁,效率高,准确性低
* 悲观锁(synchronized,ReentrantLock)
* 公平锁(每个线程抢夺锁的概率差不多)
* 非公平锁(每个线程抢夺锁的概率差异大)
* 可重入锁(又名递归锁),二次使用不死锁
* 自旋锁(轮训获取锁)

## cas算法
* 首先，它会读取内存中的某个值。
* 然后，将这个值与期望的值进行比较，判断是否相等。
* 如果相等，说明没有其他线程修改过这个值，它会将新值写入内存，完成交换操作。
* 如果不相等，说明有其他线程修改了这个值，CAS操作失败，需要重新尝试。

* 不加锁
```java
public class app8 {
    private Integer num=0;

    public void increment() {
        num++;
    }

    public int getValue() {
        return num;
    }

    public static void main(String[] args) {
        final int THREADS = 10;
        final int INCREMENTS_PER_THREAD = 1000;

        app8 demo = new app8();

        Thread[] threads = new Thread[THREADS];
        for (int i = 0; i < THREADS; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < INCREMENTS_PER_THREAD; j++) {
                    demo.increment();
                }
            });
            threads[i].start();
        }

        // 等待所有线程执行完毕
        for (int i = 0; i < THREADS; i++) {
            try {
                threads[i].join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("Final value: " + demo.getValue());
    }
}
```
* cas乐观锁
```java
package k_juc;

import java.util.concurrent.atomic.AtomicInteger;

public class app8 {
    private AtomicInteger counter = new AtomicInteger(0);
    public void increment() {
        int oldValue, newValue;
        do {
            // 获取当前计数器的值
            oldValue = counter.get();
            // 计算新的递增后的值
            newValue = oldValue + 1;
            // 使用CAS进行比较和交换
        } while (!counter.compareAndSet(oldValue, newValue));
    }

    public int getValue() {
        return counter.get();
    }

    public static void main(String[] args) {
        final int THREADS = 10;
        final int INCREMENTS_PER_THREAD = 1000;

        app8 demo = new app8();

        Thread[] threads = new Thread[THREADS];
        for (int i = 0; i < THREADS; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < INCREMENTS_PER_THREAD; j++) {
                    demo.increment();
                }
            });
            threads[i].start();
        }

        // 等待所有线程执行完毕
        for (int i = 0; i < THREADS; i++) {
            try {
                threads[i].join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        System.out.println("Final value: " + demo.getValue());
    }
}

```
## CompletableFuture的使用
* 
![img](https://img-blog.csdnimg.cn/20210312115240515.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70)

```java
package k_juc;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ThreadPoolExecutor;

public class app9 {
    public static void main(String[] args) {
        CompletableFuture<?> integerCompletableFuture = CompletableFuture.supplyAsync(() -> {
            //supplyAsync支持异步,而且有返回值
            return 1;
        }).thenApply(r -> {
            //thenApply接受值并返回值
            return "14";
        }).thenAccept(r -> {
            //thenAccept接受值并返回值
        }).thenRun(() -> {
            //thenRun接受值并返回值
        }).thenApply(r -> {
            //thenApply接受值并返回值
            return 1;
        }).whenComplete((r, e) -> {
            //thenApply接受值并返回值
        }).exceptionally(e->{
            //thenApply接受值并返回值
           return 1;
        });
        //runAsync支持异步,没有返回值
        CompletableFuture<?> integerCompletableFuture1 = CompletableFuture.runAsync(() -> {
        }).thenApply(r -> {
            return "14";
        }).thenAccept(r -> {

        }).thenRun(() -> {

        }).thenApply(r -> {
            return 1;
        }).whenComplete((r, e) -> {

        }).exceptionally(e->{
            return 1;
        });
        //获得结果和触发计算(get、getNow、join、complete)

    }
}

```






