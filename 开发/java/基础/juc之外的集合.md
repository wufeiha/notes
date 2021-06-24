以下除Vector，Hashtable，Collections.SynchronizedMap之外都是线程不安全的；

# 集合

## HashSet

### 特点

- 存储结构为HashMap，只是使用的key健，value都是new Object()

## TreeSet

### 特点

- 存储结构为TreeMap，只是使用的key健，value都是new Object()
- 有序的

## LinkedHashSet

# 符号表

## HashMap

### 特点

- 基于数组+链表结构的，每一个数组元素就是一个hash桶，里面存放链表，链表长度超过8转为红黑树。计算节点hash值取模放到数组指定位置，数组长度必须是2的n次幂（加快位运算hash速度）。
- 数组初始长度16，最大长度$2^{30}$。load factor默认0.75，总容量(threshold = length * Load factor = $2^{30} * load factor$)，数组长度每次扩容翻倍
- 默认的负载因子0.75是对空间和时间效率的一个平衡选择，建议大家不要修改，除非在时间和空间比较特殊的情况下，如果内存空间很多而又对时间效率要求很高，可以降低负载因子Load factor的值；相反，如果内存空间紧张而对时间效率要求不高，可以增加负载因子loadFactor的值，这个值可以大于1
- 在扩充HashMap的时候，不需要像JDK1.7的实现那样重新计算hash，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引+oldCap
- Key、value允许null值，也可都为null值；key不重复

## TreeMap

### 特点

- 元素必须实现`Comparable`接口，或者提供比较器，否则按默认顺序。
- 基于链表结构的红黑树
- key、value都不允许null值
- 有序的

## Hashtable

### 特点

- Key、value都不允许null值
- 数组初始长度11，最大长度$2^{30}$。load factor默认0.75，总容量(threshold = length * Load factor = $2^{30} * load factor$)，数组长度每次扩容翻倍
- 其余特性参照HashMap
- synchronized保证线程安全

## Collections.SynchronizedMap(E)

- 包装类，synchronized访问所有Collection类

## LinkedHashMap

## Properties

## WeakHashMap

# 列表

## ArrayList

### 特点

- init_capacity = 默认10；max_capacity = Integer.MAX_VALUE - 8，每次扩容50%
- 基于数组的
- 允许null值

### 函数

- `ensureCapacity(minCapacity)` 扩容容量>=minCapacity
- `trimToSize()`缩小容量至size大小

## Vector

### 特点

- init_capacity = 默认10；max_capacity = Integer.MAX_VALUE - 8。每次扩容如果指定增长数则是增长数，否则翻倍
- 基于数组的
- 允许null值

## LinkedList

# 队列

## PriorityQueue

## ArrayDeque