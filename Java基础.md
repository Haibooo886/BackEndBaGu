Java

# 1. Java线程里面有哪些状态 ？

答： unrunning 初始化线程完成但是未启动
 running 拿到CPU运行态
 wait 根据调度算法 CPU被调走
 被销毁状态

标准答案：Java `Thread.State` 有六种：NEW、RUNNABLE、BLOCKED、WAITING、TIMED_WAITING、TERMINATED。RUNNABLE 表示可运行（可能正在运行或等待 CPU）；BLOCKED 表示等待 synchronized 监视器锁；WAITING/TIMED_WAITING 分别表示无限期/限时等待。Java 没有单独的 RUNNING 枚举状态。

# 2. wait状态下的线程如何进行恢复到runing状态 ？

答： 被其他线程notify(ThreadId)唤醒，或者被notifyALL唤醒，或者其他任务完成拿到cpu

标准答案：WAITING 线程被同一监视器上的 `notify()` 或 `notifyAll()` 唤醒后，还要重新获得监视器锁，才能从 `wait()` 返回并进入 RUNNABLE；`notify()` 不接收线程 ID，也不保证立即获得 CPU。

# 3. notify和notifyAll的区别？

答： notify(ThreadId)需要指定唤醒的线程id唤醒对应的一个线程拿到CPU运行
 notifyAll则是唤醒所有wait的线程，所有线程竞争CPU

标准答案：`notify()` 唤醒该对象监视器上等待的一个线程，`notifyAll()` 唤醒全部等待线程；二者都必须在持有该对象监视器时调用，唤醒后仍需竞争锁。

# 4. 如何停止一个线程的运行？

答：

标准答案：采用协作式停止：设置 `volatile` 取消标志并定期检查，或用 `Future.cancel(true)` 发送中断请求；阻塞方法捕获 `InterruptedException` 后恢复中断标志并退出。不要使用已废弃的 `Thread.stop()`。

# 5. 调用interrupt是如何让线程抛出异常的？

答：

标准答案：`interrupt()` 只是设置中断标志。线程在 `sleep()`、`wait()`、`join()` 等可中断阻塞方法中时会抛出 `InterruptedException`；未阻塞时需通过 `isInterrupted()` 或 `Thread.interrupted()` 主动检查。

# 6. 如果是靠变量来停止线程，缺点是什么？

答：

标准答案：停止标志若没有 `volatile` 或同步保障，可能不可见；线程若长期阻塞在不可中断的 I/O 或锁等待中，也无法及时检查标志。因此通常使用 `volatile` 标志配合中断机制。

# 7. volatile保证原子性吗？

答： volatile保证可见性，不保证原子性，不可重排序
可见性： 通过volatile修饰的变量，在发生变更后会立即刷星到主内存，而其他线程需要读取此变量也必须到主内存读不会在旧的缓存读取保证了可见性。
不保证原子性： 在非原子操作的时候会出现Bing发问题。
不可重排序： 重排序指的是程序在运行的时候会被优化器改变代码的运行顺序来提效，有时候会出现并发问题，比如单例模式的双重检查锁就是通过volatile的不可重排序实现。

volatile标准答案：`volatile` 保证可见性和特定的有序性，但不保证复合操作的原子性，例如 `count++` 仍可能丢失更新。计数应使用原子类或锁；双重检查锁中的 `volatile` 用于防止对象发布重排序。

# 8. 那我们如何保证原子性？

答： 在非原子操作中可以添加syncronized可重入锁

标准答案：可使用 `synchronized`、`ReentrantLock` 等互斥锁，或使用 `AtomicInteger` 等原子类的 CAS 操作保护复合操作。

# 9. syncronized支持重入吗？ 如何实现的？

答： 支持可重入
 实现：底层是依靠操作系统的mutex lock实现每一个锁都有对应的threadid和status
 在获取锁的时候 判断threadid是否有值也就是是否有线程占领这个锁，如果没有就拿到锁并且更新threadid为自己的threadid，如果有锁占判断是否为自己的threadid如果是则为重入status+1.
 在释放锁的时候，若不是重入则直接释放
 若是重入则status-1，直到为0释放

标准答案：`synchronized` 支持可重入。同一线程再次进入已持有的监视器时增加重入次数，退出时递减，归零后才释放；JVM 通过对象头和监视器等机制实现，异常退出也会自动释放锁。

# 10. 编译型语言和解释型语言的区别

答：编译型语言：编译型语言就是运行之前先统一编译成字节码或者机器码，运行时再统一运行，跨平台能力差，例如：C/C++
 解释型语言：解释型语言是在运行时候由解释器逐行解释运行，效率较低但是跨平台能力好。例如：Python，JS

标准答案：1. 编译型语言：程序运行前，编译器把全部源代码一次性编译成机器码，生成可执行文件，直接在CPU运行。
-- 优点：执行速度快;
-- 缺点：编译产物和CPU架构绑定，跨平台差
-- 例子：C，C++，GO
 2. 解释型语言：没有提前完整编译，运行时解释器啊一边读源码一边翻译执行。
-- 优点：跨平台好
-- 缺点：执行效率偏低
-- 例子：PYTHON,JS,PHP

