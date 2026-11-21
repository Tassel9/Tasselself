# Java并发

[返回模块目录](../README.md) · [打开完整总文档](../Java后端八股.md)

## 高频题

### 并发编程的三大特性

并发编程的三大特性是：原子性、可见性、有序性。

##### 原子性

原子性指一个操作或者一段代码执行时不可被分割，要么全部执行成功，要么全部不执行。

例如 `i++`  看起来只有一行代码，但它不是原子操作。

它底层大致包含读取、加一、写回等多个步骤。多线程并发执行时，可能出现多个线程读到同一个旧值，最终导致结果错误。

解决原子性问题可以使用：

1. `synchronized` 。

2. `Lock` 。

3. 原子类，例如 `AtomicInteger` 。

##### 可见性

可见性指一个线程对共享变量的修改，其他线程能够及时看到。

如果线程修改共享变量后只更新到自己的工作内存，没有及时刷新到主内存，其他线程就可能一直读到旧值。

解决可见性问题可以使用：

1. `volatile` 。

2. `synchronized` 。

3. `Lock` 。

##### 有序性

有序性指程序执行顺序符合预期。

编译器和 CPU 可能会为了优化性能进行指令重排序。

在单线程下，只要不影响最终结果，重排序通常没问题。

但是在多线程下，重排序可能导致线程安全问题。

解决有序性问题可以使用：

1. `volatile`  禁止特定指令重排序。

2. `synchronized`  或 `Lock`  通过加锁保证临界区执行顺序。

### Java内存模型JMM是什么？

JMM 全称是 Java Memory Model，也就是 Java 内存模型。

JMM 是一套规范，主要解决多线程读写共享变量时的原子性、可见性、有序性问题。

JMM 中有主内存和工作内存的概念：

1. 主内存： 线程共享的内存区域，保存共享变量。

2. 工作内存： 每个线程私有的内存区域，保存该线程使用到的共享变量副本。

线程操作共享变量时，不是直接操作主内存，而是先把变量从主内存拷贝到自己的工作内存中，修改后再刷新回主内存。

所以多线程之间就可能出现可见性问题。

JMM 对上层提供了 `volatile` 、`synchronized` 、`final`  等关键字来保证并发安全。

### volatile关键字的作用和原理

`volatile`  主要有两个作用：

1. 保证可见性。

2. 禁止指令重排序。

`volatile`  不能保证复合操作的原子性。

例如：

```java
public static volatile int i = 0;

public static void increase() {
    i++;
}
```

这里的 `i++`  仍然不是线程安全的，因为它包含读取、计算、写回多个步骤。

`volatile`  的原理可以理解为内存屏障。

被 `volatile`  修饰的变量进行写操作时，会强制把修改刷新到主内存，并让其他 CPU 中对应缓存行失效。

其他线程再读取这个变量时，就需要重新从主内存读取最新值。

同时，`volatile`  会通过内存屏障禁止指令重排序。

### volatile适合什么场景？

`volatile`  适合一写多读，或者状态标记这类场景。

例如：

```java
private volatile boolean running = true;
```

一个线程修改 `running = false` ，其他线程能及时看到这个变化。

但是如果是 `count++`  这种复合操作，不能只依赖 `volatile` 。

这种场景应该使用 `AtomicInteger` 、`synchronized`  或 `Lock` 。

### synchronized的底层原理

`synchronized`  是 Java 中的同步关键字，主要用来保证同一时刻只有一个线程执行同步代码块或同步方法。

`synchronized`  可以保证原子性、可见性和有序性。

底层原理：

1. `synchronized`  修饰代码块时，字节码层面会生成 `monitorenter`  和 `monitorexit`  指令。

2. `monitorenter`  表示进入同步代码块，尝试获取对象关联的 Monitor 锁。

3. `monitorexit`  表示退出同步代码块，释放 Monitor 锁。

4. 每个 Java 对象都可以作为锁对象，因为每个对象都可以关联一个 Monitor。

5. Monitor 中有 Owner、EntryList、WaitSet 等结构，用来管理持锁线程、阻塞线程和等待线程。

如果 `synchronized`  修饰的是方法，则不会直接生成 `monitorenter`  和 `monitorexit` ，而是通过方法的 `ACC_SYNCHRONIZED`  标识来实现同步。

### synchronize的上锁粒度（对象/类），如果上在方法上呢？

`synchronized`  的锁粒度主要有两种：对象锁和类锁。

1. 修饰普通代码块：

```java
synchronized (this) {
}
```

锁的是当前实例对象。

2. 修饰普通方法：

```java
public synchronized void method() {
}
```

锁的也是当前实例对象，相当于 `synchronized(this)` 。

3. 修饰静态方法：

```java
public synchronized static void method() {
}
```

锁的是当前类的 Class 对象。

4. 修饰类：

```java
synchronized (Demo.class) {
}
```

锁的是这个类对应的 Class 对象。

对象锁只影响同一个对象实例，类锁会影响这个类的所有对象。

### Synchronized的锁升级过程

`synchronized`  锁升级过程大致是：

无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁。

##### 偏向锁

偏向锁主要优化同一个线程多次获取同一把锁的场景。

