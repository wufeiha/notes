## 状态切换

![java-thread-x-key-schronized-2](https://i.loli.net/2021/06/26/tjYanhg2XmM5ecH.jpg)



<img src="https://i.loli.net/2021/03/09/PhHwodsRLUZmJIX.jpg" alt="Java线程的6种状态及切换(透彻讲解)_潘建南的博客-CSDN博客_线程状态" style="zoom:50%;" />

`BLOCKED`：任意线程对Object的访问，首先要获得Object的监视器，如果获取失败，该线程就进入同步状态，线程状态变为BLOCKED，当Object的监视器占有者释放后，在同步队列中得线程就会有机会重新获取该监视器

`WAITING`：等待队列，需要被唤醒

`TIMED_WAITING`：等待队列，自动唤醒

## 方法

1. <font color='red'>stop()</font>  不建议使用，强制停止线程，某些资源得不到释放导致内存泄漏；会释放所有锁，导致同步出现问题

2. <font color='red'>interrupt()/isInterrupted()/interrupted()</font>  

   > 1. Interrupt()中断线程，如果中断`运行线程`需要被中断线程配合(即使用`interrupted()`判断配合是否中断)，在被中断内判断标志位，进行相关处理【抛出中断异常】；如果中断`阻塞线程`(例如执行sleep()/wait()/park())会抛出InterruptedException异常导致线程结束<font color=red>【注意：这个中断响应是底层实现的，不必纠结为什么阻塞不占用CPU的情况下可以响应中断】</font>。
   > 2. interrupted()是静态方法，内部调用isInterrupted(true),会清除中断位，也就是说执行Interrupt()中断线程，调用Thread.interrupted()第一次返回true,第二次返回false
   > 3. isInterrupted()，内部调用isInterrupted(false)不会清除中断位, 执行Interrupt()中断线程后，调用isInterrupted()永远返回true

3. <font color='red'>suspend()/resume()</font>  suspend()挂起线程不会释放锁，所以不调用resume()恢复线程则会发生死锁，

4. <font color='red'>yield()</font>  让出cpu进入就绪态，下个时间片可能其他线程使用，也可能还是当前线程使用

5. <font color='red'>join()</font>  会阻塞当前线程，待到调用线程执行结束后才恢复主线程

6. <font color='red'>isAlive()</font> 判断线程是否存活()，已启动还未终止线程就是存活的

7. <font color='red'>setDaemon(boolean on)/isDaemon()</font>  设置守护线程，所有用户线程执行完毕，守护线程自动退出，必须在启动之前调用方法

8. <font color='red'>holdsLock(Object obj)</font>  查看线程是否持有obj锁

9. <font color='red'>getState()</font> 返回当前线程状态

10. <font color='red'>setDefaultUncaughtExceptionHandler(UncaughtExceptionHandler eh)</font>  全局catch，捕获未处理的异常

11. <font color='red'>sleep()</font>  执行sleep()方法前不需要获得锁，如果在synchronized同步块里执行也不会释放锁

12. <font color='red'>wait()/notify()</font>  执行wait()/notify()方法前需要获得锁，也就是说必须在synchronized代码块里执行；执行完wait方法会立马释放锁，执行完notify要等到当前代码块执行完才会释放锁；注意二者逻辑顺序，否则线程不能正确唤醒

13. <font color='red'>park()</font>  执行park()方法前不需要获得锁，在synchronized同步块里执行也不会释放锁。park是通过信号量实现的，不需要注意park、unpark的顺序

## 不响应中断的状态

- `BLOCKED`状态，线程获取锁的过程。【注意AQS无这个状态，也就是AQS即使在获取锁的过程中也可以中断】
