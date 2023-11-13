##                                                                     MyBatis

### 1.MyBatis有什么优缺点？

MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生 SQL 查询，将接口和 Java 的实体类映射成数据库中的记录。

#### [#](#优点分析) 优点分析

1. **灵活**：MyBatis 不会对应用程序或数据库的现有设计强加任何限制。它使用简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO 为数据库中的记录。
2. **易于学习**：MyBatis 相对于其他 ORM 框架来说，学习曲线较低，因为它不需要开发人员学习新的领域特定语言（DSL）或复杂的 API。
3. **性能**：MyBatis 可以通过缓存和批量更新等技术来提高性能。
4. **可定制性**：MyBatis 提供了许多插件和扩展点，例如 Plugin 扩展点，它是用于拦截 MyBatis 执行前后操作的接口，可以通过实现该接口来自定义 MyBatis 的行为，这样开发者就可以根据需要进行定制。

#### [#](#缺点分析) 缺点分析

MyBatis 有以下缺点：

1. **SQL 语句依赖**：MyBatis 需要手动编写 SQL 语句，这意味着开发人员需要具备一定的 SQL 知识。此外，如果数据库模式发生变化，需要手动修改 SQL 语句，这可能会导致一些问题。
2. **XML 配置文件冗长**：MyBatis 的配置文件通常比较冗长，这可能会导致一些维护问题。此外，如果使用注解配置，代码可能会变得混乱。
3. **缺乏自动化创建**：相比于其他 ORM 框架，MyBatis 缺乏自动化。例如，它不支持自动创建表和字段。

#### [#](#小结) 小结

MyBatis 是一款灵活、易于学习、高性能和可定制的持久层框架，但它也有一些缺点，如 SQL 语句依赖、不支持自动创建表和字段等问题。

### 2.MyBatis和hibernate有什么区别？

MyBatis 和 Hibernate 是两个常用的 Java 持久化框架，它们在实现方式和使用方法上存在一些区别：

1. **对象关系映射（ORM）方式**：Hibernate 是一个全自动的 ORM 框架，通过对象关系映射技术，将数据库表与 Java 对象之间的映射关系自动管理，不需要手动编写 SQL 语句。而 MyBatis 是一个半自动的 ORM 框架，需要开发者手动编写和管理 SQL 语句。
2. **查询语言**：Hibernate 使用 Hibernate Query Language（HQL）或 Criteria API 进行数据库查询，它们是面向对象的查询语言，类似于 SQL 语法。而 MyBatis 直接使用原生的 SQL 语句进行查询，可以更灵活地进行定制和优化。
3. **学习曲线**：由于 Hibernate 提供了全自动的 ORM 特性，对于开发者来说，学习成本相对较高，需要理解和掌握其复杂的特性和概念。相比之下，MyBatis 更接近于传统的 JDBC 编程模型，学习曲线相对较低，开发者可以更灵活地控制和优化 SQL 语句。
4. **性能控制**：由于 MyBatis 需要手动编写和优化 SQL 语句，开发者可以更精确地控制查询的性能。而 Hibernate 在某些情况下可能会生成复杂的 SQL 语句，性能方面可能不如 MyBatis 灵活。

MyBatis 和 Hibernate 都是优秀的持久层框架，它们的主要区别在于对象关系映射、查询语言、学习曲线、性能控制等方面。

### 3.说一下MyBatis执行流程？

MyBatis 的执行流程可以分为以下几个步骤：

1. **读取配置文件**：读取 MyBatis 配置文件（通常是 mybatis-config.xml），该文件包含了 MyBatis 的全局配置信息，如数据库连接信息、类型别名、插件等。
2. **创建 SqlSessionFactory**：SqlSessionFactory 是 MyBatis 的核心接口之一，它负责创建 SqlSession 对象。SqlSessionFactory 可以通过 XML 配置文件或 Java 代码进行配置。
3. **创建 SqlSession**：SqlSession 是 MyBatis 的另一个核心接口，它负责与数据库进行交互。SqlSession 提供了许多方法，如 selectOne、selectList、insert、update、delete 等，可以执行 SQL 语句并返回结果。
4. **执行 SQL 语句**：SqlSession 会根据 Mapper 接口中的方法名和参数，找到对应的 SQL 语句并执行。在执行 SQL 语句之前，MyBatis 会将 `#{}`替换为实际的参数值，并将 `${}` 替换为实际的 SQL 语句。
5. **返回结果**：执行 SQL 语句后，MyBatis 会将结果映射为 Java 对象并返回。MyBatis 提供了许多映射方式，如基于 XML 的映射、注解映射、自定义映射等。

