### 多线程开发需要注意的问题

- **原子性**
   即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执。（加锁实现）
- **可见性**
   可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。（内存屏障实现）
- **有序性**（重排序）
   即程序执行的顺序按照代码的先后顺序执行。（内存屏障实现）

### 什么是CAS

CAS（Compare and swap）直译过来就是比较和替换，是一种通过硬件实现并发安全的常用技术，底层通过利用CPU的CAS指令对缓存加锁或总线加锁的方式来实现多处理器之间的原子操作。

### 为什么已经有synchronized还要AQS  

主要不是因为性能。synchronized已经做了优化，创建初期锁不会那么重，有一个锁的升级过程，主要有以下3点原因

1. **能够响应中断**。synchronized 获取锁的过程中【即进入`BLOCKED`状态，而AQS没有这个状态】无法被中断。以下测试线程h1持有锁执行任务，线程h2等待获取锁时线程的状态

   1. **synchronized**，此时h2的状态为`BLOCKED`，不可响应中断

      ```java
      public class Test {
      
        public static void main(String[] args) throws InterruptedException {
          Object lock = new Object();
          Thread h1 = new Thread(() -> {
            synchronized (lock) {
              try {
                System.out.println("aaaaaaaa");
                TimeUnit.MINUTES.sleep(100);
              } catch (InterruptedException e) {
                e.printStackTrace();
              }
            }
      
          });
          Thread h2 = new Thread(() -> {
            synchronized (lock) {
              System.out.println("bbbbbb");
            }
          });
      
          h1.start();
          TimeUnit.SECONDS.sleep(1);
          h2.start();
          TimeUnit.SECONDS.sleep(1);
          h2.interrupt();
          h2.join();
          h1.join();
      
        }
      }
      ```

      

   1. **AQS**，此时h2的状态为`TIMED`，可响应中断

      ```java
      public class Test {
      
        public static void main(String[] args) throws InterruptedException {
          ReentrantLock lock = new ReentrantLock();
          Thread h1 = new Thread(() -> {
            try {
              lock.lock();
              System.out.println("aaaaaaaa");
              TimeUnit.MINUTES.sleep(100);
            } catch (InterruptedException e) {
              e.printStackTrace();
            } finally {
              lock.unlock();
            }
          });
          Thread h2 = new Thread(() -> {
            try {
              lock.lockInterruptibly();
            } catch (InterruptedException e) {
              e.printStackTrace();
            }
            System.out.println("bbbbbb");
            lock.unlock();
          });
          h1.start();
          TimeUnit.SECONDS.sleep(1);
          h2.start();
          TimeUnit.SECONDS.sleep(10);
          h2.interrupt();
          h2.join();
          h1.join();
        }
      }
      ```

      

2. **支持超时**。如果线程在一段时间之内没有获取到锁，不是进入阻塞状态，而是返回一个错误，那这个线程也有机会释放曾经持有的锁。这样也能破坏不可抢占条件。

3. **非阻塞地获取锁**。如果尝试获取锁失败，并不进入阻塞状态，而是直接返回，那这个线程也有机会释放曾经持有的锁。这样也能破坏不可抢占条件。

4. **可以实现公平锁**，synchronized只能是非公平的

5. synchronized只支持一个Condition，而AQS支持多个

6. **可以实现共享锁**，synchronized只能是互斥的

### 自旋获取锁与阻塞获取锁（也就是切换线程上下文）的应用场合

- 如果是多核处理器，如果预计线程等待锁的时间很短，短到比线程两次上下文切换时间要少的情况下，使用自旋锁是划算的。
- 如果是多核处理器，如果预计线程等待锁的时间较长，至少比两次线程上下文切换的时间要长，建议使用互斥量。
- 如果是单核处理器，一般建议不要使用自旋锁。因为，在同一时间只有一个线程是处在运行状态，那如果运行线程发现无法获取锁，只能等待解锁，但因为自身不挂起，所以那个获取到锁的线程没有办法进入运行状态，只能等到运行线程把操作系统分给它的时间片用完，才能有机会被调度。这种情况下使用自旋锁的代价很高。

### 公平锁与非公平锁对比

**公平锁：**多个线程按照申请锁的顺序去获得锁，线程会直接进入队列去排队，永远都是队列的第一位才能得到锁。所有的线程都能得到资源，不会饿死在队列中。但是吞吐量会下降很多，队列里面除了第一个线程，其他的线程都会阻塞，cpu唤醒阻塞线程的开销会很大。
**非公平锁：**多个线程去获取锁的时候，会直接去尝试获取，如果能获取到，就直接获取到锁；获取不到，再去进入等待队列<font color=orange>（进入到队列后就和公平锁一样了）</font>。可以减少CPU唤醒线程的开销，整体的吞吐效率会高点，<font color=orange>这样可能导致队列中间的线程一直获取不到锁或者长时间获取不到锁，导致饿死。</font>

