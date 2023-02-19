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

$$ W^{\ast}, H^{\ast} = \arg\min_{W,H} \|D-WH\|_{F}^2 \quad \text{s.t.} \quad W\in\mathbb{R}^{m\times r},\quad W\geq 0 \text{ and } H \in \mathbb{R}^{r\times n},\quad H\geq 0.$$

### An iterative algorithm
The following algorithm, called Lee and Seung's [multiplicative update rule](https://en.wikipedia.org/wiki/Multiplicative_weight_update_method), solves the  NMF problem. 
- initialize: **W** and **H** with random non-negative elements
- repeat until convergence:

$${\displaystyle \mathbf {H} _{[i,j]}^{n+1}\leftarrow \mathbf {H} _{[i,j]}^{n}{\frac {((\mathbf {W} ^{n})^{T}\mathbf {V} )_{[i,j]}}{((\mathbf {W} ^{n})^{T}\mathbf {W} ^{n}\mathbf {H} ^{n})_{[i,j]}}}}$$

$${\displaystyle \mathbf {W} _{[i,j]}^{n+1}\leftarrow \mathbf {W} _{[i,j]}^{n}{\frac {(\mathbf {V} (\mathbf {H} ^{n+1})^{T})_{[i,j]}}{(\mathbf {W} ^{n}\mathbf {H} ^{n+1}(\mathbf {H} ^{n+1})^{T})_{[i,j]}}}}.$$

One drawback of this algorithm is that when an entry of $W$ or $H$ is initialized as zero or positive, it remains zero or positive throughout the iterations. Therefore, all entries should be initialized to be positive and sparsity constraints are not simply added to the algorithm unless via hard-thresholding. 

### Alternating non-negative least squares algorithm

Another algorithmic solution can be achieved via alternating non-negative least squares (ANLS). Assume we have $W$ fixed then we get the following optimization for $H$:

$$\arg\min_{H} \|D-WH\|_{F}^2 \quad \text{s.t.} \quad H \in \mathbb{R}^{r\times n},\quad H\geq 0,$$

