---
title: "Convolutional Neural Networks"
layout: page
date: 2017-03-16
---
[TOC]

## 关于
卷积网络经典文章导读，文章列表是参考 CS231N 课程。

## AlexNet
论文：Imagenet classification with deep convolutional neural networks
Alex Krizhevsky, Ilya Sutskever, **Geoffrey E Hinton**, 2012

Hinton 带学生打比赛的故事。

- 求解问题： ImageNet LSVRC-2010 比赛，1.2M高精度图片，1000分类！ILSVRC-2012 TOP5 error：15.3%，第二名是 26.2%！
- 效果： TOP1 error：37.5%， TOP2 error：17.0%。
- 网络参数：60M 参数，650,000个神经元
- 重要创新： ReLU激活函数， GPU计算卷积，dropout
- 5层卷积层+3层全连接层，卷积层的深度是很关键的，移除任何一层都将导致性能的降低！
- GTX 580 3GB GPUs 训练 5-6天

**Amazon’s Mechanical Turk crowd-sourcing tool**

- 对图像做下采样到固定大小 256x256，满足固定大小输入；对每个像素减去在整个训练集上的均值

### 结构上的创新
- ReLU 非线性：加速训练，CIFAR-10上达到25%错误率，比tanh快6倍！

相关论文：V. Nair and G. E. Hinton. Rectified linear units improve restricted boltzmann machines. In Proc. 27th International Conference on Machine Learning, 2010

- 多 GPU 训练：2个GPU
- Local Response Normalization：将错误减少1-2个点。
- Overlapping Pooling：Pooling尺寸=3，步长却是2
- 结构：前面五层是卷积层，每个卷积层分为两个部分，每个部分放在一个GPU中，在卷积过程中，第2、4、5层的两个GPU互不干扰，第3层和全连接层又相互交错连接的部分。maxpooling层在第1，2，5层卷积层，Local Response Normalization layer 在第1、2层。

<img src="/wiki/static/images/alexnet.png" />

- 每一层的详细参数：输入 224x224x3
    1. 96个11x11x3的滤波器，分为上下两部分，每部分48个
    2. 256个5x5x48的滤波器，两个GPU互不干扰
    3. 384个3x3x256的滤波器，两个GPU有交互
    4. 384个3x3x192的滤波器，两个GPU互不干扰
    5. 256个3x3x192的滤波器
    6. 全连接层为4096个神经元
- 卷积层参数：1.45M，卷积层输出为6x6x256；三个全连接层分别是：37.75M，21.92M，4.10M！！可以看到参数主要集中在卷积层最近的两个全连接层！！

### 降低过拟合技巧
<img src="/wiki/static/images/alexnet-error.png" style="float:left;width:400px;margin-right:20px;"/>

- Data Augmentation: 数据增强：
    - 平移和水平翻转，从256x256的图片，截取224x224的图片块，加上水平翻转，一张图片就变成了32x32x2=2048个样本！预测的时候；预测的时候，截取四个角+中央以及他们的水平翻转10张图片，结果取平均！
    - 加噪，有点像 denoise 的概念，对每一个像素 $(I_{xy} = [I_{xy}^R, I_{xy}^G, I_{xy}^B]^T)$，不是简单的在每个分量上简单地叠加，而是在三个通道的协方差矩阵的三个主方向上，叠加对应比例的噪声。下式中，p与lambda分别是协方差矩阵的三个特征向量和特征值，$(\alpha_i)$ 是叠加的噪声比例，服从0均值方差为0.1的高斯分布。

$$
[\mathbf{p}_1, \mathbf{p}_2, \mathbf{p}_3] [\alpha_1 \lambda_1, \alpha_2 \lambda_2, \alpha_3 \lambda_3]^T
$$

- Dropout：可以看做一种大量的神经网络的模型组合。可以解决过拟合问题，学习到鲁邦的特征，预测的时候，则将神经元的值乘以概率即可。 dropout 技术大致使得收敛的迭代次数增加一倍。

- 配置：NVIDIA GTX 580 3GB GPUs，两块



