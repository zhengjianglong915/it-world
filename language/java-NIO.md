# Java NIO
## 一. Java NIO 概述
### 流与块的比较
原来的 I/O 库(在 java.io.*中) 与 NIO 最重要的区别是数据打包和传输的方式。正如前面提到的，原来的 I/O 以流的方式处理数据，而 NIO 以块的方式处理数据。

面向流 的 I/O 系统一次一个字节地处理数据。一个输入流产生一个字节的数据，一个输出流消费一个字节的数据。为流式数据创建过滤器非常容易。链接几个过滤器，以便每个过滤器只负责单个复杂处理机制的一部分，这样也是相对简单的。不利的一面是面向流的 I/O 通常相当慢。

一个 面向块 的 I/O 系统以块的形式处理数据。每一个操作都在一步中产生或者消费一个数据块。按块处理数据比按(流式的)字节处理数据要快得多。但是面向块的 I/O 缺少一些面向流的 I/O 所具有的优雅性和简单性。

### 组成
Java NIO 由以下几个核心部分组成：

 - **Buffers**：它包含一些要写入或者刚读出的数据。可以理解为一个数组，作为数据缓存。
 - **Channels**：是一个对象，可以通过它读取和写入数据。拿NIO与原来的I/O做个比较，通道就像是流。
 - **Selectors**：Selector允许单线程处理多个 Channel。

## 二. 各组成部分
### 2.1 Channel

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

这些通道涵盖了UDP 和 TCP 网络IO，以及文件IO. **在从通道读取数据时，数据被放入到缓冲区。在有些情况下，可以将这个缓冲区直接写入另一个通道**, kafka就是利用这种通道之间copy的方式提高性能。


参考： 

 - http://ifeve.com/overview/
 - [NIO 入门](https://www.ibm.com/developerworks/cn/education/java/j-nio/j-nio.html)

### 1.1 Buffer

 - [Java NIO系列教程（三） Buffer](http://ifeve.com/tag/nio/)

Buffer 是一个对象， 它包含一些要写入或者刚读出的数据。 在 NIO 中加入 Buffer 对象，体现了新库与原 I/O 的一个重要区别。在面向流的 I/O 中，您将数据直接写入或者将数据直接读到 Stream 对象中。

在 NIO 库中，所有数据都是用缓冲区处理的。在读取数据时，它是直接读到缓冲区中的。在写入数据时，它是写入到缓冲区中的。任何时候访问 NIO 中的数据，您都是将它放到缓冲区中。

缓冲区实质上是一个数组。通常它是一个字节数组，但是也可以使用其他种类的数组。但是一个缓冲区不 仅仅 是一个数组。缓冲区提供了对数据的结构化访问，而且还可以跟踪系统的读/写进程。

以下是Java NIO里关键的Buffer实现：

 - **ByteBuffer**
 - **CharBuffer**
 - **DoubleBuffer**
 - **FloatBuffer**
 - **IntBuffer**
 - **LongBuffer**
 - **ShortBuffer**

这些Buffer覆盖了你能通过IO发送的基本数据类型：byte, short, int, long, float, double 和 char。

使用Buffer读写数据一般遵循以下四个步骤：

 - 写入数据到Buffer
 - 调用flip()方法. 用于设置读的位置和limit位置。
 - 从Buffer中读取数据
 - 调用clear()方法或者compact()方法。 clear进行重置状态变量。

#### 状态变量
可以用三个值指定缓冲区在任意时刻的状态：

 - **position**: 从通道读取时, position变量跟踪**已经写了多少数据**(写到缓冲区多少数据)。在写入通道时，您是从缓冲区中获取数据。position值跟踪从缓冲区中**获取了多少数据**。
 - **limit**：表明还有多少数据需要取出(在从缓冲区写入通道时)，或者还有多少空间可以放入数据(在从通道读入缓冲区时)。position 总是小于或者等于 limit。
 - **capacity**：缓冲区的 capacity 表明可以储存在缓冲区中的**最大数据容量**。实际上，它指定了底层数组的大小 ―- 或者至少是指定了准许我们使用的底层数组的容量。limit 决不能大于 capacity。


这三个变量一起可以跟踪缓冲区的状态和它所包含的数据。

<img src="media/15260043963031.jpg" width="500px">

#### Buffer的分配
https://blog.csdn.net/seebetpro/article/details/49184305

一般是操作数据的字节（byte）形式，这时经常会用到ByteBuffer这样一个类。ByteBuffer提供了两种静态实例方式： 

```
public static ByteBuffer allocate(int capacity)  
public static ByteBuffer allocateDirect(int capacity) 
```

- **allocate**: 产生的内存开销是在JVM中的. 当Java程序接收到外部传来的数据时，首先是被系统内存所获取，然后在由系统内存复制复制到JVM内存中供Java程序使用。

- **allocateDirect**: 分配方式产生的开销在JVM之外，以就是系统级的内存分配，即直接内存。这块内存不受虚拟机管理。能够省去系统内存复制到JVM内存复制这一步操作，效率上会有所提高。但系统级内存的分配比起JVM内存的分配要耗时得多。

<img src="media/15260055474784.jpg" width="500px">


还可以将一个现有的数组转换为缓冲区，如下所示:

```
byte array[] = new byte[1024];
ByteBuffer buffer = ByteBuffer.wrap( array );
```


#### 读数据
首先从输入通道中读一些数据到缓冲区中。第一次读取得到三个字节。它们被放到数组中从 position 开始的位置，这时 position 被设置为 0。读完之后，position 就增加到 3：

<img src="media/15260041875030.jpg" width="500px">

在第二次读取时，我们从输入通道读取另外两个字节到缓冲区中。这两个字节储存在由 position 所指定的位置上， position 因而增加 2：

<img src="media/15260044398335.jpg" width="500px">

#### flip
flip操作是用于将读转换为写操作，即(从通道读取的数据写入到缓存区转换为从缓冲区读数据写入到通道)。如读入数据后执行flig()，这个方法做两件非常重要的事：

1. 它将 limit 设置为当前 position。表示后续可以从缓冲区中读取的数据最大长度。
2. 它将 position 设置为 0。 表示从缓冲区读取数据的位置。

<img src="media/15260046978308.jpg" width="500px">

#### 写入

在第一次写入时，我们从缓冲区中取四个字节并将它们写入输出通道。这使得 position 增加到 4，而 limit 不变，如下所示：

<img src="media/15260048821018.jpg" width="500px">

#### clear
这个方法重设缓冲区以便接收更多的字节。是将写操作转换为读操作(从缓冲区读取数据写入通道转换为从通道读取数据写入缓冲区) Clear 做两种非常重要的事情：

1. 它将 limit 设置为与 capacity 相同。
2. 它设置 position 为 0。

这时候和原始结构一样：

<img src="media/15260050416517.jpg" width="500px">

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