第一次获取锁时，JVM 会在对象头 Mark Word 中记录线程 ID。

之后同一个线程再次进入同步块时，只需要判断线程 ID 是否是自己，如果是，就直接进入，不需要CAS。

##### 轻量级锁

当其他线程也来竞争这把锁时，偏向锁会被撤销，升级为轻量级锁。

轻量级锁主要通过 CAS 自旋来竞争锁。

如果竞争不激烈，线程自旋一会儿就能拿到锁，就避免了线程阻塞和唤醒的开销。

##### 重量级锁

如果竞争激烈，自旋一直失败，锁会膨胀为重量级锁。

重量级锁依赖操作系统互斥量，竞争失败的线程会进入阻塞状态，不再消耗 CPU。

总结：

锁升级的设计是为了适配不同竞争强度的场景。竞争很少时用偏向锁，竞争较轻时用轻量级锁，竞争激烈时升级为重量级锁。

### 什么是轻量级锁、重量级锁？

##### 轻量级锁

轻量级锁是 JDK1.6 后对 `synchronized`  的优化。

它主要通过 CAS 和自旋在用户态完成加锁和解锁，不需要立即进入操作系统内核态，所以开销较小。

适合锁竞争不激烈、同步代码块执行很快的场景。

##### 重量级锁

重量级锁依赖操作系统的互斥量 mutex。

竞争失败的线程会被阻塞挂起，后续需要操作系统唤醒。

这会涉及用户态和内核态切换，以及线程上下文切换，所以开销较大。

适合竞争激烈、临界区执行时间较长的场景。

### 乐观锁是怎样实现的？

乐观锁一般通过版本号机制或者 CAS 实现。

##### 版本号机制

在数据表中增加一个版本号字段 `version` 。

更新数据前先读取当前版本号，更新时判断版本号是否还是之前读取的值。

如果一致，说明期间没有其他线程修改，可以更新，并把版本号加一。

如果不一致，说明数据已经被别人修改，本次更新失败或重试。

SQL 大致是：

```sql
UPDATE table
SET value = new_value, version = version + 1
WHERE id = 1 AND version = current_version;
```

##### CAS

CAS 全称是 Compare And Swap，比较并交换。

CAS 会比较内存中的当前值和预期值是否相等。

如果相等，就把当前值更新为新值。

如果不相等，说明数据被其他线程修改过，当前线程更新失败，可以自旋重试。

### CAS是什么？

CAS 涉及三个操作数：

1. V：要更新的变量值。

2. E：预期值。

3. N：要写入的新值。

只有当 V 的值等于 E 时，才会把 V 更新为 N。

如果 V 不等于 E，说明这个变量已经被其他线程修改过，本次 CAS 失败。

CAS 是一个原子操作，底层依赖 CPU 原子指令。

在 Java 中，很多 CAS 操作是通过 `Unsafe`  类的 native 方法完成的。

### CAS会有什么问题？

CAS 常见问题：

1. ABA 问题： 一个变量从 A 变成 B，又变回 A。另一个线程 CAS 时发现还是 A，就误以为没有变化。

2. 自旋开销： 如果竞争激烈，CAS 一直失败，线程会不断自旋重试，浪费 CPU。

3. 只能保证单个变量原子操作： 普通 CAS 更适合单个共享变量，多个变量的一致性处理起来更复杂。

ABA 问题可以通过版本号或 `AtomicStampedReference`  解决。

### 乐观锁和悲观锁及其适用场景

##### 悲观锁

悲观锁总是假设共享资源每次访问都会有冲突。

所以在访问共享资源前先加锁，其他线程必须等待锁释放后才能访问。

Java 中 `synchronized` 、`ReentrantLock`  就是悲观锁思想的实现。

悲观锁适合写多读少、竞争激烈、数据一致性要求高的场景。

缺点是线程阻塞、上下文切换开销较大，也可能出现死锁。

##### 乐观锁

乐观锁假设冲突概率比较低，不会一开始就加锁。

它在提交更新时再检查数据是否被其他线程修改过。

Java 中的原子类，比如 `AtomicInteger` ，底层就大量使用 CAS。

乐观锁适合读多写少、冲突概率低的场景。

缺点是冲突频繁时会不断失败重试，导致 CPU 开销升高。

### cas的底层原理

CAS 并发原语在 Java 中主要通过 `Unsafe`  类的 native 方法实现。

调用 `Unsafe`  中的 CAS 方法时，JVM 会帮我们执行底层 CPU 的原子指令。

CAS 是一种系统原语，执行过程不能被中断，所以可以保证比较和交换这个过程的原子性。

JUC 包中的原子类、AQS、ConcurrentHashMap 等底层都大量使用 CAS。

### threadlocal 原理

`ThreadLocal`  是实现线程局部变量的类。

它会为每个使用该变量的线程创建独立副本，不同线程之间互不影响。

`ThreadLocal`  主要用于线程隔离，而不是线程共享。

`ThreadLocal`  和 `synchronized`  的区别：

1. `synchronized`  是通过加锁保证多个线程访问共享资源时的安全。

2. `ThreadLocal`  是给每个线程一份自己的变量副本，让线程之间不共享数据。

