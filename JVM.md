### JVM

#### 1、运行时数据区

<img src="C:\Users\z\AppData\Roaming\Typora\typora-user-images\image-20210218094753987.png" alt="image-20210218094753987" style="zoom: 50%;" />

+ 程序计数器

  > 程序计数器是一块比较小的内存空间，它可以看作是当前线程所执行的字节码的行号指示器，字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令。每条线程都需要有一个独立的程序计数器，各条线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存

+ 虚拟机栈

  > 虚拟机栈属于线程私有的。虚拟机栈描述的是Java方法执行的线程内存模型：每个方法被执行的时候，Java虚拟机都会同步创建一个栈帧（Stack Frame）用于存储局部变量表、操作数栈、动态连接、方法出口等信息。每一个方法被调用直至执行完毕的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。
  >
  > 由于HotSpot虚拟机中并不区分虚拟机栈和本地方法栈，因此对于HotSpot来说，-Xoss参数（设置本地方法栈大小）虽然存在，但实际上是没有任何效果的，栈容量只能由-Xss参数来设定。

+ 本地方法栈

  > 本地方法栈和虚拟机栈作用很相似，区别就是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则是为虚拟机使用到的本地（Native）方法服务。

+ 堆

  > 堆（Java Heap）是虚拟机所管理的内存中最大的一块。是被所有线程共享的一块内存区域
  >
  > Java堆是垃圾收集器管理的内存区域，
  >
  > Java堆既可以被实现成固定大小的，也可以是可扩展的，不过当前主流的Java虚拟机都是按照可扩展来实现的（通过参数-Xmx和-Xms设定）。如果在Java堆中没有内存完成实例分配，并且堆也无法再扩展时，Java虚拟机将会抛出OutOfMemoryError异常

+ 方法区

  > 线程共享的内存区域，它用于存储已被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等数据
  >
  > 到了JDK 7的HotSpot，已经把原本放在永久代的字符串常量池、静态变量等移出，而到了JDK 8，终于完全废弃了永久代的概念，改用与JRockit、J9一样在本地内存中实现的元空间（Meta-space）来代替，把JDK 7中永久代还剩余的内容（主要是类型信息）全部移到元空间中。

  + 运行时常量池

    > 运行时常量池（Runtime Constant Pool）是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池表（Constant Pool Table），用于存放编译期生成的各种字面量与符号引用，这部分内容将在类加载后存放到方法区的运行时常量池中

####  2、直接内存

> 在JDK 1.4中新加入了NIO（New Input/Output）类，引入了一种基于通道（Channel）与缓冲区（Buffer）的I/O方式，它可以使用Native函数库直接分配堆外内存，然后通过一个存储在Java堆里面的DirectByteBuffer对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在Java堆和Native堆中来回复制数据

#### 3、GC

##### 1、可达性分析

<img src="C:\Users\z\AppData\Roaming\Typora\typora-user-images\image-20210218145619397.png" alt="image-20210218145619397" style="zoom: 67%;" />

	>通过一系列称为“GC Roots”的根对象作为起始节点集，从这些节点开始，根据引用关系向下搜索，搜索过程所走过的路径称为“引用链”（Reference Chain），如果某个对象到GC Roots间没有任何引用链相连，或者用图论的话来说就是从GC Roots到这个对象不可达时，则证明此对象是不可能再被使用的。

##### 2、GC Root

> + 在虚拟机栈（栈帧中的本地变量表）中引用的对象，譬如各个线程被调用的方法堆栈中使用到的参数、局部变量、临时变量等
> + 在方法区中常量引用的对象，譬如字符串常量池中的引用
> + 在方法区中类静态属性引用的对象，譬如java类的引用类型静态变量
> + 在本地方法栈中JNI（Native方法）引用的对象
> + java虚拟机内部的引用，如基本数据对应的class对象，一些常驻的异常对象（NullException等），还有系统类加载器
> + 所有被同步锁（synchronized关键字）持有的对象
> + 反映Java虚拟机内部情况的JMXBean、JVMTI中注册的回调、本地代码缓存等。

##### 3、垃圾收集算法

