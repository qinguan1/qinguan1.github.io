# nio入门概念

作者：qinguan
`<br/>`博客：[https://bugstack.cn](https://bugstack.cn)

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

## 3、Selector

Selector 是Java NIO中的一个核心组件，也称为多路复用器，用于检查一个或多个NIO Channel（通道）的状态是否可读、可写。

这使得 `使用单个线程管理多个网络链接成为可能，避免了使用多个线程和进行线程上下文切换带来的开销。`