ThreadLocal 的实现：

1. 每个 `Thread`  对象中都有一个 `ThreadLocalMap` 。

2. `ThreadLocalMap`  中有一个 Entry 数组。

3. Entry 的 key 是 `ThreadLocal`  对象，value 是存入的数据。

4. `ThreadLocal.set()`  时，会获取当前线程的 `ThreadLocalMap` ，然后用当前`ThreadLocal`  作为 key，把 value 存进去。

5. `ThreadLocal.get()`  时，也是先获取当前线程的 `ThreadLocalMap` ，再通过当前`ThreadLocal`  取出 value。

### ThreadLocal内存泄漏

`ThreadLocalMap`  中 Entry 的 key 是 `ThreadLocal`  的弱引用，value 是强引用。

如果 `ThreadLocal`  没有外部强引用，GC 时 key 会被回收，变成 `null` 。

但是 value 仍然被 `ThreadLocalMap`  强引用着。

如果是普通线程，线程执行结束后，整个 `Thread`  和 `ThreadLocalMap`  都会被回收，问题不大。

但是在线程池中，核心线程会被反复复用，线程不会结束，`ThreadLocalMap`  也不会被回收。

这时 key 为 `null`  的 Entry 中，value 可能一直无法释放，造成内存泄漏。

解决方案：

使用完 `ThreadLocal`  后，在 `finally`  中调用 `remove()` 。

### AQS 原理、构建了哪些锁

AQS 全称是 AbstractQueuedSynchronizer。

它是阻塞式锁和同步器工具的基础框架。

AQS 主要由两部分组成：

1. `state`  状态变量。

2. FIFO 双向等待队列。

`state`  用来表示共享资源的状态。

例如：

1. 在 `ReentrantLock`  中，`state = 0`  表示没有加锁，`state > 0`  表示加锁次数，也表示可重入次数。

2. 在 `CountDownLatch`  中，`state`  表示剩余计数。

如果线程获取资源失败，就会被封装成 Node 节点，加入 FIFO 等待队列。

资源释放后，会唤醒队列中的后继节点。

基于 AQS 实现的组件有：

1. `ReentrantLock` 。

2. `CountDownLatch` 。

3. `Semaphore` 。

4. `ReentrantReadWriteLock` 。

### ReentrantLock是什么，和synchronized相比有什么区别？

`ReentrantLock`  是 JUC 包下的可重入锁，底层基于 AQS 实现。

它和 `synchronized`  都是可重入锁。

区别：

1. `synchronized`  是 Java 关键字，JVM 层面实现；`ReentrantLock`  是 JDK 提供的类，代码层面实现。

2. `synchronized`  自动加锁和释放锁；`ReentrantLock`  需要手动调用 `lock()`  和`unlock()` 。

3. `ReentrantLock`  支持公平锁和非公平锁，默认非公平锁；`synchronized`  一般是非公平锁。

4. `ReentrantLock`  支持可中断、可超时获取锁，使用更灵活。

5. 简单同步场景用 `synchronized`  更方便；复杂锁控制场景可以用 `ReentrantLock` 。

使用 `ReentrantLock`  时，必须在 `finally`  中释放锁：

```text
lock.lock();
try {
    // 业务代码
} finally {
    lock.unlock();
}
```

### 死锁的必要条件有哪些？

死锁产生的四个必要条件：

1. 互斥条件： 资源同一时间只能被一个线程占用。

2. 请求和保持条件： 线程已经持有一个资源，同时又请求新的资源。

3. 不可剥夺条件： 线程已经获得的资源，在使用完之前不能被强制剥夺。

4. 循环等待条件： 多个线程之间形成头尾相接的资源等待关系。

只要破坏其中任意一个条件，就可以避免死锁。

### 如何避免死锁？

避免死锁可以从破坏死锁四个必要条件入手。

常见方式：

1. 固定加锁顺序，避免循环等待。

2. 尽量缩小锁粒度和锁持有时间。

3. 避免在持有锁时继续申请其他锁。

4. 使用 `tryLock()`  设置超时时间，获取不到锁就释放已有资源并重试。

5. 使用并发工具类代替手写复杂锁逻辑。

线上排查死锁时，可以使用 `jstack`  查看线程堆栈。

### 线程池的好处？

线程池的主要作用是资源管理和线程复用。

好处：

1. 避免频繁创建和销毁线程带来的性能开销。

2. 提高响应速度，任务来了可以直接复用已有线程。

3. 统一管理线程数量，避免无限制创建线程导致资源耗尽。

4. 可以通过队列、拒绝策略等机制控制任务提交速度。

### 线程池的参数都有哪些？

`ThreadPoolExecutor`  的核心参数有 7 个：

1. corePoolSize： 核心线程数。

2. maximumPoolSize： 最大线程数。

3. keepAliveTime： 非核心线程空闲存活时间。

4. unit： 存活时间单位。

5. workQueue： 任务阻塞队列。

6. threadFactory： 线程工厂，用来创建线程，可以设置线程名、是否守护线程等。

7. handler： 拒绝策略。

常见工作队列：

