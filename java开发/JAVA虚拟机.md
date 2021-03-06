[TOC]

### JAVA内存区域

JVM的组成：

- 运行时数据区
- 类加载器，在JVM启动时或者类运行时将需要的class加载到JVM中
- 执行引擎，负责执行class文件中包含的字节码指令，相当于实际机器上的CPU
- 本地方法调用，调用C或者C++实现的本地方法的代码返回结果

![jvm_memory.png](https://i.loli.net/2018/10/23/5bcedaa314447.png)

运行时数据区：

------

> ​	*程序计数器*是一块较小的内存空间，它可以看作是当前线程所执行的字节码的行号指示器。一个处理器（对于多核处理器来说是一个内核）都只会执行一条线程中的指令，每条线程都需要有一个独立的程序计数器。

> ​	*JAVA虚拟机栈*描述的是Java方法执行的内存模型，每个方法在执行的同时都会创建一个栈帧（Stack Frame）用于存储局部变量表、操作数栈、动态链接、方法出口等信息。局部变量表存放了编译器可知的各种基本数据类型、对象引用（reference类型）和returnAddress类型（指向了一条字节码指令的地址）。
>
> ​	局部变量表所需的内存空间在编译期间完成分配，当进入一个方法时，这个方法需要在栈帧中分配多大的局部变量空间是完全确定的，且在方法运行期间不会改变局部变量表的大小。

> ​	*本地方法栈*与虚拟机栈所发挥的最用很相似，它为虚拟机使用的Native方法服务。

> ​	*Java*堆是Java虚拟机所管理的内存中最大的一块，是被所有线程共享的一块内存区域，在虚拟机启动时创建，唯一目的时存放对象实例。所有的对象实例以及数组都要在堆上分配。
>
> ​	Java堆可以细分为：新生代和老生代；Eden空间、From Survivor空间、To Survivor空间等。还可以划分处多个线程私有的分配缓冲区（TLAB），为了更好或者更快地分配内存。JAVA堆可以处于物理上不连续的内存空间，只要逻辑上是连续的即可。

> ​	*方法区*是各个线程共享的内存区域，存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。不需要连续的内存和可以选择固定大小和可拓展，还可以选择不实现垃圾收集。该区域内存回收目标主要针对常量池的回收和对类型的卸载。

> *运行时常量池*是方法区的一部分。在类加载后存放各种字面量和符号引用。并非预置入class文件中常量池的内同才能进入运行时常量池，运行期间也可能将新的常量放入池中（String类的intern方法）。

运行时数据区：方法区、虚拟机栈、堆、本地方法栈、程序计数器

线程共享：方法区、堆

线程私有：程序计数器、Java虚拟机栈、本地方法栈

Java 7中运行时常量池已经移到了堆中。Java 8 移除了方法区，增加了元数据区

![](F:\mycode\knowledgeArrangement\java开发\JVM1.8.png)

直接内存并不是虚拟机运行时数据区的一部分，但会受到本机总内存大小以及处理器寻址空间的限制。

+ 直接内存：非Java标准，是JVM以外的本地内存，在Java4出现的NIO中，为了防止Java堆和Native堆之间往复的数据复制带来的性能损耗，此后NIO可以使用Native的方式直接在Native堆分配内存。JDK中有一种基于通道（Channel）和缓冲区（Buffer）的内存分配方式，将由C语言实现的native函数库分配在直接内存中，用存储在JVM堆中的DirectByteBuffer来引用。
+ 元数据区（方法区的实现）：Java7以及之前是使用的永久代来实现方法区（永久代就是HotSpot虚拟机对虚拟机规范中方法区的一种实现方式。），大小是在启动时固定的。Java8中用元空间替代了永久代，元空间并不在虚拟机中，而是使用本地内存，并且大小可以是自动增长的，这样减少了OOM的可能性。元空间存储JIT即时编译后的native代码，可能还存在短指针数据区CCS。**元数据区的大小默认无限制，根据内存大小**动态改变，`-XX:MetaspaceSize`分配元数据区的初始大小，`-XX:MaxMetaspaceSize`元数据区的最大值，超过此值时会GC，默认无限制
+ 堆区: Java7之后运行时常量池和静态变量从方法区移到这里，为Java8移除永久代的做好准备

### JVM常用指令

##### JVM堆大小的设置

```
-Xmx 10240m -Xms 10240m -Xmn5120m -XXSurvivorRatio=3
```

解析：

-Xmx：最大堆大小

-Xms：初始堆大小

-Xmn：年轻代最大大小

-XXSurvivorRatio：年轻代中Eden区与Survivor区的大小比值

年轻代5120m， Eden：Survivor=3，Survivor区大小=1024m（Survivor区有两个，即将年轻代分为5份，每个Survivor区占一份），总大小为2048m。

-Xms初始堆大小即最小内存值为10240m。

下面来解释下几个重要参数的含义：

-Xms 和 -Xmx (-XX:InitialHeapSize 和 -XX:MaxHeapSize)：指定JVM初始占用的堆内存和最大堆内存。JVM也是一个软件，也必须要获取本机的物理内

存，然后JVM会负责管理向操作系统申请到的内存资源。JVM启动的时候会向操作系统申请 -Xms 设置的内存，JVM启动后运行一段时间，如果发现内存空间

不足，会再次向操作系统申请内存。JVM能够获取到的最大堆内存是-Xmx设置的值。

-XX:NewSize 和 -Xmn(-XX:MaxNewSize)：指定JVM启动时分配的新生代内存和新生代最大内存。

-XX:SurvivorRatio：设置新生代中1个Eden区与1个Survivor区的大小比值。在hotspot虚拟机中，新生代 = 1个Eden + 2个Survivor。如果新生代内存是10M，SurvivorRatio=8，那么Eden区占8M，2个Survivor区各占1M。

-XX:NewRatio：指定老年代/新生代的堆内存比例。在hotspot虚拟机中，堆内存 = 新生代 + 老年代。如果-XX:NewRatio=4表示年轻代与年老代所占比值为1:4,年轻代占整个堆内存的1/5。在设置了-XX:MaxNewSize的情况下，-XX:NewRatio的值会被忽略，老年代的内存=堆内存 - 新生代内存。老年代的最大内存 = 堆内存 - 新生代 最大内存。

-XX:OldSize：设置JVM启动分配的老年代内存大小，类似于新生代内存的初始大小-XX:NewSize。

-XX:PermSize 和 -XX:MaxPermSize：指定JVM中的永久代(方法区)的大小。可以看到：永久代不属于堆内存，堆内存只包含新生代和老年代。

可以发现：堆内存、新生代内存、老年代内存、永久代内存，都有一个初始内存，还有一个最大内存。

### 为什么移除永久代

+ 由于永久代内存经常不够用或发生内存泄露，爆出异常 java.lang.OutOfMemoryError: PermGen
  + 字符串存在永久代中，容易出现性能问题和内存溢出。

  + 类及方法的信息等比较难确定其大小，因此对于永久代的大小指定比较困难，太小容易出现永久代溢出，太大则容易导致老年代溢出
+ 永久代会为 GC 带来不必要的复杂度，并且回收效率偏低

### 静态常量池与运行时常量池的区别

+ 静态常量池：用于存放编译器生成的各种字面量和符号引用（符号引用区别于直接引用，后者在class字节码文件被虚拟机解析之后，符号引用将被替换为直接引用）
+ 运行时常量池：（静态）常量池中的内容在类加载（这里的类加载指class字节码文件经过加载连接初始化的过程）后存放入方法区的运行时常量池中。相对于静态常量池，运行时常量池具有动态性，在程序运行的时候可能将新的常量放入运行时常量池中，比如使用String类的intern方法

对静态常量池直观的理解，它是编译器编译java代码之后所产生的常量，这里的常量跟编写代码的常量不同，指的是类、接口、方法和字段的描述信息，比如类的名称和其基类。“静态”，是因为它们只是一个class的描述信息而已，还没有具备被执行的能力。在该class文件被JVM装载完成之后，静态常量池中的内容将被解析，并放到运行时常量池中。

静态变量    jdk7之后存储在堆中，jdk7及之前位于方法区
实例变量    作为对象的一部分，保存在堆中。
局部变量    基本类型字面量保存于栈中，引用类型对象保存在堆中

### JVM中的哪些地方会发生OutOfMemoryError

在JVM运行时数据区域中方法区、堆是线程共享的，而程序计数器、Java虚拟机栈和本地方法栈是线程私有的。

除了程序计数器没有OutOfMemoryError,Java虚拟机栈、本地方法栈、堆、方法区都可能抛出OutOfMemoryError。

虚拟机栈和本地方法栈一般是StackOverFlowError，如果 JVM 试图去扩展栈空间的的时候失败，则会抛出 OutOfMemoryError

直接内存也可能会出现OutOfMemoryError

> 对象的创建：
>
> 1. 检查指令new参数能否在常量池中定位到一个类的符号引用，可以转2
> 2. 检查符号引用代表的类是否已被加载、解析和初始化过，是转4，否转3
> 3. 执行相应的类加载过程
> 4. 为新生对象分配内存
> 5. 将分配到的内存空间都初始化为0值（除了对象头），如果使用TLAB，这个过程可以放到TLAB分配时进行 
> 6. 对对象进行必要的设置（对象头信息）
> 7. 执行init方法

内存分配方式：指针碰撞、空闲列表；选择哪种分配方式由Java堆是否规整决定，Java堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定。

解决并发情况下对象创建的线程安全问题，两种方案：

1. 对分配内存空间的动作进行同步处理（CAS+失败重试）
2. 为每个线程在Java堆中预先分配一小块内存（本地线程分配缓冲TLAB），只有TLAB用完并重新分配TLAB时才需要同步锁定

对象的内存布局：对象头（Header）、实例数据（Instance Data）和对齐填充（Padding）

HotSpot虚拟机的对象头：

1. 存储对象自身的运行时数据（Mark Word）;
2. 类型指针；
3. （Java数组）记录数组长度的数据；

对象头信息是与对象自身定义的数据无关的额外存储成本，考虑到虚拟机的空间效率，Mark Word被设计成一个非固定的数据结构以便在极小的空间内存储尽量多的信息，根据对象的状态复用自己的存储空间。

Mark Word里默认存储对象的HashCode、分代年龄和锁标记位

并不是所有的虚拟机实现都要在对象数据上保留类型指针，查找对象的元数据信息并不一定要经过对象本身

实例数据部分字段的存储顺序会受到虚拟机分配策略参数和字段在Java源码中定义顺序的影响；HotSpot中相同宽度的字段总是被分配到一起。

对齐填充并不是必然存在的

需要通过栈上的reference数据来操作堆上具体对象，对象访问方式取决于虚拟机实现，主流的访问方式为句柄和直接指针。

1. 若使用句柄访问，Java堆中会划分出一块内存作为句柄池，reference中存储的就是对象的句柄地址，句柄中包含了对象实例数据与类型数据各自的地址信息。
2. 使用直接地址访问时，reference中存储的直接就是对象地址。

句柄访问的好处是reference中存储的是稳定的句柄地址，在对象被移动时只改变句柄中的实例数据指针，而reference本身不需要修改。直接指针访问速度快

HotSpot使用直接指针

### 垃圾收集器与内存分配策略

------

#### 什么是GC？

GC是JVM中的自动内存管理和垃圾清理机制。主要完成3件事：1、确定哪些内存需要回收。2、如何执行GC。3、什么时候需要执行GC。

引用计数法的缺点是它很难解决对象之间的相互循环引用的问题

> 可达性分析算法
>
> 通过一系列的称为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链，当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用的。没有覆盖finalize()方法或者finalize()方法已经调用过的对象直接回收。若需要执行finalize()方法的对象进入F-Queue队列，第二次可达性分析如果还没有复活则回收。
>
> 可达性分析工作必须在一个能确保一致性的快照中进行。这导致GC时必须停顿所有Java执行线程（Stop The World）。

目前主流的Java虚拟机采用的是准确式GC，即执行系统停顿下来之后，并不需要一个不漏地检查完所有执行上下文和全局的引用位置，虚拟机可以直接得知哪些地方存放着对象引用。比如在HotSpot中使用OopMap的数据结构来达到这一目的。在类加载完成的时候，HotSpot就把对象内什么偏移量上是什么类型的数据计算出来，在JIT编译过程中，也会在特定的位置记录下栈和寄存器中哪些位置是引用。这样GC在扫描时就可以直接得知这些信息了。

#### 浅析OopMap

如果JVM选择不记录任何GC Roots数据，它就无法区分内存中某个位置上的数据到底应该被解读为引用类型还是基本类型，这种情况下，JVM就会采取保守式GC，从一些已知位置（比如JVM栈）开始扫描内存，扫描时检查指针像不像一个指向GC堆中的指针，这里涉及上下边界检查（GC堆的上下界是已知的）、对齐检查（分配内存空间的时候会有对齐要求，比如4字节对齐，那么不能被4整除的数字肯定不是指针）。然后递归地扫描。

保守式GC的好处是相对来说实现简单些，而且可以方便的用在对GC没有特别支持的编程语言里提供自动内存管理功能。

保守式GC的缺点有： 
1、会有部分对象本来应该已经死了，但有疑似指针指向它们，使它们逃过GC的收集。这对程序语义来说是安全的，因为所有应该活着的对象都会是活的；但对内存占用量来说就不是件好事，总会有一些已经不需要的数据还占用着GC堆空间。具体实现可以通过一些调节来让这种无用对象的比例少一些，可以缓解（但不能根治）内存占用量大的问题。

2、由于不知道疑似指针是否真的是指针，所以它们的值都不能改写；移动对象就意味着要修正指针。换言之，对象就不可移动了。有一种办法可以在使用保守式GC的同时支持对象的移动，那就是增加一个间接层，不直接通过指针来实现引用，而是添加一层“句柄”（handle）在中间，所有引用先指到一个句柄表里，再从句柄表找到实际对象。这样，要移动对象的话，只要修改句柄表里的内容即可。但是这样的话引用的访问速度就降低了。Sun JDK的Classic VM用过这种全handle的设计，但效果实在算不上好

3、耗时

由于JVM要支持丰富的反射功能，本来就需要让对象能了解自身的结构，而这种信息GC也可以利用上，所以很少有JVM会用完全保守式的GC。

##### 半保守式GC

JVM可以选择在栈上不记录类型信息，而在对象上记录类型信息。这样的话，扫描栈的时候仍然会跟上面说的过程一样，但扫描到GC堆内的对象时因为对象带有足够类型信息了，JVM就能够判断出在该对象内什么位置的数据是引用类型了。这种是“半保守式GC”，也称为“根上保守（conservative with respect to the roots）”。为了支持半保守式GC，运行时需要在对象上带有足够的元数据。如果是JVM的话，这些数据可能在类加载器或者对象模型的模块里计算得到，但不需要JIT编译器的特别支持。由于半保守式GC在堆内部的数据是准确的，所以它可以在直接使用指针来实现引用的条件下支持部分对象的移动，方法是只将保守扫描能直接扫到的对象设置为不可移动（pinned），而从它们出发再扫描到的对象就可以移动了。 

完全保守的GC通常使用不移动对象的[算法](http://lib.csdn.net/base/datastructure)，例如mark-sweep。半保守方式的GC既可以使用mark-sweep，也可以使用移动部分对象的算法。

##### 准确式GC

与保守式GC相对的是“准确式GC”，原文可以是precise GC、exact GC、accurate GC或者type accurate GC。

是什么东西“准确”呢？关键就是“类型”，也就是说给定某个位置上的某块数据，要能知道它的准确类型是什么，这样才可以合理地解读数据的含义；GC所关心的含义就是“这块数据是不是指针”。要实现这样的GC，JVM就要能够判断出所有位置上的数据是不是指向GC堆里的引用，包括活动记录（栈+寄存器）里的数据。

办法：

1. 让数据自身带上标记（tag）。这种做法在JVM里不常见，但在别的一些语言实现里有体现。
2. 让编译器为每个方法生成特别的扫描代码。我还没见过JVM实现里这么做的
3. 从外部记录下类型信息，存成映射表。现在三种主流的高性能JVM实现，HotSpot、JRockit和J9都是这样做的。其中，HotSpot把这样的数据结构叫做OopMap，JRockit里叫做livemap，J9里叫做GC map。

要实现这种功能，需要虚拟机里的解释器和JIT编译器都有相应的支持，由它们来生成足够的元数据提供给GC。

使用这样的映射表一般有两种方式： 
1、每次都遍历原始的映射表，循环的一个个偏移量扫描过去；这种用法也叫“解释式”； 
2、为每个映射表生成一块定制的扫描代码（想像扫描映射表的循环被展开的样子），以后每次要用映射表就直接执行生成的扫描代码；这种用法也叫“编译式”。

在HotSpot中，对象的类型信息里有记录自己的OopMap，记录了在该类型的对象内什么偏移量上是什么类型的数据。所以从对象开始向外的扫描可以是准确的；这些数据是在类加载过程中计算得到的。

可以把oopMap简单理解成是调试信息。 在源代码里面每个变量都是有类型的，但是编译之后的代码就只有变量在栈上的位置了。oopMap就是一个附加的信息，告诉你栈上哪个位置本来是个什么东西。 这个信息是在JIT编译时跟机器码一起产生的。因为只有编译器知道源代码跟产生的代码的对应关系。 每个方法可能会有好几个oopMap，就是根据safepoint把一个方法的代码分成几段，每一段代码一个oopMap，作用域自然也仅限于这一段代码。 循环中引用多个对象，肯定会有多个变量，编译后占据栈上的多个位置。那这段代码的oopMap就会包含多条记录。

每个被JIT编译过后的方法也会在一些特定的位置记录下OopMap，记录了执行到该方法的某条指令的时候，栈上和寄存器里哪些位置是引用。这样GC在扫描栈的时候就会查询这些OopMap就知道哪里是引用了。这些特定的位置主要在： 
1、循环的末尾 
2、方法临返回前 / 调用方法的call指令后 
3、可能抛异常的位置

这种位置被称为“安全点”（safepoint）。

平时这些OopMap都是压缩了存在内存里的；在GC的时候才按需解压出来使用。 
HotSpot是用“解释式”的方式来使用OopMap的，每次都循环变量里面的项来扫描对应的偏移量。

对Java线程中的JNI方法，它们既不是由JVM里的解释器执行的，也不是由JVM的JIT编译器生成的，所以会缺少OopMap信息。那么GC碰到这样的栈帧该如何维持准确性呢？ 
HotSpot的解决方法是：所有经过JNI调用边界（调用JNI方法传入的参数、从JNI方法传回的返回值）的引用都必须用“句柄”（handle）包装起来。JNI需要调用[Java ](http://lib.csdn.net/base/java)API的时候也必须自己用句柄包装指针。在这种实现中，JNI方法里写的“jobject”实际上不是直接指向对象的指针，而是先指向一个句柄，通过句柄才能间接访问到对象。这样在扫描到JNI方法的时候就不需要扫描它的栈帧了——只要扫描句柄表就可以得到所有从JNI方法能访问到的GC堆里的对象。 
但这也就意味着调用JNI方法会有句柄的包装/拆包装的开销，是导致JNI方法的调用比较慢的原因之一。

#### 安全点

HotSpot没必要为每条指令都生成OopMap，只在特定的位置记录这些信息，这些信息被称为安全点，即程序执行时并非在所有地方都能停下来开始GC，只有在到达安全点才能暂停GC。安全点的选定基本上是以程序“是否具有让程序长时间执行的特征”为标准进行选定的。“长时间执行”的最明显特征就是指令序列的复用，比如方法调用、循环跳转、异常跳转等。具备这些功能的指令才会产生Safepoint。

执行GC时线程中断的方式：

抢先式中断：首先中断所有线程，如果发现有线程中断的地方不在安全点上，就恢复线程，让它跑到安全点上。（几乎没有虚拟机实现）

主动式中断：简单设置一个标志，各个线程执行时主动去轮询这个标志，发现中断标志为真时就自己中断挂起。轮询标志的地方和安全点是重合的，另外加上创建对象需要分配内存的地方。

#### 安全区域

当程序没有被分配CPU时间，比如sleep或者blocked状态，这时候线程无法响应JVM的中断请求，"走"到安全的地方去中断挂起，JVM也不会等待线程重新分配CPU时间。这里使用安全区域解决。安全区域指代在一段代码片段中，引用关系不会发生变化。这个区域中的任何地方开始GC都是安全的。在线程执行到safe region中的代码时，首先标时自己进入了安全区域，这样，当JVM发起GC时，它要检查系统是否已经完成了根节点枚举（或者整个GC过程），如果完成了，那么线程将继续执行，否则它就必须等待知道收到可以离开安全区域的信号为止。

#### 回收方法区

Java虚拟机规范中说可以不要求虚拟机在方法区中实现垃圾回收

永久代的垃圾收集主要回收两部分内容：废弃常量和无用的类

判断废弃常量的标准很简单，当前系统中没有任何引用，就会被清理

而判断一个无用的类的条件如下：

1. 该类的所有实例都已经被回收
2. 加载该类的ClassLoader已经被回收
3. 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法

满足上述三个方法后就可以被回收，但不是必然回收，虚拟机给了参数来控制

在大量使用反射、动态代理、CGLib等ByteCode框架、动态生成JSP以及OSGi这类频繁自定义ClassLoader的场景都需要虚拟机具备类卸载的功能。

#### 垃圾收集算法

##### 标记-清除算法

不足有两个：一是效率问题，标记和清除两个过程的效率都不高；二是空间问题，标记清除后会产生大量的内存碎片

##### 复制算法

将可用内存按容量大小分为大小相等的两块，每次只是用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后把已经使用过的内存空间一次清理掉。内存划分并不需要1比1，而是将内存块分为一块较大的Eden空间和两块较小的Survivor空间，每次使用Eden和其中一块Survivor。每次回收时，将Eden和Survivor中还存活着的对象一次性复制到另一个Survivor空间上，最后清理掉Eden和Survivor空间。HotSpot虚拟机默认的Eden和Survivor的大小比例是8：1。当Survivor空间不够时，需要依赖其他内存（老年代）进行分配担保

##### 标记-整理算法

复制算法在对象存活率较高时效率变低，并且如果即不想浪费很多空间，又不想分配额外的空间来进行分配担保。在老年代中提出了标记-整理算法。在标记过程之后，让所有存活的对象都朝一端移动，然后直接清理掉端边以外的内存。

##### 分代收集算法

根据对象存活周期的不同将内存划分为几块。一般将Java堆分为新生代和老生代，这样就可以根据各个年代的特点选择合适的收集算法。新生代选用复制算法，老年代使用标记-清理或者标记-整理算法。

#### 垃圾收集器



#### 什么时候回收

gc分为：minior gc（清理年轻代）、major gc（清理老年代）、full gc（清理整个堆空间）
1、eden满了minor gc
2、老年代剩余空间即将不足full gc
3、空间分配担保策略，尽量尝试minor gc，失败则full gc
4、统计得到每次Minor GC晋升到老年代的平均大小大于老年代的剩余空间直接full GC

#### 可作为GC Roots的对象包括下面几种：

1. 虚拟机栈（当前活跃栈帧中的本地变量表）中指向GC堆的对象的引用；
2. 方法区中引用类型静态变量；
3. 方法区中引用类型常量（public static final）；
4. 本地方法栈中JNI（即一般说的Native方法）引用的对象

**注意。是一组必须活跃的引用，不是对象**

Tracing GC的根本思路就是：给定一个集合的引用作为根出发，通过引用关系遍历对象图，能被遍历到的（可到达的）对象就被判定为存活，其余对象（也就是没有被遍历到的）就自然被判定为死亡。注意再注意：tracing GC的本质是通过找出所有活对象来把其余空间认定为“无用”，而不是找出所有死掉的对象并回收它们占用的空间。
GC roots这组引用是tracing GC的**起点**。要实现语义正确的tracing GC，就必须要能完整枚举出**所有的GC roots**，否则就可能会漏扫描应该存活的对象，导致GC错误回收了这些被漏扫的活对象。

注意普通的成员变量是不能作为GC Roots的。因为成员变量是和对象共存亡的

参考：https://www.zhihu.com/question/53613423/answer/135743258

### 不建议显式的声明 System.gc()

因为显式声明是做堆内存全扫描，也就是 Full GC ，是需要停止所有的活动的(Stop The World Collection)，对应用很大可能存在影响。

调用 System.gc() 方法后，不会立即执行 Full GC ，而是虚拟机自己决定的

不同的引用类型，主要体现的是对象不同的可达性（reachable）状态和对垃圾收集的影响

对于一个普通的对象，如果没有其他的引用关系，只要超过了引用的作用域或者显式地将相应（强）引用赋值为 null，就是可以被垃圾收集的了，当然具体回收时机还是要看垃圾收集策略

### 引用的概念

Java中引用可以分为：

> - 强引用。代码中普遍存在，只要强引用在，垃圾回收器永远不会回收掉被引用的对象；
>- 软引用。描述一些还有用但并非必须的对象，在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围之中进行第二次回收（SoftReference类实现）；
> - 弱引用。比软引用再弱一点，被弱引用关联的对象只能生存到下一次垃圾收集发生之前；（使用实例：ThreadLocal类）
> - 虚引用。一个对象设置虚引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知（PhantomReference类）。

> 如何判定对象死亡：
>
> ​	要宣告一个对象死亡，至少要经历两次标记过程：如果对象在进行可达性分析之后发现没有与GC Roots相连的引用链，那么它们就会被第一次标记并且进行一次筛选，筛选的条件是该对象是否有必要执行finalize方法。当对象没有覆盖finalize方法或者该方法已经被虚拟机调用过，虚拟机将这两种情况视为“没有必要执行”，将被直接回收。
>
> ​	若对象被判定为有必要执行，那么这些对象将被放置在一个叫F-Queue的队列中，并稍后由一个由虚拟机自动建立的、低优先级的Finalizer线程去执行它。虚拟机会触发这个方法，但是不会等待它运行结束，稍后GC将对F-Queue队列中的对象进行第二次小规模标记，若此时这些对象还没有与引用链上的对象建立关联，那么它就真的被回收了。任何一个对象的finalize方法都只会被系统自动调用一次。

### 虚引用与软引用和弱引用的区别？软引用的好处？

虚引用与软引用和弱引用的一个区别在于： 虚引用必须和引用队列（ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。程序如果发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动。

软引用可以加速JVM对垃圾内存的回收速度，可以维护系统的运行安全，防止内存溢出（OutOfMemory）等问题的产生

方法区的垃圾收集主要回收两部分内容：废弃常量和无用的类。只要没有其它地方引用了这个常量，此时发生内存回收，而且必要的话，就会被清理出常量池。

判定一个类是否是无用的类要满足三个条件：

- 该类的所有实例都已经被回收
- 加载该类的[ClassLoader](https://www.cnblogs.com/z00377750/p/9175549.html)已经被回收
- 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法

满足上述三个条件的无用类可以回收，但不像对象那样，不使用了就会必然回收。

垃圾收集算法

> 标记-清除算法
>
> 不足点：效率问题，标记清除两个过程的效率都不高；空间问题，标记清除后会产生大量不连续的内存碎片。
>
> 复制算法
>
> 将可用内存分为按容量划分为大小相等的两块，每次只使用其中一块。当其中一块内存使用完了，就将还存活着的对象复制到另外一块上面，然后将已使用过的内存空间一次清理掉。前提是对象存活率较低，否则效率会低。
>
> 不足点：内存缩小，代价太高
>
> 标记-整理算法
>
> 标记过程和标记-清除算法一样，但后续步骤让所有存活对象向一端移动，然后直接清理掉端边界外的内存
>
> 分代收集算法
>
> 当前的商业虚拟机的垃圾收集都采用的算法。根据对象存活周期的不同将内存划分为几块，一般是把Java堆分成新生代和老生代。在新生代中，每次垃圾收集时都有大批对象死去，只有少量存活，就选用复制算法。老年代中对象存活率高，没有额外空间对它进行分配担保，就使用标记-整理算法来进行回收。

#### 分代收集算法：为什么要分为新生代和老年代？

分代指的是根据对象存活周期的不同将内存分为几块，一般将java堆分为新生代和老年代，这样我们就可以根据各个年代的特点选择合适的垃圾收集算法。

分代的理由：优化GC性能。如果没有分代，那所有的对象都在一块，GC的时候要找到哪些对象没用，这样就会对堆的所有区域进行扫描。而很多对象都是朝生夕死的，如果分代的话，可以把新创建的对象放到某一地方，当GC的时候先把这块存“朝生夕死”对象的区域进行回收，这样就会腾出很大的空间。

在垃圾收集算法的选择方面：对于新生代，每次收集都会有大量对象死去，所以可以选择复制算法，只需要付出少量对象的复制成本就可以完成每次垃圾收集。而老年代的对象存活率比较高，所以我们可以选择“标记-清理”或“标记-整理”算法进行垃圾收集

HotSpot中上述算法的实现

> 枚举根节点
>
> ​	可作为GC Roots的节点主要在全局性的引用与执行上下文（栈帧中的本地变量表）中，逐个检查这里的引用会消耗很多时间。可达性分析对执行时间敏感，在GC进行时必须停顿所有Java线程。
>
> ​	准确式GC，当执行系统停顿后并不需要检查每个执行上下文和全局的引用位置，虚拟机有办法直接得知哪里存放着对象引用。OopMap--映射表（需要解释器和JIT编译器的支持），GC停顿的时候，虚拟机可以通过OopMap这样的一个映射表知道，在对象内的什么偏移量上是什么类型的数据，而且特定的位置记录着栈和寄存器中哪些位置是引用

> 安全点
>
> 在指令的特定位置记录OopMap的这些位置就是“安全点”，程序只能在到达这些安全点后开始GC。具有指令复用特征的位置才能作为安全点，比如方法调用、循环跳转、异常跳转等位置。
>
> 两种线程停顿开始GC的方案：
>
> - 抢占式中断，在GC发生时，将所有线程中断，如果发现有线程没在安全点上中断，就恢复线程，让它跑到安全点上；
> - 主动式中断，设置一个轮询标志，位置与安全点重合（另外在加上创建对象需要分配内存的地方），让各个线程轮询这个标志，发现标志为真时就自己中断挂起

> 安全区域
>
> 当线程处于sleep或者blocked状态时，如何响应JVM的中断请求，走到安全点中断呢？使用安全区域，当线程走到安全区域中的代码时，先表示自己进入了安全区域，当这段时间里JVM开始GC时，就可以不管处于安全区域状态的线程了。当线程离开安全区域时，检查系统是否完成了GC过程，如果完成就继续执行，否则挂起等待直到收到可以安全离开安全区域的信号为止。

### 查看JVM内存使用详情的命令：

jmap：jmap命令是一个可以输出所有内存中对象的工具，甚至可以将VM 中的heap，以二进制输出成文本。

jhat也是jdk内置的工具之一。主要是用来分析java堆的命令，可以将堆中的对象以html的形式显示出来，包括对象的数量，大小等等，并支持**对象查询语言**。

第一步：导出堆

\# jmap -dump:live,format=b,file=/path/heap.bin 进程ID

\# jmap -dump:format=b,file=/path/heap.bin 进程ID

**live参数：**

表示我们需要抓取目前在生命周期内的内存对象,也就是说GC收不走的对象,然后我们绝大部分情况下,需要的看的就是这些内存。而且会减小dump文件的大小。

第二步：分析堆文件

\#jhat -J-Xmx512M a1.log

说明：有时dump出来的堆很大，在启动时会报堆空间不足的错误，可加参数：jhat -J-Xmx512m <heap dump file>。这个内存大小可根据自己电脑进行设置。

第三步：查看html

解析Java堆转储文件,并启动一个 web server

### Java命令：常见命令

在JDK的bin目彔下,包含了java命令及其他实用工具。

> jps:查看本机的Java中进程信息。
>  jstack:打印线程的栈信息,制作线程Dump。
>  jmap:打印内存映射,制作堆Dump。
>  jstat:性能监控工具。
>  jhat:内存分析工具。
>  jconsole:简易的可视化控制台。
>  jvisualvm:功能强大的控制台

##### 使用命令行制作Dump

`jstack`:打印线程的栈信息,制作线程Dump。

`jmap`:打印内存映射,制作堆Dump。

##### 步骤：

1. 检查虚拟机版本（java -version）
2. 找出目标Java应用的进程ID（jps）
3. 使用jstack命令制作线程Dump • Linux环境下使用kill命令制作线程Dump
4. 使用jmap命令制作堆Dump

### 什么是GC停顿

GC停顿是指GC进行时必须停顿所有的Java执行线程，因为进行可达性分析时，必须保证系统中对象的引用关系不会发生变化，否则可达性分析的准确性无法得到保证。即使是（几乎）不会发生停顿的CMS收集器在枚举根节点时也必须要停顿。

垃圾收集器

垃圾收集器是内存回收的具体实现。

并发和并行在垃圾收集器的上下文语境中的解释：

​	并行（Parallel）：之多条垃圾线程并行工作，但此时用户线程仍然处于等待状态；

​	并发（Concurrent）：指用户线程与垃圾收集线程同时执行（但不一定是并行的，可能会交替执行），用户程序在继续运行，而垃圾收集程序运行于另一个CPU上。

> Serial收集器
>
> 新生代收集器，单线程，只会使用一个CPU或一条收集线程去完成垃圾收集工作，在进行垃圾收集时，必须暂停其他所有工作线程，直到它收集结束。JVM在用户不可见的情况下把用户正常工作的线程全部停掉。简单高效，适用于运行在Client模式下的虚拟机
>
> ParNew收集器
>
> Serial收集器的多线程版本，除了Serial之外，只有它可以和CMS收集器配合使用
>
> Parallel Scavenge收集器
>
> 新生代收集器。其他收集器的关注点是尽可能地缩短垃圾收集时用户线程的停顿时间，而Parallel Scavenge收集器的目标是达到一个可控的吞吐量。停顿时间越短就越适合需要与用户交互的程序，而提高吞吐量则可以高效地利用CPU时间，尽快地完成程序的运算任务。GC停顿时间缩短是以牺牲吞吐量和新生代空间来换取的。（多线程，可能无法与用户线程同时运行）
>
> Serial Old收集器
>
> Serial的老年代版本，单线程，使用“标记-整理”算法。
>
> Parallel Old收集器
>
> Parallel Scavenge收集器的老年代版本，采用“标记-整理”算法
>
> CMS收集器
>
> 老年代收集器
>
> 以获取最短回收停顿时间为目标的收集器，基于“标记-清除”算法，运作过程如下：
>
> 1. 初始标记
> 2. 并发标记
> 3. 重新标记
> 4. 并发清除
>
> 初始标记、重新标记需要暂停所有用户线程（Stop The World）。初始标记仅仅需要标记GC Roots能直接关联到的对象，速度很快；并发标记就是进行GC Roots Tracing的过程；重新标记则是为了修正并发标记期间因为用户程序运行而导致标记产生变动的对象的标记记录，这个阶段停顿时间比一般初始标记阶段稍长，但远比并发标记时间短。
>
> 并发标记和并发清除过程收集器线程都可以与用户线程一起工作。
>
> 缺点如下：
>
> - CMS收集器对CPU资源非常敏感；
> - CMS收集器无法处理浮动垃圾（Floating Garbage）；由于并发处理阶段用户线程还在运行，伴随程序运行会有新的垃圾不断产生，CMS无法在当次收集中处理它们，只好等待下次GC时处理。因为工作线程的运行，CMS收集器不能像其他收集器那样等到老年代几乎被填满在进行收集，需要预留一部分空间供用户程序使用。如果CMS运行期间预留的内存无法满足程序需要，就会出现一次“Concurrent Mode Failure”，此时虚拟机将启用老年代收集器重新进行老年代的垃圾收集，这样停顿时间就太长了。
> - “标记-清除”会产生大量空间碎片，如果无法找到足够大的连续存储空间，就会不得不触发一次Full GC。在CMS要开启FULL GC时开启内存碎片的合并整理过程，内存整理过程无法并发，使得停顿时间变长。
>
> G1收集器
>
> 面向服务器的垃圾收集器,主要针对配备多颗处理器及大容量内存的机器. 以极高概率满足GC停顿时间要求的同时,还具备高吞吐量性能特征
>
> 特点：
>
> 并发运行；可作用于新生代或老年代；标记-整理算法+复制算法；响应速度优先；面向服务端应用。
>
> 在G1算法中，采用了另外一种完全不同的方式组织堆内存，堆内存被划分为多个大小相等的内存块（Region），每个Region是逻辑连续的一段内存，并跟踪这些区域里的垃圾价值（回收可获得空间/回收所需时间的经验值）大小，在后台维护一个优先列表，每次根据允许的收集时间，优先回收垃圾最多的区域（这也是Garbage First名称的由来）。总而言之，区域划分和有优先级的区域回收，保证了G1收集器在有限的时间内可以获得最高的收集效率。
>
> 步骤：
>
> - 初始标记
> - 并发标记
> - 最终标记
> - 筛选回收
>
> G1中提供了三种模式的垃圾回收模式：young gc、mixed gc和full gc

jdk1.8中**默认**的是Parallel Scavenge+Seroal Old的垃圾收集器组合

#### 内存分配和回收策略

对象主要分配在新生代的Eden区上，如果启动了本地线程分配缓冲，将按线程优先在TLAB上分配。少数情况可能直接分配在老年代中。当Eden区没有足够空间进行分配时，虚拟机将发起一次Minor GC

MInor GC：指发生在新生代的垃圾收集动作

Major GC/Full GC：发生在老年代的GC，一般出现时经常伴随至少一次的Minor GC（不绝对），Major GC一般比Minor GC慢10倍以上 

Full GC：整堆包括新生代和老年代的垃圾回收称为Full GC（HotSpot VM里，除了CMS之外，其它能收集老年代的GC都会同时收集整个GC堆，包括新生代）

大对象指大量需要连续内存的Java对象。虚拟机提供参数令大于这个阈值的对象直接在老年代分配，这样做的目的是避免在Eden区以及两个Survivor区之间发生大量的内存复制

对象优先在Eden区分配，如果Eden区不够，则进行一次Minor GC

虚拟机给每个对象定义了一个对象年龄计数器，如果对象在Eden出生并经过第一次Minor GC后仍然存活，并且能被Survivor容纳的话，将被移动到Survivor空间中，并且对象年龄置为1。对象在Survivor区中每熬过一次Minor GC，年龄就增加1岁，当年龄增加到一定程度，就被晋升到老年代中，该阈值通过参数设置。如果Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，无需等到年龄到达阈值时。

在GC开始的时候，对象只会存在于Eden区和名为“From”的Survivor区，Survivor区“To”是空的。紧接着进行GC，Eden区中所有存活的对象都会被复制到“To”，而在“From”区中，仍存活的对象会根据他们的年龄值来决定去向。年龄达到一定值(年龄阈值，可以通过`-XX:MaxTenuringThreshold`来设置，默认15)的对象会被移动到年老代中，没有达到阈值的对象会被复制到“To”区域。经过这次GC后，Eden区和From区已经被清空。这个时候，“From”和“To”会交换他们的角色，也就是新的“To”就是上次GC前的“From”，新的“From”就是上次GC前的“To”。不管怎样，都会保证名为To的Survivor区域是空的。Minor GC会一直重复这样的过程，直到“To”区被填满，“To”区被填满之后，会将所有对象移动到年老代中。

##### 对象的分配规则

+ 对象优先在Eden区

  如果 Eden 区无法分配，会触发Minor GC：将Eden区和s0区活着的对象复制到s1区。然后s0,s1互换

  + Minor GC中存活的object 如果超过tenuring threshold，则晋升到老年代
  + 如果Minor GC时s1无法存放Eden和s0的对象，则会触发内存担保机制，无法存放的对象放入老年代

+ 大对象直接进入老年代（大对象是指需要大量连续内存空间的对象）。避免在新生代发生大量的内存拷贝

+ 长期存活的对象进入老年代。每经过一次Minor GC年龄加1，达到年龄时进入老年代

+ 动态判断对象的年龄：如果 Survivor 区中相同年龄的所有对象大小的总和大于 Survivor 空间的一半，年龄大于或等于该年龄的对象可以直接进入老年代

+ 空间分配担保：Minor GC前，会检查是老年代最大可用连续空间是否大于新生代所有对象总空间。如果大于，则GC是安全的，否则检查HandlePromotionFailure是否允许担保失败。如果允许，则检查老年代最大可用连续空间是否大于晋升到老年代对象的平均大小，如果大于则进行Minor GC。如果小于则Full GC。如果设置不允许担保失败，则直接触发Full GC。（**上述的分配担保只在JDK 6中存在**）

  **新的担保机制**：HandlePromotionFailure参数已经取消，判断老年代的连续空间大小是否大于新生代对象总大小或者历次晋升的平均大小，如果大于则进行Minor GC，否则进行Full GC

#### 为什么新生代要分成Eden区和Survivor区？为什么要有两个Survivor分区

如果没有Survivor，Eden区每进行一次Minor GC，存活的对象就会被送到老年代。老年代很快被填满，触发Major GC（因为Major GC一般伴随着Minor GC，也可以看做触发了Full GC）。老年代的内存空间远大于新生代，进行一次Full GC消耗的时间比Minor GC长得多。频繁进行Full GC会导致程序变慢。

两个Survivor区是为了避免内存碎片化。首先根据上面的说法必须没置Survivor区，如果只有一个Survivor区，当Eden区满时触发Minor GC，此时Eden区活着的对象移到Survivor区，不断进行，后面在Minor GC时，Eden和Survivor中都有存活的对象，将Eden的对象放入Survivor区，此时Survivor区会出现内存碎片化。

通过两个Survivor区，在Minor GC时，将Eden区和From Survivor区的存活对象复制到To Survivor区，然后清空Eden和From区，并颠倒From区和To区的逻辑关系，这样Survivor区中也不会有内存碎片。整个过程中，在一次Minor GC后，To survivor永远是空的，From survivor无碎片

也就是**通过两个Survivor区可以减少Full GC的次数及避免内存碎片**

#### Minor GC、Major GC、Full GC及触发条件

在HotSpot中准解分类有两种

+ 部分GC
  + Young GC:新生代GC，或者叫Minor GC,只收集young gen。
  + old GC:只收集老年代，只有CMS的并行收集有这个模式
  + Mixed GC：收集整个young gen以及部分old gen的GC。只有G1有这个模式
+ Full GC：Major GC通常是跟full GC是等价的
  + 收集整个堆，包括young gen、old gen、perm gen

触发条件：

+ young GC：当young gen中的eden区分配满的时候触发
+ full GC：
  + 准备要触发一次young GC时，如果发young GC的平均晋升大小比目前old gen剩余的空间大，则不会触发young GC而是转为触发full GC
  + 如果有perm gen的话，要在perm gen分配空间但已经没有足够空间时，也要触发一次full GC
  + System.gc()
  + heap dump带GC（？）

### 类文件结构

------

Class文件中包含了Java虚拟机指令集和符号表以及若干其他辅助信息。有一些Java语言本身无法有效支持的语言特性不代表字节码本身无法有效支持，这也为其他语言实现一些有别于Java的语言特性提供了基础。

任何一个Class文件都对应着唯一一个类或接口的定义信息，但反过来说，类或接口不一定都得定义在文件里。

以8位字节为基础单位的二进制流，各个数据项目紧凑排列，没有空隙存在。超过8位字节的数据项按照高位在前（大端）的方式分割成若干个8位字节进行存储。（高位字节在地址最低位，地位字节在地址最高位）

存储结构的两种数据类型：无符号数和表

当需要描述同一类型但数量不定的多个数据时，经常会使用一个前置的容量计数器加若干个连续的数据项形式，称之为某一类型的集合。

无论是顺序还是数量，甚至于数据存储的字节序都是被严格限定的。

前八个字节，魔数（前四个字节）和版本号（后四字节，前2字节次版本号，后2字节主版本号）

常量池

> 占用Class文件空间最大的数据项目之一，Class文件中第一个出现的表类型数据项目。
>
> 常量池入口放置u2类型的数据，代表常量池容量计数器，计数从1开始，满足后面指向常量池的索引值的数据在特定情况下表达”不应用任何一个常量池项目“的含义。
>
> 存放两大类常量：字面量和符号引用
>
> 字面量：文本字符串、声明为final的常量值等
>
> 字面量和符号引用。字面量比较接近于java语言层面的的常量概念，如文本字符串、声明为final的常量值等。而符号引用则属于编译原理方面的概念。包括下面三类常量：类和接口的全限定名，字段的名称和描述符，方法的名称和描述符
>
> 符号引用：
>
> - 类和接口的全限定名
> - 字段的名称和描述符
> - 方法的名称和描述符
>
> 虚拟机在加载Class文件时动态连接，在Class文件中不会保存各个方法、字段的最终内存布局信息，因此这些方法、字段的符号引用不经过运行期转换的话就无法得到真正的内存入口地址，也就无法被虚拟机直接应用。当虚拟机运行时，需要从常量池获得对应的符号引用，再在类创建时或运行时解析、翻译到具体的内存地址中
>
> 常量池中每一项常量都是一个表，现有14中表。表开始的第一位是u1类型的标志位，代表这个常量属于哪种常量类型。由于Class文件中方法、字段等都需要引用CONSTANT_Utf8_info型常量来描述名称，所以CONSTANT_Utf8_info型常量的最大长度也就是Java中的方法、字段名的最大长度。

访问标志

用于识别一些类或者接口层次的访问信息，包括：是类还是接口、是否被定义为public类型、是否定义为abstract类型、是否被声明为final（如果是类的话）。有16位可用，但只定义了其中8个

类索引、父类索引与接口索引集合

类索引、父类索引都是一个u2类型的数据，接口索引是一组u2类型的数据的集合。类索引用于确定这个类的全限定名，父类索引用于确定这个类的父类的全限定名。所有Java类都有父类，除了Object之外。接口索引集合用来描述这个类实现了哪些接口

字段表集合

用于描述接口或者类中声明的变量。字段包括类级变量以及实例级变量，不包括在方法内部声明的局部变量。

每个字段表的格式：access_flags+name_index+descriptor_index+属性表集合

描述符（descriptor_index）的作用：描述字段的数据类型、方法的参数列表（数量、类型和顺序）和返回值，基本数据类型以及代表无返回值的void类型都用一个大写字符来表示，而对象类型则用字符L加对象的全限定名来表示。对于数组类型，每一维度将使用前置的"["字符来描述，比如String\[\]\[\]将被记录为"[[Ljava/lang/String;"。

用描述符描述方法时，按照先参数列表，后返回值的顺序描述，参数列表按照参数的严格顺序放在一组小括号"()"内。字段表集合中不会列出从超类或者父接口中继承而来的字段。但又可能列出原本Java代码中不存在的字段，譬如内部类中为了保持对外部类的访问性，会自动添加指向外部类实例的字段。

在Java中语言中字段是无法重载的，两个字段的数据类型、修饰符不管是否相同，都必须使用不一样的名称，但是对于字节码来说，如果两个字段的描述符不一致，那字段重名就是合法的。

方法表集合

volatile和transient不能修饰方法，方法中的Java代码经过编译器译成字节码指令后，存放在方法属性表集合中Code属性中。

如果父类方法在字类中没有被重写，方法表集合中就不会出现来自父类的方法信息。但是同样地，有可能会出现来自编译器自动添加的方法，最典型的就是类构造器"<clinit>"和实例构造器"<init>"方法。

Java中的重载除了要求与原方法具有相同的简单名称之外，还必须拥有一个与原方法不同的特征签名，特征签名就是一个方法中各个参数在常量池中的字段符号引用的集合，返回值不会包含在特征签名中，Java语言里面是无法仅仅依靠返回值的不同来对一个已有方法进行重载的。但在Class文件格式中，如果两个方法拥有相同的名称和特征签名，但是返回值不同，那么也可以合法地共存于同一个Class文件中。**字节码的特征签名还包含方法返回值和受查异常表。**

重写：运行时的多态性

重载：编译时的多态性

属性表集合

Class文件、字段表、方法表都可以携带自己的属性集合。

属性表集合不要求各个属性表有严格的顺序，并且只要不与已有的属性名重复，任何人实现的编译器都可以向属性表中写入自己定义的属性信息，Java虚拟机运行时会忽略掉它不认识的属性。虚拟机规范中预定义的属性有21项。

> Code属性
>
> Java程序方法体中的代码经过Javac编译器处理后，最终变为字节码指令存储在Code属性内。并非所有的方法表都必须存在这个属性，比如抽象类中的方法或者接口就不存在Code属性。Code属性值的长度为整个属性表的长度减去6个字节。max_stack代表操作数栈深度的最大值，max_locals代表了局部变量表所需的存储空间，单位为slot，1slot等于4字节。局部变量表存放方法参数（包括this）、显示异常处理器的参数、方法中定义的局部变量。局部变量表可以重用。
>
> javac编译器编译的时候把对this关键字的访问转变为对一个普通方法参数的访问，然后在虚拟机调用实例方法时自动传入此参数。
>
> 编译器使用异常表而不是简单的跳转指令来实现Java异常及finally处理机制。

> Exceptions属性
>
> 列举出方法中可能抛出的受查异常，也就是方法描述时在throws关键字后列举的异常

> ConstantValue属性
>
> 通知虚拟机自动为静态变量赋值。只有被static关键字修饰的变量才能使用这项属性。对于类变量，有两种赋值方式可选择：在类构造器<clinit>方法或者使用ConstantValue属性。如果是同时使用final和static来修饰一个变量，并且变量的数据类型是基本类型或者String的话，就生成ConstantValue属性（在类加载的时候）来进行初始化，否则会选择在<clinit>方法中进行初始化。

### 虚似机类加载

------

​	虚拟机把描述类的数据从class文件加载到内存，并对数据进行校验、转换解析和初始化。最终形成可以被虚拟机最直接使用的java类型的过程就是虚拟机的类加载机制

​	java语言中类型的加载连接以及初始化过程都是在程序运行期间完成的

#### 类加载的过程（Java类是如何加载的）

类从加载到虚拟机内存到卸载出内存，整个生命周期包括：加载、验证、准备、解析、初始化、使用和卸载。其中验证、准备、解析统称为连接。

1. 首先是**加载阶段**（Loading），它是 Java 将字节码数据从不同的数据源读取到 JVM 中，并映射为 JVM 认可的数据结构（Class 对象）；主要有3步：根据类的全限定名找到class文件。（类加载器）；将二进制字节流的存储结构转化为特定的数据结构，存储在方法区中；在内存中创建`java.lang.Class`类型的对象。如果输入数据不是 ClassFile 的结构，则会抛出 ClassFormatError。

   加载阶段是用户参与的阶段，我们可以自定义类加载器，去实现自己的类加载过程。

2. 第二阶段是**链接**（Linking），这是核心的步骤，简单说是把原始的类定义信息平滑地转化入 JVM 运行的过程中。这里可进一步细分为三个步骤：

   + 验证（Verification），这是虚拟机安全的重要保障，JVM 需要核验字节信息是符合 Java 虚拟机规范的，否则就被认为是 VerifyError，这样就防止了恶意信息或者不合规的信息危害 JVM 的运行，验证阶段有可能触发更多 class 的加载。

   + 准备（Preparation），创建类或接口中的静态变量，并初始化静态变量的初始值。但这里的“初始化”和下面的显式初始化阶段是有区别的，侧重点在于分配所需要的内存空间，不会去执行更进一步的 JVM 指令。准备阶段是正式为类变量分配内存并设置类变量初始值的阶段，这些变量所使用的内存都将在方法区中进行分配。

   + 解析（Resolution），在这一步会将常量池中的符号引用（symbolic reference）替换为直接引用。在Java 虚拟机规范中，详细介绍了类、接口、方法和字段等各个方面的解析。

3. 最后是**初始化**（initialization），这一步真正去执行类初始化的代码逻辑，包括静态字段赋值的动作，以及执行类定义中的静态初始化块内的逻辑，编译器在编译阶段就会把这部分逻辑整理好，父类型的初始化逻辑优先于当前类型的逻辑。注意初始化阶段只对静态成员和静态块执行

加载、验证、准备、初始化和卸载这五个阶段的顺序是确定，类的加载过程必须按这种顺序开始。而解析阶段不一定，可以在初始化后开始。注意这里的顺序指开始顺序。而不是说按顺序执行，这些阶段是**互相交叉**进行的。只要开始的顺序是上面的顺序即可。

#### 什么时候会触发/不触发初始化

对于初始化阶段，JVM规范规定有且只有5种情况必须立即对类进行初始化：

+ 遇到`new/putstatic/getstatic/invokestatic`。也就是在用new实例化对象，读取或设置一个类的静态字段时（被final修饰，已在编译期把结果放入常量池的静态字段除外），以及调用一个类的静态方法时
+ 使用java.lang.reflect包的方法对类进行反射调用，如Class.forName
+ 当初始化一个类时，若父类还未初始化，则先触发父类初始化
+ 当JVM启动时，用户指定一个要执行的主类，虚拟机会先初始化这个主类
+ 使用JDK 1.7动态语言支持时，若一个java.lang.invoke.MethodHandle实例最终的解析结果REF_getStatic、REF_putStatic、REF_invokestatic的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则先触发初始化

这5种情况会触发类的初始化，有且只有这5种情况，这5种情况称为对类进行主动引用。

除此之外，所有引用类的方式都不会触发初始化，称为被动引用。

被动引用的例子:

+ 子类引用父类的静态字段，只会导致父类的初始化，不会导致子类初始化。也就是说只有直接定义这个字段的类才会被初始化。
+ 在定义对象数组时，如`SubClass p[]=new SubClass[10]`，不会导致SubClass类初始化，触发的是`[L SubClass`的初始化。
+ 类中的定义是静态常量时`public static final String HE="Hello"`，这个常量在编译时就已经确定了，编译时已经存储到NotInitialization类的常量池中，所以`类.HE`实际是调用`NotInitialization.HE`。所以不会导致引用类的初始化。

注意：接口的加载过程与类基本一致，主要区别是：一个接口在初始化是并不要求其父接口全都完成了初始化，只有在真正用到父接口时才会初始化。而在初始化一个类时，要求其父类全部都已经初始化过了

### 类加载器有什么作用？有哪些类加载器

类加载器实现对类文件的加载，同时由加载它的类加载器和这个类本身一同确立其在Java虚拟机中的唯一性。

类加载器种类：

+ 启动类加载器（Bootstrap ClassLoader）：加载 jre/lib 或者被-Xbootclasspath参数所指定的路径中的下面的 jar 文件，如 rt.jar
+ 其他类加载器：全都继承自java.lang.ClassLoader类
  + 拓展类加载器（Extension ClassLoader）：由`sun.misc.Launcher$ExtClassLoader`实现，它负责加载`\lib\ext`目录中的，或者被java.ext.dirs系统变量所指定的路径中的所有类库，开发者可以直接使用扩展类加载器
  + 应用程序类加载器（Application ClassLoader）：这个类加载器由`sun.misc.Launcher$AppClassLoader`实现。由于这个类加载器是ClassLoader中的getSystemClassLoader()方法的返回值，所以一般也称它为系统类加载器。它负责加载用户类路径（ClassPath）上所指定的类库，开发者可以直接使用这个类加载器

![](F:\mycode\knowledgeArrangement\java开发\类加载器.png)

### 双亲委派模型：什么是双亲委派模型

双亲委派模型指的是类加载之间的层次关系，如上图。

双亲委派模型的要求：除了顶层的启动类加载器外，其余的类加载器都应当有自己的父类加载器。

双亲委派模型的工作过程：

1. 当前 ClassLoader 首先从自己已经加载的类中，查询是否此类已经加载，如果已经加载则直接返回原来已经加载的类
2. 当前 ClassLoader 的缓存中没有找到被加载的类的时候

   + 委托父类加载器去加载，父类加载器采用同样的策略，首先查看自己的缓存，然后委托父类的父类去加载，一直到 bootstrap ClassLoader
   + 当所有的父类加载器都没有加载的时候，再由当前的类加载器加载，并将其放入它自己的缓存中，以便下次有加载请求的时候直接返回

具体的过程在Classloader中的loadClass方法中

### 双亲委派模型的好处

+ 可以避免重复加载，当父亲已经加载了该类的时候，子类不需要再次加载
+ 安全性：避免用户自己编写的类动态替换 Java 的一些核心类，比如 String

### 双亲委派模型的破坏

双亲委派模型是jdk1.2才提出的！！

1. 继承ClassLoader类（jdk1.0出现该类）并重写loadClass()方法。因为虚拟机在进行类加载的时候会调用加载器的私有方法loadClassInternal()，这个方法会调用自己的loadClass()方法，也就破坏了双亲委派模型。
   因此在jdk1.2后推荐重写findClass()方法，这样子在loadClass()方法的逻辑里如果父类加载失败，则会调用自己的findClass()来完成加载。
2. 基础类需要调用用户代码时。因为基础的类一般由启动加载器加载，而用户代码则是由应用程序加载器加载。典型就是JNDI中的线程上下文加载器，父类加载器会请求子类加载器来完成类加载，也就是逆向使用类加载。
3. OSGI，OSGI环境下，类加载器不再是双亲委派模型中的树状结构，而是更复杂的网状结构。其很多类加载动作是交给平级的类加载器中进行。

### Tomcat为什么要破坏双亲委派模型

Tomcat可能会有以下需求

+ 部署多个应用程序，可能会使用相同第三方类库的不同版本，因此需要实现Java类库的互相隔离。
+ 多个应用程序相同版本的第三方库需要可以共享（考虑到方法区内存）。
+ tomcat也有自己的依赖，这应该和应用程序分开
+ JSP文件也会编译成class文件，jsp在运行后可能会修改，这要求web容器能在jsp修改后不用重启

如果使用默认的类加载机制

+ 无法加载相同库的不同版本，因为全限定名不同
+ jsp文件修改后，类名并没有变，加载器会取已经加载的类

增加的5种类加载器：

+ Catalina：加载tomcat启动所需的类
+ Shared:加载对Wen应用程序可见但对Tomcat不可见的类库
+ Common：加载对Tomcat内部类和所有Web应用程序都可见的其他类 所有应用共享,lib目录
+ Webapp：为部署在单个Tomcat实例中的每个Web应用程序创建一个类加载器 ，加载`WEB-INF/classes和WEB-INF/lib`的jar中的类
+ Jsper:每个Jsp文件对应的类加载器

加载顺序：

1. JVM 的 Bootstrap 类
2. Web 应用的 /WEB-INF/classes 类
3. Web 应用的 /WEB-INF/lib/*.jar 类，保证了不同的版本的相同库
4. System 类加载器的类
5. Common 类加载器的类

### 静态分派和动态分派

```java
Humnan man = new Man();
```

上述代码中，变量man拥有两个类型，一个静态类型Human，一个实际类型Man，静态类型在编译期间可知。

在编译阶段，Java编译器会根据参数的静态类型决定调用哪个重载版本.这种依赖静态类型来定位方法执行版本的分派动作是是静态分派

在重载中，函数匹配是照向上转型的顺序匹配。如果参数是基本类型，首先匹配一样的参数类型，然后向上转型(如char依次匹配char/int/long/float/double)，再次是封装类型，如Integer形参接受int,之后匹配 的就是封装类的父类(如Character实现了Serializable，所以如果形参为Serializable也能够匹配char类型的实参)，最后是可变长参数。

动态分派

在运行期间根据参数的实际类型确定方法执行版本的过程称为动态分派，动态分派和多态性中的重写（override）有着紧密的联系。