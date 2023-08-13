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
## 中断 interrupt isInterrupted interrupted(static)

* interrupt 中断自己这个线程(处于阻塞状态会抛错，不会将中断状态改为true，join,sleep,wait,其余只是将中断状态设置为true)
* isInterrupted 返回中断标志值
* interrupted(static) 返回当前线程中断状态，并进行线程

## 线程控制中断

* volatile关键字
* 原子型实现控制线程结束

```java
static volatile boolean isStop = false;
static AtomicBoolean atomicBoolean = new AtomicBoolean(false);
    public static void main(String[] args) {
        new Thread(()->{
           while (true){
               if(atomicBoolean.get()){
                   break;
               }
               try {
                   Thread.sleep(1000);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
               System.out.println("hello volatile");
           }
        }).start();
        new Thread(()->{
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            atomicBoolean.set(true);
        }).start();
    }
```

* api中断

```java
    public static void main(String[] args) {
        Thread hello_volatile = new Thread(() -> {
            while (true) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    System.out.println("退出线程");
                    break;
                }
                System.out.println("hello volatile");
            }
        });
        hello_volatile.start();
        Thread thread = new Thread(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            //设置中断标志
            hello_volatile.interrupt();
        });
        thread.start();
    }
```

## LockSupport

* park 阻塞
* unpark(Thread) 唤醒 

```java
        Thread come_in = new Thread(() -> {
            for (int i = 0; i < 2; i++) {
                System.out.println("come in");
                //会阻塞
                LockSupport.park();
                System.out.println("被唤醒");
            }

        });
        come_in.start();
        Thread.sleep(2000);
        //发通行证*2最多只有一个
        LockSupport.unpark(come_in);
        LockSupport.unpark(come_in);
/*      come in
        被唤醒
        come in*/
    }
```

## JMM内存模型

* cpu速度和内存数据读写数据不一样(不同的线程，有各自的内存区，对这个内存的读写要返回给主内存，主内存反应给其他线程的内存区，这个时间点其他线程会出现数据读错)
* 用来解决这种问题的模型
* 围绕多线程的原子性，可见性，有序性展开的

![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202308052122605.png)

## 多线程先行发生原则(底层对happens-before，满足以下原则，知道就行)

* 次序规则(先声明后使用)
* 锁定规则(先抢锁后释放)
* volatile变量规则(先写变量后读变量)
* 传递规则(a先于b，b先于c，a先于c)
* 线程启动原则(先就绪，在运行)
* 线程中断(先设置中断，再检测中断)
* 线程终止(先判断线程是否存活，再执行线程操作)
* 对象终结原则(先初始化，再垃圾回收)


## volatile

* 被这个关键字修改的变量有以下特性
  * 可见性，对一个volatile的读，总是能看到(任意线程)对这个变脸最后的写入
  * 有序性，对volatile修饰的变量的读写操作前后加上各种特定的内存屏障来禁止指令重排序来保障有序性
  * 不保证原子性，对于任意单个volatile变量的读写具有原子性，但类似于volatile++这种复合操作不具有原子性

1. 当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值立即刷新回主内存中。
2. 当读一个volatile变量时，JMM会把该线程对应的本地内存设置为无效，直接从主内存中读取共享变量
3. 所以volatile的写内存语义是直接刷新到主内存中，读的内存语义是直接从主内存中读取


## volatile变量存在内存屏障

* 作用
  * 写操作的可见性： 当一个线程写入一个 volatile 变量时，编译器和处理器会确保该写操作在之后的读操作之前被执行。这意味着其他线程在读取该 volatile 变量时，将会看到最新的写入值，而不是缓存中的旧值。
  * 读操作的有序性： 当一个线程读取一个 volatile 变量时，编译器和处理器会确保该读操作不会被重排到其他操作之前。这确保了在读取 volatile 变量时，它的值是最新的，而不是之前操作的结果。
* 功能：内存屏障对volatile变量的写在屏障前执行，读在后执行，确保先写后读,读取数据可以读取最新的消息

## 读写屏障插入策略

![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202308061014515.png)

![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202308061016161.png)

![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202308061017680.png)


## 案例

* 编译器和处理器在重排序时，会遵守数据依赖性，不会改变存在依赖关系的两个操作的执行,但不同处理器和不同线程之间的数据性不会被编译器和处理器考虑，其只会作用于单处理器和单线程环境，下面三种情况，只要重排序两个操作的执行顺序，程序的执行结果就会被改变

| 名称   | 代码示例 | 说明                         |
| ------ | -------- | ---------------------------- |
| 写后读 | a=1;b=a; | 写一个变量之后，再读这个位置 |
| 写后写 | a=1;a-2; | 写一个变量之后，再写这个变量 |
| 读后写 | a=b;b=1; | 读一个变量之后，再写这个变量 |

## 最佳实践

* 单一赋值，but包含复合运算赋值不可以