1. `ArrayBlockingQueue` ：基于数组的有界阻塞队列。

2. `LinkedBlockingQueue` ：基于链表的阻塞队列，不指定容量时接近无界。

3. `SynchronousQueue` ：不存储任务，任务必须直接交给工作线程。

4. `PriorityBlockingQueue` ：优先级阻塞队列。

### 线程池提交任务的流程

线程池提交任务流程：

1. 提交任务后，先判断当前线程数是否小于核心线程数。

2. 如果小于核心线程数，就创建核心线程执行任务。

3. 如果核心线程数已满，就把任务放入阻塞队列。

4. 如果阻塞队列也满了，就创建非核心线程执行任务。

5. 如果当前线程数已经达到最大线程数，就触发拒绝策略。

所以，如果核心线程数已满，新任务不是直接创建新线程，而是先进入队列。

只有核心线程数满了并且队列也满了，才会创建非核心线程。

### 线程池拒绝策略有哪些？

JDK 提供了四种常见拒绝策略：

1. AbortPolicy： 默认策略，直接抛出 `RejectedExecutionException` 。

2. CallerRunsPolicy： 由提交任务的线程自己执行任务，可以降低任务提交速度。

3. DiscardPolicy： 直接丢弃任务，不抛异常。

4. DiscardOldestPolicy： 丢弃队列中最早的任务，然后尝试提交当前任务。

如果任务很重要，不能随便丢弃，一般不建议使用静默丢弃策略。

### 创建线程的几种方式

常见创建线程方式：

1. 继承 `Thread`  类。

2. 实现 `Runnable`  接口。

3. 实现 `Callable`  接口，结合 `FutureTask`  使用。

4. 使用 `ExecutorService`  线程池。

实际开发中一般不建议频繁手动创建线程，而是优先使用线程池统一管理线程资源。

### Runnable和Callable的区别

区别主要有三个：

1. 返回值不同： `Runnable`  的 `run()`  没有返回值；`Callable`  的 `call()`  可以有返回值。

2. 异常处理不同： `Runnable`  不能直接抛出受检异常；`Callable`  可以抛出受检异常。

3. 执行方式不同： `Runnable`  可以直接交给 `Thread`  执行，也可以提交线程池；`Callable`  通常提交给线程池，返回 `Future`  获取结果。

`Callable`  适合需要获取异步执行结果的任务。

### execute和submit的区别

`execute()`  和 `submit()`  都可以向线程池提交任务。

区别：

1. `execute()`  只能提交 `Runnable` ，没有返回值。

2. `submit()`  可以提交 `Runnable`  或 `Callable` ，会返回 `Future` 。

3. `execute()`  中任务异常通常会直接抛出到线程的异常处理器。

4. `submit()`  中任务异常会被封装在 `Future`  中，调用 `Future.get()`  时才会抛出。

如果需要拿到任务结果或异常，使用 `submit()` 。

如果只是执行一个无返回值任务，可以使用 `execute()` 。

### countdownlatch和cyclicbarrier的区别

##### CountDownLatch

`CountDownLatch`  是倒计时器。

它用一个计数器初始化。

调用 `await()`  的线程会等待，直到其他线程调用 `countDown()`  把计数器减到 0。

计数器减到 0 后，等待的线程继续执行。

`CountDownLatch`  是一次性的，不能重置。

使用场景：

主线程等待多个子任务执行完成后，再继续汇总结果。

##### CyclicBarrier

`CyclicBarrier`  是循环屏障。

它允许一组线程互相等待，直到所有线程都到达同一个屏障点，大家再一起继续执行。

`CyclicBarrier`  可以重复使用。

使用场景：

多个线程互相等待，比如多个运动员都准备好后，裁判才说开始。

##### 区别

1. `CountDownLatch`  是一个或多个线程等待其他线程完成。

2. `CyclicBarrier`  是多个线程互相等待。

3. `CountDownLatch`  不能重置，`CyclicBarrier`  可以重复使用。

### Java并发容器

并发容器重点回答 `ConcurrentHashMap`  和 `CopyOnWriteArrayList` 。

##### ConcurrentHashMap

`HashMap`  不是线程安全的。

如果简单使用 `Collections.synchronizedMap()` ，相当于使用全局锁，性能较差。

`ConcurrentHashMap`  是线程安全的 Map。

JDK1.7 中使用分段锁，底层是 `Segment`  数组 + `HashEntry`  数组 + 链表。

JDK1.8 中改为 `Node`  数组 + 链表 + 红黑树，并发控制主要使用 CAS + `synchronized` 。

##### CopyOnWriteArrayList

`CopyOnWriteArrayList`  是线程安全的 List。

它的核心是写时复制。

读操作不加锁，写操作加锁，写入时复制一个新数组，在新数组上修改，最后把引用指向新数组。

适合读多写少的场景。

缺点是写操作成本高、占用更多内存，并且读到的数据可能不是最新的。

### 线程的状态有哪些？

Java 线程状态有六种：

1. `NEW` ：新建状态，线程对象创建后还没有调用 `start()` 。

2. `RUNNABLE` ：可运行状态，调用 `start()`  后进入该状态。Java 中就绪和运行都属于`RUNNABLE` 。

