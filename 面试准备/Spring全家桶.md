##                                                               Spring全家桶

### 1.什么是Spring框架？

Spring 框架是一个用于构建企业级 Java 应用程序的开源框架。它提供了一种综合性的编程和配置模型，用于开发灵活、可扩展、可维护的应用程序。Spring 框架提供了许多功能和特性，帮助开发人员快速构建企业应用程序。

以下是 Spring 框架的一些核心特点：

1. **轻量级**：Spring 框架采用了松耦合的设计原则，仅依赖于少量的第三方库，因此它是一个轻量级的框架。开发人员可以根据需要选择使用 Spring 的特定功能，而无需引入整个框架。
2. **控制反转（IoC）**：Spring 框架通过控制反转（IoC）容器管理应用程序中的对象及其依赖关系。通过 IoC 容器，开发人员可以将对象的创建、组装和生命周期管理交给 Spring 框架处理，从而实现了松耦合和可测试性。
3. **面向切面编程（AOP）**：Spring 框架支持面向切面编程，可以通过 AOP 在应用程序中实现横切关注点的模块化。例如，日志记录、事务管理和安全性等横切关注点可以通过 AOP 进行集中处理，而不会侵入业务逻辑的代码。
4. **声明式事务管理**：Spring 框架提供了声明式事务管理的支持。通过使用注解或 XML 配置，开发人员可以将事务管理逻辑与业务逻辑分离，并且可以轻松地在方法或类级别上应用事务。
5. **框架整合**：Spring 框架可以与许多其他开源框架和技术无缝集成，如 Hibernate、MyBatis、JPA、Struts 和 JSF 等。这使得开发人员可以使用 Spring 框架来整合和协调不同的技术，构建全面的企业应用程序。
6. **测试支持**：Spring 框架提供了广泛的测试支持，包括单元测试和集成测试。它提供了一个专门的测试上下文，可以轻松地编写和执行单元测试，以验证应用程序的行为和功能。

总之，Spring 框架简化了企业级 Java 应用程序的开发过程，提供了一种模块化、可维护和可测试的编程模型，广泛应用于 Java 开发社区。

### 2.什么是IOC？

IoC（Inversion of Control，控制反转）是 Spring 框架的核心概念之一，它是一种设计原则和编程模式，用于实现松耦合和可测试的应用程序。在传统的编程模式中，对象之间的创建、组装和管理都是由开发人员手动完成的，而在 IoC 模式下，这些责任被委托给一个容器来管理。

在 IoC 模式中，对象之间的依赖关系被反转了，即由开发人员手动控制对象之间的依赖关系变为由容器自动注入依赖。这种反转的控制使得应用程序的各个模块之间解耦，提高了代码的灵活性、可维护性和可测试性。

IoC 的实现依赖于一个称为 IoC 容器的组件。IoC 容器负责创建和管理对象，以及解决对象之间的依赖关系。开发人员只需在配置文件（如 XML 配置文件）或使用注解方式中指定对象的依赖关系和其他配置细节，容器就会根据这些配置信息动态地实例化对象、注入依赖并管理对象的生命周期。

#### [#](#实现手段) 实现手段

IoC 容器通过以下两种主要的方式来实现控制反转：

1. 依赖注入（Dependency Injection，DI）：依赖注入是 IoC 的一种具体实现方式，通过将依赖关系注入到对象中，实现了对象之间的解耦。容器负责查找依赖对象，并将其自动注入到相应的对象中。依赖注入可以通过构造函数、Setter 方法或接口注入来完成。
2. 依赖查找（Dependency Lookup）：依赖查找是另一种 IoC 的实现方式，它通过容器提供的 API，开发人员手动查找和获取所需的依赖对象。开发人员在代码中通过容器提供的接口来获取所需的对象实例，从而实现了对象之间的解耦。

#### [#](#优点分析) 优点分析

相比于传统的程序开发，使用 IoC 的好处在于：

- 降低了代码之间的耦合度，使程序变得简单。
- 可维护性好，对象更易扩展和重用。
- IoC 容器管理对象，简化开发难度，节省开发时间。

总之，IoC 是 Spring 框架的基石，是 Spring 框架众多特性的基础。

### 3.依赖注入和依赖查找有什么区别？

依赖注入和依赖查找都是实现 IoC 的两种常用方法，它们的区别如下。

#### [#](#依赖注入) 依赖注入

依赖注入是一种将依赖关系从一个对象传递到另一个对象的技术。在依赖注入中，对象不再负责创建或查找它所依赖的对象，而是将依赖关系委托给 IoC 容器。容器在创建对象时，自动将依赖的对象注入到它所依赖的对象中。

依赖注入的优点是可以减少对象之间的耦合，使代码更加灵活和可维护。在 Spring 框架中，依赖注入通过注解或 XML 配置文件来实现。

#### [#](#依赖查找) 依赖查找

依赖查找是一种从 IoC 容器中查找依赖对象的技术。在依赖查找中，对象负责查找它所依赖的对象，而不是将依赖关系委托给容器。容器只负责管理对象的生命周期，而不负责对象之间的依赖关系。

依赖查找的优点是可以更加精细地控制对象之间的依赖关系，但是它也会增加对象之间的耦合度。在 Spring 框架中，依赖查找通过 ApplicationContext 接口的 getBean() 方法来实现。

#### [#](#小结) 小结

因此，依赖注入和依赖查找的区别在于，依赖注入是将依赖关系委托给容器，由容器来管理对象之间的依赖关系；而依赖查找是由对象自己来查找它所依赖的对象，容器只负责管理对象的生命周期。

### 4.IOC和DI有什么区别？

IoC 和 DI 都是 Spring 框架中的核心概念，它们的区别在于:

- **IoC（Inverse of Control，控制反转）**：它是一种思想，主要解决程序设计中的对象依赖关系管理问题。在 IoC 思想中，对象的创建权反转给第三方容器，由容器进行对象的创建及依赖关系的管理。
- **DI（Dependency Injection，依赖注入）**：它是 IoC 思想的具体实现方式之一，用于实现 IoC。在 Spring 中，依赖注入是指:在对象创建时，由容器自动将依赖对象注入到需要依赖的对象中。

简单来说，它们的关系是：

- IoC 是一种思想、理念，定义了对象创建和依赖关系处理的方式。
- DI 是 IoC 思想的具体实现方式之一，实际提供对象依赖关系的注入功能。

所以 IoC 是更基础和广义的概念，DI 可以说是 IoC 的一种实现手段。大多数情况下，我们提到 IoC 的时候，其实意味着 DI，因为 DI 已经是 IoC 最常见和广泛使用的实现方式了。

例如在 Spring 框架中：

