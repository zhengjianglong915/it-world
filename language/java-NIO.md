# Java NIO
## 一. Java NIO 概述
### 流与块的比较
原来的 I/O 库(在 java.io.*中) 与 NIO 最重要的区别是数据打包和传输的方式。正如前面提到的，原来的 I/O 以流的方式处理数据，而 NIO 以块的方式处理数据。

面向流 的 I/O 系统一次一个字节地处理数据。一个输入流产生一个字节的数据，一个输出流消费一个字节的数据。为流式数据创建过滤器非常容易。链接几个过滤器，以便每个过滤器只负责单个复杂处理机制的一部分，这样也是相对简单的。不利的一面是，面向流的 I/O 通常相当慢。

一个 面向块 的 I/O 系统以块的形式处理数据。每一个操作都在一步中产生或者消费一个数据块。按块处理数据比按(流式的)字节处理数据要快得多。但是面向块的 I/O 缺少一些面向流的 I/O 所具有的优雅性和简单性。

### 组成
Java NIO 由以下几个核心部分组成：

 - Buffers
 - Channels
 - Selectors

### 1.1 Channel

Channel是一个对象，可以通过它读取和写入数据。拿NIO与原来的I/O做个比较，通道就像是流。Java NIO的通道类似流，但又有些不同：

 - 既可以从通道中读取数据，又可以写数据到通道。但流的读写通常是单向的。
 - 通道可以异步地读写。
 - 通道中的数据总是要先读到一个Buffer，或者总是要从一个Buffer中写入。

从通道读取数据到缓冲区，从缓冲区写入数据到通道，如图所示：

<img src="media/15256598490613.jpg" width="230px">

下面是JAVA NIO中的一些主要Channel的实现：

 - **FileChannel**： 从文件中读写数据。
 - **DatagramChannel**： 能通过UDP读写网络中的数据。
 - **SocketChannel**： 能通过TCP读写网络中的数据。
 - **ServerSocketChannel**：可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel。

这些通道涵盖了UDP 和 TCP 网络IO，以及文件IO.


参考： 

 - http://ifeve.com/overview/
 - [NIO 入门](https://www.ibm.com/developerworks/cn/education/java/j-nio/j-nio.html)

### 1.1 Buffer

Buffer 是一个对象， 它包含一些要写入或者刚读出的数据。 在 NIO 中加入 Buffer 对象，体现了新库与原 I/O 的一个重要区别。在面向流的 I/O 中，您将数据直接写入或者将数据直接读到 Stream 对象中。

在 NIO 库中，所有数据都是用缓冲区处理的。在读取数据时，它是直接读到缓冲区中的。在写入数据时，它是写入到缓冲区中的。任何时候访问 NIO 中的数据，您都是将它放到缓冲区中。

缓冲区实质上是一个数组。通常它是一个字节数组，但是也可以使用其他种类的数组。但是一个缓冲区不 仅仅 是一个数组。缓冲区提供了对数据的结构化访问，而且还可以跟踪系统的读/写进程。

以下是Java NIO里关键的Buffer实现：

 - ByteBuffer
 - CharBuffer
 - DoubleBuffer
 - FloatBuffer
 - IntBuffer
 - LongBuffer
 - ShortBuffer

这些Buffer覆盖了你能通过IO发送的基本数据类型：byte, short, int, long, float, double 和 char。

### 1.3 Selector
Selector允许单线程处理多个 Channel。如果你的应用打开了多个连接（通道），但每个连接的流量都很低，使用Selector就会很方便。例如，在一个聊天服务器中。这是在一个单线程中使用一个Selector处理3个Channel的图示：

<img src="media/15256855348164.jpg" width="330px">

## 二. 实践
读和写是 I/O 的基本过程。从一个通道中读取很简单：只需创建一个缓冲区，然后让通道将数据读到这个缓冲区中。写入也相当简单：创建一个缓冲区，用数据填充它，然后让通道用这些数据来执行写入操作。


### 从文件中读取
如果使用原来的 I/O，那么我们只需创建一个 FileInputStream 并从它那里读取。而在 NIO 中，情况稍有不同：我们首先从 FileInputStream 获取一个 Channel 对象，然后使用这个通道来读取数据。

在 NIO 系统中，任何时候执行一个读操作，您都是从通道中读取，但是您不是 直接 从通道读取。因为所有数据最终都驻留在缓冲区中，所以您是从通道读到缓冲区中。

因此读取文件涉及三个步骤：

 - (1) 从 FileInputStream 获取 Channel.
 - (2) 创建 Buffer
 - (3) 将数据从 Channel 读到 Buffer 中。


```
public void read(String path) throws IOException{
   // FileOutputStream 也可以
   RandomAccessFile file = new RandomAccessFile(getClass().getClassLoader().getResource(path).getPath(), "rw");

   // (1) 从 FileInputStream 获取 Channel
   FileChannel fileChannel = file.getChannel();

   // (2) 创建 Buffer
   ByteBuffer buffer = ByteBuffer.allocate(48);

   // (3) 将数据从 Channel 读到 Buffer 中
   int bytesRead = fileChannel.read(buffer);
   System.out.println(bytesRead);
   while (bytesRead != -1) {
       System.out.println("Read " + bytesRead);
       buffer.flip();

       while (buffer.hasRemaining()) {
           System.out.println((char) buffer.get());
       }

       buffer.clear();
       bytesRead = fileChannel.read(buffer);
   }

   file.close();
}
```

### 文件写入

具体步骤:
 
 - 首先从 FileOutputStream 获取一个通道
 - 创建一个缓冲区并在其中放入一些数据 
 - 最后一步是写入缓冲区中


```
private void write(String path, byte[] content) throws IOException {
   // 1. 获取通道
   FileOutputStream out = new FileOutputStream(path);
   FileChannel channel = out.getChannel();

   // 2. 创建buffer, 并存入数据
   ByteBuffer buffer = ByteBuffer.allocate(1024);
   for (int i = 0; i < content.length; ++i) {
       buffer.put(content[i]);
   }
   buffer.flip();  // 关键

   // 3. 写入缓冲区
   channel.write(buffer);

   out.close();
}
```

### 读写结合