3. `BLOCKED` ：阻塞状态，通常是等待获取 `synchronized`  锁。

4. `WAITING` ：无限等待状态，例如调用 `wait()` 、`join()` 。

5. `TIMED_WAITING` ：计时等待状态，例如调用 `sleep(time)` 、`wait(time)` 、`join(time)` 。

6. `TERMINATED` ：终止状态，线程执行完成。

## 扩展题

### MM如何保证原子性、可见性、有序性？

##### 原子性

原子性问题典型例子是 `i++` 。

`i++`  看起来是一行代码，但是字节码层面大致包含读取、加一、写回多个步骤。

多线程交叉执行时，可能导致最终结果小于预期。

解决方式：

1. 使用 `synchronized` 。

2. 使用 `Lock` 。

3. 使用原子类，例如 `AtomicInteger` 。

##### 可见性

可见性问题指一个线程修改共享变量后，其他线程不能及时看到。

原因是线程可能只修改了工作内存中的副本，没有及时刷新到主内存。

解决方式：

1. 使用 `volatile` 。

2. 使用 `synchronized` 。

3. 使用 `Lock` 。

`volatile`  写会把变量刷新到主内存，并让其他线程缓存失效。

##### 有序性

有序性问题指编译器或 CPU 可能对指令进行重排序。

单线程下重排序不影响最终结果，但多线程下可能出问题。

解决方式：

1. 使用 `volatile`  禁止特定重排序。

2. 使用 `synchronized`  或 `Lock`  保证临界区执行顺序。

### volatile的内存屏障

`volatile`  通过插入内存屏障来保证可见性和有序性。

可以简单理解为：

1. `volatile`  写之前，前面的普通写不能重排序到它后面。

2. `volatile`  写之后，后面的读写不能重排序到它前面。

3. `volatile`  读之后，后面的普通读写不能重排序到它前面。

常见屏障：

1. `StoreStore`  屏障。

2. `StoreLoad`  屏障。

3. `LoadLoad`  屏障。

4. `LoadStore`  屏障。

这部分一般属于扩展，不需要一上来讲太底层。面试中可以先说 `volatile`  保证可见性和禁止重排序，再补充内存屏障。

### MESI协议和volatile有什么关系？

MESI 是 CPU 缓存一致性协议。

它的核心思想是：

当某个 CPU 修改共享变量时，如果其他 CPU 缓存中也有这个变量的副本，会通知其他 CPU 把对应缓存行置为无效。

其他 CPU 再读取这个变量时，发现自己的缓存行无效，就需要重新从主内存读取。

`volatile`  写操作会通过 lock 前缀指令和缓存一致性机制，让其他线程看到最新值。

所以可以说，`volatile`  在底层依赖内存屏障和缓存一致性协议来实现可见性。

### happens-before是什么？

`happens-before`  是 JMM 中用来描述操作之间可见性关系的规则。

如果操作 A happens-before 操作 B，那么 A 的执行结果对 B 可见，并且 A 的执行顺序排在 B 之前。

常见规则：

1. 程序顺序规则：同一个线程内，前面的操作 happens-before 后面的操作。

2. 监视器锁规则：对一个锁的解锁 happens-before 后续对这个锁的加锁。

3. volatile 规则：对一个 volatile 变量的写 happens-before 后续对这个变量的读。

4. 线程启动规则：调用 `Thread.start()`  happens-before 线程中的动作。

5. 线程终止规则：线程中的所有操作 happens-before 其他线程检测到该线程终止。

6. 传递性：A happens-before B，B happens-before C，则 A happens-before C。

### synchronized为什么是可重入锁？

可重入锁指同一个线程已经持有某把锁时，可以再次获取这把锁，不会被自己阻塞。

`synchronized`  是可重入锁。

底层 Monitor 中会维护锁的持有线程和重入计数。

当线程第一次获取锁时，计数加一。

同一个线程再次进入同一把锁保护的代码时，计数继续加一。

每退出一次同步代码块，计数减一。

当计数减到 0 时，锁才真正释放。

### synchronized锁升级为什么需要这个过程？

锁升级是 JVM 为了适配不同竞争强度设计的优化。

如果一开始就使用重量级锁，会涉及线程阻塞、唤醒、用户态和内核态切换，开销很大。

所以 JDK1.6 后对 `synchronized`  做了优化：

1. 没有竞争时，使用偏向锁，优化同一个线程反复进入同步块的场景。

2. 轻微竞争时，使用轻量级锁，通过 CAS 自旋避免线程阻塞。

3. 竞争激烈时，升级为重量级锁，让竞争失败的线程阻塞，避免 CPU 空转。

所以锁升级的核心思想是：在保证线程安全的前提下，根据竞争强度选择不同成本的锁实现。

### synchronized锁升级详细过程

##### 无锁

对象最开始处于无锁状态。

对象头 Mark Word 中会存储哈希码、GC 分代年龄、锁标志位等信息。

##### 偏向锁

当一个线程第一次进入同步代码块时，JVM 会尝试把对象头中的线程 ID 设置为当前线程 ID。

如果后续还是同一个线程进入，就直接比较线程 ID，不需要 CAS。

