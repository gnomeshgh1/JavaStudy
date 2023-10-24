###                                                      Java基础

#### 1.什么是方法重载？返回值算方法重载吗？

在 Java 中，方法重载是指在同一个类中定义多个方法，它们具有相同的名称但参数列表不同。方法重载的定义如下：



```java
public void myMethod(int arg1) {
    // 方法体
}

public void myMethod(int arg1, int arg2) {
    // 方法体
}

public void myMethod(String arg1) {
    // 方法体
}
```

##### [#](#返回值不同不算方法重载) 返回值不同不算方法重载



```java
public String myMethod(int arg1) {
    // 方法体
}

public int myMethod(int arg1) {
    // 方法体
}
```

因为不同的返回值类型，JVM 没办法分辨到底要调用哪个方法，比如以下代码：



```java
// 方法调用
myMethod(1);
```

更深层次的原因：JVM 调用方法是通过方法签名来判断到底要调用哪个方法的，而方法签名 = 方法名称 + 参数类型 + 参数个数组成的一个唯一值，这个唯一值就是方法签名。从方法签名的组成可以看出，返回类型不是方法签名的组成部分，所以不同的返回类型也就不算方法重载了，因为它不能让 JVM 确定要调用的具体方法。

### 2.ArrayList和LinkedList有什么区别

ArrayList 和 LinkedList 都是 Java 中的 List 接口的实现类。

