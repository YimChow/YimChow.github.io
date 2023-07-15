# Tell Me What Happened: Unifying Text-guided Video Completion via Multimodal Masked Video Generation

### Abstract

背景：给定前几个静态帧的情况下，生成一个视频是具有挑战性的，因为它预测了具有时间一致性的合理的未来帧。除了视频预测，从最后一帧进行回卷或在头尾之间进行填充的能力也至关重要，但它们很少被用于视频补全。由于只有几帧的提示，最终可能得到不同的结果。因此一个能够遵循自然语言进行视频补全的系统可能会显著的提高可控性。

引入了新的任务：文本指导的视频补全(TVC)

本文工作：提出多模型掩码视频生成(MMVG)来解决TVC任务

训练阶段：MMVG将视频帧离散化为视觉的token并且掩码大部分来在任意时间点执行视频补全。

推理阶段：单个MMVG模型可以解决3种TVC，包括视频预测，回卷和填充。

### Intro

只有前一帧不足以生成，而对人来说，语言又是最直接的交流方式。

相比于视频预测，视频回卷和填充很少被研究。

提出了多模态掩码视频生成。具体而言，通过时序感知的VQGAN将视频帧表示为离散的视觉令牌。

不同于自回归模型，MMVG用编码器解码器的方法执行视频补全。具体而言，提出了掩码机制来掩盖视频的不同区域，并且将其作为输入和指令一起反馈给多模态编码器。

### Text-guided Video Completion(TVC)

#### 任务的定义

三类任务：预测（从前一帧补全）、回卷（从后一帧补全）、填充（从头尾之间补全），并且在文本的限制下。

在训练过程中，定义视频对$\mathcal V$，以及对应的文本指导$\mathcal X$，其中，$\mathcal V$包含$N$帧，即$\{v_1,\cdots,v_N\}$。目标是训练一个统一的模型，可以生成补全的$\mathcal V$，在给定任意时间的部分帧以及$\mathcal X$的情况下。

#### 多模态掩码视频生成

##### Overview

![image-20230519164722182](./assets/image-20230519164722182.png)

为了建模视频和语言，提出了时间感知的VQGAN来将一帧表示为视觉符号，将其转换为与文字相同的离散空间。

掩码方法：missing fragment被独特的[SPAN]标志替代，并且在多个时间点考虑视觉指导。

多模态的编码器消耗文本和部分缺失的视频，解码器学习从任意引导帧种生成完整的视频。

##### 时间感知的离散视觉符号

VQGAN进一步通过一个GAN训练的Transformer模块来建模潜在空间的先验分布。

如果VQGAN直接被应用在视频上，就可能忽视了帧之间时间上的相关性，将每一帧视为独立的图像，从而导致视频重建不流畅。

为了灵活的解决，本工作提出了时序感知的VQGAN来将时序联系注入到潜在表示中。首先遵循VQGAN通过重构视频帧$v_i$来学习没目标视觉标记$z_i$
$$
\begin{align}
z_i&=q(\text{Enc}^Q(v_i)|C)\\
v_i&=\text{Dec}^Q(z_i)\\
\mathcal L_{VQ}&=||\hat v_i-v_i||_1+||sg[\text{Enc}^Q(v_i)]-C_{z_i}||^2_2\\
&+\beta||sg[C_{z_i}]-\text{Enc}^Q(v_i)||^2_2+||\mathcal F(\hat v_i)-\mathcal F(v_i)||_1
\end{align}
$$
$q$是量化操作，采取在可训练码本$C$中选取最近邻得到。$sg$为stop-gradient操作。

为了在$z$中注入时间关系，T-VQ使用引入的对比时序推理进行训练。
$$
\begin{align}
o_i&=\text{FC}^\text{T}(z_i,z_j)\\
\mathcal L_T&=\text{BCELoss}(o_i, 0\text{ if }i>j\text{ else }1)
\end{align}
$$
其中$j$是相同视频中随机抽取的一帧，FC^T是MLP分类器，BCELoss是二值交叉熵。**从$\mathcal L_T$中可以学习到时序信息。**$z$中暗含了时间的相关性。

> 如何得到时序信息？从深度学习网络的loss中去隐含，将这个处理交给黑盒

##### 从掩码视频中生成

$\mathcal M$掩盖了大多数的视频帧，以概率$p$，并且替换为一个独立的[SPAN] token，得到掩码后的视频$\mathcal {\bar V}$。本文的目标是在$\mathcal X$的指导下恢复缺失的区域。

为了在视觉和语言模态之间进行建模，将Enc^Q应用于离散视觉标记，使用CLIP tokenize $\mathcal X$为word tokens $\{w_i\}^L_{i=1}$
$$
\begin{align}
f_i^w,f_j^v&=\text{LP}^w(w_i),\text{LP}^v(z_j)\\
\{h\}&=\text{Enc}^M([\{f^w\},\{f^v\}])
\end{align}
$$
LP为linear projection（线性投影）
$$
\begin{align}
\hat z_t&=\text{Dec}^M(\{\hat z_1,\cdots,\hat z_{t-1}\}|\{h\})\\
\mathcal L_t&=\text{CELoss}(\hat z_t, z_t)\\
\mathcal L_M&=\sum_{t=1}^N\mathcal L_t
\end{align}
$$
解码器基于VideoSwin建造。

为了使$\mathcal M$更加高效，使用一个自适应的概率$p$而不是随机采样
$$
p_t=p_t+\alpha((\frac{\mathcal L_t}{\mathcal L_M}\sum p)-p_t)
$$

##### 在推理时使用统一的TVC

prediction: $[\{w\},\{z_1,[\text{SPAN}]\}]$

rewind: $[\{w\},\{[\text{SPAN}],z_N\}]$

infilling: $[\{w\},\{z_1,[\text{SPAN}],z_N\}]$

#### MMVG的学习

首先训练T-VQ，然后训练MMVG

### 一些思考：生成和压缩的一些共通点

###### 这篇文章有压缩吗？

从损失函数的角度来看，没有加入比特率作为损失，也就是说，在训练的时候并没有关心潜在表示中压缩的大小，本文的目标依然是能生成更好的视频帧

###### 那与压缩的关系在哪？

跟压缩有关的就是量化操作。本文采用软量化（soft-to-hard vq)

