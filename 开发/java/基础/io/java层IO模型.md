## Reactor模式

### 简介

<font color=red>支持网络IO通道，不支持文件IO。</font>底层实现：Linux 2.6之前是select、poll，2.6之后是epoll；Windows是IOCP(真正的异步)

> 1. 应用程序注册读就绪事件和相关联的事件处理器到事件分离器上（对应的selector）
> 2. 事件分离器等待事件的发生
> 3. 当发生读就绪事件的时候，事件分离器调用第一步注册的事件处理器
> 4. 事件处理器首先执行实际的读操作，然后进行业务的处理

### 几种模式

#### **单reactor单回调线程结构图**。

经典应用：libevent、libuv的network IO、Cronet的底层IO
![e3.png](https://i.loli.net/2021/06/24/6OKHX4bmzoQLGei.png)

#### **单reactor多回调线程结构**。
经典应用：libuv的file IO

![21215116-d912aa6ee03547e3865d734288368b41.png](https://i.loli.net/2021/06/24/mD1yNCsiAbkuV6Z.png)

#### **多reactor多回调线程结构**，

经典应用：netty
![21215443-4e028b9ed85542169c62377970d745a2.png](https://i.loli.net/2021/06/24/bV9wrykvgPM5BoE.png)

### 适用场景

- <font color=red>Reactor实现相对简单，对于耗时短的处理场景处理高效；</font>
- <font color=red>事件的串行化对应用是透明的，可以顺序的同步执行而不需要加锁；</font>
- <font color=red>将与应用无关的多路分解和分配机制和与应用相关的回调函数分离开来，</font>
- <font color=green>Reactor处理耗时长的操作会造成事件分发的阻塞，影响到后续事件的处理；</font>

同时接收多个服务请求，并且依次同步的处理它们的事件驱动程序。适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，并发局限于应用中。 


## Proactor模式

###  简介

<font color=red>支持网络IO通道，也支持文件IO。</font>底层实现：Linux 2.6之前是select、poll，2.6之后是epoll；Windows是IOCP(真正的异步)

> 1. 应用程序初始化一个异步读取操作，然后注册相应的事件处理器，**此时事件处理器不关注读取就绪事件，而是关注读取完成事件，这是区别于Reactor的关键。**
> 2. 事件分离器等待读取操作完成事件
> 3. 在事件分离器等待读取操作完成的时候，操作系统调用内核线程完成读取操作（异步IO都是操作系统负责将数据读写到应用传递进来的缓冲区供应用程序操作，操作系统扮演了重要角色），并将读取的内容放入用户传递过来的缓存区中。这也是区别于Reactor的一点，Proactor中，应用程序需要传递缓存区。
> 4. 事件分离器捕获到读取完成事件后，激活应用程序注册的事件处理器，**事件处理器直接从缓存区读取数据，而不需要进行实际的读取操作。**

流程与reactor类似，区别在于proactor在IO ready事件触发后，完成IO操作再通知应用回调。经典应用例如boost asio异步IO库。虽然在linux平台还是基于epoll，但是内部实现了异步操作处理器(Asynchronous Operation Processor)以及异步事件分离器(Asynchronous Event Demultiplexer)将IO操作与应用回调隔离。<font color=red>注意：这是用户系统级别的异步，而不是内核级别的异步，读写完成操作不是由内核自动完成的，而是用户线程模拟完成的。</font>
boost.asio结构与流程图
![proactor.png](https://i.loli.net/2021/06/24/S5WvCRz4hwD18Oo.png)

### 适用场景

- <font color=red>Proactor性能更高，能够处理耗时长的并发场景；</font>
- <font color=green>Proactor实现逻辑复杂；依赖操作系统对异步的支持，目前实现了纯异步操作的操作系统少，实现优秀的如windows IOCP，但由于其windows系统用于服务器的局限性，目前应用范围较小；而Unix/Linux系统对纯异步的支持有限（通过epoll来实现）；</font>
  AIO方式适用于连接数目多且连接比较长（重操作）的架构，比如相册服务器。