+ 标记-清除算法（Mark-Sweep）

  > 首先标记出所有需要回收的对象，在标记完成后，统一回收掉所有被标记的对象，也可以反过来，标记存活的对象，统一回收所有未被标记的对象。标记过程就是对象是否属于垃圾的判定过程
  >
  > 缺点：
  >
  > 执行效率不稳定，如果Java堆中包含大量对象，而且其中大部分是需要被回收的，这时必须进行大量标记和清除的动作，导致标记和清除两个过程的执行效率都随对象数量增长而降低；
  >
  > 内存空间的碎片化问题，标记、清除之后会产生大量不连续的内存碎片，空间碎片太多可能会导致当以后在程序运行过程中需要分配较大对象时无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作

![image-20210218152143090](C:\Users\z\AppData\Roaming\Typora\typora-user-images\image-20210218152143090.png)

+ 标记-复制算法

  > 为了解决标记-清除算法面对大量可回收对象时执行效率低的问题
  >
  > 将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉
  >
  > 缺点：空间浪费未免太多了

  ![image-20210218152357065](C:\Users\z\AppData\Roaming\Typora\typora-user-images\image-20210218152357065.png)

+ 标记-整理算法

  > 针对老年代对象的存亡特征，1974年Edward Lueders提出了另外一种有针对性的“标记-整理”（Mark-Compact）算法，其中的标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向内存空间一端移动，然后直接清理掉边界以外的内存;
  >
  > 标记-清除算法与标记-整理算法的本质差异在于前者是一种非移动式的回收算法，而后者是移动式的。是否移动回收后的存活对象是一项优缺点并存的风险决策

  ![image-20210218152600932](C:\Users\z\AppData\Roaming\Typora\typora-user-images\image-20210218152600932.png)



##### 4、垃圾回收器

<img src="C:\Users\z\AppData\Roaming\Typora\typora-user-images\image-20210218154041595.png" alt="image-20210218154041595" style="zoom:50%;" />

+ Serial收集器

  > 这个收集器是一个单线程工作的收集器，但它的“单线程”的意义并不仅仅是说明它只会使用一个处理器或一条收集线程去完成垃圾收集工作，更重要的是强调在它进行垃圾收集时，必须暂停其他所有工作线程，直到它收集结束。“Stop The World”
  >
  > ![image-20210218154304776](C:\Users\z\AppData\Roaming\Typora\typora-user-images\image-20210218154304776.png)
  >
  > Serial收集器对于运行在客户端模式下的虚拟机来说是一个很好的选择

+ ParNew收集器

  > ParNew收集器实质上是Serial收集器的多线程并行版本
  >
  > ![image-20210218154503610](C:\Users\z\AppData\Roaming\Typora\typora-user-images\image-20210218154503610.png)
  >
  > 除了Serial收集器外，目前只有它能与CMS收集器配合工作

+ Parallel Scavenge收集器

  > Parallel Scavenge收集器也是一款新生代收集器，它同样是基于标记-复制算法实现的收集器，也是能够并行收集的多线程收集器
  >
  > Parallel Scavenge收集器的特点是它的关注点与其他收集器不同，CMS等收集器的关注点是尽可能地缩短垃圾收集时用户线程的停顿时间，而Parallel Scavenge收集器的目标则是达到一个可控制的吞吐量（Throughput）。所谓吞吐量就是处理器用于运行用户代码的时间与处理器总消耗时间的比值;
  >
  > Parallel Scavenge收集器提供了两个参数用于精确控制吞吐量，分别是
  >
  > + 控制最大垃圾收集停顿时间的-XX：MaxGCPauseMillis参数
  >
  >   > 参数允许的值是一个大于0的毫秒数，收集器将尽力保证内存回收花费的时间不超过用户设定值
  >
  > + 直接设置吞吐量大小的-XX：GCTimeRatio参数
  >
  >   > 参数的值则应当是一个大于0小于100的整数，也就是垃圾收集时间占总时间的比率，相当于吞吐量的倒数
  >
  > + -XX：+UseAdaptiveSizePolicy  
  >
  >   > 这个参数被激活之后，就不需要人工指定新生代的大小（-Xmn）、Eden与Survivor区的比例（-XX：SurvivorRatio）、晋升老年代对象大小（-XX：PretenureSizeThreshold）等细节参数了，虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大的吞吐量