- IoC 体现为 Spring 容器承担了对象创建及依赖关系管理的控制权。
- DI 体现为 Spring 容器通过构造方法注入、Setter 方法注入等方式，将依赖对象注入到需要依赖的对象中。

所以综上，IoC 和 DI 之间的关系可以这样理解:

- IoC 是理论，DI 是实践。
- IoC 是思想，DI 是手段。
- IoC 是整体，DI 是部分。

### 5.说一下DI实现原理？

DI（依赖注入）是一种将依赖关系从一个对象传递到另一个对象的技术。在 Spring 框架中，DI 是通过使用注释（如 @Autowired、@Qualifier 和 @Value）来实现的。

#### [#](#di-实现原理) DI 实现原理

DI 的实现原理是通过反射机制实现的。在 Spring 框架中，当容器创建一个对象时，它会检查该对象的依赖关系，并使用反射机制查找依赖对象。然后，容器将依赖对象注入到该对象中。

具体来说，当使用 @Autowired 注释时，Spring 容器会自动查找与该类型匹配的 bean，并将其注入到该字段中。如果有多个匹配的 bean，则可以使用 @Qualifier 注释来指定要注入的 bean 的名称。

当使用 @Value 注释时，Spring 容器会将属性值注入到该字段中。属性值可以从配置文件中读取，也可以是硬编码的值。

#### [#](#优点分析) 优点分析

DI 的优点是可以减少对象之间的耦合，使代码更加灵活和可维护。通过将依赖关系委托给容器，对象之间的依赖关系变得更加松散，从而使代码更加模块化和易于测试。

因此，DI 是 Spring 框架中实现 IoC 的重要技术之一。它可以使代码更加灵活和可维护，从而提高开发效率和代码质量。

### 6.什么是AOP？

AOP（Aspect-Oriented Programming，面向切面编程）是一种软件开发的编程范式，用于将跨越多个模块的（横切）关注点从核心业务逻辑中分离出来，使得横切关注点的定义和应用能够更加集中和重用。

在传统的面向对象编程中，程序的功能逻辑被分散在各个对象中，而横切关注点（如日志记录、事务管理、安全控制等）则分散在多个对象之间，导致代码重复、可维护性差，并且难以修改和扩展。AOP 的目标就是解决这些问题。

AOP 通过引入横切关注点，将其与核心业务逻辑分离，并以模块化的方式进行管理。它通过切面（Aspect）来描述横切关注点，切面是对横切关注点的封装。切面定义了在何处、何时和如何应用横切关注点。在 AOP 中，切面可以横跨多个对象，独立于核心业务逻辑。

#### [#](#aop-组成) AOP 组成

AOP 的实现依赖于以下几个概念：

- **切面（Aspect）**：切面是横切关注点的模块化单元，它将通知和切点组合在一起，描述了在何处、何时和如何应用横切关注点。
- **切点（Pointcut）**：用于定义哪些连接点被切面关注，即切面要织入的具体位置。
- **连接点（Join Point）**：在程序执行过程中的某个特定点，例如方法调用、异常抛出等。
- **通知（Advice）**：切面在特定切点上执行的代码，包括在连接点之前、之后或周围执行的行为。
- **织入（Weaving）**：将切面应用到目标对象中的过程，可以在编译时、加载时或运行时进行。

#### [#](#优点分析) 优点分析

AOP 的优点是可以将横切关注点从应用程序的核心业务逻辑中分离出来，以便更好地实现模块化和复用。通过使用 AOP，可以将通用的功能（如日志记录、性能统计、事务管理等）封装成切面，然后在需要的地方进行重用，从而提高代码的可维护性和可重用性。

### 7.AOP实现技术有哪些？

AOP（面向切面编程）是一种编程范式，它允许将横切关注点从应用程序的核心业务逻辑中分离出来，以便更好地实现模块化和复用。

AOP 常见实现技术有以下两种：

1. 静态代理：静态代理是一种在编译时就已经确定代理关系的代理方式。在静态代理中，代理类和被代理类都要实现同一个接口或继承同一个父类，代理类中包含了被代理类的实例，并在调用被代理类的方法前后执行相应的操作。静态代理的优点是实现简单，易于理解和掌握，但是它的缺点是需要为每个被代理类编写一个代理类，当被代理类的数量增多时，代码量会变得很大。
2. 动态代理：动态代理是一种在运行时动态生成代理类的代理方式。在动态代理中，代理类不需要实现同一个接口或继承同一个父类，而是通过 Java 反射机制动态生成代理类，并在调用被代理类的方法前后执行相应的操作。动态代理的优点是可以为多个被代理类生成同一个代理类，从而减少了代码量，但是它的缺点是实现相对复杂，需要了解 Java 反射机制和动态生成字节码的技术。

### 8.动态代理是如何实现的？

动态代理是一种在运行时动态生成代理类的代理方式。动态代理的常用实现方法有以下两种。

#### [#](#_1-jdk-动态代理) 1. JDK 动态代理

JDK 动态代理是一种使用 Java 标准库中的 java.lang.reflect.Proxy 类来实现动态代理的技术。在 JDK 动态代理中，被代理类必须实现一个或多个接口，并通过 InvocationHandler 接口来实现代理类的具体逻辑。

具体来说，当使用 JDK 动态代理时，需要定义一个实现 InvocationHandler 接口的类，并在该类中实现代理类的具体逻辑。然后，通过 Proxy.newProxyInstance() 方法来创建代理类的实例。该方法接受三个参数：类加载器、代理类要实现的接口列表和 InvocationHandler 对象，如下代码所示：



```java
import org.example.demo.service.AliPayService;
import org.example.demo.service.PayService;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

//动态代理：使用JDK提供的api（InvocationHandler、Proxy实现），此种方式实现，要求被代理类必须实现接口
public class PayServiceJDKInvocationHandler implements InvocationHandler {
    
    //目标对象即就是被代理对象
    private Object target;
    
    public PayServiceJDKInvocationHandler( Object target) {
        this.target = target;
    }
    
    //proxy代理对象
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //1.安全检查
        System.out.println("安全检查");
        //2.记录日志
        System.out.println("记录日志");
        //3.时间统计开始
        System.out.println("记录开始时间");

        //通过反射调用被代理类的方法
        Object retVal = method.invoke(target, args);

        //4.时间统计结束
        System.out.println("记录结束时间");
        return retVal;
    }

    public static void main(String[] args) {

        PayService target=  new AliPayService();
        //方法调用处理器
        InvocationHandler handler = 
            new PayServiceJDKInvocationHandler(target);
        //创建一个代理类：通过被代理类、被代理实现的接口、方法调用处理器来创建
        PayService proxy = (PayService) Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                new Class[]{PayService.class},
                handler
        );
        proxy.pay();
    }
}
```