偏向锁适合同一个线程反复获取同一把锁的场景。

##### 偏向锁撤销

如果有其他线程来竞争这把锁，偏向锁会被撤销。

JVM 会检查原来持有偏向锁的线程是否还存活、是否还在同步块内。

如果原线程已经不再持有锁，可以把对象恢复为无锁或重新偏向。

如果原线程仍在同步块内，就升级为轻量级锁。

##### 轻量级锁

线程会在自己的栈帧中创建锁记录，把对象头 Mark Word 复制到锁记录中。

然后通过 CAS 尝试把对象头更新为指向锁记录的指针。

CAS 成功就获取轻量级锁。

CAS 失败说明发生竞争，线程会尝试自旋。

##### 重量级锁

如果自旋失败，或者竞争线程过多，轻量级锁会膨胀为重量级锁。

对象头 Mark Word 会指向 Monitor。

竞争失败的线程进入 EntryList 阻塞等待。

持锁线程释放锁后，唤醒等待线程重新竞争。

### 锁自旋，是乐观锁发生可能性大还是悲观锁可能性大？

锁自旋更常见于乐观锁，尤其是 CAS。

CAS 的工作方式就是比较并交换，失败后可以不断重试，这个重试过程就是自旋。

乐观锁假设冲突概率低，所以短暂自旋的成本通常小于线程挂起和唤醒。

悲观锁中也可能有自旋。

例如 `synchronized`  的轻量级锁会先尝试 CAS 自旋，避免直接阻塞。

但是在悲观锁中，自旋更多是一种优化手段。

如果竞争激烈，自旋失败后仍然会进入阻塞。

### ReentrantLock如何实现公平锁和非公平锁？

`ReentrantLock`  默认是非公平锁。

公平锁指线程严格按照请求锁的顺序获取锁。

非公平锁允许线程插队抢锁。

底层基于 AQS。

当线程没有获取到锁时，会被封装成 Node 节点加入 AQS 的 FIFO 双向队列。

公平锁实现：

线程尝试获取锁时，会先判断 AQS 队列中是否已经有线程在等待。

如果有，就排到队尾等待，不插队。

非公平锁实现：

线程来了会先尝试直接 CAS 抢锁。

如果抢不到，再加入 AQS 队列。

非公平锁吞吐量通常更高，但是可能导致某些线程等待时间较长。

### AQS独占模式和共享模式

AQS 有两种资源获取模式：独占模式和共享模式。

##### 独占模式

同一时刻只有一个线程能获取资源。

典型实现是 `ReentrantLock` 。

在 `ReentrantLock`  中，`state`  表示锁的持有状态和重入次数。

##### 共享模式

允许多个线程同时获取资源。

典型实现是 `CountDownLatch` 、`Semaphore` 、`ReentrantReadWriteLock`  的读锁。

在 `CountDownLatch`  中，`state`  表示剩余计数。

在 `Semaphore`  中，`state`  表示剩余许可证数量。

### Semaphore是什么？

`Semaphore`  是信号量，用来控制同时访问某个资源的线程数量。

它内部维护许可证数量。

线程访问资源前先调用 `acquire()`  获取许可证。

访问完成后调用 `release()`  归还许可证。

如果许可证数量为 0，新的线程就会阻塞等待。

使用场景：

1. 限流。

2. 控制连接池资源数量。

3. 控制同一时间访问某接口或资源的线程数。

### CountDownLatch、CyclicBarrier、Semaphore区别

`CountDownLatch` ：

1. 倒计时器。

2. 一个或多个线程等待其他线程完成。

3. 计数器减到 0 后放行。

4. 一次性，不能重置。

`CyclicBarrier` ：

1. 循环屏障。

2. 多个线程互相等待。

3. 所有线程都到达屏障点后一起继续。

4. 可以重复使用。

`Semaphore` ：

1. 信号量。

2. 控制同一时间最多有多少线程访问资源。

3. 可以理解为许可证。

### wait和sleep的区别

`wait()`  和 `sleep()`  都能让线程暂停执行，但是区别很大。

1. `wait()`  是 `Object`  的方法，`sleep()`  是 `Thread`  的静态方法。

2. `wait()`  必须在同步代码块或同步方法中调用，`sleep()`  不要求持有锁。

3. `wait()`  会释放锁，`sleep()`  不会释放锁。

4. `wait()`  需要通过 `notify()`  或 `notifyAll()`  唤醒，也可以设置超时时间。

5. `sleep()`  时间到了会自动进入可运行状态。

### notify和notifyAll的区别

`notify()`  会唤醒 WaitSet 中的一个等待线程。

`notifyAll()`  会唤醒 WaitSet 中的所有等待线程。

被唤醒的线程不会立刻执行，它们还需要重新竞争锁。

一般更推荐使用 `notifyAll()` ，避免因为唤醒了不满足条件的线程而导致其他线程一直等待。

使用 `wait()`  时通常要放在 while 循环中判断条件：

```java
synchronized (lock) {
    while (!condition) {
        lock.wait();
    }
}
```

原因是线程可能被虚假唤醒，或者被唤醒后条件仍然不满足。

