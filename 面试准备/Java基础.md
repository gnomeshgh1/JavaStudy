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

