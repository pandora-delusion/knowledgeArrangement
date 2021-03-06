### RTMP 协议

​	RTMP协议是Real Time Message Protocol(实时信息传输协议)的缩写，它是由Adobe公司提出的一种应用层的协议，用来解决多媒体数据传输流的多路复用（Multiplexing）和分包（packetizing）的问题。

​	RTMP协议是应用层协议，是要靠底层可靠的传输层协议（通常是TCP）来保证信息传输的可靠性的。在基于传输层协议的链接建立完成后，RTMP协议也要客户端和服务器通过“握手”来建立基于传输层链接之上的RTMP Connection链接，在Connection链接上会传输一些控制信息，其中CreateStream命令会创建一个Stream链接，用于传输具体的音视频数据和控制这些信息传输的命令信息。RTMP协议传输时会对数据做自己的格式化，这种格式的消息我们称之为RTMP Message，而实际传输的时候为了更好地实现多路复用、分包和信息的公平性，发送端会把Message划分为带有Message ID的Chunk，每个Chunk可能是一个单独的Message，也可能是Message的一部分，在接受端会根据chunk中包含的data的长度，message id和message的长度把chunk还原成完整的Message，从而实现信息的收发。

### IO多路复用

​	IO multiplexing就是我们说的select，poll，epoll，有些地方也称这种IO方式为event driven IO。select/epoll的好处就在于单个process就可以同时处理多个网络连接的IO。它的基本原理就是select，poll，epoll这个function会不断的轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程。

​	当用户进程调用了select，那么整个进程会被block，而同时，kernel会“监视”所有select负责的socket，当任何一个socket中的数据准备好了，select就会返回。这个时候用户进程再调用read操作，将数据从kernel拷贝到用户进程。

### IO模型

除了上面介绍的IO多路复用模型，linux下面还有四种可用的IO模型，分别是：

##### 阻塞式IO

进程或线程调用某个函数，该函数需要满足特定条件才能向下执行，如果条件不满足，则会使调用进程或线程阻塞，让出CPU控制权，并一直持续到条件满足为止。默认情况下，所有套接字都是阻塞的。

##### 非阻塞式IO

非阻塞式IO不会使调用进程或线程永远阻塞，具体表现为：如果IO操作不能完成，则立即出错返回，调用进程或线程继续向下执行。这样我们的I/O操作函数将不断的测试数据是否已经准备好，如果没有准备好，继续测试，直到数据准备好为止。在这个不断测试的过程中，会大量的占用CPU的时间。

##### 信号驱动IO

首先我们允许套接口进行信号驱动I/O,并安装一个信号处理函数，进程继续运行并不阻塞。当数据准备好时，进程会收到一个SIGIO信号，可以在信号处理函数中调用I/O操作函数处理数据。

##### 异步IO模型

当一个异步过程调用发出后，调用者不能立刻得到结果。实际处理这个调用的部件在完成后，通过状态、通知和回调来通知调用者的输入输出操作



**select、poll、epoll区别总结：**

select：一般操作系统均有实现，单个进程可监视的文件描述符数量有限，对socket进行扫描是线性扫描，即采用轮询的方法，效率较低，随着FD的增加造成遍历速度慢的”线性性能下降问题“。再消息传递方式上，内核需要将消息传递到用户空间，都需要内核拷贝动作。

**select具有O(n)的无差别轮询复杂度**，同时处理的流越多，无差别轮询时间就越长。

poll：poll本质上和select没有区别，它将用户传入的数组拷贝到内核空间，然后查询每个fd对应的设备状态， 但是它没有最大连接数限制，原因是它是基于链表来存储的。也是线性扫描，效率低。再消息传递方式上，内核需要将消息传递到用户空间，都需要内核拷贝动作

epoll：虽然连接数有上限，但是很大，1G内存的机器上可以打开10万左右的连接。因为epoll内核中实现是根据每个fd上的callback函数来实现的，只有活跃的socket才会主动调用callback，所以在活跃socket较少的情况下，使用epoll没有前面两者的线性下降的性能问题，但是所有socket都很活跃的情况下，可能会有性能问题。再消息传递方式上，epoll通过内核和用户空间共享一块内存来实现的。



**但select，poll，epoll本质上都是同步I/O，因为他们都需要在读写事件就绪后自己负责进行读写，也就是说这个读写过程是阻塞的**，而异步I/O则无需自己负责进行读写，异步I/O的实现会负责把数据从内核拷贝到用户空间。 

### 系统结构

![服务器类图](F:\mycode\knowledgeArrangement\项目介绍\clip_image002.png)

​	Header、Message和Command类是对RTMP协议中定义的数据结构的实现。Header类封装chunk头部的信息，Message类封装rtmp消息，由header和data域组成，Command是消息命令类的封装。Protocol类则负责握手（parseHandshake方法），chunk的组装拆解（parseMessages，write方法），对协议控制消息和用户控制消息的处理（protocolMessage方法）。但保留处理命令消息的实现，即messageReceived方法留给Protocol类的子类Client类实现。Client类负责管理客户端连接的具体操作和对收到的命令消息的处理（messageReceived方法）和流（Stream类）对象的管理。FlashServer类将监听客户端socket的连接和Client对象的创建委托给Server类实现，自己则负责处理客户端（或者流）各种命令消息的响应。 App类代表服务器端具体的应用，FlashServer中将回调App类中的方法，App类为了提供用户更多具体服务而设计，供以后拓展使用。以后的应用类需要继承App类。FLV类则负责将音视频消息流保存为FLV文件和从FLV文件中解析发送消息流。

#### 多线程并发模式，一个线程一个链接的优点

一定程度上极大地提高了服务器的吞吐量，因为之前的请求在read阻塞以后，不会影响到后续的请求，因为他们在不同的线程中。

#### 多线程并发模式，一个链接一个线程的缺点

缺点在于资源要求太高，系统中创建线程是需要比较高的系统资源的，如果连接数太高，系统无法承受，而且，线程的反复创建-销毁也需要代价。

改进方法是：

采用基于事件驱动的设计，当有事件触发时，才会调用处理器进行数据处理。使用Reactor模式，对线程的数量进行控制，一个线程处理大量的事件。

#### 单线程模式Reactor的缺点：

但某个socket的回调阻塞时，会导致其他活跃的socket的回调都得不到执行，并且导致整个服务不能接受新的client请求。这种单线程模型不能充分利用多核资源，所以实际使用的不多。因此单线程模型仅仅适用于业务处理组件能够快速完成的场景。

#### 多线程模式的Reactor

将业务逻辑（回调部分）的执行交给线程池（进程池）处理，用多线程进行业务处理。

对于Reactor而言，仍然为单个线程，如果服务器为多核CPU，可以将Reactor拆分为多个线程

#### Reactor编程的优点

1）响应快，不必为单个同步时间所阻塞，虽然Reactor本身依然是同步的；

2）编程相对简单，可以最大程度的避免复杂的多线程及同步问题，并且避免了多线程/进程的切换开销；

3）可扩展性，可以方便的通过增加Reactor实例个数来充分利用CPU资源；

4）可复用性，reactor框架本身与具体事件处理逻辑无关，具有很高的复用性；