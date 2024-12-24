---
title: "Deepspeed Zero Note"
description: "deepspeed-zero-note"
keywords: "deepspeed,zero,note"

date: 2024-12-23T01:38:49+08:00
lastmod: 2024-12-23T01:38:49+08:00

categories:
  - DeepSpeed
tags:
  - DeepSpeed
  - Data Parallel

# 原文作者
# Post's origin author name
#author:
# 原文链接
# Post's origin link URL
#link:
# 图片链接，用在open graph和twitter卡片上
# Image source link that will use in open graph and twitter card
#imgs:
# 在首页展开内容
# Expand content on the home page
#expand: true
# 外部链接地址，访问时直接跳转
# It's means that will redirecting to external links
#extlink:
# 在当前页面关闭评论功能
# Disabled comment plugins in this post
#comment:
#  enable: false
# 关闭文章目录功能
# Disable table of content
#toc: false
# 绝对访问路径
# Absolute link for visit
#url: "deepspeed-zero-note.html"
# 开启文章置顶，数字越小越靠前
# Sticky post set-top in home page and the smaller nubmer will more forward.
#weight: 1
# 开启数学公式渲染，可选值： mathjax, katex
# Support Math Formulas render, options: mathjax, katex
math: mathjax
# 开启各种图渲染，如流程图、时序图、类图等
# Enable chart render, such as: flow, sequence, classes etc
#mermaid: true
---

# Data Parallel

假设模型参数量 $\Phi$，数量级 B。共 $N$ 张卡参与训练，因此数据分为 $N$ 份下发到各卡。

1. 每张卡上放一份 模型参数+优化器状态，不妨设其存储开销 $K\Phi$，单位 GB。
2. 分别在 $N$ 张卡 forward 和 backward ，每张卡得到 *自身占有的数据所对应的梯度*，存储开销 $4\Phi$。
3. 将得到的 $N$ 份梯度做 *AllReduce 操作（一种卡间 collective operation）*，每张卡得到 *全部数据所对应的 $N$ 份梯度*
4. 进行 optimizer step 以更新模型参数。

> 整个 Data Parallel 优化的主线：存储、通信开销。其中存储开销来自每张卡上固定需要存储的 模型参数+优化器状态，通信开销来自部分数据 forward 获得的梯度做 AllReduce 操作以获得全部数据做对应 forward 操作的梯度。存储、通信开销的单位均为 GB。

## 混合精度训练

1. 每张卡会放一份 fp32 模型参数+优化器状态，存储开销 $K\Phi$。
2. **将 fp32 参数转换为 fp16 或 bf16 参数**，存储开销为 $2\Phi$。
3. 数据分为 $N$ 份，分别在 $N$ 张卡 forward 和 backward 以**得到 $N$ 份局部 fp16 或 bf16 梯度**，存储开销 $2\Phi$。
4. 对**fp16 或 bf16 梯度**做 **AllReduce 操作**，即可在每张卡上都得到所有数据聚合的**fp16 或 bf16 梯度**。
5. **使用 fp16 或 bf16 梯度**进行 optimizer step 以更新 fp32 模型参数。

> 混合精度训练主要在计算开销和模型精度上做权衡。从 Data Parallel 的角度看，混合精度训练也减少了通信开销，因为产生通信开销的 模型参数+梯度 部分使用了 fp16 或 bf16 低精度项。后文关于各类 Data Parallel 的存储、通信开销分析都是在混合精度训练的背景下完成的。

## DP
在 DP 算法中，混合精度训练流程中的 **AllReduce 操作**基于卡间 **Worker-Server 拓扑结构**直接实现。Worker卡计算 fp16 或 bf16 单卡数据的梯度并上传到Server卡，Server卡聚合并下发全部数据的梯度给Worker卡。Worker单卡通信开销：$2\Phi$。$Server单卡通信开销：$2N\Phi$。这里假设$N$张Worker卡与1张Server卡。Server卡带宽成为瓶颈。

## DDP
在 DDP 算法中，混合精度训练流程中的 **AllReduce 操作**基于卡间 **Ring 拓扑结构**分别实现 ReduceScatter 操作和 AllGather 操作，从而间接实现。循环 $N - 1$ 轮，每张卡从环中前一张卡收到 fp16 或 bf16 单卡数据梯度 的部分聚合结果，聚合上自己的单卡数据梯度，并继续传送给环中后一张卡，通信开销 $(N - 1)\frac{2\Phi}{N}，约为 $2\Phi$。

## Zero Redundancy Optimization (ZeRO) DP 
核心思想是任何参数、梯度、优化器状态都只在需要使用完整值的时候才通过通信构造出完整值，用完后即丢弃完整值，继续保留原属于当前卡的那部分值，以备下一次通信。这大幅减少存储开销，但小幅增加通信开销。

ZeRO DP 默认卡间为 Ring 拓扑结构。 

### Stage 1

