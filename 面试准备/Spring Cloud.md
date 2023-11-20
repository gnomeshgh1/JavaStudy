##                                                          Spring Cloud

### 1.如何实现分布式锁？

分布式锁是一种用于保证分布式系统中多个进程或线程同步访问共享资源的技术。同时它又是面试中的常见问题，所以我们本文就重点来看分布式锁的具体实现（含实现代码）。

在分布式系统中，由于各个节点之间的网络通信延迟、故障等原因，可能会导致数据不一致的问题。分布式锁通过协调多个节点的行为，保证在任何时刻只有一个节点可以访问共享资源，以避免数据的不一致性和冲突。

#### [#](#_1-分布式锁要求) 1.分布式锁要求

分布式锁通常需要满足以下几个要求：

1. **互斥性**：在任意时刻只能有一个客户端持有锁。
2. **不会发生死锁**：即使持有锁的客户端发生故障，也能保证锁最终会被释放。
3. **具有容错性**：分布式锁需要能够容忍节点故障等异常情况，保证系统的稳定性。

#### [#](#_2-实现方案) 2.实现方案

在 Java 中，实现分布式锁的方案有多种，包括：

1. **基于数据库实现的分布式锁**：可以通过数据库的乐观锁或悲观锁实现分布式锁，但是由于数据库的 IO 操作比较慢，不适合高并发场景。
2. **基于 ZooKeeper 实现的分布式锁**：ZooKeeper 是一个高可用性的分布式协调服务，可以通过它来实现分布式锁。但是使用 ZooKeeper 需要部署额外的服务，增加了系统复杂度。
3. **基于 Redis 实现的分布式锁**：Redis 是一个高性能的内存数据库，支持分布式部署，可以通过Redis的原子操作实现分布式锁，而且具有高性能和高可用性。

#### [#](#_3-数据库分布式锁) 3.数据库分布式锁

数据库的乐观锁或悲观锁都可以实现分布式锁，下面分别来看。

#### [#](#_3-1-悲观锁) 3.1 悲观锁

在数据库中使用 for update 关键字可以实现悲观锁，我们在 Mapper 中添加 for update 即可对数据加锁，实现代码如下：



```xml
<!-- UserMapper.xml -->
<select id="selectByIdForUpdate" resultType="User">
    SELECT * FROM user WHERE id = #{id} FOR UPDATE
</select>
```

在 Service 中调用 Mapper 方法，即可获取到加锁的数据：



```java
@Transactional
public void updateWithPessimisticLock(int id, String name) {
    User user = userMapper.selectByIdForUpdate(id);
    if (user != null) {
        user.setName(name);
        userMapper.update(user);
    } else {
        throw new RuntimeException("数据不存在");
    }
}
```

#### [#](#_3-2-乐观锁) 3.2 乐观锁

在 MyBatis 中，可以通过给表添加一个版本号字段来实现乐观锁。在 Mapper 中，使用 <update> 标签定义更新语句，同时使用 set 标签设置版本号的增量。



```java
<!-- UserMapper.xml -->
<update id="updateWithOptimisticLock">
    UPDATE user SET
    name = #{name},
    version = version + 1
    WHERE id = #{id} AND version = #{version}
</update>
```

在 Service 中调用 Mapper 方法，需要传入更新数据的版本号。如果更新失败，说明数据已经被其他事务修改，具体实现代码如下：



```java
@Transactional
public void updateWithOptimisticLock(int id, String name, int version) {
    User user = userMapper.selectById(id);
    if (user != null) {
        user.setName(name);
        user.setVersion(version);
        int rows = userMapper.updateWithOptimisticLock(user);
        if (rows == 0) {
            throw new RuntimeException("数据已被其他事务修改");
        }
    } else {
        throw new RuntimeException("数据不存在");
    }
}
```

#### [#](#_4-zookeeper-分布式锁) 4.Zookeeper 分布式锁

在 Spring Boot 中，可以使用 Curator 框架来实现 ZooKeeper 分布式锁，具体实现分为以下 3 步：

1. 引入 Curator 和 ZooKeeper 客户端依赖；
2. 配置 ZooKeeper 连接信息；
3. 编写分布式锁实现类。

#### [#](#_4-1-引入-curator-和-zookeeper) 4.1 引入 Curator 和 ZooKeeper



```xml
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>latest</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>latest</version>
</dependency>
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>latest</version>
</dependency>
```

#### [#](#_4-2-配置-zookeeper-连接) 4.2 配置 ZooKeeper 连接

在 application.yml 中添加 ZooKeeper 连接配置：



```yaml
spring:
  zookeeper:
    connect-string: localhost:2181
    namespace: demo
```

#### [#](#_4-3-编写分布式锁实现类) 4.3 编写分布式锁实现类



```java
@Component
public class DistributedLock {

    @Autowired
    private CuratorFramework curatorFramework;

    /**
     * 获取分布式锁
     *
     * @param lockPath   锁路径
     * @param waitTime   等待时间
     * @param leaseTime  锁持有时间
     * @param timeUnit   时间单位
     * @return 锁对象
     * @throws Exception 获取锁异常
     */
    public InterProcessMutex acquire(String lockPath, long waitTime, long leaseTime, TimeUnit timeUnit) throws Exception {
        InterProcessMutex lock = new InterProcessMutex(curatorFramework, lockPath);
        if (!lock.acquire(waitTime, timeUnit)) {
            throw new RuntimeException("获取分布式锁失败");
        }
        if (leaseTime > 0) {
            lock.acquire(leaseTime, timeUnit);
        }
        return lock;
    }

    /**
     * 释放分布式锁
     *
     * @param lock 锁对象
     * @throws Exception 释放锁异常
     */
    public void release(InterProcessMutex lock) throws Exception {
        if (lock != null) {
            lock.release();
        }
    }
}
```

#### [#](#_5-redis-分布式锁) 5.Redis 分布式锁

我们可以使用 Redis 客户端 Redisson 实现分布式锁，它的实现步骤如下：

1. 添加 Redisson 依赖
2. 配置 Redisson 连接信息
3. 编写分布式锁代码类

#### [#](#_5-1-添加-redisson-依赖) 5.1 添加 Redisson 依赖

在 pom.xml 中添加如下配置：



```xml
<!-- https://mvnrepository.com/artifact/org.redisson/redisson-spring-boot-starter -->
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>3.20.0</version>
</dependency>
```

#### [#](#_5-2-配置-redisson-连接) 5.2 配置 Redisson 连接

在 Spring Boot 项目的配置文件 application.yml 中添加 Redisson 配置：



```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      database: 0
redisson:
  codec: org.redisson.codec.JsonJacksonCodec
  single-server-config:
    address: "redis://${spring.data.redis.host}:${spring.redis.port}"
    database: "${spring.data.redis.database}"
    password: "${spring.data.redis.password}"
```

#### [#](#_5-3-编写分布式锁代码类) 5.3 编写分布式锁代码类



```java
import jakarta.annotation.Resource;
import org.redisson.Redisson;
import org.redisson.api.RLock;
import org.springframework.stereotype.Service;

import java.util.concurrent.TimeUnit;

@Service
public class RedissonLockService {
    @Resource
    private Redisson redisson;

    /**
     * 加锁
     *
     * @param key     分布式锁的 key
     * @param timeout 超时时间
     * @param unit    时间单位
     * @return
     */
    public boolean tryLock(String key, long timeout, TimeUnit unit) {
        RLock lock = redisson.getLock(key);
        try {
            return lock.tryLock(timeout, unit);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return false;
        }
    }

    /**
     * 释放分布式锁
     *
     * @param key 分布式锁的 key
     */
    public void unlock(String key) {
        RLock lock = redisson.getLock(key);
        lock.unlock();
    }
}
```

#### [#](#_6-redis-vs-zookeeper) 6.Redis VS Zookeeper

Redis 和 ZooKeeper 都可以用来实现分布式锁，它们在实现分布式锁的机制和原理上有所不同，具体区别如下：

1. **数据存储方式**：Redis 将锁信息存储在内存中，而 ZooKeeper 将锁信息存储在 ZooKeeper 的节点上，因此 ZooKeeper 需要更多的磁盘空间。
2. **锁的释放**：Redis 的锁是通过设置锁的过期时间来自动释放的，而 ZooKeeper 的锁需要手动释放，如果锁的持有者出现宕机或网络中断等情况，需要等待锁的超时时间才能自动释放。
3. **锁的竞争机制**：Redis 使用的是单机锁，即所有请求都直接连接到同一台 Redis 服务器，容易发生单点故障；而 ZooKeeper 使用的是分布式锁，即所有请求都连接到 ZooKeeper 集群，具有较好的可用性和可扩展性。
4. **一致性**：Redis 的锁是非严格意义下的分布式锁，因为在多台机器上运行多个进程时，由于 Redis 的主从同步可能会存在数据不一致的问题；而 ZooKeeper 是强一致性的分布式系统，保证了数据的一致性。
5. **性能**：Redis 的性能比 ZooKeeper 更高，因为 Redis 将锁信息存储在内存中，而 ZooKeeper 需要进行磁盘读写操作。

总之，Redis 适合实现简单的分布式锁场景，而 ZooKeeper 适合实现复杂的分布式协调场景，也就是 ZooKeeper 适合强一致性的分布式系统。

