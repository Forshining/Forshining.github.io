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

SAGA源于SAG，即stochastic averaging gradient,只不过SAGA得到的估计是对于SAG得到的估计的无偏纠正；其主要思想是创建一个n维数组，其中n为预先设定的数，每次对1,...,n随机抽取，利用存储的所有梯度做一个迭代，将之后得到的迭代点计算抽中的样本所对应的梯度，在数组中进行更新，一般形式如下：

$$
\begin{align}
    g_i^{(k)} &= \nabla f_{i,s_k} (x_i^{(k)}) - \nabla f_{i,s_k} (\hat{x_i^{(k)}}) + \frac{1}{N} \sum_{k=1}^n \nabla f_{i,k} (\hat{x_i^{(k)}}) \\
    x_i^{(k+1)} &= x_i^{(k)} - \alpha g_i^k
\end{align}
$$

其中，$\hat{x_i^{(k)}}$代表的是对于$\nabla f_i(x)$最近一次的迭代点，每次求得新的$x_i^{(k)}$之后，我们将$x_k$对应的梯度进行更新，其他梯度保持不变。

## SVRG

由于SAGA开辟了一个O(n)的一个内存空间，当n较大时会影响运算速度，而SVRG则舍弃了开辟内存空间储存近期梯度的方法，而是提出一个双循环的方法，即每一段时间迭代之后设置一个检查点，计算一次全梯度，并以此全梯度作为标准进行下一段时间的迭代，一般形式可以表示为如下：

$$
\begin{align}
  v_i^{(k)} &= \nabla f_{i,s_k} (x_i^{(k)}) - (\nabla f_{i,s_k}(\tilde{x_i^{(k)}}) - \nabla f(\tilde{x_i^{(k)}})) \\ 
  x_i^{(k+1)} &= x_i^{(k)} - \alpha v_i^{(k)}
\end{align}
$$

其中，$\tilde{x_i^{(k)}$是在以上迭代T（预先设定）次之后得到，可以选择T次迭代得到的x的平均值，或者通过随机抽样的方式。

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

其中，$d_i^0 := \nabla f_i(x_i^0)$，取极限之后我们容易看出，local averaging gradient会趋近于全梯度。同时我们可以通过时间序列当中innovation的概念来理解gradient tracking。

# Topology of Network/Communications

由于现实中通讯节点很难满足和其他所有节点结合，因此在网络当中，通讯性能和通讯节点的拓扑结构对于提升优化性能来说非常重要，下面我们给出几种常见以及有效的拓扑结构。

## Common Topology

常用的网络结构有：环状结构，星形结构，2D-grid,2D-torus,$\frac{1}{2}$-random graph

## Static Expotential Graph

而在实际训练过程当中，static expotential graph是一种高效但是缺乏可解释性的一种拓扑结构；其具体表示为一个节点只连接和该节点距离为$2^0$,$2^1$,...,$2^{ceiling(log_2(n-1))}$的通讯节点，而对应的权重矩阵表示为每一个点对于连通节点的权重都是等同的。这里笔者合理猜测，这种拓扑结构的可解释性在于，expotential形式的连接能够将DGD算法当中的迭代点通过consensus步骤更快获得minimizer，换句话说，static exponential graph能够使得一个通讯节点能够整合更全面的信息（全梯度）。

## One-peer Expotential Graph

考虑到static expotential graph的通讯消耗还是会比环状结构或者网格结构大，我们将static expotential graph分解为若干个互不相交的子图，其中每一个子图中的每一个节点都只有一个连通的通讯节点，而且距离均为$2^0$,$2^1$,...,$2^{ceiling(log_2(n-1))}$，训练时按照子图中节点距离，从小到大的顺序循环训练，这样可以减少一次训练时的通讯开支。


