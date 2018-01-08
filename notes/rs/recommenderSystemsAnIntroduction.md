---
layout: post
notes: true
subtitle: "Recommender Systems An Introduction"
comments: false
author: "Dietmar Jannach, Markus Zanker, Alexander Felfernig, Gerhard Friedrich（蒋凡 译）"
date: 2018-01-07 00:00:00

---

![](/img/notes/rs/recommenderSystemsAnIntroduction/recommender_systems_an_introduction.jpg)

*   目录
{:toc }

# 第1章 引言

推荐系统必须开发并维护一个**用户模型**(user model)或**用户记录**(user profile)保存用户的偏好。

用户的偏好可以通过监测用户行为**隐式地**获取，也可以由推荐系统询问访问者**显示地**获取。

系统生成个性化推荐列表时利用什么样额外的信息：基于**群体**或**协同**的方法。

## 第一部分：基本概念

### 1.1.1 协同过滤推荐

基本思想：如果用户在过去有相同的偏好，那么他们在未来也会有相似的偏好。

协同过滤（CF, Collaborative Filtering）。

常见问题：

*	如何发现与我们要推荐的用户有着相似偏好的用户？
*	如何衡量相似度？
*	如何处理还没有购买经历的新用户？
*	如果只有很少的评分该怎么办？
*	除了利用相似的用户之外，还有哪些技术可以用来预测某个用户是否喜欢其物品？

纯粹的协同过滤方法不会利用或要求任何有关物品本身的知识。

### 1.1.2 基于内容的推荐

推荐系统的两个目的：

*	用于激发用户去做某件事情
*	解决**信息过载**的工具

基于内容推荐的核心：能够得到物品的描述和这些特征的重要记录。

常见问题：

*	系统如何自动获取并持续改进用户记录？
*	如何决定哪个物品匹配或者至少能接近、符合用户的兴趣？
*	什么技术能自动抽取或学习物品的描述，从而减少人工标注？

基于内容推荐的两大优点：

*	不需要大规模用户就可以达到适度的推荐精准度
*	一旦得到物品的属性就能立刻推荐新物品

### 1.1.3 基于只是的推荐

在**基于知识的方法**中，推荐系统通常会用到有关当前用户和有效物品的额外信息（如**基于约束的推荐**）。

为了弥补缺少个性化，基于约束的推荐同样需要维护用户记录。

在许多基于知识的推荐系统中，用户需求必须通过交互引导得出。

常见问题：

*	哪种领域知识能表示成知识库？
*	什么机制可根据用户的特点来选择和排名物品？
*	如何在没有购买记录的领域获取用户信息？如何处理用户直接给出的偏好信息？
*	哪种交互方式能够用于交互式推荐系统？
*	设计对话时，要考虑哪些个性化因素才能确保准确获得用户偏好信息？


### 1.1.4 混合推荐方法

常见问题：

*	哪种方法能被组合，特定组合的前提是什么？
*	两个或多个推荐算法是应该顺序计算？还是采用其他混合方式?
*	不同方法的结果如何赋以权重，可以动态决定吗？

### 1.1.5 推荐系统的解释

解释是为了让用户更容易理解推荐系统的推理脉络。

常见问题：

*	推荐系统在解释其推荐结果的同时如何提高用户对系统的信任度？
*	推荐策略如何影响解释推荐的方式？
*	能通过解释让用户相信系统给出的建议是”公正的“或者不偏颇的吗？

### 1.1.6 评估推荐系统

推荐系统领域研究的主要推动力是提高推荐质量。

更好地描述用户体验或系统目标的3种评估方法设计方案：实验、半实验和非实验。

常见问题：

*	哪些研究设计适用于评估推荐系统？
*	如何利用历史数据实验评估推荐系统？
*	什么衡量标准适合不同的评估目标？
*	现有评估技术的局限是什么？尤其是在推荐系统的会话性或商业价值方面。

### 1.1.7 案例研究

常见问题：

*	推荐系统的商业价值是什么？
*	它能帮助提高销售额获奖更多访问者转化为购买者吗？
*	不同推荐算法在效果上有差别吗？在哪种情况下应该使用哪种技术？

## 1.2 第二部分：最新进展

涉及问题：

*	隐私和鲁棒性
*	在线消费决策
*	社交和语义网背景下的推荐系统
*	无处不在的应用

# 第2章 协同过滤推荐

主要思想：利用已有用户群过去的行为或意见预测当前用户最可能喜欢哪些东西或对哪些东西感兴趣。

存粹的协同方法的输入数据只有给定的用户-物品评分矩阵，输出数据一般有以下几种类型：

1.	表示当前用户对物品喜欢或不喜欢程度的预测数值
2.	n项推荐物品的列表（不包含当前用户已经购买的物品）

## 2.1 基于用户的最近邻推荐

**基于用户的最近邻推荐**(user-based nearest neighbor recommendation)主要思想：

*	首先，给定一个评分数据集和当前（活跃）用户的ID作为输入，找出与当前用户过去有相似偏好的其他用户，这些用户称为**对等用户**或**最近邻**；
*	然后，对当前用户没有见过的每个产品p，利用其近邻对p的评分计算预测值。

潜在假设：

1.	如果用户过去有相似的偏好，那么他们未来也会有相似的偏好；
2.	用户偏好不会随时间而变化。

### 2.1.1 第一个例子

用户-物品评分矩阵

### 2.1.2 更好的相似度和赋权体系

计算用户间接近程度的方法：