+ Serial Old收集器

  > Serial Old是Serial收集器的老年代版本，它同样是一个单线程收集器，使用标记-整理算法
  >
  > ![image-20210218155156445](C:\Users\z\AppData\Roaming\Typora\typora-user-images\image-20210218155156445.png)

+ Parallel Old收集器

  > Parallel Old是Parallel Scavenge收集器的老年代版本，支持多线程并发收集，基于标记-整理算法实现
  >
  > ![image-20210218155233569](C:\Users\z\AppData\Roaming\Typora\typora-user-images\image-20210218155233569.png)

+ CMS收集器

  > CMS（Concurrent Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器
  >
  > CMS收集器是基于标记-清除算法实现的
  >
  > 运作过程：
  >
  > + 初始标记（CMS initial mark）   
  >
  >   > 需要STW，初始标记仅仅只是标记一下GCRoots能直接关联到的对象，速度很快
  >
  > + 并发标记（CMS concurrent mark）
  >
  >   > 并发标记阶段就是从GC Roots的直接关联对象开始遍历整个对象图的过程，这个过程耗时较长但是不需要停顿用户线程，可以与垃圾收集线程一起并发运行
  >
  > + 重新标记（CMS remark）        
  >
  >   > 需要STW,为了修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间通常会比初始标记阶段稍长一些，但也远比并发标记阶段的时间短
  >
  > + 并发清除（CMS concurrent sweep）
  >
  >   > 清理删除掉标记阶段判断的已经死亡的对象，由于不需要移动存活对象，所以这个阶段也是可以与用户线程同时并发的
  >
  > ![image-20210218155528936](C:\Users\z\AppData\Roaming\Typora\typora-user-images\image-20210218155528936.png)

+ Garbage First收集器 (G1)

  > JDK 9下默认收集器就是G1
  >
  > G1垃圾收集器技术发展历史上的里程碑式的成果，它开创了收集器面向局部收集的设计思路和基于Region的内存布局形式
  >
  > 它可以面向堆内存任何部分来组成回收集（Collection Set，一般简称CSet）进行回收，衡量标准不再是它属于哪个分代，而是哪块内存中存放的垃圾数量最多，回收收益最大，这就是G1收集器的Mixed GC模式
  >
  > G1开创的基于Region的堆内存布局是它能够实现这个目标的关键。虽然G1也仍是遵循分代收集理论设计的，但其堆内存的布局与其他收集器有非常明显的差异：G1不再坚持固定大小以及固定数量的分代区域划分，而是把连续的Java堆划分为多个大小相等的独立区域（Region），每一个Region都可以根据需要，扮演新生代的Eden空间、Survivor空间，或者老年代空间。收集器能够对扮演不同角色的Region采用不同的策略去处理，这样无论是新创建的对象还是已经存活了一段时间、熬过多次收集的旧对象都能获取很好的收集效果。Region中还有一类特殊的Humongous区域，专门用来存储大对象。G1认为只要大小超过了一个Region容量一半的对象即可判定为大对象。每个Region的大小可以通过参数-XX：G1HeapRegionSize设定，取值范围为1MB～32MB，且应为2的N次幂。而对于那些超过了整个Region容量的超级大对象，将会被存放在N个连续的Humongous Region之中，G1的大多数行为都把Humongous Region作为老年代的一部分来进行看待.
  >
  > 虽然G1仍然保留新生代和老年代的概念，但新生代和老年代不再是固定的了，它们都是一系列区域（不需要连续）的动态集合。G1收集器之所以能建立可预测的停顿时间模型，是因为它将Region作为单次回收的最小单元，即每次收集到的内存空间都是Region大小的整数倍，这样可以有计划地避免在整个Java堆中进行全区域的垃圾收集。更具体的处理思路是让G1收集器去跟踪各个Region里面的垃圾堆积的“价值”大小，价值即回收所获得的空间大小以及回收所需时间的经验值，然后在后台维护一个优先级列表，每次根据用户设定允许的收集停顿时间（使用参数-XX：MaxGCPauseMillis指定，默认值是200毫秒），优先处理回收价值收益最大的那些Region，这也就是“Garbage First”名字的由来。这种使用Region划分内存空间，以及具有优先级的区域回收方式，保证了G1收集器在有限的时间内获取尽可能高的收集效率
  >
  > ![image-20210218160115139](C:\Users\z\AppData\Roaming\Typora\typora-user-images\image-20210218160115139.png)

