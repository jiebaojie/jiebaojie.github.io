---
layout: post
notes: true
subtitle: "人工智能基础（高中版）"
comments: false
author: "汤晓鸥 陈玉琨"
date: 2018-06-27 00:00:00

---

![](/img/notes/machinelearning/fundamentalsOfArtificialIntelligence/fundamentals_of_artificial_intelligence.jpg)

*   目录
{:toc }

# 第一章 人工智能：新时代的开启

## 1.1 跨越时空：铭铭的一天

## 1.2 光辉岁月：人工智能简史

*	横空出世（上世纪四五十年代）
*	第一次浪潮（1956——1974）：伟大的首航
*	第二次浪潮（1980——1987）：专家系统的兴衰
*	第三次浪潮（2011年至今）：厚积薄发，再造辉煌

## 1.3 百花齐放：人工智能在各行各业的应用

*	安防
*	医疗
*	智能客服
*	自动驾驶
*	工业制造

## 1.4 初露真容：人工智能与机器学习

### 什么是人工智能

人工智能是通过机器来模拟人类认知能力的技术。

人工智能最核心的能力就是根据给定的输入做出判断或预测，比如：

*	在人脸识别应用中，它是根据输入的照片，判断照片中的人是谁。
*	在语音识别中，它可以根据人说话的音频信号，判断说话的内容。
*	在医疗诊断中，它可以根据输入的医疗影像，判断疾病的成因和性质。
*	在电子商务网站中，它可以根据一个用户过去的购买记录，预测这位用户对什么商品感兴趣，从而让网站做出相应的推荐。
*	在金融应用中，它可以根据一只股票过去的价格和交易信息，预测它未来的价格走势。
*	在围棋对弈中，它可以根据当前的盘面形势，预测选择某个落子的胜率。

### 从数据中学习

*	监督学习（supervised learning）：监督学习要求为每个样本提供预测量的真实值。
*	无监督学习（unsupervised learning）：希望可以在不提供监督信息（预测量的真实值）的条件下进行学习。
*	半监督学习（semi-supervised learning）：介于监督学习与无监督学习之间，它要求对小部分的样本提供预测量的真实值。

### 在行动中学习

在机器学习的实际应用中，我们还会遇到另一种类型的问题：利用学习得到的模型来指导行动。

强化学习（reinforcement learning）：强化学习的目标是获得一个策略（policy）去指导行动。与监督学习不同，强化学习不需要一系列包含输入与预测的样本，它是在行动中学习。

一个强化学习模型一般包含如下几个部分：

*	一组可以动态变化的状态（state）。
*	一组可以选取的动作（action）。
*	一个可以和决策主体（agent）进行交互的环境（environment）。这个环境会决定每个动作后状态如何变化。
*	回报（reward）规则。当决策主体通过行动使状态发生变化时，它会获得回报或者受到惩罚（回报为负值）。

强化学习会从一个初始的策略开始。通常情况下，初始策略不一定很理想。在学习过程中，决策主体通过行动和环境进行交互，不断获得反馈（回报或者惩罚），并根据反馈调整优化策略。这是一种非常强大的学习方式。持续不断的强化学习甚至获得比人类更优的决策机制。

## 1.5 本章小结

人工智能是研究如何通过机器来模拟人类认知能力的学科。它可以通过人工定义或者从数据和行动中学习的方式获得预测和决策的能力。通过过去几十年的努力，人工智能已经获得了长足的发展，并且在多个行业得到了成功的应用。

# 第二章 牛刀小试：察异辨花

## 2.1 初学乍练：分类任务

分类器（classifier）：完成分类任务的人工智能系统。

## 2.2 含英咀华：提取特征

特征的质量很大程度上决定了分类器最终分类效果的好坏。

*	对于图像，人们设计出了方向梯度直方图
*	对于声音，人们设计出了梅尔频率倒谱系数
*	对于视频，有光流直方图
*	对于文本，有词频率-逆文档频率等

### 特征向量

向量（vector）：(x<sub>1</sub>, x<sub>2</sub>)

### 特征点和特征空间

这些表示特征向量的点称为特征点（feature point）；所有这些特征点构成的空间被称为特征空间（feature space）。

## 2.3 分门别类：分类器

分类器就是一个由特征向量到预测类别的函数。