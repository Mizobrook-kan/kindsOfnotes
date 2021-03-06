**进程**

从CPU要地址空间的就是进程。

（我是这么想的，首先进程和线程都是描述占用CPU时间片的一段程序执行过程，进程和线程都可以被操作系统调度，比如线程有调度算法吧，涉及到调度算法这些就是操作系统在管理，但是用户态也可以完成这个事情。


进程和线程这两个概念只在操作系统层面能区分开。进程是线程的容器，这是因为进程能从CPU要到地址空间，管理的线程从进程的地址空间中再划分出一片空间给线程，进程和线程都要时间片但CPU只能执行线程中的一段程序。


地址空间本身就是一种物理内存的虚拟化了，

~~地址空间——虚拟内存——分页机制~~

~~能够唤起系统调用的就是进程。~~

~~~~进程能够被操作系统打断并通过切换来分时占用 CPU 资源；~~需要 **地址空间** 来放置代码和数据；有从开始到结束运行这样的生命周期。~~~~强大的动态变化功能：进程可以在运行的过程中，创建 **子进程** 、 用新的 **程序** 内容覆盖已有的 **程序** 内容。~~

~~UNIX 系内核并经过简化的精简版进程模型。在该模型下，若想对进程进行管理，实现创建、退出等操作，核心就在于 `fork/exec/waitpid` 三个系统调用。~~

进程是上下文切换之间的程序执行的部分。

~~具体细节：初始进程的创建、进程切换机制、进程调度机制、进程生成机制、进程资源回收机制~~

**进程** (Process) 的含义是 **在操作系统管理下的程序的一次执行过程**。从动态和静态的角度，

它的动态性主要体现在：

1. 它是一个过程，从时间上来看有开始也有结束；
2. 在该过程中对于可执行文件中给出的需求要相应对 **硬件/虚拟资源** 进行 **动态绑定和解绑** 。

在内核中，需要有一个进程管理器

**可以运行在用户态、内核态**

三种基本状态：就绪态、运行态、等待态

僵尸进程：进程通过 `exit` 系统调用退出之后，它所占用的资源并不能够立即全部回收。当进程退出的时候内核立即回收一部分资源并将该进程标记为 **僵尸进程** (Zombie Process) 。

**协程**

是程序执行中一个单一的顺序控制流程，建立在线程之上（即一个线程上可以有多个协程），但又是比线程更加轻量级的处理器调度对象。**分时复用的方式运行多个协程**，用户态负责管理和调度，语言层面支持协程，切换在用户态完成，切换的代价比线程从用户态到内核态的代价小很多。有大量IO操作业务的情况下，我们采用协程替换线程，可以到达很好的效果，一是降低了系统内存，二是减少了系统切换开销，因此系统的性能也会提升。**在协程调用阻塞IO操作的时候，操作系统会让线程进入阻塞状态，当前的协程和其它绑定在该线程之上的协程都会陷入阻塞而得不到调度**。

支持异步。

**线程**

**线程是程序执行中一个单一的顺序控制流程，线程是处理器调度和分派的基本单位**。对于线程的调度和管理，可以在操作系统层面完成，也可以在用户态的线程库中完成。如果是在用户态的线程库中完成，操作系统是“看不到”这样的线程的，也就谈不上对这样线程的管理了。**各个线程之间共享进程的地址空间，但线程要有自己独立的栈（用于函数访问，局部变量等）和独立的控制流**。可以抢占式，也可非抢占式。

**进程与线程的切换流程**

进程：特权级切换、异常处理，程序执行的上下文切换、地址映射、地址空间、虚存管理

进程切换分两步：

1、切换**页表**以使用新的地址空间，一旦去切换上下文，处理器中所有已经缓存的内存地址一瞬间都作废了。

2、切换内核栈和硬件上下文。

线程切换只需要第二步。

**为什么虚拟地址空间切换会比较耗时**

地址空间切换就是进程切换，切换过程需要查找页表从虚拟地址到物理地址，


CPU访问数据和指令的内存地址是虚拟地址，硬件机制（MMU内存管理单元+页表查询）完成地址转换找到物理地址。分页内存管理，大量节约了内存占用，需要MMU更多的隐式访存。

所以解决办法就是加入快表，存储虚拟地址和物理地址的键值对。

进程切换需要清空快表，只能查找虚拟页表完成多次访存经过硬件机制得到新的进程的物理地址。所以表现出来就是进程切换比较耗时。

~~进程都有自己的虚拟地址空间，把虚拟地址转换为物理地址需要查找页表，页表查找是一个很慢的过程，因此通常使用Cache来缓存常用的地址映射，这样可以加速页表查找，这个Cache就是TLB（translation Lookaside Buffer，TLB本质上就是一个Cache，是用来加速页表查找的）。~~

~~由于每个进程都有自己的虚拟地址空间，那么显然每个进程都有自己的页表，那么**当进程切换后页表也要进行切换，页表切换后TLB就失效了**，Cache失效导致命中率降低，那么虚拟地址转换为物理地址就会变慢，表现出来的就是程序运行会变慢，而线程切换则不会导致TLB失效，因为线程无需切换地址空间，因此我们通常说线程切换要比较进程切换块，原因就在这里。~~
