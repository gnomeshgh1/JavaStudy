##                                                         Java并发

### 1.为什么HashMap线程不安全？

  线程安全是指在多线程（并发）环境下，多个线程同时操作同一个对象时，不会出现不符合预期的错误结果。然而 HashMap 却是线程不安全的，也就是当多个线程同时操作 HashMap 时，会出现不确定的错误结果。

#### [#](#为什么线程不安全) 为什么线程不安全？

HashMap 非线程安全主要是因为 HashMap 的设计中，未采用任何同步机制（锁机制）来保证其安全性。 这一点在它的源码注释中也有说明，如下图所示：

![img](https://javacn.site/image/1681374940658-0c0ea665-9e10-4482-a793-4324a30a5abd.png)

#### [#](#线程不安全问题展现) 线程不安全问题展现

HashMap 线程不安全主要体现在以下两方面：

1. 在 JDK 1.7 中的死循环问题
2. 所有版本中的数据覆盖问题

#### [#](#死循环问题) 死循环问题

死循环问题是指在并发环境下，因为多个线程同时进行 put 操作，导致链表形成环形数据结构，一旦形成环形数据结构，在 get(key) 的时候就会产生死循环。如下图所示：

![img](https://javacn.site/image/1681186887864-1799ef94-b6a5-4b08-9aa9-9c147ebeaefc.png)

#### [#](#死循环原因) 死循环原因

HashMap 导致死循环的原因是由以下条件共同导致的：

1. HashMap 使用头插法进行数据插入（JDK 1.8 之前）；
2. 多线程同时添加；
3. 触发了 HashMap 扩容。

当满足以上所有条件时，HashMap 就会出现死循环问题。

#### [#](#数据覆盖问题) 数据覆盖问题

数据覆盖问题发生在并发添加元素的场景下，它不止出现在 JDK 1.7 版本中，其他版本中也存在此问题，数据覆盖产生的流程如下：

1. 线程 T1 进行添加时，判断某个位置可以插入元素，但还没有真正的进行插入操作，自己时间片就用完了。
2. 线程 T2 也执行添加操作，并且 T2 产生的哈希值和 T1 相同，也就是 T2 即将要存储的位置和 T1 相同，因为此位置尚未插入值（T1 线程执行了一半），于是 T2 就把自己的值存入到当前位置了。
3. T1 恢复执行之后，因为非空判断已经执行完了，它感知不到此位置已经有值了，于是就把自己的值也插入到了此位置，那么 T2 的值就被覆盖了。

具体执行流程如下图所示。

#### [#](#数据覆盖执行步骤一) 数据覆盖执行步骤一

线程 T1 准备将数据 k1:v1 插入到 Null 处，但还没有真正的执行，自己的时间片就用完了，进入休眠状态了，如下图所示：

![img](https://javacn.site/image/1641978698149-ef4c29c6-c869-4c2c-baff-6a18d77f62e5.png)

#### [#](#数据覆盖执行步骤二) 数据覆盖执行步骤二

线程 T2 准备将数据 k2:v2 插入到 Null 处，因为此处现在并未有值，如果此处有值的话，它会使用链式法将数据插入到下一个没值的位置上，但判断之后发现此处并未有值，那么就直接进行数据插入了，如下图所示：

![img](https://javacn.site/image/1641978706016-38fb0503-0b97-443e-b5d1-4282fb4c4f74.png)

#### [#](#数据覆盖执行步骤三) 数据覆盖执行步骤三

线程 T2 执行完成之后，线程 T1 恢复执行，因为线程 T1 之前已经判断过此位置没值了，所以会直接插入，此时线程 T2 插入的值就被覆盖了，如下图所示：

![img](https://javacn.site/image/1641978862811-2f8344ae-20ed-4bbf-869c-04f9ae617dfe.png)

#### [#](#解决方案) 解决方案

HashMap 非线程安全的解决方案有以下几个：

1. 使用线程安全容器 ConcurrentHashMap 替代 HashMap；
2. 使用线程安全容器 Hashtable 替代 HashMap（此方式性能不高，不推荐使用）；
3. 在进行 HashMap 操作时，使用 synchronized 或 Lock 加锁执行。

#### [#](#小结) 小结

HashMap 线程不安全的主要原因，是因为 HashMap 的设计中未采用任何同步机制（锁机制）来保证其安全性。它的线程不安全主要表现在死循环问题和数据覆盖的问题上。可以实现线程安全的容器，如 ConcurrentHashMap 或 Hashtable，或者是使用同步机制 synchronized 或 Lock 来保证其并发操作的安全性。

### 2.ConcurrentHashMap如何实现线程安全？

##### 众所周知 ConcurrentHashMap 是 HashMap 的多线程版本，HashMap 在并发操作时会有各种问题，比如死循环问题、数据覆盖等问题。而这些问题，只要使用 ConcurrentHashMap 就可以完美解决了，那问题来了，ConcurrentHashMap 是如何保证线程安全的？它的底层又是如何实现的？

#### [#](#concurrenthashmap-线程安全实现简述) ConcurrentHashMap 线程安全实现简述

ConcurrentHashMap 在 JDK 1.7 时，使用的是分段锁也就是 Segment 来实现线程安全的。 然而它在 JDK 1.8 之后，使用的是 CAS + synchronized 或 CAS + volatile 来实现线程安全的。

#### [#](#jdk-1-7-底层结构) JDK 1.7 底层结构

ConcurrentHashMap 在不同的 JDK 版本中实现是不同的，**在 JDK 1.7 中它使用的是数组加链表的形式实现的，而数组又分为：大数组 Segment 和小数组 HashEntry。**大数组 Segment 可以理解为 MySQL 中的数据库，而每个数据库（Segment）中又有很多张表 HashEntry，每个 HashEntry 中又有多条数据，这些数据是用链表连接的，如下图所示： ![image.png](https://javacn.site/image/1642158936354-73c81ea9-5965-4d81-9b02-5bd0503febc1.png)

#### [#](#jdk-1-7-线程安全实现) JDK 1.7 线程安全实现

了解了 ConcurrentHashMap 的底层实现，再看它的线程安全实现就比较简单了。 接下来，我们通过添加元素 put 方法，来看 JDK 1.7 中 ConcurrentHashMap 是如何保证线程安全的，具体实现源码如下：



```java
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    // 在往该 Segment 写入前，先确保获取到锁
    HashEntry<K,V> node = tryLock() ? null : scanAndLockForPut(key, hash, value); 
    V oldValue;
    try {
        // Segment 内部数组
        HashEntry<K,V>[] tab = table;
        int index = (tab.length - 1) & hash;
        HashEntry<K,V> first = entryAt(tab, index);
        for (HashEntry<K,V> e = first;;) {
            if (e != null) {
                K k;
                // 更新已有值...
            }
            else {
                // 放置 HashEntry 到特定位置，如果超过阈值则进行 rehash
                // 忽略其他代码...
            }
        }
    } finally {
        // 释放锁
        unlock();
    }
    return oldValue;
}
```

从上述源码我们可以看出，Segment 本身是基于 ReentrantLock 实现的加锁和释放锁的操作，这样就能保证多个线程同时访问 ConcurrentHashMap 时，同一时间只有一个线程能操作相应的节点，这样就保证了 ConcurrentHashMap 的线程安全了。

也就是说 ConcurrentHashMap 的线程安全是建立在 Segment 加锁的基础上的，所以我们把它称之为分段锁或片段锁，如下图所示：![image.png](https://javacn.site/image/1642158949058-7f5ba545-ab82-4f8f-bd44-10eab9a80794.png)

在 JDK 1.7 中，ConcurrentHashMap 虽然是线程安全的，但因为它的底层实现是数组 + 链表的形式，所以在数据比较多的情况下访问是很慢的，因为要遍历整个链表，而 JDK 1.8 则使用了数组 + 链表/红黑树的方式优化了 ConcurrentHashMap 的实现，具体实现结构如下：![image.png](https://javacn.site/image/1642160854966-9782789c-27a9-4541-acad-598a7b5569da.png) 链表升级为红黑树的规则：当链表长度大于 8，并且数组的长度大于 64 时，链表就会升级为红黑树的结构。

> PS：ConcurrentHashMap 在 JDK 1.8 虽然保留了 Segment 的定义，但这仅仅是为了保证序列化时的兼容性，不再有任何结构上的用处了。

#### [#](#jdk-1-8-线程安全实现) JDK 1.8 线程安全实现

在 JDK 1.8 中 ConcurrentHashMap 使用的是 CAS + volatile 或 synchronized 的方式来保证线程安全的，它的核心实现源码如下：



```java
final V putVal(K key, V value, boolean onlyIfAbsent) { if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh; K fk; V fv;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) { // 节点为空
            // 利用 CAS 去进行无锁线程安全操作，如果 bin 是空的
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value)))
                break; 
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else if (onlyIfAbsent
                 && fh == hash
                 && ((fk = f.key) == key || (fk != null && key.equals(fk)))
                 && (fv = f.val) != null)
            return fv;
        else {
            V oldVal = null;
            synchronized (f) {
                   // 细粒度的同步修改操作... 
                }
            }
            // 如果超过阈值，升级为红黑树
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```

从上述源码可以看出，在 JDK 1.8 中，添加元素时首先会判断容器是否为空，如果为空则使用 volatile 加 CAS 来初始化。如果容器不为空则根据存储的元素计算该位置是否为空，如果为空则利用 CAS 设置该节点；如果不为空则使用 synchronize 加锁，遍历桶中的数据，替换或新增节点到桶中，最后再判断是否需要转为红黑树，这样就能保证并发访问时的线程安全了。

我们把上述流程简化一下，我们可以简单的认为在 JDK 1.8 中，ConcurrentHashMap 是在头节点加锁来保证线程安全的，锁的粒度相比 Segment 来说更小了，发生冲突和加锁的频率降低了，并发操作的性能就提高了。而且 JDK 1.8 使用的是红黑树优化了之前的固定链表，那么当数据量比较大的时候，查询性能也得到了很大的提升，从之前的 O(n) 优化到了 O(logn) 的时间复杂度，具体加锁示意图如下： ![image.png](https://javacn.site/image/1642212323805-e0213870-0401-429c-8638-b15515998165.png)

#### [#](#小结) 小结

ConcurrentHashMap 在 JDK 1.7 时使用的是数据加链表的形式实现的，其中数组分为两类：大数组 Segment 和小数组 HashEntry，而加锁是通过给 Segment 添加 ReentrantLock 锁来实现线程安全的。而 JDK 1.8 中 ConcurrentHashMap 使用的是数组+链表/红黑树的方式实现的，它是通过 CAS 或 synchronized 来实现线程安全的，并且它的锁粒度更小，查询性能也更高。

### 3.ConcurrentHashMap为什么不允许插入null？

在 Java 语言中，ConcurrentHashMap 和 Hashtable 这些线程安全的集合是不允许 key 或 value 插入 null 值的，但是非线程安全的 HashMap 又可以插入 null 值，这到底是为什么呢？

#### [#](#null-值插入) null 值插入

首先给 HashMap 插入 null 值，实现代码如下：



```java
HashMap<String, Object> map = new HashMap();
// 插入 null 值
map.put(null, null);
if (map.containsKey(null)) {
    System.out.println("存在 null");
} else {
    System.out.println("不存在 null");
}
```

以上程序的执行结果如下：![image.png](https://javacn.site/image/1642217154217-5f67a292-68be-4e53-8abe-4e95c8b1aa4e.png) 从上述结果可以看出，HashMap 是允许 key 或 value 插入 null 值的。 接着我们使用同样的方式尝试给 ConcurrentHashMap 的 key 和 value 插入 null 值，实现代码如下：![image.png](https://javacn.site/image/1642222189371-67574557-0199-496a-a969-ba20659986f9.png)编译阶段没有报错，执行以上程序，得到的结果如下： ![image.png](https://javacn.site/image/1642222281218-7db4916a-0a5e-4e56-b0f7-ed9a504b3a18.png) 从上述报错信息可以看出，使用 ConcurrentHashMap 是不能插入 null 值的，否者程序在运行期间就会报空指针异常。

> PS：Hashtable 使用与 ConcurrentHashMap 类似，这里就不再重复演示了。

#### [#](#concurrenthashmap-源码分析) ConcurrentHashMap 源码分析

为了寻找报错的原因，我们尝试打开 ConcurrentHashMap 的源码一探究竟。 打开 ConcurrentHashMap 添加元素的方法 put 实现源码如下： ![image.png](https://javacn.site/image/1642222570142-554b416a-2af5-41b6-9ddd-a68166ac4e00.png) 从上述源码可以看出，在添加方法的第一句就加了判断：如果 key 值为 null 或者是 value 值为 null，就直接抛出异常 NullPointerException 空指针异常，这就是咱们前面程序报错的原因了。

#### [#](#探索最终原因) 探索最终原因

通过上面源码分析，我们似乎已经找到了 ConcurrentHashMap 不允许插入 null 值的原因，用一句话概括就是：乌龟的屁股“规定”！ 然而，这个原因是不能说服面试官的，虽然源码是这样设计的，但我们要思考的是，这样设计背后更深层次的原因，为什么 ConcurrentHashMap 不允许插入 null？而 HashMap 又允许插入 null 呢？

#### [#](#二义性问题) 二义性问题

所谓的二义性问题是指含义不清或不明确。 我们假设 ConcurrentHashMap 允许插入 null，那么此时就会有二义性问题，它的二义性含义有两个：

1. 值没有在集合中，所以返回 null。
2. 值就是 null，所以返回的就是它原本的 null 值。

可以看出这就是 ConcurrentHashMap 的二义性问题，那为什么 HashMap 就不怕二义性问题呢？

#### [#](#可证伪的-hashmap) 可证伪的 HashMap

上面说到 HashMap 是不怕二义性问题的，为什么呢？ 这是因为 HashMap 的设计是给单线程使用的，所以如果查询到了 null 值，我们可以通过 hashMap.containsKey(key) 的方法来区分这个 null 值到底是存入的 null？还是压根不存在的 null？这样二义性问题就得到了解决，所以 HashMap 不怕二义性问题。

#### [#](#不可证伪的-concurrenthashmap) 不可证伪的 ConcurrentHashMap

而 ConcurrentHashMap 就不一样了，因为 ConcurrentHashMap 使用的场景是多线程，所以它的情况更加复杂。 我们假设 ConcurrentHashMap 可以存入 null 值，有这样一个场景，现在有一个线程 A 调用了 concurrentHashMap.containsKey(key)，我们期望返回的结果是 false，但在我们调用 concurrentHashMap.containsKey(key) 之后，未返回结果之前，线程 B 又调用了 concurrentHashMap.put(key,null) 存入了 null 值，那么线程 A 最终返回的结果就是 true 了，这个结果和我们之前预想的 false 完全不一样。

也就是说，多线程的状况非常复杂，我们没办法判断某一个时刻返回的 null 值，到底是值为 null，还是压根就不存在，也就是二义性问题不可被证伪，所以 ConcurrentHashMap 才会在源码中这样设计，直接杜绝 key 或 value 为 null 的歧义问题。

#### [#](#concurrenthashmap-设计者的回答) ConcurrentHashMap 设计者的回答

对于 ConcurrentHashMap 不允许插入 null 值的问题，有人问过 ConcurrentHashMap 的作者 Doug Lea，以下是他回复的邮件内容：

> The main reason that nulls aren't allowed in ConcurrentMaps (ConcurrentHashMaps, ConcurrentSkipListMaps) is that ambiguities that may be just barely tolerable in non-concurrent maps can't be accommodated. The main one is that if map.get(key) returns null, you can't detect whether the key explicitly maps to null vs the key isn't mapped. In a non-concurrent map, you can check this via map.contains(key),but in a concurrent one, the map might have changed between calls.
>
> Further digressing: I personally think that allowing nulls in Maps (also Sets) is an open invitation for programs to contain errors that remain undetected until they break at just the wrong time. (Whether to allow nulls even in non-concurrent Maps/Sets is one of the few design issues surrounding Collections that Josh Bloch and I have long disagreed about.)
>
> > It is very difficult to check for null keys and values in my entire application .
>
> Would it be easier to declare somewhere   static final Object NULL = new Object(); and replace all use of nulls in uses of maps with NULL?
>
> -Doug

以上信件的主要意思是，Doug Lea 认为这样设计最主要的原因是：不容忍在并发场景下出现歧义！

#### [#](#小结) 小结

在 Java 语言中，HashMap 这种单线程下使用的集合是可以设置 null 值的，而并发集合如 ConcurrentHashMap 或 Hashtable 是不允许给 key 或 value 设置 null 值的，这是 JDK 源码层面直接实现的，这样设计的目的主要是为了防止并发场景下的歧义问题。

### 4.进程和线程有什么区别？

进程和线程是操作系统中的两个核心概念，它们的出现是为了更好地管理计算机资源和提高系统的运行效率，使用它们可以实现多任务同时运行，从而提高了系统资源的利用率。

#### [#](#定义) 定义

进程和线程的区别也是很明显的，进程是资源分配的最小单位，线程是 CPU 调度的最小单位。具体来说：

- 进程（Process）是指正在运行的一个程序的实例。每个进程都拥有的资源：堆、栈、虚存空间（页表）、文件描述符等信息。在 Java 中，每个进程都由一个主线程（main thread）启动。当进程运行时，操作系统会为其分配一个进程号，并将其作为一个独立的实体来进行管理。
- 线程（Thread）是指进程中的一个执行单元，是 CPU 调度和分派的基本单位，它是比进程更小的能独立运行的基本单位。在 Java 中，每个线程都拥有自己的栈空间和程序计数器，并且可以访问共享的堆内存。

#### [#](#举例说明) 举例说明

举例例子，假设你正在玩一款游戏，那么这个过程可以被看作是一个进程。在这个进程中，有很多不同的任务需要同时运行，比如说渲染游戏画面、处理玩家的输入、播放音效等等，这些任务可以被看作是不同的线程。 具体来说，渲染游戏画面的任务可以被看作是一个线程，处理玩家的输入也是一个线程，播放音效也可以是一个线程。这些线程可以同时运行，并且它们之间可以共享进程的内存空间，以便彼此之间传递信息和数据。

另外，如果你同时打开了两个不同的游戏，那么这就是两个独立的进程。这两个进程之间无法直接共享内存空间，它们需要使用进程间通信（IPC）机制来进行通信和数据传递。而且如果一个游戏崩溃了，另一个游戏不会受到影响，因为它们是独立的进程。

#### [#](#代码使用) 代码使用

#### [#](#进程使用) 进程使用

在 Java 中，创建进程的示例代码如下：



```java
public class CreateProcessDemo {
    public static void main(String[] args) {
        try {
            // 通过 ProcessBuilder 创建一个新的进程
            ProcessBuilder processBuilder = new ProcessBuilder("notepad.exe");
            Process process = processBuilder.start();

            // 等待进程结束
            int exitCode = process.waitFor();
            System.out.println("进程已结束，退出码为：" + exitCode);
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

在这个示例中，我们使用了 ProcessBuilder 类来创建一个新的进程，这里我们创建了一个名为 notepad.exe 的进程，即打开记事本应用程序。

然后，我们调用 waitFor() 方法等待进程结束，并获取进程的退出码。最后，我们将退出码输出到控制台。 当你运行这个程序时，会看到一个新的记事本窗口弹出来，这就是我们创建的进程。当你关闭记事本窗口时，程序会继续执行，输出进程的退出码。

需要注意的是，使用进程时要格外小心，因为它们可以直接操作系统资源，并对系统造成影响。所以在使用进程时，一定要谨慎处理，并确保代码的安全性和稳定性。

#### [#](#线程使用) 线程使用

在 Java 中，创建线程的示例代码如下：



```java
public class CreateThreadDemo {
    public static void main(String[] args) {
        // 创建一个新的线程
        Thread thread = new Thread(() -> {
            System.out.println("线程开始执行");
            try {
                Thread.sleep(5000); // 模拟线程执行耗时任务
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("线程执行完成");
        });
        // 启动线程
        thread.start();
        System.out.println("主线程继续执行");
    }
}
```

在这个示例中，我们使用了 Thread 类来创建一个新的线程。在 Thread 的构造函数中，我们传入了一个 Runnable 接口的实现，该实现定义了线程的任务，这里我们只是简单地输出一些信息并模拟了一个耗时任务。 然后，我们调用 start() 方法来启动线程。当线程启动后，它会执行 run() 方法中定义的任务，同时主线程会继续执行。在这个示例中，我们在主线程中输出了一条信息。

需要注意的是，使用线程时同样需要谨慎处理，因为线程可以直接操作共享资源，并对系统造成影响。在使用线程时，需要确保线程的安全性和稳定性，同时避免出现线程安全问题，比如死锁等问题。

#### [#](#进程-vs-线程) 进程 VS 线程

线程可以看作是轻量级的进程，一个进程中包含了多个线程，因此多个线程间可以共享进程资源，线程和进程的关系如下图所示： ![image.png](https://javacn.site/image/1643163979179-992c9f4b-2a31-4418-8f67-fb4498a4ebd9.png) 其中，堆和方法区是可以共享的区域，而程序计数器和栈是每个线程私有的。

- 程序计数器是一块内存区域，用来记录线程当前要执行的指令地址。
- 栈是用来记录每个线程自己的局部变量的。
- 堆中存放的是当前程序创建的所有对象。
- 方法区存放的是常量和静态变量等信息。

总的来说，进程和线程的区别主要有以下几点：

1. 进程是系统分配资源的基本单位，线程是程序执行的基本单位；
2. 进程拥有独立的内存空间和资源，而线程则共享进程的内存和资源；
3. 进程之间的通信比较复杂，而线程之间可以直接共享数据；
4. 进程的切换代价比较大，需要保存上下文和状态，而线程的切换代价比较小，因为它们共享进程的资源。

#### [#](#小结) 小结

进程是系统分配资源的基本单位，线程是程序执行的基本单位。一个进程中至少包含一个线程，线程不能独立于进程而存在。进程不能共享资源，而线程可以。线程可以看作是轻量级的进程，但它们的定义、资源共享方式、切换代价等都不同。

### 5.线程等待和唤醒有几种实现？

在 Java 中，线程等待和唤醒的实现手段有以下几种方式：

1. Object 类下的 wait()、notify() 和 notifyAll() 方法；
2. Condition 类下的 await()、signal() 和 signalAll() 方法；
3. LockSupport 类下的 park() 和 unpark() 方法。

为什么一个线程等待和唤醒机制就需要这么多的实现方式呢？别着急，咱们先来看实现，再来说原因。

#### [#](#一、wait-notify-notifyall) 一、wait/notify/notifyAll

Object 类的方法说明：

1. wait()：让当前线程处于等待状态，并释放当前拥有的锁；
2. notify()：随机唤醒等待该锁的其他线程，重新获取锁，并执行后续的流程，只能唤醒一个线程；
3. notifyAll()：唤醒所有等待该锁的线程（锁只有一把，虽然所有线程被唤醒，但所有线程需要排队执行）。

示例代码如下：



```java
Object lock = new Object();
// 创建线程并执行
new Thread(() -> {
    System.out.println("线程1：开始执行");
    synchronized (lock) {
        try {
            System.out.println("线程1：进入等待");
            lock.wait();
            System.out.println("线程1：继续执行");
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        System.out.println("线程1：执行完成");
    }
}).start();

Thread.sleep(1000);
synchronized (lock) {
    // 唤醒线程
    System.out.println("执行 notifyAll()");
    lock.notifyAll();
}
```

#### [#](#二、await-signal-signalall) 二、await/signal/signalAll

Condition 类的方法说明：

1. await()：对应 Object 的 wait() 方法，线程等待；
2. signal()：对应 Object 的 notify() 方法，随机唤醒一个线程；
3. signalAll()：对应 Object 的 notifyAll() 方法，唤醒所有线程。

示例代码如下：



```java
// 创建 Condition 对象
Lock lock = new ReentrantLock();
Condition condition = lock.newCondition(); // lock 下可创建多个 Condition
// 加锁
lock.lock();
try {
    // 业务方法......
    // 1.进入等待状态
    condition.await();
    // 2.唤醒操作
    condition.signal();
} catch (InterruptedException e) {
    e.printStackTrace();
} finally {
    lock.unlock();
}
```

#### [#](#三、park-unpark) 三、park/unpark

LockSupport 类的方法说明：

1. LockSupport.park()：休眠当前线程。
2. LockSupport.unpark(线程对象)：唤醒某一个指定的线程。

> PS：LockSupport 无需配锁（synchronized 或 Lock）一起使用。

示例代码如下：



```java
public static void main(String[] args) throws InterruptedException {
    Thread t1 = new Thread(() -> {
        LockSupport.park();
        System.out.println("线程1");
    }, "线程1");
    t1.start();
    Thread t2 = new Thread(() -> {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("唤醒线程1");
        LockSupport.unpark(t1);
    }, "线程2");
    t2.start();
}
```

#### [#](#四、小结) 四、小结

为什么一个线程等待和唤醒的功能需要这么多的实现呢？

1. **LockSupport 存在的必要性**：前两种方法 notify 方法以及 signal 方法都是随机唤醒，如果存在多个等待线程的话，可能会唤醒不应该唤醒的线程，因此有 LockSupport 类下的 park 和 unpark 方法指定唤醒线程是非常有必要的。
2. **Condition 存在的必要性**：Condition 相比于 Object 类的 wait 和 notify/notifyAll 方法，前者可以创建多个等待集，例如，我们可以创建一个生产者等待唤醒对象，和一个消费者等待唤醒对象，这样我们就能实现生产者只能唤醒消费者，而消费者只能唤醒生产者的业务逻辑了，如下代码所示：



```java
// 创建 Condition 对象
private Lock lock = new ReentrantLock();
// 生产者的 Condition 对象
private Condition producerCondition = lock.newCondition();
// 消费者的 Condition 对象
private Condition consumerCondition = lock.newCondition();
```

也就是 Condition 是 Object 等待唤醒模型的升级，Object 类可以实现的功能它都能实现，但 Condition 能实现的功能，Object 却不能实现，这就是 Condition 类存在的必要性。

> 那问题来了，为什么还有会 Object 的 wait 和 notify 方法呢？ 因为 Object 类诞生的比较早，也就是说 Condition 和 LockSupport 都是 JDK 后期版本才出现的功能，所以就有了现在这么多线程唤醒和等待的方法了。

### 6.什么是线程通讯，如何通讯？

线程通讯指的是多个线程之间通过共享内存或消息传递等方式来协调和同步它们的执行。在多线程编程中，通常会出现多个线程需要共同完成某个任务的情况，这时就需要线程之间进行通讯，以保证任务能够顺利地执行。

线程通讯的实现方式主要有以下两种：

共享内存：多个线程可以访问同一个共享内存区域，通过读取和写入内存中的数据来进行通讯和同步。 消息传递：多个线程之间通过消息队列、管道、信号量等机制来传递信息和同步状态。

#### [#](#常见场景) 常见场景

线程通讯的常见场景有以下几个：

1. 多个线程共同完成某个任务：例如一个爬虫程序需要多个线程同时抓取不同的网页，然后将抓取结果合并保存到数据库中。这时需要线程通讯来协调各个线程的执行顺序和共享数据。
2. 避免资源冲突：多个线程访问共享资源时可能会引发竞争条件，例如多个线程同时读写一个文件或数据库。这时需要线程通讯来同步线程之间的数据访问，避免资源冲突。
3. 保证顺序执行：在某些情况下，需要保证多个线程按照一定的顺序执行，例如一个多线程排序算法。这时需要线程通讯来协调各个线程的执行顺序。
4. 线程之间的互斥和同步：有些场景需要确保只有一个线程能够访问某个共享资源，例如一个计数器。这时需要使用线程通讯机制来实现线程之间的互斥和同步。

#### [#](#实现方法) 实现方法

线程通讯的实现方法有以下几种：

1. 等待和通知机制：使用 Object 类的 wait() 和 notify() 方法来实现线程之间的通讯。当一个线程需要等待另一个线程执行完某个操作时，它可以调用 wait() 方法使自己进入等待状态，同时释放占有的锁，等待其他线程调用 notify() 或 notifyAll() 方法来唤醒它。被唤醒的线程会重新尝试获取锁并继续执行。
2. 信号量机制：使用 Java 中的 Semaphore 类来实现线程之间的同步和互斥。Semaphore 是一个计数器，用来控制同时访问某个资源的线程数。当某个线程需要访问共享资源时，它必须先从 Semaphore 中获取一个许可证，如果已经没有许可证可用，线程就会被阻塞，直到其他线程释放了许可证。
3. 栅栏机制：使用 Java 中的 CyclicBarrier 类来实现多个线程之间的同步，它允许多个线程在指定的屏障处等待，并在所有线程都达到屏障时继续执行。
4. 锁机制：使用 Java 中的 Lock 接口和 Condition 接口来实现线程之间的同步和互斥。Lock 是一种更高级的互斥机制，它允许多个条件变量（Condition）并支持在同一个锁上等待和唤醒。

#### [#](#具体代码实现) 具体代码实现

#### [#](#等待通知实现) 等待通知实现

以下是一个简单的 wait() 和 notify() 方法的等待通知示例：



```java
public class WaitNotifyDemo {
    public static void main(String[] args) {
        Object lock = new Object();
        ThreadA threadA = new ThreadA(lock);
        ThreadB threadB = new ThreadB(lock);
        threadA.start();
        threadB.start();
    }
    static class ThreadA extends Thread {
        private Object lock;
        public ThreadA(Object lock) {
            this.lock = lock;
        }
        public void run() {
            synchronized (lock) {
                System.out.println("ThreadA start...");
                try {
                    lock.wait(); // 线程A等待
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("ThreadA end...");
            }
        }
    }
    static class ThreadB extends Thread {
        private Object lock;

        public ThreadB(Object lock) {
            this.lock = lock;
        }
        public void run() {
            synchronized (lock) {
                System.out.println("ThreadB start...");
                lock.notify(); // 唤醒线程A
                System.out.println("ThreadB end...");
            }
        }
    }
}
```

在这个示例中，定义了一个共享对象 lock，ThreadA 线程先获取 lock 锁，并调用 lock.wait() 方法进入等待状态。ThreadB 线程在获取 lock 锁之后，调用 lock.notify() 方法唤醒 ThreadA 线程，然后 ThreadB 线程执行完毕。 运行以上程序的执行结果如下：

> ThreadA start... ThreadB start... ThreadB end... ThreadA end...

#### [#](#信号量实现) 信号量实现

在 Java 中使用 Semaphore 实现信号量，Semaphore 是一个计数器，用来控制同时访问某个资源的线程数。当某个线程需要访问共享资源时，它必须先从 Semaphore 中获取一个许可证，如果已经没有许可证可用，线程就会被阻塞，直到其他线程释放了许可证。它的示例代码如下：



```java
import java.util.concurrent.Semaphore;

public class SemaphoreDemo {
    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(2);

        for (int i = 0; i < 5; i++) {
            new Thread(new Worker(i, semaphore)).start();
        }
    }

    static class Worker implements Runnable {
        private int id;
        private Semaphore semaphore;

        public Worker(int id, Semaphore semaphore) {
            this.id = id;
            this.semaphore = semaphore;
        }

        @Override
        public void run() {
            try {
                semaphore.acquire();
                System.out.println("Worker " + id + " acquired permit.");
                Thread.sleep(1000);
                System.out.println("Worker " + id + " released permit.");
                semaphore.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

在这个示例中，创建了一个 Semaphore 对象，并且设置了许可数为 2。然后创建了 5 个 Worker 线程，每个 Worker 线程需要获取 Semaphore 的许可才能执行任务。每个 Worker 线程在执行任务之前先调用 semaphore.acquire() 方法获取许可，如果没有许可则会阻塞，直到 Semaphore 释放许可。执行完任务之后调用 semaphore.release() 方法释放许可。 运行以上程序的执行结果如下：

> Worker 0 acquired permit. Worker 1 acquired permit. Worker 1 released permit. Worker 0 released permit. Worker 2 acquired permit. Worker 3 acquired permit. Worker 2 released permit. Worker 4 acquired permit. Worker 3 released permit. Worker 4 released permit.

#### [#](#栅栏实现) 栅栏实现

在 Java 中，可以使用 CyclicBarrier 或 CountDownLatch 来实现线程的同步，它们两个使用类似，接下来我们就是 CyclicBarrier 来演示一下线程的同步，CyclicBarrier 的示例代码如下：



```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierDemo {
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(3, new Runnable() {
            @Override
            public void run() {
                System.out.println("All threads have reached the barrier.");
            }
        });

        for (int i = 1; i <= 3; i++) {
            new Thread(new Worker(i, cyclicBarrier)).start();
        }
    }

    static class Worker implements Runnable {
        private int id;
        private CyclicBarrier cyclicBarrier;

        public Worker(int id, CyclicBarrier cyclicBarrier) {
            this.id = id;
            this.cyclicBarrier = cyclicBarrier;
        }

        @Override
        public void run() {
            try {
                System.out.println("Worker " + id + " is working.");
                Thread.sleep((long) (Math.random() * 2000));
                System.out.println("Worker " + id + " has reached the barrier.");
                cyclicBarrier.await();
                System.out.println("Worker " + id + " is continuing the work.");
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }
}
```

在这个示例中，创建了一个 CyclicBarrier 对象，并且设置了参与线程数为 3。然后创建了 3 个 Worker 线程，每个 Worker 线程会先执行一些任务，然后等待其他线程到达 Barrier。所有线程都到达 Barrier 之后，Barrier 会释放所有线程并执行设置的 Runnable 任务。 运行以上程序的执行结果如下：

> Worker 2 is working. Worker 3 is working. Worker 1 is working. Worker 3 has reached the barrier. Worker 1 has reached the barrier. Worker 2 has reached the barrier. All threads have reached the barrier. Worker 2 is continuing the work. Worker 3 is continuing the work. Worker 1 is continuing the work.

从以上执行结果可以看出，CyclicBarrier 保证了所有 Worker 线程都到达 Barrier 之后才能继续执行后面的任务，这样可以保证线程之间的同步和协作。在本示例中，所有线程都在 Barrier 处等待了一段时间，等所有线程都到达 Barrier 之后才继续执行后面的任务。

#### [#](#锁机制实现) 锁机制实现

以下是一个使用 Condition 的示例：



```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ConditionDemo {

    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();
    private volatile boolean flag = false;

    public static void main(String[] args) {
        ConditionDemo demo = new ConditionDemo();
        new Thread(demo::waitCondition).start();
        new Thread(demo::signalCondition).start();
    }

    private void waitCondition() {
        lock.lock();
        try {
            while (!flag) {
                System.out.println(Thread.currentThread().getName() + " is waiting for signal.");
                condition.await();
            }
            System.out.println(Thread.currentThread().getName() + " received signal.");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    private void signalCondition() {
        lock.lock();
        try {
            Thread.sleep(3000); // 模拟等待一段时间后发送信号
            flag = true;
            System.out.println(Thread.currentThread().getName() + " sends signal.");
            condition.signalAll();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

在这个示例中，创建了一个 Condition 对象和一个 Lock 对象，然后创建了两个线程，一个线程等待 Condition 信号，另一个线程发送 Condition 信号。

等待线程在获得锁后，判断标志位是否为 true，如果为 false，则等待 Condition 信号；如果为 true，则继续执行后面的任务。

发送线程在获得锁后，等待一段时间后，将标志位设置为 true，并且发送 Condition 信号。 运行以上程序的执行结果如下：

> Thread-0 is waiting for signal. Thread-1 sends signal. Thread-0 received signal.

从上面执行结果可以看出，等待线程在等待 Condition 信号的时候被阻塞，直到发送线程发送了 Condition 信号，等待线程才继续执行后面的任务。Condition 对象提供了一种更加灵活的线程通信方式，可以精确地控制线程的等待和唤醒。

#### [#](#小结) 小结

线程通讯指的是多个线程之间通过共享内存或消息传递等方式来协调和同步它们的执行，它的实现方法有很多：比如 wait() 和 notify() 的等待和通知机制、Semaphore 信号量机制、CyclicBarrier 栅栏机制，以及 Condition 的锁机制等。

### 7.如何停止线程？

在 Java 中，可以通过调用线程的 interrupt() 方法来中止线程。但是，这并不意味着线程会立即停止执行，它只是设置了一个中断标志，线程可以通过检查这个标志来自行终止。 具体来说，当线程被中断时，可以通过以下方式来检查中断标志：

1. 调用 Thread.currentThread().isInterrupted() 方法检查当前线程是否被中断。
2. 调用 Thread.interrupted() 方法检查当前线程是否被中断，并清除中断状态。

在实际应用中，如果需要中止一个线程，可以在执行任务的循环中检查中断标志，如果中断标志被设置，则退出循环，从而中止线程的执行。

下面是一个简单的示例代码，演示如何使用中断标志来中止线程：



```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        try {
            // 判断线程是否中止
            while (!Thread.currentThread().isInterrupted()) {
                // 执行任务
            }
        } catch (InterruptedException e) {
            // 处理异常
        } finally {
            // 执行清理操作
        }
    }
}
public class Main {
    public static void main(String[] args) {
        Thread t = new Thread(new MyRunnable());
        t.start();
        
        // 中止线程
        t.interrupt();
    }
}
```

在上面的示例代码中，MyRunnable 类实现了 Runnable 接口，重写了 run() 方法，在方法中使用了中断标志来控制循环的执行。在 Main 类中，创建了一个 MyRunnable 对象，并通过 Thread 类将其封装成一个线程。然后，通过调用 interrupt() 方法来中止线程的执行。在 MyRunnable 类的 run() 方法中，检查了线程的中断状态，并在中断状态被设置时退出了循环，从而中止了线程的执行。

### 8.线程池有什么优点？

线程池是一种管理和复用线程资源的机制，它由一个线程池管理器和一组工作线程组成。线程池管理器负责创建和销毁线程池，以及管理线程池中的工作线程。工作线程则负责执行具体的任务。

线程池的主要作用是管理和复用线程资源，避免了线程的频繁创建和销毁所带来的开销。 线程池包含两个重要的组成部分：

1. 线程池大小：指线程池中所能容纳的最大线程数。线程池大小一般根据系统的负载情况和硬件资源来设置。
2. 工作队列：用于存放等待执行的任务。当线程池中的工作线程已经全部被占用时，新的任务将被加入到工作队列中等待执行。

#### [#](#线程池创建方式) 线程池创建方式

Java中线程池的创建方式主要有以下几种：

1. 使用 ThreadPoolExecutor 类手动创建：通过 ThreadPoolExecutor 类的构造函数自定义线程池的参数，包括核心线程数、最大线程数、线程存活时间、任务队列等。
2. 使用 Executors 类提供的工厂方法创建：通过 Executors 类提供的一些静态工厂方法创建线程池，例如 newFixedThreadPool、newSingleThreadExecutor、newCachedThreadPool 等。
3. 使用 Spring 框架提供的 ThreadPoolTaskExecutor 类：在 Spring 框架中可以通过 ThreadPoolTaskExecutor 类来创建线程池。

不同的创建方式适用于不同的场景，通常可以根据实际情况选择合适的方式创建线程池。手动创建 ThreadPoolExecutor 类可以灵活地配置线程池参数，但需要对线程池的各项参数有一定的了解；使用 Executors 工厂方法可以快速创建线程池，但可能无法满足特定的需求，且容易出现内存溢出的情况；而 Spring 框架提供的 ThreadPoolTaskExecutor 类则只能在 Spring 框架中使用。

#### [#](#线程池使用) 线程池使用

#### [#](#threadpoolexecutor-使用) ThreadPoolExecutor 使用

ThreadPoolExecutor 线程的创建与使用示例如下：



```java
import java.util.concurrent.*;

public class ThreadPoolDemo {

    public static void main(String[] args) {
        // 创建线程池
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            2, // 核心线程数
            5, // 最大线程数
            60, // 线程池中线程的空闲时间（单位为秒）
            TimeUnit.SECONDS, // 时间单位
            new ArrayBlockingQueue<>(10) // 任务队列
        );
        // 提交任务
        for (int i = 1; i <= 20; i++) {
            final int taskId = i;
            executor.submit(() -> {
                System.out.println("执行任务：" + taskId + "，线程名称：" + Thread.currentThread().getName());
                try {
                    Thread.sleep(1000); // 模拟任务执行时间
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
        // 关闭线程池
        executor.shutdown();
    }
}
```

在上面的示例中，我们通过 ThreadPoolExecutor 类手动创建了一个线程池，设置了线程池的核心线程数为 2，最大线程数为 5，线程空闲时间为 60 秒，任务队列为长度为 10 的 ArrayBlockingQueue。然后我们通过 submit() 方法向线程池中提交了 20 个任务，每个任务会在执行时输出自己的编号和执行线程的名称，然后睡眠1秒钟模拟任务执行时间。最后，我们调用 shutdown() 方法关闭线程池。

#### [#](#executors-使用) Executors 使用



```java
import java.util.concurrent.*;

public class ExecutorsDemo {
    public static void main(String[] args) {
        // 创建线程池
        ExecutorService executor = Executors.newFixedThreadPool(5);
        // 提交任务
        for (int i = 1; i <= 10; i++) {
            final int taskId = i;
            executor.submit(() -> {
                System.out.println("执行任务：" + taskId + "，线程名称：" + Thread.currentThread().getName());
                try {
                    Thread.sleep(1000); // 模拟任务执行时间
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
        // 关闭线程池
        executor.shutdown();
    }
}
```

在上面的示例中，我们使用了 Executors 工厂类中的 newFixedThreadPool() 方法创建了一个线程池，该线程池有 5 个固定线程。然后我们通过 submit() 方法向线程池中提交了 10 个任务，每个任务会在执行时输出自己的编号和执行线程的名称，然后睡眠 1 秒钟模拟任务执行时间。 最后，我们调用 shutdown() 方法关闭线程池。值得注意的是，使用 Executors 工厂类创建线程池虽然非常简单，但是在实际生产环境中并不推荐，因为这种方式很容易导致线程资源被耗尽，从而影响系统的性能和稳定性。

#### [#](#threadpooltaskexecutor-使用) ThreadPoolTaskExecutor 使用

Spring 中 ThreadPoolTaskExecutor 的使用示例如下：



```java
import java.util.concurrent.*;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

public class ThreadPoolTaskExecutorDemo {

    public static void main(String[] args) {
        // 创建线程池
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2); // 核心线程数
        executor.setMaxPoolSize(5); // 最大线程数
        executor.setQueueCapacity(10); // 任务队列长度
        executor.initialize();
        // 提交任务
        for (int i = 1; i <= 20; i++) {
            final int taskId = i;
            executor.submit(() -> {
                System.out.println("执行任务：" + taskId + "，线程名称：" + Thread.currentThread().getName());
                try {
                    Thread.sleep(1000); // 模拟任务执行时间
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
        // 关闭线程池
        executor.shutdown();
    }
}
```

在上面的示例中，我们使用了 Spring 框架中的 ThreadPoolTaskExecutor 类创建了一个线程池，设置了线程池的核心线程数为 2，最大线程数为 5，任务队列长度为 10。然后我们通过 submit() 方法向线程池中提交了 20 个任务，每个任务会在执行时输出自己的编号和执行线程的名称，然后睡眠1秒钟模拟任务执行时间。最后，我们调用 shutdown() 方法关闭线程池。注意，在使用 Spring 框架中的 ThreadPoolTaskExecutor 类创建线程池时，我们需要先调用 initialize() 方法进行初始化。

> ThreadPoolTaskExecutor 底层还是通过 JDK 中提供的 ThreadPoolExecutor 类实现的。

#### [#](#线程池优点详解) 线程池优点详解

线程池相比于线程来说，它不需要频繁的创建和销毁线程，线程一旦创建之后，默认情况下就会一直保持在线程池中，等到有任务来了，再用这些已有的线程来执行任务，如下图所示：![image.png](https://javacn.site/image/1643163207461-87b04d2a-b8fb-4a18-9b39-85bea0cd6e9a.png)

#### [#](#优点1-复用线程-降低资源消耗) 优点1：复用线程，降低资源消耗

线程在创建时要开辟虚拟机栈、本地方法栈、程序计数器等私有线程的内存空间，而销毁时又要回收这些私有空间资源，如下图所示： ![image.png](https://javacn.site/image/1643164009933-de38fae0-2e8b-474d-8ada-1c843cedf602.png) 而线程池创建了线程之后就会放在线程池中，因此线程池相比于线程来说，第一个优点就是**可以复用线程、减低系统资源的消耗**。

#### [#](#优点2-提高响应速度) 优点2：提高响应速度

线程池是复用已有线程来执行任务的，而线程是在有任务时才新建的，所以相比于线程来说，线程池能够更快的响应任务和执行任务。

#### [#](#优点3-管控线程数和任务数) 优点3：管控线程数和任务数

线程池提供了更多的管理功能，这里管理功能主要体现在以下两个方面：

1. 控制最大并发数：线程池可以创建固定的线程数，从而避免了无限创建线程的问题。当线程创建过多时，会导致系统执行变慢，因为 CPU 核数是一定的、能同时处理的任务数也是一定的，而线程过多时就会造成线程恶意争抢和线程频繁切换的问题，从而导致程序执行变慢，所以合适的线程数才是高性能运行的关键。
2. 控制任务最大数：如果任务无限多，而内存又不足的情况下，就会导致程序执行报错，而线程池可以控制最大任务数，当任务超过一定数量之后，就会采用拒绝策略来处理多出的任务，从而保证了系统可以健康的运行。

#### [#](#优点4-更多增强功能) 优点4：更多增强功能

线程池相比于线程来说提供了更多的功能，比如定时执行和周期执行等功能。

#### [#](#小结) 小结

线程池是一种管理和复用线程资源的机制。相比于线程，它具备四个主要优势：1.复用线程，降低了资源消耗；2.提高响应速度；3.提供了管理线程数和任务数的能力；4.更多增强功能。

### 9.说一下线程池参数的意义？

问的线程池的参数含义，一定指的是手动创建线程池 ThreadPoolExecutor 参数的含义。 ThreadPoolExecutor 最多包含 7 个参数，如以下源码所示：



```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    //...
}
```

这 7 个参数分别是：

1. corePoolSize：核心线程数。
2. maximumPoolSize：最大线程数。
3. keepAliveTime：空闲线程存活时间。
4. TimeUnit：时间单位。
5. BlockingQueue：线程池任务队列。
6. ThreadFactory：创建线程的工厂。
7. RejectedExecutionHandler：拒绝策略。

#### [#](#参数1-corepoolsize) 参数1：corePoolSize

**核心线程数：是指线程池中长期存活的线程数。**

> 这就好比古代大户人家，会长期雇佣一些“长工”来给他们干活，这些人一般比较稳定，无论这一年的活多活少，这些人都不会被辞退，都是长期生活在大户人家的。

#### [#](#参数2-maximumpoolsize) 参数2：maximumPoolSize

**最大线程数：线程池允许创建的最大线程数量，当线程池的任务队列满了之后，可以创建的最大线程数。**

> 这是古代大户人家最多可以雇佣的人数，比如某个节日或大户人家有人过寿时，因为活太多，仅靠“长工”是完不成任务，这时就会再招聘一些“短工”一起来干活，这个最大线程数就是“长工”+“短工”的总人数，也就是招聘的人数不能超过 maximumPoolSize。

#### [#](#注意事项) 注意事项

最大线程数 maximumPoolSize 的值不能小于核心线程数 corePoolSize，否则在程序运行时会报 IllegalArgumentException 非法参数异常，如下图所示： ![image.png](https://javacn.site/image/1643183657709-dc15ad1c-52c8-4b00-b04a-38c5a342ec98.png)

#### [#](#参数3-keepalivetime) 参数3：keepAliveTime

**空闲线程存活时间，当线程池中没有任务时，会销毁一些线程，销毁的线程数=maximumPoolSize（最大线程数）-corePoolSize（核心线程数）。**

> 还是以大户人家为例，当大户人家比较忙的时候就会雇佣一些“短工”来干活，但等干完活之后，不忙了，就会将这些“短工”辞退掉，而 keepAliveTime 就是用来描述没活之后，短工可以在大户人家待的（最长）时间。

#### [#](#参数4-timeunit) 参数4：TimeUnit

**时间单位：空闲线程存活时间的描述单位**，此参数是配合参数 3 使用的。 参数 3 是一个 long 类型的值，比如参数 3 传递的是 1，那么这个 1 表示的是 1 天？还是 1 小时？还是 1 秒钟？是由参数 4 说了算的。 TimeUnit 有以下 7 个值：

1. TimeUnit.DAYS：天
2. TimeUnit.HOURS：小时
3. TimeUnit.MINUTES：分
4. TimeUnit.SECONDS：秒
5. TimeUnit.MILLISECONDS：毫秒
6. TimeUnit.MICROSECONDS：微妙
7. TimeUnit.NANOSECONDS：纳秒

#### [#](#参数5-blockingqueue) 参数5：BlockingQueue

**阻塞队列：线程池存放任务的队列，用来存储线程池的所有待执行任务。** 它可以设置以下几个值：

1. ArrayBlockingQueue：一个由数组结构组成的有界阻塞队列。
2. LinkedBlockingQueue：一个由链表结构组成的有界阻塞队列。
3. SynchronousQueue：一个不存储元素的阻塞队列，即直接提交给线程不保持它们。
4. PriorityBlockingQueue：一个支持优先级排序的无界阻塞队列。
5. DelayQueue：一个使用优先级队列实现的无界阻塞队列，只有在延迟期满时才能从中提取元素。
6. LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。与SynchronousQueue类似，还含有非阻塞方法。
7. LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

比较常用的是 LinkedBlockingQueue，线程池的排队策略和 BlockingQueue 息息相关。

#### [#](#参数6-threadfactory) 参数6：ThreadFactory

**线程工厂：线程池创建线程时调用的工厂方法，通过此方法可以设置线程的优先级、线程命名规则以及线程类型（用户线程还是守护线程）等。** 线程工厂的使用示例如下：



```java
public static void main(String[] args) {
    // 创建线程工厂
    ThreadFactory threadFactory = new ThreadFactory() {
        @Override
        public Thread newThread(Runnable r) {
            // 创建线程池中的线程
            Thread thread = new Thread(r);
            // 设置线程名称
            thread.setName("Thread-" + r.hashCode());
            // 设置线程优先级（最大值：10）
            thread.setPriority(Thread.MAX_PRIORITY);
            //......
            return thread;
        }
    };
    // 创建线程池
    ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(10, 10, 0,
                                                                   TimeUnit.SECONDS, new LinkedBlockingQueue<>(),
                                                                   threadFactory); // 使用自定义的线程工厂
    threadPoolExecutor.submit(new Runnable() {
        @Override
        public void run() {
            Thread thread = Thread.currentThread();
            System.out.println(String.format("线程：%s，线程优先级：%d",
                                             thread.getName(), thread.getPriority()));
        }
    });
}
```

以上程序的执行结果如下：![image.png](https://javacn.site/image/1643185011046-3e5f7f49-7c50-42aa-8b3b-a1c2ed21ed47.png) 从上述执行结果可以看出，自定义线程工厂起作用了，线程的名称和线程的优先级都是通过线程工厂设置的。

#### [#](#参数7-rejectedexecutionhandler) 参数7：RejectedExecutionHandler

**拒绝策略：当线程池的任务超出线程池队列可以存储的最大值之后，执行的策略。** 默认的拒绝策略有以下 4 种：

- AbortPolicy：拒绝并抛出异常。
- CallerRunsPolicy：使用当前调用的线程来执行此任务。
- DiscardOldestPolicy：抛弃队列头部（最旧）的一个任务，并执行当前任务。
- DiscardPolicy：忽略并抛弃当前任务。

线程池的默认策略是 AbortPolicy 拒绝并抛出异常。

#### [#](#小结) 小结

本文介绍了线程池的 7 大参数：

1. corePoolSize：核心线程数，线程池正常情况下保持的线程数，大户人家“长工”的数量。
2. maximumPoolSize：最大线程数，当线程池繁忙时最多可以拥有的线程数，大户人家“长工”+“短工”的总数量。
3. keepAliveTime：空闲线程存活时间，没有活之后“短工”可以生存的最大时间。
4. TimeUnit：时间单位，配合参数 3 一起使用，用于描述参数 3 的时间单位。
5. BlockingQueue：线程池的任务队列，用于保存线程池待执行任务的容器。
6. ThreadFactory：线程工厂，用于创建线程池中线程的工厂方法，通过它可以设置线程的命名规则、优先级和线程类型。
7. RejectedExecutionHandler：拒绝策略，当任务量超过线程池可以保存的最大任务数时，执行的策略。

### 10.线程池如何执行，拒绝策略有哪些？

要搞懂线程池的执行流程，最好的方式是去看它的源码，它的源码如下：



```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    int c = ctl.get();
    // 当前工作的线程数小于核心线程数
    if (workerCountOf(c) < corePoolSize) {
        // 创建新的线程执行此任务
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    // 检查线程池是否处于运行状态，如果是则把任务添加到队列
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        // 再次检线程池是否处于运行状态，防止在第一次校验通过后线程池关闭
        // 如果是非运行状态，则将刚加入队列的任务移除
        if (! isRunning(recheck) && remove(command))
            reject(command);
        // 如果线程池的线程数为 0 时（当 corePoolSize 设置为 0 时会发生）
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false); // 新建线程执行任务
    }
    // 核心线程都在忙且队列都已爆满，尝试新启动一个线程执行失败
    else if (!addWorker(command, false)) 
        // 执行拒绝策略
        reject(command);
}
```

从上述源码我们可以看出，当任务来了之后，**线程池的执行流程是：先判断当前线程数是否大于核心线程数？如果结果为 false，则新建线程并执行任务；如果结果为 true，则判断任务队列是否已满？如果结果为 false，则把任务添加到任务队列中等待线程执行，否则则判断当前线程数量是否超过最大线程数？如果结果为 false，则新建线程执行此任务，否则将执行线程池的拒绝策略**，如下图所示：![image.png](https://javacn.site/image/1643197315390-fc6734df-ee2d-4c80-b623-d9b48958ba69.png)

#### [#](#线程池拒绝策略) 线程池拒绝策略

当任务过多且线程池的任务队列已满时，此时就会执行线程池的拒绝策略，线程池的拒绝策略默认有以下 4 种：

1. AbortPolicy：中止策略，线程池会抛出异常并中止执行此任务；
2. CallerRunsPolicy：把任务交给添加此任务的（main）线程来执行；
3. DiscardPolicy：忽略此任务，忽略最新的一个任务；
4. DiscardOldestPolicy：忽略最早的任务，最先加入队列的任务。

默认的拒绝策略为 AbortPolicy 中止策略。

#### [#](#discardpolicy拒绝策略) DiscardPolicy拒绝策略

接下来我们以 DiscardPolicy 忽略此任务，忽略最新的一个任务为例，演示一下拒绝策略的具体使用，实现代码如下：



```java
public static void main(String[] args) {
    // 任务的具体方法
    Runnable runnable = new Runnable() {
        @Override
        public void run() {
            System.out.println("当前任务被执行,执行时间:" + new Date() +
                               " 执行线程:" + Thread.currentThread().getName());
            try {
                // 等待 1s
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    };
    // 创建线程,线程的任务队列的长度为 1
    ThreadPoolExecutor threadPool = new ThreadPoolExecutor(1, 1,
                                                           100, TimeUnit.SECONDS, new LinkedBlockingQueue<>(1),
                                                           new ThreadPoolExecutor.DiscardPolicy());
    // 添加并执行 4 个任务
    threadPool.execute(runnable);
    threadPool.execute(runnable);
    threadPool.execute(runnable);
    threadPool.execute(runnable);
    // 线程池执行完任务，关闭线程池
    threadPool.shutdown();
}
```

以上程序的执行结果如下： ![image.png](https://javacn.site/image/1643198263954-9240ff07-21c7-42f9-8322-b91a1db646b4.png)从上述执行结果可以看出，给线程池添加了 4 个任务，而线程池只执行了 2 个任务就结束了，其他两个任务执行了拒绝策略 DiscardPolicy 被忽略了，这就是拒绝策略的作用。

#### [#](#abortpolicy拒绝策略) AbortPolicy拒绝策略

为了和 DiscardPolicy 拒绝策略对比，我们来演示一下 JDK 默认的拒绝策略 AbortPolicy 中止策略，线程池会抛出异常并中止执行此任务，示例代码如下：



```java
public static void main(String[] args) {
    // 任务的具体方法
    Runnable runnable = new Runnable() {
        @Override
        public void run() {
            System.out.println("当前任务被执行,执行时间:" + new Date() +
                               " 执行线程:" + Thread.currentThread().getName());
            try {
                // 等待 1s
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    };
    // 创建线程,线程的任务队列的长度为 1
    ThreadPoolExecutor threadPool = new ThreadPoolExecutor(1, 1,
                                                           100, TimeUnit.SECONDS, new LinkedBlockingQueue<>(1),
                                                           new ThreadPoolExecutor.AbortPolicy()); // 显式指定拒绝策略，也可以忽略此设置，它为默认拒绝策略
    // 添加并执行 4 个任务
    threadPool.execute(runnable);
    threadPool.execute(runnable);
    threadPool.execute(runnable);
    threadPool.execute(runnable);
    // 线程池执行完任务，关闭线程池
    threadPool.shutdown();
}
```

以上程序的执行结果如下： ![image.png](https://javacn.site/image/1643198623428-413dd4ce-77a2-4d87-83ae-50a4d034c955.png) 从结果可以看出，给线程池添加了 4 个任务，线程池正常执行了 2 个任务，其他两个任务执行了中止策略，并抛出了拒绝执行的异常 RejectedExecutionException。

#### [#](#自定义拒绝策略) 自定义拒绝策略

当然除了 JDK 提供的四种拒绝策略之外，我们还可以实现通过 new RejectedExecutionHandler，并重写 rejectedExecution 方法来实现自定义拒绝策略，实现代码如下：



```java
public static void main(String[] args) {
    // 任务的具体方法
    Runnable runnable = new Runnable() {
        @Override
        public void run() {
            System.out.println("当前任务被执行,执行时间:" + new Date() +
                               " 执行线程:" + Thread.currentThread().getName());
            try {
                // 等待 1s
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    };
    // 创建线程,线程的任务队列的长度为 1
    ThreadPoolExecutor threadPool = new ThreadPoolExecutor(1, 1,
                                                           100, TimeUnit.SECONDS, new LinkedBlockingQueue<>(1),
                                                           new RejectedExecutionHandler() {
                                                               @Override
                                                               public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
                                                                   // 执行自定义拒绝策略的相关操作
                                                                   System.out.println("我是自定义拒绝策略~");
                                                               }
                                                           });
    // 添加并执行 4 个任务
    threadPool.execute(runnable);
    threadPool.execute(runnable);
    threadPool.execute(runnable);
    threadPool.execute(runnable);
}
```

以上程序的执行结果如下：![image.png](https://javacn.site/image/1643197939009-1ce5f5b8-82c0-4184-884f-6b90eee8a6af.png)

#### [#](#小结) 小结

线程池的执行流程有 3 个重要的判断点（判断顺序依次往后）：判断当前线程数和核心线程数、判断当前任务队列是否已满、判断当前线程数是否已达到最大线程数。如果经过以上 3 个判断，得到的结果都会 true，则会执行线程池的拒绝策略。JDK 提供了 4 种拒绝策略，我们还可以通过 new RejectedExecutionHandler 并重写 rejectedExecution 方法来实现自定义拒绝策略。

### 11.如何停止线程池？

在 Java 中，停止线程池可以通过以下两个步骤来实现：

1. 调用方法停止线程池： 
   1. 调用线程池的 shutdown() 方法来关闭线程池。该方法会停止线程池的接受新任务，并尝试将所有未完成的任务完成执行；
   2. 调用线程池的 shutdownNow() 方法来关闭线程池。该方法会停止线程池的接受新任务，并尝试停止所有正在执行的任务。该方法会返回一个未完成任务的列表，这些任务将被取消。
2. 等待线程池停止：在关闭线程池后，通过调用 awaitTermination() 方法来等待所有任务完成执行。该方法会阻塞当前线程，直到所有任务完成执行或者等待超时。

下面是一个示例代码，演示如何中止线程池：



```java
ExecutorService executor = Executors.newFixedThreadPool(10);
// 提交任务到线程池
for (int i = 0; i < 100; i++) {
    executor.submit(new MyTask());
}
// 关闭线程池
executor.shutdown();
try {
    // 等待所有任务完成执行
    if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
        // 如果等待超时，强制关闭线程池
        executor.shutdownNow();
    }
} catch (InterruptedException e) {
    // 处理异常
}
```

在上面的示例代码中，首先创建了一个线程池，然后提交了 100 个任务到线程池中。然后，通过调用 shutdown() 方法关闭线程池，再通过调用 awaitTermination() 方法等待所有任务完成执行。如果等待超时，将强制调用 shutdownNow() 方法来停止所有正在执行的任务。最后，在 catch 块中处理中断异常。