### 量化评估
用最后的4096维特征作为图像向量，评估图像的相似度，效果很不错，用 auto-encoder 于这些特征上比在raw data上效果应该会更好。

<img src="/wiki/static/images/alexnet-res.png" />

## ZFNet
论文：Zeiler M D, Fergus R. Visualizing and Understanding Convolutional Networks[C]. european conference on computer vision, 2013: 818-833.

- ZFNet 在 AlexNet 上改进的不多，主要贡献在 CNN 的可视化。
- 解释 AlexNet 为什么效果好（主要是通过可视化分析），以及怎么进一步改进。
- 数据集：Caltech-101，Caltech-256.
- 可视化技术：**解卷积**，通过显示激活任意一层的单一的 feature map 的输入图像的方法，可视化某个神经元学到的东西。 **Zeiler, M., Taylor, G., Fergus, R.: Adaptive deconvolutional networks for mid and high level feature learning. In: ICCV (2011)**
- 敏感性分析：通过遮蔽输入图片的一部分，展示图片的哪一部分对分类结果比较重要。
- 对 AlexNet 改进，并迁移到其他任务，只将最后一层 softmax 重新训练，有监督的 pre-training。
- 之前的可视化工作一直停留在第一层。
- 通过梯度下降最大化某个神经元的输出，从而找出最优激励图像（BP to Image）**Dumitru Erhan, Yoshua Bengio, Aaron Courville, and Pascal Vincent，Visualizing higher-layer features of a deep network，2009**，没有解释神经元的不变性！？
- 计算在最优点处的 Hessian 矩阵，理解这种不变性？
- 解卷积是无监督学习，相当于一个探针，探测一个已经学好的网络

<img src="/wiki/static/images/zfnet-deconv.png" />

- 解卷积过程：将同一层的其他神经元置0，将该层作为解卷积的输入，依次经历了(i) unpool, (ii) rectify and (iii) filter
- Unpooling: Max-pooling 不可逆，为了解决这个问题，在做 Max-pooling 的时候，用一个 switch 变量记录最大值的位置。*问题，可视化的时候，没有正向卷积过程，这个 switch 变量从哪来？*
- Rectification：直接将重构信号通过 ReLU？
- Filtering：将卷积核做水平、垂直翻转后，再进行卷积。这就可以解卷积了？不应该要做个逆滤波？

> 解卷积解释：设原始信号为 $(f)$，卷积核为$(k)$，解卷积核为$(k')$，那么经过卷积和解卷积，信号变为
> $(f * k * k')$，利用卷积运算的结合律，也可以表达为 $( f * (k * k') )$，如果要使得解卷积后的信号
> 和原始信号一致，那么需要 $( k * k' = \delta )$，即两个卷积核的卷积为单位冲击函数，也就是
> $( \sum_{x',y'} k(x - x', y - y') k'(x', y') = \delta(x, y))$，即只有在$(x=0,y=0)$时为1，
> 其他情况为0。这里将卷积核水平和垂直翻转后，相当于 $( \sum_{x',y'} k(x - x', y - y') k(-x', -y'))$
> 可以看到，当x和y都为0时取得最大值（达到匹配），其他情况虽然不为0，但小于匹配的时候的值，所以可以看做逆滤波的一种近似实现. 不过简单试验结果表明，这种近似太粗糙了。

- CNN 训练的输入是[-128,128]，居然没有归一化？！初始化是随机取的，幅度为$(10^{-2})$

### 卷积网络可视化
![zfnet-res](/wiki/static/images/zfnet-res.png)

- 特征可视化：选取TOP9
- 结构选择：11x11滤波器改为7x7，stride减少到2，从而使得第1，2层滤波器提取到更多有用的信息。？？
- 遮挡敏感性：测试分类器是否真的检测到了图片中的目标，还是只是用周围的信息。
- 选取第5层最强的 feature map 的响应值之和，随着遮挡的位置的变化。可视化的结果如图(b)。

![zfnet-res2](/wiki/static/images/zfnet-res2.png)

### 特征泛化能力
- 利用 ImageNet 学出来的模型，应用到其他任务，例如：Caltech
- 只改变最后一层，前面的层都固定不变。