+ Shenandoah收集器

  > Shenandoah是一款只有OpenJDK才会包含，而OracleJDK里反而不存在的收集器
  >
  > 工作过程（9个阶段）
  >
  > + 初始标记
  >
  >   > 与G1一样，首先标记与GC Roots直接关联的对象，这个阶段仍是“Stop The World”的，但停顿时间与堆大小无关，只与GC Roots的数量相关
  >
  > + 并发标记
  >
  >   > 与G1一样，遍历对象图，标记出全部可达的对象，这个阶段是与用户线程一起并发的，时间长短取决于堆中存活对象的数量以及对象图的结构复杂程度
  >
  > + 最终标记
  >
  >   > 与G1一样，处理剩余的SATB扫描，并在这个阶段统计出回收价值最高的Region，将这些Region构成一组回收集（Collection Set）。最终标记阶段也会有一小段短暂的停顿。
  >
  > + 并发清理
  >
  >   > 个阶段用于清理那些整个区域内连一个存活对象都没有找到的Region（这类Region被称为Immediate Garbage Region）
  >
  > + 并发回收
  >
  >   > 并发回收阶段是Shenandoah与之前HotSpot中其他收集器的核心差异。在这个阶段，Shenandoah要把回收集里面的存活对象先复制一份到其他未被使用的Region之中。复制对象这件事情如果将用户线程冻结起来再做那是相当简单的，但如果两者必须要同时并发进行的话，就变得复杂起来了。其困难点是在移动对象的同时，用户线程仍然可能不停对被移动的对象进行读写访问，移动对象是一次性的行为，但移动之后整个内存中所有指向该对象的引用都还是旧对象的地址，这是很难一瞬间全部改变过来的。对于并发回收阶段遇到的这些困难，Shenandoah将会通过读屏障和被称为“Brooks Pointers”的转发指针来解决（讲解完Shenandoah整个工作过程之后笔者还要再回头介绍它）。并发回收阶段运行的时间长短取决于回收集的大小
  >
  > + 初始引用更新
  >
  >   > 并发回收阶段复制对象结束后，还需要把堆中所有指向旧对象的引用修正到复制后的新地址，这个操作称为引用更新。引用更新的初始化阶段实际上并未做什么具体的处理，设立这个阶段只是为了建立一个线程集合点，确保所有并发回收阶段中进行的收集器线程都已完成分配给它们的对象移动任务而已。初始引用更新时间很短，会产生一个非常短暂的停顿。
  >
  > + 并发引用更新
  >
  >   > 真正开始进行引用更新操作，这个阶段是与用户线程一起并发的，时间长短取决于内存中涉及的引用数量的多少。并发引用更新与并发标记不同，它不再需要沿着对象图来搜索，只需要按照内存物理地址的顺序，线性地搜索出引用类型，把旧值改为新值即可。
  >
  > + 最终引用更新
  >
  >   > 解决了堆中的引用更新后，还要修正存在于GC Roots中的引用。这个阶段是Shenandoah的最后一次停顿，停顿时间只与GC Roots的数量相关
  >
  > + 并发清理
  >
  >   > 经过并发回收和引用更新之后，整个回收集中所有的Region已再无存活对象，这些Region都变成Immediate Garbage Regions了，最后再调用一次并发清理过程来回收这些Region的内存空间，供以后新对象分配使用
  >
  > ![image-20210218160631142](C:\Users\z\AppData\Roaming\Typora\typora-user-images\image-20210218160631142.png)

