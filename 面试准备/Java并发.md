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

### 12.如何判断线程池任务已执行完？

无论是在项目开发中，还是在面试中过程中，总会被问到或使用到并发编程来完成项目中的某个功能。

例如某个复杂的查询，无法使用一个查询语句来完成此功能，此时我们就需要执行多个查询语句，然后再将各自查询的结果，组装之后返回给前端了，那么这种场景下，我们就必须使用线程池来进行并发查询了。

> PS：磊哥做的最复杂的查询，总共关联了 21 张表，在和产品及需求方的多次沟通下，才将查询的业务从 21 张表，降到了至少要查询 12 张表（非常难搞），那么这种场景下是无法使用一个查询语句来实现的，那么并发查询是必须要给安排上的。

#### [#](#_1-需求分析) 1.需求分析

线程池的使用并不复杂，麻烦的是如何判断线程池中的任务已经全部执行完了？因为我们要等所有任务都执行完之后，才能进行数据的组装和返回，所以接下来，我们就来看如何判断线程中的任务是否已经全部执行完？

#### [#](#_2-实现概述) 2.实现概述

判断线程池中的任务是否执行完的方法有很多，比如以下几个：

1. 使用 getCompletedTaskCount() 统计已经执行完的任务，和 getTaskCount() 线程池的总任务进行对比，如果相等则说明线程池的任务执行完了，否则既未执行完。
2. 使用 FutureTask 等待所有任务执行完，线程池的任务就执行完了。
3. 使用 CountDownLatch 或 CyclicBarrier 等待所有线程都执行完之后，再执行后续流程。

具体实现代码如下。

#### [#](#_3-具体实现) 3.具体实现

#### [#](#_3-1-统计完成任务数) 3.1 统计完成任务数

通过判断线程池中的计划执行任务数和已完成任务数，来判断线程池是否已经全部执行完，如果**计划执行任务数=已完成任务数**，那么线程池的任务就全部执行完了，否则就未执行完。

示例代码如下：



```java
private static void isCompletedByTaskCount(ThreadPoolExecutor threadPool) {
    while (threadPool.getTaskCount() != threadPool.getCompletedTaskCount()) {
    }
}
```

以上程序执行结果如下：

