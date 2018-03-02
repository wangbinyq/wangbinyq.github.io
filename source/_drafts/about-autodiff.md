---
title: 自动微分技术
tags: 
  Machine Learning
---

机器学习算法的一般模式是, 定义一个模型, 然后数据集上训练这个这个模型, 最后我们用这个模型去预测未知数据. 大部分的模型在训练时都要求误差函数(真实数据与模型预测的差别)的值尽可能的小. 常用的方法就是 [`梯度下降法 Gradient descent`](https://en.wikipedia.org/wiki/Gradient_descent) 及其变形 (比如[随机梯度下降 (SGD)](https://en.wikipedia.org/wiki/Stochastic_gradient_descent)).