### 为什么不推荐使用Executors创建线程池？

不推荐使用 `Executors`  创建线程池，主要是为了避免资源耗尽风险。

`newFixedThreadPool()`  和 `newSingleThreadExecutor()` ：

它们使用的队列长度可能是 `Integer.MAX_VALUE` ，任务堆积过多可能导致 OOM。

`newCachedThreadPool()` ：

最大线程数可能是 `Integer.MAX_VALUE` ，如果任务过多，可能创建大量线程，导致 OOM。

所以更推荐直接使用 `ThreadPoolExecutor` ，明确指定核心线程数、最大线程数、队列大小、线程工厂和拒绝策略。

### 线程池核心线程数和最大线程数怎么设定？

常见经验：

1. CPU 密集型任务：线程数可以设置为 CPU 核数 + 1。

2. IO 密集型任务：线程数可以设置为 2 * CPU 核数。

更通用的公式：

```text
线程数 = CPU核数 * (1 + 线程等待时间 / 线程运行时间)
```

但是这些公式只是参考，不建议硬套。

因为一个应用中可能有多个线程池，也有其他业务线程。

实际应该先根据公式设置一个初始值，再通过压测、监控和业务耗时不断调整。

目标是让 CPU 利用率合理，同时任务等待时间和接口响应时间都在可接受范围内。

### 上线后你会关注线程池的哪些指标？

上线后主要关注以下指标：

##### 运行时状态指标

1. 核心线程数。

2. 活跃线程数。

3. 最大线程数。

4. 队列大小。

如果活跃线程数长期接近最大线程数，说明线程池负载较高。

如果队列长期堆积，说明任务处理速度跟不上提交速度。

##### 任务处理效率指标

1. 任务提交速率。

2. 任务完成速率。

3. 任务平均耗时。

4. 任务等待时间。

如果完成速率长期低于提交速率，就会出现任务堆积。

##### 资源占用指标

1. CPU 使用率。

2. 内存占用。

3. 线程阻塞时间。

如果 CPU 高但吞吐低，可能存在锁竞争或计算型任务过重。

如果 CPU 低但队列堆积，可能是 IO 阻塞或线程数不足。

##### 异常指标

1. 拒绝任务数。

2. 线程创建和销毁频率。

3. 死锁检测。

拒绝任务增多时，需要检查最大线程数、核心线程数、队列容量和任务耗时。

### 动态线程池是什么？

动态线程池指线程池的核心参数可以在线上运行时动态调整。

常见做法：

1. 把核心线程数、最大线程数、队列容量、拒绝策略等参数暴露出来。

2. 参数从代码配置迁移到配置中心。

3. 配置变更后，线程池实时生效。

4. 结合线程池监控指标判断是否需要调整。

动态线程池适合复杂业务系统。

因为线上流量会变化，一个固定参数不一定能长期适配所有场景。

### Future和CompletableFuture是什么？

`Future`  表示异步任务的执行结果。

通过 `submit()`  提交 `Callable`  后，会返回 `Future` 。

可以通过 `Future.get()`  获取结果。

缺点是：

1. `get()`  会阻塞。

2. 不方便做任务编排。

3. 不方便处理多个异步任务之间的依赖关系。

`CompletableFuture`  是更强大的异步编程工具。

它支持：

1. 异步执行任务。

2. 任务完成后回调。

3. 多任务组合。

4. 异常处理。

5. 链式编排。

常见方法：

1. `supplyAsync()` ：异步执行有返回值任务。

2. `runAsync()` ：异步执行无返回值任务。

3. `thenApply()` ：对上一步结果进行转换。

4. `thenAccept()` ：消费结果。

5. `exceptionally()` ：处理异常。

### 原子类为什么线程安全？

JUC 中的原子类，比如 `AtomicInteger` 、`AtomicLong` 、`AtomicReference` ，底层主要依赖 CAS + volatile。

`volatile`  保证变量的可见性。

CAS 保证更新过程的原子性。

例如 `AtomicInteger.getAndIncrement()`  内部会循环读取当前值，计算新值，然后 CAS 更新。

如果 CAS 失败，说明期间被其他线程修改了，就重新读取并重试。

原子类适合简单变量的原子操作，代码比 `synchronized`  更简洁。

但是如果是多个变量之间的一致性，单个原子类就不够了。

### LongAdder和AtomicLong有什么区别？

`AtomicLong`  通过 CAS 更新一个变量。

低并发下性能很好。

但是高并发下，多个线程都竞争同一个变量，CAS 失败重试会很多，CPU 开销较高。

`LongAdder`  使用分段累加思想。

它把一个热点变量拆成多个 Cell，不同线程更新不同 Cell，最后求和时再把 base 和 Cell 累加。

优点是高并发写入性能更好。

缺点是读取结果不是强一致的瞬时值，适合统计计数类场景。

### 线程上下文切换是什么？

线程上下文切换指 CPU 从执行一个线程切换到执行另一个线程。

切换时需要保存当前线程的状态，比如程序计数器、寄存器、栈指针等。

然后再恢复另一个线程的状态。

上下文切换有成本。

