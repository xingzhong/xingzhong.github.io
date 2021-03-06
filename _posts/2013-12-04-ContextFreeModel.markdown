---
layout: post
title:  "Context-free Model (Draft)"
date:   2013-12-01 12:40:26
categories: MachineLearning 
---

# 引言

上一篇文章的大致是这个思路，我们知道高斯模型可以很好而方便的描述一些数据，但是很多情况下一个高斯模型往往不能完美描述所有数据。
所以我们先后使用了多个独立的高斯模型，和多个有一阶条件相关或者线性变换相关的高斯模型来描述数据，当然随着模型的增多，和他们之间关系的增多，模型的复杂度和参数个数都有了相应的提高。


如果从系统的角度来分析上文的模型，我们发现他们都可以被称之为一个[动态系统] (dynamical system)。该系统有两方面组成，一个是确定变换的高斯状态，一个是不确定的高斯输出。
确定的状态，无法被直接观测，所以需要从不确定的观测值中进行推断。

单独的一个高斯模型对应的是唯一不变的一个高斯状态，[混合高斯模型]对应的是多个不变的高斯状态，而[隐含马尔科夫模型]和[卡尔曼滤波器]对应的是多个存在马尔科夫链或过程性质的线性转换高斯模型。

那么对待一组高斯状态，是否还有其他确定却比以上模型更复杂的关系呢？
在自然语言处理学科之中，语义之间存在这一定的关系。
常见的[n-gram]模型基本就是一个显式n阶马尔科夫模型，在此之上[上下文无关文法]也可以用来组织更为复杂的语义模型。
因此我们是否可以在以上动态系统的基础之上假设，其状态的变化规律符合上下文无关文法呢？

# 上下文无关文法
首先我们需要研究一下什么是[上下文无关文法]，然后需要了解在什么样的真实数据之中存在这种情况。

类似于马尔科夫模型，上下文无关文法的目标是，描述或者说限制子模型之间的相互关系。
马尔科夫模型说如果已知当前状态，无论其他以前的状态是什么，我们都可以知道下一个状态可能是谁，可能性有多大。
而上下文无关文法说如果已知当前状态，无论其他以前的状态是什么，当前的状态都可以被重写为多个状态发生的集合。
因此我们知道，每一个马尔科夫状态都必然对应一个状态输出，多个输出对应的是多个马尔科夫状态的转移路径。
每一个上下文无关状态对应的是一个或多个状态，这些状态有的可以输出(terminals)，有的可以继续对应更多的子状态(non-terminals)。
那么多个输出可以对应一个或多个上下文无关状态的树形结构。


从数据入手，常见的模型是自底向上(bottom-up)逐步抽象构造的。
数据中需要识别的模式(pattern)是数据模型所构造的。
马尔科夫模型对应的模式是单层模型，即不同模型之间只有转移关系。
而上下文无关文法对应的模式是一个层级模型，即不同模型之间是从属的树形关系。
这样的模式是有一定道理的，比如在时间序列之中，有的模式具有较长时间的相关距离，和较为复杂的连接顺序。在这种情况下，仅仅使用其相对应的一阶转移概率不足以刻画模式的特点。

例如，数据的隐含模型序列如下，

`\[
	a\;b\;a\;b\;b\;c\;a\;a\;b\;a\;b\;b\;c\;a
\]`

对`\( a\;b\;c \)`三个状态之间的关系建模，使用马尔科夫模型可以得到如下最优一阶状态转移矩阵。

`\[
	\begin{bmatrix}
		0.2 & 0.8 & 0.0\\
		0.33 & 0.33 & 0.33 \\
		1.0 & 0.0 & 0.0
	\end{bmatrix}
\]`

而如果使用上下文无关文法表示该序列，则存在以下一种语法树，

![tree]({{ site.url }}/assets/treeExample.png)

而其语法模型可由以下的类 Chomsky 范式表示，

`\[
	S \to N_1 \; S \;[0.5]\;|\; N_1\;[0.5]\\
	N_1 \to N_2 \; N_3 \;[1.0]\\
	N_2 \to N_4 \; N_2 \;[0.5]\;|\; N_4 \;[0.5] \\
	N_3 \to N_5 \; a \;[1.0]\\
	N_4 \to a\;b\;[1.0]\\
	N_5 \to b\;c\;[1.0]
\]`

比较两个生成模型，马尔科夫模型抓住了状态的一阶转移矩阵，也就是说如果一个序列的一阶马尔科夫关系亦如状态转移矩阵所示，则该序列即可被该马尔科夫模型所表示。
而上下文无关文法则抓住了不同模型之间以及生成时的层次关系。


具体到以上数据序列，模型`\( b \)`同时属于`\( N_4 \)` 与 `\( N_5 \)`两个高级别抽象模型，但在两个不同的模式之中，模型`\( b \)`作用不尽相同。
而对于马尔科夫模型，模型`\( b \)`在与模型`\( a \)`和`\( c \)`的转移过程之中，没有任何区别。

由于两个模型在构造上的不同，因此必然存在模型序列，使得其在两模型之间的相似度不同。

例如，测试序列，

`\[
	a\; b\; b\; b\; b\; c\; a\; a\; b\; c\; a\; b\; a\; b
\]`

拥有与原序列同样的马尔科夫特性，但不满足上文提到的上下文无关文法。

# 模型的定义

为了使用上下文无关语法描述实空间数据，类似于高斯马尔科夫模型，我们需要为每一个语法系统的 terminal 构造相应的隐含状态与其对应的高斯输出模型。
而对于输入实空间数据，似然函数可以通过上下文无关文法的CYK算法构造。


[动态系统]: http://en.wikipedia.org/wiki/Dynamical_systems
[n-gram]: http://en.wikipedia.org/wiki/N-gram
[上下文无关文法]: http://en.wikipedia.org/wiki/Context-free_grammar