---
title: Foundations of Convolutional Neural Networks
toc: true
type: docs
date: "2020-01-04T00:00:00+01:00"
draft: false

menu:
  deep_learning:
   parent: Convolutional Neural Networks
   weight: 2
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 1
---

## Overview

一般计算机视觉问题包含以下几类:

1. 图像分类(Image classification)
2. 目标检测(Object Detection)
3. 风格转换(Neural Style Transfer)

面临计算机视觉问题时，如何处理图像数据变得很重要，对于一般图片 $1000 \times 1000 \times 3$, 神经网络的输入层将有 $300,000,000$ 数据, 那么网络权重 $W$ 则很大，这会导致两个后果:

1. 数据量很少，但是神经网络很复杂，容易出现过拟合
2. 所需要的内存和计算量很大

## 卷积运算

神经网络从浅层到深层，可以检测出图片的边缘特征，局部特征，最后一层则识别整体的面部轮廓， 这些都是通过卷积神经网络实现的

### 边缘检测

垂直检测(Vertical Edges) and 水平检测(Horizontal Edges), 如下图所示:

<img src="/img/cnn/edge_example.png" width="350px" />

边缘检测可以通过相应的 filter 实现，如下图所示

<img src="/img/cnn/Vertical-Edge-Detection.jpg" width="350px" />
原始图片尺寸为 6x6, 通过中间的矩阵之后，尺寸为 4x4. 按照如图所示的方式进行计算，卷积运算的求解是在原始矩阵中选择与 filter 相同大小的区域，一一对应相乘后求和，将所得的结果构成一个矩阵

- 下面是一个示例
  <img src="/img/cnn/Convolutional-operation-example.jpg" width="350px" />
  将矩阵看做一个图像，最右边中间的亮的区域对应于最左边图像中间的黑白分界线  
  下面一张图，表示的更加明显
  <img src="/img/cnn/Convolutional-operation.jpg" width="350px" />

- vertical filter and horizontal filter
  <img src="/img/cnn/Vertical-and-Horizontal-Filter.png" width="350px" />
- Sobel filter and Scharr filter, 增加了中间行的权重， 可以提高结果的稳定性
  <img src="/img/cnn/Sobel-Filter-and-Scharr-Filter.png" width="350px" />
- filter 中的值可以设置为参数，通过模型训练得到，神经网络通过反向传播学习到一些低级特性，使得可以对图像的边缘进行检测，不仅仅局限于水平和垂直方向

## Padding

假设输入图像大小为 $n\times n$, filter 大小为 $f\times f$, 则输出图片大小为 $(n-f+1) \times (n-f+1)$

因此，存在两个问题

1. 卷积运算后，图像的尺寸会减小
2. 原始图片的角落，边缘地区的输出采样很少，因此很多边缘地区的信息被丢失

一般可以使用 padding 方式，对图像进行填充, 通常填充的值为0
  <img src="/img/cnn/Padding.jpg" width="350px" />
如果对每个方向的padding 数为 p, 则padding后的图像大小 $(n+2p) \times (n+2p)$, 通过 $f\times f$的filter 之后， 大小则为 $(n+2p-f+1)\times (n+2p-f+1)$  

进行卷积操作时，有两种选择  

  - valid 卷积： 没有 padding, 直接卷积， $(n-f+1) \times (n-f+1)$
  - same 卷积，进行 padding, 使得卷积后结果大小与输入大小一致，此时有 $p = \frac{f-1}{2}$

f 通常为奇数， 原因包括 same 卷积 中可以得到整数，并且 filter 有一个可以表示其所在位置的中心点

## Stride

通过设置步长来压缩一些信息， 步长表示 filter 在原始图像的水平方向和垂直方向上每次移动的距离。步长设置为2的卷积过程表示为
  <img src="/img/cnn/Stride.jpg" width="350px" />

设步长为 $s$, 填充长度为 $p$, 输入图片大小为 $n\times n$, filter 大小为 $f\times f$, 则卷积后的图片大小为 $$\left\lfloor\frac{n+2 p-f}{s}+1\right\rfloor \times\left\lfloor\frac{n+2 p-f}{s}+1\right\rfloor$$
向下取整表示原始矩阵的蓝框完全包含在图像内部时，才进行计算.

目前为止我们学习的“卷积”实际上被称为互相关（cross-correlation），而非数学意义上的卷积。真正的卷积操作在做元素乘积求和之前，要将滤波器沿水平和垂直轴翻转（相当于旋转 180 度）

## 高维卷积

如果对三通道的RGB进行卷积运算，对应的 filter 也是三通道的，这有点类似我们的眼镜，是 see through 这三个维度，
对每个通道 (R, G, B) 与对应的 filter 进行卷积运算求和，然后将三个通道的值相加，如下图所示，是对 27 个乘积和作为一个输出

  <img src="/img/cnn/Convolutions-on-RGB-image.png" width="400px" />

如果想进行更多的边缘检测，可以增加 filter 的数量，如下图所示

  <img src="/img/cnn/More-Filters.jpg" width="400px" />