*	改进余弦相似度（在基于物品的推荐技术中更胜一筹
*	Spearman秩相关系数
*	均方差
*	Pearson相关系数（在基于用户的推荐技术中更胜一筹）

让两个用户对有争议的物品达成共识会比对广受欢迎的物品达成共识更有价值，解决方案：

*	对物品的评分进行变换，降低对广受欢迎物品有同样看法的相对重要性
*	通过**方差权重因子**，提高了具有高方差评分值的物品，也就是指有争议的物品的作用。

基于近邻评分的预测方法在遇到当前用户只为非常少的共同物品评分时会出错，导致不准的预测，解决方案：

*	**重要性赋权**
*	**样本扩展**：强调那些接近+1和-1的值，对原始数值乘以一个常量p来调整近邻的权值。

### 2.1.3 选择近邻

降低近邻集规模的两种方法：

*	为用户相似度定义一个具体的最小阈值：潜在问题是阈值过高，**覆盖率**降低；阈值太低，近邻规模不会显著降低。
*	k近邻：潜在问题是k值的选择，20到50个近邻似乎比较合理

## 2.2 基于物品的最近邻推荐

主要思想：利用物品间相似度，而不是用户间相似度来计算预测值。

基于物品推荐的方法就是简单地找到用户对相似物品的评分，来预测目标物品的评分。

### 2.2.1 余弦相似度量

在基于物品的推荐方法中，**余弦相似度**由于效果精确，已经被证实是一种标准的度量体系。


### 2.2.2 基于物品过滤的数据预处理

离线预计算：事先构建一个**物品相似度矩阵**。

降低矩阵复杂度：

*	仅考虑那些与其他物品同时评分数最少的物品
*	对每个物品只记录有限的近邻

风险：增加无法预测某个特定物品的风险

相对用户相似度而言，物品相似度更稳定，预处理计算不会过于影响预测准确度。

**二次采样**：可以随机选取数据的子集，或者忽略那些仅有非常少量评分或仅包含非常热门物品的用户记录。

## 2.3 关于评分

### 2.3.1 隐式和显示评分

显示评分主要问题：需要推荐系统的用户额外付出，而用户很可能由于看不到好处而不愿意提供这些评分。

隐式评分主要问题：难以确定用户行为是否被正确解释。

### 2.3.2 数据稀疏和冷启动问题

基于图的方法：主要思想式利用假定用户品味的“传递性”，并由此增强额外信息矩阵

**缺省投票**式另外一种用来处理稀疏评分数据的技术。这种思路就是给那些只有一两个用户评过分的物品赋以缺省值。

将两种不同类型的相似度（用户相似度或物品相似度）组合起来提高预测准确率。

冷启动问题式稀疏问题的一个特例。此类问题包括：

1.	如何向还没给任务物品评分的新用户推荐
2.	如何处理从未被评过分或购买过的物品

这两类问题都可以通过混合方式来解决，即利用额外的外部信息。对于新用户问题，其他策略也可能奏效：

1.	在推荐之前要求用户给出最低限度数量的评分。
2.	Eigentaste算法要求用户提供**标准集合**的评分也是一种类似的策略。

## 2.4 更多基于模型和预处理的方法

协同推荐技术一般分为两类：

*	基于记忆：传统的基于用户技术式基于记忆的（原始评分数据保存在内存中，直接生成推荐结果）。
*	基于模型：首先处理原始数据，就像基于物品的过滤和某些降维技术。运行时，只需要预计算或“学习过”的模型就能预测。

### 2.4.1 矩阵因子分解

方法：

*	**奇异值分解**（SVD）：发现文档中的潜在因子
*	**主成分分析**（Eigentaste）：用**主成分分析**（PCA）对评分数据预处理，过滤得出数据中“最重要”的方面，以解释大多数变量。
*	**概率潜在语义分析**（pLSA）：发现用户社区和评分数据里隐藏的兴趣模式，基于这种方法可以达到很好的准确度级别。

所有的预处理方法都必须解决数据更新问题：如何整合新的评分而不用重新计算整个“模型”。

### 2.4.2 关联规则挖掘

**关联规则挖掘**是一种在大规模交易中识别类似规则关系模式的通用技术。这种技术的典型应用是从超市里经常同时购买的商品中发掘成对或成组的商品。

### 2.4.3 基于概率分析的推荐方法

用概率方法实现协同过滤，最初非常简单的方法是将预测问题看作**分类问题**，如**贝叶斯分类器**。

## 2.5 近来实际的方法和系统

### 2.5.1 Slope One 预测器

思想：基于“热门度差异”，也就是对用户来说物品之间的评分差异。

### 2.5.2 Google 新闻个性化推荐引擎

主要挑战：

1.	实时产生列表只允许至多在1秒钟内生成内容；
2.	由于新物品的连续流入，“物品目录”变化频繁，与此同时其他文章很快就变得过时了。
3.	还有一个目标：及时反馈用户的交互和要考虑用户最近阅读的文章。

组合使用基于模型和基于记忆的技术：

*	基于模型：依赖两种聚类技术：**概率潜在语义索引**（PLSI）和一种新方法MinHash。
*	基于记忆：为了处理新用户，推荐系统基于记忆的方法分析“伴随浏览量”显得非常重要。

## 2.6 讨论和小结

协同过滤技术不可能应用于每个领域：例如一个没有购买历史的汽车销售系统，或需要更多用户偏好细节的系统。

协同过滤技术要求用户社区处于某个特定规模，需要足够的用户或评分数据。

## 2.7 书目注释

