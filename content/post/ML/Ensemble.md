---
title: Ensemble Method
date: 2019-09-27
tags:
    - machine learning
categories:
    - Learn
summary: 集成方法
---

Combine multiple base-learners to imporve applicability across different domains or generalization performance in a specific task

## Ensemble Strategy

- 将多个学习器结合可以带来三个方面的好处
  1. 增强泛化性能
  2. 计算有可能陷入局部最小， 降低局部最小的风险
  3. 扩大假设空间
  {{< figure src="/img/ensemble_01.png" title="Three advantages" lightbox="true" >}}

- Average
  - Simple averaging
    $$H(X) = \frac1T \sum_{i=1}^{T} h_i(x)$$
  - Weighted averaging
    $$H(x) = \sum_{i=1}^{T} \omega_i h_i(x)$$

- Voting
  - learner $h_i$ predicte a label from the collection of class $\{c_1, c_2, \dots, c_N\}$. The learner $h_i$ generate a vector $(h_i^1(x); h_i^2(x); \dots; h_i^N(x))$ from the sample $x$.
  - Marority voting

  $$H(\boldsymbol{x})=\left\{\begin{array}{ll}{c_{j},} & {\text { if } \sum_{i=1}^{T} h_{i}^{j}(\boldsymbol{x})>0.5 \sum_{k=1}^{N} \sum_{i=1}^{T} h_{i}^{k}(\boldsymbol{x})} \\ {\text { reject, }} & {\text { otherwise. }}\end{array}\right.$$

  可以拒绝， 即保证要提供可靠的结果
  - Plurality voting (预测为票数最多的标记)
    $$H(\boldsymbol{x})=c_{\arg \max_j  \sum_{i=1}^{T} h_{i}^{j}(\boldsymbol{x})}$$
  - Weighted Voting (加权平均)
      $$H(\boldsymbol{x})=c_{\arg \max_j  \sum_{i=1}^{T} \omega_i h_{i}^{j}(\boldsymbol{x})}$$

- 学习法(stacking)
   Stacking 从初始数据集训练出初级学习器， 然后生成一个新数据集用于训练次级学习器。 在新数据集中， 初级学习器的输出被当做样例输入特征， 而初始样本的标记被当做样例标记
  {{< figure src="/img/ensem_02.png" title="Stacking Method" lightbox="true" >}}
  - 若直接使用初级学习器的训练集产生次级训练集， 则过拟合的风险较大
  - K-Fold： 初始训练集被划为$k$ 个大小相似的集合 $D_1, D_2, \dots D_k$, 令 $\overline{D}_j = D \backslash D_j$
    - 给定 T 个 base learner, 初级学习器 $h_t^j$ 通过在 $\overline{D_j}$ 上使用 第$t$ 个学习算法
    - 对 $D_j$ 中每个样本 $x_i$, 令$z\_{it} = h\_t^j(x_i)$, 则由 $x_i$ 产生的次级训练样例的示例部分为 $z\_i = (z\_{i1}; z\_{i2}; \dots z\_{iT})$, 标记部分为 $y_i$
    - 交叉验证过程结束后， 从T个初级学习器产生的次级训练集为 $D'=(Z\_i, y\_i)\_{i=1}^m$
    - 将 $D'$ 用于训练次级学习器
  
## Bagging

- Bagging is a voting method, but base-learners are made different deliberately
  - Train them using slightly different training sets
  - Generate $L$ slightly different samples from a given sample is done by **bootstrap**: given $X$ of size N, we draw N points randomly from $X$ with replacement to get $X^{(j)}$
  - Train a base-learner for ecah $X^{(j)}$

## Boosting

- In bagging, generate "uncorrelated" base-learners is left to chance and unstability of the learning method. (让下面的 learner train 前几个learner错误的地方)
- 考虑 binary classification: $d^{(j)}(x) \in {1, -1}$
- Cominber three `weak learners` to generate a `strong learner`
  - A week learner has error probability less than 1/2
  - A strong learner has arbitrarily small error probability
- Traning Algorithm
  1. Given a large traning set, randomly divide in into three
  2. Use $X^{(1)}$ to train the first learner $d^{(1)}$ and feed $X^{(2)}$ to $d^{(1)}$    
  3. Use all points misclassified by $d^{d(1)}$ and $X^{(2)}$ to train $d^{(2)}$. Then feed $X^{(3)}$ to $d^{(1)}$ and $d^{(2)}$
  4. Use the points on which $d^{(1)}$ and $d^{(2)}$ disgree to train $d^{(3)}$
- Tesing Algorithm
  1. Feed a point it to $d^{(1)}$ and $d^{(2)}$ first. If outpus agree, use them as final predction
  2. Otherwise the output of $d^{(3)}$ is taken
- Example
  {{< figure src="/img/ensem_03.jpg" title="Boosting example" lightbox="true" >}}
- Disadvantages
  Requires a large traning set to afford the three-way split
  
## AdaBoost

- Use the same training set over and over again, but how to make points "larger".
- Modify the probabilities of drawing the instances as a function of error
- Notation:
  - $\text{Pr}^{(i,j)}$: probability that an example $x^{(i)}, y^{(i)}$ is drawn to train the $j$th base-learner $d^{(j)}$
  - $\varepsilon^{(j)} = \sum_i \text{Pr}^{(i,j)} 1(y^{(i)} \neq d^{(j)}x^{(i)})$  error rate of $d^{(j)}$ on its training set.
- Algorithm
  1. Initalize $\text{Pr}^{(i,j)} = \frac1N$ for all i
  2. start from $j = 1$:
      1. Randomly draw $N$ examples for $X$ with probabilities $\text{Pr}^{(i,j)}$ and use them to train $d^{(j)}$
      2. Stop adding new base-learners if $\varepsilon^{(j)} \geq \frac12$
      3. Define $\alpha_j=\frac12 \log \left( \frac{1-\varepsilon^{(j)}}{\varepsilon^{(j)}}\right)>0$ and set $\text{Pr}^{(i,j+1)}  = \text{Pr}^{(i,j)} \exp(-\alpha_j y^{(i)} d^{(j)} (x^{(i)}))$ for all $i$
      4. Normalize $\Pr^{(i, j+1)}$
- Testing
  1. Given $x$, calcualte $\hat{y}^{(j)}$ for all $j$
  2. Make final prediction $\hat{y}$ by voting $\hat{y} = \sum_j \alpha_j d^{(j)}(x)$
- Example
  <img src="/img/ensem_04.jpg" width="400px"/>

- Why AdaBoost Works
  - Empirical study: AdaBoost reduces overfitting as $L$ grows, even when there is no training error
  <img src="/img/ensem_05.jpg" width="400px"/>
  - AdaBoost **increases margin** (Same as SVM)
    - A large margin imporves generalizability
    - Define margin of prediction of an example $(x^{(i)}, y^{(i)}) \in X$ as:
      $$margin(x^{(i)}, y^{(i)}) = y^{(i)}f(x^{(i)}) = \sum_{j:y^{(i)} = d^{(j)}(x^{(i)})} \alpha_j - \sum_{j:y^{(i)} \neq d^{(j)}(x^{(i)})} \alpha_j$$
    <img src="/img/ensem_06.png" width="600px"/>
