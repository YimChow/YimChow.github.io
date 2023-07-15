# High-Fidelity Generative Image Compression

### Abstract

将GAN和学习压缩结合得到SOTA的生成有损压缩系统。

与之前工作的不同：

1. 重构是visually pleasing，在感知上和输入相同。
2. 范围很广的比特率，
3. 方法可以应用于高分辨率图像

### Intro

之前工作中压缩因子越高，图像质量退化越明显。

不同的度量会产生不同的缺点。

rate-distortion-perception trade-off：更好的感知质量往往会导致更差的失真率。

贡献：

1. 提出生成压缩方法来达到和原图接近的高质量的重构
2. 用FID，KID，NIQE，LPIPS以及传统的PSNR，MS-SSIM等指标评估方法。
3. 研究了提出框架和组成部分

### Method

#### 背景

##### 条件GAN

##### 神经图像压缩

#### 公式和优化

$$
\mathcal L_{EGP}=\mathbb E_{x\sim p_X}[\lambda r(y)+d(x,x^{'})-\beta\log(D(x^{'},y))],\\
\mathcal L_D=\mathbb E_{x\sim p_X}[-\log(1-D(x^{'},y))]+\mathbb E_{x\sim p_X}[-\log (D(x,y))].
$$

在本文的设置中，有两项和比特率不一致。对于固定的$\lambda$，不同的其他超参会导致不同的比特率，导致比较的困难。为了缓解这一现象，使用了超参“目标码率”$r_t$，并且替换掉$\lambda$，如果$r(y)>r_t$，则$\lambda=\lambda^{(a)}$，否则$\lambda=\lambda^{(b)}$。设定$\lambda^{(a)}\gg\lambda^{(b)}$

#### 网络结构

![image-20230605214236952](C:\Users\Yimin\AppData\Roaming\Typora\typora-user-images\image-20230605214236952.png)

用SpectralNorm替换InstaceNorm

为了缓解darkening artifacts，使用channelNorm，在通道间进行归一化。

#### 用户调研

### Experiments

### Results