```java
volatile int a =10;
//修改值
a=11；
//复合修改不行
a++;
```

* 状态标志，判断业务是否结束

```java
volatile boolean flag =false
```

* 开销较低的读，写锁策略

```java
    static class num{
        private volatile int num=0;

        public int getNum() {
            return num;
        }

        public synchronized  void setNum() {
            this.num++;
        }
    }
```

* DCL双端锁的发布(volatile保证先new出来实例，再指向，禁止重排序)

```java
class singleTom {
    private volatile static singleTom instance;

    private singleTom() {
    }

    public static singleTom getInstance() {
        if (instance == null) {
            synchronized (singleTom.class) {
                if (instance == null) {
                    instance = new singleTom();
                }
            }
        }
        return instance;
    }
}
```

## 小结

* volatile写之前的操作，都禁止重排序到 volatile 之后
* volatile读之后的操作，都禁止重排序到 volatile 之前
* volatile写之后 volatile 读，禁止重排序

## 原子类

* getAndIncrement在自增失败会重试

```java
  public static void main(String[] args) throws InterruptedException {
        Num num = new Num();


        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                for (int j = 0; j < 1000; j++) {
                    num.setNum();
                    num.setNum0();
                }
            }).start();
        }
        Thread.sleep(1000);
        System.out.println(num.getNum());
        System.out.println(num.getNum0());
    }

class Num {
    private final AtomicInteger atomicInteger = new AtomicInteger(0);
    private volatile int num = 0;

    public int getNum0() {
        return num;
    }
    public int getNum() {
        return atomicInteger.get();
    }
    public void setNum0() {
        num++;
    }
    public void setNum() {
        atomicInteger.getAndIncrement();
    }
}
//打印
//10000
//9465
```

## 原子引用

```java
    public static void main(String[] args) {
        AtomicReference<User> ref = new AtomicReference<>();
        User ledger = new User("ledger", 20);
        ref.set(ledger);
        //true
        System.out.println(ref.compareAndSet(ledger, new User("ledger2", 20)));
        //false
        System.out.println(ref.compareAndSet(ledger, new User("ledger2", 20)));
    }
class User{
    String name;
    int age;
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
    public User() {
    }
}
```

* 使用原子引用实现乐观自旋锁

```java
    public static void main(String[] args) {
        SpinLockDemo spinLockDemo = new SpinLockDemo();
        new Thread(() -> {
            try {
                spinLockDemo.lock();
                Thread.sleep(1000);
                spinLockDemo.unlock();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "a").start();

        new Thread(() -> {
            try {
                spinLockDemo.lock();
                Thread.sleep(1000);
                spinLockDemo.unlock();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "b").start();
    }

class SpinLockDemo {
    //自旋锁
    private AtomicReference<Thread> atomicReference = new AtomicReference<>();

    public void lock() throws InterruptedException {
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName()+"进入线程");
        while (!atomicReference.compareAndSet(null, thread)) {
            System.out.println("阻塞");
            Thread.sleep(500);
        }
    }
    public void unlock() {
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName()+"退出线程");
        atomicReference.compareAndSet(thread, null);
    }
}

```

## CAS存在的问题

* ABA问题，a线程将原子类改成B，b线程拿到觉得是B，拿走之后，a将B又改为A;
* 循环开销很大(cpu损耗)

## ABA问题的解决
* 使用版本号控制
```java
        Book book = new Book();
        book.setAge(10);
        book.setName("ledger");
        AtomicStampedReference<Book> reference = new AtomicStampedReference<>(book, 1);

        System.out.println(reference.getReference());//Book{name = ledger, age = 10}
        System.out.println(reference.getStamp());//1
        Book mysql = new Book("mysql", 20);
        boolean b = reference.compareAndSet(book, mysql, 1, reference.getStamp() + 1);
        System.out.println(b);//true
        System.out.println(reference.getReference());//Book{name = mysql, age = 20}
```
## 原子类详解
* 基本 new AtomicInteger();
* 数组 new AtomicIntegerArray(10);
* 引用
* 对象属性修改
* 原子操作增强类

* 常用api
```java
    static final int SIZE = 50;

    public static void main(String[] args) throws InterruptedException {
        //它允许一个或多个线程等待其他线程完成一组操作，然后再继续执行
        CountDownLatch countDownLatch = new CountDownLatch(SIZE);
        for (int i = 0; i < SIZE; i++) {
            new Thread(()->{
                try {
                    for (int j = 0; j < 1000; j++) {
                        myNumber.addPlusPlus();
                    }
                } finally {
                    countDownLatch.countDown();
                }
            }).start();
        }
        //等待线程都执行完成
        countDownLatch.await();
        AtomicInteger atomicInteger = myNumber.atomicInteger;
        System.out.println(atomicInteger.get());
    }
    static class myNumber{
        static AtomicInteger atomicInteger = new AtomicInteger();

        public static void addPlusPlus(){
            atomicInteger.incrementAndGet();
        }
}
```
* 原子引用
    * AtomicMarkableReference(可以有一个标记，来用于标记有没有修改过)