JDK 动态代理的优点是实现简单，易于理解和掌握，但是它的缺点是只能代理实现了接口的类，无法代理没有实现接口的类。

#### [#](#_2-cglib-动态代理) 2. CGLIB 动态代理

CGLIB 动态代理是一种使用 CGLIB 库来实现动态代理的技术。在 CGLIB 动态代理中，代理类不需要实现接口，而是通过继承被代理类来实现代理。 具体来说，当使用 CGLIB 动态代理时，需要定义一个继承被代理类的子类，并在该子类中实现代理类的具体逻辑。然后，通过 Enhancer.create() 方法来创建代理类的实例。该方法接受一个类作为参数，表示要代理的类，如下代码所示：



```java
import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;
import org.example.demo.service.AliPayService;
import org.example.demo.service.PayService;

import java.lang.reflect.Method;

public class PayServiceCGLIBInterceptor implements MethodInterceptor {

    //被代理对象
    private Object target;
    
    public PayServiceCGLIBInterceptor(Object target){
        this.target = target;
    }
    
    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        //1.安全检查
        System.out.println("安全检查");
        //2.记录日志
        System.out.println("记录日志");
        //3.时间统计开始
        System.out.println("记录开始时间");

        //通过cglib的代理方法调用
        Object retVal = methodProxy.invoke(target, args);

        //4.时间统计结束
        System.out.println("记录结束时间");
        return retVal;
    }
    
    public static void main(String[] args) {
        PayService target=  new AliPayService();
        PayService proxy= (PayService) Enhancer.create(target.getClass(),new PayServiceCGLIBInterceptor(target));
        proxy.pay();
    }
}
```

CGLIB 动态代理的优点是可以代理没有实现接口的类，但是它的缺点是实现相对复杂，需要了解 CGLIB 库的使用方法。

#### [#](#小结) 小结

综上所述，动态代理的实现方法主要有 JDK 动态代理和 CGLIB 动态代理。JDK 动态代理中，代理类必须实现一个或多个接口，而 CGLIB 动态代理中，代理类不需要实现接口，但代理类不能是 final 类型，因为它是通过定义一个被代理类的子类来实现动态代理的，因此开发者需要根据具体的需求选择合适的技术来实现动态代理。

### 9.JDK动态代理和CGLIB有什么区别？

JDK 动态代理和 CGLIB 动态代理都是常见的动态代理实现技术，但它们有以下区别：

- JDK 动态代理基于接口，要求目标对象实现接口；CGLIB 动态代理基于类，可以代理没有实现接口的目标对象。
- JDK 动态代理使用 java.lang.reflect.Proxy 和 java.lang.reflect.InvocationHandler 来生成代理对象；CGLIB 动态代理使用 CGLIB 库来生成代理对象。
- JDK 动态代理生成的代理对象是目标对象的接口实现；CGLIB 动态代理生成的代理对象是目标对象的子类。
- JDK 动态代理性能相对较高，生成代理对象速度较快；CGLIB 动态代理性能相对较低，生成代理对象速度较慢。
- CGLIB 动态代理无法代理 final 类和 final 方法；JDK 动态代理可以代理任意类。

> PS：在 Spring 框架中，即使用了 JDK 动态代理又使用 CGLIB，默认情况下使用的是 JDK 动态代理，但是如果目标对象没有实现接口，则会使用 CGLIB 动态代理。

#### [#](#小结) 小结

简单来说，JDK 动态代理要求被代理类实现接口，而 CGLIB 要求被代理类不能是 final 修饰的最终类，在 JDK 8 以上的版本中，因为 JDK 动态代理做了专门的优化，所以它的性能要比 CGLIB 高。

### 10.Bean有几种作用域？

在 Spring 中，Bean 的作用域指的是 Bean 实例的生命周期和可见范围。

Spring 中的 Bean 作用域主要有以下几种：

#### [#](#_1-singleton) 1.singleton

singleton 是 Spring 中默认的 Bean 作用域，它表示在整个应用程序中只存在一个 Bean 实例。每次请求该 Bean 时，都会返回同一个实例。

#### [#](#_2-prototype) 2.prototype

prototype 表示每次请求该 Bean 时都会创建一个新的实例。每个实例都有自己的属性值和状态，因此它们之间是相互独立的。

#### [#](#_3-request) 3.request

request 表示在一次 HTTP 请求中只存在一个 Bean 实例。在同一个请求中，多次请求该 Bean 时都会返回同一个实例。不同的请求之间，该 Bean 的实例是相互独立的。

#### [#](#_4-session) 4.session

session 表示在一个 HTTP Session 中只存在一个 Bean 实例。在同一个 Session 中，多次请求该 Bean 时都会返回同一个实例。不同的 Session 之间，该 Bean 的实例是相互独立的。

#### [#](#_5-application) 5.application

application 表示在一个 ServletContext 中只存在一个 Bean 实例。该作用域只在 Spring ApplicationContext 上下文中有效。

#### [#](#_6-websocket) 6.websocket

websocket 表示在一个 WebSocket 中只存在一个 Bean 实例。该作用域只在 Spring ApplicationContext 上下文中有效。

### 11.单例Bean线程安全吗？

无状态的单例 Bean 是线程安全的，而有状态的单例 Bean 是非线程安全的，所以总的来说单例 Bean 还是非线程安全的。

#### [#](#什么是有状态和无状态) 什么是有状态和无状态？

有状态的 Bean 是指 Bean 中包含了状态，比如成员变量，而无状态的 Bean 是指 Bean 中不包含状态，比如没有成员变量，或者成员变量都是 final 的。

#### [#](#为什么非线程安全) 为什么非线程安全？

Spring 默认的 Bean 是单例模式，意味着容器中只有一个 Bean 实例，所有的线程都会使用并操作这个唯一的 Bean 实例，那么多个线程同时调用修改这个单例 Bean，就会产生线程安全问题。 举个例子：



```java
@Component
public class SingletonBean {
    private int counter = 0;

    public int getCounter() {
        return counter++; 
    }
}
```

这是一个简单的单例 Bean，有一个计数器，每调用一次加 1，当多个线程同时调用这个 Bean 的 getCounter() 方法时，因为 counter++ 是非原子性操作（先查询再加等），所以最终的结果就会比实际的加等次数少，这就是线程安全问题。

#### [#](#如何保证线程安全) 如何保证线程安全？

Spring 中保证单例 Bean 线程安全的手段有以下几个：

