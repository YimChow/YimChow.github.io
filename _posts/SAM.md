---
layout: post
title: SAM
categories: [分割, 大模型]
description: 关于SAM的那些事，以及如何应用到压缩
keywords: SAM, 压缩
mermaid: false
sequence: false
flow: false
mathjax: true
mindmap: false
mindmap2: false
---
参照：[A Comprehensive Survey on Segment Anything Model for Vision and Beyond](https://arxiv.org/pdf/2305.08196.pdf)

# SAM为什么很重要？

当今AI领域已经步入了通用智能时代（AGI），它是指人工智能系统能够执行广泛的任务，并表现出与人类相似的智能水平。正因此，迫切需要设计一类通用的模型（称之为Foundation models，基础模型），这类模型在大数据集中进行训练，并且可以适应各种下游任务。在NLP领域，有了GPT-4作为基础模型。而在CV领域，最近提出的Segment Anything Model（SAM）模型在打破分割边界方面取得了重大进展，推动了计算机视觉领域基础模型的发展。

# 什么是基础模型？

基础模型是一种新的构建AI系统的范式，这种范式可以适应多种多样的下游任务，因为这种基础模型是在大量数据下训练，通常使用自监督学习技术。这可以允许它们学习通用的表征和能力，这种表征能够转变为不同的域和不同的应用。

这种基础模型的概念发展可以追溯到在NLP领域自监督学习和深度学习的兴起，这使得从原始文本中学习到强大的表示成为可能。比如BERT，T5，GPT系列。

在CV领域中，最近的基础模型尝试利用大语言模型（LLM），并且在从多样的、大规模的图像文本数据中学习广泛的视觉表征上展现了强大的表现力。比如CLIP, ALIGN, Florence, VLBERT, X-LXMERT, DALL-E，尝试着捕捉视觉文本之间跨模态的交互。

# 什么是SAM模型

SAM模型（Segment Anything Model）是Meta在2023年提出的一个分割一切（SA）的项目。在项目中，研究者想要提出能够一统分割任务的单个模型，因此，研究者将任务分成了三步：Task, Model, Data

![SAM](/images/assets/SAM_overview.png)

### Task

SA项目最重要的目标是提供一个可以用广泛的功能，并且能迅速应用到多个已有的和新的分割任务当中的模型，并且可以零样本地迁移到新的数据分布和任务当中。因为许多复杂的功能可以理解为许多简单的现有工具的组合。

### Model

![SAM_model](/images/assets/SAM_model.png)

模型包含三部分：强大的图片编码器，一个prompt编码器以及一个掩码解码器。

### Data

数据驱动，分为人工、半自动、全自动三部分

# SAM模型当下的一些应用

在大模型时代，重复的做一些大模型能干碎的任务无疑是螳臂当车的。所以对于大模型更重要的是如何利用它强大的能力来做一些下游任务。

### 图像编辑

![IA](/images/assets/SAM_IA.png)

Inpaint Anything（IA）解决了修复相关的问题，通过将SAM和当下SOTA的生成模型结合。

![EE](/images/assets/SAM_EE.png)

相似的idea出现在Edit Everything，使用CLIP来进行提示

### 风格迁移

![A2A](/images/assets/SAM_A2A.png)

风格迁移旨在将风格从指定图像上迁移到另外的内容图上。