```java
static AtomicMarkableReference<Integer> atomicMarkableReference = new AtomicMarkableReference<>(100, true);
    public static void main(String[] args) {
         new Thread(()->{
             boolean marked = atomicMarkableReference.isMarked();
             try {
                 Thread.sleep(100);
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }
             System.out.println("marked"+marked);
             boolean b = atomicMarkableReference.compareAndSet(100, 1000, marked, !marked);
             System.out.println("b1"+b);
         }).start();
        new Thread(()->{
            boolean marked = atomicMarkableReference.isMarked();
            try {
                Thread.sleep(600);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("marked22"+marked);
            boolean b = atomicMarkableReference.compareAndSet(100, 2000, marked, !marked);
            System.out.println("b2"+b);
        }).start();

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(atomicMarkableReference.getReference());
        System.out.println(atomicMarkableReference.isMarked());
    }
    // markedtrue
    // b1true
    // marked22true
    // b2false
    // 1000
    // false
```
* 原子字段更新(使用反射对指定类的指定volatile字段进行更新)
    * AtomicIntegerFieldUpdater
    * AtomicLongFieldUpdater
    * AtomicReferenceFieldUpdater




* AtomicIntegerFieldUpdater案例(给int类型字段封装成原子类)
```java
        BankAccount bankAccount = new BankAccount();
        CountDownLatch countDownLatch=new CountDownLatch(10);
        for (int i = 0; i < countDownLatch.getCount(); i++) {
            new Thread(()->{
                try {
                    for (int j = 0; j < 1000; j++) {
                        bankAccount.transfer(bankAccount);
                    }
                } finally {
                    countDownLatch.countDown();
                }
            }).start();
        }
        countDownLatch.await();
        System.out.println(bankAccount.money);

    static class BankAccount{
        String bankName="CCB";
        String bankNo="45454156154145";
        String owner="z3";
        //必须使用public和volatile
        public volatile int  money=0;
        //微创手术，就对这个字段原子操作
        static final AtomicIntegerFieldUpdater<BankAccount> fieldUpdater=
            AtomicIntegerFieldUpdater.newUpdater(BankAccount.class,"money");
        //不加synchronized
        public void transfer(BankAccount bankAccount){
            fieldUpdater.getAndIncrement(bankAccount);
        }
    }
```
## 性能更好的LongAdder和LongAccumulator,类似AtomicLong
* 减少了乐观锁的重试次数
```java
    //初始是0,api方法和AtomicLong类似
    LongAdder longAdder=new LongAdder();
    LongAccumulator longAccumulator=new LongAccumulator((x,y)->x+y,10);
    //当做y和x来做计算
    longAccumulator.accumulate(10);
    //获取并重置
    System.out.println(longAccumulator.getThenReset());
    //重置
    longAccumulator.reset();
    //获取
    System.out.println(longAccumulator.get());
```
## LongAdder为啥快这么多
* 源码分析
    * add 方法

* AtomicReferenceFieldUpdater案例(给任何类型(使用泛型)字段封装成原子类)

```java
    MyVar myVar = new MyVar();
    for (int i = 0; i < 10; i++) {
        new Thread(()->{
            myVar.init(myVar);
        }).start();
    }
    Thread.sleep(1000);

class MyVar {
    public volatile Boolean isInit = Boolean.FALSE;

    static final AtomicReferenceFieldUpdater<MyVar, Boolean> referenceFieldUpdater =
            AtomicReferenceFieldUpdater.newUpdater(MyVar.class, Boolean.class, "isInit");

    public void init(MyVar myVar) {
        if (referenceFieldUpdater.compareAndSet(myVar, Boolean.FALSE, true)) {
            System.out.println("init");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        } else {
            System.out.println("已经有线程进入了");
        }
    }
}
```
## LongAdder(低争用和AtomicLong类似，但是高争用和AtomicLong有高吞吐量但是空间消耗高)
* api
```java
        LongAdder longAdder = new LongAdder();
        //加14
        longAdder.add(114);
        //置空
        longAdder.reset();
        //加1
        longAdder.increment();
        //拿取值之后清零
        longAdder.sumThenReset();
        //拿取值
        System.out.println(longAdder.sum());
```
## LongAccumulator(自定义计算规则)
```java
        //自定义运算规则
        LongAccumulator longAccumulator =
                new LongAccumulator((x,y)-> x-y,0);

        //计算
        longAccumulator.accumulate(10);//-10
        longAccumulator.accumulate(10);//-20
        System.out.println(longAccumulator.get());
```
## LongAdder源码分析
* Striped64是LongAdder父类

![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202308092005502.png)

* cell是Striped64中的一个静态内部类