如果线程数过多、锁竞争严重、频繁阻塞唤醒，就会导致上下文切换增多，系统性能下降。

减少上下文切换的方法：

1. 合理设置线程池大小。

2. 减少锁竞争。

3. 缩小锁粒度。

4. 使用无锁并发或 CAS。

5. 避免线程频繁创建销毁。

### 如何排查CPU飙高？

常见排查流程：

1. 使用 `top`  找到 CPU 占用高的 Java 进程 PID。

2. 使用 `top -Hp PID`  找到 CPU 占用高的线程 ID。

3. 把线程 ID 转成 16 进制。

4. 使用 `jstack PID | grep 16`进制线程`ID -A 30`  查看线程堆栈。

5. 根据堆栈定位具体代码。

常见原因：

1. 死循环。

2. 大量自旋。

3. 锁竞争。

4. 频繁 GC。

5. 线程池任务堆积导致大量线程竞争。

### 如何排查死锁？

可以使用 `jstack` 。

如果 JVM 检测到 Java 层面的死锁，`jstack`  输出中通常会出现 `Found one Java-leveldeadlock` 。

排查思路：

1. 找到互相等待的线程。

2. 查看每个线程持有什么锁、等待什么锁。

3. 找到对应业务代码。

4. 调整加锁顺序或减少嵌套锁。

也可以通过 `ThreadMXBean.findDeadlockedThreads()`  在程序中检测死锁。

### Java并发扩展中容易混淆的点

1. `volatile`  能保证可见性和有序性，但不能保证 `i++`  这类复合操作的原子性。

2. `synchronized`  锁的是对象，不是代码；静态方法锁的是 Class 对象。

3. `wait()`  会释放锁，`sleep()`  不会释放锁。

4. `notify()`  唤醒线程后，被唤醒线程还要重新竞争锁。

5. `ThreadLocal`  是线程隔离，不是解决共享变量加锁问题。

6. 线程池核心线程满了后，任务先进入队列，不是马上创建非核心线程。

7. `submit()`  的异常会封装到 `Future`  里，不调用 `get()`  可能感知不到。

8. `CountDownLatch`  不能重置，`CyclicBarrier`  可以复用。

9. `ReentrantLock`  默认是非公平锁。

10. CAS 在低冲突场景性能好，高冲突场景可能因为自旋导致 CPU 开销高。

## 后端面经与场景补充

<!-- 原PDF第482页；面经原题号 19 -->

### 线程池的工作流程

原文未附答案。

<!-- 原PDF第482页；面经原题号 20 -->

### 手撕阻塞队列

```java
import java.util.LinkedList;
import java.util.Queue;

public class BlockingQueue<T> {
    private Queue<T> queue = new LinkedList<>();
    private int capacity;

    public BlockingQueue(int capacity) {
        this.capacity = capacity;
    }

    public synchronized void put(T item) throws InterruptedException {
        while (queue.size() == capacity) {
            wait();
        }
        queue.add(item);
        notifyAll();
    }

    public synchronized T take() throws InterruptedException {
        while (queue.isEmpty()) {
            wait();
        }
        T item = queue.poll();
        notifyAll();
        return item;
    }

    public static void main(String[] args) {
        BlockingQueue<Integer> blockingQueue = new BlockingQueue<>(5);

        // Producer thread
        Thread producer = new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    blockingQueue.put(i);
                    System.out.println("Produced: " + i);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        // Consumer thread
        Thread consumer = new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    int item = blockingQueue.take();
                    System.out.println("Consumed: " + item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        producer.start();
        consumer.start();
    }
}
```

<!-- 原PDF第490页；面经原题号 43 -->

### 公平和非公平锁

公平锁 vs 非公平锁

公平锁 (Fair Lock)

定义：多个线程按照申请锁的时间顺序获取锁（FIFO 队列）

特点：

- 新请求线程必须加入等待队列末尾

- 锁释放时，总是唤醒等待时间最长的线程

- 保证所有线程都能公平获得执行机会优点：避免线程饥饿缺点：上下文切换开销大，吞吐量较低

非公平锁 (Non-fair Lock)

定义：线程获取锁的顺序与申请时间无关，允许插队

特点：

- 新请求线程可直接尝试抢锁（即使队列中有等待线程）

- 锁释放时，新线程可能比等待线程更快获得锁

- 默认实现方式（性能优化）优点：吞吐量高，减少上下文切换缺点：可能造成线程饥饿

<!-- 原PDF第491页；面经原题号 44 -->

### synchronized和reentrantlock区别是什么

原文未附答案。

<!-- 原PDF第499页；面经原题号 64 -->

### threadlock和hashmap当中的hash算法有什么不一样的

冲突处理：

- `ThreadLocalMap` ： 使用线性探测法。发生冲突时（计算出的索引位置已被占用），它会顺序检查下一个位置（`(i + 1) % table.length` ），直到找到空位或匹配的 key。这要求初始分布必须非常好。

- `HashMap` ： 使用拉链法。每个桶（数组元素）是一个链表（或达到阈值后转为红黑树）。发生冲突时，新的键值对会添加到对应桶的链表（或树）中。这比线性探测更能容忍一定程度的冲突。