1. 变为原型 Bean：在 Bean 上添加 @Scope("prototype") 注解，将其变为多例 Bean。这样每次注入时返回一个新的实例，避免竞争。
2. 加锁：在 Bean 中对需要同步的方法或代码块添加同步锁 @Synchronized 或使用 Java 中的线程同步工具 ReentrantLock 等。
3. 使用线程安全的集合：如 Vector、Hashtable 代替 ArrayList、HashMap 等非线程安全集合。
4. 变为无状态 Bean：不在 Bean 中保存状态，让 Bean 成为无状态 Bean。无状态的 Bean 没有共享变量，自然也无须考虑线程安全问题。
5. 使用线程局部变量 ThreadLocal：在方法内部使用线程局部变量 ThreadLocal，因为 ThreadLocal 是线程独享的，所以也不存在线程安全问题。



### 12.Bean有几种注入方法？

**在 Spring 中实现依赖注入的常见方式有以下 3 种：**

1. **属性注入（Field Injection）；**
2. **Setter 注入（Setter Injection）；**
3. **构造方法注入（Constructor Injection）。**

它们的具体使用和优缺点分析如下。

#### [#](#_1-属性注入) 1.属性注入

**属性注入是我们最熟悉，也是日常开发中使用最多的一种注入方式**，它的实现代码如下：



```java
@RestController
public class UserController {
    // 属性对象
    @Autowired
    private UserService userService;

    @RequestMapping("/add")
    public UserInfo add(String username, String password) {
        return userService.add(username, password);
    }
}
```

#### [#](#_1-1-优点分析) 1.1 优点分析

**属性注入最大的优点就是实现简单、使用简单**，只需要给变量上添加一个注解（@Autowired），就可以在不 new 对象的情况下，直接获得注入的对象了（这就是 DI 的功能和魅力所在），所以它的优点就是使用简单。

#### [#](#_1-2-缺点分析) 1.2 缺点分析

然而，属性注入虽然使用简单，但也存在着很多问题，甚至编译器 Idea 都会提醒你“不建议使用此注入方式”，Idea 的提示信息如下：