![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202308092006680.png)

### 方法和属性

* ![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202308092009349.png)

## 解析

* 内部有一个base变量，一个Cell[]数组,所有求和
    * base变量：低并发，直接累加到该变量上
    * Cell[]数组：高并发，累加到各个进程自己的槽Cell[i]中

## 分析源码
* add
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202308092034420.png)

* cell是存放增加的数值,cells数组用来存放cell的
```java
    public void add(long x) {
        Cell[] cs; long b, v; int m; Cell c;
        //1. (第一次进入add)新开窗口为null ->  casBase方法直接调用compareAndSet变更原子类,操作成功返回true
        //2. (多次进入)新开窗口为null -> casBase方法直接调用compareAndSet变更原子类,操作失败返回false
            //2.1 uncontended
        if ((cs = cells) != null || !casBase(b = base, b + x)) {
            //2.1 无竞争的
            boolean uncontended = true;
            //2.1 cells是空true
            if (cs == null || (m = cs.length - 1) < 0 ||
                //getProbe这个是获取线程的hash值的
                (c = cs[getProbe() & m]) == null ||
                //uncontended这个是相对于base而言的(没有竞争的)
                !(uncontended = c.cas(v = c.value, v + x)))
                //初始化cells
                longAccumulate(x, null, uncontended);
        }
    }
```

* casBase方法来操作base值
```java
    //用来操作base数值
    final boolean casBase(long cmp, long val) {
        return BASE.compareAndSet(this, cmp, val);
    }
```
* Striped64是LongAdder的父类,里面的longAccumulate方法实现计算
```java
//初始化cells 扩容cells 在扩容过程中执行的操作(交给base处理)
final void longAccumulate(long x, LongBinaryOperator fn,
                              boolean wasUncontended) {
        //当前线程的hash值
        int h;
        //当前线程的hash值是0就强制初始化
        if ((h = getProbe()) == 0) {
            ThreadLocalRandom.current(); // force initialization
            h = getProbe();
            wasUncontended = true;
        }
        //最后一个插槽为空就位true
        boolean collide = false;                // True if last slot nonempty
        //无限循环
        done: for (;;) {
            //cs当前的cells c计算出来的cells上位置的cell n为cells的长度  v为x值
            Cell[] cs; Cell c; int n; long v;
//如果cells已经有了  长度大于0 就在cells处理逻辑(执行任务或者扩容)
            if ((cs = cells) != null && (n = cs.length) > 0) {
                //数组长度和线程hash做与运算 找出这个位置的cell 值为空(没有被赋值过)
                if ((c = cs[(n - 1) & h]) == null) {
                    //没有在初始化线程
                    if (cellsBusy == 0) {       // Try to attach new Cell
                        Cell r = new Cell(x);   // Optimistically create
                        //双重校验加抢锁
                        if (cellsBusy == 0 && casCellsBusy()) {
                            try {               // Recheck under lock
                                Cell[] rs; int m, j;
                                //双重校验 不为空校验和长度校验 长度和线程hash算出来的结果位子为空 
                                if ((rs = cells) != null &&
                                    (m = rs.length) > 0 &&
                                    rs[j = (m - 1) & h] == null) {
                                    //将新建的,里面存放x的cell存cells
                                    rs[j] = r;
                                    break done;
                                }
                            } finally {
                                //释放锁
                                cellsBusy = 0;
                            }
                            continue;           // Slot is now non-empty
                        }
                    }
                    collide = false;
                }
                //要存入的cell不为null或者是锁没抢到 
                else if (!wasUncontended)       // CAS already known to fail
                    //竞争不激烈(根据add里面的一次传入c.cas(v = c.value, v + x)修改的结果决定,传进来的都是false)
                    //无竞争的改为true 然后下一次循环
                    wasUncontended = true;      // Continue after rehash
                // 数组在,位置不为null,竞争激烈  c原子存入x值
                else if (c.cas(v = c.value,
                               (fn == null) ? v + x : fn.applyAsLong(v, x)))
                    break;
                // cells的长度大于等于cpu核数或者cells在走判断语句的过程中被其他线程修改了,就直接不扩容了,collide标记为false,走下一次循环
                else if (n >= NCPU || cells != cs)
                    collide = false;            // At max size or stale
                //如果没有标记过(和上面那个打配合的,我愿称之为反复横跳)
                else if (!collide)
                    collide = true;
                //扩容操作 cellsBusy等于0 ,获取锁 casCellsBusy将cellsBusy修改为1,cellsBusy被 volatile 修饰,具有可见性
                else if (cellsBusy == 0 && casCellsBusy()) {
                    try {
                        //双重校验
                        if (cells == cs)        // Expand table unless stale
                            //拷贝数组并扩容一倍
                            cells = Arrays.copyOf(cs, n << 1);
                    } finally {
                        //释放锁
                        cellsBusy = 0;
                    }
                    //
                    collide = false;
                    continue;                   // Retry with expanded table
                }
                //没抢到锁的话,就要重新计算hash值进入下一次的循环了
                h = advanceProbe(h);
            }
//初始化cells casCellsBusy将cellsBusy修改成1,相当于是加锁了
            else if (cellsBusy == 0 && cells == cs && casCellsBusy()) {
                try {                           // Initialize table
                    //双重校验,判断cells是不是已经被修改了
                    if (cells == cs) {
                        //初始化长度为2的数组
                        Cell[] rs = new Cell[2];
                        //根据线程的hash来算应该放在什么地方
                        rs[h & 1] = new Cell(x);
                        //将rs赋值给cells
                        cells = rs;
                        //跳出循环
                        break done;
                    }
                } finally {
                    //释放锁
                    cellsBusy = 0;
                }
            }
// 如果数组为空,而且创建数组抢不到锁的话,走base的修改,修改成功就跳出循环,修改失败就下一轮循环
            else if (casBase(v = base,
                             (fn == null) ? v + x : fn.applyAsLong(v, x)))
                break done;
        }
    }
```
* 将所有结果累加
```java
    public long sum() {
        Cell[] cs = cells;
        long sum = base;
        if (cs != null) {
            for (Cell c : cs)
                if (c != null)
                    sum += c.value;
        }
        return sum;
    }
```
* 总结
    * 在base忙不过来的时候,会选择使用cells[]来帮助统计,cell里面存放的值就就是要统计的值
    * 在add方法中,总是优先走的使用base来统计数据,如果没成功,就使用longAccumulate
    * longAccumulate方法中,有三个主要的if循环分支,分别是扩容cells,初始化cells,存base,cells存在就不回走后两个了
    * 扩容cells中有6个if分支,主要的就是创建cell,赋值,增加cell的值,比较cpu个数和cells长度,一样或者超过了就不扩容,扩容