+ ZGC收集器

  > JDK 11中新加入的具有实验性质的低延迟垃圾收集器，是由Oracle公司研发的
  >
  > ZGC收集器是一款基于Region内存布局的，（暂时）不设分代的，使用了读屏障、染色指针和内存多重映射等技术来实现可并发的标记-整理算法的，以低延迟为首要目标的一款垃圾收集器
  >
  > ZGC收集器有一个标志性的设计是它采用的染色指针技术（Colored Pointer，其他类似的技术中可能将它称为Tag Pointer或者Version Pointer）
  >
  > ZGC的运作过程大致可划分为以下四个大的阶段。全部四个阶段都是可以并发执行的，仅是两个阶段中间会存在短暂的停顿小阶段
  >
  > + 并发标记（Concurrent Mark）：与G1、Shenandoah一样，并发标记是遍历对象图做可达性分析的阶段，前后也要经过类似于G1、Shenandoah的初始标记、最终标记（尽管ZGC中的名字不叫这些）的短暂停顿，而且这些停顿阶段所做的事情在目标上也是相类似的。与G1、Shenandoah不同的是，ZGC的标记是在指针上而不是在对象上进行的，标记阶段会更新染色指针中的Marked 0、Marked 1标志位。
  > + 并发预备重分配（Concurrent Prepare for Relocate）：这个阶段需要根据特定的查询条件统计得出本次收集过程要清理哪些Region，将这些Region组成重分配集（Relocation Set）。重分配集与G1收集器的回收集（Collection Set）还是有区别的，ZGC划分Region的目的并非为了像G1那样做收益优先的增量回收。相反，ZGC每次回收都会扫描所有的Region，用范围更大的扫描成本换取省去G1中记忆集的维护成本。因此，ZGC的重分配集只是决定了里面的存活对象会被重新复制到其他的Region中，里面的Region会被释放，而并不能说回收行为就只是针对这个集合里面的Region进行，因为标记过程是针对全堆的。此外，在JDK 12的ZGC中开始支持的类卸载以及弱引用的处理，也是在这个阶段中完成的。
  > + 并发重分配（Concurrent Relocate）：重分配是ZGC执行过程中的核心阶段，这个过程要把重分配集中的存活对象复制到新的Region上，并为重分配集中的每个Region维护一个转发表（ForwardTable），记录从旧对象到新对象的转向关系。得益于染色指针的支持，ZGC收集器能仅从引用上就明确得知一个对象是否处于重分配集之中，如果用户线程此时并发访问了位于重分配集中的对象，这次访问将会被预置的内存屏障所截获，然后立即根据Region上的转发表记录将访问转发到新复制的对象上，并同时修正更新该引用的值，使其直接指向新对象，ZGC将这种行为称为指针的“自愈”（Self-Healing）能力。这样做的好处是只有第一次访问旧对象会陷入转发，也就是只慢一次，对比Shenandoah的Brooks转发指针，那是每次对象访问都必须付出的固定开销，简单地说就是每次都慢，因此ZGC对用户程序的运行时负载要比Shenandoah来得更低一些。还有另外一个直接的好处是由于染色指针的存在，一旦重分配集中某个Region的存活对象都复制完毕后，这个Region就可以立即释放用于新对象的分配（但是转发表还得留着不能释放掉），哪怕堆中还有很多指向这个对象的未更新指针也没有关系，这些旧指针一旦被使用，它们都是可以自愈的。
  > + 并发重映射（Concurrent Remap）：重映射所做的就是修正整个堆中指向重分配集中旧对象的所有引用，这一点从目标角度看是与Shenandoah并发引用更新阶段一样的，但是ZGC的并发重映射并不是一个必须要“迫切”去完成的任务，因为前面说过，即使是旧引用，它也是可以自愈的，最多只是第一次使用时多一次转发和修正操作。重映射清理这些旧引用的主要目的是为了不变慢（还有清理结束后可以释放转发表这样的附带收益），所以说这并不是很“迫切”。因此，ZGC很巧妙地把并发重映射阶段要做的工作，合并到了下一次垃圾收集循环中的并发标记阶段里去完成，反正它们都是要遍历所有对象的，这样合并就节省了一次遍历对象图的开销。一旦所有指针都被修正之后，原来记录新旧对象关系的转发表就可以释放掉了。

##### 5、虚拟机及垃圾收集器日志

+ 如何查看GC信息

  > JDK9之前 -XX：+PrintGC       
  >
  > JDK9之后 -Xlog：gc：      eg:  java -Xlog:gc GCTest

+ 如何查看GC详细信息

  > 在JDK 9之前使用-XX：+PrintGCDetails
  >
  > 在JDK 9之后使用-X-log：gc*   eg:  java -Xlog:gc* GCTest

+ 查看GC前后的堆、方法区可用容量变化

  > JDK 9之前使用-XX：+PrintHeapAtGC
  >
  > JDK 9之后使用-Xlog：gc+heap=debug：

