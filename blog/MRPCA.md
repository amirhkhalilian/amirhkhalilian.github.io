+++
title = "Overlaying Model for Video Foreground-Background Subtraction"
date = Date(2020, 08, 01)
hasmath = true
tags = ["research", "signal processing"]
mintoclevel = 2
maxtoclevel = 2
descr = """
We forgo the additive assumption and instead propose a formulation of an overlaying model, which acknowledges that the foreground object is overlaid on top of the background and is occluding it
"""
+++

# {{ title }}

\toc

## Abstract
Moving object detection in a given video sequence is a pivotal step in many computer vision applications such as video surveillance. Robust Principal Component Analysis (RPCA) performs low-rank and sparse decomposition to accomplish such a task. A fundamental issue with the RPCA is the assumption that the low-rank and sparse components are added at each pixel, whereas in reality, the moving foreground is overlaid on the background. We propose the masked decomposition (i.e. an overlaying model) where each element either belongs to the low-rank or the sparse component, decided by a mask.  We introduce the Masked-RPCA (MRPCA) algorithm to recover the mask (hence the sparse object) and the low-rank components simultaneously, via a non-convex formulation. An adapted version of the Douglas--Rachford splitting algorithm is utilized to solve the proposed formulation.

## Additive model
Let us consider the matrix $M\in\mathbb{R}^{mn\times k}$ constructed by vectorizing the video frames of size $m\times n$ in to $mn$-vectors and stacking them as columns of the matrix $M$. Robust Principal Component Analysis (RPCA) attempts to efficiently find matrices $L$ and $S$ such that $M=L+S$ with $L$ as the low-rank and $S$ as the sparse component.The foreground-background subtraction via RPCA model can be achieved by solving the convex optimization problem as
$$
    \underset{L,S}{\mathrm{minimize}} \quad \|L\|_\ast + \lambda \|S\|_1 \quad\mathrm{subject~to~}\quad M = L+S
$$
with variables $L$ and $S$, where $\|X\|_\ast$ denotes the nuclear norm (sum of singular values), $\|X\|_1$ denotes the sum of absolute value of the entries (not the $\ell_1$-norm in the matrix sense), and $\lambda$ is a regularization parameter. This formulation is also known as principal component pursuit (PCP) method in the literature and can be solved by utilizing the augmented Lagrange multiplier (ALM) method.
\figenv{}{/assets/blog/MRPCA/vidTOmat.png}{width:85%}
\figenv{}{/assets/blog/MRPCA/MLS.png}{width:85%}

## Overlaying model
Additive perturbation is not a realistic assumption in video foreground-background decomposition. One immediate consequence of the additive model is that the recovered foreground object would not have the correct color. We forgo the additive assumption and instead propose a formulation of an overlaying model, which acknowledges that the foreground object is overlaid on top of the background and is occluding it (rather than simply being added).

The goal is to find matrices $L\in \mathbb{R}^{mn\times k}$ (with low-rank structure) and $W\in\{0,1\}^{mn \times k}$ (with sparse structure) such that the overlaying model is satisfied. A plausible formulation of such problem can be written as
$$
\underset{L,W}{\mathrm{minimize}}\quad \mathrm{rank}(L)+ \lambda \|W\|_0 + \iota_{[0,1]}\left(W\right)+ \frac{\rho}{2} \left\|(W-1)\circ(L-M)\right\|_F^2
$$
where $\lambda$ is a regularization parameter and $\rho$ is a penalty parameter. The indicator function $\iota_{[0,1]}\left(W\right)$ ensures $W \in [0,1]^{mn\times k}$. The algorithm to solve this problem is not immediately obvious. The details of how this probelm is solved can be found in the related [paper](https://ieeexplore.ieee.org/abstract/document/9264716).

## Sample results
A frame of the resulting background and mask for different variants of the overlaying model are shown in this figure. Additionally, last row shows the duality gap between the split variables and is an indicator for the convergence of the algorithm showing that all proposed methods converge within few tens of iterations. Histogram of the recovered mask for all different methods is shown in following figure. As we can observe, the non-convex priors lead to a separation of the $\{0,1\}$ values.

\figenv{}{/assets/blog/MRPCA/StatCompRankLnuc.png}{width:85%}

## Matlab implementation
The matlab implementation is provided [here](https://github.com/amirhkhalilian/Masked-RPCA).

## Related publications

* ~~~<u>A. Khalilian-Gourtani</u>~~~, S. Minaee, and Y. Wang. *"Masked-RPCA: Moving Object Detection with an Overlaying Model"*, IEEE Open Journal of Signal Processing, vol. 1, 2020. [pdf](https://ieeexplore.ieee.org/abstract/document/9264716)

* ~~~<u>A. Khalilian-Gourtani</u>~~~, S. Minaee, and Y. Wang. *"Masked-RPCA: Sparse and low-rank decomposition under overlaying model and application to moving object detection."* arXiv preprint, 2019. [pdf](https://arxiv.org/abs/1909.08049)
