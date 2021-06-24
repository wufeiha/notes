### 派生图

![并发集合](https://i.loli.net/2021/06/17/zTFae7L6X5WtwE1.png)

### 有界队列无界队列的选择

在稳定性要求特别高的系统中，为了防止生产者速度过快，导致内存溢出，只能选择有界队列；同时，为了减少Java的垃圾回收对系统性能的影响，会尽量选择数组实现的队列。

### 阻塞队列与非阻塞队列的选择

- 适用阻塞队列的好处:多线程操作共同的队列时不需要额外的同步,另外就是队列会自动平衡负载,即那边(生产与消费两边)处理快了
  就会被阻塞掉,从而减少两边的处理速度差距。
- 非阻塞队列性能高，相对而言稳定性略差
- LinkedBlockingQueue多用于任务队列，ConcurrentLinkedQueue多用于消息队列

### 双端队列用途

- 双端队列最常用的地方就是实现一个长度动态变化的窗口或者连续区间。
- 工作窃取模式

### 各个容器的iterator 的线程安全程度

### Splititerator 使用及其线程安全程度

- `boolean tryAdvance(Consumer action);` 该方法会处理每个元素，如果没有元素处理，则应该返回false，否则返回true。
- `default void forEachRemaining(Consumer action)` 该方法有默认实现，功能后面会介绍。
- `Spliterator trySplit();` 将一个Spliterator分割成多个Spliterator。分割的Spliterator被用于每个子线程进行处理，从而达到并发处理的效果。
- `long estimateSize();` 该方法返回值并不会对代码正确性产生影响，但是会影响代码的执行线程数，后续会介绍一下
- `int characteristics();` 给出stream流具有的特性，不同的特性，不仅是会对流的计算有优化作用，更可能对计算结果会产生影响，后续会稍作介绍。
- `default Comparator getComparator()` 对sorted的流，给出比较器。后续给出研究代码。