+ 查看GC过程中用户线程并发时间以及停顿的时间

  > JDK 9之前使用-XX：+Print-GCApplicationConcurrentTime以及-XX：+PrintGCApplicationStoppedTime
  >
  > JDK 9之后使用-Xlog：safepoint：

+ 查看收集器Ergonomics机制（自动设置堆空间各分代区域大小、收集目标等内容，从Parallel收集器开始支持）自动调节的相关信息

  > JDK 9之前使用-XX：+PrintAdaptive-SizePolicy
  >
  > JDK 9之后使用-Xlog：gc+ergo*=trace：

+ 查看熬过收集后剩余对象的年龄分布信息

  > JDK 9前使用-XX：+PrintTenuring-Distribution
  >
  > JDK 9之后使用-Xlog：gc+age=trace：

##### 6、内存分配与回收策略

+ 对象优先在Eden分配

  > -XX：Survivor-Ratio=8   决定了新生代中Eden区与一个Survivor区的空间比例是8∶1

+ 大对象直接进入老年代

  > 大对象就是指需要大量连续内存空间的Java对象，最典型的大对象便是那种很长的字符串，或者元素数量很庞大的数组
  >
  > -XX：PretenureSizeThreshold参数，指定大于该设置值的对象直接在老年代分配，这样做的目的就是避免在Eden区及两个Survivor区之间来回复制，产生大量的内存复制操作
  >
  > 注意：-XX：PretenureSizeThreshold参数只对Serial和ParNew两款新生代收集器有效，HotSpot的其他新生代收集器，如Parallel Scavenge并不支持这个参数。如果必须使用此参数进行调优，可考虑ParNew加CMS的收集器组合

+ 长期存活的对象将进入老年代

  > 当它的年龄增加到一定程度（默认为15），就会被晋升到老年代中。对象晋升老年代的年龄阈值，可以通过参数
  >
  > -XX：MaxTenuringThreshold  设置老年代的年龄阈值。

+ 动态对象年龄判定

  > HotSpot虚拟机并不是永远要求对象的年龄必须达到-XX：MaxTenuringThreshold才能晋升老年代，如果在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，无须等到-XX：MaxTenuringThreshold中要求的年龄。

+ 空间分配担保

  > 在发生Minor GC之前，虚拟机必须先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果这个条件成立，那这一次Minor GC可以确保是安全的。如果不成立，则虚拟机会先查看-XX：HandlePromotionFailure参数的设置值是否允许担保失败（Handle Promotion Failure）；如果允许，那会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试进行一次Minor GC，尽管这次Minor GC是有风险的；如果小于，或者-XX：HandlePromotionFailure设置不允许冒险，那这时就要改为进行一次Full GC

#### 4、（实战）虚拟机性能监控、故障处理工具

##### 1、基础故障处理工具

+  jps：虚拟机进程状况工具(JVM Process Status Tool)

  > 可以列出正在运行的虚拟机进程，并显示虚拟机执行主类（Main Class，main()函数所在的类）名称以及这些进程的本地虚拟机唯一ID（LVMID，Local Virtual Machine Identifier）
  >
  > ![image-20210218163212073](C:\Users\z\AppData\Roaming\Typora\typora-user-images\image-20210218163212073.png)

+  jstat：虚拟机统计信息监视工具(JVM Statistics Monitoring Tool)

  > 用于监视虚拟机各种运行状态信息的命令行工具。它可以显示本地或者远程[插图]虚拟机进程中的类加载、内存、垃圾收集、即时编译等运行时数据，在没有GUI图形界面、只提供了纯文本控制台环境的服务器上，它将是运行期定位虚拟机性能问题的常用工具。
  >
  > ![image-20210218163332010](C:\Users\z\AppData\Roaming\Typora\typora-user-images\image-20210218163332010.png)

+  jinfo：Java配置信息工具(Configuration Info for Java)

  > 实时查看和调整虚拟机各项参数
  >
  > jinfo的-flag选项 查询未被显式指定的参数的系统默认值
  >
  > jinfo的-sysprops选项把虚拟机进程的System.getProperties()的内容打印出来