## ThreadLocal

* 初始化
```java
ThreadLocal<Integer> integerThreadLocal = new ThreadLocal<>();//null
ThreadLocal<Integer> integerThreadLocal1 = ThreadLocal.withInitial(() -> 1);
```
* tip
    * 如果线程池一直不关，存在线程复用，但是线程一直有threadlocal，那么就会内存泄露

## 源码分析看这边
[具体看这个文章](https://ledgerhhh.art/index.php/archives/20/)

* 内存泄露
    * 强、软、弱、虚引用


类型   | 特点	| 用途和注意事项
--- | --- | ---
强引用 | 不会被垃圾回收器回收	| 普通对象引用，当没有引用时，垃圾回收器无法回收对象(需要手动gc)。可能导致内存泄漏。
软引用 | 在内存不足时会被垃圾回收器回收	| 适用于实现内存敏感的高速缓存。可通过软引用来保存可以重建的对象，但并不是绝对保证不被回收。
弱引用 | 在下一次垃圾回收时会被垃圾回收器回收	| 适用于实现内存敏感的缓存，但比软引用更容易被回收。防止内存泄漏。
虚引用 | 无法通过虚引用来获取对象的引用	| 通常与ReferenceQueue结合使用，用于跟踪对象被垃圾回收的时机，用于清理与对象相关的资源。不影响垃圾回收。

## 为什么ThreadLocalMap里面的entry使用弱引用(内存溢出的解答)

* Thread和Thread是两个不同的类
* 线程结束之后要清除ThreadLocalMap中的数据
* 键为弱引用，如果被垃圾回收之后，值就会被一个为null的键强引用，无法回收


```java
    static class Entry extends WeakReference<ThreadLocal<?>> {
        Object value;
        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
```
## 最佳实践

* 建议使用初始化

```java
    ThreadLocal<Integer> sThreadLocal = ThreadLocal.withInitial(()->1);
```
* 建议使用static来修饰ThreadLocal
* 用完之后一定要清除掉remove方法

## 内存布局之布局简介
```java
Object O=new Object();
//Object在方法区
//O在栈
//new Object()在堆
```
* 对象实例在堆内存中华的储存布局可以划分为三个部分，对象头，实例数据，和对齐填充

* 对象头
    * 对象标记 Mark Word (8B)
    * 类元信息 Class Pointer (8B)
    * 长度Length(数组特有) 

* 实例数据 instance data

* 对齐填充 padding

> 提问
1. hashcode()记录在对象什么地方
2. synchronized(o) 这个记录在对象什么地方
3. 手动收垃圾(如果躲过了gc镰刀15次就会放置到了养老区)

* 解答

![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202308112230519.png)

### Mark Word

* 哈希码
* GC标记
* GC次数
* 同步锁标记
* 偏向锁持有者

![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202308112243575.png)
* tip
    * 没有数据的话就只会有对象头(大小为16B)

### 虚拟机的机栈
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202308112248869.png)

