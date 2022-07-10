### 基础术语

Sample rate

bit depth

number of channels

decibel

Amplitude modulation

#### 其他

数据压缩，如果有数据压缩，解码器必须将数据转换回时域才能开始播放。

现在的有损压缩技术使用心理声学的知识减小音频损失。

数据压缩也引入了另一个音频数据的特征，bit rate。

有损压缩的比特率大幅降低了，比如mp3和aac，30~500kbps

#### STFT（short- time Fourier transform）

$\operatorname{STFT}\{x[n]\} \equiv X[m, k]=\sum_{n=m R}^{N+m R-1} x[n] \mathrm{e}^{-j(n-m R) k 2 \pi / N} \quad 0 \leq k \leq N-1$。spectrogram。

DFT中的magnitude和相位可以表示频率，这是一种表示。另一种表示是power spectrum，$P(k)=|X(k)|^{2} / N^{2}$。

#### aliasing

#### 修改和处理数字信号

引入difference equation，计算第n个输出样本的一个公式。

线性时不变的数字滤波器，differnece equation是$y[n]=b_{0} x[n]+b_{1} x[n-1]+\ldots b_{N} x[n-N]-a_{1} y[n-1]-\ldots a_{M} y[n-M]$。

另一种表示，混合响应（impulse response），
