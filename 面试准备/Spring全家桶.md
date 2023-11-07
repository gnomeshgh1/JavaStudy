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