* 看一眼
```java
public class app2 {
    public static void main(String[] args) {
        aaa aaa = new aaa();
        System.out.println(ClassLayout.parseInstance(aaa).toPrintable());
    }
}
class aaa{
    private String name="ledger";
    private int age;
}
```
* 打印结果
```html
com.ledger3.aaa object internals:
OFF  SZ               TYPE DESCRIPTION               VALUE
  0   8                    (object header: mark)     0x0000000000000001 (non-biasable; age: 0)
  8   4                    (object header: class)    0xf800c392
 12   4                int aaa.age                   0
 16   4   java.lang.String aaa.name                  (object)
 20   4                    (object alignment gap)    
Instance size: 24 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
```

## 压缩指针
* 将类型指针压缩到8B，是默认开启的

## 锁升级
* 无锁
* 偏向锁
* 轻量锁
* 重量锁

* 锁升级的功能主要依赖于MarkWord中锁的标志位和释放偏向锁标志位
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202308120943844.png)

* tip：没有调用hashcode方法，就看不见hash
```java
    public static void main(String[] args) {
        Object o = new Object();
        //o.hashCode();
        System.out.println(ClassLayout.parseInstance(o).toPrintable());
    }
```
```java
OFF  SZ   TYPE DESCRIPTION               VALUE
  0   8        (object header: mark)     0x0000000000000001 (non-biasable; age: 0)
  8   4        (object header: class)    0xf80001e5
 12   4        (object alignment gap)    
Instance size: 16 bytes
```
* 使用hash()方法
```java
    public static void main(String[] args) {
        Object o = new Object();
        o.hashCode();
        System.out.println(ClassLayout.parseInstance(o).toPrintable());
    }
```
```java
java.lang.Object object internals:
OFF  SZ   TYPE DESCRIPTION               VALUE
  0   8        (object header: mark)     0x0000001ef7fe8e01 (hash: 0x1ef7fe8e; age: 0)
  8   4        (object header: class)    0xf80001e5
 12   4        (object alignment gap)    
Instance size: 16 bytes
```
### 四种锁的结构示意图
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202308121426428.png)

### 偏向锁
* 当线程第一次竞争抢到锁之后，通过操作修改mark word线程偏向id，偏向模式。
* 如果不存在其他线程竞争，那么持有偏向锁的线程就永远不需要进行同步
* 可以避免用户态到内核态的切换

* tips：
    * 默认情况下synchronized是启用偏向锁的，同一个线程会多次抢到锁，而且很多次都偏向于这个线程(偏向一个线程多次)
    * 偏向锁在遇到其他线程竞争的时候才会释放锁，线程是不会主动释放偏向锁的
    * 启动偏向锁有延迟，是四秒钟
    * 偏向锁和hashcode的位置冲突
        * 偏向锁之前的(前四秒)直接转换为轻量级锁，
        * 偏向锁的过程中华，如果要计算hashcode的话，就要将偏向锁转换为重量 锁(轻量级和重量级的Lock Recode可以和hashcode共存)

### 轻量锁CAS
* 多个线程同时抢夺资源
* 撤销偏向锁
* 使用CAS；来替换MarkWord里面的线程id为新线程id
* 全局安全点来撤销全局偏向锁

#### 撤销偏向锁
* 偏向锁使用一种等到竞争出现才释放锁的机制，只有当其他线程竞争锁时，持有偏向锁的原来线程才会被撤销。撤销需要等待全局安全点(该时间点上没有字节码正在执行)，同时检查持有偏向锁的线程是否还在执行:
1. 第一个线程正在执行synchronized方法(处于同步块)，它还没有执行完，其它线程来抢夺，该偏向锁会被取消掉并出现锁升级.
此时轻量级锁由原持有偏向锁的线程持有，继续执行其同步代码，而正在竞争的线程会进入自旋等待获得该轻量级锁。
线的头失
2. 第一个线程执行完成synchronized方法(退出同步块)，则将对象头设置成无锁状态并撤销偏向锁，重新偏向。

* 轻量级锁的原理
    * JVM会为每个线程在当前线程的栈中创建用于存储锁记录的空间，官方称为Displaced Mark Word。若一个线程获得锁时发现是轻量级锁，会把锁的MarkWord复制到自己的Displaced Mak Word里面。然后线程尝试用CAS将锁的MakWord替换为指向锁记录的指针。如果成功，当前线程获得锁，如果失败，表示Mark word已经被替换成了其他线程的锁记录，说明在与其它线程竞争锁，当前线程就尝试使用自旋来获取锁。

* 锁升级
    * 在释放锁时，当前线程会使用CAS操作将Displaced Mark Word的内容复制回锁的Mak Word里面。如果没有发生竞争，那么这个复制的操作会成功。如果有其他线程因为自旋多次导致轻量级锁级成了重量级锁，那么CAS操作会失败，此时会释放锁并唤醒被阻塞的线程。


### 重量级锁


