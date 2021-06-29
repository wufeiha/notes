源码分析： https://www.pdai.tech/md/java/thread/java-thread-x-lock-AbstractQueuedSynchronizer.html

### AQS对CLH队列的增强

1. 支持阻塞而不是一直自旋，竞争激烈时，阻塞性能更好

2. 支持可重入

3. 支持取消节点

4. 支持中断(内部调用LockSupport.park()方法使线程阻塞，而该方法支持响应中断)

5. 支持独占和共享两种模式

6. 支持Condition，Condition替代对象监听器(Monitor)用来等待，唤醒线程，用于线程间的协作，<font color=red>注意：只有拍他锁支持Condition</font>

### 整体结构

   ![img](https://i.loli.net/2021/04/23/Rbj1DQa57czxeEw.png)

### 同步队列结构

#### 结构图

![img](https://i.loli.net/2021/04/23/lko3f7SuItWjdpN.png)

#### 同步队列特点

- 同步队列头结点是哨兵结点，表示获取锁对应的线程结点。
- **当获取锁时，其前驱结点必定为头结点。**获取锁后，需要将头结点指向当前线程对应的结点。
- 当释放锁时，需要通过 unparkSuccessor 方法唤醒头结点的后继结点。

#### 与传统CLH的区别

1. 同步队列是双向链表。事实上，和二叉树一样，双向链表目前也没有无锁算法的实现。双向链表需要同时设置前驱和后继结点，这两次操作只能保证一个是原子性的。

2. **node.pre 一定可以遍历所有结点，是线程安全的**，而后继结点 node.next 则是线程不安全的。也就是说，node.pre 一定可以遍历整个链表，而 node.next 则不一定。至于为什么选择前驱结点而不是后继结点，会在 "第五部分 - AQS 无锁队列" 中进一步分析。

### Node节点
#### 状态说明

- **CANCELLED(1)**：表示当前结点已取消调度。<font color=red>因为超时或者中断</font>，结点会被设置为取消状态，进入该状态后的结点将不会再变化。注意，**只有 CANCELLED 是正值，因此正值表示结点已被取消，而负值表示有效等待状态。**
- **SIGNAL(-1)**：表示后继结点在等待当前结点唤醒。后继结点入队时，会将前继结点的状态更新为 SIGNAL。
- **CONDITION(-2)**：表示结点等待在Condition上，当其他线程调用了Condition的signal()方法后，CONDITION状态的结点将**从等待队列转移到同步队列中**，等待获取同步锁。
- **PROPAGATE(-3)**：共享模式下，前继结点不仅会唤醒其后继结点，同时也可能会唤醒后继的后继结点。
- **INITIAL(0)**：新节点入队时的默认状态。

#### 状态变化过程

![img](https://i.loli.net/2021/04/23/9fJvIdi5tXe6Ppm.png)

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9BM2liY2ljMVhlMGlhU3JtN1BtaDdBOXBHZlJTRUlsRFJyTGVOY2dxdkNlaWNoTEV0MG1rd3Q0VmhLREp0amI2aE1ZQ2daaWMyOWRHMlpoVm94dTFzSjVFeklRLzY0MA?x-oss-process=image/format,png)

### 相关方法

#### 可使用的方法

- <font color=red>void acquire(int arg)</font>	独占式获取同步状态，如果当前线程获取同步状态成功，则由该方法返回，否则，将会进入同步队列等待，该方法将会调用重写的tryAcquire(int arg)方法
- <font color=red>void acquireInterruptibly(int arg)</font>	与acquire(int arg)相同，但是该方法响应中断，当前线程未获取到同步状态而进入同步队列中，如果当前线程被中断，则该方法会抛出InterruptedException 并返回
- <font color=red>void acquireShared(int arg)</font>	共享式的获取同步状态，如果当前线程未获取到同步状态,将会进入同步队列等待，与独占式获取的主要区别是在同一时刻可以有多个线程获取到同步状态
- <font color=red>void acquireSharedInterruptibly(int arg)</font>	可响应中断
- <font color=red>boolean tryAcquireNanos(int arg,long nanos)</font>	在acquireInterruptibly(int arg)基础上增加了超时限制，如果当前线程在超时时间内没有获取到同步状态。那么将会返回false. 如果获取到了返回true
- <font color=red>boolean tryAcquireSharedNanos(int arg, long nanos)</font>	
- <font color=red>boolean release(int arg)</font>	独占式的释放同步状态，该方法会在释放同步状态之后，将同步队列中第个节点包含的线程唤醒
- <font color=red>boolean releaseShared(int arg)</font>	共享式的释放同步状态
- boolean hasQueuedThreads()  是否有正在等待获取同步状态的线程【包含Cancel节点】
- boolean hasContended() 是否曾经有过获取同步状态的线程
- Thread getFirstQueuedThread() 获取同步队列第一个节点
- boolean isQueued(Thread thread) 该线程是否在同步队列中
- boolean hasQueuedPredecessors() 判断有没有别的线程排在了当前线程的前面
- int getQueueLength() 获取同步队列上的线程数量
- Collection<Thread> getQueuedThreads() 获取同步队列上的线程集合
- Collection<Thread> getExclusiveQueuedThreads() 获取同步队列上排他模式的线程集合
- Collection<Thread> getSharedQueuedThreads() 获取同步队列上共享模式的线程集合
- boolean owns(ConditionObject condition). 查询给定的 ConditionObject 是否使用此同步器作为其锁
- boolean hasWaiters(ConditionObject condition) 判断对应等待队列是否有线程等待
- int getWaitQueueLength(ConditionObject condition)获取对应等待队列上的线程的数量
- Collection<Thread> getWaitingThreads(ConditionObject condition) 获取对应等待队列上的线程集合

#### 需要重写的方法

- protected boolean tryAcquiret(int arg)	独占式获取同步状态，实现该方法需要查询当前状态,判断网步状态是否符合预期，然后再进行CAS设置同步状态
- protected boolean tryRelease(int arg)	独占式释放同少状态， 等待获取同步状态的线程将有机会获取同步状态
- protected int tryAcquireshared(in ag)	共享式获取同步状态，返回大于等于0的值，表示获取成功，反之获取失败
- protected boolean tryReleaseShared(int arg)	共享式释放同步状态
- protected boolean isHeldExchusively()	判断是否当前线程所独占

#### 内部方法分析

- ![img](https://upload-images.jianshu.io/upload_images/2791990-e0e37cc586e353ab?imageMogr2/auto-orient/strip|imageView2/2/w/651/format/webp)
- addWaiter：通过 enq 方法添加到同步队列中。需要注意的是 addWaiter 方法会尝试一次添加到同步队列中，如果不成功，再调用 enq 自旋添加到同步队列中。
- acquireQueued：同步队列中节点按规则获取同步状态
- shouldParkAfterFailedAcquire：将前驱结点的状态修改成 SIGNAL，同时会清理已经 CANCELLED 的结点。注意，只有前驱结点的状态为 SIGNAL，当它释放锁时才会唤醒后继结点。
- parkAndCheckInterrupt：挂起线程，并判断线程在自旋过程中，是否被中断过。

- unparkSuccessor：唤醒后继结点。

### 一些节点变化理解

- 获取同步状态成功后，节点在同步队列中消失
- await()后，条件队列新增节点
- signal()后，条件队列删除节点，同步队列新增节点
