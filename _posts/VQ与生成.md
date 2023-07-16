

---
layout: post
title: VQ与生成
categories: [VQ, 生成模型]
description: 简单了解下VQ方法，包括VQVAE，VQGAN等
keywords: VQVAE, VQGAN 
mermaid: false
sequence: false
flow: false
mathjax: true
mindmap: false
mindmap2: false
---

# VQ与生成

## VQVAE

### VAE

[变分自编码器VAE：原来是这么一回事 | 附开源代码 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/34998569)

变分自编码器。它的目标和GAN一致，希望构建一个从隐变量$$Z$$生成目标数据$$X$$的模型。

$$z=E(x), \hat x=D(z)$$

而变分意思是$$z$$满足各项同性的高斯分布（先验），这样训练结束后，可以从这个先验上采集$$z$$，只是用decoder生成

从概率的角度：

让encoder去学习一个条件分布$$q_\phi(z|x)$$，让decoder去学习型另一个条件概率$$p_\theta(x|z)$$，同时$$z$$服从先验分布$$p(x)$$，所以loss函数可以写作：

$$
ELBO(\theta,\phi)=\mathbb E_{z\sim q_\phi(z|x)}[\log p_\theta(x|z)]-KL(q_\phi(z|x)||p(z))
$$

ELBO即evidence low bound，即evidence（$$x$$）的最小期望

生成模型的难点在于判断生成分布和真实分布的相似度。

假设隐向量服从正态分布，从中采样出$$Z$$，然后生成出$$\hat X$$，怎么判断$$X$$和$$\hat X$$的分布是一致的？目前通过KL散度是不可行的，因为KL散度是根据两个概率分布表达式来计算相似度的，然而现在并不知道$$X$$和$$\hat X$$的概率分布表达式。也就是说，现在我们有的仅仅是$$X=\{x_1,\cdots,x_n\}$$和$$\hat X=\{\hat x_1,\cdots,\hat x_n\}$$

在解决这个问题上，VAE和GAN采取了不同的思路。

GAN认为，我们现在没有这样的度量，那我就训练出这样的度量出来。GAN说要有度量，于是discriminator诞生了。

VAE则采用迂回的方式：

$$p(X)=\sum_Zp(X|Z)p(Z)$$；假设$$Z$$服从正态分布，根据$$Z$$来计算$$X$$

然而在整个VAE模型中，并没有使用先验分布$$p(Z)$$，而是使用后验分布$$p(Z|X)$$，假设$$p(Z|X)$$是正态分布。因为如果仅仅是假设先验分布是正态分布，从$$p(Z)$$中采样出一个$$z_i$$，我们并不知道这个$$z_i$$对应哪个真实的$$x_i$$，但是$$p(Z|x_k)$$一定是专属于$$x_k$$的

于是使用神经网络去拟合分布的均值和方差，能够从这个专属分布中采样出$$z_k$$出来，经过生成器后得到$$\hat x_k=g(z_k)$$，然后最小化$$D(\hat x_k,x_k)$$

> 问题：
>
> 1. 重构过程受到噪声影响，但如果方差为0，就完全没有随机性，vae会退化成普通的ae

$$
p(Z)=\sum_Xp(Z|X)p(X)=\sum_X\mathcal N(0,I)p(X)=\mathcal N(0,I)\sum_Xp(X)=\mathcal N(0,I)
$$

如果所有的$$p(Z|X)$$都很接近标准正态分布，那么$$p(Z)$$也同样是正态分布，所以可以放心地从 $$\mathcal N(0,I)$$采样。

为了使模型具有生成能力，VAE要求每个$$p(z_i|x_i)$$都要向正态分布看齐

如果看齐？使用loss

$$
\mathcal L_{\mu,\sigma^2}=\frac{1}{2}\sum_{i=1}^d(\mu_{(i)}^2+\sigma_{(i)}^2-\log\sigma_{(i)}^2-1)
$$

