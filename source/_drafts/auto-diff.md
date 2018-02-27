---
title: 自动微分机
tags:
  Translate
  Math
  Machine Learning
from: http://www.columbia.edu/~ahd2125/post/2015/12/5/
---

简介:
> 在优化或机器学习中, 使用`梯度下降法`可以求解大部分的问题, 而`梯度下降法`的主要计算量就是计算函数在某一点的微分(导数). 对于一般简单问题, 根据求导法则已经可以推导出大部分的函数微分. 然而随着问题变得复杂, 手动推导会容易出错, 并且效率低下. 因此我们需要使用`自动微分方法` (Automatic Differentiation).

全文翻译:

一年前, 我看到了一篇研究[自动微分](http://alexey.radul.name/ideas/2013/introduction-to-automatic-differentiation/)的文章, 一种自动计算微分的强大技术(同时还能推广到求解梯度(偏微分), 雅可比矩阵(多元微分)). 但是, 这不是最吸引人的地方, 毕竟我们可以通过有限差分近似计算微分, 只要 $h$ 足够小:

$$ \frac{df}{dx} \approx \frac{f(x + h) - f(x)}{h} $$

不幸的是, 这种数值计算方法在实践中常常不能正常工作. 很小的 $h$ 会产生浮点数截断, 而太大的 $h$ 会使误差变大. 而自动微分的方法可以完全避免这些问题: 它能精确的计算微分, 因此结果的精度只受到浮点数精度的影响.

`自动微分`的应用是十分的明显的. Google 的新机器学习框架 [TensorFlow](https://www.tensorflow.org/) 以及它的竞争者(启发者?) [Theano](http://deeplearning.net/software/theano/index.html), 都大量的应用了`自动微分`技术.

<!-- more -->

## 自动微分是什么

自动微分就是一种高级的函数求导链式法则. 当编写一个函数时, 我们仅能使用一些基本的操作(比如: 加法, 乘法). 任意复杂的函数都是这些简单操作的组合, 比如 $\frac{\log 2x}{x ^ x}$.

换一种说法就是, 任意复杂函数 $f$ 可以写成一系列简单函数 $f_k$ 的顺序组合:

$$ f = f_0 \circ f_1 \circ f_2 \circ \ldots \circ f_n $$

因为每个简单函数 $f_k$ 都有一个简单微分, 我们可以很容易的通过链式法则计算出 $\frac{df}{dx}$.

虽然这里使用了单变量的函数 $f: \mathbb{R} \rightarrow \mathbb{R}$ 作为例子, 但是自动微分可以很容易的推广到多变量函数 $f: \mathbb{R}^n \rightarrow \mathbb{R}^m$.

## 前向模式 (Forward Mode)

根据如何应用链式法则, 自动微分分为两种模式. 为了更直观地理解自动微分, 我们先从前向模式自动微分开始.

## 偏微分
对于一个函数 $f$, 我们可以构建一个`计算图`(有向无环图)来表示函数. 例如, 有这样一个函数 $f(x, y) = \cos x \sin y + \frac{x}{y}$, 可以构建如下的图:

![计算图](/images/auto-diff/Fig1.png)

图中每个节点表示一个简单函数, 每条边表示数据的流动. 本例中, 最上方节点 $w_7$ 表示函数 $f(x, y)$ 的值, 而最下方的节点 $w_1$ 和 $w_2$ 表示输入变量.

前向微分通过递归的求解父节点的微分得到. 比如, 我们来计算上例中的偏微分 $\frac{\partial{f}}{\partial{x}}$. 首先我们将 $\frac{\partial{f}}{\partial{x}}$ 表示成微分算子(比如 $\frac{\partial{f}}{\partial{x}} = Df$, 当 $D =
\frac{\partial}{\partial x}$ 时). 则我们有 $Df = Dw_7$ 以及:

$$
\begin{align}
D w_7 &= D (w_5 + w_6) = D w_5 + D w_6 \\\\
D w_6 &= D \frac{w_1}{w_2} = \frac{w_1 D w_2 - w_2 D w_1}{w_2 ^ 2} \\\\
D w_5 &= D w_3 w_4 = w_3 D w_4 + w_4 D w_3 \\\\
D w_4 &= D \sin w_2 = \cos w_2 \cdot D w_2 \\\\
D w_3 &= D \cos w_1 = -\sin w_1 \cdot D  w_1 \\\\
D w_2 &= D y \\\\
D w_1 &= D x
\end{align}
$$

最终我们所需要求的值 $Df$ 只依赖于 $x$, $y$, $Dw_1$ 和 $Dw_2$. 令 $D =
\frac{\partial}{\partial x}$, 则 $Dx = 1$, $Dy = 0$. 如果令 $D =
\frac{\partial}{\partial y}$, 则只需要修改 $Dx$ (从 1 到 0) 和 $Dy$ (从 0 到 1).

上述方法可以推广到计算任意的方向导数, 只要要求导的方向矢量 $\langle D x, D y \rangle$ 是单位向量. 用 $\nabla$ 替换 $D$ 以及矢量运算替换标量运算, 就可以计算梯度了.


为什么叫这个方法前向模式? 因为实际值是从图的底开始流向顶端, 就像我们执行这个表达式一样. 信息流动的方向跟我们执行表达式一样(自底向上), 因此我们就叫它 `前向模式`. 可以预测到, `反向模式` 中信息是从上向下流动的.

## 运行时复杂度

计算简单函数的微分需要求得函数各部分的值以及微分. 比如乘法计算:

$$ D w_i w_j = w_i D w_j + w_j D w_i $$

需要求出 $w_i$, $w_j$ 以及它们的微分 $D w_i$ 和 $D w_j$. 令 $CD(w)$ 为节点 $w$ 微分的计算成本, $CV(w)$ 为节点 $w$ 值的计算成本, 那么:

$$ CD(w_i w_j) = CD(w_i) + CD(w_j) + CV(w_i) + CV(w_j) + 3 $$

其中 3 表示额外的 3 次运算 (2 次乘法, 一次加法). 跟一般地有:

$$ CD(w) \le \sum_{w_k \in \text{children}(w)} CD(w_k) +
          \sum_{w_k \in \text{children}(w)} CV(w_k) + c $$

其中 $c$ 表示常数.

另外我们利用一些方法, 我们可以减少节点值的计算次数, 比如 $xxxxxxxxxxxxxxxxx$ 中, 不作任何处理的情况下, 可能需要计算 $xxx$ 8 到 9 次, 而如果将值保存下来, 可以减少计算次数, 则可以得到:

$$ CD(w) + CV(w) \le \sum_{w_k \in \text{children}(w)}  (CD(w_k) + CV(w_k) ) + c + 1 $$

如果要求的函数是 $f: \mathbb{R}^n \rightarrow \mathbb{R}$, 由 $P(f)$ 个简单函数组成, 那么 $ CD(f) + CV(f) $ 就是 $P(f)$ 的线性关系.

如果要求的是梯度, 那么计算成本就是 $nP(f)$ 的线性关系.
