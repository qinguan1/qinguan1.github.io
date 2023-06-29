# 一文讲透JVM垃圾回收

作者：qinguan
<br/>博客：[https://bugstack.cn](https://bugstack.cn)

> 故不积跬步，无以至千里；不积小流，无以成江海！🌻


既然要进行垃圾回收，那么第一步就要识别垃圾，第二步就是清理垃圾。

## 如何识别垃圾对象
### 引用计数法
引用计数法是一种内存管理技术，它的基本思想是跟踪每个对象被引用的次数，并在引用计数为零时自动释放对象所占用的内存空间。
无法解决循环引用问题
![引用计数法](https://raw.githubusercontent.com/qinguan1/qinguan1.github.io/main/docs/assets/img/qinguan/引用计数法.png)

### 可达性分析算法

可达性分析算法是一种垃圾回收算法，用于判断一个对象是否可达（即是否还有引用指向它），从而确定是否可以将该对象回收。
可达性分析算法的基本思想是从一组根对象（通常称为GC root）开始，递归遍历所有可能被引用的对象，将所有遍历到的对象标记为“已访问”，最终未被标记的对象即为垃圾对象，可以被回收。这种算法也是目前主流垃圾回收器所使用的算法。

![可达性分析算法](https://raw.githubusercontent.com/qinguan1/qinguan1.github.io/main/docs/assets/img/qinguan/可达性分析算法.png)

## 分代收集理论
分代收集理论是一种垃圾回收算法的理论基础，它将堆内存分为不同的年代，每个年代采用不同的垃圾回收策略。其中，新生代采用标记-复制算法，老年代采用标记-清除算法或标记-整理算法。
分代收集理论是一种整体的垃圾回收策略，具体的回收算法又分为好几种。

## 有哪些垃圾回收算法

### 标记-清除
标记-清除算法是一种垃圾回收算法，也是一种基于可达性分析的垃圾回收算法。
它的基本思想是在垃圾回收过程中，先标记出所有被引用的对象，然后清除所有未被标记的对象，从而释放它们占用的内存空间。
这个算法的主要缺点是它需要在垃圾回收过程中暂停程序的执行，因为在标记和清除的过程中，需要遍历整个堆内存，这会导致程序的执行速度变慢。另外，标记-清除算法还会产生内存碎片，这会影响程序的性能和内存利用率。
它的基本步骤有两个：
1. 标记阶段：标记可达对象
2. 清除阶段：清理未标记对象

![gc-标记清除](https://raw.githubusercontent.com/qinguan1/qinguan1.github.io/main/docs/assets/img/qinguan/gc-标记清除.png)

特点：`位置不连续，容易产生碎片`
### 标记-整理
标记整理算法是一种垃圾回收算法，主要用于`老年代`的垃圾回收。
它的基本思路是先标记出所有存活的对象，然后将这些对象向一端移动，然后清除掉另一端的所有垃圾对象。这样可以使得内存空间更加连续，从而提高内存的利用率。
标记整理算法主要适用于存活对象比较多、垃圾对象比较少的场景，因为移动对象需要较大的复制代价，如果存活对象太少，移动的代价会超过清除的代价，导致效率降低。
它的基本步骤有两个：
1. 标记阶段：它的第一个阶段与<kbd>标记清除算法</kbd>是一模一样的。
2. 整理阶段：移动所有`存活的对象`，且按照内存地址次序依次排列，然后将末端内存地址以后的内存全部回收。
![gc-标记整理](https://raw.githubusercontent.com/qinguan1/qinguan1.github.io/main/docs/assets/img/qinguan/gc-标记整理.png)

特点：`位置连续，没有碎片，效率偏低`
### 复制
标记复制算法是一种垃圾回收算法，主要用于`新生代`的垃圾回收。
它的基本思路是将新生代内存空间分为两个相等的区域，每次只使用其中一个区域，当这个区域满了之后，将其中存活的对象复制到另一个区域中，然后清空原来的区域。
这样可以保证内存空间的连续性，同时也减少了垃圾回收的复杂度。
标记复制算法主要适用于存活对象比较少、垃圾对象比较多的场景，因为复制的代价比较小，但是如果存活对象太多，复制的代价会变得很大，导致**效率降低**。
![gc-复制](https://raw.githubusercontent.com/qinguan1/qinguan1.github.io/main/docs/assets/img/qinguan/gc-复制.png)

特点：`位置连续，没有碎片，空间浪费严重`

## JVM堆模型  

![JVM堆模型](https://raw.githubusercontent.com/qinguan1/qinguan1.github.io/main/docs/assets/img/qinguan/JVM堆模型.png)

观察上面的堆模型我们不难看出

## 有哪些垃圾收集器

如果说收集算法是内存回收的方法论，那么垃圾收集器就是内存回收的具体实现。

### Serial收集器(-XX:+UseSerialGC -XX:+UseSerialOldGC)
Serial（串行）收集器是最基本、历史最悠久的垃圾收集器了。大家看名字就知道这个收集器是一个单线程收集器了。它的 `单线程`的意义不仅仅意味着它只会使用一条垃圾收集线程去完成垃圾收集工作，更重要的是它在进行垃圾收集工作的时候必须**暂停其他所有的工作线程**（ "Stop The World" ），直到它收集结束。
算法策略：`新生代采用复制算法，老年代采用标记-整理算法`
![Serial收集器](https://raw.githubusercontent.com/qinguan1/qinguan1.github.io/main/docs/assets/img/qinguan/Serial收集器.png)

### Parallel Scavenge收集器(-XX:+UseParallelGC(年轻代),-XX:+UseParallelOldGC(老年代))
Parallel收集器其实就是Serial收集器的多线程版本，除了使用多线程进行垃圾收集外，其余行为（控制参数、收集算法、回收策略等等）和Serial收集器类似。默认的收集线程数跟cpu核数相同，当然也可以用参数(-XX:ParallelGCThreads)指定收集线程数，但是一般不推荐修改。
Parallel Scavenge收集器关注点是吞吐量（高效率的利用CPU）。CMS等垃圾收集器的关注点更多的是用户线程的停顿时间（提高用户体验）。所谓吞吐量就是CPU中用于运行用户代码的时间与CPU总消耗时间的比值。 Parallel Scavenge收集器提供了很多参数供用户找到最合适的停顿时间或最大吞吐量，如果对于收集器运作不太了解的话，可以选择把内存管理优化交给虚拟机去完成也是一个不错的选择。
算法策略：`新生代采用复制算法，老年代采用标记-整理算法`
![Parallel Scavenge收集器](https://raw.githubusercontent.com/qinguan1/qinguan1.github.io/main/docs/assets/img/qinguan/Parallel Scavenge收集器.png)

Parallel Old收集器是Parallel Scavenge收集器的老年代版本。使用多线程和“标记-整理”算法。在注重吞吐量以及CPU资源的场合，都可以优先考虑 Parallel Scavenge收集器和Parallel Old收集器(JDK8默认的新生代和老年代收集器)。


### ParNew收集器(-XX:+UseParNewGC)
ParNew收集器其实跟Parallel收集器很类似，区别主要在于它可以和CMS收集器配合使用。
算法策略：`新生代采用复制算法，老年代采用标记-整理算法`

### CMS收集器(-XX:+UseConcMarkSweepGC(old))
CMS（Concurrent Mark Sweep）收集器是一种以获取`最短回收停顿时间`为目标的收集器。
它非常符合在注重用户体验的应用上使用，它是HotSpot虚拟机第一款真正意义上的`并发收集器`，第一次实现了让垃圾收集线程与用户线程（基本上）同时工作。
![CMS收集器](https://raw.githubusercontent.com/qinguan1/qinguan1.github.io/main/docs/assets/img/qinguan/CMS收集器.png)

CMS收集器是一种 “**标记-清除**”算法实现的，它的运作过程相比于前面几种垃圾收集器来说更加复杂一些。
整个过程分为四个步骤：
- `初始标记`： 暂停所有的其他线程(STW)，并记录下gc roots直接能引用的对象，速度很快。
- `并发标记`： 并发标记阶段就是从GC Roots的直接关联对象开始遍历整个对象图的过程， 这个过程耗时较长但是不需要停顿用户线程，可以与垃圾收集线程一起并发运行。因为用户程序继续运行，可能会有导致已经标记过的对象状态发生改变。
- `重新标记`： 重新标记阶段就是为了修正并发标记期间因为用户程序继续运行而导致标记产生变动的那一部分对象的标记记录(主要是处理漏标问题)，这个阶段的停顿时间一般会比初始标记阶段的时间稍长，远远比并发标记阶段时间短。主要用到**三色标记里的增量更新算法**(见下面详解)做重新标记。
- `并发清理`： 开启用户线程，同时GC线程开始对未标记的区域做清扫。这个阶段如果有新增对象会被标记为黑色不做任何处理(见下面三色标记算法详解)。
- `并发重置`：重置本次GC过程中的标记数据。

它是一款具有划时代意义的优秀垃圾收集器，主要优点：`并发收集、低停顿`。
但是它有下面几个明显的缺点：
- 对CPU资源敏感（会和服务抢资源）；
- 无法处理浮动垃圾(在并发标记和并发清理阶段又产生垃圾，这种浮动垃圾只能等到下一次gc再清理了)；
- 它使用的“**标记-清除**”算法会导致收集结束时会有大量空间碎片产生，当然通过参数**-XX:+UseCMSCompactAtFullCollection**可以让jvm在执行完标记清除后再做整理；
- 执行过程中的不确定性，会存在上一次垃圾回收还没执行完，然后垃圾回收又被触发的情况，特别是在并发标记和并发清理阶段会出现，一边回收，系统一边运行，也许没回收完就再次触发full gc，也就是"concurrent mode failure"，此时会进入stop the world，用serial old垃圾收集器来回收

CMS的相关核心参数：
**-XX:+UseConcMarkSweepGC**：启用cms 
**-XX:ConcGCThreads**：并发的GC线程数
**-XX:+UseCMSCompactAtFullCollection**：FullGC之后做压缩整理（减少碎片）
**-XX:CMSFullGCsBeforeCompaction**：多少次FullGC之后压缩一次，默认是0，代表每次FullGC后都会压缩一次  
**-XX:CMSInitiatingOccupancyFraction**: 当老年代使用达到该比例时会触发FullGC（默认是92，这是百分比）
**-XX:+UseCMSInitiatingOccupancyOnly**：只使用设定的回收阈值(-XX:CMSInitiatingOccupancyFraction设定的值)，如果不指定，JVM仅在第一次使用设定值，后续则会自动调整
**-XX:+CMSScavengeBeforeRemark**：在CMS GC前启动一次minor gc，降低CMS GC标记阶段(也会对年轻代一起做标记，如果在minor gc就干掉了很多对垃圾对象，标记阶段就会减少一些标记时间)时的开销，一般CMS的GC耗时 80%都在标记阶段
**-XX:+CMSParallellnitialMarkEnabled**：表示在初始标记的时候多线程执行，缩短STW
**-XX:+CMSParallelRemarkEnabled**：在重新标记的时候多线程执行，缩短STW;

### 常见的垃圾回收器的组合
![常见的垃圾回收器的组合](https://raw.githubusercontent.com/qinguan1/qinguan1.github.io/main/docs/assets/img/qinguan/常见的垃圾回收器的组合.png)