重参数技巧：在$$\mathcal N(\mu,\sigma^2)$$中采样一个$$Z$$，相当于从$$\mathcal N(0,I)$$中采样一个$$\epsilon$$，然后令$$Z=\mu+\epsilon\times\sigma$$

KL散度：

$$
KL(p(x)||q(x))=\int p(x)\ln\frac{p(x)}{q(x)}dx
$$

### VQ-VAE

##### 推理

相比vae最大的改变是，vae中隐变量$$z$$的每一维都是连续的值，而vq-vae中$$z$$的每一维之间都是离散。这样做是为了符合自然界的模态。所以vq-vae可以成功的建模重要特征

vq的过程首先有一个codebook，在这个embedding table中寻找和这个向量最近的，用这个index来代表这个向量。

1. codebook是一个$$K\times D$$的table
2. 将一张图片encode后得到一个$$H\times W\times D$$的特征图$$z_e(x)$$
3. 将这$$H\times W$$个$$D$$维向量分别去找codebook中最近的$$e_i$$，用index表示，得到$$q(z|x)$$
4. 将$$z_e(x)$$用codebook里最近的$$e_i$$替换后，可以得到$$z_q(x)$$，进入解码器重构

$$q(z|x)$$中每个数字都是离散的整数，可以把这个数字写成独热码，从而可以看作概率分布，总共有$$K$$维，每一维代表codebook中的$$e_i$$的概率，从VAE的角度，给这个$$K$$维分布一个均匀分布作为先验，即$$p_i=\frac{1}{K}$$，从而$$KL(q_\phi(z|x)||p(z))$$这一项就变为常数$$\log K$$

##### 训练

VQ-VAE可以训练的参数有三部分：encoder，decoder，codebook

损失函数：

$$
\mathcal L=\log p(x|z_q(x))+||sg[z_e(x)]-e||^2_2+\beta||z_e(x)-sg[e]||^2_2
$$

sg为stop gradient，即梯度反传到此为止，不再往前传。

第一项是用来训练encoder和decoder的。在梯度反传的时候，$$z_q$$的梯度直接给了$$z_e$$

第二项叫codebook loss，只训练codebook，目的是让codebook和embedding向各自最近的$$z_e(x)$$靠近

第三项叫commitment loss，只训练encoder，目的是鼓励encoder的输出尽可能接近选择的码本向量，防止在不同的codebook embedding之间反复横跳

如何让各个$$z_i$$之间有关系：使用PixelCNN，对$$z_i$$建立一个自回归模型，即

$$p(z_1,z_2,z_3\cdots)=p(z_1)p(z_2|z_1)p(z_3|z_1,z_2)\cdots$$

## VQGAN

#### 离散编码特征表示方法

$$
z_q=q(\hat z)=(\arg\min_{z_k\in\mathcal Z}||\hat z_{ij}-z_k||)\in\mathbb R^{h\times w\times n_z}
$$

训练方法：

$$
\mathcal L(E,G,Z)=||x-\hat x||+||sg[E(x)]-z_q||^2_2+\beta||sg[z_q]-E(x)||^2_2
$$

第一项为重建损失，第二项和第三项分别训练了codebook和encoder

判别器：

$$
\mathcal L_{GAN}(\{E,G,Z\},D)=[\log D(x)+\log(1-D(\hat x))]
$$

#### Tranformer自回归式生成

采用GPT-2，本来是用来处理语言模型的，迁移到VQGAN中，就是先预测出一个code，再一步步通过已经预测好的code去推断下一个code（这里的code都是指codebook中的）

code的预测过程可以被视作自回归预测：当已有编码$$s_{<i}$$后，Transformer试图去学习预测下一个编码，即预测分布$$p(s)=\prod_i p(s_i|s_{<i})$$，这就可以表示为最大化对数似然分布：

$$
\mathcal L_\text{Tranformer}=\mathbb E_{x\sim p(x)}[-\log p(s)]
$$
