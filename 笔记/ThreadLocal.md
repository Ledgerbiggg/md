## ThreadLocal
### 示例
* ThreadLocal是用来维护线程中唯一变量的一个对象

* 看以下一段程序
```java
    static ThreadLocal<String> threadLocal=new ThreadLocal<>();
    static ThreadLocal<String> threadLocal2=new ThreadLocal<>();

    public static void main(String[] args) {
        threadLocal.set("ledger");
        threadLocal2.set("ledger2");
        System.out.println(threadLocal.get());
        System.out.println(threadLocal2.get());
        for (int i = 0; i < 3; i++) {
            Thread thread = new Thread(() -> {
                System.out.println(Thread.currentThread().getName()+threadLocal.get());
                System.out.println(Thread.currentThread().getName()+threadLocal2.get());
            });
            thread.start();
        }
    }
```

* 打印结果为
```html
ledger
ledger2
Thread-0null
Thread-1null
Thread-0null
Thread-1null
Thread-2null
Thread-2null
```
* 只有在主线程中获取到这个变量,但是其他的线程中,无法获取这个变量

### 解释
* 翻进去源码可以看到
```java
threadLocal.set("ledger");
```
可以看到
```java
    public void set(T value) {
        //获取当前线程(谁调用这个set方法，就是谁的线程，main方法中调用，就是main的线程)
        Thread t = Thread.currentThread();
        //根据当前线程获取到一个ThreadLocalMap对象
        ThreadLocalMap map = getMap(t);
        //在这个ThreadLocalMap里面set一个传入的值
        if (map != null) {
            //这个hash表里面有就存(键的this指向的是静态常量threadLocal)
            map.set(this, value);
        } else {
            //hash表不存在就创一个
            createMap(t, value);
        }
    }
```
* getMap()根据传入的线程(案例里面是main线程)获取属性(threadLocals),这个对象是一个hash表结构，threadLocals是线程类的一个属性
```java
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
```
* createMap()创建threadLocals并且往里面存一个值(健是this，值是set里面传的参数)
```java
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
```
* new ThreadLocalMap构造(根据firstKey算在哈希表存放的位置(案例中是threadLocal和threadLocal2)，使用hash()和table的长度算出索引并存放)，ThreadLocalMap是一个哈希表的结构
```java
        ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }
```
* 这里的firstKey是main线程里面的静态变量threadLocal(以this为健，以“ledger为值”，但是firstValue是不一样的),如果第二次存的话(threadLocal.set("ledger0"))，就会覆盖原来的存入的"ledger"，一个threadlocal只能存一次(threadLocal里面有就不会存了，就会覆盖)
* 像是之后又new了一个threadlocal(后面将其称呼为this2)，因为this2在ThreadLocalMap中没有找到，就有以this2为健，ledger2为值存进去了，存的都是main线程的threadLocals，子线程因为没有调用set方法，所以不会在set里面调用getMap方法(从当前线程中获取threadLocals来存数据)
* get方法()获取线程中的ThreadLocalMap中的this对象(这个this是那个静态变量中的threadLocal1，和threadLocal2)
* 获取方法就很简单了，就是根据this为健，在threadLocals获取存在的键值(类名是ThreadLocalMap)，threadLocal获取以this为健的,也就是"ledger"，threadLocal2获取以this2为健的数据也就是"ledger2"
```java
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            //谁调用get，this就指向谁(threadLocal调用，this指向threadLocal，threadLocal2调用，this指向threadLocal2)
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```
### 总结
* 每个线程中都有一个属性(threadLocals)其本质是一个hash表，在存放threadlocal里面的变量的时候，根据set方法的使用线程，拿出该线程的threadLocals，使用this为键set和get值
* 这些值是存放在内存中的，如果线程结束，没有及时处理，就会在每个线程结束，在内存中存放一个变量，导致内存泄露












