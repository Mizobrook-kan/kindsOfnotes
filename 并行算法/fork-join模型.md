**fork-join 模型**

在任何时候，你可以从当前的线程T0分出（fork）一些新的线程，当这些线程全部执行完毕，他们可以合并（join）回来，T0接着执行。

~~允许nested parallelism，但不会必然产生nested parallelism，与数据依赖有关。~~

每个fork出来的线程也可以fork出其他线程，但同时需要将这些线程重新合并回来。

并行算法的设计标准，包括粒度（granularity）、

c++的parlay、rust的rayon/tokio，它们的实现原理都是fork-join。每个算法都可以画出DAG，但不是每个算法都能用fork-join模型将其并行化。

fork-join应用的是分治思想，

**算法的DAG**

对于我们所有算法中的操作，根据其依赖关系可以画成一张 DAG（Directed acyclic graph，有向无环图）。其中每一个节点代表一个操作，每一条有向边 A->B 意味着B这个操作必须等待A操作进行完才可以执行。

**前缀和**

*y~i~* = *y*~*i* − 1~ + *x~i，前缀和要求定义二元运算符，满足加法结合律，从WPSD到字符串处理。*

前缀和既可以是有穷序列也可以是无穷序列，从数学的角度看前缀和构成了有穷/无穷序列上的向量空间中的线性映射。

{\displaystyle y_{i}=\bigoplus _{j=0}^{i}x_{j}}![{\displaystyle y_{i}=\bigoplus _{j=0}^{i}x_{j}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/42349bccb33e9cb4a755225108014f3d736c8aed)

inclusive版的前缀和![{\displaystyle y_{i}=\bigoplus _{j=0}^{i}x_{j}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/42349bccb33e9cb4a755225108014f3d736c8aed)，exclusive版的前缀和![{\displaystyle y_{i}=\bigoplus _{j=0}^{i-1}x_{j}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/c8e8f03f127a070cf9d7a08d5520c6463cc3a464)。

**两种版本的DAG**

**实现**
