---
layout: post
title: 面向购物篮消费者的零售分类规划中的品类管理和协调
date: 2018-01-16 19:41:00
tags: 论文解读
categories: 经济学
toc: true
---


《Category Management and Coordination in Retail Assortment Planning in the Presence of Basket Shopping Consumers》论文解读
<!--more-->


面向购物篮消费者的零售分类规划中的品类管理和协调

> 论文研究多商品品类和basket shopping consumers购物篮消费者（希望一次购物同时购买多品类的消费者）情况下的分类规划问题。介绍了双头垄断模式下零售商针对品类如何设定价格和多样性水平，且消费者选择去零售店或者不去购买是依据每个品类的效用来决定的。CM的通例是针对分类采用分权制，即每个品类经理负责最大化它所负责品类的利润。另一方面，零售商品类管理也可以采用集中制。论文说明CM模式无法找到最优解决方案，且和最优方案相比提供的多样性少且售价较高。通过数值分析，论文展示了CM模式的利润损失是显著的。最后，论文提出了采用basket profits（购物篮利润）而不是accounting profits（会计利润）来进行分权管理。使用point-of-sale（销售点）数据可以很容易对basket profits进行评估，评估结果表明论文提出的方法是近最优的解决方案。

### insight：
1. 零售商采取的有效分类是将销量低的商品排除掉，然而这样会使品类的多样性减少，因为造成客户流失，交易量也随之降低。
2. 针对购物篮用户，如果他找不到自己想要的某个品类，很有可能他将放弃已挑选的所有商品，转向另一个零售商店。
3. 针对多品类的研究表明：消费者购物篮中的两个品类可能会相关，原因可能是它们的互补特性，或者偶然性。消费者对店铺的选择取决于总的购物篮效用。
4. 经验主义表明：为招徕顾客而亏本销售的策略仅会带来较少的交易量增长。商品价格和多样性以及其他营销组合变量影响零售店市场份额占比的外因。

### 论述：
1. 如果有basket shoppers，CM和OPT相比提供的多样性少且售价高，且该情况下CM不太利于做决策。
2. 随着basket shoppers的数量增多，CM的利润损失很显著。
3. 寻求在CM模式下的近似最优解决方案：调整利润的计算方式，即对品类利润采用basket profits来评估。

### 专有名词：
1. game-theoretic situation 博弈论
2. noncooperative game 非合作博弈论
3. consumer choice model 消费者选择模型
4. random utility approach 随机效用理论
5. cherry-picking
6. Hotelling typemodel 霍特林模型
7. MNL choice model MNL选择模型
8. nested MNL framework 嵌套MNL框架
9. supermodular game 超模博弈：每个参与者增加其策略所引起的边际效用随着对手策略的递增而增加。
10. Pareto best 帕雷托最优
11. first-order optimality conditions 一阶最优性条件
12. duopoly 双头垄断
13. Bertrand price competition 伯特兰德价格竞争模型

### 模型：
1. 品类内商品的绝对利润相同。
2. 如果所有的零售商都提高售价并降低多样性，则市场份额会降低。

### 固定售价下的多样性竞争：

1. 零售商的最佳反应（已知其他零售商的多样性决定）：CM和OPT
CM是很明显的非合作博弈论，因为每个品类的销售经理都希望自己负责的品类利润最大。
OPT的一阶最优条件是基于从每个消费者获取的总利润，而CM解决方案是基于品类利润。
对比最佳反应：存在basket shopper时
论证说明在有basket shoppers是CM不会找到最优解决方案，且提供的品类多样性会少于最优方案。
总结：CM和OPT零售商当对非basket shoppers的市场占有份额少于一半时，都倾向于向独立品类分配更多的多样性。当大于一半时，OPT会对basket categories分配更多的多样性，而CM则不会。
2. 双头垄断
CM-CM：没有一个零售商可以单独占有市场一半的份额。且它们合并的份额最大是2/3.
对比结论：在均衡下CM较OPT提供较少的多样性，
囚徒困境的模式下OPT-OPT是较好的选择。
3. basket profits下的CM
OPT模式不容易实现，因为它需要评估所有购物类型下的顾客数以及函数的N维优化。
CM的主要缺点是它不能识别多样性之间的相关性，低估了多样性带来的边际利润。
提出basket profits的概念：从单个消费者赚取的总利润。
basket profit可以很方便地从point-of-sale(POS)数据中进行计算。

### 售价和多样性竞争
提出attractiveness levels的概念，可以决定价格和多样性，显著简化了双头竞争下的分析。
最佳反应：
吸引力水平随着竞争者的降低而降低。CM受价格决定最优反应。
CM提供的吸引力分类和多样性都少于OPT。
总结：CM提供较低的多样性和较高的售价，导致品类的整体吸引力低。
