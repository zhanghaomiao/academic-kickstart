---
title: 蓄水池采样
date: 2020-05-11
tags:
    - Sampling
    - Algorithm
categories:
    - Learn
summary: 蓄水池采样
---

## 问题

给定一个数据流，数据流长度N很大，且N直到处理完所有数据之前都不可知，请问如何在只遍历一遍数据（O(N)）的情况下，能够随机选取出m个不重复的数据。

## 解法

1. 如果接收的数据量小于m，则依次放入蓄水池。
2. 当接收到第i个数据时，i >= m，在[0, i]范围内取以随机数d，若d的落在[0, m-1]范围内，则用接收到的第i个数据替换蓄水池中的第d个数据。
3. 重复步骤2。

## 解释

1. 这里需要证明处理完这整个数据流之后，所以数被选择的概率都能有 m / N
2. i<=m 时， 第 i 个数据进入 reservior 的概率是 1
3. i>m 时， 从 [1, i] 选择随机数 d, 如果 d <= m , 则使用第 i 个数据替换 第 d 个数据， 第 i 个数据进入 reservoir概率是   m / i
4. i<=m时， 第(m+1) 次会替换掉池中数据的概率  m/(m+1), 替换第 i 个数据概率 1/m, 则 m+1 次替换掉 i个数据概率 为  1/(m+1), 不被替换概率为 m/(m+1), 第 (m+2) 次处理不替换第 i 个数据概率 (m+1) / (m+2)， 依次计算可得第 i 个数据不被替换的概率为 m / N
5. i > m 时， 接收 i+1 个数据可能替换第 i 个数据， 第 i 个数据不被替换概率 i / N
6. i<=m 时 ，第 i 个数据留在 reservoir 概率是 $1*m/N$,  i>m 时， 第 i 个数据留在 reservoir 概率为 $m /i * i/ N = m / N$

## 练习

reservior 深度为 1

1. https://leetcode-cn.com/problems/random-pick-index/
2. https://leetcode-cn.com/problems/linked-list-random-node


<iframe allow="autoplay *; encrypted-media *;" frameborder="0" height="150" style="width:100%;max-width:660px;overflow:hidden;background:transparent;" sandbox="allow-forms allow-popups allow-same-origin allow-scripts allow-storage-access-by-user-activation allow-top-navigation-by-user-activation" src="https://embed.music.apple.com/cn/album/%E8%B6%85%E4%BA%BA%E4%B8%8D%E4%BC%9A%E9%A3%9E/536247746?i=536248204"></iframe>