> 强一致性是指系统中的所有节点在任何时刻看到的数据都是一致的。ZooKeeper 中的数据是有序的树形结构，每个节点都有唯一的路径标识符，所有节点都共享同一份数据，当任何一个节点对数据进行修改时，所有节点都会收到通知，更新数据，并确保数据的一致性。 在 ZooKeeper 中，强一致性体现在数据的读写操作上。ZooKeeper 使用 ZAB（ZooKeeper Atomic Broadcast）协议来保证数据的一致性，该协议确保了数据更新的顺序，所有的数据更新都需要经过集群中的大多数节点确认，保证了数据的一致性和可靠性。

#### [#](#小结) 小结

在 Java 中，使用数据库、ZooKeeper 和 Redis 都可以实现分布式锁。但数据库 IO 操作比较慢，不适合高并发场景；Redis 执行效率最高，但在主从切换时，可能会出现锁丢失的情况；ZooKeeper 是一个高可用性的分布式协调服务，可以保证数据的强一致性，但是使用 ZooKeeper 需要部署额外的服务，增加了系统复杂度。所以没有最好的解决方案，只有最合适自己的解决方案。

### 2.常见的负载均衡策略有哪些？

负载均衡策略是实现负载均衡器的关键，而负载均衡器又是分布式系统中不可或缺的重要组件。使用它有助于提高系统的整体性能、可用性、可靠性和安全性，同时支持系统的扩展和故障容忍性。对于处理大量请求的应用程序和微服务架构来说，负载均衡器是不可或缺的重要工具。

#### [#](#负载均衡分类) 负载均衡分类

负载均衡分为服务器端负载均衡和客户端负载均衡。

