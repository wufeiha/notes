参考：https://www.pdai.tech/md/java/thread/java-thread-x-juc-executor-ForkJoinPool.html

# 概述

### Fork/Join

Fork 就是把一个大任务切分为若干个子任务并行地执行，Join 就是合并这些子任务的执行结果，最后得到这个大任务的结果。Fork/Join 框架使用的是工作窃取算法。

### 工作窃取算法

工作窃取算法是指某个线程从其他队列里窃取任务来执行。对于一个比较大的任务，可以把它分割为若干个互不依赖的子任务，为了减少线程间的竞争，把这些子任务分别放到不同的队列里<font color=red>(这里不要混淆，fork出的子任务会被放在当前work线程访问的队列中，由于当前线程的任务可能窃取自别的队列，所以实现了不同子任务在不同的队列)</font>。但是，有的线程会先把自己队列里的任务干完，而其他线程对应的队列里还有任务需要处理，于是它就去其他线程的队列里窃取一个任务来执行。由于此时它们访问同一个队列，为了减小竞争，通常会使用双端队列。<font color=orange>被窃取任务的线程永远从双端队列的头部获取任务，窃取任务的线程永远从双端队列的尾部获取任务。</font><font color=red>每个work线程都有自己的workQueue，但不是每个workQueue都有自己的work</font>

### 工作窃取算法的优缺点

优点：充分利用线程进行并行计算，减少了线程间的竞争。
缺点：双端队列只存在一个任务时会导致竞争，会消耗更多的系统资源，因为需要创建多个线程和多个双端队列。

# 原理

## 关键对象

- 任务对象: `ForkJoinTask` (包括`RecursiveTask`、`RecursiveAction` 和 `CountedCompleter`)

  > - RecursiveTask 是一个可以递归执行的 ForkJoinTask
  > - RecursiveAction 是一个无返回值的 RecursiveTask
  > - CountedCompleter 在任务完成执行后会触发执行一个自定义的钩子函数

- 执行Fork/Join任务的线程: `ForkJoinWorkerThread`

- 线程池: `ForkJoinPool`

ForkJoinPool 只接收 ForkJoinTask 任务(在实际使用中，也可以接收 Runnable/Callable 任务，但在真正运行时，也会把这些任务封装成 ForkJoinTask 类型的任务)。在实际运用中，我们一般都会继承 RecursiveTask 、RecursiveAction 或 CountedCompleter 来实现我们的业务需求，而不会直接继承 ForkJoinTask 类。

## 工作窃取

- 每个线程都有自己的一个WorkQueue，该工作队列是一个双端队列。
- 队列支持三个功能push、pop、poll
- push/pop只能被队列的所有者线程调用，而poll可以被其他线程调用。
- 划分的子任务调用fork时，都会被push到自己的队列中。
- 默认情况下，工作线程从自己的双端队列获出任务并执行。
- 当自己的队列为空时，线程随机从另一个线程的队列末尾调用poll方法窃取任务。

