##                                                                              JVM

### 1.JVM是如何运行的？

JVM（Java Virtual Machine，Java虚拟机）是 Java 程序的运行环境，它负责将 Java 字节码翻译成机器代码并执行。也就是说 Java 代码之所以能够运行，主要是依靠 JVM 来实现的。

JVM 整体的大概执行流程是这样的：

1. 程序在执行之前先要把 Java 代码转换成字节码（class 文件），JVM 首先需要把字节码通过一定的方式**类加载器（ClassLoader）** 把文件加载到内存中**运行时数据区（Runtime Data Area）**；
2. 但字节码文件是 JVM 的一套指令集规范，并不能直接交个底层操作系统去执行，因此需要特定的命令解析器，也就是 JVM 的执行引擎（Execution Engine）会**将字节码翻译成底层系统指令再交由 CPU 去执行；**
3. **在执行的过程中，也需要调用其他语言的接口，如通过调用本地库接口（Native Interface）** 来实现整个程序的运行，如下图所示：

![image.png](https://javacn.site/image/1673751702610-28d9b08a-0b77-4119-aa72-f616f492288c.png)

所以，整体来看， JVM 主要通过分为以下 4 个部分，来执行 Java 程序的，它们分别是：

1. 类加载器（ClassLoader）
2. 运行时数据区（Runtime Data Area）
3. 执行引擎（Execution Engine）
4. 本地库接口（Native Interface）

### 2.什么是类加载器？

类加载器（Class Loader）是 Java 虚拟机（JVM）的重要组成部分，负责将字节码文件加载到内存中并转换为可执行的类。

类加载总共分为以下四种：

1. **启动类加载器（Bootstrap Class Loader）**：它是 JVM 的内部组件，负责加载 Java 核心类库（如java.lang）和其他被系统类加载器所需要的类。启动类加载器是由 JVM 实现提供的，通常使用本地代码来实现。
2. **扩展类加载器（Extension Class Loader）**：它是 sun.misc.Launcher$ExtClassLoader 类的实例，负责加载 Java 的扩展类库（如 java.util、java.net）等。扩展类加载器通常从 java.ext.dirs 系统属性所指定的目录或 JDK 的扩展目录中加载类。
3. **系统类加载器（System Class Loader）**：也称为应用类加载器（Application Class Loader），它是sun.misc.Launcher$AppClassLoader 类的实例，负责加载应用程序的类。系统类加载器通常从 CLASSPATH 环境变量所指定的目录或 JVM 的类路径中加载类。
4. **用户自定义类加载器（User-defined Class Loader）**：这是开发人员根据需要自己实现的类加载器。用户自定义类加载器可以根据特定的加载策略和需求来加载类，例如从特定的网络位置、数据库或其他非传统来源加载类。

如下图所示： ![image.png](https://javacn.site/image/1684135672079-af9234ff-ab54-4f89-b48e-a4fab4348576.png)

### 3.类是如何被加载的？

Java 中类加载总共分为以下 5 个步骤。

#### [#](#_1-加载) 1.加载

加载（Loading）：查找并加载类的二进制数据。这个过程可以通过类的全限定名来完成，也可以通过其他方式完成，比如使用 ClassLoader.loadClass() 方法。

在加载 Loading 阶段，Java 虚拟机需要完成以下 3 件事：

- 通过一个类的全限定名来获取定义此类的二进制字节流；
- 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构；
- 在内存中生成一个代表这个类的 java.lang.Class 对象，作为方法区这个类的各种数据的访问入口。

#### [#](#_2-验证) 2.验证

验证（Verification）：验证加载的类是否符合 Java 虚拟机规范，比如是否有正确的文件格式、是否有正确的访问权限等。

验证是连接阶段的第一步，这一阶段的目的是确保 Class 文件的字节 流中包含的信息符合《Java虚拟机规范》的全部约束要求，保证这些信 息被当作代码运行后不会危害虚拟机自身的安全。

验证选项：

- 文件格式验证
- 字节码验证
- 符号引用验证...

#### [#](#_3-准备) 3.准备

准备（Preparation）：为类的静态变量分配内存，并设置默认初始值。

准备阶段是正式为类中定义的变量（即静态变量，被static修饰的变量）分配内存并设置类变量初始值的阶段。

比如此时有这样一行代码：

> public static int value = 123;

它是初始化 value 的 int 值为 0，而非 123。

#### [#](#_4-解析) 4.解析

解析（Resolution）：将类中的符号引用转换为直接引用，比如将类中的方法名转换为实际的内存地址。

解析阶段是 Java 虚拟机将常量池内的符号引用替换为直接引用的过程，也就是初始化常量的过程。

#### [#](#_5-初始化) 5.初始化

初始化（Initialization）：执行类的初始化代码，包括静态变量赋值和静态代码块的执行。

#### [#](#小结) 小结

以上 5 个步骤总共分为 3 个大步骤：

1. **加载：** 查找并加载类的二进制数据。

2. 连接：

    将 Java 类的二进制数据合并到 JVM 运行状态之中。 

   1. 验证：验证加载的类是否符合 Java 虚拟机规范。
   2. 准备：为类的静态变量分配内存，并设置默认初始值。
   3. 解析：将类中的符号引用转换为直接引用。

3. **初始化：** 执行类的初始化代码，包括静态变量赋值和静态代码块的执行。

### 4.什么是双亲委派模型？

双亲委派模型是 Java 类加载器的一种工作机制。

它是指当一个类加载器需要加载一个类时，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最 终都应该传送到最顶层的启动类加载器中，只有当父加载器反馈自己无 法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去完成加载。

自 JDK 1.2 以来，Java 一直保持着三层类加载器、双亲委派的类加载架构器，如下图所示：

![image.png](https://javacn.site/image/1684138730926-305d311d-f3fa-42d3-94fa-0165570b5df9.png)

其中：

- 启动类加载器：加载 JDK 中 lib 目录中 Java 的核心类库，即$JAVA_HOME/lib目录。 扩展类加载器。加载 lib/ext 目录下的类；
- 应用程序类加载器：加载我们写的应用程序；
- 自定义类加载器：根据自己的需求定制类加载器。

双亲委派模型的优点是：

1. **避免重复加载类**：比如 A 类和 B 类都有一个父类 C 类，那么当 A 启动时就会将 C 类加载起来，那么在 B 类进行加载时就不需要在重复加载 C 类了。
2. **更安全**：使用双亲委派模型也可以保证了 Java 的核心 API 不被篡改，如果没有使用双亲委派模型，而是每个类加载器加载自己的话就会出现一些问题，比如我们编写一个称为 java.lang.Object 类的话，那么程序运行的时候，系统就会出现多个不同的 Object 类，而有些 Object 类又是用户自己提供的因此安全性就不能得到保证了。

### 5.说一下JVM内存布局？

通常所说的 JVM 内存布局，一般指的是 JVM 运行时数据区（Runtime Data Area），也就是当字节码被类加载器加载之后的执行区域划分。

《Java虚拟机规范》中将 JVM 运行时数据区域划分为以下 5 部分：

1. **程序计数器（Program Counter Register）**：用于记录当前线程执行的字节码指令地址，是线程私有的，线程切换不会影响程序计数器的值。
2. **Java 虚拟机栈（Java Virtual Machine Stacks）**：用于存储方法执行时的局部变量表、操作数栈、动态链接、方法出口等信息，也是线程私有的。每个方法在执行时都会创建一个栈帧，栈帧包含了方法的局部变量表、操作数栈等信息。
3. **本地方法栈（Native Method Stack**）：与 Java 虚拟机栈类似，用于存储本地方法的信息。
4. **Java 堆（Java Heap）**：用于存储对象实例和数组，是 JVM 中最大的一块内存区域，它是所有线程共享的。堆通常被划分为年轻代和老年代，以支持垃圾回收机制。

- **年轻代（Young Generation**）：用于存放新创建的对象。年轻代又分为 Eden 区和两个 Survivor 区（通常是一个 From 区和一个 To 区），对象首先被分配在 Eden 区，经过垃圾回收后存活的对象会被移到 Survivor 区，经过多次回收后仍然存活的对象会晋升到老年代。
- **老年代（Old Generation）**：用于存放存活时间较长的对象。老年代主要存放长时间存活的对象或从年轻代晋升过来的对象。

1. **方法区（Methed Area**）：用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。方法区也是所有线程共享的。

如下图所示![image.png](https://javacn.site/image/1662979290097-558b8691-e861-44f9-930c-04b891ed2830.png)

### 6.堆和栈有什么区别？

对于 JVM 来说，堆和栈都是 JVM 的重要组成部分，主要用于存储数据和执行程序。它们有以下几个主要区别：

1. **数据结构**：堆是一种动态分配的内存区域，用于存储对象实例和数组。它是所有线程共享的，对象在堆中通过引用进行访问。栈是为每个线程分配的内存区域，用于存储方法调用和局部变量等信息；栈的数据结构类似于栈（先进后出），每个方法的调用会创建一个栈帧，方法的参数、局部变量和返回值都存储在栈帧中。
2. **分配方式**：堆是由 Java 虚拟机动态分配和管理的，通过垃圾回收机制来自动释放不再使用的对象内存；栈的内存分配是自动的，随着方法的进入和退出而自动分配和释放。当方法被调用时，栈会为方法的参数和局部变量分配内存，当方法执行完毕时，栈会自动释放这些内存。
3. **内存管理**：堆的内存管理由 Java 虚拟机负责，它使用垃圾回收机制自动回收不再使用的对象内存，从而避免了内存泄漏和手动释放内存的问题；栈的内存管理是自动的，当方法调用结束时，栈会自动释放该方法所使用的内存，不需要额外的操作。
4. **空间大小**：堆的大小通常比栈大得多，因为堆需要存储大量的对象实例和数组；栈的大小通常比较小，因为它仅用于存储方法的调用和局部变量等信息。
5. **生命周期**：堆中的对象生命周期可以比较长，可以在程序的不同部分进行引用和使用；栈中的数据生命周期较短，当方法调用结束时，栈中的数据就会被自动释放。

总结起来，堆和栈在数据结构、分配方式、内存管理、空间大小和生命周期等方面存在差异。堆适合存储动态分配的对象，而栈适合存储方法的调用和局部变量等信息。

### 7.永久代和方法区有什么区别？

方法区是《Java虚拟机规范》中定义的内存区域，用于存储类的结构信息（如类的字节码、常量池、字段和方法信息等），而 Java 默认虚拟机 HotSpot 中，在 JDK 1.8 之前的版本中，是通过永久代来实现方法区的，但 JDK 1.8 之后，永久代被元空间（Metaspace）取代。

所以，方法区和永久代的区别在于，方法区是规范，而永久代（和元空间）是具体实现。

### 8.为什么要使用元空间替代永久代？

Java 官方在 JEP 122: Remove the Permanent Generation（移除永久代）中给出了答案，原文内容如下：

> **Motivation（动机）** This is part of the JRockit and Hotspot convergence effort. JRockit customers do not need to configure the permanent generation (since JRockit does not have a permanent generation) and are accustomed to not configuring the permanent generation.

以上内容翻译成中文大意是：

> 这是 JRockit 虚拟机和 HotSpot 虚拟机融合工作的一部分。JRockit 客户不需要配置永久层代（因为 JRockit 没有永久代），所以要移除永久代。

JRockit 是 Java 官方收购的一家号称史上运行最快的 Java 虚拟机厂商，之后 Java 官方在 JDK 8 时将 JRockit 虚拟机和 HotSpot 虚拟机进行了整合。

> PS：JEP 是 JDK Enhancement Proposal 的缩写，翻译成中文是 JDK 改进提案。你也可以把它理解为 JDK 的更新文档。

通过官方的描述，我们似乎找到了答案，也就是说，**之所以要取消“永久代”是因为 Java 官方收购了 JRockit，之后在将 JRockit 和 HotSpot 进行整合时，因为 JRockit 中没有“永久代”，所以把永久代给移除了**。

#### [#](#元空间优点) 元空间优点

相比于永久代，元空间具有更好的灵活性和扩展性，可以更好地满足不同应用程序的需求。 永久代的大小是固定的，当加载的类信息、常量池等数据超过了永久代的大小时，就会导致内存溢出。而元空间的大小可以根据需要进行调整，不再受到固定大小的限制。同时，元空间的数据可以存储在本地内存中，不再受到 Java 堆大小的限制。因此，使用元空间替代永久代可以提高程序的灵活性和稳定性。 所以，元空间的优势主要体现在以下两点：

#### [#](#_1-提高稳定性-降低-oom) 1.提高稳定性，降低 OOM

当使用永久代实现方法区时，永久代的最大容量受制于 PermSize 和 MaxPermSize 参数设置的大小，而这两个参数的大小又很难确定，因为在程序运行时需要加载多少类是很难估算的，如果这两个参数设置的过小就会频繁的触发 FullGC 和导致 OOM（Out of Memory，内存溢出）。 但是，当使用元空间替代了永久代之后，出现 OOM 的几率就被大大降低了，因为元空间使用的是本地内存，这样元空间的大小就只和本地内存的大小有关了，从而大大降低了 OOM 的问题。

#### [#](#_2-降低运维成本) 2.降低运维成本

因为元空间使用的是本地内存，这样就无需运维人员再去专门设置和调整元空间的大小了。

#### [#](#小结) 小结

之所以，使用元空间替代永久代，主要是因为官方需要将 HotSpot 虚拟机和 JRockit 融合，而 JRockit 虚拟机不存在永久代的概念，所以要去掉永久代的概念，除此之外使用元空间还可以提高稳定性、降低 OOM 以及节省运维成本。

### 9.内存溢出和内存泄漏有什么区别？

内存溢出（Memory Overflow）和内存泄漏（Memory Leak）是两个与内存管理相关的问题，它们有以下区别：

1. 定义不同

   ： 

   1. 内存溢出指的是在程序运行过程中申请的内存超出了可用内存资源的情况，导致无法继续分配所需的内存，从而引发异常。
   2. 内存泄漏指的是在程序中无意中保留了不再需要的对象引用，导致这些对象无法被垃圾回收机制回收，进而占用了不必要的内存空间。

2. 产生原因不同

   ： 

   1. 内存溢出通常是由于程序运行时需要的内存超过了可用的内存资源，或者是存在大量占用内存的对象无法被及时释放。常见的内存溢出原因包括创建过多的对象、递归调用导致栈溢出等。
   2. 内存泄漏则是由于程序中存在不正确的对象引用管理，例如对象被误持有引用、缓存未清理等。

3. 影响不同

   ： 

   1. 内存溢出会导致程序抛出 OutOfMemoryError 异常，程序无法继续执行。
   2. 内存泄漏则会导致内存资源的浪费，长时间运行下会导致可用内存逐渐减少，最终可能导致内存溢出。

4. 解决方案不同

   ： 

   1. 对于内存溢出，可以通过增加可用内存、调整程序逻辑、优化资源使用等方式来解决。
   2. 而对于内存泄漏，需要通过检查和修复对象引用管理问题，确保不再使用的对象能够被垃圾回收机制正确释放。

#### [#](#小结) 小结

内存溢出和内存泄漏是两个与内存管理相关的问题，它们在定义、产生原因、影响和解决方案等方面都有很大的不同。

### 10.判断对象是否存活？

在 JVM 中，判断对象是否存活的常见算法有以下两种：

1. 引用计数算法
2. 可达性分析算法

#### [#](#_1-引用计数器算法) 1.引用计数器算法

引用计数器算法的实现思路是，给对象增加一个引用计数器，每当有一个地方引用它时，计数器就 +1；当引用失效时，计数器就 -1；任何时刻计数器为 0 的对象就是不能再被使用的，即对象已"死"。

**引用计数法的优点**：实现简单，判定效率也比较高。

**引用计数法的缺点**：是引用计数法无法解决对象的循环引用问题。

#### [#](#_2-可达性分析算法) 2.可达性分析算法

可达性分析算法是通过一系列称为"GC Roots"的对象作为起始点，从这些节点开始向下搜索，搜索走过的路径称之为"引用链"，当一个对象到 GC Roots 没有任何的引用链相连时（从 GC Roots 到这个对象不可达）时，证明此对象是不可用的。以下图为例：

![image.png](https://javacn.site/image/1673755606204-e56266f6-e3e4-4cf8-8189-0678b0b113dc.png)

对象 Object5-Object7 之间虽然彼此还有关联，但是它们到 GC Roots 是不可达的，因此它们会被判定为可回收对象。 目前主流的 Java 虚拟机使用的都是可达性分析算法来判断对象是否存活的。

### 11.什么对象可以作为GC Roots？

在 JVM 中，可作为 GC Roots 的对象有以下几种：

1. Java 虚拟机栈（栈帧中的本地变量表）中引用的对象；
2. 方法区中类静态属性引用的对象；
3. 方法区中常量引用的对象；
4. 本地方法栈中 JNI（Native方法）引用的对象。

本地变量表中的引用对象可以通过 Idea 的插件 jcllasslib 查看，如下图所示：

![img](https://javacn.site/image/1687056587087-7ca29a25-1b9a-4865-b636-0063bc0109a3.png)

### 12.常见的垃圾回收算法有哪些？

垃圾收集器有两个重要的功能：第一，先识别和标记死亡对象；第二，使用合理的垃圾回收算法回收垃圾。那常见的垃圾回收算法有哪些呢？HotSpot 官方默认的虚拟机采用的有什么哪种垃圾回收算法呢？接下来我们一起来看。

常见的垃圾回收算法有以下 4 个：

1. 标记-清除算法；
2. 复制算法；
3. 标记-整理算法；
4. 分代算法。

#### [#](#_1-标记-清除算法) 1.标记-清除算法

**标记-清除（Mark-Sweep）算法属于早期的垃圾回收算法，它是由标记阶段和清除阶段构成的。标记阶段会给所有的存活对象做上标记，而清除阶段会把没有被![img](https://javacn.site/image/1664260174322-42957aa2-0902-4a0e-aaf4-4e0c91ec2ff1.png)

从上图可以看出，标记-清除算法有一个最大的问题就是会产生内存空间的碎片化问题，也就是说标记-清除算法执行完成之后会产生大量的不连续内存，这样当程序需要分配一个大对象时，因为没有足够的连续内存而导致需要提前触发一次垃圾回收动作。

**优点**：实现简单。

**缺点**：产生不连续的内存碎片，如果程序需要分配一个连续内存的大对象时，就需要提前触发一次垃圾回收。

#### [#](#_2-复制算法) 2.复制算法

**复制算法是将内存分为大小相同的两块区域，每次只使用其中的一块区域，这样在进行垃圾回收时就可以直接将存活的东西复制到新的内存上，然后再把另一块内存全部清理掉。** 这样就不会产生内存碎片的问题了，其执行流程如下图所示：

![img](https://javacn.site/image/1664260823476-32c26c0b-8c81-48aa-986b-17b1ea23f5ea.png)

从上图可以看出：**使用复制算法是可以解决内存碎片的问题的，但同时也带来了新的问题。因为需要将内存分为大小相同的两块内存，那么内存的实际可用量其实只有原来的一半，这样此算法导致了内存的可用率大幅降低了**。

**优点**：执行效率高，没有内存碎片的问题。

**缺点**：空间利用率低，因为复制算法每次只能使用一半的内存。

#### [#](#_3-标记-整理算法) 3.标记-整理算法

**标记-整理算法是由两个阶段组成的：标记阶段和整理阶段。其中标记阶段和标记-清除算法的标记阶段一样，不同的是后面的一个阶段，标记-整理算法的后一个阶段不是直接对内存进行清除，而是把所有存活的对象移动到内存的一端，然后把另一端的所有死亡对象全部清除**，执行流程图如下图所示：

![img](https://javacn.site/image/1664261583094-8e597c85-3007-4a97-9e18-1dda1002d845.png)

**优点**：解决了内存碎片问题，比复制算法空间利用率高。

**缺点**：因为有局部对象移动，所以效率不是很高。

#### [#](#_4-分代算法) 4.分代算法

**分代算法并不能是某种具体的算法，而是一种策略，我们就姑且称它为分代算法吧。目前 HotSpot 虚拟机使用的就是此算法，在 HotSpot 虚拟机中将垃圾回收区域堆划分为两个模块：新生代和老生代**，如下图所示：

![img](https://javacn.site/image/1664266886282-5da0eedf-139d-430e-acc0-3337ec37f3fa.png)

**为什么要将堆分为新生代和老生代呢？** 因为对象分为两种，绝大多数对象都是朝生夕灭的，也就是用完一次之后就不用了，而剩下一小部分对象是要重复使用多次的，将不同的对象划分到不同的区域，不同的区域使用不同的算法进行垃圾回收，这样可以大大提高 Java 虚拟机的工作效率。

#### [#](#不同区域不同算法) 不同区域不同算法

在分代算法中对于不同区域采用的具体算法也是不同的，新生代存放的大部分数据是朝生夕灭的，所以**新生代使用的是效率最高的复制算法；而老生代使用的是标记-清除或标记-整理算法，如果标记-清除可以满足需要那么就使用效率更好的标记-清除算法，如果标记-清除算法不能满足需要就使用标记-整理算法**。

#### [#](#小结) 小结

标记-清除算法效率较高，但存在内存碎片；复制算法效率最高，也没有内存碎片，但内存利用率不高，而标记-整理算法效率不算高，但不存在内存碎片，并且不存在内存利用率的问题；而 HotSpot 在 JDK 8 之前使用的是分代算法（分代策略），将垃圾回收区域分为新生代和老生代，新生代采用复制算法，老生代采用标记-清除或标记-整理算法。

### 13.垃圾回收器有哪些？

JVM 常见的垃圾回收器有以下几个：

1. Serial/Serial Old：单线程垃圾回收器；
2. ParNew：多线程的垃圾回收器（Serial 的多线程版本）；
3. Parallel Scavenge/Parallel Old：吞吐量优先的垃圾回收器【JDK8 默认的垃圾回收器】；
4. CMS：最小等待时间优先的垃圾收集器；
5. G1：可控垃圾回收时间的垃圾收集器【JDK 9 之后（HotSpot）默认的垃圾回收器】；
6. ZGC：停顿时间超短（不超过 10ms）的情况下尽量提高垃圾回收吞吐量的垃圾收集器【JDK 15 之后默认的垃圾回收器】。

![image.png](https://javacn.site/image/1673773605681-2c0ab7ef-8f06-4483-951b-2f4d89d9c973.png)

### 14.什么是FULL GC？

Full GC（Full Garbage Collection）是指对整个堆内存进行垃圾回收的过程。在进行 Full GC 时，会对年轻代和老年代（以及永久代或元数据区）中的所有对象进行回收。

Full GC 通常发生在以下情况之一：

1. 显式触发：通过调用 System.gc() 方法显式触发垃圾回收。虽然调用该方法只是向 JVM 发出建议，但在某些情况下，JVM 可能会选择执行 Full GC。
2. 老年代空间不足：当老年代空间不足时，无法进行对象的分配，会触发 Full GC。此时，Full GC 的目标是回收老年代中的无效对象，以释放空间供新的对象分配。
3. 永久代或元数据区空间不足：在使用永久代（Java 8 之前）或元数据区（Java 8 及之后）存储类的元数据信息时，如果空间不足，会触发 Full GC。

Full GC 是一种较为耗时的操作，因为它需要扫描和回收整个堆内存。在 Full GC 过程中，应用程序的执行通常会暂停，这可能会导致较长的停顿时间（长时间的停顿会影响应用程序的响应性能）。 为了避免频繁的 Full GC，通常采取一些优化措施，如合理设置堆大小、调优垃圾回收参数、减少对象的创建和存活时间等。

### 15.说一下ZGC?

ZGC（Z Garbage Collector）是一种低延迟的垃圾回收器，是 JDK 11 引入的一项垃圾回收技术。它主要针对大内存、多核心的应用场景，旨在减少垃圾回收带来的停顿时间。

#### [#](#主要特点) 主要特点

ZGC 的主要特点包括：

1. 低停顿时间：ZGC 设计的目标之一是尽量减少垃圾回收带来的停顿时间。它采用了并发的垃圾回收方式，尽可能地与应用程序并发执行，以减少停顿的时间和影响。在目标范围内，单次垃圾回收的停顿时间通常不超过 10 毫秒。
2. 大内存支持：ZGC 被设计为支持非常大的堆内存。它能有效地处理数十到数百 GB 大小的堆，并且具有可控的、较为恒定的停顿时间。
3. 并发回收：ZGC 采用了全程并发的垃圾回收方式。它会与应用程序同时运行，通过在并发阶段遍历并标记存活对象，并在并发预备阶段处理待回收的对象。这样可以将停顿时间分散到整个应用程序运行过程中，减少对应用程序的影响。
4. 空间压缩：ZGC 会对堆进行动态的空间压缩，以避免堆内存碎片化的问题。这有助于提高堆的使用效率，并减少内存的浪费。

#### [#](#核心技术) 核心技术

ZGC 有以下几项核心技术来达成毫秒级停顿和大内存支持的目标：

1. **并发标记**：ZGC 采用增量式并发标记算法来实现并发垃圾回收。它不会在标记阶段产生长时间停顿，可以与用户线程并发运行。
2. **粉碎压缩**：ZGC 采用粉碎压缩算法来避免产生内存碎片。它会将内存按照特定大小（例如 2MB）分为多个区域，然后将存活对象连续放置在相邻区域，释放掉边界外的内存空间。这可以最大限度减少内存碎片。
3. **直接内存映射**：ZGC 会直接映射内存空间，而不需要进行内存分配。这可以避免统计堆内存碎片情况所带来的性能消耗。
4. **微任务**：ZGC 采用了微任务（Microtasks）机制来增量完成垃圾回收工作，从而不会产生长时间停顿。它会将总工作分割为多个微任务，这些微任务会在安全点（Safepoint）之间执行。
5. **可扩展的堆内存**：ZGC 不需要指定最小堆（Xmn）和最大堆（Xmx）大小，它可以跟踪堆内存变化并根据需要动态调整堆空间大小。这使得 ZGC 可以支持将近 4TB 的堆内存。
6. **可插拔组件**：ZGC 是一个独立的 GC 组件，它不依赖于 Gradle 等构建工具，可以与不同的工具或框架一起使用，这增强了其可移植性。

这些核心技术的运用使得 ZGC 可以实现毫秒级别的 GC 停顿，并支持将近 4TB 的大内存，适用于对低延迟和大内存有要求的应用场景。

#### [#](#小结) 小结

ZGC 是一种低延迟的垃圾回收器，是 JDK 11 引入的一项垃圾回收技术，也是 JDK 15 之后默认的垃圾回收器，它可以实现毫秒级停顿和大内存支持，适用于需要低延迟和高吞吐量的场景。

