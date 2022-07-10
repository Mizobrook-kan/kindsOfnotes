**无锁编程**

细粒度控制避免数据竞争

无锁编程是指使用原语操作避免小对象（例如单字或双字）的数据竞争，没有数据竞争的原语操作叫做原子操作，可以用于实现更高级的并发机制，例如锁、线程、无锁数据结构。

原子操作保证了内存状态由特定的内存状态。

memory_order枚举类型，memory_order_relaxed表示没有操作会排序内存

memory_order_release,memory_order_acq_rel, 和memory_order_seq_cst表示store操作在受影响的内存区域执行一次释放操作

memory_order_consume：

memory_order_acquire,memory_order_acq_rel, 和memory_order_seq_cst：一个load操作在受影响的内存区域执行一次acquire操作

例如，使用原子load和store表示relaxed内存序。

```
// thread 1:
r1 = y.load(memor y_order_relaxed);
x.store(r1,memory_order_relaxed);
// thread 2:
r2 = x.load(memory_order_relaxed);
y.store(42,memor y_order_relaxed);
```

memory_order_relaxed下的执行顺序是

```
y.store(42,memor y_order_relaxed);
r1 = y.load(memor y_order_relaxed);
x.store(r1,memory_order_relaxed);
r2 = x.load(memory_order_relaxed);
```

**task库**

任务级并发：future、promise、packaged_task、async()

future比mutex交换信息方面更好，mutex比atomic实现一些简单逻辑更好。

对于管理共享数据的问题以及技术，看完之后应该理解到共享数据最好应该是避免的。通信暗示着数据共享，但应该避免由程序员直接控制共享逻辑。

* 处理并发时程序员可能遇到的问题
* 标准库提供的并发设施的详细描述
