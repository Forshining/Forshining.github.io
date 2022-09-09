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

# Variance-Reduction Techniques

而在SGD中为了获得较好的收敛效果，通常我们会做一个bounded variance的假设，从而使得SGD在逐渐趋于零的步长的设定下能够得到次线性的收敛性，在理论上这体现在引入的bounded variance的假设在上界估计中起到了作用，因此我们在确定下步长之后，考虑缩小bounded variance来提高SGD的速度，也就是方差缩减技术(variance-reduction technique)，常见的方差缩减方法有：SAGA,SVRG：这些方法的核心思想是统计计算当中的control covariates，即利用一个已知期望的随机变量对我们感兴趣的变量做一个无偏估计，利用Rao-Blackwell不等式可以使得目标变量的方差缩小，我将在之后对方差缩减技术做一个介绍，这里不再赘述。而这种方差缩减技术同样可以用在分布式优化的问题中。

## SAGA

## SVRG

# Remove the influence of data-heterogeneity

在分布式优化问题当中，数据的异质性体现在$\nabla f_i(x^{(k)}) \neq \nabla f_j(x^{(k)})$，也就是同一个点的梯度可能是不相等的，而一个特殊的场合就是假设该分布式优化问题通过算法（e.g. DGD）得到了一个minimizer，但我们能得到的只有$\sum_i \nabla f_i(x^{\*}) = 0$，所以可以知道在下一次迭代之后不会停留在$x^{\*}$，也就是$x^{\*}$不是一个固定点；而一个根本但不大可能实现的方法就是求得全梯度并以此进行更新，在分布式优化的问题当中这会导致极大的通讯消耗(communication cost)，因此为了解决数据异质性问题，就有了如下的几种有效的解决方法。

## EXTRA: An Exact first-order algorithm for decentralized consensus optimization

EXTRA的更新方式如下：

$$
\begin{align}
    x^{(k+2)} &= W x^{(k+1)} - \alpha \nabla f(x^{(k+1)}) \\ 
    x^{(k+1)} &= \tilde{W} x^{(k)} - \alpha \nabla f(x^{(k)})
\end{align}
$$

其中，$\tilde{W} = \frac{I + W}{2}或者我们将上两式进行合并，可以得到如下的两种形式：

$$x^{(k+2)} - x^{(k+1)} = W x^{(k+1)} - \tilde{W} x^{(k)} - \alpha (\nabla f(x^{(k+1)}) - \nabla f(x^{(k)})) $$

$$x^{(k+2)} = \tilde{W} x^{(k+1)} - \alpha f(x^{(k+1)}) + \sum_{t=0}^{k+1} (W - \tilde{W}) x^{(k)}$$

从最后一条式子中，我们可以将EXTRA看成是DGD的迭代算法的一个矫正，而且我们可以知道由于$(W - \tilde{W}) x^{(k)}$这一项是逐渐趋于零的，也可以看成是在不断迭代的过程中，需要矫正的量逐渐变小，也同时说明了这个求和是必要的。

## Exact Diffusion/NIDS/$D^2$

$$
\begin{align}
    \psi_i^{(k+1)} &= x_i^{(k)} - \alpha \nabla f_i(x_i^{(k)}) \\ 
    \phi_i^{(k+1)} &= \psi_i^{(k+1)} + x_i^{(k)} - \psi_i^{(k)} \\
    x_i^{(k+1)} &= \sum_{j \in N_i} w_{ij} \phi_j^{(k+1)}
\end{align}
$$

其中，第二条式子表示对于得到的local update进行矫正，如果矫正项$x_i^{(k)} - \psi_i^{(k)}$为0，则该算法退化成为DSGD算法；将上三式合并，我们可以得到：

$$x_i^{(k+1)} = \sum_{j \in N_i} w_{ij} (2x_i^{(k)} - x_i^{(k-1)} + \alpha (\nabla f(x_i^{(k)}) - \nabla f(x_i^{(k-1)})))$$

对上式取k极限我们可以知道，即使存在数据异质性，Exact-diffusion可以保持minimizer不变化。

## Gradient Tracking Techniques

如果我们能够求得到全梯度，那么就可以在取得minimizer之后的迭代过程中能够保持固定，但显然在分布式优化问题当中，全梯度的计算会消耗很大的通讯资源，而且有些节点之间本身无法进行通讯；因此可以采取一种妥协的方法，即对连通的节点进行一次local averaging来近似全梯度，之后利用local averaging gradient作为梯度迭代，一般形式如下：

$$
\begin{align}
  x_i^{(k+1)} &= \sum_{j \in N_i} w_{ij} x_r^{(k)} - \alpha d_i^k \\
  d_i^k &= \sum_{j \in N_i} d_j^k + \nabla f_i(x_i^{(k+1)}) - \nabla f_i(x_i^{(k)})
\end{align}
$$

其中，$d_i^0 := \nabla f_i(x_i^0)$，取极限之后我们容易看出，local averaging gradient会趋近于全梯度。

# Topology of Network/Communication

## Common Topology

## Static Expotential Graph

## One-peer Expotential Graph

## Directed graph