![img](https://i.loli.net/2021/06/28/IG5AyMzbnrqilc6.png)

## 执行流程

- 工作队列的数量与线程池并发参数有关系（也就是线程的数量）
- 每个工作队列容量，初始容量$2^{13}$，初始容量$2^{26}$，2倍扩容
- fork()子任务增加到自身队列中（奇数下标，外部任务为偶数下标），空闲线程会自动窃取任务

![img](https://i.loli.net/2021/06/28/Bp3vR9k4PdZxGyV.png)

## ForkJoinPool状态

```java
private static final int  RSLOCK     = 1;       //线程池被锁定
private static final int  RSIGNAL    = 1 << 1;    //线程池有线程需要唤醒
private static final int  STARTED    = 1 << 2;  //线程池已经初始化
private static final int  STOP       = 1 << 29;    //线程池停止
private static final int  TERMINATED = 1 << 30; //线程池终止
private static final int  SHUTDOWN   = 1 << 31; //线程池关闭
```

# 使用

## ForkJoinPool

### ForkJoinPool参数

- parallelism: 并行度，默认为CPU数，最小为1
- factory: 工作线程工厂；
- handler: 处理工作线程运行任务时的异常情况类，默认为null；暂时用不到
- asyncMode: 是否为异步模式，默认为 false。如果为true，表示子任务的执行遵循 FIFO 顺序并且任务不能被合并(join)，这种模式适用于工作线程只运行事件类型的异步任务。

### ForkJoinPool外部任务提交方式

- invoke()会等待任务计算完毕并返回计算结果；
- execute()是直接向池提交一个任务来异步执行，无返回结果；
- submit()也是异步执行，但是会返回提交的任务，在适当的时候可通过task.get()获取执行结果。

### 子任务提交方式

当调用 ForkJoinTask 的 fork() 方法时，程序会调用 `ForkJoinPool.WorkQueue` 的 `push()` 方法异步地执行这个任务，然后立即返回结果。代码如下：

```java
public final ForkJoinTask<V> fork() {
    Thread t;
    if ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread)
        ((ForkJoinWorkerThread)t).workQueue.push(this);
    else
        ForkJoinPool.common.externalPush(this);
    return this;
}
```

说明: 如果当前线程是 Worker 线程，说明当前任务是fork分割的子任务，通过ForkJoinPool.workQueue.push()方法直接把任务放到自己的等待队列中；否则调用ForkJoinPool.externalPush()提交到一个随机的等待队列中(外部任务)

## ForkJoinTask

### ForkJoinTask异常处理

ForkJoinTask 在执行的时候可能抛出异常，但没有办法在主线程中直接捕获异常，所以 ForkJoinTask 提供了 `isCompletedAbnormally()` 方法检查任务是否已经抛出异常或已经被取消。`getException()` 方法返回 `Throwable` 对象，如果任务被取消了则返回 `CancellationException`，如果任务没有完成或者没有抛出异常则返回 `null`。

### ForkJoinTask方法

#### fork() 

当调用 ForkJoinTask 的 fork() 方法时，程序会调用 `ForkJoinPool.WorkQueue` 的 `push()` 方法异步地执行这个任务，然后立即返回结果。代码如下：

```java
public final ForkJoinTask<V> fork() {
    Thread t;
    if ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread)
        ((ForkJoinWorkerThread)t).workQueue.push(this);
    else
        ForkJoinPool.common.externalPush(this);
    return this;
}
```

<font color=red>fork出的子任务会被放在当前work线程访问的队列中，由于当前线程的任务可能窃取自别的队列，所以实现了不同子任务在不同的队列</font>

#### compute() 

直接执行并阻塞等待结果，一些子类特有的方法。

划分成两个子任务后，不要同时调用两个子任务的fork()方法。

表面上看上去两个子任务都fork()，然后join()两次似乎更自然。但事实证明，直接调用compute()效率更高。因为直接调用子任务的compute()方法实际上就是在当前的工作线程进行了计算(线程重用)，这比“将子任务提交到工作队列，线程又从工作队列中拿任务”快得多。

```java
right.fork(); // 计算右边的任务 
long leftAns = left.compute(); // 计算左边的任务(同时右边任务也在计算) 
long rightAns = right.join(); // 等待右边的结果 
return leftAns + rightAns;
```

#### invoke() 

当前线程直接执行并阻塞等待结果。

#### join() 

阻塞等待fork()的结果

`ForkJoinTask的join()和invoke()区别`：都可以用来获取任务的执行结果(另外还有get方法也是调用了doJoin来获取任务结果，但是会响应运行时异常)，它们对外部提交任务的执行方式一致，都是通过externalAwaitDone方法等待执行结果。不同的是invoke()方法会直接执行当前任务；而join()方法则是在当前任务在队列 top 位时(通过tryUnpush方法判断)才能执行，如果当前任务不在 top 位或者任务执行失败调用ForkJoinPool.awaitJoin方法帮助执行或阻塞当前 join 任务。(所以在官方文档中建议了我们对ForkJoinTask任务的调用顺序，一对 fork-join操作一般按照如下顺序调用: a.fork(); b.fork(); b.join(); a.join();。因为任务 b 是后面进入队列，也就是说它是在栈顶的(top 位)，在它fork()之后直接调用join()就可以直接执行而不会调用ForkJoinPool.awaitJoin方法去等待。)

## 例子

```java
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.Future;
import java.util.concurrent.RecursiveTask;

public class CountTask extends RecursiveTask<Integer> {
    private static final int THRESHOLD = 2;
    private int start;
    private int end;

    public CountTask(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        int sum = 0;
        boolean canCompute = (end - start) <= THRESHOLD;
        if (canCompute) { // 如果任务足够小，就计算任务
            for (int i = start; i <= end; i++) {
                sum += i;
            }
        } else { // 如果任务大于阈值，分裂成两个子任务执行
            int middle = (start + end) / 2;
            CountTask leftTask = new CountTask(start, middle);
            CountTask rightTask = new CountTask(middle + 1, end);

            // 执行子任务
            leftTask.fork();
            rightTask.fork();

            // 等待子任务执行完，并得到其结果，注意顺序，此时rightTash在top位，先执行rightTash.join()是任务直接
           //执行而无须等待
            int rightResult = rightTask.join();
            int leftResult = leftTask.join();

            // 合并子任务
            sum = leftResult + rightResult;
        }
        return sum;
    }

    public static void main(String[] args) {
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        CountTask countTask = new CountTask(1, 100);
        Future<Integer> result = forkJoinPool.submit(countTask);
        try {
            if (countTask.isCompletedAbnormally()) {
                System.out.println(countTask.getException());
            }
            System.out.println(result.get());
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

# 注意

## 避免不必要的fork()

划分成两个子任务后，不要同时调用两个子任务的fork()方法。

表面上看上去两个子任务都fork()，然后join()两次似乎更自然。但事实证明，直接调用compute()效率更高。因为直接调用子任务的compute()方法实际上就是在当前的工作线程进行了计算(线程重用)，这比“将子任务提交到工作队列，线程又从工作队列中拿任务”快得多。

> 当一个大任务被划分成两个以上的子任务时，尽可能使用前面说到的三个衍生的invokeAll方法，因为使用它们能避免不必要的fork()。

## 注意fork()、compute()、join()的顺序

为了两个任务并行，三个方法的调用顺序需要万分注意。

```java
right.fork(); // 计算右边的任务
long leftAns = left.compute(); // 计算左边的任务(同时右边任务也在计算)
long rightAns = right.join(); // 等待右边的结果
return leftAns + rightAns;
```

如果我们写成:

```java
left.fork(); // 计算完左边的任务
long leftAns = left.join(); // 等待左边的计算结果
long rightAns = right.compute(); // 再计算右边的任务
return leftAns + rightAns;  
```

或者

```java
long rightAns = right.compute(); // 计算完右边的任务
left.fork(); // 再计算左边的任务
long leftAns = left.join(); // 等待左边的计算结果
return leftAns + rightAns;
```

这两种实际上都没有并行。

## 选择合适的子任务粒度

选择划分子任务的粒度(顺序执行的阈值)很重要，因为使用Fork/Join框架并不一定比顺序执行任务的效率高: 如果任务太大，则无法提高并行的吞吐量；如果任务太小，子任务的调度开销可能会大于并行计算的性能提升，我们还要考虑创建子任务、fork()子任务、线程调度以及合并子任务处理结果的耗时以及相应的内存消耗。

官方文档给出的粗略经验是: 任务应该执行`100~10000`个基本的计算步骤。决定子任务的粒度的最好办法是实践，通过实际测试结果来确定这个阈值才是“上上策”。

> 和其他Java代码一样，Fork/Join框架测试时需要“预热”或者说执行几遍才会被JIT(Just-in-time)编译器优化，所以测试性能之前跑几遍程序很重要。

## 避免重量级任务划分与结果合并

Fork/Join的很多使用场景都用到数组或者List等数据结构，子任务在某个分区中运行，最典型的例子如并行排序和并行查找。拆分子任务以及合并处理结果的时候，应该尽量避免System.arraycopy这样耗时耗空间的操作，从而最小化任务的处理开销。