注意：Java属于半编译半解释：源码编译成字节码，字节码不直接跑CPU，交给JVM解释/即时编译执行，实现一次编译到处执行。
关键区分：编译型：源码->>机器码;Java：源码->>字节码;解释型：源码实时翻译

# 11. 动态数组的实现有哪些？

答： Vector和Arraylist

标准答案：1. ArrayList：最常用，非线程安全
 2. Vector：线程安全，方法加synchronized，性能差
 3. CopyOnWriteArraryList：并发场景动态数组，写时复制
底层原理：内部维护一个Object[]数组，对外提供可变长度API，容量不足自动扩容拷贝数组。

# 12. Arraylist和Vector的比较

答：Vector是线程安全的，Arraylist是非线程安全的
 Vector扩容是扩容0.5倍，Arraylist是扩容1倍

参考答案：1. 线程安全：ArrayList：方法不加锁，非线程安全，并发修改会出现并发修改异常;
 Vector：大部分方法加synchronized，线程安全，并发性能差
 2. 扩容机制：ArrayList：默认扩容为原来1.5倍
 Vector：默认扩容为原来2倍;支持自定义扩容增量
 3. 历史与使用：Vector是JDK早期集合，性能差，现在开发基本不使用;ArrayList是日常首选

# 13. HashMap的扩容条件是什么？

答：数组空间达到阈值，并且添加的此次元素发生hash碰撞

标准答案：
JDK1.8HashMap扩容触发条件：

1. map中元素总数量size>=threshold（阈值=当前数组容量x负载因子，默认负载因子0.75）;
2. 当前数组容量还没有达到最大容量 MAXIMUM_CAPACITY。

-- 注意：扩容和hash碰撞无关。
-- 树化条件：链表>8，并且数组容量>=64，链表转为红黑树;数组容量小于64，优先扩容，不树化。

# 14. 乐观锁和悲观锁

答： 悲观锁：默认会出现并发问题，在访问数据前加锁，别的线程想拿到数据就阻塞等待，拿到锁才能执行业务
    乐观锁：默认大概率不会出现并发问题，不上锁;更新的时候去检查，期间数据有没有被别人修改过，如果被修改过就放弃本次操作，报错或者重试。

注意：乐观锁在高并发下大量重试，CPU消耗高;会出现ABA问题

# 15. 什么是ABA问题

-- CAS：比较内存当前值和预期值是否相等，相等则更新
ABA问题：线程1读取变量值A-->线程2把值A改为B又改为A-->线程1执行CAS，看到内存还是A，认为数据没有被修改过执行更新（实际上中间数据被修改过，CAS感知不到）

ABA问题解决：1. 增加版本号;每次修改版本号+1;CAS比较的时候，不仅比较数据值，还要比较版本  号。即便数据ABA，版本号改变了，CAS失败。
            2. Java中AtomicStampedReference不仅存数据，额外维护一个stamp邮戳（版本标记）;CAS的时候同时校验值+stamp版本戳，版本戳变化就更新失败，解决ABA。

# 16. ArrayList和LinkedList的区别

答：底层实现：ArrayList的底层是object[]的动态数组，LinkedList的底层实现是双向链表
    查询增删：ArrayList支持索引随机查询时间复杂度O(1)，LinkedList随机查找需要从头遍历时间复杂度O(n);ArrayList在尾部插入删除效率高，如果在头部或者中间插入删除需要移动大量元素时间复杂度O(n)，LinkedList在头尾增删效率高只需要修改指针O(1)，中间增删效率低需要先遍历再修改0(n)
    空间占用：ArrayList只需要存储元数据，LinkedList每个节点都还需要存储前驱和后继指针所以空间占用较大
    内存分配：ArrayList在内存中是一段连续空间，LinkedList在内存中不是连续的，因为有指针。

# 17. ConcurrentHashMap

答：JDK1.7以及以前
    底层：Segment+HashEntry数组实现;Segment继承ReentrantLock，相当于一把锁，将map分为多个Segment分段锁起来;每个Segment内部都是HashEntry+链表;
    JDK1.8以及以后
    底层：数组+链表+红黑树，结构跟hashmap完全一致;锁的细粒度方面不再锁多个数组，而是锁链表头节点或者红黑树头节点;锁的细粒度大幅降低，并发性能提高;

# 18. 阻塞队列

答：阻塞队列是附加了两个特殊功能的队列
    1. 在队列为空的时候，消费者等待队列变为非空再取元素
    2. 在队列为满的时候，生产者等待队列可用，再存元素

# 19. 线程安全的List

答：1. Vector
    底层：动态数组，方法全部加 synchronized（方法级锁）
    锁粒度大：每次 add/get/remove 都锁住整个对象。
    缺点：并发性能差；扩容默认 2 倍；JDK 古老类，开发几乎不用。
    2. Collections.synchronizedList()
    包装模式，把普通List(ArrayList)包装成线程安全的集合
    List<String> list = Collections.synchronizedList(new ArrayList<>());
    3.CopyOnWriteArrayList
    底层原理：
    1. 读操作：不加锁，直接读原数组，性能很高
    2. 写操作：加ReentrantLock锁，先拷贝一份新数组，在新数组上修改;修改完成之后将引用指向新数组。旧数组还在，正在读的线程继续读旧数组。