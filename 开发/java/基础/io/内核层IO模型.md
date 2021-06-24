通常用户进程中的一个完整IO分为两阶段：用户进程空间到内核空间、内核空间到设备空间（磁盘、网络等)。

### 同步阻塞IO模型

流程：线程发起IO系统调用后会被被阻塞，转到内核空间处理，整个IO处理完毕后返回数据。
优缺点：一般需要给每个IO请求分配一个IO线程，因此系统开销大。
典型应用：阻塞socket、Java BIO；
![20161028200138896.jpg](https://i.loli.net/2021/06/23/8FnqXV9kruUap2l.png)

### 同步非阻塞IO模型

流程：线程不断轮询读取内核IO设备缓冲区，如果没数据则立即返回EWOULDBLOCK，有则返回数据。
优缺点：CPU消耗多，无效IO多。
典型应用：非阻塞socket（设置为NONBLOCK）
![20161028200139219.jpg](https://i.loli.net/2021/06/24/OpW7K8QBqEUojXs.png)

### 同步IO复用模型

流程：线程调用select/poll/epoll传入多个设备fd，然后阻塞或者轮询等待。如果有IO设备准备好则返回可读条件，用户主动调IO读写，如果没有则继续阻塞。**但select，poll，epoll本质上都是同步I/O，因为他们都需要在读写事件就绪后自己负责进行读写，也就是说这个读写过程是阻塞的**，而异步I/O则无需自己负责进行读写，异步I/O的实现会负责把数据从内核拷贝到用户空间。 
优缺点：相比前两种模型，IO复用可以监听多个IO设备，所以一个线程内可以处理多个网络请求。
典型应用：nginx、Java NIO、Netty。

![20161028200139703.jpg](https://i.loli.net/2021/06/24/ItEKJFWPagyVuGH.png)

#### select

- 时间复杂度O(n)
- 它仅仅知道了，有I/O事件发生了，却并不知道是哪那几个流（可能有一个，多个，甚至全部），我们只能无差别轮询所有流，找出能读出数据，或者写入数据的流，对他们进行操作。所以**select具有O(n)的无差别轮询复杂度**，同时处理的流越多，无差别轮询时间就越长。
- 单个进程可监视的fd数量被限制，即能监听端口的大小有限，32位机默认是1024个。64位机默认是2048
- 内核需要将消息传递到用户空间，都需要内核拷贝动作

#### poll

- poll本质上和select没有区别， **但是它没有最大连接数的限制**，原因是它是基于链表来存储的.

#### epoll

- 时间复杂度O(1)
- linux独有，java NIO在linux平台上的底层实现
- **epoll可以理解为event poll**，不同于忙轮询和无差别轮询，epoll会把哪个流发生了怎样的I/O事件通知我们。所以我们说epoll实际上是**事件驱动（每个事件关联上fd）**的，只有活跃可用的FD才会调用callback函数；即Epoll最大的优点就在于它只管你“活跃”的连接，而跟连接总数无关，因此在实际的网络环境中，Epoll的效率就会远远高于select和poll
- 无限制，1G的内存上能监听约10万个端口
- 内存拷贝，利用mmap()文件映射内存加速与内核空间的消息传递；即epoll使用mmap减少复制开销，epoll通过内核和用户空间共享一块内存来实现的

### 异步信号驱动IO模型

流程：利用linux信号机制，用sigaction函数将SIGIO读写信号以及handler回调函数存在内核队列中。当设备IO缓冲区可写或可读时触发SIGIO中断，返回设备fd并回调handler。
优缺点：这种异步回调方式避免用户或内核主动轮询设备造成的资源浪费，但是此方式会存在问题，首先handler是在中断环境下运行，多线程不稳定而且要考虑信号的平台兼容性，其次是SIGIO信号被POSIX定义为standard signals不会进入队列，所以同时存在多个SIGIO会只触发第一个SIGIO。
![20161028200140021.jpg](https://i.loli.net/2021/06/24/y17TtMLO5vsqAdk.png)

### 异步IO模型

#### window

- iocp，<font color=red>真正的内核异步</font>，java NIO在window平台上的底层实现。

#### linux

- glibc，<font color=red>在用户空间用多线程封装内核IO实现异步，伪异步。</font>大致流程是用户调用aio_API传入aiocb("asynchronous I/O control block")结构体，aio将操作入队列并立即返回。当内核拷贝完数据之后，会触发signal通知用户aiocb已经准备好，也可以通过调用aio_error检查aiocb状态是否完成。因为是在用户空间实现的异步，频繁的线程和内核态切换导致明显的性能问题和不可弹性伸缩，Linux Manual也注明这是内核aio成熟前的过渡方案。

![ABUIABAEGAAgvMOrwAUoypzSBzCABTjtAg.png](https://i.loli.net/2021/06/24/lQfUF4rIRHDh1Ew.png)

- libaio/内核aio，<font color=red>真正的内核异步</font>允许应用引入libaio即可使用aio，否则应用需要自己定义syscall。流程是首先调用io_setup获取aio_context，然后构造一个或多个iocb(包含fd,buffer)结构体通过io_submit提交到内核队列中，最后用阻塞或轮询方式调用io_getevents获取io_event判断IO是否完成。这是linux官方方案，但是由于设计上的缺陷，目前只支持O_DIRECT(绕过内核缓冲区直接IO)方式open的文件设备，其他情况要不返回错误，要不退化到同步阻塞方式在io_submit中读写。尽管功能仍然不成熟，但是性能上有人验证一个进程内用aio读写16个fd，与16个进程pread的IO吞吐量一致。新版本nginx支持，但是开启会丧失了零拷贝的特征