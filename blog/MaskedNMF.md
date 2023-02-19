+++
title = "Masked Non-negative Matrix Factorization"
date = Date(2023, 02, 20)
hasmath = true
tags = ["research", "signal processing"]
mintoclevel = 2
maxtoclevel = 2
descr = """
A solver for the NMF problem with missing data points.
"""
+++

# {{ title }}

\toc

## Introduction
Non-negative matrix factorization is a powerful tool for dimensionality reduction and data analysis. The goal  in matrix factorization, in general, is to summarize a given data matrix with two low-rank factors. In this non-negative matrix factorization (NMF), we require the factor matrices to have only non-negative elements. This is in contrast to some other factorization methods, such as principle component analysis (PCA). In this brief article, we are interested in scenarios where not all the elements in the given data matrix are measured or valid. In this case, we want to "skip" these missing elements when solving the optimization problem and finding the factor matrices.

## The non-negative matrix factorization problem 
Given the non-negative data matrix $D\in\mathbb{R}^{m\times n}_{\geq 0}$  we want to find matrices $W\in\mathbb{R}^{m\times r}_{\geq 0}$ and $H\in\mathbb{R}^{r\times n}_{\geq 0}$  such that $D\approx WH$. The optimization problem can be written as 

$$ W^{\ast}, H^{\ast} = \arg\min_{W,H} \|D-WH\|_{F}^2 \quad \text{s.t.} \quad W\in\mathbb{R}^{m\times r},~~ W\geq 0 \text{~~and~~} H \in \mathbb{R}^{r\times n},~~ H\geq 0.$$

### An iterative algorithm
The following algorithm, called Lee and Seung's [multiplicative update rule](https://en.wikipedia.org/wiki/Multiplicative_Weight_Update_Method "Multiplicative Weight Update Method"), solves the  NMF problem. 
- initialize: **W** and **H** non negative
- repeat until convergence:

$${\displaystyle \mathbf {H} _{[i,j]}^{n+1}\leftarrow \mathbf {H} _{[i,j]}^{n}{\frac {((\mathbf {W} ^{n})^{T}\mathbf {V} )_{[i,j]}}{((\mathbf {W} ^{n})^{T}\mathbf {W} ^{n}\mathbf {H} ^{n})_{[i,j]}}}}$$

and

$${\displaystyle \mathbf {W} _{[i,j]}^{n+1}\leftarrow \mathbf {W} _{[i,j]}^{n}{\frac {(\mathbf {V} (\mathbf {H} ^{n+1})^{T})_{[i,j]}}{(\mathbf {W} ^{n}\mathbf {H} ^{n+1}(\mathbf {H} ^{n+1})^{T})_{[i,j]}}}}.$$

One drawback of this algorithm is that when an entry of $W$ or $H$ is initialized as zero or positive, it remains zero or positive throughout the iterations. Therefore, all entries should be initialized to be positive and sparsity constraints are not simply added to the algorithm unless via hard-thresholding. 

### Alternating non-negative least squares

Another algorithmic solution can be achieved via alternating non-negative least squares (ANLS). Assume we have $W$ fixed then we get the following optimization for $H$:

$$\arg\min_{H} \|D-WH\|_{F}^2 \quad \text{s.t.} \quad H \in \mathbb{R}^{r\times n},\quad H\geq 0,$$

which is the least squares problem for $H$ with a non-negativity constraint. The solution to this problem can be determined by solving $W^\top W H=W^\top D$ for $H$ followed by a projection onto $[0,1]$. We can also use the method proposed in [this paper](https://ieeexplore.ieee.org/document/4781130) to solve this problem. The code can be fount [here](http://www.cc.gatech.edu/~hpark/software/nmf_bpas.zip).  We need `nnlsm_blockpivot.m` and `solveNormalEqComb.m` files from the link. Now for fixed $H$ we can write:

$$\arg\min_{W} \|D^\top-H^\top W^\top\|_{F}^2 \quad \text{s.t.} \quad W \in \mathbb{R}^{m\times r},\quad W\geq 0,$$

This can be minimized by solving $H H^\top W^\top = H D^\top$ for $W$, followed by the projection onto $[0,1]$. It is also recommended that we normalize the columns of $W$ to unit $\ell_2$-norm and scale the rows of $H$ accordingly after each iteration. Using changes in $\|D-WH\|_F$ per iteration, we can define the stopping criteria.

## Matlab implementation
The matlab implementation is provided [here](https://github.com/amirhkhalilian/RayleighRiceModel).
