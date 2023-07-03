---
layout:     post
title:      "[Ebor] Deep Learning Practice in Multi-Factor Modeling 多因子深度学习实践整理"
subtitle:   " \"Every little bit helps\""
date:       2023-07-01 12:00:00
author:     "Yibo Li"
header-img: "img/post-bg-2015.jpg"
mathjax: true
catalog: true
tags:
    - Deep Learning
    - Quant
---

# [Ebor] Issues About Deep Learning Practice in Quant Modeling

2023-06-22

Yibo Li

yli12@stevens.edu; yiboli@link.cuhk.edu.hk

http://eborlee.github.io

## Table Of Contents

* TOC
{:toc}

### 1. Methods of calculating feature importances in deep learning models

1. 中信的做法：*《多因子量化选股系列专题研究—多视角理解基于相关性的因子优化量化与配置》*
<img src="https://github.com/eborlee/eborlee.github.io/blob/main/docImg/1.png?raw=true" alt="image-20230424134815699" style="zoom: 33%;" />

2. Integrated Gradients

```python
import numpy as np

# 定义ReLU激活函数
def relu(x):
    return np.maximum(0, x)

# 定义线性层
def linear_layer(x, weights, bias):
    return np.dot(x, weights.T) + bias

# 定义梯度计算函数，这里简化了ReLU的梯度计算
def gradient(x, weights):
    return weights if x > 0 else 0

class SimpleModel:
    def __init__(self):
        # 初始化第一层和第二层的权重和偏置
        self.weights1 = np.array([[-4, -3, -2], [-1, 0, 1], [2, 3, 4]])
        self.bias1 = np.array([0, 0, 0])
        self.weights2 = np.array([[-3, -2, -1], [1, 2, 3]])
        self.bias2 = np.array([1, 1])

    def forward(self, x):
        # 前向传播过程
        self.x1 = linear_layer(x, self.weights1, self.bias1)  # 第一层线性变换
        self.x2 = relu(self.x1)  # 第一层激活函数
        self.x3 = linear_layer(self.x2, self.weights2, self.bias2)  # 第二层线性变换
        return self.x3  # 返回网络输出

    def integrated_gradients(self, x, baseline, steps=50):
        # 计算从baseline到输入x的差值
        x_diff = x - baseline
        # 生成从baseline到x的等差序列，用于后续的积分计算
        step_inputs = np.array([baseline + x_diff * i / steps for i in range(steps + 1)])
        # 计算每一步的梯度
        step_gradients = np.array([gradient(self.forward(step_input), self.weights1) for step_input in step_inputs])
        # 计算平均梯度
        avg_gradients = np.average(step_gradients, axis=0)
        # 计算积分梯度（Integrated Gradients）
        integrated_gradients = x_diff * avg_gradients
        return integrated_gradients

# 使用示例
model = SimpleModel()
input = np.array([1, 2, 3])
baseline = np.array([0, 0, 0])
ig_attributions = model.integrated_gradients(input, baseline)
print('Integrated Gradients Attributions:', ig_attributions)

```

1

### 2 Batch-Normalization层对时间序列数据整批处理导致引入未来数据的问题

> **Ebor:**
>
> 使用lstm模型时，对数据进行batch normalization是否会导致引入未来数据？这里未来数据的影响对预测有何不良影响？
>
> 
>
> **ChatGPT:**
>
> 在时间序列分析中，使用Batch Normalization（BN）需要谨慎，因为BN在计算过程中使用了整个批次的数据，这可能会导致数据泄露，即在处理当前时间步的数据时引入了未来的信息。**<u>这是因为BN在计算每个特征的均值和方差时，使用的是整个批次的数据，包括当前时间步之后的数据。</u>**
>
> 为了避免这种情况，你可以使用其他的标准化方法，如Layer Normalization或Instance Normalization，这些方法只在单个数据样本的范围内进行标准化，不会引入其他样本的信息。另外，你也可以考虑使用滑动窗口或指数平滑等方法来计算均值和方差，这些方法只使用当前和过去的数据，不会引入未来的信息。
>
> 总的来说，如果你在处理时间序列数据时使用Batch Normalization，你需要确保你的实现方式不会引入未来的信息。
>
> 问题在于，如果我们在训练阶段使用了未来的数据（即，使用了整个批次的数据来计算BN的均值和方差），那么**<u>模型可能会学习到一些在预测阶段无法使用的模式或依赖关系。这可能会导致模型的预测性能下降。</u>**
>
> 因此，虽然在预测阶段我们不会使用未来的数据，但在训练阶段使用未来的数据仍然可能会导致问题。这就是为什么在处理时间序列数据时，我们需要谨慎使用Batch Normalization的原因。

