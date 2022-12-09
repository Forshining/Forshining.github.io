---
title: 'Lower Bounds for Minimax Problems'
date: 2022-12-08
permalink: /posts/2022/12/blog-post-LBMP/
tags:
  - Information Theory
  - Statistics
---

# Minimax lower bounds
In the classical estimation problem in statistics, mean-squared error is a frequantly-used risk function to measure the fitness of certain estimator of certain parameters, which we could generalize as the following formulations:

For a distribution $P \in \mathbb{P}$, we assume that we receive i.i.d. observations $X_i$ drawn according to specific distribution P. Based on these data, our aim is to estimate an unknown parameter $\theta(P) \in \Theta$.  To evaluate the quality of an estimator $\hat{\theta}$, we first give two notions: let $\rho: \Theta \times \Theta \rightarrow \mathbb{R}_{+}$ denote a (semi)metric on the parameter space $\Theta$ and $\Phi: \mathbb{R}_+ \rightarrow \mathbb{R}_+$ be a non-decreasing function with constraint $\Theta(0) = 0$. Hence, we could assess the quality of the estimate $\hat{\theta}(X_1,\cdots,X_n)$ in terms of the generalized risk: 
$$\mathbb{E}_P[\Phi(\rho(\hat{\theta}(X_1,\cdots,X_n),\theta(P)))]$$
which could be interpreted as the expectation of the penalty function in distribution P of the distance between the estimator and true parameter under some specific (semi)metric.

However, sometimes this kind of estimator will not be a good measure when we consider the risk functional not in a pointwise sense. There are two common approaches to tackle this thorny problem: one is via Bayesian view and another is the Wald method which also refers to the minimax risk function and we will give an introduction to the second approach, that is to find the $\hat{\theta}$ to minimize the maximum risk $sup_{P \in \mathbb{P}} \mathbb{E}_P[\Phi(\rho(\hat{\theta}(X_1,\cdots,X_n),\theta(P)))]$. We formalize this with strong intuition as follows:
$$\mathfrak{M}_n(\theta(P),\Phi \circ \rho) = inf_{\hat{\theta}} sup_{P \in \mathbb{P}} \mathbb{E}_P[\Phi(\rho(\hat{\theta}(X_1,\cdots,X_n),\theta(P)))]$$
Providing lower bound gives the convergence rate on some sense, which is significant to make comparisons among different estimators to conclude the best one. Accordingly, there are a few tools which are widely used in proving the lower bound of minimax problems, such as Le Cam's, Fano's and Assouad method that we will illustrate in the following context.

## Methodology
### Priliminaries

## Le Cam Method
Le Cam method provides lower bounds for single binary hypothesis testing problems, which utilize the connection between the hypothesis testing error and total variance distance. Suppose that we set the hypothesis on the specific two distributions $P_1$ and $P_2$ and conduct any test $\Phi: \mathcal{X} \rightarrow \{1,2\}$, we formulize the probability of the testing error: 
$$\mathbb{P}(\Phi(X) \neq V) = \frac{1}{2}P_1(\Phi(X) \neq 1) + \frac{1}{2}P_2(\Phi(X) \neq 2)$$
Then according to the inequality: 
$$\mathop{inf}\limits_{\Phi} \{P_1(\Phi(X) \neq 1) + P_2(\Phi(X) \neq 2)\} = 1 - ||P_1 - P_2||_{TV}$$
we have: 
$$\mathfrak{M}_n(\theta(P),\Phi \circ \rho) \geq \Phi(\delta) \mathbb{P}(\Phi(X_1^n) \neq V) \geq \frac{1}{2} \Phi(\delta) [1 - ||P_1 - P_2||_{TV}]$$
Hence, the following strategy is to find suitable $P_1$ and $P_2$ to bound the total variance distance and the minimax risk sequantially.
## Fano Method
### Classical global Fano Method
The core of Fano method is the Fano inequality stated as follows: For any Markov chain $V \rightarrow X \rightarrow \hat{V}$, 
$$h_2(\mathbb{P}(\widehat{V} \neq V))+\mathbb{P}(\widehat{V} \neq V) \log (|\mathcal{V}|-1) \geq H(V \mid \widehat{V})$$
where $h_2(p) = plogp + (1-p)log(1-p)$ and we could obtain the corollary when the distribution of $V \in \mathcal{V}$ is uniform:
$$\mathbb{P}(\widehat{V} \neq V) \geq 1-\frac{I(V ; X)+\log 2}{\log (|\mathcal{V}|)} \Rightarrow \inf _{\Psi} \mathbb{P}(\Psi(X) \neq V) \geq 1-\frac{I(V ; X)+\log 2}{\log |\mathcal{V}|}$$ and hence: 
$$\mathfrak{M}(\theta(\mathcal{P}) ; \Phi \circ \rho) \geq \Phi(\delta)\left(1-\frac{I(V ; X)+\log 2}{\log |\mathcal{V}|}\right)$$
Here we thnik of the r.h.s as a function of $\delta$ that balance the tradeoff between $\Phi(\delta)$ and the rest one and the strategy is similar to the previous Le Cam method.

Note that the Fano's method could handle the multiple hypothesis test which means the cardinality of $\mathcal{V} > 2$.

### Local Fano Method
Additionally, we could obtain the weaken version of Fano's method, named as local Fano method through based on the weakening of mixture representation of KL-divergence into mutual information and the local-packing construction(see the preliminaries above).

From Jensen inequality, defined the average distribution $\bar{P} = \frac{1}{\mathcal{V}} \sum_{i=1}^n P_i$, we have:
$$I(V ; X)=\frac{1}{|\mathcal{V}|} \sum_{v \in \mathcal{V}} D_{\mathrm{kl}}\left(P_v \| \bar{P}\right) \leq \frac{1}{|\mathcal{V}|^2} \sum_{v, v^{\prime}} D_{\mathrm{kl}}\left(P_v \| P_{v^{\prime}}\right)$$
Then, we could update the inequality: 
$$\mathfrak{M}(\theta(\mathcal{P}) ; \Phi \circ \rho) \geq \Phi(\delta)\left(1-\frac{\frac{1}{|\mathcal{V}|^2} \sum_{v, v^{\prime}} D_{\mathrm{kl}}\left(P_v || P_{v^{\prime}}\right)+\log 2}{\log |\mathcal{V}|}\right)$$
Utilizing the Jensen inequality above and the definition of local-packing, if we could find out a local-packing satisfying $log(\mathcal{V}) \geq 2(\kappa^2 \delta^2 + log2)$, the following lower bound is obtained:
$$\mathfrak{M}(\theta(\mathcal{P}) ; \Phi \circ \rho) \geq \frac{1}{2} \Phi(\delta)$$
### Distance-Based Fano Method





