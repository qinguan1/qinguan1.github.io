# nio入门概念

作者：qinguan
`<br/>`博客：[https://bugstack.cn](https://bugstack.cn)

> 故不积跬步，无以至千里；不积小流，无以成江海！🌻

前言

在NIO中，所有的数据都需要通过通道流的形式传输，数据不可以直接在通道处理，必须把数据读取到缓冲区Buffer或从Buffer写入到通道。

# 一、三大组件

## 1、Channel & Buffer

### 1.1 Channel

**Channel**是一个用于处理数据的通道，可以异步地读写数据。

我们可以从 channel 将数据读入 buffer，也可以将 buffer 的数据写入 channel；相较于***stream***而言，要么是inputStream要么是outputStream。

它是Java NIO中最重要的通道的实现，有如下四种类型：

1. FileChannel：从文件中读写数据。
2. DatagramChannel：通过UDP读写网络中的数据。
3. SocketChannel：进行网络通信（客户端/服务器端）。
4. ServerSocketChannel：监听新进来的连接（服务器端）。

### 1.2 Buffer

**Buffer**是一个类似于数组的容器，可以保存多个类型相同的数据。它是NIO中数据交互的媒介，数据读、写都是通过Buffer实现的。

从结构上看，Buffer类似一个数组，它有一个 `limit`值表示缓冲区中数据的最大容量，一个 `position`值用于指定下一个可以被读取的缓冲区位置索引。