which is the least squares problem for $H$ with a non-negativity constraint. The solution to this problem can be determined by solving $W^\top W H=W^\top D$ for $H$ followed by a projection onto $\mathbb{R}_{\geq 0}$. We can also use the method proposed in [this paper](https://ieeexplore.ieee.org/document/4781130) to solve this problem. The code can be found [here](http://www.cc.gatech.edu/~hpark/software/nmf_bpas.zip). Now for fixed $H$ we can write:

$$\arg\min_{W} \|D^\top-H^\top W^\top\|_{F}^2 \quad \text{s.t.} \quad W \in \mathbb{R}^{m\times r},\quad W\geq 0,$$

This can be minimized by solving $H H^\top W^\top = H D^\top$ for $W$, followed by the projection onto $\mathbb{R}_{\geq 0}$. It is also recommended that we normalize the columns of $W$ to unit $\ell_2$-norm and scale the rows of $H$ accordingly after each iteration. Using changes in $\|D-WH\|_F$ per iteration, we can define the stopping criteria.

## Masked non-negative matrix factorization
In real application, we might have a case where the columns of $D$ represent different data-points and each row represents a separate features. In this case, we might have a situation where some of the features are missing from the dataset. In the NMF formulation, we do not want to consider the error over these missing values. In other words, we only wanna consider the error on the elements in $D$ that are valid or measured and discard the error on the rest. It is fair to assume that we know which elements are missing and use this information when formulating the problem.

Let $M\in\{0,1\}^{m\times n}$  be the masking matrix such that $M_{ij}=0$ iff $D_{ij}$ is missing, otherwise $M_{ij}=1$. In this case we only want to consider the error on measured signals. So we get,

$$ W^{\ast}, H^{\ast} = \arg\min_{W,H} \|M\circ(D-WH)\|_{F}^2 \quad \text{s.t.} \quad W\in\mathbb{R}^{m\times r},\quad W\geq 0 \text{ and } H \in \mathbb{R}^{r\times n},\quad H\geq 0,$$

where $\circ$ denotes the element-wise multiplication. The loss function in this case only considers the error in elements of $D$ where $M$ is non-zero.

### How do we add the Masking to an iterative solver?
We can apply the ANLS algorithm to the masked problem. When we fix $W$ we get the following optimization over $H$,

$$\arg\min_{H} \|M\circ(D-WH)\|_{F}^2 \quad \text{s.t.} \quad H \in \mathbb{R}^{r\times n},\quad H\geq 0.$$

Following the ANLS method we have to solve $W^\top M\circ(W H)=W^\top (M\circ D)$ for $H$ followed by a projection onto $\mathbb{R}_{\geq 0}$. The derivation [here](http://alexhwilliams.info/itsneuronalblog/2018/02/26/censored-lstsq/) shows the solution without the non-negative constraint. Let $M=[m_1,\cdots,m_n]$ where $m_j$ is the $j$-th column of $M$. Then, these equations can be solved for each column of  $H$ separately by solving the linear system $W^\top \mathrm{diag}(m_j) W\,h_j=W^\top \mathrm{diag}(m_j)\,d_j$  for $h_j$. We note that this can still be solved by `nnlsm_blockpivot.m` method from [this paper](https://ieeexplore.ieee.org/document/4781130). When an element in $m_j$ is zero,  we remove the corresponding row from the linear system (technically there is no need to do this and we can directly form $W^\top \mathrm{diag}(m_j) W$ and $W^\top \mathrm{diag}(m_j)\,d_j$ and solve the linear system, however, by removing the zeros we make sure that $W^\top \mathrm{diag}(m_j) W$ does not become unnecessarily ill-conditioned).

Now for fixed $H$ we can write:

$$\arg\min_{W} \|M^\top\circ(D^\top-H^\top W^\top)\|_{F}^2 \quad \text{s.t.} \quad W \in \mathbb{R}^{m\times r},\quad W\geq 0,$$

which is solved by the system of equations $H\,M^\top \circ (H^\top W^\top) = H D^\top$ for $W$ followed by a projection onto $\mathbb{R}_{\geq 0}$. Similarly, this can be diagonalized for rows of the matrix $W$ and solved via the same method.

## Implementation in Matlab
I implemented this algorithm in Matlab and you can find the code [here](https://github.com/amirhkhalilian/masked-nnmf). The core solver is the [`masked-nnmf`](https://github.com/amirhkhalilian/masked-nnmf/blob/main/solvers/masked_nnmf.m) function. Here is a brief overview of the main iteration.

The solver function receives three main inputs:
- `D`: the data matrix of size `m x n`.
- `M`: the mask matrix of size `m x n` with elements `Mij = 0` iff `Dij` is missing.
- `r`: desired rank.

The main loop consists of an update over $H$ followed by an update over $W$. Given the matrix `M` we can precompute the rows in $W_j$ and $d_j$ that we need to use when solving for $h_j$. Similarly, we find columns to use in each row of $d_i$ and $H_i$ when solving for rows  $w_i$. We write the  update iteration as:

```matlab
% update H
WT = W';
parfor (j = 1:size(D,2), num_cores) % loop over columns of D
	WT_mj_dj = transpose(sum(W(rows2use{j},:).*D(rows2use{j},j),1));
	WT_mj_W = WT(:,rows2use{j}) * W(rows2use{j},:);
	H(:,j) = nnlsm_blockpivot(WT_mj_W, WT_mj_dj, true);
end

% update W
HT = H';
parfor (i = 1:size(D,1), num_cores) % loop over rows of D
	H_mi_di = sum(H(:,cols2use{i}).*D(i,cols2use{i}),2);
	H_mi_H = H(:,cols2use{i}) * HT(cols2use{i},:);
	W(i,:) = nnlsm_blockpivot(H_mi_H, H_mi_di, true);
end
```
