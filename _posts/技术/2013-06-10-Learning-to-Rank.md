# Learning to Rank基本思路初探

既然是Learning，那么就必然要用到机器学习的理论和算法。可是，主要的机器学习算法，诸如SVM，DBN等，均是面向向量空间的算法。于是，如何将Ranking这个目标与机器学习算法对接起来，便成为Learning to Rank的最主要的研究方向。其他的研究，例如用boosting减小偏差，用bagging减小方差等等，都是在解决这个问题的基础上，机器学习算法在这个领域的应用。

在一般的Learning to Rank场景中，可用的信息包括query-document 对的特征，及其rank信息。Rank信息有三种形式：

1. 对于一个query，每个document与之相关性评分值（Rank Score）；
2. 对于同一query，两document与之相关性偏好值（Preference）；
3. 对于同一query，所有document与之相关性强弱的排列顺序（Rank List）。

这三种信息在一定程度上可以相互转化。

Learning to Rank 的最终目的，就是学习一个rank score 函数 f, 使得对每个query-document对（q,d），f(q,d)的值，或两两比较，或者排序得到的列表符合可用的信息。

下面总结一下对于这些形式的信息，Learning to Rank学界是如何用机器学习的方法进行处理的。

## 从Rank Score中学习

对于相关性评分形式的Ranking信息，具有机器学习背景的朋友容易想到，可以直接利用Regression等方法对rank score函数f进行学习。由于相关性评分值一般是分为几个固定的离散等级，因此也可以使用分类的方法进行学习。

## 从Preference 中学习

对于相关性比较对的Ranking信息，可以看成以一对query-document对的特征为输入，相关性强弱比较数据为输出，建立回归或者分类学习任务。各种算法在处理输入信息、输出信息以及采用的具体学习方法上各有特色。

### 学习Rank Score 函数

采用该途径的算法直接使用学习机器来估计 Rank Score 函数，而采用基于Preference的误差来训练学习机器，使之输出符合 Preference 的 Rank Score。

1. RankNet: 输入：每个query-docuement 特征向量；目标函数：根据评分函数构造偏好概率，偏好概率与真实偏好的交叉熵；学习算法：神经网络；

2. RankBoost: 输入：两个query-document特征向量；目标：评分函数差估计偏好函数的指数误差；学习算法：adaboost；

3. GBRank: 输入：两个query-document特征向量；目标：评分函数差估计偏好函数的截半平方误差；学习算法：Gradient Boosting Tree；

### 直接学习 Preference 函数

采用该途径的算法使用学习机器来估计 Preference 函数，因此其输入需要包含两个(q,d)对的信息。

1. SortNet: 输入：两个query-document特征向量；目标函数：偏好函数与目标值的均方误差；学习算法：神经网络；

2. RankingSVM: 输入：两个query-document特征向量的差值；目标函数：SVM估计偏好函数的误差；学习算法：SVM；

## 从排序列表中学习

对于排序列表形式的Ranking信息，各种方法均学习一个Rank Score函数，使得函数输出最优化目标函数。关于目标函数的选择，则有两种主要的思路：

### 优化MAP、NDCG等排序列表的目标函数

优化目标函数是最为直接的想法。但是，由于Ranking 的目标函数，如Map，NDCG等都是不连续的函数，无法使用常用的基于梯度等信息的传统优化方法。对此问题，有三种解决方案：

1. 用能够较好拟合目标函数的光滑函数来替代目标函数。采用该思路的方法有Soft Rank等；

2. 优化目标函数的一个可处理的上界。采用该思路的方法有 SVM-map等；

3. 采用类似Adaboost，GA等可处理非连续目标函数的优化方法，直接优化目标函数。采用该思路的方法有AdaRank等。

### 优化Ranking List的概率模型

该思路对Ranking List进行概率建模，并优化学习所得的Ranking List概率分布使之逼近已知的Ranking List概率分布。ListNet 与 ListMLE是采用该途径的两种方法。他们之间的区别在于ListNet采用交叉熵，而ListMLE则采用似然函数作为优化目标。