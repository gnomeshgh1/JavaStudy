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

### 11.哈希冲突的解决方案有哪些？

哈希冲突是指在哈希表中，两个或多个元素被映射到了同一个位置的情况。



```java
String str1 = "3C";
String str2 = "2b";
int hashCode1 = str1.hashCode();
int hashCode2 = str2.hashCode();
System.out.println("字符串: " + str1 + ", hashCode: " + hashCode1);
System.out.println("字符串: " + str2 + ", hashCode: " + hashCode2);
```

程序的运行结果如下： ![image.png](https://javacn.site/image/1681181377546-3973e798-286a-4085-bf04-c230c73a1c61.png)不同的字符串，却拥有了相同的 hashCode 这就是哈希冲突。因为元素的位置是根据 hashCode 的值进行定位的，此时它们的 hashCode 相同，但一个位置只能存储一个值，这就是哈希冲突。

#### [#](#解决哈希冲突) 解决哈希冲突

在 Java 中，解决哈希冲突的常用方法有以下三种：链地址法、开放地址法和再哈希法。

1. **链地址法（Separate Chaining）**：将哈希表中的每个桶都设置为一个链表，当发生哈希冲突时，将新的元素插入到链表的末尾。这种方法的优点是简单易懂，适用于元素数量较多的情况。缺点是当链表过长时，查询效率会降低。
2. **开放地址法（Open Addressing）**：当发生哈希冲突时，通过一定的探测方法（如线性探测、二次探测、双重哈希等）在哈希表中寻找下一个可用的位置。这种方法的优点是不需要额外的存储空间，适用于元素数量较少的情况。缺点是容易产生聚集现象，即某些桶中的元素过多，而其他桶中的元素很少。
3. **再哈希法（Rehashing）**：当发生哈希冲突时，使用另一个哈希函数计算出一个新的哈希值，然后将元素插入到对应的桶中。这种方法的优点是简单易懂，适用于元素数量较少的情况。缺点是需要额外的哈希函数，且当哈希函数不够随机时，容易产生聚集现象。

#### [#](#链地址法-vs-开放地址法) 链地址法 VS 开放地址法

链地址法和开放地址法个人觉得以下几点不同：

1. **存储结构不同**：链地址法规定了存储的结构为链表（每个桶为一个链表），每次将值存储到链表的末尾；而开放地址法未规定存储的结构，所以它可以是链表也可以是树结构等。
2. **查找方式不同**：链地址法查找时，先通过哈希函数计算出哈希值，然后在哈希表中查找对应的链表，再遍历链表查找对应的值。而开放地址法查找时，先通过哈希函数计算出哈希值，然后在哈希表中查找对应的值，如果查找到的值不是要查找的值，就继续查找下一个值，直到查找到为止。
3. **插入方法不同**：链地址法插入时，先通过哈希函数计算出哈希值，然后在哈希表中查找对应的链表，再将值插入到链表的末尾。而开放地址法插入时，是通过一定的探测方法，如线性探测、二次探测、双重哈希等，在哈希表中寻找下一个可用的位置。所以链地址法插入方法实现非常简单，而开放地址法插入方法实现相对复杂。

#### [#](#线性探测-vs-二次探测) 线性探测 VS 二次探测

线性探测是发生哈希冲突时，线性探测会在哈希表中寻找下一个可用的位置，具体来说，它会检查哈希表中下一个位置是否为空，如果为空，则将元素插入该位置；如果不为空，则继续检查下一个位置，直到找到一个空闲的位置为止。

二次探测是发生哈希冲突时，二次探测会使用一个二次探测序列来寻找下一个可用的位置，具体来说，它会计算出一个二次探测序列，然后依次检查哈希表中的每个位置，直到找到一个空闲的位置为止。二次探测的优点是相对于线性探测来说，它更加均匀地分布元素，缺点是当哈希表的大小改变时，需要重新计算二次探测序列。

具体来说，二次探测序列是一个二次函数，它的形式如下：

> f(i) = i^2

其中，i 表示探测的步数，f(i) 表示探测的位置。

例如，当发生哈希冲突时，如果哈希表中的第 k 个位置已经被占用，那么二次探测会依次检查第 k+1^2、第 k-1^2、第 k+2^2、第 k-2^2、第 k+3^2、第 k-3^2……等位置，直到找到一个空闲的位置为止。

二次探测的优点是相对于线性探测来说，它更加均匀地分布元素，但缺点是容易产生二次探测聚集现象，即某些桶中的元素过多，而其他桶中的元素很少。

#### [#](#hashmap-如何解决哈希冲突) HashMap 如何解决哈希冲突？

在 Java 中，HashMap 使用的是链地址法解决哈希冲突的，对于存在冲突的 key，HashMap 会把这些 key 组成一个单向链表，之后使用尾插法把这个 key 保存到链表尾部。 ![img](https://javacn.site/image/1641790380364-b5d8bd89-aa43-49a1-99eb-5606b21d4806.png)







### 12.什么是反射，使用场景有哪些？

在 Java 中，反射是指在运行时检查和操作类、接口、字段、方法等程序结构的能力。通过反射，可以在运行时获取类的信息，创建类的实例，调用类的方法，访问和修改类的字段等。

#### [#](#反射实现) 反射实现

先定义一个需要被反射的类对象 User：



```java
public class User {
    public String name = "张三";
    private int age = 18;

    public void publicMethod() {
        System.out.println("do public method");
    }

    private void privateMethod() {
        System.out.println("do private method");
    }

    public static void staticMethod() {
        System.out.println("do static method");
    }
}
```

#### [#](#_1-反射执行公共方法) 1.反射执行公共方法

通过反射实现公共方法的调用，主要分为以下 3 步：



```java
// 1.反射得到对象
Class<?> clazz = Class.forName("User");
// 2.得到方法
Method method = clazz.getDeclaredMethod("publicMethod");
// 3.执行普通方法
method.invoke(clazz.getDeclaredConstructor().newInstance());
```

其中第 3 步，如果是 JDK 9 之前的版本使用以下代码替代：

> method.invoke(clazz.newInstance());

JDK 9 之后，使用 Class.newInstance() 方法被弃用了。

#### [#](#_2-反射执行私有方法) 2.反射执行私有方法



```java
// 1.反射得到对象
Class<?> clazz = Class.forName("User");
// 2.得到方法
Method method = clazz.getDeclaredMethod("publicMethod");
// 得到私有方法
Method privateMethod = clazz.getDeclaredMethod("privateMethod");
// 设置私有方法可访问
privateMethod.setAccessible(true);
// 执行私有方法
privateMethod.invoke(clazz.getDeclaredConstructor().newInstance());
```

#### [#](#_3-反射执行静态方法) 3.反射执行静态方法



```java
// 1.反射得到对象
Class<?> clazz = Class.forName("User");
// 2.得到方法
Method method = clazz.getDeclaredMethod("publicMethod");
// 得到静态方法
Method staticMethod = clazz.getDeclaredMethod("staticMethod");
// 执行静态方法
staticMethod.invoke(clazz);
```

#### [#](#_4-反射得到公共属性值) 4.反射得到公共属性值



```java
// 反射得到对象
Class<?> clazz = Class.forName("User");
// 得到公共属性
Field field = clazz.getDeclaredField("name");
// 得到属性值
String name = (String) field.get(
        clazz.getDeclaredConstructor().newInstance());
// 打印属性值
System.out.println("name -> " + name);
```

#### [#](#_5-反射得到私有属性值) 5.反射得到私有属性值



```java
// 反射得到对象
Class<?> clazz = Class.forName("User");
// 得到私有属性
Field privateField = clazz.getDeclaredField("age");
// 设置私有属性可访问
privateField.setAccessible(true);
// 得到属性值
int age = (int) privateField.get(
        clazz.getDeclaredConstructor().newInstance());
// 打印属性值
System.out.println("age -> " + age);
```

#### [#](#使用场景) 使用场景

反射的使用场景有很多，以下是比较常见的几种反射的使用场景：

1. 编程开发工具的代码提示，如 IDEA 或 Eclipse 等，在写代码时会有代码（属性或方法名）提示，这就是通过反射实现的。
2. 很多知名的框架如 Spring，为了让程序更简洁、更优雅，以及功能更丰富，也会使用到反射，比如 Spring 中的依赖注入就是通过反射实现的。
3. 数据库连接框架也会使用反射来实现调用不同类型的数据库（驱动）。

#### [#](#优缺点分析) 优缺点分析

反射的优点如下：

1. 灵活性：使用反射可以在运行时动态加载类，而不需要在编译时就将类加载到程序中。这对于需要动态扩展程序功能的情况非常有用。
2. 可扩展性：使用反射可以使程序更加灵活和可扩展，同时也可以提高程序的可维护性和可测试性。
3. 实现更多功能：许多框架都使用反射来实现自动化配置和依赖注入等功能。例如，Spring 框架就使用反射来实现依赖注入。

反射的缺点如下：

1. 性能问题：使用反射会带来一定的性能问题，因为反射需要在运行时动态获取类的信息，这比在编译时就获取信息要慢。
2. 安全问题：使用反射可以访问和修改类的字段和方法，这可能会导致安全问题。因此，在使用反射时需要格外小心，确保不会对程序的安全性造成影响。

#### [#](#小结) 小结

反射是指在运行时检查和操作类、接口、字段、方法等程序结构的能力。通过反射，可以在运行时获取类的信息，创建类的实例，调用类的方法，访问和修改类的字段等。通过反射可以提高程序的灵活性和可扩展性，可以实现更多的功能。但在使用反射时需要考虑性能问题以及安全等问题。

### 13.浅克隆和深克隆有什么区别？

#### 什么是克隆？

在编程中，克隆是指创建一个与原始对象相同的新对象。这个新对象通常具有与原始对象相同的属性和方法，但是它们是两个不同的对象，它们在内存中的位置不同。 在 Java 中，可以通过实现 Cloneable 接口和重写 clone() 方法来实现对象的克隆。

#### [#](#什么是浅克隆和深克隆-它们有什么区别) 什么是浅克隆和深克隆？它们有什么区别？

在 Java 中，克隆可以分为深克隆和浅克隆两种。它们的区别在于克隆出来的新对象是否与原始对象共享引用类型的属性。具体来说：

- 浅克隆：克隆出来的新对象与原始对象共享引用类型的属性。也就是说，新对象中的引用类型属性指向的是原始对象中相同的引用类型属性。如果修改了新对象中的引用类型属性，原始对象中的相应属性也会被修改。在 Java 中，可以通过实现 Cloneable 接口和重写 clone() 方法来实现浅克隆。
- 深克隆：克隆出来的新对象与原始对象不共享引用类型的属性。也就是说，新对象中的引用类型属性指向的是新的对象，而不是原始对象中相同的引用类型属性。如果修改了新对象中的引用类型属性，原始对象中的相应属性不会被修改。

#### [#](#浅克隆实现) 浅克隆实现



```java
public class CloneDemo {
    public static void main(String[] args) {
        Person p1 = new Person();
        p1.setName("张三");
        p1.setAge(18);
        Address address = new Address();
        address.setCity("北京");
        p1.setAddress(address);
        // 克隆 p1 对象
        Person p2 = p1.clone();
        System.out.println(p1 == p2); // false
        System.out.println(p1.getAddress() == p2.getAddress()); // true
    }
}
class Person implements Cloneable {
    private String name;
    private int age;
    private Address address; // 引用类型
    @Override
    public Person clone() {
        Person person = null;
        try {
            person = (Person) super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return person;
    }
    // 忽略 getter 和 setter 方法
}

class Address {
    private String city;

    // 忽略 getter 和 setter 方法
}
```

#### [#](#深克隆实现) 深克隆实现

深克隆的实现方法有很多，比如以下几个：

1. 所有引用属性都实现克隆，整个对象就变成了深克隆。
2. 使用 JDK 自带的字节流序列化和反序列化对象实现深克隆。
3. 使用第三方工具实现深克隆，比如 Apache Commons Lang。
4. 使用 JSON 工具，如 GSON、FastJSON、Jackson 序列化和反序列化对象实现深克隆。

比较常用的深克隆实现是，第一种让所有引用类型的属性实现克隆，和第四种使用 JSON 工具实现深克隆。

> 在 Java 中，序列化是指将对象转换为字节流的过程，以便可以将其存储在文件中、通过网络发送或在进程之间传递。反序列化是指将字节流转换回对象的过程。

#### [#](#深克隆实现一-引用属性实现克隆) 深克隆实现一：引用属性实现克隆



```java
import lombok.Getter;
import lombok.Setter;

public class CloneDemo {
    public static void main(String[] args) {
        Person p1 = new Person();
        p1.setName("张三");
        p1.setAge(18);
        // 引用类型
        Address address = new Address();
        address.setCity("北京");
        p1.setAddress(address);
        // 克隆 p1 对象
        Person p2 = p1.clone();
        // 对比引用类型的地址值是否相同
        System.out.println(p1.getAddress() == p2.getAddress()); // false
    }
}

@Getter
@Setter
class Person implements Cloneable {
    private String name;
    private int age;
    private Address address; // 引用类型

    @Override
    public Person clone() {
        Person person = null;
        try {
            person = (Person) super.clone();
            // 克隆引用类型
            person.setAddress(person.getAddress().clone());
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return person;
    }
}

@Getter
@Setter
class Address implements Cloneable {
    private String city;

    @Override
    public Address clone() {
        Address address = null;
        try {
            address = (Address) super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return address;
    }
}
```

#### [#](#深克隆实现二-使用-json-工具) 深克隆实现二：使用 JSON 工具

使用 Google 的 GSON（JSON）工具类来实现：



```java
import com.google.gson.Gson;
import lombok.Getter;
import lombok.Setter;

public class CloneDemo {
    public static void main(String[] args) {
        Person p1 = new Person();
        p1.setName("张三");
        p1.setAge(18);
        // 引用类型
        Address address = new Address();
        address.setCity("北京");
        p1.setAddress(address);
        // JSON 工具类
        Gson gson = new Gson();
        // 序列化
        String json = gson.toJson(p1);
        // 克隆 p1 对象 | 反序列化
        Person p2 = gson.fromJson(json, Person.class);
        // 对比引用类型的地址值是否相同
        System.out.println(p1.getAddress() == p2.getAddress()); //false
    }
}

@Getter
@Setter
class Person implements Cloneable {
    private String name;
    private int age;
    private Address address; // 引用类型

    @Override
    public Person clone() {
        Person person = null;
        try {
            person = (Person) super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return person;
    }
}

@Getter
@Setter
class Address {
    private String city;
}
```

#### [#](#小结) 小结

克隆是指创建一个与原始对象相同的新对象。克隆可以分为深克隆和浅克隆两种。它们的区别在于克隆出来的新对象是否与原始对象共享引用类型的属性。

### 14.说一说java内存模型

首先，当问到 Java 内存模型的时候，一定要注意，这块不要和 JVM 内存布局（JVM 运行时数据区域）搞混了，这块问的是 Java 内存模型，Java Memory Model，简称 JMM，而不是 JVM 的内存布局。

**Java 内存模型是用来定义 Java 线程和内存之间的操作规范的，目的是解决多线程正确执行的问题**。Java 内存模型规范的定义确保了多线程程序的可见性、有序性和原子性，从而保证了线程之间正确的交互和数据一致性。 Java 内存模型主要包括以下内容：

1. **主内存（Main Memory）**：所有线程共享的内存区域，包含了对象的字段、方法和运行时常量池等数据。
2. **工作内存（Working Memory）**：每个线程拥有自己的工作内存，用于存储主内存中的数据的副本。线程只能直接操作工作内存中的数据。
3. **内存间交互操作**：线程通过读取和写入操作与主内存进行交互。读操作将数据从主内存复制到工作内存，写操作将修改后的数据刷新到主内存。
4. **原子性（Atomicity）**：JMM 保证基本数据类型（如 int、long）的读写操作具有原子性，即不会被其他线程干扰，保证操作的完整性。
5. **可见性（Visibility）**：JMM 确保一个线程对共享变量的修改对其他线程可见。这意味着一个线程在工作内存中修改了数据后，必须将最新的数据刷新到主内存，以便其他线程可以读取到更新后的数据。
6. **有序性（Ordering）**：JMM 保证程序的执行顺序按照一定的规则进行，不会出现随机的重排序现象。这包括了编译器重排序、处理器重排序和内存重排序等。

Java 内存模型通过以上规则和语义，提供了一种统一的内存访问方式，使得多线程程序的行为可预测、可理解，并帮助开发者编写正确和高效的多线程代码。开发者可以利用 JMM 提供的同步机制（如关键字 volatile、synchronized、Lock 等）来实现线程之间的同步和通信，以确保线程安全和数据一致性。

内存模型的简单执行示例图如下：

![image.png](https://javacn.site/image/1687333041836-cc30f37e-c955-43ee-92e8-8b044b0ea650.png)

#### [#](#为什么要有java内存模型) 为什么要有Java内存模型？

只所以要有 Java 内存模型的原因有两个：

1. CPU 缓存和主内存数据一致性的问题
2. 操作系统优化指令重排序的问题

我们分别来看下这两个问题。

#### [#](#缓存一致性问题) 缓存一致性问题

要讲明白缓存一致性问题，要从计算机的内存结构说起，它的结构是这样的：

![image.png](https://javacn.site/image/1687348549816-1a322b7a-3a22-4544-b1e6-be2cd07f1ccc.png)

所以从上面可以看出计算机的重要组成部分包含以下内容：

1. CPU
2. CPU 寄存器：也叫 L1 缓存，一级缓存。
3. CPU 高速缓存：也叫 L2 缓存，二级缓存。
4. （主）内存

> 当然，部分高端机器还有 L3 三级缓存。

由于主内存与 CPU 处理器的运算能力之间有数量级的差距，所以在传统计算机内存架构中会引入高速缓存（L2）来作为主存和处理器之间的缓冲，CPU 将常用的数据放在高速缓存中，运算结束后 CPU 再讲运算结果同步到主内存中，这样就会导致多个线程在进行操作和同步时，导致 CPU 缓存和主内存数据不一致的问题。

#### [#](#操作系统优化和指令重排序问题) 操作系统优化和指令重排序问题

并且，由于 JIT（Just In Time，即时编译）技术的存在，它可能会对代码进行优化，比如将原本执行顺序为 a -> b -> c 的流程，“优化”成 a -> c -> b 了，但这样优化之后，可能会导致我们的程序在某些场景执行出错，比如单例模式双重效验锁的场景，这就是典型的好心办坏事的事例。

#### [#](#结论) 结论

所以，为了防止缓存一致性问题和操作系统指令重排序导致的问题，于是就有了 Java 内存模型，来规范和定义多线程的可见性、有序性和原子性，从而保证了线程之间正确的交互和数据一致性。

Java 内存模型定义了很多东西，比如以下这些：

- 所有的变量都存储在主内存（Main Memory）中。
- 每个线程都有一个私有的本地内存（Local Memory），本地内存中存储了该线程以读/写共享变量的拷贝副本。
- 线程对变量的所有操作都必须在本地内存中进行，而不能直接读写主内存。
- 不同的线程之间无法直接访问对方本地内存中的变量。

就是咱们文章刚开始画的那附图。

#### [#](#主内存和工作内存交互规范) 主内存和工作内存交互规范

为了更好的控制主内存和本地内存的交互，Java 内存模型定义了八种操作来实现（以下内容只需要简单了解即可）：

- lock：锁定，作用于主内存的变量，把一个变量标识为一条线程独占状态。
- unlock：解锁，作用于主内存变量，把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
- read：读取，作用于主内存变量，把一个变量值从主内存传输到线程的工作内存中，以便随后的 load 动作使用
- load：载入，作用于工作内存的变量，它把 read 操作从主内存中得到的变量值放入工作内存的变量副本中。
- use：使用，作用于工作内存的变量，把工作内存中的一个变量值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作。
- assign：赋值，作用于工作内存的变量，它把一个从执行引擎接收到的值赋值给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
- store：存储，作用于工作内存的变量，把工作内存中的一个变量的值传送到主内存中，以便随后的write的操作。
- write：写入，作用于主内存的变量，它把 store 操作从工作内存中一个变量的值传送到主内存的变量中。

> 注意：工作内存也就是本地内存的意思。

#### [#](#小结) 小结

Java 内存模型（Java Memory Model，JMM）是一种规范，定义了 Java 程序中多线程环境下内存访问和操作的规则和语义，主要是解决 CPU 缓存一致性问题和操作系统优化指令重排序的问题的。



​     





​    