总之，MyBatis 的执行流程包括读取配置文件、创建 SqlSessionFactory、创建 SqlSession、执行 SQL 语句和返回结果等步骤。

### 4.${}和#{}有什么区别？

`${}` 和 `#{}` 在 MyBatis 中都是用于 SQL 参数替换的符号，它们的区别主要体现在以下几个方面：

1. **功能不同**：`${}` 是直接替换，而 `#{}`是预处理；
2. **使用场景不同**：普通参数使用 `#{}`，如果传递的是 SQL 命令或 SQL 关键字，需要使用 `${}`，但在使用前一定要做好安全验证；
3. **安全性不同**：使用 `${}` 存在安全问题，如 SQL 注入，而 `#{}` 则不存在安全问题。

所以，`${}` 和 `#{}` 在 MyBatis 中都是用于 SQL 参数替换的符号，然而使用 `${}` 可能存在安全问题，所以在能用 `#{}` 时，尽量使用 `#{}` 时，否则可以考虑在充分验证了参数安全之后使用 `${}`。

### 5.什么是SQL注入？

SQL 注入即是指应用程序对用户输入数据的合法性没有判断或过滤不严，攻击者可以在应用程序中事先定义好的查询语句的结尾上添加额外的 SQL 语句，在管理员不知情的情况下实现非法操作，以此来实现欺骗数据库服务器执行非授权的任意查询，从而进一步得到相应的数据信息。 比如以下代码：



```java
<select id="isLogin" resultType="com.example.demo.model.User">
    select * from userinfo where username='${name}' and password='${pwd}'
</select>
```

sql 注入代码：“' or 1='1”，如下图所示：

