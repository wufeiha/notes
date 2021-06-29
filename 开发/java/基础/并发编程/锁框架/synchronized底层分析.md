## 状态切换

![java-thread-x-key-schronized-2](https://i.loli.net/2021/06/26/tjYanhg2XmM5ecH.jpg)



<img src="https://i.loli.net/2021/03/09/PhHwodsRLUZmJIX.jpg" alt="Java线程的6种状态及切换(透彻讲解)_潘建南的博客-CSDN博客_线程状态" style="zoom:50%;" />

`BLOCKED`：任意线程对Object的访问，首先要获得Object的监视器，如果获取失败，该线程就进入同步状态，线程状态变为BLOCKED，当Object的监视器占有者释放后，在同步队列中得线程就会有机会重新获取该监视器

`WAITING`：等待队列，需要被唤醒

`TIMED_WAITING`：等待队列，自动唤醒

## 锁升级过程

无锁$\rightarrow$偏向锁$\rightarrow$轻量级锁$\rightarrow$重量级锁；过程不可逆

![image-20210626134942443](https://i.loli.net/2021/06/26/KG9cMoE5gW38u4x.png)

### 偏向锁

<font color=red>偏向锁使用了一种等到竞争出现才释放锁的机制</font>,所以当其他线程尝试竞争偏向锁时, 持有偏向锁的线程才会释放锁。偏向锁的撤销,需要等待全局安全点(在这个时间点上没有正在执行的字节码)。它会首先暂停拥有偏向锁的线程,然后检查持有偏向锁的线程是否活着, 如果线程不处于活动状态,则将对象头设置成无锁状态;如果线程仍然活着,拥有偏向锁的栈会被执行,遍历偏向对象的锁记录,栈中的锁记录和对象头的Mark Word要么重新偏向于其他线程,要么恢复到无锁或者标记对象不适合作为偏向锁,最后唤醒暂停的线程。

<img src="https://i.loli.net/2021/06/26/CSlVHzD7Kksngfb.png" alt="image-20210626134318001" style="zoom:50%;" />

### 轻量级锁

**`加锁`:**线程在执行同步块之前,JVM会先在当前线程的栈桢中创建用于存储锁记录的空间,并将对象头中的Mark W ord复制到锁记录中,官方称为Displaced Mark Word。然后线程尝试使用CAS将对象头中的Mark Word替换为指向锁记录的指针。如果成功,当前线程获得锁,如果失败,表示其他线程竞争锁,当前线程便尝试使用自旋来获取锁

**`解锁`:**轻量级解锁时,会使用原子的CAS操作将Displaced Mark Word替换回到对象头（获取锁有个自适应自旋的过程，多次获取失败膨胀为重量级锁）,如果成功,则表示没有竞争发生。如果失败,表示当前锁存在竞争,锁就会膨胀成重量级锁。图2-2是两个线程同时争夺锁,导致锁膨胀的流程图

<img src="https://i.loli.net/2021/06/26/X6PUyoK7suMwcvH.png" alt="image-20210626134534050" style="zoom:50%;" />

### 重量级锁

Synchronized 的重量级锁是通过对象内部的一个叫做监视器锁（monitor）来实现的，监视器锁本质又是依赖于底层的操作系统的 Mutex Lock（互斥锁）来实现的，每次获取和释放锁都会带来**用户态和内核态的切换**，从而增加系统的**性能开销**。

## 锁优化手段

`锁粗化(Lock Coarsening)`：也就是减少不必要的紧连在一起的unlock，lock操作，将多个连续的锁扩展成一个范围更大的锁。

`锁消除(Lock Elimination)`：通过运行时JIT编译器的逃逸分析来消除一些没有在当前同步块以外被其他线程共享的数据的锁保护，通过逃逸分析也可以在线程本地Stack上进行对象空间的分配(同时还可以减少Heap上的垃圾收集开销)。

`适应性自旋(Adaptive Spinning)`：当线程在获取轻量级锁的过程中执行CAS操作失败时，在进入与monitor相关联的操作系统重量级锁(mutex semaphore)前会进入忙等待(Spinning)然后再次尝试，当尝试一定的次数后如果仍然没有成功则调用与该monitor关联的semaphore(即互斥锁)进入到阻塞状态