![img](https://javacn.site/image/1690946153714-05a562a0-2075-47e6-b9c9-72ddc8893347.png)

**方法说明**

- getTaskCount()：返回计划执行的任务总数。由于任务和线程的状态可能在计算过程中动态变化，因此返回的值只是一个近似值。
- getCompletedTaskCount()：返回完成执行任务的总数。因为任务和线程的状态可能在计算过程中动态地改变，所以返回的值只是一个近似值，但是在连续的调用中并不会减少。

**缺点分析**

此判断方法的缺点是 getTaskCount() 和 getCompletedTaskCount() 返回的是一个近似值，因为线程池中的任务和线程的状态可能在计算过程中动态变化，所以它们两个返回的都是一个近似值。

#### [#](#_3-2-futuretask) 3.2 FutureTask

FutrueTask 的优势是任务判断精准，调用每个 FutrueTask 的 get 方法就是等待该任务执行完，如下代码所示：



```java
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.FutureTask;

/**
 * 使用 FutrueTask 等待线程池执行完全部任务
 */
public class FutureTaskDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 创建一个固定大小的线程池
        ExecutorService executor = Executors.newFixedThreadPool(3);
        // 创建任务
        FutureTask<Integer> task1 = new FutureTask<>(() -> {
            System.out.println("Task 1 start");
            Thread.sleep(2000);
            System.out.println("Task 1 end");
            return 1;
        });
        FutureTask<Integer> task2 = new FutureTask<>(() -> {
            System.out.println("Task 2 start");
            Thread.sleep(3000);
            System.out.println("Task 2 end");
            return 2;
        });
        FutureTask<Integer> task3 = new FutureTask<>(() -> {
            System.out.println("Task 3 start");
            Thread.sleep(1500);
            System.out.println("Task 3 end");
            return 3;
        });
        // 提交三个任务给线程池
        executor.submit(task1);
        executor.submit(task2);
        executor.submit(task3);

        // 等待所有任务执行完毕并获取结果
        int result1 = task1.get();
        int result2 = task2.get();
        int result3 = task3.get();
        System.out.println("Do main thread.");
    }
}
```

以上程序的执行结果如下：

![img](https://javacn.site/image/1690946467682-939e4921-d168-4bc1-8aec-de9625c783cd.png)

#### [#](#_3-3-countdownlatch和cyclicbarrier) 3.3 CountDownLatch和CyclicBarrier

CountDownLatch 和 CyclicBarrier 类似，都是等待所有任务到达某个点之后，再进行后续的操作，如下图所示：

![img](https://javacn.site/image/1642477854879-c3ff68ae-0917-4482-8c05-1d97d718b5cf.gif)

CountDownLatch 使用的示例代码如下：



```java
public static void main(String[] args) throws InterruptedException {
    // 创建线程池
    ThreadPoolExecutor threadPool = new ThreadPoolExecutor(10, 20,
    	0, TimeUnit.SECONDS, new LinkedBlockingDeque<>(1024));
    final int taskCount = 5;    // 任务总数
    // 单次计数器
    CountDownLatch countDownLatch = new CountDownLatch(taskCount); // ①
    // 添加任务
    for (int i = 0; i < taskCount; i++) {
        final int finalI = i;
        threadPool.submit(new Runnable() {
            @Override
            public void run() {
                try {
                    // 随机休眠 0-4s
                    int sleepTime = new Random().nextInt(5);
                    TimeUnit.SECONDS.sleep(sleepTime);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(String.format("任务%d执行完成", finalI));
                // 线程执行完，计数器 -1
                countDownLatch.countDown();  // ②
            }
        });
    }
    // 阻塞等待线程池任务执行完
    countDownLatch.await();  // ③
    // 线程池执行完
    System.out.println();
    System.out.println("线程池任务执行完成！");
}
```

> 代码说明：以上代码中标识为 ①、②、③ 的代码行是核心实现代码，其中：
>
> ① 是声明一个包含了 5 个任务的计数器；
>
> ② 是每个任务执行完之后计数器 -1；
>
> ③ 是阻塞等待计数器 CountDownLatch 减为 0，表示任务都执行完了，可以执行 await 方法后面的业务代码了。

以上程序的执行结果如下：

![img](https://javacn.site/image/1642477077446-3921af47-848d-4a53-a5a2-58e571cc03ce.png)

**缺点分析**

CountDownLatch 缺点是计数器只能使用一次，CountDownLatch 创建之后不能被重复使用。

CyclicBarrier 和 CountDownLatch 类似，它可以理解为一个可以重复使用的循环计数器，CyclicBarrier 可以调用 reset 方法将自己重置到初始状态，CyclicBarrier 具体实现代码如下：



```java
public static void main(String[] args) throws InterruptedException {
    // 创建线程池
    ThreadPoolExecutor threadPool = new ThreadPoolExecutor(10, 20,
    	0, TimeUnit.SECONDS, new LinkedBlockingDeque<>(1024));
    final int taskCount = 5;    // 任务总数
    // 循环计数器 ①
    CyclicBarrier cyclicBarrier = new CyclicBarrier(taskCount, new Runnable() {
        @Override
        public void run() {
            // 线程池执行完
            System.out.println();
            System.out.println("线程池所有任务已执行完！");
        }
    });
    // 添加任务
    for (int i = 0; i < taskCount; i++) {
        final int finalI = i;
        threadPool.submit(new Runnable() {
            @Override
            public void run() {
                try {
                    // 随机休眠 0-4s
                    int sleepTime = new Random().nextInt(5);
                    TimeUnit.SECONDS.sleep(sleepTime);
                    System.out.println(String.format("任务%d执行完成", finalI));
                    // 线程执行完
                    cyclicBarrier.await(); // ②
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }
        });
    }
}
```

以上程序的执行结果如下：

![img](https://javacn.site/image/1642478691437-1d4a9b32-dc55-4439-a818-bc5374993324.png)

**方法说明**

CyclicBarrier 有 3 个重要的方法：

1. 构造方法：构造方法可以传递两个参数，参数 1 是计数器的数量 parties，参数 2 是计数器为 0 时，也就是任务都执行完之后可以执行的事件（方法）。
2. await 方法：在 CyclicBarrier 上进行阻塞等待，当调用此方法时 CyclicBarrier 的内部计数器会 -1，直到发生以下情形之一： 
   1. 在 CyclicBarrier 上等待的线程数量达到 parties，也就是计数器的声明数量时，则所有线程被释放，继续执行。
   2. 当前线程被中断，则抛出 InterruptedException 异常，并停止等待，继续执行。
   3. 其他等待的线程被中断，则当前线程抛出 BrokenBarrierException 异常，并停止等待，继续执行。
   4. 其他等待的线程超时，则当前线程抛出 BrokenBarrierException 异常，并停止等待，继续执行。
   5. 其他线程调用 CyclicBarrier.reset() 方法，则当前线程抛出 BrokenBarrierException 异常，并停止等待，继续执行。
3. reset 方法：使得CyclicBarrier回归初始状态，直观来看它做了两件事： 
   1. 如果有正在等待的线程，则会抛出 BrokenBarrierException 异常，且这些线程停止等待，继续执行。
   2. 将是否破损标志位 broken 置为 false。

**优缺点分析**

CyclicBarrier 从设计的复杂度到使用的复杂度都高于 CountDownLatch，相比于 CountDownLatch 来说它的优点是可以重复使用（只需调用 reset 就能恢复到初始状态），缺点是使用难度较高。

#### [#](#小结) 小结

在实现判断线程池任务是否执行完成的方案中，通过统计线程池执行完任务的方式（实现方法 1），以及实现方法 3（CountDownLatch 或 CyclicBarrier）等统计，都是“不记名”的，只关注数量，不关注（具体）对象，所以这些方式都有可能受到外界代码的影响，因此使用 FutureTask 等待具体任务执行完的方式是最推荐的判断方法。

### 13.什么是Volatile？

volatile 是一种关键字，用于保证多线程情况下共享变量的可见性。当一个变量被声明为 volatile 时，每个线程在访问该变量时都会立即刷新其本地内存（工作内存）中该变量的值，确保所有线程都能读到最新的值。并且使用 volatile 可以禁止指令重排序，这样就能有效的预防，因为指令优化（重排序）而导致的线程安全问题。

也就是说 volatile 有两个主要功能：保证内存可见性和禁止指令重排序。下来我们具体来看这两个功能。

#### [#](#内存可见性) 内存可见性

说到内存可见性问题就不得不提 Java 内存模型，Java 内存模型（Java Memory Model）简称为 JMM，主要是用来屏蔽不同硬件和操作系统的内存访问差异的，因为在不同的硬件和不同的操作系统下，内存的访问是有一定的差异得，这种差异会导致相同的代码在不同的硬件和不同的操作系统下有着不一样的行为，而 Java 内存模型就是解决这个差异，统一相同代码在不同硬件和不同操作系统下的差异的。

Java 内存模型规定：所有的变量（实例变量和静态变量）都必须存储在主内存中，每个线程也会有自己的工作内存，线程的工作内存保存了该线程用到的变量和主内存的副本拷贝，线程对变量的操作都在工作内存中进行。线程不能直接读写主内存中的变量，如下图所示： ![image.png](https://javacn.site/image/1651323306848-34a76507-9603-4b9e-ad1f-8b7b9d4fd524.png) 然而，Java 内存模型会带来一个新的问题，那就是内存可见性问题，也就是当某个线程修改了主内存中共享变量的值之后，其他线程不能感知到此值被修改了，它会一直使用自己工作内存中的“旧值”，这样程序的执行结果就不符合我们的预期了，这就是内存可见性问题，我们用以下代码来演示一下这个问题：



```java
private static boolean flag = false;
public static void main(String[] args) {
    Thread t1 = new Thread(new Runnable() {
        @Override
        public void run() {
            while (!flag) {

            }
            System.out.println("终止执行");
        }
    });
    t1.start();
    Thread t2 = new Thread(new Runnable() {
        @Override
        public void run() {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("设置 flag=true");
            flag = true;
        }
    });
    t2.start();
}
```

以上代码我们预期的结果是，在线程 1 执行了 1s 之后，线程 2 将 flag 变量修改为 true，之后线程 1 终止执行，然而，因为线程 1 感知不到 flag 变量发生了修改，也就是内存可见性问题，所以会导致线程 1 会永远的执行下去，最终我们看到的结果是这样的： ![image.png](https://javacn.site/image/1651322607045-ed01b7ec-821e-4d1e-889b-c2673557f375.png) 如何解决以上问题呢？只需要给变量 flag 加上 volatile 修饰即可，具体的实现代码如下：



```java
private volatile static boolean flag = false;
public static void main(String[] args) {
    Thread t1 = new Thread(new Runnable() {
        @Override
        public void run() {
            while (!flag) {

            }
            System.out.println("终止执行");
        }
    });
    t1.start();
    Thread t2 = new Thread(new Runnable() {
        @Override
        public void run() {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("设置 flag=true");
            flag = true;
        }
    });
    t2.start();
}
```

以上程序的执行结果如下图所示：![image.png](https://javacn.site/image/1651322718252-6e6ad544-b048-4e14-9f98-0e982f02e343.png)

## [#](#禁止指令重排序) 禁止指令重排序

指令重排序是指编译器或 CPU 为了优化程序的执行性能，而对指令进行重新排序的一种手段。

指令重排序的实现初衷是好的，但是在多线程执行中，如果执行了指令重排序可能会导致程序执行出错。指令重排序最典型的一个问题就发生在单例模式中，比如以下问题代码：



```java
public class Singleton {
    private Singleton() {}
    private static Singleton instance = null;
    public static Singleton getInstance() {
        if (instance == null) { // ①
            synchronized (Singleton.class) {
            	if (instance == null) {
                	instance = new Singleton(); // ②
                }
            }
        }
        return instance;
    }
}
```

以上问题发生在代码 ② 这一行“instance = new Singleton();”，这行代码**看似只是一个创建对象的过程，然而它的实际执行却分为以下 3 步：**

1. **创建内存空间。**
2. **在内存空间中初始化对象 Singleton。**
3. **将内存地址赋值给 instance 对象（执行了此步骤，instance 就不等于 null 了）。**

**如果此变量不加 volatile，那么线程 1 在执行到上述代码的第 ② 处时就可能会执行指令重排序，将原本是 1、2、3 的执行顺序，重排为 1、3、2。但是特殊情况下，线程 1 在执行完第 3 步之后，如果来了线程 2 执行到上述代码的第 ① 处，判断 instance 对象已经不为 null，但此时线程 1 还未将对象实例化完，那么线程 2 将会得到一个被实例化“一半”的对象，从而导致程序执行出错，这就是为什么要给私有变量添加 volatile 的原因了。** 要使以上单例模式变为线程安全的程序，需要给 instance 变量添加 volatile 修饰，它的最终实现代码如下：



```java
public class Singleton {
    private Singleton() {}
    // 使用 volatile 禁止指令重排序
    private static volatile Singleton instance = null; // 【主要是此行代码发生了变化】
    public static Singleton getInstance() {
        if (instance == null) { // ①
            synchronized (Singleton.class) {
            	if (instance == null) {
                	instance = new Singleton(); // ②
                }
            }
        }
        return instance;
    }
}
```

## [#](#小结) 小结

volatile 是 Java 并发编程的重要组成部分，它的主要作用有两个：保证内存的可见性和禁止指令重排序。volatile 常使用在一写多读的场景中，比如 CopyOnWriteArrayList 集合，它在操作的时候会把全部数据复制出来对写操作加锁，修改完之后再使用 setArray 方法把此数组赋值为更新后的值，使用 volatile 可以使读线程很快的告知到数组被修改，不会进行指令重排，操作完成后就可以对其他线程可见了。

### 14.volatile底层是如何实现的？

在 Java 中，volatile 是一种关键字，用于修饰变量。使用 volatile 关键字修饰的变量具有可见性和有序性，但不保证原子性。

#### [#](#相关定义说明) 相关定义说明

原子性（Atomicity）：即一个操作或者多个操作，要么全部执行，并且执行的过程不会被任何因素打断，要么都不执行。

有序性（Ordering）：指指令在执行过程中的顺序，一个操作执行在另一个操作之前或者在其执行之后。即程序执行的顺序按照代码的先后顺序执行。

可见性（Visibility）：指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

#### [#](#volatile-实现原理) volatile 实现原理

volatile 关键字在底层的实现主要是通过内存屏障（memory barrier）来实现的。内存屏障是一种 CPU 指令，用于强制执行 CPU 的内部缓存与主内存之间的数据同步。

在 Java 中，当线程读取一个 volatile 变量时，会从主内存中读取变量的最新值，并把它存储到线程的工作内存中。当线程写入一个 volatile 变量时，会把变量的值写入到线程的工作内存中，并强制将这个值刷新到主内存中。这样就保证了 volatile 变量的可见性和有序性。

在 Java 5 之后，volatile 的实现还引入了“内存屏障插入”的机制，内存屏障插入是指在指令序列中插入内存屏障以保证变量的可见性和有序性。

#### [#](#主内存和工作内存) 主内存和工作内存

Java 内存模型规定，所有的变量（实例变量和静态变量）都必须存储在主内存中，每个线程也会有自己的工作内存，线程的工作内存保存了该线程用到的变量和主内存的副本拷贝，线程对变量的操作都在工作内存中进行。线程不能直接读写主内存中的变量，如下图所示： ![image.png](https://javacn.site/image/1651323306848-34a76507-9603-4b9e-ad1f-8b7b9d4fd524.png) 这样设计的目的主要是为了提升程序的并发性能以及多线程之间的可见性问题。

主内存是 Java 虚拟机中的一块共享内存，所有线程都可以访问。而每个线程还有自己的工作内存，线程的工作内存中存储了主内存中的变量副本的拷贝。这样做的好处是，线程之间不需要同步所有变量的读写操作，只需要同步主内存中的变量即可，这样可以提高程序的执行效率。同时，由于每个线程都有自己的工作内存，因此线程之间的变量操作互相不影响，从而提高了程序的并发性能。

#### [#](#内存屏障) 内存屏障

内存屏障是一种硬件机制，用于控制 CPU 缓存和主内存之间的数据同步。在 Java 中，内存屏障通常有两种：读屏障和写屏障。

在有内存屏障的地方，会禁止指令重排序，即屏障下面的代码不能跟屏障上面的代码交换执行顺序。在有内存屏障的地方，线程修改完共享变量以后会马上把该变量从本地内存写回到主内存，并且让其他线程本地内存中该变量副本失效（使用 MESI 协议）。

MESI 协议是一种缓存一致性协议，它是支持写回（write-back）缓存的最常用协议。MESI 协议基于总线嗅探机制实现了事务串形化，也用状态机机制降低了总线带宽压力，做到了 CPU 缓存一致性。MESI 协议这 4 个字母代表 4 个状态，分别是：Modified（已修改）、Exclusive（独占）、Shared（共享）、Invalidated（已失效）。

### 15.为什么单例一定要加volatile？

单例模式的实现方法有很多种，如饿汉模式、懒汉模式、静态内部类和枚举等，当面试官问到“为什么单例模式一定要加 volatile？”时，那么他指的是为什么懒汉模式中的私有变量要加 volatile？

> 懒汉模式指的是对象的创建是懒加载的方式，并不是在程序启动时就创建对象，而是第一次被真正使用时才创建对象。

要解释为什么要加 volatile？我们先来看懒汉模式的具体实现代码：



```java
public class Singleton {
    // 1.防止外部直接 new 对象破坏单例模式
    private Singleton() {}
    // 2.通过私有变量保存单例对象【添加了 volatile 修饰】
    private static volatile Singleton instance = null;
    // 3.提供公共获取单例对象的方法
    public static Singleton getInstance() {
        if (instance == null) { // 第 1 次效验
            synchronized (Singleton.class) {
            	if (instance == null) { // 第 2 次效验
                	instance = new Singleton(); 
                }
            }
        }
        return instance;
    }
}
```

从上述代码可以看出，为了保证线程安全和高性能，代码中使用了两次 if 和 synchronized 来保证程序的执行。那既然已经有 synchronized 来保证线程安全了，为什么还要给变量加 volatile 呢？

在解释这个问题之前，我们先要搞懂一个前置知识：volatile 有什么用呢？

## [#](#_1-volatile-作用) 1.volatile 作用

volatile 有两个主要的作用，第一，解决内存可见性问题，第二，防止指令重排序。

#### [#](#_1-1-内存可见性问题) 1.1 内存可见性问题

**所谓内存可见性问题，指的是多个线程同时操作一个变量，其中某个线程修改了变量的值之后，其他线程感知不到变量的修改，这就是内存可见性问题。****而使用 volatile 就可以解决内存可见性问题**，比如以下代码，当没有添加 volatile 时，它的实现如下：



```java
private static boolean flag = false;
public static void main(String[] args) {
    Thread t1 = new Thread(new Runnable() {
        @Override
        public void run() {
            // 如果 flag 变量为 true 就终止执行
            while (!flag) {

            }
            System.out.println("终止执行");
        }
    });
    t1.start();
    // 1s 之后将 flag 变量的值修改为 true
    Thread t2 = new Thread(new Runnable() {
        @Override
        public void run() {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("设置 flag 变量的值为 true！");
            flag = true;
        }
    });
    t2.start();
}
```

以上程序的执行结果如下： ![image.png](https://javacn.site/image/1650458547207-82d75caf-c3a0-4934-a83e-af74bb269a1d.png) 然而，以上程序执行了 N 久之后，依然没有结束执行，这说明线程 2 在修改了 flag 变量之后，线程 1 根本没有感知到变量的修改。 那么接下来，我们尝试给 flag 加上 volatile，实现代码如下：



```java
public class volatileTest {
    private static volatile boolean flag = false;
    public static void main(String[] args) {
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                // 如果 flag 变量为 true 就终止执行
                while (!flag) {

                }
                System.out.println("终止执行");
            }
        });
        t1.start();
        // 1s 之后将 flag 变量的值修改为 true
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("设置 flag 变量的值为 true！");
                flag = true;
            }
        });
        t2.start();
    }
}
```

以上程序的执行结果如下： ![image.png](https://javacn.site/image/1650458765573-e1fb8e93-21cb-4b17-9e55-a8021bb1aeb4.png) 从上述执行结果我们可以看出，使用 volatile 之后就可以解决程序中的内存可见性问题了。

#### [#](#_1-2-防止指令重排序) 1.2 防止指令重排序

指令重排序是指在程序执行过程中，编译器或 JVM 常常会对指令进行重新排序，已提高程序的执行性能。 指令重排序的设计初衷确实很好，在单线程中也能发挥很棒的作用，然而在多线程中，使用指令重排序就可能会导致线程安全问题了。

> 所谓线程安全问题是指程序的执行结果，和我们的预期不相符。比如我们预期的正确结果是 0，但程序的执行结果却是 1，那么这就是线程安全问题。

而使用 volatile 可以禁止指令重排序，从而保证程序在多线程运行时能够正确执行。

#### [#](#_2-为什么要用-volatile) 2.为什么要用 volatile？

回到主题，我们**在单例模式中使用 volatile，主要是使用 volatile 可以禁止指令重排序，从而保证程序的正常运行**。这里可能会有读者提出疑问，不是已经使用了 synchronized 来保证线程安全吗？那为什么还要再加 volatile 呢？看下面的代码：



```java
public class Singleton {
    private Singleton() {}
    // 使用 volatile 禁止指令重排序
    private static volatile Singleton instance = null;
    public static Singleton getInstance() {
        if (instance == null) { // ①
            synchronized (Singleton.class) {
            	if (instance == null) {
                	instance = new Singleton(); // ②
                }
            }
        }
        return instance;
    }
}
```

注意观察上述代码，我标记了第 ① 处和第 ② 处的两行代码。给私有变量加 volatile 主要是为了防止第 ② 处执行时，也就是“instance = new Singleton()”执行时的指令重排序的，这行代码**看似只是一个创建对象的过程，然而它的实际执行却分为以下 3 步：**

1. **创建内存空间。**
2. **在内存空间中初始化对象 Singleton。**
3. **将内存地址赋值给 instance 对象（执行了此步骤，instance 就不等于 null 了）。**

试想一下，**如果不加 volatile，那么线程 1 在执行到上述代码的第 ② 处时就可能会执行指令重排序，将原本是 1、2、3 的执行顺序，重排为 1、3、2。但是特殊情况下，线程 1 在执行完第 3 步之后，如果来了线程 2 执行到上述代码的第 ① 处，判断 instance 对象已经不为 null，但此时线程 1 还未将对象实例化完，那么线程 2 将会得到一个被实例化“一半”的对象，从而导致程序执行出错，这就是为什么要给私有变量添加 volatile 的原因了。**

#### [#](#小结) 小结

使用 volatile 可以解决内存可见性问题和防止指令重排序，我们在单例模式中使用 volatile 主要是使用 volatile 的后一个特性（防止指令重排序），从而避免多线程执行的情况下，因为指令重排序而导致某些线程得到一个未被完全实例化的对象，从而导致程序执行出错的情况。

### 16.保证线程安全的手段有哪些？

在 Java 中，多线程并发操作同一个共享变量时，就可能会发生线程安全问题。 在 Java 中保证线程安全的常用手段有以下三个：

1. 使用锁机制：锁机制是一种用于控制多个线程对共享资源进行访问的机制。在 Java 中，锁机制主要有两种：synchronized 关键字和 Lock 接口。synchronized 关键字是 Java 中最基本的锁机制，它可以用来修饰方法或代码块，以实现对共享资源的互斥访问。而 Lock 接口是 Java5 中新增的一种锁机制，它提供了比 synchronized 更强大、更灵活的锁定机制，例如可重入锁、读写锁等；
2. 使用线程安全的容器：如 ConcurrentHashMap、Hashtable、Vector。需要注意的是，线程安全的容器底层通常也是使用锁机制实现的；
3. 使用本地变量：线程本地变量是一种特殊的变量，它只能被同一个线程访问。在 Java 中，线程本地变量可以通过 ThreadLocal 类来实现。每个 ThreadLocal 对象都可以存储一个线程本地变量，而且每个线程都有自己的一份线程本地变量副本，因此不同的线程之间互不干扰。

### 17.synchronized 和 Lock有什么区别？

synchronized 和 Lock 都是 Java 中用于实现线程同步的机制，它们都可以保证线程安全。

#### [#](#synchronized-介绍与使用) synchronized 介绍与使用

synchronized 可用来修饰普通方法、静态方法和代码块，当一个线程访问一个被 synchronized 修饰的方法或者代码块时，会自动获取该对象的锁，其他线程将会被阻塞，直到该线程执行完毕并释放锁。这样就保证了多个线程对共享资源的操作的互斥性，从而避免了数据的不一致性和线程安全问题。 synchronized 基本使用如下：



```java
public class SynchronizedDemo {
    private int count = 0;
    public synchronized void increment() {
        count++;
    }
    public synchronized int getCount() {
        return count;
    }
}
```

此时我们再使用多线程调用上面类的 increment 或 getCount 时，就不会出现线程安全问题了，如下代码所示：



```java
public class SynchronizedDemoTest {
    public static void main(String[] args) {
        SynchronizedDemo demo = new SynchronizedDemo();
        Runnable r = () -> {
            for (int i = 0; i < 1000; i++) {
                demo.increment();
            }
        };

        Thread t1 = new Thread(r);
        Thread t2 = new Thread(r);

        t1.start();
        t2.start();

        try {
            t1.join();
            t2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Count: " + demo.getCount());
    }
}
```

#### [#](#lock-介绍与使用) Lock 介绍与使用

Lock 是一种线程同步的机制，它与 synchronized 相似，可以用于控制对共享资源的访问。相比于 synchronized，Lock 的特点在于更加灵活，支持更多的操作。 Lock 接口定义了以下方法：

- lock()：获取锁，如果锁已被其他线程占用，则阻塞当前线程。
- tryLock()：尝试获取锁，如果锁已被其他线程占用，则返回 false，否则返回 true。
- tryLock(long timeout, TimeUnit unit)：尝试获取锁，在指定的时间范围内获取到锁则返回 true，否则返回 false。
- unlock()：释放锁。

相比于 synchronized，Lock 的优点在于：

- 粒度更细：synchronized 关键字只能对整个方法或代码块进行同步，而 Lock 可以对单个变量或对象进行同步。
- 支持公平锁：synchronized 不支持公平锁，而 Lock 可以通过构造函数指定锁是否是公平锁。
- 支持多个条件变量：Lock 可以创建多个条件变量，即多个等待队列。

Lock 的实现类有很多，比较常用的有 ReentrantLock 和 ReentrantReadWriteLock。 需要注意的是，使用 Lock 时需要手动获取和释放锁，否则会导致死锁等问题。因此，一般来说建议使用 try-finally 语句块来确保锁的正确释放。例如：



```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Counter {
    private int count = 0;
    private Lock lock = new ReentrantLock();

    public void increment() {
        // 加锁
        lock.lock();
        try {
            count++;
        } finally {
            // 释放锁
            lock.unlock();
        }
    }

    public void decrement() {
        // 加锁
        lock.lock();
        try {
            count--;
        } finally {
            // 释放锁
            lock.unlock();
        }
    }
    public int getCount() {
        return count;
    }
}
```

#### [#](#synchronized-vs-lock) synchronized VS Lock

synchronized 和 Lock 主要的区别有以下几个方面：

1. 锁的获取方式：synchronized 是隐式获取锁的，即在进入 synchronized 代码块或方法时自动获取锁，退出时自动释放锁；而 Lock 需要程序显式地获取锁和释放锁，即需要调用 lock() 方法获取锁，调用 unlock() 方法释放锁。
2. 锁的性质：synchronized 是可重入的互斥锁，即同一个线程可以多次获得同一把锁，而且锁的释放也只能由获得锁的线程来释放；Lock 可以是可重入的互斥锁，也可以是非可重入的互斥锁，还可以是读写锁。
3. 锁的粒度：synchronized 是以代码块和方法为单位进行加锁和解锁，而 Lock 可以精确地控制锁的范围，可以支持多个条件变量。
4. 性能：在低并发的情况下，synchronized 的性能优于 Lock，因为 Lock 需要显式地获取和释放锁，而 synchronized 是在 JVM 层面实现的；在高并发的情况下，Lock 的性能可能优于 synchronized，因为 Lock 可以更好地支持高并发和读写分离的场景。

总的来说，synchronized 的使用更加简单，但是在某些场景下会受到性能的限制；而 Lock 则更加灵活，可以更精确地控制锁的范围和条件变量，但是使用起来比较繁琐。需要根据具体的业务场景和性能需求来选择使用哪种锁机制。

### 18.synchronized 底层是如何实现的？

synchronized 是 Java 中最基本的锁机制，使用它可以实现对共享资源的互斥访问。当一个线程访问被 synchronized 修饰的方法或代码块时，它会自动获取锁，其他线程只能排队等待该线程释放锁。

#### [#](#底层实现—监视器) 底层实现—监视器

想要了解 synchronized 的底层实现，需要从 synchronized 生成的字节码说起，比如以下程序：



```java
public class App {
    public static void main(String[] args) {
        synchronized (App.class) {
            System.out.println("Hello World!");
        }
    }
}
```

它的字节码如下：

![image.png](https://javacn.site/image/1681808250453-1a30cc01-dc7d-4a6d-aa32-972bfc42a88e.png)

从上述结果可以看出，在 main 方法中多了一对 monitorenter 和 monitorexit 的指令，它们的含义是：

- monitorenter：表示进入监视器；
- monitorexit：表示退出监视器。

由此可知 synchronized 的底层是通过 Monitor（监视器）实现的。

#### [#](#监视器具体实现) 监视器具体实现

在 HotSpot 虚拟机中，监视器 Monitor 底层是由 C++实现的，它的实现对象是 ObjectMonitor。ObjectMonitor 结构体的实现源码如下：



```cpp
ObjectMonitor::ObjectMonitor() {  
  _header       = NULL;  
  _count       = 0;  
  _waiters      = 0,  
  _recursions   = 0;       //线程的重入次数
  _object       = NULL;  
  _owner        = NULL;    //标识拥有该monitor的线程
  _WaitSet      = NULL;    //等待线程组成的双向循环链表，_WaitSet是第一个节点
  _WaitSetLock  = 0 ;  
  _Responsible  = NULL ;  
  _succ         = NULL ;  
  _cxq          = NULL ;    //多线程竞争锁进入时的单向链表
  FreeNext      = NULL ;  
  _EntryList    = NULL ;    //_owner从该双向循环链表中唤醒线程结点，_EntryList是第一个节点
  _SpinFreq     = 0 ;  
  _SpinClock    = 0 ;  
  OwnerIsThread = 0 ;  
} 
```

在以上代码中有几个关键的属性：

- _count：记录该线程获取锁的次数（也就是前前后后，这个线程一共获取此锁多少次）。
- _recursions：锁的重入次数。
- _owner：The Owner 拥有者，是持有该 ObjectMonitor（监视器）对象的线程；
- _EntryList：EntryList 监控集合，存放的是处于阻塞状态的线程队列，在多线程下，竞争失败的线程会进入 EntryList 队列。
- _WaitSet：WaitSet 待授权集合，存放的是处于 wait 状态的线程队列，当线程执行了 wait() 方法之后，会进入 WaitSet 队列。

监视器执行的流程是这样的：

1. 线程通过 CAS（对比并替换）尝试获取锁，如果获取成功，就将 _owner 字段设置为当前线程，说明当前线程已经持有锁，并将 _recursions 重入次数的属性 +1。如果获取失败则先通过自旋 CAS 尝试获取锁，如果还是失败则将当前线程放入到 EntryList 监控队列（阻塞）。
2. 当拥有锁的线程执行了 wait 方法之后，线程释放锁，将 owner 变量恢复为 null 状态，同时将该线程放入 WaitSet 待授权队列中等待被唤醒。
3. 当调用 notify 方法时，随机唤醒 WaitSet 队列中的某一个线程，当调用 notifyAll 时唤醒所有的 WaitSet 中的线程尝试获取锁。
4. 线程执行完释放了锁之后，会唤醒 EntryList 中的所有线程尝试获取锁。

以上就是监视器的执行流程，执行流程如下图所示： ![image.png](https://javacn.site/image/1643445271234-fcd0bfab-bbce-4883-91d7-39f8fe40303b.png)

#### [#](#小结) 小结

synchronized 是通过 Monitor 监视器实现的，而监视器又是通过 C++ 代码实现的，它的具体执行流程是：线程先通过自旋 CAS 的方式尝试获取锁，如果获取失败就进入 EntrySet（监控）集合，如果获取成功就拥有该锁。而拥有锁的线程当调用 wait() 方法时，会释放锁并进入 WaitSet（待授权）集合，直到其他线程调用 notify 或 notifyAll 方法时才会尝试再次获取锁。线程正常执行完成之后，就会通知 EntrySet 集合中的线程，让它们尝试获取锁。

### 19.产生死锁的条件有哪些？

死锁（Dead Lock）指的是两个或两个以上的运算单元（进程、线程或协程），互相持有对方所需的资源，导致它们都无法向前推进，从而导致永久阻塞的问题就是死锁。

比如线程 1 拥有了锁 A 的情况下试图获取锁 B，而线程 2 又在拥有了锁 B 的情况下试图获取锁 A，这样双方就进入相互阻塞等待的情况，如下图所示：

![img](https://javacn.site/image/1628849381323-8eaac55e-5fe7-4149-996f-c62bede2299e.png)

死锁的代码实现如下：



```java
public class DeadlockDemo {
    public static void main(String[] args) {
        Object lock1 = new Object();
        Object lock2 = new Object();

        Thread thread1 = new Thread(() -> {
            synchronized (lock1) {
                System.out.println("Thread 1 acquired lock1");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                synchronized (lock2) {
                    System.out.println("Thread 1 acquired lock2");
                }
            }
        });

        Thread thread2 = new Thread(() -> {
            synchronized (lock2) {
                System.out.println("Thread 2 acquired lock2");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                synchronized (lock1) {
                    System.out.println("Thread 2 acquired lock1");
                }
            }
        });

        thread1.start();
        thread2.start();
    }
}
```

在上面的示例中，我们创建了两个锁 lock1 和 lock2，并在两个线程中分别获取这两个锁，但是获取的顺序不同。当 thread1 获取 lock1 后，它会在持有锁 lock1 的情况下尝试获取 lock2，而当 thread2 获取 lock2 后，它会在持有锁 lock2 的情况下尝试获取 lock1。 如果这两个线程启动后，thread1 先获取 lock1 并且在获取 lock2 之前休眠，那么 thread2 就会获取 lock2，然后在尝试获取 lock1 时被阻塞。此时，thread1 就会在获取 lock2 时被阻塞。两个线程都在等待对方释放锁，从而形成了死锁。

#### [#](#死锁-4-大条件) 死锁 4 大条件

死锁的产生需要满足以下 4 个条件：

1. **互斥条件**：指运算单元（进程、线程或协程）对所分配到的资源具有排它性，也就是说在一段时间内某个锁资源只能被一个运算单元所占用。
2. **请求和保持条件**：指运算单元已经保持至少一个资源，但又提出了新的资源请求，而该资源已被其它运算单元占有，此时请求运算单元阻塞，但又对自己已获得的其它资源保持不放。
3. **不可剥夺条件**：指运算单元已获得的资源，在未使用完之前，不能被剥夺。
4. **环路等待条件**：指在发生死锁时，必然存在运算单元和资源的环形链，即运算单元正在等待另一个运算单元占用的资源，而对方又在等待自己占用的资源，从而造成环路等待的情况。

只有以上 4 个条件同时满足，才会造成死锁。

#### [#](#解决死锁) 解决死锁

死锁的常用解决方案有以下两个：

1. 按照顺序加锁：尝试让所有线程按照同一顺序获取锁，从而避免死锁。
2. 设置获取锁的超时时间：尝试获取锁的线程在规定时间内没有获取到锁，就放弃获取锁，避免因为长时间等待锁而引起的死锁。

#### [#](#死锁排查工具) 死锁排查工具

有一些工具可以帮助排查死锁问题，常见的工具有以下几个：

1. jstack：可以查看 Java 应用程序的线程状态和调用堆栈，可用于发现死锁线程的状态。
2. jconsole 和 JVisualVM：这些是 Java 自带的监视工具，可以用于监视线程、内存、CPU 使用率等信息，从而帮助排查死锁问题。
3. Thread Dump Analyzer（TDA）：是一个开源的线程转储分析器，可用于分析和诊断 Java 应用程序中的死锁问题。
4. Eclipse TPTP：是一个开源的性能测试工具平台，其中包含了一个名为 Thread Profiler 的工具，可以用于跟踪线程运行时的信息，从而诊断死锁问题。

### 20.什么是CAS？

CAS（Compare and Swap）是一种轻量级的同步操作，也是乐观锁的一种实现，它用于实现多线程环境下的并发算法。CAS 操作包含三个操作数：内存位置（或者说是一个变量的引用）、预期的值和新值。如果内存位置的值和预期值相等，那么处理器会自动将该位置的值更新为新值，否则不进行任何操作。

在多线程环境中，CAS 可以实现非阻塞算法，避免了使用锁所带来的上下文切换、调度延迟、死锁等问题，因此被广泛应用于并发编程中。

#### [#](#cas-示例) CAS 示例

在 Java 中，CAS 操作被封装在 Atomic 类中，例如 AtomicInteger 类就是利用了 CAS 操作来实现线程安全的自增操作。同时，Java 还提供了一些工具类来支持 CAS 操作，例如 Unsafe 类，它提供了一些原始的 CAS 操作方法，供 JVM 内部使用，比如以下是基于 Unsafe 类的 CAS 示例：



```java
import sun.misc.Unsafe;

import java.lang.reflect.Field;
import java.util.concurrent.TimeUnit;

public class CASDemo {
    private volatile int value = 0;
    private static Unsafe unsafe;
    private static long valueOffset;

    static {
        try {
            // 通过反射获取rt.jar包中的Unsafe类，默认Unsafe类是不能使用的
            Field field = Unsafe.class.getDeclaredField("theUnsafe");
            field.setAccessible(true);
            unsafe = (Unsafe) field.get(null);
            valueOffset = unsafe.objectFieldOffset(CASDemo.class.getDeclaredField("value"));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public void addOne() {
        int current;
        do {
            current = unsafe.getIntVolatile(this, valueOffset);
        } while (!unsafe.compareAndSwapInt(this, valueOffset, current, current + 1));
    }

    public static void main(String[] args) throws InterruptedException {
        final CASDemo casDemo = new CASDemo();
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    casDemo.addOne();
                }
            }).start();
        }
        TimeUnit.SECONDS.sleep(5);
        System.out.println(casDemo.value);
    }
}
```

以上程序的执行结果为：

> 10000

程序开启了 10 个线程，每个线程调用 1000 次，最终执行的结果是 10000，说明以上程序是线程安全的。

#### [#](#cas-执行流程) CAS 执行流程

CAS 执行的具体流程如下：

1. 将需要修改的值从主内存中读入本地线程缓存（工作内存）；
2. 执行 CAS 操作，将本地线程缓存中的值与主内存中的值进行比较；
3. 如果本地线程缓存中的值与主内存中的值相等，则将需要修改的值在本地线程缓存中修改；
4. 如果修改成功，将修改后的值写入主内存，并返回修改结果；如果失败，则返回当前主内存中的值；
5. 在多线程并发执行的情况下，如果多个线程同时执行 CAS 操作，只有一个线程的 CAS 操作会成功，其他线程的 CAS 操作都会失败，这也是 CAS 的原子性保证。

### 21.什么是ABA问题？如何解决？

#### ABA 问题

所谓的 ABA 问题是指在并发编程中，如果一个变量初次读取的时候是 A 值，它的值被改成了 B，然后又其他线程把 B 值改成了 A，而另一个早期线程在对比值时会误以为此值没有发生改变，但其实已经发生变化了，这就是 ABA 问题。

比如：张三去银行取钱，余额有 200 元，张三取 100 元，但因为程序的问题，启动了两个线程，线程一和线程二进行比对扣款，线程一获取原本有 200 元，扣除 100 元，余额等于 100 元，此时李四给张三转账 100 元，于是启动了线程三抢先在线程二之前执行了转账操作，把 100 元又变成了 200 元，而此时线程二对比自己事先拿到的 200 元和此时经过改动的 200 元值一样，就进行了减法操作，把余额又变成了 100 元。这显然不是我们要的正确结果，我们想要的结果是余额减少了 100 元，又增加了 100 元，余额还是 200 元，而此时余额变成了 100 元，显然有悖常理，这就是著名的 ABA 的问题。

执行流程如下：

- 线程一：取款，获取原值 200 元，与 200 元比对成功，减去 100 元，修改结果为 100 元。
- 线程二：取款，获取原值 200 元，阻塞等待修改。
- 线程三：转账，获取原值 100 元，与 100 元比对成功，加上 100 元，修改结果为 200 元。
- 线程二：取款，恢复执行，原值为 200 元，与 200 元对比成功，减去 100 元，修改结果为 100 元。

最终的结果是 100 元。

#### [#](#解决-aba) 解决 ABA

解决 ABA 问题的一种方法是使用带版本号的 CAS，也称为双重 CAS（Double CAS）或者版本号 CAS。具体来说，每次进行 CAS 操作时，不仅需要比较要修改的内存地址的值与期望的值是否相等，还需要比较这个内存地址的版本号是否与期望的版本号相等。如果相等，才进行修改操作。这样，在修改后的值后面追加上一个版本号，即使变量的值从 A 变成了 B 再变成了 A，版本号也会发生变化，从而避免了误判。

以下是一个使用 AtomicStampedReference 来解决 ABA 问题的示例代码：



```java
import java.util.concurrent.atomic.AtomicStampedReference;

public class ABADemo {

    private static AtomicStampedReference<Integer> atomicStampedRef = new AtomicStampedReference<>(1, 0);

    public static void main(String[] args) throws InterruptedException {
        System.out.println("初始值：" + atomicStampedRef.getReference() + "，版本号：" + atomicStampedRef.getStamp());

        // 线程 1 先执行一次 CAS 操作，期望值为 1，新值为 2，版本号为 0
        Thread thread1 = new Thread(() -> {
            int stamp = atomicStampedRef.getStamp();
            atomicStampedRef.compareAndSet(1, 2, stamp, stamp + 1);
        });

        // 线程 2 先 sleep 1 秒，让线程 1 先执行一次 CAS 操作，然后再执行一次 CAS 操作，期望值为 2，新值为 1，版本号为 1
        Thread thread2 = new Thread(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            int stamp = atomicStampedRef.getStamp();
            atomicStampedRef.compareAndSet(2, 1, stamp, stamp + 1);
        });

        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();

        System.out.println("最终值：" + atomicStampedRef.getReference() + "，版本号：" + atomicStampedRef.getStamp());
    }
}
```

以上程序的执行结果为：

> 初始值：1，版本号：0
>
> 最终值：1，版本号：2

从输出结果可以看出，即使变量的值从 1 变成了 2 再变成了 1，使用带版本号的 CAS 操作也能正确判断变量是否发生了变化。

### 22.什么是AQS？

AQS（AbstractQueuedSynchronizer）是一个用于实现各种同步器的抽象类，是 JUC（java.util.concurrent）并发包中的核心类之一，JUC 中的许多并发工具类和接口都是基于 AQS 实现的。它提供了一种基于队列的、高效的、可扩展的同步机制，是实现锁、信号量、倒计时器等同步器的基础。

> 同步器：同步器指的是用于控制多线程访问共享资源的机制。同步器可以保证在同一时间只有一个线程可以访问共享资源，从而避免了多线程访问共享资源时可能出现的数据竞争和不一致性问题。Java 中的同步器包括 synchronized 关键字、ReentrantLock、Semaphore、CountDownLatch 等。

#### [#](#核心思想) 核心思想

AQS 的核心思想是利用一个双向队列来保存等待锁的线程，同时利用一个 state 变量来表示锁的状态。AQS 的同步器可以分为独占模式和共享模式两种。独占模式是指同一时刻只允许一个线程获取锁，常见的实现类有 ReentrantLock；共享模式是指同一时刻允许多个线程同时获取锁，常见的实现类有 Semaphore、CountDownLatch、CyclicBarrier 等。

#### [#](#资源共享模式) 资源共享模式

AQS 中资源共享模式分为两种：

1. 独占模式：AQS 维护了一个同步队列，该队列中保存了所有等待获取锁的线程。当一个线程尝试获取锁时，如果锁已经被其他线程持有，则将该线程加入到同步队列的尾部，并挂起线程，等待锁被释放。当锁被释放时，从同步队列中取出一个线程，使其获取锁，同时将它从队列中移除，唤醒该线程继续执行。独占模式又分为公平锁和非公平锁： 
   1. 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁；
   2. 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的。
2. 共享模式：AQS 维护了一个等待队列和一个共享计数器。共享计数器表示当前允许获取锁的线程数，当一个线程尝试获取锁时，如果当前允许获取锁的线程数已经达到了最大值，则将该线程加入到等待队列中，并挂起线程，等待其他线程释放锁或者共享计数器增加。当锁被释放时，会从等待队列中取出一个线程，使其获取锁，同时将它从队列中移除，唤醒该线程继续执行。

AQS 提供了一些方法，允许我们在自定义同步器中使用 AQS 的同步机制。其中包括 acquire()、release()、tryAcquire()、tryRelease() 等方法，这些方法的具体实现会因同步器的不同而有所区别。

#### [#](#小结) 小结

AQS 是 Java 并发编程中非常重要的一个类，它提供了基础的同步机制，可以实现各种同步器，并为高级的并发工具类和接口提供支持。熟练掌握 AQS 的使用，对于编写高效、线程安全的并发程序是非常有帮助的，JUC（java.util.concurrent）中的许多并发工具类和接口都是基于 AQS 实现的。

































