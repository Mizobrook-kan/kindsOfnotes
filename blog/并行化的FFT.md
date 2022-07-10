### FFT回顾

先快速回顾一遍FFT的公式。傅里叶变换![{\mathcal {F}}\left(\mathbf {x} \right)](https://wikimedia.org/api/rest_v1/media/math/render/svg/b6e05b13c0b665dcf6b7af0029024a1a1627bc20)满足周期性：

$X_{k+N} \triangleq \sum_{n=0}^{N-1} x_{n} e^{-\frac{i 2 \pi}{N}(k+N) n}=\sum_{n=0}^{N-1} x_{n} e^{-\frac{i 2 \pi}{N} k n} \underbrace{e^{-i 2 \pi n}}_{1}=\sum_{n=0}^{N-1} x_{n} e^{-\frac{i 2 \pi}{N} k n}=X_{k}$。

![{\displaystyle X_{k}=\sum _{n=0}^{N-1}x_{n}e^{-i2\pi kn/N}\qquad k=0,\ldots ,N-1,}](https://wikimedia.org/api/rest_v1/media/math/render/svg/393b5c5a5c668495629828600cde4611b0fa2f5a)

$W_{N} = e^{-\frac{i 2 \pi}{N}}$

相位因子$e^{-i 2 \pi k n / N}$，代入欧拉公式得到$\cos \left(\frac{2 \pi}{N} k n\right)-i \cdot \sin \left(\frac{2 \pi}{N} k n\right)$，观察得到傅里叶变换的周期是N。三角函数同时具有对称性，$X_{k+N/2} = -X_k$。

### 2-FFT


$\sum_{m=0}^{(N / 2)-1} x(2 m) W_{N}^{2 m k}$，$\sum_{m=0}^{(N / 2)-1} x(2 m+1) W_{N}^{k(2 m+1)}$。

$W_{N}^{2}=W_{N / 2}$（直接代入即可验证等式成立），替换后有$\sum_{m=0}^{(N / 2)-1} x(2 m) W_{N/2}^{ m k} + W_{N}^k \sum_{m=0}^{(N / 2)-1} x(2 m+1) W_{N/2}^{km}$，此时周期为$N/2$。（中文版有推导这个2路的时间复杂度，我主要描述算法执行过程。）



### 4-FFT

项目采用HTML5技术设计频谱可视化、可添加声音效果的音乐播放器，模拟均衡器的使用效果

### ~~蝶形网络~~

~~Cooley-Tukey算法使用的是2路蝶形网络。~~


2路蝶形网络：$N=2^n$，运用分治方法，复数加法和复数乘法的数量可以削减到$N(N+6)/2$和$N^2/2$。如果再使用对称性和周期性，复数加法和乘法数量削减到$N\log_2{N}$和$N/2\log_2{N}$。它的计算过程大致是图

$$
\begin{aligned}
&y_{0}=x_{0}+x_{1} \omega_{n}^{k} \\
&y_{1}=x_{0}-x_{1} \omega_{n}^{k}
\end{aligned}
$$

4路蝶形网络：$N=4^n$，复数乘法和加法数量削减到$(3N/8)\log_2{N}$和$(3N/2)\log_2{N}$。