!![image.png](https://javacn.site/image/1661222936883-0936a19d-392a-4f73-8a6f-a47b345fbdae.png)

属性注入的缺点主要包含以下 3 个：

1. 功能性问题：无法注入一个不可变的对象（final 修饰的对象）；
2. 通用性问题：只能适应于 IoC 容器；
3. 设计原则问题：更容易违背单一设计原则。

接下来我们一一来看。

#### [#](#缺点1-功能性问题) 缺点1：功能性问题

**使用属性注入无法注入一个不可变的对象（final 修饰的对象）**，如下图所示：

![image.png](https://javacn.site/image/1661224383171-cf2a75f2-1069-4a65-838b-f4abbc488948.png)

原因也很简单：**在 Java 中 final 对象（不可变）要么直接赋值，要么在构造方法中赋值，所以当使用属性注入 final 对象时，它不符合 Java 中 final 的使用规范，所以就不能注入成功了。**

> PS：如果要注入一个不可变的对象，要怎么实现呢？使用下面的构造方法注入即可。

#### [#](#缺点2-通用性问题) 缺点2：通用性问题

**使用属性注入的方式只适用于 IoC 框架（容器）**，如果将属性注入的代码移植到其他非 IoC 的框架中，那么代码就无效了，所以属性注入的通用性不是很好。

#### [#](#缺点3-设计原则问题) 缺点3：设计原则问题

使用属性注入的方式，因为使用起来很简单，所以开发者很容易在一个类中同时注入多个对象，而这些对象的注入是否有必要？是否符合程序设计中的单一职责原则？就变成了一个问题。 但可以肯定的是，**注入实现越简单，那么滥用它的概率也越大，所以出现违背单一职责原则的概率也越大**。 注意：**这里强调的是违背设计原则（单一职责）的可能性，而不是一定会违背设计原则**，二者有着本质的区别。

#### [#](#_2-setter-注入) 2.Setter 注入

Setter 注入的实现代码如下：



```java
@RestController
public class UserController {
    // Setter 注入
    private UserService userService;

    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService;
    }

    @RequestMapping("/add")
    public UserInfo add(String username, String password) {
        return userService.add(username, password);
    }
}
```

#### [#](#优缺点分析) 优缺点分析

从上面代码可以看出，Setter 注入比属性注入要麻烦很多。 **要说 Setter 注入有什么优点的话，那么首当其冲的就是它完全符合单一职责的设计原则，因为每一个 Setter 只针对一个对象**。 但它的缺点也很明显，它的缺点主要体现在以下 2 点：

1. 不能注入不可变对象（final 修饰的对象）；
2. 注入的对象可被修改。

接下来我们一一来看。

#### [#](#缺点1-不能注入不可变对象) 缺点1：不能注入不可变对象

使用 Setter 注入依然不能注入不可变对象，比如以下注入会报错：

![image.png](https://javacn.site/image/1661245141492-a84d361d-c7da-46d4-9c0d-2ebd0049193e.png)

#### [#](#缺点2-注入对象可被修改) 缺点2：注入对象可被修改

Setter 注入提供了 setXXX 的方法，意味着你可以在任何时候、在任何地方，通过调用 setXXX 的方法来改变注入对象，所以 **Setter 注入的问题是，被注入的对象可能随时被修改**。

#### [#](#_3-构造方法注入) 3.构造方法注入

**构造方法注入是 Spring 官方从 4.x 之后推荐的注入方式**，它的实现代码如下：



```java
@RestController
public class UserController {
    // 构造方法注入
    private UserService userService;

    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }

    @RequestMapping("/add")
    public UserInfo add(String username, String password) {
        return userService.add(username, password);
    }
}
```

当然，**如果当前的类中只有一个构造方法，那么 @Autowired 也可以省略**，所以以上代码还可以这样写：



```java
@RestController
public class UserController {
    // 构造方法注入
    private UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @RequestMapping("/add")
    public UserInfo add(String username, String password) {
        return userService.add(username, password);
    }
}
```

#### [#](#优点分析) 优点分析

构造方法注入相比于前两种注入方法，它可以注入不可变对象，并且它只会执行一次，也不存在像 Setter 注入那样，被注入的对象随时被修改的情况，它的优点有以下 4 个：

1. 可注入不可变对象；
2. 注入对象不会被修改；
3. 注入对象会被完全初始化；
4. 通用性更好。

接下来我们一一来看。

#### [#](#优点1-注入不可变对象) 优点1：注入不可变对象

使用构造方法注入可以注入不可变对象，如下代码所示：

![image.png](https://javacn.site/image/1661245255409-60844dd2-2c82-422a-b5e7-c9ba4d8258cb.png)

#### [#](#优点2-注入对象不会被修改) 优点2：注入对象不会被修改

构造方法注入不会像 Setter 注入那样，**构造方法在对象创建时只会执行一次，因此它不存在注入对象被随时（调用）修改的情况。**

#### [#](#优点3-完全初始化) 优点3：完全初始化

因为依赖对象是在构造方法中执行的，而构造方法是在对象创建之初执行的，因此被注入的对象在使用之前，会被完全初始化，这也是构造方法注入的优点之一。

#### [#](#优点4-通用性更好) 优点4：通用性更好

构造方法和属性注入不同，构造方法注入可适用于任何环境，无论是 IoC 框架还是非 IoC 框架，构造方法注入的代码都是通用的，所以它的通用性更好。

#### [#](#小结) 小结

依赖注入的常见实现方式有 3 种：属性注入、Setter 注入和构造方法注入。其中属性注入的写法最简单，所以日常项目中使用的频率最高，但它的通用性不好；而 Spring 官方推荐的是构造方法注入，它可以注入不可变对象，其通用性也更好，如果是注入可变对象，那么可以考虑使用 Setter 注入。



### 13.Autowired底层如何实现？

@Autowired 是 Spring 框架中常用的注解之一，它可以自动装配 Bean，使得开发人员可以更加方便地使用 Spring 框架，但 @Autowired 底层是如何实现的呢？

**Spring 中的 @Autowired 注解是通过依赖注入（DI）实现的**，依赖注入是一种设计模式，它将对象的创建和依赖关系的管理从应用程序代码中分离出来，使得应用程序更加灵活和可维护。

具体来说，当 Spring 容器启动时，它会扫描应用程序中的所有 Bean，并将它们存储在一个 BeanFactory 中。当应用程序需要使用某个 Bean 时，Spring 容器会自动将该 Bean 注入到应用程序中。

但再往底层说，DI 是通过 Java 反射机制实现的。具体来说，当 Spring 容器需要注入某个 Bean 时，它会使用 Java 反射机制来查找符合条件的 Bean，并将其注入到应用程序中。

所以说，@Autowired 注解是通过 DI 的方式，底层通过 Java 的反射机制来实现的。



### 14.@Autowired和@Resource有什么区别？

在 Spring 中，@Autowired 和 @Resource 都是用于注入 Bean 对象的注解。它们的作用类似，但是有以下几点区别。

#### [#](#_1-来源不同) 1.来源不同

**@Autowired 和 @Resource 来自不同的“父类”，其中 @Autowired 是 Spring 定义的注解，而 @Resource 是 Java 定义的注解，它来自于 JSR-250（Java 250 规范提案）。**

> 小知识：JSR 是 Java Specification Requests 的缩写，意思是“Java 规范提案”。任何人都可以提交 JSR 给 Java 官方，但只有最终确定的 JSR，才会以 JSR-XXX 的格式发布，如 JSR-250，而被发布的 JSR 就可以看作是 Java 语言的规范或标准。

#### [#](#_2-依赖查找顺序不同) 2.依赖查找顺序不同

依赖注入的功能，是通过先在 Spring IoC 容器中查找对象，再将对象注入引入到当前类中。而查找有分为两种实现：按名称（byName）查找或按类型（byType）查找，其中 @Autowired 和 @Resource 都是既使用了名称查找又使用了类型查找，但二者进行查找的顺序却截然相反。

#### [#](#_2-1-autowired-查找顺序) 2.1 @Autowired 查找顺序

**@Autowired 是先根据类型（byType）查找，如果存在多个 Bean 再根据名称（byName）进行查找**，它的具体查找流程如下：

![image.png](https://javacn.site/image/1661336351795-8853fa15-58de-4654-9ce8-de1c337ded4a.png)

关于以上流程，可以通过查看 Spring 源码中的 org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor#postProcessPropertyValues 实现分析得出，源码执行流程如下图所示：

![image.png](https://javacn.site/image/1661342216350-632b984b-9d62-43cc-83f3-4c90cc6868d8.png)

#### [#](#_2-2-resource-查找顺序) 2.2 @Resource 查找顺序

**@Resource 是先根据名称查找，如果（根据名称）查找不到，再根据类型进行查找**，它的具体流程如下图所示：

![image.png](https://javacn.site/image/1661336664586-fda70e38-4426-45f9-a3b4-21365fc4771c.png)

关于以上流程可以在 Spring 源码的 org.springframework.context.annotation.CommonAnnotationBeanPostProcessor#postProcessPropertyValues 中分析得出。虽然 @Resource 是 JSR-250 定义的，但是由 Spring 提供了具体实现，它的源码实现如下：

![image.png](https://javacn.site/image/1661342304670-11383d4d-b51f-4f4f-b2ee-0c588bef817d.png)

#### [#](#_2-3-查找顺序小结) 2.3 查找顺序小结

由上面的分析可以得出：

- **@Autowired 先根据类型（byType）查找，如果存在多个（Bean）再根据名称（byName）进行查找；**
- **@Resource 先根据名称（byName）查找，如果（根据名称）查找不到，再根据类型（byType）进行查找。**

#### [#](#_3-支持的参数不同) 3.支持的参数不同

@Autowired 和 @Resource 在使用时都可以设置参数，比如给 @Resource 注解设置 name 和 type 参数，实现代码如下：



```java
@Resource(name = "userinfo", type = UserInfo.class)
private UserInfo user;
```

但**二者支持的参数以及参数的个数完全不同，其中 @Autowired 只支持设置一个 required 的参数，而 @Resource 支持 7 个参数**，支持的参数如下图所示：

![image.png](https://javacn.site/image/1661340802624-50dd0f3b-0bc5-45e8-b919-c0744547a644.png)

![image.png](https://javacn.site/image/1661340740782-0f671f38-37ae-4ada-9d39-6b4d19a2181a.png)

#### [#](#_4-依赖注入的支持不同) 4.依赖注入的支持不同

@Autowired 和 @Resource 支持依赖注入的用法不同，常见依赖注入有以下 3 种实现：

1. 属性注入
2. 构造方法注入
3. Setter 注入

这 3 种实现注入的实现代码如下。

**a) 属性注入**



```java
@RestController
public class UserController {
    // 属性注入
    @Autowired
    private UserService userService;

    @RequestMapping("/add")
    public UserInfo add(String username, String password) {
        return userService.add(username, password);
    }
}
```

**b) 构造方法注入**



```java
@RestController
public class UserController {
    // 构造方法注入
    private UserService userService;

    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }

    @RequestMapping("/add")
    public UserInfo add(String username, String password) {
        return userService.add(username, password);
    }
}
```

**c) Setter 注入**



```java
@RestController
public class UserController {
    // Setter 注入
    private UserService userService;

    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService;
    }

    @RequestMapping("/add")
    public UserInfo add(String username, String password) {
        return userService.add(username, password);
    }
}
```

其中，**@Autowired 支持属性注入、构造方法注入和 Setter 注入，而 @Resource 只支持属性注入和 Setter 注入**，当使用 @Resource 实现构造方法注入时就会提示以下错误：

![image.png](https://javacn.site/image/1661341346342-317ea714-5825-4dc9-96a0-f0a118158380.png)

#### [#](#_5-编译器提示不同) 5.编译器提示不同

**当使用 IDEA 专业版在编写依赖注入的代码时，如果注入的是 Mapper 对象，那么使用 @Autowired 编译器会提示报错信息**，报错内容如下图所示：

![image.png](https://javacn.site/image/1661341520045-d7a16c66-55af-4052-9387-0d0ddee4e3d4.png)

虽然 IDEA 会出现报错信息，但程序是可以正常执行的。 然后，我们再**将依赖注入的注解更改为 @Resource 就不会出现报错信息了**，具体实现如下：

![image.png](https://javacn.site/image/1661341563376-2e74ab93-0779-4d65-9260-520119657419.png)

#### [#](#小结) 小结

@Autowired 和 @Resource 都是用来实现依赖注入的注解（在 Spring/Spring Boot 项目中），但二者却有着 5 点不同：

1. 来源不同：@Autowired 来自 Spring 框架，而 @Resource 来自于（Java）JSR-250；
2. 依赖查找的顺序不同：@Autowired 先根据类型再根据名称查询，而 @Resource 先根据名称再根据类型查询；
3. 支持的参数不同：@Autowired 只支持设置 1 个参数，而 @Resource 支持设置 7 个参数；
4. 依赖注入的用法支持不同：@Autowired 既支持构造方法注入，又支持属性注入和 Setter 注入，而 @Resource 只支持属性注入和 Setter 注入；
5. 编译器 IDEA 的提示不同：当注入 Mapper 对象时，使用 @Autowired 注解编译器会提示错误，而使用 @Resource 注解则不会提示错误



### 15.说一下Bean的生命周期？

在 Spring 中，Bean 的生命周期指的是 Bean 实例从创建到销毁的整个过程。Spring 容器负责管理 Bean 的生命周期，包括实例化、属性赋值、初始化、销毁等过程。

Bean 的生命周期可以分为以下几个阶段：

#### [#](#_1-实例化) 1. 实例化

在 Spring 容器启动时，会根据配置文件或注解等方式创建 Bean 的实例，**也就是说实例化就是为 Bean 对象分配内存空间**。根据 Bean 的作用域不同，实例化的方式也不同。例如，singleton 类型的 Bean 在容器启动时就会被实例化，而 prototype 类型的 Bean 则是在每次请求时才会被实例化。

#### [#](#_2-属性赋值) 2. 属性赋值

在 Bean 实例化后，Spring 容器会自动将配置文件或注解中指定的属性值注入到 Bean 中。属性注入可以通过构造函数注入、Setter 方法注入、注解注入等方式实现。

#### [#](#_3-初始化) 3. 初始化

在属性注入完成后，Spring 容器会调用 Bean 的初始化方法。Bean 的初始化方法可以通过实现 InitializingBean 接口、@PostConstruct 注解等方式实现。在初始化方法中，可以进行一些初始化操作，例如建立数据库连接、加载配置文件等。

#### [#](#_4-使用) 4. 使用

在 Bean 初始化完成后，Bean 就可以被应用程序使用了。在应用程序中，可以通过 Spring 容器获取 Bean 的实例，并调用 Bean 的方法。

#### [#](#_5-销毁) 5. 销毁

在应用程序关闭时，Spring 容器会自动销毁所有的 Bean 实例。Bean 的销毁方法可以通过实现 DisposableBean 接口、@PreDestroy 注解等方式实现。在销毁方法中，可以进行一些清理操作，例如释放资源、关闭数据库连接等。

#### [#](#小结) 小结

综上所述，Spring 中的 Bean 生命周期包括实例化、属性赋值、初始化、使用和销毁等阶段。开发者可以通过实现接口或注解等方式来管理 Bean 的生命周期。

### 16.BeanFactory和FactoryBean有什么区别？

BeanFactory 和 FactoryBean 都是 Spring 框架中两个核心的概念，虽然它们都与对象创建和管理有关，但它们在功能和使用方式上存在着一些重要的区别。

#### [#](#beanfactory) BeanFactory

BeanFactory 是 Spring 框架的基本容器，负责管理和创建应用程序中的对象。它是一个工厂模式的实现，可以根据配置信息创建和管理各种类型的 Java 对象。BeanFactory 的主要职责是实例化 Bean、处理 Bean 之间的依赖关系、注入属性以及在需要时销毁 Bean。

BeanFactory 使用延迟初始化策略，即只有在请求获取 Bean 实例时才会进行实例化。这种方式可以减少资源消耗，特别是在应用程序启动时有大量的Bean需要创建时。BeanFactory 使用配置文件（如 XML）或注解来定义 Bean 和它们之间的关系。

BeanFactory 简单实现代码如下：



```java
// 得到 BeanFactory 对象
BeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource("spring-config.xml"));
// 使用 BeanFactory 得到 User 对象
beanFactory.getBean("user");
```

#### [#](#factorybean) FactoryBean

FactoryBean 是一个特殊的 Bean，它实现了 Spring 的 FactoryBean 接口。与普通的 Bean 不同，FactoryBean 负责创建其他 Bean 实例。它是一种更加灵活和可扩展的机制，可以通过编程的方式动态地创建和配置Bean。

FactoryBean 接口定义了三个方法：getObject()、getObjectType() 和 isSingleton()。getObject() 方法返回由 FactoryBean 创建的 Bean 实例，getObjectType() 方法返回创建的Bean的类型，而 isSingleton() 方法用于指示创建的 Bean 是否是单例。

FactoryBean 的实现类可以自定义创建和管理 Bean 的逻辑。例如，它可以根据条件选择性地创建不同的实例，或者在创建 Bean 之前进行一些初始化操作。FactoryBean 通常在 Spring 配置文件中配置，并由 BeanFactory 负责实例化和管理。

FactoryBean 简单示例如下：



```java
import org.springframework.beans.factory.FactoryBean;

public class MyBeanFactory implements FactoryBean<MyBean> {

    @Override
    public MyBean getObject() throws Exception {
        // 在这里定义创建 Bean 的逻辑
        MyBean myBean = new MyBean();
        // 进行一些初始化操作
        myBean.setName("Example Bean");
        // 返回创建的 Bean 实例
        return myBean;
    }

    @Override
    public Class<?> getObjectType() {
        // 返回创建的 Bean 的类型
        return MyBean.class;
    }

    @Override
    public boolean isSingleton() {
        // 指示创建的 Bean 是否是单例
        return true;
    }
}
```

在上面的示例中，我们创建了一个名为 MyBeanFactory 的类，它实现了 FactoryBean 接口。在 getObject() 方法中，我们定义了创建 MyBean 实例的逻辑，并在此处进行了一些初始化操作。getObjectType() 方法返回了我们要创建的 Bean 的类型，即 MyBean.class。最后，我们通过 isSingleton() 方法指示创建的 Bean 是单例还是多例。

之后，我们需要在 Spring 配置文件中，我们可以将 MyBeanFactory 配置为一个 Bean，并使用它来创建我们需要的 Bean 实例。示例配置如下：



```xml
<bean id="myBeanFactory" class="com.example.MyBeanFactory"/>
<bean id="myBean" factory-bean="myBeanFactory" factory-method="getObject"/>
```

在上述配置中，我们首先定义了一个名为 myBeanFactory 的 Bean，它是我们实现的 MyBeanFactory 类的实例。接下来，我们使用 factory-bean 属性指定 myBeanFactory 作为工厂 Bean，并使用 factory-method 属性指定 getObject 方法来创建我们需要的 Bean 实例，即 myBean 类。

#### [#](#区别) 区别

BeanFactory 是 Spring 的基本容器，用于创建和管理 Bean 实例，而 FactoryBean 是一个特殊的 Bean，用于创建其他 Bean 实例。

下面是 BeanFactory 和 FactoryBean 之间的一些关键区别：

1. **功能**：BeanFactory 是一个容器，负责管理和创建 Bean 实例，处理依赖关系和属性注入等操作。FactoryBean 是一个接口，定义了创建 Bean 的规范和逻辑，它负责创建其他 Bean 实例。
2. **使用方式**：BeanFactory 使用配置文件或注解来定义 Bean 和它们之间的关系，它使用延迟初始化策略，即只有在需要时才创建 Bean 实例。FactoryBean 通常在 Spring 配置文件中配置，并由 BeanFactory 负责实例化和管理。
3. **创建的对象**：BeanFactory 创建和管理普通的 Bean 实例，而 FactoryBean 创建其他 Bean 实例。
4. **灵活性**：FactoryBean 具有更高的灵活性，因为它允许自定义的逻辑来创建和配置 Bean 实例。FactoryBean 的实现类可以根据特定的条件选择性地创建不同的 Bean 实例，或者在创建 Bean 之前进行一些初始化操作。这使得 FactoryBean 在某些情况下比 BeanFactory 更加强大和可扩展。
5. **返回类型**：BeanFactory 返回的是 Bean 实例本身，而 FactoryBean 返回的是由 FactoryBean 创建的 Bean 实例。因此，当使用 FactoryBean 时，需要通过调用 getObject() 方法来获取创建的 Bean 实例。

#### [#](#小结) 小结

BeanFactory 和 FactoryBean 是 Spring 框架中的两个关键概念，用于创建和管理 Bean 实例。BeanFactory 是 Spring 的基本容器，负责创建和管理 Bean 实例的，而 FactoryBean 是一个特殊的 Bean，它实现了 FactoryBean 接口，负责创建其他 Bean 实例，并提供一些初始化 Bean 的设置。

### 17.Spring中使用了哪些设计模式？

Spring 是一个非常流行的 Java 开发框架，它使用了很多设计模式来实现其功能，以下是 Spring 中包含的一些常见的设计模式。

#### [#](#_1-工厂模式) 1.工厂模式

工厂模式是一种创建型设计模式，它提供了一种创建对象的方式，使得应用程序可以更加灵活和可维护。在 Spring 中，FactoryBean 就是一个工厂模式的实现，使用它的工厂模式就可以创建出来其他的 Bean 对象。

#### [#](#_2-单例模式) 2.单例模式

单例模式是一种创建型设计模式，它保证一个类只有一个实例，并提供了一个全局访问点。在 Spring 中，Bean 默认是单例的，这意味着每个 Bean 只会被创建一次，并且可以在整个应用程序中共享。

#### [#](#_3-代理模式) 3.代理模式

代理模式是一种结构型设计模式，它允许开发人员在不修改原有代码的情况下，向应用程序中添加新的功能。在 Spring AOP（面向切面编程）就是使用代理模式的实现，它允许开发人员在方法调用前后执行一些自定义的操作，比如日志记录、性能监控等。

#### [#](#_4-观察者模式) 4.观察者模式

观察者模式是一种行为型设计模式，它定义了一种一对多的依赖关系，使得当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知并自动更新。Spring 事件驱动模型使用观察者模式，ApplicationEventPublisher 事件发布者将事件发布给 ApplicationEventMulticaster 事件广播器，该广播器将事件派发给 @EventListener 注解的事件监听者。

#### [#](#_5-模板方法模式) 5.模板方法模式

模板方法模式是一种行为型设计模式，它定义了一个算法的骨架，将一些步骤延迟到子类中实现。在 Spring 中，JdbcTemplate 就是一个模板方法模式的实现，它提供了一种简单的方式来执行 SQL 查询和更新操作。

#### [#](#_6-适配器模式) 6.适配器模式

适配器模式是一种结构型设计模式，它允许开发人员将一个类的接口转换成另一个类的接口，以满足客户端的需求。在 Spring 中，适配器模式常用于将不同类型的对象转换成统一的接口，比如将 Servlet API 转换成 Spring MVC 的控制器接口。

#### [#](#_7-策略模式) 7.策略模式

策略模式是一种行为型设计模式，它定义了一系列算法，并将每个算法封装起来，使得它们可以互相替换。Spring 中的 TaskExecutor，TaskExecutor 提供了很多实现，比如以下这些：

- SyncTaskExecutor：直接在调用线程中执行任务,没有真正的异步；
- SimpleAsyncTaskExecutor：使用单线程池异步执行任务；
- ConcurrentTaskExecutor：使用线程池异步执行任务；
- SimpleTransactionalTaskExecutor：支持事务的 SimpleAsyncTaskExecutor。 这样，我们可以根据自己的需求选择不同的实现策略，使用策略模式的好处有以下这些：

1. 可以在不修改原代码的基础上选择不同的算法或策略；
2. 可减少程序中的条件语句，根据环境改变选择合适的策略；
3. 扩展性好，如果有新的策略出现，只需要创建一个新的策略类，无须修改原代码。

### 18.Spring和SpringBoot有什么区别？

Spring 和 Spring Boot 都是 Java 开发中非常流行的框架，它们都可以用于构建企业级应用程序。虽然它们都是 Spring 框架的一部分，但是它们之间还是有一些区别的。

#### [#](#spring) Spring

Spring 是一个轻量级的开源框架，它提供了一种简单的方式来构建企业级应用程序。Spring 框架的核心是 IoC（Inversion of Control）和 AOP（Aspect Oriented Programming）两个概念。

IoC 是一种设计模式，它将对象的创建和依赖关系的管理从应用程序代码中分离出来，使得应用程序更加灵活和可维护。

AOP 是一种编程范式，它允许开发人员在不修改原有代码的情况下，向应用程序中添加新的功能。

Spring 框架提供了很多模块，包括核心容器、数据访问、Web、AOP、消息、测试等。开发人员可以根据自己的需求选择合适的模块来构建应用程序。

#### [#](#spring-boot) Spring Boot

Spring Boot 本质上是 Spring 框架的延伸和扩展，它的诞生是为了简化 Spring 框架初始搭建以及开发的过程，使用它可以不再依赖 Spring 应用程序中的 XML 配置，为更快、更高效的开发 Spring 提供更加有力的支持。 Spring Boot 也提供了很多特性，包括自动配置、嵌入式 Web 服务器、健康检查、度量指标、安全性等。开发人员可以通过使用 Spring Boot Starter 来快速集成常用的第三方库和框架，比如 Spring Data、Spring Security、MyBatis、Redis 等。

#### [#](#小结) 小结

Spring 和 Spring Boot 的区别在于它们的目标和用途不同。Spring 是一个轻量级的开源框架，它提供了一种简单的方式来构建企业级应用程序。Spring Boot 则是 Spring 框架的延伸和扩展，它提供了一种快速构建应用程序的方式。开发人员可以通过使用 Spring Boot Starter 来快速集成常用的第三方库和框架，使得开发人员可以快速构建出一个可运行的应用程序。

### 19.SpringBoot有什么优点？

Spring Boot 是一个基于 Spring 框架的快速开发框架，它有以下优点：

1. **简化配置**： Spring Boot 采用约定大于配置的原则，提供了自动配置的特性，大部分情况下无需手动配置，可以快速启动和运行应用程序。同时，Spring Boot 提供了统一的配置模型，集成了大量常用的第三方库和框架，简化了配置过程。
2. **内嵌服务器**： Spring Boot 集成了常用的内嵌式服务器，如 Tomcat、Jetty 和 Undertow 等。这意味着不再需要单独安装和配置外部服务器，可以直接运行 Spring Boot 应用程序，简化了部署和发布过程。
3. **自动装配**： Spring Boot 提供了自动装配机制，根据应用程序的依赖关系和配置信息，智能地自动配置 Spring 的各种组件和功能，大大减少了开发人员的手动配置工作，提高了开发效率。
4. **起步依赖**： Spring Boot 引入了起步依赖（Starter Dependencies）的概念，它是一种可用于快速集成相关技术栈的依赖项集合。起步依赖能够自动处理依赖冲突和版本兼容性，并提供了默认的配置和依赖管理，简化了构建和管理项目的过程。
5. **自动化监控和管理**： Spring Boot 集成了 Actuator 模块，提供了对应用程序的自动化监控、管理和运维支持。通过 Actuator，可以获取应用程序的健康状况、性能指标、配置信息等，方便运维人员进行故障排查和性能优化。
6. **丰富的生态系统**： Spring Boot 建立在 Spring Framework 的基础上，可以无缝集成 Spring 的各种功能和扩展，如 Spring Data、Spring Security、Spring Integration 等。同时，Spring Boot 还提供了大量的第三方库和插件，可以方便地集成其他技术栈，构建全栈式应用程序。
7. **可扩展性和灵活性**： 尽管 Spring Boot 提供了很多自动化的功能和约定，但它也保持了良好的可扩展性和灵活性。开发人员可以根据自己的需求进行自定义配置和扩展，以满足特定的业务需求。

综上所述，Spring Boot 是一个强大而又灵活的开发框架，具有简化配置、快速开发、自动化监控、微服务支持等诸多优点。它极大地提高了开发效率、降低了开发成本，并且在行业中得到了广泛的认可和应用。

### 20.什么是SpringBoot的自动装配？

Spring Boot 的自动装配（Auto-configuration）是 Spring Boot 框架的核心特性之一。通过自动装配可以自动配置和加载 Spring Boot 所需的各种组件和功能，从而大大的减少开发人员手动配置的工作。

在传统的 Spring 应用程序中，我们需要手动配置各种组件，如数据源、Web 容器、事务管理器等。这些配置需要编写大量的 XML 配置文件或 Java 配置类，增加了开发的工作量和复杂性。而 Spring Boot 的自动装配通过约定大于配置的原则，根据项目的依赖和配置信息，自动进行配置，使得开发人员无需进行大量的手动配置。

#### [#](#自动装配工作原理) 自动装配工作原理

Spring Boot 在启动时，会检索所有的 Spring 模块，找到符合条件的配置并应用到应用上下文中。这个过程发生在 SpringApplication 这个类中。

Spring Boot 自动装配主要依靠两部分：

1. SpringFactoriesLoader 驱动：在启动过程中会加载 META-INF/spring.factories 配置文件，获取自动装配相关的配置类信息。
2. 条件装配：Spring Boot 不会永远都自动装配，它会根据类路径下是否存在某个名称符合命名规则的自动装配类来决定是否进行自动装配。这就是条件装配，通过 @Conditional 条件注解完成。

#### [#](#自动装配流程) 自动装配流程

Spring Boot 自动装配执行流程如下：

1. Spring Boot 启动时会创建一个 SpringApplication 实例，该实例存储了应用相关信息，它负责启动并运行应用。
2. 实例化 SpringApplication 时，会自动装载 META-INF/spring.factories 中配置的自动装配类。
3. SpringApplication 实例调用 run() 方法启动应用。
4. 在 run() 方法中，实例会创建默认的应用上下文 Environment 以及 ApplicationContext。
5. SpringApplication 会通过 ListableBeanFactory 加载应用上下文 ApplicationContext 中的所有 BeanDefinition。
6. 在 BeanDefinition 加载过程中，SpringApplication 会检测是否存在基于 @Conditional 条件装配注解的自动装配类。
7. 如果存在且 @Conditional 条件校验成功，则会装配这些自动装配类。
8. 这些自动装配类通过 @EnableAutoConfiguration、@Configuration 等注解，装配默认的 Spring Bean。
9. 装配完成后，Spring Boot 将启动应用，这里会启动嵌入的 Web 服务器，如 Tomcat 并发布 Web 应用。
10. 发布完成，Spring Boot 应用启动成功。

#### [#](#小结) 小结

自动装配是 Spring Boot 框架中的一个重要特性，它可以帮助开发人员更加方便地使用 Spring Boot 框架，提高开发效率，保证应用程序的正确性和稳定性。