+ jmap：Java内存映像工具（Memory Map for Java）

  > 用于生成堆转储快照（一般称为heapdump或dump文件）
  >
  > 如果不使用jmap命令，要想获取Java堆转储快照也还有一些比较“暴力”的手段：譬如-XX：+HeapDumpOnOutOfMemoryError参数，可以让虚拟机在内存溢出异常出现之后自动生成堆转储快照文件，通过-XX：+HeapDumpOnCtrlBreak参数则可以使用[Ctrl]+[Break]键让虚拟机生成堆转储快照文件，又或者在Linux系统下通过Kill-3命令发送进程退出信号“恐吓”一下虚拟机，也能顺利拿到堆转储快照
  >
  > jmap的作用并不仅仅是为了获取堆转储快照，它还可以查询finalize执行队列、Java堆和方法区的详细信息，如空间使用率、当前用的是哪种收集器等
  >
  > ![image-20210218163627560](C:\Users\z\AppData\Roaming\Typora\typora-user-images\image-20210218163627560.png)

+  jhat：虚拟机堆转储快照分析工具(JVM Heap Analysis Tool)

  > 与jmap搭配使用，来分析jmap生成的堆转储快照
  >
  > jhat内置了一个微型的HTTP/Web服务器，生成堆转储快照的分析结果后，可以在浏览器中查看
  >
  > 一般很少使用

+ jstack：Java堆栈跟踪工具（Stack Trace for Java）

  > 用于生成虚拟机当前时刻的线程快照（一般称为threaddump或者javacore文件）
  >
  > 线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的目的通常是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间挂起等，都是导致线程长时间停顿的常见原因。线程出现停顿时通过jstack来查看各个线程的调用堆栈，就可以获知没有响应的线程到底在后台做些什么事情，或者等待着什么资源
  >
  > ![image-20210218163814376](C:\Users\z\AppData\Roaming\Typora\typora-user-images\image-20210218163814376.png)
  >
  > 从JDK 5起，java.lang.Thread类新增了一个getAllStackTraces()方法用于获取虚拟机中所有线程的StackTraceElement对象。使用这个方法可以通过简单的几行代码完成jstack的大部分功能，在实际项目中不妨调用这个方法做个管理员页面，可以随时使用浏览器来查看线程堆栈

##### 2、可视化故障处理工具

+ JHSDB：基于服务性代理的调试工具

  > JHSDB是一款基于服务性代理（Serviceability Agent，SA）实现的进程外调试工具。服务性代理是HotSpot虚拟机中一组用于映射Java虚拟机运行信息的、主要基于Java语言（含少量JNI代码）实现的API集合
  >
  > 使用命令进入JHSDB的图形化模式，并使其附加进程11180：
  >
  > jdsdb hsdb --pid 11180

+ JConsole：Java监视与管理控制台

  > JConsole（Java Monitoring and Management Console）是一款基于JMX（Java Manage-mentExtensions）的可视化监视、管理工具。它的主要功能是通过JMX的MBean（Managed Bean）对系统进行信息收集和参数动态调整。JMX是一种开放性的技术，不仅可以用在虚拟机本身的管理上，还可以运行于虚拟机之上的软件中，典型的如中间件大多也基于JMX来实现管理与监控。虚拟机对JMX MBean的访问也是完全开放的，可以使用代码调用API、支持JMX协议的管理控制台，或者其他符合JMX规范的软件进行访问
  >
  > 通过JDK/bin目录下的jconsole.exe启动JCon-sole后，会自动搜索出本机运行的所有虚拟机进程，而不需要用户自己使用jps来查询
  >
  > JMX支持跨服务器的管理，也可以使用下面的“远程进程”功能来连接远程服务器，对远程虚拟机进行监控

+ VisualVM：多合-故障处理工具

  > VisualVM（All-in-One Java Troubleshooting Tool）是功能最强大的运行监视和故障处理程序之一，曾经在很长一段时间内是Oracle官方主力发展的虚拟机故障处理工具。Oracle曾在VisualVM的软件说明中写上了“All-in-One”的字样，预示着它除了常规的运行监视、故障处理外，还将提供其他方面的能力，譬如性能分析（Profiling）。VisualVM的性能分析功能比起JProfiler、YourKit等专业且收费的Profiling工具都不遑多让。而且相比这些第三方工具，VisualVM还有一个很大的优点：不需要被监视的程序基于特殊Agent去运行，因此它的通用性很强，对应用程序实际性能的影响也较小，使得它可以直接应用在生产环境中。这个优点是JProfiler、YourKit等工具无法与之媲美的