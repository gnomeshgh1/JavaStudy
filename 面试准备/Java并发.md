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