1. 每张卡上放 **$\frac{1}{N}$ 份 fp32 模型参数+优化器状态**，存储开销为 $\frac{K\Phi}{N}$。
2. 将 fp32 参数转换为 fp16 或 bf16 参数，存储开销为 $2\Phi$。
3. 数据分为 $N$ 份，分别在 $N$ 张卡 forward 和 backward 以得到 $N$ 份局部数据的 fp16 或 bf16 梯度，存储开销为 $2\Phi$。
4. 对 fp16 或 bf16 梯度做 AllReduce 操作，即可在每张卡上都得到所有数据聚合的fp16 或 bf16 梯度，通信开销 $2\Phi$。只保留 **$\frac{1}{N}$份 fp16 或 bf16 梯度** 。
5. 使用 **$\frac{1}{N}$份 fp16 或 bf16 梯度** 和 **$\frac{1}{N}$份 优化器状态** 进行 optimizer step 以更新 **$\frac{1}{N}$份 fp32 模型参数**。
6. 对 **$\frac{1}{N}$份 fp32 模型参数** 做 AllGather 操作，得到完整 fp32 模型参数，通信开销为 $\Phi$。

需要注意的是分析出的通信开销为 $3\Phi$，但在实际应用中此值为 $2\Phi$。原因如下：理论认为步骤4中没有拆分**fp16 或 bf16 梯度**，所以需要做 AllReduce 操作获取完整梯度；实践中步骤5处只需要**使用$\frac{1}{N}$份 fp16 或 bf16 梯度**参与更新参数，因此只需做 ReduceScatter 操作即可，通信开销 $\Phi$，因此总通信开销为 $2\Phi$。

### Stage 2
1. 每张卡上放 **$\frac{1}{N}$ 份 fp32 模型参数+优化器状态**，存储开销为 $\frac{K\Phi}{N}$。
2. 将 fp32 参数转换为 fp16 或 bf16 参数，存储开销为 $2\Phi$。
3. 数据分为 $N$ 份，分别在 $N$ 张卡 forward 和 backward 以**得到 $N$ 份局部数据的 fp16 或 bf16 梯度**。存储开销为 $\frac{2\Phi}{N}$。
4. 对 **$\frac{1}{N}$份 fp16 或 bf16 梯度** 做 ReduceScatter 操作，即可在每张卡上都得到所有数据聚合的**fp16 或 bf16 梯度**，通信开销 $\Phi$。
5. 使用 fp16 或 bf16 梯度和 **$\frac{1}{N}$份 优化器状态** 进行 optimizer step 以更新 **$\frac{1}{N}$份 fp32 模型参数**。
6. 对 **$\frac{1}{N}$份 fp32 模型参数** 做 AllGather 操作，得到完整 fp32 模型参数，通信开销为 $\Phi$。

总通信开销为$2\Phi$。

> Stage 2从理论上支持Stage 1把 AllReduce 改成 ReduceScatter 的操作。

### Stage 3
1. 每张卡上放 **$\frac{1}{N}$ 份 fp32 模型参数+优化器状态**，存储开销为 $\frac{K\Phi}{N}$。
2. 将 fp32 参数转换为 fp16 或 bf16 参数。每张卡 **只保留$\frac{1}{N}$份 fp16 或 bf16 参数 **。存储开销为 $\frac{2\Phi}{N}$。
3. 对 **$\frac{1}{N}$ 份 fp16 或 bf16 参数** 做 AllGather 操作，得到完整 fp16 或 bf16 参数，通信开销为 $\Phi$。
4. 数据分为 $N$ 份，分别在 $N$ 张卡 forward。完成后立刻把不是自己维护的 fp32 参数丢弃。
5. 对 **$\frac{1}{N}$ 份 fp16 或 bf16 参数** 做 AllGather 操作，得到完整 fp16 或 bf16 参数，通信开销为 $\Phi$。
6. 分别在 $N$ 张卡 backward，得到完整的局部数据的 fp16 或 bf16 梯度。
7. 对 当前卡的 **$\frac{1}{N}$ 份 fp16 或 bf16 梯度** 做 ReduceScatter 操作，得到全局数据聚合的 **$\frac{1}{N}$ 份 fp16 或 bf16 梯度**，通信开销为 $\Phi$。完成后立刻把不是自己维护的 fp16 或 bf16 梯度丢弃。
8. 使用当前卡维护的 **$\frac{1}{N}$ 份 fp16 或 bf16 梯度** 和 **$\frac{1}{N}$ 份 优化器状态** 进行 optimizer step 以更新 **$\frac{1}{N}$ 份 fp32 模型参数**。由于只维护部分参数，因此无需再对参数做 AllReduce 操作。

总通信开销为 $3\Phi$。

## 参考链接
https://blog.csdn.net/dpppBR/article/details/80445569

https://blog.csdn.net/weixin_43336281/article/details/139483368

https://zhuanlan.zhihu.com/p/618865052

<!--more-->