![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202308121540333.png)

锁 | 特点 
--- | ---
偏向锁 | 适用于单线程适用的情况，在不存在锁竞争的时候进入同步方法/代码块则使用偏向锁。
轻量级锁 | 适用于竞争较不激烈的情况(这和乐观锁的使用范围类似)，存在竞争时升级为轻量级锁，轻量级锁采用的是自旋锁，如果同步方法/代码块执行时间很短的话，采用轻量级锁虽然会占用cpu资源但是相对比使用重量级锁还是更高效。
重量级锁 | 适用于竞争激烈的情况，如果同步方法/代码块执行时间很长，那么使用轻量级锁自旋带来的性能消耗就比使用重量级锁更严重，这时候就需要升级为重量级锁。

## JIT编译器对锁的优化

JIT: just in time compiler 即时编译器
* 锁消除(锁的存在吗没有任何意义)
```java
class my1{
   public void m1(){
       Object o = new Object();
       
        synchronized (o){
            System.out.println("-----------");
        }
   }
}
```
* 锁粗化
```java
//两者一样，前者会被即时编译器编译成后者
static Object o=new Object();
    public static void main(String[] args) {
        new Thread(()->{
            synchronized (o){
                System.out.println("111");
            }
            synchronized (o){
                System.out.println("222");
            }
            synchronized (o){
                System.out.println("333");
            }
            synchronized (o){
                System.out.println("444");
            }
        }).start();

        new Thread(()->{
            synchronized (o){
                System.out.println("111");
                System.out.println("222");
                System.out.println("333");
                System.out.println("444");
            }
        }).start();
    }
```
## AQS
### AQS入门级别的
* (抽象的队列同步器)
* 继承树
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202308121642427.png)


* 工作流程
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202308121637026.png)

## AQS可以干啥

* 每一个线程对象那个被封装到node里面，放在一个双向队列里面，一个状态来标示锁的占用

* 状态位
* 队列同步器
    * head 头指针
    * tail 尾指针
    * Node (存放在双向队列)
        * waitStatus
        * prev 上一个
        * next 下一个
        * thread 线程

## AQS之state和CLH队列
* state：0(空闲) >=1(非空闲)
* CLH：双链表对列，通过自旋等待，state判断是否阻塞，尾部入队，头部出队