![img](https://javacn.site/image/1680786457841-a70692d5-62eb-4e0a-917d-1de864419482.png)

但它们有以下不同：

1. 它们的底层实现不同：ArrayList 是基于动态数组的数据结构，而 LinkedList 是基于链表的数据结构。
2. 随机访问性能不同：ArrayList 优于 LinkedList，因为 ArrayList 可以根据下标以 O(1) 时间复杂度对元素进行随机访问。而 LinkedList 的访问时间复杂度为 O(n)，因为它需要遍历整个链表才能找到指定的元素。
3. 插入和删除性能不同：LinkedList 优于 ArrayList，因为 LinkedList 的插入和删除操作时间复杂度为 O(1)，而 ArrayList 的时间复杂度为 O(n)。

#### [#](#arraylist-基础用法) ArrayList 基础用法



```java
import java.util.ArrayList;
public class Main {
  public static void main(String[] args) {
    ArrayList<String> tech = new ArrayList<String>();
    tech.add("Java");
    tech.add("SQL");
    tech.add("Redis");
    System.out.println(tech);
  }
}
```

#### [#](#linkedlist-基本用法) LinkedList 基本用法



```java
import java.util.LinkedList;

public class Main {
  public static void main(String[] args) {
    LinkedList<String> tech = new LinkedList<String>();
    tech.add("Java");
    tech.add("SQL");
    tech.add("Redis");
    System.out.println(tech);
  }
}
```

#### [#](#小结) 小结

ArrayList 和 LinkedList 都是 List 接口的实现类，但它们的底层实现（结构）不同、随机访问的性能和添加/删除的效率不同。如果是随机访问比较多的业务场景可以选择使用 ArrayList，如果添加和删除比较多的业务场景可以选择使用 LinkedList。

### 3.ArrayList和Vector有什么区别

ArrayList 和 Vector 实现了 List 接口，它们都是动态数组的实现，它们也拥有相同的方法，对元素进行添加、删除、查找等操作，如下代码所示：



```java
import java.util.ArrayList;
import java.util.Vector;

public class Main {
    public static void main(String[] args) {
        ArrayList<String> arrayList = new ArrayList<>();
        Vector<String> vector = new Vector<>();

        // 添加元素
        arrayList.add("Java");
        arrayList.add("Python");
        arrayList.add("C++");

        vector.add("Java");
        vector.add("Python");
        vector.add("C++");

        // 获取元素
        System.out.println(arrayList.get(0));
        System.out.println(vector.get(0));

        // 删除元素
        arrayList.remove(0);
        vector.remove(0);

        // 获取元素个数
        System.out.println(arrayList.size());
        System.out.println(vector.size());
    }
}
```

但它们有以下区别：

1. 线程安全性：Vector 是线程安全的，而 ArrayList 不是。所以在多线程环境下，应该使用 Vector。
2. 性能：由于 Vector 是线程安全的，所以它的性能通常比 ArrayList 差。在单线程环境下，ArrayList 比 Vector 快。
3. 初始容量增长方式：当容量不足时，ArrayList 默认会增加 50% 的容量，而 Vector 会将容量翻倍。这意味着在添加元素时，ArrayList 需要更频繁地进行扩容操作，而 Vector 则更适合于存储大量数据。

综上所述，如果不需要考虑线程安全问题，并且需要高效的存取操作，则 ArrayList 是更好的选择；如果需要考虑线程安全以及更好的数据存储能力，则应该选择 Vector。

### 4.抽象类和普通类有什么区别？

在 Java 中，普通类和抽象类是两种不同的类类型。普通类是可以直接实例化的类，而抽象类则不能直接实例化。抽象类通常用于定义一些基本的行为和属性，而具体的实现则由其子类来完成。以下是普通类和抽象类的一些区别：

1. 实例化：普通类可以直接实例化，而抽象类不能直接实例化。
2. 方法：抽象类中既包含抽象方法又可以包含具体的方法，而普通类只能包含普通方法。
3. 实现：普通类实现接口需要重写接口中的方法，而抽象类可以实现接口方法也可以不实现。

以下是一个普通类和一个抽象类的示例代码：



```java
// 普通类
public class MyClass {
    public void myMethod() {
        System.out.println("我是普通类");
    }
}

// 抽象类
public abstract class MyAbstractClass {
    public abstract void myAbstractMethod();
    public void myMethod() {
        System.out.println("我是抽象类");
    }
}
```

### 5.抽象类和接口有什么区别？

在 Java 中，抽象类和接口是两种不同的类类型。它们都不能直接实例化，并且它们都是用来定义一些基本的属性和方法的，但它们有以下几点不同：

1. 定义：定义的关键字不同，抽象类是 abstract，而接口是 interface。
2. 方法：抽象类可以包含抽象方法和具体方法，而接口只能包含方法声明（抽象方法）。
3. 方法访问控制符：抽象类无限制，只是抽象类中的抽象方法不能被 private 修饰；而接口有限制，接口默认的是 public 控制符。
4. 实现：一个类只能继承一个抽象类，但可以实现多个接口。
5. 变量：抽象类可以包含实例变量和静态变量，而接口只能包含常量。
6. 构造函数：抽象类可以有构造函数，而接口不能有构造函数。

以下是一个抽象类和一个接口的示例代码：



```java
// 抽象类
public abstract class MyAbstractClass {
    public abstract void myAbstractMethod();
    private void myMethod() {
        System.out.println("This is a method in an abstract class.");
    }
    public int myVariable = 0;
    public static int myStaticVariable = 0;
    public MyAbstractClass() {
        System.out.println("This is a constructor in an abstract class.");
    }
}

// 接口
public interface MyInterface {
    void myAbstractMethod();
    int MY_CONSTANT = 0;
}
```

### 6.HashMap和Hashtable有什么区别？

HashMap 和 Hashtable 都实现了 Map 接口，都是 Java 中用于存储键值对的数据结构，它们的底层数据结构都是数组加链表的形式（默认情况下），但它们存在以下几点不同：

1. 线程安全：Hashtable 是线程安全的，而 HashMap 是非线程安全的。
2. 性能：因为 Hashtable 使用了 synchronized 给整个方法添加了锁，所以相比于 HashMap 来说，它的性能不如 HashMap。
3. 存储：HashMap 允许 key 和 value 为 null，而 Hashtable 不允许存储 null 键和 null 值。

> Hashtable 不能存储 null 键和 null 值是因为，它的 key 值要进行哈希计算，如果为 null 的话，无法调用该方法，还是会抛出空指针异常。而 value 值为 null 的话，Hashtable 源码中会主动抛出空指针异常。![img](https://javacn.site/image/1681010004799-94551d63-64ac-4484-8496-3ab84719f17c.png)
>
> 
>
> HashMap 允许 key 和 value 为 null 的原因是因为在 HashMap 中对 null 值进行了特殊处理，如果为 null 时会把它赋值为 0，如下源码所示：![img](https://javacn.site/image/1681010086900-3f1a6931-505b-445a-905b-fd1319110ef9.png)

#### [#](#hashmap-和-hashtable-基础使用) HashMap 和 Hashtable 基础使用



```java
import java.util.HashMap;
import java.util.Hashtable;

public class HashtableDemo {
    public static void main(String[] args) {
        Hashtable<String, String> table = new Hashtable<>();
        table.put("A", "Apple");
        table.put("B", "Ball");
        table.forEach((k, v) -> System.out.println(k + " " + v));

        HashMap<String, String> map = new HashMap<>();
        map.put("A", "Apple");
        map.put("B", "Ball");
        map.forEach((k, v) -> System.out.println(k + " " + v));
    }
}
```

#### [#](#hashtable-不推荐使用) Hashtable 不推荐使用

虽然 Hashtable 是线程安全的，但在多线程环境下官方也不推荐使用 Hashtable，因为 Hashtable 是给整个方法添加 synchronized 来实现线程安全的，所以它的性能很差。官方推荐在多线程环境下，使用线程安全的 ConcurrentHashMap 来完成数据存储。

> ConcurrentHashMap 锁粒度更细，在多线程环境下的性能表现更好。

### 7.HashMap和HashSet有什么区别？

HashMap 和 HashSet 都是 Java 中的集合类，但它们有以下几点区别：

1. HashSet 实现了 Set 接口，只存储对象；HashMap 实现了 Map 接口，用于存储键值对。
2. HashSet 底层是用 HashMap 存储的，HashSet 封装了一系列 HashMap 的方法，HashSet 将（自己的）值保存到 HashMap 的 Key 里面了。
3. HashSet 不允许集合中有重复的值（如果有重复的值，会插入失败），而 HashMap 键不能重复，值可以重复（如果键重复会覆盖原来的值）。

#### [#](#hashmap-基础使用) HashMap 基础使用



```java
Map<String, String> map = new HashMap<>();
map.put("A", "Apple");
map.put("B", "Ball");
map.put("C", "Cat");
map.forEach((k, v) -> System.out.println(k + " " + v));
```

#### [#](#hashset-基础使用) HashSet 基础使用



```java
Set<String> set = new HashSet<>();
set.add("A");
set.add("B");
set.add("C");
set.forEach(System.out::println);
```

#### [#](#小结) 小结

HashSet 适用于只存储对象的情况，而 HashMap 适用于需要存储键值对的情况，可以根据键快速查找值。HashSet 底层是用 HashMap 存储的，用它可以存储不重复的值。

### 8.HashMap的负载因子，为什么是0.75

HashMap 负载因子 load factor，也叫做扩容因子和装载因子，它是 HashMap 在进行扩容时的一个阈值，当 HashMap 中的元素个数超过了容量乘以负载因子时，就会进行扩容。默认的负载因子是 0.75，也就是说当 HashMap 中的元素个数超过了容量的 75% 时，就会进行扩容。当然，我们也可以通过构造函数来指定负载因子，如下所示： ![image.png](https://javacn.site/image/1681377671384-8f877f44-e8e7-46d9-bbaf-fcf711e47793.png)

#### [#](#扩容计算公式) 扩容计算公式

HashMap 扩容的计算公式是：initialCapacity * loadFactor = HashMap 扩容。 其中，initialCapacity 是初始容量，默认值为 16（懒加载机制，只有当第一次 put 的时候才创建），loadFactor 是负载因子，默认值为 0.75。也就是说当 16 * 0.75 = 12 时，HashMap 就会开始扩容。

#### [#](#为什么要进行扩容) 为什么要进行扩容？

HashMap 扩容的目的是为了减少哈希冲突，提高 HashMap 性能的。

#### [#](#为什么默认负载因子是-0-75) 为什么默认负载因子是 0.75？

HashMap 负载因子 loadFactor 的默认值是 0.75，为什么是 0.75 呢？官方给的答案是这样的：

> As a general rule, the default load factor (.75) offers a good tradeoff between time and space costs. Higher values decrease the space overhead but increase the lookup cost (reflected in most of the operations of the HashMap class, including get and put). The expected number of entries in the map and its load factor should be taken into account when setting its initial capacity, so as to minimize the number of rehash operations. If the initial capacity is greater than the maximum number of entries divided by the load factor, no rehash operations will ever occur.

上面的意思，简单来说是默认负载因子为 0.75，是因为它提供了空间和时间复杂度之间的良好平衡。 负载因子太低会导致大量的空桶浪费空间，负载因子太高会导致大量的碰撞，降低性能。0.75 的负载因子在这两个因素之间取得了良好的平衡。

#### [#](#负载因子-0-75-的科学推测) 负载因子 0.75 的科学推测

也就是说官方并未对负载因子为 0.75 做过的的解释，只是大概的说了一下，0.75 是空间和时间复杂度的平衡，但更多的细节是未做说明的，然而 Stack Overflow 一位大神 [https://stackoverflow.com/questions/10901752/what-is-the-significance-of-load-factor-in-hashmapopen in new window](https://stackoverflow.com/questions/10901752/what-is-the-significance-of-load-factor-in-hashmap) 从科学的角度推测了这个问题的答案。 简单来说是通过二项式哈希函数的冲突概率来解释 0.75 这个问题的。 假设一个哈希桶为空和非空的概率为 0.5，我们用 s 表示容量，n 表示已添加元素个数。 用 s 表示添加的键的大小和 n 个键的数目。根据二项式定理，桶为空的概率为：

> P(0) = C(n, 0) * (1/s)^0 * (1 - 1/s)^(n - 0)

因此，如果桶中元素个数小于以下数值，则桶可能是空的：

> log(2)/log(s/(s - 1))

当 s 趋于无穷大时，如果增加的键的数量是 P(0) = 0.5，那么 n/s 很快趋近于 log(2)，而 log(2) ~ 0.693。 所以，合理值大概在 0.7 左右，这就是对负载因子为 0.75 的一个科学推测。

#### [#](#小结) 小结

负载因子 loadFactor 是 HashMap 在进行扩容时的一个阈值，扩容的计算公式是：initialCapacity * loadFactor = HashMap 扩容。它的默认值为 0.75，此值提供了空间和时间复杂度之间的良好平衡。

### 9.HashMap的底层是如何实现的？

HashMap 在不同的 JDK 版本下的实现是不同的，在 JDK 1.7 时，HashMap 底层是通过数组 + 链表实现的；而在 JDK 1.8 时，HashMap 底层是通过数组 + 链表或红黑树实现的。

具体来说，HashMap 内部维护了一个数组，每个数组元素又是一个链表或者红黑树，每个链表或者红黑树节点存储了一个键值对。当需要存储新的键值对时，HashMap 会根据键的哈希值确定其在数组中的位置，如果该位置已经有了其他键值对，则通过链表或红黑树解决冲突，将新的键值对添加到链表或红黑树的末尾。当链表或红黑树长度达到一定程度后，HashMap 会自动将链表转换为红黑树，以提高查找效率。

如下图所示，HashMap 在 JDK 1.7 中的实现如下图所示：![image.png](https://javacn.site/image/1641789988036-e0b9a8f2-30c6-4e83-8794-c0315f41da71.png) 在 JDK 1.8 时，HashMap 如下图所示： ![image.png](https://javacn.site/image/1641790380364-b5d8bd89-aa43-49a1-99eb-5606b21d4806.png)

#### [#](#链表和红黑树互转流程) 链表和红黑树互转流程

#### [#](#链表升级为红黑树) 链表升级为红黑树

在 JDK 1.8 之后，HashMap 默认是先使用数组 + 链表存储数据，但当满足以下两个条件时：

1. 链表的数量大于阈值（默认是 8）
2. 并且数组长度大于 64 时

为了（查询）的性能考虑会将链表升级为红黑树进行存储，具体执行流程如下：

1. 创建新的红黑树对象，并将链表内所有的键值对全部添加到红黑树中。
2. 将原来的链表引用指向新创建的红黑树。

#### [#](#红黑树退化为链表) 红黑树退化为链表

当进行了删除操作，导致红黑树的节点小于等于 6 时，会发生退化，将红黑树转换为链表。这是因为当节点数量较少时，红黑树对性能的提升并不明显，反而占用了更多的内存空间。具体执行流程如下：

1. 从红黑树的根节点开始，按照中序遍历的顺序将所有节点加入到一个新的链表中。
2. 将原来的红黑树引用指向新创建的链表。

#### [#](#小结) 小结

HashMap 在 JDK 1.7 时，是通过数组 + 链表实现的，而在 JDK 1.8 时，HashMap 是通过数组 + 链表或红黑树实现的。在 JDK 1.8 之后，如果链表的数量大于阈值（默认为 8），并且数组长度大于 64 时，为了查询效率会将链表升级为红黑树，但当红黑树的节点小于等于 6 时，为了节省内存空间会将红黑树退化为链表。

### 10.为什么HashMap会死循环

HashMap 死循环发生在 JDK 1.8 之前的版本中，它是指在并发环境下，因为多个线程同时进行 put 操作，导致链表形成环形数据结构，一旦形成环形数据结构，在 get(key) 的时候就会产生死循环。如下图所示：

![img](/image/1681186887864-1799ef94-b6a5-4b08-9aa9-9c147ebeaefc.png)

#### [#](#死循环原因) 死循环原因

HashMap 导致死循环的原因是由以下条件共同导致的：

1. HashMap 使用头插法进行数据插入（JDK 1.8 之前）；
2. 多线程同时添加；
3. 触发了 HashMap 扩容。

#### [#](#什么是头插法) 什么是头插法？

头插法是指新来的值会取代原有的值，插入到链表的头部，如下图所示。 原链表如下图所示：

![img](https://javacn.site/image/1641895435908-d50435c7-56be-4d32-ba59-e069a1bbc5ce.png)

此时使用头插入插入一个元素 Z，如下图所示：

![img](https://javacn.site/image/1641895742722-c9a11269-2746-4115-a7b6-74aa43987b70.png)

头插法会导致 HashMap 在进行扩容时，链表的顺序发生反转，如下图所示：![img](https://javacn.site/image/1641896197742-25b61907-47e1-4cb5-b003-9b4074b72ce7.png)png) 因为在 HashMap 扩容时，会先从旧 HashMap 的头节点读取并插入到新 HashMap 节点中，旧节点的读取顺序是 A -> B -> C，于是插入到新 HashMap 中的顺序就变成了 C -> B -> A，这样就破坏了链表的顺序，导致了链表反转。

#### [#](#死循环产生过程) 死循环产生过程

#### [#](#死循环执行步骤1) 死循环执行步骤1

死循环是因为并发 HashMap 扩容导致的，并发扩容的第一步，线程 T1 和线程 T2 要对 HashMap 进行扩容操作，此时 T1 和 T2 指向的是链表的头结点元素 A，而 T1 和 T2 的下一个节点，也就是 T1.next 和 T2.next 指向的是 B 节点，如下图所示：

![img](https://javacn.site/image/1641896824515-f36af0da-8c0a-4d41-a43c-d48c52204881.png)

#### [#](#死循环执行步骤2) 死循环执行步骤2

死循环的第二步操作是，线程 T2 时间片用完进入休眠状态，而线程 T1 开始执行扩容操作，一直到线程 T1 扩容完成后，线程 T2 才被唤醒，扩容之后的场景如下图所示： ![image.png](https://javacn.site/image/1641898700546-c414add5-9035-4085-8bdb-e0e7a9eba936.png)从上图可知线程 T1 执行之后，因为是头插法，所以 HashMap 的顺序已经发生了改变，但线程 T2 对于发生的一切是不可知的，所以它的指向元素依然没变，如上图展示的那样，T2 指向的是 A 元素，T2.next 指向的节点是 B 元素。

#### [#](#死循环执行步骤3) 死循环执行步骤3

当线程 T1 执行完，而线程 T2 恢复执行时，死循环就建立了，如下图所示：

![img](https://javacn.site/image/1641897351523-7ec0a83d-d5d0-457a-8bed-d356c48393a0.png)

因为 T1 执行完扩容之后 B 节点的下一个节点是 A，而 T2 线程指向的首节点是 A，第二个节点是 B，这个顺序刚好和 T1 扩完容完之后的节点顺序是相反的。**T1 执行完之后的顺序是 B 到 A，而 T2 的顺序是 A 到 B，这样 A 节点和 B 节点就形成死循环了**，这就是 HashMap 死循环导致的原因。

#### [#](#解决方案) 解决方案

HashMap 死循环的常用解决方案有以下几个：

1. 升级到高版本 JDK（JDK 1.8 以上），高版本 JDK 使用的是尾插法插入新元素的，所以不会产生死循环的问题；
2. 使用线程安全容器 ConcurrentHashMap 替代（推荐使用此方案）；
3. 使用线程安全容器 Hashtable 替代（性能低，不建议使用）；
4. 使用 synchronized 或 Lock 加锁 HashMap 之后，再进行操作，相当于多线程排队执行（比较麻烦，也不建议使用）。

#### [#](#小结) 小结

HashMap 死循环发生在 JDK 1.7 版本中，形成死循环的原因是 HashMap 在 JDK 1.7 使用的是头插法，头插法 + 多线程并发操作 + HashMap 扩容，这几个点加在一起就形成了 HashMap 的死循环，解决死循环可以采用线程安全容器 ConcurrentHashMap 替代。