设输入图片的尺寸 $n \times n \times n_c$ , filter 的尺寸为 $f\times f \times n_c$, 则卷积运算后的图片尺寸为 $(n-f+1) \times (n-f+1) \times n_c'$,   $n_c'$ 为 filter 的数目

## 单层卷积神经网络
  <img src="/img/cnn/One-Layer-of-a-Convolutional-Network.jpg" width="400px" />
  单层卷积神经网络多了激活函数和偏移量，filter 的数值对应 $W^{[l]}$, 卷积运算对应 $W^{[l]}$ 与 $A^{[l-1]}$的乘积
  $$\begin{array}{c}Z^{[l]}=W^{[l]} A^{[l-1]}+b \\\\ A^{[l]}=g^{[l]}\left(Z^{[l]}\right)\end{array}$$

  对于 $3 \times 3$的 filter, 总共有 28 各参数，不予图片尺寸相关，因此 CNN 的参数相较于标准神经网络要小很多

### 符合表示


设 $l$ 层为卷积层：

* $f^{[l]}$：**滤波器的高（或宽）**
* $p^{[l]}$：**填充长度**
* $s^{[l]}$：**步长**
* $n^{[l]}\_c$：**滤波器组的数量**

* **输入维度**：$n^{[l-1]}\_H \times n^{[l-1]}\_W \times n^{[l-1]}\_c$ 。其中 $n^{[l-1]}\_H$表示输入图片的高，$n^{[l-1]}\_W$表示输入图片的宽。之前的示例中输入图片的高和宽都相同，但是实际中也可能不同，因此加上下标予以区分。

* **输出维度**：$n^{[l]}\_H \times n^{[l]}\_W \times n^{[l]}\_c$ 。其中

$$n^{[l]}\_H = \biggl\lfloor \frac{n^{[l-1]}\_H+2p^{[l]}-f^{[l]}}{s^{[l]}}+1   \biggr\rfloor$$

$$n^{[l]}\_W = \biggl\lfloor \frac{n^{[l-1]}\_W+2p^{[l]}-f^{[l]}}{s^{[l]}}+1   \biggr\rfloor$$

* **每个滤波器组的维度**：$f^{[l]} \times f^{[l]} \times n^{[l-1]}\_c$ 。其中$n^{[l-1]}\_c$ 为输入图片通道数（也称深度）。
* **权重维度**：$f^{[l]} \times f^{[l]} \times n^{[l-1]}\_c \times n^{[l]}\_c$
* **偏置维度**：$1 \times 1 \times 1 \times n^{[l]}\_c$

由于深度学习的相关文献并未对卷积标示法达成一致，因此不同的资料关于高度、宽度和通道数的顺序可能不同。有些作者会将通道数放在首位，需要根据标示自行分辨。

## 简单神经网络

  <img src="/img/cnn/Simple-Convolutional-Network-Example.jpg" width="400px" />
  典型的神经网络包含三层， Convolution layer, Pooling layer and Fully Connected Layer.

## Pooling Layer
  
  作用是减小模型大小，提高计算速度， 以及减小噪声，提高提取特征的稳健性。  采用最多的一种 Pooling 方式是 Max Pooling, 如下图所示
  <img src="/img/cnn/Max-Pooling.png" width="400px" />
  另外一种是 Average Pooling, 求区域的平均值
  <img src="/img/cnn/Average-Pooling.png" width="400px" />
  Pooling 设有一组超参数，即 filter 大小，步长 s, 选用 MaxPooling 或者 AveragePooling 但并没有参数需要学习，

  设 Pooling 之前输入的维度为 $n_H \times n_W \times n_c$
  则输出为

  {{<formula>}}
  $$\left\lfloor\frac{n_{H}-f}{s}+1\right\rfloor \times\left\lfloor\frac{n_{W}-f}{s}+1\right\rfloor \times n_{c}$$
  {{</formula>}}

## Example
  <img src="/img/cnn/CNN-Example.jpg" width="700px" />
  各个层的参数表示为

   | Activation shape | Activation Size | #parameters
    :-: | :-: | :-: | :-:
    **Input:** | (32, 32, 3) | 3072 | 0
    **CONV1(f=5, s=1)** | (28, 28, 6) | 4704 | 158
    **POOL1** | (14, 14, 6) | 1176 | 0
    **CONV2(f=5, s=1)** | (10, 10, 16) | 1600 | 416
    **POOL2** | (5, 5, 16) | 400 | 0
    **FC3** | (120, 1) | 120 | 48120
    **FC4** | (84, 1) | 84 | 10164
    **Softmax** | (10, 1) | 10 | 850

## 为什么使用卷积
卷积过程有效的减小了CNN 参数数量，主要通过两种方式

* **参数共享（Parameter sharing）**：特征检测如果适用于图片的某个区域，那么它也可能适用于图片的其他区域。即在卷积过程中，不管输入有多大，一个特征探测器（滤波器）就能对整个输入的某一特征进行探测。
* **稀疏连接（Sparsity of connections）**：在每一层中，由于滤波器的尺寸限制，输入和输出之间的连接是稀疏的，每个输出值只取决于输入在局部的一小部分值
池化过程则在卷积后很好地聚合了特征，通过降维来减少运算量。
