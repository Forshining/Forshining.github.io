---
title: 'Distributed Optimization Algorithms'
date: 2022-09-09
permalink: /posts/2022/07/blog-post-DistriOptim/
tags:
  - convex optimization
  - decentralized optimization
---

近几天在学习分布式优化，这里对我从参考书和论文中的算法设计历史脉络以及open problems做一个整理：

# Decentralized Gradient Descent (DGD)

DGD算法的一般形式如下：

$$x^{(k+1)} = W x^{(k)} - \alpha^k \nabla f(x^{(k)})$$

DGD算法本质上是将优化过程分成了两部分：一部分是consensus,即利用Weight matrix将连通节点的信息做一个沟通；另一部分就是传统的梯度下降，这里的梯度下降是针对每一个local节点。而DGD有一个显著的缺点：如果DGD当中的步长选择常数的话，那么会得到inexact convergence；而如果选择逐渐趋于零的步长(diminishing step size)，那么虽然可以得到exact convergence, 但是会造成较慢的收敛速度，这在实际应用当中是一个棘手的问题。

# Decentrailized Stochastic Gradient Descent (DSGD)

而在中心化算法当中效果很好的SGD也能在分布式优化当中应用起来，他的一般更新形式如下：

$$x^{(k+1)} = W x^{(k)} - \alpha^k \nabla f_i(x^{(k)})$$

而在SGD中为了获得较好的收敛效果，通常我们会做一个bounded variance的假设，从而使得SGD在逐渐趋于零的步长的设定下能够得到次线性的收敛性，在理论上这体现在引入的bounded variance的假设在上界估计中起到了作用，因此我们在确定下步长之后，考虑缩小bounded variance来提高SGD的速度，也就是方差缩减技术(variance-reduction technique)，常见的方差缩减方法有：SAGA,SVRG：这些方法的核心思想是统计计算当中的control covariates，即利用一个已知期望的随机变量对我们感兴趣的变量做一个无偏估计，利用Rao-Blackwell不等式可以使得目标变量的方差缩小，我将在之后对方差缩减技术做一个介绍，这里不再赘述。而这种方差缩减技术同样可以用在分布式优化的问题中。

其次，对于一阶的分布式优化算法，通常会遇到一个问题：假设我们得到了一个分布式优化的一致解(consensus solution)，因为我们只能得到的条件是各个节点的梯度之和为0，而不能确定每一个节点在一致解上的梯度为0，因此在下一次迭代的过程中，得到的就可能不是这个一致解。而解决这个问题的根本但不大可能实现的办法就是求得全梯度并用全梯度更新，所以一个可行且有效的方法是利用局部gradient tracking来近似全局梯度。

# Variance-Reduction Techniques

# Remove the influence of data-heterogeneity

在分布式优化问题当中，数据的异质性体现在$f_i(x^{(k)}) \neq f_j(x^{(k)})$，也就是同一个点的梯度可能是不相等的，而一个特殊的场合就是假设该分布式优化问题通过算法（e.g. DGD）得到了一个minimizer，但我们能得到的只有$\sum_i \nabla f_i(x^{\*}) = 0$，所以可以知道在下一次迭代之后不会停留在$x^{\*}$，也就是$x^{\*}$不是一个固定点；而一个根本但不大可能实现的方法就是求得全梯度并以此进行更新，在分布式优化的问题当中这会导致极大的通讯消耗(communication cost)，因此为了解决数据异质性问题，就有了如下的几种有效的解决方法。

## EXTRA: An Exact first-order algorithm for decentralized consensus optimization

## Exact Diffusion/NIDS/$D^2$

## Gradient Tracking Techniques


