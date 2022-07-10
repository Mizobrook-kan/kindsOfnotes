#### 前言

模拟信号，数字信号

*当一个信号经过一个系统(如滤波）时，我们就说已经对该信号进行了处理。在这种情况下，
对信号进行处理的含义是从有用信号中对噪声和干扰信号进行滤波。一般来说，系统由对信号所
执行的操作所表征。例如，如果操作是线性的，那么系统就称为线性的。如果对信号的操作是非
线性的，那么系统就称为非线性的，等等。这类操作通常指信号处理。*

~~为什么需要信号处理？信号处理涉及信号表示、信号变换、信号操纵。例如，分离两个或多个信号，可以执行的操作包括addition、multiplication或convolution，或者增强一些信号分量，或者估计某些参数。~~

~~模拟信号：https://en.wikipedia.org/wiki/Analog_signal~~

~~信号是由一个或几个自变量的函数所描述的。函数值（即因变量）可
以是~~实值标量、复值量或矢量。~~例如，信号~~

~~实值信号、复数值信号、矢量信号~~

~~连续时间信号$x_{a}(t)=A \cos (\Omega t+\theta), \quad-\infty<t<\infty$.~~

连续时间正弦信号：$x_{a}(t)=A \mathrm{e}^{\mathrm{j}(\Omega t+\theta)}$。$\Omega=2 \pi F$。代入欧拉公式$\mathrm{e}^{\pm \mathrm{j} \phi}=\cos \phi \pm \mathrm{j} \sin \phi$，我们可以得到

离散时间信号正弦形：

$x(n)=A \cos (2 \pi f n+\theta), \quad-\infty<n<\infty$。该离散信号的周期为$N$，当且仅当存在一个整数$k$满足$f=\frac{k}{N}$。

离散时间信号复指数形：~~当$f_0$是有理数的时候，复指数的周期为$N$，有$s_{k+N}(n)=\mathrm{e}^{\mathrm{j} 2 \pi n(k+N) / N}=\mathrm{e}^{\mathrm{j} 2 \pi n} s_{k}(n)=s_{k}(n)$，这表明该组周期复指数信号具有$N$个不同的周期复指数序列，组中的所有成员均具有$N$个样本的公共周期。选取任何连续$N$个复指数，即从~~

谐相关复指数定义为$s_{k}(n)=\mathrm{e}^{\mathrm{j} 2 \pi k n / N}, \quad k=0,1,2, \cdots, N-1$

$x(n)=\sum_{k=0}^{N-1} c_{k} s_{k}(n)=\sum_{k=0}^{N-1} c_{k} \mathrm{e}^{\mathrm{j} 2 \pi k n / N}$，其中傅里叶系数$c_{k}, \quad k=0,1,2, \cdots, N-1$是任意的复常数，这个形式是傅里叶级数展开。序列$s_{k}(n)$称为$x(n)$的第$k$次谐波。基础频率是$\frac{k}{N}$。

谐相关的复指数信号（harmonically related complex exponentials）：这种信号是一组周期复指数信号，其基础频率（fundamental frequency）是单个正频率的倍数。

#### LTI系统

~~给定离散时间信号$x(n)$，对于所有整数$n,-\infty<n<\infty$。即$x(n)$不是通过采样模拟信号得到的。如果$x(n)$是从模拟信号$x_{a}(t)$采样得到的，那么$x(n) \equiv x_{a}(n T)$，$T$是两个连续样本间的间隔时间，即采样周期。~~

~~离散时间信号~~

~~$x(n)$视为信号的第$n$个样本，。~~

离散时间信号的分类：~~信号的能量$E$定义为$E \equiv \sum_{n=-\infty}^{\infty}|x(n)|^{2}$，~~

能量信号：$E \equiv \sum_{n=-\infty}^{\infty}|x(n)|^{2}$，如果$E$有限，那么$x(n)$称为能量信号。

功率信号：$P=\lim _{N \rightarrow \infty} \frac{1}{2 N+1} \sum_{n=-N}^{N}|x(n)|^{2}$，如果$P$有限，那么是功率信号

周期信号：$x(n+N)=x(n) $，对于所有$n$。最小值$N$称为（基本）周期。

非周期信号：如果没有满足周期公式的$N$值，那么是非周期信号。

对称信号：实信号，$x(-n)=x(n)$。

反对称信号：$x(-n)=-x(n)$。

**时不变**：如果一个系统是时不变的，那么对任意时间平移量$k$，$x(n-k) \stackrel{\mathcal{J}}{\longrightarrow} y(n-k)$。

**~~时不变：一个弛豫系统$T$是时不变或者平移不变系统，当且仅当$x(n) \stackrel{\mathcal{T}}{\longrightarrow} y(n)$。测试方法，~~**

**线性：一个系统是线性的，当且仅当对任意输入序列$x_{1}(n)$和$x_{2}(n)$，以及任意常数$a_{1}$和$a_{2}$，有$\mathcal{T}\left[a_{1} x_{1}(n)+a_{2} x_{2}(n)\right]=a_{1} \mathcal{T}\left[x_{1}(n)\right]+a_{2} \mathcal{T}\left[x_{2}(n)\right]$（两个系统的叠加）。**

**~~因果：一个因果系统应该满足$y(n)=F[x(n), x(n-1), x(n-2), \cdots]$，即一个系统在任意时刻$n$的输出仅依赖于当前和过去的输入，而与将来的输入无关。~~**

**~~稳定：一个系统称为有界输入-有界输出（BIBO）稳定，当且仅当每个有界输入产生有界的输出。存在有限数$M_{x}$和$M_{y}$，有$|x(n)| \leqslant M_{x}<\infty$与$\quad|y(n)| \leqslant M_{y}<\infty$。~~**

~~如果线性系统是延迟响应（delayed impulse）的线性组合$x[n]=\sum_{k=-\infty}^{\infty} x[k] \delta[n-k]$，这种线性系统完全由延迟响应（impulse reponse）刻画。特别地，令$h_{k}[n]$为$\delta[n-k]$的系统响应，$n=k$为延迟出现的位置。那么输出序列$y[n]=T\left\{\sum_{k=-\infty}^{\infty} x[k] \delta[n-k]\right\}$，根据叠加原则，我们可以改写成$y[n]=\sum_{k=-\infty}^{\infty} x[k] T\{\delta[n-k]\}=\sum_{k=-\infty}^{\infty} x[k] h_{k}[n]$。~~



~~两种分析线性系统对给定输入信号反应或响应的方法，一种是差分方程$\sum_{k=0}^{N} a_{k} y[n-k]=\sum_{m=0}^{M} b_{m} x[n-m]$，第二种方法是~~


~~时域特性：~~

~~卷积（convolution）公式：~~

~~差分（difference）方程：~~

~~自相关（autocorrelation）：~~

~~互相关（crosscorrelation）：~~