![img](https://javacn.site/image/1638872908997-7d1ae9f2-477c-4736-b609-327a1a4e8489.png)

从上述结果可以看出，以上程序在应用程序不知情的情况下实现非法操作，以此来实现欺骗数据库服务器执行非授权的敏感数据。

### 6.什么是动态SQL？

动态 SQL 是指可以根据不同的参数信息来动态拼接的不确定的 SQL 叫做动态 SQL，MyBatis 动态 SQL 的主要元素有：if、choose/when/otherwise、trim、where、set、foreach 等。

以 if 标签的使用为例：



```java
<select id="findUser" parameterType="com.interview.entity.User" resultType="com.interview.entity.User">
      select * from t_user where 1=1
      <if test="id!=null">
        and id = #{id}
      </if>
      <if test="username!=null">
        and username = #{username}
      </if>
      <if test="password!=null">
        and password = #{password}
      </if>
</select>
```

当调用此方法时，如果传递 id 参数了，那么生成的 SQL 是这样的：

> select * from t_user where 1=1 and id=n

如果不传递 id 参数，那么生成的 SQL 是这样的：

> select * from t_user where 1=1

这就是动态 SQL，根据不同的参数生成不同的 SQL 语句。

### 7.什么是MyBatis二级缓存？

MyBatis 二级缓存是 MyBatis 中非常重要的一个特性，它的作用是减少数据库的查询次数，提高系统性能。

二级缓存包含两级缓存：一级缓存和二级缓存。

1. 一级缓存是 SqlSession 级别的，是 MyBatis 自带的缓存功能，并且无法关闭，因此当有两个 SqlSession 访问相同的 SQL 时，一级缓存也不会生效，需要查询两次数据库；
2. 二级缓存是 Mapper 级别的，只要是同一个 Mapper，无论使用多少个 SqlSession 来操作，数据都是共享的，多个不同的 SqlSession 可以共用二级缓存，MyBatis 二级缓存默认是关闭的，需要使用时可手动开启，二级缓存也可以使用第三方的缓存，比如，使用 Ehcache 作为二级缓存。

#### [#](#一级缓存-vs-二级缓存) 一级缓存 VS 二级缓存

一级缓存和二级缓存的区别如下：

1. 一级缓存是 SqlSession 级别的缓存，它的作用域是同一个 SqlSession，同一个 SqlSession 中的多次查询会共享同一个缓存。二级缓存是 Mapper 级别的缓存，它的作用域是同一个 Mapper，同一个 Mapper 中的多次查询会共享同一个缓存。
2. 一级缓存是默认开启的，不需要手动配置。二级缓存需要手动配置，需要在 Mapper.xml 文件中添加 <cache> 标签。
3. 一级缓存的生命周期是和 SqlSession 一样长的，当 SqlSession 关闭时，一级缓存也会被清空。二级缓存的生命周期是和 MapperFactory 一样长的，当应用程序关闭时，二级缓存也会被清空。
4. 一级缓存只能用于同一个 SqlSession 中的多次查询，不能用于跨 SqlSession 的查询。二级缓存可以用于跨 SqlSession 的查询，多个 SqlSession 可以共享同一个二级缓存。
5. 一级缓存是线程私有的，不同的 SqlSession 之间的缓存数据不会互相干扰。二级缓存是线程共享的，多个 SqlSession 可以共享同一个二级缓存，需要考虑线程安全问题。

#### [#](#开启二级缓存) 开启二级缓存

二级缓存默认是不开启的，我们需要手动开启二级缓存。 二级缓存开启需要两步：

1. 在 mapper xml 中添加 <cache> 标签；
2. 在需要缓存的标签上设置 useCache="true"。

完整示例实现如下：

![image.png](https://javacn.site/image/1679468506722-85dea6f6-ee0a-415c-9235-e5e6f3f20be7.png)



```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mybatis.demo.mapper.StudentMapper">
    <cache/>
    <select id="getStudentCount" resultType="Integer" useCache="true">
        select count(*) from student
    </select>
</mapper>
```

编写单元测试代码：



```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class StudentMapperTest {
    @Autowired
    private StudentMapper studentMapper;

    @Test
    void getStudentCount() {
        int count = studentMapper.getStudentCount();
        System.out.println("查询结果：" + count);
        int count2 = studentMapper.getStudentCount();
        System.out.println("查询结果2：" + count2);
    }
}
```

执行以上单元测试的执行结果如下：

![image.png](https://javacn.site/image/1679468390659-f9481d2b-8113-49f7-8c43-49fce91ebecc.png)

从以上结果可以看出，两次查询虽然使用了不同的 SqlSession，但第二次查询使用了缓存，并未查询数据库。

### 8.MyBatis有哪些设计模式？

MyBatis 源码中的几个主要设计模式，即工厂模式、建造者模式、单例模式、适配器模式、代理模式、模板方法模式等，具体内容如下。

#### [#](#_1-工厂模式) 1.工厂模式

工厂模式想必都比较熟悉，它是 Java 中最常用的设计模式之一。工厂模式就是提供一个工厂类，当有客户端需要调用的时候，只调用这个工厂类就可以得到自己想要的结果，从而无需关注某类的具体实现过程。这就好比你去餐馆吃饭，可以直接点菜，而不用考虑厨师是怎么做的。

工厂模式在 MyBatis 中的典型代表是 SqlSessionFactory。SqlSession 是 MyBatis 中的重要 Java 接口，可以通过该接口来执行 SQL 命令、获取映射器示例和管理事务，而 SqlSessionFactory 正是用来产生 SqlSession 对象的，所以它在 MyBatis 中是比较核心的接口之一。

#### [#](#_2-建造者模式) 2.建造者模式

建造者模式指的是将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。也就是说建造者模式是通过多个模块一步步实现了对象的构建，相同的构建过程可以创建不同的产品。

例如，组装电脑，最终的产品就是一台主机，然而不同的人对它的要求是不同的，比如设计人员需要显卡配置高的；而影片爱好者则需要硬盘足够大的（能把视频都保存起来），但对于显卡却没有太大的要求，我们的装机人员根据每个人不同的要求，组装相应电脑的过程就是建造者模式。

建造者模式在 MyBatis 中的典型代表是 SqlSessionFactoryBuilder。普通的对象都是通过 new 关键字直接创建的，但是如果创建对象需要的构造参数很多，且不能保证每个参数都是正确的或者不能一次性得到构建所需的所有参数，那么就需要将构建逻辑从对象本身抽离出来，让对象只关注功能，把构建交给构建类，这样可以简化对象的构建，也可以达到分步构建对象的目的，而 SqlSessionFactoryBuilder 的构建过程正是如此。

#### [#](#_3-单例模式) 3.单例模式

单例模式（Singleton Pattern）是 Java 中最简单的设计模式之一，此模式保证某个类在运行期间，只有一个实例对外提供服务，而这个类被称为单例类。

单例模式也比较好理解，比如一个人一生当中只能有一个真实的身份证号，每个收费站的窗口都只能一辆车子一辆车子的经过，类似的场景都是属于单例模式。单例模式在 MyBatis 中的典型代表是 ErrorContext。

ErrorContext 是线程级别的的单例，每个线程中有一个此对象的单例，用于记录该线程的执行环境的错误信息。

#### [#](#_4-适配器模式) 4.适配器模式

适配器模式是指将一个不兼容的接口转换成另一个可以兼容的接口，这样就可以使那些不兼容的类可以一起工作。 例如，最早之前我们用的耳机都是圆形的，而现在大多数的耳机和电源都统一成了方形的 typec 接口，那之前的圆形耳机就不能使用了，只能买一个适配器把圆形接口转化成方形的，如下图所示：

![image.png](https://javacn.site/image/1674391008528-030b3e0a-8728-4085-bd12-885dcc9f24f2.png)

而这个转换头就相当于程序中的适配器模式，适配器模式在 MyBatis 中的典型代表是 Log。

MyBatis 中的日志模块适配了以下多种日志类型：

- SLF4J
- Apache Commons Logging
- Log4j 2
- Log4j
- JDK logging

#### [#](#_5-代理模式) 5.代理模式

代理模式指的是给某一个对象提供一个代理对象，并由代理对象控制原对象的调用。

代理模式在生活中也比较常见，比如我们常见的超市、小卖店其实都是一个个“代理”，他们的最上游是一个个生产厂家，他们这些代理负责把厂家生产出来的产品卖出去。

代理模式在 MyBatis 中的典型代表是 MapperProxyFactory。

MapperProxyFactory 的 newInstance() 方法就是生成一个具体的代理来实现某个功能。

#### [#](#_6-模板方法模式) 6.模板方法模式

模板方法模式是最常用的设计模式之一，它是指定义一个操作算法的骨架，而将一些步骤的实现延迟到子类中去实现，使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。此模式是基于继承的思想实现代码复用的。

例如，我们喝茶的一般步骤都是这样的：

- 把热水烧开
- 把茶叶放入壶中
- 等待一分钟左右
- 把茶倒入杯子中
- 喝茶

整个过程都是固定的，唯一变的就是泡入茶叶种类的不同，比如今天喝的是绿茶，明天可能喝的是红茶，那么我们就可以把流程定义为一个模板，而把茶叶的种类延伸到子类中去实现，这就是模板方法的实现思路。

模板方法在 MyBatis 中的典型代表是 BaseExecutor，在 MyBatis 中 BaseExecutor 实现了大部分SQL 执行的逻辑。

#### [#](#_7-装饰器模式) 7.装饰器模式

装饰器模式允许向一个现有的对象添加新的功能，同时又不改变其结构，这种类型的设计模式属于结构型模式，它是作为现有类的一个包装。

装饰器模式在生活中很常见，比如装修房子，我们在不改变房子结构的同时，给房子添加了很多的点缀；比如安装了天然气报警器，增加了热水器等附加的功能都属于装饰器模式。

装饰器模式在 MyBatis 中的典型代表是 Cache。

Cache 除了有数据存储和缓存的基本功能外（由 PerpetualCache 永久缓存实现），还有其他附加的 Cache 类，比如先进先出的 FifoCache、最近最少使用的 LruCache、防止多线程并发访问的 SynchronizedCache 等众多附加功能的缓存类。