1. 服务器端负载均衡指的是存放在服务器端的负载均衡器，例如 Nginx、HAProxy、F5 等。![img](https://javacn.site/image/1693549866206-e46bccf6-f385-4fda-bc09-35cd738ea56c.png)
2. 客户端负载均衡指的是嵌套在客户端的负载均衡器，例如 Ribbon。![img](https://javacn.site/image/1693550211355-75652839-dfb7-43a3-98ad-f11dd7f9f20d.png)

#### [#](#常见负载均衡策略) 常见负载均衡策略

但无论是服务器端负载均衡和客户端负载均衡，它们的负载均衡策略都是相同的，因为负载均衡策略本质上是一种思想。

常见的负载均衡策略有以下几个：

1. **轮询（Round Robin）**：轮询策略按照顺序将每个新的请求分发给后端服务器，依次循环。这是一种最简单的负载均衡策略，适用于后端服务器的性能相近，且每个请求的处理时间大致相同的情况。
2. **随机选择（Random）**：随机选择策略随机选择一个后端服务器来处理每个新的请求。这种策略适用于后端服务器性能相似，且每个请求的处理时间相近的情况，但不保证请求的分发是均匀的。
3. **最少连接（Least Connections）**：最少连接策略将请求分发给当前连接数最少的后端服务器。这可以确保负载均衡在后端服务器的连接负载上均衡，但需要维护连接计数。
4. **IP 哈希（IP Hash）**：IP 哈希策略使用客户端的 IP 地址来计算哈希值，然后将请求发送到与哈希值对应的后端服务器。这种策略可用于确保来自同一客户端的请求都被发送到同一台后端服务器，适用于需要会话保持的情况。
5. **加权轮询（Weighted Round Robin）**：加权轮询策略给每个后端服务器分配一个权重值，然后按照权重值比例来分发请求。这可以用来处理后端服务器性能不均衡的情况，将更多的请求分发给性能更高的服务器。
6. **加权随机选择（Weighted Random）**：加权随机选择策略与加权轮询类似，但是按照权重值来随机选择后端服务器。这也可以用来处理后端服务器性能不均衡的情况，但是分发更随机。
7. **最短响应时间（Least Response Time）**：最短响应时间策略会测量每个后端服务器的响应时间，并将请求发送到响应时间最短的服务器。这种策略可以确保客户端获得最快的响应，适用于要求低延迟的应用。

#### [#](#小结) 小结

负载均衡分为服务器端负载均衡和客户端负载均衡，但无了是那种负载均衡器，它的常用策略都是一样的，有轮询、随机选择、最少连接 IP 哈希、加权轮询、加权随机和最短响应时间。

### 3.限流的算法有哪些？

限流的实现算法有很多，但常见的限流算法有三种：计数器算法、漏桶算法和令牌桶算法。

#### [#](#_1-计数器算法) 1.计数器算法

计数器算法是在一定的时间间隔里，记录请求次数，当请求次数超过该时间限制时，就把计数器清零，然后重新计算。当请求次数超过间隔内的最大次数时，拒绝访问。

计数器算法的实现比较简单，但存在“突刺现象”。

突刺现象是指，比如限流 QPS（每秒查询率）为 100，算法的实现思路就是从第一个请求进来开始计时，在接下来的 1 秒内，每来一个请求，就把计数加 1，如果累加的数字达到了 100，后续的请求就会被全部拒绝。等到 1 秒结束后，把计数恢复成 0，重新开始计数。如果在单位时间 1 秒内的前 10 毫秒处理了 100 个请求，那么后面的 990 毫秒会请求拒绝所有的请求，我们把这种现象称为“突刺现象”。

计数器算法的简单实现代码如下：



```java
import java.util.Calendar;
import java.util.Date;
import java.util.Random;

public class CounterLimit {
    // 记录上次统计时间
    static Date lastDate = new Date();
    // 初始化值
    static int counter = 0;
    // 限流方法
    static boolean countLimit() {
        // 获取当前时间
        Date now = new Date();
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(now);
        // 当前分
        int minute = calendar.get(Calendar.MINUTE);
        calendar.setTime(lastDate);
        int lastMinute = calendar.get(Calendar.MINUTE);
        if (minute != lastMinute) {
            lastDate = now;
            counter = 0;
        }
        ++counter;
        return counter >= 100; // 判断计数器是否大于每分钟限定的值。
    }
 
    // 测试方法
    public static void main(String[] args) {
        for (; ; ) {
            // 模拟一秒
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            Random random = new Random();
            int i = random.nextInt(3);
            // 模拟1秒内请求1次
            if (i == 1) {
                if (countLimit()) {
                    System.out.println("限流了" + counter);
 
                } else {
                    System.out.println("没限流" + counter);
                }
            } else if (i == 2) { // 模拟1秒内请求2次
                for (int j = 0; j < 2; j++) {
                    if (countLimit()) {
                        System.out.println("限流了" + counter);
                    } else {
                        System.out.println("没限流" + counter);
                    }
                }
            } else { // 模拟1秒内请求10次
                for (int j = 0; j < 10; j++) {
                    if (countLimit()) {
                        System.out.println("限流了" + counter);
                    } else {
                        System.out.println("没限流" + counter);
                    }
                }
            }
        }
    }
}
```

#### [#](#_2-漏桶算法) 2.漏桶算法

漏桶算法的实现思路是，有一个固定容量的漏桶，水流（请求）可以按照任意速率先进入到漏桶里，但漏桶总是以固定的速率匀速流出，当流入量过大的时候（超过桶的容量），则多余水流（请求）直接溢出。如下图所示：

![img](https://javacn.site/image/1676808386916-e4409c10-aae6-4666-8cab-d9049d3e40ca.png)

漏桶算法提供了一种机制，通过它可以让突发流量被整形，以便为系统提供稳定的请求，比如 Sentinel 中流量整形（匀速排队功能）就是此算法实现的，如下图所示：

![image.png](https://javacn.site/image/1676809544253-d64825cc-b888-4fa5-ba4f-f9a9c98fb337.png)

更多 Sentinel 内容详见：[https://mp.weixin.qq.com/s/nF5f18BP8hscqIEmIFRN8Qopen in new window](https://mp.weixin.qq.com/s/nF5f18BP8hscqIEmIFRN8Q)

#### [#](#_3-令牌桶算法) 3.令牌桶算法

令牌按固定的速率被放入令牌桶中，桶中最多存放 N 个令牌（Token），当桶装满时，新添加的令牌被丢弃或拒绝。当请求到达时，将从桶中删除 1 个令牌。令牌桶中的令牌不仅可以被移除，还可以往里添加，所以为了保证接口随时有数据通过，必须不停地往桶里加令牌。由此可见，往桶里加令牌的速度就决定了数据通过接口的速度。我们通过控制往令牌桶里加令牌的速度从而控制接口的流量。 令牌桶的实现原理如下图所示：

![img](https://javacn.site/image/1676809614495-359430ce-d5d1-4f3f-b035-9d2b21ef1121.png)

#### [#](#_4-漏桶算法-vs-令牌桶算法) 4.漏桶算法 VS 令牌桶算法

漏桶算法是按照常量固定速率流出请求的，流入请求速率任意，当流入的请求数累积到漏桶容量时，新流入的请求被拒绝。令牌桶算法是按照固定速率往桶中添加令牌的，请求是否被处理需要看桶中的令牌是否足够，当令牌数减为零时，拒绝新的请求。令牌桶算法允许突发请求，只要有令牌就可以处理，允许一定程度的突发流量。漏桶算法限制的是常量流出速率，从而使突发流入速率平滑。 比如服务器空闲时，理论上使用漏桶算法服务器可以直接处理一次洪峰（一次洪水过程的最大流量），但是漏桶算法处理请求的速率是恒定的，因此，前期服务器资源只能根据恒定的漏水速度逐步处理请求，无法直接处理这次洪峰。而使用令牌桶算法就不存在这个问题，因为它可以先把令牌桶一次性装满，处理一次洪峰之后再走限流。

#### [#](#小结) 小结

限流的常见算法有以下 3 种：

1. 计数器算法：实现简单，但有突刺现象；
2. 漏桶算法：固定速率处理请求，处理任意流量更加平滑，可以实现流量整形；
3. 令牌桶算法：通过控制桶中的令牌实现限流，可以处理一定的突发流量，比如处理一次洪峰。

#### 

### 4.熔断和降级有什么区别？

熔断和降级都是系统自我保护的一种机制，但二者又有所不同，它们的区别主要体现在以下几点：

1. 概念不同
2. 触发条件不同
3. 归属关系不同

#### [#](#_1-概念不同) 1.概念不同

#### [#](#_1-1-熔断概念) 1.1 熔断概念

“熔断”一词早期来自股票市场。熔断（Circuit Breaker）也叫自动停盘机制，是指当股指波幅达到规定的熔断点时，交易所为控制风险采取的暂停交易措施。比如 2020 年 3 月 9 日，纽约股市开盘出现暴跌，随后跌幅达到 7% 上限，触发熔断机制，停止交易 15 分钟，恢复交易后跌幅有所减缓。

而**熔断在程序中，有“断开”的意思，当系统繁忙时，程序为了保证整体的稳定性，会暂时停止服务一段时间，以保证系统的稳定性。**

如果没有熔断机制的话，会导致联机故障和服务雪崩等问题，如下图所示：

![img](https://javacn.site/image/1676532606555-4d8bfa48-f766-433a-bef6-b5ee876a731f.gif)

#### [#](#_1-2-降级概念) 1.2 降级概念

**降级（Degradation）降低级别的意思，它是指程序在出现问题时，仍能保证有限功能可用的一种机制。**

比如电商交易系统在双 11 时，使用的人比较多，此时如果开放所有功能，可能会导致系统不可用，所以此时可以开启降级功能，优先保证支付功能可用，而其他非核心功能，如评论、物流、商品介绍等功能可以暂时关闭。

所以，从上述信息可以看出：**降级是一种退而求其次的选择，而熔断却是整体不可用**。

#### [#](#_2-触发条件不同) 2.触发条件不同

不同框架的熔断和降级的触发条件是不同的，本文咱们以经典的 Spring Cloud 组件 Hystrix 为例，来说明触发条件的问题。

#### [#](#_2-1-hystrix-熔断触发条件) 2.1 Hystrix 熔断触发条件

默认情况 hystrix 如果检测到 10 秒内请求的失败率超过 50%，就触发熔断机制。之后每隔 5 秒重新尝试请求微服务，如果微服务不能响应，继续走熔断机制。如果微服务可达，则关闭熔断机制，恢复正常请求。

![image.png](https://javacn.site/image/1676534836456-53b9b783-1e80-4fda-b044-e4990c051007.png)

#### [#](#_2-2-hystrix-降级触发条件) 2.2 Hystrix 降级触发条件

默认情况下，hystrix 在以下 4 种条件下都会触发降级机制：

1. 方法抛出 HystrixBadRequestException
2. 方法调用超时
3. 熔断器开启拦截调用
4. 线程池或队列或信号量已满

虽然 hystrix 组件的触发机制，不能代表所有的熔断和降级机制，但足矣说明此问题。

#### [#](#_3-归属关系不同) 3.归属关系不同

**熔断时可能会调用降级机制，而降级时通常不会调用熔断机制。**因为熔断是从全局出发，为了保证系统稳定性而停用服务，而降级是退而求其次，提供一种保底的解决方案，所以它们的归属关系是不同（熔断 > 降级）。

#### [#](#题外话) 题外话

当然，某些框架如 Sentinel，它早期在 Dashboard 控制台中可能叫“降级”，但在新版中新版本又叫“熔断”，如下图所示：

![img](https://javacn.site/image/1676536073962-58abcf3e-4d90-4d36-a57d-dcf5f9669c3f.png)

但在两个版本中都是通过同一个异常类型 DegradeException 来监听的，如下代码所示：

![image.png](https://javacn.site/image/1676536185697-8938f546-2918-4d52-a45d-0928894e71ab.png)

所以，在 Sentinel 中，熔断和降级功能指的都是同一件事，也侧面证明了“熔断”和“降级”概念的相似性。但我们要知道它们本质上是不同的，就像两个双胞胎，不能因为他们长得像，就说他们是同一个人。

#### [#](#小结) 小结

熔断和降级都是程序在我保护的一种机制，但二者在概念、触发条件、归属关系上都是不同的。熔断更偏向于全局视角的自我保护（机制），而降级则偏向于具体模块“退而请其次”的解决方案。

### 5.分布式事务二阶段和三阶段？

在分布式事务中，通常使用两阶段协议或三阶段协议来保障分布式事务的正常运行，它也是 X/Open 公司定义的一套解决分布式事务的规范。

> X/Open 公司是由多家国际计算机厂商所组成的联盟组织，它建立之初是为了向 UNIX 环境提供标准。

分布式事务是指在分布式系统中，多个节点之间进行的事务操作。比如在分布式系统中，用户在下单时，需要同时创建订单信息和减库存的操作，然而创建订单信息和减库存是分布在不同服务器和不同数据库中的，如下图所示：

![img](https://javacn.site/image/1690255185728-10d333e8-55cb-4c32-b2b4-b0f5b1ea14b9.png)

此时我们就需要一个分布式事务介入，保证所有操作，要么一起提交，要么一起回滚。

#### [#](#_1-两阶段提交) 1.两阶段提交

两阶段提交（Two-Phase Commit，简称 2PC）是一种分布式事务协议，确保所有参与者在提交或回滚事务时都处于一致的状态。2PC 协议包含以下两个阶段：

1. 准备阶段（prepare phase）：在这个阶段，事务协调者（Transaction Coordinator）向所有参与者（Transaction Participant）发出准备请求，询问它们是否准备好提交事务。参与者执行所有必要的操作，并回复协调者是否准备好提交事务。如果所有参与者都回复准备好提交事务，协调者将进入下一个阶段。如果任何参与者不能准备好提交事务，协调者将通知所有参与者回滚事务。
2. 提交阶段（commit phase）：在这个阶段，如果所有参与者都已准备好提交事务，则协调者向所有参与者发送提交请求。参与者执行所有必要的操作，并将其结果记录在持久性存储中。一旦所有参与者都已提交事务，协调者将向它们发送确认请求。如果任何参与者未能提交事务，则协调者将通知所有参与者回滚事务。

2PC 协议可以确保分布式事务的原子性和一致性，但是其效率较低，可能会出现阻塞等问题。因此，在实际应用中，可以使用其他分布式事务协议，如 3PC（Three-Phase Commit）或 Paxos 协议来代替。

#### [#](#两阶段提交问题) 两阶段提交问题

两阶段提交存在以下几个问题：

1. 同步阻塞问题：执行过程中，所有参与节点都是事务阻塞型的。当参与者占有公共资源时，其他第三方节点访问公共资源不得不处于阻塞状态。也就是说从投票阶段到提交阶段完成这段时间，资源是被锁住的。
2. 单点故障：由于协调者的重要性，一旦协调者发生故障。参与者会一直阻塞下去。尤其在第二阶段，协调者发生故障，那么所有的参与者还都处于锁定事务资源的状态中，而无法继续完成事务操作。
3. 数据不一致问题：在 2PC 最后提交阶段中，当协调者向参与者发送 commit 请求之后，发生了局部网络异常或者在发送 commit 请求过程中协调者发生了故障，这会导致只有一部分参与者接受到了 commit 请求。而在这部分参与者接到 commit 请求之后就会执行 commit 操作。但是其他部分未接到 commit 请求的机器则无法执行事务提交，于是整个分布式系统便出现了数据不一致性的现象。

#### [#](#_2-三阶段提交) 2.三阶段提交

三阶段提交（Three-Phase Commit，简称3PC）是在 2PC 协议的基础上添加了一个额外的阶段来解决 2PC 协议可能出现的阻塞问题。 3PC 协议包含三个阶段：

1. CanCommit 阶段（询问阶段）：在这个阶段，事务协调者（Transaction Coordinator）向所有参与者（Transaction Participant）发出 CanCommit 请求，询问它们是否准备好提交事务。参与者执行所有必要的操作，并回复协调者它们是否可以提交事务。
2. PreCommit 阶段（准备阶段）：如果所有参与者都回复可以提交事务，则协调者将向所有参与者发送PreCommit 请求，通知它们准备提交事务。参与者执行所有必要的操作，并回复协调者它们是否已经准备好提交事务。
3. DoCommit 阶段（提交阶段）：如果所有参与者都已经准备好提交事务，则协调者将向所有参与者发送DoCommit 请求，通知它们提交事务。参与者执行所有必要的操作，并将其结果记录在持久性存储中。一旦所有参与者都已提交事务，协调者将向它们发送确认请求。如果任何参与者未能提交事务，则协调者将通知所有参与者回滚事务。

与 2PC 协议相比，3PC 协议将 CanCommit 阶段（询问阶段）添加到协议中，使参与者能够在 CanCommit 阶段发现并解决可能导致阻塞的问题。这样，3PC 协议能够更快地执行提交或回滚事务，并减少不必要的等待时间。需要注意的是，与 2PC 协议相比，3PC 协议仍然可能存在阻塞的问题。

#### [#](#_3-两阶段提交-vs-三阶段提交) 3.两阶段提交 VS 三阶段提交

2PC 和 3PC 是分布式事务中两种常见的协议，3PC 可以看作是 2PC 协议的改进版本，相比于 2PC 它有两点改进：

1. 引入了超时机制，同时在协调者和参与者中都引入超时机制（2PC 只有协调者有超时机制）；
2. 3PC 相比于 2PC 增加了 CanCommit 阶段，可以尽早的发现问题，从而避免了后续的阻塞和无效操作。

也就是说，3PC 相比于 2PC，因为引入了超时机制，所以发生阻塞的几率变小了；同时 3PC 把之前 2PC 的准备阶段一分为二，变成了两步，这样就多了一个缓冲阶段，保证了在最后提交阶段之前各参与节点的状态是一致的。

#### [#](#_4-数据一致性问题和解决方案) 4.数据一致性问题和解决方案

3PC 虽然可以减少同步阻塞问题和单点故障问题，但依然存在数据一致性问题（概率很小），而解决数据一致性问题的方案有很多，比如 Paxos 算法或柔性事物机制等。

#### [#](#_4-1-paxos-算法) 4.1 Paxos 算法

Paxos 算法是一种基于消息传递的分布式一致性算法，并在 2013 年获得了图灵奖。 图灵奖（ACM A.M. Turing Award）是计算机科学领域最高荣誉之一，由美国计算机协会（ACM）于 1966 年设立，每年颁发一次，表彰对计算机科学领域做出杰出贡献的人士或团体。 简单来说，Paxos 算法是一种分布式共识算法，用于在分布式系统中实现数据的一致性和共识，保证分布式系统中不同节点之间的数据同步和一致性。 Paxos 算法由三个角色组成：提议者、接受者和学习者。当一个节点需要发起一个提议时，它会向其他节点发送一个提议，接受者会接收到这个提议，并对其进行处理，可能会拒绝提议，也可能会接受提议。如果有足够多的节点接受了该提议，那么提议就会被确定下来，并且通知给所有学习者，最终所有节点都会达成共识。 Paxos 算法看起来很简单，但它实际上是非常的复杂。 Paxos 算法应用的产品也很多，比如以下几个：

- Redis：Redis 是一个内存数据库，使用 Paxos 算法实现了分布式锁服务和主从复制等功能。
- MySQL：MySQL 5.7 推出的用来取代传统的主从复制的 MySQL Group Replication 等。
- ZooKeeper：ZooKeeper 是一个分布式协调服务，使用 Paxos 算法实现了分布式锁服务和数据一致性等功能。
- Apache Cassandra：Cassandra 是一个分布式数据库系统，使用 Paxos 算法实现了数据的一致性和复制等功能。
- Google Chubby：Chubby 是 Google 内部使用的分布式锁服务，使用 Paxos 算法实现了分布式锁服务和命名服务等功能。

#### [#](#_4-2-柔性事务) 4.2 柔性事务

柔性事务机制：允许一定时间内不同节点的数据不一致，但要求最终一致的机制。 柔性事物有 TCC 补偿事物、可靠消息事物（MQ 事物）等。

#### [#](#小结) 小结

在分布式事务中，通常使用两阶段或三阶段提交协议来保障分布式事务的正常执行。两阶段协议包含准备阶段和提交阶段，然而它存在同步阻塞问题、单点故障和数据一致性问题。

而三阶段协议可以看作是两阶段协议的改进版，它将两阶段的准备阶段一分为二，多了一个询问阶段，保证了提交阶段之前各参与节点的状态是一致的，同时引入了超时机制，减少了同步阻塞问题发生的几率。但 2PC 和 3PC 都存在数据一致性问题，此时可以采用 Paxos 算法或柔性事务机制等方案来解决事务一致性问题。

### 6.网关如何限流？

网关（Gateway）是微服务中不可缺少的一部分，它是微服务中提供了统一访问地址的组件，充当了客户端和内部微服务之间的中介。网关主要负责流量路由和转发，将外部请求引导到相应的微服务实例上，同时提供一些功能，如身份认证、授权、限流、监控、日志记录等。

网关的主要作用有以下几个：

1. **路由功能**：网关可以根据目标地址的不同，选择最佳的路径将数据包从源网络路由到目标网络。它通过维护路由表来确定数据包的转发方向，并选择最优的路径。
2. **安全控制（统一认证授权）**：网关可以实施网络安全策略，对进出的数据包进行检查和过滤。它可以验证和授权来自源网络的数据包，并阻止未经授权的访问。防火墙是一种常见的网关设备，用于过滤和保护网络免受恶意攻击和未经授权的访问。
3. **协议转换**：不同网络使用不同的通信协议，网关可以进行协议转换，使得不同网络的设备可以互相通信。例如，例如将 HTTPS 协议转换成 HTTP 协议。
4. **网络地址转换（NAT）**：网关还可以执行网络地址转换，将内部网络使用的私有 IP 地址转换为外部网络使用的公共 IP 地址，以实现多台计算机共享一个公共 IP 地址出去上网。

#### [#](#_1-关于限流) 1.关于限流

为了保护后端微服务免受突发高流量请求的影响，确保系统的稳定和可靠性，所以在网关层必须“限流”操作。

限流是一种流量控制的策略，用于限制系统处理请求的速率或数量，以保护系统免受过载或攻击的影响。通过限制请求的数量或速率，可以平衡系统和资源之间的压力，确保系统在可接受的范围内运行。

限流的常见策略通常有以下几种：

1. **请求速率限流**：限制单位时间内系统可以接受的最大请求数量。例如，每秒最多处理 100 个请求。当请求超过限制时，可以选择拒绝或延迟处理这些请求。
2. **并发请求数限流**：限制同时处理的请求数量。例如，限制系统只能同时处理100个并发请求。当并发请求数超过限制时，可以选择拒绝或排队等待。
3. **用户级别限流**：根据用户进行限流，限制每个用户的请求频率或数量。例如，限制每个用户每分钟只能发送 10 个请求。当用户请求超过限制时，可以选择拒绝或延迟处理。
4. **API 级别限流**：根据 API 接口进行限流，限制每个接口的请求频率或数量。例如，限制某个接口每秒只能处理 50 个请求。当接口请求超过限制时，可以选择拒绝或延迟处理。

当然，我们也可以在程序中使用多种策略混合限流，以保证内部微服务的稳定性。

#### [#](#_2-如何实现限流) 2.如何实现限流？

了解了网关和限流的相关内容之后，我们以目前主流的网关组件 Spring Cloud Gateway 为例，来实现一下限流功能。

Spring Cloud Gateway 实现限流的方式有两种：

1. 使用内置 Filter（过滤器）实现限流。
2. 使用限流组件 Spring Cloud Alibaba Sentinel 或者 Spring Cloud Netflix Hystrix 实现限流。

那既然 Spring Cloud Gateway 中已经内置了限流功能，那我们接下来就来看 Spring Cloud Gateway 内置限流是如何实现的？

Spring Cloud Gateway 内置的限流器为 RequestRateLimiter GatewayFilter Factory，官网说明文档：[https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-requestratelimiter-gatewayfilter-factoryopen in new window](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-requestratelimiter-gatewayfilter-factory)

Spring Cloud Gateway 支持和 Redis 一起来实现限流功能，它的实现步骤如下：

1. 在网关项目中添加 Redis 框架依赖
2. 创建限流规则
3. 配置限流过滤器

具体实现如下。

#### [#](#_2-1-添加-redis-框架依赖) 2.1 添加 Redis 框架依赖

在项目的 pom.xml 中，添加以下配置信息（添加 Redis 框架依赖支持）：



```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```

#### [#](#_2-2-创建限流规则) 2.2 创建限流规则

接下来我们新建一个限流规则定义类，实现一下根据 IP 进行限流的功能，实现示例代码如下：



```java
import org.springframework.cloud.gateway.filter.ratelimit.KeyResolver;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

@Component
public class IpAddressKeyResolver implements KeyResolver {
    @Override
    public Mono<String> resolve(ServerWebExchange exchange) {
        return Mono.just(exchange.getRequest().getRemoteAddress().
                         getHostString());
    }
}
```

这一步其实是在配置限流器的限流参数 KeyResolver，也就是限流功能的依赖“凭证”。

> PS：当然，我们还可以通过 URL、方法名、用户等进行限流操作，只需要修改此步骤中的限流凭证，也就是 KeyResolver 即可。

#### [#](#_2-3-配置限流过滤器) 2.3 配置限流过滤器

在网关项目的配置文件中，添加以下配置信息：



```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: retry
          uri: lb://nacos-discovery-demo
          predicates:
            - Path=/retry/**
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 1
                redis-rate-limiter.burstCapacity: 1
                keyResolver: '#{@ipAddressKeyResolver}' # spEL表达式
  data:
    redis:
      host: 127.0.0.1
      port: 16379
      database: 0
```

其中，name 必须等于“RequestRateLimiter”内置限流过滤器，其他参数的含义如下：

- **redis-rate-limiter.replenishRate**：令牌填充速度：每秒允许请求数。
- **redis-rate-limiter.burstCapacity**：令牌桶容量：最大令牌数。
- **keyResolver**：根据哪个 key 进行限流，它的值是 spEL 表达式。

> SpEL（Spring Expression Language，Spring 表达式语言）是 Spring 框架中用于提供灵活、强大的表达式解析和求值功能的统一表达式语言。它可以在运行时动态地解析和求值字符串表达式，通常用于配置文件中的属性值、注解、XML 配置等地方。

#### [#](#注意事项) 注意事项

当 Spring Cloud Gateway 配合 Redis 实现限流的时候，它对于 Redis 的版本是有要求的，因为它在限流时调用了一个 Redis 高版本的函数，所以 **Redis Server 版本太低，限流无效**，Redis Server 最好是 5.X 以上。

#### [#](#_2-4-限流测试) 2.4 限流测试

最后，我们频繁的访问：http://localhost:10086/retry/test 就会看到如下限流信息：

![img](https://javacn.site/image/1700444363342-6a81a5af-2474-4539-b2a4-dabff7bbf85c.png)

#### [#](#_3-限流实现算法) 3.限流实现算法

Spring Cloud Gateway 内置限流功能使用的算法是**令牌桶限流算法**。

**令牌桶限流算法**：令牌按固定的速率被放入令牌桶中，桶中最多存放 N 个令牌（Token），当桶装满时，新添加的令牌被丢弃或拒绝。当请求到达时，将从桶中删除 1 个令牌。令牌桶中的令牌不仅可以被移除，还可以往里添加，所以为了保证接口随时有数据通过，必须不停地往桶里加令牌。由此可见，往桶里加令牌的速度就决定了数据通过接口的速度。我们通过控制往令牌桶里加令牌的速度从而控制接口的流量。 令牌桶执行流程如下图所示：

![img](https://javacn.site/image/1700444170835-a1f01eef-92e8-4e73-b49b-8cc36f1f7c21.png)

常见的限流算法还有：计数器算法、滑动计数器算法、漏桶算法等，更多介绍参考我之前写个的文章：[https://www.javacn.site/interview/springcloud/loadbalancer.htmlopen in new window](https://www.javacn.site/interview/springcloud/loadbalancer.html)

#### [#](#_4-限流实现原理) 4.限流实现原理

Spring Cloud Gateway 执行过程如下图所示：

![img](https://javacn.site/image/1700444916882-51240814-d2d0-4ade-b644-3a4efe1c0de6.png)

从图中可以看出，所有的请求来了之后，会先走过滤器，只有过滤器通过之后，才能调用后续的内部微服务，这样我们就可以通过过滤器来控制微服务的调用，从而实现限流功能了。

Spring Cloud Gateway 过滤器是基于令牌桶算法来限制请求的速率，该过滤器根据配置的限流规则，在指定的时间窗口内分配一定数量的令牌，每个令牌代表一个允许通过的请求，当一个请求到达时，如果没有可用的令牌，则请求将被阻塞或拒绝。

令牌桶的执行过程如下：

1. **初始化**：在加载过滤器工厂时，会基于给定的限流规则创建一个限流器，该限流器包含了令牌桶算法的逻辑。默认情况下，令牌桶是按照固定速率进行填充，也可以配置为令牌桶按照令牌令牌的方式进行填充。
2. **请求处理**：每当有请求进来时，限流器会检查当前令牌桶中是否有可用的令牌。如果有可用的令牌，则请求会被放行，令牌桶中的令牌数量减少；如果没有可用的令牌，则请求会被阻塞或拒绝。
3. **令牌桶填充**：限流器会定期填充令牌桶，即向令牌桶中添加新的令牌。填充的速率取决于限流规则中配置的速率值。
4. **令牌桶容量控制**：限流器还会根据限流规则中配置的令牌桶容量，控制令牌桶中的令牌数量。如果令牌桶已满，则多余的令牌会被丢弃。

#### [#](#小结) 小结

主流网关组件 Spring Cloud Gateway 实现限流的方式主要有两种：内置限流过滤器和外部限流组件，如 Sentinel、Hystrix 等。而最简单的限流功能，我们只需要使用 Spring Cloud Gateway 过滤器 + Redis 即可（实现），其使用的是令牌桶的限流算法来实现限流功能的。

### 7.sentinel怎么和nacos进行双向通讯？

默认情况下 Sentinel 只能接收到 Nacos 推送的消息，但不能将自己控制台修改的信息同步给 Nacos，如下图所示：

![img](https://javacn.site/image/1697591741548-2581e6f0-d45d-4bdd-a741-6ad723666387.png)

但是在生成环境下，我们为了更方便的操作，是需要将 Sentinel 控制台修改的规则也同步到 Nacos 的，所以在这种情况下我们就需要修改 Sentinel 的源码，让其可以实现和 Nacos 的双向通讯，如下图所示：

![img](https://javacn.site/image/1697591932798-24768300-39a4-47e8-84a4-62f3545a7e38.png)

改造之后的交互流程如下图所示：

![img](https://javacn.site/image/1697532094541-f82e4c33-9ffe-430d-aa0d-9a1ef6c455ef.png)

Sentinel 同步规则至数据源，例如将 Sentinel 的规则，同步规则至 Nacos 数据源的改造步骤很多，但整体实现难度不大，下面我们一起来看吧。

#### [#](#_1-下载sentinel源码) 1.下载Sentinel源码

下载地址：[https://github.com/alibaba/Sentinelopen in new window](https://github.com/alibaba/Sentinel)

> PS：本文 Sentinel 使用的版本是 1.8.6。

下载源码之后，使用 idea 打开里面的 sentinel-dashboard 项目，如下图所示：

![img](https://javacn.site/image/1697529300493-2ea43890-cf4b-4daf-a1db-1a4c5477c20b.png)

#### [#](#_2-修改pom-xml) 2.修改pom.xml

将 sentinel-datasource-nacos 底下的 scope 注释掉，如下图所示：

![img](https://javacn.site/image/1697460424345-df5831c1-3b09-441e-98d3-4ebbabe509cd.png)

> PS：因为官方提供的 Nacos 持久化实例，是在 test 目录下进行单元测试的，而我们是用于生产环境，所以需要将 scope 中的 test 去掉。

#### [#](#_3-移动单元测试代码) 3.移动单元测试代码

将 test/com.alibaba.csp.sentinel.dashboard.rule.nacos 下所有文件复制到 src/main/java/com.alibaba.csp.sentinel.dashboard.rule 目录下，如下图所示：

![img](https://javacn.site/image/1697529725144-e36eacc6-cbc8-4f35-b98c-512835a0dcb7.png)

#### [#](#_4-新建nacospropertiesconfiguration文件) 4.新建NacosPropertiesConfiguration文件

在 com.alibaba.csp.sentinel.dashboard.rule 下创建 Nacos 配置文件的读取类，实现代码如下：



```java
package com.alibaba.csp.sentinel.dashboard.rule;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

@ConfigurationProperties(prefix = "sentinel.nacos")
@Configuration
public class NacosPropertiesConfiguration {
    private String serverAddr;
    private String dataId;
    private String groupId;
    private String namespace;
    private String username;
    private String password;
    // 省略 Getter/Setter 代码
}
```

#### [#](#_5-修改nacosconfig文件) 5.修改NacosConfig文件

只修改 NacosConfig 中的 nacosConfigService 方法，修改后的代码如下：



```java
@Bean
public ConfigService nacosConfigService(NacosPropertiesConfiguration nacosPropertiesConfiguration) throws Exception {
    Properties properties = new Properties();
    properties.put(PropertyKeyConst.SERVER_ADDR, nacosPropertiesConfiguration.getServerAddr());
    properties.put(PropertyKeyConst.NAMESPACE, nacosPropertiesConfiguration.getNamespace());
    properties.put(PropertyKeyConst.USERNAME,nacosPropertiesConfiguration.getUsername());
    properties.put(PropertyKeyConst.PASSWORD,nacosPropertiesConfiguration.getPassword());
    return ConfigFactory.createConfigService(properties);
//        return ConfigFactory.createConfigService("localhost"); // 原代码
}
```

#### [#](#_6-修改flowcontrollerv2文件) 6.修改FlowControllerV2文件

修改 com.alibaba.csp.sentinel.dashboard.controller.v2 目录下的 FlowControllerV2 文件：

![img](https://javacn.site/image/1697530109517-7a615817-4fe5-4581-b553-cd78f150819b.png)

修改后代码：



```java
@Autowired
@Qualifier("flowRuleNacosProvider")
private DynamicRuleProvider<List<FlowRuleEntity>> ruleProvider;
@Autowired
@Qualifier("flowRuleNacosPublisher")
private DynamicRulePublisher<List<FlowRuleEntity>> rulePublisher;
```

> PS：此操作的目的是开启 Controller 层操作 Nacos 的开关。

如下图所示：

![img](https://javacn.site/image/1697460150039-4817280c-158f-4ddf-b7ca-d148bbf82a03.png)

#### [#](#_7-修改配置信息) 7.修改配置信息

在 application.properties 中设置 Nacos 连接信息，配置如下：



```properties
sentinel.nacos.serverAddr=localhost:8848
sentinel.nacos.username=nacos
sentinel.nacos.password=nacos
sentinel.nacos.namespace=
sentinel.nacos.groupId=DEFAULT_GROUP
sentinel.nacos.dataId=sentinel-dashboard-demo-sentinel
```

#### [#](#_8-修改sidebar-html) 8.修改sidebar.html

修改 webapp/resources/app/scripts/directives/sidebar/sidebar.html 文件：

![img](https://javacn.site/image/1697530569346-9098e3c3-8cab-4bce-a9fa-4c180555fefb.png)

搜索“dashboard.flowV1”改为“dashboard.flow”，如下图所示：

![img](https://javacn.site/image/1697530696591-e11161d6-03ed-4b57-b8b0-727d4fea0902.png)

#### [#](#_9-修改identity-js) 9.修改identity.js

identity.js 文件有两处修改，它位于 webapp/resources/app/scripts/controllers/identity.js 目录。

### [#](#_9-1-第一处修改) 9.1 第一处修改

将“FlowServiceV1”修改为“FlowServiceV2”，如下图所示：

![img](https://javacn.site/image/1697530886404-79b9d25b-82e0-45e0-8ad7-5894f722b6d9.png)

#### [#](#_9-2-第二处修改) 9.2 第二处修改

搜索“/dashboard/flow/”修改为“/dashboard/v2/flow/”，如下图所示：

![img](https://javacn.site/image/1697531052873-7e03b23e-6e94-44fd-b6eb-e7404895da95.png)

> PS：修改 identity.js 文件主要是用于在 Sentinel 点击资源的“流控”按钮添加规则后将信息同步给 Nacos。

#### [#](#小结) 小结

Sentinel Dashboard 默认情况下，只能将配置规则保存到内存中，这样就会程序重启后配置规则丢失的情况，因此我们需要给 Sentinel 设置一个数据源，并且要和数据源之间实现双向通讯，所以我们需要修改 Sentinel 的源码。源码的改造步骤虽然很多，但只要逐一核对和修改就可以实现 Sentinel 生成环境的配置了。

### 8.nacos注册中心有几种调用方式？

Spring Cloud Alibaba Nacos 作为近几年最热门的注册中心和配置中心，也被国内无数公司所使用，今天我们就来看下 Nacos 作为注册中心时，调用它的接口有几种方式？

#### [#](#_1-什么是注册中心) 1.什么是注册中心？

注册中心（Registry）是一种用于服务发现和服务注册的分布式系统组件。它是在微服务架构中起关键作用的一部分，用于管理和维护服务实例的信息以及它们的状态。

它的执行流程如下图所示：

![img](https://javacn.site/image/1698569395003-a6306bf8-e959-45a4-ac0c-a221814bdd90.png)

注册中心充当了服务之间的中介和协调者，它的主要功能有以下这些：

1. **服务注册**：服务提供者将自己的服务实例信息（例如 IP 地址、端口号、服务名称等）注册到注册中心。通过注册中心，服务提供者可以将自己的存在告知其他服务。
2. **服务发现**：服务消费者通过向注册中心查询服务信息，获取可用的服务实例列表。通过注册中心，服务消费者可以找到并连接到需要调用的服务。
3. **健康检查与负载均衡**：注册中心可以定期检查注册的服务实例的健康状态，并从可用实例中进行负载均衡，确保请求可以被正确地转发到可用的服务实例。
4. **动态扩容与缩容**：在注册中心中注册的服务实例信息可以方便地进行动态的增加和减少。当有新的服务实例上线时，可以自动地将其注册到注册中心。当服务实例下线时，注册中心会将其从服务列表中删除。

使用注册中心有以下优势和好处：

- **服务自动发现和负载均衡**：服务消费者无需手动配置目标服务的地址，而是通过注册中心动态获取可用的服务实例，并通过负载均衡算法选择合适的实例进行调用。
- **服务弹性和可扩展性**：新的服务实例可以动态注册，并在发生故障或需要扩展时快速提供更多的实例，从而提供更高的服务弹性和可扩展性。
- **中心化管理和监控**：注册中心提供了中心化的服务管理和监控功能，可以对服务实例的状态、健康状况和流量等进行监控和管理。
- **降低耦合和提高灵活性**：服务间的通信不再直接依赖硬编码的地址，而是通过注册中心进行解耦，使得服务的部署和变更更加灵活和可控。

常见的注册中心包括 ZooKeeper、Eureka、Nacos 等。这些注册中心可以作为微服务架构中的核心组件，用于实现服务的自动发现、负载均衡和动态扩容等功能。

#### [#](#_2-方法概述) 2.方法概述

当 Nacos 中注册了 Restful 接口时（一种软件架构风格，它是基于标准的 HTTP 协议和 URI 的一组约束和原则），其调用方式主要有以下两种：

1. 使用 RestTemplate + Spring Cloud LoadBalancer
2. 使用 OpenFeign + Spring Cloud LoadBalancer

#### [#](#_3-resttemplate-loadbalancer调用) 3.RestTemplate+LoadBalancer调用

此方案的实现有以下 3 个关键步骤：

1. 添加依赖：nacos + loadbalancer
2. 设置配置文件
3. 编写调用代码

具体实现如下。

#### [#](#_3-1-添加依赖) 3.1 添加依赖



```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

#### [#](#_3-2-设置配置文件) 3.2 设置配置文件



```yaml
spring:
  application:
    name: nacos-discovery-business
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        username: nacos
        password: nacos
        register-enabled: false
```

#### [#](#_3-3-编写调用代码) 3.3 编写调用代码

此步骤又分为以下两步：

1. 给 RestTemplate 增加 LoadBalanced 支持
2. 使用 RestTemplate 调用接口

#### [#](#_3-3-1-resttemplate添加loadbalanced) 3.3.1 RestTemplate添加LoadBalanced

在 Spring Boot 启动类上添加“@EnableDiscoveryClient”注解，并使用“@LoadBalanced”注解替换 IoC 容器中的 RestTemplate，具体实现代码如下：



```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
@EnableDiscoveryClient
public class BusinessApplication {
    @LoadBalanced
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    public static void main(String[] args) {
        SpringApplication.run(BusinessApplication.class, args);
    }
}
```

#### [#](#_3-3-2-使用resttemplate) 3.3.2 使用RestTemplate



```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
@RequestMapping("/business")
public class BusinessController2 {
    @Autowired
    private RestTemplate restTemplate;
    @RequestMapping("/getnamebyid")
    public String getNameById(Integer id){
        return restTemplate.getForObject("http://nacos-discovery-demo/user/getnamebyid?id="+id,
                String.class);
    }
}
```

#### [#](#_4-openfeign-loadbalancer调用) 4.OpenFeign+LoadBalancer调用

此步骤又分为以下 5 步：

1. 添加依赖：nacos + openfeign + loadbalancer
2. 设置配置文件
3. 开启 openfeign 支持
4. 编写 service 代码
5. 调用 service 代码

具体实现如下。

#### [#](#_4-1-添加依赖) 4.1 添加依赖



```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
 <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-openfeign</artifactId>
  </dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

#### [#](#_4-2-设置配置文件) 4.2 设置配置文件



```yaml
spring:
  application:
    name: nacos-discovery-business
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        username: nacos
        password: nacos
        register-enabled: false
```

#### [#](#_4-3-开启openfeign) 4.3 开启OpenFeign

在 Spring Boot 启动类上添加 @EnableFeignClients 注解。

#### [#](#_4-4-编写service) 4.4 编写Service



```java
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.stereotype.Service;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Service
@FeignClient(name = "nacos-producer") // name 为生产者的服务名
public interface UserService {
    @RequestMapping("/user/getinfo") // 调用生产者的接口
    String getInfo(@RequestParam String name);
}
```

#### [#](#_4-5-调用service) 4.5 调用Service



```java
import com.example.consumer.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class OrderController {
    @Autowired
    private UserService userService;
    @RequestMapping("/order")
    public String getOrder(@RequestParam String name){
        return userService.getInfo(name);
    }
}
```

#### [#](#_5-获取本文源码) 5.获取本文源码

因平台不能上传附件，所以想要获取本文完整源码，请联系我：gg_stone，备注：Nacos 源码，不然不予通过。

#### [#](#_6-版本说明) 6.版本说明

本文案例基于以下版本：

- JDK 17
- Spring Boot 3.x
- Spring Cloud Alibaba 2022.0.0.0
- Nacos 2.2.3

#### [#](#_7-小结) 7.小结

注册中心作为微服务中不可或缺的重要组件，在微服务中充当着中介和协调者的作用。而 Nacos 作为近几年来，国内最热门的注册中心，其 Restf 接口调用有两种方式：RestTemplate + LoadBalancer 和 OpenFeign + LoadBalancer，开发者可以根据自己的实际需求，选择相应的调用方式。

### 9.nacos有几种负载均衡策略？

Nacos 作为目前主流的微服务中间件，包含了两个顶级的微服务功能：配置中心和注册中心。

#### [#](#_1-配置中心扫盲) 1.配置中心扫盲

配置中心是一种集中化管理配置的服务，通俗易懂的说就是将本地配置文件“云端化”。 这样做的好处有以下几个:

1. **集中管理配置信息**：配置中心将不同服务的配置信息集中放在一起进行管理，实现了配置信息的集中存储。
2. **动态更新配置**：配置中心中的配置信息可以通过操作界面或 API 进行动态更新，无需重启服务就可以应用最新的配置信息。
3. **配置信息共享**：将配置集中在配置中心中，不同的服务实例可以共享同一套配置信息。
4. **配置信息安全**：配置中心可以对配置信息提供安全管理、权限控制等管理功能。
5. **信息追溯**：支持配置版本管理、历史记录等管理功能。

当然，配置中心不可能有负载均衡的功能，所以略过，咱们直接来看注册中心。

#### [#](#_2-注册中心扫盲) 2.注册中心扫盲

注册中心（Registry）是分布式系统中的一个组件，**用于实现服务的注册与发现**。注册中心用于管理服务实例的元数据信息，并提供服务发现和路由的功能。

在微服务架构中，服务之间经常需要互相调用和通信。注册中心的作用是为服务提供一个集中管理和协调的中心，默认情况下，服务将自己的信息注册到注册中心，其他服务可以通过查询注册中心的信息来发现和调用目标服务。

注册中心的核心功能包括以下几个：

1. **服务注册**：服务提供者在启动时将自己的信息（比如 IP 地址、端口号、服务名称等）注册到注册中心。注册中心维护着一张服务实例的清单。
2. **服务发现**：服务消费者通过向注册中心查询服务信息，获取可用的服务实例列表。通过注册中心，服务消费者能够找到并连接到目标服务。
3. **健康检查**：注册中心可以定时检查服务实例的健康状态，并根据服务的状态更新服务实例的可用性。
4. **负载均衡**：注册中心可以根据负载均衡策略，将请求分发给不同的服务实例，以实现负载均衡和服务高可用。
5. **服务路由**：在一些高级注册中心中，还可以定义服务路由规则，将请求路由到不同的服务实例，实现更灵活的流量控制和管理。

#### [#](#_3-注册中心与负载均衡) 3.注册中心与负载均衡

负载均衡严格的来说，并不算是传统注册中心的功能。一般来说服务发现的完整流程应该是先从注 册中心获取到服务的实例列表，然后再根据自身的需求，来选择其中的部分实例或者按照⼀定的流 量分配机制来访问不同的服务提供者，因此注册中心本身⼀般不限定服务消费者的访问策略。

例如 Eureka、Zookeeper 包括 Consul，本身都没有去实现可配置及可扩展的负载均衡机制，Eureka 的 负载均衡是由 Ribbon 来完成的，而 Consul 则是由 Fabio 做负载均衡。

也就是说注册中心和负载均衡，其实完全属于两个不同的东西，注册中心主要提供服务的注册，以及将服务注册的列表交给消费者，至于消费者要使用哪种负载均衡策略？完全可以由自己决定。此时消费者可以通过客户端负载均衡器来实现服务的选择和调用，例如客户端负载均衡器 Ribbon 或 Spring Cloud LoadBalancer。

#### [#](#_4-客户端与服务端负载均衡) 4.客户端与服务端负载均衡

客户端负载均衡器通常位于服务的消费者端，主要负责将请求合理地分发给不同的服务提供者。工作原理是客户端在发起请求前，通过负载均衡算法选择一个合适的服务实例进行请求。客户端根据服务实例的健康度、负载状况等指标来决定选择哪个服务实例。常见的客户端负载均衡器有 Ribbon、Feign 等。

![img](https://javacn.site/image/1693550211355-75652839-dfb7-43a3-98ad-f11dd7f9f20d.png)

服务端负载均衡器通常被称为反向代理服务器或负载均衡器，它位于服务的提供者端，接收客户端的请求，并根据一定的负载均衡策略将请求分发给后端的多个服务实例。工作原理是将客户端的请求集中到负载均衡器，由负载均衡器将请求分发给多台服务提供者。常见的服务器端负载均衡器有 Nginx、HAProxy 等。

![img](https://javacn.site/image/1693549866206-e46bccf6-f385-4fda-bc09-35cd738ea56c.png)

**客户端负载均衡 VS 服务端负载均衡**

- 客户端负载均衡器的优点是可以实现本地的负载均衡算法，避免了对注册中心的频繁调用，降低了网络开销。它的缺点是每个客户端都需要集成负载均衡器，导致代码冗余和维护复杂性。
- 服务器负载均衡器的优点是可以集中管理请求流量，提供一致的负载均衡策略和配置，对客户端透明。它的缺点是服务器端负载均衡器通常需要独立部署和配置，增加了系统的复杂性和维护成本。并且它很可能成为整个系统的瓶颈（因为客户端需要频繁的调用），所以此时需要考虑其性能和可靠性等问题。

#### [#](#_5-nacos和负载均衡) 5.Nacos和负载均衡

然而 Nacos 的注册中心和传统的注册中心不太一样，例如 Eureka、Zookeeper、Consul 等。因为 Nacos 在 0.7.0 之后（包含此版本），它内置了以下两种负载均衡策略：

1.**基于权重的负载均衡策略**，这个在 Nacos 服务编辑的时候也可以看到其设置：

![img](https://javacn.site/image/1698740261064-1f545931-a519-4400-9036-fab93213f66e.png)

2.**基于第三方 CMDB（地域就近访问）标签的负载均衡策略**，这个可以参考官方说明文档：[https://nacos.io/zh-cn/blog/cmdb.htmlopen in new window](https://nacos.io/zh-cn/blog/cmdb.html)

#### [#](#小结) 小结

注册中心和负载均衡器严格意义上来说是两个东西，但 Nacos 注册中心中，内置了两种负载均衡策略：基于权重和基于 CMDB（低于就近访问）的负载均衡策略。

#### [#](#思考) 思考

那么问题来了，既然 Nacos 中内置了基于权重的负载均衡策略，那为什么修改 Nacos 中的权重值，在服务端调用时，却没看到任何变化呢？



### 10.读过框架源码吗？举例说明一下？

前两天有朋友面试“淘天集团”，也就是“淘宝”+“天猫”的组合，最后被面试官问到了这道题：“**你看过哪些开源框架的源码？举例说明一下**”。

诚然，这是一道比较考验应聘者基本功的问题，也是很好区分“好学生”和“普通学生”的一道经典的开放性问题。

那这个问题应该怎么回答呢？

## [#](#解答思路) 解答思路

我这给大家提供两个思路吧：

1. 可以回答比较常见的，你比较熟悉的源码，例如 Spring Boot 收到请求之后，执行流程的源码。
2. 还可以回答 Spring Cloud 微服务中，某个组件执行的流程源码，这样能很好的体现你对微服务比较熟悉，因为微服务在公司中应用比较广泛，所以回答的好，是一个极大的加分项。

#### [#](#_1-spring-boot-源码分析) 1.Spring Boot 源码分析

Spring Boot 在收到请求之后，会先执行前端控制器 DispatcherServlet，并调用其父类 FrameworkServlet 中的 service 方法，其核心源码如下：



```java
/**
 * Override the parent class implementation in order to intercept PATCH requests.
 */
@Override
protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    HttpMethod httpMethod = HttpMethod.resolve(request.getMethod());
    if (httpMethod == HttpMethod.PATCH || httpMethod == null) {
        processRequest(request, response);
    } else {
        super.service(request, response);
    }
}
```

继续往下看，processRequest 实现源码如下：



```java
protected final void processRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
 // 省略一堆初始化配置
   
   try {
       // 真正执行逻辑的方法
       doService(request, response);
   }
   catch (ServletException | IOException ex) {
       ...
   }
}
```

doService 实现源码如下：



```java
protected abstract void doService(HttpServletRequest request, HttpServletResponse response) throws Exception;
```

doService 是抽象方法，由其之类 DispatcherServlet 来重写实现，其核心源码如下：



```java
@Override
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
    // 省略初始化过程...
    try {
        doDispatch(request, response);
    }
    finally {
		// 省略其他...
    }
}
```

此时就进入到了 DispatcherServlet 中的 doDispatch 方法了：



```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    // 获取原生请求
    HttpServletRequest processedRequest = request;
    // 获取Handler执行链
    HandlerExecutionChain mappedHandler = null;
    // 是否为文件上传请求, 默认为false
    boolean multipartRequestParsed = false;
    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    try {
        ModelAndView mv = null;
        Exception dispatchException = null;
        try {
            // 检查是否为文件上传请求
            processedRequest = checkMultipart(request);
            multipartRequestParsed = (processedRequest != request);
            // Determine handler for the current request.
            // 获取能处理此请求的Handler
            mappedHandler = getHandler(processedRequest);
            if (mappedHandler == null) {
                noHandlerFound(processedRequest, response);
                return;
            }
            // Determine handler adapter for the current request.
            // 获取适配器
            HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
            // Process last-modified header, if supported by the handler.
            String method = request.getMethod();
            boolean isGet = "GET".equals(method);
            if (isGet || "HEAD".equals(method)) {
                long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                    return;
                }
            }
            // 执行拦截器（链）的前置处理
            if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                return;
            }
            // 真正的执行对应方法
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
            if (asyncManager.isConcurrentHandlingStarted()) {
                return;
            }
            applyDefaultViewName(processedRequest, mv);
            mappedHandler.applyPostHandle(processedRequest, response, mv);
        }
        // 忽略其他...
}
```

通过上述的源码我们可以看到，请求的核心代码都在 doDispatch 中，他里面包含的主要执行流程有以下这些：

1. **调用 HandlerExecutionChain 获取处理器**：DispatcherServlet 首先调用 getHandler 方法，通过 HandlerMapping 获取请求对应的 HandlerExecutionChain 对象，包含了处理器方法和拦截器列表。
2. **调用 HandlerAdapter 执行处理器方法**：DispatcherServlet 使用 HandlerAdapter 来执行处理器方法。根据 HandlerExecutionChain 中的处理器方法类型不同，选择对应的 HandlerAdapter 进行处理。常用的适配器有 RequestMappingHandlerAdapter 和 HttpRequestHandlerAdapter。
3. **解析请求参数**：DispatcherServlet 调用 HandlerAdapter 的 handle 方法，解析请求参数，并将解析后的参数传递给处理器方法执行。
4. **调用处理器方法**：DispatcherServlet 通过反射机制调用处理器方法，执行业务逻辑。
5. **处理拦截器**：在调用处理器方法前后，DispatcherServlet 会调用拦截器的 preHandle 和 postHandle方法进行相应的处理。
6. **渲染视图**：处理器方法执行完成后，DispatcherServlet 会通过 ViewResolver 解析视图名称，找到对应的 View 对象，并将模型数据传递给 View 进行渲染。
7. **生成响应**：View 会将渲染后的视图内容生成响应数据。

#### [#](#_2-spring-cloud-源码) 2.Spring Cloud 源码

Spring Cloud 组件有很多，你可以挑一个源码实现比较简单的组件来讲，这里推荐 Spring Cloud LoadBalancer，因为其核心源码的实现比较简单。

Spring Cloud LoadBalancer 中内置了两种负载均衡策略：

1. 轮询负载均衡策略
2. 随机负载均衡策略

轮询负载均衡策略的核心实现源码如下：



```java
// ++i 去负数，得到一个正数值
int pos = this.position.incrementAndGet() & Integer.MAX_VALUE;
// 正数值和服务实例个数取余 -> 实现轮询
ServiceInstance instance = (ServiceInstance)instances.get(pos % instances.size());
// 将实例返回给调用者
return new DefaultResponse(instance);
```

随机负载均衡策略的核心实现源码如下：



```java
// 通过 ThreadLocalRandom 获取一个随机数，最大值为服务实例的个数
int index = ThreadLocalRandom.current().nextInt(instances.size());
// 得到实例
ServiceInstance instance = (ServiceInstance)instances.get(index);
// 返回
return new DefaultResponse(instance);
```

#### [#](#小结) 小结

开源框架的源码在面试中经常会被问到，但只因如此，就去完整的看某个框架的源码，其实还是挺难的。第一，框架中的源码很多，很难一次性看懂。第二，即使能看懂，看完之后也会很快忘记（因为内容太多了）。此时，不如挑一些框架中的经典实现源码来看，其性价比更高，既能学到框架中的精髓，又能搞定面试，是一个不错的选择。

### 11.如何实现全链路灰度发布？

灰度发布（Gray Release，也称为灰度发布或金丝雀发布）是指在软件或服务发布过程中，将新版本的功能或服务以较小的比例引入到生产环境中，仅向部分用户或节点提供新功能的一种发布策略。

在传统的全量发布中，新版本的功能会一次性全部部署到所有的用户或节点上。然而，这种方式潜在的风险是，如果新版本存在缺陷或问题，可能会对所有用户或节点产生严重的影响，导致系统崩溃或服务不可用。

相比之下，灰度发布采用较小的规模，并逐步将新版本的功能引入到生产环境中，仅向一小部分用户或节点提供新功能。通过持续监测和评估，可以在发现问题时及时回滚或修复。这种逐步引入新版本的方式可以降低风险，并提高系统的稳定性和可靠性。

#### [#](#_1-实现思路) 1.实现思路

灰色发布的常见实现思路有以下几种：

- **根据用户划分**：根据用户标识或用户组进行划分，在整个用户群体中只选择一小部分用户获得新功能。
- **根据地域划分**：在不同地区或不同节点上进行划分，在其中的一小部分地区或节点进行新功能的发布。
- **根据流量划分**：根据流量的百分比或请求次数进行划分，只将一部分请求流量引导到新功能上。

而在生产环境中，比较常用的是根据用户标识来实现灰色发布，也就是说先让一小部分用户体验新功能，以发现新服务中可能存在的某种缺陷或不足。

#### [#](#_2-具体实现) 2.具体实现

Spring Cloud 全链路灰色发布的关键实现思路如下图所示：

![img](https://javacn.site/image/1699839830272-97d73ab3-a490-45dc-b6bb-d5cdfb5260ed.png)

灰度发布的具体实现步骤如下：

1. 前端程序在灰度测试的用户 Header 头中打上标签，例如在 Header 中添加“grap-tag: true”，其表示要进行灰常测试（访问灰度服务），而其他则为访问正式服务。
2. 在负载均衡器 Spring Cloud LoadBalancer 中，拿到 Header 中的“grap-tag”进行判断，如果此标签不为空，并等于“true”的话，表示要访问灰度发布的服务，否则只访问正式的服务。
3. 在网关 Spring Cloud Gateway 中，将 Header 标签“grap-tag: true”继续往下一个调用服务中传递。
4. 在后续的调用服务中，需要实现以下两个关键功能： 
   1. 在负载均衡器 Spring Cloud LoadBalancer 中，判断灰度发布标签，将请求分发到对应服务。
   2. 将灰度发布标签（如果存在），继续传递给下一个调用的服务。

经过第四步的反复传递之后，整个 Spring Cloud 全链路的灰度发布就完成了。

#### [#](#_3-核心实现思路和代码) 3.核心实现思路和代码

灰度发布的关键实现技术和代码如下。

#### [#](#_3-1-区分正式服务和灰度服务) 3.1 区分正式服务和灰度服务

在灰度发布的执行流程中，有一个核心的问题，如果在 Spring Cloud LoadBalancer 进行服务调用时，区分正式服务和灰度服务呢？

这个问题的解决方案是：在灰度服务既注册中心的 MetaData（元数据）中标识自己为灰度服务即可，而元数据中没有标识（灰度服务）的则为正式服务，以 Nacos 为例，它的设置如下：



```yaml
spring:
  application:
    name: canary-user-service
  cloud:
    nacos:
      discovery:
        username: nacos
        password: nacos
        server-addr: localhost:8848
        namespace: public
        register-enabled: true 
        metadata: { "grap-tag":"true" } # 标识自己为灰度服务
```

#### [#](#_3-2-负载均衡调用灰度服务) 3.2 负载均衡调用灰度服务

Spring Cloud LoadBalancer 判断并调用灰度服务的关键实现代码如下：



```java
private Response<ServiceInstance> getInstanceResponse(List<ServiceInstance> instances,
                                                          Request request) {
        // 实例为空
        if (instances.isEmpty()) {
            if (log.isWarnEnabled()) {
                log.warn("No servers available for service: " + this.serviceId);
            }
            return new EmptyResponse();
        } else { // 服务不为空
            RequestDataContext dataContext = (RequestDataContext) request.getContext();
            HttpHeaders headers = dataContext.getClientRequest().getHeaders();
            // 判断是否为灰度发布（请求）
            if (headers.get(GlobalVariables.GRAY_KEY) != null &&
                    headers.get(GlobalVariables.GRAY_KEY).get(0).equals("true")) {
                // 灰度发布请求，得到新服务实例列表
                List<ServiceInstance> findInstances = instances.stream().
                        filter(s -> s.getMetadata().get(GlobalVariables.GRAY_KEY) != null &&
                                s.getMetadata().get(GlobalVariables.GRAY_KEY).equals("true"))
                        .toList();
                if (findInstances.size() > 0) { // 存在灰度发布节点
                    instances = findInstances;
                }
            } else { // 查询非灰度发布节点
                // 灰度发布测试请求，得到新服务实例列表
                instances = instances.stream().
                        filter(s -> s.getMetadata().get(GlobalVariables.GRAY_KEY) == null ||
                                !s.getMetadata().get(GlobalVariables.GRAY_KEY).equals("true"))
                        .toList();
            }
            // 随机正数值 ++i（ & 去负数）
            int pos = this.position.incrementAndGet() & Integer.MAX_VALUE;
            // ++i 数值 % 实例数 取模 -> 轮询算法
            int index = pos % instances.size();
            // 得到服务实例方法
            ServiceInstance instance = (ServiceInstance) instances.get(index);
            return new DefaultResponse(instance);
        }
    }
```

以上代码为自定义负载均衡器，并使用了轮询算法。如果 Header 中有灰度标签，则只查询灰度服务的节点实例，否则则查询出所有的正式节点实例（以供服务调用或服务转发）。

#### [#](#_3-3-网关传递灰度标识) 3.3 网关传递灰度标识

要在网关 Spring Cloud Gateway 中传递灰度标识，只需要在 Gateway 的全局自定义过滤器中设置 Response 的 Header 即可，具体实现代码如下：



```java
package com.example.gateway.config;

import com.loadbalancer.canary.common.GlobalVariables;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.http.HttpStatus;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.http.server.reactive.ServerHttpResponse;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

@Component
public class LoadBalancerFilter implements GlobalFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 得到 request、response 对象
        ServerHttpRequest request = exchange.getRequest();
        ServerHttpResponse response = exchange.getResponse();
        if (request.getQueryParams().getFirst(GlobalVariables.GRAY_KEY) != null) {
            // 设置金丝雀标识
            response.getHeaders().set(GlobalVariables.GRAY_KEY,
                    "true");
        }
        // 此步骤正常，执行下一步
        return chain.filter(exchange);
    }
}
```

#### [#](#_3-4-openfeign-传递灰度标签) 3.4 Openfeign 传递灰度标签

HTTP 调用工具 Openfeign 传递灰度标签的实现代码如下：



```java
import feign.RequestInterceptor;
import feign.RequestTemplate;
import jakarta.servlet.http.HttpServletRequest;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import java.util.Enumeration;
import java.util.LinkedHashMap;
import java.util.Map;

@Component
public class FeignRequestInterceptor implements RequestInterceptor {
    @Override
    public void apply(RequestTemplate template) {
        // 从 RequestContextHolder 中获取 HttpServletRequest
        ServletRequestAttributes attributes = (ServletRequestAttributes)
                RequestContextHolder.getRequestAttributes();
        // 获取 RequestContextHolder 中的信息
        Map<String, String> headers = getHeaders(attributes.getRequest());
        // 放入 openfeign 的 RequestTemplate 中
        for (Map.Entry<String, String> entry : headers.entrySet()) {
            template.header(entry.getKey(), entry.getValue());
        }
    }

    /**
     * 获取原请求头
     */
    private Map<String, String> getHeaders(HttpServletRequest request) {
        Map<String, String> map = new LinkedHashMap<>();
        Enumeration<String> enumeration = request.getHeaderNames();
        if (enumeration != null) {
            while (enumeration.hasMoreElements()) {
                String key = enumeration.nextElement();
                String value = request.getHeader(key);
                map.put(key, value);
            }
        }
        return map;
    }
}
```

#### [#](#小结) 小结

灰度发布是微服务时代保证生产环境安全的必备措施，而其关键实现思路是：

1、注册中心区分正常服务和灰度服务；

2、负载均衡正确转发正常服务和灰度服务；

3、网关和 HTTP 工具传递灰度标签。

这样，我们就完整的实现 Spring Cloud 全链路灰度发布功能了。

