# nio入门概念

作者：qinguan
博客：[https://bugstack.cn](https://bugstack.cn)

> 故不积跬步，无以至千里；不积小流，无以成江海！🌻

前言

在NIO中，所有的数据都需要通过通道流的形式传输，数据不可以直接在通道处理，必须把数据读取到缓冲区Buffer或从Buffer写入到通道。

# 一、三大组件

## 1、Channel

**Channel**是一个用于处理数据的通道，可以异步地读写数据。

我们可以从 channel 将数据读入 buffer，也可以将 buffer 的数据写入 channel；相较于***stream***而言，要么是inputStream要么是outputStream。

它是Java NIO中最重要的通道的实现，有如下四种类型：

- FileChannel：从文件中读写数据。
- DatagramChannel：通过UDP读写网络中的数据。
- SocketChannel：进行网络通信（客户端/服务器端）。
- ServerSocketChannel：监听新进来的连接（服务器端）。

## 2、Buffer

**Buffer**是一个类似于数组的容器，可以保存多个类型相同的数据。它是NIO中数据交互的媒介，数据读、写都是通过Buffer实现的。

从结构上看，Buffer类似一个数组，它有一个 `limit`值表示缓冲区中数据的最大容量，一个 `position`值用于指定下一个可以被读取的缓冲区位置索引。

Buffer则用来缓冲读写数据，常见的 buffer 有：

* **`ByteBuffer（字节缓冲区）`**

  * **MappedByteBuffer**
  * **DirectByteBuffer**
  * **HeapByteBuffer**
* **ShortBuffer（短缓冲区）**
* **IntBuffer（整型缓冲区）**
* **LongBuffer（长缓冲区）**
* **FloatBuffer（浮点缓冲区）**
* **DoubleBuffer（双精度浮点缓冲区）**
* **CharBuffer（字符缓冲区）**

### 2.1 ByteBuffer

ByteBuffer 重要属性：

* **capacity** （缓冲区中可以存储的字节的最大数量）
* **position** （是下一个要被读或写的元素的索引）
* **limit**（缓冲区的读或写限制）

一开始：

![1700462824126](image/nio入门概念/1700462824126.png)

写模式下，position 是写入位置，limit 等于容量，下图表示写入了 4 个字节后的状态：

![1700463102363](image/nio入门概念/1700463102363.png)

flip() 动作发生后，position 切换为读取位置，limit 切换为读取限制：

![1700463161861](image/nio入门概念/1700463161861.png)

读取 4 个字节后，状态：

![1700463251446](image/nio入门概念/1700463251446.png)

clear 动作发生后，状态：

![1700463283367](image/nio入门概念/1700463283367.png)

compact 方法，是把未读完的部分向前压缩，然后切换至写模式：

![1700463307815](image/nio入门概念/1700463307815.png)

ByteBuffer 正确使用姿势：

1. 向 buffer 写入数据，例如调用 channel.read(buffer)
2. 调用 flip() 切换至 `读模式`
3. 从 buffer 读取数据，例如调用 buffer.get()
4. 调用 clear() 或 compact() 切换至 `写模式`
5. 重复1~4步骤

## 3、Selector

Selector 是Java NIO中的一个核心组件，也称为多路复用器，用于检查一个或多个NIO Channel（通道）的状态是否可读、可写。

这使得 `使用单个线程管理多个网络链接成为可能，避免了使用多个线程和进行线程上下文切换带来的开销。`

Selector 的作用就是配合一个线程来管理多个 channel，获取这些 channel 上发生的事件，这些 channel 工作在非阻塞模式下，不会让线程吊死在一个 channel 上。

`适合连接数特别多，但流量低的场景（low traffic）`

# 二、网络编程

## 2.1 阻塞与非阻塞

**阻塞**：在没有数据可读时，包括数据复制过程中，线程必须阻塞等待，不会占用 cpu，但线程相当于闲置

- 单线程下，阻塞方法之间相互影响，几乎不能正常工作，需要多线程支持
- 多线程下，有新的问题，体现在以下方面：
  - 32 位 jvm 一个线程 320k，64 位 jvm 一个线程 1024k，如果连接数过多，必然导致 OOM，并且线程太多，反而会因为频繁上下文切换导致性能降低
  - 可以采用线程池技术来减少线程数和线程上下文切换，但治标不治本，如果有很多连接建立，但长时间 inactive，会阻塞线程池中所有线程，因此不适合长连接，只适合短连接

**非阻塞**：在某个 Channel 没有可读事件时，线程不必阻塞，它可以去处理其它有可读事件的 Channel

- 非阻塞模式下，相关方法都会不会让线程暂停
- 非阻塞模式下，即使没有连接建立，和可读数据，线程仍然在不断运行，白白浪费了 cpu
- 数据复制过程中，线程实际还是阻塞的（AIO 改进的地方）

多路复用：单线程可以配合 Selector 完成对多个 Channel 可读写事件的监控，这称之为多路复用

- 多路复用仅针对网络 IO、普通文件 IO 没法利用多路复用
- 如果不用 Selector 的非阻塞模式，线程大部分时间都在做无用功，而 Selector 能够保证
  - 有可连接事件时才去连接
  - 有可读事件才去读取
  - 有可写事件才去写入（限于网络传输能力，Channel 未必时时可写，一旦 Channel 可写，会触发 Selector 的可写事件）