## Node节点介绍
```java

// 共享型的等待锁
static final Node SHARED = new Node();
// 独占型的等待锁
static final Node EXCLUSIVE = null;
// 线程获取锁的请求已经取消了
static final int CANCELLED =  1;
// 线程已经准备好了，就等待资源释放
static final int SIGNAL = -1;
// 节点在等待队列中，节点等待唤醒
static final int CONDITION = -2;
// (传播)全部通知，share情况下面可用
static final int PROPAGATE = -3;


// 等待状态(被初始化的时候默认是0)
volatile int waitStatus;
// 上一个节点
volatile Node prev;
// 下一个节点
volatile Node next;
// 当前线程
volatile Thread thread;
// 指向下一个处于CONDITION状态的节点
Node nextWaiter;
//返回前驱节点
predecessor()
```
## AQS源码分析(以ReentrantLock为例)
```java
    //实现了Lock
    public class ReentrantLock implements Lock, java.io.Serializable{}
    //静态内部类(继承了AbstractQueuedSynchronizer)
    abstract static class Sync extends AbstractQueuedSynchronizer {}
    //不传参数默认是非公平锁
    public ReentrantLock() {
        sync = new NonfairSync();
    }

    public void lock() {
        sync.lock();
    }
    //非公平锁的实现方法
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        final void lock() {     
            //尝试直接抢锁
            if (compareAndSetState(0, 1))
                //设置当前占用线程是自己
                setExclusiveOwnerThread(Thread.currentThread());
            else
                //尝试抢锁
                acquire(1);
        }
    }
    //公平锁的实现方法
    static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;
        final void lock() {
            //尝试抢锁
            acquire(1);
        }
    }

    //acquiref方法实现在父类AbstractQueuedSynchronizer中
    public final void acquire(int arg) {
    //tryAcquire尝试抢锁、没成功就会addWaiter,就会去排队
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
    }

    //非公平的抢
    final boolean nonfairTryAcquire(int acquires) {
    //
    final Thread current = Thread.currentThread();
    //获取状态
    int c = getState();
    //没被占用是吗？
    if (c == 0) {
        //占用
        if (compareAndSetState(0, acquires)) {
            //设置线程是自己
            setExclusiveOwnerThread(current);
            //返回占用成功的布尔
            return true;
        }
    }
    //当前线程是自己了已经是吗
    else if (current == getExclusiveOwnerThread()) {
        //这是可重入锁的抢锁次数
        int nextc = c + acquires;
        //释放锁过多，没锁释放报错
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        //返回占用成功的布尔
        return true;
    }
    return false;
}
    //公平的抢
     protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                //hasQueuedPredecessors()看看你前面是不是有人在排队
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }

```
* 排队的代码(AbstractQueuedSynchronizer中实现)
```java
    private Node addWaiter(Node mode) {
        //创建节点，设置模式
        Node node = new Node(Thread.currentThread(), mode);
        //前一个指针是尾指针
        Node pred = tail;
        //尾指针不为空
        if (pred != null) {
            //这个节点的前一个是尾指针
            node.prev = pred;
            // 原子更换尾指针
            if (compareAndSetTail(pred, node)) {
                // 设置尾指针的下一个是这个节点
                pred.next = node;
                //返回这个节点
                return node;
            }
        }
        // 如果尾指针等于空就要初始化队列，初始化之后就不会走这里了
        enq(node);
        return node;
    }
    //反复入队
    private Node enq(final Node node) {
        //就是头执政设置成空节点，后面添加东西
        for (;;) {
            Node t = tail;
            //尾指针为空
            if (t == null) { // Must initialize
                //设置一个空节点为头节点
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                //尾指针不为空
                //设置当节点的前一个节点是尾指针
                node.prev = t;
                //设置尾节点
                if (compareAndSetTail(t, node)) {
                    //成功就把尾节点的下一个设置成为
                    t.next = node;
                    return t;
                }
                //尾节点设置不成功，就循环，下一轮再设置自己的的前节点是尾节点
            }
        }
    }
    // 进入队列之后的方法
    final boolean acquireQueued(final Node node, int arg) {
    // 默认不中断
    boolean failed = true;
    try {
        // 
        boolean interrupted = false;
        for (;;) {
            // 获得前置节点
            final Node p = node.predecessor();
            //如果前一个节点是头节点(就抢锁)
            if (p == head && tryAcquire(arg)) {
                //当前节点抢到就设置头节点是当前节点
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            // shouldParkAfterFailedAcquire将前置节点改为-1
            if (shouldParkAfterFailedAcquire(p, node) &&
                //将当前节点阻塞
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        // 异常情况出队伍
        if (failed)
            cancelAcquire(node);
    }
}

private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        // node 初始化的时候就是0 
        int ws = pred.waitStatus;
        //SIGNAL=-1
        if (ws == Node.SIGNAL)
            return true;
        if (ws > 0) {
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            //前置节点的等待状态改为-1
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
// 当前线程被阻塞
private final boolean parkAndCheckInterrupt() {
    //阻塞当前节点
    LockSupport.park(this);
    
    return Thread.interrupted();
}
```
* unlock
```java
    public void unlock() {
        sync.release(1);
    }
    // AbstractQueuedSynchronizer里面的方法
    public final boolean release(int arg) {
    //释放成功
    if (tryRelease(arg)) {
        Node h = head;
        // 查验头结点状态
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}

    protected final boolean tryRelease(int releases) {
    //getState 获取占用状态 
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())、
        // 不允许不持有锁的对象释放锁
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        // 如果没人占用
        free = true;
        // 就设置线程是空
        setExclusiveOwnerThread(null);
    }
    // 设置状态c
    setState(c);
    return free;
}
    private void unparkSuccessor(Node node) {

           int ws = node.waitStatus;

           if (ws < 0)
               //更改头结点状态
               compareAndSetWaitStatus(node, ws, 0);    
           //获取下一个节点
           Node s = node.next;
           //
           if (s == null || s.waitStatus > 0) {
               s = null;
               for (Node t = tail; t != null && t != node; t = t.prev)
                   if (t.waitStatus <= 0)
                       s = t;
           }
           if (s != null)
               //唤醒下一个节点的
               LockSupport.unpark(s.thread);
       }
```
* 取消队列
```java
 private void cancelAcquire(Node node) {
        if (node == null)
            return;
        //清除线程
        node.thread = null;
        //获取前一个
        Node pred = node.prev;
        //一直获取前一个(获取不是要退出的，waitStatus>0表示是要退出的)
        while (pred.waitStatus > 0)
            node.prev = pred = pred.prev;
        //前一个的下一个
        Node predNext = pred.next;
        //设置当前的线程是取消状态
        node.waitStatus = Node.CANCELLED;
        
        if (node == tail && compareAndSetTail(node, pred)) {
            //如果是尾节点，就至二级将前置节点的下一个设置成null
            compareAndSetNext(pred, predNext, null);
        } else {
            //不是尾节点
            int ws;
            // 前置节点不为和头部节点
            if (pred != head &&
                ((ws = pred.waitStatus) == Node.SIGNAL ||
                 (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
                pred.thread != null) {
                //当前节点的下一个
                Node next = node.next;
                if (next != null && next.waitStatus <= 0)
                    //设置给前面节点的下一个
                    compareAndSetNext(pred, predNext, next);
            } else {
                unparkSuccessor(node);
            }
            node.next = node; // help GC
        }
    }
```











